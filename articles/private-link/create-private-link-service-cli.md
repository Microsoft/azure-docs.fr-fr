---
title: Créer un service Azure Private Link à l’aide d’Azure CLI
description: Découvrez comment créer un service Azure Private Link à l’aide d’Azure CLI
services: private-link
author: malopMSFT
ms.service: private-link
ms.topic: how-to
ms.date: 09/16/2019
ms.author: allensu
ms.openlocfilehash: cfffafaab2e2d4ef6b165ef03beb827342c94608
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96018050"
---
# <a name="create-a-private-link-service-using-azure-cli"></a>Créer un service Private Link à l’aide d’Azure CLI
Cet article explique comment créer un service Private Link dans Azure à l’aide d’Azure CLI.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la dernière version de la ligne de commande Azure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-private-link-service"></a>Créer un service Liaison privée
### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Avant de pouvoir créer un réseau virtuel, vous devez créer un groupe de ressources qui hébergera le réseau virtuel. Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). Cet exemple crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *westcentralus* :

```azurecli-interactive
az group create --name myResourceGroup --location westcentralus
```
### <a name="create-a-virtual-network"></a>Créez un réseau virtuel
Créez un réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create). Cet exemple crée un réseau virtuel par défaut nommé *myVirtualNetwork* avec un sous-réseau nommé *mySubnet* :

```azurecli-interactive
az network vnet create --resource-group myResourceGroup --name myVirtualNetwork --address-prefix 10.0.0.0/16  
```
### <a name="create-a-subnet"></a>Créer un sous-réseau
Créez un sous-réseau pour le réseau virtuel avec la commande [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create). Cet exemple crée un sous-réseau nommé *mySubnet* dans le réseau virtuel *myVirtualNetwork* :

