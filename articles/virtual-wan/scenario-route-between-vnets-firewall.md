---
title: 'Scénario : Routage personnalisé du Pare-feu Azure pour Virtual WAN'
titleSuffix: Azure Virtual WAN
description: Scénarios de routage du trafic entre réseaux virtuels directement, mais utilisation du Pare-feu Azure pour les flux de trafic de réseau virtuel -> Internet/branche et de branche à réseau virtuel
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: d083607782f96744ecbd7d23976f77ee53fec49d
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94515567"
---
# <a name="scenario-azure-firewall---custom"></a>Scénario : Pare-feu Azure - personnalisé

Lorsque vous travaillez avec le routage de hub virtuel Virtual WAN, il existe un certain nombre de scénarios disponibles. Dans ce scénario, l’objectif n’est pas d’acheminer le trafic directement entre les réseaux virtuels, mais d’utiliser Pare-feu Azure pour les flux de trafic des réseaux virtuels vers Internet et la branche, et de la branche vers les réseaux virtuels.

## <a name="design"></a><a name="design"></a>Conception

Pour déterminer le nombre de tables de routage nécessaires, vous pouvez créer une matrice de connectivité, où chaque cellule indique si une source (ligne) peut communiquer avec une destination (colonne). La matrice de connectivité dans ce scénario est simple, mais nous pouvons toujours l’examiner par souci de cohérence avec d’autres scénarios.

**Matrice de connectivité**

| Du           | Par :      | *Réseaux virtuels*      | *Branches*    | *Internet*   |
|---             |---       |---           |---            |---           |
| **Réseaux virtuels**      |   &#8594;|    Direct    |     AzFW      |     AzFW     |
| **Branches**   |   &#8594;|    AzFW      |    Direct     |    Direct    |

Dans le tableau précédent, « Direct » représente une connectivité directe entre deux connexions sans le trafic qui traverse le Pare-feu Azure dans le réseau étendu virtuel, et « AzFW » indique que le flux passe par le Pare-feu Azure. Comme il existe deux modèles de connectivité distincts dans la matrice, nous aurons besoin de deux tables de routage qui seront configurées comme suit :

* Réseaux virtuels :
  * Table de routage associée : **RT_VNet**
  * Propagation aux tables de routage : **RT_VNet**
* Branches :
  * Table de routage associée : **Par défaut**
  * Propagation aux tables de routage : **Par défaut**

> [!NOTE]
> Vous pouvez créer une instance Virtual WAN distincte avec un seul hub virtuel sécurisé dans chaque région, puis connecter les Virtual WAN entre eux via un VPN site à site.

Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

## <a name="workflow"></a><a name="workflow"></a>Flux de travail

Dans ce scénario, vous souhaitez acheminer le trafic via le Pare-feu Azure pour le trafic de réseau virtuel à Internet, de réseau virtuel à branche ou de branche à réseau virtuel, mais voulez un trafic direct de réseau virtuel à réseau virtuel. Si vous avez utilisé Azure Firewall Manager, les paramètres de routage sont automatiquement renseignés dans la **table de routage par défaut**. Le trafic privé s’applique au réseau virtuel et aux branches, et le trafic Internet à 0.0.0.0/0.

Les connexions VPN, ExpressRoute et VPN utilisateur sont collectivement appelées branches, et associées à la même table de routage (par défaut). Toutes les connexions VPN, ExpressRoute et VPN utilisateur propagent des itinéraires vers le même ensemble de tables de routage. Pour configurer ce scénario, envisagez les étapes suivantes :

1. Créer une table de routage personnalisée **RT_VNet**.
1. Créer un itinéraire pour activer le trafic de réseau virtuel à Internet et de réseau virtuel à branche : 0.0.0.0/0 avec le tronçon suivant pointant vers le Pare-feu Azure. Dans la section Propagation, vous allez vous assurer que les réseaux virtuels sont sélectionnés, ce qui garantit des itinéraires plus spécifiques, et permet un flux de trafic direct de réseau virtuel à réseau virtuel.

   * Dans **Association :** sélectionnez Réseaux virtuels, ce qui implique que les réseaux virtuels atteindront la destination en fonction des itinéraires de cette table de routage.
   * Dans **Propagation :** sélectionnez Réseaux virtuels, ce qui implique que les réseaux virtuels se propagent à cette table de routage. En d’autres mots, des itinéraires plus spécifiques se propageront vers cette table de routage, garantissant ainsi un flux de trafic direct entre réseaux virtuels.

1. Ajoutez un itinéraire statique agrégé pour réseaux les virtuels dans la **table de routage par défaut** pour activer le flux de branche à réseau virtuel via le Pare-feu Azure.

   * N’oubliez pas que les branches sont associées et se propagent vers la table de routage par défaut.
   * Les branches ne se propagent pas vers la table de routage RT_VNet. Cela garantit le flux de trafic de réseau virtuel à branche passe par le Pare-feu Azure.

Cela entraîne des modifications de configuration de routage, comme l’illustre la **Figure 1**.

**Figure 1**

:::image type="content" source="./media/routing-scenarios/between-vnets-firewall/routing.png" alt-text="Figure 1":::

> [!NOTE]
> Les hubs Virtual WAN et les réseaux virtuels connectés doivent se trouver dans la même région Azure.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Virtual WAN, consultez la [FAQ](virtual-wan-faq.md).
* Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).
* Pour plus d’informations sur la configuration du routage de hub virtuel, consultez [Guide pratique pour configurer le routage de hub virtuel](how-to-virtual-hub-routing.md).
