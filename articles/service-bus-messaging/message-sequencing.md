---
title: Séquencement et horodatage des messages Azure Service Bus | Microsoft Docs
description: Cet article explique comment conserver le séquencement et le classement (avec timestamps) des messages Azure Service Bus.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: fdb18802e576ad114fd3f783d5efd7bb826a5f94
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85341177"
---
# <a name="message-sequencing-and-timestamps"></a>Séquencement et horodatage des messages

Le séquencement et l’horodatage sont deux fonctionnalités qui sont toujours activées sur toutes les entités Service Bus et qui sont exposées par le biais des propriétés [SequenceNumber](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.sequencenumber) et [EnqueuedTimeUtc](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.enqueuedtimeutc) des messages reçus ou parcourus.

Dans les scénarios où le respect de l’ordre absolu des messages est important et/ou dans lesquels un consommateur a besoin de suivre les messages avec un identificateur unique fiable, le répartiteur horodate les messages avec des numéros de séquence incrémentiels continus en lien avec la file d’attente ou la rubrique. Pour les entités partitionnées, le numéro de séquence est déterminé par rapport à la partition.

La valeur **SequenceNumber** est un entier 64 bits unique qui est assigné à un message accepté et stocké par le répartiteur. Cette valeur est l’identificateur interne du message. Pour les entités partitionnées, les 16 premiers bits correspondent à l’identificateur de la partition. Les numéros de séquence sont remis à zéro quand la plage de 48/64 bits est épuisée.

Le numéro de séquence peut être considéré comme un identificateur unique fiable, car il est assigné par une autorité centrale et neutre, et pas par les clients. Il représente également l’ordre d’arrivée réel. Il est plus précis qu’un horodatage comme critère de tri, car les horodatages n’offrent pas toujours une résolution suffisamment élevée à des débits de messages extrêmes et peuvent subir des variations temporelles (toutefois minimes) dans les situations où la propriété du répartiteur change entre les nœuds.

L’ordre d’arrivée absolu est important, notamment, dans des scénarios d’entreprise où un nombre limité de produits vendus sont fournis selon le principe du « premier arrivé, premier servi » jusqu’à épuisement des stocks. La vente de tickets de concert en est un bon exemple.

La fonctionnalité d’horodatage agit comme une autorité neutre et fiable qui capture l’heure UTC exacte de l’arrivée d’un message, reflétée dans la propriété **EnqueuedTimeUtc**. Cette valeur est utile dans les scénarios d’entreprise dépendant des délais, par exemple, si un élément de travail a été envoyé à une date donnée avant minuit, mais que le traitement est loin dans le backlog de la liste d’attente.

## <a name="scheduled-messages"></a>Messages planifiés

Vous pouvez envoyer des messages dans une file d’attente ou une rubrique pour les traiter en différé, par exemple, si vous avez besoin de planifier un travail de sorte qu’il soit disponible pour être traité par un système à un moment précis. Cette fonctionnalité permet d’implémenter un planificateur temporel distribué fiable.

Les messages planifiés n’apparaissent réellement dans la file d’attente qu’à l’heure définie de la mise en file d’attente. Avant ce moment, les messages planifiés peuvent être annulés. L’annulation supprime le message.

Vous pouvez planifier un message soit en définissant la propriété [ScheduledEnqueueTimeUtc](/dotnet/api/microsoft.azure.servicebus.message.scheduledenqueuetimeutc) au moment de l’envoi du message via le chemin d’envoi normal, soit explicitement avec l’API [ScheduleMessageAsync](/dotnet/api/microsoft.azure.servicebus.queueclient.schedulemessageasync#Microsoft_Azure_ServiceBus_QueueClient_ScheduleMessageAsync_Microsoft_Azure_ServiceBus_Message_System_DateTimeOffset_). Cette dernière retourne immédiatement la valeur **SequenceNumber** du message planifié, que vous pouvez utiliser ultérieurement pour annuler le message planifié si nécessaire. Les messages planifiés et leurs numéros de séquence peuvent également être découverts avec la fonctionnalité de [parcours des messages](message-browsing.md).

La valeur **SequenceNumber** d’un message planifié est valide seulement si le message est dans cet état. Quand le message passe à l’état actif, il est ajouté à la file d’attente comme s’il venait d’être mis en file d’attente, ce qui implique l’affectation d’une nouvelle valeur **SequenceNumber**.

Étant donné que la fonctionnalité s’applique à des messages individuels et que les messages ne peuvent être mis en file d’attente qu’une seule fois, Service Bus ne prend pas en charge les planifications récurrentes des messages.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la messagerie Service Bus, consultez les articles suivants :

* [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md)
* [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Utilisation des rubriques et abonnements Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)