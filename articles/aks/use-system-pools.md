---
title: Utiliser des pools de nœuds système dans Azure Kubernetes Service (AKS)
description: Découvrez comment créer et gérer des pools de nœuds système dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 06/18/2020
ms.author: mlearned
ms.custom: fasttrack-edit, devx-track-azurecli
ms.openlocfilehash: 9c9479fca538c36f4f5eb430c4befb76e39370e6
ms.sourcegitcommit: 693df7d78dfd5393a28bf1508e3e7487e2132293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92900012"
---
# <a name="manage-system-node-pools-in-azure-kubernetes-service-aks"></a>Gérer des pools de nœuds système dans Azure Kubernetes Service (AKS)

Dans Azure Kubernetes Service (AKS), les nœuds d’une même configuration sont regroupés dans des *pools de nœuds* . Les pools de nœuds contiennent les machines virtuelles sous-jacentes qui exécutent vos applications. Les pools de nœuds système et les pools de nœuds utilisateur sont deux modes de pool de nœuds différents pour vos clusters AKS. Les pools de nœuds système sont principalement utilisés pour héberger des pods système critiques, tels que `CoreDNS` et `metrics-server`. Les pools de nœuds utilisateur sont principalement utilisés pour héberger vos pods d'application. Cela étant, les pods d’application peuvent être planifiés sur des pools de nœuds système si vous souhaitez n'avoir qu’un pool dans votre cluster AKS. Chaque cluster AKS doit contenir au moins un pool de nœuds système avec au moins un nœud.

> [!Important]
> Si vous exécutez un pool de nœuds système unique pour votre cluster AKS dans un environnement de production, nous vous recommandons d’utiliser au moins trois nœuds pour le pool de nœuds.

## <a name="before-you-begin"></a>Avant de commencer

* La version 2.3.1 (ou ultérieure) d’Azure CLI doit être installée et configurée. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

## <a name="limitations"></a>Limites

Les limitations suivantes s’appliquent lorsque vous créez et gérez les clusters AKS prenant en charge les pools de nœuds système.

