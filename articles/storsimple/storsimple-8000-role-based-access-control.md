---
title: "StorSimple でロールベースのアクセス制御を使用する | Microsoft Docs"
description: "StorSimple のコンテキストで Azure ロールベースのアクセス制御 (RBAC) を使用する方法について説明します。"
services: storsimple
documentationcenter: 
author: alkohli
manager: jconnoc
editor: 
ms.assetid: 
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/11/2017
ms.author: alkohli
ms.openlocfilehash: d040849360a47c611d44b3a5d7649c685dcc8068
ms.sourcegitcommit: d03907a25fb7f22bec6a33c9c91b877897e96197
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/12/2017
---
# <a name="role-based-access-control-for-storsimple"></a>StorSimple でロールベースのアクセス制御を使用する

この記事では、StorSimple デバイスで Azure ロールベースのアクセス制御 (RBAC) を使用する方法について、簡単に説明します。 RBAC は、Azure の粒度の細かいアクセス管理を提供します。 RBAC を使用して、StorSimple のすべてのユーザーに無制限のアクセス権を与える代わりに、仕事を行うために必要な適切なアクセス権をユーザーに付与します。 Azure でのアクセス管理の基本については、「[Azure ポータルでのロールベースの Access Control の基礎を確認する](../active-directory/role-based-access-control-what-is.md)」を参照してください。

この記事は、Azure ポータルで Update 3.0 以降を実行している StorSimple 8000 シリーズのデバイスに適用されます。

## <a name="rbac-roles-for-storsimple"></a>StorSimple の RBAC ロール

RBAC は、ロールに基づいて割り当てることができます。 ロールは、環境内で利用できるリソースに基づく特定のアクセス許可レベルを保証します。 StorSimple ユーザーが選択できるロールには、組み込みまたはカスタムの 2 種類があります。

