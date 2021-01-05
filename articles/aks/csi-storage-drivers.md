---
title: Activer des pilotes CSI (Container Storage interface) sur Azure Kubernetes Service (AKS)
description: Découvrez comment activer des pilotes CSI (Container Storage interface) pour des disques Azure et Azure Files dans un cluster Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 08/27/2020
author: palma21
ms.openlocfilehash: 2af4f9e2ea1dc0fcb8e5f40e0024297124292b49
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92074819"
---
# <a name="enable-container-storage-interface-csi-drivers-for-azure-disks-and-azure-files-on-azure-kubernetes-service-aks-preview"></a>Activer des pilotes CSI (Container Storage interface) pour des disques Azure et Azure Files sur Azure Kubernetes Service (AKS) (préversion)

CSI (Container Storage Interface) est une norme pour exposer des systèmes de stockage de blocs et de fichiers arbitraires à des charges de travail conteneurisées sur Kubernetes. En adoptant et en utilisant CSI, AKS (Azure Kubernetes Service) peut écrire, déployer et itérer des plug-ins afin d’exposer de nouveaux systèmes de stockage ou d’améliorer les systèmes existants dans Kubernetes sans avoir à toucher au code Kubernetes principal ou à attendre ses cycles de publication.

La prise en charge du pilote de stockage CSI sur AKS vous permet d’utiliser en mode natif :
- Des [*disques Azure*](azure-disk-csi.md), qui peuvent être utilisés pour créer une ressource *DataDisk* Kubernetes. Les disques peuvent utiliser un Stockage Premium Azure, assorti de disques SSD haute performance, ou un Stockage Standard Azure, assorti de disques HDD ordinaires ou de disques SSD standard. Pour la plupart des charges de travail de production et de développement, utilisez le Stockage Premium. Les disques Azure étant montés avec le mode d’accès *ReadWriteOnce*, ils ne sont disponibles que pour un seul pod. Pour les volumes de stockage accessibles par plusieurs pods simultanément, utilisez Azure Files.
- [*Azure Files*](azure-files-csi.md), qui peut être utilisé pour monter un partage SMB 3.0 assorti d’un compte Stockage Azure sur des pods. Avec Azure Files, vous pouvez partager des données entre plusieurs nœuds et pods. Azure Files peut utiliser un Stockage Standard Azure, assorti de disques HDD standard, ou un Stockage Premium Azure, assorti de disques SSD haute performance.

> [!IMPORTANT]
> À partir de Kubernetes version 1.21, Kubernetes utilise les pilotes CSI uniquement et par défaut. Ces pilotes représentent l’avenir de la prise en charge du stockage dans Kubernetes.
>
> Les *pilotes dans l’arborescence* font référence aux pilotes de stockage actuels, qui font partie du code Kubernetes principal, par opposition aux nouveaux pilotes CSI, qui sont des plug-ins.

## <a name="limitations"></a>Limites

