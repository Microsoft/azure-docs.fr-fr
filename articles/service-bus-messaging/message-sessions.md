---
title: Sessions de messagerie Azure Service Bus | Microsoft Docs
description: Cet article explique comment utiliser des sessions pour permettre un traitement conjoint et ordonné de séquences illimitées de messages associés.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 05efc550e119186a2925c13d3fcfed11bec17251
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86511294"
---
# <a name="message-sessions"></a>Sessions de message
Les sessions Microsoft Azure Service Bus permettent un traitement conjoint et chronologique de séquences illimitées de messages associés. Vous pouvez utiliser des sessions dans des modèles **premier entré, premier sorti (FIFO**) et **requête-réponse**. Cet article explique comment utiliser des sessions pour implémenter ces modèles lors de l’utilisation de Service Bus. 

> [!NOTE]
> Le niveau de base de Service Bus ne prend pas en charge les sessions. Les niveaux standard et premium prennent en charge les sessions. Pour connaître les différences entre ces niveaux, voir [Tarification de Service Bus](https://azure.microsoft.com/pricing/details/service-bus/).

## <a name="first-in-first-out-fifo-pattern"></a>Modèle premier entré, premier sorti (FIFO)
Pour garantir l’application de la méthode FIFO dans Service Bus, utilisez des sessions. Service Bus n’est pas normatif concernant la nature de la relation entre les messages, et ne définit aucun modèle spécifique pour déterminer le début et la fin d’une séquence de messages.

Lors de l’envoi de messages dans une rubrique ou dans une file d’attente, tout expéditeur peut créer une session en définissant la propriété [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid#Microsoft_Azure_ServiceBus_Message_SessionId) sur un identificateur défini par l’application unique pour la session. Au niveau du protocole AMQP 1.0, cette valeur est mappée sur la propriété *group-id*.

Dans les files d’attente ou abonnements prenant en compte la session, les sessions entrent en action quand au moins un message présente la valeur [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid#Microsoft_Azure_ServiceBus_Message_SessionId) de la session. Une fois qu’une session existe, aucune heure ou API ne définissent son expiration ou sa disparition. Il est théoriquement possible de recevoir un message pour une session le jour même, puis le message suivant un an après, et si la valeur **SessionId** correspond, Service Bus considère alors qu’il s’agit de la même session.

Toutefois, en général, une application détermine précisément le début et la fin d’un ensemble de messages associés. Service Bus ne définit pas de règles spécifiques.

Un exemple de délimitation d’une séquence dans le cadre du transfert d’un fichier consiste à définir la propriété **Label** du premier message sur **start**, celle des messages intermédiaires sur **content**, et celle du dernier message sur **end**. La position relative des messages de contenu peut être calculée comme étant égale à la différence entre la valeur *SequenceNumber* du message actuel et la valeur *SequenceNumber* du message présentant la valeur **start**.

La fonctionnalité de session dans Service Bus autorise une opération de réception spécifique, sous la forme [MessageSession](/dotnet/api/microsoft.servicebus.messaging.messagesession) dans les API C# et Java. Vous activez cette fonctionnalité en définissant la propriété [requiresSession](/azure/templates/microsoft.servicebus/namespaces/queues#property-values) sur la file d’attente ou l’abonnement par le biais d’Azure Resource Manager, ou en définissant l’indicateur dans le portail. Cette opération est obligatoire avant de tenter d’utiliser les opérations API associées.

Dans le portail, définissez l’indicateur avec la case à cocher suivante :

![Capture d’écran de la boîte de dialogue Créer une file d’attente avec l’option Activer les sessions sélectionnée et mise en évidence en rouge.][2]

> [!NOTE]
> Lorsque les sessions sont activées sur une file d’attente ou un abonnement, les applications client ***ne peuvent plus*** envoyer/recevoir des messages standard. Tous les messages doivent être envoyés dans le cadre d’une session (en définissant l’ID de session) et reçus en recevant la session.

Les API relatives aux sessions existent sur les clients de file d’attente et d’abonnement. Il existe un modèle impératif qui contrôle le moment de la réception des sessions et des messages, ainsi qu’un modèle basé sur un gestionnaire, similaire à *OnMessage*, qui simplifie la gestion de la boucle de réception.

### <a name="session-features"></a>Fonctionnalités de session

Les sessions assurent un démultiplexage simultané de flux de messages entrelacés tout en préservant et en garantissant la remise chronologique des messages.

![Diagramme montrant comment la fonctionnalité Sessions préserve la livraison chronologique des messages.][1]

Un destinataire [MessageSession](/dotnet/api/microsoft.servicebus.messaging.messagesession) est créé par le client qui accepte une session. Le client appelle [QueueClient.AcceptMessageSession](/dotnet/api/microsoft.servicebus.messaging.queueclient.acceptmessagesession#Microsoft_ServiceBus_Messaging_QueueClient_AcceptMessageSession) ou [QueueClient.AcceptMessageSessionAsync](/dotnet/api/microsoft.servicebus.messaging.queueclient.acceptmessagesessionasync#Microsoft_ServiceBus_Messaging_QueueClient_AcceptMessageSessionAsync) en C#. Dans le modèle de rappel réactif, il inscrit un gestionnaire de sessions.

Lorsque l’objet [MessageSession](/dotnet/api/microsoft.servicebus.messaging.messagesession) est accepté et détenu par un client, celui-ci maintient un verrouillage exclusif sur tous les messages présentant la valeur [SessionId](/dotnet/api/microsoft.servicebus.messaging.messagesession.sessionid#Microsoft_ServiceBus_Messaging_MessageSession_SessionId) de cette session dans la file d’attente ou dans l’abonnement, ainsi que sur tous les messages présentant cette valeur **SessionId** qui continuent d’arriver pendant toute la durée de la session.

Ce verrouillage est désactivé lors de l’appel des méthodes **Close** ou **CloseAsync**, ou lorsque le verrouillage arrive à expiration dans les cas où l’application ne parvient pas à effectuer l’opération de fermeture. Le verrouillage de session doit être traité de la même façon qu’un verrouillage exclusif sur un fichier, ce qui signifie que l’application doit fermer la session dès qu’elle n’en a plus besoin ou qu’elle n’attend plus aucun message.

Lorsque plusieurs destinataires simultanés extraient des données de la file d’attente, les messages appartenant à une session spécifique sont envoyés au destinataire concerné qui détient actuellement le verrouillage pour cette session. Grâce à cette opération, un flux de messages entrelacé dans une file d’attente ou dans un abonnement est correctement démultiplexé vers différents destinataires qui peuvent également résider sur des machines clientes distinctes, étant donné que la gestion des verrouillages se produit côté service, au sein de Service Bus.

L’illustration précédente montre trois récepteurs de session simultanée. Une Session avec `SessionId` = 4 ne contenant aucun client actif, propriétaire, aucun message n’est remis à partir de cette session. Une session se comporte à maints égards comme une sous-file d’attente.

Le verrouillage de session détenu par le destinataire de session constitue une protection pour les verrouillages de message utilisés par le mode de règlement *peek-lock*. Un seul destinataire peut avoir un verrou sur une session. Un destinataire peut avoir de nombreux messages en vol, mais les messages seront reçus dans l’ordre. L’abandon d’un message entraîne un nouveau traitement de ce message lors de l’opération de réception suivante.

### <a name="message-session-state"></a>État d’une session de messagerie

Lorsque les workflows sont traités dans des systèmes de cloud à grande échelle et à haute disponibilité, le gestionnaire de workflows associé à une session spécifique doit pouvoir redevenir opérationnel en cas de défaillance inattendue, et peut reprendre un travail incomplet sur un processus ou une machine différents de que ceux sur lesquels le travail a commencé.

La fonction d’état de session active une annotation définie par l’application d’une session de messagerie dans le répartiteur, de sorte que l’état de traitement enregistré concernant cette session devient immédiatement disponible lorsque la session est acquise par un nouveau processeur.

Du point de vue de Service Bus, l’état d’une session de messagerie est un objet binaire opaque qui peut contenir une taille de données équivalant à un message, autrement dit 256 Ko pour Service Bus Standard et 1 Mo dans le cas de Service Bus Premium. L’état de traitement relatif à une session peut être stocké dans l’état de session, ou l’état de session peut pointer vers un emplacement de stockage ou un enregistrement de base de données contenant cette information.

Les API de gestion de l’état de session, [SetState](/dotnet/api/microsoft.servicebus.messaging.messagesession.setstate#Microsoft_ServiceBus_Messaging_MessageSession_SetState_System_IO_Stream_) et [GetState](/dotnet/api/microsoft.servicebus.messaging.messagesession.getstate#Microsoft_ServiceBus_Messaging_MessageSession_GetState), sont disponibles dans l’objet [MessageSession](/dotnet/api/microsoft.servicebus.messaging.messagesession) des API C# et Java. Une session pour laquelle aucun état de session n’a été précédemment défini renvoie une référence **null** pour **GetState**. L’annulation de l’état de session précédemment défini s’effectue à l’aide de [SetState(null)](/dotnet/api/microsoft.servicebus.messaging.messagesession.setstate#Microsoft_ServiceBus_Messaging_MessageSession_SetState_System_IO_Stream_).

L’état de session persiste tant qu’il n’est pas effacé (retour de la valeur **Null**), même si tous les messages dans une session donnée sont consommés.

Toutes les sessions existantes dans une file d’attente ou un abonnement peuvent être énumérées à l’aide de la méthode **SessionBrowser** de l’API Java et au moyen de [GetMessageSessions](/dotnet/api/microsoft.servicebus.messaging.queueclient.getmessagesessions#Microsoft_ServiceBus_Messaging_QueueClient_GetMessageSessions) sur [QueueClient](/dotnet/api/microsoft.servicebus.messaging.queueclient) et [SubscriptionClient](/dotnet/api/microsoft.servicebus.messaging.subscriptionclient) dans le client .NET Framework.

L’état de session stocké dans une file d’attente ou dans un abonnement est pris en compte dans le quota de stockage de cette entité. Lorsque l’application a terminé avec une session, il est donc recommandé de faire en sorte que l’application supprime l’état de session conservé afin d’éviter un coût de gestion externe.

### <a name="impact-of-delivery-count"></a>Impact du nombre de livraisons

La définition du nombre de livraisons par message dans le contexte de sessions diffère légèrement de la définition en l’absence de sessions. Voici une table résumant le moment où le nombre de livraisons est incrémenté.

| Scénario | Le nombre de livraisons du message est-il incrémenté |
|----------|---------------------------------------------|
| La session est acceptée, mais le verrouillage de session expire (en raison du délai d’expiration) | Oui |
| La session est acceptée, les messages de la session ne sont pas terminés (même s’ils sont verrouillés) et la session est fermée | Non |
| La session est acceptée, les messages sont terminés, puis la session est explicitement fermée | S.O. (il s’agit du flux standard. Ici, les messages sont supprimés de la session) |

## <a name="request-response-pattern"></a>Modèle requête-réponse
Le [modèle requête-réponse](https://www.enterpriseintegrationpatterns.com/patterns/messaging/RequestReply.html) est un modèle d’intégration bien établi qui permet à l’application de l’expéditeur d’envoyer une demande et fournit au destinataire un moyen de renvoyer correctement une réponse à l’application de l’expéditeur. Ce modèle a généralement besoin d’une file d’attente ou d’une rubrique de courte durée de vie à laquelle l’application puisse envoyer des réponses. Dans ce scénario, les sessions fournissent une solution alternative simple avec une sémantique comparable. 

Plusieurs applications peuvent envoyer leurs requêtes à une seule file d’attente de requêtes, avec un paramètre d’en-tête spécifique défini pour identifier de manière unique l’application de l’expéditeur. L’application du destinataire peut traiter les requêtes arrivant dans la file d’attente et envoyer des réponses sur une file d’attente activée par la session, en définissant l’ID de session sur l’identificateur unique que l’expéditeur a envoyé sur le message de requête. L’application qui a envoyé la requête peut alors recevoir des messages sur l’ID de session spécifique et traiter correctement les réponses.

> [!NOTE]
> L’application qui envoie les requêtes initiales doit connaître l’ID de session et utiliser `SessionClient.AcceptMessageSession(SessionID)` pour verrouiller la session sur laquelle elle attend la réponse. Il est judicieux d’utiliser un GUID qui identifie de façon unique l’instance de l’application en tant qu’ID de session. Il ne doit y avoir aucun gestionnaire de session ou `AcceptMessageSession(timeout)` sur la file d’attente pour s’assurer que les réponses sont disponibles pour verrouillage et traitement par des destinataires spécifiques.

## <a name="next-steps"></a>Étapes suivantes

- Consultez les [exemples Microsoft.Azure.ServiceBus](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/Sessions) ou les [exemples Microsoft.ServiceBus.Messaging](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/Sessions) pour obtenir un exemple d’utilisation du client .NET Framework pour traiter les messages prenant en charge la session. 

Pour plus d’informations sur la messagerie Service Bus, consultez les articles suivants :

* [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md)
* [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Utilisation des rubriques et abonnements Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)

[1]: ./media/message-sessions/sessions.png
[2]: ./media/message-sessions/queue-sessions.png
