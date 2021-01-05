---
title: Ajouter le protocole IPv6 à une application IPv4 dans un réseau virtuel Azure – Azure CLI
titlesuffix: Azure Virtual Network
description: Cet article explique comment déployer des adresses IPv6 dans une application existante dans un réseau virtuel Azure avec Azure CLI.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: de8d63fb5181e324738e082cbb36a40ee59e36b0
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "96005246"
---
# <a name="add-ipv6-to-an-ipv4-application-in-azure-virtual-network---azure-cli"></a>Ajouter le protocole IPv6 à une application IPv4 dans un réseau virtuel Azure – Azure CLI

Cet article montre comment ajouter des adresses IPv6 à une application qui utilise une adresse IP publique IPv4 dans un réseau virtuel Azure pour un équilibreur de charge Standard Load Balancer avec Azure CLI. La mise à niveau sur place comprend un réseau virtuel et un sous-réseau, un équilibreur de charge standard avec des configurations front-end IPv4 + IPv6, des machines virtuelles avec des cartes réseau qui ont des configurations IPv4 + IPv6, un groupe de sécurité réseau et des adresses IP publiques.

## <a name="prerequisites"></a>Prérequis

- Cet article suppose que vous avez déployé un équilibreur de charge standard comme cela est décrit dans [Démarrage rapide : Créer un équilibreur Standard Load Balancer – Azure CLI](../load-balancer/quickstart-load-balancer-standard-public-cli.md).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Cet article nécessite la version 2.0.28 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-ipv6-addresses"></a>Créer des adresses IPv6

Créez des adresses IPv6 publiques avec [az network public-ip create](/cli/azure/network/public-ip) pour votre équilibreur Standard Load Balancer. L’exemple suivant crée une adresse IP publique IPv6 nommée *PublicIP_v6* dans le groupe de ressources *myResourceGroupSLB* :

```azurecli-interactive
az network public-ip create \
--name PublicIP_v6 \
--resource-group MyResourceGroupSLB \
--location EastUS \
--sku Standard \
--allocation-method static \
--version IPv6
```

## <a name="configure-ipv6-load-balancer-frontend"></a>Configurer l’équilibreur de charge IPv6 front-end

Configurez l’équilibreur de charge avec la nouvelle adresse IP IPv6 avec [az network lb frontend-ip create](https://docs.microsoft.com/cli/azure/network/lb/frontend-ip?view=azure-cli-latest#az-network-lb-frontend-ip-create) :

```azurecli-interactive
az network lb frontend-ip create \
--lb-name myLoadBalancer \
--name dsLbFrontEnd_v6 \
--resource-group MyResourceGroupSLB \
--public-ip-address PublicIP_v6
```

## <a name="configure-ipv6-load-balancer-backend-pool"></a>Configurer le pool back-end de l’équilibreur de charge IPv6

Créez le pool back-end pour les cartes réseau comportant des adresses IPv6 avec [az network lb address-pool create](https://docs.microsoft.com/cli/azure/network/lb/address-pool?view=azure-cli-latest#az-network-lb-address-pool-create) :

```azurecli-interactive
az network lb address-pool create \
--lb-name myLoadBalancer \
--name dsLbBackEndPool_v6 \
--resource-group MyResourceGroupSLB
```

## <a name="configure-ipv6-load-balancer-rules"></a>Configurer les règles d’équilibrage de charge IPv6

Créez des règles d’équilibrage de charge IPv6 avec [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule?view=azure-cli-latest#az-network-lb-rule-create).

```azurecli-interactive
az network lb rule create \
--lb-name myLoadBalancer \
--name dsLBrule_v6 \
--resource-group MyResourceGroupSLB \
--frontend-ip-name dsLbFrontEnd_v6 \
--protocol Tcp \
--frontend-port 80 \
--backend-port 80 \
--backend-pool-name dsLbBackEndPool_v6
```

## <a name="add-ipv6-address-ranges"></a>Ajouter des plages d’adresses IPv6

Ajoutez des plages d’adresses IPv6 au réseau virtuel et au sous-réseau hébergeant l’équilibreur de charge, comme suit :

```azurecli-interactive
az network vnet update \
--name myVnet  \
--resource-group MyResourceGroupSLB \
--address-prefixes  "10.0.0.0/16"  "fd00:db8:deca::/48"

az network vnet subnet update \
--vnet-name myVnet \
--name mySubnet \
--resource-group MyResourceGroupSLB \
--address-prefixes  "10.0.0.0/24"  "fd00:db8:deca:deed::/64"  
```

## <a name="add-ipv6-configuration-to-nics"></a>Ajouter la configuration IPv6 à des cartes réseau

Configurez les cartes réseau de machines virtuelles comportant une adresse IPv6 avec [az network nic ip-config create](https://docs.microsoft.com/cli/azure/network/nic/ip-config?view=azure-cli-latest#az-network-nic-ip-config-create) :

```azurecli-interactive
az network nic ip-config create \
--name dsIp6Config_NIC1 \
--nic-name myNicVM1 \
--resource-group MyResourceGroupSLB \
--vnet-name myVnet \
--subnet mySubnet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name dsLB

az network nic ip-config create \
--name dsIp6Config_NIC2 \
--nic-name myNicVM2 \
--resource-group MyResourceGroupSLB \
--vnet-name myVnet \
--subnet mySubnet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name myLoadBalancer

az network nic ip-config create \
--name dsIp6Config_NIC3 \
--nic-name myNicVM3 \
--resource-group MyResourceGroupSLB \
--vnet-name myVnet \
--subnet mySubnet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name myLoadBalancer
```

## <a name="view-ipv6-dual-stack-virtual-network-in-azure-portal"></a>Afficher le réseau virtuel double pile IPv6 dans le Portail Microsoft Azure

Vous pouvez afficher le réseau virtuel double pile IPv6 dans le Portail Microsoft Azure en procédant comme suit :
1. Dans la barre de recherche du portail, entrez *myVnet*.
2. Quand le nom **myVnet** apparaît dans les résultats de la recherche, sélectionnez-le. Cela permet d’afficher la page **Vue d’ensemble** du réseau virtuel double pile nommé *myVNet*. Le réseau virtuel double pile montre les trois cartes réseau, avec des configurations IPv4 et IPv6 situées dans le sous-réseau double pile nommé *mySubnet*.

  ![Réseau virtuel double pile IPv6 dans Azure](./media/ipv6-add-to-existing-vnet-powershell/ipv6-dual-stack-vnet.png)


## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez utiliser la commande [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées.

```azurecli-interactive
az group delete --name MyAzureResourceGroupSLB
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez mis à jour un équilibreur de charge standard existant avec une configuration d’adresse IP front-end IPv4 en une configuration double pile (IPv4 et IPv6). Vous avez également ajouté des configurations IPv6 aux cartes réseau des machines virtuelles du pool back-end. Pour en savoir plus sur la prise en charge du protocole IPv6 dans les réseaux virtuels Azure, voir [What is IPv6 for Azure Virtual Network?](ipv6-overview.md) (Rôle d’iPv6 pour un réseau virtuel Azure).
