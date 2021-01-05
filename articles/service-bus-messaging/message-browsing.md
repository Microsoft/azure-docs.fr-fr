---
title: Azure Service Bus – Exploration des messages
description: Le parcours et l’aperçu des messages Service Bus permettent à un client Azure Service Bus d’énumérer tous les messages d’une file d’attente ou d’un abonnement.
ms.topic: article
ms.date: 11/11/2020
ms.openlocfilehash: c52c9c967d4eada1a931e188ed4d25f7691cfb91
ms.sourcegitcommit: dc342bef86e822358efe2d363958f6075bcfc22a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94553639"
---
# <a name="message-browsing"></a>Parcours des messages

Le parcours des messages, ou « Peek », permet à un client Service Bus d’énumérer tous les messages d’une file d’attente ou d’un abonnement, à des fins de diagnostic et de débogage.

L’opération Peek sur une file d’attente renvoie tous les messages de la file d’attente, et pas seulement ceux qui sont disponibles pour une acquisition immédiate avec `Receive()` ou la boucle `OnMessage()`. La propriété `State` de chaque message vous indique si le message est actif (prêt à être reçu), [différé](message-deferral.md) ou [planifié](message-sequencing.md). L’opération Peek sur un abonnement renvoie tous les messages, à l’exception des messages planifiés dans le journal des messages d’abonnement. 

Les messages utilisés et arrivés à expiration sont nettoyés par une exécution « nettoyage de la mémoire » asynchrone. Cette étape ne se produit pas nécessairement immédiatement après l’expiration des messages. C’est la raison pour laquelle `Peek` peut renvoyer des messages qui ont déjà expiré. Ces messages seront supprimés ou mis en file d’attente de lettres mortes la prochaine fois qu’une opération de réception sera appelée sur la file d’attente ou l’abonnement. Gardez ce comportement à l’esprit lorsque vous tentez de récupérer des messages différés de la file d’attente. Un message arrivé à expiration n’est plus éligible pour la récupération normale par d’autres moyens, même quand il est retourné par Peek. Le renvoi de ces messages est délibéré, car Peek est un outil de diagnostic reflétant l’état actuel du journal.

Peek renvoie également les messages qui ont été verrouillés et qui sont actuellement traités par d’autres destinataires. Toutefois, étant donné que Peek renvoie un instantané déconnecté, l’état de verrou d’un message ne peut pas être observé sur des messages aperçus. Les propriétés [LockedUntilUtc](/dotnet/api/microsoft.azure.servicebus.message.systempropertiescollection.lockeduntilutc) et [LockToken](/dotnet/api/microsoft.azure.servicebus.message.systempropertiescollection.locktoken#Microsoft_Azure_ServiceBus_Message_SystemPropertiesCollection_LockToken) lèvent une [InvalidOperationException](/dotnet/api/system.invalidoperationexception) lorsque l’application tente de les lire.

## <a name="peek-apis"></a>API de l’outil Peek

Les méthodes [Peek/PeekAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.peekasync#Microsoft_Azure_ServiceBus_Core_MessageReceiver_PeekAsync) et [PeekBatch/PeekBatchAsync](/dotnet/api/microsoft.servicebus.messaging.queueclient.peekbatchasync#Microsoft_ServiceBus_Messaging_QueueClient_PeekBatchAsync_System_Int64_System_Int32_) sont fournies dans les bibliothèques de client .NET et Java, ainsi que sur les objets récepteurs : **MessageReceiver**, **MessageSession**. Peek fonctionne sur les files d’attente, les abonnements et leurs files d’attente de lettres mortes respectives.

Quand elle est appelée à plusieurs reprises, l’opération **Peek** énumère tous les messages du journal de la file d’attente ou de l’abonnement, du plus petit numéro de séquence disponible au plus grand. C’est l’ordre dans lequel les messages ont été empilés, et non l’ordre dans lequel ils peuvent être récupérés au final.

[PeekBatch](/dotnet/api/microsoft.servicebus.messaging.queueclient.peekbatch#Microsoft_ServiceBus_Messaging_QueueClient_PeekBatch_System_Int32_) récupère plusieurs messages et les retourne sous la forme d’une énumération. Si aucun message n’est disponible, l’objet d’énumération est vide (non Null).

Vous pouvez également utiliser une surcharge de la méthode en définissant un [SequenceNumber](/dotnet/api/microsoft.azure.servicebus.message.systempropertiescollection.sequencenumber#Microsoft_Azure_ServiceBus_Message_SystemPropertiesCollection_SequenceNumber) de démarrage, puis appeler la surcharge de la méthode sans paramètre pour continuer l’énumération. **PeekBatch** fonctionne de façon équivalente, mais récupère tout un ensemble de messages à la fois.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la messagerie Service Bus, consultez les articles suivants :

* [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md)
* [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Utilisation des rubriques et abonnements Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)
