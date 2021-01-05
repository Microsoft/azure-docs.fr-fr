---
title: Configurer un réseau Azure CNI dans AKS (Azure Kubernetes Service)
description: Découvrez comment configurer un réseau (avancé) Azure CNI dans AKS (Azure Kubernetes Service), notamment pour déployer un cluster AKS dans un réseau et un sous-réseau virtuels existants.
services: container-service
ms.topic: article
ms.date: 06/03/2019
ms.openlocfilehash: 58c2c597c7a75c801af91cd735561071250bda2c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000570"
---
# <a name="configure-azure-cni-networking-in-azure-kubernetes-service-aks"></a>Configurer un réseau Azure CNI dans AKS (Azure Kubernetes Service)

Par défaut, les clusters AKS utilisent [kubenet][kubenet], et un réseau et un sous-réseau virtuels sont automatiquement créés. Avec *kubenet*, les nœuds obtiennent une adresse IP d’un sous-réseau du réseau virtuel. La traduction d’adresses réseau (NAT) est ensuite configurée sur les nœuds, et les pods reçoivent une adresse IP « cachée » derrière l’adresse IP du nœud. Cette approche réduit le nombre d’adresses IP que vous avez besoin de réserver dans votre espace réseau à l’usage des pods.

Avec l’interface [Azure Container Networking Interface (CNI)][cni-networking], chaque pod reçoit une adresse IP du sous-réseau et est accessible directement. Ces adresses IP doivent être uniques dans votre espace réseau et doivent être planifiées à l’avance. Chaque nœud possède un paramètre de configuration pour le nombre maximal de pods qu’il prend en charge. Le nombre équivalent d’adresses IP par nœud est alors réservé à l’avance pour ce nœud. Cette approche nécessite davantage de planification. De plus, elle conduit souvent à l’épuisement des adresses IP ou à la nécessité de regénérer les clusters dans un sous-réseau plus vaste à mesure que vos demandes d’applications augmentent.

Cet article vous montre comment utiliser les réseaux *Azure CNI* afin de créer et d’utiliser un sous-réseau de réseau virtuel pour un cluster AKS. Pour plus d’informations sur les options et considérations relatives aux réseaux, consultez [Concepts de réseau pour Kubernetes et AKS][aks-network-concepts].

## <a name="prerequisites"></a>Prérequis

