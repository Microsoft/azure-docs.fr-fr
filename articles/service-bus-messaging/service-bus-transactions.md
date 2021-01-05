---
title: Vue d'ensemble du traitement des transactions dans Azure Service Bus
description: Cet article offre une vue d'ensemble du traitement transactionnel et de la fonctionnalité « Envoyer par » dans Azure Service Bus.
ms.topic: article
ms.date: 10/28/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 9162b8578fe4f48cc3740b38d9d84ffaa2f260de
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023599"
---
# <a name="overview-of-service-bus-transaction-processing"></a>Vue d’ensemble du traitement des transactions Service Bus

Cet article décrit les fonctionnalités de transaction de Microsoft Azure Service Bus. Le propos est principalement illustré avec l’exemple [AMPQ Transactions with Service Bus](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/TransactionsAndSendVia/TransactionsAndSendVia/AMQPTransactionsSendVia) (Transactions AMPQ avec Service Bus). Cet article se limite à une vue d’ensemble du traitement des transactions et de la fonctionnalité de *transfert* de Service Bus, tandis que l’exemple des transactions atomiques est de portée plus étendue et plus complexe.

## <a name="transactions-in-service-bus"></a>Transactions dans Service Bus

Une *transaction* regroupe plusieurs opérations dans une *étendue d’exécution*. Par nature, ce type de transaction doit garantir que toutes les opérations appartenant à un groupe d’opérations donné réussissent ou échouent ensemble. À cet égard, les transactions agissent comme une unité, ce qui est souvent désigné par le terme *atomicité*.

Service Bus est un courtier de messages transactionnel. Il garantit l’intégrité transactionnelle de toutes les opérations internes par rapport à ses banques de messages. Tous les transferts de messages dans Service Bus, notamment le déplacement des messages vers une [file d’attente de lettres mortes](service-bus-dead-letter-queues.md) ou le [transfert automatique](service-bus-auto-forwarding.md) de messages entre des entités, sont transactionnels. Par conséquent, si Service Bus accepte un message, il a déjà été stocké et étiqueté avec un numéro de séquence. Dès lors, tous les transferts de messages dans Service Bus sont des opérations coordonnées entre les entités et n’entraînent jamais de perte (réussite de la source et échec de la cible) ou de duplication (échec de la source et réussite de la cible) du message.

Service Bus prend en charge les opérations de regroupement par rapport à une entité de messagerie unique (file d’attente, rubrique, abonnement) dans l’étendue d’une transaction. Par exemple, vous pouvez envoyer plusieurs messages à une file d’attente à partir d’une étendue de transaction, et les messages seront validés dans le journal de la file d’attente uniquement lorsque la transaction se terminera correctement.

## <a name="operations-within-a-transaction-scope"></a>Opérations au sein d’une étendue de transaction

Les opérations qui peuvent être effectuées dans une étendue de transaction sont les suivantes :

* **[QueueClient](/dotnet/api/microsoft.azure.servicebus.queueclient), [MessageSender](/dotnet/api/microsoft.azure.servicebus.core.messagesender), [TopicClient](/dotnet/api/microsoft.azure.servicebus.topicclient)**  : `Send`, `SendAsync`, `SendBatch`, `SendBatchAsync`
* **[BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage)** : `Complete`, `CompleteAsync`, `Abandon`, `AbandonAsync`, `Deadletter`, `DeadletterAsync`, `Defer`, `DeferAsync`, `RenewLock`, `RenewLockAsync` 

Les opérations de réception ne sont pas incluses, car on suppose que l’application acquiert d’abord des messages à l’aide du mode [ReceiveMode.PeekLock](/dotnet/api/microsoft.azure.servicebus.receivemode) dans une boucle de réception ou avec un rappel [OnMessage](/dotnet/api/microsoft.servicebus.messaging.queueclient.onmessage) avant d’ouvrir une étendue de transaction pour le traitement du message.

Le traitement du message (exécution, abandon, lettre morte, report) se produit dans l’étendue du résultat global de la transaction et selon ce résultat.

## <a name="transfers-and-send-via"></a>Transferts

Pour permettre la remise transactionnelle des données d’une file d’attente ou d’une rubrique à un processeur, puis à une autre file d’attente ou rubrique, Service Bus prend en charge les *transferts*. Dans une opération de transfert, un expéditeur envoie d’abord un message à une *file d’attente ou à une rubrique de transfert*. Celle-ci déplace immédiatement le message vers la file d’attente ou rubrique de destination prévue suivant une implémentation de transfert fiable (sur laquelle repose également la fonctionnalité de transfert automatique). Le message n’est jamais validé dans le journal de la file d’attente ou de la rubrique de transfert de façon à devenir accessible aux consommateurs de la file d’attente ou rubrique de transfert.

La puissance de cette fonctionnalité transactionnelle se révèle lorsque la file d’attente ou la rubrique de transfert est elle-même la source des messages d’entrée de l’expéditeur. En d’autres termes, Service Bus peut transférer le message à la file d’attente ou à la rubrique de destination « via » la file d’attente ou rubrique de transfert, tout en effectuant une opération d’exécution complète (ou différée ou de lettres mortes) sur le message d’entrée, en une seule opération atomique. 

### <a name="see-it-in-code"></a>Apparence dans le code

Pour configurer ces transferts, vous devez créer un expéditeur de message qui cible la file d’attente de destination via la file d’attente de transfert. Il vous faut également un destinataire qui extrait les messages de cette même file d’attente. Par exemple :

```csharp
var connection = new ServiceBusConnection(connectionString);

var sender = new MessageSender(connection, QueueName);
var receiver = new MessageReceiver(connection, QueueName);
```

Une simple transaction utilise alors ces éléments, comme dans l’exemple suivant. Pour vous référer à l’exemple complet, veuillez consulter le [code source sur GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/TransactionsAndSendVia/TransactionsAndSendVia/AMQPTransactionsSendVia) :

```csharp
var receivedMessage = await receiver.ReceiveAsync();

using (var ts = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
{
    try
    {
        // do some processing
        if (receivedMessage != null)
            await receiver.CompleteAsync(receivedMessage.SystemProperties.LockToken);

        var myMsgBody = new MyMessage
        {
            Name = "Some name",
            Address = "Some street address",
            ZipCode = "Some zip code"
        };

        // send message
        var message = myMsgBody.AsMessage();
        await sender.SendAsync(message).ConfigureAwait(false);
        Console.WriteLine("Message has been sent");

        // complete the transaction
        ts.Complete();
    }
    catch (Exception ex)
    {
        // This rolls back send and complete in case an exception happens
        ts.Dispose();
        Console.WriteLine(ex.ToString());
    }
}
```

## <a name="timeout"></a>Délai d'expiration
Une transaction expire au bout de 2 minutes. Le minuteur de transaction démarre lorsque la première opération de la transaction démarre. 

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les files d’attente de lettres mortes Service Bus, consultez les articles suivants :

* [Utilisation des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Chaînage des entités Service Bus avec transfert automatique](service-bus-auto-forwarding.md)
* [Exemple de transfert automatique](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/AutoForward)
* Exemple [Atomic Transactions with Service Bus](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/AtomicTransactions) (Transactions atomiques avec Service Bus)
* [Comparaison des files d’attente Azure et Service Bus](service-bus-azure-and-service-bus-queues-compared-contrasted.md)


