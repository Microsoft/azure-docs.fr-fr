---
title: Réagir aux événements Azure Maps à l'aide d'Event Grid
description: Apprenez à réagir aux événements Azure Maps impliquant des limites géographiques. Apprenez à écouter les événements Azure Maps et à utiliser Event Grid pour rediriger ces événements vers des gestionnaires d'événements.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/16/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 44b2cab814d4e51c4eb0c3ce3322b898f5b26414
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92889764"
---
# <a name="react-to-azure-maps-events-by-using-event-grid"></a>Réagir aux événements Azure Maps à l'aide d'Event Grid

Azure Maps s’intègre à Azure Event Grid pour permettre aux utilisateurs d’envoyer des notifications d’événements à d’autres services et de déclencher des processus en aval. L’objectif de cet article est de vous aider à configurer vos applications métier afin de détecter les événements Azure Maps. Ceci permet aux utilisateurs de réagir à des événements critiques de façon fiable, évolutive et sécurisée. Par exemple, les utilisateurs peuvent créer une application pour mettre à jour une base de données, créer un ticket et remettre une notification par e-mail chaque fois qu’un appareil franchit une limite géographique.

Azure Event Grid est un service de routage d’événement complètement managé qui utilise le modèle publication-abonnement. Event Grid offre une prise en charge intégrée des services Azure comme [Azure Functions](../azure-functions/functions-overview.md) et [Azure Logic Apps](../azure-functions/functions-overview.md). Il peut envoyer des alertes d’événement à des services non-Azure à l’aide de webhooks. Pour obtenir une liste complète des gestionnaires d’événements qui prennent en charge Event Grid, consultez [Présentation d’Azure Event Grid](../event-grid/overview.md).


![Modèle Azure Event Grid fonctionnel](./media/azure-maps-event-grid-integration/azure-event-grid-functional-model.png)


## <a name="azure-maps-events-types"></a>Types d'événements Azure Maps

Event Grid utilise les [abonnements aux événements](../event-grid/concepts.md#event-subscriptions) pour acheminer les messages d’événements vers les abonnés. Un compte Azure Maps émet les types d'événements suivants : 

| Type d'événement | Description |
| ---------- | ----------- |
| Microsoft.Maps.GeofenceEntered | Déclenché quand les coordonnées reçues sont passées de l’extérieur à l’intérieur d’une limite géographique donnée |
| Microsoft.Maps.GeofenceExited | Déclenché quand les coordonnées reçues sont passées de l’intérieur à l’extérieur d’une limite géographique donnée |
| Microsoft.Maps.GeofenceResult | Déclenché chaque fois qu’une requête de geofencing renvoie un résultat, quel que soit l’état. |

## <a name="event-schema"></a>Schéma d’événement

L’exemple suivant présente le schéma de GeofenceResult :

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

## <a name="tips-for-consuming-events"></a>Conseils relatifs à la consommation d’événements

Les applications qui gèrent des événements de limite géographique Azure Maps doivent suivre certaines pratiques recommandées :

* Configurez plusieurs abonnements pour router les événements vers le même gestionnaire d’événements. Il est important de ne pas présumer que les événements proviennent d'une source particulière. Vérifiez toujours le sujet du message pour vous être sûr qu’il provient bien de la source attendue.
* Utilisez le champ `X-Correlation-id` de l’en-tête de la réponse pour savoir si vos informations sur les objets sont à jour. Les messages peuvent arriver dans le désordre ou après un certain délai.
* Quand une requête GET ou POST dans l’API Geofence est appelée avec le paramètre de mode défini sur `EnterAndExit`, un événement d’entrée ou de sortie est généré pour chaque géométrie de la limite géographique dont l’état a changé par rapport à l’appel précédent de l’API Geofence.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l'utilisation du geofencing afin de contrôler les opérations sur un site de construction, consultez :

> [!div class="nextstepaction"] 
> [Configurer une limite géographique à l'aide d'Azure Maps](tutorial-geofence.md)