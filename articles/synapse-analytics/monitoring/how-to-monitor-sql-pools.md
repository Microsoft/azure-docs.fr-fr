---
title: Comment surveiller des pools SQL dans Synapse Studio
description: Découvrez comment surveiller vos pools SQL à l’aide de Synapse Studio.
services: synapse-analytics
author: matt1883
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: monitoring
ms.date: 11/30/2020
ms.author: mahi
ms.reviewer: mahi
ms.openlocfilehash: 63b63526333435387c3ff5b5c9d5599ec851c1a8
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465345"
---
# <a name="use-synapse-studio-to-monitor-your-sql-pools"></a>Utiliser Synapse Studio pour surveiller vos pools SQL

Avec Synapse Studio, vous pouvez exécuter des scripts SQL sur les pools SQL dans votre espace de travail.

Cet article explique comment surveiller vos pools SQL, ce qui vous permet de garder un œil sur l’état et l’activité de vos pools.

## <a name="access-sql-pools-list"></a>Accéder à la liste des pools SQL

Pour afficher la liste des pools SQL dans votre espace de travail, commencez par [ouvrir Synapse Studio](https://web.azuresynapse.net/), puis sélectionnez votre espace de travail.

![Se connecter à l’espace de travail](./media/common/login-workspace.png)

Une fois que vous avez ouvert votre espace de travail, sélectionnez la section **Monitor** sur la gauche.

![Sélectionner un hub Monitor](./media/common/left-nav.png)

Sélectionnez **Pools SQL** pour afficher la liste des pools SQL.

 ![Sélectionner des pools SQL](./media/how-to-monitor-sql-pools/monitor-hub-nav-sql-pools.png)

## <a name="filter-your-sql-pools"></a>Filtrer vos pools SQL

Vous pouvez filtrer la liste des pools SQL pour retenir ceux qui vous intéressent. Les filtres en haut de l’écran vous permettent de spécifier un champ sur lequel filtrer.

Par exemple, vous pouvez filtrer l’affichage pour voir uniquement les pools SQL contenant le nom « salesrecords » :

![Exemple de filtre](./media/how-to-monitor-sql-pools/filter-example.png)

## <a name="view-details-about-a-specific-sql-pool"></a>Afficher les détails d’un pool SQL spécifique

Pour afficher les détails de l’un de vos pools SQL, sélectionnez-le.

![Détails du pool SQL](./media/how-to-monitor-sql-pools/sql-pool-details.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la surveillance des exécutions de pipeline, consultez l’article [Surveiller les exécutions de pipeline dans Synapse Studio](how-to-monitor-pipeline-runs.md). 

Pour plus d’informations sur la surveillance des requêtes SQL, consultez l’article [Surveiller les requêtes SQL dans Synapse Studio](how-to-monitor-sql-requests.md).