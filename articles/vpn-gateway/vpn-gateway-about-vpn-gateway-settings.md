---
title: 'Passerelle VPN Azure : paramètres de configuration'
description: En savoir plus sur les ressources et les paramètres de la passerelle VPN pour un réseau virtuel créé dans le modèle de déploiement Resource Manager.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 10/21/2020
ms.author: cherylmc
ms.openlocfilehash: 1aba87b2139fb8a7d395fb3180d2074e47310fa9
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010818"
---
# <a name="about-vpn-gateway-configuration-settings"></a>À propos des paramètres de configuration de la passerelle VPN

Une passerelle VPN est un type de passerelle de réseau virtuel qui envoie le trafic chiffré entre votre réseau virtuel et votre emplacement local à travers une connexion publique. Vous pouvez également utiliser une passerelle VPN pour acheminer le trafic entre réseaux virtuels sur l’infrastructure Azure.

Une connexion de passerelle VPN s’appuie sur la configuration de plusieurs ressources, contenant chacune des paramètres configurables. Les sections de cet article présentent les ressources et les paramètres relatifs à une passerelle VPN pour un réseau virtuel créé dans le modèle de déploiement Resource Manager. Vous trouverez les descriptions et les diagrammes de topologie de chaque solution de connexion dans l’article [À propos la passerelle VPN](vpn-gateway-about-vpngateways.md).

Les valeurs figurant dans cet article s’appliquent à des passerelles VPN (passerelles de réseau virtuel) qui utilisent le -GatewayType Vpn. Cet article ne couvre pas l’ensemble des types de passerelles, ni les passerelles redondantes interzone.

* Pour les valeurs qui s’appliquent au -GatewayType 'ExpressRoute', consultez [Passerelles de réseau virtuel pour ExpressRoute](../expressroute/expressroute-about-virtual-network-gateways.md).

* Pour les passerelles redondantes interzone, consultez l’article [À propos des passerelles redondantes pour les Zones de disponibilité](about-zone-redundant-vnet-gateways.md).

* Pour le service Virtual WAN, consultez l’article [À propos du WAN virtuel](../virtual-wan/virtual-wan-about.md).

## <a name="gateway-types"></a><a name="gwtype"></a>Types de passerelle

Chaque réseau virtuel ne peut posséder qu’une seule passerelle de réseau virtuel de chaque type. Lorsque vous créez une passerelle de réseau virtuel, vous devez vous assurer que le type de passerelle convient pour votre configuration.

Les valeurs disponibles pour -GatewayType sont :

* Vpn
* ExpressRoute

Une passerelle VPN nécessite le `-GatewayType` *VPN*.

Exemple :

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
-Location 'West US' -IpConfigurations $gwipconfig -GatewayType Vpn `
-VpnType RouteBased
```

## <a name="gateway-skus"></a><a name="gwsku"></a>SKU de passerelle

[!INCLUDE [vpn-gateway-gwsku-include](../../includes/vpn-gateway-gwsku-include.md)]

### <a name="configure-a-gateway-sku"></a>Configurer une référence SKU de passerelle

**Azure portal**

Si vous utilisez le portail Azure pour créer une passerelle de réseau virtuel Resource Manager, vous pouvez sélectionner la référence SKU de la passerelle dans la liste déroulante. Les options qui vous sont présentées correspondent au type de passerelle et au type de VPN sélectionnés.

**PowerShell**

