---
title: Utiliser des groupes de placement de proximité pour réduire la latence dans les clusters AKS (Azure Kubernetes Service)
description: Découvrez comment utiliser des groupes de placement de proximité pour réduire la latence pour les charges de travail de votre cluster AKS.
services: container-service
manager: gwallace
ms.topic: article
ms.date: 10/19/2020
ms.openlocfilehash: fa81e293bc5e53a852bdb404f9e6d41c4297647b
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93349033"
---
# <a name="reduce-latency-with-proximity-placement-groups"></a>Réduire la latence au moyen de groupes de placement de proximité

> [!Note]
> Lors de l’utilisation de groupes de placement de proximité sur AKS, la colocation s’applique uniquement aux nœuds d’agent. La latence entre les nœuds et entre les pods hébergés correspondants est plus faible. La colocation n’affecte pas l’emplacement du plan de contrôle d’un cluster.

Quand vous déployez votre application dans Azure, la répartition des instances de machine virtuelle sur différentes régions ou zones de disponibilité engendre une latence au niveau du réseau, qui peut impacter les performances globales de votre application. Un groupe de placement de proximité est un regroupement logique utilisé pour s’assurer que les ressources de calcul Azure se trouvent proches les unes des autres. Certaines applications telles que les jeux, les simulations d’ingénierie et les transactions à haute fréquence (HFT) exigent que la latence soit faible et que les tâches s’exécutent rapidement. Dans ces types de scénarios de calcul haute performance (HPC), envisagez d’utiliser des [groupes de placement de proximité](../virtual-machines/linux/co-location.md#proximity-placement-groups) (PPG) pour les pools de nœuds de votre cluster.

## <a name="before-you-begin"></a>Avant de commencer

Pour les besoins de cet article, vous devez utiliser Azure CLI version 2.14 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli-install].

### <a name="limitations"></a>Limites

* Un groupe de placement de proximité peut mapper à une zone de disponibilité au maximum.
* Un pool de nœuds doit utiliser Virtual Machine Scale Sets pour associer un groupe de placement de proximité.
* Un pool de nœuds peut associer un groupe de placement de proximité uniquement lors de la création du pool de nœuds.

## <a name="node-pools-and-proximity-placement-groups"></a>Pools de nœuds et groupes de placement de proximité

La première ressource que vous déployez avec un groupe de placement de proximité est attachée à un centre de données spécifique. Les ressources supplémentaires déployées par la suite avec le même groupe de placement de proximité sont colocalisées dans le même centre de données. Une fois que toutes les ressources utilisant le groupe de placement de proximité ont été arrêtées (libérées) ou supprimées, elles ne sont plus attachées.

* Il est possible d’associer de nombreux pools de nœuds à un seul et même groupe de placement de proximité.
* Chaque pool de nœuds ne peut être associé qu’à un seul groupe de placement de proximité.

### <a name="configure-proximity-placement-groups-with-availability-zones"></a>Configurer des groupes de placement de proximité avec des zones de disponibilité

> [!NOTE]
> Alors que les groupes de placement de proximité nécessitent un pool de nœuds pour utiliser tout au plus une zone de disponibilité, le [SLA de machine virtuelle Azure de référence de 99,9 %](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_9/) est toujours en vigueur pour les machines virtuelles dans une seule zone.

Les groupes de placement de proximité représentent un concept de pool de nœuds et sont associés à chaque pool de nœuds individuels. L’utilisation d’une ressource PPG n’a aucun impact sur la disponibilité du plan de contrôle AKS. Cela peut avoir un impact sur la façon dont un cluster doit être conçu avec des zones. Pour garantir la répartition d’un cluster sur plusieurs zones, la conception suivante est recommandée.

* Approvisionnez un cluster avec le premier pool de nœuds système avec trois zones et sans groupe de placement de proximité associé. Cela garantit que les pods du système atterrissent dans un pool de nœuds dédié qui s’étendra sur plusieurs zones.
* Ajoutez des pools de nœuds utilisateur supplémentaires avec une zone unique et un groupe de placement de proximité associé à chaque pool. Par exemple, nodepool1 dans la zone 1 avec PPG1, nodepool2 dans la zone 2 avec PPG2 et nodepool3 dans la zone 3 avec PPG3. Cela garantit qu’au niveau d’un cluster, les nœuds sont répartis entre plusieurs zones et chaque pool de nœuds individuel est colocalisé dans la zone désignée avec une ressource PPG dédiée.

