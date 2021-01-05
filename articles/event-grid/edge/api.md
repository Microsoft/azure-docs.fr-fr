---
title: API REST– Azure Event Grid IoT Edge | Microsoft Docs
description: API REST sur Event Grid sur IoT Edge.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 414487d460d897eff787b11915db560706b29eb4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86171752"
---
# <a name="rest-api"></a>API REST
Cet article décrit les API REST d’Azure Event Grid sur IoT Edge

## <a name="common-api-behavior"></a>Comportement courant de l’API

### <a name="base-url"></a>URL de base
Les API suivantes sont exposées par Event Grid sur IoT Edge via HTTP (port 5888) et HTTPS (port 4438).

* URL de base pour HTTP : http://eventgridmodule:5888
* URL de base pour HTTPS : https://eventgridmodule:4438

### <a name="request-query-string"></a>Demander une chaîne de requête
Toutes les requêtes d’API requièrent le paramètre de chaîne de requête suivant :

```?api-version=2019-01-01-preview```

### <a name="request-content-type"></a>Type de contenu de la requête
Toutes les requêtes API doivent avoir une chaîne **Content-Type**.

Dans le cas de **EventGridSchema** ou **CustomSchema**, la valeur de Content-Type peut être l’une des valeurs suivantes :

```Content-Type: application/json```

```Content-Type: application/json; charset=utf-8```

Dans le **CloudEventSchemaV1_0** en mode structuré, la valeur de Content-Type peut être l’une des valeurs suivantes :

```Content-Type: application/cloudevents+json```
    
```Content-Type: application/cloudevents+json; charset=utf-8```
    
```Content-Type: application/cloudevents-batch+json```
    
```Content-Type: application/cloudevents-batch+json; charset=utf-8```

