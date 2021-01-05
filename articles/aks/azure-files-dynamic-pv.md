---
title: Créer dynamiquement un partage Azure Files
titleSuffix: Azure Kubernetes Service
description: Découvrir comment créer un volume persistant de manière dynamique avec Azure Files pour une utilisation simultanée avec plusieurs pods dans Azure Kubernetes Service (ACS)
services: container-service
ms.topic: article
ms.date: 07/01/2020
ms.openlocfilehash: 2ad2affee34348e8c2fc7b734c8b49d0aec8db40
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96744907"
---
# <a name="dynamically-create-and-use-a-persistent-volume-with-azure-files-in-azure-kubernetes-service-aks"></a>Créer et utiliser un volume persistant de manière dynamique avec Azure Files dans Azure Kubernetes Service (AKS)

Un volume persistant représente un élément de stockage provisionné pour une utilisation dans des pods Kubernetes. Un volume persistant peut être utilisé par un ou plusieurs pods, et être provisionné de façon statique ou dynamique. Si plusieurs pods doivent accéder simultanément au même volume de stockage, vous pouvez utiliser Azure Files pour vous connecter à l’aide du [protocole SMB (Server Message Block)][smb-overview]. Cet article vous montre comment créer un partage Azure Files de manière dynamique utilisé par plusieurs pods dans un cluster Azure Kubernetes Service (AKS).

Pour plus d’informations sur les volumes Kubernetes, consultez [Options de stockage pour les applications dans AKS][concepts-storage].

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

Azure CLI 2.0.59 (ou une version ultérieure) doit également être installé et configuré. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

## <a name="create-a-storage-class"></a>Créer une classe de stockage

Une classe de stockage permet de définir la façon dont un partage de fichiers Azure est créé. Un compte de stockage est automatiquement créé dans le [groupe de ressources de nœud][node-resource-group] pour être utilisé avec la classe de stockage afin de contenir les partages de fichiers Azure. Faites votre choix parmi les [redondances de stockage Azure][storage-skus] suivantes pour *skuName* :

* *Standard_LRS* : stockage localement redondant (LRS) standard
* *Standard_GRS* : stockage géoredondant (GRS) standard
* *Standard_ZRS* : stockage redondant interzone (ZRS) standard
* *Standard_RAGRS* : stockage géographiquement redondant avec accès en lecture (RA-GRS) standard
* *Premium_LRS* - Stockage Premium localement redondant (LRS)
* *Premium_ZRS* : stockage redondant interzone (ZRS) premium

> [!NOTE]
> Azure Files prend en charge le stockage premium dans les clusters AKS qui exécutent Kubernetes 1.13 ou version ultérieure. Le partage de fichiers premium minimum est de 100 Go.

Pour plus d’informations sur les classes de stockage Kubernetes pour Azure Files, consultez [Classes de stockage Kubernetes][kubernetes-storage-classes].

Créez un fichier nommé `azure-file-sc.yaml` et copiez-le dans l’exemple de manifeste suivant. Pour plus d’informations sur *mountOptions*, consultez la section [Options de montage][mount-options].

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: my-azurefile
provisioner: kubernetes.io/azure-file
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict
  - actimeo=30
parameters:
  skuName: Standard_LRS
```

Créez la classe de stockage avec la commande [kubectl apply][kubectl-apply] :

```console
kubectl apply -f azure-file-sc.yaml
```

## <a name="create-a-persistent-volume-claim"></a>Créer une revendication de volume persistant

Une revendication de volume persistant utilise l’objet de classe de stockage pour provisionner dynamiquement un partage de fichiers Azure. Le code YAML suivant permet de créer une revendication de volume persistant d’une taille de *5 Go* avec un accès *ReadWriteMany*. Pour plus d’informations sur les modes d’accès, consultez la documentation [Kubernetes sur les volumes persistants][access-modes].

Maintenant, créez un fichier nommé `azure-file-pvc.yaml`, et copiez-y le code YAML suivant. Vérifiez que *storageClassName* correspond à la classe de stockage créée à la dernière étape :

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-azurefile
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: my-azurefile
  resources:
    requests:
      storage: 5Gi
```

> [!NOTE]
> Si vous utilisez la référence SKU *Premium_LRS* pour votre classe de stockage, la valeur minimale de *stockage* doit être *100 Gi*.

Créez la revendication de volume persistant avec la commande [kubectl apply][kubectl-apply] :

```console
kubectl apply -f azure-file-pvc.yaml
```

