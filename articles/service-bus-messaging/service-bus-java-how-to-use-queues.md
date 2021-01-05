---
title: Utiliser des files d’attente Azure Service Bus avec Java (azure-messaging-servicebus)
description: Dans ce tutoriel, vous allez apprendre à utiliser Java afin d’envoyer et de recevoir des messages à partir d’une file d’attente Azure Service Bus. Vous utiliserez le nouveau package azure-messaging-servicebus.
ms.devlang: Java
ms.topic: quickstart
ms.date: 11/09/2020
ms.custom: seo-java-july2019, seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: d95c96e76a3463a77cc64234a909cc1e3d093837
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97630222"
---
# <a name="send-messages-to-and-receive-messages-from-azure-service-bus-queues-java"></a>Envoyer et recevoir des messages à partir de files d’attente Azure Service Bus (Java)
Dans ce guide de démarrage rapide, vous allez créer une application Java afin d’envoyer et de recevoir des messages à partir d’une file d’attente Azure Service Bus. 

> [!IMPORTANT]
> Ce guide de démarrage rapide utilise le nouveau package azure-messaging-servicebus. Pour un guide de démarrage rapide qui utilise l’ancien package azure-servicebus, consultez [Envoyer et recevoir des messages à l’aide de azure-servicebus](service-bus-java-how-to-use-queues-legacy.md).


