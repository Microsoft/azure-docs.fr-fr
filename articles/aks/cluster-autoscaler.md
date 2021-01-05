---
title: Utilisation du programme de mise à l’échelle automatique de cluster d’Azure Kubernetes Service (AKS)
description: Découvrez comment utiliser le programme de mise à l’échelle automatique de cluster pour mettre automatiquement à l’échelle votre cluster afin de répondre aux demandes applicatives d’un cluster Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 07/18/2019
ms.openlocfilehash: e644a931152c83a5232c8233d519f7807ab708af
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92542639"
---
# <a name="automatically-scale-a-cluster-to-meet-application-demands-on-azure-kubernetes-service-aks"></a>Mise à l’échelle automatique d’un cluster pour répondre aux demandes applicatives d’Azure Kubernetes Service (AKS)

Pour suivre le rythme des demandes applicatives d’Azure Kubernetes Service (ACS), vous devrez peut-être ajuster le nombre de nœuds qui exécutent vos charges de travail. Le composant programme de mise à l’échelle automatique de cluster peut surveiller les pods de votre cluster qui ne peuvent pas être planifiés en raison de contraintes de ressources. Lorsque des problèmes sont détectés, le nombre de nœuds du pool de nœuds est augmenté pour répondre à la demande applicative. Les pods exécutés sont également régulièrement vérifiés sur les nœuds dont le nombre est réduit au besoin. Cette possibilité d’augmenter ou de réduire automatiquement le nombre de nœuds dans votre cluster AKS vous permet d’exécuter un cluster efficace et économique.

Cet article vous montre comment activer et gérer le programme de mise à l’échelle automatique de cluster dans un cluster AKS.

## <a name="before-you-begin"></a>Avant de commencer

Pour les besoins de cet article, vous devez utiliser Azure CLI version 2.0.76 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli-install].

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

