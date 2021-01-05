---
title: Configurer la haute disponibilité - Hyperscale (Citus) - Azure Database pour PostgreSQL
description: Guide pratique pour activer ou désactiver la haute disponibilité
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 07/27/2020
ms.openlocfilehash: 46b842994cbcf7efe66d5992c79246d77626e268
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90907394"
---
# <a name="configure-hyperscale-citus-high-availability"></a>Configurer la haute disponibilité Hyperscale (Citus)

Azure Database pour PostgreSQL - Hyperscale (Citus) fournit une haute disponibilité pour éviter les temps d’arrêt des bases de données. Quand la haute disponibilité est activée, un nœud de secours est attribué à chaque nœud d’un groupe de serveurs. Si le nœud d’origine n’est plus sain, son nœud de secours est promu pour le remplacer.

> [!IMPORTANT]
> Étant donné que la haute disponibilité double le nombre de serveurs d’un groupe, elle doublera également le coût.

Il est possible d’activer la haute disponibilité lors de la création d’un groupe de serveurs, ou par la suite sous l’onglet **Calcul + stockage** pour votre groupe de serveurs dans le portail Azure. L’interface utilisateur est similaire dans les deux cas. Faites coulisser le curseur de l’option **Haute disponibilité** de NON vers OUI :

:::image type="content" source="./media/howto-hyperscale-high-availability/01-ha-slider.png" alt-text="Curseur de haute disponibilité":::

Cliquez sur le bouton **Enregistrer** pour appliquer votre sélection. L’activation de la haute disponibilité peut prendre un certain temps, car le groupe de serveurs provisionne des nœuds de secours et leur envoie des données en streaming.

L’onglet **Vue d’ensemble** du groupe de serveurs liste tous les nœuds et leurs nœuds de secours, ainsi qu’une colonne **Haute disponibilité** indiquant si la haute disponibilité est correctement activée pour chaque nœud.

:::image type="content" source="./media/howto-hyperscale-high-availability/02-ha-column.png" alt-text="Curseur de haute disponibilité":::

### <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur la [haute disponibilité](concepts-hyperscale-high-availability.md).
