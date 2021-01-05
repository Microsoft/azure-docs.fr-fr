---
title: Transferts, verrouillages et règlement des messages Azure Service Bus
description: Cet article fournit une vue d’ensemble des opérations de transferts, verrouillages et règlement des messages Azure Service Bus.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: fd71edd12e478bcd5f14815c105c14482cf7e2bd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89020029"
---
# <a name="message-transfers-locks-and-settlement"></a>Transferts, verrouillages et règlement des messages

La fonctionnalité centrale d’un répartiteur de messages tel que Service Bus est d’accepter les messages envoyés dans une file d’attente ou une rubrique et de garantir leur disponibilité jusqu’à leur récupération ultérieure. *Envoi* est le terme couramment utilisé pour désigner le transfert d’un message vers le répartiteur de messages. *Réception* est le terme couramment utilisé pour désigner le transfert d’un message vers un client destinataire.

Quand un client envoie un message, il souhaite généralement savoir si le message a correctement été transféré au répartiteur et accepté par celui-ci ou, au contraire, si une erreur s’est produite pendant l’opération. Cet accusé de réception positif ou négatif permet au client et au répartiteur de connaître de manière fiable l’état du transfert du message. Ce processus est appelé *règlement*.

De la même façon, quand le répartiteur transfère un message à un client, le répartiteur et le client ont besoin de savoir si le message a bien été traité pour pouvoir ensuite être supprimé, ou s’il n’a pas pu être remis ou traité et doit donc être remis une nouvelle fois.

## <a name="settling-send-operations"></a>Règlement des opérations d’envoi

Sur tous les clients d’API Service Bus pris en charge, les opérations d’envoi dans Service Bus sont toujours réglées de façon explicite, c’est-à-dire que l’opération d’API attend de recevoir la confirmation de l’acceptation de Service Bus pour terminer l’opération d’envoi.

Si le message est rejeté par Service Bus, le rejet contient un indicateur et un texte d’erreur, avec un ID de suivi (« tracking-id »). Le rejet inclut également des informations indiquant si une nouvelle tentative de l’opération a des chances de réussir. Sur le client, ces informations sont fournies sous la forme d’une exception, qui est déclenchée à l’attention de l’appelant de l’opération d’envoi. Si le message a été accepté, l’opération se termine en mode silencieux.

Avec le protocole AMQP, qui est le protocole exclusif pour le client .NET Standard et le client Java, mais [qui est une option pour le client .NET Framework](service-bus-amqp-dotnet.md), les transferts et règlements de messages sont traités en pipeline de manière totalement asynchrone. Il est donc recommandé d’utiliser les variantes d’API du modèle de programmation asynchrone.

Cela permet à l’expéditeur de transmettre rapidement plusieurs messages à la suite sur le réseau sans avoir à attendre que chaque message soit accepté, comme cela est le cas avec le protocole SBMP ou HTTP 1.1. Ces opérations d’envoi asynchrones se terminent quand les messages respectifs sont acceptés et stockés sur les entités partitionnées ou en cas de chevauchement de l’opération d’envoi sur différentes entités. Les opérations d’envoi peuvent également se terminer indépendamment de l’ordre d’envoi initial.

La stratégie choisie pour traiter le résultat des opérations d’envoi peut immédiatement et fortement impacter les performances de votre application. Les exemples de cette section sont écrits en C#, mais ils s’appliquent de façon équivalente aux futurs Java.

Si une application générant des rafales de messages, comme illustré ici avec une boucle simple, devait attendre la fin de chaque opération d’envoi avant d’envoyer le message suivant, pour les API synchrones comme pour les API asynchrones, l’envoi de dix messages prendrait dix allers-retours séquentiels complets pour le règlement.

En supposant une distance de latence aller-retour TCP de 70 millisecondes entre un site local et Service Bus, et en donnant seulement un délai de 10 ms à Service Bus pour accepter et stocker chaque message, l’exécution de la boucle suivante prendrait au moins huit secondes, sans prendre en compte le temps de transfert de la charge utile ni les effets d’une surcharge du routage potentielle :

```csharp
for (int i = 0; i < 100; i++)
{
  // creating the message omitted for brevity
  await client.SendAsync(…);
}
```

Si l’application démarre les dix opérations d’envoi asynchrones à la suite et attend que chacune d’elles soit terminée, les allers-retours pour ces dix opérations se chevauchent dans la durée. Les dix messages sont immédiatement transférés l’un après l’autre, potentiellement en partageant les trames TCP. La durée totale du transfert dépend en grande partie du temps réseau nécessaire pour envoyer les messages au répartiteur.

