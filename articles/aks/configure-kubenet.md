---
title: Configurer la mise en réseau kubenet dans Azure Kubernetes Service (AKS)
description: Découvrez comment configurer le réseau kubenet (de base) dans Azure Kubernetes Service (AKS) pour déployer un cluster AKS dans un réseau virtuel et un sous-réseau existants.
services: container-service
ms.topic: article
ms.date: 06/02/2020
ms.reviewer: nieberts, jomore
ms.openlocfilehash: 6cb083e823583105f04aaa59a99357b2b2b2426b
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97034052"
---
# <a name="use-kubenet-networking-with-your-own-ip-address-ranges-in-azure-kubernetes-service-aks"></a>Utiliser la mise en réseau kubenet avec vos propres plages d’adresses IP dans Azure Kubernetes Service (AKS)

Par défaut, les clusters AKS utilisent [kubenet][kubenet]. Par ailleurs, un réseau et un sous-réseau virtuels Azure sont automatiquement créés. Avec *kubenet*, les nœuds obtiennent une adresse IP du sous-réseau du réseau virtuel Azure. Les pods reçoivent une adresse IP du sous-réseau de réseau virtuel Azure des nœuds à partir d’un espace d’adressage logiquement différent. La traduction d’adresses réseau (NAT) est ensuite configurée afin que les pods puissent accéder aux ressources sur le réseau virtuel Azure. L’adresse IP source du trafic fait l’objet d’une opération NAT sur l’adresse IP principale du nœud. Cette approche réduit considérablement le nombre d’adresses IP que vous devez réserver dans votre espace réseau pour que les pods les utilisent.

Avec l’interface [Azure Container Networking Interface (CNI)][cni-networking], chaque pod reçoit une adresse IP du sous-réseau et est accessible directement. Ces adresses IP doivent être uniques dans votre espace réseau et doivent être planifiées à l’avance. Chaque nœud possède un paramètre de configuration pour le nombre maximal de pods qu’il prend en charge. Le nombre équivalent d’adresses IP par nœud est alors réservé à l’avance pour ce nœud. Cette approche nécessite davantage de planification. De plus, elle conduit souvent à l’épuisement des adresses IP ou à la nécessité de regénérer les clusters dans un sous-réseau plus vaste à mesure que vos demandes d’applications augmentent. Vous pouvez configurer le nombre maximal de modules pouvant être déployés sur un nœud au moment de la création du cluster ou lors de la création de pools de nœuds. Si vous ne spécifiez pas maxPods lors de la création de pools de nœuds, vous recevez la valeur par défaut de 110 pour kubenet.

Cet article vous montre comment utiliser la mise en réseau *kubenet* pour créer et utiliser un sous-réseau de réseau virtuel pour un cluster AKS. Pour plus d’informations sur les options et considérations relatives aux réseaux, consultez [Concepts de réseau pour Kubernetes et AKS][aks-network-concepts].

## <a name="prerequisites"></a>Prérequis

