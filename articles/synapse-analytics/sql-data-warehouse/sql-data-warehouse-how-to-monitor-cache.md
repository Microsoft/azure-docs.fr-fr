---
title: Optimiser le cache Gen2
description: Découvrez comment surveiller votre cache Gen2 à l’aide du portail Microsoft Azure.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.subservice: sql-dw
ms.topic: conceptual
ms.date: 11/20/2020
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: 041751b5b23dbb3153f1ae638303579a860c0e5b
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95020161"
---
# <a name="how-to-monitor-the-adaptive-cache"></a>Procédure de surveillance du cache adaptatif

Cet article décrit la procédure de surveillance et de résolution des problèmes de lenteur des performances des requêtes en déterminant si votre charge de travail exploite de façon optimale le cache adaptatif des pools SQL dédiés.

L’architecture de stockage d’un pool SQL dédié nivelle automatiquement vos segments columnstore les plus fréquemment interrogés dans un cache résidant sur des disques SSD basés sur NVMe. Vous obtiendrez des performances supérieures lorsque vos requêtes récupèrent des segments qui résident dans le cache.
 
## <a name="troubleshoot-using-the-azure-portal"></a>Résolution des problèmes à l’aide du portail Azure

Vous pouvez utiliser Azure Monitor pour afficher les mesures du cache afin de résoudre les problèmes de performances des requêtes. Commencez par accéder au portail Microsoft Azure et cliquez sur **Superviser**, **Métriques** et **+ Sélectionner une étendue** :

![Capture d’écran montrant l’option Sélectionner une étendue sélectionnée dans Métriques sur le portail Azure.](./media/sql-data-warehouse-how-to-monitor-cache/cache-0.png)

Utilisez les barres de recherche et de liste déroulante pour rechercher votre pool SQL dédié. Sélectionnez ensuite Appliquer.

![Capture d’écran montrant le volet Sélectionner une étendue dans lequel vous pouvez sélectionner votre entrepôt de données.](./media/sql-data-warehouse-how-to-monitor-cache/cache-1.png)

Les métriques clés pour la résolution des problèmes du cache sont **Pourcentage d’accès au cache** et **Pourcentage de cache utilisé**. Sélectionnez **Pourcentage d’accès au cache**, puis utilisez le bouton **Ajouter une métrique** pour ajouter **Pourcentage de cache utilisé**. 

![Métriques du cache](./media/sql-data-warehouse-how-to-monitor-cache/cache-2.png)

## <a name="cache-hit-and-used-percentage"></a>Correspondance dans le cache et pourcentage utilisé

La matrice ci-dessous décrit les scénarios en fonction des valeurs des métriques de cache :

|                                | **Pourcentage d’accès au cache élevé** | **Pourcentage d’accès au cache faible** |
| :----------------------------: | :---------------------------: | :--------------------------: |
| **Pourcentage de cache utilisé élevé** |          Scénario 1           |          Scénario 2          |
| **Pourcentage de cache utilisé faible**  |          Scénario 3           |          Scénario 4          |

**Scénario 1 :** Vous utilisez votre cache de manière optimale. [Détecter les problèmes](sql-data-warehouse-manage-monitor.md) d’autres éléments qui peuvent ralentir vos requêtes.

**Scénario 2 :** Votre jeu de données de travail en cours ne peut pas tenir dans le cache, ce qui provoque un faible pourcentage d’accès au cache en raison de lectures physiques. Tenez compte de la montée en puissance de votre niveau de performance, puis réexécutez votre charge de travail pour remplir le cache.

**Scénario 3 :** Il est probable que votre requête s’exécute lentement pour des raisons non liées au cache. [Détecter les problèmes](sql-data-warehouse-manage-monitor.md) d’autres éléments qui peuvent ralentir vos requêtes. Vous pouvez également envisager une [descente en puissance de votre instance](sql-data-warehouse-manage-monitor.md) pour réduire la taille du cache afin de diminuer les coûts. 

**Scénario 4 :** Vous aviez un cache froid, ce qui pourrait être la raison pour laquelle votre requête a été lente. Envisagez de réexécuter votre requête, car votre jeu de données de travail doit maintenant être mis en cache. 

> [!IMPORTANT]
> Si le pourcentage de correspondance dans le cache ou le pourcentage de cache utilisé n’est pas mis à jour après une réexécution de votre charge de travail, votre plage de travail réside peut-être déjà en mémoire. Seules les tables columnstore du cluster sont mises en cache.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur le réglage des performances des requêtes générales, consultez [Surveillance de l’exécution des rêquêtes](sql-data-warehouse-manage-monitor.md#monitor-query-execution).
