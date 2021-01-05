---
title: Recevoir des événements d’Azure Event Hubs ou lui en envoyer en utilisant Java (dernière version)
description: Cet article décrit la procédure à suivre pour créer une application Java qui reçoit des événements d’Azure Event Hubs et lui en envoie à l’aide du dernier package azure-messaging-eventhubs.
ms.topic: quickstart
ms.date: 06/23/2020
ms.custom: devx-track-java
ms.openlocfilehash: f543fae8087a7dd3a18da7b44bc2896d7607f3d2
ms.sourcegitcommit: 638f326d02d108cf7e62e996adef32f2b2896fd5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91728962"
---
# <a name="use-java-to-send-events-to-or-receive-events-from-azure-event-hubs-azure-messaging-eventhubs"></a>Utiliser Java pour recevoir des événements d’Azure Event Hubs ou lui en envoyer (azure-messaging-eventhubs)
Ce guide de démarrage rapide montre comment recevoir des événements d’un hub d’événements et lui en envoyer en utilisant le package Java **azure-messaging-eventhubs**.

> [!IMPORTANT]
> Ce guide de démarrage rapide utilise le nouveau package **azure-messaging-eventhubs**. Pour un guide de démarrage rapide qui utilise les anciens packages **azure-eventhubs** et **azure-eventhubs-eph**, consultez [Envoyer et recevoir des événements à l’aide d’azure-eventhubs et azure-eventhubs-eph](event-hubs-java-get-started-send-legacy.md). 


## <a name="prerequisites"></a>Prérequis
Si vous débutez avec Azure Event Hubs, consultez la [vue d’ensemble d’Event Hubs](event-hubs-about.md) avant de suivre ce guide de démarrage rapide. 

Pour effectuer ce démarrage rapide, vous avez besoin de ce qui suit :

