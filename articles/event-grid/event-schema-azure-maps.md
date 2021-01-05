---
title: Azure Maps en tant que source Event Grid
description: Décrit les propriétés et le schéma qui sont fournis pour les événements Azure Maps avec Azure Event Grid
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 4203bdf5222278b698d656835afebd9769557303
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87461984"
---
# <a name="azure-maps-as-an-event-grid-source"></a>Azure Maps en tant que source Event Grid

Cet article fournit les propriétés et les schémas des événements Azure Maps. Pour une présentation des schémas d’événements, consultez [Schéma d’événements Azure Event Grid](./event-schema.md). Cet article fournit également une liste de guides de démarrage rapide et de tutoriels permettant d’utiliser Azure Maps en tant que source d’événement.

## <a name="event-grid-event-schema"></a>Schéma d’événement Event Grid

### <a name="available-event-types"></a>Types d’événement disponibles

Un compte Azure Maps émet les types d'événements suivants :

| Type d'événement | Description |
| ---------- | ----------- |
| Microsoft.Maps.GeofenceEntered | Déclenché lorsque les coordonnées reçues sont passées de l’extérieur d’une limite géographique donnée à l’intérieur d’une zone donnée. |
| Microsoft.Maps.GeofenceExited | Déclenché lorsque les coordonnées reçues sont passées de l’intérieur d’une limite géographique donnée à l’extérieur de celle-ci. |
| Microsoft.Maps.GeofenceResult | Déclenché chaque fois qu’une requête de geofencing renvoie un résultat, quel que soit l’état. |

### <a name="event-examples"></a>Exemples d’événement

L’exemple suivant montre le schéma d’un événement **GeofenceEntered**

```JSON
{   
   "id":"7f8446e2-1ac7-4234-8425-303726ea3981", 
   "topic":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Maps/accounts/{accountName}", 
   "subject":"/spatial/geofence/udid/{udid}/id/{eventId}", 
   "data":{   
      "geometries":[   
         {   
            "deviceId":"device_1", 
            "udId":"1a13b444-4acf-32ab-ce4e-9ca4af20b169", 
            "geometryId":"2", 
            "distance":-999.0, 
            "nearestLat":47.618786, 
            "nearestLon":-122.132151 
         } 
      ], 
      "expiredGeofenceGeometryId":[   
      ], 
      "invalidPeriodGeofenceGeometryId":[   
      ] 
   }, 
   "eventType":"Microsoft.Maps.GeofenceEntered", 
   "eventTime":"2018-11-08T00:54:17.6408601Z", 
   "metadataVersion":"1", 
   "dataVersion":"1.0" 
}
```

L’exemple suivant montre le schéma de **GeofenceResult**. 

```JSON
{   
   "id":"451675de-a67d-4929-876c-5c2bf0b2c000", 
   "topic":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Maps/accounts/{accountName}", 
   "subject":"/spatial/geofence/udid/{udid}/id/{eventId}", 
   "data":{   
      "geometries":[   
         {   
            "deviceId":"device_1", 
            "udId":"1a13b444-4acf-32ab-ce4e-9ca4af20b169", 
            "geometryId":"1", 
            "distance":999.0, 
            "nearestLat":47.609833, 
            "nearestLon":-122.148274 
         }, 
         {   
            "deviceId":"device_1", 
            "udId":"1a13b444-4acf-32ab-ce4e-9ca4af20b169", 
            "geometryId":"2", 
            "distance":999.0, 
            "nearestLat":47.621954, 
            "nearestLon":-122.131841 
         } 
      ], 
      "expiredGeofenceGeometryId":[   
      ], 
      "invalidPeriodGeofenceGeometryId":[   
      ] 
   }, 
   "eventType":"Microsoft.Maps.GeofenceResult", 
   "eventTime":"2018-11-08T00:52:08.0954283Z", 
   "metadataVersion":"1", 
   "dataVersion":"1.0" 
}
```

### <a name="event-properties"></a>Propriétés d’événement

Un événement contient les données générales suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| topic | string | Chemin d’accès complet à la source de l’événement. Ce champ n’est pas modifiable. Event Grid fournit cette valeur. |
| subject | string | Chemin de l’objet de l’événement, défini par le serveur de publication. |
| eventType | string | Un des types d’événements inscrits pour cette source d’événement. |
| eventTime | string | L’heure à quelle l’événement est généré selon l’heure UTC du fournisseur. |
| id | string | Identificateur unique de l’événement. |
| data | object | Données d’événement de geofencing. |
| dataVersion | string | Version du schéma de l’objet de données. Le serveur de publication définit la version du schéma. |
| metadataVersion | string | Version du schéma des métadonnées d’événement. Event Grid définit le schéma des propriétés de niveau supérieur. Event Grid fournit cette valeur. |

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| apiCategory | string | Catégorie d’API de l’événement. |
| apiName | string | Nom d’API de l’événement. |
| problèmes | object | Répertorie les problèmes rencontrés durant le traitement. Si des problèmes sont retournés, aucune géométrie n’est retournée avec la réponse. |
| responseCode | nombre | Code de réponse HTTP |
| geometries | object | Répertorie les géométries de la limite qui contiennent la position des coordonnées ou chevauchent le searchBuffer autour de la position. |