L’exemple PowerShell suivant spécifie la `-GatewaySku` en tant que VpnGw1. Lorsque vous utilisez PowerShell pour créer une passerelle, vous devez d’abord créer la configuration IP, puis utiliser une variable qui y fait référence. Dans cet exemple, la variable de configuration est $gwipconfig.

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name VNet1GW -ResourceGroupName TestRG1 `
-Location 'US East' -IpConfigurations $gwipconfig -GatewaySku VpnGw1 `
-GatewayType Vpn -VpnType RouteBased
```

**Azure CLI**

```azurecli
az network vnet-gateway create --name VNet1GW --public-ip-address VNet1GWPIP --resource-group TestRG1 --vnet VNet1 --gateway-type Vpn --vpn-type RouteBased --sku VpnGw1 --no-wait
```

###  <a name="resizing-or-changing-a-sku"></a><a name="resizechange"></a>Redimensionnement ou modification d’une référence SKU

Si vous disposez d’une passerelle VPN et que vous souhaitez utiliser une autre référence SKU de passerelle, vous avez le choix entre redimensionner votre référence SKU de passerelle ou remplacer par une autre référence SKU. Lorsque vous remplacez par une autre référence SKU de passerelle, vous supprimez la passerelle existante dans son ensemble et en créez une nouvelle. La création d’une passerelle peut prendre jusqu’à 45 minutes. En comparaison, lorsque vous redimensionnez une référence SKU de passerelle, le temps d’arrêt sera faible, car vous n’aurez pas à supprimer et à recréer la passerelle. Si vous avez la possibilité de redimensionner votre référence SKU de passerelle plutôt que de la modifier, vous devez le faire. Toutefois, il existe des règles concernant le redimensionnement :

1. À l’exception de la référence SKU de base, vous pouvez redimensionner une référence SKU de passerelle VPN vers une autre référence SKU de passerelle VPN au sein de la même génération (Generation1 ou Generation2). Par exemple, VpnGw1 de Generation1 peut être redimensionné en VpnGw2 de Generation1, mais pas en VpnGw2 de Generation2.
2. Lors de l’utilisation des anciennes références SKU de passerelle, vous pouvez effectuer un redimensionnement entre les références SKU Basic, Standard et HighPerformance.
3. Vous **ne pouvez pas** effectuer de redimensionnement pour passer des références SKU de base/standard/haute performance à des références SKU VpnGw. Au lieu de cela, vous devez [effectuer la modification](#change) vers les nouvelles références SKU.

#### <a name="to-resize-a-gateway"></a><a name="resizegwsku"></a>Pour redimensionner une passerelle

**Azure portal**

[!INCLUDE [Resize a SKU - portal](../../includes/vpn-gateway-resize-gw-portal-include.md)]

**PowerShell**

[!INCLUDE [Resize a SKU](../../includes/vpn-gateway-gwsku-resize-include.md)]

####  <a name="to-change-from-an-old-legacy-sku-to-a-new-sku"></a><a name="change"></a>Pour passer d’un ancienne référence SKU (héritée) à une nouvelle référence SKU

[!INCLUDE [Change a SKU](../../includes/vpn-gateway-gwsku-change-legacy-sku-include.md)]

## <a name="connection-types"></a><a name="connectiontype"></a>Types de connexion

Dans le modèle de déploiement de Resource Manager, chaque configuration nécessite un type spécifique de connexion de passerelle de réseau virtuel. Les valeurs de PowerShell pour Resource Manager disponibles pour `-ConnectionType` sont :

* IPsec
* Vnet2Vnet
* ExpressRoute
* VPNClient

Dans l’exemple PowerShell suivant, nous créons une connexion S2S qui nécessite le type de connexion *IPsec*.

```azurepowershell-interactive
New-AzVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg `
-Location 'West US' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
-ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'
```

## <a name="vpn-types"></a><a name="vpntype"></a>Types de VPN

Lorsque vous créez la passerelle de réseau virtuel d’une configuration de passerelle VPN, vous devez spécifier un type de VPN. Le type de VPN que vous choisissez dépend de la topologie de connexion que vous souhaitez créer. Par exemple, une connexion P2S nécessite un VPN de type basé sur un itinéraire. Un type VPN peut également dépendre du matériel utilisé. Les configurations S2S nécessitent un périphérique VPN. Certains périphériques VPN seront ne prennent en charge qu’un certain type de VPN.

Le type de VPN que vous choisissez doit satisfaire à toutes les exigences de connexion de la solution que vous souhaitez créer. Par exemple, si vous souhaitez créer une connexion de passerelle VPN S2S et une connexion de passerelle VPN P2S pour le même réseau virtuel, vous utiliserez un VPN de type *basé sur un itinéraire* car P2S requiert un VPN de type basé sur un itinéraire. Vous devez également vérifier que votre périphérique VPN prend en charge une connexion VPN basée sur un itinéraire. 

Une fois qu’une passerelle de réseau virtuel a été créée, vous ne pouvez plus modifier le type de VPN. Vous devez supprimer la passerelle de réseau virtuel et en créer une nouvelle. Il existe deux types de VPN :

[!INCLUDE [vpn-gateway-vpntype](../../includes/vpn-gateway-vpntype-include.md)]

L’exemple PowerShell suivant spécifie la `-VpnType` en tant que *RouteBased*. Lorsque vous créez une passerelle, vous devez vous assurer que -VpnType convient pour votre configuration.

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
-Location 'West US' -IpConfigurations $gwipconfig `
-GatewayType Vpn -VpnType RouteBased
```

## <a name="gateway-requirements"></a><a name="requirements"></a>Conditions requises de la passerelle

[!INCLUDE [vpn-gateway-table-requirements](../../includes/vpn-gateway-table-requirements-include.md)]

## <a name="gateway-subnet"></a><a name="gwsub"></a>Sous-réseau de passerelle

