---
title: Ajouter un pool de nœuds spot à un cluster Azure Kubernetes Service (AKS)
description: Découvrez comment ajouter un pool de nœuds spot à un cluster Azure Kubernetes Service (AKS).
services: container-service
ms.service: container-service
ms.topic: article
ms.date: 10/19/2020
ms.openlocfilehash: 5fd97560c3a6e41b49beb957c7b8d79369799c21
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93078949"
---
# <a name="add-a-spot-node-pool-to-an-azure-kubernetes-service-aks-cluster"></a>Ajouter un pool de nœuds spot à un cluster Azure Kubernetes Service (AKS)

Un pool de nœuds spot est un pool de nœuds associé à un [groupe de machines virtuelles identiques spot][vmss-spot]. L’utilisation de machines virtuelles spot pour les nœuds avec votre cluster AKS vous permet de tirer parti de la capacité inutilisée dans Azure tout en réalisant des économies substantielles. La quantité disponible de capacité non utilisée peut varier en fonction de nombreux facteurs, comme la taille des nœuds, la région et l’heure de la journée.

Lors du déploiement d’un pool de nœuds spot, Azure alloue les nœuds spot si de la capacité est disponible. Il n’existe, par contre, aucun contrat de niveau de service pour les nœuds spot. Un groupe identique spot, sur lequel s’appuie le pool de nœuds spot, est déployé dans un domaine d’erreur unique. Il n’offre aucune garantie de haute disponibilité. Dès qu’Azure a besoin de récupérer toute la capacité, l’infrastructure Azure supprime les nœuds spot.

Les nœuds spot sont idéaux pour les charges de travail qui peuvent gérer les interruptions, les arrêts prématurés ou les évictions. Par exemple, les charges de travail telles que les travaux de traitement par lots, les environnements de développement et de test ainsi que les charges de calcul importantes peuvent se révéler de bons candidats pour la planification sur un pool de nœuds spot.

Dans cet article, vous ajoutez un pool de nœuds spot secondaire à un cluster Azure Kubernetes Service (AKS) existant.

Cet article suppose une compréhension élémentaire des concepts de Kubernetes et d’Azure Load Balancer. Pour plus d’informations, consultez [Concepts de base de Kubernetes pour AKS (Azure Kubernetes Service)][kubernetes-concepts].

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="before-you-begin"></a>Avant de commencer

Lorsque vous créez un cluster pour utiliser un pool de nœuds spot, le cluster en question doit également utiliser Virtual Machine Scale Sets pour les pools de nœuds et l’équilibreur de charge de référence SKU *Standard*. Après avoir créé votre cluster, vous devez également ajouter un pool de nœuds supplémentaire pour utiliser un pool de nœuds spot. L’ajout d’un pool de nœuds supplémentaire est abordé dans une étape ultérieure.

Pour les besoins de cet article, vous devez utiliser Azure CLI version 2.14 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli-install].

### <a name="limitations"></a>Limites

Les limitations suivantes s’appliquent lorsque vous créez et gérez les clusters AKS avec un pool de nœuds spot :

* Un pool de nœuds spot ne peut pas être le pool de nœuds par défaut du cluster. Un pool de nœuds spot ne peut être utilisé que pour un pool secondaire.
* Vous ne pouvez pas mettre à niveau un pool de nœuds spot, car ce type de pool ne peut pas garantir l’isolation et le drainage. Pour effectuer des opérations telles que la mise à niveau de la version Kubernetes, vous devez remplacer votre pool de nœuds spot existant par un nouveau pool. Pour remplacer un pool de nœuds spot, créez un nouveau pool de nœuds spot avec une autre version de Kubernetes, attendez que son état soit *Prêt* , puis supprimez l’ancien pool de nœuds.
* Le plan de contrôle et les pools de nœuds ne peuvent pas être mis à niveau en même temps. Vous devez les mettre à niveau séparément, ou supprimer le pool de nœuds spot pour mettre à niveau le plan de contrôle et les pools de nœuds restants en même temps.
* Un pool de nœuds spot doit utiliser Virtual Machine Scale Sets.
* Vous ne pouvez pas modifier ScaleSetPriority ou SpotMaxPrice après la création.
* Lors de la définition de SpotMaxPrice, la valeur doit être -1 ou une valeur positive allant jusqu’à cinq décimales.
* Un pool de nœuds spot aura l’étiquette *kubernetes.azure.com/scalesetpriority:spot* , la teinte *kubernetes.azure.com/scalesetpriority=spot:NoSchedule* et les pods système auront une anti-affinité.
* Vous devez ajouter une [tolérance correspondante][spot-toleration] pour planifier des charges de travail sur un pool de nœuds spot.

