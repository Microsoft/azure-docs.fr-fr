---
title: Comment surveiller les requêtes SQL dans Synapse Studio
description: Découvrez comment surveiller vos requêtes SQL à l’aide de Synapse Studio.
services: synapse-analytics
author: matt1883
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: monitoring
ms.date: 11/30/2020
ms.author: mahi
ms.reviewer: mahi
ms.openlocfilehash: 7296fec91decaf1bd80829f9f3a743a518fbb7a7
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465342"
---
# <a name="use-synapse-studio-to-monitor-your-sql-requests"></a>Utiliser Synapse Studio pour surveiller vos requêtes SQL

Avec Synapse Studio, vous pouvez exécuter des scripts SQL sur les pools SQL dans votre espace de travail.

Cet article explique comment surveiller vos requêtes SQL, ce qui vous permet de garder un œil sur l’état des requêtes en cours d’exécution et de découvrir les détails des requêtes historiques.

## <a name="access-sql-requests-list"></a>Accéder à la liste des requêtes SQL

Pour afficher la liste des requêtes SQL dans votre espace de travail, commencez par [ouvrir Synapse Studio](https://web.azuresynapse.net/), puis sélectionnez votre espace de travail.

![Se connecter à l’espace de travail](./media/common/login-workspace.png)

Une fois que vous avez ouvert votre espace de travail, sélectionnez la section **Monitor** sur la gauche.

![Sélectionner un hub Monitor](./media/common/left-nav.png)

Sélectionnez **Requêtes SQL** pour afficher la liste des requêtes SQL.

 ![Sélectionner des requêtes SQL](./media/how-to-monitor-sql-requests/monitor-hub-nav-sql-requests.png)

## <a name="select-a-sql-pool"></a>Sélectionner un pool SQL

Par défaut, l’historique des requêtes SQL pour le pool SQL serverless intégré apparaît dans cet affichage. Vous pouvez sélectionner l’un de vos pools SQL dédiés pour afficher l’historique des requêtes SQL de ce pool.

![Sélectionner un pool SQL](./media/how-to-monitor-sql-requests/select-pool.png)

## <a name="filter-your-sql-requests"></a>Filtrer vos requêtes SQL

Vous pouvez filtrer la liste des requêtes SQL pour retenir celles qui vous intéressent. Les filtres en haut de l’écran vous permettent de spécifier un champ sur lequel filtrer.

Par exemple, vous pouvez filtrer l’affichage afin de voir uniquement les requêtes SQL soumises par `maria@contoso.com` :

![Exemple de filtre](./media/how-to-monitor-sql-requests/filter-example.png)

## <a name="view-details-about-a-specific-sql-request"></a>Afficher les détails d’une requête SQL spécifique

Pour afficher les détails de l’une de vos requêtes SQL, ouvrez-la, puis accédez à la vue Détails. Pour les requêtes complexes s’exécutant sur des pools SQL dédiés, vous pouvez surveiller leur progression.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la surveillance des exécutions de pipeline, consultez l’article [Surveiller les exécutions de pipeline dans Synapse Studio](how-to-monitor-pipeline-runs.md). 

Pour plus d’informations sur la surveillance d’applications Apache Spark, consultez l’article [Superviser les applications Apache Spark dans Synapse Studio](how-to-monitor-spark-applications.md).