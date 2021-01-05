---
title: Comment surveiller les applications Apache Spark dans Synapse Studio
description: Découvrez comment superviser vos applications Apache Spark en utilisant Synapse Studio.
services: synapse-analytics
author: matt1883
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: monitoring
ms.date: 11/30/2020
ms.author: mahi
ms.reviewer: mahi
ms.openlocfilehash: c1545efc43d034dba5b8ffe8d19b9bbee95dff68
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96455467"
---
# <a name="use-synapse-studio-to-monitor-your-apache-spark-applications"></a>Utiliser Synapse Studio pour surveiller vos applications Apache Spark

Azure Synapse Analytics vous permet d’utiliser Spark pour exécuter des bloc-notes, des travaux et d’autres types d’applications sur vos pools Spark dans votre espace de travail.

Cet article explique comment superviser vos applications Apache Spark, et ainsi garder un œil sur leur état, leurs problèmes et leur progression.

## <a name="access-apache-spark-applications-list"></a>Accéder à la liste des applications Apache Spark

Pour voir la liste des applications Apache Spark dans votre espace de travail, commencez par [ouvrir Synapse Studio](https://web.azuresynapse.net/) et sélectionnez votre espace de travail.

![Se connecter à l’espace de travail](./media/common/login-workspace.png)

Une fois que vous avez ouvert votre espace de travail, sélectionnez la section **Monitor** sur la gauche.

![Sélectionner un hub Monitor](./media/common/left-nav.png)

Sélectionnez **Applications Apache Spark** pour voir la liste de ces applications.

 ![Sélectionner des applications Spark](./media/how-to-monitor-spark-applications/monitor-hub-nav-spark-applications.png)

## <a name="filter-your-apache-spark-applications"></a>Filtrer vos applications Apache Spark

Vous pouvez filtrer la liste des applications Apache Spark en la limitant à celles qui vous intéressent. Les filtres en haut de l’écran vous permettent de spécifier un champ sur lequel filtrer.

Par exemple, vous pouvez filtrer la vue pour voir uniquement les applications Apache Spark contenant le nom « sales » :

![Exemple de filtre](./media/how-to-monitor-spark-applications/filter-example.png)

## <a name="view-details-about-a-specific-apache-spark-application"></a>Affichere des détails d’une application Apache Spark spécifique

Pour voir les détails sur l’une de vos applications Apache Spark, sélectionnez l’application Apache Spark concernée. Si l’application Apache Spark est toujours en cours d’exécution, vous pouvez superviser la progression. [En savoir plus](apache-spark-applications.md).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la surveillance des exécutions de pipeline, consultez l’article [Surveiller les exécutions de pipeline avec Synapse Studio](how-to-monitor-pipeline-runs.md). 

Pour plus d’informations sur le débogage d’une application Apache Spark, consultez l’article [Superviser les applications Apache Spark sur Synapse Studio](apache-spark-applications.md).