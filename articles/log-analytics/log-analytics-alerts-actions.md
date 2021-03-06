---
title: "Azure Log Analytics のアラートへの応答 | Microsoft Docs"
description: "Log Analytics のアラートは、Azure ワークスペース内の重要な情報を識別し、問題について事前に通知したり、問題を修正するためのアクションを呼び出したりできます。  この記事では、アラート ルールを作成する方法と、実行できるさまざまなアクションの詳細について説明します。"
services: log-analytics
documentationcenter: 
author: bwren
manager: jwhit
editor: tysonn
ms.assetid: 6cfd2a46-b6a2-4f79-a67b-08ce488f9a91
ms.service: log-analytics
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/08/2018
ms.author: bwren
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e80481f074bc196caae7c03f54134eaef0fb46d5
ms.sourcegitcommit: 9292e15fc80cc9df3e62731bafdcb0bb98c256e1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/10/2018
---
# <a name="add-actions-to-alert-rules-in-log-analytics"></a>Log Analytics のアラート ルールへのアクションの追加
[Log Analytics でアラートを作成する](log-analytics-alerts.md)際に、1 つまたは複数の操作を実行[アラート ルールを構成する](log-analytics-alerts.md)ことができます。  この記事では、使用できるさまざまなアクションと、それぞれの構成に関する詳細を示します。

