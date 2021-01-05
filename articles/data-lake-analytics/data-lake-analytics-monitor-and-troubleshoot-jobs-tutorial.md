---
title: Superviser Azure Data Lake Analytics - Portail Azure
description: Cet article décrit comment dépanner des travaux Azure Data Lake Analytics dans le portail Azure.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: troubleshooting
ms.date: 12/05/2016
ms.openlocfilehash: ddcf41a333d261bd9c5f669fde724a25eeba670e
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92220276"
---
# <a name="monitor-jobs-in-azure-data-lake-analytics-using-the-azure-portal"></a>Effectuer le monitoring des tâches dans Azure Data Lake Analytics avec le Portail Azure

## <a name="to-see-all-the-jobs"></a>Pour voir tous les travaux

1. À partir du portail Azure, cliquez sur **Microsoft Azure** dans le coin supérieur gauche.

2. Cliquez sur la vignette indiquant le nom de votre compte Analytique Data Lake.  Le résumé du travail est affiché sur la vignette **Gestion des tâches** .

   ![Gestion du travail Analytique Data Lake Azure](./media/data-lake-analytics-monitor-and-troubleshoot-tutorial/data-lake-analytics-job-management.png)

    La gestion des tâches vous offre un aperçu de l’état du travail. Notez qu’il s’agit d’une tâche ayant échoué.
3. Cliquez sur la vignette **Gestion de la tâche** pour rechercher les travaux. Les travaux sont classés dans les catégories **En cours**, **En file d’attente** et **Terminé**. Vous devez voir le travail qui a échoué dans la section **Terminé** . Il doit être le premier sur la liste. Lorsque vous avez un grand nombre de travaux, vous pouvez cliquer sur **Filtrer** pour vous aider à localiser les travaux.

   ![Travail de filtre d’Analytique Data Lake Azure](./media/data-lake-analytics-monitor-and-troubleshoot-tutorial/data-lake-analytics-filter-jobs.png)

4. Cliquez sur le travail ayant échoué dans la liste pour ouvrir les détails du travail :

   ![Travail d’échec d’Analytique Data Lake Azure](./media/data-lake-analytics-monitor-and-troubleshoot-tutorial/data-lake-analytics-failed-job.png)

    Remarquez le bouton **Renvoyer** . Après avoir résolu le problème, vous pouvez renvoyer le travail.

5. Cliquez sur la partie mise en évidence de la capture d’écran précédente pour ouvrir les détails de l’erreur.  Le résultat suivant doit s’afficher :

   ![Détails sur le travail d’échec d’Analytique Data Lake Azure](./media/data-lake-analytics-monitor-and-troubleshoot-tutorial/data-lake-analytics-failed-job-details.png)

   Elle vous indique que le dossier source est introuvable.

6. Cliquez sur **Dupliquer le Script**.

7. Mettez à jour le chemin d’accès **FROM** sur : `/Samples/Data/SearchLog.tsv`

8. Cliquez sur **Envoyer le travail**.

## <a name="next-steps"></a>Étapes suivantes

* [Présentation d’Analytique Data Lake Azure](data-lake-analytics-overview.md)
* [Prise en main d’Analytique Data Lake à l’aide d’Azure PowerShell](data-lake-analytics-get-started-powershell.md)
* [Gestion d’Azure Data Lake Analytics à l’aide du portail Azure](data-lake-analytics-manage-use-portal.md)