* **組み込みのロール** - 組み込みのロールは、所有者、共同作成者、閲覧者、またはユーザー アクセス管理者が可能です。 詳細については、「[Azure ロールベースのアクセス制御の組み込みのロール](../active-directory/role-based-access-control-what-is.md#built-in-roles)」を参照してください。

* **カスタム ロール** - 組み込みのロールがニーズに適合しない場合は、StorSimple 用のカスタム RBAC ロールを作成できます。 カスタム RBAC ロールを作成するには、組み込みのロールから始めてそれを編集した後、環境にインポートします。 ロールのダウンロードとアップロードは Azure PowerShell または Azure CLI を使用して管理されます。 詳細については、「[Azure のロールベースのアクセス制御のためのカスタム ロールを作成する](../active-directory/role-based-access-control-custom-roles.md)」を参照してください。

Azure ポータルの StorSimple デバイス ユーザーのために使用できる異なるロールを表示するには、StorSimple デバイス マネージャー サービスに移動し、**[Access control (IAM)]、[ロール]** の順に選択します。


## <a name="create-a-custom-role-for-storsimple-infrastructure-administrator"></a>StorSimple インフラストラクチャ管理者用のカスタム ロールを作成する

次の例では、組み込みのロールである**閲覧者**から始めています。このロールのユーザーは、すべてのリソース スコープを表示できますが、編集または新規作成は実行できません。 このロールを拡張して、新しいカスタム ロールである StorSimple インフラストラクチャ管理者を作成します。このロールは、StorSimple デバイスのインフラストラクチャを管理できるユーザーに割り当てられます。

1. Windows PowerShell を管理者として実行します。

2. Azure にログインします。

    `Login-AzureRMAccount`

3. 閲覧者ロールを JSON テンプレートとしてコンピューターにエクスポートします。

    ```
    Get-AzureRMRoleDefinition -Name "Reader"

    Get-AzureRMRoleDefinition -Name "Reader" | ConvertTo-Json | Out-File C:\ssrbaccustom.json
    ```

4. Visual Studio で JSON ファイルを開きます。 標準的な RBAC ロールは、**Actions**、**NotActions**、**AssignableScopes** という 3 つのメイン セクションで構成されていることがわかります。

    **Actions** セクションには、このロールに許可されているすべての操作が記述されます。 各アクションはリソース プロバイダーから割り当てられます。 StorSimple インフラストラクチャ管理者では、`Microsoft.StorSimple` リソース プロバイダーを使用します。

    PowerShell を使用して、利用可能かつサブスクリプションに登録されているすべてのリソース プロバイダーを表示します。

    `Get-AzureRMResourceProvider`

    リソース プロバイダーの管理に利用できるすべての PowerShell コマンドレットを確認することもできます。

    **NotActions** セクションには、特定の RBAC ロールでは制限されているアクションが記述されます。 この例では、制限されているアクションはありません。
    
    **AssignableScopes** の下には、サブスクリプション ID が記述されます。 RBAC ロールが使用されるサブスクリプションの明示的なサブスクリプション ID がロールに含まれていることを確認します。 正しいサブスクリプション ID が指定されていない場合、サブスクリプションにロールをインポートすることはできません。

    前の考慮事項を念頭に置いて、ファイルを編集します。

    ```
    {
        "Name":  "StorSimple Infrastructure Admin",
        "Id":  "<guid>",
        "IsCustom":  true,
        "Description":  "Lets you view everything, but not make any changes except for Clear alerts, Clear settings, install, download etc.",
        "Actions":  [
                        "Microsoft.StorSimple/managers/alerts/read",
                        "Microsoft.StorSimple/managers/devices/volumeContainers/read",
                        "Microsoft.StorSimple/managers/devices/jobs/read",
                        "Microsoft.StorSimple/managers/devices/alertSettings/read",
                        "Microsoft.StorSimple/managers/devices/alertSettings/write",
                        "Microsoft.StorSimple/managers/clearAlerts/action",
                        "Microsoft.StorSimple/managers/devices/networkSettings/read",
                        "Microsoft.StorSimple/managers/devices/publishSupportPackage/action",
                        "Microsoft.StorSimple/managers/devices/scanForUpdates/action",
                        "Microsoft.StorSimple/managers/devices/metrics/read"

                    ],
        "NotActions":  [

                    ],
        "AssignableScopes":  [
                                "/subscriptions/<subscription_ID>/"
                            ]
    }
    ```

6. カスタム RBAC ロールを環境にインポートします。

    `New-AzureRMRoleDefinition -InputFile "C:\ssrbaccustom.json"`


これで、このロールが **[アクセス制御]** ブレードのロールの一覧に表示されます。

![RBAC ロールの表示](./media/storsimple-8000-role-based-access-control/rbac-role-types.png)

詳細については、[PowerShell を使用したカスタム RBAC ロールの作成](../active-directory/role-based-access-control-create-custom-roles-for-internal-external-users.md#create-a-custom-rbac-role-to-open-support-requests-using-powershell)に関する記事を参照してください。

### <a name="sample-output-for-custom-role-creation-via-the-powershell"></a>Powershell によるカスタム ロールの作成のサンプル出力

```
PS C:\WINDOWS\system32> Login-AzureRMAccount

Environment           : AzureCloud
Account               : john.doe@contoso.com
TenantId              : <tenant_ID>
SubscriptionId        : <subscription_ID>
SubscriptionName      : Internal Consumption
CurrentStorageAccount :

PS C:\WINDOWS\system32> Get-AzureRMRoleDefinition -Name "Reader"

Name             : Reader
Id               : <guid>
IsCustom         : False
Description      : Lets you view everything, but not make any changes.
Actions          : {*/read}
NotActions       : {}
AssignableScopes : {/}

PS C:\WINDOWS\system32> Get-AzureRMRoleDefinition -Name "Reader" | ConvertTo-Json | Out-File C:\ssrbaccustom.json

PS C:\WINDOWS\system32> New-AzureRMRoleDefinition -InputFile "C:\ssrbaccustom.json"

Name             : StorSimple Infrastructure Admin
Id               : <tenant_ID>
IsCustom         : True
Description      : Lets you view everything, but not make any changes except for Clear alerts, Clear settings, install,
                   download etc.
Actions          : {Microsoft.StorSimple/managers/alerts/read,
                   Microsoft.StorSimple/managers/devices/volumeContainers/read,
                   Microsoft.StorSimple/managers/devices/jobs/read,
                   Microsoft.StorSimple/managers/devices/alertSettings/read...}
NotActions       : {}
AssignableScopes : {/subscriptions/<subscription_ID>/}

PS C:\WINDOWS\system32>
```

## <a name="add-users-to-the-custom-role"></a>カスタム ロールにユーザーを追加する

アクセス権は、リソース内、リソース グループ内、またはサブスクリプション内から付与します。これは、ロール割り当てのスコープになります。 アクセス権を付与するときは、親ノードに付与されたアクセス権は子ノードに継承されることを忘れないでください。 詳細については、「[リソースの階層とアクセスの継承](../active-directory/role-based-access-control-what-is.md#resource-hierarchy-and-access-inheritance)」を参照してください。

1. **[アクセス制御 (IAM)]** を選択します。 アクセス制御ブレードで **[+追加]** を選択します。

    ![RBAC ロールにアクセス権を追加する](./media/storsimple-8000-role-based-access-control/rbac-add-role.png)

2. 割り当てるロールを選択します。ここでは、**[StorSimple インフラストラクチャ管理者]** を選択します。

3. ディレクトリで、アクセス権を付与するユーザー、グループ、またはアプリケーションを選択します。 ディレクトリは、表示名、電子メール アドレス、およびオブジェクト識別子を使用して検索できます。

4. **[保存]** を選択して割り当てを作成します。

    ![RBAC ロールにアクセス許可を追加する](./media/storsimple-8000-role-based-access-control/rbac-create-role-infra-admin.png)

**ユーザー追加中**通知によって進行状況が追跡されます。 ユーザーが正常に追加されると、アクセス制御のユーザーの一覧が更新されます。

## <a name="view-permissions-for-the-custom-role"></a>カスタム ロールのアクセス許可を表示する

ロールを作成した後、Azure ポータルでこのロールに関連付けられているアクセス許可を確認できます。

1. このロールに関連付けられているアクセス許可を表示するには、**[アクセス制御 (IAM)]、[ロール]、[StorSimple インフラストラクチャ管理者]** の順に移動します。このロールのユーザーの一覧が表示されます。

2. StorSimple インフラストラクチャ管理者ユーザーを選択し、**[アクセス許可]** をクリックします。

    ![StorSimple インフラストラクチャ管理者ロールのアクセス許可を表示する](./media/storsimple-8000-role-based-access-control/rbac-roles-view-permissions.png)

3. このロールに関連付けられているアクセス許可が表示されます。

    ![StorSimple インフラストラクチャ管理者ロールのユーザーを表示する](./media/storsimple-8000-role-based-access-control/rbac-infra-admin-permissions1.png)


## <a name="next-steps"></a>次のステップ

[内部ユーザーと外部ユーザーへのカスタム ロールの割り当て](../active-directory/role-based-access-control-create-custom-roles-for-internal-external-users.md)に関する記事で詳細を確認します。

