---
title: Présentation de la surveillance des tâches dans Azure Stream Analytics
description: Cet article décrit comment surveiller les travaux Azure Stream Analytics dans le Portail Azure.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 06/21/2018
ms.custom: seodec18
ms.openlocfilehash: f8e0d49c2a35a1e97c79e6d4b8c867a4a8ab88b3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006676"
---
# <a name="understand-stream-analytics-job-monitoring-and-how-to-monitor-queries"></a>Présentation de la surveillance des tâches Stream Analytics et des requêtes

## <a name="introduction-the-monitor-page"></a>Introduction : page de surveillance
Le portail Azure affiche les mesures de performances clés qui peuvent servir à surveiller et résoudre les problèmes affectant les performances de vos requêtes et de vos travaux. Pour voir ces métriques, accédez au travail Stream Analytics dont vous souhaitez consulter les métriques et affichez la section **Surveillance** dans la page Vue d’ensemble.  

![Lien vers la page de surveillance des tâches Azure Stream Analytics](./media/stream-analytics-monitoring/02-stream-analytics-monitoring-block.png)

Une fenêtre s’affiche comme suit :

![Tableau de bord de surveillance des tâches Azure Stream Analytics](./media/stream-analytics-monitoring/01-stream-analytics-monitoring.png)  

## <a name="metrics-available-for-stream-analytics"></a>Mesures disponibles pour Stream Analytics
| Métrique                 | Définition                               |
| ---------------------- | ---------------------------------------- |
| Événements d'entrée en backlog       | Nombre d’événements d’entrée qui sont en backlog. Une valeur non nulle pour cette métrique implique que votre travail n’est pas en mesure de suivre le nombre d’événements entrants. Si cette valeur est croissante ou constamment différente de zéro, vous devez effectuer un scale-out de votre travail. Vous trouverez plus d’informations en consultant [Comprendre et ajuster les unités de diffusion en continu](stream-analytics-streaming-unit-consumption.md). |
| Erreurs de conversion de données | Nombre d’événements de sortie qui n’ont pas pu être convertis dans le schéma de sortie attendu. La stratégie de l’erreur peut être modifiée sur 'Drop' pour supprimer les événements confrontés à ce scénario. |
| Événements d’entrée précoces       | Événements dont l’horodatage d’application est antérieure à leur heure d’arrivée de plus de 5 minutes. |
| Requêtes de fonction ayant échoué | Nombre d’appels à la fonction Azure Machine Learning ayant échoué (le cas échéant). |
| Événements de fonction        | Nombre d’événements envoyés à la fonction Azure Machine Learning (le cas échéant). |
| Requêtes de fonction      | Nombre d’appels à la fonction Azure Machine Learning (le cas échéant). |
| Erreurs de désérialisation d’entrée       | Nombre d’événements d’entrée qui n’ont pas pu être désérialisés.  |
| Octets des événements d’entrée      | Quantité de données reçues par le travail Stream Analytics, en octets. Cela permet de valider que les événements sont envoyés à la source d’entrée. |
| Événements d’entrée           | Nombre d’enregistrements désérialisé à partir des événements d’entrée. Ce nombre n’inclut pas les événements entrants qui génèrent des erreurs de désérialisation. Les mêmes événements peuvent être ingérés par Stream Analytics plusieurs fois dans des scénarios tels que des récupérations internes et des jointures réflexives. Par conséquent, il est recommandé de ne pas s’attendre à ce que les métriques d’événements d’entrée et de sortie correspondent si votre travail comprend une requête « directe » simple. |
| Sources d'entrée reçues       | Nombre de messages reçus par le travail. Pour Event Hub, un message est un EventData unique. Pour un objet blob, un message est un objet blob unique. Veuillez noter que les sources d’entrée sont comptabilisées avant la désérialisation. En cas d’erreurs de désérialisation, les sources d’entrée peuvent être supérieures aux événements d’entrée. Sinon, elles peuvent être inférieures ou égales aux événements d’entrée dans la mesure où chaque message peut contenir plusieurs événements. |
| Événements d’entrée tardifs      | Événements reçus plus tard que la valeur configurée dans la fenêtre de tolérance d’arrivée tardive. En savoir plus sur les [Considérations relatives à l’ordre des événements Azure Stream Analytics](./stream-analytics-time-handling.md). |
| Événements non ordonnés    | Nombre d’événements reçus dans le désordre qui ont été supprimés ou dont l’horodatage a été réglé, en fonction de la stratégie de classement des événements. Cela peut être affecté par la configuration du paramètre de la plage de tolérance pour les événements en désordre. |
| Événements de sortie          | Quantité de données envoyées par le travail Stream Analytics à la cible de sortie, en nombre d’événements. |
| Erreurs d’exécution         | Nombre total d’erreurs liées au traitement des requêtes (à l’exception des erreurs détectées lors de l’ingestion d’événements ou de la génération de résultats) |
| Utilisation de % d’unités de diffusion       | Si l’utilisation des ressources est régulièrement supérieure à 80 %, le délai en filigrane augmente; tout comme le nombre d’événements retardés. Dans ce cas, vous pouvez d’augmenter les unités de streaming. Une utilisation intensive indique que la tâche atteint une limite proche de la quantité maximale de ressources allouées. |
| Délai en filigrane       | Le délai en filigrane maximal sur toutes les partitions de toutes les sorties du travail. |

Vous pouvez utiliser ces mesures pour [surveiller les performances de votre travail Stream Analytics](./stream-analytics-set-up-alerts.md#scenarios-to-monitor). 

## <a name="customizing-monitoring-in-the-azure-portal"></a>Personnalisation de la surveillance dans le portail Azure
Vous pouvez régler le type de graphique, les mesures affichées et la période dans les paramètres Modifier le graphique. Pour plus d’informations, consultez [Personnalisation de la surveillance](../azure-monitor/platform/data-platform.md).

  ![Graphique représentant le temps de surveillance des requêtes Stream Analytics](./media/stream-analytics-monitoring/08-stream-analytics-monitoring.png)  


## <a name="latest-output"></a>Dernière sortie
Un autre point de données intéressant pour surveiller votre travail est l’heure de la dernière sortie, indiquée dans la page Vue d’ensemble.
Cette heure correspond à l’heure d’application (c’est-à-dire l’heure donnée via l’horodatage dans les données d’événement) de la dernière sortie de votre travail.

## <a name="get-help"></a>Obtenir de l’aide
Pour obtenir de l’aide supplémentaire, essayez notre [page de questions Microsoft Q&R sur Azure Stream Analytics](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Étapes suivantes
* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)
