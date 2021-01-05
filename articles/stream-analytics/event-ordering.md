---
title: Configuration de stratégies de classement des événements pour Azure Stream Analytics
description: Cet article explique comment configurer les paramètres de classement des événements dans Stream Analytics
author: sidram
ms.author: sidram
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 08/06/2020
ms.openlocfilehash: 80567a211f08d6322c80b6645f8b70ec7df64b59
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130661"
---
# <a name="configuring-event-ordering-policies-for-azure-stream-analytics"></a>Configuration de stratégies de classement des événements pour Azure Stream Analytics

Cet article explique comment configurer et utiliser des stratégies de classement des événements tardifs et désordonnés dans Azure Stream Analytics. Ces stratégies sont appliquées uniquement lorsque vous utilisez la clause [TIMESTAMP BY](/stream-analytics-query/timestamp-by-azure-stream-analytics) dans votre requête, et ne sont appliquées que pour des sources d’entrée cloud.

## <a name="event-time-and-arrival-time"></a>Heure de l’événement et heure d’arrivée

Votre tâche Stream Analytics peut traiter les événements en fonction de *heure de l’événement* ou de l’ *heure d’arrivée*. L’ **heure de l’application/événement** est l’horodatage présent dans la charge utile d’événement (moment auquel l’événement a été généré). L’ **heure d’arrivée** est l’horodatage du moment où l’événement a été reçu à la source d’entrée (Event Hubs/IoT Hub/Stockage Blob). 

Par défaut, Stream Analytics traite les événements par *heure d’arrivée* , mais vous pouvez choisir de les traiter par *heure d’événement* en utilisant la clause [TIMESTAMP BY](/stream-analytics-query/timestamp-by-azure-stream-analytics) dans votre requête. Les stratégies d’arrivée tardive et dans le désordre s’appliquent uniquement si vous traitez les événements en fonction de l’heure d’événement. Tenez compte des exigences de latence et d’exactitude au moment de configurer ces paramètres pour votre scénario. 

## <a name="what-is-late-arrival-policy"></a>Qu’est-ce qu’une stratégie d’arrivée tardive ?

Parfois, les événements arrivent en retard pour diverses raisons. Par exemple, si un événement arrive avec 40 secondes de retard et que l’heure de l’événement est 00:10:00, son heure d’arrivée sera 00:10:40. Si vous définissez la stratégie d’arrivée tardive sur 15 secondes, tout événement qui arrivera après 15 secondes sera soit supprimé (Stream Analytics ne le traitera pas) soit son heure d’événement sera ajustée. Dans l’exemple ci-dessus, l’événement est arrivé avec 40 secondes de retard (c’est-à-dire au-delà de la valeur définie de la stratégie). De ce fait, l’heure de l’événement est ajustée par rapport à la valeur maximale de la stratégie d’arrivée tardive, soit 00:10:25 (heure d’arrivée – valeur de la stratégie d’arrivée tardive). La valeur par défaut de la stratégie d’arrivée tardive est de 5 secondes.

## <a name="what-is-out-of-order-policy"></a>Qu’est-ce qu’une stratégie d’événements en désordre ? 

Les événements peuvent aussi arriver en désordre. Une fois que l’heure de l’événement a été ajustée en fonction de la stratégie d’arrivée tardive, vous pouvez aussi choisir de supprimer ou d’ajuster automatiquement les événements qui sont en désordre. Si vous définissez cette stratégie sur 8 secondes, les événements qui arrivent en désordre mais dans la fenêtre de 8 secondes sont réordonnés en fonction de l’heure de l’événement. Les événements qui arrivent après sont soit supprimés soit ajustés par rapport à la valeur maximale de la stratégie d’événements en désordre. La valeur par défaut de la stratégie d’événements en désordre est de 0 seconde. 

## <a name="adjust-or-drop-events"></a>Ajuster ou supprimer les événements

Si des événements arrivent en retard ou en désordre par rapport aux stratégies que vous avez configurées, vous pouvez soit supprimer ces événements (Stream Analytics ne les traite pas) soit ajuster leur heure d’événement.

Voyons comment ces stratégies fonctionnent à travers un exemple.
<br> **Stratégie d’arrivée tardive :** 15 secondes
<br> **Stratégie d’événements en désordre :** 8 secondes

| N ° d’événement | Heure de l’événement | Heure d’arrivée | System.Timestamp | Explication |
| --- | --- | --- | --- | --- |
| **1** | 00:10:00  | 00:10:40  | 00:10:25  | Événement arrivé en retard et en dehors du niveau de tolérance. L’heure de l’événement est donc ajustée par rapport à la tolérance maximale d’arrivée tardive.  |
| **2** | 00:10:30 | 00:10:41  | 00:10:30  | Événement arrivé en retard, mais dans le niveau de tolérance. L’heure de l’événement n’est donc pas ajustée.  |
| **3** | 00:10:42 | 00:10:42 | 00:10:42 | Événement arrivé à temps. Aucun ajustement n’est nécessaire.  |
| **4** | 00:10:38  | 00:10:43  | 00:10:38 | Événement arrivé en désordre, mais dans la tolérance de 8 secondes. L’heure de l’événement n’est donc pas ajustée. Pour des besoins analytiques, cet événement est considéré comme étant antérieur à l’événement numéro 4.  |
| **5** | 00:10:35 | 00:10:45  | 00:10:37 | Événement arrivé en désordre et en dehors de la tolérance de 8 secondes. Par conséquent, l’heure de l’événement est ajustée par rapport à la tolérance de désordre maximale. |

