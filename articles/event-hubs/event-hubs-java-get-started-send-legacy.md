---
title: Recevoir des événements d’Azure Event Hubs ou lui en envoyer en utilisant Java (hérité)
description: Cet article décrit la procédure à suivre pour créer une application Java qui reçoit des événements d’Azure Event Hubs et lui en envoie à l’aide de l’ancien package azure-eventhubs.
ms.topic: quickstart
ms.date: 06/23/2020
ms.custom: devx-track-java
ms.openlocfilehash: e86ca76f3eb661e1407a02b58e60b62b391f5702
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97607981"
---
# <a name="use-java-to-send-events-to-or-receive-events-from-azure-event-hubs-azure-eventhubs"></a>Utiliser Java pour recevoir des événements d’Azure Event Hubs ou lui en envoyer (azure-eventhubs)

Ce guide de démarrage rapide montre comment recevoir des événements d’un hub d’événements et lui en envoyer à l’aide du package Java **azure-eventhubs**.

> [!WARNING]
> Ce guide de démarrage rapide utilise les anciens packages **azure-eventhubs** et **azure-eventhubs-eph**. Pour un guide de démarrage rapide qui utilise la dernière version du package **azure-messaging-eventhubs**, consultez [Envoyer et recevoir des événements à l’aide d’azure-messaging-eventhubs](event-hubs-java-get-started-send.md). Pour migrer votre application de l’ancien package vers le nouveau, consultez [Guide to migrate from azure-eventhubs to azure-messaging-eventhubs](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/migration-guide.md) (Guide de migration d’azure-eventhubs vers azure-messaging-eventhubs). 

## <a name="prerequisites"></a>Prérequis

Si vous débutez avec Azure Event Hubs, consultez la [vue d’ensemble d’Event Hubs](event-hubs-about.md) avant de suivre ce guide de démarrage rapide. 

Pour effectuer ce démarrage rapide, vous avez besoin de ce qui suit :

