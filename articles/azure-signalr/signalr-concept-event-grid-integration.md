---
title: Réagir aux événements Azure SignalR Service
description: Utilisez Azure Event Grid pour vous abonner à des événements Azure SignalR Service. D’autres services en aval peuvent être déclenchés par ces événements.
services: azure-signalr,event-grid
author: chenyl
ms.author: chenyl
ms.reviewer: zhshang
ms.date: 11/13/2019
ms.topic: conceptual
ms.service: signalr
ms.openlocfilehash: 77c8887ac19c6ce4c7d83734bdd2b44d9213914d
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92151109"
---
# <a name="reacting-to-azure-signalr-service-events"></a>Réaction aux événements d’Azure SignalR Service

Les événements Azure SignalR Service permettent aux applications de réagir aux connexions établies ou interrompues à l’aide d’architectures sans serveur modernes. et sans qu’il soit nécessaire de faire appel à du code complexe ou à des services d’interrogation coûteux et inefficaces.  Au lieu de cela, les événements sont envoyés (push) via [Azure Event Grid](https://azure.microsoft.com/services/event-grid/) aux abonnés, comme [Azure Functions](https://azure.microsoft.com/services/functions/), [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/), ou même à votre propre écouteur HTTP personnalisé. Avec Azure SignalR, vous ne payez que ce que vous utilisez.

Les événements Azure SignalR Service sont envoyés de manière fiable au service Event Grid qui fournit des services de livraison fiables à vos applications via des stratégies enrichies de nouvelle tentative et de livraison de lettres mortes. Pour plus d’informations, consultez [Distribution et nouvelle tentative de distribution de messages avec Azure Grid](../event-grid/delivery-and-retry.md).

![Modèle de Event Grid](/azure/event-grid/media/overview/functional-model.png)

## <a name="serverless-state"></a>État sans serveur
Les événements Azure SignalR Service ne sont actifs que lorsque les connexions clientes sont dans un état serverless. Si un client n’achemine pas vers un serveur hub, il passe à l’état serverless. Le mode classique fonctionne uniquement lorsque le hub utilisé pour les connexions clientes n’a pas de serveur hub. Il est recommandé d’utiliser le mode serverless. Pour plus d’informations sur le mode de service, consultez [Comment choisir le mode de service](https://github.com/Azure/azure-signalr/blob/dev/docs/faq.md#what-is-the-meaning-of-service-mode-defaultserverlessclassic-how-can-i-choose).

## <a name="available-azure-signalr-service-events"></a>Événements Azure SignalR Service disponibles
Event Grid utilise les [abonnements aux événements](../event-grid/concepts.md#event-subscriptions) pour acheminer les messages d’événements vers les abonnés. Les abonnements d’événement Azure SignalR Service prennent en charge deux types d’événements :  

|Nom de l'événement|Description|
|----------|-----------|
|`Microsoft.SignalRService.ClientConnectionConnected`|Déclenché lorsqu’une connexion client est établie.|
|`Microsoft.SignalRService.ClientConnectionDisconnected`|Déclenché lorsqu’une connexion client est interrompue.|

## <a name="event-schema"></a>Schéma d’événement
Les événements Azure SignalR Service contiennent toutes les informations dont vous avez besoin pour répondre aux modifications de vos données. Vous pouvez identifier un événement Azure SignalR Service avec la propriété eventType qui commence par « Microsoft.SignalRService ». Plus d’informations sur l’utilisation des propriétés d’événement Event Grid sont documentées dans [schéma d’événement Event Grid](../event-grid/event-schema.md).  

Voici un exemple d’événement de connexion client :
```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/signalr-rg/providers/Microsoft.SignalRService/SignalR/signalr-resource",
  "subject": "/hub/chat",
  "eventType": "Microsoft.SignalRService.ClientConnectionConnected",
  "eventTime": "2019-06-10T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "timestamp": "2019-06-10T18:41:00.9584103Z",
    "hubName": "chat",
    "connectionId": "crH0uxVSvP61p5wkFY1x1A",
    "userId": "user-eymwyo23"
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

Pour plus d’informations, consultez [Schéma d’événements SignalR Service](../event-grid/event-schema-azure-signalr.md).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur Event Grid et essayer les événements Azure SignalR Service :

> [!div class="nextstepaction"]
> [Essayez un exemple d’intégration Event Grid avec Azure SignalR Service](./signalr-howto-event-grid-integration.md)
> [À propos d’Event Grid](../event-grid/overview.md)