* Voir [Quotas, restrictions de taille de machine virtuelle et disponibilité des régions dans Azure Kubernetes Service (AKS)][quotas-skus-regions].
* Le cluster AKS doit être élaboré avec des groupes de machines virtuelles identiques en tant que type de machine virtuelle et l’équilibreur de charge de niveau tarifaire *Standard* .
* Le nom d’un pool de nœuds ne peut contenir que des caractères alphanumériques minuscules et doit commencer par une lettre minuscule. Pour les pools de nœuds Linux, la longueur doit être comprise entre 1 et 12 caractères. Pour les pools de nœuds Windows, la longueur doit être comprise entre 1 et 6 caractères.
* Pour définir un mode de pool de nœuds, vous devez utiliser l’API version 2020-03-01 ou ultérieure. Les clusters créés sur des versions d’API antérieures à 2020-03-01 contiennent uniquement des pools de nœuds utilisateur, mais ils peuvent être migrés pour contenir des pools de nœuds système en suivant les [étapes de mise à jour du mode de pool](#update-existing-cluster-system-and-user-node-pools).
* Le mode d’un pool de nœuds est une propriété obligatoire qui doit être définie explicitement si vous utilisez des modèles ARM ou des appels d’API directs.

## <a name="system-and-user-node-pools"></a>Pools de nœuds système et utilisateur

Pour un pool de nœuds système, AKS attribue automatiquement l’étiquette **kubernetes.azure.com/mode: system** à ses nœuds. C’est pourquoi AKS préfère planifier des pods du système sur les pools de nœuds qui portent cette étiquette. Cette étiquette ne vous empêche pas de planifier des pods d’application sur des pools de nœuds système. Cependant, nous vous recommandons d’isoler les pods système cruciaux de vos pods d’application afin d’éviter que des pods d’application mal configurés ou malveillants ne tuent accidentellement les pods système. Vous pouvez appliquer ce comportement en créant un pool de nœuds système dédié. Utilisez la teinte `CriticalAddonsOnly=true:NoSchedule` pour empêcher la planification de pods d’application sur les pools de nœuds système.

Les pools de nœuds système présentent les restrictions suivantes :

* Le paramètre osType des pools de nœuds système doit être Linux.
* Le paramètre osType des pools de nœuds utilisateur peut être Linux ou Windows.
* Les pools de nœuds système doivent contenir au moins un nœud, et les pools de nœuds utilisateur peuvent contenir zéro, un ou plusieurs nœuds.
* Les pools de nœuds système nécessitent une référence SKU de machine virtuelle d’au moins 2 processeurs virtuels et de 4 Go de mémoire.
* Les pools de nœuds système doivent prendre en charge au moins 30 pods, comme décrit par la [formule des valeurs minimale et maximale pour les pods][maximum-pods].
* Les pools de nœuds Spot nécessitent des pools de nœuds utilisateur.
* L’ajout d’un pool de nœuds système supplémentaire ou la modification du pool de nœuds système ne déplace *PAS* automatiquement les pods. Les pods système peuvent continuer à s’exécuter sur le même pool de nœuds, même si vous le transformez en pool de nœuds utilisateur. Si vous supprimez ou réduisez la taille d’un pool de nœuds exécutant des pods système correspondant précédemment à un pool de nœuds système, ces pods système sont redéployés avec la planification préféré vers le nouveau pool de nœuds système.

Vous pouvez effectuer les opérations suivantes avec des pools de nœuds :

* Créer un pool de nœuds système dédié (préférer la planification de pods système aux pools de nœuds de `mode:system`)
* Modifiez un pool de nœuds système en pool de nœuds utilisateur, à condition de disposer d’un autre pool de nœuds système pour prendre sa place dans le cluster AKS.
* Modifiez un pool de nœuds utilisateur en pool de nœuds système.
* Supprimez des pools de nœuds utilisateur.
* Vous pouvez supprimer un pool de nœuds système à condition de disposer d’un autre pool de nœuds système pour prendre sa place dans le cluster AKS.
* Un cluster AKS peut avoir plusieurs pools de nœuds système et nécessite au moins un pool de nœuds système.
* Si vous souhaitez modifier différents paramètres immuables sur des pools de nœuds existants, vous pouvez créer de nouveaux pools de nœuds pour les remplacer. Un exemple consiste à ajouter un nouveau pool de nœuds avec un nouveau paramètre maxPods et à supprimer l’ancien pool de nœuds.

## <a name="create-a-new-aks-cluster-with-a-system-node-pool"></a>Créer un cluster AKS avec un pool de nœuds système

Lorsque vous créez un cluster AKS, vous créez automatiquement un pool de nœuds système avec un nœud unique. Par défaut, le pool de nœuds initial est en mode de type système. Lorsque vous créez des pools de nœuds avec `az aks nodepool add`, ces pools de nœuds sont des pools de nœuds utilisateur, sauf si vous spécifiez explicitement le paramètre mode.

L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* dans la région *USA Est* .

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Utilisez la commande [az aks create][az-aks-create] pour créer un cluster AKS. L’exemple suivant crée un cluster nommé *myAKSCluster* avec un pool système dédié contenant un seul nœud. Pour vos charges de travail de production, assurez-vous d’utiliser des pools de nœuds système avec au moins trois nœuds. Cette opération peut prendre plusieurs minutes.

```azurecli-interactive
# Create a new AKS cluster with a single system pool
az aks create -g myResourceGroup --name myAKSCluster --node-count 1 --generate-ssh-keys
```

## <a name="add-a-dedicated-system-node-pool-to-an-existing-aks-cluster"></a>Ajouter un pool de nœuds système dédié à un cluster AKS existant

> [!Important]
> Vous ne pouvez pas modifier les teintes de nœuds par l’intermédiaire de l’interface CLI après la création du pool de nœuds.

Vous pouvez ajouter un ou plusieurs pools de nœuds système à des clusters AKS existants. Il est recommandé de planifier vos pods d’application sur des pools de nœuds utilisateur et de dédier des pools de nœuds système uniquement aux pods système qui sont cruciaux. Cela empêche les pods d’application malveillants de tuer accidentellement des pods système. Appliquez ce comportement avec la [teinte][aks-taints] `CriticalAddonsOnly=true:NoSchedule` pour vos pools de nœuds système. 

La commande suivante ajoute un pool de nœuds dédié en mode de type système avec trois nœuds, par défaut.

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name systempool \
    --node-count 3 \
    --node-taints CriticalAddonsOnly=true:NoSchedule \
    --mode System
```
## <a name="show-details-for-your-node-pool"></a>Afficher les détails de votre pool de nœuds

Vous pouvez vérifier les détails de votre pool de nœuds à l’aide de la commande suivante.  

```azurecli-interactive
az aks nodepool show -g myResourceGroup --cluster-name myAKSCluster -n systempool
```

Un mode de type **Système** est défini pour les pools de nœuds système, et un mode de type **Utilisateur** est défini pour les pools de nœuds utilisateur. Dans le cas d’un pool système, vérifiez que la teinte est définie sur `CriticalAddonsOnly=true:NoSchedule`, ce qui empêchera les pods d’application d’être planifiées sur ce pool de nœuds.

```output
{
  "agentPoolType": "VirtualMachineScaleSets",
  "availabilityZones": null,
  "count": 1,
  "enableAutoScaling": null,
  "enableNodePublicIp": false,
  "id": "/subscriptions/yourSubscriptionId/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster/agentPools/systempool",
  "maxCount": null,
  "maxPods": 110,
  "minCount": null,
  "mode": "System",
  "name": "systempool",
  "nodeImageVersion": "AKSUbuntu-1604-2020.06.30",
  "nodeLabels": {},
  "nodeTaints": [
    "CriticalAddonsOnly=true:NoSchedule"
  ],
  "orchestratorVersion": "1.16.10",
  "osDiskSizeGb": 128,
  "osType": "Linux",
  "provisioningState": "Failed",
  "proximityPlacementGroupId": null,
  "resourceGroup": "myResourceGroup",
  "scaleSetEvictionPolicy": null,
  "scaleSetPriority": null,
  "spotMaxPrice": null,
  "tags": null,
  "type": "Microsoft.ContainerService/managedClusters/agentPools",
  "upgradeSettings": {
    "maxSurge": null
  },
  "vmSize": "Standard_DS2_v2",
  "vnetSubnetId": null
}
```

## <a name="update-existing-cluster-system-and-user-node-pools"></a>Mettre à jour les pools de nœuds utilisateur et système existants dans un cluster

> [!NOTE]
> Pour définir un mode de pool de nœuds système, vous devez utiliser l’API version 2020-03-01 ou ultérieure. Les clusters créés sur des versions d’API antérieures à 2020-03-01 contiennent uniquement des pools de nœuds utilisateur. Pour bénéficier des fonctionnalités des pools de nœuds système et de leurs avantages sur des clusters plus anciens, mettez à jour le mode des pools de nœuds existants en utilisant les commandes suivantes dans la dernière version d’Azure CLI.

Vous pouvez modifier les modes pour les pools de nœuds système et utilisateur. Vous pouvez modifier un pool de nœuds système en pool de nœuds utilisateur seulement si un autre pool de nœuds système existe déjà sur le cluster AKS.

Cette commande modifie un pool de nœuds système en pool de nœuds utilisateur.

```azurecli-interactive
az aks nodepool update -g myResourceGroup --cluster-name myAKSCluster -n mynodepool --mode user
```

Cette commande modifie un pool de nœuds utilisateur en pool de nœuds système.

```azurecli-interactive
az aks nodepool update -g myResourceGroup --cluster-name myAKSCluster -n mynodepool --mode system
```

## <a name="delete-a-system-node-pool"></a>Supprimer un pool de nœuds système

> [!Note]
> Pour utiliser des pools de nœuds système sur des clusters AKS avant l’API version 2020-03-02, ajoutez un nouveau pool de nœuds système, puis supprimez le pool de nœuds par défaut d’origine.

Vous devez disposer d’au moins deux pools de nœuds système sur votre cluster AKS pour pouvoir en supprimer un.

```azurecli-interactive
az aks nodepool delete -g myResourceGroup --cluster-name myAKSCluster -n mynodepool
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer le cluster, utilisez la commande [az group delete][az-group-delete] pour supprimer le groupe de ressources AKS :

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```



## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris comment créer et gérer des pools de nœuds système dans un cluster AKS. Pour plus d’informations sur l’utilisation de plusieurs pools de nœuds, consultez [Utiliser plusieurs pools de nœuds][use-multiple-node-pools].

<!-- EXTERNAL LINKS -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-taint]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#taint
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubernetes-labels]: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
[kubernetes-label-syntax]: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set

<!-- INTERNAL LINKS -->
[aks-taints]: use-multiple-node-pools.md#setting-nodepool-taints
[aks-windows]: windows-container-cli.md
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-nodepool-add]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-add
[az-aks-nodepool-list]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-list
[az-aks-nodepool-update]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-update
[az-aks-nodepool-upgrade]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-upgrade
[az-aks-nodepool-scale]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-scale
[az-aks-nodepool-delete]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-delete
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[az-group-deployment-create]: /cli/azure/group/deployment#az-group-deployment-create
[gpu-cluster]: gpu-cluster.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[quotas-skus-regions]: quotas-skus-regions.md
[supported-versions]: supported-kubernetes-versions.md
[tag-limitation]: ../azure-resource-manager/management/tag-resources.md
[taints-tolerations]: operator-best-practices-advanced-scheduler.md#provide-dedicated-nodes-using-taints-and-tolerations
[vm-sizes]: ../virtual-machines/sizes.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[maximum-pods]: configure-azure-cni.md#maximum-pods-per-node
