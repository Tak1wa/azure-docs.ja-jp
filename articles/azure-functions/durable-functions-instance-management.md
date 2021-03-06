---
title: Durable Functions でのインスタンスの管理 - Azure
description: Azure Functions の Durable Functions 拡張機能でインスタンスを管理する方法を説明します。
services: functions
author: cgillum
manager: cfowler
editor: ''
tags: ''
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 09/29/2017
ms.author: azfuncdf
ms.openlocfilehash: 9cea9b18cd7434a34138d5cecad8a8fd7f10d2e5
ms.sourcegitcommit: 8aab1aab0135fad24987a311b42a1c25a839e9f3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/16/2018
---
# <a name="manage-instances-in-durable-functions-azure-functions"></a>Durable Functions でのインスタンスの管理 (Azure Functions)

[Durable Functions](durable-functions-overview.md) オーケストレーション インスタンスは、開始、終了、照会したり、通知イベントを送信したりできます。 インスタンス管理はすべて、[オーケストレーション クライアント バインド](durable-functions-bindings.md)を使用して行います。 この記事では、各インスタンスの管理操作について詳しく説明します。

## <a name="starting-instances"></a>インスタンスの開始

[DurableOrchestrationClient](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html) の [StartNewAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_StartNewAsync_) メソッドは、オーケストレーター関数の新しいインスタンスを開始します。 このクラスのインスタンスを取得するには、`orchestrationClient` バインドを使用します。 内部的には、このメソッドは、メッセージをコントロール キューにエンキューし、これにより `orchestrationTrigger` トリガー バインドを使用する、指定された名前の関数がトリガーされます。 

タスクが完了するのは、オーケストレーション プロセスが開始するときです。 オーケストレーション プロセスは30 秒以内に開始する必要があります。 これよりも長くかかると `TimeoutException` がスローされます。 

[StartNewAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_StartNewAsync_) のパラメーターは次のとおりです。

* **Name**: スケジュールを設定するオーケストレーター関数の名前。
* **Input**: オーケストレーター関数に入力として渡す必要がある、JSON でシリアル化できるデータ。
* **InstanceId**: (省略可能) インスタンスの一意の ID。 指定しない場合は、ランダムなインスタンス ID が生成されます。

単純な C# の例を次に示します。

```csharp
[FunctionName("HelloWorldManualStart")]
public static async Task Run(
    [ManualTrigger] string input,
    [OrchestrationClient] DurableOrchestrationClient starter,
    TraceWriter log)
{
    string instanceId = await starter.StartNewAsync("HelloWorld", input);
    log.Info($"Started orchestration with ID = '{instanceId}'.");
}
```

.NET 以外の言語では、関数の出力バインドを使用して、新しいインスタンスを開始することもできます。 この場合、上記の 3 つのパラメーターをフィールドとして持つ、JSON でシリアル化できる任意のオブジェクトを使用できます。 たとえば、次の Node.js 関数を考えてみましょう。

```js
module.exports = function (context, input) {
    var id = generateSomeUniqueId();
    context.bindings.starter = [{
        FunctionName: "HelloWorld",
        Input: input,
        InstanceId: id
    }];

    context.done(null);
};
```

> [!NOTE]
> インスタンス ID にはランダムな識別子を使用することをお勧めします。 これにより、複数の VM にわたってオーケストレーター関数をスケールするとき、負荷が均等に分散されるようになります。 ランダムではないインスタンスの ID は、外部ソースから ID を取得するとき、または[シングルトン オーケストレーター](durable-functions-singletons.md) パターンを実装するときに使用します。

## <a name="querying-instances"></a>インスタンスの照会

