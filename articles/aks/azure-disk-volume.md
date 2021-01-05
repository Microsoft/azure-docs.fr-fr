---
title: Créer un volume statique pour des pods sur Azure Kubernetes Service (AKS)
description: Découvrez comment créer manuellement un volume avec des disques Azure pour une utilisation avec un pod sur Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 03/01/2019
ms.openlocfilehash: d44c8a7241308c26a3f1148ec70a7a5730dd0c89
ms.sourcegitcommit: 693df7d78dfd5393a28bf1508e3e7487e2132293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92900857"
---
# <a name="manually-create-and-use-a-volume-with-azure-disks-in-azure-kubernetes-service-aks"></a>Créer manuellement et utiliser un volume avec des disques Azure sur Azure Kubernetes Service (AKS)

Les applications basées sur des conteneurs doivent souvent consulter et conserver des données dans un volume de données externe. Si un pod unique a besoin d’accéder au stockage, vous pouvez utiliser des disques Azure pour présenter un volume natif pour une utilisation de l’application. Cet article vous montre comment créer manuellement un disque Azure et comment l’attacher à un pod sur AKS.

> [!NOTE]
> Un disque Azure ne peut être monté que sur un pod unique à la fois. S'il vous faut partager un volume persistant entre plusieurs pods, utilisez [Azure Files][azure-files-volume].

Pour plus d’informations sur les volumes Kubernetes, consultez [Options de stockage pour les applications dans AKS][concepts-storage].

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

Azure CLI 2.0.59 (ou une version ultérieure) doit également être installé et configuré. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

## <a name="create-an-azure-disk"></a>Créer un disque Azure

Lorsque vous créez un disque Azure pour une utilisation avec AKS, vous pouvez créer la ressource de disque sur le groupe de ressources de **nœuds** . Cette approche permet au cluster AKS d’accéder et de gérer la ressource de disque. Si, à la place, vous créez le disque dans un groupe de ressources distinct, vous devez donner au service principal Azure Kubernetes Service (AKS) pour votre cluster le rôle `Contributor` dans le groupe de ressource du disque. Vous pouvez également utiliser l’identité managée affectée par le système pour les autorisations, au lieu du principal du service. Pour plus d’informations, consultez [Utiliser des identités managées](use-managed-identity.md).

Pour cet article, créez le disque dans le groupe de ressources de nœuds. Tout d’abord, obtenez le nom du groupe de ressources avec la commande [az aks show][az-aks-show] et ajoutez le paramètre de requête `--query nodeResourceGroup`. L’exemple suivant obtient les le groupe de ressources du nœud pour le cluster AKS nommé *myAKSCluster* dans le groupe de ressources nommé *myResourceGroup* :

```azurecli-interactive
$ az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv

MC_myResourceGroup_myAKSCluster_eastus
```

À présent, créez un disque à l’aide de la commande [az disk create][az-disk-create]. Spécifiez en premier le nom de groupe de ressources de nœuds obtenu dans la commande précédente, puis celui pour la ressource de disque, par exemple *myAKSDisk* . L’exemple suivant crée un disque de *20* Gio et génère l’ID du disque après sa création. Si vous avez besoin de créer un disque pour une utilisation avec des conteneurs Windows Server, ajoutez le paramètre `--os-type windows` pour formater correctement le disque.

```azurecli-interactive
az disk create \
  --resource-group MC_myResourceGroup_myAKSCluster_eastus \
  --name myAKSDisk \
  --size-gb 20 \
  --query id --output tsv
```

> [!NOTE]
> Les disques Azure sont facturés par référence (SKU) pour une taille donnée. Ces références SKU vont de 32 Gio pour les disques S4 ou P4 à 32 Tio pour les disques S80 ou P80 (en préversion). Le débit et les performances d’E/S d’un disque managé Premium dépendent à la fois de la référence SKU et de la taille d’instance des nœuds dans le cluster AKS. Consultez [Tarification et niveau de performance des disques managés][managed-disk-pricing-performance].