- **Abonnement Microsoft Azure**. Pour utiliser les services Azure, y compris Azure Event Hubs, vous avez besoin d’un abonnement.  Si vous n’avez pas de compte Azure, vous pouvez vous inscrire à un [essai gratuit](https://azure.microsoft.com/free/) ou utiliser les avantages de votre abonnement MSDN quand vous [créez un compte](https://azure.microsoft.com).
- Un environnement de développement Java. Ce guide de démarrage rapide utilise [Eclipse](https://www.eclipse.org/).
- **Créez un espace de noms Event Hubs et un Event Hub**. La première étape consiste à utiliser le [portail Azure](https://portal.azure.com) pour créer un espace de noms de type Event Hubs et obtenir les informations de gestion nécessaires à votre application pour communiquer avec le concentrateur d’événements. Pour créer un espace de noms et un hub d’événements, suivez la procédure décrite dans [cet article](event-hubs-create.md). Ensuite, obtenez la valeur de la clé d’accès du hub d’événements en suivant les instructions de l’article : [Obtenir la chaîne de connexion](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Vous utilisez la clé d’accès dans le code que vous écrivez plus loin dans ce guide de démarrage rapide. Le nom de clé par défaut est : **RootManageSharedAccessKey**.

## <a name="send-events"></a>Envoyer des événements 
Cette section montre comment créer une application Java pour envoyer des événements à un hub d’événements. 

> [!NOTE]
> Vous pouvez télécharger ce guide de démarrage rapide sous forme d’exemple depuis [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/Java/Basic/SimpleSend), remplacer les chaînes `EventHubConnectionString` et `EventHubName` par les valeurs de votre hub d’événements, puis l’exécuter. Vous pouvez également suivre les étapes de ce guide de démarrage rapide pour créer le vôtre.

### <a name="add-reference-to-azure-event-hubs-library"></a>Ajouter une référence à la bibliothèque Azure Event Hubs

La bibliothèque cliente de Java pour les concentrateurs d’événements peut être utilisée dans les projets Maven à partir du [référentiel central Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs%22). Vous pouvez référencer cette bibliothèque à l’aide de la déclaration de dépendance suivante au sein de votre fichier de projet Maven :

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-eventhubs</artifactId>
    <version>2.2.0</version>
</dependency>
```

Pour différents types d’environnements de génération, vous pouvez obtenir explicitement les fichiers JAR les plus récents à partir du [référentiel central Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs%22).  

Pour un éditeur d’événements simple, importez le package *com.microsoft.azure.eventhubs* pour les classes clientes Event Hubs et le package *com.microsoft.azure.servicebus* pour les classes utilitaires, telles que les exceptions courantes qui sont partagées avec le client de messagerie Azure Service Bus. 

### <a name="write-code-to-send-messages-to-the-event-hub"></a>Écriture de code pour envoyer des messages à un hub d’événements

Pour l’exemple suivant, créez tout d’abord un nouveau projet Maven pour une application de console/shell dans votre environnement de développement Java favori. Ajoutez une classe nommée `SimpleSend`, et ajoutez-y le code suivant :

```java
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.microsoft.azure.eventhubs.ConnectionStringBuilder;
import com.microsoft.azure.eventhubs.EventData;
import com.microsoft.azure.eventhubs.EventHubClient;
import com.microsoft.azure.eventhubs.EventHubException;

import java.io.IOException;
import java.nio.charset.Charset;
import java.time.Instant;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;

public class SimpleSend {

    public static void main(String[] args)
            throws EventHubException, ExecutionException, InterruptedException, IOException {

    }
 }
```

### <a name="construct-connection-string"></a>Construire la chaîne de connexion

Utilisez la classe ConnectionStringBuilder pour construire une valeur de chaîne de connexion et passer à l’instance client Event Hubs. Remplacez les espaces réservés par les valeurs obtenues lors de la création de l’espace de noms et du concentrateur d’événements :

```java
        final ConnectionStringBuilder connStr = new ConnectionStringBuilder()
                .setNamespaceName("<EVENTHUB NAMESPACE") 
                .setEventHubName("EVENT HUB")
                .setSasKeyName("RootManageSharedAccessKey")
                .setSasKey("SHARED ACCESS KEY");
```

### <a name="write-code-to-send-events"></a>Écrire du code pour envoyer des événements

Créez un événement unique en transformant une chaîne dans son encodage UTF-8 octets. Ensuite, créez une instance cliente Event Hubs à partir de la chaîne de connexion et envoyez le message :   

```java 
        final Gson gson = new GsonBuilder().create();

        // The Executor handles all asynchronous tasks and this is passed to the EventHubClient instance.
        // This enables the user to segregate their thread pool based on the work load.
        // This pool can then be shared across multiple EventHubClient instances.
        // The following sample uses a single thread executor, as there is only one EventHubClient instance,
        // handling different flavors of ingestion to Event Hubs here.
        final ScheduledExecutorService executorService = Executors.newScheduledThreadPool(4);

        // Each EventHubClient instance spins up a new TCP/TLS connection, which is expensive.
        // It is always a best practice to reuse these instances. The following sample shows this.
        final EventHubClient ehClient = EventHubClient.createSync(connStr.toString(), executorService);

        try {
            for (int i = 0; i < 10; i++) {

                String payload = "Message " + Integer.toString(i);
                byte[] payloadBytes = gson.toJson(payload).getBytes(Charset.defaultCharset());
                EventData sendEvent = EventData.create(payloadBytes);

                // Send - not tied to any partition
                // Event Hubs service will round-robin the events across all Event Hubs partitions.
                // This is the recommended & most reliable way to send to Event Hubs.
                ehClient.sendSync(sendEvent);
            }

            System.out.println(Instant.now() + ": Send Complete...");
            System.out.println("Press Enter to stop.");
            System.in.read();
        } finally {
            ehClient.closeSync();
            executorService.shutdown();
        }

``` 

Générez et exécutez le programme, et assurez-vous qu’il n’y a aucune erreur.

Félicitations ! Vous venez d’envoyer des messages à un concentrateur d’événements.

### <a name="appendix-how-messages-are-routed-to-eventhub-partitions"></a>Annexe : Comment les messages sont acheminés vers les partitions EventHub

Avant que les messages ne soient récupérés par les consommateurs, ils doivent d’abord être publiés sur les partitions par les éditeurs. Lorsque les messages sont publiés sur Event Hub de façon synchrone à l’aide de la méthode sendSync() sur l’objet com.microsoft.azure.eventhubs.EventHubClient, le message peut être envoyé à une partition spécifique ou distribué à toutes les partitions disponibles par tourniquet (round robin) selon que la clé de partition est spécifiée ou non.

Lorsqu’une chaîne représentant la clé de partition est spécifiée, la clé sera hachée pour déterminer à quelle partition envoyer l’événement.

Lorsque la clé de partition n’est pas définie, les messages sont distribués par tourniquet (round robin) à toutes les partitions disponibles

```java
// Serialize the event into bytes
byte[] payloadBytes = gson.toJson(messagePayload).getBytes(Charset.defaultCharset());

// Use the bytes to construct an {@link EventData} object
EventData sendEvent = EventData.create(payloadBytes);

// Transmits the event to event hub without a partition key
// If a partition key is not set, then we will round-robin to all topic partitions
eventHubClient.sendSync(sendEvent);

//  the partitionKey will be hash'ed to determine the partitionId to send the eventData to.
eventHubClient.sendSync(sendEvent, partitionKey);

// close the client at the end of your program
eventHubClient.closeSync();

```

## <a name="receive-events"></a>Recevoir des événements
Le code de ce didacticiel est basé sur le [code EventProcessorSample sur GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/Java/Basic/EventProcessorSample), que vous pouvez analyser pour afficher la version complète de l’application en cours.

### <a name="receive-messages-with-eventprocessorhost-in-java"></a>Recevoir des messages avec EventProcessorHost en Java

**EventProcessorHost** est une classe Java qui simplifie la réception d’événements provenant d’Event Hubs grâce à la gestion des points de contrôle permanents et des réceptions en parallèle d’Event Hubs. EventProcessorHost permet de répartir des événements sur plusieurs récepteurs, même quand ils sont hébergés dans des nœuds différents. Cet exemple illustre l'utilisation de la classe EventProcessorHost pour un récepteur unique.

### <a name="create-a-storage-account"></a>Créez un compte de stockage.

Pour utiliser EventProcessorHost, vous devez disposer d’un [compte Stockage Azure][compte Stockage Azure] :

1. Connectez-vous au [Portail Azure](https://portal.azure.com), puis sélectionnez sur **Créer une ressource** sur la gauche de l’écran.
2. Sélectionnez **Stockage**, puis **Compte de stockage**. Dans la fenêtre **Créer un compte de stockage**, saisissez un nom pour votre compte de stockage. Renseignez les autres champs, sélectionnez la région souhaitée, puis sélectionnez **Créer**.

    ![Créer un compte de stockage dans le portail Azure](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-azure-storage-account.png)

3. Sélectionnez le compte de stockage que vous venez de créer, puis sélectionnez **Clés d'accès** :

    ![Accéder à vos clés d’accès dans le portail Azure](./media/event-hubs-dotnet-framework-getstarted-receive-eph/select-azure-storage-access-keys.png)

    Copiez la valeur key1 dans un emplacement temporaire. Vous l’utiliserez ultérieurement dans ce tutoriel.

### <a name="create-a-java-project-using-the-eventprocessor-host"></a>Créer un projet Java à l’aide de l’hôte EventProcessor

La bibliothèque cliente Java pour Event Hubs est utilisable dans les projets Maven à partir du [référentiel central Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs-eph%22). Elle peut être référencée à l’aide de la déclaration de dépendance ci-après dans votre fichier projet Maven : 

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-eventhubs</artifactId>
    <version>2.2.0</version>
</dependency>
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-eventhubs-eph</artifactId>
    <version>2.4.0</version>
</dependency>
```

Pour différents types d’environnements de génération, vous pouvez obtenir explicitement les fichiers JAR les plus récents à partir du [référentiel central Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs-eph%22).

1. Pour l’exemple suivant, créez tout d’abord un nouveau projet Maven pour une application de console/shell dans votre environnement de développement Java favori. La classe est appelée `ErrorNotificationHandler`.     

    ```java
    import java.util.function.Consumer;
    import com.microsoft.azure.eventprocessorhost.ExceptionReceivedEventArgs;

    public class ErrorNotificationHandler implements Consumer<ExceptionReceivedEventArgs>
    {
        @Override
        public void accept(ExceptionReceivedEventArgs t)
        {
            System.out.println("SAMPLE: Host " + t.getHostname() + " received general error notification during " + t.getAction() + ": " + t.getException().toString());
        }
    }
    ```
2. Utilisez le code suivant pour créer une classe appelée `EventProcessorSample`. Remplacez les espaces réservés par les valeurs utilisées lorsque vous avez créé le concentrateur d’événements et le compte de stockage :

   ```java
   package com.microsoft.azure.eventhubs.samples.eventprocessorsample;

   import com.microsoft.azure.eventhubs.ConnectionStringBuilder;
   import com.microsoft.azure.eventhubs.EventData;
   import com.microsoft.azure.eventprocessorhost.CloseReason;
   import com.microsoft.azure.eventprocessorhost.EventProcessorHost;
   import com.microsoft.azure.eventprocessorhost.EventProcessorOptions;
   import com.microsoft.azure.eventprocessorhost.ExceptionReceivedEventArgs;
   import com.microsoft.azure.eventprocessorhost.IEventProcessor;
   import com.microsoft.azure.eventprocessorhost.PartitionContext;

   import java.util.concurrent.ExecutionException;
   import java.util.function.Consumer;

   public class EventProcessorSample
   {
       public static void main(String args[]) throws InterruptedException, ExecutionException
       {
           String consumerGroupName = "$Default";
           String namespaceName = "----NamespaceName----";
           String eventHubName = "----EventHubName----";
           String sasKeyName = "----SharedAccessSignatureKeyName----";
           String sasKey = "----SharedAccessSignatureKey----";
           String storageConnectionString = "----AzureStorageConnectionString----";
           String storageContainerName = "----StorageContainerName----";
           String hostNamePrefix = "----HostNamePrefix----";

           ConnectionStringBuilder eventHubConnectionString = new ConnectionStringBuilder()
                .setNamespaceName(namespaceName)
                .setEventHubName(eventHubName)
                .setSasKeyName(sasKeyName)
                .setSasKey(sasKey);

           EventProcessorHost host = new EventProcessorHost(
                EventProcessorHost.createHostName(hostNamePrefix),
                eventHubName,
                consumerGroupName,
                eventHubConnectionString.toString(),
                storageConnectionString,
                storageContainerName);

           System.out.println("Registering host named " + host.getHostName());
           EventProcessorOptions options = new EventProcessorOptions();
           options.setExceptionNotification(new ErrorNotificationHandler());

           host.registerEventProcessor(EventProcessor.class, options)
           .whenComplete((unused, e) ->
           {
               if (e != null)
               {
                   System.out.println("Failure while registering: " + e.toString());
                   if (e.getCause() != null)
                   {
                       System.out.println("Inner exception: " + e.getCause().toString());
                   }
               }
           })
           .thenAccept((unused) ->
           {
               System.out.println("Press enter to stop.");
                 try 
               {
                   System.in.read();
               }
               catch (Exception e)
               {
                   System.out.println("Keyboard read failed: " + e.toString());
               }
           })
           .thenCompose((unused) ->
            {
                return host.unregisterEventProcessor();
           })
           .exceptionally((e) ->
           {
               System.out.println("Failure while unregistering: " + e.toString());
               if (e.getCause() != null)
               {
                   System.out.println("Inner exception: " + e.getCause().toString());
               }
               return null;
           })
           .get(); // Wait for everything to finish before exiting main!

           System.out.println("End of sample");
       }
   }
   ```
3. Créez une autre classe appelée `EventProcessor` à l’aide du code suivant :

    ```java
    public static class EventProcessor implements IEventProcessor
    {
        private int checkpointBatchingCount = 0;

        // OnOpen is called when a new event processor instance is created by the host. 
        @Override
        public void onOpen(PartitionContext context) throws Exception
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " is opening");
        }

        // OnClose is called when an event processor instance is being shut down. 
        @Override
        public void onClose(PartitionContext context, CloseReason reason) throws Exception
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " is closing for reason " + reason.toString());
        }

        // onError is called when an error occurs in EventProcessorHost code that is tied to this partition, such as a receiver failure.
        @Override
        public void onError(PartitionContext context, Throwable error)
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " onError: " + error.toString());
        }

        // onEvents is called when events are received on this partition of the Event Hub. 
        @Override
        public void onEvents(PartitionContext context, Iterable<EventData> events) throws Exception
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " got event batch");
            int eventCount = 0;
            for (EventData data : events)
            {
                try
                {
                    System.out.println("SAMPLE (" + context.getPartitionId() + "," + data.getSystemProperties().getOffset() + "," +
                            data.getSystemProperties().getSequenceNumber() + "): " + new String(data.getBytes(), "UTF8"));
                    eventCount++;

                    // Checkpointing persists the current position in the event stream for this partition and means that the next
                    // time any host opens an event processor on this event hub+consumer group+partition combination, it will start
                    // receiving at the event after this one. 
                    this.checkpointBatchingCount++;
                    if ((checkpointBatchingCount % 5) == 0)
                    {
                        System.out.println("SAMPLE: Partition " + context.getPartitionId() + " checkpointing at " +
                               data.getSystemProperties().getOffset() + "," + data.getSystemProperties().getSequenceNumber());
                        // Checkpoints are created asynchronously. It is important to wait for the result of checkpointing
                        // before exiting onEvents or before creating the next checkpoint, to detect errors and to ensure proper ordering.
                        context.checkpoint(data).get();
                    }
                }
                catch (Exception e)
                {
                    System.out.println("Processing failed for an event: " + e.toString());
                }
            }
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " batch size was " + eventCount + " for host " + context.getOwner());
        }
    }
    ```

Ce didacticiel utilise une seule instance de EventProcessorHost. Pour augmenter le débit, nous recommandons d’exécuter plusieurs instances d’EventProcessorHost, de préférence sur des machines séparées.  Cela offre également davantage de redondance. Dans ces cas, les différentes instances se coordonnent automatiquement entre elles afin d'équilibrer la charge des événements reçus. Si vous souhaitez que plusieurs récepteurs traitent *tous* les événements, vous devez utiliser le concept **ConsumerGroup** . Lors de la réception des événements à partir de différents ordinateurs, il peut être utile de spécifier des noms pour les instances de EventProcessorHost basées sur les ordinateurs (ou rôles) dans lesquels ils sont déployés.

### <a name="publishing-messages-to-eventhub"></a>Publication de messages sur EventHub

Avant que les messages ne soient récupérés par les consommateurs, ils doivent d’abord être publiés sur les partitions par les éditeurs. Il est important de noter que lorsque les messages sont publiés sur Event Hub de façon synchrone à l’aide de la méthode sendSync() sur l’objet com.microsoft.azure.eventhubs.EventHubClient, le message peut être envoyé à une partition spécifique ou distribué à toutes les partitions disponibles par tourniquet (round robin) selon que la clé de partition est spécifiée ou non.

Lorsqu’une chaîne représentant la clé de partition est spécifiée, la clé est hachée pour déterminer à quelle partition envoyer l’événement.

Lorsque la clé de partition n’est pas définie, les messages sont distribués par tourniquet (round robin) à toutes les partitions disponibles

```java
// Serialize the event into bytes
byte[] payloadBytes = gson.toJson(messagePayload).getBytes(Charset.defaultCharset());

// Use the bytes to construct an {@link EventData} object
EventData sendEvent = EventData.create(payloadBytes);

// Transmits the event to event hub without a partition key
// If a partition key is not set, then we will round-robin to all topic partitions
eventHubClient.sendSync(sendEvent);

//  the partitionKey will be hash'ed to determine the partitionId to send the eventData to.
eventHubClient.sendSync(sendEvent, partitionKey);

```

### <a name="implementing-a-custom-checkpointmanager-for-eventprocessorhost-eph"></a>Mise en œuvre d’un CheckpointManager personnalisé pour EventProcessorHost (EPH)

L’API fournit un mécanisme pour implémenter votre gestionnaire de point de contrôle personnalisé pour les scénarios où la mise en œuvre par défaut n’est pas compatible avec votre cas d’utilisation.

Le gestionnaire de point de contrôle par défaut utilise le stockage blob, mais si vous remplacez le gestionnaire de point de contrôle utilisé par EPH par votre propre mise en œuvre, vous pouvez utiliser n’importe quel magasin souhaité pour sauvegarder votre mise en œuvre du gestionnaire de point de contrôle.

Créer une classe qui met en œuvre l’interface com.microsoft.azure.eventprocessorhost.ICheckpointManager

Utiliser votre mise en œuvre personnalisée du gestionnaire de point de contrôle (com.microsoft.azure.eventprocessorhost.ICheckpointManager)

Dans votre implémentation, vous pouvez remplacer le mécanisme de points de contrôle par défaut et implémenter nos propres points de contrôle en fonction de votre propre magasin de données (comme SQL Server, CosmosDB et Cache Azure pour Redis). Nous recommandons que le magasin utilisé pour sauvegarder votre mise en œuvre du gestionnaire de point de contrôle soit accessible à toutes les instances EPH qui traitent des événements pour le groupe de consommateurs.

Vous pouvez utiliser n’importe quel magasin de données disponible dans votre environnement.

La classe com.microsoft.azure.eventprocessorhost.EventProcessorHost vous fournit deux constructeurs qui vous permettent de remplacer le gestionnaire de point de contrôle pour votre EventProcessorHost.

## <a name="next-steps"></a>Étapes suivantes
Consultez les articles suivants : 

- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Fonctionnalités et terminologie dans Azure Event Hubs](event-hubs-features.md)
- [FAQ sur les hubs d’événements](event-hubs-faq.md)