* Le réseau virtuel du cluster AKS doit autoriser les connexions Internet sortantes.
* Ne créez pas plus d’un cluster AKS dans le même sous-réseau.
* Les clusters AKS ne peuvent pas utiliser `169.254.0.0/16`, `172.30.0.0/16`, `172.31.0.0/16` ou `192.0.2.0/24` pour la plage d’adresses de service Kubernetes, la plage d’adresses de pod ou la plage d’adresses de réseau virtuel de cluster.
* Le principal du service utilisé par le cluster AKS doit avoir au moins le rôle [Contributeur de réseau](../role-based-access-control/built-in-roles.md#network-contributor) sur le sous-réseau de votre réseau virtuel. Vous devez également disposer des autorisations appropriées, par exemple le propriétaire de l’abonnement, pour créer un principal de service et lui attribuer des autorisations. Si vous souhaitez définir un [rôle personnalisé](../role-based-access-control/custom-roles.md) au lieu d’utiliser le rôle de contributeur de réseau intégré, les autorisations suivantes sont nécessaires :
  * `Microsoft.Network/virtualNetworks/subnets/join/action`
  * `Microsoft.Network/virtualNetworks/subnets/read`

> [!WARNING]
> Pour utiliser des pools de nœuds Windows Server, vous devez utiliser Azure CNI. L’utilisation de kubenet comme modèle de réseau n’est pas disponible pour les conteneurs Windows Server.

## <a name="before-you-begin"></a>Avant de commencer

Azure CLI version 2.0.65 ou ultérieure doit être installé et configuré. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

## <a name="overview-of-kubenet-networking-with-your-own-subnet"></a>Vue d’ensemble de la mise en réseau kubenet avec votre propre sous-réseau

Dans de nombreux environnements, vous avez défini des réseaux et des sous-réseaux virtuels avec des plages d’adresses IP allouées. Ces ressources de réseau virtuel sont utilisées pour prendre en charge plusieurs services et applications. Pour fournir une connectivité réseau, les clusters AKS peuvent utiliser *kubenet* (mise en réseau de base) ou Azure CNI (*mise en réseau avancée*).

Avec *kubenet*, seuls les nœuds reçoivent une adresse IP dans le sous-réseau de réseau virtuel. Les pods ne peuvent pas communiquer directement entre eux. Au lieu de cela, les routes définies par l’utilisateur (UDR) et le transfert IP sont utilisés pour la connectivité entre les pods sur les nœuds. Par défaut, la configuration des itinéraires définis par l’utilisateur et des transferts IP est créée et gérée par le service AKS, mais vous avez la possibilité d’[utiliser votre propre table de routage pour la gestion des itinéraires personnalisés][byo-subnet-route-table]. Vous pouvez également déployer des pods derrière un service qui reçoit une adresse IP attribuée et équilibre la charge du trafic pour l’application. Le diagramme suivant illustre la façon dont les nœuds AKS reçoivent une adresse IP dans le sous-réseau de réseau virtuel, mais pas les pods :

![Modèle de réseau kubenet avec un cluster AKS](media/use-kubenet/kubenet-overview.png)

Azure prend en charge un maximum de 400 routes dans une route UDR. Vous ne pouvez donc pas avoir un cluster AKS comportant plus de 400 nœuds. Les [nœuds virtuels][virtual-nodes] AKS et les stratégies réseau Azure ne sont pas pris en charge avec *kubenet*.  Vous pouvez utiliser des [stratégies réseau Calico][calico-network-policies], car elles sont prises en charge avec kubenet.

Avec *Azure CNI*, chaque pod reçoit une adresse IP dans le sous-réseau IP et peut communiquer directement avec d’autres pods et services. La taille de vos clusters peut être identique à la plage d’adresses IP que vous spécifiez. Toutefois, la plage d’adresses IP doit être planifiée à l’avance, et toutes les adresses IP sont consommées par les nœuds AKS en fonction du nombre maximal de pods qu’ils peuvent prendre en charge. Les scénarios et fonctionnalités réseau avancé comme les [nœuds virtuels][virtual-nodes] ou les stratégies réseau (Azure ou Calico) sont pris en charge avec *Azure CNI*.

### <a name="limitations--considerations-for-kubenet"></a>Limitations et considérations relatives à kubenet

* Un tronçon supplémentaire est nécessaire dans la conception de kubenet, ce qui ajoute une latence mineure à la communication des pods.
* L’utilisation de kubenet requiert des tables de routage et des itinéraires définis par l’utilisateur, ce qui complique les opérations.
* Le design de kubenet ne permet pas la prise en charge de l’adressage direct aux pods.
* Contrairement aux clusters Azure CNI, plusieurs clusters kubenet ne peuvent pas partager un sous-réseau.
* Les fonctionnalités **non prises en charge sur kubenet** comprennent :
   * les [stratégies réseau Azure](use-network-policies.md#create-an-aks-cluster-and-enable-network-policy), mais les stratégies réseau Calico sont prises en charge sur kubenet ;
   * les [pools de nœuds Windows](./windows-faq.md) ;
   * le [module complémentaire de nœuds virtuels](virtual-nodes.md#network-requirements).

### <a name="ip-address-availability-and-exhaustion"></a>Disponibilité et épuisement des adresses IP

Avec *Azure CNI*, une plage d’adresses IP attribuée trop petite pour ensuite ajouter des nœuds supplémentaires quand vous mettez à l’échelle ou mettez à niveau un cluster constitue un problème courant. L’équipe réseau peut également ne pas être en mesure d’émettre une plage d’adresses IP suffisamment grande pour prendre en charge vos demandes d’applications attendues.

À titre de compromis, vous pouvez créer un cluster AKS qui utilise *kubenet* et vous connecter à un sous-réseau de réseau virtuel existant. Cette approche permet aux nœuds de recevoir des adresses IP définies sans avoir besoin de réserver un grand nombre d’adresses IP à l’avance pour tous les pods potentiels qui pourraient s’exécuter dans le cluster.

Avec *kubenet*, vous pouvez utiliser une plage d’adresses IP beaucoup plus petite et prendre en charge de grands clusters et les demandes d’applications. Par exemple, même avec une plage d’adresses IP  */27* sur votre sous-réseau, vous pouvez exécuter un cluster de 20 à 25 nœuds avec suffisamment de place pour effectuer une mise à l’échelle ou une mise à niveau. Cette taille de cluster prend en charge jusqu’à *2 200 à 2 750* pods (avec un maximum par défaut de 110 pods par nœud). Le nombre maximal de pods par nœud que vous pouvez configurer avec *kubenet* dans AKS est 110.

Les calculs de base suivants comparent la différence entre les modèles de réseaux :

- **kubenet** : une simple plage d’adresses IP  */24* peut prendre en charge jusqu’à *251* nœuds dans le cluster (chaque sous-réseau de réseau virtuel Azure réserve les trois premières adresses IP pour les opérations de gestion)
  - Ce nombre de nœuds peut prendre en charge jusqu’à *27 610* pods (avec un maximum par défaut de 110 pods par nœud avec *kubenet*)
- **Azure CNI** : cette même plage de sous-réseau  */24* de base peut seulement prendre en charge un maximum de *8* nœuds dans le cluster
  - Ce nombre de nœuds peut seulement prendre en charge jusqu’à *240* pods (avec un maximum par défaut de 30 pods par nœud avec *Azure CNI*)

> [!NOTE]
> Ces valeurs maximales ne prennent pas en compte les opérations de mise à niveau ou de mise à l’échelle. Dans la pratique, vous ne pouvez pas exécuter le nombre maximal de nœuds que la plage d’adresses IP de sous-réseau prend en charge. Vous devez laisser certaines adresses IP disponibles pour qu’elles puissent être utilisées pendant les opérations de mise à l’échelle ou de mise à niveau.

### <a name="virtual-network-peering-and-expressroute-connections"></a>Peering de réseau virtuel et connexions ExpressRoute

Pour fournir une connectivité locale, les approches des réseaux *kubenet* et *Azure CNI* peuvent toutes les deux utiliser le [Peering de réseaux virtuels Azure][vnet-peering] ou les [connexions ExpressRoute][express-route]. Planifiez vos plages d’adresses IP avec soin pour éviter le chevauchement et un routage incorrect du trafic. Par exemple, de nombreux réseaux locaux utilisent une plage d’adresses *10.0.0.0/8* qui est publiée sur la connexion ExpressRoute. Il est recommandé de créer vos clusters AKS dans des sous-réseaux de réseau virtuel Azure en dehors de cette plage d’adresses, comme *172.16.0.0/16*.

### <a name="choose-a-network-model-to-use"></a>Choisir un modèle de réseau à utiliser

Le choix du plug-in réseau à utiliser pour votre cluster AKS est généralement un équilibre entre les besoins de flexibilité et les besoins de configuration avancée. Les considérations suivantes aident à déterminer quand chaque modèle de réseau peut être le plus approprié.

Utilisez *kubenet* quand :

- Vous avez un espace d’adressage IP limité.
- La majeure partie de la communication des pods s’effectue dans le cluster.
- Vous n’avez pas besoin de fonctionnalités AKS avancées comme des nœuds virtuels ou une stratégie réseau Azure.  Utilisez des [stratégies réseau Calico][calico-network-policies].

Utilisez *Azure CNI* quand :

- Vous disposez d’espace d’adressage IP disponible.
- La majeure partie de la communication des pods s’effectue avec des ressources hors du cluster.
- Vous ne souhaitez pas gérer les itinéraires définis par l’utilisateur pour la connectivité des pods.
- Vous avez besoin de fonctionnalités avancées AKS comme des nœuds virtuels ou une stratégie réseau Azure.  Utilisez des [stratégies réseau Calico][calico-network-policies].

Pour vous aider à déterminer quel modèle de réseau utiliser, consultez les sections [Comparer des modèles de réseau et Étendue du support][network-comparisons].

## <a name="create-a-virtual-network-and-subnet"></a>Créer un réseau virtuel et un sous-réseau

Pour bien commencer avec l’utilisation de *kubenet* et de votre propre sous-réseau de réseau virtuel, commencez par créer un groupe de ressources à l’aide de la commande [az group create][az-group-create]. L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Si vous n’avez pas de réseau virtuel et de sous-réseau existants à utiliser, créez ces ressources réseau à l’aide de la commande [az network vnet create][az-network-vnet-create]. Dans l’exemple suivant, le réseau virtuel est nommé *myVnet* avec le préfixe d’adresse *192.168.0.0/16*. Un sous-réseau appelé *myAKSSubnet* avec le préfixe d’adresse *192.168.1.0/24* est créé.

```azurecli-interactive
az network vnet create \
    --resource-group myResourceGroup \
    --name myAKSVnet \
    --address-prefixes 192.168.0.0/16 \
    --subnet-name myAKSSubnet \
    --subnet-prefix 192.168.1.0/24
```

## <a name="create-a-service-principal-and-assign-permissions"></a>Créer un principal de service et attribuer des autorisations

Pour permettre à un cluster AKS d’interagir avec d’autres ressources Azure, un principal du service Azure Active Directory est utilisé. Le principal du service doit disposer d’autorisations pour gérer le réseau virtuel et le sous-réseau que les nœuds AKS utilisent. Pour créer un principal de service, utilisez la commande [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] :

```azurecli-interactive
az ad sp create-for-rbac --skip-assignment
```

L’exemple de sortie suivant montre l’ID d’application et le mot de passe de votre principal de service. Ces valeurs sont utilisées dans des étapes supplémentaires pour attribuer un rôle au principal de service, puis créer le cluster AKS :

```azurecli
az ad sp create-for-rbac --skip-assignment
```

```output
{
  "appId": "476b3636-5eda-4c0e-9751-849e70b5cfad",
  "displayName": "azure-cli-2019-01-09-22-29-24",
  "name": "http://azure-cli-2019-01-09-22-29-24",
  "password": "a1024cd7-af7b-469f-8fd7-b293ecbb174e",
  "tenant": "72f998bf-85f1-41cf-92ab-2e7cd014db46"
}
```

Pour attribuer les délégations correctes dans les étapes restantes, utilisez les commandes [az network vnet show][az-network-vnet-show] et [az network vnet subnet show][az-network-vnet-subnet-show] pour obtenir les ID de ressource nécessaires. Ces ID de ressource sont stockés sous forme de variables et référencés dans les étapes restantes :

```azurecli-interactive
VNET_ID=$(az network vnet show --resource-group myResourceGroup --name myAKSVnet --query id -o tsv)
SUBNET_ID=$(az network vnet subnet show --resource-group myResourceGroup --vnet-name myAKSVnet --name myAKSSubnet --query id -o tsv)
```

Attribuez maintenant le principal de service pour les autorisations de *Contributeur réseau* à votre cluster AKS sur le réseau virtuel à l’aide de la commande [az role assignment create][az-role-assignment-create]. Fournissez votre propre *\<appId>* , comme indiqué dans la sortie de la commande précédente pour créer le principal de service :

```azurecli-interactive
az role assignment create --assignee <appId> --scope $VNET_ID --role "Network Contributor"
```

## <a name="create-an-aks-cluster-in-the-virtual-network"></a>Créer un cluster AKS dans le réseau virtuel

Vous venez de créer un réseau virtuel et un sous-réseau. Vous avez également créé et attribué des autorisations pour un principal de service afin d’utiliser ces ressources réseau. Créez maintenant un cluster AKS dans votre réseau virtuel et sous-réseau à l’aide la commande [az aks create][az-aks-create]. Définissez vos propres *\<appId>* et *\<password>* de principal de service, comme indiqué dans la sortie de la commande précédente pour créer le principal de service.

Les plages d’adresses IP suivantes sont également définies dans le cadre du processus de création du cluster :

* *--service-cidr* est utilisée pour affecter des services internes dans l’adresse IP d’un cluster AKS. Cette plage d’adresses IP doit être un espace d’adressage qui n’est pas utilisé ailleurs dans votre environnement réseau, y compris dans des plages réseau locales, si vous connectez, ou envisagez de connecter, vos réseaux virtuels Azure à l’aide d’ExpressRoute ou d’une connexion VPN site à site.

* L’adresse *--dns-service-ip* doit être l’adresse  *.10* de la plage d’adresses IP de votre service.

* *--pod-cidr* doit être un grand espace d’adressage qui n’est pas utilisé ailleurs dans votre environnement réseau. Cette plage inclut les plages de réseau local si vous connectez ou envisagez de connecter vos réseaux virtuels Azure à l’aide d’Express Route ou d’une connexion VPN de site à site.
    * Cette plage d’adresses doit être suffisamment grande pour contenir le nombre de nœuds que vous prévoyez d’obtenir par le biais d’un scale-up. Vous ne pouvez pas changer cette plage d’adresses une fois le cluster déployé si vous avez besoin de davantage d’adresses pour des nœuds supplémentaires.
    * La plage d’adresses IP de pod est utilisée pour attribuer un espace d’adressage  */24* pour chaque nœud du cluster. Dans l’exemple suivant, l’adresse *--pod-cidr* *10.244.0.0/16* attribue le premier nœud *10.244.0.0/24*, le deuxième nœud *10.244.1.0/24* et le troisième nœud *10.244.2.0/24*.
    * À mesure que le cluster est mis à l’échelle ou à niveau, la plateforme Azure continue d’attribuer une plage d’adresses IP de pod à chaque nouveau nœud.
    
* Le pont *--docker-bridge-address* permet aux nœuds AKS de communiquer avec la plateforme de gestion sous-jacente. Cette adresse IP ne doit pas être dans la plage d’adresses IP du réseau virtuel de votre cluster, ni chevaucher d’autres plages d’adresses actuellement utilisées sur votre réseau.

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 3 \
    --network-plugin kubenet \
    --service-cidr 10.0.0.0/16 \
    --dns-service-ip 10.0.0.10 \
    --pod-cidr 10.244.0.0/16 \
    --docker-bridge-address 172.17.0.1/16 \
    --vnet-subnet-id $SUBNET_ID \
    --service-principal <appId> \
    --client-secret <password>
```

> [!Note]
> Si vous souhaitez permettre à un cluster AKS d’inclure une [stratégie réseau Calico][calico-network-policies], vous pouvez utiliser la commande suivante.

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 3 \
    --network-plugin kubenet --network-policy calico \
    --service-cidr 10.0.0.0/16 \
    --dns-service-ip 10.0.0.10 \
    --pod-cidr 10.244.0.0/16 \
    --docker-bridge-address 172.17.0.1/16 \
    --vnet-subnet-id $SUBNET_ID \
    --service-principal <appId> \
    --client-secret <password>
```

Lorsque vous créez un cluster AKS, un groupe de sécurité réseau et une table de route sont automatiquement créés. Ces ressources réseau sont gérées par le plan de contrôle AKS. Le groupe de sécurité réseau est automatiquement associé aux cartes réseau virtuelles sur vos nœuds. La table de routage est automatiquement associée au sous-réseau de réseau virtuel. Les règles de groupe de sécurité réseau et les tables de routage sont automatiquement mises à jour quand vous créez et exposez des services.

## <a name="bring-your-own-subnet-and-route-table-with-kubenet"></a>Utiliser votre propre sous-réseau et votre propre table de route avec kubenet

Avec kubenet, une table de route doit exister sur les sous-réseaux de votre cluster. AKS prend en charge l'utilisation de votre propre sous-réseau et de votre propre table de route.

Si votre sous-réseau personnalisé ne contient pas de table de route, AKS en crée une pour vous et y ajoute des règles tout au long du cycle de vie du cluster. Si votre sous-réseau personnalisé contient une table de route lorsque vous créez votre cluster, AKS la reconnaît lors des opérations de cluster et ajoute/met à jour les règles en conséquence pour les opérations du fournisseur de cloud.

> [!WARNING]
> Des règles personnalisées peuvent être ajoutées à la table de route personnalisée et mises à jour. Toutefois, des règles sont ajoutées par le fournisseur de cloud Kubernetes, lesquelles ne doit pas être mises à jour ni supprimées. Les règles telles que 0.0.0.0/0 doivent toujours exister sur une table de route donnée et être mappées à la cible de votre passerelle Internet, telle qu’une appliance virtuelle réseau ou une autre passerelle de sortie. Faites attention, lorsque vous mettez à jour les règles, à ce que seules vos règles personnalisées soient modifiées.

En savoir plus sur la configuration d’une [table de route personnalisée][custom-route-table].

La mise en réseau Kubenet nécessite que les règles de la table de route soient organisées pour acheminer correctement les demandes. En raison de cette conception, les tables de route doivent être soigneusement tenues à jour pour chaque cluster qui en dépend. Plusieurs clusters ne peuvent pas partager de table de route, car les CIDR de pods des différents clusters peuvent se chevaucher, ce qui provoque un routage inattendu et interrompu. Lorsque vous configurez plusieurs clusters sur le même réseau virtuel ou dédiez un réseau virtuel à chaque cluster, vérifiez que les limitations suivantes sont prises en compte.

Limites :

* Les autorisations doivent être attribuées avant la création du cluster. Veillez à utiliser un principal de service doté d'autorisations d'écriture sur votre sous-réseau et votre table de route personnalisés.
* Une table de route personnalisée doit être associée au sous-réseau avant de créer le cluster AKS.
* La ressource de table de route associée ne peut pas être mise à jour après la création du cluster. Même si la ressource de table de route ne peut pas être mise à jour, les règles personnalisées peuvent être modifiées dans la table de route.
* Chaque cluster AKS doit utiliser une table de route unique pour tous les sous-réseaux associés au cluster. Vous ne pouvez pas réutiliser une table de route avec plusieurs clusters en raison du risque de chevauchement des CIDR de pods et de règles d’acheminement contradictoires.

Après avoir créé une table de route personnalisée et l’avoir associée à votre sous-réseau dans votre réseau virtuel, vous pouvez créer un nouveau cluster AKS qui utilise votre table de route.
Vous devez utiliser l’ID de sous-réseau pour l’emplacement où vous envisagez de déployer votre cluster AKS. Ce sous-réseau doit également être associé à votre table de route personnalisée.

```azurecli-interactive
# Find your subnet ID
az network vnet subnet list --resource-group
                            --vnet-name
                            [--subscription]
```

```azurecli-interactive
# Create a kubernetes cluster with with a custom subnet preconfigured with a route table
az aks create -g MyResourceGroup -n MyManagedCluster --vnet-subnet-id MySubnetID
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant qu’un cluster AKS est déployé dans votre sous-réseau de réseau virtuel existant, vous pouvez utiliser le cluster comme d’habitude. Commencez par [créer des applications à l’aide d’Azure Dev Spaces][dev-spaces], [déployer des applications existantes à l’aide de Helm][use-helm], ou [créer des applications à l’aide de Helm][develop-helm].

<!-- LINKS - External -->
[dev-spaces]: ../dev-spaces/index.yml
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet
[Calico-network-policies]: https://docs.projectcalico.org/v3.9/security/calico-network-policy

<!-- LINKS - Internal -->
[install-azure-cli]: /cli/azure/install-azure-cli
[aks-network-concepts]: concepts-network.md
[az-group-create]: /cli/azure/group#az-group-create
[az-network-vnet-create]: /cli/azure/network/vnet#az-network-vnet-create
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-network-vnet-show]: /cli/azure/network/vnet#az-network-vnet-show
[az-network-vnet-subnet-show]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-aks-create]: /cli/azure/aks#az-aks-create
[byo-subnet-route-table]: #bring-your-own-subnet-and-route-table-with-kubenet
[develop-helm]: quickstart-helm.md
[use-helm]: kubernetes-helm.md
[virtual-nodes]: virtual-nodes-cli.md
[vnet-peering]: ../virtual-network/virtual-network-peering-overview.md
[express-route]: ../expressroute/expressroute-introduction.md
[network-comparisons]: concepts-network.md#compare-network-models
[custom-route-table]: ../virtual-network/manage-route-table.md
