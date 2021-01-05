---
title: Déboguer des requêtes Azure Stream Analytics en local à l’aide du diagramme de travail dans Visual Studio Code
description: Cet article décrit comment déboguer des requêtes localement à l’aide du diagramme de travail dans les extensions Azure Stream Analytics pour Visual Studio Code.
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 06/23/2020
ms.openlocfilehash: 766d3f8d4d4c03fb4fe173dbb12eb85bb09cd71a
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93124167"
---
# <a name="debug-azure-stream-analytics-queries-locally-using-job-diagram-in-visual-studio-code"></a>Déboguer des requêtes Azure Stream Analytics en local à l’aide du diagramme de travail dans Visual Studio Code

Les travaux de diffusion en continu qui ne génèrent aucun résultat ou des résultats inattendus nécessitent souvent un dépannage. L’extension de Visual Studio Code pour Azure Stream Analytics intègre les diagrammes de travaux, les métriques, les journaux de diagnostic et les résultats intermédiaires pour vous aider à isoler rapidement la source d’un problème. Vous pouvez utiliser le diagramme de travail lorsque vous testez votre requête localement pour examiner le jeu de résultats intermédiaire et les métriques pour chaque étape.

## <a name="debug-a-query-using-job-diagram"></a>Déboguer une requête à l’aide du diagramme de travail

Un script Azure Stream Analytics est utilisé pour transformer des données d’entrée en données de sortie. Le diagramme de la tâche montre la façon dont les données circulent des sources d’entrée (Event Hub, IoT Hub, etc.), via plusieurs étapes de requête jusqu’aux récepteurs de sortie. Chaque étape de la requête est mappée à un jeu de résultats temporaire défini dans le script à l’aide d’une instruction `WITH`. Vous pouvez afficher les données et les métriques de chaque étape de la requête dans chaque jeu de résultats intermédiaire pour trouver la source d’un problème.

> [!NOTE]
> Ce diagramme de travail affiche uniquement les données et les métriques pour les tests locaux dans un nœud unique. Il ne doit pas être utilisé pour le réglage des performances et la résolution des problèmes.

### <a name="start-local-testing"></a>Commencer les tests locaux

Utilisez ce [Démarrage rapide](quick-create-visual-studio-code.md) pour apprendre à créer un travail Stream Analytics à l’aide de Visual Studio Code ou [exporter un travail existant vers un projet local](visual-studio-code-explore-jobs.md). Les informations d’identification pour les entrées et les sorties sont automatiquement remplies pour les travaux exportés.

Si vous souhaitez tester la requête avec des données d’entrée locales, suivez ces [instructions](visual-studio-code-local-run.md). Si vous souhaitez effectuer un test avec une entrée dynamique, [configurez votre entrée](stream-analytics-add-inputs.md) et passez à l’étape suivante. 

Ouvrez le fichier de script *\.asaql* et sélectionnez **Exécuter localement**. Ensuite, sélectionnez **Utiliser une entrée locale** ou **Utiliser une entrée en direct**. Le diagramme de tâche apparaît sur le côté droit de la fenêtre.

### <a name="view-the-output-and-intermediate-result-set"></a>Afficher la sortie et le jeu de résultats intermédiaire  

1. Toutes les sorties de tâche s’affichent dans la fenêtre de résultats en bas à droite de la fenêtre de Visual Studio Code.

   > [!div class="mx-imgBorder"]
   > ![Résultats de sortie de la tâche](./media/debug-locally-using-job-diagram-vs-code/job-output-results.png)

2. Sélectionnez l’étape de la requête pour accéder au script. Vous êtes automatiquement dirigé vers le script correspondant dans l’éditeur sur la gauche. Les résultats intermédiaires s’affichent dans la fenêtre de résultats en bas à droite de la fenêtre de Visual Studio Code.

   > [!div class="mx-imgBorder"]
   > ![Aperçu des résultats dans le diagramme de travail](./media/debug-locally-using-job-diagram-vs-code/preview-result.png)

### <a name="view-metrics"></a>Afficher les mesures

Dans cette section, vous explorez les métriques disponibles pour chaque partie du diagramme.

1. Sélectionnez l’onglet **Métriques** en regard de l’onglet **Résultats** situé en bas à droite de la fenêtre Visual Studio Code.

