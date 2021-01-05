---
title: Informations de référence sur l’analyse des données Azure IoT Hub
description: Matériel de référence important nécessaire pour analyser Azure IoT Hub
author: robinsh
ms.author: robinsh
ms.topic: reference
ms.service: iot-hub
ms.date: 10/22/2020
ms.openlocfilehash: 609c44991e7deca183321758fabdef80b71210c5
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460850"
---
# <a name="monitoring-azure-iot-hub-data-reference"></a>Informations de référence sur l’analyse des données Azure IoT Hub

Pour plus d’informations sur la collecte et l’analyse des données de supervision concernant Azure IoT Hub, consultez [Analyse d’Azure IoT Hub](monitor-iot-hub.md).

## <a name="metrics"></a>Mesures

Cette section répertorie toutes les métriques de plateforme collectées automatiquement pour Azure IoT Hub. L’espace de noms du fournisseur de ressources pour les métriques IoT Hub est **Microsoft.Devices** et l’espace de noms de type est **IoTHubs**.

Les sous-sections suivantes présentent les métriques de la plateforme IoT Hub par catégorie générale et les répertorient en fonction du nom d’affichage sous lequel elles apparaissent dans le portail Azure. Des informations sont également fournies concernant les métriques qui apparaissent dans chaque sous-section.

