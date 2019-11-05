---
title: Utilisation du programme de mise à l’échelle automatique de cluster d’Azure Kubernetes Service (AKS)
description: Découvrez comment utiliser le programme de mise à l’échelle automatique de cluster pour mettre automatiquement à l’échelle votre cluster afin de répondre aux demandes applicatives d’un cluster Azure Kubernetes Service (AKS).
services: container-service
author: mlearned
ms.service: container-service
ms.topic: article
ms.date: 07/18/2019
ms.author: mlearned
ms.openlocfilehash: 9d7a404b767d3975cefd55e1db8487fbb45042e2
ms.sourcegitcommit: 42748f80351b336b7a5b6335786096da49febf6a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2019
ms.locfileid: "72174182"
---
# <a name="Automatically-scale-a-cluster-to-meet-application-demands-on-azure-kubernetes-service-aks"></a>Mettre à l’échelle automatiquement un cluster pour répondre à des demandes applicatives d’Azure Kubernetes Service (AKS)

Pour suivre le rythme des demandes applicatives d’Azure Kubernetes Service (ACS), vous devrez peut-être ajuster le nombre de nœuds qui exécutent vos charges de travail. Le composant programme de mise à l’échelle automatique de cluster peut surveiller les pods de votre cluster qui ne peuvent pas être planifiés en raison de contraintes de ressources. Lorsque des problèmes sont détectés, le nombre de nœuds du pool de nœuds est augmenté pour répondre à la demande applicative. Les pods exécutés sont également régulièrement vérifiés sur les nœuds dont le nombre est réduit au besoin. Cette possibilité d’augmenter ou de réduire automatiquement le nombre de nœuds dans votre cluster AKS vous permet d’exécuter un cluster efficace et économique.

Cet article vous montre comment activer et gérer le programme de mise à l’échelle automatique de cluster dans un cluster AKS.

## <a name="before-you-begin"></a>Avant de commencer

Pour les besoins de cet article, vous devez utiliser Azure CLI version 2.0.76 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli-install].

## <a name="limitations"></a>Limites

Les limitations suivantes s’appliquent lorsque vous créez et gérez des clusters AKS qui utilisent l’autoscaler de cluster :

* Le module complémentaire de routage d’application HTTP ne peut pas être utilisé.

## <a name="about-the-cluster-autoscaler"></a>À propos du programme de mise à l’échelle automatique de cluster

Pour faire face à l’évolution des demandes applicatives, par exemple entre les journées de travail et les soirées ou pendant les week-ends, les clusters ont souvent besoin d’être mis à l’échelle automatiquement. Les clusters AKS peuvent être mis à l’échelle de deux manières :

* Le **programme de mise à l’échelle automatique de cluster** surveille les pods qui ne peuvent pas être planifiés sur les nœuds en raison de contraintes de ressources. Le cluster augmente alors automatiquement le nombre de nœuds.
* Le **programme de mise à l’échelle automatique de pods élastique** utilise le serveur de mesures d’un cluster Kubernetes pour surveiller la demande en ressources de pods. Si une application a besoin de davantage de ressources, le nombre de pods est automatiquement augmenté pour répondre à la demande.

![Le programme de mise à l’échelle automatique de cluster et le programme de mise à l’échelle automatique de pods élastique fonctionnent souvent ensemble pour assurer la prise en charge des demandes applicatives requises.](media/autoscaler/cluster-autoscaler.png)

Ces deux programmes de mise à l’échelle automatique peuvent également réduire le nombre de pods et de nœuds en fonction des besoins. Le programme de mise à l’échelle automatique de cluster diminue le nombre de nœuds en cas de capacité inutilisée pendant une période de temps. Les pods d’un nœud à supprimer par le programme de mise à l’échelle automatique de cluster sont planifiés en toute sécurité ailleurs dans le cluster. Le programme de mise à l’échelle automatique de cluster peut être incapable de réduire le nombre de nœuds si des pods ne peuvent pas être déplacés, notamment dans les situations suivantes :

* Un pod est créé directement et n’est pas pris en charge par un objet de contrôleur, par exemple un déploiement ou un jeu de réplicas.
* Un budget d’interruption de pods est trop restrictif et n’autorise pas la diminution du nombre de pods au-dessous d’un certain seuil.
* Un pod utilise des sélecteurs de nœud ou l’anti-affinité qui ne peuvent pas être respectés s’ils sont planifiés sur un autre nœud.

Pour plus d’informations sur l’incapacité du programme de mise à l’échelle automatique de cluster à réduire le nombre de nœuds, consultez l’article [What types of pods can prevent the cluster autoscaler from removing a node?][autoscaler-scaledown] (Quels sont les types de pods pouvant empêcher la suppression d’un nœud par le programme de mise à l’échelle automatique de cluster ?)

