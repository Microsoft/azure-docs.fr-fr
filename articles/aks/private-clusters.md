---
title: Créer un cluster Azure Kubernetes Service privé
description: Découvrez comment créer un cluster Azure Kubernetes Service (AKS) privé
services: container-service
ms.topic: article
ms.date: 7/17/2020
ms.openlocfilehash: 450d68e26c5a3fc1ecfbaf6a3be6b5f698ee65e3
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183258"
---
# <a name="create-a-private-azure-kubernetes-service-cluster"></a>Créer un cluster Azure Kubernetes Service privé

Dans un cluster privé, le plan de contrôle ou le serveur d’API a des adresses IP internes qui sont définies dans le document [RFC1918 - Address Allocation for Private Internets](https://tools.ietf.org/html/rfc1918) (RFC1918 - Allocation d’adresses pour les réseaux Internet privés). En utilisant un cluster privé, vous pouvez vous assurer que le trafic réseau entre votre serveur d’API et vos pools de nœuds reste sur le réseau privé uniquement.

Le plan de contrôle ou le serveur d’API se trouve dans un abonnement Azure géré par Azure Kubernetes Service (AKS). Le cluster ou le pool de nœuds d’un client se trouve dans l’abonnement du client. Le serveur et le cluster ou le pool de nœuds peuvent communiquer entre eux par le biais du [service Azure Private Link][private-link-service] dans le réseau virtuel du serveur d’API et d’un point de terminaison privé exposé dans le sous-réseau du cluster AKS du client.

## <a name="region-availability"></a>Disponibilité des régions

Le cluster privé est disponible dans les régions public Azure Government et Azure China 21Vianet où [AKS est pris en charge](https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service).

> [!NOTE]
> Les sites Azure Government sont pris en charge. Cependant, US Gov Texas n’est pas pris en charge pour le moment, en raison de l’absence de prise en charge du service Liaison privée.

## <a name="prerequisites"></a>Prérequis

* Azure CLI version 2.2.0 ou ultérieure

## <a name="create-a-private-aks-cluster"></a>Créer un cluster AKS privé

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources ou utilisez un groupe de ressources existant pour votre cluster AKS.

```azurecli-interactive
az group create -l westus -n MyResourceGroup
```

### <a name="default-basic-networking"></a>Mise en réseau de base par défaut 

```azurecli-interactive
az aks create -n <private-cluster-name> -g <private-cluster-resource-group> --load-balancer-sku standard --enable-private-cluster  
```
Où `--enable-private-cluster` est un indicateur obligatoire pour un cluster privé. 

### <a name="advanced-networking"></a>Mise en réseau avancée  

```azurecli-interactive
az aks create \
    --resource-group <private-cluster-resource-group> \
    --name <private-cluster-name> \
    --load-balancer-sku standard \
    --enable-private-cluster \
    --network-plugin azure \
    --vnet-subnet-id <subnet-id> \
    --docker-bridge-address 172.17.0.1/16 \
    --dns-service-ip 10.2.0.10 \
    --service-cidr 10.2.0.0/24 
```
Où `--enable-private-cluster` est un indicateur obligatoire pour un cluster privé. 

> [!NOTE]
> Si le CIDR d’adresse de pont Docker (172.17.0.1/16) est en conflit avec le CIDR de sous-réseau, changez l’adresse de pont Docker en conséquence.

## <a name="options-for-connecting-to-the-private-cluster"></a>Options de connexion au cluster privé

Le point de terminaison du serveur d’API n’a pas d’adresse IP publique. Pour gérer le serveur d’API, vous devez utiliser une machine virtuelle qui a accès au réseau virtuel (VNet) Azure du cluster AKS. Il existe plusieurs options pour établir une connectivité réseau avec le cluster privé.

* Créez une machine virtuelle dans le même réseau virtuel Azure (VNet) que le cluster AKS.
* Utilisez une machine virtuelle dans un réseau distinct et configurez l'[appairage de réseaux virtuels][virtual-network-peering].  Pour plus d'informations sur cette option, consultez la section ci-dessous.
* Utilisez une [connexion ExpressRoute ou VPN][express-route-or-VPN].

La création d’une machine virtuelle dans le même réseau virtuel que le cluster AKS constitue l’option la plus simple.  ExpressRoute et les VPN présentent des coûts et une complexité de mise en réseau supplémentaires.  L’appairage de réseaux virtuels implique la planification de vos plages CIDR réseau pour veiller à ce qu'aucune plage ne se chevauche.

## <a name="virtual-network-peering"></a>Peering de réseau virtuel

Comme indiqué, l’appairage de réseaux virtuels est un moyen d’accéder à votre cluster privé. Pour utiliser l’appairage de réseaux virtuels, vous devez configurer une liaison entre le réseau virtuel et la zone DNS privée.
    
1. Accédez au groupe de ressources du nœud dans le portail Azure.  
2. Sélectionnez la zone DNS privée.   
3. Dans le volet de gauche, sélectionnez le lien **Réseau virtuel**.  
4. Créez un lien permettant d’ajouter le réseau virtuel de la machine virtuelle à la zone DNS privée. Il faut quelques minutes pour que le lien de zone DNS soit disponible.  
5. Dans le portail Azure, accédez au groupe de ressources contenant le réseau virtuel de votre cluster.  
6. Dans le volet de droite, sélectionnez le réseau virtuel. Le nom du réseau virtuel se présente au format *aks-vnet-\** .  
7. Dans le volet de gauche, sélectionnez **Appairages**.  
8. Sélectionnez **Ajouter**, ajoutez le réseau virtuel de la machine virtuelle, puis créez l’appairage.  
9. Accédez au réseau virtuel sur lequel se trouve la machine virtuelle, sélectionnez **Appairages**, sélectionnez le réseau virtuel AKS, puis créez l’appairage. Si les plages d’adresses sur le réseau virtuel AKS et le réseau virtuel de la machine virtuelle sont en conflit, l’appairage échoue. Pour plus d’informations, consultez [Appairage de réseaux virtuels][virtual-network-peering].

## <a name="hub-and-spoke-with-custom-dns"></a>Hub-and-spoke avec DNS personnalisé

[Les architectures Hub-and-spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) sont couramment utilisées pour déployer des réseaux dans Azure. Dans la plupart de ces déploiements, les paramètres DNS des réseaux virtuels spoke sont configurés pour faire référence à un redirecteur DNS central afin d’autoriser la résolution DNS locale et basée sur Azure. Lors du déploiement d’un cluster AKS dans un environnement de mise en réseau de ce type, certaines considérations spéciales doivent être prises en compte.

![Hub-and-spoke de cluster privé](media/private-clusters/aks-private-hub-spoke.png)

1. Par défaut, lorsqu’un cluster privé est approvisionné, un point de terminaison privé (1) et une zone DNS privée (2) sont créés dans le groupe de ressources managé par le cluster. Le cluster utilise un enregistrement A dans la zone privée pour résoudre l’adresse IP du point de terminaison privé pour la communication avec le serveur d’API.

2. La zone DNS privée est liée uniquement au réseau virtuel auquel les nœuds de cluster sont attachés (3). Cela signifie que le point de terminaison privé peut uniquement être résolu par les hôtes de ce réseau virtuel lié. Dans les scénarios où aucun DNS personnalisé n’est configuré sur le réseau virtuel (par défaut), cela fonctionne sans problème, car les hôtes pointent vers l’adresse 168.63.129.16 pour le DNS qui peut résoudre les enregistrements dans la zone DNS privée en raison de la liaison.

3. Dans les scénarios où le réseau virtuel contenant votre cluster présente des paramètres DNS personnalisés (4), le déploiement du cluster échoue, sauf si la zone DNS privée est liée au réseau virtuel qui contient les programmes de résolution DNS personnalisés (5). Ce lien peut être créé manuellement après la création de la zone privée lors de l’approvisionnement du cluster ou via l’automatisation lors de la détection de la création de la zone à l’aide de mécanismes de déploiement basés sur les événements (par exemple, Azure Event Grid et Azure Functions).

## <a name="dependencies"></a>Les dépendances  

* Le service Liaison privée est pris en charge uniquement sur Azure Load Balancer Standard. Azure Load Balancer De base n’est pas pris en charge.  
* Pour utiliser un serveur DNS personnalisé, ajoutez l’IP Azure DNS 168.63.129.16 en tant que serveur DNS en amont dans le serveur DNS personnalisé.

## <a name="limitations"></a>Limites 
* Les plages d’adresses IP autorisées ne peuvent pas être appliquées au point de terminaison du serveur d’API privé, elles sont uniquement applicables au serveur d’API public.
* Les [zones de disponibilité][availability-zones] sont actuellement prises en charge pour certaines régions. 
* Les [limitations du service Azure Private Link][private-link-service] s’appliquent aux clusters privés.
* Aucune prise en charge des agents hébergés par Microsoft Azure DevOps avec des clusters privés. Envisagez d’utiliser des [agents auto-hébergés][devops-agents]. 
* Pour les clients qui doivent activer Azure Container Registry afin d’utiliser des clusters AKS privés, le réseau virtuel Container Registry doit être appairé avec le réseau virtuel du cluster d’agent.
* Aucune prise en charge de Azure Dev Spaces
* Aucune prise en charge de la conversion de clusters AKS existants en clusters privés
* La suppression ou la modification du point de terminaison privé dans le sous-réseau du client entraîne l’arrêt du fonctionnement du cluster. 
* Les données actives Azure Monitor pour conteneurs ne sont actuellement pas prises en charge.
* Le Contrat SLA de durée de fonctionnement n’est pas pris en charge.


<!-- LINKS - internal -->
[az-provider-register]: /cli/azure/provider?view=azure-cli-latest#az-provider-register
[az-feature-list]: /cli/azure/feature?view=azure-cli-latest#az-feature-list
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[private-link-service]: ../private-link/private-link-service-overview.md#limitations
[virtual-network-peering]: ../virtual-network/virtual-network-peering-overview.md
[azure-bastion]: ../bastion/tutorial-create-host-portal.md
[express-route-or-vpn]: ../expressroute/expressroute-about-virtual-network-gateways.md
[devops-agents]: /azure/devops/pipelines/agents/agents?view=azure-devops
[availability-zones]: availability-zones.md