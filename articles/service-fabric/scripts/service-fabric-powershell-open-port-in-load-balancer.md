---
title: "Azure PowerShell スクリプト サンプル - ロード バランサーのアプリケーション ポートを開く | Microsoft Docs"
description: "Azure PowerShell スクリプト サンプル - Service Fabric アプリケーションのために Azure ロード バランサーのポートを開きます。"
services: service-fabric
documentationcenter: 
author: rwike77
manager: timlt
editor: 
tags: azure-service-management
ms.assetid: 
ms.service: service-fabric
ms.workload: multiple
ms.devlang: na
ms.topic: sample
ms.date: 12/08/2017
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: c643fc9e575a8e836a361893d78348bbd627a425
ms.sourcegitcommit: 42ee5ea09d9684ed7a71e7974ceb141d525361c9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/09/2017
---
# <a name="open-an-application-port-in-the-azure-load-balancer"></a>Azure ロード バランサーのアプリケーション ポートを開く

Azure で実行する Service Fabric アプリケーションは、Azure ロード バランサー上にあります。 このサンプル スクリプトは、Azure ロード バランサーのポートを開いて、Service Fabric アプリケーションが外部クライアントと通信できるようにします。 必要に応じてパラメーターをカスタマイズします。 クラスターがネットワーク セキュリティ グループに含まれている場合は、受信トラフィックを許可する[受信ネットワーク セキュリティ グループ ルールを追加](service-fabric-powershell-add-nsg-rule.md)します。

必要に応じて、Service Fabric PowerShell モジュールを、[Service Fabric SDK](../service-fabric-get-started.md) と共にインストールします。 

## <a name="sample-script"></a>サンプル スクリプト

[!code-powershell[main](../../../powershell_scripts/service-fabric/open-port-in-load-balancer/open-port-in-load-balancer.ps1 "Open a port in the load balancer")]

## <a name="script-explanation"></a>スクリプトの説明

このスクリプトでは、次のコマンドを使用します。 表内の各コマンドは、それぞれのドキュメントにリンクされています。

| コマンド | メモ |
|---|---|
| [Get-AzureRmResource](/powershell/module/azurerm.resources/get-azurermresource) | Azure リソースを取得します。  |
| [Get-AzureRmLoadBalancer](/powershell/module/azurerm.network/get-azurermloadbalancer) | Azure ロード バランサーを取得します。 |
| [Add-AzureRmLoadBalancerProbeConfig](/powershell/module/azurerm.network/add-azurermloadbalancerprobeconfig) | プローブの構成をロード バランサーに追加します。|
| [Get-AzureRmLoadBalancerProbeConfig](/powershell/module/azurerm.network/get-azurermloadbalancerprobeconfig) | ロード バランサーのプローブの構成を取得します。 |
| [Add-AzureRmLoadBalancerRuleConfig](/powershell/module/azurerm.network/add-azurermloadbalancerruleconfig) | 規則の構成をロード バランサーに追加します。 |
| [Set-AzureRmLoadBalancer](/powershell/module/azurerm.network/set-azurermloadbalancer) | ロード バランサーの目標状態を設定します。 |

## <a name="next-steps"></a>次のステップ

Azure PowerShell モジュールの詳細については、[Azure PowerShell のドキュメント](/powershell/azure/overview)を参照してください。

その他の Azure Service Fabric 用 PowerShell サンプルは、[Azure PowerShell サンプル](../service-fabric-powershell-samples.md)のページにあります。
