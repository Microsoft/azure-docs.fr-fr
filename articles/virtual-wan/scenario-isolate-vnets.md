---
title: 'Scénario : Isolation des réseaux virtuels'
titleSuffix: Azure Virtual WAN
description: Scénarios pour le routage - Isolation des réseaux virtuels
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: e5e2ce17be6d8a1fa82d8a92b9b788f0bd2a37b8
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92424749"
---
# <a name="scenario-isolating-vnets"></a>Scénario : Isolation des réseaux virtuels

Lorsque vous travaillez avec le routage de hub virtuel Virtual WAN, il existe un certain nombre de scénarios disponibles. Dans ce scénario, l’objectif est d’empêcher des réseaux virtuels de pouvoir en atteindre d’autres. C’est ce que l’on appelle l’isolation des réseaux virtuels. Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

## <a name="design"></a><a name="design"></a>Conception

Dans ce scénario, la charge de travail au sein d’un réseau virtuel donné reste isolée et n’est pas en mesure de communiquer avec d’autres réseaux virtuels. Toutefois, les réseaux virtuels sont nécessaires pour atteindre toutes les branches (VPN, ER et VPN utilisateur). Pour déterminer le nombre de tables de routage nécessaires, vous pouvez créer une matrice de connectivité. Pour ce scénario, celle-ci se présente comme le tableau suivant, où chaque cellule indique si une source (ligne) peut communiquer avec une destination (colonne) :

| À partir |   À |  *Réseaux virtuels* | *Branches* |
| -------------- | -------- | ---------- | ---|
| Réseaux virtuels     | &#8594;| Direct |   Direct    |
| Branches   | &#8594;|  Direct  |   Direct    |

Chacune des cellules du tableau précédent indique si une connexion Virtual WAN (côté « De » du flux, les en-têtes de lignes) communique avec un préfixe de destination (côté « À » du flux, en-têtes de colonne en italique). Dans ce scénario, il n’y a pas de pare-feu ni d’appliances virtuelles réseau, de sorte que les communications circulent directement sur Virtual WAN (d’où le mot « Direct » dans le tableau).

Cette matrice de connectivité nous donne deux modèles de ligne différents, qui traduisent en deux tables de routage. Le Virtual WAN possédant déjà une table de routage par défaut, nous aurons besoin d’une autre table de routage. Pour cet exemple, nous allons nommer la table de routage **RT_VNET**.

Des réseaux virtuels seront associés à cette table de routage **RT_VNET**. Étant donné qu’ils ont besoin d’une connectivité aux branches, les branches doivent se propager à **RT_VNET** (autrement, les réseaux virtuels n’apprendront pas les préfixes de branches). Étant donné que les branches sont toujours associées à la table de routage par défaut, les réseaux virtuels devront se propager à la table de routage par défaut. Voici donc la conception finale :

* Réseaux virtuels :
  * Table de routage associée : **RT_VNET**
  * Propagation aux tables de routage : **Par défaut**
* Branches :
  * Table de routage associée : **Par défaut**
  * Propagation aux tables de routage : **RT_VNET** et **Par défaut**

Notez que, dans la mesure où seules les branches se propagent à la table de routage **RT_VNET** , il s’agit des seuls préfixes que les réseaux virtuels apprendront, et non de ceux d’autres réseaux virtuels.

Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

## <a name="workflow"></a><a name="workflow"></a>Flux de travail

Pour configurer ce scénario, prenez en compte les étapes suivantes :

1. Créez une table de routage personnalisée dans chaque Hub. Dans l’exemple, la table de routage est **RT_VNet**. Pour créer une table de routage, consultez le [Guide pratique pour configurer le routage de hub virtuel](how-to-virtual-hub-routing.md). Pour plus d’informations sur les tables de routage, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).
2. Lorsque vous créez la table de routage **RT_VNet** , configurez les paramètres suivants :

   * **Association**  : Sélectionnez les réseaux virtuels que vous souhaitez isoler.
   * **Propagation**  : Sélectionnez l’option pour les branches, ce qui implique que les connexions de branche (VPN/ER/P2S) propageront les itinéraires vers cette table de routage.

:::image type="content" source="./media/routing-scenarios/isolated/isolated-vnets.png" alt-text="Réseaux virtuels isolés":::

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Virtual WAN, consultez la [FAQ](virtual-wan-faq.md).
* Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).
