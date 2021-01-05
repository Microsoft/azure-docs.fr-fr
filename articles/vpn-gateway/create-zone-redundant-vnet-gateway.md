---
title: Créer une passerelle de réseau virtuel redondante interzone dans les zones de disponibilité Azure
description: Découvrez comment déployer des passerelles VPN redondantes interzone et des passerelles ExpressRoute dans les zones de disponibilité Azure.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: cherylmc
ms.openlocfilehash: 2eaf1470e2d861ecfc1c1bc96f6040a1c3e0a644
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89425226"
---
# <a name="create-a-zone-redundant-virtual-network-gateway-in-azure-availability-zones"></a>Créer une passerelle de réseau virtuel redondante interzone dans les zones de disponibilité Azure

Vous pouvez déployer des passerelles VPN et ExpressRoute dans des zones de disponibilité Azure. Cela apporte de la résilience, de l’extensibilité et une plus grande disponibilité aux passerelles de réseau virtuel. Le déploiement de passerelles dans les zones de disponibilité Azure sépare les passerelles physiquement et logiquement au sein d’une région, tout en protégeant votre connectivité de réseau local à Azure à partir d’échecs au niveau de la zone. Pour plus d’informations, voir [À propos des passerelles de réseau virtuel redondante interzone](about-zone-redundant-vnet-gateways.md) et [À propos des zones de disponibilité Azure](../availability-zones/az-overview.md).

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [powershell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

## <a name="1-declare-your-variables"></a><a name="variables"></a>1. Déclarer vos variables

Déclarez les variables que vous souhaitez utiliser. Utilisez l’exemple ci-dessous en utilisant vos propres valeurs si nécessaire. Si vous fermez votre session PowerShell/Cloud Shell au cours de l’exercice, copiez et collez les valeurs une nouvelle fois pour redéclarer les variables. Lorsque vous spécifiez un emplacement, vérifiez que la région que vous spécifiez est prise en charge. Pour plus d’informations, visitez le [FAQ](#faq).

```azurepowershell-interactive
$RG1         = "TestRG1"
$VNet1       = "VNet1"
$Location1   = "CentralUS"
$FESubnet1   = "FrontEnd"
$BESubnet1   = "Backend"
$GwSubnet1   = "GatewaySubnet"
$VNet1Prefix = "10.1.0.0/16"
$FEPrefix1   = "10.1.0.0/24"
$BEPrefix1   = "10.1.1.0/24"
$GwPrefix1   = "10.1.255.0/27"
$Gw1         = "VNet1GW"
$GwIP1       = "VNet1GWIP"
$GwIPConf1   = "gwipconf1"
```

## <a name="2-create-the-virtual-network"></a><a name="configure"></a>2. Créer un réseau virtuel

Créez un groupe de ressources.

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName $RG1 -Location $Location1
```

Créez un réseau virtuel.

```azurepowershell-interactive
$fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubnet1 -AddressPrefix $FEPrefix1
$besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubnet1 -AddressPrefix $BEPrefix1
$vnet = New-AzVirtualNetwork -Name $VNet1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNet1Prefix -Subnet $fesub1,$besub1
```

## <a name="3-add-the-gateway-subnet"></a><a name="gwsub"></a>3. Ajouter le sous-réseau de passerelle

Le sous-réseau de passerelle contient les adresses IP réservées utilisées par les services de passerelle de réseau virtuel. Appuyez-vous sur les exemples suivants pour ajouter et définir un sous-réseau de passerelle :

Ajoutez le sous-réseau de passerelle.

```azurepowershell-interactive
$getvnet = Get-AzVirtualNetwork -ResourceGroupName $RG1 -Name VNet1
Add-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.1.255.0/27 -VirtualNetwork $getvnet
```

Définissez la configuration de sous-réseau de passerelle du réseau virtuel.

```azurepowershell-interactive
$getvnet | Set-AzVirtualNetwork
```
## <a name="4-request-a-public-ip-address"></a><a name="publicip"></a>4. Demander une adresse IP publique
 
Dans cette étape, choisissez les instructions qui s’appliquent à la passerelle que vous souhaitez créer. La sélection des zones pour le déploiement des passerelles varie selon les zones spécifiées pour l’adresse IP publique.

### <a name="for-zone-redundant-gateways"></a><a name="ipzoneredundant"></a>Pour les passerelles redondantes interzone

Demandez une adresse IP publique avec une référence SKU PublicIpaddress **Standard** et ne spécifiez aucune zone. Dans ce cas, l’adresse IP publique Standard créée sera une adresse IP publique redondante interzone.   

```azurepowershell-interactive
$pip1 = New-AzPublicIpAddress -ResourceGroup $RG1 -Location $Location1 -Name $GwIP1 -AllocationMethod Static -Sku Standard
```

### <a name="for-zonal-gateways"></a><a name="ipzonalgw"></a>Pour les passerelles zonales

Demandez une adresse IP publique avec une référence SKU PublicIpaddress **Standard**. Spécifiez la zone (1, 2 ou 3). Toutes les instances de la passerelle seront déployées dans cette zone.

```azurepowershell-interactive
$pip1 = New-AzPublicIpAddress -ResourceGroup $RG1 -Location $Location1 -Name $GwIP1 -AllocationMethod Static -Sku Standard -Zone 1
```

### <a name="for-regional-gateways"></a><a name="ipregionalgw"></a>Pour les passerelles régionales

Demandez une adresse IP publique avec une référence SKU PublicIpaddress **De base**. Dans ce cas, la passerelle est déployée comme une passerelle régionale et n’a pas de redondance de zone intégrée. Les instances de passerelle sont créées dans des zones, respectivement.

```azurepowershell-interactive
$pip1 = New-AzPublicIpAddress -ResourceGroup $RG1 -Location $Location1 -Name $GwIP1 -AllocationMethod Dynamic -Sku Basic
```
## <a name="5-create-the-ip-configuration"></a><a name="gwipconfig"></a>5. Créer la configuration de l’adresse IP

```azurepowershell-interactive
$getvnet = Get-AzVirtualNetwork -ResourceGroupName $RG1 -Name $VNet1
$subnet = Get-AzVirtualNetworkSubnetConfig -Name $GwSubnet1 -VirtualNetwork $getvnet
$gwipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GwIPConf1 -Subnet $subnet -PublicIpAddress $pip1
```

## <a name="6-create-the-gateway"></a><a name="gwconfig"></a>6. Créer la passerelle

Créez la passerelle de réseau virtuel.

### <a name="for-expressroute"></a>Pour la passerelle ExpressRoute

```azurepowershell-interactive
New-AzVirtualNetworkGateway -ResourceGroup $RG1 -Location $Location1 -Name $Gw1 -IpConfigurations $GwIPConf1 -GatewayType ExpressRoute -GatewaySku ErGw1AZ
```

### <a name="for-vpn-gateway"></a>Pour la passerelle VPN

```azurepowershell-interactive
New-AzVirtualNetworkGateway -ResourceGroup $RG1 -Location $Location1 -Name $Gw1 -IpConfigurations $GwIPConf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1AZ
```

## <a name="faq"></a><a name="faq"></a>Forum Aux Questions

### <a name="what-will-change-when-i-deploy-these-new-skus"></a>Qu’est-ce qui va changer lorsque je déploierai ces nouvelles références SKU ?

De votre point de vue, vous pouvez déployer vos passerelles avec la redondance dans une zone. Cela signifie que toutes les instances des passerelles seront déployées sur des zones de disponibilité Azure, et que chaque zone de disponibilité est un autre domaine d’erreur et de mise à jour. Cela rend vos passerelles plus fiables, disponibles et résilientes aux défaillances de zone.

### <a name="can-i-use-the-azure-portal"></a>Puis-je utiliser le portail Azure ?

Oui, vous pouvez utiliser le Portail Azure pour déployer les nouvelles références SKU. Toutefois, vous ne les verrez que dans les régions Azure disposant de Zones de disponibilité Azure.

### <a name="what-regions-are-available-for-me-to-use-the-new-skus"></a>Dans quelles régions puis-je utiliser les nouvelles références SKU ?

Consultez [Zones de disponibilité](../availability-zones/az-region.md) pour obtenir la dernière liste des régions disponibles.

### <a name="can-i-changemigrateupgrade-my-existing-virtual-network-gateways-to-zone-redundant-or-zonal-gateways"></a>Puis-je changer/migrer/mettre à niveau mes passerelles de réseau virtuel existantes vers des passerelles zonales ou redondantes interzone ?

La migration de vos passerelles de réseau virtuel existantes vers des passerelles zonales ou redondantes interzone n’est pas prise en charge pour le moment. Toutefois, vous pouvez supprimer votre passerelle existante et recréer une passerelle zonale ou redondante interzone.

### <a name="can-i-deploy-both-vpn-and-express-route-gateways-in-same-virtual-network"></a>Puis-je déployer des passerelles VPN et ExpressRoute dans le même réseau virtuel ?

La coexistence de passerelles VPN et ExpressRoute dans le même réseau virtuel est prise en charge. Toutefois, vous devez réserver une plage d’adresses IP de /27 pour le sous-réseau de passerelle.
