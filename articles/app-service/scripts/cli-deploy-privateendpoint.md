---
title: 'Interface CLI : Déployer un point de terminaison privé pour une application web avec Azure CLI'
description: Découvrez comment utiliser Azure CLI pour déployer un point de terminaison privé pour votre application web
author: ericgre
ms.assetid: a56faf72-7237-41e7-85ce-da8346f2bcaa
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/06/2020
ms.author: ericg
ms.service: app-service
ms.workload: web
ms.openlocfilehash: 21f937d95c6cd0dafa27daae50d7a74648af7bdc
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97006104"
---
# <a name="create-an-app-service-app-and-deploy-private-endpoint-using-azure-cli"></a>Créer une application App Service et déployer un point de terminaison privé à l’aide d’Azure CLI

Cet exemple de script crée une application dans App Service avec les ressources associées, puis déploie un point de terminaison privé.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Ce tutoriel nécessite l’interface Azure CLI version 2.0.28 ou ultérieure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Avant de pouvoir créer des ressources, vous devez créer un groupe de ressources pour héberger l’application web, le réseau virtuel et tout autre composant réseau. Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). Cet exemple crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *francecentral* :

```azurecli-interactive
az group create --name myResourceGroup --location francecentral 
```

## <a name="create-an-app-service-plan"></a>Créer un plan App Service

Vous devez créer un plan App Service pour héberger votre application web.
Créez un plan App Service avec la commande [az appservice plan create](/cli/azure/appservice/plan#az-appservice-plan-create).
Cet exemple crée un plan App Service nommé *myAppServicePlan* à l’emplacement *francecentral*, avec la référence SKU *P1V2* et un seul Worker : 

```azurecli-interactive
az appservice plan create \
--name myAppServicePlan \
--resource-group myResourceGroup \
--location francecentral \
--sku P1V2 \
--number-of-workers 1
```

## <a name="create-a-web-app"></a>Créer une application web

Maintenant que vous disposez d’un plan App Service, vous pouvez déployer une application web.
Créez une application web avec [az appservice plan create](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create.
Cet exemple crée une application web nommée *mySiteName* dans le plan nommé *myAppServicePlan*.

```azurecli-interactive
az webapp create \
--name mySiteName \
--resource-group myResourceGroup \
--plan myAppServicePlan
```

## <a name="create-a-vnet"></a>Créer un réseau virtuel

Créez un réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet). Cet exemple crée un réseau virtuel par défaut nommé *myVNet* avec un sous-réseau nommé *mySubnet* :

```azurecli-interactive
az network vnet create \
--name myVNet \
--resource-group myResourceGroup \
--location francecentral \
--address-prefixes 10.8.0.0/16 \
--subnet-name mySubnet \
--subnet-prefixes 10.8.100.0/24
```

## <a name="configure-the-subnet"></a>Configurer le sous-réseau 

Vous devez mettre à jour le sous-réseau pour désactiver les stratégies réseau du point de terminaison privé. Mettez à jour une configuration de sous-réseau nommée *mySubnet* avec [az network vnet subnet update](https://docs.microsoft.com/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) :

```azurecli-interactive
az network vnet subnet update \
--name mySubnet \
--resource-group myResourceGroup \
--vnet-name myVNet \
--disable-private-endpoint-network-policies true
```

## <a name="create-the-private-endpoint"></a>Créer l’instance Private Endpoint

Créez le point de terminaison privé de votre application web avec [az network private-endpoint create](/cli/azure/network/private-endpoint). Cet exemple crée un point de terminaison privé nommé *myPrivateEndpoint* dans le réseau virtuel *myVNet* et le sous-réseau *mySubnet*, avec une connexion nommée *myConnectionName* établie avec l’ID de ressource de l’application web /subscriptions/SubscriptionID/resourceGroups/myResourceGroup/providers/Microsoft.Web/sites/myWebApp. Le paramètre de groupe est *sites* pour le type d’application Web. 

```azurecli-interactive
az network private-endpoint create \
--name myPrivateEndpoint \
--resource-group myResourceGroup \
--vnet-name myVNet \
--subnet mySubnet \
--connection-name myConnectionName \
--private-connection-resource-id /subscriptions/SubscriptionID/resourceGroups/myResourceGroup/providers/Microsoft.Web/sites/myWebApp \
--group-id sites
```

## <a name="configure-the-private-zone"></a>Configurer la zone privée

À la fin, vous devez créer une zone DNS privée nommée *privatelink.azurewebsites.net* et la lier au réseau virtuel pour résoudre le nom DNS de l’application web.


```azurecli-interactive
az network private-dns zone create \
--name privatelink.azurewebsites.net \
--resource-group myResourceGroup

az network private-dns link vnet create \
--name myDNSLink \
--resource-group myResourceGroup \
--registration-enabled false \
--virtual-network myVNet \
--zone-name privatelink.azurewebsites.net

az network private-endpoint dns-zone-group create \
--name myZoneGroup \
--resource-group myResourceGroup \
--endpoint-name myPrivateEndpoint \
--private-dns-zone privatelink.azurewebsites.net \
--zone-name privatelink.azurewebsites.net
```






[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]


## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](/cli/azure).
- Vous trouverez des exemples supplémentaires de scripts CLI App Service dans la [documentation relative à Azure App Service](../samples-cli.md).
