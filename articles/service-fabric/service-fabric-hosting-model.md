---
title: Azure Service Fabric ホスティング モデル | Microsoft Docs
description: デプロイされた Servic Fabric サービスのレプリカ (またはインスタンス) と、サービス ホスト プロセスとの関係を説明します。
services: service-fabric
documentationcenter: .net
author: harahma
manager: timlt
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/15/2017
ms.author: harahma
ms.openlocfilehash: 0206a9a486e3511834a23b3cc3f20f236a1cc261
ms.sourcegitcommit: 8aab1aab0135fad24987a311b42a1c25a839e9f3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/16/2018
---
# <a name="service-fabric-hosting-model"></a>Service Fabric ホスティング モデル
この記事では、Service Fabric によって提供されるアプリケーション ホスティング モデルの概要、および**共有プロセス** モデルと**専有プロセス** モデルとの相違点について説明します。 デプロイされたアプリケーションが Service Fabric ノード上でどのように表示されるかについて、また Servic Fabric サービスのレプリカ (またはインスタンス) とサービス ホスト プロセスとの間の関係について説明します。

先に進む前に、[Service Fabric のアプリケーション モデル][a1]について理解を深め、さまざまな概念とその概念間の関係について理解してください。 

> [!NOTE]
> この記事では、明記されている場合を除き、わかりやすくするために次のとおりとします。
>
> - "*レプリカ*" という言葉を使用する場合は、ステートフル サービスのレプリカ、またはステートレス サービスのインスタンスを意味します。
> - *CodePackage* は、*ServiceType* を登録する *ServiceHost* プロセスに相当するものとして扱われ、その *ServiceType* のサービスのレプリカをホストします。
>

ホスティング モデルを理解するための例を紹介します。 たとえば、'MyAppType' という *ApplicationType* があり、これは 'MyServiceType' という *ServiceType* を持つとします。  'MyServiceType' は、'MyServicePackage' という *ServicePackage* によって提供され、'MyServicePackage' は 'MyCodePackage' という *CodePackage* を持ちます。 'MyCodePackage' は、'MyServiceType' という *ServiceType* を実行時に登録します。

3 ノード クラスターがあるとして、'MyAppType' というタイプの **fabric:/App1** という "*アプリケーション*" を作成します。 この **fabric:/App1** という "*アプリケーション*" の内側に、2 つのパーティション (たとえば **P1** & **P2**) とパーティションごとに 3 つのレプリカを持つ、'MyServiceType' というタイプの **fabric:/App1/ServiceA** というサービスを作成します。 次の図は、最終的にノードにデプロイされるこのアプリケーションのビューを示しています。

<center>
![デプロイされたアプリケーションのノードのビュー][node-view-one]
</center>

Service Fabric は、両方のパーティションのレプリカをホスティングする 'MyCodePackage' を開始する 'MyServicePackage' をアクティブ化します。  たとえば、**P1** & **P2** です。 クラスター内のノードと同じ数のレプリカをパーティションごとに選択するので、クラスターのノードはすべて同じビューを持ちます。 1 つのパーティション (たとえば **P3**) とパーティションごとに 3 つのレプリカを持つ **fabric:/App1** というアプリケーション内に、**fabric:/App1/ServiceB** というもう 1 つのサービスを作成します。 次の図は、このノードの新しいビューを示しています。

<center>
![デプロイされたアプリケーションのノードのビュー][node-view-two]
</center>

ご覧のとおり Service Fabric は、アクティブ化した既存の 'MyServicePackage' 内に **fabric:/App1/ServiceB** というサービスの **P3** パーティションの新しいレプリカを追加しました。 次に、'MyAppType' というタイプの **fabric:/App2** という別の "*アプリケーション*" を作成します。 **fabric:/App2** の内側に、2 つのパーティション (たとえば **P4** & **P5**) とパーティションごとに 3 つのレプリカを持つ、**fabric:/App2/ServiceA** というサービスを作成します。 次の図は、この新しいノードのビューを示しています。

<center>
![デプロイされたアプリケーションのノードのビュー][node-view-three]
</center>

Service Fabric は 'MyServicePackage' の新しいコピーをアクティブ化し、これによって 'MyCodePackage' の新しいコピーが起動されます。 サービス **fabric:/App2/ServiceA** の両方のパーティション (たとえば、**P4** & **P5**) のレプリカが、この 'MyCodePackage' という新しいコピーに配置されています。

