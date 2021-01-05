---
title: Bien démarrer avec les files d’attente Azure Service Bus (Azure.Messaging.ServiceBus)
description: Dans ce tutoriel, vous créez une application C# .NET Core afin d’échanger des messages avec une file d’attente Service Bus.
ms.topic: quickstart
ms.tgt_pltfrm: dotnet
ms.date: 11/13/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: f55af61a061bf3a3897569058aace728f7465b64
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96862120"
---
# <a name="send-messages-to-and-receive-messages-from-azure-service-bus-queues-net"></a>Échanger des messages avec des files d’attente Azure Service Bus (.NET)
Dans ce tutoriel, vous créez une application console .NET Core afin d’échanger des messages avec une file d’attente Service Bus à l’aide du package **Azure.Messaging.ServiceBus**. 

> [!Important]
> Ce guide de démarrage rapide utilise le nouveau package Azure.Messaging.ServiceBus. Pour un guide de démarrage rapide qui utilise l’ancien package Microsoft.Azure.ServiceBus, consultez [Envoyer et recevoir des événements à l’aide du package Microsoft.Azure.ServiceBus](service-bus-dotnet-get-started-with-queues-legacy.md).

## <a name="prerequisites"></a>Prérequis

- [Visual Studio 2019](https://www.visualstudio.com/vs)
- Un abonnement Azure. Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Vous pouvez [activer les avantages de votre abonnement MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) ou [vous inscrire pour un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Si vous n’avez pas de file d’attente à utiliser, suivez les étapes de l’article [Utiliser le portail Azure pour créer une file d’attente Service Bus](service-bus-quickstart-portal.md) pour créer une file d’attente. Notez la **chaîne de connexion** pour votre espace de noms Service Bus et le nom de la **file d’attente** que vous avez créée.

## <a name="send-messages-to-a-queue"></a>Envoi de messages à une file d'attente
Dans ce guide de démarrage rapide, vous allez créer une application console C# .NET Core pour envoyer des messages à la file d’attente.

### <a name="create-a-console-application"></a>Création d’une application console
Ouvrez Visual Studio et créez un projet **Application de console (.NET Core)** pour C#. 

### <a name="add-the-service-bus-nuget-package"></a>Ajout du package NuGet Service Bus

1. Cliquez avec le bouton droit sur le projet créé et sélectionnez **Gérer les packages NuGet**.
1. Sélectionnez **Parcourir**. Recherchez et sélectionnez **[Azure.Messaging.ServiceBus](https://www.nuget.org/packages/Azure.Messaging.ServiceBus/)** .
1. Sélectionnez **Installer** pour procéder à l’installation, puis fermez le gestionnaire de package NuGet.

### <a name="add-code-to-send-messages-to-the-queue"></a>Ajouter du code pour envoyer des messages à la file d’attente

1. Dans *Program.cs*, ajoutez les instructions `using` suivantes en haut de la définition de l’espace de noms, avant la déclaration de classe :

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    
    using Azure.Messaging.ServiceBus;
    ```

1. Dans la classe `Program`, déclarez les variables suivantes :

    ```csharp
        static string connectionString = "<NAMESPACE CONNECTION STRING>";
        static string queueName = "<QUEUE NAME>";
    ```

    Entrez votre chaîne de connexion pour l’espace de noms en tant que variable `connectionString`. Entrez le nom de votre file d’attente.

1. Juste en dessous de la méthode `Main()`, ajoutez la méthode `SendMessagesAsync()` suivante qui envoie un message :

    ```csharp
        static async Task SendMessageAsync()
        {
            // create a Service Bus client 
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {
                // create a sender for the queue 
                ServiceBusSender sender = client.CreateSender(queueName);

                // create a message that we can send
                ServiceBusMessage message = new ServiceBusMessage("Hello world!");

                // send the message
                await sender.SendMessageAsync(message);
                Console.WriteLine($"Sent a single message to the queue: {queueName}");
            }
        }
    ```
1. Ajoutez une méthode nommée `CreateMessages` à la classe `Program` pour créer une file d’attente (file d’attente .NET) de messages. En général, vous obtenez ces messages à partir de différentes parties de votre application. Ici, nous créons une file d’attente d’exemples de messages.

    ```csharp
        static Queue<ServiceBusMessage> CreateMessages()
        {
            // create a queue containing the messages and return it to the caller
            Queue<ServiceBusMessage> messages = new Queue<ServiceBusMessage>();
            messages.Enqueue(new ServiceBusMessage("First message in the batch"));
            messages.Enqueue(new ServiceBusMessage("Second message in the batch"));
            messages.Enqueue(new ServiceBusMessage("Third message in the batch"));
            return messages;
        }
    ```
1. Ajoutez une méthode nommée `SendMessageBatchAsync` à la classe `Program` et ajoutez le code suivant. Cette méthode prend une file d’attente de messages et prépare un ou plusieurs lots à envoyer à la file d’attente Service Bus. 

    ```csharp
        static async Task SendMessageBatchAsync()
        {
            // create a Service Bus client 
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {
                // create a sender for the queue 
                ServiceBusSender sender = client.CreateSender(queueName);

                // get the messages to be sent to the Service Bus queue
                Queue<ServiceBusMessage> messages = CreateMessages();

                // total number of messages to be sent to the Service Bus queue
                int messageCount = messages.Count;

                // while all messages are not sent to the Service Bus queue
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

                Console.WriteLine($"Sent a batch of {messageCount} messages to the topic: {queueName}");
            }
        }
    ```
1. Remplacez la méthode `Main()` par la méthode **asynchrone** `Main` suivante. Elle appelle ces deux méthodes Send pour envoyer un message unique et un lot de messages à la file d’attente. 

    ```csharp
        static async Task Main()
        {
            // send a message to the queue
            await SendMessageAsync();

            // send a batch of messages to the queue
            await SendMessageBatchAsync();
        }
    ```
5. Exécutez l'application. Vous devriez voir les messages suivants. 

    ```console
    Sent a single message to the queue: myqueue
    Sent a batch of messages to the queue: myqueue
    ```       
1. Dans le portail Azure, procédez comme suit :
    1. Accédez à votre espace de noms Service Bus. 
    1. Dans la page **Vue d’ensemble**, sélectionnez la file d’attente dans le volet central inférieur. 
    1. Remarquez les valeurs figurant dans la section **Bases**.

    :::image type="content" source="./media/service-bus-dotnet-get-started-with-queues/sent-messages-essentials.png" alt-text="Messages reçus avec nombre et taille" lightbox="./media/service-bus-dotnet-get-started-with-queues/sent-messages-essentials.png":::

    Remarquez les valeurs suivantes :
    - La valeur **Nombre de messages actifs** pour la file d’attente est à présent **4**. Chaque fois que vous exécutez cette application d’expéditeur sans récupérer les messages, cette valeur augmente de 4.
    - La taille actuelle de la file d’attente incrémente la valeur **ACTUEL** dans **Fondamentaux** chaque fois que l’application ajoute des messages à la file d’attente.
    - Dans le graphique **Messages** de la section inférieure **Métriques**, vous pouvez voir qu’il y a quatre messages entrants pour la file d’attente. 

## <a name="receive-messages-from-a-queue"></a>Réception des messages d'une file d'attente
Dans cette section, vous allez ajouter du code pour récupérer les messages de la file d’attente.

1. Ajoutez les méthodes suivantes à la classe `Program` pour traiter les messages et les erreurs éventuelles. 

    ```csharp
        // handle received messages
        static async Task MessageHandler(ProcessMessageEventArgs args)
        {
            string body = args.Message.Body.ToString();
            Console.WriteLine($"Received: {body}");

            // complete the message. messages is deleted from the queue. 
            await args.CompleteMessageAsync(args.Message);
        }

        // handle any errors when receiving messages
        static Task ErrorHandler(ProcessErrorEventArgs args)
        {
            Console.WriteLine(args.Exception.ToString());
            return Task.CompletedTask;
        }
    ```
1. Ajoutez une méthode nommée `ReceiveMessagesAsync` à la classe `Program` et ajoutez le code suivant pour recevoir les messages. 

    ```csharp
        static async Task ReceiveMessagesAsync()
        {
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {
                // create a processor that we can use to process the messages
                ServiceBusProcessor processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions());

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
1. Ajoutez un appel à la méthode `ReceiveMessagesAsync` à partir de la méthode `Main`. Mettez en commentaire la méthode `SendMessagesAsync` si vous souhaitez tester uniquement la réception des messages. Dans le cas contraire, vous voyez quatre autres messages envoyés à la file d’attente. 

    ```csharp
        static async Task Main()
        {
            // send a message to the queue
            await SendMessageAsync();

            // send a batch of messages to the queue
            await SendMessageBatchAsync();

            // receive message from the queue
            await ReceiveMessagesAsync();
        }
    ```

## <a name="run-the-app"></a>Exécuter l’application
Exécutez l'application. Attendez une minute, puis appuyez sur une touche quelconque pour cesser de recevoir des messages. Vous devriez voir la sortie suivante (barre d’espace pour la touche). 

```console
Sent a single message to the queue: myqueue
Sent a batch of messages to the queue: myqueue
Wait for a minute and then press any key to end the processing
Received: Hello world!
Received: First message in the batch
Received: Second message in the batch
Received: Third message in the batch
Received: Hello world!
Received: First message in the batch
Received: Second message in the batch
Received: Third message in the batch

Stopping the receiver...
Stopped receiving messages
```

Vérifiez à nouveau le portail. 

- Les valeurs **Nombre de messages actifs** et **ACTUEL** sont à présent de **0**.
- Dans le graphique **Messages** de la section inférieure **Métriques**, vous pouvez voir huit messages entrants et huit messages sortants pour la file d’attente. 

    :::image type="content" source="./media/service-bus-dotnet-get-started-with-queues/queue-messages-size-final.png" alt-text="Messages actifs et taille après réception" lightbox="./media/service-bus-dotnet-get-started-with-queues/queue-messages-size-final.png":::

## <a name="next-steps"></a>Étapes suivantes
Consultez la documentation et les exemples suivants :

- [Bibliothèque de client Azure Service Bus pour .NET – Fichier Lisez-moi](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus)
- [Exemples sur GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus/samples)
- [Informations de référence sur l’API .NET](/dotnet/api/azure.messaging.servicebus?preserve-view=true&view=azure-dotnet-preview)