## <a name="prerequisites"></a>Prérequis
- Un abonnement Azure. Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Vous pouvez [activer les avantages de votre abonnement MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) ou [vous inscrire pour un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Si vous n’avez pas de file d’attente à utiliser, suivez les étapes de l’article [Utiliser le portail Azure pour créer une file d’attente Service Bus](service-bus-quickstart-portal.md) pour créer une file d’attente. Notez la **chaîne de connexion** pour votre espace de noms Service Bus et le nom de la **file d’attente** que vous avez créée.
- Installez le [kit de développement logiciel (SDK) Azure pour Java][Azure SDK for Java]. Si vous utilisez Eclipse, vous pouvez installer [Azure Toolkit for Eclipse][Azure Toolkit for Eclipse], qui inclut le SDK Azure pour Java. Vous pouvez ensuite ajouter les **bibliothèques Microsoft Azure pour Java** à votre projet. Si vous utilisez IntelliJ, consultez [Installer Azure Toolkit for IntelliJ](/azure/developer/java/toolkit-for-intellij/installation). 


## <a name="send-messages-to-a-queue"></a>Envoi de messages à une file d'attente
Dans cette section, vous allez créer un projet de console Java et ajouter du code pour envoyer des messages à la file d’attente que vous avez créée. 

### <a name="create-a-java-console-project"></a>Créer un projet de console Java
Créez un projet Java avec Eclipse ou un outil de votre choix. 

### <a name="configure-your-application-to-use-service-bus"></a>Configuration de votre application pour l’utilisation de Service Bus
Ajoutez une référence à la bibliothèque Azure Service Bus. La bibliothèque de client Java pour Service Bus est disponible dans le [Référentiel central Maven](https://search.maven.org/search?q=a:azure-messaging-servicebus). Vous pouvez référencer cette bibliothèque à l’aide de la déclaration de dépendance suivante au sein de votre fichier de projet Maven :

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-messaging-servicebus</artifactId>
    <version>7.0.0</version>
</dependency>
```

### <a name="add-code-to-send-messages-to-the-queue"></a>Ajouter du code pour envoyer des messages à la file d’attente
1. Ajoutez les instructions `import` suivantes au début du fichier Java. 

    ```java
    import com.azure.messaging.servicebus.*;
    import com.azure.messaging.servicebus.models.*;
    import java.util.concurrent.TimeUnit;
    import java.util.function.Consumer;
    import java.util.Arrays;
    import java.util.List;
    ```    
5. Dans la classe, définissez des variables pour stocker la chaîne de connexion et le nom de la file d’attente, comme indiqué ci-dessous : 

    ```java
    static String connectionString = "<NAMESPACE CONNECTION STRING>";
    static String queueName = "<QUEUE NAME>";    
    ```

    Remplacez `<NAMESPACE CONNECTION STRING>` par la chaîne de connexion de votre espace de noms Service Bus, et remplacez `<QUEUE NAME>` par le nom de la file d’attente.
3. Ajoutez une méthode nommée `sendMessage` dans la classe pour envoyer un message à la file d’attente. 

    ```java
    static void sendMessage()
    {
        // create a Service Bus Sender client for the queue 
        ServiceBusSenderClient senderClient = new ServiceBusClientBuilder()
                .connectionString(connectionString)
                .sender()
                .queueName(queueName)
                .buildClient();
        
        // send one message to the queue
        senderClient.sendMessage(new ServiceBusMessage("Hello, World!"));
        System.out.println("Sent a single message to the queue: " + queueName);        
    }
    ```
1. Ajoutez une méthode nommée `createMessages` dans la classe pour créer une liste de messages. En général, vous recevez ces messages à partir de différentes parties de votre application. Ici, nous créons une liste d’exemples de messages.

    ```java
    static List<ServiceBusMessage> createMessages()
    {
        // create a list of messages and return it to the caller
        ServiceBusMessage[] messages = {
                new ServiceBusMessage("First message"),
                new ServiceBusMessage("Second message"),
                new ServiceBusMessage("Third message")
        };
        return Arrays.asList(messages);
    }
    ```
1. Ajoutez une méthode nommée `sendMessageBatch` pour envoyer des messages à la file d’attente que vous avez créée. Cette méthode crée un `ServiceBusSenderClient` pour la file d’attente, appelle la méthode `createMessages` pour obtenir la liste des messages, prépare un ou plusieurs lots, et envoie les lots à la file d’attente. 

```java
    static void sendMessageBatch()
    {
        // create a Service Bus Sender client for the queue 
        ServiceBusSenderClient senderClient = new ServiceBusClientBuilder()
                .connectionString(connectionString)
                .sender()
                .queueName(queueName)
                .buildClient();

        // Creates an ServiceBusMessageBatch where the ServiceBus.
        ServiceBusMessageBatch messageBatch = senderClient.createMessageBatch();        
        
        // create a list of messages
        List<ServiceBusMessage> listOfMessages = createMessages();
        
        // We try to add as many messages as a batch can fit based on the maximum size and send to Service Bus when
        // the batch can hold no more messages. Create a new batch for next set of messages and repeat until all
        // messages are sent.        
        for (ServiceBusMessage message : listOfMessages) {
            if (messageBatch.tryAddMessage(message)) {
                continue;
            }

            // The batch is full, so we create a new batch and send the batch.
            senderClient.sendMessages(messageBatch);
            System.out.println("Sent a batch of messages to the queue: " + queueName);
            
            // create a new batch
            messageBatch = senderClient.createMessageBatch();

            // Add that message that we couldn't before.
            if (!messageBatch.tryAddMessage(message)) {
                System.err.printf("Message is too large for an empty batch. Skipping. Max size: %s.", messageBatch.getMaxSizeInBytes());
            }
        }
        
        if (messageBatch.getCount() > 0) {
            senderClient.sendMessages(messageBatch);
            System.out.println("Sent a batch of messages to the queue: " + queueName);
        }

        //close the client
        senderClient.close();
    }
```

## <a name="receive-messages-from-a-queue"></a>Réception des messages d'une file d'attente
Dans cette section, vous allez ajouter du code pour récupérer des messages à partir de la file d’attente. 

1. Ajoutez une méthode nommée `receiveMessages` pour recevoir des messages à partir de la file d’attente. Cette méthode crée un `ServiceBusProcessorClient` pour la file d’attente en spécifiant un gestionnaire pour le traitement des messages et un autre pour la gestion des erreurs. Ensuite, elle démarre le processeur, attend quelques secondes, imprime les messages reçus, puis arrête et ferme le processeur.

    ```java
    // handles received messages
    static void receiveMessages() throws InterruptedException
    {
        // consumer that processes a single message received from Service Bus
        Consumer<ServiceBusReceivedMessageContext> messageProcessor = context -> {
            ServiceBusReceivedMessage message = context.getMessage();
            System.out.println("Received message: " + message.getBody().toString());
        };

        // handles any errors that occur when receiving messages
        Consumer<Throwable> errorHandler = throwable -> {
            System.out.println("Error when receiving messages: " + throwable.getMessage());
            if (throwable instanceof ServiceBusReceiverException) {
                ServiceBusReceiverException serviceBusReceiverException = (ServiceBusReceiverException) throwable;
                System.out.println("Error source: " + serviceBusReceiverException.getErrorSource());
            }
        };

        // create an instance of the processor through the ServiceBusClientBuilder
        ServiceBusProcessorClient processorClient = new ServiceBusClientBuilder()
            .connectionString(connectionString)
            .processor()
            .queueName(queueName)
            .processMessage(messageProcessor)
            .processError(errorHandler)
            .buildProcessorClient();

        System.out.println("Starting the processor");
        processorClient.start();

        TimeUnit.SECONDS.sleep(10);
        System.out.println("Stopping and closing the processor");
        processorClient.close();        
    }    
    ```
2. Mettez à jour la méthode `main` pour appeler les méthodes `sendMessage`, `sendMessageBatch`et `receiveMessages`, et pour lever `InterruptedException`.     

    ```java
    public static void main(String[] args) throws InterruptedException {        
        sendMessage();
        sendMessageBatch();
        receiveMessages();
    }   
    ```

## <a name="run-the-app"></a>Exécuter l’application
Lorsque vous exécutez l’application, les messages suivants s’affichent dans la fenêtre de console. 

```console
Sent a single message to the queue: myqueue
Sent a batch of messages to the queue: myqueue
Starting the processor
Received message: Hello, World!
Received message: First message in the batch
Received message: Second message in the batch
Received message: Three message in the batch
Stopping and closing the processor
```

Dans la page **Vue d’ensemble** de l’espace de noms Service Bus dans le portail Azure, vous pouvez voir le nombre de messages **entrants** et **sortants**. Vous devrez peut-être attendre environ une minute puis actualiser la page pour voir les valeurs les plus récentes. 

:::image type="content" source="./media/service-bus-java-how-to-use-queues/overview-incoming-outgoing-messages.png" alt-text="Nombre de messages entrants et sortants" lightbox="./media/service-bus-java-how-to-use-queues/overview-incoming-outgoing-messages.png":::

Sélectionnez la file d’attente dans cette page **Vue d’ensemble** pour accéder à la page **File d’attente Service Bus**. Le nombre de messages **entrants** et **sortants** est visible dans cette page, ainsi que d’autres informations telles que la **taille actuelle** de la file d’attente, la **taille maximale** et le **nombre de messages actifs**. 

:::image type="content" source="./media/service-bus-java-how-to-use-queues/queue-details.png" alt-text="Détails de la file d’attente" lightbox="./media/service-bus-java-how-to-use-queues/queue-details.png":::



## <a name="next-steps"></a>Étapes suivantes
Voir la documentation et les exemples suivants :

- [Bibliothèque de client Azure Service Bus pour Java - Lisez-moi](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/servicebus/azure-messaging-servicebus/README.md)
- [Exemples sur GitHub](https://docs.microsoft.com/samples/azure/azure-sdk-for-java/servicebus-samples/)
- [Informations de référence sur l’API Java](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-messaging-servicebus/7.0.0/index.html)

[Azure SDK for Java]: /azure/developer/java/sdk/java-sdk-azure-get-started
[Azure Toolkit for Eclipse]: /azure/developer/java/toolkit-for-eclipse/installation
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
