---
title: 'Scénario : Isolation personnalisée pour réseaux virtuels'
titleSuffix: Azure Virtual WAN
description: Scénarios de routage - empêcher les réseaux virtuels sélectionnés de pouvoir s’atteindre mutuellement
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: cherylmc
ms.openlocfilehash: ca1ee8418bc08d70a031d81a15dc1b4ace2f1a3a
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92461819"
---
# <a name="scenario-custom-isolation-for-vnets"></a>Scénario : Isolation personnalisée pour réseaux virtuels

Lorsque vous travaillez avec le routage de hub virtuel Virtual WAN, il existe un certain nombre de scénarios disponibles. Dans un scénario d’isolation personnalisée pour réseaux virtuels, l’objectif est d’empêcher un ensemble spécifique de réseaux virtuels de pouvoir atteindre d’autres ensembles de réseaux virtuels spécifiques. Toutefois, les réseaux virtuels sont nécessaires pour atteindre toutes les branches (VPN/ER/VPN utilisateur). Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

## <a name="design"></a><a name="design"></a>Conception

Pour déterminer le nombre de tables de routage nécessaires, vous pouvez créer une matrice de connectivité. Pour ce scénario, celle-ci se présente comme suit, où chaque cellule indique si une source (ligne) peut communiquer avec une destination (colonne) :

| Du | Par :| *Réseaux virtuels bleus* | *Réseaux virtuels rouges* | *Branches*|
|---|---|---|---|---|
| **Réseaux virtuels bleus** |   &#8594;|   Direct     |           |  Direct |
| **Réseaux virtuels rouges**  |   &#8594;|              |   Direct  |  Direct |
| **Branches**   |   &#8594;|   Direct     |   Direct  |  Direct |

Chacune des cellules du tableau précédent indique si une connexion Virtual WAN (côté « De » du flux, les en-têtes de lignes) communique avec une destination (côté « À » du flux, en-têtes de colonne en italique). Dans ce scénario, il n’y a pas de pare-feu ni d’appliances virtuelles réseau, de sorte que les communications circulent directement sur Virtual WAN (d’où le mot « Direct » dans le tableau).

Le nombre de modèles de ligne différents sera le nombre de tables de routage dont nous aurons besoin dans ce scénario. Ici, trois tables de routage de routage sont appelées **RT_BLUE** et **RT_RED** pour les réseaux virtuels, et **Par défaut** pour les branches. N’oubliez pas que les branches doivent toujours être associées à la table de routage Par défaut.

Les branches devront apprendre les préfixes des réseaux virtuels rouges et bleus, de sorte que tous les réseaux virtuels se propagent à la table Par défaut (en plus de la table d’origine **RT_BLUE** ou **RT_RED** ). Les réseaux virtuels bleus et rouges devront apprendre les préfixes des branches, afin que les branches se propagent elles aussi aux tables de routage **RT_BLUE** et **RT_RED**. Voici donc la conception finale :

* Réseaux virtuels bleus :
  * Table de routage associée : **RT_BLUE**
  * Propagation aux tables de routage : **RT_BLUE** et **Par défaut**
* Réseaux virtuels rouges :
  * Table de routage associée : **RT_RED**
  * Propagation aux tables de routage : **RT_RED** et **Par défaut**
* Branches :
  * Table de routage associée : **Par défaut**
  * Propagation aux tables de routage : **RT_BLUE** , **RT_RED** et **Par défaut**

> [!NOTE]
> Étant donné que toutes les branches doivent être associées à la table de routage Par défaut et se propager vers le même ensemble de tables de routage, toutes les branches ont le même profil de connectivité. Autrement dit, le concept Rouge/Bleu des réseaux virtuels ne peut pas être appliqué aux branches.

> [!NOTE]
> Si votre WAN virtuel est déployé sur plusieurs régions, vous devez créer les tables de routage **RT_BLUE** et **RT_RED** dans chaque hub, et les itinéraires de chaque connexion de réseau virtuel doivent être propagés aux tables de routage dans chaque hub virtuel à l’aide des étiquettes de propagation.

Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

## <a name="workflow"></a><a name="architecture"></a>Flux de travail

Dans **figure 1** , il y a des connexions de réseaux virtuels en bleu et en rouge.

* Les réseaux virtuels connectés en bleu peuvent s’atteindre entre eux, ainsi qu’atteindre toutes les connexions de branches (VPN/ER/P2S).
* Les réseaux virtuels connectés en rouge peuvent s’atteindre entre eux, ainsi qu’atteindre toutes les connexions de branches (VPN/ER/P2S).

Suivez les étapes ci-dessous lors de la configuration du routage.

1. Créez deux tables de routage personnalisées dans le Portail Azure, **RT_BLUE** et **RT_RED**.
2. Pour la table de routage **RT_BLUE** , pour les paramètres suivants :
   * **Association**  : Sélectionnez tous les réseaux virtuels bleus.
   * **Propagation**  : Pour les branches, sélectionnez l’option pour les branches, ce qui implique que les connexions de branche (VPN/ER/P2S) propageront les itinéraires vers cette table de routage.
3. Répétez les mêmes étapes pour la table de routage **RT_RED** pour les réseaux virtuels rouges et les branches (VPN/ER/P2S).

Cela entraînera des modifications de configuration de routage, comme illustré dans la figure ci-dessous

**Figure 1**

:::image type="content" source="./media/routing-scenarios/custom-isolation/custom.png" alt-text="figure 1":::

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Virtual WAN, consultez la [FAQ](virtual-wan-faq.md).
* Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).