## <a name="create-a-new-aks-cluster-with-a-proximity-placement-group"></a>Créer un cluster AKS avec un groupe de placement de proximité

L’exemple suivant utilise la commande [az group create][az-group-create] pour créer un groupe de ressources nommé *myResourceGroup* dans la région *centralus*. Un cluster AKS nommé *myAKSCluster* est alors créé à l’aide de la commande [az aks create][az-aks-create].

L’accélération réseau améliore considérablement les performances des machines virtuelles sur le réseau. Dans l’idéal, utilisez des groupes de placement de proximité conjointement avec l’accélération réseau. Par défaut, AKS utilise l’accélération réseau sur des [instances de machines virtuelles prises en charge](../virtual-network/create-vm-accelerated-networking-cli.md?toc=/azure/virtual-machines/linux/toc.json#limitations-and-constraints), ce qui inclut la plupart des machines virtuelles Azure dotées de deux processeurs virtuels ou plus.

Créez un cluster AKS avec un groupe de placement de proximité associé au premier pool de nœuds système :

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location centralus
```
Exécutez la commande suivante et notez l’ID qui est retourné :

```azurecli-interactive
# Create proximity placement group
az ppg create -n myPPG -g myResourceGroup -l centralus -t standard
```

La commande génère une sortie, qui contient la valeur *id* à spécifier dans des commandes CLI ultérieures :

```output
{
  "availabilitySets": null,
  "colocationStatus": null,
  "id": "/subscriptions/yourSubscriptionID/resourceGroups/myResourceGroup/providers/Microsoft.Compute/proximityPlacementGroups/myPPG",
  "location": "centralus",
  "name": "myPPG",
  "proximityPlacementGroupType": "Standard",
  "resourceGroup": "myResourceGroup",
  "tags": {},
  "type": "Microsoft.Compute/proximityPlacementGroups",
  "virtualMachineScaleSets": null,
  "virtualMachines": null
}
```

Spécifiez l’ID de ressource du groupe de placement de proximité pour la valeur *myPPGResourceID* dans la commande ci-dessous :

```azurecli-interactive
# Create an AKS cluster that uses a proximity placement group for the initial system node pool only. The PPG has no effect on the cluster control plane.
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --ppg myPPGResourceID
```

## <a name="add-a-proximity-placement-group-to-an-existing-cluster"></a>Ajouter un groupe de placement de proximité à un cluster existant

Vous pouvez ajouter un groupe de placement de proximité à un cluster existant en créant un autre pool de nœuds. Vous pouvez ensuite éventuellement migrer les charges de travail existantes vers le nouveau pool de nœuds, puis supprimer le pool de nœuds initial.

Utilisez le même groupe de placement de proximité que celui que vous avez créé précédemment. Vous vous assurez ainsi que les nœuds d’agent dans les deux pools de nœuds de votre cluster AKS se trouvent physiquement dans le même centre de données.

Utilisez l’ID de ressource du groupe de placement de proximité que vous avez créé précédemment, puis ajoutez un nouveau pool de nœuds à l’aide de la commande [`az aks nodepool add`][az-aks-nodepool-add] :

```azurecli-interactive
# Add a new node pool that uses a proximity placement group, use a --node-count = 1 for testing
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-count 1 \
    --ppg myPPGResourceID
```

## <a name="clean-up"></a>Nettoyer

Pour supprimer le cluster, utilisez la commande [`az group delete`][az-group-delete] qui supprime le groupe de ressources AKS :

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

## <a name="next-steps"></a>Étapes suivantes

* Découvrez-en plus sur les [groupes de placement de proximité][proximity-placement-groups].

<!-- LINKS - Internal -->
[azure-ad-rbac]: azure-ad-rbac.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-get-upgrades]: /cli/azure/aks#az-aks-get-upgrades
[az-aks-upgrade]: /cli/azure/aks#az-aks-upgrade
[az-aks-show]: /cli/azure/aks#az-aks-show
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[proximity-placement-groups]: ../virtual-machines/linux/co-location.md#proximity-placement-groups
[az-aks-create]: /cli/azure/aks#az-aks-create
[system-pool]: ./use-system-pools.md
[az-aks-nodepool-add]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-add
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