## <a name="shared-process-model"></a>共有プロセス モデル
上記のモデルは Service Fabric が提供する既定のホスティング モデルであり、**共有プロセス** モデルと呼ばれます。 このモデルでは、特定の*アプリケーション*について、特定の *ServicePackage* のコピーを 1 つだけ*ノード* (これは内部のすべての *CodePackage* を起動します) 上でアクティブ化します。特定の *ServiceType* のすべてのサービスのすべてのレプリカが、*ServiceType* を登録する *CodePackage* に追加されます。 つまり、特定の *ServiceType* のノード上ですべてのサービスのすべてのレプリカが同じプロセスを共有します。

## <a name="exclusive-process-model"></a>専有プロセス モデル
Service Fabric が提供するもう 1 つのホスティング モデルが、**専有プロセス** モデルです。 このモデルでは、各レプリカの追加について、特定の*ノード*上で Service Fabric が *ServicePackage* (これは内部のすべての *CodePackage* を開始します) の新しいコピーをアクティブ化します。レプリカが所属するサービスの *ServiceType* を登録した *CodePackage* にレプリカが追加されます。 つまり、各レプリカは自身の専用のプロセスで稼働します。 

このモデルは、Service Fabric のバージョン 5.6 以降でサポートされています。 **専有プロセス** モデルは、サービス作成時に **ServicePackageActivationMode** を 'ExclusiveProcess' として指定することで選択できます ([PowerShell][p1]、[REST][r1]、または [FabricClient][c1] を使用します)。

```powershell
PS C:\>New-ServiceFabricService -ApplicationName "fabric:/App1" -ServiceName "fabric:/App1/ServiceA" -ServiceTypeName "MyServiceType" -Stateless -PartitionSchemeSingleton -InstanceCount -1 -ServicePackageActivationMode "ExclusiveProcess"
```

```csharp
var serviceDescription = new StatelessServiceDescription
{
    ApplicationName = new Uri("fabric:/App1"),
    ServiceName = new Uri("fabric:/App1/ServiceA"),
    ServiceTypeName = "MyServiceType",
    PartitionSchemeDescription = new SingletonPartitionSchemeDescription(),
    InstanceCount = -1,
    ServicePackageActivationMode = ServicePackageActivationMode.ExclusiveProcess
};

var fabricClient = new FabricClient(clusterEndpoints);
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

アプリケーション マニフェストで既定のサービスが指定されていれば、次に示すように **ServicePackageActivationMode** 属性を指定することで **Exclusive Process** モデルを選択できます。

```xml
<DefaultServices>
  <Service Name="MyService" ServicePackageActivationMode="ExclusiveProcess">
    <StatelessService ServiceTypeName="MyServiceType" InstanceCount="1">
      <SingletonPartition/>
    </StatelessService>
  </Service>