Une fois que c’est terminé, le partage de fichiers est créé. Un secret Kubernetes incluant des informations d’identification et des informations de connexion est également créé. Vous pouvez utiliser la commande [kubectl get][kubectl-get] pour voir l’état de la revendication de volume persistant :

```console
$ kubectl get pvc my-azurefile

NAME           STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
my-azurefile   Bound     pvc-8436e62e-a0d9-11e5-8521-5a8664dc0477   5Gi        RWX            my-azurefile      5m
```

## <a name="use-the-persistent-volume"></a>Utiliser le volume persistant

Le code YAML ci-après crée un pod qui utilise la revendication de volume persistant *my-azurefile* pour monter le partage de fichiers Azure dans le chemin */mnt/azure*. Pour les conteneurs Windows Server, spécifiez un *chemin de montage* en utilisant la convention de chemin Windows, par exemple, *'D:'* .

Créez un fichier nommé `azure-pvc-files.yaml` et copiez-y le code YAML suivant. Vérifiez que *claimName* correspond à la revendication de volume persistant créée à la dernière étape.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
    - mountPath: "/mnt/azure"
      name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: my-azurefile
```

Créez le pod avec la commande [kubectl apply][kubectl-apply].

```console
kubectl apply -f azure-pvc-files.yaml
```

Vous disposez maintenant d’un pod en cours d’exécution avec le partage Azure Files monté dans le répertoire */mnt/azure*. Cette configuration peut s’afficher lors de l’inspection de votre pod par le biais de `kubectl describe pod mypod`. La sortie de l’exemple condensé suivant montre le volume monté dans le conteneur :

```
Containers:
  mypod:
    Container ID:   docker://053bc9c0df72232d755aa040bfba8b533fa696b123876108dec400e364d2523e
    Image:          mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    Image ID:       docker-pullable://nginx@sha256:d85914d547a6c92faa39ce7058bd7529baacab7e0cd4255442b04577c4d1f424
    State:          Running
      Started:      Fri, 01 Mar 2019 23:56:16 +0000
    Ready:          True
    Mounts:
      /mnt/azure from volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8rv4z (ro)
[...]
Volumes:
  volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  my-azurefile
    ReadOnly:   false
[...]
```

## <a name="mount-options"></a>Options de montage

La valeur par défaut de *fileMode* et *dirMode* est *0777* pour Kubernetes 1.13.0 et versions ultérieures. Si vous créez le volume persistant de manière dynamique avec une classe de stockage, les options de montage peuvent être spécifiées sur l'objet de classe de stockage. L’exemple suivant définit *0777* :

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: my-azurefile
provisioner: kubernetes.io/azure-file
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict
  - actimeo=30
parameters:
  skuName: Standard_LRS
```

## <a name="next-steps"></a>Étapes suivantes

Pour connaître les meilleures pratiques associées, consultez [Meilleures pratiques relatives au stockage et aux sauvegardes dans Azure Kubernetes Service (AKS)][operator-best-practices-storage].

Pour connaître les paramètres de classe de stockage, consultez [Dynamic Provision](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/docs/driver-parameters.md#dynamic-provision) (Provisionnement dynamique).

Apprenez-en davantage sur les volumes persistants Kubernetes utilisant Azure Files.

> [!div class="nextstepaction"]
> [Plug-in Kubernetes pour Azure Files][kubernetes-files]

<!-- LINKS - external -->
[access-modes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-files]: https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_file/README.md
[kubernetes-secret]: https://kubernetes.io/docs/concepts/configuration/secret/
[kubernetes-security-context]: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
[kubernetes-storage-classes]: https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-file
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
[pv-static]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#static
[smb-overview]: /windows/desktop/FileIO/microsoft-smb-protocol-and-cifs-protocol-overview

<!-- LINKS - internal -->
[az-group-create]: /cli/azure/group#az-group-create
[az-group-list]: /cli/azure/group#az-group-list
[az-resource-show]: /cli/azure/aks#az-aks-show
[az-storage-account-create]: /cli/azure/storage/account#az-storage-account-create
[az-storage-create]: /cli/azure/storage/account#az-storage-account-create
[az-storage-key-list]: /cli/azure/storage/account/keys#az-storage-account-keys-list
[az-storage-share-create]: /cli/azure/storage/share#az-storage-share-create
[mount-options]: #mount-options
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[storage-skus]: ../storage/common/storage-redundancy.md
[kubernetes-rbac]: concepts-identity.md#role-based-access-controls-rbac
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
[node-resource-group]: faq.md#why-are-two-resource-groups-created-with-aks