Le programme de mise à l’échelle automatique de cluster utilise des paramètres de démarrage pour des éléments tels que les intervalles de temps entre les événements de mise à l’échelle et les seuils de ressources. Ces paramètres sont définis par la plateforme Azure et ne sont pas exposés actuellement pour que vous les ajustiez. Pour plus d’informations sur les paramètres utilisés par le programme de mise à l’échelle automatique de cluster, consultez l’article [What are the cluster autoscaler parameters?][autoscaler-parameters] (Quels sont les paramètres du programme de mise à l’échelle automatique de cluster ?).

Les programmes de mise à l’échelle automatique de cluster et de pods élastique peuvent fonctionner ensemble et sont souvent déployés dans un même cluster. Lorsqu’ils sont combinés, le programme de mise à l’échelle automatique de pods élastique se concentre sur l’exécution du nombre de pods requis pour répondre à la demande applicative. Le programme de mise à l’échelle automatique de cluster est axé sur l’exécution du nombre de nœuds nécessaires pour assurer la prise en charge des pods planifiés.

> [!NOTE]
> La mise à l’échelle manuelle est désactivée lorsque vous utilisez le programme de mise à l’échelle automatique de cluster. Laissez le programme de mise à l’échelle automatique de cluster déterminer le nombre de nœuds requis. Si vous souhaitez mettre à l’échelle manuellement votre cluster, [désactivez le programme de mise à l’échelle automatique de cluster](#disable-the-cluster-autoscaler).

## <a name="create-an-aks-cluster-and-enable-the-cluster-autoscaler"></a>Création d’un cluster AKS et activation du programme de mise à l’échelle automatique de cluster

Utilisez la commande [az aks create][az-aks-create] pour créer un cluster AKS. Pour activer et configurer le programme de mise à l’échelle automatique de cluster sur le pool de nœuds du cluster, utilisez le paramètre *--enable-cluster-autoscaler*, puis spécifiez un nœud *--min-count* et un *--max-count*.

> [!IMPORTANT]
> L’autoscaler de cluster est un composant Kubernetes. Bien que le cluster AKS utilise un groupe de machines virtuelles identiques défini pour les nœuds, n’activez pas ou n’éditez pas manuellement les paramètres pour la mise à l’échelle automatique du groupe de machines virtuelles identiques dans le portail Azure ou en utilisant l’interface de ligne de commande Azure. Laissez le programme de mise à l’échelle automatique de cluster Kubernetes gérer les paramètres de mise à l’échelle requis. Pour plus d’informations, consultez [Puis-je modifier les ressources AKS dans le groupe de ressources de nœuds ?](faq.md#can-i-modify-tags-and-other-properties-of-the-aks-resources-in-the-node-resource-group)

L’exemple suivant crée un cluster AKS avec un pool de nœuds unique soutenu par un groupe de machines virtuelles identiques. Il active également la mise à l’échelle automatique de cluster sur le pool de nœuds pour le cluster et définit un minimum de *1* et un maximum de *3* nœuds :

```azurecli-interactive
# First create a resource group
az group create --name myResourceGroup --location eastus

# Now create the AKS cluster and enable the cluster autoscaler
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 1 \
  --enable-vmss \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 3
```

> [!NOTE]
> Si vous spécifiez un paramètre *--kubernetes-version* lors de l’exécution de `az aks create`, celui-ci doit être égal ou supérieur au numéro de version minimale requis indiqué dans la section précédente [Avant de commencer](#before-you-begin).

La création du cluster et la configuration des paramètres du programme de mise à l’échelle automatique de cluster prennent quelques minutes.

## <a name="change-the-cluster-autoscaler-settings"></a>Modification des paramètres du programme de mise à l’échelle automatique de cluster

> [!IMPORTANT]
> Si la fonctionnalité de *pools d’agents multiples* est activée sur votre abonnement, passez à la [section relative à la mise à l’échelle automatique à l’aide de pools d’agents multiples](#use-the-cluster-autoscaler-with-multiple-node-pools-enabled). Les clusters dans lesquels des pools d’agents multiples sont activés nécessitent l’utilisation de l’ensemble de commandes `az aks nodepool` au lieu de `az aks` pour modifier des propriétés spécifiques du pool de nœuds. Les instructions ci-dessous partent du principe que vous n’avez pas activé des pools de nœuds multiples. Pour vérifier si vous les avez activés, exécutez la commande `az feature  list -o table` et recherchez `Microsoft.ContainerService/multiagentpoolpreview`.

Dans l’étape précédente de création d’un cluster AKS ou de mise à jour d’un pool de nœuds existant, le nombre minimal de nœuds du programme de mise à l’échelle automatique de cluster a été défini sur *1* tandis que le nombre maximal de nœuds a été défini sur *3*. Si vos demandes applicatives changent, vous devrez peut-être ajuster le nombre de nœuds du programme de mise à l’échelle automatique de cluster.

Pour modifier le nombre de nœuds, utilisez la commande [az aks update][az-aks-update].

```azurecli-interactive
az aks update \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --update-cluster-autoscaler \
  --min-count 1 \
  --max-count 5
```

L’exemple ci-dessus met à jour le programme de mise à l’échelle automatique du cluster sur le pool de nœuds dans *myAKSCluster* en définissant un minimum de *1* et un maximum de *5* nœuds.

Surveillez les performances de vos applications et services, puis ajustez les nombres de nœuds du programme de mise à l’échelle automatique de cluster pour les faire correspondre aux performances requises.

## <a name="disable-the-cluster-autoscaler"></a>Désactivation du programme de mise à l’échelle automatique de cluster

Si vous ne souhaitez plus utiliser le programme de mise à l’échelle automatique du cluster, vous pouvez le désactiver à l’aide de la commande [az aks update][az-aks-update], en spécifiant le paramètre *--disable-cluster-autoscaler*. Les nœuds ne sont pas supprimés lorsque le programme de mise à l’échelle automatique de cluster est désactivé.

```azurecli-interactive
az aks update \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --disable-cluster-autoscaler
```

Vous pouvez mettre à l’échelle votre cluster manuellement après avoir désactivé le programme de mise à l’échelle automatique du cluster à l’aide de la commande [az aks scale][az-aks-scale]. Si vous utilisez le programme de mise à l’échelle automatique de pod horizontal, cette fonctionnalité continue de s’exécuter avec le programme de mise à l’échelle automatique de cluster désactivé, mais la planification des pods peut devenir impossible si toutes les ressources de nœuds sont utilisées.

## <a name="re-enable-a-disabled-cluster-autoscaler"></a>Réactiver un programme de mise à l’échelle automatique du cluster désactivé

Si vous ne souhaitez pas réactiver le programme de mise à l’échelle automatique sur un cluster existant, vous pouvez le réactiver à l’aide de la commande [az aks update][az-aks-update], en spécifiant le paramètre *--enable-cluster-autoscaler*.

## <a name="use-the-cluster-autoscaler-with-multiple-node-pools-enabled"></a>Utiliser le programme de mise à l’échelle automatique du cluster avec la fonctionnalité de pools de nœuds multiples activée

Vous pouvez utiliser le programme de mise à l’échelle automatique du cluster avec la [fonctionnalité de pools de nœuds multiples](use-multiple-node-pools.md) activée. Lisez ce document pour apprendre à activer des pools de nœuds multiples et à ajouter des pools de nœuds supplémentaires à un cluster existant. Si vous utilisez les deux fonctionnalités ensemble, vous activez le programme de mise à l’échelle automatique du cluster sur chaque pool de nœuds dans le cluster, et pouvez leur transmettre des règles de mise à l’échelle automatique uniques.

La commande ci-dessous part du principe que vous avez suivi les [instructions initiales](#create-an-aks-cluster-and-enable-the-cluster-autoscaler) plus haut dans ce document, et que vous souhaitez mettre à jour le nombre maximal de nœuds d’un pool existant de *3* à *5*. Utilisez la commande [az aks nodepool update][az-aks-nodepool-update] pour mettre à jour les paramètres d’un pool de nœuds existant.

```azurecli-interactive
az aks nodepool update \
  --resource-group myResourceGroup \
  --cluster-name multipoolcluster \
  --name mynodepool \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 5
```

Vous pouvez désactiver le programme de mise à l’échelle automatique du cluster avec la commande [az aks nodepool update][az-aks-nodepool-update] en transmettant le paramètre `--disable-cluster-autoscaler`.

```azurecli-interactive
az aks nodepool update \
  --resource-group myResourceGroup \
  --cluster-name multipoolcluster \
  --name mynodepool \
  --disable-cluster-autoscaler
```

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a montré comment mettre automatiquement à l’échelle le nombre de nœuds AKS. Vous pouvez également utiliser le programme de mise à l’échelle automatique de pods élastique pour ajuster automatiquement le nombre de pods qui exécutent votre application. Pour obtenir des instructions sur l’utilisation du programme de mise à l’échelle automatique de pods élastique, consultez l’article [Mettre à l’échelle des applications dans Azure Kubernetes Service (AKS)][aks-scale-apps].

<!-- LINKS - internal -->
[aks-upgrade]: upgrade-cluster.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-extension-add]: /cli/azure/extension#az-extension-add
[aks-scale-apps]: tutorial-kubernetes-scale.md
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-scale]: /cli/azure/aks#az-aks-scale
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update

<!-- LINKS - external -->
[az-aks-update]: https://github.com/Azure/azure-cli-extensions/tree/master/src/aks-preview
[az-aks-nodepool-update]: https://github.com/Azure/azure-cli-extensions/tree/master/src/aks-preview#enable-cluster-auto-scaler-for-a-node-pool
[autoscaler-scaledown]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-types-of-pods-can-prevent-ca-from-removing-a-node
[autoscaler-parameters]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-are-the-parameters-to-ca