- Cette fonctionnalité ne peut être définie qu’au moment de la création du cluster.
- La version mineure minimale Kubernetes qui prend en charge les pilotes CSI est v1.17.
- Pendant la préversion, la classe de stockage par défaut est toujours la [même classe de stockage dans l’arborescence](concepts-storage.md#storage-classes). Une fois cette fonctionnalité en disponibilité générale, la classe de stockage par défaut sera la classe de stockage `managed-csi` et les classes de stockage dans l’arborescence seront supprimées.
- Pendant la première phase de préversion, seul Azure CLI est pris en charge.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

### <a name="register-the-enableazurediskfilecsidriver-preview-feature"></a>Inscrire la fonctionnalité d’évaluation `EnableAzureDiskFileCSIDriver`

Pour créer un cluster AKS qui peut utiliser des pilotes CSI pour les disques Azure et Azure Files, vous devez activer l’indicateur de fonctionnalité `EnableAzureDiskFileCSIDriver` sur votre abonnement.

Inscrivez l’indicateur de fonctionnalité `EnableAzureDiskFileCSIDriver` à l’aide de la commande [az feature register][az-feature-register], comme indiqué dans l’exemple suivant :

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "EnableAzureDiskFileCSIDriver"
```

Quelques minutes sont nécessaires pour que l’état s’affiche *Registered* (Inscrit). Vérifiez l’état de l’inscription à l’aide de la commande [az feature list][az-feature-list] :

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/EnableAzureDiskFileCSIDriver')].{Name:name,State:properties.state}"
```

Lorsque vous êtes prêt, actualisez l’inscription du fournisseur de ressources *Microsoft.ContainerService* à l’aide de la commande [az provider register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

### <a name="install-aks-preview-cli-extension"></a>Installer l’extension CLI de préversion d’aks

Pour créer un cluster AKS ou un pool de nœuds pouvant utiliser les pilotes de stockage CSI, vous avez besoin de la dernière extension Azure CLI *aks-preview*. Installez l’extension Azure CLI *aks-preview* à l’aide de la commande [az extension add][az-extension-add]. Ou installez toutes les mises à jour disponibles à l’aide de la commande [az extension update][az-extension-update].

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
``` 


## <a name="create-a-new-cluster-that-can-use-csi-storage-drivers"></a>Créer un cluster qui peut utiliser des pilotes de stockage CSI

Créez un cluster capable d’utiliser des pilotes de stockage CSI pour les disques Azure et Azure Files à l’aide des commandes CLI suivantes. Utilisez l’indicateur `--aks-custom-headers` pour définir la fonctionnalité `EnableAzureDiskFileCSIDriver`.

Créez un groupe de ressources Azure :

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location canadacentral
```

Créez le cluster AKS avec prise en charge des pilotes de stockage CSI :

```azurecli-interactive
# Create an AKS-managed Azure AD cluster
az aks create -g MyResourceGroup -n MyManagedCluster --network-plugin azure -k 1.17.9 --aks-custom-headers EnableAzureDiskFileCSIDriver=true
```

Si vous souhaitez créer des clusters dans des pilotes de stockage dans l’arborescence au lieu de pilotes de stockage CSI, vous pouvez le faire en omettant le paramètre `--aks-custom-headers` personnalisé.


Vérifiez le nombre de volumes sur disque Azure que vous pouvez attacher à ce nœud en exécutant :

```console
$ kubectl get nodes
aks-nodepool1-25371499-vmss000000
aks-nodepool1-25371499-vmss000001
aks-nodepool1-25371499-vmss000002

$ echo $(kubectl get CSINode <NODE NAME> -o jsonpath="{.spec.drivers[1].allocatable.count}")
8
```

## <a name="next-steps"></a>Étapes suivantes

- Pour utiliser le lecteur CSI pour les disques Azure, consultez [Utiliser des disques Azure avec des pilotes CSI](azure-disk-csi.md).
- Pour utiliser le lecteur CSI pour Azure Files, consultez [Utiliser Azure Files avec des pilotes CSI](azure-files-csi.md).
- Pour plus d’informations sur les bonnes pratiques en matière de stockage, consultez [Meilleures pratiques relatives au stockage et aux sauvegardes dans Azure Kubernetes Service (AKS)][operator-best-practices-storage].

<!-- LINKS - external -->
[access-modes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-storage-classes]: https://kubernetes.io/docs/concepts/storage/storage-classes/
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
[managed-disk-pricing-performance]: https://azure.microsoft.com/pricing/details/managed-disks/

<!-- LINKS - internal -->
[azure-disk-volume]: azure-disk-volume.md
[azure-files-pvc]: azure-files-dynamic-pv.md
[premium-storage]: ../virtual-machines/disks-types.md
[az-disk-list]: /cli/azure/disk#az-disk-list
[az-snapshot-create]: /cli/azure/snapshot#az-snapshot-create
[az-disk-create]: /cli/azure/disk#az-disk-create
[az-disk-show]: /cli/azure/disk#az-disk-show
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
[storage-class-concepts]: concepts-storage.md#storage-classes
[az-extension-add]: /cli/azure/extension?view=azure-cli-latest#az-extension-add&preserve-view=true
[az-extension-update]: /cli/azure/extension?view=azure-cli-latest#az-extension-update&preserve-view=true
[az-feature-register]: /cli/azure/feature?view=azure-cli-latest#az-feature-register&preserve-view=true
[az-feature-list]: /cli/azure/feature?view=azure-cli-latest#az-feature-list&preserve-view=true
[az-provider-register]: /cli/azure/provider?view=azure-cli-latest#az-provider-register&preserve-view=true