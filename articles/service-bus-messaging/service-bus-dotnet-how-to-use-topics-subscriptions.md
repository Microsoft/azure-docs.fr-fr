---
title: Envoyer des messages à des rubriques Azure Service Bus à l’aide d’azure-messaging-servicebus
description: Ce guide de démarrage rapide vous montre comment envoyer des messages à des rubriques Azure Service Bus à l’aide du package azure-messaging-servicebus.
ms.topic: quickstart
ms.tgt_pltfrm: dotnet
ms.date: 11/13/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 5dfdf846410e9b622f3ef1e9006aa9846de9979c
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498708"
---
# <a name="send-messages-to-an-azure-service-bus-topic-and-receive-messages-from-subscriptions-to-the-topic-net"></a>Envoyer des messages à une rubrique Azure Service Bus et recevoir des messages à partir des abonnements à cette rubrique (.NET)
Ce tutoriel vous montre comment créer une application console .NET Core qui envoie des messages à une rubrique Service Bus et reçoit des messages d’un abonnement à cette rubrique. 

> [!Important]
> Ce guide de démarrage rapide utilise le nouveau package **Azure.Messaging.ServiceBus**. Pour un guide de démarrage rapide qui utilise l’ancien package Microsoft.Azure.ServiceBus, consultez [Envoyer et recevoir des messages à l’aide du package Microsoft.Azure.ServiceBus](service-bus-dotnet-how-to-use-topics-subscriptions-legacy.md).

## <a name="prerequisites"></a>Prérequis

