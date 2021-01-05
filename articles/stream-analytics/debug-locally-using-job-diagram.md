---
title: Déboguer des requêtes Azure Stream Analytics en local à l’aide du diagramme de travail dans Visual Studio
description: Cet article décrit comment déboguer des requêtes localement à l’aide du diagramme de travail dans les outils Azure Stream Analytics pour Visual Studio.
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 01/23/2020
ms.openlocfilehash: c32ba468979e4566d8e349d481eddca7d407b659
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130984"
---
# <a name="debug-azure-stream-analytics-queries-locally-using-job-diagram-in-visual-studio"></a>Déboguer des requêtes Azure Stream Analytics en local à l’aide du diagramme de travail dans Visual Studio

Les travaux qui ne produisent aucun résultat ou des résultats inattendus sont des scénarios courants de résolution des problèmes pour les requêtes de diffusion en continu. Vous pouvez utiliser le diagramme de travail lorsque vous testez votre requête localement dans Visual Studio pour examiner le jeu de résultats intermédiaire et les métriques pour chaque étape. Les diagrammes de travail peuvent vous aider à isoler rapidement la source d’un problème lorsque vous détectez un problème.

## <a name="debug-a-query-using-job-diagram"></a>Déboguer une requête à l’aide du diagramme de travail

Un script Azure Stream Analytics est utilisé pour transformer des données d’entrée en données de sortie. Le diagramme de travail montre la façon dont les données circulent des sources d’entrée (Event Hub, IoT Hub, etc.), via plusieurs étapes de requête et, enfin, jusqu’aux récepteurs de sortie. Chaque étape de la requête est mappée à un jeu de résultats temporaire défini dans le script à l’aide d’une instruction `WITH`. Vous pouvez afficher les données et les métriques de chaque étape de la requête dans chaque jeu de résultats intermédiaire pour trouver la source d’un problème.

> [!NOTE]
> Ce diagramme de travail affiche uniquement les données et les métriques pour les tests locaux dans un nœud unique. Il ne doit pas être utilisé pour le réglage des performances et la résolution des problèmes.

### <a name="start-local-testing"></a>Commencer les tests locaux

