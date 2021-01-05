---
title: À propos de la mise à niveau de la bande passante du circuit | Azure ExpressRoute
description: Dans cet article, découvrez les bonnes pratiques de mise à niveau de la bande passante d’un circuit ExpressRoute
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: duau
ms.openlocfilehash: 7831e7944321e074c312853e1534c47970ebdfdb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89397948"
---
# <a name="about-upgrading-expressroute-circuit-bandwidth"></a>À propos de la mise à niveau de la bande passante d’un circuit ExpressRoute

ExpressRoute permet une connectivité dédiée et privée au réseau global de Microsoft. La connectivité est facilitée par le réseau d’un partenaire ExpressRoute, ou par une connexion directe aux appareils Microsoft Enterprise Edge (MSEE). Une fois la connectivité physique configurée et testée, vous pouvez activer la connectivité de couche 2 et de couche 3 en créant un circuit ExpressRoute et en configurant le peering.

## <a name="upgrade-circuit-bandwidth"></a><a name="upgrade"></a>Mettre à niveau la bande passante d’un circuit

Pour mettre correctement à niveau la bande passante d’un circuit, le partenaire ExpressRoute ou ExpressRoute Direct doit disposer d’une [quantité de bande passante disponible suffisante](#considerations).

Si la capacité est disponible, vous pouvez mettre à niveau le circuit à l’aide des méthodes suivantes :

* [Azure portal](expressroute-howto-circuit-portal-resource-manager.md#modify)
* [PowerShell](expressroute-howto-circuit-arm.md#modify)
* [Azure CLI](howto-circuit-cli.md#modify)

## <a name="capacity-considerations"></a><a name="considerations"></a>Considérations relatives à la capacité

### <a name="insufficient-expressroute-partner-bandwidth"></a><a name="bandwidth"></a>Bande passante du partenaire ExpressRoute insuffisante

Si le partenaire ExpressRoute ne dispose pas d’une capacité suffisante, vous devez créer un autre circuit, configuré pour la bande passante souhaitée. Pour maintenir la connectivité, ne supprimez pas l’ancien circuit tant que le circuit nouvellement créé n’est pas provisionné, que le peering n’a pas été configuré et (en ce qui concerne le peering privé) que l’objet de connexion à la passerelle de réseau virtuel ExpressRoute n’a pas été provisionné.

Si votre partenaire ExpressRoute ne dispose pas de suffisamment d’une capacité disponible, vous devez demander une capacité supplémentaire à l’emplacement de peering souhaité. Une fois la nouvelle capacité approvisionnée, vous pouvez utiliser les étapes de la section [Mettre à niveau la bande passante d’un circuit](#upgrade) pour créer un circuit, configurer la connectivité et supprimer l’ancien circuit.


### <a name="insufficient-expressroute-direct-bandwidth"></a><a name="bandwidth"></a>Bande passante ExpressRoute Direct insuffisante

Si ExpressRoute direct ne dispose pas d’une capacité suffisante, vous pouvez soit supprimer les circuits associés à la ressource ExpressRoute Direct qui ne sont plus nécessaires, soit créer une autre ressource ExpressRoute Direct. Pour obtenir des instructions sur la gestion de la ressource ExpressRoute Direct, consultez [Guide pratique pour configurer ExpressRoute Direct](how-to-expressroute-direct-portal.md).

## <a name="next-steps"></a>Étapes suivantes

* [Créer et modifier un circuit](expressroute-howto-circuit-portal-resource-manager.md)
* [Créer et modifier une configuration de peering](expressroute-howto-routing-portal-resource-manager.md)
* [Lier un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md)