Le programme de mise à l’échelle automatique de cluster utilise des paramètres de démarrage pour des éléments tels que les intervalles de temps entre les événements de mise à l’échelle et les seuils de ressources. Pour plus d’informations sur les paramètres utilisés par le programme de mise à l’échelle automatique de cluster, consultez l’article [Utilisation du profil de mise à l’échelle automatique](#using-the-autoscaler-profile).

Les programmes de mise à l’échelle automatique de cluster et de pods élastique peuvent fonctionner ensemble et sont souvent déployés dans un même cluster. Lorsqu’ils sont combinés, le programme de mise à l’échelle automatique de pods élastique se concentre sur l’exécution du nombre de pods requis pour répondre à la demande applicative. Le programme de mise à l’échelle automatique de cluster est axé sur l’exécution du nombre de nœuds nécessaires pour assurer la prise en charge des pods planifiés.

> [!NOTE]
> La mise à l’échelle manuelle est désactivée lorsque vous utilisez le programme de mise à l’échelle automatique de cluster. Laissez le programme de mise à l’échelle automatique de cluster déterminer le nombre de nœuds requis. Si vous souhaitez mettre à l’échelle manuellement votre cluster, [désactivez le programme de mise à l’échelle automatique de cluster](#disable-the-cluster-autoscaler).

## <a name="create-an-aks-cluster-and-enable-the-cluster-autoscaler"></a>Création d’un cluster AKS et activation du programme de mise à l’échelle automatique de cluster

Utilisez la commande [az aks create][az-aks-create] pour créer un cluster AKS. Pour activer et configurer le programme de mise à l’échelle automatique de cluster sur le pool de nœuds du cluster, utilisez le paramètre `--enable-cluster-autoscaler`, puis spécifiez un nœud `--min-count` et un `--max-count`.

> [!IMPORTANT]
> L’autoscaler de cluster est un composant Kubernetes. Bien que le cluster AKS utilise un groupe de machines virtuelles identiques défini pour les nœuds, n’activez pas ou n’éditez pas manuellement les paramètres pour la mise à l’échelle automatique du groupe de machines virtuelles identiques dans le portail Azure ou en utilisant l’interface de ligne de commande Azure. Laissez le programme de mise à l’échelle automatique de cluster Kubernetes gérer les paramètres de mise à l’échelle requis. Pour plus d’informations, consultez [Puis-je modifier les ressources AKS dans le groupe de ressources de nœuds ?][aks-faq-node-resource-group]

L’exemple suivant crée un cluster AKS avec un pool de nœuds unique soutenu par un groupe de machines virtuelles identiques. Il active également la mise à l’échelle automatique de cluster sur le pool de nœuds pour le cluster et définit un minimum de *1* et un maximum de *3*  nœuds :

```azurecli-interactive
# First create a resource group
az group create --name myResourceGroup --location eastus

# Now create the AKS cluster and enable the cluster autoscaler
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 1 \
  --vm-set-type VirtualMachineScaleSets \
  --load-balancer-sku standard \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 3
```

La création du cluster et la configuration des paramètres du programme de mise à l’échelle automatique de cluster prennent quelques minutes.

## <a name="update-an-existing-aks-cluster-to-enable-the-cluster-autoscaler"></a>Activation du programme de mise à l’échelle automatique de cluster sur un cluster AKS existant

Utilisez la commande [az aks update][az-aks-update] pour activer et configurer la mise à l’échelle automatique de cluster sur le pool de nœuds du cluster existant. Utilisez le paramètre `--enable-cluster-autoscaler`, puis spécifiez un `--min-count` et un `--max-count` de nœuds.

> [!IMPORTANT]
> L’autoscaler de cluster est un composant Kubernetes. Bien que le cluster AKS utilise un groupe de machines virtuelles identiques défini pour les nœuds, n’activez pas ou n’éditez pas manuellement les paramètres pour la mise à l’échelle automatique du groupe de machines virtuelles identiques dans le portail Azure ou en utilisant l’interface de ligne de commande Azure. Laissez le programme de mise à l’échelle automatique de cluster Kubernetes gérer les paramètres de mise à l’échelle requis. Pour plus d’informations, consultez [Puis-je modifier les ressources AKS dans le groupe de ressources de nœuds ?][aks-faq-node-resource-group]

L’exemple suivant met à jour un cluster AKS existant pour activer la mise à l’échelle automatique de cluster sur le pool de nœuds pour le cluster et définit un minimum de *1* et un maximum de *3*  nœuds :

```azurecli-interactive
az aks update \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 3
```

La mise à jour du cluster et la configuration des paramètres du programme de mise à l’échelle automatique de cluster prennent quelques minutes.

## <a name="change-the-cluster-autoscaler-settings"></a>Modification des paramètres du programme de mise à l’échelle automatique de cluster

> [!IMPORTANT]
> Si vous disposez de plusieurs pools de nœuds dans votre cluster AKS, passez à la section relative à la [mise à l’échelle automatique à l’aide de pools d’agents multiples](#use-the-cluster-autoscaler-with-multiple-node-pools-enabled). Les clusters avec des pools d’agents multiples nécessitent l’utilisation de l’ensemble de commandes `az aks nodepool` au lieu de `az aks` pour modifier des propriétés spécifiques du pool de nœuds.

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

L’exemple ci-dessus met à jour le programme de mise à l’échelle automatique du cluster sur le pool de nœuds dans *myAKSCluster* en définissant un minimum de *1* et un maximum de *5*  nœuds.

> [!NOTE]
> Le programme de mise à l’échelle automatique de cluster prend les décisions de mise à l’échelle en fonction des nombres minimum et maximum définis sur chaque pool de nœuds, mais il ne les applique pas après la mise à jour des nombres minimum ou maximum. Par exemple, définir un nombre minimum de 5 lorsque le nombre actuel de nœuds est 3 n’effectue pas immédiatement un Scale-up du pool sur 5. Si le nombre minimum sur le pool de nœuds a une valeur supérieure au nombre actuel de nœuds, les nouveaux paramètres minimum ou maximum seront respectés lorsqu’il y aura suffisamment de pods non planifiables qui nécessiteraient deux nouveaux nœuds supplémentaires et déclencheraient un événement du programme de mise à l’échelle automatique. Après l’événement de mise à l’échelle, les nouvelles limites de nombre sont respectées.

Surveillez les performances de vos applications et services, puis ajustez les nombres de nœuds du programme de mise à l’échelle automatique de cluster pour les faire correspondre aux performances requises.

## <a name="using-the-autoscaler-profile"></a>Utilisation du profil de mise à l’échelle automatique

Vous pouvez également configurer une plus grande précision dans les détails de la mise à l’échelle automatique de cluster en changeant les valeurs par défaut du profil de mise à l’échelle automatique sur l’ensemble du cluster. Par exemple, un événement de scale-down se produit lorsque des nœuds sont sous-utilisés après 10 minutes. Si vous aviez des charges de travail qui se sont exécutées toutes les 15 minutes, vous souhaitez peut-être modifier le profil de mise à l’échelle automatique pour effectuer un scale-down des nœuds sous-utilisés après 15 ou 20 minutes. Lorsque vous activez l’option de mise à l’échelle automatique de cluster, un profil par défaut est utilisé, à moins de spécifier différents paramètres. Le profil de mise à l’échelle automatique de cluster contient les paramètres suivants, qu’il est possible de mettre à jour :

| Paramètre                          | Description                                                                              | Valeur par défaut |
|----------------------------------|------------------------------------------------------------------------------------------|---------------|
| scan-interval                    | Fréquence à laquelle le cluster est réévalué pour effectuer un scale-up ou un scale-down                                    | 10 secondes    |
| scale-down-delay-after-add       | Durée, après le scale-up, à l’issue de laquelle s’effectue la reprise de l’évaluation de scale-down                               | 10 minutes    |
| scale-down-delay-after-delete    | Durée, après la suppression du nœud, à l’issue de laquelle s’effectue la reprise de l’évaluation de scale-down                          | scan-interval |
| scale-down-delay-after-failure   | Durée, après un échec de scale-down, à l’issue de laquelle s’effectue la reprise de l’évaluation de scale-down                     | 3 minutes     |
| scale-down-unneeded-time         | Durée pendant laquelle un nœud doit être inutile avant d’être éligible pour un scale-down                  | 10 minutes    |
| scale-down-unready-time          | Durée pendant laquelle un nœud non prêt doit être inutile avant d’être éligible pour un scale-down         | 20 minutes    |
| scale-down-utilization-threshold | Niveau d’utilisation du nœud (défini en tant que somme des ressources demandées, divisée par la capacité) en dessous duquel un nœud peut être pris en compte pour un scale-down | 0.5 |
| max-graceful-termination-sec     | Nombre maximal de secondes pendant lesquelles la mise à l’échelle automatique de cluster attend l’arrêt d’un pod lors d’une tentative de scale-down d’un nœud. | 600 secondes   |
| balance-similar-node-groups      | Détecte les pools de nœuds similaires et équilibre le nombre de nœuds entre eux                 | false         |
| expander                         | Type de pool de nœuds [expander](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-are-expanders) à utiliser dans un scale-up Valeurs possibles : `most-pods`, `random`, `least-waste` | random | 
| skip-nodes-with-local-storage    | Si le programme de mise à l’échelle automatique de cluster ne supprime jamais les nœuds dotés de pods avec stockage local, par exemple EmptyDir ou HostPath | true |
| skip-nodes-with-system-pods      | Si le programme de mise à l’échelle automatique de cluster ne supprime jamais les nœuds dotés de pods de kube-system (à l’exception de DaemonSet ou de pods miroir) | true | 
| max-empty-bulk-delete            | Nombre maximal de nœuds vides pouvant être supprimés en même temps                      | 10 nœuds      |
| new-pod-scale-up-delay           | Pour les scénarios tels que la mise à l’échelle en rafales ou par lots où vous ne souhaitez pas que l’autorité de certification agisse avant que le planificateur Kubernetes puisse planifier tous les pods, vous pouvez indiquer à l’autorité de certification d’ignorer les pods non planifiés avant qu’ils n’aient atteint une certaine ancienneté                                                                                                                | 10 secondes    |
| max-total-unready-percentage     | Pourcentage maximal de nœuds non prêts dans le cluster. Une fois ce pourcentage dépassé, l’autorité de certification arrête les opérations | 45 % | 
| ok-total-unready-count           | Nombre de nœuds non prêts autorisés, quelle que soit la valeur de max-total-unready-percentage            | 3 nœuds       |

> [!IMPORTANT]
> Le profil de mise à l’échelle automatique de cluster modifie tous les pools de nœuds qui utilisent la mise à l’échelle automatique de cluster. Vous ne pouvez pas définir de profil de mise à l’échelle automatique par pool de nœuds.
>
> Le profil de mise à l'échelle automatique de cluster requiert la version  *2.11.1* ou ultérieure d'Azure CLI. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli-install].

### <a name="set-the-cluster-autoscaler-profile-on-an-existing-aks-cluster"></a>Définir le profil de mise à l’échelle automatique de cluster sur un cluster AKS existant

Utilisez la commande [az aks update][az-aks-update-preview] avec le paramètre *cluster-autoscaler-profile* pour définir le profil de mise à l’échelle automatique de cluster sur votre cluster. L’exemple suivant configure le paramètre d’intervalle d’analyse sur 30 s dans le profil.

```azurecli-interactive
az aks update \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --cluster-autoscaler-profile scan-interval=30s
```

Quand vous activez la mise à l’échelle automatique de cluster sur des pools de nœuds dans le cluster, ces derniers utiliseront également le profil de mise à l’échelle automatique de cluster. Par exemple :

```azurecli-interactive
az aks nodepool update \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name mynodepool \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 3
```

> [!IMPORTANT]
> Lorsque vous définissez le profil de mise à l’échelle automatique de cluster, tous les pools de nœuds existants dont l’option de mise à l’échelle automatique de cluster est activée commenceront immédiatement à utiliser le profil.

### <a name="set-the-cluster-autoscaler-profile-when-creating-an-aks-cluster"></a>Définir le profil de mise à l’échelle automatique de cluster lors de la création d’un cluster AKS

Vous pouvez également utiliser le paramètre *cluster-autoscaler-profile* lors de la création de votre cluster. Par exemple :

```azurecli-interactive
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 1 \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 3 \
  --cluster-autoscaler-profile scan-interval=30s
```

La commande ci-dessus crée un cluster AKS et définit l’intervalle d’analyse sur 30 secondes pour le profil de mise à l’échelle automatique sur l’ensemble du cluster. La commande active également la mise à l’échelle automatique de cluster sur le pool de nœuds initial, elle définit le nombre de nœuds minimal sur 1, et le nombre de nœuds maximal sur 3.

### <a name="reset-cluster-autoscaler-profile-to-default-values"></a>Rétablir les valeurs par défaut du profil de mise à l’échelle automatique de cluster

Utilisez la commande [az aks update][az-aks-update-preview] pour réinitialiser le profil de mise à l’échelle automatique de cluster sur votre cluster.

```azurecli-interactive
az aks update \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --cluster-autoscaler-profile ""
```

## <a name="disable-the-cluster-autoscaler"></a>Désactivation du programme de mise à l’échelle automatique de cluster

Si vous ne souhaitez plus utiliser le programme de mise à l’échelle automatique du cluster, vous pouvez le désactiver à l’aide de la commande [az aks update][az-aks-update-preview], en spécifiant le paramètre `--disable-cluster-autoscaler`. Les nœuds ne sont pas supprimés lorsque le programme de mise à l’échelle automatique de cluster est désactivé.

```azurecli-interactive
az aks update \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --disable-cluster-autoscaler
```

Vous pouvez mettre à l’échelle votre cluster manuellement après avoir désactivé le programme de mise à l’échelle automatique du cluster à l’aide de la commande [az aks scale][az-aks-scale]. Si vous utilisez le programme de mise à l’échelle automatique de pod horizontal, cette fonctionnalité continue de s’exécuter avec le programme de mise à l’échelle automatique de cluster désactivé, mais la planification des pods peut devenir impossible si toutes les ressources de nœuds sont utilisées.

## <a name="re-enable-a-disabled-cluster-autoscaler"></a>Réactiver un programme de mise à l’échelle automatique du cluster désactivé

Si vous ne souhaitez pas réactiver le programme de mise à l’échelle automatique sur un cluster existant, vous pouvez le réactiver à l’aide de la commande [az aks update][az-aks-update-preview], en spécifiant les paramètres `--enable-cluster-autoscaler`, `--min-count` et `--max-count`.

## <a name="retrieve-cluster-autoscaler-logs-and-status"></a>Récupérer les journaux et l’état de la mise à l’échelle automatique des clusters

Pour diagnostiquer et déboguer des événements de mise à l’échelle automatique, les journaux et l’état peuvent être récupérés à partir du module complémentaire de mise à l’échelle automatique.

AKS gère la mise à l’échelle automatique des clusters en votre nom et l’exécute dans le plan de contrôle managé. Vous pouvez activer le nœud de plan de contrôle pour afficher les journaux et les opérations de l’autorité de certification.

Pour configurer les journaux à envoyer (push), de la mise à l’échelle automatique de cluster à Log Analytics, suivez ces étapes.

1. Configurez une règle pour les journaux de ressources, afin d’envoyer (push) les journaux de la mise à l’échelle automatique de cluster vers Log Analytics. [Les instructions sont détaillées ici][aks-view-master-logs] ; veillez à cocher la case `cluster-autoscaler` lors de la sélection des options pour « Journaux d’activité ».
1. Sélectionnez la section « Journaux » de votre cluster via le portail Azure.
1. Dans Log Analytics, entrez l’exemple de requête suivant :

```
AzureDiagnostics
| where Category == "cluster-autoscaler"
```

Vous devez voir des journaux similaires à l’exemple suivant, à condition qu’il y ait des journaux à récupérer.

![Journaux d’activité Log Analytics](media/autoscaler/autoscaler-logs.png)

Le programme de mise à l’échelle automatique de clusters écrit également l’état d’intégrité sur un élément `configmap` nommé `cluster-autoscaler-status`. Pour récupérer ces journaux, exécutez la commande `kubectl` suivante. Un état d’intégrité sera signalé pour chaque pool de nœuds configuré avec la mise à l’échelle automatique de clusters.

```
kubectl get configmap -n kube-system cluster-autoscaler-status -o yaml
```

Pour en savoir plus sur ce qui est enregistré à partir de la mise à l’échelle automatique, consultez la FAQ relative au [projet GitHub Kubernetes/mise à l’échelle automatique][kubernetes-faq].

## <a name="use-the-cluster-autoscaler-with-multiple-node-pools-enabled"></a>Utiliser le programme de mise à l’échelle automatique du cluster avec la fonctionnalité de pools de nœuds multiples activée

Vous pouvez utiliser le programme de mise à l’échelle automatique du cluster avec des [pools de nœuds multiples][aks-multiple-node-pools] activés. Lisez ce document pour apprendre à activer des pools de nœuds multiples et à ajouter des pools de nœuds supplémentaires à un cluster existant. Si vous utilisez les deux fonctionnalités ensemble, vous activez le programme de mise à l’échelle automatique du cluster sur chaque pool de nœuds dans le cluster, et pouvez leur transmettre des règles de mise à l’échelle automatique uniques.

La commande ci-dessous part du principe que vous avez suivi les [instructions initiales](#create-an-aks-cluster-and-enable-the-cluster-autoscaler) plus haut dans ce document, et que vous souhaitez mettre à jour le nombre maximal de nœuds d’un pool existant de *3* à *5*. Utilisez la commande [az aks nodepool update][az-aks-nodepool-update] pour mettre à jour les paramètres d’un pool de nœuds existant.

```azurecli-interactive
az aks nodepool update \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name nodepool1 \
  --update-cluster-autoscaler \
  --min-count 1 \
  --max-count 5
```

Vous pouvez désactiver le programme de mise à l’échelle automatique du cluster avec la commande [az aks nodepool update][az-aks-nodepool-update] en transmettant le paramètre `--disable-cluster-autoscaler`.

```azurecli-interactive
az aks nodepool update \
  --resource-group myResourceGroup \
  --cluster-name myAKSCluster \
  --name nodepool1 \
  --disable-cluster-autoscaler
```

Si vous ne souhaitez pas réactiver le programme de mise à l’échelle automatique sur un cluster existant, vous pouvez le réactiver à l’aide de la commande [az aks nodepool][az-aks-nodepool-update], en spécifiant les paramètres `--enable-cluster-autoscaler`, `--min-count` et `--max-count`.

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a montré comment mettre automatiquement à l’échelle le nombre de nœuds AKS. Vous pouvez également utiliser le programme de mise à l’échelle automatique de pods élastique pour ajuster automatiquement le nombre de pods qui exécutent votre application. Pour obtenir des instructions sur l’utilisation du programme de mise à l’échelle automatique de pods élastique, consultez l’article [Mettre à l’échelle des applications dans Azure Kubernetes Service (AKS)][aks-scale-apps].

<!-- LINKS - internal -->
[aks-faq]: faq.md
[aks-faq-node-resource-group]: faq.md#can-i-modify-tags-and-other-properties-of-the-aks-resources-in-the-node-resource-group
[aks-multiple-node-pools]: use-multiple-node-pools.md
[aks-scale-apps]: tutorial-kubernetes-scale.md
[aks-support-policies]: support-policies.md
[aks-upgrade]: upgrade-cluster.md
[aks-view-master-logs]: ./view-master-logs.md#enable-resource-logs
[autoscaler-profile-properties]: #using-the-autoscaler-profile
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-update]: /cli/azure/aks#az-aks-update
[az-aks-scale]: /cli/azure/aks#az-aks-scale
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register

<!-- LINKS - external -->
[az-aks-update-preview]: https://github.com/Azure/azure-cli-extensions/tree/master/src/aks-preview
[az-aks-nodepool-update]: https://github.com/Azure/azure-cli-extensions/tree/master/src/aks-preview#enable-cluster-auto-scaler-for-a-node-pool
[autoscaler-scaledown]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-types-of-pods-can-prevent-ca-from-removing-a-node
[autoscaler-parameters]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-are-the-parameters-to-ca
[kubernetes-faq]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#ca-doesnt-work-but-it-used-to-work-yesterday-why
