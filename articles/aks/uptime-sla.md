---
title: Azure Kubernetes Service (AKS) avec contrat SLA de durée de fonctionnement
description: Découvrez l’offre de Contrat SLA de durée de fonctionnement facultative pour le serveur d’API Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 06/24/2020
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 51430a37f34600d3191a646df7c1890b6d94f481
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033015"
---
# <a name="azure-kubernetes-service-aks-uptime-sla"></a>Contrat SLA de durée de fonctionnement Azure Kubernetes Service (AKS)

Le contrat SLA de durée de fonctionnement est une fonctionnalité facultative permettant de bénéficier d’un contrat SLA soutenu et élevé pour un cluster. Le contrat SLA de durée de fonctionnement garantit une disponibilité de 99,95 % du point de terminaison de serveur de l’API Kubernetes pour les clusters qui utilisent des [Zones de disponibilité][availability-zones] et 99,9 % de disponibilité pour les clusters qui n’utilisent pas de Zones de disponibilité. AKS utilise les réplicas de nœud principal entre les domaines de mise à jour et d’erreur pour garantir la satisfaction des exigences de contrat SLA.

Les clients ayant besoin d’un contrat SLA pour répondre aux exigences de conformité ou nécessitant l’extension d’un contrat SLA à leurs utilisateurs finaux doivent activer cette fonctionnalité. Les clients avec des charges de travail critiques qui bénéficieront d’un contrat SLA de durée de fonctionnement plus élevée peuvent également en bénéficier. L’utilisation de la fonctionnalité Contrat SLA de durée de fonctionnement avec les Zones de disponibilité permet d’obtenir une disponibilité plus élevée pour la durée de bon fonctionnement du serveur d’API Kubernetes.  

Les clients peuvent toujours créer un nombre illimité de clusters gratuits avec un objectif de niveau de service (SLO) de 99,5 % et opter pour leur durée de bon fonctionnement SLO ou SLA préférée en fonction des besoins.

> [!Important]
> Pour les clusters avec verrouillage de sortie, consultez [limiter le trafic de sortie](limit-egress-traffic.md) pour ouvrir les ports appropriés.

## <a name="region-availability"></a>Disponibilité des régions

Le contrat SLA de durée de fonctionnement est disponible dans les régions publiques et dans les régions Azure Government où [AKS est pris en charge](https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service).

## <a name="limitations"></a>Limites

## <a name="sla-terms-and-conditions"></a>Conditions générales des contrats SLA

Le contrat SLA de durée de fonctionnement est une fonctionnalité payante et activée par cluster. Le tarif du contrat SLA de durée de fonctionnement est déterminé par le nombre de clusters discrets, et non par la taille des clusters individuels. Consultez [Détails de la tarification du contrat SLA de durée de fonctionnement](https://azure.microsoft.com/pricing/details/kubernetes-service/) pour en savoir plus.

## <a name="before-you-begin"></a>Avant de commencer

* Installez [Azure CLI](/cli/azure/install-azure-cli) version 2.8.0 ou ultérieure

## <a name="creating-a-new-cluster-with-uptime-sla"></a>Création d’un cluster avec un contrat SLA de durée de fonctionnement

> [!NOTE]
> Actuellement, si vous activez le contrat SLA de durée de fonctionnement, il n’existe aucun moyen de le supprimer d’un cluster.

Pour créer un nouveau cluster avec le contrat SLA de durée de fonctionnement, vous utilisez Azure CLI.

L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli-interactive
# Create a resource group
az group create --name myResourceGroup --location eastus
```
Utilisez la commande [`az aks create`][az-aks-create] pour créer un cluster AKS. L’exemple suivant crée un cluster à un nœud nommé *myAKSCluster*. Cette opération prend plusieurs minutes :

```azurecli-interactive
# Create an AKS cluster with uptime SLA
az aks create --resource-group myResourceGroup --name myAKSCluster --uptime-sla --node-count 1
```
Au bout de quelques minutes, la commande se termine et retourne des informations au format JSON sur le cluster. L’extrait de code JSON suivant montre le niveau payant pour la référence SKU, indiquant que votre cluster est activé avec un contrat SLA de durée de fonctionnement :

```output
  },
  "sku": {
    "name": "Basic",
    "tier": "Paid"
  },
```

## <a name="modify-an-existing-cluster-to-use-uptime-sla"></a>Modifier un cluster existant pour utiliser un contrat SLA de durée de fonctionnement

Vous pouvez éventuellement mettre à jour vos clusters existants pour utiliser un contrat SLA de durée de fonctionnement.

Si vous aviez créé un cluster AKS lors des étapes précédentes, supprimez le groupe de ressources :

```azurecli-interactive
# Delete the existing cluster by deleting the resource group 
az group delete --name myResourceGroup --yes --no-wait
```

Créez un autre groupe de ressources :

```azurecli-interactive
# Create a resource group
az group create --name myResourceGroup --location eastus
```

Créez un autre cluster, sans utiliser de contrat SLA de durée de fonctionnement :

```azurecli-interactive
# Create a new cluster without uptime SLA
az aks create --resource-group myResourceGroup --name myAKSCluster--node-count 1
```

Utilisez la commande [`az aks update`][az-aks-nodepool-update] pour mettre à jour le cluster existant :

```azurecli-interactive
# Update an existing cluster to use Uptime SLA
 az aks update --resource-group myResourceGroup --name myAKSCluster --uptime-sla
 ```

 L’extrait de code JSON suivant montre le niveau payant pour la référence SKU, indiquant que votre cluster est activé avec un contrat SLA de durée de fonctionnement :

 ```output
  },
  "sku": {
    "name": "Basic",
    "tier": "Paid"
  },
  ```

## <a name="clean-up"></a>Nettoyer

Pour éviter des frais, nettoyez toutes les ressources que vous avez créées. Pour supprimer le cluster, utilisez la commande [`az group delete`][az-group-delete] qui supprime le groupe de ressources AKS :

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```


## <a name="next-steps"></a>Étapes suivantes

Utilisez [Zones de disponibilité][availability-zones] pour augmenter la haute disponibilité avec les charges de travail de votre cluster AKS.

Configurez votre cluster pour [limiter le trafic de sortie](limit-egress-traffic.md).

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
[region-availability]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service

<!-- LINKS - Internal -->
[vm-skus]: ../virtual-machines/sizes.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[faq]: ./faq.md
[availability-zones]: ./availability-zones.md
[az-aks-create]: /cli/azure/aks?#az-aks-create
[limit-egress-traffic]: ./limit-egress-traffic.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-aks-nodepool-update]: /cli/azure/aks/nodepool?#az-aks-nodepool-update
[az-group-delete]: /cli/azure/group#az-group-delete