- [Visual Studio 2019](https://www.visualstudio.com/vs)
- Un abonnement Azure. Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Vous pouvez activer les [avantages de votre abonnement Visual Studio ou MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) ou vous inscrire pour créer un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Suivez les étapes dans [Démarrage rapide : Utiliser le portail Azure pour créer une rubrique Service Bus et des abonnements à cette rubrique](service-bus-quickstart-topics-subscriptions-portal.md). Notez la chaîne de connexion, le nom de la rubrique et un nom d’abonnement. Vous n’utiliserez qu’un seul abonnement dans ce guide de démarrage rapide. 

## <a name="send-messages-to-a-topic"></a>Envoi de messages à une rubrique
Dans cette section, vous allez créer une application console .NET Core dans Visual Studio et ajouter du code pour envoyer des messages à la rubrique que vous avez créée. 

### <a name="create-a-console-application"></a>Création d’une application console
Ouvrez Visual Studio et créez un projet **Application de console (.NET Core)** pour C#. 

### <a name="add-the-service-bus-nuget-package"></a>Ajout du package NuGet Service Bus

1. Cliquez avec le bouton droit sur le projet créé et sélectionnez **Gérer les packages NuGet**.
1. Sélectionnez **Parcourir**. Recherchez et sélectionnez **[Azure.Messaging.ServiceBus](https://www.nuget.org/packages/Azure.Messaging.ServiceBus/)** .
1. Sélectionnez **Installer** pour procéder à l’installation, puis fermez le gestionnaire de package NuGet.

### <a name="add-code-to-send-messages-to-the-topic"></a>Ajouter du code pour envoyer des messages à la rubrique 

1. Dans Program.cs, ajoutez les instructions `using` suivantes en haut de la définition de l’espace de noms, avant la déclaration de classe :
   
    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Azure.Messaging.ServiceBus;
    ```
1. Dans la classe `Program`, déclarez les variables suivantes :

    ```csharp
        static string connectionString = "<NAMESPACE CONNECTION STRING>";
        static string topicName = "<TOPIC NAME>";
        static string subscriptionName = "<SUBSCRIPTION NAME>";
    ```

    Remplacez les valeurs suivantes :
    - `<NAMESPACE CONNECTION STRING>` par la chaîne de connexion de votre espace de noms Service Bus
    - `<TOPIC NAME>` par le nom de la rubrique
    - `<SUBSCRIPTION NAME>` par le nom de l’abonnement
2. Ajoutez une méthode nommée `SendMessageToTopicAsync` qui envoie un message à la rubrique. 

    ```csharp
        static async Task SendMessageToTopicAsync()
        {
            // create a Service Bus client 
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {
                // create a sender for the topic
                ServiceBusSender sender = client.CreateSender(topicName);
                await sender.SendMessageAsync(new ServiceBusMessage("Hello, World!"));
                Console.WriteLine($"Sent a single message to the topic: {topicName}");
            }
        }
    ```
1. Ajoutez une méthode nommée `CreateMessages` à la classe `Program` pour créer une file d’attente (file d’attente .NET) de messages. En général, vous obtenez ces messages à partir de différentes parties de votre application. Ici, nous créons une file d’attente d’exemples de messages.

    ```csharp
        static Queue<ServiceBusMessage> CreateMessages()
        {
            // create a queue containing the messages and return it to the caller
            Queue<ServiceBusMessage> messages = new Queue<ServiceBusMessage>();
            messages.Enqueue(new ServiceBusMessage("First message"));
            messages.Enqueue(new ServiceBusMessage("Second message"));
            messages.Enqueue(new ServiceBusMessage("Third message"));
            return messages;
        }
    ```
1. Ajoutez une méthode nommée `SendMessageBatchAsync` à la classe `Program` et ajoutez le code suivant. Cette méthode prend une file d’attente de messages et prépare un ou plusieurs lots à envoyer à la rubrique Service Bus. 

    ```csharp
        static async Task SendMessageBatchToTopicAsync()
        {
            // create a Service Bus client 
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {

                // create a sender for the topic 
                ServiceBusSender sender = client.CreateSender(topicName);

                // get the messages to be sent to the Service Bus topic
                Queue<ServiceBusMessage> messages = CreateMessages();

                // total number of messages to be sent to the Service Bus topic
                int messageCount = messages.Count;

                // while all messages are not sent to the Service Bus topic
                while (messages.Count > 0)
                {
                    // start a new batch 
                    using ServiceBusMessageBatch messageBatch = await sender.CreateMessageBatchAsync();

                    // add the first message to the batch
                    if (messageBatch.TryAddMessage(messages.Peek()))
                    {
                        // dequeue the message from the .NET queue once the message is added to the batch
                        messages.Dequeue();
                    }
                    else
                    {
                        // if the first message can't fit, then it is too large for the batch
                        throw new Exception($"Message {messageCount - messages.Count} is too large and cannot be sent.");
                    }

                    // add as many messages as possible to the current batch
                    while (messages.Count > 0 && messageBatch.TryAddMessage(messages.Peek()))
                    {
                        // dequeue the message from the .NET queue as it has been added to the batch
                        messages.Dequeue();
                    }

                    // now, send the batch
                    await sender.SendMessagesAsync(messageBatch);

                    // if there are any remaining messages in the .NET queue, the while loop repeats 
                }

                Console.WriteLine($"Sent a batch of {messageCount} messages to the topic: {topicName}");
            }
        }
    ```
1. Remplacez la méthode `Main()` par la méthode **asynchrone** `Main` suivante. Elle appelle ces deux méthodes Send pour envoyer un message unique et un lot de messages à la rubrique.  

    ```csharp
        static async Task Main()
        {
            // send a message to the topic
            await SendMessageToTopicAsync();

            // send a batch of messages to the topic
            await SendMessageBatchToTopicAsync();
        }
    ```
5. Exécutez l'application. Vous devez normalement voir la sortie suivante.

    ```console
    Sent a single message to the topic: mytopic
    Sent a batch of 3 messages to the topic: mytopic
    ```
1. Dans le portail Azure, procédez comme suit :
    1. Accédez à votre espace de noms Service Bus. 
    1. Dans la page **Vue d’ensemble**, dans le volet central inférieur, basculez vers l’onglet **Rubriques**, puis sélectionnez la rubrique Service Bus. Dans l’exemple suivant, il s’agit de `mytopic`.
    
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/select-topic.png" alt-text="Sélectionner la rubrique":::
    1. Dans la page **Rubrique Service Bus**, dans le graphique **Messages** de la section inférieure **Métriques**, vous pouvez voir quatre messages entrants pour la rubrique. Si vous ne voyez pas cette valeur, attendez quelques minutes et actualisez la page pour voir une mise à jour du graphique. 

        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/sent-messages-essentials.png" alt-text="Messages envoyés à la rubrique" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/sent-messages-essentials.png":::
    4. Sélectionnez l’abonnement dans le volet inférieur. Dans l’exemple suivant, il s’agit de **S1**. Dans la page **Abonnement Service Bus**, vous voyez que **Nombre de messages actifs** a pour valeur **4**. L’abonnement a reçu les quatre messages que vous avez envoyés à la rubrique, mais aucun destinataire ne les a encore collectés. 
    
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page.png" alt-text="Messages reçus au niveau de l’abonnement" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page.png":::
    

## <a name="receive-messages-from-a-subscription"></a>Réception des messages d’un abonnement

1. Ajoutez les méthodes suivantes à la classe `Program` pour traiter les messages et les erreurs éventuelles. 

    ```csharp
        static async Task MessageHandler(ProcessMessageEventArgs args)
        {
            string body = args.Message.Body.ToString();
            Console.WriteLine($"Received: {body} from subscription: {subscriptionName}");

            // complete the message. messages is deleted from the queue. 
            await args.CompleteMessageAsync(args.Message);
        }

        static Task ErrorHandler(ProcessErrorEventArgs args)
        {
            Console.WriteLine(args.Exception.ToString());
            return Task.CompletedTask;
        }
    ```
1. Ajoutez la méthode `ReceiveMessagesFromSubscriptionAsync` suivante à la classe `Program`.

    ```csharp
        static async Task ReceiveMessagesFromSubscriptionAsync()
        {
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {
                // create a processor that we can use to process the messages
                ServiceBusProcessor processor = client.CreateProcessor(topicName, subscriptionName, new ServiceBusProcessorOptions());

                // add handler to process messages
                processor.ProcessMessageAsync += MessageHandler;

                // add handler to process any errors
                processor.ProcessErrorAsync += ErrorHandler;

                // start processing 
                await processor.StartProcessingAsync();

                Console.WriteLine("Wait for a minute and then press any key to end the processing");
                Console.ReadKey();

                // stop processing 
                Console.WriteLine("\nStopping the receiver...");
                await processor.StopProcessingAsync();
                Console.WriteLine("Stopped receiving messages");
            }
        }
    ```
1. Ajoutez un appel à la méthode `ReceiveMessagesFromSubscriptionAsync` dans la méthode `Main`. Mettez en commentaire la méthode `SendMessagesToTopicAsync` si vous souhaitez tester uniquement la réception des messages. Dans le cas contraire, vous voyez quatre autres messages envoyés à la rubrique. 

    ```csharp
        static async Task Main()
        {
            // send a message to the topic
            await SendMessageToTopicAsync();

            // send a batch of messages to the topic
            await SendMessageBatchToTopicAsync();

            // receive messages from the subscription
            await ReceiveMessagesFromSubscriptionAsync();
        }
    ```
## <a name="run-the-app"></a>Exécuter l’application
Exécutez l'application. Attendez une minute, puis appuyez sur une touche quelconque pour cesser de recevoir des messages. Vous devriez voir la sortie suivante (barre d’espace pour la touche). 

```console
Sent a single message to the topic: mytopic
Sent a batch of 3 messages to the topic: mytopic
Wait for a minute and then press any key to end the processing
Received: Hello, World! from subscription: mysub
Received: First message from subscription: mysub
Received: Second message from subscription: mysub
Received: Third message from subscription: mysub
Received: Hello, World! from subscription: mysub
Received: First message from subscription: mysub
Received: Second message from subscription: mysub
Received: Third message from subscription: mysub

Stopping the receiver...
Stopped receiving messages
```

Vérifiez à nouveau le portail. 

- Dans la page **Rubrique Service Bus**, dans le graphique **Messages**, vous voyez huit messages entrants et huit messages sortants. Si vous ne voyez pas ces valeurs, attendez quelques minutes et actualisez la page pour voir une mise à jour du graphique. 

    :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/messages-size-final.png" alt-text="Messages envoyés et reçus" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/messages-size-final.png":::
- Dans la page **Abonnement Service Bus**, vous voyez que **Nombre de messages actifs** est égal à zéro. Cela est dû au fait qu’un destinataire a reçu des messages de cet abonnement et a terminé les messages. 

    :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page-final.png" alt-text="Nombre de messages actifs au niveau de l’abonnement à la fin" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page-final.png":::
    


## <a name="next-steps"></a>Étapes suivantes
Consultez la documentation et les exemples suivants :

- [Bibliothèque de client Azure Service Bus pour .NET – Fichier Lisez-moi](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus)
- [Exemples sur GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus/samples)
- [Informations de référence sur l’API .NET](/dotnet/api/azure.messaging.servicebus?preserve-view=true&view=azure-dotnet-preview)