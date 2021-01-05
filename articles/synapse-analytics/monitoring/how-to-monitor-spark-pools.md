---
title: Comment surveiller les pools Apache Spark dans Synapse Studio
description: Découvrez comment surveiller vos pools Apache Spark à l’aide de Synapse Studio.
services: synapse-analytics
author: matt1883
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: monitoring
ms.date: 11/30/2020
ms.author: mahi
ms.reviewer: mahi
ms.openlocfilehash: 8d95897e0c2d58b2a3955918be945800eed9ba56
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465316"
---
# <a name="use-synapse-studio-to-monitor-your-apache-spark-pools"></a>Utiliser Synapse Studio pour surveiller vos pools Apache Spark

Azure Synapse Analytics vous permet d’utiliser Spark pour exécuter des bloc-notes, des travaux et d’autres types d’applications sur des pools Spark dans votre espace de travail.

Cet article explique comment surveiller vos pools Apache Spark, ce qui vous permet de garder un œil sur leur état, notamment le nombre de vCore utilisés par les différents utilisateurs de l’espace de travail.

## <a name="access-spark-pools-list"></a>Accéder à la liste des pools Spark

Pour voir la liste des pools Apache Spark dans votre espace de travail, commencez par [ouvrir Synapse Studio](https://web.azuresynapse.net/), puis sélectionnez votre espace de travail.

![Se connecter à l’espace de travail](./media/common/login-workspace.png)

Une fois que vous avez ouvert votre espace de travail, sélectionnez la section **Monitor** sur la gauche.

![Sélectionner un hub Monitor](./media/common/left-nav.png)

Sélectionnez **Pools Apache Spark** pour afficher la liste des pools de Apache Spark.

 ![Sélectionner Pools Apache Spark](./media/how-to-monitor-spark-pools/monitor-hub-nav-spark-pools.png)

## <a name="filter-your-spark-pools"></a>Filtrer vos pools Spark

Vous pouvez filtrer la liste des pools Spark pour retenir ceux qui vous intéressent. Les filtres en haut de l’écran vous permettent de spécifier un champ sur lequel filtrer.

Par exemple, vous pouvez filtrer l’affichage pour voir uniquement les pools Spark contenant le nom « dataprep » :

![Exemple de filtre](./media/how-to-monitor-spark-pools/filter-example.png)

## <a name="view-details-about-a-specific-spark-pool"></a>Afficher les détails d’un pool Spark spécifique

Pour afficher les détails de l’un de vos pools Spark, sélectionnez-le.

![Détails du pool Apache Spark](./media/how-to-monitor-spark-pools/spark-pool-details.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la surveillance des exécutions de pipeline, consultez l’article [Surveiller les exécutions de pipeline dans Synapse Studio](how-to-monitor-pipeline-runs.md). 

Pour plus d’informations sur la surveillance d’applications Apache Spark, consultez l’article [Superviser les applications Apache Spark dans Synapse Studio](how-to-monitor-spark-applications.md).