</DefaultServices>
```
前の例で作業を進め、2 つのパーティション (たとえば **P6** & **P7**) とパーティションごとに 3 つのレプリカを持つ **fabric:/App1** というアプリケーションに **fabric:/App1/ServiceC** というもう 1 つのサービスを作成します。**ServicePackageActivationMode** を 'ExclusiveProcess' に設定します。 次の図は、ノードの新しいビューを示しています。

<center>
![デプロイされたアプリケーションのノードのビュー][node-view-four]
</center>

ご覧のとおり Service Fabric は、'MyServicePackage' (**P6** & **P7** のパーティションのレプリカごとに 1 つ) の新しい 2 つのコピーをアクティブ化しました。*CodePackage* の専用のコピーに各レプリカが追加されました。 また、ここでは、**専有プロセス** モデルを使用する場合、特定の*アプリケーション*について、1 つの*ノード*上で特定の *ServicePackage* の複数のコピーをアクティブ化できることに注意してください。 上の例では、**fabric:/App1** の 'MyServicePackage' の 3 つのコピーがアクティブ化されています。 この 'MyServicePackage' のアクティブなコピーにはそれぞれ、**ServicePackageActivationId** が関連付けられています。この ID で **fabric:/App1** "*アプリケーション*" 内のコピーを特定します。

上の例の **fabric:/App2** のような*アプリケーション*に**共有プロセス** モデルのみを使用する場合、1 つの*ノード*に *ServicePackage* のアクティブなコピーは 1 つだけ存在し、*ServicePackage* のこのアクティブ化の **ServicePackageActivationId** は '空の文字列' になります。

> [!NOTE]
>- **共有プロセス** ホスティング モデルは、**SharedProcess** と等しい **ServicePackageActivationMode** に対応しています。 これは既定のホスティング モデルであり、サービス作成時に **ServicePackageActivationMode** を指定する必要はありません。
>
>- **専有プロセス** ホスティング モデルは、**ExclusiveProcess** に設定されている **ServicePackageActivationMode** に対応しています。サービス作成時にはこの ServicePackageAtivationMode を専有として指定する必要があります。 
>
>- クエリを実行して[サービスの説明][p2]の **ServicePackageActivationMode** の値を確認することで、サービスのホスティング モデルを知ることができます。
>
>

## <a name="working-with-deployed-service-package"></a>デプロイされたサービス パッケージの操作
ノード上の *ServicePackage* のアクティブなコピーは、[デプロイされたサービス パッケージ][p3]と呼ばれます。 前述のとおり、**専有プロセス** モデルを使用してサービスを作成した場合、特定の "*アプリケーション*" について、同じ *ServicePackage* にデプロイされたサービス パッケージが複数存在する可能性があります。 デプロイされたサービス パッケージに固有の操作 (たとえば、[デプロイされたサービス パッケージの正常性のレポート][p4]や、[デプロイされたサービス パッケージのコード パッケージの再開][p5]) を行う場合は、個別のデプロイされたサービス パッケージを特定できるように **ServicePackageActivationId** を指定する必要があります。

デプロイされたサービス パッケージの **ServicePackageActivationId** は、ノード上の[デプロイされたサービス パッケージ][p3]のリストをクエリ実行することで取得できます。 ノード上にある、[デプロイされたサービスのタイプ][p6]、[デプロイされたレプリカ][p7]、[デプロイされたコード パッケージ][p8]をクエリ実行する場合、クエリ結果には親のデプロイされたサービス パッケージの **ServicePackageActivationId** も含まれます。

> [!NOTE]
>- **共有プロセス** ホスティング モデルでは、特定の*アプリケーション*について、特定の 1 つの*ノード*上でアクティブ化できる *ServicePackage* のコピーは 1 つだけです。 このコピーは "*空の文字列*" と等しい **ServicePackageActivationId** を持つため、デプロイされたサービス パッケージに関連する操作を行う際に、この ID を指定する必要はありません。 
>
> - **専有プロセス** ホスティング モデルでは、特定の*アプリケーション*について、特定の 1 つの*ノード*上でアクティブ化できる *ServicePackage* のコピーは 1 つ以上です。 各アクティブ化には "*空ではない*" **ServicePackageActivationId** があり、デプロイされたサービス パッケージ関連の操作を行う際に指定されます。 
>
> - **ServicePackageActivationId** が指定されない場合は、既定で '空の文字列' になります。 **共有プロセス** モデルでアクティブ化されたデプロイされたサービス パッケージが存在する場合は、そのパッケージに対して操作が行われ、それ以外の場合は操作が失敗します。
>
> - **ServicePackageActivationId** を一度だけクエリ実行してキャッシュすることは、その ID が動的に生成され、さまざまな理由で変更できてしまうため、行わないでください。 **ServicePackageActivationId** を必要とする操作を行う前に、ノード上の[デプロイされたサービス パッケージ][p3]のリストを最初にクエリ実行し、以降はクエリ結果に含まれている **ServicePackageActivationId** を使用して元の操作を行うことをお勧めします。
>
>

## <a name="guest-executable-and-container-applications"></a>ゲスト実行可能ファイルとコンテナー アプリケーション
Service Fabric は、[ゲスト実行可能ファイル][a2]と[コンテナー][a3] アプリケーションを自己完結型のステートレス サービスとして扱います。つまり、*ServiceHost* (プロセスまたはコンテナー) 内に Service Fabric ランタイムはありません。 これらのサービスは自己完結型であるため、*ServiceHost* ごとのレプリカの数はこれらのサービスに適用できません。 これらのサービスを使用した最も一般的な構成は、-1 (つまり、クラスターの各ノード上で実行されるサービス コードの 1 つのコピー) と等しい [InstanceCount][c2] を使用した単一パーティションです。 

これらのサービスの既定の **ServicePackageActivationMode** は **SharedProcess** (共有プロセス) であり、この場合 Service Fabric は特定の "*アプリケーション*" について特定の 1 つの *Node* (ノード) 上で *ServicePackage* のコピーを 1 つだけアクティブ化します。  つまり、*Node* を実行するサービス コードのコピーは 1 つのみです。 (*ServiceManifest* で指定されている) *ServiceType* のサービス (*Service1* から *ServiceN*) を複数作成するとき、またはサービスが複数のパーティションで分けられているときに、*Node* 上で実行するサービス コードのコピーが複数必要な場合は、サービス作成時に **ServicePackageActivationMode** を **ExclusiveProcess** (専有プロセス) として指定する必要があります。

## <a name="changing-hosting-model-of-an-existing-service"></a>既存のサービスのホスティング モデルを変更する
アップグレードや更新のメカニズム (またはアプリケーション マニフェストの既定のサービス仕様) で既存のサービスのホスティング モデルを**共有プロセス**から**専有プロセス**に変更する (または専有プロセスから共有プロセスに変更する) ことは現在できません。 この機能は今後のバージョンでサポートされる予定です。

## <a name="choosing-between-shared-process-and-exclusive-process-model"></a>共有プロセス モデルか専有プロセス モデルを選択する
これらのホスティング モデルはどちらも長所と短所があり、ユーザーはどのモデルが最もご自身の要求に適合しているかを評価する必要があります。 **共有プロセス** モデルは、生成されるプロセスが少なかったり、同じプロセスの複数のレプリカがポートを共有できたりするため、OS リソースをより効率的に使用できます。ただし、レプリカのうちの 1 つでエラーが発生してサービス ホストを停止する必要が生じた場合は、同じプロセスの他のすべてのレプリカに影響が出ます。

 **専有プロセス** モデルは、各レプリカを分離して独自のプロセスで動作させることができ、正常に動作しないレプリカが他のレプリカに影響を与えることはありません。 このモデルは、通信プロトコルでポート共有がサポートされていない場合に役に立ちます。 レプリカ レベルでリソース ガバナンスを適用することが容易になります。 ただし、**専有プロセス**は、ノード上の各レプリカに 1 つずつプロセスが発生するため、OS リソースをより多く消費します。

## <a name="exclusive-process-model-and-application-model-considerations"></a>専有プロセス モデルとアプリケーション モデルに関する考慮事項
Service Fabric のアプリケーションのモデルを設定するお勧めの方法は、*ServicePackage* ごとに 1 つの *ServiceType* を設定してそれを維持することです。このモデルならほとんどのアプリケーションが正常に動作します。 

Service Fabric では、特定のユース ケースを想定し、1 つの *ServicePackage* につき複数の *ServiceType* も許容しています (また、1 つの *CodePackage* が複数の *ServiceType* を登録できます)。 これらの構成が役に立つ状況を次に示します。

- 発生するプロセスが少なく、プロセスごとのレプリカの密度を高めて OS リソースの使用率を最適化したい場合。
- 初期化またはメモリの消費が多い共通のデータを、ServiceTypes が異なるレプリカで共有する必要がある場合。
- 無料のサービスを提供するときなどに、サービスのすべてのレプリカを同じプロセスにしてリソースの使用を制限したい場合。

**専有プロセス** ホスティング モデルは、*ServicePackage* ごとに複数の *ServiceTypes* を持つアプリケーション モデルには適していません。 これは、*ServicePackage* ごとに複数の *ServiceTypes* を設定できる機能が、レプリカ間で共有するリソースの使用率を高めたり、プロセスごとのレプリカの密度を高めたりするために設計されているからです。 これは**専有プロセス** モデルの設計目的とは正反対です。

異なる *CodePackage* にそれぞれの *ServiceType* を設定し、*ServicePackage* ごとに複数の *ServiceTypes* を設定する場合を考えてみましょう。 たとえば、次のような 2 つの *CodePackage* を持つ 'MultiTypeServicePackge' という *ServicePackage* があるとします。

- 'MyServiceTypeA' という *ServiceType* を登録している 'MyCodePackageA'。
- 'MyServiceTypeB' という *ServiceType* を登録している 'MyCodePackageB'。

次に、**fabric:/SpecialApp** という*アプリケーション*を作成し、**fabric:/SpecialApp** 内に次の 2 つのサービスを**専有プロセス** モデルで作成します。

- 2 つのパーティション (たとえば **P1** および **P2**) とパーティションごとに 3 つのレプリカを持つ 'MyServiceTypeA' というタイプの **fabric:/SpecialApp/ServiceA** というサービス。
- 2 つのパーティション (たとえば **P3** および **P4**) とパーティションごとに 3 つのレプリカを持つ 'MyServiceTypeB' というタイプの **fabric:/SpecialApp/ServiceB** というサービス。

特定のノード上で、両方のサービスはそれぞれ 2 つのレプリカを持ちます。 サービスを**専有プロセス** モデルで作成したため、Service Fabric は各レプリカについて 'MyServicePackge' の新しいコピーをアクティブ化します。 'MultiTypeServicePackge' の各アクティブ化によって、'MyCodePackageA' と 'MyCodePackageB' のコピーが起動します。 ただし、'MultiTypeServicePackge' がアクティブ化されたレプリカをホストするのは、'MyCodePackageA' または 'MyCodePackageB' のどちらか 1 つだけです。 次の図で、このノードのビューを示します。

<center>
![デプロイされたアプリケーションのノードのビュー][node-view-five]
</center>

**fabric:/SpecialApp/ServiceA** サービスの **P1** パーティションのレプリカについて 'MultiTypeServicePackge' をアクティブ化した場合、'MyCodePackageA' はレプリカをホストしますが、'MyCodePackageB' は起動されて実行されます。 同様に、**fabric:/SpecialApp/ServiceB** サービスの **P3** パーティションのレプリカについて 'MultiTypeServicePackge' をアクティブ化した場合、'MyCodePackageB' はレプリカをホストしますが、'MyCodePackageA' は起動され実行されるだけです。その他のパターンでも同様です。 そのため、*ServicePackage* ごとの (異なる *ServiceTypes* を登録している) *CodePackage* の数が多くなれば、それだけリソース使用の冗長性が高くなることになります。 
 
 ただし、**共有プロセス** モデルで **fabric:/SpecialApp/ServiceA** と **fabric:/SpecialApp/ServiceB** というサービスを作成した場合、先にも述べたとおり、Service Fabric は **fabric:/SpecialApp** "*アプリケーション*" の 'MultiTypeServicePackge' のコピーを 1 つだけアクティブ化します。 'MyCodePackageA' は **fabric:/SpecialApp/ServiceA** サービス (正確には、'MyServiceTypeA' タイプのすべてのサービス) のすべてのレプリカをホストします。 'MyCodePackageB' は **fabric:/SpecialApp/ServiceB** サービス (正確には、'MyServiceTypeB' タイプのすべてのサービス) のすべてのレプリカをホストします。 次の図は、この設定のノードのビューを示しています。 

<center>
![デプロイされたアプリケーションのノードのビュー][node-view-six]
</center>

前の例を見ると、'MyCodePackageA' に 'MyServiceTypeA' と 'MyServiceTypeB' の両方が登録されていて、かつ 'MyCodePackageB' が存在しなければ、冗長な *CodePackage* は 1 つも実行されていないように見えるかもしれません。 ただし、前にも述べたとおり、これは正しいです。 このアプリケーション モデルは**専有プロセス** ホスティング モデルとは一致しません。 それぞれのレプリカに専用のプロセスを設定することが目的なのであれば、同じ *CodePackage* の両方の *ServiceTypes* を登録する必要はありません。 それぞれの *ServiceType* をそれ自身の *ServicePacakge* に設定するほうが自然です。

## <a name="next-steps"></a>次の手順
[アプリケーションをパッケージ化][a4]し、デプロイの準備を行います。

[アプリケーションのデプロイと削除][a5]に関するページでは、PowerShell を使用してアプリケーション インスタンスを管理する方法について説明しています。

<!--Image references-->
[node-view-one]: ./media/service-fabric-hosting-model/node-view-one.png
[node-view-two]: ./media/service-fabric-hosting-model/node-view-two.png
[node-view-three]: ./media/service-fabric-hosting-model/node-view-three.png
[node-view-four]: ./media/service-fabric-hosting-model/node-view-four.png
[node-view-five]: ./media/service-fabric-hosting-model/node-view-five.png
[node-view-six]: ./media/service-fabric-hosting-model/node-view-six.png

<!--Link references--In actual articles, you only need a single period before the slash-->
[a1]: service-fabric-application-model.md
[a2]: service-fabric-guest-executables-introduction.md
[a3]: service-fabric-containers-overview.md
[a4]: service-fabric-package-apps.md
[a5]: service-fabric-deploy-remove-applications.md

[r1]: https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-createservice

[c1]: https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync
[c2]: https://docs.microsoft.com/dotnet/api/system.fabric.description.statelessservicedescription.instancecount

[p1]: https://docs.microsoft.com/powershell/servicefabric/vlatest/new-servicefabricservice
[p2]: https://docs.microsoft.com/powershell/servicefabric/vlatest/get-servicefabricservicedescription
[p3]: https://docs.microsoft.com/powershell/servicefabric/vlatest/get-servicefabricdeployedservicePackage
[p4]: https://docs.microsoft.com/powershell/servicefabric/vlatest/send-servicefabricdeployedservicepackagehealthreport
[p5]: https://docs.microsoft.com/powershell/servicefabric/vlatest/restart-servicefabricdeployedcodepackage
[p6]: https://docs.microsoft.com/powershell/servicefabric/vlatest/get-servicefabricdeployedservicetype
[p7]: https://docs.microsoft.com/powershell/servicefabric/vlatest/get-servicefabricdeployedreplica
[p8]: https://docs.microsoft.com/powershell/servicefabric/vlatest/get-servicefabricdeployedcodepackage