## <a name="add-a-spot-node-pool-to-an-aks-cluster"></a>Ajouter un pool de nœuds spot à un cluster AKS

Vous devez ajouter un pool de nœuds spot à un cluster existant pour lequel plusieurs pools de nœuds sont activés. Des informations supplémentaires sur la création d’un cluster AKS avec plusieurs pools de nœuds sont disponibles [ici][use-multiple-node-pools].

Créez un pool de nœuds à l’aide de la commande [az aks nodepool add][az-aks-nodepool-add].
```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name spotnodepool \
    --priority Spot \
    --eviction-policy Delete \
    --spot-max-price -1 \
    --enable-cluster-autoscaler \
    --min-count 1 \
    --max-count 3 \
    --no-wait
```

Par défaut, vous créez un pool de nœuds avec un paramètre *priority* de *Regular* dans votre cluster AKS quand vous créez un cluster avec plusieurs pools de nœuds. La commande ci-dessus ajoute un pool de nœuds auxiliaire à un cluster AKS existant avec un paramètre *priority* de *Spot*. Le paramètre *priority* de *Spot* fait du pool de nœuds un pool de nœuds spot. Dans l’exemple ci-dessus, le paramètre *eviction-policy* est défini sur *Delete* , qui est la valeur par défaut. Lorsque vous affectez à [eviction-policy][eviction-policy] la valeur *Delete* , les nœuds du groupe identique sous-jacent du pool de nœuds sont supprimés lorsqu’ils sont éliminés. Vous pouvez également définir la stratégie d’éviction sur *Deallocate*. Lorsque vous définissez la stratégie d’éviction sur *Deallocate* , les nœuds du groupe identique sous-jacent sont définis sur l’état stopped-deallocated lors de l’éviction. Les nœuds dans l’état stopped-deallocated sont comptabilisés dans votre quota de calcul, et peuvent provoquer des problèmes de mise à l’échelle ou de mise à niveau du cluster. Les valeurs de *priority* et de *eviction-policy* ne peuvent être définies que pendant la création du pool de nœuds. Ces valeurs ne peuvent pas être mises à jour ultérieurement.

La commande active également la [mise à l’échelle automatique de cluster][cluster-autoscaler], qu’il est recommandée d’utiliser avec des pools de nœuds spot. En fonction des charges de travail en cours d’exécution dans votre cluster, la mise à l’échelle automatique de cluster augmente ou diminue le nombre de nœuds dans le pool de nœuds. Pour les pools de nœuds spot, la mise à l’échelle automatique de cluster augmente le nombre de nœuds après une éviction, si des nœuds supplémentaires sont toujours nécessaires. Si vous modifiez le nombre maximal de nœuds dont un pool de nœuds peut disposer, vous devez également ajuster la valeur de `maxCount` qui est associée à la mise à l’échelle automatique de cluster. Si vous n’utilisez pas de mise à l’échelle automatique de cluster, en cas d’éviction, le pool spot finira par diminuer jusqu’à zéro et nécessitera une opération manuelle pour recevoir des nœuds spot supplémentaires.

