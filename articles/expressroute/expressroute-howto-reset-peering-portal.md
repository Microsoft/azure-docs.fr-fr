---
title: 'Azure ExpressRoute : Réinitialiser un peering de circuit à l’aide du portail Azure'
description: Découvrez comment désactiver et activer les peerings d’un circuit Azure ExpressRoute à l’aide du portail Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 11/30/2020
ms.author: duau
ms.openlocfilehash: d4d6b0b0cce4f5304f7c5790ef2bda05633be52f
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580960"
---
# <a name="reset-expressroute-circuit-peerings-use-the-azure-portal"></a>Réinitialiser des peerings de circuit ExpressRoute à l’aide du portail Azure

Cet article décrit comment désactiver et activer les peerings d’un circuit ExpressRoute à l’aide du portail Azure. Lorsque vous désactivez un peering, la session BGP sur les connexions principale et secondaire de votre circuit ExpressRoute s’arrête. Vous perdez la connectivité via ce peering Microsoft. Lorsque vous activez un peering, la session BGP sur les connexions principale et secondaire de votre circuit ExpressRoute s’affiche. Vous retrouvez la connectivité via ce peering à Microsoft. Vous pouvez activer et désactiver le peering Microsoft et le peering privé Azure de manière indépendante sur un circuit ExpressRoute. Lorsque vous configurez les peerings pour la première fois sur votre circuit ExpressRoute, ils sont activés par défaut.

Il existe quelques scénarios où il peut s’avérer utile de réinitialiser vos peerings ExpressRoute.
* Testez votre conception et votre implémentation de récupération d’urgence. Par exemple, vous avez deux circuits ExpressRoute. Vous pouvez désactiver les peerings d’un circuit et forcer votre trafic réseau à basculer sur l’autre circuit.
* Activez le protocole Bidirectional Forwarding Detection (BFD) sur le peering privé Azure ou le peering Microsoft de votre circuit ExpressRoute. Le protocole BFD est activé par défaut sur le Peering privé Azure si votre circuit ExpressRoute a été créé après le 1er août 2018 et sur un Peering Microsoft. Si votre circuit ExpressRoute a été créé après le 10 janvier 2020. Si votre circuit a été créé avant, BFD n’a pas été activé. Vous pouvez activer BFD en désactivant le peering, puis en le réactivant. 

### <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Dans un navigateur, accédez au [portail Azure](https://portal.azure.com) et connectez-vous avec votre compte Azure.

## <a name="reset-a-peering"></a>Réinitialiser un peering

1. Sélectionnez le circuit dont vous souhaitez modifier la configuration du peering.

    :::image type="content" source="./media/expressroute-howto-reset-peering-portal/expressroute-circuit-list.png" alt-text="Liste de circuits ExpressRoute":::

1. Sélectionnez la configuration de peering que vous souhaitez activer ou désactiver.

    :::image type="content" source="./media/expressroute-howto-reset-peering-portal/expressroute-circuit.png" alt-text="Vue d’ensemble de circuit ExpressRoute":::

1. Désactivez la case à cocher **Activer le peering**, puis sélectionnez **Enregistrer** pour désactiver la configuration du peering.

    :::image type="content" source="./media/expressroute-howto-reset-peering-portal/disable-peering.png" alt-text="Désactiver un peering privé":::

1. Vous pouvez réactiver le peering en activant la case à cocher **Activer le peering**, puis sélectionner **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes
Si vous avez besoin d’aide pour résoudre un problème avec ExpressRoute, consultez les articles suivants :
* [Vérification de la connectivité ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
* [Résolution des problèmes de performances réseau](expressroute-troubleshooting-network-performance.md)
