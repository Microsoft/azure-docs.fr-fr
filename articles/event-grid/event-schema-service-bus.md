---
title: Azure Service Bus en tant que source Event Grid
description: Décrit les propriétés qui sont fournies pour les événements Service Bus avec Azure Event Grid
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 81293321b3a8fb989023a231c905996b4059bd81
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86121132"
---
# <a name="azure-service-bus-as-an-event-grid-source"></a>Azure Service Bus en tant que source Event Grid

Cet article décrit les propriétés et le schéma des événements Service Bus. Pour une présentation des schémas d’événements, consultez [Schéma d’événements Azure Event Grid](event-schema.md).

## <a name="event-grid-event-schema"></a>Schéma d’événement Event Grid

### <a name="available-event-types"></a>Types d’événement disponibles

Service Bus émet les types d’événements suivants :

| Type d'événement | Description |
| ---------- | ----------- |
| Microsoft.ServiceBus.ActiveMessagesAvailableWithNoListeners | Événement levé lorsque des messages actifs sont présents dans une file d’attente ou un abonnement et qu’aucun récepteur n’est à l’écoute. |
| Microsoft.ServiceBus.DeadletterMessagesAvailableWithNoListener | Événement levé lorsque des messages actifs sont présents dans une file d’attente de lettres mortes et qu’aucun récepteur n’est actif. |

### <a name="example-event"></a>Exemple d’événement

L’exemple suivant montre le schéma d’un événement avec des messages actifs sans récepteurs :

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourcegroups/{your-rg}/providers/Microsoft.ServiceBus/namespaces/{your-service-bus-namespace}",
  "subject": "topics/{your-service-bus-topic}/subscriptions/{your-service-bus-subscription}",
  "eventType": "Microsoft.ServiceBus.ActiveMessagesAvailableWithNoListeners",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://{your-service-bus-namespace}.servicebus.windows.net/{your-topic}/subscriptions/{your-service-bus-subscription}/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

Le schéma pour un événement de file d’attente de lettres mortes est similaire :

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourcegroups/{your-rg}/providers/Microsoft.ServiceBus/namespaces/{your-service-bus-namespace}",
  "subject": "topics/{your-service-bus-topic}/subscriptions/{your-service-bus-subscription}",
  "eventType": "Microsoft.ServiceBus.DeadletterMessagesAvailableWithNoListener",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://{your-service-bus-namespace}.servicebus.windows.net/{your-topic}/subscriptions/{your-service-bus-subscription}/$deadletterqueue/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
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
| data | object | Données d’événement de stockage Blob. |
| dataVersion | string | Version du schéma de l’objet de données. Le serveur de publication définit la version du schéma. |
| metadataVersion | string | Version du schéma des métadonnées d’événement. Event Grid définit le schéma des propriétés de niveau supérieur. Event Grid fournit cette valeur. |

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| namespaceName | string | Espace de noms Service Bus dans lequel figure la ressource. |
| requestUri | string | URI vers la file d’attente spécifique ou l’abonnement qui génère l’événement. |
| entityType | string | Type d’entité Service Bus générant des événements (file d’attente ou abonnement). |
| queueName | string | File d’attente contenant des messages actives en cas d’abonnement à une file d’attente. Valeur null si des rubriques / abonnements sont utilisés. |
| topicName | string | Rubrique à laquelle appartient l’abonnement Service Bus contenant les messages actifs. Valeur null si une file d’attente est utilisée. |
| subscriptionName | string | Abonnement Service Bus contenant les messages actifs. Valeur null si une file d’attente est utilisée. |

## <a name="tutorials-and-how-tos"></a>Tutoriels et articles de procédures
|Intitulé  |Description  |
|---------|---------|
| [Tutoriel : Exemples d’intégration d’Azure Service Bus et d’Azure Event Grid](../service-bus-messaging/service-bus-to-event-grid-integration-example.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Event Grid envoie des messages à partir de la rubrique Service Bus à l’application de fonction et à l’application logique. |
| [Intégration d’Azure Service Bus à Event Grid](../service-bus-messaging/service-bus-to-event-grid-integration-concept.md) | Vue d’ensemble de l’intégration de Service Bus à Event Grid. |

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation d’Azure Event Grid, consultez [Présentation d’Event Grid](overview.md).
* Pour plus d’informations sur la création d’un abonnement Azure Event Grid, consultez [Schéma d’abonnement à Event Grid](subscription-creation-schema.md).
* Pour plus d’informations sur l’utilisation d’Azure Event Grid avec Service Bus, consultez la page [Vue d’ensemble de l’intégration d’Azure Service Bus et Event Grid](../service-bus-messaging/service-bus-to-event-grid-integration-concept.md).
* Essayez de [recevoir des événements Service Bus avec Functions ou Logic Apps](../service-bus-messaging/service-bus-to-event-grid-integration-example.md?toc=%2fazure%2fevent-grid%2ftoc.json).