> [!Important]
> Planifiez uniquement les charges de travail sur les pools de nœuds spot capables de gérer les interruptions, tels que les travaux de traitement par lots et les environnements de test. Nous vous recommandons de configurer des [teintes et des tolérances][taints-tolerations] sur votre pool de nœuds spot pour vous assurer que seules les charges de travail pouvant gérer les évictions de nœuds sont planifiées sur un pool de nœuds spot. Par exemple, la commande ci-dessus ajoute par défaut une teinte de *kubernetes.azure.com/scalesetpriority=spot:NoSchedule* , de telle sorte que seuls les pods avec une tolérance correspondante sont planifiés sur ce nœud.

## <a name="verify-the-spot-node-pool"></a>Vérifier le pool de nœuds spot

Pour vérifier que votre pool de nœuds a été ajouté en tant que pool de nœuds spot :

```azurecli
az aks nodepool show --resource-group myResourceGroup --cluster-name myAKSCluster --name spotnodepool
```

Contrôlez que *scaleSetPriority* est *Spot*.

Pour planifier l’exécution d’un pod sur un nœud spot, ajoutez une tolérance correspondant à la teinte appliquée à votre nœud spot. L’exemple suivant montre la partie d’un fichier YAML qui définit une tolérance correspondant à une teinte *kubernetes.azure.com/scalesetpriority=spot:NoSchedule* utilisée à l’étape précédente.

```yaml
spec:
  containers:
  - name: spot-example
  tolerations:
  - key: "kubernetes.azure.com/scalesetpriority"
    operator: "Equal"
    value: "spot"
    effect: "NoSchedule"
   ...
```

Quand un pod est déployé avec cette tolérance, Kubernetes peut planifier le pod sur les nœuds avec la teinte appliquée.

## <a name="max-price-for-a-spot-pool"></a>Prix maximal pour un pool spot
Les [tarifs des instances spot sont variables][pricing-spot], en fonction de la région et de la référence SKU. Pour plus d’informations, consultez les prix pour [Linux][pricing-linux] et [Windows][pricing-windows].

En raison de la variabilité des tarifs, vous avez la possibilité de définir un prix maximal en dollars américains (USD) ayant jusqu’à 5 décimales. Par exemple, la valeur *0,98765* correspond à un prix maximal de 0,98765 dollars US par heure. Si vous définissez *-1* comme prix maximal, l’instance n’est pas supprimée en fonction du prix. Le prix de l’instance sera le prix actuel de Spot ou le prix d’une instance standard, la valeur la plus faible étant retenue, à condition que la capacité et le quota soient disponibles.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à ajouter un pool de nœuds spot à un cluster AKS. Pour plus d’informations sur la façon de contrôler les pods entre les pools de nœuds, consultez [Bonnes pratiques relatives aux fonctionnalités avancées du planificateur dans AKS][operator-best-practices-advanced-scheduler].

<!-- LINKS - External -->
[kubernetes-services]: https://kubernetes.io/docs/concepts/services-networking/service/

<!-- LINKS - Internal -->
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-nodepool-add]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-add
[cluster-autoscaler]: cluster-autoscaler.md
[eviction-policy]: ../virtual-machine-scale-sets/use-spot.md#eviction-policy
[kubernetes-concepts]: concepts-clusters-workloads.md
[operator-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[pricing-linux]: https://azure.microsoft.com/pricing/details/virtual-machine-scale-sets/linux/
[pricing-spot]: ../virtual-machine-scale-sets/use-spot.md#pricing
[pricing-windows]: https://azure.microsoft.com/pricing/details/virtual-machine-scale-sets/windows/
[spot-toleration]: #verify-the-spot-node-pool
[taints-tolerations]: operator-best-practices-advanced-scheduler.md#provide-dedicated-nodes-using-taints-and-tolerations
[use-multiple-node-pools]: use-multiple-node-pools.md
[vmss-spot]: ../virtual-machine-scale-sets/use-spot.md