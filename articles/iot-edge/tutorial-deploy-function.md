---
title: "Azure IoT Edge で Azure 関数を展開する | Microsoft Docs"
description: "Azure 関数をモジュールとしてエッジ デバイスに展開します"
services: iot-edge
keywords: 
author: kgremban
manager: timlt
ms.author: v-jamebr
ms.date: 11/15/2017
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
ms.openlocfilehash: 1dfe46d307a076ae02362c4bba292602001ed915
ms.sourcegitcommit: 42ee5ea09d9684ed7a71e7974ceb141d525361c9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2017
---
# <a name="deploy-azure-function-as-an-iot-edge-module---preview"></a>Azure 関数を IoT Edge モジュールとして展開する - プレビュー
Azure Functions を使用して、ビジネス ロジックを実装するコードを IoT Edge デバイスに直接展開できます。 このチュートリアルでは、[Windows][lnk-tutorial1-win] または [Linux][lnk-tutorial1-lin] のシミュレートされたデバイスに Azure IoT Edge を展開するチュートリアルで作成した、シミュレートされた IoT Edge デバイスで、センサー データをフィルター処理する Azure 関数を作成および展開します。 このチュートリアルで学習する内容は次のとおりです。     

> [!div class="checklist"]
> * Visual Studio Code を使用して、Azure 関数を作成する
> * VS Code と Docker を使用して Docker イメージを作成し、レジストリに発行する 
> * モジュールを IoT Edge デバイスに展開する
> * 生成されたデータを表示する


このチュートリアルで作成した Azure 関数は、デバイスによって生成される温度データをフィルター処理し、指定されたしきい値を温度が上回っているときにのみ、上流の Azure IoT Hub にメッセージを送信します。 

## <a name="prerequisites"></a>前提条件