L’objet d’erreur est retourné lorsqu’une erreur se produit dans l’API Maps. L’objet erreur comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| error | ErrorDetails |Cet objet est retourné lorsqu’une erreur se produit dans l’API Maps.  |

L’objet ErrorDetails est retourné lorsqu’une erreur se produit dans l’API Maps. L’ErrorDetails ou l’objet comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| code | string | Code d’état HTTP. |
| message | string | Si elle est disponible, description de l’erreur compréhensible par l’utilisateur. |
| innererror | InnerError | En cas de disponibilité, un objet contenant des informations sur l’intention reconnue. |

L’InnerError est un objet contenant des informations spécifiques au service pour l’erreur. L’objet InnerError comporte les propriétés suivantes : 

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| code | string | Message d’erreur. |

L’objet geometries énumère les ID de géométrie des limites géographiques qui ont expiré par rapport au délai d’attente utilisateur dans la requête. L’objet geometries possède des éléments géométriques avec les propriétés suivantes : 

| Propriété | Type | Description |
|:-------- |:---- |:----------- |
| deviceid | string | ID de l’appareil. |
| distance | string | <p>Distance entre les coordonnées et la bordure la plus proche de la limite géographique. Une valeur positive signifie que la coordonnée se situe à l’extérieur de la limite géographique. Si la coordonnée se situe à l’extérieur de la limite géographique mais que la valeur de searchBuffer est en dehors de la bordure de limite géographique la plus proche, la valeur est égale à 999. Une valeur négative signifie que la coordonnée se situe à l’intérieur de la limite géographique. Si la coordonnée se situe à l’intérieur du polygone mais que la valeur de searchBuffer est en dehors de la bordure de limite géographique la plus proche, la valeur est égale à -999. Une valeur égale à 999 signifie qu’il y a une grande probabilité que la coordonnée se trouve en dehors de la limite géographique. Une valeur égale à 999 signifie qu’il y a une grande probabilité que la coordonnée se trouve en deçà de la limite géographique.<p> |
| geometryid |string | ID unique identifiant la géométrie de la limite géographique. |
| nearestlat | nombre | Latitude du point plus proche de la géométrie. |
| nearestlon | nombre | Longitude du point plus proche de la géométrie. |
| udId | string | ID unique renvoyé par le service de téléchargement de l’utilisateur lors du téléchargement d’une limite géographique. N’appartient pas à l’API de publication de geofencing. |

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| expiredGeofenceGeometryId | string[] | Répertorie l’ID de géométrie de la limite géographique qui a expiré par rapport au délai d’attente utilisateur dans la requête. |
| geometries | geometries[] |Répertorie les géométries de la limite qui contiennent la position des coordonnées ou chevauchent le searchBuffer autour de la position. |
| invalidPeriodGeofenceGeometryId | string[]  | Répertorie l’ID de géométrie de la limite géographique dont la période n’est pas valide par rapport au délai d’attente utilisateur dans la requête. |
| isEventPublished | boolean | Vrai si au moins un événement est publié à l’abonné de l’événement Azure Maps, faux si aucun événement n’est publié à l’abonné de l’événement Azure Maps |

## <a name="tutorials-and-how-tos"></a>Tutoriels et articles de procédures
|Intitulé  |Description  |
|---------|---------|
| [Réagir aux événements Azure Maps à l’aide d’Event Grid](../azure-maps/azure-maps-event-grid-integration.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Vue d’ensemble de l’intégration d’Azure Maps à Event Grid. |
| [Tutoriel : Définir une limite géographique](../azure-maps/tutorial-geofence.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Ce tutoriel vous montre les étapes de base qui sont nécessaires à la configuration d’une limite géographique à l’aide d’Azure Maps. Vous utilisez Azure Event Grid pour diffuser en continu les résultats de la limite géographique et pour configurer une notification en fonction de ces résultats. |

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation d’Azure Event Grid, consultez [Présentation d’Event Grid](overview.md).
* Pour plus d’informations sur la création d’un abonnement Azure Event Grid, consultez [Schéma d’abonnement à Event Grid](subscription-creation-schema.md).
