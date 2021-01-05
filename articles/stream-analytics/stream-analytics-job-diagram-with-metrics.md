---
title: Débogage piloté par les données dans Azure Stream Analytics
description: Cet article décrit comment résoudre les problèmes liés à votre travail Azure Stream Analytics en utilisant le diagramme de travail et les mesures du portail Azure.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 05/01/2017
ms.openlocfilehash: e4ee868125ee30d40ef0d9ca5a42881ad003e9a8
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93123980"
---
# <a name="data-driven-debugging-by-using-the-job-diagram"></a>Débogage piloté par les données à l’aide du diagramme de travail

Le diagramme de travail dans le panneau **Surveillance** sur le portail Azure peut vous aider à visualiser votre pipeline de travail. Il montre les entrées, les sorties et les étapes de requête. Vous pouvez utiliser le diagramme de travail pour examiner les mesures pour chaque étape, afin d’isoler plus rapidement la source d’un problème lors de la résolution de problèmes.

## <a name="using-the-job-diagram"></a>Utilisation du diagramme de travail

Dans le portail Azure, à partir d’un travail Stream Analytics, sous **SUPPORT + DÉPANNAGE** , sélectionnez **Diagramme de travail**  :

![Diagramme de travail avec des mesures - emplacement](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-1.png)

Sélectionnez chaque étape de requête pour afficher la section correspondante dans un volet d’édition de requête. Un graphique de mesures pour l’étape s’affiche dans un volet inférieur de la page.

![Diagramme de travail avec des mesures - travail de base](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-2.png)

Pour afficher les partitions de l’entrée Azure Event Hubs, sélectionnez **...** . Un menu contextuel s’affiche. Vous pouvez également voir la fusion d’entrée.

![Diagramme de travail avec des mesures - étendre une partition](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-3.png)

Pour afficher le graphique de mesures pour une seule partition, sélectionnez le nœud de la partition. Les mesures sont affichées en bas de la page.

![Diagramme de travail avec des mesures - mesures supplémentaires](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-4.png)

Pour afficher le graphique de mesures pour une fusion, sélectionnez le nœud de fusion. Le graphique suivant montre qu’aucun événement n’a été supprimé ou ajusté.

![Diagramme de travail avec des mesures - grille](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-5.png)

Pour afficher les informations relatives à la valeur et à l’heure de la mesure, pointez sur le graphique.

![Diagramme de travail avec des mesures - pointer](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-6.png)

## <a name="troubleshoot-by-using-metrics"></a>Résoudre les problèmes à l’aide de mesures

La mesure **QueryLastProcessedTime** indique quand une étape spécifique a reçu des données. En examinant la topologie, vous pouvez respecter une séquence inverse depuis le processeur de sortie afin d’identifier l’étape qui ne reçoit pas les données. Si une étape ne reçoit pas de données, passez à l’étape de requête qui précède. Vérifiez si l’étape précédente de la requête dispose d’une fenêtre de temps et si suffisamment de temps s’est écoulé pour afficher les données. (Notez que les fenêtres de temps sont alignées sur l’heure.)
 
Si l’étape de requête précédente est un processeur d’entrée, utilisez les mesures d’entrée afin de répondre plus facilement aux questions ciblées. Elles peuvent vous aider à déterminer si un travail récupère des données à partir de ses sources d’entrée. Si la requête est partitionnée, examinez chaque partition.
 
### <a name="how-much-data-is-being-read"></a>Quelle est la quantité de données lue ?

*   **InputEventsSourcesTotal** correspond au nombre d’unités de données lues. Par exemple, le nombre d’objets blob.
*   **InputEventsTotal** correspond au nombre d’événements lus. Cette mesure est disponible par partition.
*   **InputEventsInBytesTotal** correspond au nombre d’octets lus.
*   **InputEventsLastArrivalTime** est mis à jour avec chaque durée de file d’attente des événements reçus.
 
### <a name="is-time-moving-forward-if-actual-events-are-read-punctuation-might-not-be-issued"></a>La chronologie progresse-t-elle ? Si des événements réels sont lus, il est possible qu’aucune ponctuation ne soit émise.

*   **InputEventsLastPunctuationTime** indique qu’une ponctuation a été émise afin de maintenir la progression chronologique. Si la ponctuation n’est pas émise, le flux de données peut être bloqué.
 
### <a name="are-there-any-errors-in-the-input"></a>La sortie contient-elle des erreurs ?

*   **InputEventsEventDataNullTotal** correspond au nombre d’événements présentant des données nulles.
*   **InputEventsSerializerErrorsTotal** correspond au nombre d’événements qui n’ont pas pu être désérialisés correctement.
*   **InputEventsDegradedTotal** correspond au nombre d’événements présentant un problème autre que la désérialisation.
 
### <a name="are-events-being-dropped-or-adjusted"></a>Les événements sont-ils supprimés ou modifiés ?

*   **InputEventsEarlyTotal** correspond au nombre d’événements présentant un horodatage d’application antérieur à la limite supérieure.
*   **InputEventsLateTotal** correspond au nombre d’événements présentant un horodatage d’application postérieur à la limite supérieure.
*   **InputEventsDroppedBeforeApplicationStartTimeTotal** fournit le nombre d’événements abandonnés avant l’heure de début du travail.
 
### <a name="are-we-falling-behind-in-reading-data"></a>Sommes-nous en retard en matière de lecture des données ?

*   **Événements d’entrée en backlog (total)** indique le nombre de messages supplémentaires devant être lus pour les entrées Event Hub et Azure IoT Hub. Lorsque ce nombre est supérieur à 0, cela signifie que votre travail ne peut pas traiter les données aussi rapidement qu’elles arrivent. Dans ce cas, vous devrez peut-être augmenter le nombre d’unités de streaming et/ou de vous assurer que votre travail peut être parallélisée. Vous pouvez voir plus d’informations à ce sujet à la [page de parallélisation des requêtes](./stream-analytics-parallelization.md). 


## <a name="get-help"></a>Obtenir de l’aide
Pour obtenir une assistance supplémentaires, consultez notre [page de questions Microsoft Q&R sur Azure Stream Analytics](/answers/topics/azure-stream-analytics.html). 

## <a name="next-steps"></a>Étapes suivantes
* [Présentation de Stream Analytics](stream-analytics-introduction.md)
* [Prise en main de Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Mise à l’échelle des travaux Stream Analytics](stream-analytics-scale-jobs.md)
* [Informations de référence sur le langage de requête Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion de Stream Analytics](/rest/api/streamanalytics/)