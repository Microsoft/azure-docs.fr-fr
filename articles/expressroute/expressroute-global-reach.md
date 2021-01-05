---
title: 'Azure ExpressRoute : Se connecter à Microsoft Cloud à l’aide de Global Reach'
description: Découvrez comment Azure ExpressRoute Global Reach peut associer des circuits ExpressRoute afin de constituer un réseau privé entre vos réseaux locaux.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 09/18/2020
ms.author: duau
ms.custom: references_regions
ms.openlocfilehash: e3f9105037c049a53f1b7b99da96dd857070fcc7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90987614"
---
# <a name="expressroute-global-reach"></a>Service Global Reach d’ExpressRoute
ExpressRoute vous permet de connecter vos réseaux locaux à Microsoft Cloud de manière privée et résiliente. Vous pouvez accéder à de nombreux services cloud Microsoft, comme Azure et Microsoft 365, à partir de votre centre de données privé ou de votre réseau d’entreprise. Par exemple, vous pouvez avoir une succursale à San Francisco avec un circuit ExpressRoute dans la Silicon Valley, et une autre à Londres avec un circuit ExpressRoute dans cette même ville. Les deux succursales disposent d’une connectivité haut débit avec les ressources Azure situées dans les régions USA Ouest et Royaume-Uni Sud, mais elles ne peuvent pas se connecter et envoyer des données directement entre elles. En d’autres termes, 10.0.1.0/24 peut envoyer des données aux réseaux 10.0.3.0/24 et 10.0.4.0/24, mais PAS au réseau 10.0.2.0/24.

![Schéma illustrant des circuits non liés avec ExpressRoute Global Reach.][1]

Avec **ExpressRoute Global Reach**, vous pouvez associer des circuits ExpressRoute afin de constituer un réseau privé entre vos réseaux locaux. Dans l’exemple ci-dessus, grâce à l’ajout d’ExpressRoute Global Reach, votre bureau de San Francisco (10.0.1.0/24) peut échanger directement des données avec le bureau de Londres (10.0.2.0/24) par le biais des circuits ExpressRoute existants et du réseau mondial de Microsoft. 

![Schéma illustrant des circuits liés avec ExpressRoute Global Reach.][2]

## <a name="use-case"></a>Cas d’utilisation
ExpressRoute Global Reach est conçu pour compléter l’implémentation du réseau étendu (WAN) de votre fournisseur de services et connecter vos succursales à travers le monde. Par exemple, si votre fournisseur de services opère principalement aux États-Unis et a connecté toutes vos branches dans ce pays, mais qu’il n’opère ni au Japon ni à Hong Kong (R.A.S.), ExpressRoute Global Reach vous permet de travailler avec un fournisseur de services local, et Microsoft connecte vos branches situées là-bas à celles figurant aux États-Unis à l’aide d’ExpressRoute et de notre réseau mondial.

![Schéma illustrant un cas d’usage pour ExpressRoute Global Reach.][3]

## <a name="availability"></a>Disponibilité 
La fonctionnalité ExpressRoute Global Reach est prise en charge aux emplacements suivants. 

> [!NOTE] 
> Pour activer ExpressRoute Global Reach entre [différentes régions](expressroute-locations-providers.md#locations), vous devez disposer de circuits **SKU Premium**.

* Australie
* Canada
* France
* Allemagne
* Hong Kong (R.A.S.)
* Irlande
* Japon
* Corée du Sud
* Pays-Bas
* Nouvelle-Zélande
* Norvège
* Singapour
* Suède
* Suisse
* Royaume-Uni
* États-Unis

## <a name="next-steps"></a>Étapes suivantes
- Consulter les [Questions fréquentes (FAQ) sur Global Reach](expressroute-faqs.md#globalreach).
- Découvrir comment [activer Global Reach](expressroute-howto-set-global-reach.md).
- Découvrir comment [lier un circuit ExpressRoute à votre réseau virtuel](expressroute-howto-linkvnet-arm.md).

<!--Image References-->
[1]: ./media/expressroute-global-reach/1.png "schéma sans portée globale"
[2]: ./media/expressroute-global-reach/2.png "schéma avec portée globale"
[3]: ./media/expressroute-global-reach/3.png "cas d’usage de global reach"