* クイック スタートまたは前のチュートリアルで作成した Azure IoT Edge デバイス。
* [Visual Studio Code](https://code.visualstudio.com/)。 
* [Visual Studio Code 用の C# (OmniSharp を使用) 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)  
* [Visual Studio Code 用の Azure IoT Edge 拡張機能](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge)  
* [Docker](https://docs.docker.com/engine/installation/)。 このチュートリアルでは、プラットフォームの Community Edition (CE) で十分です。 
* [.NET Core 2.0 SDK](https://www.microsoft.com/net/core#windowscmd)。 

## <a name="create-a-container-registry"></a>コンテナー レジストリの作成
このチュートリアルでは、VS Code 用の Azure IoT Edge 拡張機能を使用してモジュールをビルドし、ファイルから**コンテナー イメージ**を作成します。 その後、このイメージを**レジストリ**にプッシュし、格納および管理します。 最後に、レジストリからイメージを展開し、IoT Edge デバイスで実行します。  

このチュートリアルでは、Docker と互換性のある任意のレジストリをご利用いただけます。 クラウドで使用できる 2 つの一般的な Docker レジストリ サービスは、[Azure Container Registry](https://docs.microsoft.com/azure/container-registry/) と [Docker Hub](https://docs.docker.com/docker-hub/repos/#viewing-repository-tags) です。 このチュートリアルでは、Azure Container Registry を使用します。 

1. [Azure Portal](https://portal.azure.com) で、**[リソースの作成]** > **[コンテナー]** > **[Azure Container Registry]** の順に選択します。
2. レジストリに名前を付けて、サブスクリプション、リソース グループを選択し、SKU を **[Basic]** に設定します。 
3. **[作成]**を選択します。
4. コンテナー レジストリが作成されたら、そこに移動し、**[アクセス キー]** を選択します。 
5. **[管理者ユーザー]** を **[有効]** に切り替えます。
6. **ログイン サーバー**、**ユーザー名**、および**パスワード**の値をコピーします。 これらの値を、このチュートリアルで後ほど使用します。 

## <a name="create-a-function-project"></a>関数プロジェクトを作成する
次の手順では、Visual Studio Code と Azure IoT Edge 拡張機能を使用して、IoT Edge 関数を作成する方法を説明します。
1. Visual Studio Code を開きます。
2. VS Code 統合ターミナルを開くには、**[表示]** > **[統合ターミナル]** を選択します。
3. dotnet で **AzureIoTEdgeFunction** テンプレートをインストール (または更新) するには、統合ターミナルで次のコマンドを実行します。

    ```cmd/sh
    dotnet new -i Microsoft.Azure.IoT.Edge.Function
    ```
2. 新しいモジュールのプロジェクトを作成します。 次のコマンドにより、現在の作業フォルダーに **FilterFunction** プロジェクト フォルダーが作成されます。

    ```cmd/sh
    dotnet new aziotedgefunction -n FilterFunction
    ```

3. **[ファイル]** > **[フォルダーを開く]** を選択し、**FilterFunction** フォルダーを参照して、VS Code でプロジェクトを開きます。
4. VS Code エクスプローラーで、**EdgeHubTrigger-Csharp** フォルダーを展開し、**run.csx** ファイルを開きます。
5. このファイルの内容を次のコードに置き換えます。

   ```csharp
   #r "Microsoft.Azure.Devices.Client"
   #r "Newtonsoft.Json"

   using System.IO;
   using Microsoft.Azure.Devices.Client;
   using Newtonsoft.Json;

   // Filter messages based on the temperature value in the body of the message and the temperature threshold value.
   public static async Task Run(Message messageReceived, IAsyncCollector<Message> output, TraceWriter log)
   {
        const int temperatureThreshold = 25;
        byte[] messageBytes = messageReceived.GetBytes();
        var messageString = System.Text.Encoding.UTF8.GetString(messageBytes);

        if (!string.IsNullOrEmpty(messageString))
        {
            // Get the body of the message and deserialize it
            var messageBody = JsonConvert.DeserializeObject<MessageBody>(messageString);

            if (messageBody != null && messageBody.machine.temperature > temperatureThreshold)
            {
                // Send the message to the output as the temperature value is greater than the threashold
                var filteredMessage = new Message(messageBytes);
                // Copy the properties of the original message into the new Message object
                foreach (KeyValuePair<string, string> prop in messageReceived.Properties)
                {
                    filteredMessage.Properties.Add(prop.Key, prop.Value);
                }
                // Add a new property to the message to indicate it is an alert
                filteredMessage.Properties.Add("MessageType", "Alert");
                // Send the message        
                await output.AddAsync(filteredMessage);
                log.Info("Received and transferred a message with temperature above the threshold");
            }
        }
    }

    //Define the expected schema for the body of incoming messages
    class MessageBody
    {
        public Machine machine {get;set;}
        public Ambient ambient {get; set;}
        public string timeCreated {get; set;}
    }
    class Machine
    {
       public double temperature {get; set;}
       public double pressure {get; set;}         
    }
    class Ambient
    {
       public double temperature {get; set;}
       public int humidity {get; set;}         
    }
   ```

11. ファイルを保存します。

## <a name="publish-a-docker-image"></a>Docker イメージを発行する

1. Docker イメージをビルドします。
    1. VS Code エクスプローラーで、**Docker** フォルダーを展開します。 次に、コンテナー プラットフォームのフォルダー、**linux-x64** または **windows-nano** を展開します。 
    2. **[Dockerfile]** ファイルを右クリックし、**[Build IoT Edge module Docker image] (IoT Edge モジュール Docker イメージのビルド)** をクリックします。 
    3. **FilterFunction** プロジェクト フォルダーに移動し、**[Select Folder as EXE_DIR]\(EXE_DIR としてフォルダーを選択\)** をクリックします。 
    4. VS Code ウィンドウの上部にあるポップアップ テキスト ボックスで、イメージの名前を入力します。 (例: `<your container registry address>/filterfunction:latest`)。 コンテナー レジストリ アドレスは、レジストリからコピーしたログイン サーバーと同じです。 `<your container registry name>.azurecr.io` の形式にする必要があります。
 
4. Docker にサインインします。 統合ターミナルで、次のコマンドを入力します。 

   ```csh/sh
   docker login -u <username> -p <password> <Login server>
   ```
        
   このコマンドで使用するユーザー名、パスワード、ログイン サーバーを見つけるには、Azure Portal (https://portal.azure.com) に移動します。 **[すべてのリソース]** から、Azure Container Registry のタイルをクリックして、プロパティを開き、**[アクセス キー]** をクリックします。 **ユーザー名**、**パスワード**、**ログイン サーバー**の各フィールドの値をコピーします。 

3. イメージを Docker リポジトリにプッシュします。 **[表示]** > **[コマンド パレット...]** を選択し、**[Edge: Push IoT Edge module Docker image]\(Edge: IoT Edge モジュール Docker イメージをプッシュ\)** を検索します。
4. ポップアップ テキスト ボックスに、手順 1.d. で使用したイメージ名を入力します。

## <a name="add-registry-credentials-to-your-edge-device"></a>レジストリ資格情報を Edge デバイスに追加する
Edge デバイスを実行しているコンピューターの Edge ランタイムに、レジストリの資格情報を追加します。 これにより、コンテナーを取得するためのランタイム アクセスが得られます。 

- Windows の場合は、次のコマンドを実行します。
    
    ```cmd/sh
    iotedgectl login --address <your container registry address> --username <username> --password <password> 
    ```

- Linux の場合は、次のコマンドを実行します。
    
    ```cmd/sh
    sudo iotedgectl login --address <your container registry address> --username <username> --password <password> 
    ```

## <a name="run-the-solution"></a>ソリューションを実行する

1. **Azure Portal** で、IoT ハブに移動します。
2. **IoT Edge (プレビュー)** に移動し、IoT Edge デバイスを選択します。
1. **[Set Modules] (モジュールの設定)** を選択します。 
2. **tempSensor** モジュールをこのデバイスに既に展開している場合は、自動的に設定されます。 そうでない場合は、次の手順に従ってモジュールを追加します。
    1. **[Add IoT Edge Module]\(IoT Edge モジュールの追加\)** を選びます。
    2. **[名前]** フィールドに「`tempSensor`」と入力します。
    3. **[イメージの URI]** フィールドに「`microsoft/azureiotedge-simulated-temperature-sensor:1.0-preview`」と入力します。
    4. 他の設定はそのままにして、**[保存]** をクリックします。
1. **filterFunction** モジュールを追加します。
    1. もう一度 **[Add IoT Edge Module] (IoT Edge モジュールの追加)** を選択します。
    2. **[名前]** フィールドに「`filterFunction`」と入力します。
    3. **[イメージ]** フィールドに、イメージ アドレスを入力します (`<docker registry address>/filterfunction:latest` など)。
    74. **[Save]** をクリックします。
2. **[次へ]** をクリックします。
3. **[Specify Routes] (ルートの指定)** の手順で、下記の JSON をテキスト ボックスにコピーします。 1 つ目のルートは、"input1" エンドポイント経由で、温度センサーからフィルター モジュールにメッセージを転送します。 2 つ目のルートは、フィルター モジュールから IoT ハブにメッセージを転送します。 このルートでは、`$upstream` は特別な転送先で、Edge Hub に対して、メッセージを IoT ハブに送信するように指示します。 

    ```json
    {
       "routes":{
          "sensorToFilter":"FROM /messages/modules/tempSensor/outputs/temperatureOutput INTO BrokeredEndpoint(\"/modules/filterFunction/inputs/input1\")",
          "filterToIoTHub":"FROM /messages/modules/filterFunction/outputs/* INTO $upstream"
       }
    }
    ```

4. **[次へ]** をクリックします。
5. **[Review Template] (テンプレートのレビュー)** の手順で、**[送信]** をクリックします。 
6. IoT Edge デバイスの詳細ページに戻り、**[更新]** をクリックします。 新しい **filterfunction** モジュールが、**tempSensor** モジュールおよび **IoT Edge ランタイム**と一緒に実行されています。 

## <a name="view-generated-data"></a>生成されたデータを表示する

IoT Edge デバイスから IoT ハブに送信されたデバイスからクラウドへのメッセージを監視するには:
1. Azure IoT ツールキット拡張機能を IoT ハブの接続文字列で構成します。 
    1. Azue Portal で IoT ハブに移動し、**[共有アクセス ポリシー]** を選択します。 
    2. **[iothubowner]** を選択し、**[接続文字列 - 主キー]** の値をコピーします。
    1. VS Code エクスプローラーで、**[IOT HUB DEVICES]\(IoT ハブ デバイス\)** をクリックし、**[...]** をクリックします。 
    1. **[Set IoT Hub Connection String]\(IoT ハブ接続文字列の設定\)** を選択し、ポップアップ ウィンドウに IoT ハブ接続文字列を入力します。 

1. IoT ハブに到着するデータを監視するには、**[表示]** > **[コマンド パレット...]** を選択し、**[IoT: Start monitoring D2C message]\(IoT: D2C メッセージの監視を開始\)** を検索します。 
2. データの監視を停止するには、コマンド パレットで **[IoT: Stop monitoring D2C message]\(IoT: D2C メッセージの監視を停止\)** を使用します。 

## <a name="next-steps"></a>次のステップ

このチュートリアルでは、IoT Edge デバイスで生成された生データをフィルター処理するコードを含む、Azure 関数を作成しました。 Azure IoT Edge を探索し続けるには、Azure IoT Edge デバイスをゲートウェイとして使用する方法を確認します。 

> [!div class="nextstepaction"]
> [IoT Edge ゲートウェイ デバイスを作成する](how-to-create-transparent-gateway.md)

<!--Links-->
[lnk-tutorial1-win]: tutorial-simulate-device-windows.md
[lnk-tutorial1-lin]: tutorial-simulate-device-linux.md