L’ID de ressource de disque s’affiche une fois la commande complétée avec succès, comme illustré dans l’exemple de sortie suivant. Cet ID de disque est utilisé pour monter le disque à l’étape suivante.

```console
/subscriptions/<subscriptionID>/resourceGroups/MC_myAKSCluster_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
```

## <a name="mount-disk-as-volume"></a>Monter le disque en tant que volume

Pour monter le disque Azure dans votre pod, configurez le volume dans les spécifications du conteneur. Créez un fichier nommé `azure-disk-pod.yaml` avec le contenu suivant. Mettez à jour `diskName` avec le nom du disque créé à l’étape précédente, et `diskURI` avec l’ID du disque indiqué en sortie de la commande de création de disque. Si vous le souhaitez, mettez à jour `mountPath`. Il s’agit du chemin du disque Azure qui a été monté dans le pod. Pour les conteneurs Windows Server, spécifiez un *chemin de montage* en utilisant la convention de chemin Windows, par exemple, *'D:'* .

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    name: mypod
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
      - name: azure
        mountPath: /mnt/azure
  volumes:
      - name: azure
        azureDisk:
          kind: Managed
          diskName: myAKSDisk
          diskURI: /subscriptions/<subscriptionID>/resourceGroups/MC_myAKSCluster_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
```

Utilisez la commande `kubectl` pour créer le pod.

```console
kubectl apply -f azure-disk-pod.yaml
```

Vous disposez maintenant d’un pod en cours d’exécution sur lequel est monté un disque Azure à l’emplacement suivant : `/mnt/azure`. Vous pouvez utiliser `kubectl describe pod mypod` pour vérifier que le disque est monté correctement. La sortie de l’exemple condensé suivant montre le volume monté dans le conteneur :

```
[...]
Volumes:
  azure:
    Type:         AzureDisk (an Azure Data Disk mount on the host and bind mount to the pod)
    DiskName:     myAKSDisk
    DiskURI:      /subscriptions/<subscriptionID/resourceGroups/MC_myResourceGroupAKS_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
    Kind:         Managed
    FSType:       ext4
    CachingMode:  ReadWrite
    ReadOnly:     false
  default-token-z5sd7:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-z5sd7
    Optional:    false
[...]
Events:
  Type    Reason                 Age   From                               Message
  ----    ------                 ----  ----                               -------
  Normal  Scheduled              1m    default-scheduler                  Successfully assigned mypod to aks-nodepool1-79590246-0
  Normal  SuccessfulMountVolume  1m    kubelet, aks-nodepool1-79590246-0  MountVolume.SetUp succeeded for volume "default-token-z5sd7"
  Normal  SuccessfulMountVolume  41s   kubelet, aks-nodepool1-79590246-0  MountVolume.SetUp succeeded for volume "azure"
[...]
```

## <a name="next-steps"></a>Étapes suivantes

Pour connaître les meilleures pratiques associées, consultez [Meilleures pratiques relatives au stockage et aux sauvegardes dans Azure Kubernetes Service (AKS)][operator-best-practices-storage].

Pour plus d’informations sur l’interaction des clusters AKS avec les disques Azure, consultez le [Plug-in Kubernetes pour les disques Azure][kubernetes-disks].

<!-- LINKS - external -->
[kubernetes-disks]: https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_disk/README.md
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/volumes/
[managed-disk-pricing-performance]: https://azure.microsoft.com/pricing/details/managed-disks/

<!-- LINKS - internal -->
[az-disk-list]: /cli/azure/disk#az-disk-list
[az-disk-create]: /cli/azure/disk#az-disk-create
[az-group-list]: /cli/azure/group#az-group-list
[az-resource-show]: /cli/azure/resource#az-resource-show
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[az-aks-show]: /cli/azure/aks#az-aks-show
[install-azure-cli]: /cli/azure/install-azure-cli
[azure-files-volume]: azure-files-volume.md
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