[DurableOrchestrationClient](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html) クラスの [GetStatusAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_GetStatusAsync_) メソッドは、オーケストレーション インスタンスの状態を照会します。 これは、パラメーターとして `instanceId` (必須)、`showHistory` (省略可能)、`showHistoryOutput` (省略可能) を使用します。 `showHistory` を `true` に設定すると、応答に実行履歴が含まれます。 `showHistoryOutput` も `true` に設定すると、実行履歴にアクティビティ出力が含まれます。 このメソッドは次のプロパティを持つオブジェクトを返します。

* **Name**: オーケストレーター関数の名前。
* **InstanceId**: オーケストレーションのインスタンス ID (`instanceId` 入力と同じにする必要があります)。
* **CreatedTime**: オーケストレーター関数の実行が開始された時刻。
* **LastUpdatedTime**: オーケストレーションが最後にチェックポイントされた時刻。
* **Input**: JSON 値としての関数の入力。
* **Output**: JSON 値として関数の出力 (関数が完了している場合)。 オーケストレーター関数が失敗した場合、このプロパティには、エラーの詳細が含まれます。 オーケストレーター関数が終了した場合、このプロパティには、提供されている終了の理由が含まれます (存在する場合)。
* **RuntimeStatus**: 次のいずれかの値を指定できます。
    * **Running**: インスタンスの実行が開始されました。
    * **Completed**: インスタンスが正常に完了しました。
    * **ContinuedAsNew**: インスタンスが新しい履歴で再起動されました。 これは遷移状態です。
    * **Failed**: インスタンスが失敗し、エラーが発生しました。
    * **Terminated**: インスタンスが突然終了しました。
* **History**: オーケストレーションの実行履歴。 このフィールドにデータが設定されるのは、`showHistory` を `true` に設定した場合のみです。
    
インスタンスが存在しないか、まだ開始されていない場合、このメソッドは `null` を返します。

```csharp
[FunctionName("GetStatus")]
public static async Task Run(
    [OrchestrationClient] DurableOrchestrationClient client,
    [ManualTrigger] string instanceId)
{
    var status = await client.GetStatusAsync(instanceId);
    // do something based on the current status.
}
```

> [!NOTE]
> インスタンスのクエリは、現在、C# オーケストレーター関数でのみサポートされます。

## <a name="terminating-instances"></a>インスタンスの終了

実行中のインスタンスを終了するには、[DurableOrchestrationClient](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html) クラスの [TerminateAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_TerminateAsync_) メソッドを使用します。 2 つのパラメーター `instanceId` と `reason` 文字列は、ログとインスタンス状態に書き込まれます。 終了インスタンスは、次の `await` ポイントに到達するとすぐに実行が停止されます。また、すでに `await` にある場合は、直ちに終了します。

```csharp
[FunctionName("TerminateInstance")]
public static Task Run(
    [OrchestrationClient] DurableOrchestrationClient client,
    [ManualTrigger] string instanceId)
{
    string reason = "It was time to be done.";
    return client.TerminateAsync(instanceId, reason);
}
```

> [!NOTE]
> インスタンスの終了は、現在、C# オーケストレーター関数でのみサポートされます。

## <a name="sending-events-to-instances"></a>インスタンスへのイベントの送信

実行中のインスタンスにイベント通知を送信するには、[DurableOrchestrationClient](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html) クラスの [RaiseEventAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_RaiseEventAsync_) メソッドを使用します。 こうしたイベントを処理できるのは、[WaitForExternalEvent](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_WaitForExternalEvent_) への呼び出しを待っているインスタンスです。 

[RaiseEventAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_RaiseEventAsync_) のパラメーターは次のとおりです。

* **InstanceId**: インスタンスの一意の ID。
* **EventName**: 送信するイベントの名前。
* **EventData**: インスタンスに送信する JSON でシリアル化できるペイロード。

```csharp
#r "Microsoft.Azure.WebJobs.Extensions.DurableTask"

[FunctionName("RaiseEvent")]
public static Task Run(
    [OrchestrationClient] DurableOrchestrationClient client,
    [ManualTrigger] string instanceId)
{
    int[] eventData = new int[] { 1, 2, 3 };
    return client.RaiseEventAsync(instanceId, "MyEvent", eventData);
}
```