Avant de créer votre passerelle VPN, vous devez d’abord créer un sous-réseau de passerelle. Le sous-réseau de passerelle contient les adresses IP utilisées par les machines virtuelles et les services de passerelle de réseau virtuel. Lors de la création de votre passerelle de réseau virtuel, les machines virtuelles de passerelle sont déployées dans le sous-réseau de passerelle et configurées avec les paramètres de passerelle VPN requis. Ne déployez jamais d’autres éléments (des machines virtuelles supplémentaires, par exemple) dans le sous-réseau de passerelle. Pour fonctionner correctement, le sous-réseau de passerelle doit être nommé ’GatewaySubnet’. En nommant le sous-réseau de passerelle « GatewaySubnet », Azure est informé qu'il s’agit du sous-réseau dans lequel déployer les machines virtuelles et les services de passerelle de réseau virtuel.

>[!NOTE]
>[!INCLUDE [vpn-gateway-gwudr-warning.md](../../includes/vpn-gateway-gwudr-warning.md)]
>

Lorsque vous créez le sous-réseau de passerelle, vous spécifiez le nombre d’adresses IP que contient le sous-réseau. Les adresses IP dans le sous-réseau de passerelle sont allouées aux machines virtuelles et aux services de passerelle. Certaines configurations nécessitent plus d’adresses IP que d’autres. 

Pour planifier la taille de votre sous-réseau de passerelle, consultez la documentation correspondant à la configuration que vous envisagez de créer. Par exemple, la configuration de coexistence ExpressRoute/passerelle VPN nécessite un sous-réseau de passerelle plus important que la plupart des autres configurations. De plus, assurez-vous que votre sous-réseau de passerelle contient suffisamment d’adresses IP pour pouvoir prendre en charge d’éventuelles nouvelles configurations. Même si vous pouvez créer un petit sous-réseau de passerelle (jusqu’à une taille /29), nous vous recommandons d’en créer un de taille /27 ou supérieure (/27, /26, etc.) si l’espace d’adressage disponible est suffisant. Ceci conviendra à la plupart des configurations.

L’exemple PowerShell Resource Manager suivant montre un sous-réseau de passerelle nommé GatewaySubnet. Vous pouvez voir que la notation CIDR spécifie une taille /27, ce qui permet d’avoir un nombre suffisamment élevé d’adresses IP pour la plupart des configurations actuelles.

```azurepowershell-interactive
Add-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/27
```

[!INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="local-network-gateways"></a><a name="lng"></a>Passerelles de réseau local

Une passerelle de réseau local diffère d'une passerelle de réseau virtuel. Lorsque vous créez une configuration de passerelle VPN, la passerelle du réseau local représente généralement votre réseau et l’appareil VPN correspondant. Dans le modèle de déploiement classique, la passerelle de réseau local a été appelée Site local.

Vous donnez un nom ou nom de domaine complet à la passerelle de réseau local (l’adresse IP publique de l’appareil VPN local) et spécifiez les préfixes d’adresse qui se situent dans l’emplacement local. Azure examine les préfixes d’adresse de destination pour le trafic réseau, consulte la configuration que vous avez spécifiée pour votre passerelle de réseau local, et route les paquets en conséquence. Si vous utilisez Border Gateway Protocol (BGP) sur votre périphérique VPN, vous devez fournir l’adresse IP du pair BGP de votre périphérique VPN et le numéro de système autonome (ASN) de votre réseau local. Vous spécifiez également des passerelles de réseau local pour les configurations avec interconnexion de réseaux virtuels qui utilisent une connexion de passerelle VPN.

L’exemple PowerShell suivant crée une nouvelle passerelle de réseau local :

```azurepowershell-interactive
New-AzLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg `
-Location 'West US' -GatewayIpAddress '23.99.221.164' -AddressPrefix '10.5.51.0/24'
```

Parfois, vous devez modifier les paramètres de passerelle de réseau local. C’est le cas, par exemple, lorsque vous ajoutez ou modifiez la plage d’adresses, ou lorsque l’adresse IP du périphérique VPN change. Voir [Modification des paramètres de passerelle de réseau local à l’aide de PowerShell](vpn-gateway-modify-local-network-gateway.md).

## <a name="rest-apis-powershell-cmdlets-and-cli"></a><a name="resources"></a>API REST, cmdlets PowerShell et CLI

Pour accéder à des ressources techniques supplémentaires et connaître les exigences spécifiques en matière de syntaxe lors de l’utilisation d’API REST, d’applets de commande PowerShell ou d’Azure CLI pour les configurations de passerelles VPN, consultez les pages suivantes :

| **Classique** | **Resource Manager** |
| --- | --- |
| [PowerShell](/powershell/module/az.network/#networking) |[PowerShell](/powershell/module/az.network#vpn) |
| [REST API](/previous-versions/azure/reference/jj154113(v=azure.100)) |[REST API](/rest/api/network/virtualnetworkgateways) |
| Non pris en charge | [Azure CLI](/cli/azure/network/vnet-gateway)|

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les configurations de connexion disponible, consultez la rubrique [À propos de la passerelle VPN](vpn-gateway-about-vpngateways.md).