* Le réseau virtuel du cluster AKS doit autoriser les connexions Internet sortantes.
* Les clusters AKS ne peuvent pas utiliser `169.254.0.0/16`, `172.30.0.0/16`, `172.31.0.0/16` ou `192.0.2.0/24` pour la plage d'adresses de service Kubernetes, la plage d'adresses de pod ou la plage d'adresses de réseau virtuel de cluster. 
* Le principal du service utilisé par le cluster AKS doit disposer au moins des autorisations [Contributeur de réseau](../role-based-access-control/built-in-roles.md#network-contributor) sur le sous-réseau de votre réseau virtuel. Si vous souhaitez définir un [rôle personnalisé](../role-based-access-control/custom-roles.md) au lieu d’utiliser le rôle de contributeur de réseau intégré, les autorisations suivantes sont nécessaires :
  * `Microsoft.Network/virtualNetworks/subnets/join/action`
  * `Microsoft.Network/virtualNetworks/subnets/read`
* Au lieu d’utiliser le principal de service, vous pouvez utiliser l’identité managée affectée par le système pour les autorisations. Pour plus d’informations, consultez [Utiliser des identités managées](use-managed-identity.md).
* Le sous-réseau affecté au pool de nœuds AKS ne peut pas être un [sous-réseau délégué](../virtual-network/subnet-delegation-overview.md).

## <a name="plan-ip-addressing-for-your-cluster"></a>Planifier l’adressage IP pour votre cluster

Les clusters configurés avec les réseaux Azure CNI nécessitent une planification supplémentaire. La taille de votre réseau virtuel et de son sous-réseau doit être suffisante pour prendre en charge le nombre de pods que vous envisagez d’exécuter, ainsi que le nombre de nœuds du cluster.

Les adresses IP des pods et des nœuds de cluster sont affectées à partir du sous-réseau spécifié du réseau virtuel. Chaque nœud est configuré avec une adresse IP principale. Par défaut, 30 adresses IP supplémentaires sont préconfigurés par CNI Azure et affectées à des pods planifiés sur le nœud. Lorsque vous effectuez un scale-out de votre cluster, chaque nœud est configuré de manière similaire avec des adresses IP du sous-réseau. Vous pouvez également voir le nombre [maximal de pods par nœud](#maximum-pods-per-node).

> [!IMPORTANT]
> Le nombre d’adresses IP requises doit prendre en compte des considérations relatives aux opérations de mise à niveau et à l’échelle. Si vous définissez la plage d’adresses IP pour prendre en charge uniquement un nombre fixe de nœuds, vous ne pouvez pas mettre à niveau ou à l’échelle votre cluster.
>
> - Lorsque vous **mettez à niveau** votre cluster AKS, un nouveau nœud est déployé dans le cluster. Les services et charges de travail commencent à s’exécuter sur le nouveau nœud, et le nœud plus ancien est supprimé du cluster. Ce processus de mise à niveau propagée nécessite la disponibilité d’un minimum d’adresses IP ou de bloc supplémentaires. Le nombre de nœuds est alors `n + 1`.
>   - Cette considération est particulièrement importante lorsque vous utilisez des pools de nœuds Windows Server. Les nœuds Windows Server dans AKS n’appliquent pas automatiquement de mises à jour ; à la place, vous effectuez une mise à niveau sur le pool de nœuds. Cette mise à niveau déploie les nouveaux nœuds avec les derniers correctifs de sécurité et image de nœud de base Windows Server 2019. Pour plus d’informations sur la mise à niveau d’un pool de nœuds Windows Server, consultez [Upgrade a node pool][nodepool-upgrade] (Mettre à niveau un pool de nœuds).
>
> - Lorsque vous **mettez à l’échelle** un cluster AKS, un nouveau nœud est déployé dans le cluster. Les services et charges de travail commencent à s’exécuter sur le nouveau nœud. Votre plage d’adresses IP doit prendre en compte la manière dont vous voulez augmenter le nombre de nœuds et de pods que votre cluster prend en charge. Un nœud supplémentaire pour les opérations de mise à niveau doit également être inclus. Le nombre de nœuds est alors `n + number-of-additional-scaled-nodes-you-anticipate + 1`.

Si vous vous attendez à ce que vos nœuds exécutent le nombre maximal de pods, et détruisent et déploient régulièrement des pods, vous devez également prendre en compte des adresses IP supplémentaires par nœud. Ces adresses IP supplémentaires tiennent compte du fait que la suppression d’un service et la libération de l’adresse IP pour le déploiement et l’acquisition de l’adresse d’un nouveau service peuvent prendre quelques secondes.

Le plan d’adressage IP pour un cluster AKS se compose d’un réseau virtuel, d’au moins un sous-réseau pour les nœuds et les pods, et d’une plage d’adresses de service Kubernetes.

| Plage Azure/ressource Azure | Limites et tailles |
| --------- | ------------- |
| Réseau virtuel | Le réseau virtuel Azure peut être aussi volumineux que la valeur /8, mais est limité à 65 536 adresses IP configurées. Avant de configurer votre espace d’adressage, prenez en compte tous vos besoins en matière de mise en réseau, dont la communication avec des services dans d’autres réseaux virtuels. Par exemple, si vous configurez un espace d’adressage trop important, vous risquez de rencontrer des problèmes de chevauchement avec d’autres espaces d’adressage au sein de votre réseau.|
| Subnet | Doit pouvoir contenir les nœuds, les pods, ainsi que toutes les ressources Kubernetes et Azure qui peuvent être provisionnées dans votre cluster. Par exemple, si vous déployez un équilibreur de charge interne Azure, ses adresses IP frontend sont allouées à partir du sous-réseau du cluster, et non à partir des adresses IP non publiques. La taille du sous-réseau doit également prendre en compte les opérations de mise à niveau ou de futurs besoins de mise à l’échelle.<p />Pour calculer la taille de sous-réseau *minimale*, dont celle d’un nœud supplémentaire pour les opérations de mise à niveau : `(number of nodes + 1) + ((number of nodes + 1) * maximum pods per node that you configure)`<p/>Exemple pour un cluster à 50 nœuds : `(51) + (51  * 30 (default)) = 1,581` (/21 ou plus)<p/>Exemple pour un cluster de 50 nœuds incluant également un approvisionnement pour porter l’échelle à 10 nœuds supplémentaires : `(61) + (61 * 30 (default)) = 1,891` (/21 ou plus)<p>Si vous ne spécifiez pas de nombre maximal de pods par nœud lorsque vous créez votre cluster, le nombre maximal de pods par nœud est de *30*. Le nombre minimal d’adresses IP requises est basé sur cette valeur. Si vous calculez vos exigences d’adresse IP minimales sur une autre valeur maximale, consultez [comment configurer le nombre maximal de pods par nœud](#configure-maximum---new-clusters) pour définir cette valeur lorsque vous déployez votre cluster. |
| Plage d’adresses de service Kubernetes | Cette plage ne doit être utilisée par aucun élément réseau sur ce réseau virtuel ou connecté à celui-ci. Le CIDR d’adresse du service doit être inférieur à /12. Vous pouvez réutiliser cette plage sur différents clusters AKS. |
| Adresse IP du service DNS Kubernetes | Adresse IP dans la plage d'adresses de service Kubernetes, qui sera utilisée par la détection de service de cluster. N’utilisez pas la première adresse IP de votre plage d’adresses (1, par exemple). La première adresse de votre plage de sous-réseaux est utilisée pour l’adresse *kubernetes.default.svc.cluster.local*. |
| Adresse de pont Docker | L'adresse réseau du pont Docker représente l'adresse réseau *docker0* par défaut présente dans toutes les installations Docker. Bien que le pont *docker0* ne soit pas utilisé par les clusters AKS ou les pods eux-mêmes, vous devez définir cette adresse pour continuer à prendre en charge des scénarios tels que *docker build* au sein du cluster AKS. Vous devez sélectionner un CIDR pour l'adresse réseau du pont docker, sinon le docker choisira automatiquement un sous-réseau qui pourrait entrer en conflit avec d'autres CIDR. Vous devez choisir un espace d'adressage qui n'entre pas en collision avec le reste des CIDR de vos réseaux, y compris le CIDR du service du cluster et le CIDR du pod. Valeur par défaut : 172.17.0.1/16. Vous pouvez réutiliser cette plage sur différents clusters AKS. |

## <a name="maximum-pods-per-node"></a>Nombre maximal de pods par nœud

Le nombre maximal de pods par nœud dans un cluster AKS est de 250. Le nombre maximal *par défaut* de pods par nœud varie selon qu’il s’agit d’un réseau *kubenet* ou *Azure CNI*, mais aussi en fonction de la méthode de déploiement du cluster.

| Méthode de déploiement | Kubenet par défaut | Azure CNI par défaut | Configurable au moment du déploiement |
| -- | :--: | :--: | -- |
| Azure CLI | 110 | 30 | Oui (jusqu’à 250) |
| Modèle Resource Manager | 110 | 30 | Oui (jusqu’à 250) |
| Portail | 110 | 30 | Non |

### <a name="configure-maximum---new-clusters"></a>Configurer un maximum : nouveaux clusters

Vous pouvez configurer le nombre maximal de pods par nœud au moment du déploiement du cluster ou lorsque vous ajoutez de nouveaux pools de nœuds. Si vous procédez au déploiement avec Azure CLI ou avec un modèle Resource Manager, vous pouvez définir la valeur du nombre maximal de pods par nœud avec un maximum de 250.

Si vous ne spécifiez pas le paramètre maxPods lors de la création de pools de nœuds, vous recevez une valeur par défaut de 30 pour Azure CNI.

Une valeur minimale pour le nombre maximal de pods par nœud est appliquée afin de garantir l’espace des blocs système critiques pour l’intégrité du cluster. La valeur minimale qui peut être définie pour le nombre maximal de pods par nœud est égale à 10 si, et seulement si, la configuration de chaque pool de nœuds a un espace pour un minimum de 30 pods. Par exemple, la définition du nombre maximal de pods par nœud sur la valeur minimale de 10 nécessite que chaque pool de nœuds inclue un minimum de 3 nœuds. Cette exigence s’applique également à chaque pool de nœuds créé, de sorte que, si la valeur 10 est définie comme le nombre maximal de pods par nœud, chaque pool de nœuds ajouté doit inclure au moins 3 nœuds.

| Mise en réseau | Minimum | Maximale |
| -- | :--: | :--: |
| Azure CNI | 10 | 250 |
| Kubenet | 10 | 110 |

> [!NOTE]
> La valeur minimale dans le tableau ci-dessus est strictement appliquée par le service AKS. Vous ne pouvez pas définir une valeur maxPods inférieure à la valeur minimale, car cela peut empêcher le cluster de démarrer.

* **Azure CLI** : spécifiez l’argument `--max-pods` lorsque vous déployez un cluster avec la commande [az aks create][az-aks-create]. La valeur maximale est 250.
* **Modèle Resource Manager** : spécifiez la propriété `maxPods` dans l’objet [ManagedClusterAgentPoolProfile] lorsque vous déployez un cluster avec un modèle Resource Manager. La valeur maximale est 250.
* **Portail Azure**: vous ne pouvez pas modifier le nombre maximal de pods par nœud lorsque vous déployez un cluster avec le portail Azure. Les clusters de réseau Azure CNI sont limités à 30 pods par nœud quand vous effectuez le déploiement à partir du portail Azure.

### <a name="configure-maximum---existing-clusters"></a>Configurer un maximum : clusters existants

Le paramètre maxPod par nœud peut être défini lorsque vous créez un pool de nœuds. Si vous devez augmenter le paramètre maxPod par nœud sur un cluster existant, ajoutez un nouveau pool de nœuds avec le nouveau nombre de maxPod souhaité. Après la migration de vos pods vers le nouveau pool, supprimez le pool plus ancien. Pour supprimer un pool plus ancien dans un cluster, assurez-vous de définir les modes de pool de nœuds comme défini dans le [document de pools de nœuds système][system-node-pools].

## <a name="deployment-parameters"></a>Paramètres de déploiement

Quand vous créez un cluster AKS, les paramètres suivants sont configurables pour les réseaux Azure CNI :

**Réseau virtuel** : réseau virtuel dans lequel vous souhaitez déployer le cluster Kubernetes. Si vous souhaitez créer un réseau virtuel pour votre cluster, sélectionnez *Créer un nouveau*, puis suivez la procédure décrite dans la section *Créer un réseau virtuel*. Pour plus d’informations sur les limites et quotas d’un réseau virtuel Azure, voir [Abonnement Azure et limites, quotas et contraintes du service](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits).

**Sous-réseau** : sous-réseau du réseau virtuel dans lequel vous souhaitez déployer le cluster. Si vous souhaitez créer un nouveau sous-réseau dans le réseau virtuel pour votre cluster, sélectionnez *Créer un nouveau*, puis exécutez la procédure décrite dans la section *Créer un sous-réseau*. Pour une connectivité hybride, la plage d’adresses ne doit pas chevaucher d’autres réseaux virtuels dans votre environnement.

**Plage d’adresses du service Kubernetes** : il s’agit de l’ensemble d’adresses IP virtuelles que Kubernetes attribue aux [services][services] internes dans votre cluster. Vous pouvez utiliser n’importe quelle plage d’adresses privées répondant aux exigences suivantes :

* ne peut pas figurer dans la plage d’adresses IP de réseau virtuel de votre cluster ;
* ne peut pas présenter de chevauchement avec un autre réseau virtuel avec lequel le réseau virtuel du cluster s’apparie ;
* ne doit avoir aucun élément en commun avec des adresses IP locales ;
* ne doit pas être dans les plages `169.254.0.0/16`, `172.30.0.0/16`, `172.31.0.0/16` ou `192.0.2.0/24`

Bien qu’il soit techniquement possible de spécifier une plage d’adresses de service dans le même réseau virtuel que votre cluster, cette manière de procéder n’est pas recommandée. Tout chevauchement entre des plages d’adresses IP est susceptible d’entraîner des comportements imprévisibles. Pour plus d’informations, consultez la section [FAQ](#frequently-asked-questions) de cet article. Pour plus d’informations sur les services Kubernetes, consultez[Services][services] dans la documentation de Kubernetes.

**Adresse IP du service DNS Kubernetes** :  adresse IP du service DNS du cluster. Cette adresse doit se situer dans la *plage d’adresses du service Kubernetes*. N’utilisez pas la première adresse IP de votre plage d’adresses (1, par exemple). La première adresse de votre plage de sous-réseaux est utilisée pour l’adresse *kubernetes.default.svc.cluster.local*.

**Adresse du pont Docker** : L'adresse réseau du pont Docker représente l'adresse réseau *docker0* par défaut présente dans toutes les installations Docker. Bien que le pont *docker0* ne soit pas utilisé par les clusters AKS ou les pods eux-mêmes, vous devez définir cette adresse pour continuer à prendre en charge des scénarios tels que *docker build* au sein du cluster AKS. Vous devez sélectionner un CIDR pour l'adresse réseau du pont docker, sinon le docker choisira automatiquement un sous-réseau qui pourrait entrer en conflit avec d'autres CIDR. Vous devez choisir un espace d'adressage qui n'entre pas en collision avec le reste des CIDR de vos réseaux, y compris le CIDR du service du cluster et le CIDR du pod.

## <a name="configure-networking---cli"></a>Configurer la mise en réseau – Interface de ligne de commande

Lors de la création d’un cluster AKS avec Azure CLI, il est également possible de configurer un réseau Azure CNI. Utilisez les commandes suivantes pour créer un cluster AKS en activant un réseau Azure CNI.

Tout d’abord, récupérez l’ID de la ressource du sous-réseau auquel le cluster AKS sera joint :

```azurecli-interactive
$ az network vnet subnet list \
    --resource-group myVnet \
    --vnet-name myVnet \
    --query "[0].id" --output tsv

/subscriptions/<guid>/resourceGroups/myVnet/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/default
```

Utilisez la commande [az aks create][az-aks-create] avec l’argument `--network-plugin azure` pour créer un cluster avec mise en réseau avancée. Remplacez la valeur `--vnet-subnet-id` par l’ID du sous-réseau recueilli à l’étape précédente :

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --network-plugin azure \
    --vnet-subnet-id <subnet-id> \
    --docker-bridge-address 172.17.0.1/16 \
    --dns-service-ip 10.2.0.10 \
    --service-cidr 10.2.0.0/24 \
    --generate-ssh-keys
```

## <a name="configure-networking---portal"></a>Configurer la mise en réseau – Portail

La capture d’écran suivante de votre portail Azure représente un exemple de configuration de ces paramètres durant la création du cluster AKS :

![Configuration de la mise en réseau avancée dans le portail Azure][portal-01-networking-advanced]

## <a name="frequently-asked-questions"></a>Forum aux questions

La série suivante de questions-réponses s’applique à la configuration de réseaux **Azure CNI**.

* *Puis-je déployer des machines virtuelles dans le sous-réseau de mon cluster ?*

  Oui.

* *Quelle adresse IP source les systèmes externes voient-ils pour le trafic en provenance d’un pod compatible avec Azure CNI ?*

  Les systèmes dans le même réseau virtuel que le cluster AKS voient l’adresse IP du pod comme l’adresse source pour tout trafic en provenance du pod. Les systèmes situés en dehors du réseau virtuel du cluster AKS voient l’adresse IP du nœud en tant qu’adresse source pour tout le trafic en provenance du pod. 

* *Puis-je configurer des stratégies de réseau spécifiques aux pods ?*

  Oui, la stratégie de réseau Kubernetes est disponible dans AKS. Pour la prise en main, consultez [Sécuriser le trafic entre les pods avec des stratégies réseau dans Azure Kubernetes Service (AKS)][network-policy].

* *Le nombre maximal de pods pouvant être déployés sur un nœud peut-il être configuré ?*

  Oui, lorsque vous déployez un cluster avec l’interface CLI Azure ou un modèle Resource Manager. Consultez [Nombre maximal de pods par nœud](#maximum-pods-per-node).

  Vous ne pouvez pas modifier le nombre maximal de pods par nœud sur un cluster existant.

* *Comment configurer des propriétés supplémentaires pour le sous-réseau développé lors de la création du cluster AKS ? Par exemple, des points de terminaison du service.*

  La liste complète des propriétés pour le réseau virtuel et les sous- réseaux développés durant la création du cluster AKS peut être configurée dans la page de configuration du réseau virtuel standard du portail Azure.

* *Puis-je utiliser un autre sous-réseau dans le réseau virtuel de Mon cluster pour la* **plage d’adresses du service Kubernetes** ?

  Cette configuration, bien que possible, n’est pas recommandée. La plage d’adresses du service est un jeu d’adresses IP virtuelles que Kubernetes attribue aux services internes dans votre cluster. Azure Networking ne peut pas voir la plage d’adresses IP de service du cluster Kubernetes. En raison de ce manque de visibilité, il est possible de créer ultérieurement un sous-réseau dans le réseau virtuel du cluster, qui chevauche la plage d’adresses de service. Si un chevauchement de ce type se produit, Kubernetes peut affecter à un service une adresse IP déjà utilisée par une autre ressource dans le sous-réseau, ce qui provoque un comportement imprévisible ou des échecs. Utilisez une plage d’adresses en dehors du réseau virtuel du cluster pour éviter tout risque de chevauchement.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la mise en réseau dans AKS, consultez les articles suivants :

- [Utiliser une adresse IP statique avec l’équilibrage de charge d’Azure Kubernetes Service (AKS)](static-ip.md)
- [Utiliser un équilibreur de charge interne avec Azure Container Service (AKS)](internal-lb.md)

- [Créer un contrôleur d’entrée dans Azure Kubernetes Service (AKS)][aks-ingress-basic]
- [Routage d’applications HTTP][aks-http-app-routing]
- [Create an ingress controller to an internal virtual network in Azure Kubernetes Service (AKS)][aks-ingress-internal] (Créer un contrôleur d’entrée pour un réseau virtuel interne dans Azure Kubernetes Service (AKS))
- [Create an HTTPS ingress controller on Azure Kubernetes Service (AKS)][aks-ingress-tls] (Créer un contrôleur d’entrée HTTPS dans Azure Kubernetes Service (AKS))
- [Create an ingress controller with a static public IP address in Azure Kubernetes Service (AKS)][aks-ingress-static-tls] (Créer un contrôleur d’entrée avec une adresse IP publique statique dans Azure Kubernetes Service (AKS))

### <a name="aks-engine"></a>Moteur AKS

Le [moteur Azure Kubernetes Service (moteur AKS)][aks-engine] est un projet open source générant des modèles Azure Resource Manager que vous pouvez utiliser pour déployer des clusters Kubernetes sur Azure.

Les clusters Kubernetes créés avec le moteur AKS prennent en charge les plug-ins [kubenet][kubenet] et [Azure CNI][cni-networking]. En l’état, les deux scénarios de réseau sont pris en charge par le moteur AKS.

<!-- IMAGES -->
[advanced-networking-diagram-01]: ./media/networking-overview/advanced-networking-diagram-01.png
[portal-01-networking-advanced]: ./media/networking-overview/portal-01-networking-advanced.png

<!-- LINKS - External -->
[aks-engine]: https://github.com/Azure/aks-engine
[services]: https://kubernetes.io/docs/concepts/services-networking/service/
[portal]: https://portal.azure.com
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet

<!-- LINKS - Internal -->
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[aks-ssh]: ssh.md
[ManagedClusterAgentPoolProfile]: /azure/templates/microsoft.containerservice/managedclusters#managedclusteragentpoolprofile-object
[aks-network-concepts]: concepts-network.md
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-internal]: ingress-internal-ip.md
[network-policy]: use-network-policies.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[network-comparisons]: concepts-network.md#compare-network-models
[system-node-pools]: use-system-pools.md