En reprenant les mêmes hypothèses que pour la boucle précédente, la durée d’exécution totale avec chevauchement pour la boucle suivante devrait largement rester sous le seuil d’une seconde :

```csharp
var tasks = new List<Task>();
for (int i = 0; i < 100; i++)
{
  tasks.Add(client.SendAsync(…));
}
await Task.WhenAll(tasks);
```

Il est important de savoir que tous les modèles de programmation asynchrones utilisent une forme de file d’attente de travail masquée et basée sur la mémoire, qui contient les opérations en attente. Quand [SendAsync](/dotnet/api/microsoft.azure.servicebus.queueclient.sendasync#Microsoft_Azure_ServiceBus_QueueClient_SendAsync_Microsoft_Azure_ServiceBus_Message_) (C#) ou **Send** (Java) retourne le résultat, le travail d’envoi est mis dans cette file d’attente de travail, mais le protocole commence son action uniquement quand c’est au tour de ce travail d’être exécuté. Si vous utilisez du code qui envoie (push) souvent des rafales de messages et pour lequel la fiabilité est un critère important, évitez de lancer l’envoi d’un trop grand nombre de messages à la fois, car tous les messages envoyés restent en mémoire jusqu’à ce qu’ils soient réellement mis sur le réseau.

Les sémaphores, comme ceux contenus dans l’extrait de code C# suivant, sont des objets de synchronisation qui permettent d’imposer une limitation au niveau de l’application si nécessaire. Cette utilisation de sémaphore permet l’envoi simultané de dix messages au maximum. L’un des dix verrous de sémaphore disponibles est pris avant l’envoi et est libéré seulement à la fin de l’envoi. Au 11e passage, la boucle attend qu’au moins un des précédents envois soit terminé, puis libère son verrou :

```csharp
var semaphore = new SemaphoreSlim(10);

var tasks = new List<Task>();
for (int i = 0; i < 100; i++)
{
  await semaphore.WaitAsync();

  tasks.Add(client.SendAsync(…).ContinueWith((t)=>semaphore.Release()));
}
await Task.WhenAll(tasks);
```

Les applications ne doivent **jamais** démarrer une opération d’envoi asynchrone en mode « Fire and Forget » sans récupérer le résultat de l’opération. En effet, cela risque d’entraîner le chargement de la file d’attente de travail interne et masquée jusqu’à épuisement de la mémoire et d’empêcher l’application de détecter les erreurs d’envoi :

```csharp
for (int i = 0; i < 100; i++)
{

  client.SendAsync(message); // DON’T DO THIS
}
```

Avec un client AMQP de bas niveau, Service Bus accepte également les transferts « préréglés ». Un transfert préréglé est une opération en mode « Fire and Forget » dont le résultat n’est jamais retourné au client et pour laquelle le message est considéré comme réglé dès l’envoi. Le fait qu’aucun résultat ne soit retourné au client implique aussi qu’il n’y a pas de données actionnables disponibles pour les diagnostics, ce qui signifie que ce mode n’est pas éligible à l’aide par le support Azure.

## <a name="settling-receive-operations"></a>Règlement des opérations de réception

Pour les opérations de réception, les clients d’API Service Bus utilisent deux modes explicites : *Receive-and-Delete* et *Peek-Lock*.

### <a name="receiveanddelete"></a>ReceiveAndDelete

Le mode [Receive-and-Delete](/dotnet/api/microsoft.servicebus.messaging.receivemode) indique au répartiteur qu’il doit considérer tous les messages qu’il envoie au client destinataire comme réglés au moment de l’envoi. Cela signifie qu’un message est considéré comme consommé dès que le répartiteur l’a mis sur le réseau. Si le transfert du message échoue, le message est perdu.

L’avantage de ce mode est que le destinataire n’a aucune autre action à effectuer sur le message et qu’il ne perd pas de temps à attendre le résultat du règlement. Si les données contenues dans les messages ont une faible valeur et/ou restent pertinentes pendant une durée très courte, ce mode est un bon choix.

### <a name="peeklock"></a>PeekLock

Le mode [Peek-Lock](/dotnet/api/microsoft.servicebus.messaging.receivemode) indique au répartiteur que le client destinataire souhaite régler explicitement les messages reçus. Le message est mis à la disposition du destinataire en vue de son traitement, mais il est verrouillé en mode exclusif dans le service pour qu’il ne soit pas visible par d’autres destinataires concurrents. La durée du verrou est initialement définie au niveau de la file d’attente ou de l’abonnement, mais elle peut être étendue par le client propriétaire du verrou à l’aide d’une opération [RenewLock](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.renewlockasync#Microsoft_Azure_ServiceBus_Core_MessageReceiver_RenewLockAsync_System_String_).

Quand un message est verrouillé, les autres clients qui reçoivent des messages de la même file d’attente ou du même abonnement peuvent acquérir des verrous et récupérer les messages disponibles suivants qui ne sont pas verrouillés. Quand le verrou sur un message est explicitement libéré ou qu’il expire, le message peut remonter jusqu’à la première position dans l’ordre de récupération pour être remis une nouvelle fois.

Si le verrou du message est libéré à plusieurs reprises par les destinataires ou s’il expire un nombre de fois supérieur à la valeur définie par [maxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount#Microsoft_ServiceBus_Messaging_QueueDescription_MaxDeliveryCount), le message est automatiquement supprimé de la file d’attente ou de l’abonnement et il est mis dans la file d’attente de lettres mortes associée.

Le client destinataire démarre le règlement d’un message reçu avec un accusé de réception positif lorsqu’il appelle [Complete](/dotnet/api/microsoft.servicebus.messaging.queueclient.complete#Microsoft_ServiceBus_Messaging_QueueClient_Complete_System_Guid_) au niveau de l’API. Cette opération indique au répartiteur que le message a été traité correctement. Le message est alors supprimé de la file d’attente ou de l’abonnement. Le répartiteur envoie une réponse à l’intention de règlement du destinataire, dans laquelle il indique si le règlement peut être effectué.

Quand le client destinataire ne parvient pas à traiter un message, mais souhaite que le message soit remis une nouvelle fois, il peut demander explicitement la libération et le déverrouillage instantanés du message en appelant [Abandon](/dotnet/api/microsoft.servicebus.messaging.queueclient.abandon), ou il peut ne rien faire et laisser le verrou expirer naturellement.

Si un client destinataire échoue à traiter un message et qu’il sait que le résultat sera le même après une deuxième remise du message et une nouvelle tentative de l’opération, il peut choisir de rejeter le message. Cette action entraîne le déplacement du message vers la file d’attente de lettres mortes, avec l’appel de [DeadLetter](/dotnet/api/microsoft.servicebus.messaging.queueclient.deadletter), et permet également de définir une propriété personnalisée contenant un code de raison qui peut être récupéré avec le message à partir de la file d’attente de lettres mortes.

Le report est un cas particulier de règlement et fait l’objet d’un article séparé.

Les opérations **Complete** ou **Deadletter** et les opérations **RenewLock** peuvent échouer en raison de problèmes réseau, si le verrou maintenu a expiré ou si d’autres conditions côté service empêchent le règlement. Dans l’un de ces cas, le service envoie un accusé de réception négatif qui est exposé sous forme d’exception dans les clients d’API. Si la raison est l’interruption d’une connexion réseau, le verrou est supprimé, car Service Bus ne prend pas en charge la récupération des liens AMQP existants sur une autre connexion.

Si une opération **Complete** échoue, ce qui arrive généralement à la fin du traitement d’un message et, dans certains cas, quelques minutes après le début du travail, l’application destinataire peut choisir de garder l’état du travail et d’ignorer le même message qui est remis une deuxième fois, ou de ne pas conserver le résultat du travail et de recommencer l’opération après une nouvelle remise du message.

Le mécanisme standard permettant d’identifier les remises de messages dupliqués consiste à vérifier l’ID du message (message-id). Cet identificateur doit être défini par l’expéditeur à une valeur unique, éventuellement en lien avec un identificateur du processus initial. Un planificateur de travaux préfère généralement définir l’ID du message sur l’identificateur du travail qu’il essaie d’assigner à un processus Worker avec le processus Worker donné. De cette façon, le processus Worker ignore la deuxième occurrence de l’assignation du travail si ce travail est déjà fait.

> [!IMPORTANT]
> Il est important de noter que le verrou acquis par PeekLock sur le message est volatil et peut être perdu dans les conditions suivantes :
>   * Mise à jour du service
>   * Mise à jour du système d’exploitation
>   * Changement des propriétés de l’entité (file d’attente, rubrique, abonnement) pendant le maintien du verrou
>
> Quand le verrou est perdu, Azure Service Bus génère une LockLostException qui est exposée sur le code de l’application cliente. Dans ce cas, la logique de nouvelle tentative par défaut du client doit automatiquement démarrer et recommencer l’opération.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la messagerie Service Bus, consultez les articles suivants :

* [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md)
* [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Utilisation des rubriques et abonnements Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)
