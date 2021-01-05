---
title: Surveiller les exécutions de pipeline à l’aide de Synapse Studio
description: Utilisez la fonctionnalité Synapse Studio pour surveiller les exécutions du pipeline de votre espace de travail.
services: synapse-analytics
author: matt1883
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: monitoring
ms.date: 11/30/2020
ms.author: mahi
ms.reviewer: mahi
ms.openlocfilehash: dbd890117c78274392d5745e0563332371b404c5
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452575"
---
# <a name="use-synapse-studio-to-monitor-your-workspace-pipeline-runs"></a>Utiliser Synapse Studio pour surveiller les exécutions du pipeline de votre espace de travail

Azure Synapse Analytics vous permet de créer des pipelines complexes capables d'automatiser et d'intégrer le déplacement et la transformation des données, ainsi que des activités de calcul au sein de votre solution. Vous pouvez créer et surveiller ces pipelines à l’aide de Synapse Studio.

Cet article explique comment surveiller les exécutions de vos pipelines, et ainsi garder un œil sur leur état, leurs problèmes et leur progression.

## <a name="access-pipeline-runs-list"></a>Accéder à la liste des exécutions du pipeline

Pour afficher la liste des exécutions du pipeline dans votre espace de travail, commencez par [ouvrir la fonctionnalité Synapse Studio](https://web.azuresynapse.net/) et sélectionner votre espace de travail.

![Se connecter à l’espace de travail](./media/common/login-workspace.png)

Une fois que vous avez ouvert votre espace de travail, sélectionnez la section **Monitor** sur la gauche.

![Sélectionner un hub Monitor](./media/common/left-nav.png)

Sélectionnez **Exécutions de pipeline** pour afficher la liste des exécutions.

![Sélectionner les exécutions de pipeline](./media/how-to-monitor-pipeline-runs/monitor-hub-nav-pipelineruns.png)

## <a name="filter-your-pipeline-runs"></a>Filtrer les exécutions de votre pipeline

Vous pouvez filtrer la liste des exécutions de pipeline en la limitant à celles qui vous intéressent. Les filtres en haut de l’écran vous permettent de spécifier un champ sur lequel filtrer.

Par exemple, vous pouvez filtrer l’affichage pour ne voir que les exécutions du pipeline nommé « holiday » :

![Exemple de filtre](./media/how-to-monitor-pipeline-runs/filter-example.png)

## <a name="view-details-about-a-specific-pipeline-run"></a>Afficher les détails relatifs à l’exécution d’un pipeline spécifique

Pour afficher les détails relatifs à l’exécution de votre pipeline, sélectionnez celle-ci. Affichez ensuite les exécutions d’activités associées à l’exécution du pipeline. Si le pipeline est toujours en cours d’exécution, vous pouvez surveiller la progression. 
  
## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la surveillance des applications, consultez l’article [Surveiller les applications Apache Spark](how-to-monitor-spark-applications.md). 
