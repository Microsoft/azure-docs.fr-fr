---
title: 'Scénario : any-to-any'
titleSuffix: Azure Virtual WAN
description: Scénarios pour routage - any-to-any
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: 09dddad24794491b53a11f7b0e4347f43f11598b
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92440482"
---
# <a name="scenario-any-to-any"></a>Scénario : Any-to-any

Lorsque vous travaillez avec le routage de hub virtuel Virtual WAN, il existe un certain nombre de scénarios disponibles. Dans un scénario Any-to-any, tout spoke peut atteindre un autre spoke. En présence de plusieurs hubs, le routage de hub à hub (ou « entre hubs ») est activé par défaut dans le Virtual WAN standard. Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

## <a name="design"></a><a name="design"></a>Conception

Pour déterminer le nombre de tables de routage nécessaires dans un scénario de Virtual WAN, vous pouvez créer une matrice de connectivité, où chaque cellule indique si une source (ligne) peut communiquer avec une destination (colonne).

| À partir |   À |  *Réseaux virtuels* | *Branches* |
| -------------- | -------- | ---------- | ---|
| Réseaux virtuels     | &#8594;| Direct | Direct |
| Branches   | &#8594;| Direct  | Direct |

Chacune des cellules du tableau précédent indique si une connexion de Virtual WAN (côté « De » du flux, les en-têtes de lignes) communique avec préfixe de destination (côté « À » du flux, en-têtes de colonne en italique). Dans ce scénario, il n’y a pas de pare-feu ou d’appliances virtuelles réseau, ainsi la communication circule directement sur Virtual WAN (d’où le mot « Direct » dans le tableau).

Étant donné que toutes les connexions à partir de réseaux virtuels et de branches (VPN, ExpressRoute et VPN utilisateur) ont les mêmes exigences de connectivité, une seule table de routage est requise. Par conséquent, toutes les connexions sont associées et propagées à la même table de routage, la table de routage par défaut :

* Réseaux virtuels :
  * Table de routage associée : **Par défaut**
  * Propagation aux tables de routage : **Par défaut**
* Branches :
  * Table de routage associée : **Par défaut**
  * Propagation aux tables de routage : **Par défaut**

Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

## <a name="architecture"></a><a name="architecture"></a>Architecture

Dans la **Figure 1** , tous les réseaux virtuels et les branches (VPN, ExpressRoute, P2S) peuvent se connecter les uns aux autres. Dans un hub virtuel, les connexions fonctionnent comme suit :

* Une connexion VPN connecte un site VPN à une passerelle VPN.
* Une connexion de réseau virtuel connecte un réseau virtuel à un hub virtuel. Le routeur du hub virtuel fournit la fonctionnalité de transit entre les réseaux virtuels.
* Une connexion ExpressRoute connecte un circuit ExpressRoute à une passerelle ExpressRoute.

Ces connexions (par défaut au moment de la création) sont associées à la table de routage par défaut, sauf si vous configurez la configuration de routage de la connexion soit sur **Aucun** , soit sur une table de routage personnalisée. Ces connexions propagent également les itinéraires par défaut vers la table de routage par défaut. C’est ce qui permet un scénario any-to-any où tout spoke (VNet, VPN, ER, P2S) peut atteindre un autre spoke.

**Figure 1**

:::image type="content" source="./media/routing-scenarios/any-any/figure-1.png" alt-text="figure 1":::

## <a name="workflow"></a><a name="workflow"></a>Flux de travail

Ce scénario est activé par défaut pour le Virtual WAN standard. Si le paramètre branche à branche est désactivé dans la configuration WAN, cela empêchera la connectivité entre les spokes de branche. VPN/ExpressRoute/utilisateur VPN sont considérés comme des spokes de branche dans Virtual WAN

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Virtual WAN, consultez la [FAQ](virtual-wan-faq.md).
* Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).