Utilisez ce [Démarrage rapide](stream-analytics-quick-create-vs.md) pour apprendre à créer un travail Stream Analytics à l’aide de Visual Studio ou [exporter un travail existant vers un projet local](stream-analytics-vs-tools.md#export-jobs-to-a-project). Si vous souhaitez tester la requête avec des données d’entrée locales, suivez ces [instructions](stream-analytics-live-data-local-testing.md). Si vous souhaitez effectuer un test avec une entrée dynamique, passez à l’étape suivante.

> [!NOTE]
> Si vous exportez un travail vers un projet local et que vous souhaitez effectuer un test par rapport à un flux d’entrée dynamique, vous devez spécifier à nouveau les informations d’identification pour toutes les entrées.  

Choisissez la source d’entrée et de sortie dans l’éditeur de script et sélectionnez **Exécuter localement**. Le diagramme de travail apparaît sur le côté droit.

### <a name="view-the-intermediate-result-set"></a>Afficher le jeu de résultats intermédiaire  

1. Sélectionnez l’étape de la requête pour accéder au script. Vous êtes automatiquement dirigé vers le script correspondant dans l’éditeur sur la gauche.

   ![Accéder au script dans le diagramme de travail](./media/debug-locally-using-job-diagram/navigate-script.png)

2. Sélectionnez l’étape de la requête, puis **Aperçu** dans la boîte de dialogue qui apparaît. Le jeu de résultats s’affiche dans un onglet de la fenêtre de résultats inférieure.

   ![Aperçu des résultats dans le diagramme de travail](./media/debug-locally-using-job-diagram/preview-result.png)

### <a name="view-step-metrics"></a>Afficher les métriques de l’étape

Dans cette section, vous explorez les métriques disponibles pour chaque partie du diagramme.

#### <a name="input-sources-live-stream"></a>Sources d’entrée (stream en direct)

![Sources d’entrée dynamique dans le diagramme de travail](./media/debug-locally-using-job-diagram/live-input.png)

|Métrique|Description|
|-|-|
|**TaxiRide**| Nom de l'entrée.|
|**Concentrateur d’événements** | Type de source d’entrée.|
|**Événements**|Nombre d’événements lus.|
|**Sources d’événements en backlog**|Nombre de messages supplémentaires devant être lus pour les entrées Event Hubs et IoT Hub.|
|**Événements en octets**|Nombre d'octets lus.|
| **Événements détériorés**|Nombre d’événements présentant un problème autre que la désérialisation.|
|**Événements précoces**| Nombre d’événements présentant un horodatage d’application antérieur à la limite supérieure.|
|**Événements tardifs**| Nombre d’événements présentant un horodatage d’application postérieur à la limite supérieure.|
|**Sources d’événement**| Nombre d’unités de données lues. Par exemple, le nombre d’objets blob.|

#### <a name="input-sources-local-input"></a>Sources d’entrée (entrée locale)

![Sources d’entrée locale dans le diagramme de travail](./media/debug-locally-using-job-diagram/local-input.png)

|Métrique|Description|
|-|-|
|**TaxiRide**| Nom de l'entrée.|
|**Nombre de lignes**| Nombre de lignes générées à partir de l’étape.|
|**Taille des données**| Taille des données générées à partir de cette étape.|
|**Entrée locale**| Utilise les données locales comme entrée.|

#### <a name="query-steps"></a>Étapes de la requête

![Étape de la requête dans le diagramme de travail](./media/debug-locally-using-job-diagram/query-step.png)

|Métrique|Description|
|-|-|
|**TripData**|Nom du jeu de résultats temporaire.|
|**Nombre de lignes**| Nombre de lignes générées à partir de l’étape.|
|**Taille des données**| Taille des données générées à partir de cette étape.|
  
#### <a name="output-sinks-live-output"></a>Récepteurs de sortie (sortie dynamique)

![Diagramme de travail représentant les récepteurs de sortie locale.](./media/debug-locally-using-job-diagram/live-output.png)

|Métrique|Description|
|-|-|
|**regionaggEH**|Nom de la sortie.|
|**Événements**|Nombre d’événements générés par les récepteurs.|

#### <a name="output-sinks-local-output"></a>Récepteurs de sortie (sortie locale)

![Récepteurs de sortie locale dans le diagramme de travail](./media/debug-locally-using-job-diagram/local-output.png)

|Métrique|Description|
|-|-|
|**regionaggEH**|Nom de la sortie.|
|**Sortie locale**| Résultat obtenu dans un fichier local.|
|**Nombre de lignes**| Nombre de lignes sorties dans le fichier local.|
|**Taille des données**| Taille de la sortie des données dans le fichier local.|

### <a name="close-job-diagram"></a>Fermer le diagramme de travail

Si vous n’avez plus besoin du diagramme de travail, sélectionnez **Fermer** dans le coin supérieur droit. Après avoir fermé la fenêtre du diagramme, vous devez recommencer le test local pour l’afficher.

### <a name="view-job-level-metrics-and-stop-running"></a>Afficher les métriques au niveau du travail et arrêter l’exécution

D’autres métriques au niveau du travail s’affichent dans la console contextuelle. Appuyez sur **Ctrl+C** dans la console si vous souhaitez arrêter le travail.

![Arrêter le travail dans le diagramme de travail](./media/debug-locally-using-job-diagram/stop-job.png)

## <a name="limitations"></a>Limites

* Les récepteurs de sortie Power BI et Azure Data Lake Storage Gen1 ne sont pas pris en charge en raison des limitations du modèle d’authentification.

* Seules les options d’entrée dans le cloud offrent une prise en charge de [stratégies de temps](./stream-analytics-time-handling.md) ; les options d’entrée locale n’en proposent pas.

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Créer un travail Stream Analytics à l'aide de Visual Studio](stream-analytics-quick-create-vs.md)
* [Utiliser Visual Studio pour afficher les tâches Azure Stream Analytics](stream-analytics-vs-tools.md)
* [Tester des données actives localement à l’aide des outils Azure Stream Analytics pour Visual Studio (préversion)](stream-analytics-live-data-local-testing.md)