- **Abonnement Microsoft Azure**. Pour utiliser les services Azure, y compris Azure Event Hubs, vous avez besoin d’un abonnement.  Si vous n’avez pas de compte Azure, vous pouvez vous inscrire à un [essai gratuit](https://azure.microsoft.com/free/) ou utiliser les avantages de votre abonnement MSDN quand vous [créez un compte](https://azure.microsoft.com).
- Un environnement de développement Java. Ce guide de démarrage rapide utilise [Eclipse](https://www.eclipse.org/). Le Kit de développement Java (JDK) avec version 8 ou ultérieure est nécessaire. 
- **Créez un espace de noms Event Hubs et un Event Hub**. La première étape consiste à utiliser le [portail Azure](https://portal.azure.com) pour créer un espace de noms de type Event Hubs et obtenir les informations de gestion nécessaires à votre application pour communiquer avec le concentrateur d’événements. Pour créer un espace de noms et un hub d’événements, suivez la procédure décrite dans [cet article](event-hubs-create.md). Ensuite, obtenez la **chaîne de connexion de l’espace de noms Event Hubs** en suivant les instructions à partir de l’article : [Obtenir la chaîne de connexion](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Vous utiliserez la chaîne de connexion plus loin dans ce guide de démarrage rapide.

## <a name="send-events"></a>Envoyer des événements 
Cette section montre comment créer une application Java pour envoyer des événements à un hub d’événements. 

### <a name="add-reference-to-azure-event-hubs-library"></a>Ajouter une référence à la bibliothèque Azure Event Hubs

La bibliothèque cliente de Java pour Event Hubs est disponible dans le [Référentiel central Maven](https://search.maven.org/search?q=a:azure-messaging-eventhubs). Vous pouvez référencer cette bibliothèque à l’aide de la déclaration de dépendance suivante au sein de votre fichier de projet Maven :

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-messaging-eventhubs</artifactId>
    <version>5.0.1</version>
</dependency>
```

### <a name="write-code-to-send-messages-to-the-event-hub"></a>Écriture de code pour envoyer des messages à un hub d’événements

Pour l’exemple suivant, créez tout d’abord un nouveau projet Maven pour une application de console/shell dans votre environnement de développement Java favori. Ajoutez une classe nommée `Sender`, et ajoutez-y le code suivant :

```java
import com.azure.messaging.eventhubs.*;
import static java.nio.charset.StandardCharsets.UTF_8;

public class Sender {
       public static void main(String[] args) {
    }
}
```

### <a name="connection-string-and-event-hub"></a>Chaîne de connexion et hub d’événements
Ce code utilise la chaîne de connexion à l’espace de noms Event Hubs et le nom du hub d’événements pour créer un client Event Hubs. 

```java
String connectionString = "<CONNECTION STRING to EVENT HUBS NAMESPACE>";
String eventHubName = "<EVENT HUB NAME>";
```

### <a name="create-an-event-hubs-producer-client"></a>Créer un client producteur Event Hubs 
Ce code crée un objet client producteur utilisé pour produire/envoyer des événements au hub d’événements. 

```java
EventHubProducerClient producer = new EventHubClientBuilder()
    .connectionString(connectionString, eventHubName)
    .buildProducerClient();
```

### <a name="prepare-a-batch-of-events"></a>Préparer un lot d’événements
Ce code prépare un lot d’événements. 

```java
EventDataBatch batch = producer.createBatch();
batch.tryAdd(new EventData("First event"));
batch.tryAdd(new EventData("Second event"));
batch.tryAdd(new EventData("Third event"));
batch.tryAdd(new EventData("Fourth event"));
batch.tryAdd(new EventData("Fifth event"));
```

### <a name="send-the-batch-of-events-to-the-event-hub"></a>Envoyer le lot d’événements au hub d’événements
Ce code envoie au hub d’événements le lot d’événements que vous avez préparé à l’étape précédente. Le code suivant garde la main jusqu’à la fin de l’opération d’envoi. 

```java
producer.send(batch);
```

### <a name="close-and-cleanup"></a>Fermer et nettoyer
Ce code ferme le producteur. 

```java
producer.close();
```
### <a name="complete-code-to-send-events"></a>Code complet pour envoyer des événements
Voici le code complet pour envoyer des événements au hub d’événements. 

```java
import com.azure.messaging.eventhubs.*;

public class Sender {
    public static void main(String[] args) {
        final String connectionString = "EVENT HUBS NAMESPACE CONNECTION STRING";
        final String eventHubName = "EVENT HUB NAME";

        // create a producer using the namespace connection string and event hub name
        EventHubProducerClient producer = new EventHubClientBuilder()
            .connectionString(connectionString, eventHubName)
            .buildProducerClient();

        // prepare a batch of events to send to the event hub    
        EventDataBatch batch = producer.createBatch();
        batch.tryAdd(new EventData("First event"));
        batch.tryAdd(new EventData("Second event"));
        batch.tryAdd(new EventData("Third event"));
        batch.tryAdd(new EventData("Fourth event"));
        batch.tryAdd(new EventData("Fifth event"));

        // send the batch of events to the event hub
        producer.send(batch);

        // close the producer
        producer.close();
    }
}
```

Générez le programme et assurez-vous qu’il n’y a aucune erreur. Vous exécuterez ce programme après avoir exécuté le programme récepteur. 

## <a name="receive-events"></a>Recevoir des événements
Le code de ce tutoriel est basé sur l’[exemple EventProcessorClient sur GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/EventProcessorBlobCheckpointStoreSample.java), que vous pouvez analyser pour voir la version complète de l’application en cours.

> [!WARNING]
> Si vous exécutez ce code sur Azure Stack Hub, vous rencontrerez des erreurs d’exécution, sauf si vous ciblez une version spécifique de l’API Stockage. Ceci est dû au fait que le SDK Event Hubs utilise la dernière API Stockage Azure disponible dans Azure, qui peut ne pas être pas disponible sur votre plateforme Azure Stack Hub. Azure Stack Hub peut prendre en charge une autre version du SDK Stockage Blob que celles généralement disponibles sur Azure. Si vous utilisez Stockage Blob Azure comme magasin de points de contrôle, vérifiez la [version de l’API Stockage Azure prise en charge pour votre build Azure Stack Hub](/azure-stack/user/azure-stack-acs-differences?#api-version) et ciblez cette version dans votre code. 
>
> Par exemple, si vous exécutez sur Azure Stack Hub version 2005, la version la plus élevée disponible pour le service Stockage est la version 2019-02-02. Par défaut, la bibliothèque de client du SDK Event Hubs utilise la version la plus récente disponible sur Azure (2019-07-07 au moment de la publication du SDK). Dans ce cas, en plus des étapes suivantes de cette section, vous devrez également ajouter du code pour cibler la version 2019-02-02 de l’API du service Stockage. Pour obtenir un exemple sur la façon de cibler une version spécifique de l’API de stockage, consultez [cet exemple sur GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/EventProcessorWithCustomStorageVersion.java). 


### <a name="create-an-azure-storage-and-a-blob-container"></a>Créer un compte Stockage Azure et un conteneur de blobs
Dans ce guide de démarrage rapide, vous utilisez Stockage Azure (plus particulièrement, Stockage Blob) comme magasin de points de contrôle. Le marquage de point de contrôle est un processus par lequel un processeur d’événements marque ou valide la position du dernier événement correctement traité dans une partition. Le marquage d’un point de contrôle s’effectue généralement au sein de la fonction qui traite les événements. Pour en savoir plus sur le marquage de point de contrôle, consultez [Processeur d’événements](event-processor-balance-partition-load.md).

Suivez les étapes ci-dessous pour créer un compte Stockage Azure. 

1. [Création d’un compte de stockage Azure](../storage/common/storage-account-create.md?tabs=azure-portal)
2. Créer un [conteneur d’objets blob](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container)
3. [Obtenir la chaîne de connexion au compte de stockage](../storage/common/storage-configure-connection-string.md)

    Notez la **chaîne de connexion** et le **nom du conteneur**. Vous les utiliserez dans le code de réception. 

### <a name="add-event-hubs-libraries-to-your-java-project"></a>Ajouter des bibliothèques Event Hubs à votre projet Java
Ajoutez les dépendances suivantes dans le fichier pom.xml. 

- [azure-messaging-eventhubs](https://search.maven.org/search?q=a:azure-messaging-eventhubs)
- [azure-messaging-eventhubs-checkpointstore-blob](https://search.maven.org/search?q=a:azure-messaging-eventhubs-checkpointstore-blob)

```xml
<dependencies>
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-messaging-eventhubs</artifactId>
        <version>5.1.1</version>
    </dependency>
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-messaging-eventhubs-checkpointstore-blob</artifactId>
        <version>1.1.1</version>
    </dependency>
</dependencies>
```

1. Ajoutez les instructions **import** suivantes au début du fichier Java. 

    ```java
    import com.azure.messaging.eventhubs.EventHubClientBuilder;
    import com.azure.messaging.eventhubs.EventProcessorClient;
    import com.azure.messaging.eventhubs.EventProcessorClientBuilder;
    import com.azure.messaging.eventhubs.checkpointstore.blob.BlobCheckpointStore;
    import com.azure.messaging.eventhubs.models.ErrorContext;
    import com.azure.messaging.eventhubs.models.EventContext;
    import com.azure.storage.blob.BlobContainerAsyncClient;
    import com.azure.storage.blob.BlobContainerClientBuilder;
    import java.util.function.Consumer;
    import java.util.concurrent.TimeUnit;
    ```
2. Créez une classe nommée `Receiver`, puis ajoutez les variables de chaîne suivantes à la classe. Remplacez les espaces réservés par les valeurs correctes. 
    ```java
    private static final String EH_NAMESPACE_CONNECTION_STRING = "<EVENT HUBS NAMESPACE CONNECTION STRING>";
    private static final String eventHubName = "<EVENT HUB NAME>";
    private static final String STORAGE_CONNECTION_STRING = "<AZURE STORAGE CONNECTION STRING>";
    private static final String STORAGE_CONTAINER_NAME = "<AZURE STORAGE CONTAINER NAME>";
    ```
3. Ajoutez la méthode `main` suivante à la classe. 

    ```java
    public static void main(String[] args) throws Exception {
        // Create a blob container client that you use later to build an event processor client to receive and process events
        BlobContainerAsyncClient blobContainerAsyncClient = new BlobContainerClientBuilder()
            .connectionString(STORAGE_CONNECTION_STRING) 
            .containerName(STORAGE_CONTAINER_NAME) 
            .buildAsyncClient();
    
        // Create a builder object that you will use later to build an event processor client to receive and process events and errors.
        EventProcessorClientBuilder eventProcessorClientBuilder = new EventProcessorClientBuilder()
            .connectionString(EH_NAMESPACE_CONNECTION_STRING, eventHubName) 
            .consumerGroup(EventHubClientBuilder.DEFAULT_CONSUMER_GROUP_NAME)
            .processEvent(PARTITION_PROCESSOR) 
            .processError(ERROR_HANDLER) 
            .checkpointStore(new BlobCheckpointStore(blobContainerAsyncClient)); 

        // Use the builder object to create an event processor client 
        EventProcessorClient eventProcessorClient = eventProcessorClientBuilder.buildEventProcessorClient();

        System.out.println("Starting event processor");
        eventProcessorClient.start();

        System.out.println("Press enter to stop.");
        System.in.read();

        System.out.println("Stopping event processor");
        eventProcessorClient.stop();
        System.out.println("Event processor stopped.");

        System.out.println("Exiting process");
    }    
    ```
4. Ajoutez les deux méthodes d’assistance (`PARTITION_PROCESSOR` et `ERROR_HANDLER`) qui traitent les événements et les erreurs à la classe `Receiver`. 

    ```java
    public static final Consumer<EventContext> PARTITION_PROCESSOR = eventContext -> {
        System.out.printf("Processing event from partition %s with sequence number %d with body: %s %n", 
                eventContext.getPartitionContext().getPartitionId(), eventContext.getEventData().getSequenceNumber(), eventContext.getEventData().getBodyAsString());

        if (eventContext.getEventData().getSequenceNumber() % 10 == 0) {
            eventContext.updateCheckpoint();
        }
    };
    
    public static final Consumer<ErrorContext> ERROR_HANDLER = errorContext -> {
        System.out.printf("Error occurred in partition processor for partition %s, %s.%n",
            errorContext.getPartitionContext().getPartitionId(),
            errorContext.getThrowable());
    };    
    ```
3. Le code complet doit ressembler à ceci : 

    ```java

    import com.azure.messaging.eventhubs.EventHubClientBuilder;
    import com.azure.messaging.eventhubs.EventProcessorClient;
    import com.azure.messaging.eventhubs.EventProcessorClientBuilder;
    import com.azure.messaging.eventhubs.checkpointstore.blob.BlobCheckpointStore;
    import com.azure.messaging.eventhubs.models.ErrorContext;
    import com.azure.messaging.eventhubs.models.EventContext;
    import com.azure.storage.blob.BlobContainerAsyncClient;
    import com.azure.storage.blob.BlobContainerClientBuilder;
    import java.util.function.Consumer;
    import java.util.concurrent.TimeUnit;
    
    public class Receiver {
        
        private static final String EH_NAMESPACE_CONNECTION_STRING = "<EVENT HUBS NAMESPACE CONNECTION STRING>";
        private static final String eventHubName = "<EVENT HUB NAME>";
        private static final String STORAGE_CONNECTION_STRING = "<AZURE STORAGE CONNECTION STRING>";
        private static final String STORAGE_CONTAINER_NAME = "<AZURE STORAGE CONTAINER NAME>";
    
        public static final Consumer<EventContext> PARTITION_PROCESSOR = eventContext -> {
        System.out.printf("Processing event from partition %s with sequence number %d with body: %s %n", 
                eventContext.getPartitionContext().getPartitionId(), eventContext.getEventData().getSequenceNumber(), eventContext.getEventData().getBodyAsString());

            if (eventContext.getEventData().getSequenceNumber() % 10 == 0) {
                eventContext.updateCheckpoint();
            }
        };
        
        public static final Consumer<ErrorContext> ERROR_HANDLER = errorContext -> {
            System.out.printf("Error occurred in partition processor for partition %s, %s.%n",
                errorContext.getPartitionContext().getPartitionId(),
                errorContext.getThrowable());
        };
        
        public static void main(String[] args) throws Exception {
            BlobContainerAsyncClient blobContainerAsyncClient = new BlobContainerClientBuilder()
                .connectionString(STORAGE_CONNECTION_STRING)
                .containerName(STORAGE_CONTAINER_NAME)
                .buildAsyncClient();
    
            EventProcessorClientBuilder eventProcessorClientBuilder = new EventProcessorClientBuilder()
                .connectionString(EH_NAMESPACE_CONNECTION_STRING, eventHubName)
                .consumerGroup(EventHubClientBuilder.DEFAULT_CONSUMER_GROUP_NAME)
                .processEvent(PARTITION_PROCESSOR)
                .processError(ERROR_HANDLER)
                .checkpointStore(new BlobCheckpointStore(blobContainerAsyncClient));
    
            EventProcessorClient eventProcessorClient = eventProcessorClientBuilder.buildEventProcessorClient();

            System.out.println("Starting event processor");
            eventProcessorClient.start();
        
            System.out.println("Press enter to stop.");
            System.in.read();
        
            System.out.println("Stopping event processor");
            eventProcessorClient.stop();
            System.out.println("Event processor stopped.");
        
            System.out.println("Exiting process");
        }
        
    }
    ```
3. Générez le programme et assurez-vous qu’il n’y a aucune erreur. 

## <a name="run-the-applications"></a>Exécution des applications
1. Exécutez d’abord l’application **réceptrice**.
1. Ensuite, exécutez l’application **émettrice**. 
1. Dans la fenêtre d’application **réceptrice**, vérifiez que vous voyez les événements qui ont été publiés par l’application émettrice.
1. Appuyez sur **Entrée** dans la fenêtre d’application réceptrice pour arrêter l’application. 

## <a name="next-steps"></a>Étapes suivantes
Consultez les exemples suivants sur GitHub :

- [Exemples azure-messaging-eventhubs](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs/src/samples/java/com/azure/messaging/eventhubs)
- [Exemples azure-messaging-eventhubs-checkpointstore-blob](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob)  
