---
title: Tester des données actives avec Azure Stream Analytics pour Visual Studio
description: Découvrez comment tester votre tâche Azure Stream Analytics localement à l’aide de données de streaming en direct.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/07/2018
ms.custom: seodec18
ms.openlocfilehash: 0788d8b4f659364fcedc690185c9159a24434125
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93123912"
---
# <a name="test-live-data-locally-using-azure-stream-analytics-tools-for-visual-studio-preview"></a>Tester des données actives localement à l’aide des outils Azure Stream Analytics pour Visual Studio (préversion)

Les outils Azure Stream Analytics pour Visual Studio vous permettent de tester localement des tâches à partir de l’IDE à l’aide de flux d’événements en direct à partir d’Azure Event Hub, d’IoT Hub et de Stockage Blob. Le test local des données actives ne peut pas remplacer le [test de performances et de scalabilité](stream-analytics-streaming-unit-consumption.md) que vous pouvez effectuer dans le cloud, mais vous pouvez gagner du temps pendant les tests fonctionnels en n’ayant pas à effectuer d’envoi dans le cloud chaque fois que vous voulez tester votre tâche Stream Analytics. Cette fonctionnalité est en préversion et ne doit pas être utilisée pour les charges de travail en production.

## <a name="testing-options"></a>Options de test

Les options de test local suivantes sont prises en charge :

|**Input**  |**Sortie**  |**Type de tâche**  |
|---------|---------|---------|
|Données statiques locales   |  Données statiques locales   |   Cloud/Edge |
|Données d’entrée actives   |  Données statiques locales   |   Cloud |
|Données d’entrée actives   |  Données de sortie actives   |   Cloud |

## <a name="local-testing-with-live-data"></a>Test local avec des données actives

1. Une fois que vous avez créé un [projet cloud Azure Stream Analytics dans Visual Studio](stream-analytics-quick-create-vs.md), ouvrez **script.asaql**. Par défaut, le test local utilise l’entrée locale et la sortie locale.

   ![Entrée locale et sortie locale Visual Studio Azure Stream Analytics](./media/stream-analytics-live-data-local-testing/stream-analytics-local-testing-local-input-output.png)

2. Pour tester des données actives, choisissez **Utiliser l’entrée dans le cloud** dans la zone de liste déroulante.

   ![Entrée de cloud active Visual Studio Azure Stream Analytics](./media/stream-analytics-live-data-local-testing/stream-analytics-local-testing-cloud-input.png)

3. Définissez l’option **Heure de début** pour définir le moment où la tâche commence à traiter les données d’entrée. La tâche devra peut-être lire les données d’entrée à l’avance pour garantir des résultats précis. La durée par défaut est définie sur 30 minutes avant l’heure actuelle.

   ![Heure de démarrage des données actives Visual Studio Azure Stream Analytics](./media/stream-analytics-live-data-local-testing/stream-analytics-local-testing-cloud-input-start-time.png)

4. Cliquez sur **Exécuter localement**. Une fenêtre de console s’affiche avec la progression de l’exécution et les métriques de la tâche. Si vous voulez arrêter le processus, vous pouvez le faire manuellement. 

   ![Fenêtre de traitement des données actives Visual Studio Azure Stream Analytics](./media/stream-analytics-live-data-local-testing/stream-analytics-local-testing-cloud-input-process-window.png)

   Les résultats de sortie sont actualisés toutes les trois secondes avec les 500 premières lignes de sortie dans la fenêtre de résultat de l’exécution locale, et les fichiers de sortie sont placés dans le dossier **ASALocalRun** du chemin de votre projet. Vous pouvez également ouvrir les fichiers de sortie en cliquant sur le bouton **Ouvrir le dossier de résultats** dans la fenêtre de résultat de l’exécution locale.

   ![Ouvrir le dossier de résultats pour des données actives Visual Studio Azure Stream Analytics](./media/stream-analytics-live-data-local-testing/stream-analytics-local-testing-cloud-input-open-results-folder.png)

5. Si vous voulez générer les résultats dans vos récepteurs de sortie cloud, choisissez **Sortie vers le cloud** dans la deuxième zone déroulante. Power BI et Azure Data Lake Storage ne sont pas des récepteurs de sortie pris en charge.

   ![Sortie de données actives vers le cloud Visual Studio Azure Stream Analytics](./media/stream-analytics-live-data-local-testing/stream-analytics-local-testing-cloud-output.png)
 
## <a name="limitations"></a>Limites

* Power BI et Azure Data Lake Storage ne sont pas des récepteurs de sortie pris en charge en raison des limitations du modèle d’authentification.

* Seules les options d’entrée dans le cloud offrent une prise en charge de [stratégies de temps](./stream-analytics-time-handling.md) ; les options d’entrée locale n’en proposent pas.

## <a name="next-steps"></a>Étapes suivantes

* [Créer une tâche Stream Analytics à l’aide d’Azure Stream Analytics Tools pour Visual Studio](stream-analytics-quick-create-vs.md)
* [Tester des requêtes Stream Analytics localement avec Visual Studio](stream-analytics-vs-tools-local-run.md)
* [Utiliser Visual Studio pour afficher les tâches Azure Stream Analytics](stream-analytics-vs-tools.md)