---
title: "オンプレミスのデータ ゲートウェイをインストールする | Microsoft Docs"
description: "オンプレミスのデータ ゲートウェイをインストールして構成する方法について説明します。"
services: analysis-services
documentationcenter: 
author: minewiskan
manager: kfile
editor: 
tags: 
ms.assetid: 
ms.service: analysis-services
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: na
ms.date: 02/14/2018
ms.author: owend
ms.openlocfilehash: c2cbe1c60f67c689a38d1585245610a6fa73bff4
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/21/2018
---
# <a name="install-and-configure-an-on-premises-data-gateway"></a>オンプレミスのデータ ゲートウェイをインストールして構成する
同じリージョン内の 1 つまたは複数の Azure Analysis Services サーバーがオンプレミスのデータ ソースに接続する場合は、オンプレミスのデータ ゲートウェイが必要です。 ゲートウェイの詳細については、[オンプレミスのデータ ゲートウェイ](analysis-services-gateway.md)に関する記事を参照してください。

## <a name="prerequisites"></a>前提条件
**最低限必要なもの**

* .NET Framework 4.5
* Windows 7/Windows Server 2008 R2 (以降) の 64 ビット版

**推奨されるもの:**

* 8 コア CPU
* 8 GB メモリ
* Windows 2012 R2 (以降) の 64 ビット バージョン

**重要な考慮事項**

* セットアップ中に、ゲートウェイを Azure に登録するときは、サブスクリプションの既定のリージョンが選択されます。 別のリージョンを選択できます。 複数のリージョンにサーバーがある場合は、各リージョン用のゲートウェイをインストールする必要があります。 
* ドメイン コントローラーにゲートウェイをインストールすることはできません。
* 1 台のコンピューターにインストールできるゲートウェイは 1 つだけです。
* 常に稼働していてスリープ状態にならないコンピューターにゲートウェイをインストールします。
* ネットワークにワイヤレス接続されているコンピューターにはゲートウェイをインストールしないでください。 パフォーマンスが低下することがあります。
* ゲートウェイを登録するサブスクリプションと同じ[テナント](https://msdn.microsoft.com/library/azure/jj573650.aspx#BKMK_WhatIsAnAzureADTenant)の Azure AD アカウントを使用して Azure にサインインします。 ゲートウェイのインストールと登録では、Azure B2B (guest) アカウントはサポートされません。


## <a name="download"></a>ダウンロード
 [ゲートウェイをダウンロードする](https://aka.ms/azureasgateway)

## <a name="install"></a>インストール

1. セットアップを実行します。

2. 場所を選択し、条項に同意し、**[インストール]** をクリックします。

   ![インストール場所とライセンス条項](media/analysis-services-gateway-install/aas-gateway-installer-accept.png)

3. Azure にサインインします。 アカウントは、テナントの Azure Active Directory でなければなりません。 このアカウントがゲートウェイ管理者で使用されます。 ゲートウェイのインストールと登録では、Azure B2B (guest) アカウントはサポートされません。

   ![Azure へのサインイン](media/analysis-services-gateway-install/aas-gateway-installer-account.png)

   > [!NOTE]
   > ドメイン アカウントでサインインした場合は、Azure AD 内の組織アカウントにマップされます。 組織アカウントがゲートウェイ管理者として使用されます。

## <a name="register"></a>登録
Azure 内にゲートウェイ リソースを作成するためには、ゲートウェイ クラウド サービスを使用してインストールしたローカル インスタンスを登録する必要があります。 

1.  **[このコンピューターに新しいゲートウェイを登録します]** を選択します。

    ![Register](media/analysis-services-gateway-install/aas-gateway-register-new.png)

2. ゲートウェイの名前と回復キーを入力します。 既定では、ゲートウェイは、サブスクリプションの既定のリージョンを使用します。 別のリージョンを選択する必要がある場合は、**[リージョンの変更]** を選択します。

    > [!IMPORTANT]
    > 回復キーを安全な場所に保存します。 ゲートウェイの引き継ぎ、移行、復元には回復キーが必要となります。 

   ![Register](media/analysis-services-gateway-install/aas-gateway-register-name.png)


## <a name="create-resource"></a>Azure ゲートウェイ リソースを作成する
ゲートウェイをインストールして登録した後、Azure サブスクリプションにゲートウェイ リソースを作成する必要があります。 ゲートウェイを登録するときに使用したのと同じアカウントを使用して Azure にサインインします。

1. Azure ポータルで、**[新しいサービスの作成]** > **[エンタープライズ統合]** > **[オンプレミスのデータ ゲートウェイ]** > **[作成]** をクリックします。

   ![ゲートウェイ リソースを作成する](media/analysis-services-gateway-install/aas-gateway-new-azure-resource.png)

2. **[接続ゲートウェイの作成]** で、次の設定を入力します。

    * **名前**: ゲートウェイ リソースの名前を入力します。 

    * **サブスクリプション**: ゲートウェイ リソースに関連付ける Azure サブスクリプションを選択します。 
   
      既定のサブスクリプションは、サインインするために使用した Azure アカウントに基づきます。

    * **[リソース グループ]**: リソース グループを作成するか、既存のリソース グループを選択します。

    * **場所**: ゲートウェイを登録したリージョンを選択します。

    * **インストール名**: ゲートウェイのインストールが既に選択されていない場合は、登録済みのゲートウェイを選択します。 

    完了したら、 **[作成]**をクリックします。

## <a name="connect-servers"></a>サーバーをゲートウェイ リソースに接続する

1. Azure Analysis Services サーバーの概要で、**[オンプレミスのデータ ゲートウェイ]** をクリックします。

   ![サーバーをゲートウェイに接続する](media/analysis-services-gateway-install/aas-gateway-connect-server.png)

2. **[接続するオンプレミスのデータ ゲートウェイを選択します:]** で、ゲートウェイ リソースを選択し、**[選択されたゲートウェイの接続]** をクリックします。

   ![サーバーをゲートウェイ リソースに接続する](media/analysis-services-gateway-install/aas-gateway-connect-resource.png)

    > [!NOTE]
    > ゲートウェイが一覧に表示されない場合、ゲートウェイの登録時に指定したのと同じリージョンにサーバーが存在していない可能性があります。 

これで終了です。 ポートを開くか、トラブルシューティングを実行する必要がある場合は、[オンプレミスのデータ ゲートウェイ](analysis-services-gateway.md)に関する記事を必ず確認してください。

## <a name="next-steps"></a>次の手順
* [Analysis Services を管理する](analysis-services-manage.md)   
* [Azure Analysis Services からデータを取得する](analysis-services-connect.md)