## <a name="can-these-settings-delay-output-of-my-job"></a>Ces paramètres peuvent-ils retarder la sortie de ma tâche ? 

Oui. Par défaut, la stratégie d’événements en désordre est définie sur zéro (00 minute et 00 seconde). Si vous modifiez la valeur par défaut, la première sortie de votre tâche est retardée de la durée correspondant à cette valeur (ou plus). 

Si l’une des partitions de vos entrées ne reçoit pas d’événements, attendez-vous à ce que votre sortie soit retardée de la durée correspondant à la valeur de la stratégie d’arrivée tardive. Pour en connaître la raison, lisez la section relative à l’erreur InputPartition. 

## <a name="i-see-lateinputevents-messages-in-my-activity-log"></a>Mon journal d’activité contient des messages LateInputEvents

Ces messages sont là pour vous informer que des événements sont arrivés en retard et qu’ils seront supprimés ou ajustés en fonction de votre configuration. Vous pouvez ignorer ces messages si vous avez configuré la stratégie d’arrivée tardive de façon appropriée. 

Voici un exemple de ce message : <br>
<code>
{"message Time":"2019-02-04 17:11:52Z","error":null,
"message":"First Occurred: 02/04/2019 17:11:48 | Resource Name: ASAjob | Message: Source 'ASAjob' had 24 data errors of kind 'LateInputEvent' between processing times '2019-02-04T17:10:49.7250696Z' and '2019-02-04T17:11:48.7563961Z'. Input event with application timestamp '2019-02-04T17:05:51.6050000' and arrival time '2019-02-04T17:10:44.3090000' was sent later than configured tolerance.","type":"DiagnosticMessage","correlation ID":"49efa148-4asd-4fe0-869d-a40ba4d7ef3b"} 
</code>

## <a name="i-see-inputpartitionnotprogressing-in-my-activity-log"></a>Je vois InputPartitionNotProgressing dans mon journal d’activité

Il est probable que votre source d’entrée (Event Hub/IoT Hub) comporte plusieurs partitions. Azure Stream Analytics génère une sortie pour l’horodatage t1 une fois seulement que toutes les partitions combinées parviennent au moins au moment t1. Par exemple, supposons que la requête lit une partition de hub d’événements qui a deux partitions. L’une des partitions, P1, a des événements jusqu’au moment t1. L’autre partition, P2, a des événements jusqu’au moment t1 + x. Dans ce cas, la sortie est générée jusqu’au moment t1. Cependant, s’il existe une clause Partition by PartitionId explicite, les deux partitions progressent indépendamment. 

Quand plusieurs partitions du même flux d’entrée sont combinées, la tolérance d’arrivée tardive correspond au délai maximal d’attente de nouvelles données pour chaque partition. S’il existe une partition dans votre hub d’événements ou si IoT Hub ne reçoit pas d’entrées, la chronologie de cette partition ne progresse pas tant qu’elle n’atteint pas le seuil de tolérance d’arrivée tardive. Cela retarde votre sortie de la durée correspondant au seuil de tolérance d’arrivée tardive. En pareil cas, il se peut que le message d’erreur suivant s’affiche :
<br><code>
{"message Time":"2/3/2019 8:54:16 PM UTC","message":"Input Partition [2] does not have additional data for more than [5] minute(s). Partition will not progress until either events arrive or late arrival threshold is met.","type":"InputPartitionNotProgressing","correlation ID":"2328d411-52c7-4100-ba01-1e860c757fc2"} 
</code><br><br>
Ce message vous informe qu’au moins une partition de votre entrée est vide et que votre sortie sera retardée de la durée correspondant au seuil d’arrivée tardive. Pour pallier ce problème, suivez l’une ou l’autre de ces recommandations :  
1. Vérifiez que toutes les partitions de votre hub d’événements/hub IoT reçoit une entrée. 
2. Utilisez une clause Partition by PartitionID dans votre requête. 

## <a name="why-do-i-see-a-delay-of-5-seconds-even-when-my-late-arrival-policy-is-set-to-0"></a>Pourquoi est-ce que je vois un délai de 5 secondes même lorsque ma stratégie d’arrivée tardive est définie sur 0 ?
Cela se produit lorsqu’il existe une partition d’entrée qui n’a jamais reçu d’entrée. Vous pouvez vérifier les métriques d’entrée par partition pour valider ce comportement. 

Quand une partition n’a pas de données pendant une durée supérieure au seuil d’arrivée tardive configuré, Stream Analytics avance l’horodatage de l’application, comme expliqué dans la section Considérations relatives à l’organisation des événements. Cela nécessite une estimation de l’heure d’arrivée. Si la partition n’a jamais eu de données, Stream Analytics estime l’heure d’arrivée comme suit : *heure locale - 5 secondes*. Par conséquent, ces partitions qui n’ont jamais eu de données peuvent afficher un délai en filigrane de 5 secondes.  

## <a name="next-steps"></a>Étapes suivantes
* [Considérations relatives à la gestion du temps](stream-analytics-time-handling.md)
* [Mesures disponibles dans Stream Analytics](./stream-analytics-monitoring.md#metrics-available-for-stream-analytics)