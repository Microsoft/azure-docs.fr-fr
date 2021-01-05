---
title: Azure Service Bus - Report des messages
description: Cet article explique comment reporter la remise des messages Azure Service Bus. Le message reste dans la file d’attente ou l’abonnement, mais il est mis de côté.
ms.topic: article
ms.date: 06/23/2020
ms.custom: fasttrack-edit
ms.openlocfilehash: 7c9ec55de24c97df3530d80deef55ed87be84077
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86511277"
---
# <a name="message-deferral"></a>Report de message

Lorsqu’un client de file d’attente ou d’abonnement reçoit un message qu’il souhaite traiter, mais pour lequel le traitement n’est pas possible actuellement en raison des circonstances particulières à l’intérieur de l’application, il a la possibilité de « reporter » la récupération du message. Le message reste dans la file d’attente ou l’abonnement, mais il est mis de côté.

Le report est une fonctionnalité créée spécifiquement pour des scénarios de traitement de flux de travail. Pour les infrastructures de flux de travail, certaines opérations doivent être traitées dans un ordre particulier et doivent différer le traitement de messages spécifiques jusqu’à ce que le travail préalable prescrit basé sur les autres messages soit terminé.

Un exemple simple illustrant cela est une séquence de traitement d’une commande dans laquelle une notification de paiement provenant d’un fournisseur de paiement externe s’affiche dans un système avant que le bon de commande associé ait été transmis du magasin au système de traitement. Dans ce cas, le système de traitement peut reporter le traitement de la notification de paiement jusqu’à ce qu’une commande y soit associée. Dans les scénarios de rendez-vous, dans lesquels des messages provenant de différentes sources font avancer un flux de travail, l’ordre d’exécution en temps réel peut effectivement être correct, mais les messages reflétant les résultats peuvent arriver dans le désordre.

Enfin, le report permet de faciliter la réorganisation des messages, de leur ordre d’arrivée à l’ordre dans lequel ils peuvent être traités, tout en laissant en toute sécurité ces messages (dont le traitement doit être différé) dans le magasin de messages.

> [!NOTE]
> Les messages différés ne sont pas automatiquement déplacés vers la file d'attente de lettres mortes [après leur expiration](./service-bus-dead-letter-queues.md#exceeding-timetolive). Ce comportement est lié à la conception.

## <a name="message-deferral-apis"></a>API de report de message

L’API est [BrokeredMessage.Defer](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.defer?view=azureservicebus-4.1.1#Microsoft_ServiceBus_Messaging_BrokeredMessage_Defer) ou [BrokeredMessage.DeferAsync](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.deferasync?view=azureservicebus-4.1.1#Microsoft_ServiceBus_Messaging_BrokeredMessage_DeferAsync) dans le client .NET Framework, [MessageReceiver.DeferAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.deferasync) dans le client .NET Standard et [IMessageReceiver.defer](/java/api/com.microsoft.azure.servicebus.imessagereceiver.defer?view=azure-java-stable) ou [IMessageReceiver.deferAsync](/java/api/com.microsoft.azure.servicebus.imessagereceiver.deferasync?view=azure-java-stable) dans le client Java. 

Les messages reportés restent dans la file d’attente principale avec tous les autres messages actifs (contrairement aux messages de type lettres mortes qui se trouvent dans une sous-file d’attente), mais ils ne peuvent plus être reçus via les fonctions Receive/ReceiveAsync standard. Les messages reportés peuvent être détectés via la [navigation dans les messages](message-browsing.md) si une application perd leur trace.

Pour récupérer un message reporté, son propriétaire doit se souvenir du [SequenceNumber](/dotnet/api/microsoft.azure.servicebus.message.systempropertiescollection.sequencenumber#Microsoft_Azure_ServiceBus_Message_SystemPropertiesCollection_SequenceNumber), car ce numéro permet de reporter le message. Les destinataires qui connaissent le numéro de séquence d’un message reporté peuvent, plus tard, recevoir le message de façon explicite avec le `Receive(sequenceNumber)`.

Si un message ne peut pas être traité en raison de l’indisponibilité temporaire d’une ressource nécessaire à la gestion de ce message, mais que le traitement du message ne doit pas être sommairement interrompu, un moyen pour mettre le message de côté pendant quelques minutes consiste à rappeler le **SequenceNumber** dans un [message planifié](message-sequencing.md) à publier après un court instant, puis à récupérer le message reporté lorsque le message planifié arrive. Si un gestionnaire de messages dépend d’une base de données pour toutes les opérations et si la base de données est temporairement indisponible, il ne doit pas utiliser le report, mais plutôt interrompre complètement la réception des messages jusqu’à ce que la base de données soit à nouveau disponible.


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la messagerie Service Bus, consultez les articles suivants :

* [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md)
* [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Utilisation des rubriques et abonnements Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)
