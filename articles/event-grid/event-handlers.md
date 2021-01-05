---
title: Gestionnaires d’événements Azure Event Grid
description: Décrit les gestionnaires d’événements pris en charge pour Azure Event Grid. Azure Automation, Functions, Event Hubs, Connexions hybrides, Logic Apps, Service Bus, Stockage File d’attente, Webhooks.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 76f7fff3e0a3e1c0c554e452987bf944503358b8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86117013"
---
# <a name="event-handlers-in-azure-event-grid"></a>Gestionnaires d’événements dans Azure Event Grid
Un gestionnaire d’événements désigne l’endroit où l’événement est envoyé. Le gestionnaire effectue des actions supplémentaires pour traiter l’événement. Plusieurs services Azure sont automatiquement configurés pour gérer les événements. Vous pouvez également utiliser n’importe quel webhook pour la gestion des événements. À cette fin, il n’est pas nécessaire que le webhook soit hébergé dans Azure. Event Grid ne prend en charge que les points de terminaison Webhook HTTPS.

## <a name="supported-event-handlers"></a>Gestionnaires d’événements pris en charge
Voici les gestionnaires d’événements pris en charge : 

- [Webhooks](handler-webhooks.md). Les runbooks et applications logiques Azure Automation sont pris en charge via des webhooks. 
- [Azure Functions](handler-functions.md)
- [Event Hubs](handler-event-hubs.md)
- [Connexions hybrides Relay](handler-relay-hybrid-connections.md)
- [Files d’attente et rubriques Service Bus](handler-service-bus.md)
- [Files d’attente de stockage](handler-storage-queues.md)

## <a name="next-steps"></a>Étapes suivantes
- Pour une présentation d’Event Grid, consultez [À propos d’Event Grid](overview.md).