2. Dans la liste déroulante, sélectionnez **Tâche**. Vous pouvez sélectionner n’importe quel espace vide dans un nœud de graphique pour accéder aux métriques au niveau de la tâche. Cette vue contient toutes les métriques, qui sont mises à jour toutes les 10 secondes lorsque la tâche est en cours d’exécution. Vous pouvez sélectionner ou désélectionner les métriques sur le côté droit pour les afficher dans les graphiques.

   > [!div class="mx-imgBorder"]
   > ![Métriques du diagramme de la tâche](./media/debug-locally-using-job-diagram-vs-code/job-metrics.png)

3. Sélectionnez le nom de la source de données d’entrée dans la liste déroulante pour afficher les métriques d’entrée. La source d’entrée dans la capture d’écran ci-dessous est appelée *quotes*. Pour plus d’informations sur les métriques d’entrée, consultez [Présentation de la supervision des travaux Stream Analytics et des requêtes](stream-analytics-monitoring.md).

   > [!div class="mx-imgBorder"]
   > ![Métriques d’entrée du diagramme de tâche](./media/debug-locally-using-job-diagram-vs-code/input-metrics.png)

4. Sélectionnez une étape de requête dans le diagramme de tâche ou sélectionnez le nom de l’étape dans la liste déroulante pour afficher les métriques au niveau de l’étape. Le délai en filigrane est la seule mesure de l’étape disponible.

   > [!div class="mx-imgBorder"]
   > ![Métriques de l’étape](./media/debug-locally-using-job-diagram-vs-code/step-metrics.png)

5. Sélectionnez une sortie dans le diagramme ou dans la liste déroulante pour afficher les métriques liées à la sortie. Pour plus d’informations sur les métriques de sortie, consultez [Présentation de la supervision des travaux Stream Analytics et des requêtes](stream-analytics-monitoring.md). Les récepteurs de sortie dynamique ne sont pas pris en charge.

   > [!div class="mx-imgBorder"]
   > ![Métriques de sortie](./media/debug-locally-using-job-diagram-vs-code/output-metrics.png)

### <a name="view-diagnostic-logs"></a>Afficher les journaux de diagnostic

Les journaux de diagnostic au niveau de la tâche contiennent des informations de diagnostic pour les sources de données d'entrée et les récepteurs de sortie. Lorsque vous sélectionnez un nœud d’entrée ou de sortie, seuls les journaux correspondants sont affichés. Aucun journal n’est affiché si vous sélectionnez une étape de requête. Vous pouvez trouver tous les journaux au niveau de la tâche, et les filtrer par gravité et par temps.

   > [!div class="mx-imgBorder"]
   > ![Journaux de diagnostic](./media/debug-locally-using-job-diagram-vs-code/diagnostic-logs.png)

   Sélectionnez une entrée de journal pour afficher l’intégralité du message.

   > [!div class="mx-imgBorder"]
   > ![Message des journaux de diagnostic](./media/debug-locally-using-job-diagram-vs-code/diagnostic-logs-message.png)


## <a name="other-job-diagram-features"></a>Autres fonctionnalités du diagramme de tâche

Vous pouvez sélectionner **Arrêter** ou **Suspendre** à partir de la barre d’outils en fonction de vos besoins. Une fois la tâche suspendue, vous pouvez la reprendre à partir de la dernière sortie.

> [!div class="mx-imgBorder"]
> ![Arrêter ou suspendre la tâche](./media/debug-locally-using-job-diagram-vs-code/stop-pause-job.png)

Sélectionnez **Résumé de la tâche** en haut à droite du diagramme pour afficher les propriétés et les configurations de votre tâche locale.

> [!div class="mx-imgBorder"]
> ![Résumé de la tâche locale](./media/debug-locally-using-job-diagram-vs-code/job-summary.png)

## <a name="limitations"></a>Limites

* Les récepteurs de sortie dynamique ne sont pas pris en charge dans l’exécution locale.

* Exécuter une tâche en local avec la fonction JavaScript n'est pris en charge que par le système d'exploitation Windows.

* Le code personnalisé C# et les fonctions Azure Machine Learning ne sont pas pris en charge. 

* Seules les options d’entrée dans le cloud offrent une prise en charge de [stratégies de temps](./stream-analytics-time-handling.md) ; les options d’entrée locale n’en proposent pas.

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Créer un travail Stream Analytics à l'aide de Visual Studio Code](quick-create-visual-studio-code.md)
* [Explorer Azure Stream Analytics avec Visual Studio Code](visual-studio-code-explore-jobs.md)
* [Tester des requêtes Stream Analytics localement avec des exemples de données à l’aide de Visual Studio Code](visual-studio-code-local-run.md)
* [Tester des travaux Azure Stream Analytics localement avec une entrée en direct à l’aide de Visual Studio Code](visual-studio-code-local-run-live-input.md)