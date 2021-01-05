---
title: À propos des passerelles de réseau virtuel redondantes interzone dans les Zones de disponibilité Azure
description: Déployez des passerelles VPN et ExpressRoute dans Zones de disponibilité Azure pour offrir une résilience, une scalabilité et une plus grande disponibilité aux passerelles de réseau virtuel.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
Customer intent: As someone with a basic network background, I want to understand zone-redundant gateways.
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: 2030469262baf406635fd170af384e154fec6ae6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89401110"
---
# <a name="about-zone-redundant-virtual-network-gateways-in-azure-availability-zones"></a>À propos des passerelles de réseau virtuel redondantes interzone dans les Zones de disponibilité Azure

Vous pouvez déployer des passerelles VPN et ExpressRoute dans des [zones de disponibilité Azure](../availability-zones/az-overview.md). Cela apporte de la résilience, de l’extensibilité et une plus grande disponibilité aux passerelles de réseau virtuel. Le déploiement de passerelles dans les zones de disponibilité Azure sépare les passerelles physiquement et logiquement au sein d’une région, tout en protégeant votre connectivité de réseau local à Azure à partir d’échecs au niveau de la zone.

### <a name="zone-redundant-gateways"></a><a name="zrgw"></a>Passerelles redondantes interzone

Pour déployer automatiquement vos passerelles de réseau virtuel entre des zones de disponibilité, vous pouvez utiliser des passerelles de réseau virtuel redondantes interzone. Avec les passerelles redondantes interzone, vous pouvez bénéficier de la résilience aux zones pour accéder à vos services évolutifs et critiques sur Azure.

<br>
<br>

![graphique des passerelles redondantes dans une zone](./media/create-zone-redundant-vnet-gateway/zonered.png)

### <a name="zonal-gateways"></a><a name="zgw"></a>Passerelles zonales

Pour le déploiement de passerelles dans une zone spécifique, vous pouvez utiliser des passerelles zonales. Lorsque vous déployez une passerelle zonale, toutes les instances de la passerelle sont déployées dans la même Zone de disponibilité.

<br>
<br>

![graphique sur les passerelles zonales](./media/create-zone-redundant-vnet-gateway/zonal.png)

## <a name="gateway-skus"></a><a name="gwskus"></a>SKU de passerelle

Les passerelles zonales et redondantes interzone sont disponibles en tant que nouvelles références SKU de passerelle. Nous avons ajouté de nouvelles références SKU de passerelle de réseau virtuel dans les régions Azure AZ. Ces références SKU sont semblables aux références SKU correspondantes qui existent déjà pour les passerelles VPN et ExpressRoute, sauf qu’elles sont spécifiques aux passerelles zonales et redondantes interzone. Vous pouvez identifier ces références SKU par le « AZ » dans leur nom.

Pour plus d’informations sur les références SKU de passerelle, consultez [SKU de passerelle VPN](vpn-gateway-about-vpngateways.md#gwsku) et [SKU de passerelle ExpressRoute](../expressroute/expressroute-about-virtual-network-gateways.md#gwsku).

## <a name="public-ip-skus"></a><a name="pipskus"></a>Références SKU d’adresse IP publique

Les passerelles zonales et redondantes interzone s’appuient sur la référence (SKU) de ressource d’adresse IP publique *Standard*. La configuration de la ressource d’adresse IP publique Azure détermine si la passerelle que vous déployez est zonale ou redondante interzone. Si vous créez une ressource d’adresse IP publique avec une référence (SKU) *De base*, la passerelle n’aura aucune redondance de zone, et les ressources de la passerelle seront régionales.

### <a name="zone-redundant-gateways"></a><a name="pipzrg"></a>Passerelles redondantes interzone

Lorsque vous créez une adresse IP publique à l’aide de la référence SKU d’adresse IP publique **Standard** sans spécifier de zone, le comportement diffère selon que la passerelle est une passerelle VPN ou une passerelle ExpressRoute. 

* Pour une passerelle VPN, les deux instances de passerelle seront déployées dans 2 de ces zones, parmi les trois zones assurant la redondance de zone. 
* Pour une passerelle ExpressRoute, dans la mesure où il peut y avoir plus de deux instances, la passerelle peut s’étendre sur les trois zones.

### <a name="zonal-gateways"></a><a name="pipzg"></a>Passerelles zonales

Lorsque vous créez une adresse IP publique à l’aide de la référence SKU d’adresse IP **Standard** et spécifiez la zone (1, 2 ou 3), toutes les instances de la passerelle sont déployées dans la même zone.

### <a name="regional-gateways"></a><a name="piprg"></a>Passerelles régionales

Lorsque vous créez une adresse IP publique à l’aide de la référence SKU d’adresse IP publique **De base**, la passerelle est déployée comme une passerelle régionale et n’a pas de redondance de zone intégrée.

## <a name="faq"></a><a name="faq"></a>Forum Aux Questions

### <a name="what-will-change-when-i-deploy-these-new-skus"></a>Qu’est-ce qui va changer lorsque je déploierai ces nouvelles références SKU ?

De votre point de vue, vous pouvez déployer vos passerelles avec la redondance dans une zone. Cela signifie que toutes les instances des passerelles seront déployées sur des zones de disponibilité Azure, et que chaque zone de disponibilité est un autre domaine d’erreur et de mise à jour. Cela rend vos passerelles plus fiables, disponibles et résilientes aux défaillances de zone.

### <a name="can-i-use-the-azure-portal"></a>Puis-je utiliser le portail Azure ?

Oui, vous pouvez utiliser le Portail Azure pour déployer les nouvelles références SKU. Toutefois, vous ne les verrez que dans les régions Azure disposant de Zones de disponibilité Azure.

### <a name="what-regions-are-available-for-me-to-use-the-new-skus"></a>Dans quelles régions puis-je utiliser les nouvelles références SKU ?

Les nouvelles références SKU sont disponibles dans les régions Azure qui possèdent les régions Zones de disponibilité Azure – USA Centre, France Centre, Europe Nord, Europe Ouest et USA Ouest 2, USA Est, USA Est 2, Asie Sud-Est, Japon Est, Royaume-Uni Sud. Prochainement, des passerelles redondantes interzone seront disponibles dans d’autres régions Azure publiques.

### <a name="can-i-changemigrateupgrade-my-existing-virtual-network-gateways-to-zone-redundant-or-zonal-gateways"></a>Puis-je changer/migrer/mettre à niveau mes passerelles de réseau virtuel existantes vers des passerelles zonales ou redondantes interzone ?

La migration de vos passerelles de réseau virtuel existantes vers des passerelles zonales ou redondantes interzone n’est pas prise en charge pour le moment. Toutefois, vous pouvez supprimer votre passerelle existante et recréer une passerelle zonale ou redondante interzone.

### <a name="can-i-deploy-both-vpn-and-express-route-gateways-in-same-virtual-network"></a>Puis-je déployer des passerelles VPN et ExpressRoute dans le même réseau virtuel ?

La coexistence de passerelles VPN et ExpressRoute dans le même réseau virtuel est prise en charge. Toutefois, vous devez réserver une plage d’adresses IP de /27 pour le sous-réseau de passerelle.

## <a name="next-steps"></a>Étapes suivantes

[Créer une passerelle de réseau virtuel redondante interzone](create-zone-redundant-vnet-gateway.md)
