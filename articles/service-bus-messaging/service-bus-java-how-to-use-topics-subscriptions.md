---
title: Utiliser des rubriques et abonnements Azure Service Bus avec Java (azure-messaging-servicebus)
description: Dans ce guide de démarrage rapide, vous allez écrire du code Java à l’aide du package azure-messaging-servicebus pour envoyer des messages à une rubrique Azure Service Bus, puis vous recevrez les messages des abonnements à cette rubrique.
ms.devlang: Java
ms.topic: quickstart
ms.date: 11/09/2020
ms.openlocfilehash: cd95b84c4b55279b40f95eef65cb2490a55d1780
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97630239"
---
# <a name="send-messages-to-an-azure-service-bus-topic-and-receive-messages-from-subscriptions-to-the-topic-java"></a>Envoyer des messages à une rubrique Azure Service Bus et recevoir des messages à partir d’abonnements à la rubrique (Java)
Dans ce guide de démarrage rapide, vous allez écrire du code Java à l’aide du package azure-messaging-servicebus pour envoyer des messages à une rubrique Azure Service Bus, puis vous recevrez les messages des abonnements à cette rubrique.

> [!IMPORTANT]
> Ce guide de démarrage rapide utilise le nouveau package azure-messaging-servicebus. Pour un guide de démarrage rapide qui utilise l’ancien package azure-servicebus, consultez [Envoyer et recevoir des messages à l’aide d’azure-servicebus](service-bus-java-how-to-use-topics-subscriptions-legacy.md).

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure. Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Vous pouvez activer les [avantages de votre abonnement Visual Studio ou MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) ou vous inscrire pour créer un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Suivez les étapes dans [Démarrage rapide : Utiliser le portail Azure pour créer une rubrique Service Bus et des abonnements à cette rubrique](service-bus-quickstart-topics-subscriptions-portal.md). Notez la chaîne de connexion, le nom de la rubrique et un nom d’abonnement. Vous n’utiliserez qu’un seul abonnement dans ce guide de démarrage rapide. 
- Installez le [kit de développement logiciel (SDK) Azure pour Java][Azure SDK for Java]. Si vous utilisez Eclipse, vous pouvez installer le [Kit de ressources Azure pour Eclipse][Azure Toolkit for Eclipse] qui inclut le kit SDK Azure pour Java. Vous pouvez ensuite ajouter les **bibliothèques Microsoft Azure pour Java** à votre projet. Si vous utilisez IntelliJ, consultez [Installer Azure Toolkit for IntelliJ](/azure/developer/java/toolkit-for-intellij/installation). 


## <a name="send-messages-to-a-topic"></a>Envoi de messages à une rubrique
Dans cette section, vous allez créer un projet de console Java et ajouter du code pour envoyer des messages à la rubrique que vous avez créée. 

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

### <a name="add-code-to-send-messages-to-the-topic"></a>Ajouter du code pour envoyer des messages à la rubrique
1. Ajoutez les instructions `import` suivantes au début du fichier Java. 

    ```java
    import com.azure.messaging.servicebus.*;
    import com.azure.messaging.servicebus.models.*;
    import java.util.concurrent.TimeUnit;
    import java.util.function.Consumer;
    import java.util.Arrays;
    import java.util.List;
    ```    
5. Dans la classe, définissez des variables pour stocker la chaîne de connexion et le nom de la rubrique, comme indiqué ci-dessous : 

    ```java
    static String connectionString = "<NAMESPACE CONNECTION STRING>";
    static String topicName = "<TOPIC NAME>";    
    static String subName = "<SUBSCRIPTION NAME>";
    ```

    Remplacez `<NAMESPACE CONNECTION STRING>` par la chaîne de connexion de votre espace de noms Service Bus, et remplacez `<TOPIC NAME>` par le nom de la rubrique.
3. Ajoutez une méthode nommée `sendMessage` dans la classe pour envoyer un message à la rubrique. 

    ```java
        static void sendMessage()
    {
        // create a Service Bus Sender client for the queue 
        ServiceBusSenderClient senderClient = new ServiceBusClientBuilder()
                .connectionString(connectionString)
                .sender()
                .topicName(topicName)
                .buildClient();
        
        // send one message to the topic
        senderClient.sendMessage(new ServiceBusMessage("Hello, World!"));
        System.out.println("Sent a single message to the topic: " + topicName);        
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
1. Ajoutez une méthode nommée `sendMessageBatch` pour envoyer des messages à la rubrique que vous avez créée. Cette méthode crée un `ServiceBusSenderClient` pour la rubrique, appelle la méthode `createMessages` pour obtenir la liste des messages, prépare un ou plusieurs lots, et envoie les lots à la rubrique. 

```java
    static void sendMessageBatch()
    {
        // create a Service Bus Sender client for the topic 
        ServiceBusSenderClient senderClient = new ServiceBusClientBuilder()
                .connectionString(connectionString)
                .sender()
                .topicName(topicName)
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
            System.out.println("Sent a batch of messages to the topic: " + topicName);
            
            // create a new batch
            messageBatch = senderClient.createMessageBatch();

            // Add that message that we couldn't before.
            if (!messageBatch.tryAddMessage(message)) {
                System.err.printf("Message is too large for an empty batch. Skipping. Max size: %s.", messageBatch.getMaxSizeInBytes());
            }
        }

        if (messageBatch.getCount() > 0) {
            senderClient.sendMessages(messageBatch);
            System.out.println("Sent a batch of messages to the topic: " + topicName);
        }

        //close the client
        senderClient.close();
    }
