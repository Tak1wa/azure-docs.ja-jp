---
title: AKS で Azure Files を使用する
description: AKS での Azure ディスクの使用
services: container-service
author: neilpeterson
manager: timlt
ms.service: container-service
ms.topic: article
ms.date: 03/06/2018
ms.author: nepeters
ms.custom: mvc
ms.openlocfilehash: a5126bc4c5e7c9cd9832f33fc908e6c8b9e02b91
ms.sourcegitcommit: 8aab1aab0135fad24987a311b42a1c25a839e9f3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/16/2018
---
# <a name="persistent-volumes-with-azure-files"></a>Azure ファイルを含む永続ボリューム

永続ボリュームとは、Kubernetes クラスターで使用するためにプロビジョニングされたストレージの一部です。 永続ボリュームは 1 つまたは複数のポッドで使用でき、動的または静的にプロビジョニングできます。 このドキュメントでは、、Azure ファイル共有を AKS クラスター内の Kubernetes 永続ボリュームとして動的にプロビジョニングする方法について詳しく説明します。 

Kubernetes 永続ボリュームについて詳しくは、[Kubernetes 永続ボリューム][kubernetes-volumes]に関するページをご覧ください。

## <a name="create-storage-account"></a>[ストレージ アカウントの作成]

Azure ファイル共有を Kubernetes ボリュームとして動的にプロビジョニングするときは、AKS クラスターと同じリソース グループに含まれている限り、任意のストレージ アカウントを使用できます。 必要であれば、AKS クラスターと同じリソース グループ内にストレージ アカウントを作成します。 

適切なリソース グループを識別するには、[az group list][az-group-list] コマンドを使用します。

```azurecli-interactive
az group list --output table
```

`MC_clustername_clustername_locaton` のような名前のリソース グループを検索します。clustername は AKS クラスターの名前、location はクラスターが展開されている Azure リージョンです。

```
Name                                 Location    Status
-----------------------------------  ----------  ---------
MC_myAKSCluster_myAKSCluster_eastus  eastus      Succeeded
myAKSCluster                         eastus      Succeeded
```

[az storage account create][az-storage-account-create] コマンドを使用して、ストレージ アカウントを作成します。 

この例を参考にして、`--resource-group` をリソース グループの名前に、`--name` を任意の名前に更新します。

```azurecli-interactive
az storage account create --resource-group MC_myAKSCluster_myAKSCluster_eastus --name mystorageaccount --location eastus --sku Standard_LRS
```

## <a name="create-storage-class"></a>ストレージ クラスの作成

ストレージ クラスを使用して、Azure ファイル共有を作成する方法を定義します。 クラス内に特定のストレージ アカウントを指定できます。 ストレージ アカウントが指定されない場合は、`skuName` と `location` が指定される必要があり、関連するリソース グループ内のすべてのストレージ アカウントが一致するかどうかの評価が行われます。

Azure ファイル用の Kubernetes ストレージ クラスについて詳しくは、[Kubernetes ストレージ クラス][kubernetes-storage-classes]に関するページをご覧ください。

`azure-file-sc.yaml` という名前のファイルを作成し、そこに次のマニフェストをコピーします。 `storageAccount` をターゲットのストレージ アカウントの名前に更新します。

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: azurefile
provisioner: kubernetes.io/azure-file
parameters:
  storageAccount: mystorageaccount
```

[kubectl create][kubectl-create] コマンドを使用して、ストレージ クラスを作成します。

```azurecli-interactive
kubectl create -f azure-file-sc.yaml
```

## <a name="create-persistent-volume-claim"></a>永続ボリューム要求の作成

永続ボリューム要求 (PVC) は、ストレージ クラス オブジェクトを使用して、Azure ファイル共有を動的にプロビジョニングします。 

次のマニフェストを使用すると、サイズが `5GB` で `ReadWriteOnce` アクセスの永続ボリューム要求を作成できます。

`azure-file-pvc.yaml` という名前のファイルを作成し、そこに次のマニフェストをコピーします。 `storageClassName` が最後の手順で作成したストレージ クラスと一致していることを確認します。

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azurefile
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: azurefile
  resources:
    requests:
      storage: 5Gi
```

[kubectl create][kubectl-create] コマンドを使用して、永続ボリューム要求を作成します。

```azurecli-interactive
kubectl create -f azure-file-pvc.yaml
```

完了すると、ファイル共有が作成されます。 接続情報と資格情報を含む Kubernetes シークレットも作成されます。

## <a name="using-the-persistent-volume"></a>永続ボリュームの使用

次のマニフェストでは、永続ボリューム要求 `azurefile` を使用して、`/mnt/azure` パスに Azure ファイル共有をマウントするポッドが作成されます。

`azure-pvc-files.yaml` という名前のファイルを作成し、そこに次のマニフェストをコピーします。 `claimName` が最後の手順で作成したPVC と一致していることを確認します。

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/mnt/azure"
        name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: azurefile
```

[kubectl create][kubectl-create] コマンドを使用して、ポッドを作成します。

```azurecli-interactive
kubectl create -f azure-pvc-files.yaml
```

これで Azure ディスクが `/mnt/azure` ディレクトリにマウントされ、ポッドが稼働状態となりました。 このボリュームのマウントは、`kubectl describe pod mypod` でポッドを調べることにより確認できます。

## <a name="next-steps"></a>次の手順

Azure Files を使用した Kubernetes 永続ボリュームについて、さらに詳しい情報を確認します。

> [!div class="nextstepaction"]
> [Azure Files 対応の Kubernetes プラグイン][kubernetes-files]

<!-- LINKS - external -->
[access-modes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
[kubectl-create]: https://kubernetes.io/docs/user-guide/kubectl/v1.8/#create
[kubectl-describe]: https://kubernetes-v1-4.github.io/docs/user-guide/kubectl/kubectl_describe/
[kubernetes-files]: https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_file/README.md
[kubernetes-secret]: https://kubernetes.io/docs/concepts/configuration/secret/
[kubernetes-security-context]: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
[kubernetes-storage-classes]: https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-file
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/

<!-- LINKS - internal -->
[az-group-create]: /cli/azure/group#az_group_create
[az-group-list]: /cli/azure/group#az_group_list
[az-storage-account-create]: /cli/azure/storage/account#az_storage_account_create
[az-storage-create]: /cli/azure/storage/account#az_storage_account_create
[az-storage-key-list]: /cli/azure/storage/account/keys#az_storage_account_keys_list
[az-storage-share-create]: /cli/azure/storage/share#az_storage_share_create
