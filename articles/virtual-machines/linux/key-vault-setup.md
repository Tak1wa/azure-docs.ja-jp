---
title: "Linux VM の Azure Key Vault の設定 | Microsoft Docs"
description: "CLI 2.0 を使用して Azure Resource Manager 仮想マシンで Key Vault を設定する方法。"
services: virtual-machines-linux
documentationcenter: 
author: singhkays
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: bccdd5ab-5ccf-4760-9039-92c6eafb15bd
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: article
ms.date: 02/24/2017
ms.author: singhkay
ms.openlocfilehash: ed1a366819911302e70b2ebdce08f60920918593
ms.sourcegitcommit: 8c3267c34fc46c681ea476fee87f5fb0bf858f9e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/09/2018
---
# <a name="how-to-set-up-key-vault-for-virtual-machines-with-the-azure-cli-20"></a>Azure CLI 2.0 を使用して仮想マシン用に Key Vault を設定する方法

Azure Resource Manager スタックでは、Key Vault により提供されるリソースとしてシークレット/証明書がモデル化されます。 Azure Key Vault の詳細については、「 [Azure Key Vault とは](../../key-vault/key-vault-whatis.md) Azure Resource Manager VM と共に Key Vault を使用するには、Key Vault の *EnabledForDeployment* プロパティを True に設定する必要があります。 この記事では、Azure CLI 2.0 を使用して Azure 仮想マシン (VM) で Key Vault を設定する方法について説明します。 これらの手順は、[Azure CLI 1.0](key-vault-setup-cli-nodejs.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) を使用して実行することもできます。

これらの手順を実行するには、[Azure CLI 2.0](/cli/azure/install-az-cli2) の最新版をインストールし、[az login](/cli/azure/reference-index#az_login) を使用して Azure アカウントにログインする必要があります。

## <a name="create-a-key-vault"></a>Key Vault の作成
[az keyvault create](/cli/azure/keyvault#az_keyvault_create) で Key Vault を作成し、デプロイメント ポリシーを割り当てます。 次の例では、`myKeyVault` という名前の Key Vault を `myResourceGroup` リソース グループに作成します。

```azurecli
az keyvault create -l westus -n myKeyVault -g myResourceGroup --enabled-for-deployment true
```

## <a name="update-a-key-vault-for-use-with-vms"></a>VM で使用するように Key Vault を更新する
[az keyvault update](/cli/azure/keyvault#az_keyvault_update) で、デプロイメント ポリシーを既存の Key Vault に設定します。 次の例では、`myResourceGroup` リソース グループにある `myKeyVault` という名前の Key Vault を更新します。

```azurecli
az keyvault update -n myKeyVault -g myResourceGroup --set properties.enabledForDeployment=true
```

## <a name="use-templates-to-set-up-key-vault"></a>テンプレートを使用して Key Vault を設定する
テンプレートを使用する場合、次のように、Key Vault リソースの `enabledForDeployment` プロパティを `true` に設定する必要があります。

```json
{
    "type": "Microsoft.KeyVault/vaults",
    "name": "ContosoKeyVault",
    "apiVersion": "2015-06-01",
    "location": "<location-of-key-vault>",
    "properties": {
    "enabledForDeployment": "true",
    ....
    ....
    }
}
```

## <a name="next-steps"></a>次の手順
テンプレートを使用して、Key Vault の作成時に構成できるその他のオプションについては、「[Key Vault の作成](https://azure.microsoft.com/documentation/templates/101-key-vault-create/)」を参照してください。