```azurecli-interactive
az network vnet subnet create --resource-group myResourceGroup --vnet-name myVirtualNetwork --name mySubnet --address-prefixes 10.0.0.0/24    
```
### <a name="create-a-internal-load-balancer"></a>Créer un équilibreur de charge interne 
Créez un équilibreur de charge interne avec la commande [az network lb create](/cli/azure/network/lb#az-network-lb-create). Cet exemple crée un équilibreur de charge interne nommé *myILB* dans le groupe de ressources nommé *myResourceGroup*. 

```azurecli-interactive
az network lb create --resource-group myResourceGroup --name myILB --sku standard --vnet-name MyVirtualNetwork --subnet mySubnet --frontend-ip-name myFrontEnd --backend-pool-name myBackEndPool
```

### <a name="create-a-load-balancer-health-probe"></a>Créer une sonde d’intégrité d’équilibreur de charge

Une sonde d’intégrité vérifie toutes les instances de machine virtuelle pour s’assurer qu’elles peuvent recevoir le trafic réseau. L’instance de machine virtuelle présentant des contrôles de sonde défaillants est supprimée de l’équilibrage de charge jusqu’à ce qu’elle revienne en ligne et que la sonde valide son intégrité. Créez une sonde d’intégrité à l’aide de la commande [az network lb probe create](/cli/azure/network/lb/probe?view=azure-cli-latest) pour surveiller l’intégrité des machines virtuelles. 

```azurecli-interactive
  az network lb probe create \
    --resource-group myResourceGroup \
    --lb-name myILB \
    --name myHealthProbe \
    --protocol tcp \
    --port 80   
```

### <a name="create-a-load-balancer-rule"></a>Créer une règle d’équilibreur de charge

Une règle d’équilibreur de charge définit la configuration IP frontale pour le trafic entrant et le pool d’IP principal pour recevoir le trafic, ainsi que le port source et le port de destination requis. Créez une règle d’équilibreur de charge *myHTTPRule* à l’aide de la commande [az network lb rule create](/cli/azure/network/lb/rule?view=azure-cli-latest) pour écouter le port 80 dans le pool frontal *myFrontEnd* et envoyer le trafic réseau équilibré en charge vers le pool d’adresses principal *myBackEndPool* à l’aide du port 80 également. 

```azurecli-interactive
  az network lb rule create \
    --resource-group myResourceGroup \
    --lb-name myILB \
    --name myHTTPRule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name myFrontEnd \
    --backend-pool-name myBackEndPool \
    --probe-name myHealthProbe  
```
### <a name="create-backend-servers"></a>Créer des serveurs principaux

Dans cet exemple, nous ne couvrons pas la création d’une machine virtuelle. Vous pouvez suivre les étapes décrites dans [Démarrage rapide : Créer un équilibreur de charge interne pour équilibrer la charge de machines virtuelles avec Azure CLI](../load-balancer/quickstart-load-balancer-standard-internal-cli.md) pour créer deux machines virtuelles à utiliser en tant que serveurs principaux pour l’équilibreur de charge. 


### <a name="disable-private-link-service-network-policies-on-subnet"></a>Désactiver les stratégies réseau du service Private Link sur le sous-réseau 
Private Link requiert une adresse IP de n’importe quel sous-réseau de votre choix au sein d’un réseau virtuel. Actuellement, nous ne prenons pas en charge les stratégies réseau sur ces adresses IP.  Par conséquent, nous devons désactiver les stratégies réseau sur le sous-réseau. Mettez à jour le sous-réseau pour désactiver les stratégies réseau Private Link avec [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update).

```azurecli-interactive
az network vnet subnet update --resource-group myResourceGroup --vnet-name myVirtualNetwork --name mySubnet --disable-private-link-service-network-policies true 
```
 
## <a name="create-a-private-link-service-using-standard-load-balancer"></a>Créer un service Azure Private Link à l’aide de Standard Load Balancer 
 
Créez un service Private Link à l’aide de la configuration d’adresse IP frontale du Standard Load Balancer avec la commande [az network private-link-service create](/cli/azure/network/private-link-service#az-network-private-link-service-create). Cet exemple crée un service Private Link nommé *myPLS* en utilisant le Standard Load Balancer nommé *myLoadBalancer* dans le groupe de ressources nommé *myResourceGroup*. 
 
```azurecli-interactive
az network private-link-service create \
--resource-group myResourceGroup \
--name myPLS \
--vnet-name myVirtualNetwork \
--subnet mySubnet \
--lb-name myILB \
--lb-frontend-ip-configs myFrontEnd \
--location westcentralus 
```
Une fois le service créé, prenez note de l’ID du service Private Link. Vous en aurez besoin plus tard pour demander la connexion à ce service.  
 
À ce stade, votre service Private Link est correctement créé et prêt à recevoir le trafic. Notez que l’exemple ci-dessus illustre uniquement la création d’un service Private Link à l’aide d’Azure CLI.  Nous n’avons pas configuré les pools principaux de l’équilibreur de charge ou une application sur les pools principaux pour écouter le trafic. Si vous souhaitez consulter des flux de trafic de bout en bout, nous vous conseillons vivement de configurer votre application derrière votre Standard Load Balancer.  
 
Nous allons ensuite montrer comment mapper ce service à une instance Private Endpoint dans un réseau virtuel différent à l’aide d’Azure CLI. Là encore, l’exemple est limité à la création de l’instance Private Endpoint et à la connexion au service Private Link créé ci-dessus à l’aide d’Azure CLI. En outre, vous pouvez créer des machines virtuelles dans le réseau virtuel pour envoyer/recevoir le trafic vers Private Endpoint.        
 
## <a name="private-endpoints"></a>Instances Private Endpoint

### <a name="create-the-virtual-network"></a>Créer un réseau virtuel 
Créez un réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create). Cet exemple crée un réseau virtuel nommé  *myPEVNet* dans le groupe de ressources nommé *myResourcegroup* : 
```azurecli-interactive
az network vnet create \
--resource-group myResourceGroup \
--name myPEVnet \
--address-prefix 10.0.0.0/16  
```
### <a name="create-the-subnet"></a>Créer le sous-réseau 
Ajoutez un sous-réseau au réseau virtuel avec la commande [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create). Cet exemple crée un sous-réseau nommé  *mySubnet* dans le réseau virtuel nommé *myPEVnet* dans le groupe de ressources nommé *myResourcegroup* : 

```azurecli-interactive 
az network vnet subnet create \
--resource-group myResourceGroup \
--vnet-name myPEVnet \
--name myPESubnet \
--address-prefixes 10.0.0.0/24 
```   
## <a name="disable-private-endpoint-network-policies-on-subnet"></a>Désactiver les stratégies réseau Private Endpoint sur le sous-réseau 
Vous pouvez créer une instance Private Endpoint dans n’importe quel sous-réseau de votre choix au sein d’un réseau virtuel. Actuellement, nous ne prenons pas en charge les stratégies réseau sur Private Endpoint.  Par conséquent, nous devons désactiver les stratégies réseau sur le sous-réseau. Mettez à jour le sous-réseau pour désactiver les stratégies réseau Private Endpoint avec [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update). 

```azurecli-interactive
az network vnet subnet update \
--resource-group myResourceGroup \
--vnet-name myPEVnet \
--name myPESubnet \
--disable-private-endpoint-network-policies true 
```
## <a name="create-private-endpoint-and-connect-to-private-link-service"></a>Créer une instance Private Endpoint et se connecter au service Private Link 
Créez un Private Endpoint pour l’utilisation du service Private Link créé ci-dessus dans votre réseau virtuel :
  
```azurecli-interactive
az network private-endpoint create \
--resource-group myResourceGroup \
--name myPE \
--vnet-name myPEVnet \
--subnet myPESubnet \
--private-connection-resource-id {PLS_resourceURI} \
--connection-name myPEConnectingPLS \
--location westcentralus 
```
Vous pouvez obtenir la valeur de *private-connection-resource-id* avec `az network private-link-service show` sur le service Private Link. L’ID ressemble à ce qui suit :   
/subscriptions/subID/resourceGroups/*resourcegroupname*/providers/Microsoft.Network/privateLinkServices/**privatelinkservicename** 
 
## <a name="show-private-link-service-connections"></a>Afficher les connexions au service Private Link 
 
Consultez les demandes de connexion sur votre service Private Link à l’aide de la commandeA [az network private-link-service show](/cli/azure/network/private-link-service#az-network-private-link-service-show).    
```azurecli-interactive 
az network private-link-service show --resource-group myResourceGroup --name myPLS 
```
## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur [Azure Private Link](private-link-service-overview.md)