| アクションを表示します。 | [説明] |
|:--|:--|
| [電子メール](#email-actions) | アラートの詳細を記載した電子メールを 1 人以上の受信者に送信します。 |
| [webhook](#webhook-actions) | 1 つの HTTP POST 要求を使用して外部プロセスを呼び出します。 |
| [Runbook](#runbook-actions) | Azure Automation で Runbook を開始します。 |


## <a name="email-actions"></a>電子メール アクション
電子メール アクションは、アラートの詳細を記載した電子メールを 1 人以上の受信者に送信します。  電子メールの件名は指定できますが、メールの内容は Log Analytics によって構築された標準の形式となります。  電子メールには、アラートの名前などの概要情報に加えて、ログ検索で返される最大 10 個のレコードの詳細情報が含まれます。  また、そのクエリに基づくレコードのセット全体を返す Log Analytics のログ検索へのリンクも含まれています。   メールの送信者は、"*Microsoft Operations Management Suite チーム &lt;noreply@oms.microsoft.com&gt;*" となります。 

電子メール アクションには、次の表に示すプロパティが必要です。

| プロパティ | [説明] |
|:--- |:--- |
| 件名 |電子メールの件名。  電子メールの本文を変更することはできません。 |
| Recipients |電子メールのすべての受信者のアドレス。  複数のアドレスを指定する場合は、アドレスをセミコロン (;) で区切ります。 |


## <a name="webhook-actions"></a>Webhook アクション

Webhook アクションは、1 つの HTTP POST 要求を使用して外部のプロセスを呼び出すことができます。  呼び出されるサービスは、Webhook をサポートし、受信したペイロードの使用方法を決定できる必要があります。  また、要求に API で認識される形式を使用すれば、Webhook を明示的にはサポートしない REST API も呼び出すことができます。  アラートに対する応答で webhook を使用する例では、[Slack](http://slack.com) でメッセージを送信したり、[PagerDuty](http://pagerduty.com/) でインシデントを作成しています。  Webhook で Slack を呼び出すアラート ルールを作成する詳細なチュートリアルについては、[Log Analytics のアラートでの Webhook](log-analytics-alerts-webhooks.md)に関する記事を参照してください。

webhook アクションには、次の表に示すプロパティが必要です。

| プロパティ | [説明] |
|:--- |:--- |
| Webhook URL |Webhook の URL。 |
| Custom JSON payload (カスタム JSON ペイロード) |Webhook と共に送信するカスタム ペイロードです。  詳細については、以下をご覧ください。 |


Webhook には、URL と共に、外部のサービスに送信されるデータである JSON 形式のペイロードが含まれます。  既定では、ペイロードには次の表に示す値が格納されます。  このペイロードは、独自のカスタム ペイロードに置き換えることができます。  その場合は、各パラメーターに対して表に示される変数を使用して、カスタム ペイロードにそれらの値を含めることができます。

>[!NOTE]
> ご使用のワークスペースが[新しい Log Analytics クエリ言語](log-analytics-log-search-upgrade.md)にアップグレードされている場合は、webhook ぺイロードが変わります。  フォーマットの詳細については、「[Azure Log Analytics REST API](https://aka.ms/loganalyticsapiresponse)」をご覧ください。  以下の[サンプル](#sample-payload)のセクションで例をご覧いただけます。

| パラメーター | 変数 | [説明] |
|:--- |:--- |:--- |
| AlertRuleName |#alertrulename |アラート ルールの名前。 |
| AlertThresholdOperator |#thresholdoperator |アラート ルールのしきい値演算子。  "*Greater than*" または "*Less than*" を使用できます。 |
| AlertThresholdValue |#thresholdvalue |アラート ルールのしきい値。 |
| LinkToSearchResults |#linktosearchresults |アラートを作成したクエリに基づいてレコードを返す Log Analytics ログ検索へのリンク。 |
| ResultCount |#searchresultcount |検索結果に含まれるレコードの数。 |
| SearchIntervalEndtimeUtc |#searchintervalendtimeutc |UTC 形式で記述したクエリの終了時刻。 |
| SearchIntervalInSeconds |#searchinterval |アラート ルールの時間枠。 |
| SearchIntervalStartTimeUtc |#searchintervalstarttimeutc |UTC 形式で記述したクエリの開始時刻。 |
| SearchQuery |#searchquery |アラート ルールで使用されるログ検索クエリ。 |
| SearchResults |以下を参照 |クエリによって返される JSON 形式のレコード。  最初の 5,000 レコードに制限されます。 |
| WorkspaceID |#workspaceid |Log Analytics ワークスペースの ID |

たとえば、 *text*という名前の 1 つのパラメーターを含む次のカスタム ペイロードを指定できます。  この Webhook で呼び出すサービスでは、このパラメーターが想定されます。

    {
        "text":"#alertrulename fired with #searchresultcount over threshold of #thresholdvalue."
    }

この例のペイロードは、Webhook に送信されると、次のような内容に解決されます。

    {
        "text":"My Alert Rule fired with 18 records over threshold of 10 ."
    }

カスタム ペイロードに検索結果を含めるには、json ペイロードの最上位レベルのプロパティとして、次の行を追加します。  

    "IncludeSearchResults":true

たとえば、アラート名と検索結果だけを含むカスタム ペイロードを作成するには、次のように入力します。 

    {
       "alertname":"#alertrulename",
       "IncludeSearchResults":true
    }


Webhook で外部サービスを開始するアラート ルールを作成する例の詳細については、[Log Analytics で アラート webhook アクションを作成して、メッセージを Slack に送信する](log-analytics-alerts-webhooks.md)方法に関する記事を参照してください。


## <a name="runbook-actions"></a>Runbook アクション
Runbook アクションは、Azure Automation で Runbook を開始します。  この種類のアクションを使用するためには、Log Analytics ワークスペースに [Automation ソリューション](log-analytics-add-solutions.md) がインストールされ、構成されている必要があります。  Automation ソリューションで構成されているオートメーション アカウントの Runbook から選択できます。

Runbook アクションには、次の表に示すプロパティが必要です。

| プロパティ | [説明] |
|:--- |:---|
| Runbook | アラートの作成時に開始する Runbook。 |
| Run on (実行先) | Runbook をクラウドで実行する場合は、**[Azure]** を指定します。  **Hybrid Runbook Worker** がインストールされたエージェントで Runbook を実行する場合は、[[ハイブリッド worker]](../automation/automation-hybrid-runbook-worker.md ) を指定します。  |

Runbook アクションは、 [Webhook](../automation/automation-webhooks.md)を使用して Runbook を開始します。  アラート ルールを作成すると、Runbook に対して、" **OMS Alert Remediation** " の後に GUID が付いた名前を持つ新しい Webhook が自動的に作成されます。  

Runbook のパラメーターを直接設定することはできませんが、[$WebhookData パラメーター](../automation/automation-webhooks.md)にアラートの詳細 (それを作成したログ検索の結果を含む) が格納されます。  この Runbook で、アラートのプロパティにアクセスするためのパラメーターとして **$WebhookData** を定義する必要があります。  アラート データは、**$WebhookData** の **RequestBody** プロパティにある **SearchResult** (Runbook アクションと標準ペイロードを使用する webhook アクションの場合) または **SearchResults** (**IncludeSearchResults":true** を含むカスタム ペイロードを使用する webhook アクション) という単一のプロパティから JSON 形式で利用できます。  このデータには、次の表に示したプロパティが存在します。

>[!NOTE]
> ご使用のワークスペースが[新しい Log Analytics クエリ言語](log-analytics-log-search-upgrade.md)にアップグレードされている場合は、runbook ペイロードが変わります。  フォーマットの詳細については、「[Azure Log Analytics REST API](https://aka.ms/loganalyticsapiresponse)」をご覧ください。  以下の[サンプル](#sample-payload)のセクションで例をご覧いただけます。  

| ノード | [説明] |
|:--- |:--- |
| id |検索のパスと GUID。 |
| __metadata |アラートに関する情報 (レコードの件数、検索結果の状態を含む)。 |
| value |検索結果のレコードごとのエントリ。  エントリの詳細は、レコードのプロパティおよび値と対応します。 |

たとえば、以下の Runbook では、ログ検索から返されたレコードを抽出し、レコードの種類ごとに異なるプロパティを割り当てています。  Runbook ではまず、JSON 形式の **RequestBody** を PowerShell からオブジェクトとして扱うことができるように変換していることに注目してください。

>[!NOTE]
> これらの Runbook はどちらも、Runbook アクションと標準ペイロードを使用する webhook アクションの結果を含むプロパティである **SearchResult** を使用します。  カスタム ペイロードを使用する webhook 応答から Runbook が呼び出される場合は、このプロパティを **SearchResults** に変更する必要があります。

次の Runbook は [レガシ Log Analytics ワークスペース](log-analytics-log-search-upgrade.md)からのペイロードを処理します。

    param ( 
        [object]$WebhookData
    )

    $RequestBody = ConvertFrom-JSON -InputObject $WebhookData.RequestBody
    $Records     = $RequestBody.SearchResult.value

    foreach ($Record in $Records)
    {
        $Computer = $Record.Computer

        if ($Record.Type -eq 'Event')
        {
            $EventNo    = $Record.EventID
            $EventLevel = $Record.EventLevelName
            $EventData  = $Record.EventData
        }

        if ($Record.Type -eq 'Perf')
        {
            $Object    = $Record.ObjectName
            $Counter   = $Record.CounterName
            $Instance  = $Record.InstanceName
            $Value     = $Record.CounterValue
        }
    }

次の Runbook は [アップグレードされた Log Analytics ワークスペース](log-analytics-log-search-upgrade.md)からのペイロードを処理します。

    param ( 
        [object]$WebhookData
    )

    $RequestBody = ConvertFrom-JSON -InputObject $WebhookData.RequestBody

    # Get all metadata properties    
    $AlertRuleName = $RequestBody.AlertRuleName
    $AlertThresholdOperator = $RequestBody.AlertThresholdOperator
    $AlertThresholdValue = $RequestBody.AlertThresholdValue
    $AlertDescription = $RequestBody.Description
    $LinktoSearchResults =$RequestBody.LinkToSearchResults
    $ResultCount =$RequestBody.ResultCount
    $Severity = $RequestBody.Severity
    $SearchQuery = $RequestBody.SearchQuery
    $WorkspaceID = $RequestBody.WorkspaceId
    $SearchWindowStartTime = $RequestBody.SearchIntervalStartTimeUtc
    $SearchWindowEndTime = $RequestBody.SearchIntervalEndtimeUtc
    $SearchWindowInterval = $RequestBody.SearchIntervalInSeconds

    # Get detailed search results
    if($RequestBody.SearchResult -ne $null)
    {
        $SearchResultRows    = $RequestBody.SearchResult.tables[0].rows 
        $SearchResultColumns = $RequestBody.SearchResult.tables[0].columns;

        foreach ($SearchResultRow in $SearchResultRows)
        {   
            $Column = 0
            $Record = New-Object –TypeName PSObject 
        
            foreach ($SearchResultColumn in $SearchResultColumns)
            {
                $Name = $SearchResultColumn.name
                $ColumnValue = $SearchResultRow[$Column]
                $Record | Add-Member –MemberType NoteProperty –Name $name –Value $ColumnValue -Force
                        
                $Column++
            }

            # Include code to work with the record. 
            # For example $Record.Computer to get the computer property from the record.
            
        }
    }



## <a name="sample-payload"></a>サンプル ペイロード
このセクションでは、レガシ ワークスペースと[アップグレードされた Log Analytics ワークスペース](log-analytics-log-search-upgrade.md)の両方での webhook と runbook アクションのサンプル ペイロードを示します。

### <a name="webhook-actions"></a>Webhook アクション
これらの例はどちらも、標準ペイロードを使用する webhook アクションの結果を含むプロパティである **SearchResult** を使用します。  検索結果を含むカスタム ペイロードを webhook が使用する場合、このプロパティは **SearchResults** になります。

#### <a name="legacy-workspace"></a>レガシ ワークスペース
レガシ ワークスペースでの、webhook アクション用サンプル ペイロードを次に示します。

    {
    "WorkspaceId": "workspaceID",
    "AlertRuleName": "WebhookAlert",
    "SearchQuery": "Type=Usage",
    "SearchResult": {
        "id": "subscriptions/subscriptionID/resourceGroups/ResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspace-workspaceID/search/SearchGUID|10.1.0.7|2017-09-27T10-30-38Z",
        "__metadata": {
        "resultType": "raw",
        "total": 1,
        "top": 2147483647,
        "RequestId": "SearchID|10.1.0.7|2017-09-27T10-30-38Z",
        "CoreSummaries": [
            {
            "Status": "Successful",
            "NumberOfDocuments": 135000000
            }
        ],
        "Status": "Successful",
        "NumberOfDocuments": 135000000,
        "StartTime": "2017-09-27T10:30:38.9453282Z",
        "LastUpdated": "2017-09-27T10:30:44.0907473Z",
        "ETag": "636421050440907473",
        "sort": [
            {
            "name": "TimeGenerated",
            "order": "desc"
            }
        ],
        "requestTime": 361
        },
        "value": [
        {
            "Computer": "-",
            "SourceSystem": "OMS",
            "TimeGenerated": "2017-09-26T13:59:59Z",
            "ResourceUri": "/subscriptions/df1ec963-d784-4d11-a779-1b3eeb9ecb78/resourcegroups/mms-eus/providers/microsoft.operationalinsights/workspaces/workspace-861bd466-5400-44be-9552-5ba40823c3aa",
            "DataType": "Operation",
            "StartTime": "2017-09-26T13:00:00Z",
            "EndTime": "2017-09-26T13:59:59Z",
            "Solution": "LogManagement",
            "BatchesWithinSla": 8,
            "BatchesOutsideSla": 0,
            "BatchesCapped": 0,
            "TotalBatches": 8,
            "AvgLatencyInSeconds": 0.0,
            "Quantity": 0.002502,
            "QuantityUnit": "MBytes",
            "IsBillable": false,
            "MeterId": "a4e29a95-5b4c-408b-80e3-113f9410566e",
            "LinkedMeterId": "00000000-0000-0000-0000-000000000000",
            "id": "954f7083-cd55-3f0a-72cb-3d78cd6444a3",
            "Type": "Usage",
            "MG": "00000000-0000-0000-0000-000000000000",
            "__metadata": {
            "Type": "Usage",
            "TimeGenerated": "2017-09-26T13:59:59Z"
            }
        }
        ]
    },
    "SearchIntervalStartTimeUtc": "2017-09-26T08:10:40Z",
    "SearchIntervalEndtimeUtc": "2017-09-26T09:10:40Z",
    "AlertThresholdOperator": "Greater Than",
    "AlertThresholdValue": 0,
    "ResultCount": 1,
    "SearchIntervalInSeconds": 3600,
    "LinkToSearchResults": "https://workspaceID.portal.mms.microsoft.com/#Workspace/search/index?_timeInterval.intervalEnd=2017-09-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Type%3DUsage",
    "Description": null,
    "Severity": "Low"
    }


#### <a name="upgraded-workspace"></a>アップグレードされたワークスペース
アップグレードされたワークスペースでの、webhook アクション用サンプル ペイロードを次に示します。

    {
    "WorkspaceId": "workspaceID",
    "AlertRuleName": "WebhookAlert",
    "SearchQuery": "Usage",
    "SearchResult": {
        "tables": [
        {
            "name": "PrimaryResult",
            "columns": [
            {
                "name": "TenantId",
                "type": "string"
            },
            {
                "name": "Computer",
                "type": "string"
            },
            {
                "name": "TimeGenerated",
                "type": "datetime"
            },
            {
                "name": "SourceSystem",
                "type": "string"
            },
            {
                "name": "StartTime",
                "type": "datetime"
            },
            {
                "name": "EndTime",
                "type": "datetime"
            },
            {
                "name": "ResourceUri",
                "type": "string"
            },
            {
                "name": "LinkedResourceUri",
                "type": "string"
            },
            {
                "name": "DataType",
                "type": "string"
            },
            {
                "name": "Solution",
                "type": "string"
            },
            {
                "name": "BatchesWithinSla",
                "type": "long"
            },
            {
                "name": "BatchesOutsideSla",
                "type": "long"
            },
            {
                "name": "BatchesCapped",
                "type": "long"
            },
            {
                "name": "TotalBatches",
                "type": "long"
            },
            {
                "name": "AvgLatencyInSeconds",
                "type": "real"
            },
            {
                "name": "Quantity",
                "type": "real"
            },
            {
                "name": "QuantityUnit",
                "type": "string"
            },
            {
                "name": "IsBillable",
                "type": "bool"
            },
            {
                "name": "MeterId",
                "type": "string"
            },
            {
                "name": "LinkedMeterId",
                "type": "string"
            },
            {
                "name": "Type",
                "type": "string"
            }
            ],
            "rows": [
            [
                "workspaceID",
                "-",
                "2017-09-26T13:59:59Z",
                "OMS",
                "2017-09-26T13:00:00Z",
                "2017-09-26T13:59:59Z",
                "/subscriptions/SubscriptionID/resourcegroups/ResourceGroupName/providers/microsoft.operationalinsights/workspaces/workspace-workspaceID",
                null,
                "Operation",
                "LogManagement",
                8,
                0,
                0,
                8,
                0,
                0.002502,
                "MBytes",
                false,
                "a4e29a95-5b4c-408b-80e3-113f9410566e",
                "00000000-0000-0000-0000-000000000000",
                "Usage"
            ]
            ]
        }
        ]
    },
    "SearchIntervalStartTimeUtc": "2017-09-26T08:10:40Z",
    "SearchIntervalEndtimeUtc": "2017-09-26T09:10:40Z",
    "AlertThresholdOperator": "Greater Than",
    "AlertThresholdValue": 0,
    "ResultCount": 1,
    "SearchIntervalInSeconds": 3600,
    "LinkToSearchResults": "https://workspaceID.portal.mms.microsoft.com/#Workspace/search/index?_timeInterval.intervalEnd=2017-09-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
    "Description": null,
    "Severity": "Low"
    }


### <a name="runbooks"></a>Runbooks

#### <a name="legacy-workspace"></a>レガシ ワークスペース
レガシ ワークスペースでの、runbook アクション用サンプル ペイロードを次に示します。

    {
        "SearchResult": {
            "id": "subscriptions/subscriptionID/resourceGroups/ResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspace-workspaceID/search/searchGUID|10.1.0.7|TimeStamp",
            "__metadata": {
                "resultType": "raw",
                "total": 1,
                "top": 2147483647,
                "RequestId": "searchGUID|10.1.0.7|2017-09-27T10-51-43Z",
                "CoreSummaries": [{
                    "Status": "Successful",
                    "NumberOfDocuments": 135000000
                }],
                "Status": "Successful",
                "NumberOfDocuments": 135000000,
                "StartTime": "2017-09-27T10:51:43.3075124Z",
                "LastUpdated": "2017-09-27T10:51:51.1002092Z",
                "ETag": "636421063111002092",
                "sort": [{
                    "name": "TimeGenerated",
                    "order": "desc"
                }],
                "requestTime": 511
            },
            "value": [{
                "Computer": "-",
                "SourceSystem": "OMS",
                "TimeGenerated": "2017-09-26T13:59:59Z",
                "ResourceUri": "/subscriptions/AnotherSubscriptionID/resourcegroups/SampleResourceGroup/providers/microsoft.operationalinsights/workspaces/workspace-workspaceID",
                "DataType": "Operation",
                "StartTime": "2017-09-26T13:00:00Z",
                "EndTime": "2017-09-26T13:59:59Z",
                "Solution": "LogManagement",
                "BatchesWithinSla": 8,
                "BatchesOutsideSla": 0,
                "BatchesCapped": 0,
                "TotalBatches": 8,
                "AvgLatencyInSeconds": 0.0,
                "Quantity": 0.002502,
                "QuantityUnit": "MBytes",
                "IsBillable": false,
                "MeterId": "a4e29a95-5b4c-408b-80e3-113f9410566e",
                "LinkedMeterId": "00000000-0000-0000-0000-000000000000",
                "id": "954f7083-cd55-3f0a-72cb-3d78cd6444a3",
                "Type": "Usage",
                "MG": "00000000-0000-0000-0000-000000000000",
                "__metadata": {
                    "Type": "Usage",
                    "TimeGenerated": "2017-09-26T13:59:59Z"
                }
            }]
        }
    }

#### <a name="upgraded-workspace"></a>アップグレードされたワークスペース
アップグレードされたワークスペースでの、runbook アクション用サンプル ペイロードを次に示します。

    {
    "WorkspaceId": "workspaceID",
    "AlertRuleName": "AutomationAlert",
    "SearchQuery": "Usage",
    "SearchResult": {
        "tables": [
        {
            "name": "PrimaryResult",
            "columns": [
            {
                "name": "TenantId",
                "type": "string"
            },
            {
                "name": "Computer",
                "type": "string"
            },
            {
                "name": "TimeGenerated",
                "type": "datetime"
            },
            {
                "name": "SourceSystem",
                "type": "string"
            },
            {
                "name": "StartTime",
                "type": "datetime"
            },
            {
                "name": "EndTime",
                "type": "datetime"
            },
            {
                "name": "ResourceUri",
                "type": "string"
            },
            {
                "name": "LinkedResourceUri",
                "type": "string"
            },
            {
                "name": "DataType",
                "type": "string"
            },
            {
                "name": "Solution",
                "type": "string"
            },
            {
                "name": "BatchesWithinSla",
                "type": "long"
            },
            {
                "name": "BatchesOutsideSla",
                "type": "long"
            },
            {
                "name": "BatchesCapped",
                "type": "long"
            },
            {
                "name": "TotalBatches",
                "type": "long"
            },
            {
                "name": "AvgLatencyInSeconds",
                "type": "real"
            },
            {
                "name": "Quantity",
                "type": "real"
            },
            {
                "name": "QuantityUnit",
                "type": "string"
            },
            {
                "name": "IsBillable",
                "type": "bool"
            },
            {
                "name": "MeterId",
                "type": "string"
            },
            {
                "name": "LinkedMeterId",
                "type": "string"
            },
            {
                "name": "Type",
                "type": "string"
            }
            ],
            "rows": [
            [
                "861bd466-5400-44be-9552-5ba40823c3aa",
                "-",
                "2017-09-26T13:59:59Z",
                "OMS",
                "2017-09-26T13:00:00Z",
                "2017-09-26T13:59:59Z",
            "/subscriptions/SubscriptionID/resourcegroups/ResourceGroupName/providers/microsoft.operationalinsights/workspaces/workspace-861bd466-5400-44be-9552-5ba40823c3aa",
                null,
                "Operation",
                "LogManagement",
                8,
                0,
                0,
                8,
                0,
                0.002502,
                "MBytes",
                false,
                "a4e29a95-5b4c-408b-80e3-113f9410566e",
                "00000000-0000-0000-0000-000000000000",
                "Usage"
            ]
        }
        ]
    },
    "SearchIntervalStartTimeUtc": "2017-09-26T08:10:40Z",
    "SearchIntervalEndtimeUtc": "2017-09-26T09:10:40Z",
    "AlertThresholdOperator": "Greater Than",
    "AlertThresholdValue": 0,
    "ResultCount": 1,
    "SearchIntervalInSeconds": 3600,
    "LinkToSearchResults": "https://workspaceID.portal.mms.microsoft.com/#Workspace/search/index?_timeInterval.intervalEnd=2017-09-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
    "Description": null,
    "Severity": "Critical"
    }



## <a name="next-steps"></a>次の手順
- アラート ルールに関する [Webhook を構成する](log-analytics-alerts-webhooks.md) チュートリアルを完了します。  
- アラートで識別された問題を修復するために [Azure Automation の Runbook](https://azure.microsoft.com/documentation/services/automation) を作成する方法について学習します。