Vous pouvez également trouver une table unique qui répertorie toutes les métriques de la plateforme IoT Hub par nom de mesure sous [Microsoft.Devices/IotHubs](../azure-monitor/platform/metrics-supported.md#microsoftdevicesiothubs) dans la documentation Azure Monitor. Sachez que cette table ne fournit pas certaines des informations, comme [les agrégations prises en charge](#supported-aggregations) pour certaines métriques, disponibles dans cet article.

Pour en savoir plus sur les métriques prises en charge par d’autres services Azure, consultez [Métriques prises en charge avec Azure Monitor](../azure-monitor/platform/metrics-supported.md).

**Rubriques de cette section**

- [Agrégations prises en charge](#supported-aggregations)
- [Métriques de commande cloud à appareil](#cloud-to-device-command-metrics)
- [Métriques de méthodes directes cloud à appareil](#cloud-to-device-direct-methods-metrics)
- [Métriques d’opérations jumelles cloud à appareil](#cloud-to-device-twin-operations-metrics)
- [Métriques des configurations](#configurations-metrics)
- [Métriques de quota quotidien](#daily-quota-metrics)
- [Mesures de l’appareil](#device-metrics)
- [Métriques de télémétrie d’appareil](#device-telemetry-metrics)
- [Métriques d’opérations jumelles appareil à cloud](#device-to-cloud-twin-operations-metrics)
- [Métriques de grille d’événements](#event-grid-metrics)
- [Métriques de travaux](#jobs-metrics)
- [Métriques de routage](#routing-metrics)
- [Métriques de requête de jumeau](#twin-query-metrics)

### <a name="supported-aggregations"></a>Agrégations prises en charge

La colonne **Type d’agrégation** dans chaque tableau correspond à l’agrégation par défaut qui est utilisée lorsque la métrique est sélectionnée pour un graphique ou une alerte.

   ![Capture d’écran montrant l’agrégation des métriques](./media/monitor-iot-hub-reference/aggregation-type.png)

Pour la plupart des métriques, tous les types d’agrégation sont valides ; toutefois, pour les métriques de comptage, celles dont la valeur de la colonne **Unité** est **Comptage**, seules certaines agrégations sont valides. Les métriques de comptage peuvent être de deux types :

* Pour les métriques de comptage **à point unique**, IoT Hub enregistre un point de données unique (essentiellement 1) chaque fois que l’opération mesurée se produit. Azure Monitor additionne ensuite ces points de données sur la granularité spécifiée. *Messages de télémétrie envoyés* et *Remises de messages cloud vers appareil terminées* sont des exemples de métriques **à point unique**. Pour ces métriques, le seul type d’agrégation pertinent est Total (Somme). Le portail vous permet de choisir la valeur minimale, la valeur maximale et la moyenne. Toutefois, ces valeurs sont toujours 1.

* Pour les métriques de comptage d’**instantanés**, IoT Hub enregistre un nombre total lorsque l’opération mesurée se produit. Actuellement, il existe trois métriques d’**instantanés** émises par IoT Hub : *Nombre total de messages utilisés*, *Nombre total d’appareils (préversion)* et *Appareils connectés (préversion)* . Étant donné que ces métriques présentent une quantité « totale » chaque fois qu’elles sont émises, leur addition sur la granularité spécifiée n’est pas justifiée. Azure Monitor vous limite à sélectionner la moyenne, la valeur minimale et la valeur maximale pour le type d’agrégation de ces métriques.

### <a name="cloud-to-device-command-metrics"></a>Métriques de commande cloud à appareil

|Nom d’affichage de la métrique|Métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|
|Messages cloud vers appareil ayant expiré (préversion)|C2DMessagesExpired|Count|Total|Nombre de messages cloud vers appareil ayant expiré|None|
|Remises de messages cloud vers appareil terminées|c2d.commands.egress.complete.success|Nombre|Total|Nombre de remises de messages cloud vers appareil terminées avec succès par l’appareil|None|
|Messages cloud vers appareil abandonnés|c2d.commands.egress.abandon.success|Nombre|Total|Nombre de messages cloud vers appareil abandonnés par l’appareil|None|
|Messages cloud vers appareil rejetés|c2d.commands.egress.reject.success|Nombre|Total|Nombre de messages cloud vers appareil rejetés par l’appareil|None|

Pour les métriques dont la valeur d’**unité** est **Comptage**, seule l’agrégation totale (somme) est valide. Les agrégations Minimum, Maximum et Moyenne retournent toujours la valeur 1. Pour en savoir plus, consultez les [agrégations prises en charge](#supported-aggregations).

### <a name="cloud-to-device-direct-methods-metrics"></a>Métriques de méthodes directes cloud à appareil

|Nom d’affichage de la métrique|Métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|
|Appels de méthode directe en échec|c2d.methods.failure|Count|Total|Total des appels de méthode directe en échec.|None|
|Taille de demande des appels de méthode directe|c2d.methods.requestSize|Octets|Average|Total des requêtes de méthode directe réussies.|None|
|Taille de réponse des appels de méthode directe|c2d.methods.responseSize|Octets|Average|Total des réponses de méthode directe réussies.|None|
|Appels de méthode directe réussis|c2d.methods.success|Count|Total|Total des appels de méthode directe réussis.|None|

Pour les métriques dont la valeur d’**unité** est **Comptage**, seule l’agrégation totale (somme) est valide. Les agrégations Minimum, Maximum et Moyenne retournent toujours la valeur 1. Pour en savoir plus, consultez les [agrégations prises en charge](#supported-aggregations).

### <a name="cloud-to-device-twin-operations-metrics"></a>Métriques d’opérations jumelles cloud à appareil

|Nom d’affichage de la métrique|Métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|
|Lectures de représentations de serveur principal en échec|c2d.twin.read.failure|Count|Total|Total des lectures de représentations en échec initiées par un serveur principal.|None|
|Mises à jour de représentations de serveur principal en échec|c2d.twin.update.failure|Count|Total|Total des mises à jour de représentations en échec initiées par un serveur principal.|None|
|Taille de la réponse des lectures de représentations de serveur principal|c2d.twin.read.size|Octets|Average|Total des lectures de représentations réussies initiées par un serveur principal.|None|
|Taille des mises à jour de représentations de serveur principal|c2d.twin.update.size|Octets|Average|Total des mises à jour de représentations réussies initiées par un serveur principal.|None|
|Lectures de représentations réussies de serveur principal|c2d.twin.read.success|Count|Total|Total des lectures de représentations réussies initiées par un serveur principal.|None|
|Mises à jour de représentations réussies de serveur principal|c2d.twin.update.success|Count|Total|Total des mises à jour de représentations réussies initiées par un serveur principal.|None|

Pour les métriques dont la valeur d’**unité** est **Comptage**, seule l’agrégation totale (somme) est valide. Les agrégations Minimum, Maximum et Moyenne retournent toujours la valeur 1. Pour en savoir plus, consultez les [agrégations prises en charge](#supported-aggregations).

### <a name="configurations-metrics"></a>Métriques des configurations

|Nom d’affichage de la métrique|Métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|
|Métriques de configuration|configurations|Count|Total|Nombre total d’opérations CRUD effectuées pour la configuration de l’appareil et le déploiement d’IoT Edge sur un ensemble d’appareils cibles. Ceci inclut également le nombre d’opérations qui modifient le jumeau d'appareil ou le jumeau de module en raison de ces configurations.|Aucun|

Pour les métriques dont la valeur d’**unité** est **Comptage**, seule l’agrégation totale (somme) est valide. Les agrégations Minimum, Maximum et Moyenne retournent toujours la valeur 1. Pour en savoir plus, consultez les [agrégations prises en charge](#supported-aggregations).

### <a name="daily-quota-metrics"></a>Métriques de quota quotidien

|Nom d’affichage de la métrique|Métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|
|Utilisation totale des données d’appareil|deviceDataUsage|Octets|Total|Nombre d’octets transférés vers et depuis tous les appareils connectés à IotHub|None|
|Utilisation totale des données d’appareil (préversion)|deviceDataUsageV2|Octets|Total|Nombre d’octets transférés vers et depuis tous les appareils connectés à IotHub|None|
|Nombre total de messages utilisés|dailyMessageQuotaUsed|Count|Average|Nombre total de messages utilisés aujourd’hui. Il s’agit d’une valeur cumulative qui est réinitialisée sur zéro à 00h00 UTC chaque jour.|None|

Pour *Nombre total de messages utilisés*, seules les agrégations Minimum, Maximum et Moyenne sont prises en charge. Pour en savoir plus, consultez les [agrégations prises en charge](#supported-aggregations).

### <a name="device-metrics"></a>Mesures de l’appareil

|Nom d’affichage de la métrique|Métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|
|Nombre total d’appareils (déconseillé)|devices.totalDevices|Count|Total|Nombre d’appareils enregistrés sur votre hub IoT|None|
|Appareils connectés (déconseillé) |devices.connectedDevices.allProtocol|Nombre|Total|Nombre d’appareils connectés à votre hub IoT|None|
|Nombre total d’appareils (préversion)|totalDeviceCount|Count|Average|Nombre d’appareils enregistrés sur votre hub IoT|None|
|Appareils connectés (préversion)|connectedDeviceCount|Count|Average|Nombre d’appareils connectés à votre hub IoT|None|

Pour *Nombre total d’appareils (déconseillé)* et *Appareils connectés (déconseillé)* , seule l’agrégation Total (Somme) est valide. Les agrégations Minimum, Maximum et Moyenne retournent toujours la valeur 1. Pour en savoir plus, consultez les [agrégations prises en charge](#supported-aggregations).

Pour *Nombre total d’appareils (préversion)* et *Appareils connectés (préversion)* , seules les agrégations Minimum, Maximum et Moyenne sont valides. Pour en savoir plus, consultez les [agrégations prises en charge](#supported-aggregations).

*Appareils connectés (préversion)* et *Nombre total d’appareils (préversion)* ne sont pas exportables via les paramètres de diagnostic.

### <a name="device-telemetry-metrics"></a>Métriques de télémétrie d’appareil

|Nom d’affichage de la métrique|Métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|
|Nombre d’erreurs de limitation|d2c.telemetry.ingress.sendThrottle|Nombre|Total|Nombre d’erreurs de limitation causées par des limitations de débit d’appareil|None|
|Tentatives d’envoi de message de télémétrie|d2c.telemetry.ingress.allProtocol|Nombre|Total|Nombre de tentatives d’envoi de messages de télémétrie appareil vers cloud à votre hub IoT|None|
|Messages de télémétrie envoyés|d2c.telemetry.ingress.success|Count|Total|Nombre de messages de télémétrie appareil vers cloud envoyés avec succès à votre hub IoT|None|

Pour les métriques dont la valeur d’**unité** est **Comptage**, seule l’agrégation totale (somme) est valide. Les agrégations Minimum, Maximum et Moyenne retournent toujours la valeur 1. Pour en savoir plus, consultez les [agrégations prises en charge](#supported-aggregations).

### <a name="device-to-cloud-twin-operations-metrics"></a>Métriques d’opérations jumelles appareil à cloud

|Nom d’affichage de la métrique|Métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|
|Lectures de représentations d’appareils en échec|d2c.twin.read.failure|Count|Total|Total des lectures de représentations en échec initiées par un appareil.|None|
|Mises à jour de représentations d’appareils en échec|d2c.twin.update.failure|Count|Total|Total des mises à jour de représentations en échec initiées par un appareil.|None|
|Taille de la réponse des lectures de représentations des appareils|d2c.twin.read.size|Octets|Average|Nombre de lectures de toutes les représentations réussies initiées par un appareil.|None|
|Taille des mises à jour de représentations d’appareils|d2c.twin.update.size|Octets|Average|Taille totale des mises à jour de représentations réussies initiées par un appareil.|None|
|Lectures de représentations réussies d’appareils|d2c.twin.read.success|Count|Total|Total des lectures de représentations réussies initiées par un appareil.|None|
|Mises à jour de représentations réussies d’appareils|d2c.twin.update.success|Count|Total|Total des mises à jour de représentations réussies initiées par un appareil.|None|

Pour les métriques dont la valeur d’**unité** est **Comptage**, seule l’agrégation totale (somme) est valide. Les agrégations Minimum, Maximum et Moyenne retournent toujours la valeur 1. Pour en savoir plus, consultez les [agrégations prises en charge](#supported-aggregations).

### <a name="event-grid-metrics"></a>Métriques de grille d’événements

|Nom d’affichage de la métrique|Métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|
|Remises Event Grid (préversion)|EventGridDeliveries|Count|Total|Nombre d’événements IoT Hub publiés dans Event Grid. Utilisez la dimension Résultat pour le nombre de requêtes ayant réussi et ayant échoué. La dimension EventType représente le type de l’événement (https://aka.ms/ioteventgrid) ).|Result,<br/>Type d’événement<br>*Pour plus d’informations, consultez [Dimensions de métrique](#metric-dimensions)* .|
|Latence d’Event Grid (préversion)|EventGridLatency|Millisecondes|Average|Latence moyenne (en millisecondes) entre le moment où l’événement Iot Hub a été généré et le moment où l’événement a été publié dans Event Grid. Ce nombre est une moyenne de tous les types d’événement. Utilisez la dimension Type d’événement pour afficher la latence d’un type d’événement spécifique.|Type d’événement<br>*Pour plus d’informations, consultez [Dimensions de métrique](#metric-dimensions)* .|

Pour les métriques dont la valeur d’**unité** est **Comptage**, seule l’agrégation totale (somme) est valide. Les agrégations Minimum, Maximum et Moyenne retournent toujours la valeur 1. Pour en savoir plus, consultez les [agrégations prises en charge](#supported-aggregations).

### <a name="jobs-metrics"></a>Métriques de travaux

|Nom d’affichage de la métrique|Métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|
|Travaux terminés|jobs.completed|Count|Total|Total des travaux terminés.|None|
|Appels en échec pour répertorier les travaux|jobs.listJobs.failure|Count|Total|Total des appels en échec pour répertorier les travaux.|None|
|Créations des travaux d’appel de méthode en échec|jobs.createDirectMethodJob.failure|Nombre|Total|Total des créations en échec des travaux d’appel de méthode directe.|None|
|Créations des travaux de mises à jour de représentations en échec|jobs.createTwinUpdateJob.failure|Nombre|Total|Total des créations en échec des travaux de mises à jour de représentations.|None|
|Annulations de travaux en échec|jobs.cancelJob.failure|Count|Total|Total des appels en échec pour annuler un travail.|None|
|Requêtes de travaux en échec|jobs.queryJobs.failure|Count|Total|Total des appels en échec pour interroger les travaux.|None|
|Travaux en échec|jobs.failed|Count|Total|Total des travaux en échec.|None|
|Appels réussis pour répertorier les travaux|jobs.listJobs.success|Count|Total|Total des appels réussis pour répertorier les travaux.|None|
|Créations réussies des travaux d’appel de méthode|jobs.createDirectMethodJob.success|Nombre|Total|Total des créations réussies des travaux d’appel de méthode directe.|None|
|Créations réussies des travaux de mises à jour de représentations|jobs.createTwinUpdateJob.<br>success|Count|Total|Total des créations réussies de travaux de mises à jour de représentations.|None|
|Annulations de travaux réussies|jobs.cancelJob.success|Count|Total|Total des appels réussis pour annuler un travail.|None|
|Requêtes de travaux réussies|jobs.queryJobs.success|Count|Total|Total des appels réussis pour interroger les travaux.|None|

Pour les métriques dont la valeur d’**unité** est **Comptage**, seule l’agrégation totale (somme) est valide. Les agrégations Minimum, Maximum et Moyenne retournent toujours la valeur 1. Pour en savoir plus, consultez les [agrégations prises en charge](#supported-aggregations).

### <a name="routing-metrics"></a>Métriques de routage

|Nom d’affichage de la métrique|Métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|
| Tentatives de livraison de routage (préversion) |RoutingDeliveries | Count | Total |Il s’agit de la métrique de livraison de routage. Utilisez les dimensions pour identifier l’état de livraison d’un point de terminaison ou d’une source de routage spécifiques.| Result,<br>RoutingSource,<br>EndpointType,<br>FailureReasonCategory,<br>EndpointName<br>*Pour plus d’informations, consultez [Dimensions de métrique](#metric-dimensions)* . |
| Taille des données de livraison du routage en octets (préversion)|RoutingDataSizeInBytesDelivered| Octets | Total |Nombre total d’octets acheminés par IoT Hub vers un point de terminaison personnalisé et un point de terminaison intégré. Utilisez les dimensions pour identifier la taille des données acheminées vers un point de terminaison ou une source de routage spécifiques.| RoutingSource,<br>EndpointType<br>EndpointName<br>*Pour plus d’informations, consultez [Dimensions de métrique](#metric-dimensions)* .|
| Latence du routage (préversion) |RoutingDeliveryLatency| Millisecondes | Average |Il s’agit de la métrique de latence de la livraison de routage. Utilisez les dimensions pour identifier l’état de livraison d’un point de terminaison ou d’une source de routage spécifiques.| RoutingSource,<br>EndpointType,<br>EndpointName<br>*Pour plus d’informations, consultez [Dimensions de métrique](#metric-dimensions)* .|
|Routage : objets blob remis au stockage|d2c.endpoints.egress.storage.blobs|Nombre|Total|Nombre de fois où le routage IoT Hub a remis des objets blob à des points de terminaison de stockage.|None|
|Routage : données remises au stockage|d2c.endpoints.egress.storage.bytes|Octets|Total|Quantité de données (octets) que le routage IoT Hub a remis aux points de terminaison de stockage.|None|
|Routage : latence des messages d’Event Hub|d2c.endpoints.latency.eventHubs|Millisecondes|Average|Latence moyenne (en millisecondes) entre les entrées de messages vers IoT Hub et dans des points de terminaison personnalisés de type Event Hub. Ceci n’inclut pas les itinéraires des messages vers les points de terminaison intégrés (événements).|None|
|Routage : latence des messages de la file d’attente Service Bus|d2c.endpoints.latency.serviceBusQueues|Millisecondes|Average|Latence moyenne (en millisecondes) entre les entrées de messages vers IoT Hub et dans un point de terminaison de file d’attente Service Bus.|None|
|Routage : latence des messages de la rubrique Service Bus|d2c.endpoints.latency.serviceBusTopics|Millisecondes|Average|Latence moyenne (en millisecondes) entre les entrées de messages vers IoT Hub et dans un point de terminaison de rubrique Service Bus.|None|
|Routage : latence des messages de messages/d’événements|d2c.endpoints.latency.builtIn.events|Millisecondes|Average|Latence moyenne (en millisecondes) entre les entrées de messages vers IoT Hub et dans le point de terminaison intégré (messages/événements) l’itinéraire de secours.|None|
|Routage : latence des messages du stockage|d2c.endpoints.latency.storage|Millisecondes|Average|Latence moyenne (en millisecondes) entre les entrées de messages vers IoT Hub et dans un point de terminaison de stockage.|None|
|Routage : messages remis à Event Hub|d2c.endpoints.egress.eventHubs|Nombre|Total|Nombre de fois où le routage IoT Hub a correctement remis des messages à des points de terminaison personnalisés de type Event Hub. Ceci n’inclut pas les itinéraires des messages vers les points de terminaison intégrés (événements).|None|
|Routage : messages remis à la file d’attente Service Bus|d2c.endpoints.egress.serviceBusQueues|Nombre|Total|Nombre de fois où le routage IoT Hub a correctement remis des messages aux points de terminaison de file d’attente Service Bus.|None|
|Routage : messages remis à la rubrique Service Bus|d2c.endpoints.egress.serviceBusTopics|Nombre|Total|Nombre de fois où le routage IoT Hub a correctement remis des messages à des points de terminaison de rubrique Service Bus.|None|
|Routage : messages remis à l’itinéraire de secours|d2c.telemetry.egress.fallback|Nombre|Total|Nombre de fois où le routage IoT Hub a remis des messages au point de terminaison associé à l’itinéraire de secours.|None|
|Routage : messages remis à des messages/événements|d2c.endpoints.egress.builtIn.events|Nombre|Total|Nombre de fois où le routage IoT Hub a correctement remis des messages au point de terminaison intégré (messages/événements) à l’itinéraire de secours.|None|
|Routage : messages remis au stockage|d2c.endpoints.egress.storage|Nombre|Total|Nombre de fois où le routage IoT Hub a correctement remis des messages à des points de terminaison de stockage.|None|
|Routage : messages de télémétrie remis|d2c.telemetry.egress.success|Nombre|Total|Nombre de fois où des messages ont été correctement remis à tous les points de terminaison à l’aide du routage IoT Hub. Si un message est routé vers plusieurs points de terminaison, cette valeur augmente d’une unité pour chaque remise réussie. Si un message est routé plusieurs fois vers le même point de terminaison, cette valeur augmente d’une unité pour chaque remise réussie.|None|
|Routage : messages de télémétrie annulés |d2c.telemetry.egress.dropped|Nombre|Total|Nombre de fois où des messages ont été annulés par le routage IoT Hub en raison de points de terminaison morts. Cette valeur ne compte pas les messages remis à un itinéraire de secours, car les messages annulés n’y sont pas remis.|None|
|Routage : messages de télémétrie incompatibles|d2c.telemetry.egress.invalid|Nombre|Total|Nombre de fois où le routage IoT Hub n’a pas réussi à remettre des messages en raison d’une incompatibilité avec le point de terminaison. Un message est incompatible avec un point de terminaison quand IoT Hub tente de remettre le message à un point de terminaison et échoue avec une erreur non temporaire. Il n’y a pas de nouvelle tentative pour les messages non valides. Cette valeur n’inclut pas les nouvelles tentatives.|None|
|Routage : messages de télémétrie orphelins |d2c.telemetry.egress.orphaned|Count|Total|Nombre de fois où des messages ont été définis comme orphelins par le routage IoT Hub car ils ne correspondaient à aucune requête de routage, lorsque la route de secours est désactivée.|None|

Pour les métriques dont la valeur d’**unité** est **Comptage**, seule l’agrégation totale (somme) est valide. Les agrégations Minimum, Maximum et Moyenne retournent toujours la valeur 1. Pour en savoir plus, consultez les [agrégations prises en charge](#supported-aggregations).

### <a name="twin-query-metrics"></a>Métriques de requête de jumeau

|Nom d’affichage de la métrique|Métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|
|Requêtes de représentations en échec|twinQueries.failure|Count|Total|Total des requêtes de représentations en échec.|None|
|Requêtes de représentations réussies|twinQueries.success|Count|Total|Total des requêtes de représentations réussies.|None|
|Taille du résultat des requêtes de représentations|twinQueries.resultSize|Octets|Average|Total de toutes les requêtes de représentations réussies.|None|

Pour les métriques dont la valeur d’**unité** est **Comptage**, seule l’agrégation totale (somme) est valide. Les agrégations Minimum, Maximum et Moyenne retournent toujours la valeur 1. Pour en savoir plus, consultez les [agrégations prises en charge](#supported-aggregations).

## <a name="metric-dimensions"></a>Dimensions de métrique

Azure IoT Hub a les dimensions suivantes associées à certaines de ses métriques de routage et de grille d’événements.

|Nom de la dimension | Description|
|---|---|
||
|**EndpointName**| Nom du point de terminaison.|
|**EndpointType**|L’une des valeurs suivantes : **eventHubs**, **serviceBusQueues**, **cosmosDB**, **serviceBusTopics**, **builtin** ou **blobStorage**.|
|**EventType**| L’un des types d’événements Event Grid suivants : **Microsoft.Devices.DeviceCreated**, **Microsoft.Devices.DeviceDeleted**, **Microsoft.Devices.DeviceConnected**, **Microsoft.Devices.DeviceDisconnected** ou **Microsoft.Devices.DeviceTelemetry**. Pour plus d’informations, consultez [Types d’événements](iot-hub-event-grid.md#event-types).|
|**FailureReasonCategory**| L’une des valeurs suivantes : **non valide**, **supprimée**, **orpheline** ou **Null**.|
|**Résultat**| **Réussite** ou **Échec**.|
|**RoutingSource**| Messages des appareils<br>Événements de changement de jumeau<br>Événements de cycle de vie d'appareil|

Pour en savoir plus sur les dimensions de métrique, consultez [Métriques multidimensionnelles](../azure-monitor/platform/data-platform-metrics.md#multi-dimensional-metrics).

## <a name="resource-logs"></a>Journaux d’activité de ressources

Cette section répertorie tous les schémas et types de catégories de journaux de ressources collectés pour Azure IoT Hub. Le fournisseur de ressources et le type de tous les journaux IoT Hub sont [Microsoft.Devices/IotHubs](../azure-monitor/platform/resource-logs-categories.md#microsoftdevicesiothubs).

**Rubriques de cette section**

- [Connexions](#connections)
- [Télémétrie de l’appareil](#device-telemetry)
- [Commandes cloud-à-appareil](#cloud-to-device-commands)
- [Opérations d’identité des appareils](#device-identity-operations)
- [Opérations de téléchargement de fichier](#file-upload-operations)
- [Itinéraires](#routes)
- [Opérations jumelles appareil à cloud](#device-to-cloud-twin-operations)
- [Opérations jumelles cloud à appareil](#cloud-to-device-twin-operations)
- [Requêtes de jumeaux](#twin-queries)
- [Opérations de travaux](#jobs-operations)
- [Méthodes directes](#direct-methods)
- [Traçage distribué (préversion)](#distributed-tracing-preview)
  - [Journaux d’activité appareil à cloud (D2C) IoT Hub](#iot-hub-d2c-device-to-cloud-logs)
  - [Journaux d’activité d’entrée IoT Hub](#iot-hub-ingress-logs)
  - [Journaux d’activité de sortie IoT Hub](#iot-hub-egress-logs)
- [Configurations](#configurations)
- [Flux d’appareils (préversion)](#device-streams-preview)

### <a name="connections"></a>Connexions

La catégorie de connexions suit les événements de connexions et de déconnexion des appareils à partir d’un hub IoT, ainsi que les erreurs. Cette catégorie est utile pour identifier les tentatives de connexion non autorisées et/ou donner l’alerte lorsque vous êtes déconnecté des appareils.

> [!NOTE]
> Pour l’état de connexion fiable des appareils, voir [Pulsation des appareils](iot-hub-devguide-identity-registry.md#device-heartbeat).

```json
{
   "records":
   [
        {
            "time": " UTC timestamp",
            "resourceId": "Resource Id",
            "operationName": "deviceConnect",
            "category": "Connections",
            "level": "Information",
            "properties": "{\"deviceId\":\"<deviceId>\",\"sdkVersion\":\"<sdkVersion>\",\"protocol\":\"<protocol>\",\"authType\":\"{\\\"scope\\\":\\\"device\\\",\\\"type\\\":\\\"sas\\\",\\\"issuer\\\":\\\"iothub\\\",\\\"acceptingIpFilterRule\\\":null}\",\"maskedIpAddress\":\"<maskedIpAddress>\"}",
            "location": "Resource location"
        }
    ]
}
```

### <a name="device-telemetry"></a>Télémétrie d’appareil

La catégorie de télémétrie d’appareil effectue le suivi des erreurs qui se produisent au niveau du hub IoT et qui sont liées au pipeline de télémétrie. Cette catégorie inclut notamment les erreurs concernant l’envoi d’événements de télémétrie (par exemple, une limitation) et la réception des événements de télémétrie (par exemple, un lecteur non autorisé). Cette catégorie ne peut pas intercepter les erreurs provoquées par le code en cours d’exécution sur l’appareil lui-même.

```json
{
    "records":
    [
        {
            "time": "UTC timestamp",
            "resourceId": "Resource Id",
            "operationName": "ingress",
            "category": "DeviceTelemetry",
            "level": "Error",
            "resultType": "Event status",
            "resultDescription": "MessageDescription",
            "properties": "{\"deviceId\":\"<deviceId>\",\"batching\":\"0\",\"messageSizeInBytes\":\"<messageSizeInBytes>\",\"EventProcessedUtcTime\":\"<UTC timestamp>\",\"EventEnqueuedUtcTime\":\"<UTC timestamp>\",\"partitionId\":\"1\"}", 
            "location": "Resource location"
        }
    ]
}
```

### <a name="cloud-to-device-commands"></a>Commandes cloud-à-appareil

La catégorie de commandes cloud-à-appareil effectue le suivi des erreurs qui se produisent au niveau du hub IoT et qui sont liées au pipeline de messages cloud-à-appareil. Cette catégorie inclut les erreurs qui surviennent suite à ces événements :

* Envoi de messages cloud-à-appareil (erreurs liées aux expéditeurs non autorisés, par exemple)
* Réception de messages cloud-à-appareil (erreurs liées au dépassement du nombre de remises, par exemple)
* Réception de commentaires sur les messages cloud-à-appareil (erreurs liées à l’expiration des commentaires, par exemple)

Cette catégorie ne signale pas les erreurs lorsque le message cloud-à-appareil est bien remis mais qu’il n’est pas correctement traité par l’appareil.

```json
{
    "records":
    [
        {
            "time": " UTC timestamp",
            "resourceId": "Resource Id",
            "operationName": "messageExpired",
            "category": "C2DCommands",
            "level": "Error",
            "resultType": "Event status",
            "resultDescription": "MessageDescription",
            "properties": "{\"deviceId\":\"<deviceId>\",\"messageId\":\"<messageId>\",\"messageSizeInBytes\":\"<messageSize>\",\"protocol\":\"Amqp\",\"deliveryAcknowledgement\":\"<None, NegativeOnly, PositiveOnly, Full>\",\"deliveryCount\":\"0\",\"expiryTime\":\"<timestamp>\",\"timeInSystem\":\"<timeInSystem>\",\"ttl\":<ttl>, \"EventProcessedUtcTime\":\"<UTC timestamp>\",\"EventEnqueuedUtcTime\":\"<UTC timestamp>\", \"maskedIpAddress\": \"<maskedIpAddress>\", \"statusCode\": \"4XX\"}",
            "location": "Resource location"
        }
    ]
}
```

### <a name="device-identity-operations"></a>Opérations d’identité des appareils

La catégorie d’opérations d’identité des appareils effectue le suivi des erreurs qui se produisent quand vous tentez de créer, mettre à jour ou supprimer une entrée dans le registre d’identité de votre hub IoT. Le suivi de cette catégorie est utile pour les scénarios d’approvisionnement.

```json
{
    "records":
    [
        {
            "time": "UTC timestamp",
            "resourceId": "Resource Id",
            "operationName": "get",
            "category": "DeviceIdentityOperations",
            "level": "Error",
            "resultType": "Event status",
            "resultDescription": "MessageDescription",
            "properties": "{\"maskedIpAddress\":\"<maskedIpAddress>\",\"deviceId\":\"<deviceId>\", \"statusCode\":\"4XX\"}",
            "location": "Resource location"
        }
    ]
}
```

### <a name="file-upload-operations"></a>Opérations de téléchargement de fichier

La catégorie de chargement de fichiers effectue le suivi des erreurs qui se produisent au niveau de l’IoT hub et qui sont liées à la fonctionnalité de chargement. Cette catégorie inclut :

* Erreurs qui se produisent avec l’URI SAP, par exemple en cas d’expiration avant qu’un appareil notifie le hub d’un téléchargement terminé.

* Échecs des téléchargements signalés par l’appareil.

* Erreurs qui se produisent lorsqu’un fichier est introuvable dans le stockage lors de la création du message de notification IoT Hub.

Cette catégorie ne peut pas détecter les erreurs qui surviennent directement pendant que l’appareil charge un fichier de stockage.

```json
{
    "records":
    [
        {
            "time": "UTC timestamp",
            "resourceId": "Resource Id",
            "operationName": "ingress",
            "category": "FileUploadOperations",
            "level": "Error",
            "resultType": "Event status",
            "resultDescription": "MessageDescription",
            "durationMs": "1",
            "properties": "{\"deviceId\":\"<deviceId>\",\"protocol\":\"<protocol>\",\"authType\":\"{\\\"scope\\\":\\\"device\\\",\\\"type\\\":\\\"sas\\\",\\\"issuer\\\":\\\"iothub\\\",\\\"acceptingIpFilterRule\\\":null}\",\"blobUri\":\"http//bloburi.com\"}",
            "location": "Resource location"
        }
    ]
}
```

### <a name="routes"></a>Itinéraires

La catégorie de [routage des messages](./iot-hub-devguide-messages-d2c.md) assure le suivi des erreurs qui se produisent pendant l’évaluation du routage des messages et de l’intégrité du point de terminaison perçue par IoT Hub. Cette catégorie inclut les événements de type :

* Une règle est évaluée sur « indéfinie »
* Un point de terminaison est signalé comme étant inactif par IoT Hub
* Toutes les erreurs reçues depuis un point de terminaison

Cette catégorie n’inclut pas les erreurs spécifiques aux messages eux-mêmes (les erreurs de limitation des appareils, par exemple) qui sont signalées dans la catégorie « télémétrie des appareils ».

```json
{
    "records":
    [
        {
            "time":"2019-12-12T03:25:14Z",
            "resourceId":"/SUBSCRIPTIONS/91R34780-3DEC-123A-BE2A-213B5500DFF0/RESOURCEGROUPS/ANON-TEST/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/ANONHUB1",
            "operationName":"endpointUnhealthy",
            "category":"Routes",
            "level":"Error",
            "resultType":"403004",
            "resultDescription":"DeviceMaximumQueueDepthExceeded",
            "properties":"{\"deviceId\":null,\"endpointName\":\"anon-sb-1\",\"messageId\":null,\"details\":\"DeviceMaximumQueueDepthExceeded\",\"routeName\":null,\"statusCode\":\"403\"}",
            "location":"westus"
        }
    ]
}
```

Voici plus de détails sur le routage des journaux de ressource :

* [Liste des codes d’erreur du routage de journaux de ressource](troubleshoot-message-routing.md#diagnostics-error-codes)
* [Liste des operationNames du routage des journaux de ressource](troubleshoot-message-routing.md#diagnostics-operation-names)

### <a name="device-to-cloud-twin-operations"></a>Opérations jumelles appareil à cloud

La catégorie des opérations jumelles appareil-à-cloud effectue le suivi des événements initiés par l’appareil sur des jumeaux d’appareil. Ces opérations peuvent inclure l’obtention d’un jumeau, la mise à jour les propriétés signalées et l’abonnement aux propriétés souhaitées.

```json
{
    "records":
    [
        {
            "time": "UTC timestamp",
            "resourceId": "Resource Id",
            "operationName": "update",
            "category": "D2CTwinOperations",
            "level": "Information",
            "durationMs": "1",
            "properties": "{\"deviceId\":\"<deviceId>\",\"protocol\":\"<protocol>\",\"authenticationType\":\"{\\\"scope\\\":\\\"device\\\",\\\"type\\\":\\\"sas\\\",\\\"issuer\\\":\\\"iothub\\\",\\\"acceptingIpFilterRule\\\":null}\"}",
            "location": "Resource location"
        }
    ]
}
```

### <a name="cloud-to-device-twin-operations"></a>Opérations jumelles cloud à appareil

La catégorie des opérations jumelles cloud-à-appareil effectue le suivi des événements initiés par le service sur des jumeaux d’appareil. Ces opérations peuvent inclure : obtenir un jumeau, mettre à jour ou remplacer des balises, et mettre à jour ou remplacer les propriétés souhaitées.

```json
{
    "records":
    [
        {
            "time": "UTC timestamp",
            "resourceId": "Resource Id",
            "operationName": "read",
            "category": "C2DTwinOperations",
            "level": "Information",
            "durationMs": "1",
            "properties": "{\"deviceId\":\"<deviceId>\",\"sdkVersion\":\"<sdkVersion>\",\"messageSize\":\"<messageSize>\"}",
            "location": "Resource location"
        }
    ]
}
```

### <a name="twin-queries"></a>Requêtes de jumeaux

La catégorie des requêtes de jumeaux génère un rapport sur les demandes de requêtes pour jumeaux d’appareil lancées dans le cloud.

```json
{
    "records":
    [
        {
            "time": "UTC timestamp",
            "resourceId": "Resource Id",
            "operationName": "query",
            "category": "TwinQueries",
            "level": "Information",
            "durationMs": "1",
            "properties": "{\"query\":\"<twin query>\",\"sdkVersion\":\"<sdkVersion>\",\"messageSize\":\"<messageSize>\",\"pageSize\":\"<pageSize>\", \"continuation\":\"<true, false>\", \"resultSize\":\"<resultSize>\"}",
            "location": "Resource location"
        }
    ]
}
```

### <a name="jobs-operations"></a>Opérations de travaux

La catégorie des opérations de travaux génère un rapport sur les demandes de travaux pour mettre à jour des jumeaux d’appareil ou appeler des méthodes directes sur plusieurs appareils. Ces demandes sont lancées dans le cloud.

```json
{
    "records":
    [
        {
            "time": "UTC timestamp",
            "resourceId": "Resource Id",
            "operationName": "jobCompleted",
            "category": "JobsOperations",
            "level": "Information",
            "durationMs": "1",
            "properties": "{\"jobId\":\"<jobId>\", \"sdkVersion\": \"<sdkVersion>\",\"messageSize\": <messageSize>,\"filter\":\"DeviceId IN ['1414ded9-b445-414d-89b9-e48e8c6285d5']\",\"startTimeUtc\":\"Wednesday, September 13, 2017\",\"duration\":\"0\"}",
            "location": "Resource location"
        }
    ]
}
```

### <a name="direct-methods"></a>Méthodes directes

La catégorie des méthodes directes assure le suivi des interactions demande-réponse envoyées à des appareils individuels. Ces demandes sont lancées dans le cloud.

```json
{
    "records":
    [
        {
            "time": "UTC timestamp",
            "resourceId": "Resource Id",
            "operationName": "send",
            "category": "DirectMethods",
            "level": "Information",
            "durationMs": "1",
            "properties": "{\"deviceId\":<messageSize>, \"RequestSize\": 1, \"ResponseSize\": 1, \"sdkVersion\": \"2017-07-11\"}",
            "location": "Resource location"
        }
    ]
}
```

### <a name="distributed-tracing-preview"></a>Traçage distribué (préversion)

La catégorie de traçage distribué suit les ID de corrélation pour les messages comportant l’en-tête de contexte de trace. Pour activer pleinement ces journaux d’activité, le code côté client doit être mis à jour en suivant [Analyser et diagnostiquer les applications IoT de bout en bout avec le traçage distribué IoT Hub (preview)](iot-hub-distributed-tracing.md).

Notez que `correlationId` est conforme à la proposition [Trace Context du W3](https://github.com/w3c/trace-context), car il contient un `trace-id` ainsi qu’un `span-id`.

#### <a name="iot-hub-d2c-device-to-cloud-logs"></a>Journaux d’activité appareil à cloud (D2C) IoT Hub

IoT Hub enregistre ce journal en recevant un message contenant des propriétés de trace valides.

```json
{
    "records":
    [
        {
            "time": "UTC timestamp",
            "resourceId": "Resource Id",
            "operationName": "DiagnosticIoTHubD2C",
            "category": "DistributedTracing",
            "correlationId": "00-8cd869a412459a25f5b4f31311223344-0144d2590aacd909-01",
            "level": "Information",
            "resultType": "Success",
            "resultDescription":"Receive message success",
            "durationMs": "",
            "properties": "{\"messageSize\": 1, \"deviceId\":\"<deviceId>\", \"callerLocalTimeUtc\": : \"2017-02-22T03:27:28.633Z\", \"calleeLocalTimeUtc\": \"2017-02-22T03:27:28.687Z\"}",
            "location": "Resource location"
        }
    ]
}
```

Ici, `durationMs` n’est pas calculé, car l’horloge IoT Hub peut ne pas être synchronisée avec l’horloge de l’appareil, ce qui fausse le calcul de durée. Nous recommandons d’écrire la logique en utilisant les horodatages de la section `properties` pour capturer les pics de latence de l’appareil vers le cloud.

| Propriété | Type | Description |
|--------------------|-----------------------------------------------|------------------------------------------------------------------------------------------------|
| **messageSize** | Integer | Taille du message appareil-à-cloud, en octets |
| **deviceId** | Chaîne de caractères alphanumériques ASCII 7 bits | Identité de l’appareil |
| **callerLocalTimeUtc** | Horodateur UTC | Heure de création du message telle que rapportée par l’horloge local de l’appareil |
| **calleeLocalTimeUtc** | Horodateur UTC | Heure de réception du message sur la passerelle d’IoT Hub telle qu’elle est rapportée par l’horloge du service IoT Hub |

#### <a name="iot-hub-ingress-logs"></a>Journaux d’activité d’entrée IoT Hub

IoT Hub enregistre ce journal lorsqu’un message contenant des propriétés de trace valides écrit dans l’Event Hub interne ou intégré.

```json
{
    "records":
    [
        {
            "time": "UTC timestamp",
            "resourceId": "Resource Id",
            "operationName": "DiagnosticIoTHubIngress",
            "category": "DistributedTracing",
            "correlationId": "00-8cd869a412459a25f5b4f31311223344-349810a9bbd28730-01",
            "level": "Information",
            "resultType": "Success",
            "resultDescription":"Ingress message success",
            "durationMs": "10",
            "properties": "{\"isRoutingEnabled\": \"true\", \"parentSpanId\":\"0144d2590aacd909\"}",
            "location": "Resource location"
        }
    ]
}
```

Dans la section `properties`, ce journal contient des informations supplémentaires sur l’entrée des messages.

| Propriété | Type | Description |
|--------------------|-----------------------------------------------|------------------------------------------------------------------------------------------------|
| **isRoutingEnabled** | String | Indique si le routage des messages est activé (true) ou non (false) dans le hub IoT |
| **parentSpanId** | String | [span-id](https://w3c.github.io/trace-context/#parent-id) du message parent, qui serait la trace du message D2C dans ce cas-ci |

#### <a name="iot-hub-egress-logs"></a>Journaux d’activité de sortie IoT Hub

IoT Hub enregistre ce journal lorsque le [routage](iot-hub-devguide-messages-d2c.md) est activé et que le message est écrit sur un [point de terminaison](iot-hub-devguide-endpoints.md). Si le routage n’est pas activé, IoT Hub n’enregistre pas ce journal.

```json
{
    "records":
    [
        {
            "time": "UTC timestamp",
            "resourceId": "Resource Id",
            "operationName": "DiagnosticIoTHubEgress",
            "category": "DistributedTracing",
            "correlationId": "00-8cd869a412459a25f5b4f31311223344-98ac3578922acd26-01",
            "level": "Information",
            "resultType": "Success",
            "resultDescription":"Egress message success",
            "durationMs": "10",
            "properties": "{\"endpointType\": \"EventHub\", \"endpointName\": \"myEventHub\", \"parentSpanId\":\"349810a9bbd28730\"}",
            "location": "Resource location"
        }
    ]
}
```

Dans la section `properties`, ce journal contient des informations supplémentaires sur l’entrée des messages.

| Propriété | Type | Description |
|--------------------|-----------------------------------------------|------------------------------------------------------------------------------------------------|
| **endpointName** | String | Nom du point de terminaison de routage |
| **endpointType** | String | Type du point de terminaison de routage |
| **parentSpanId** | String | [span-id](https://w3c.github.io/trace-context/#parent-id) du message parent, qui serait la trace du message d’entrée IoT Hub dans ce cas-ci |

### <a name="configurations"></a>Configurations

Les journaux de configuration d’IoT Hub permettent de suivre les événements et les erreurs pour l’ensemble de fonctionnalités de la gestion automatique des appareils.

```json
{
    "records":
    [
         {
             "time": "2019-09-24T17:21:52Z",
             "resourceId": "Resource Id",
             "operationName": "ReadManyConfigurations",
             "category": "Configurations",
             "resultType": "",
             "resultDescription": "",
             "level": "Information",
             "durationMs": "17",
             "properties": "{\"configurationId\":\"\",\"sdkVersion\":\"2018-06-30\",\"messageSize\":\"0\",\"statusCode\":null}",
             "location": "southcentralus"
         }
    ]
}
```

### <a name="device-streams-preview"></a>Flux d’appareils (préversion)

La catégorie des flux d’appareil assure le suivi des interactions requête-réponse envoyées à des appareils individuels.

```json
{
    "records":
    [
         {
             "time": "2019-09-19T11:12:04Z",
             "resourceId": "Resource Id",
             "operationName": "invoke",
             "category": "DeviceStreams",
             "resultType": "",
             "resultDescription": "",    
             "level": "Information",
             "durationMs": "74",
             "properties": "{\"deviceId\":\"myDevice\",\"moduleId\":\"myModule\",\"sdkVersion\":\"2019-05-01-preview\",\"requestSize\":\"3\",\"responseSize\":\"5\",\"statusCode\":null,\"requestName\":\"myRequest\",\"direction\":\"c2d\"}",
             "location": "Central US"
         }
    ]
}
```

## <a name="azure-monitor-logs-tables"></a>Tables Azure Monitor Logs
<!-- REQUIRED. Please keep heading in this order -->

Cette section fait référence à toutes les tables Kusto d’Azure Monitor Logs pertinentes pour Azure IoT Hub et disponibles pour une requête par Log Analytics. Pour obtenir la liste de ces tables et des liens vers des informations supplémentaires sur le type de ressource IoT Hub, consultez [IoT Hub](/azure/azure-monitor/reference/tables/tables-resourcetype#iot-hub) dans la référence de table Azure Monitor Logs.

Pour obtenir une référence de toutes les tables Azure Monitor Logs/Log Analytics, consultez la [référence relative aux tables Azure Monitor Logs](/azure/azure-monitor/reference/tables/tables-resourcetype).

## <a name="see-also"></a>Voir aussi

* Pour obtenir une description de l’analyse d’Azure IoT Hub, consultez [Analyse d’Azure IoT Hub](monitor-iot-hub.md).
* Pour plus d’informations sur le monitoring des ressources Azure, voir [Monitoring des ressources Azure avec Azure Monitor](../azure-monitor/insights/monitor-azure-resource.md).