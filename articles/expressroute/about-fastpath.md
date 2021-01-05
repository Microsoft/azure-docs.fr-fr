---
title: À propos d’Azure ExpressRoute FastPath
description: Découvrir Azure ExpressRoute FastPath pour envoyer le trafic en ignorant la passerelle réseau
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 03/25/2020
ms.author: duau
ms.openlocfilehash: 5c44e6c6a7a9af4edc2c711b17ec50aa2bf62c98
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92202445"
---
# <a name="about-expressroute-fastpath"></a>À propos d’ExpressRoute FastPath

La passerelle de réseau virtuel ExpressRoute est conçue pour échanger des routages réseau et acheminer le trafic du réseau. FastPath est conçu pour améliorer les performances du chemin d’accès aux données entre votre réseau local et votre réseau virtuel. Lorsqu’il est activé, FastPath envoie le trafic réseau directement vers les machines virtuelles du réseau virtuel, en contournant la passerelle.

## <a name="requirements"></a>Spécifications

### <a name="circuits"></a>Circuits

FastPath est disponible sur tous les circuits ExpressRoute.

### <a name="gateways"></a>Passerelles

FastPath nécessite toujours la création d’une passerelle réseau virtuelle pour échanger des routages entre le réseau virtuel et le réseau local. Pour plus d’informations sur les passerelles de réseau virtuel et ExpressRoute, y compris les informations de performances et les références SKU de passerelle, consultez [Passerelles de réseau virtuel ExpressRoute](expressroute-about-virtual-network-gateways.md).

Pour configurer FastPath, la passerelle de réseau virtuel doit être :

* Ultra-performance
* ErGw3AZ

## <a name="supported-features"></a>Fonctionnalités prises en charge

Bien que FastPath prenne en charge la plupart des configurations, il ne prend pas en charge les fonctionnalités suivantes :

* Routage défini par l’utilisateur sur le sous-réseau de la passerelle : si vous appliquez un routage défini par l’utilisateur (UDR) au sous-réseau de la passerelle de votre réseau virtuel, le trafic de votre réseau local continuera d’être envoyé vers la passerelle du réseau virtuel.

* Peering de réseaux virtuels : si vous avez d’autres réseaux virtuels appairés avec celui qui est connecté à ExpressRoute, le trafic de votre réseau local vers les autres réseaux virtuels (c’est-à-dire les réseaux virtuels dits « spoke ») sera toujours envoyé vers la passerelle du réseau virtuelle. La solution de contournement consiste à connecter tous les réseaux virtuels directement au circuit ExpressRoute.

* Équilibreur de charge de base : si vous déployez un équilibreur de charge interne de base dans votre réseau virtuel ou si le service PaaS Azure que vous déployez dans votre réseau virtuel utilise un équilibreur de charge interne de base, le trafic de votre réseau local vers les adresses IP virtuelles hébergées sur l’équilibreur de charge de base sera envoyé à la passerelle du réseau virtuel. La solution consiste à mettre à niveau l’équilibreur de charge de base vers un [équilibreur de charge standard](../load-balancer/load-balancer-overview.md).

* Liaison privée : si vous vous connectez à un [point de terminaison privé](../private-link/private-link-overview.md) dans votre réseau virtuel à partir de votre réseau local, la connexion passe par la passerelle du réseau virtuel.
 
## <a name="next-steps"></a>Étapes suivantes

Pour activer FastPath, consultez [Lier un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-arm.md#configure-expressroute-fastpath).