Dans le cas de **CloudEventSchemaV1_0** en mode binaire, reportez-vous à la [documentation](https://github.com/cloudevents/spec/blob/master/http-protocol-binding.md) pour plus d’informations.

### <a name="error-response"></a>Réponse d’erreur
Toutes les API retournent une erreur avec la charge utile suivante :

```json
{
    "error":
    {
        "code": "<HTTP STATUS CODE>",
        "details": 
        {
            "code": "<Detailed Error Code>",
            "message": "..."
        }
    }
}
```

## <a name="manage-topics"></a>Gérer les rubriques

### <a name="put-topic-create--update"></a>Insérer la rubrique (créer / mettre à jour)

**Requête** : ``` PUT /topics/<topic_name>?api-version=2019-01-01-preview ```

**Charge utile** :

```json
    {
        "name": "<topic_name>", // optional, inferred from URL. If specified must match URL topic_name
        "properties":
        {
            "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0" // optional
        }
    }
```

**Réponse**: HTTP 200

**Charge utile** :

```json
{
    "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>",
    "name": "<topic_name>",
    "type": "Microsoft.EventGrid/topics",
    "properties":
    {
        "endpoint": "<get_request_base_url>/topics/<topic_name>/events?api-version=2019-01-01-preview",
        "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0" // populated with EventGridSchema if not explicitly specified in PUT request
    }
}
```

### <a name="get-topic"></a>Obtenir la rubrique

**Requête** : ``` GET /topics/<topic_name>?api-version=2019-01-01-preview ```

**Réponse**: HTTP 200

**Charge utile** :
```json
{
    "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>",
    "name": "<topic_name>",
    "type": "Microsoft.EventGrid/topics",
    "properties":
    {
        "endpoint": "<request_base_url>/topics/<topic_name>/events?api-version=2019-01-01-preview",
        "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0"
    }
}
```

### <a name="get-all-topics"></a>Obtenir toutes les rubriques

**Requête** : ``` GET /topics?api-version=2019-01-01-preview ```

**Réponse**: HTTP 200

**Charge utile** :
```json
[
    {
        "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>",
        "name": "<topic_name>",
        "type": "Microsoft.EventGrid/topics",
        "properties":
        {
            "endpoint": "<request_base_url>/topics/<topic_name>/events?api-version=2019-01-01-preview",
            "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0"
        }
    },
    {
        "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>",
        "name": "<topic_name>",
        "type": "Microsoft.EventGrid/topics",
        "properties":
        {
            "endpoint": "<request_base_url>/topics/<topic_name>/events?api-version=2019-01-01-preview",
            "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0"
        }
    }
]
```

### <a name="delete-topic"></a>Supprimer la rubrique

**Requête** : ``` DELETE /topics/<topic_name>?api-version=2019-01-01-preview ```

**Réponse**: HTTP 200, empty payload

## <a name="manage-event-subscriptions"></a>Gérer les abonnements aux événements
Les exemples de cette section utilisent `EndpointType=Webhook;`. Les exemples JSON pour `EndpointType=EdgeHub / EndpointType=EventGrid` se trouvent dans la section suivante. 

### <a name="put-event-subscription-create--update"></a>Abonnement à l’événement put (créer / mettre à jour)

**Requête** : ``` PUT /topics/<topic_name>/eventSubscriptions/<subscription_name>?api-version=2019-01-01-preview ```

**Charge utile** :
```json
{
    "name": "<subscription_name>", // optional, inferred from URL. If specified must match URL subscription_name
    "properties":
    {
        "topicName": "<topic_name>", // optional, inferred from URL. If specified must match URL topic_name
        "eventDeliverySchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0", // optional
        "retryPolicy":  //optional
        {
            "eventExpiryInMinutes": 120,
            "maxDeliveryAttempts": 50
        },
        "persistencePolicy": "true",
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
            {
                "endpointUrl": "<webhook_url>",
                "maxEventsPerBatch": 10, // optional
                "preferredBatchSizeInKilobytes": 1033 // optional
            }
        },
        "filter": // optional
        {
            "subjectBeginsWith": "...",
            "subjectEndsWith": "...",
            "isSubjectCaseSensitive": true|false,
            "includedEventTypes": ["...", "..."],
            "advancedFilters":
            [
                {
                    "OperatorType": "BoolEquals",
                    "Key": "...",
                    "Value": "..."
                },
                {
                    "OperatorType": "NumberLessThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberLessThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "NumberNotIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "StringIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringNotIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringBeginsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringEndsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringContains",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                }
            ]
        }
    }
}
```

**Réponse**: HTTP 200

**Charge utile** :

```json
{
    "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>/eventSubscriptions/<subscription_name>",
    "name": "<subscription_name>",
    "type": "Microsoft.EventGrid/eventSubscriptions",
    "properties":
    {
        "topicName": "<topic_name>",
        "eventDeliverySchema": "EventGridSchema | CustomEventSchema  | CloudEventSchemaV1_0", // populated with EventGridSchema if not explicitly specified in PUT request
        "retryPolicy":  // only populated if specified in the PUT request
        {
            "eventExpiryInMinutes": 120,
            "maxDeliveryAttempts": 50
        },
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
            {
                "endpointUrl": "<webhook_url>",
                "maxEventsPerBatch": 10, // optional
                "preferredBatchSizeInKilobytes": 1033 // optional
            }
        },
        "filter": // only populated if specified in the PUT request 
        {
            "subjectBeginsWith": "...",
            "subjectEndsWith": "...",
            "isSubjectCaseSensitive": true|false,
            "includedEventTypes": ["...", "..."],
            "advancedFilters":
            [
                {
                    "OperatorType": "BoolEquals",
                    "Key": "...",
                    "Value": "..."
                },
                {
                    "OperatorType": "NumberLessThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberLessThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "NumberNotIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "StringIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringNotIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringBeginsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringEndsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringContains",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                }
            ]
        }
    }
}
```


### <a name="get-event-subscription"></a>Obtenir l’abonnement à l’événement

**Requête** : ``` GET /topics/<topic_name>/eventSubscriptions/<subscription_name>?api-version=2019-01-01-preview ```

**Réponse**: HTTP 200

**Charge utile** :
```json
{
    "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>/eventSubscriptions/<subscription_name>",
    "name": "<subscription_name>",
    "type": "Microsoft.EventGrid/eventSubscriptions",
    "properties":
    {
        "topicName": "<topic_name>",
        "eventDeliverySchema": "EventGridSchema | CustomEventSchema  | CloudEventSchemaV1_0", // populated with EventGridSchema if not explicitly specified in PUT request
        "retryPolicy":  // only populated if specified in the PUT request
        {
            "eventExpiryInMinutes": 120,
            "maxDeliveryAttempts": 50
        },
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
            {
                "endpointUrl": "<webhook_url>",
                "maxEventsPerBatch": 10, // optional
                "preferredBatchSizeInKilobytes": 1033 // optional
            }
        },
        "filter": // only populated if specified in the PUT request 
        {
            "subjectBeginsWith": "...",
            "subjectEndsWith": "...",
            "isSubjectCaseSensitive": true|false,
            "includedEventTypes": ["...", "..."],
            "advancedFilters":
            [
                {
                    "OperatorType": "BoolEquals",
                    "Key": "...",
                    "Value": "..."
                },
                {
                    "OperatorType": "NumberLessThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberLessThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "NumberNotIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "StringIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringNotIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringBeginsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringEndsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringContains",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                }
            ]
        }
    }
}
```

### <a name="get-event-subscriptions"></a>Obtenir les abonnements aux événements

**Requête** : ``` GET /topics/<topic_name>/eventSubscriptions?api-version=2019-01-01-preview ```

**Réponse**: HTTP 200

**Charge utile** :
```json
[
    {
        // same event-subscription json as that returned from Get-EventSubscription above
    },
    {
    },
    ...
]
```

### <a name="delete-event-subscription"></a>Supprimer l’abonnement à l’événement

**Requête** : ``` DELETE /topics/<topic_name>/eventSubscriptions/<subscription_name>?api-version=2019-01-01-preview ```

**Réponse**: HTTP 200, no payload


## <a name="publish-events-api"></a>Publier l’API des événements

### <a name="send-batch-of-events-in-event-grid-schema"></a>Envoyer un lot d’événements (dans le schéma Event Grid)

**Requête** : ``` POST /topics/<topic_name>/events?api-version=2019-01-01-preview ```

```json
[
    {
        "id": "<user-defined-event-id>",
        "topic": "<topic_name>",
        "subject": "",
        "eventType": "",
        "eventTime": ""
        "dataVersion": "",
        "metadataVersion": "1",
        "data": 
            ...
    }
]
```

**Réponse**: HTTP 200, empty payload


**Descriptions des champs de charge utile**
- ```Id``` est obligatoire. Il peut s’agir de n’importe quelle valeur de chaîne remplie par l’appelant. Event Grid n’effectue PAS de détection des doublons ou n’applique aucune sémantique sur ce champ.
- ```Topic``` est facultatif, mais s’il est spécifié, il doit correspondre au topic_name de l’URL de la requête
- ```Subject``` est obligatoire, il peut s’agir de n’importe quelle valeur de chaîne
- ```EventType``` est obligatoire, il peut s’agir de n’importe quelle valeur de chaîne
- ```EventTime``` est obligatoire, il n’est pas validé, mais il doit être une valeur DateTime correcte.
- ```DataVersion``` est obligatoire
- ```MetadataVersion``` est facultatif, s’il est spécifié, il DOIT s’agir d’une chaîne avec la valeur ```"1"```
- ```Data``` est facultatif et peut être n’importe quel jeton JSON (nombre, chaîne, booléen, tableau, objet)

### <a name="send-batch-of-events-in-custom-schema"></a>Envoyer un lot d’événements (dans le schéma personnalisé)

**Requête** : ``` POST /topics/<topic_name>/events?api-version=2019-01-01-preview ```

```json
[
    {
        ...
    }
]
```

**Réponse**: HTTP 200, empty payload


**Restrictions de charge utile**
- DOIT être un tableau d’événements.
- Chaque entrée de tableau DOIT être un objet JSON.
- Aucune autre contrainte (autre que la taille de la charge utile).

## <a name="examples"></a>Exemples

### <a name="set-up-topic-with-eventgrid-schema"></a>Configurer une rubrique avec le schéma EventGrid
Configure une rubrique pour exiger la publication des événements dans **eventgridschema**.

```json
    {
        "name": "myeventgridtopic",
        "properties":
        {
            "inputSchema": "EventGridSchema"
        }
    }
```

### <a name="set-up-topic-with-custom-schema"></a>Configurer une rubrique avec un schéma personnalisé
Configure une rubrique pour exiger la publication d’événements dans `customschema`.

```json
    {
        "name": "mycustomschematopic",
        "properties":
        {
            "inputSchema": "CustomSchema"
        }
    }
```

### <a name="set-up-topic-with-cloud-event-schema"></a>Configurer une rubrique avec un schéma d’événement cloud
Configure une rubrique pour exiger la publication d’événements dans `cloudeventschema`.

```json
    {
        "name": "mycloudeventschematopic",
        "properties":
        {
            "inputSchema": "CloudEventSchemaV1_0"
        }
    }
```

### <a name="set-up-webhook-as-destination-events-to-be-delivered-in-eventgridschema"></a>Configurer le webhook en tant que destination, événements à remettre dans eventgridschema
Utilisez ce type de destination pour envoyer des événements à tout autre module (qui héberge un point de terminaison HTTP) ou à n’importe quel point de terminaison adressable HTTP sur le réseau/Internet.

```json
{
    "properties":
    {
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
            {
                "endpointUrl": "<webhook_url>",
                "eventDeliverySchema": "eventgridschema",
            }
        }
    }
}
```

Contraintes sur l’attribut `endpointUrl` :
- Il ne doit pas avoir la valeur Null.
- Il doit s’agir d’une URL absolue.
- Si outbound__webhook__httpsOnly est défini sur true dans les paramètres EventGridModule, il doit s’agir de HTTPS uniquement.
- Si outbound__webhook__httpsOnly a la valeur false, il peut s’agir de HTTP ou HTTPs.

Contraintes sur la propriété `eventDeliverySchema` :
- Elle doit correspondre au schéma d’entrée de la rubrique d’abonnement.
- Elle ne peut pas avoir la valeur Null. Par défaut, il s’agit du schéma d’entrée de la rubrique.

### <a name="set-up-iot-edge-as-destination"></a>Configurer IoT Edge comme destination

Utilisez cette destination pour envoyer des événements à IoT Edge Hub et être soumis au sous-système de routage/filtrage/transfert du hub de périphérie.

```json
{
    "properties":
    {
        "destination":
        {
            "endpointType": "EdgeHub",
            "properties":
            {
                "outputName": "<eventgridmodule_output_port_name>"
            }
        }
    }
}
```

### <a name="set-up-event-grid-cloud-as-destination"></a>Configurer le cloud Event Grid comme destination

Utilisez cette destination pour envoyer des événements à Event Grid dans le cloud (Azure). Vous devez d’abord configurer une rubrique utilisateur dans le cloud à laquelle les événements doivent être envoyés, avant de créer un abonnement aux événements sur la périphérie.

```json
{
    "properties":
    {
        "destination":
        {
            "endpointType": "EventGrid",
            "properties":
            {
                "endpointUrl": "<eventgrid_user_topic_url>",
                "sasKey": "<user_topic_sas_key>",
                "topicName": "<new value to populate in forwarded EventGridEvent.Topic>" // if not specified, the Topic field on every event gets nulled out before being sent to Azure Event Grid
            }
        }
    }
}
```

EndpointUrl :
- Il ne doit pas avoir la valeur Null.
- Il doit s’agir d’une URL absolue.
- Le chemin d’accès `/api/events` doit être défini dans le chemin d’accès de l’URL de la requête.
- Il doit avoir `api-version=2018-01-01` dans la chaîne de requête.
- Si outbound__eventgrid__httpsOnly est défini sur true dans les paramètres EventGridModule (true par défaut), il doit s’agir de HTTPS uniquement.
- Si outbound__eventgrid__httpsOnly a la valeur false, il peut s’agir de HTTP ou HTTPs.
- Si outbound__eventgrid__allowInvalidHostnames a la valeur false (false par défaut), il doit cibler l’un des points de terminaison suivants :
   - `eventgrid.azure.net`
   - `eventgrid.azure.us`
   - `eventgrid.azure.cn`

SasKey :
- Ne doit pas avoir la valeur Null.

TopicName :
- Si Subscription.EventDeliverySchema est défini sur EventGridSchema, la valeur de ce champ est placée dans le champ Topic de chaque événement avant d’être transférée vers Event Grid dans le cloud.
- Si Subscription.EventDeliverySchema est défini sur CustomEventSchema, cette propriété est ignorée et la charge utile d’événement personnalisée est transférée exactement telle qu’elle a été reçue.

## <a name="set-up-event-hubs-as-a-destination"></a>Configurer Event Hubs en tant que destination

Pour publier sur un Event Hub, définissez `endpointType` sur `eventHub` et indiquez :

* connectionString : chaîne de connexion pour l'Event Hub spécifique que vous ciblez, générée par le biais d’une stratégie d’accès partagé.

    >[!NOTE]
    > La chaîne de connexion doit être propre à l’entité. L’utilisation d’une chaîne de connexion d’espace de noms ne fonctionne pas. Vous pouvez générer une chaîne de connexion spécifique à une entité en accédant au hub d’événements sur lequel vous souhaitez publier dans le portail Azure, et en cliquant sur **Stratégies d’accès partagé** pour générer une chaîne de connexion propre à l’entité.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "eventHub",
              "properties": {
                "connectionString": "<your-event-hub-connection-string>"
              }
            }
          }
        }
    ```

## <a name="set-up-service-bus-queues-as-a-destination"></a>Configurer des files d’attente Service Bus en tant que destination

Pour publier dans une file d’attente Service Bus, définissez `endpointType` sur `serviceBusQueue` et indiquez :

* connectionString : chaîne de connexion pour la file d’attente Service Bus spécifique que vous ciblez, générée par le biais d’une stratégie d’accès partagé.

    >[!NOTE]
    > La chaîne de connexion doit être propre à l’entité. L’utilisation d’une chaîne de connexion d’espace de noms ne fonctionne pas. Générez une chaîne de connexion spécifique à une entité en accédant à la file d’attente Service Bus sur laquelle vous souhaitez publier dans le portail Azure, et en cliquant sur **Stratégies d’accès partagé** pour générer une chaîne de connexion propre à l’entité.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "serviceBusQueue",
              "properties": {
                "connectionString": "<your-service-bus-queue-connection-string>"
              }
            }
          }
        }
    ```

