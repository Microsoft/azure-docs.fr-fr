---
title: Azure Service Bus - Expiration des messages
description: Cet article traite de l'expiration et de la durée de vie des messages Azure Service Bus. Passé ce délai, le message n’est plus remis.
ms.topic: article
ms.date: 09/29/2020
ms.openlocfilehash: 47f8bdb4440adfeb5197f90cdad5358a442ce6a7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91569924"
---
# <a name="message-expiration-time-to-live"></a>Expiration des messages (durée de vie)

La charge utile dans un message, ou une commande ou demande transmise par un message à un destinataire, est presque toujours soumise à une forme de délai d’expiration au niveau de l’application. Passé ce délai, le contenu n’est plus remis, ou l’opération demandée n’est plus exécutée.

Dans les environnements de développement et de test où les files d’attente et les rubriques sont souvent utilisées dans le contexte d’exécutions partielles d’applications ou de parties d’application, il est également préférable que les messages de test abandonnés soient automatiquement nettoyés de la mémoire pour permettre à la prochaine série de tests de démarrer le nettoyage.

Le délai d’expiration de chaque message peut être contrôlé à l’aide de la propriété système [TimeToLive](/dotnet/api/microsoft.azure.servicebus.message.timetolive#Microsoft_Azure_ServiceBus_Message_TimeToLive), qui définit une durée relative. Ce délai devient un instant absolu quand le message est mis en file d’attente dans l’entité. La propriété [ExpiresAtUtc](/dotnet/api/microsoft.azure.servicebus.message.expiresatutc) prend alors la valeur [(**EnqueuedTimeUtc**](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.enqueuedtimeutc#Microsoft_ServiceBus_Messaging_BrokeredMessage_EnqueuedTimeUtc) + [**TimeToLive**)](/dotnet/api/microsoft.azure.servicebus.message.timetolive#Microsoft_Azure_ServiceBus_Message_TimeToLive). Le paramètre time-to-live (TTL) sur un message réparti n'est pas appliqué lorsqu’aucun client n’écoute activement.

Quand l’instant **ExpiresAtUtc** est passé, les messages ne sont plus éligibles pour la récupération. Le délai d’expiration ne s’applique pas aux messages qui sont actuellement verrouillés pour la remise, lesquels continuent d’être traités normalement. Si le verrou expire ou si le message est abandonné, le délai d’expiration prend effet immédiatement.

Lorsque le message est verrouillé, l’application peut être en possession d’un message qui a expiré. L’application peut alors poursuivre le traitement ou choisir d’abandonner le message. C’est au responsable de l’implémentation d’en décider.

## <a name="entity-level-expiration"></a>Expiration au niveau de l’entité

Tous les messages envoyés dans une file d’attente ou une rubrique sont soumis à un délai d’expiration par défaut qui est défini au niveau de l’entité avec la propriété [defaultMessageTimeToLive](/azure/templates/microsoft.servicebus/namespaces/queues). Ce délai peut également être défini dans le portail au moment de la création et modifié plus tard. Le délai d’expiration par défaut est utilisé pour tous les messages envoyés à l’entité où la propriété [TimeToLive](/dotnet/api/microsoft.azure.servicebus.message.timetolive#Microsoft_Azure_ServiceBus_Message_TimeToLive) n’est pas définie explicitement. Le délai d’expiration par défaut sert également de plafond pour la valeur **TimeToLive**. Les messages qui ont un délai d’expiration **TimeToLive** plus long que le délai par défaut sont ajustés en mode silencieux à la valeur **defaultMessageTimeToLive** avant d’être mis en file attente.

> [!NOTE]
> La valeur par défaut [TimeToLive](/dotnet/api/microsoft.azure.servicebus.message.timetolive#Microsoft_Azure_ServiceBus_Message_TimeToLive) d’un message réparti est [TimeSpan.Max](/dotnet/api/system.timespan.maxvalue), sauf indication contraire.
>
> Pour les entités de messagerie (files d’attente et rubriques), le délai d’expiration par défaut est également [TimeSpan.Max](/dotnet/api/system.timespan.maxvalue) pour les niveaux standard et Premium de Service Bus. Pour le niveau **de base**, le délai d’expiration par défaut est de **14 jours** (qui est également le délai d’expiration maximal).

Vous pouvez éventuellement choisir de déplacer les messages expirés vers une [file d’attente de lettres mortes](service-bus-dead-letter-queues.md) en définissant la propriété [EnableDeadLetteringOnMessageExpiration](/dotnet/api/microsoft.servicebus.messaging.queuedescription.enabledeadletteringonmessageexpiration#Microsoft_ServiceBus_Messaging_QueueDescription_EnableDeadLetteringOnMessageExpiration) ou en cochant la case correspondante dans le portail. Si l’option reste désélectionnée, les messages expirés sont supprimés. La distinction entre les messages expirés déplacés vers la file d’attente de lettres mortes et les autres messages de lettres mortes est possible grâce à la valeur de la propriété [DeadletterReason](service-bus-dead-letter-queues.md#moving-messages-to-the-dlq) que le répartiteur stocke dans la section des propriétés utilisateur (sa valeur est [TTLExpiredException](service-bus-dead-letter-queues.md#moving-messages-to-the-dlq) dans le cas présent).

Dans le cas cité plus haut où le message n’est pas soumis au délai d’expiration du fait qu’il est verrouillé, et si l’indicateur est défini sur l’entité, le message est déplacé dans la file d’attente de lettres mortes aussitôt après l’abandon ou l’expiration du verrou. Toutefois, le message n’est pas déplacé s’il s’agit d’un message réglé, ce qui suppose que l’application a pu le traiter, en dépit du délai d’expiration nominal.

L’utilisation combinée de la propriété [TimeToLive](/dotnet/api/microsoft.azure.servicebus.message.timetolive#Microsoft_Azure_ServiceBus_Message_TimeToLive) et du déplacement automatique (et transactionnel) vers la file d’attente de lettres mortes à l’expiration est une solution efficace pour garantir qu’un travail donné à un gestionnaire ou un groupe de gestionnaires avec un délai d’expiration sera bien récupéré pour être traité avant la fin du délai d’expiration.

Prenons l’exemple d’un site web qui doit exécuter des travaux de manière fiable sur un serveur principal avec des contraintes de mise à l’échelle, et qui connaît parfois des pics de trafic ou doit être isolé pendant les périodes de disponibilité de ce serveur principal. En situation normale, le gestionnaire côté serveur envoie (push) les données transmises par l’utilisateur dans une file d’attente et reçoit par la suite une réponse qui confirme le traitement réussi de la transaction dans une file d’attente de réponse. En présence d’un pic de trafic, si le gestionnaire du serveur principal ne peut pas traiter ses éléments de backlog dans le temps imparti, les travaux expirés sont retournés dans la file d’attente de lettres mortes. L’utilisateur interactif peut être averti que l’opération demandée va nécessiter un peu plus de temps que normalement, et la demande peut ensuite être placée dans une autre file d’attente pour un chemin d’accès de traitement où le résultat du traitement final est envoyé à l’utilisateur par e-mail. 


## <a name="temporary-entities"></a>Entités temporaires

Les files d’attente, rubriques et abonnements Service Bus peuvent être créés en tant qu’entités temporaires, qui sont automatiquement supprimées quand elles n’ont pas été utilisées pendant une période spécifiée.
 
Le nettoyage automatique est utile dans les scénarios de développement et de test où les entités sont créées de façon dynamique et ne sont pas nettoyées après leur utilisation, en raison d’une interruption de la série de tests ou d’un débogage. Il est également utile quand une application crée des entités dynamiques, telles qu’une file d’attente de réponse, pour recevoir des réponses dans un processus de serveur web ou dans un autre objet relativement éphémère. En effet, dans ce cas, il est difficile de nettoyer correctement ces entités quand l’instance de l’objet disparaît.

La fonctionnalité est activée avec la propriété [autoDeleteOnIdle](/azure/templates/microsoft.servicebus/namespaces/queues). La propriété est définie sur la durée pendant laquelle une entité peut rester inactive (non utilisée) avant d’être automatiquement supprimée. La valeur minimale pour cette propriété est 5.
 
La propriété **autoDeleteOnIdle** doit être définie par le biais d’une opération Azure Resource Manager ou des API [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) du client .NET Framework. Vous ne pouvez pas la définir dans le portail.

## <a name="idleness"></a>Inactivité

Voici ce qui considéré comme inactivité d’entités (files d’attente, rubriques et abonnements) :

- Files d’attente
    - Aucun envoi  
    - Aucune réception  
    - Aucune mise à jour de la file d’attente  
    - Aucun message planifié  
    - Aucun parcours/affichage d’aperçu 
- Rubriques  
    - Aucun envoi  
    - Aucune mise à jour de la rubrique  
    - Aucun message planifié 
- Abonnements
    - Aucune réception  
    - Aucune mise à jour de l’abonnement  
    - Aucune nouvelle règle ajoutée à l’abonnement  
    - Aucun parcours/affichage d’aperçu  
 


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la messagerie Service Bus, consultez les articles suivants :

* [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md)
* [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Utilisation des rubriques et abonnements Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)