```

## <a name="receive-messages-from-a-subscription"></a>Réception des messages d’un abonnement
Dans cette section, vous allez ajouter du code pour récupérer les messages à partir d’un abonnement à la rubrique. 

1. Ajoutez une méthode nommée `receiveMessages` pour recevoir des messages à partir de l’abonnement. Cette méthode crée un `ServiceBusProcessorClient` pour l’abonnement en spécifiant un gestionnaire pour le traitement des messages et un autre pour la gestion des erreurs. Ensuite, elle démarre le processeur, attend quelques secondes, imprime les messages reçus, puis arrête et ferme le processeur.

    ```java
    // handles received messages
    static void receiveMessages() throws InterruptedException
    {
        // Consumer that processes a single message received from Service Bus
        Consumer<ServiceBusReceivedMessageContext> messageProcessor = context -> {
            ServiceBusReceivedMessage message = context.getMessage();
            System.out.println("Received message: " + message.getBody().toString() + " from the subscription: " + subName);
        };

        // Consumer that handles any errors that occur when receiving messages
        Consumer<Throwable> errorHandler = throwable -> {
            System.out.println("Error when receiving messages: " + throwable.getMessage());
            if (throwable instanceof ServiceBusReceiverException) {
                ServiceBusReceiverException serviceBusReceiverException = (ServiceBusReceiverException) throwable;
                System.out.println("Error source: " + serviceBusReceiverException.getErrorSource());
            }
        };

        // Create an instance of the processor through the ServiceBusClientBuilder
        ServiceBusProcessorClient processorClient = new ServiceBusClientBuilder()
            .connectionString(connectionString)
            .processor()
            .topicName(topicName)
            .subscriptionName(subName)
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
Exécutez le programme pour obtenir une sortie de ce type :

```console
Sent a batch of messages to the topic: mytopic
Starting the processor
Received message: First message from the subscription: mysub
Received message: Second message from the subscription: mysub
Received message: Third message from the subscription: mysub
Stopping and closing the processor
```

Dans la page **Vue d’ensemble** de l’espace de noms Service Bus dans le portail Azure, vous pouvez voir le nombre de messages **entrants** et **sortants**. Vous devrez peut-être attendre environ une minute puis actualiser la page pour voir les valeurs les plus récentes. 

:::image type="content" source="./media/service-bus-java-how-to-use-queues/overview-incoming-outgoing-messages.png" alt-text="Nombre de messages entrants et sortants" lightbox="./media/service-bus-java-how-to-use-queues/overview-incoming-outgoing-messages.png":::

Basculez vers l’onglet **Rubriques** dans le volet du milieu inférieur, puis sélectionnez la rubrique pour afficher la page **Rubrique Service Bus** correspondant à votre rubrique. Dans cette page, vous devez voir quatre messages entrants et quatre messages sortants dans le graphique **Messages**. 

:::image type="content" source="./media/service-bus-java-how-to-use-topics-subscriptions/topic-page-portal.png" alt-text="Messages entrants et sortants" lightbox="./media/service-bus-java-how-to-use-topics-subscriptions/topic-page-portal.png":::

Si vous commentez l’appel `receiveMessages` dans la méthode `main` et que vous réexécutez l’application, dans la page **Rubrique Service Bus** vous verrez huit messages entrants (quatre nouveaux) mais quatre messages sortants. 

:::image type="content" source="./media/service-bus-java-how-to-use-topics-subscriptions/updated-topic-page.png" alt-text="Page de rubrique mise à jour" lightbox="./media/service-bus-java-how-to-use-topics-subscriptions/updated-topic-page.png":::

Dans cette page, si vous sélectionnez un abonnement, vous accédez à la page **Abonnement Service Bus**. Elle indique entre autres le nombre de messages actifs et le nombre de messages de lettres mortes. Dans cet exemple, quatre messages actifs n’ont pas encore été reçus par un récepteur. 

:::image type="content" source="./media/service-bus-java-how-to-use-topics-subscriptions/active-message-count.png" alt-text="Nombre de messages actifs" lightbox="./media/service-bus-java-how-to-use-topics-subscriptions/active-message-count.png":::

## <a name="next-steps"></a>Étapes suivantes
Voir la documentation et les exemples suivants :

- [Bibliothèque de client Azure Service Bus pour Java - Lisez-moi](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/servicebus/azure-messaging-servicebus/README.md)
- [Exemples sur GitHub](https://docs.microsoft.com/samples/azure/azure-sdk-for-java/servicebus-samples/)
- [Informations de référence sur l’API Java](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-messaging-servicebus/7.0.0/index.html)


[Azure SDK for Java]: /java/api/overview/azure/
[Azure Toolkit for Eclipse]: /azure/developer/java/toolkit-for-eclipse/installation
[Service Bus queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[SqlFilter]: /dotnet/api/microsoft.azure.servicebus.sqlfilter
[SqlFilter.SqlExpression]: /dotnet/api/microsoft.azure.servicebus.sqlfilter.sqlexpression
[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