> [!NOTE]
> イベントの生成は、現在、C# オーケストレーター関数でのみサポートされます。

> [!WARNING]
> 指定した "*インスタンス ID*" のオーケストレーション インスタンスが存在しない場合、または指定した "*イベント名*" でインスタンスが待機していない場合、イベント メッセージは破棄されます。 この動作の詳細については、[GitHub の問題](https://github.com/Azure/azure-functions-durable-extension/issues/29)に関するトピックをご覧ください。

## <a name="wait-for-orchestration-completion"></a>オーケストレーションの完了の待機

[DurableOrchestrationClient](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html) クラスは、オーケストレーション インスタンスから実際の出力を同期して取得するために使用できる [WaitForCompletionOrCreateCheckStatusResponseAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_WaitForCompletionOrCreateCheckStatusResponseAsync_) API を公開します。 このメソッドでは、`timeout` に 10 秒および `retryInterval` に 1 秒の既定値を使用します (設定されていない場合)。  

この API の使用方法を示す HTTP トリガー関数の例を次に示します。

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HttpSyncStart.cs)]

この関数は、次のように 2 秒のタイムアウトと 0.5 秒の再試行間隔を使用して呼び出すことができます。

```bash
    http POST http://localhost:7071/orchestrators/E1_HelloSequence/wait?timeout=2&retryInterval=0.5
```

オーケストレーション インスタンスからの応答を取得するために必要な時間に応じて 2 つのケースがあります。

1. オーケストレーション インスタンスが、定義されたタイムアウト (このケースでは 2 秒) 内に完了すると、応答は、同期して提供される実際のオーケストレーション インスタンス出力です。

    ```http
        HTTP/1.1 200 OK
        Content-Type: application/json; charset=utf-8
        Date: Thu, 14 Dec 2017 06:14:29 GMT
        Server: Microsoft-HTTPAPI/2.0
        Transfer-Encoding: chunked

        [
            "Hello Tokyo!",
            "Hello Seattle!",
            "Hello London!"
        ]
    ```

2. オーケストレーション インスタンスが、定義されたタイムアウト (このケースでは 2 秒) 内に完了できない場合、応答は「**HTTP API URL の検出**」で説明されている既定のものになります。

    ```http
        HTTP/1.1 202 Accepted
        Content-Type: application/json; charset=utf-8
        Date: Thu, 14 Dec 2017 06:13:51 GMT
        Location: http://localhost:7071/admin/extensions/DurableTaskExtension/instances/d3b72dddefce4e758d92f4d411567177?taskHub={taskHub}&connection={connection}&code={systemKey}
        Retry-After: 10
        Server: Microsoft-HTTPAPI/2.0
        Transfer-Encoding: chunked

        {
            "id": "d3b72dddefce4e758d92f4d411567177",
            "sendEventPostUri": "http://localhost:7071/admin/extensions/DurableTaskExtension/instances/d3b72dddefce4e758d92f4d411567177/raiseEvent/{eventName}?taskHub={taskHub}&connection={connection}&code={systemKey}",
            "statusQueryGetUri": "http://localhost:7071/admin/extensions/DurableTaskExtension/instances/d3b72dddefce4e758d92f4d411567177?taskHub={taskHub}&connection={connection}&code={systemKey}",
            "terminatePostUri": "http://localhost:7071/admin/extensions/DurableTaskExtension/instances/d3b72dddefce4e758d92f4d411567177/terminate?reason={text}&taskHub={taskHub}&connection={connection}&code={systemKey}"
        }
    ```

> [!NOTE]
> webhook URL の形式は、実行している Azure Functions ホストのバージョンによって異なる場合があります。 上記の例は、Azure Functions 2.0 ホスト用の形式です。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [インスタンス管理に HTTP API を使用する方法を確認する](durable-functions-http-api.md)