## <a name="set-up-service-bus-topics-as-a-destination"></a>Configurer des rubriques Service Bus en tant que destination

Pour publier dans une rubrique Service Bus, définissez `endpointType` sur `serviceBusTopic` et indiquez :

* connectionString : chaîne de connexion pour la rubrique Service Bus spécifique que vous ciblez, générée par le biais d’une stratégie d’accès partagé.

    >[!NOTE]
    > La chaîne de connexion doit être propre à l’entité. L’utilisation d’une chaîne de connexion d’espace de noms ne fonctionne pas. Générez une chaîne de connexion spécifique à une entité en accédant à la rubrique Service Bus sur laquelle vous souhaitez publier dans le portail Azure, et en cliquant sur **Stratégies d’accès partagé** pour générer une chaîne de connexion propre à l’entité.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "serviceBusTopic",
              "properties": {
                "connectionString": "<your-service-bus-topic-connection-string>"
              }
            }
          }
        }
    ```

## <a name="set-up-storage-queues-as-a-destination"></a>Configurer des files d’attente de stockage en tant que destination

Pour publier dans une file d’attente de stockage, définissez `endpointType` sur `storageQueue` et indiquez :

* queueName : nom de la file d’attente de stockage dans laquelle vous publiez.
* connectionString : chaîne de connexion pour le compte de stockage dans lequel se trouve la file d'attente de stockage.

    >[!NOTE]
    > Contrairement aux Event Hubs, aux rubriques et files d’attente Service Bus, la chaîne de connexion utilisée pour les files d’attente de stockage n’est pas propre à l’entité. C’est la chaîne de connexion du compte de stockage qui doit être utilisée à la place.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "storageQueue",
              "properties": {
                "queueName": "<your-storage-queue-name>",
                "connectionString": "<your-storage-account-connection-string>"
              }
            }
          }
        }
    ```