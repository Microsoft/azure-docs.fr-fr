---
title: Gestion des événements
titleSuffix: An Azure Communication Services concept document
description: Utilisez Azure Event Grid pour déclencher des processus en fonction d’actions qui se produisent dans un service de communication.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 09/30/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: e661b99e3c5028f40ea69ddedc22c7ee2895acec
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94888723"
---
# <a name="event-handling-in-azure-communication-services"></a>Gestion des événements dans Azure Communication Services

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

Azure Communication Services s’intègre à [Azure Event Grid](https://azure.microsoft.com/services/event-grid/) pour fournir des notifications d’événements en temps réel de manière fiable, scalable et sécurisée. L’objectif de cet article est de vous aider à configurer vos applications pour être à l’écoute des événements Communication Services. Par exemple, vous pouvez mettre à jour une base de données, créer un élément de travail et envoyer une notification Push chaque fois qu’un SMS est reçu par un numéro de téléphone associé à votre ressource Communication Services.

Azure Event Grid est un service de routage d’événement complètement managé qui utilise le modèle publication-abonnement. Event Grid offre une prise en charge intégrée des services Azure comme [Azure Functions](../../azure-functions/functions-overview.md) et [Azure Logic Apps](../../azure-functions/functions-overview.md). Il peut envoyer des alertes d’événement à des services non-Azure à l’aide de webhooks. Pour obtenir une liste complète des gestionnaires d’événements qui prennent en charge Event Grid, consultez [Présentation d’Azure Event Grid](../../event-grid/overview.md).

:::image type="content" source="https://docs.microsoft.com/azure/event-grid/media/overview/functional-model.png" alt-text="Schéma montrant le modèle d’événement d’Azure Event Grid.":::

## <a name="events-types"></a>Types d’événements

Event Grid utilise les [abonnements aux événements](../../event-grid/concepts.md#event-subscriptions) pour acheminer les messages d’événements vers les abonnés. 

Azure Communication Services émet les types d’événements suivants :

| Type d'événement                                                  | Description                                                                                    |
| ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Microsoft.Communication.SMSReceived                         | Publié lorsqu’un SMS est reçu par un numéro de téléphone associé au service de communication. |
| Microsoft.Communication.SMSDeliveryReportReceived           | Publié lorsqu’un rapport de remise est reçu pour un SMS envoyé par le service de communication.     |
| Microsoft.Communication.ChatMessageReceived                | Publié lorsqu’un message est reçu pour un utilisateur dans un fil de conversation dont il est membre.        |
| Microsoft.Communication.ChatMessageEdited                   | Publié lorsqu’un message est édité dans un fil de conversation dont l’utilisateur est membre.                |
| Microsoft.Communication.ChatMessageDeleted                  | Publié lorsqu’un message est supprimé dans un fil de conversation dont l’utilisateur est membre.               |
| Microsoft.Communication.ChatThreadCreatedWithUser           | Publié lorsque l’utilisateur est ajouté en tant que membre au moment de la création d’un fil de conversation.           |
| Microsoft.Communication.ChatThreadWithUserDeleted           | Publié lorsqu’un fil de conversation est supprimé dont l’utilisateur est membre.                           |
| Microsoft.Communication.ChatThreadPropertiesUpdatedPerUser  | Publié lors de la mise à jour des propriétés d’un fil de conversation dont l’utilisateur est membre.              |
| Microsoft.Communication.ChatMemberAddedToThreadWithUser     | Publié lorsque l’utilisateur est ajouté en tant que membre à un fil de conversation.                                   |
| Microsoft.Communication.ChatMemberRemovedFromThreadWithUser | Publié lorsque l’utilisateur est supprimé d’un fil de conversation.                                         |

Vous pouvez utiliser le portail Azure ou Azure CLI pour vous abonner aux événements émis par votre ressource Communication Services. Démarrez la gestion des événements en examinant [Guide pratique pour gérer les événements SMS dans Communication Services](../quickstarts/telephony-sms/handle-sms-events.md)


## <a name="event-subjects"></a>Objets des événements

Le champ `subject` de tous les événements Communication Services identifie l’utilisateur, le numéro de téléphone ou l’entité que l’événement cible. Les préfixes courants sont utilisés pour autoriser un [filtrage Event Grid](../../event-grid/event-filtering.md) simple.

| Préfixe de l’objet                              | Entité du service de communication |
| ------------------------------------------- | ---------------------------- |
| `phonenumber/`                              | Numéro de téléphone RTPC            |
| `user/`                                     | Utilisateur de Communication Services  |
| `thread/`                                   | Fil de conversation.                 |

L’exemple suivant montre un filtre pour tous les SMS et tous les rapports de remise envoyés à tous les numéros de téléphone dotés de l’indicatif régional 555 détenus par une ressource Communication Services :

```json
"filter": {
  "includedEventTypes": [
    "Microsoft.Communication.SMSReceived",
    "Microsoft.Communication.SMSDeliveryReportReceived"
  ],
  "subjectBeginsWith": "phonenumber/1555",
}
```

## <a name="sample-event-responses"></a>Exemples de réponses à des événements

Quand un événement est déclenché, le service Event Grid envoie les données relatives à cet événement aux points de terminaison d’abonnement.

Cette section contient un exemple de ce à quoi ces données ressembleraient pour chaque événement.

### <a name="microsoftcommunicationsmsdeliveryreportreceived-event"></a>Événement Microsoft.Communication.SMSDeliveryReportReceived

```json
[{
  "id": "Outgoing_202009180022138813a09b-0cbf-4304-9b03-1546683bb910",
  "topic": "/subscriptions/{subscription-id}/resourceGroups/{group-name}/providers/microsoft.communication/communicationservices/{communication-services-resource-name}",
  "subject": "/phonenumber/15555555555",
  "data": {
    "MessageId": "Outgoing_202009180022138813a09b-0cbf-4304-9b03-1546683bb910",
    "From": "15555555555",
    "To": "+15555555555",
    "DeliveryStatus": "Delivered",
    "DeliveryStatusDetails": "No error.",
    "ReceivedTimestamp": "2020-09-18T00:22:20.2855749Z",
    "DeliveryAttempts": [
      {
        "Timestamp": "2020-09-18T00:22:14.9315918Z",
        "SegmentsSucceeded": 1,
        "SegmentsFailed": 0
      }
    ]
  },
  "eventType": "Microsoft.Communication.SMSDeliveryReportReceived",
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-09-18T00:22:20Z"
}]
```
### <a name="microsoftcommunicationsmsreceived-event"></a>Événement Microsoft.Communication.SMSReceived

```json
[{
  "id": "Incoming_20200918002745d29ebbea-3341-4466-9690-0a03af35228e",
  "topic": "/subscriptions/50ad1522-5c2c-4d9a-a6c8-67c11ecb75b8/resourcegroups/acse2e/providers/microsoft.communication/communicationservices/{communication-services-resource-name}",
  "subject": "/phonenumber/15555555555",
  "data": {
    "MessageId": "Incoming_20200918002745d29ebbea-3341-4466-9690-0a03af35228e",
    "From": "15555555555",
    "To": "15555555555",
    "Message": "Great to connect with ACS events ",
    "ReceivedTimestamp": "2020-09-18T00:27:45.32Z"
  },
  "eventType": "Microsoft.Communication.SMSReceived",
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-09-18T00:27:47Z"
}]
```

### <a name="microsoftcommunicationchatmessagereceived-event"></a>Événement Microsoft.Communication.ChatMessageReceived

```json
[{
  "id": "c13afb5f-d975-4296-a8ef-348c8fc496ee",
  "topic": "/subscriptions/{subscription-id}/resourceGroups/{group-name}/providers/Microsoft.Communication/communicationServices/{communication-services-resource-name}",
  "subject": "thread/{thread-id}/sender/{id-of-message-sender}/recipient/{id-of-message-recipient}",
  "data": {
    "messageBody": "Welcome to Azure Communication Services",
    "messageId": "1600389507167",
    "senderId": "8:acs:fac4607d-d2d0-40e5-84df-6f32ebd1251a_00000005-3e0d-e5aa-0e04-343a0d00037c",
    "senderDisplayName": "John",
    "composeTime": "2020-09-18T00:38:27.167Z",
    "type": "Text",
    "version": 1600389507167,
    "recipientId": "8:acs:fac4607d-d2d0-40e5-84df-6f32ebd1251a_00000005-3e1a-3090-6a0b-343a0d000409",
    "transactionId": "WGW1YmwRzkupk0UI0QA9ZA.1.1.1.1.1797783722.1.9",
    "threadId": "19:46df844a4c064bfaa2b3b30e385d1018@thread.v2"
  },
  "eventType": "Microsoft.Communication.ChatMessageReceived",
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-09-18T00:38:28.0946757Z"
}
]
```

### <a name="microsoftcommunicationchatmessageedited-event"></a>Événement Microsoft.Communication.ChatMessageEdited

```json
[{
  "id": "18247662-e94a-40cc-8d2f-f7357365309e",
  "topic": "/subscriptions/{subscription-id}/resourceGroups/{group-name}/providers/Microsoft.Communication/communicationServices/{communication-services-resource-name}",
  "subject": "thread/19:6d20c2f921cd402ead7d1b31b0d030cd@thread.v2/sender/8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003fe/recipient/8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003f0",
  "data": {
    "editTime": "2020-09-18T00:48:47.361Z",
    "messageBody": "Let's Chat about new communication services.",
    "messageId": "1600390097873",
    "senderId": "8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003fe",
    "senderDisplayName": "Bob(Admin)",
    "composeTime": "2020-09-18T00:48:17.873Z",
    "type": "Text",
    "version": 1600390127361,
    "recipientId": "8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003f0",
    "transactionId": "bbopOa1JZEW5NDDFLgH1ZQ.2.1.2.1.1822032097.1.5",
    "threadId": "19:6d20c2f921cd402ead7d1b31b0d030cd@thread.v2"
  },
  "eventType": "Microsoft.Communication.ChatMessageEdited",
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-09-18T00:48:48.037823Z"
}]
```

### <a name="microsoftcommunicationchatmessagedeleted-event"></a>Événement Microsoft.Communication.ChatMessageDeleted
```json
[{
  "id": "08034616-cf11-4fc2-b402-88963b93d083",
  "topic": "/subscriptions/{subscription-id}/resourceGroups/{group-name}/providers/Microsoft.Communication/communicationServices/{communication-services-resource-name}",
  "subject": "thread/19:6d20c2f921cd402ead7d1b31b0d030cd@thread.v2/sender/8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003fe/recipient/8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003f0",
  "data": {
    "deleteTime": "2020-09-18T00:48:47.361Z",
    "messageId": "1600390099195",
    "senderId": "8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003fe",
    "senderDisplayName": "Bob(Admin)",
    "composeTime": "2020-09-18T00:48:19.195Z",
    "type": "Text",
    "version": 1600390152154,
    "recipientId": "8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003f0",
    "transactionId": "mAxUjeTsG06NpObXkFcjVQ.1.1.2.1.1823015063.1.5",
    "threadId": "19:6d20c2f921cd402ead7d1b31b0d030cd@thread.v2"
  },
  "eventType": "Microsoft.Communication.ChatMessageDeleted",
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-09-18T00:49:12.6698791Z"
}]
```

### <a name="microsoftcommunicationchatthreadcreatedwithuser-event"></a>Événement Microsoft.Communication.ChatThreadCreatedWithUser 

```json
[{
  "id": "06c7c381-bb0a-4fff-aedd-919df1d52137",
  "topic": "/subscriptions/{subscription-id}/resourceGroups/{group-name}/providers/Microsoft.Communication/communicationServices/{communication-services-resource-name}",
  "subject": "thread/19:7bdf5504a23f41a79d1bd472dd40044a@thread.v2/createdBy/8:acs:73551687-f8c8-48a7-bf06-d8263f15b02a_00000005-3e5f-1bc6-f40f-343a0d0003fe/recipient/8:acs:73551687-f8c8-48a7-bf06-d8263f15b02a_00000005-3e5f-1bc6-f40f-343a0d0003f0",
  "data": {
    "createdBy": "8:acs:73551687-f8c8-48a7-bf06-d8263f15b02a_06014f-6001fc107f",
    "properties": {
      "topic": "Chat about new commuication services",
    },
    "members": [
      {
        "displayName": "Bob",
        "memberId": "8:acs:73551687-f8c8-48a7-bf06-d8263f15b02a_00000005-3e5f-1bc6-f40f-343a0d0003f0"
      },
      {
        "displayName": "John",
        "memberId": "8:acs:73551687-f8c8-48a7-bf06-d8263f15b02a_00000005-3e5f-1bc6-f40f-343a0d0003f1"
      }
    ],
    "createTime": "2020-09-17T22:06:09.988Z",
    "version": 1600380369988,
    "recipientId": "8:acs:73551687-f8c8-48a7-bf06-d8263f15b02a_00000005-3e5f-1bc6-f40f-343a0d0003f0",
    "transactionId": "9ZxrGXVXCkOTygd5iwsvAQ.1.1.1.1.1440874720.1.1",
    "threadId": "19:7bdf5504a23f41a79d1bd472dd40044a@thread.v2"
  },
  "eventType": "Microsoft.Communication.ChatThreadCreatedWithUser",
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-09-17T22:06:10.3235137Z"
}]
```

### <a name="microsoftcommunicationchatthreadwithuserdeleted-event"></a>Événement Microsoft.Communication.ChatThreadWithUserDeleted

```json
[{
  "id": "7f4fa31b-e95e-428b-a6e8-53e2553620ad",
  "topic":"/subscriptions/{subscription-id}/resourceGroups/{group-name}/providers/Microsoft.Communication/communicationServices/{communication-services-resource-name}",
  "subject": "thread/19:6d20c2f921cd402ead7d1b31b0d030cd@thread.v2/deletedBy/8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003fe/recipient/8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003f0",
  "data": {
    "deletedBy": "8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003fe",
    "deleteTime": "2020-09-18T00:49:26.3694459Z",
    "createTime": "2020-09-18T00:46:41.559Z",
    "version": 1600390071625,
    "recipientId": "8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003f0",
    "transactionId": "MoZlSM2j7kSD2b5X8bjH7Q.1.1.2.1.1823539230.1.1",
    "threadId": "19:6d20c2f921cd402ead7d1b31b0d030cd@thread.v2"
  },
  "eventType": "Microsoft.Communication.ChatThreadWithUserDeleted",
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-09-18T00:49:26.4269056Z"
}]
```

### <a name="microsoftcommunicationchatthreadpropertiesupdatedperuser-event"></a>Événement Microsoft.Communication.ChatThreadPropertiesUpdatedPerUser 

```json
[{
  "id": "47a66834-57d7-4f77-9c7d-676d45524982",
  "topic": "/subscriptions/{subscription-id}/resourceGroups/{group-name}/providers/Microsoft.Communication/communicationServices/{communication-services-resource-name}",
  "subject": "thread/19:a33a128babf04431b7fe8cbca82f4238@thread.v2/editedBy/8:acs:fac4607d-d2d0-40e5-84df-6f32ebd1251a_00000005-3e88-2b7f-ac00-343a0d0005a8/recipient/8:acs:fac4607d-d2d0-40e5-84df-6f32ebd1251a_00000005-3e88-15fa-ac00-343a0d0005a7",
  "data": {
    "editedBy": "8:acs:fac4607d-d2d0-40e5-84df-6f32ebd1251a_00000005-3e88-2b7f-ac00-343a0d0005a8",
    "editTime": "2020-09-18T00:40:38.4914428Z",
    "properties": {
      "topic": "Communication in Azure"
    },
    "createTime": "2020-09-18T00:39:02.541Z",
    "version": 1600389638481,
    "recipientId": "8:acs:fac4607d-d2d0-40e5-84df-6f32ebd1251a_00000005-3e88-15fa-ac00-343a0d0005a7",
    "transactionId": "+ah9tVwqNkCT6nUGCKIvAg.1.1.1.1.1802895561.1.1",
    "threadId": "19:a33a128babf04431b7fe8cbca82f4238@thread.v2"
  },
  "eventType": "Microsoft.Communication.ChatThreadPropertiesUpdatedPerUser",
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-09-18T00:40:38.5804349Z"
}]
```

### <a name="microsoftcommunicationchatmemberaddedtothreadwithuser-event"></a>Événement Microsoft.Communication.ChatMemberAddedToThreadWithUser

```json
[{
  "id": "4abd2b49-d1a9-4fcc-9cd7-170fa5d96443",
  "topic": "/subscriptions/{subscription-id}/resourceGroups/{group-name}/providers/Microsoft.Communication/communicationServices/{communication-services-resource-name}",
  "subject": "thread/19:6d20c2f921cd402ead7d1b31b0d030cd@thread.v2/memberAdded/8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003fe/recipient/8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003f0",
  "data": {
    "time": "2020-09-18T00:47:13.1867087Z",
    "addedBy": "8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003f1",
    "memberAdded": {
      "displayName": "John Smith",
      "memberId": "8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003fe"
    },
    "createTime": "2020-09-18T00:46:41.559Z",
    "version": 1600390033176,
    "recipientId": "8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003f0",
    "transactionId": "pVIjw/pHEEKUOUJ2DAAl5A.1.1.1.1.1818361951.1.1",
    "threadId": "19:6d20c2f921cd402ead7d1b31b0d030cd@thread.v2"
  },
  "eventType": "Microsoft.Communication.ChatMemberAddedToThreadWithUser",
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-09-18T00:47:13.2342692Z"
}]
```

### <a name="microsoftcommunicationchatmemberremovedfromthreadwithuser-event"></a>Événement Microsoft.Communication.ChatMemberRemovedFromThreadWithUser

```json
[{
  "id": "b3701976-1ea2-4d66-be68-4ec4fc1b4b96",
  "topic": "/subscriptions/{subscription-id}/resourceGroups/{group-name}/providers/Microsoft.Communication/communicationServices/{communication-services-resource-name}",
  "subject": "thread/19:6d20c2f921cd402ead7d1b31b0d030cd@thread.v2/memberRemoved/8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003fe/recipient/8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003f0",
  "data": {
    "time": "2020-09-18T00:47:51.1461742Z",
    "removedBy": "8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003f1",
    "memberRemoved": {
      "displayName": "John",
      "memberId": "8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003fe"
    },
    "createTime": "2020-09-18T00:46:41.559Z",
    "version": 1600390071131,
    "recipientId": "8:acs:5354158b-17b7-489c-9380-95d8821ff76b_00000005-3e5f-1bc6-f40f-343a0d0003f0",
    "transactionId": "G9Y+UbjVmEuxAG3O4bEyvw.1.1.1.1.1819803816.1.1",
    "threadId": "19:6d20c2f921cd402ead7d1b31b0d030cd@thread.v2"
  },
  "eventType": "Microsoft.Communication.ChatMemberRemovedFromThreadWithUser",
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-09-18T00:47:51.2244511Z"
}]
```

## <a name="quickstarts-and-how-tos"></a>Guides de démarrage rapide et guides pratiques

| Titre | Description |
| --- | --- |
| [Guide pratique pour gérer les événements SMS dans Communication Services](../quickstarts/telephony-sms/handle-sms-events.md) | Gestion de tous les événements SMS reçus par votre service de communication à l’aide d’un webhook. |


## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation d’Azure Event Grid, consultez [Présentation d’Event Grid](../../event-grid/overview.md).
* Pour obtenir une présentation des concepts d’Azure Event Grid, consultez [Concepts utilisés dans Azure Event Grid](../../event-grid/concepts.md).
* Pour obtenir une présentation des rubriques système d’Azure Event Grid, consultez [Rubriques système dans Azure Event Grid](../../event-grid/system-topics.md).