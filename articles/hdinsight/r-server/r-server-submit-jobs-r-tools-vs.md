---
title: "R Tools for Visual Studio からのジョブの送信 - Azure HDInsight | Microsoft Docs"
description: "ローカルの Visual Studio マシンから HDInsight クラスターに R ジョブを送信します。"
services: hdinsight
documentationcenter: 
author: maxluk
manager: jhubbard
editor: cgronlun
tags: azure-portal
ms.service: hdinsight
ms.custom: hdinsightactive
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/11/2018
ms.author: maxluk
ms.openlocfilehash: 1a82ba7790f739768156a8bee33a74d7130e24e1
ms.sourcegitcommit: be9a42d7b321304d9a33786ed8e2b9b972a5977e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/19/2018
---
# <a name="submit-jobs-from-r-tools-for-visual-studio"></a>R Tools for Visual Studio からのジョブの送信

[R Tools for Visual Studio](https://www.visualstudio.com/vs/rtvs/) (RTVS) は、[Visual Studio 2017](https://www.visualstudio.com/downloads/) と [Visual Studio 2015 Update 3](http://go.microsoft.com/fwlink/?LinkId=691129) 以降の両方の Community Edition (無料)、Professional Edition、Enterprise Edition 向けの無料のオープンソース拡張機能です。

RTVS は、[R インタラクティブ ウィンドウ](https://docs.microsoft.com/visualstudio/rtvs/interactive-repl) (REPL)、IntelliSense (コード補完)、ggplot2 や ggviz などの R ライブラリを介した[プロットの視覚化](https://docs.microsoft.com/visualstudio/rtvs/visualizing-data)、[R コードのデバッグ](https://docs.microsoft.com/visualstudio/rtvs/debugging)などの多彩なツールを提供することにより、R ワークフローを強化します。

## <a name="set-up-your-environment"></a>環境をセットアップする

1. [R Tools for Visual Studio](https://docs.microsoft.com/visualstudio/rtvs/installation) をインストールします。

    ![Visual Studio 2017 での RTVS のインストール](./media/r-server-submit-jobs-r-tools-vs/install-r-tools-for-vs.png)

2. *[データ サイエンスと分析のアプリケーション]* ワークロードを選択し、**[R 言語サポート]**、**[R 開発ツールのランタイム サポート]**、**[Microsoft R Client ]** の各オプションを選択します。

3. SSH 認証のための公開キーと秘密キーを用意します。
<!-- {TODO tbd, no such file yet}[use SSH with HDInsight](hdinsight-hadoop-linux-use-ssh-windows.md) -->

4. マシンに [Microsoft R Server](https://msdn.microsoft.com/microsoft-r/rserver-install-windows) をインストールします。 Microsoft R Server により、[`RevoScaleR`](https://msdn.microsoft.com/microsoft-r/scaler/scaler) 関数と `RxSpark` 関数が提供されます。

5. ローカル クライアントから HDInsight クラスターに対して `RevoScaleR` 関数を実行するための計算コンテキストを提供する、[PuTTY](http://www.putty.org/) をインストールします。

6. Visual Studio 環境にデータ サイエンスの設定を適用して、R Tools のワークスペースに新しいレイアウトを提供するオプションがあります。
    1. 現在の Visual Studio 設定を保存するには、**[ツール] > [設定のインポートとエクスポート]** コマンドを使用し、**[選択された環境設定をエクスポート]** を選択してファイル名を指定します。 それらの設定を復元するには、同じコマンドを使用し、**[選択された環境設定をインポート]** を選択します。

    2. **[R Tools]** メニュー項目に移動して、**[データ サイエンスの設定...]** を選択します。

        ![[データ サイエンスの設定...]](./media/r-server-submit-jobs-r-tools-vs/data-science-settings.png)

    > [!NOTE]
    > 手順 1 のアプローチを使用して、**[データ サイエンスの設定]** コマンドを繰り返さずに、パーソナライズされたデータ サイエンティストのレイアウトを保存および復元することもできます。

## <a name="execute-local-r-methods"></a>ローカル R メソッドを実行する

1. [Microsoft R Server HDInsight クラスター](r-server-get-started.md)を作成します。
2. [RTVS 拡張機能](https://docs.microsoft.com/visualstudio/rtvs/installation)をインストールします。
3. [サンプル zip ファイル](https://github.com/Microsoft/RTVS-docs/archive/master.zip)をダウンロードします。
4. `examples/Examples.sln` を開いて、Visual Studio でソリューションを起動します。
5. `A first look at R` ソリューション フォルダー内の `1-Getting Started with R.R` ファイルを開きます。
6. ファイルの先頭から、R Ctrl + Enter キーを押して、1 回に 1 行ずつを R インタラクティブ ウィンドウに送信します。 一部の行では、パッケージをインストールするために時間がかかる場合があります。
    * または、Ctrl + A キー を押して R ファイルのすべての行を選択した後で、Ctrl + Enter キー を押してすべて実行するか、ツール バーで [Execute Interactive]\(対話型で実行\) アイコンを選択します。
        ![[Execute interactive]\(対話形式で実行\)](./media/r-server-submit-jobs-r-tools-vs/execute-interactive.png)

7. スクリプトのすべての行を実行すると、次のような出力が表示されます。

    ![[データ サイエンスの設定...]](./media/r-server-submit-jobs-r-tools-vs/workspace.png)

## <a name="submit-jobs-to-an-hdinsight-r-cluster"></a>HDInsight R クラスターにジョブを送信する

PuTTY が搭載された Windows コンピューターから Microsoft R Server/Microsoft R Client を使用して、ローカル クライアントから HDInsight クラスターに対して分散された `RevoScaleR` 関数を実行する計算コンテキストを作成できます。 `RxSpark` を使用して計算コンテキストを作成し、ユーザー名、Hadoop クラスターのエッジ ノード、SSH スイッチなどを指定します。

1. エッジ ノードのホスト名を調べるには、Azure で HDInsight R クラスターのウィンドウを開き、[概要] ウィンドウの上部のメニューで **[SSH (Secure Shell)]** を選択します。

    ![[SSH (Secure Shell)]](./media/r-server-submit-jobs-r-tools-vs/ssh.png)

2. **[エッジ ノードのホスト名]** の値をコピーします。

    ![[エッジ ノードのホスト名]](./media/r-server-submit-jobs-r-tools-vs/edge-node.png)

3. Visual Studio の [R インタラクティブ] ウィンドウに次のコードを貼り付け、セットアップ変数の値を環境に合わせて変更します。

    ```R
    # Setup variables that connect the compute context to your HDInsight cluster
    mySshHostname <- 'r-cluster-ed-ssh.azurehdinsight.net ' # HDI secure shell hostname
    mySshUsername <- 'sshuser' # HDI SSH username
    mySshClientDir <- "C:\\Program Files (x86)\\PuTTY"
    mySshSwitches <- '-i C:\\Users\\azureuser\\r.ppk' # Path to your private ssh key
    myHdfsShareDir <- paste("/user/RevoShare", mySshUsername, sep = "/")
    myShareDir <- paste("/var/RevoShare", mySshUsername, sep = "/")
    mySshProfileScript <- "/usr/lib64/microsoft-r/3.3/hadoop/RevoHadoopEnvVars.site"

    # Create the Spark Cluster compute context
    mySparkCluster <- RxSpark(
          sshUsername = mySshUsername,
      sshHostname = mySshHostname,
      sshSwitches = mySshSwitches,
      sshProfileScript = mySshProfileScript,
      consoleOutput = TRUE,
      hdfsShareDir = myHdfsShareDir,
      shareDir = myShareDir,
      sshClientDir = mySshClientDir
    )
    
    # Set the current compute context as the Spark compute context defined above
    rxSetComputeContext(mySparkCluster)
    ```
    
    ![Spark コンテキストの設定](./media/r-server-submit-jobs-r-tools-vs/spark-context.png)

4. [R インタラクティブ] ウィンドウで次のコマンドを実行します。

    ```R
    rxHadoopCommand("version") # should return version information
    rxHadoopMakeDir("/user/RevoShare/newUser") # creates a new folder in your storage account
    rxHadoopCopy("/example/data/people.json", "/user/RevoShare/newUser") # copies file to new folder
    ```

    次のような出力が表示されます。

    ![正常に実行された rx コマンド](./media/r-server-submit-jobs-r-tools-vs/rx-commands.png)

5. `rxHadoopCopy` によって、サンプル データ フォルダーの `people.json` ファイルが新しく作成された `/user/RevoShare/newUser` フォルダーに正常にコピーされたことを確認します。

    1. Azure の HDInsight R クラスターのウィンドウで、左側のメニューから **[ストレージ アカウント]** を選択します。

        ![ストレージ アカウント](./media/r-server-submit-jobs-r-tools-vs/storage-accounts.png)

    2. クラスターの既定のストレージ アカウントを選択し、コンテナー/ディレクトリ名をメモします。

    3. ストレージ アカウントのウィンドウの左側のメニューで **[コンテナー]** を選択します。

        ![コンテナー](./media/r-server-submit-jobs-r-tools-vs/containers.png)

    4. クラスターのコンテナー名を選択し、**[user]** フォルダーを参照して (一覧の一番下の *[さらに読み込む]* のクリックが必要な場合があります)、*[RevoShare]*、**[newUser]** の順に選択します。 `newUser` フォルダーに `people.json` ファイルが表示されます。

        ![コピーされたファイル](./media/r-server-submit-jobs-r-tools-vs/copied-file.png)

6. 現在の Spark コンテキストの使用が完了したら、コンテキストを停止する必要があります。 複数のコンテキストを同時に実行することはできません。

    ```R
    rxStopEngine(mySparkCluster)
    ```

## <a name="next-steps"></a>次の手順

* [HDInsight の R Server (プレビュー) の計算コンテキストのオプション](r-server-compute-contexts.md)
* 「[HDInsight で ScaleR と SparkR を組み合わせる](../hdinsight-hadoop-r-scaler-sparkr.md)」では、航空会社のフライトの遅延を予測する例が示されています。
<!-- * You can also submit R jobs with the [R Studio Server](hdinsight-submit-jobs-from-r-studio-server.md) -->
