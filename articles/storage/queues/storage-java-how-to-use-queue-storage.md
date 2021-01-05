---
title: Utilisation du stockage de files d’attente à partir de Java - Stockage Azure
description: Apprenez à utiliser le stockage File d'attente pour créer et supprimer des files d'attente. Apprenez à insérer, consulter, obtenir et supprimer des messages à l'aide de la bibliothèque de client Stockage Azure pour Java.
author: mhopkins-msft
ms.custom: devx-track-java
ms.author: mhopkins
ms.date: 08/19/2020
ms.service: storage
ms.subservice: queues
ms.topic: how-to
ms.reviewer: dineshm
ms.openlocfilehash: c2ee32b3ced8fdcd5f9f889c4fd0183e46ad5d8d
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93346007"
---
# <a name="how-to-use-queue-storage-from-java"></a>Utilisation du stockage de files d'attente à partir de Java

[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

## <a name="overview"></a>Vue d’ensemble

Ce guide explique comment coder pour des scénarios courants à l'aide du service de stockage File d'attente Azure. Les exemples sont écrits en Java et utilisent le [Kit de développement logiciel (SDK) Azure Storage pour Java][Azure Storage SDK for Java]. Les scénarios comprennent l' **insertion** , la **consultation** , l' **obtention** et la **suppression** de messages de la file d'attente. Le code de **création** et de **suppression** de files d'attente est également abordé. Pour plus d’informations sur les files d’attente, consultez la section [Étapes suivantes](#next-steps).

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-java-application"></a>Création d’une application Java

# <a name="java-v12"></a>[Java v12](#tab/java)

Commencez par vérifier que votre système de développement répond aux conditions préalables répertoriées dans [Bibliothèque de client du stockage File d'attente Azure pour Java v12](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-queue).

Pour créer une application Java nommée *queues-how-to-v12*  :

1. Dans une fenêtre de console (par exemple cmd, PowerShell ou Bash), utilisez Maven pour créer une application de console nommée *queues-how-to-v12*. Tapez la commande **mvn** suivante pour créer un projet Java « Hello World ! ».

   ```bash
    mvn archetype:generate \
        --define interactiveMode=n \
        --define groupId=com.queues.howto \
        --define artifactId=queues-howto-v12 \
        --define archetypeArtifactId=maven-archetype-quickstart \
        --define archetypeVersion=1.4
   ```

   ```powershell
    mvn archetype:generate `
        --define interactiveMode=n `
        --define groupId=com.queues.howto `
        --define artifactId=queues-howto-v12 `
        --define archetypeArtifactId=maven-archetype-quickstart `
        --define archetypeVersion=1.4
    ```

1. Le résultat de la génération du projet doit ressembler à ceci :

    ```console
    [INFO] Scanning for projects...
    [INFO]
    [INFO] ------------------< org.apache.maven:standalone-pom >-------------------
    [INFO] Building Maven Stub Project (No POM) 1
    [INFO] --------------------------------[ pom ]---------------------------------
    [INFO]
    [INFO] >>> maven-archetype-plugin:3.1.2:generate (default-cli) > generate-sources @ standalone-pom >>>
    [INFO]
    [INFO] <<< maven-archetype-plugin:3.1.2:generate (default-cli) < generate-sources @ standalone-pom <<<
    [INFO]
    [INFO]
    [INFO] --- maven-archetype-plugin:3.1.2:generate (default-cli) @ standalone-pom ---
    [INFO] Generating project in Batch mode
    [INFO] ----------------------------------------------------------------------------
    [INFO] Using following parameters for creating project from Archetype: maven-archetype-quickstart:1.4
    [INFO] ----------------------------------------------------------------------------
    [INFO] Parameter: groupId, Value: com.queues.howto
    [INFO] Parameter: artifactId, Value: queues-howto-v12
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.queues.howto
    [INFO] Parameter: packageInPathFormat, Value: com/queues/howto
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.queues.howto
    [INFO] Parameter: groupId, Value: com.queues.howto
    [INFO] Parameter: artifactId, Value: queues-howto-v12
    [INFO] Project created from Archetype in dir: C:\queues\queues-howto-v12
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  6.775 s
    [INFO] Finished at: 2020-08-17T15:27:31-07:00
    [INFO] ------------------------------------------------------------------------
        ```

1. Switch to the newly created *queues-howto-v12* directory.

   ```console
   cd queues-howto-v12
   ```

### <a name="install-the-package"></a>Installer le package

Ouvrez le fichier *pom.xml* dans votre éditeur de texte. Ajoutez l’élément dépendance suivant au groupe de dépendances.

```xml
<dependency>
  <groupId>com.azure</groupId>
  <artifactId>azure-storage-queue</artifactId>
  <version>12.6.0</version>
</dependency>
```

# <a name="java-v8"></a>[Java v8](#tab/java8)

Commencez par vérifier que votre système de développement répond aux conditions préalables répertoriées dans [SDK Stockage Azure pour Java v8](https://github.com/azure/azure-storage-java). Suivez les instructions pour télécharger et installer les bibliothèques Stockage Azure pour Java. Vous pouvez ensuite créer une application Java à l'aide des exemples de cet article.

---

## <a name="configure-your-application-to-access-queue-storage"></a>Configuration de votre application pour accéder au stockage de files d'attente

Ajoutez les instructions import suivantes au début du fichier Java dans lequel vous voulez utiliser des API de stockage Azure pour accéder aux files d'attente :

# <a name="java-v12"></a>[Java v12](#tab/java)

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_ImportStatements":::

# <a name="java-v8"></a>[Java v8](#tab/java8)

```java
// Include the following imports to use queue APIs.
import com.microsoft.azure.storage.*;
import com.microsoft.azure.storage.queue.*;
```

---

## <a name="set-up-an-azure-storage-connection-string"></a>Configuration d’une chaîne de connexion au stockage Azure

Un client de stockage Azure utilise une chaîne de connexion de stockage pour accéder aux services de gestion des données. Procurez-vous le nom et la clé d'accès principale de votre compte de stockage sur le [portail Azure](https://portal.azure.com). Utilisez-les comme valeurs *AccountName* et *AccountKey* dans la chaîne de connexion. Cet exemple vous montre comment déclarer un champ statique pour qu’il contienne une chaîne de connexion :

# <a name="java-v12"></a>[Java v12](#tab/java)

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_ConnectionString":::

# <a name="java-v8"></a>[Java v8](#tab/java8)

```java
// Define the connection-string with your values.
final String storageConnectionString =
    "DefaultEndpointsProtocol=https;" +
    "AccountName=your_storage_account;" +
    "AccountKey=your_storage_account_key";
```

Vous pouvez stocker cette chaîne dans le fichier de configuration du service appelé *ServiceConfiguration.cscfg*. Pour une application exécutée dans un rôle Microsoft Azure, accédez à la chaîne de connexion en appelant **RoleEnvironment.getConfigurationSettings**. Voici un exemple de code permettant d'extraire la chaîne de connexion à partir d'un élément **Setting** nommé *StorageConnectionString*  :

```java
// Retrieve storage account from connection-string.
String storageConnectionString =
    RoleEnvironment.getConfigurationSettings().get("StorageConnectionString");
```

---

Les exemples ci-dessous partent du principe que vous avez utilisé un objet **String** contenant la chaîne de connexion de stockage.

## <a name="how-to-create-a-queue"></a>Procédure : Créer une file d’attente

# <a name="java-v12"></a>[Java v12](#tab/java)

Un objet **QueueClient** contient les opérations permettant d'interagir avec une file d'attente. Le code suivant crée un objet **QueueClient**. Utilisez l'objet **QueueClient** pour créer la file d'attente que vous souhaitez utiliser.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_CreateQueue":::

# <a name="java-v8"></a>[Java v8](#tab/java8)

Un objet **CloudQueueClient** vous permet d'obtenir les objets de référence pour les files d'attente. Le code suivant crée un objet **CloudQueueClient** . (Remarque : D’autres méthodes permettent de créer des objets **CloudStorageAccount**. Pour plus d’informations, consultez la section **CloudStorageAccount** dans la page [Référence du Kit de développement logiciel (SDK) du client Azure Storage].

Utilisez l'objet **CloudQueueClient** pour obtenir une référence pointant vers la file d'attente à utiliser. Si la file d'attente n'existe pas, vous pouvez la créer :

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
       CloudStorageAccount.parse(storageConnectionString);

   // Create the queue client.
   CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

   // Retrieve a reference to a queue.
   CloudQueue queue = queueClient.getQueueReference("myqueue");

   // Create the queue if it doesn't already exist.
   queue.createIfNotExists();
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-add-a-message-to-a-queue"></a>Procédure : Ajout d'un message à une file d'attente

# <a name="java-v12"></a>[Java v12](#tab/java)

Pour insérer un message dans une file d'attente existante, appelez la méthode **sendMessage**. Un message peut être une chaîne (au format UTF-8) ou un tableau d'octets. Le code suivant envoie un message de type chaîne dans la file d'attente.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_AddMessage":::

# <a name="java-v8"></a>[Java v8](#tab/java8)

Pour insérer un message dans une file d'attente existante, commencez par créer un **CloudQueueMessage**. Appelez ensuite la méthode **addMessage** . Un **CloudQueueMessage** peut être créé à partir d'une chaîne (au format UTF-8) ou d'un tableau d'octets. Voici le code qui crée une file d'attente (si elle n'existe pas) et insère le message « Hello, World ».

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
       CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // Create the queue if it doesn't already exist.
    queue.createIfNotExists();

    // Create a message and add it to the queue.
    CloudQueueMessage message = new CloudQueueMessage("Hello, World");
    queue.addMessage(message);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-peek-at-the-next-message"></a>Procédure : Lecture furtive du message suivant

Vous pouvez lire furtivement le message au début de la file d'attente sans l'enlever de la file d'attente en appelant **peekMessage**.

# <a name="java-v12"></a>[Java v12](#tab/java)

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_PeekMessage":::

# <a name="java-v8"></a>[Java v8](#tab/java8)

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
       CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // Peek at the next message.
    CloudQueueMessage peekedMessage = queue.peekMessage();

    // Output the message value.
    if (peekedMessage != null)
    {
      System.out.println(peekedMessage.getMessageContentAsString());
   }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-change-the-contents-of-a-queued-message"></a>Procédure : Modification du contenu d'un message en file d'attente

Vous pouvez modifier le contenu d'un message placé dans la file d'attente. Si le message représente une tâche, vous pouvez utiliser cette fonctionnalité pour mettre à jour l'état. Le code suivant met à jour un message de la file d'attente avec un nouveau contenu et ajoute 30 secondes au délai d'expiration de la visibilité. Grâce à la prolongation du délai d'expiration de la visibilité, le client dispose de 30 secondes supplémentaires pour continuer à travailler sur le message. Vous pouvez également définir un nombre de nouvelles tentatives. Au bout de *n* nouvelles tentatives, le message sera supprimé. Ce scénario protège du déclenchement d'une erreur d'application par un message chaque fois qu'il est traité.

# <a name="java-v12"></a>[Java v12](#tab/java)

L'exemple de code suivant effectue une recherche dans la file d'attente des messages, recherche le premier message dont le contenu correspond à une chaîne de recherche, modifie le contenu du message, puis se ferme.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_UpdateSearchMessage":::

# <a name="java-v8"></a>[Java v8](#tab/java8)

L'exemple de code suivant effectue une recherche dans la file d'attente des messages, recherche le premier message dont le contenu correspond à « Hello, World », modifie le contenu du message, puis se ferme.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // The maximum number of messages that can be retrieved is 32.
    final int MAX_NUMBER_OF_MESSAGES_TO_PEEK = 32;

    // Loop through the messages in the queue.
    for (CloudQueueMessage message : queue.retrieveMessages(MAX_NUMBER_OF_MESSAGES_TO_PEEK,1,null,null))
    {
        // Check for a specific string.
        if (message.getMessageContentAsString().equals("Hello, World"))
        {
            // Modify the content of the first matching message.
            message.setMessageContent("Updated contents.");
            // Set it to be visible in 30 seconds.
            EnumSet<MessageUpdateFields> updateFields =
                EnumSet.of(MessageUpdateFields.CONTENT,
                MessageUpdateFields.VISIBILITY);
            // Update the message.
            queue.updateMessage(message, 30, updateFields, null, null);
            break;
        }
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

L'exemple de code suivant met simplement à jour le premier message visible dans la file d'attente.

# <a name="java-v12"></a>[Java v12](#tab/java)

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_UpdateFirstMessage":::

# <a name="java-v8"></a>[Java v8](#tab/java8)

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
       CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // Retrieve the first visible message in the queue.
    CloudQueueMessage message = queue.retrieveMessage();

    if (message != null)
    {
        // Modify the message content.
        message.setMessageContent("Updated contents.");
        // Set it to be visible in 60 seconds.
        EnumSet<MessageUpdateFields> updateFields =
            EnumSet.of(MessageUpdateFields.CONTENT,
            MessageUpdateFields.VISIBILITY);
        // Update the message.
        queue.updateMessage(message, 60, updateFields, null, null);
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-get-the-queue-length"></a>Procédure : Obtention de la longueur de la file d'attente

Vous pouvez obtenir une estimation du nombre de messages dans une file d'attente.

# <a name="java-v12"></a>[Java v12](#tab/java)

La méthode **getProperties** demande plusieurs valeurs actuelles au service de File d'attente. L'une de ces valeurs correspond au nombre de messages figurant dans une file d'attente. Ce nombre est approximatif étant donné que des messages peuvent être ajoutés ou supprimés après votre demande. La méthode **getApproximateMessageCount** renvoie la dernière valeur extraite par l'appel à **getProperties** , sans appeler le service de File d'attente.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_GetQueueLength":::

# <a name="java-v8"></a>[Java v8](#tab/java8)

La méthode **downloadAttributes** demande plusieurs valeurs actuelles au service de File d'attente. L'une de ces valeurs correspond au nombre de messages figurant dans une file d'attente. Ce nombre est approximatif étant donné que des messages peuvent être ajoutés ou supprimés après votre demande. La méthode **getApproximateMessageCount** renvoie la dernière valeur extraite par l’appel à **downloadAttributes** , sans appeler le service de File d’attente.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
       CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

   // Download the approximate message count from the server.
    queue.downloadAttributes();

    // Retrieve the newly cached approximate message count.
    long cachedMessageCount = queue.getApproximateMessageCount();

    // Display the queue length.
    System.out.println(String.format("Queue length: %d", cachedMessageCount));
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-dequeue-the-next-message"></a>Procédure : Suppression du message suivant de la file d’attente

# <a name="java-v12"></a>[Java v12](#tab/java)

Votre code enlève un message d'une file d'attente en deux étapes. Lorsque vous appelez **receiveMessage** , vous obtenez le message suivant d'une file d'attente. Un message renvoyé par **receiveMessage** devient invisible de tout autre code lisant les messages de cette file d'attente. Par défaut, ce message reste invisible pendant 30 secondes. Pour finaliser la suppression du message de la file d'attente, vous devez aussi appeler **deleteMessage**. Si votre code ne parvient pas à traiter un message, ce processus en deux étapes vous permet de recevoir le même message et de réessayer. Votre code appelle **deleteMessage** juste après le traitement du message.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_DequeueMessage":::

# <a name="java-v8"></a>[Java v8](#tab/java8)

Votre code enlève un message d'une file d'attente en deux étapes. Lorsque vous appelez **retrieveMessage** , vous obtenez le message suivant dans une file d'attente. Un message renvoyé par **retrieveMessage** devient invisible de tout autre code lisant les messages de cette file d'attente. Par défaut, ce message reste invisible pendant 30 secondes. Pour finaliser la suppression du message de la file d'attente, vous devez aussi appeler **deleteMessage**. Si votre code ne parvient pas à traiter un message, ce processus en deux étapes vous permet de recevoir le même message et de réessayer. Votre code appelle **deleteMessage** juste après le traitement du message.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // Retrieve the first visible message in the queue.
    CloudQueueMessage retrievedMessage = queue.retrieveMessage();

    if (retrievedMessage != null)
    {
        // Process the message in less than 30 seconds, and then delete the message.
        queue.deleteMessage(retrievedMessage);
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="additional-options-for-dequeuing-messages"></a>Options supplémentaires pour la suppression des messages dans la file d'attente

Il existe deux façons de personnaliser l'extraction des messages à partir d'une file d'attente. Premièrement, procurez-vous un lot de messages (jusqu'à 32). Deuxièmement, définissez un délai d'expiration de l'invisibilité plus long ou plus court afin d'accorder à votre code plus ou moins de temps pour traiter complètement chaque message.

# <a name="java-v12"></a>[Java v12](#tab/java)

L'exemple de code suivant utilise la méthode **receiveMessages** pour obtenir 20 messages en un appel. Ensuite, il traite chaque message à l'aide d'une boucle **for** . Il définit également le délai d'expiration de l'invisibilité sur cinq minutes (300 secondes) pour chaque message. Le délai d'expiration démarre pour tous les messages en même temps. Lorsque cinq minutes se sont écoulées depuis l'appel à **receiveMessages** , tous les messages non supprimés redeviennent visibles.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_DequeueMessages":::

# <a name="java-v8"></a>[Java v8](#tab/java8)

L'exemple de code suivant utilise la méthode **retrieveMessages** pour obtenir 20 messages en un appel. Ensuite, il traite chaque message à l'aide d'une boucle **for** . Il définit également le délai d'expiration de l'invisibilité sur cinq minutes (300 secondes) pour chaque message. Le délai d'expiration démarre pour tous les messages en même temps. Lorsque cinq minutes se sont écoulées depuis l'appel à **retrieveMessages** , tous les messages non supprimés redeviennent visibles.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // Retrieve 20 messages from the queue with a visibility timeout of 300 seconds.
    for (CloudQueueMessage message : queue.retrieveMessages(20, 300, null, null)) {
        // Do processing for all messages in less than 5 minutes,
        // deleting each message after processing.
        queue.deleteMessage(message);
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-list-the-queues"></a>Procédure : Répertorier les files d’attente

# <a name="java-v12"></a>[Java v12](#tab/java)

Pour obtenir la liste des files d'attente en cours, appelez la méthode **QueueServiceClient.listQueues()** , qui renvoie une collection d'objets **QueueItem**.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_ListQueues":::

# <a name="java-v8"></a>[Java v8](#tab/java8)

Pour obtenir la liste des files d’attente en cours, appelez la méthode **CloudQueueClient.listQueues()** , qui renvoie une collection d’objets **CloudQueue**.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient =
        storageAccount.createCloudQueueClient();

    // Loop through the collection of queues.
    for (CloudQueue queue : queueClient.listQueues())
    {
        // Output each queue name.
        System.out.println(queue.getName());
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-delete-a-queue"></a>Procédure : Suppression d'une file d'attente

# <a name="java-v12"></a>[Java v12](#tab/java)

Pour supprimer une file d'attente et tous les messages qu'elle contient, appelez la méthode **delete** sur l'objet **QueueClient**.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_DeleteMessageQueue":::

# <a name="java-v8"></a>[Java v8](#tab/java8)

Pour supprimer une file d’attente et tous les messages qu’elle contient, appelez la méthode **deleteIfExists** sur l’objet **CloudQueue**.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // Delete the queue if it exists.
    queue.deleteIfExists();
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

[!INCLUDE [storage-check-out-samples-java](../../../includes/storage-check-out-samples-java.md)]

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous connaissez les bases du stockage des files d'attente, consultez les liens suivants pour apprendre à exécuter les tâches de stockage plus complexes.

- [Kit de développement logiciel (SDK) Azure Storage pour Java][Azure Storage SDK for Java]
- [Référence du Kit de développement logiciel (SDK) du client Azure Storage][Azure Storage Client SDK Reference]
- [API REST des services d’Azure Storage][Azure Storage Services REST API]
- [Blog de l'équipe Azure Storage][Azure Storage Team Blog]

[Azure SDK for Java]: https://github.com/azure/azure-sdk-for-java
[Azure Storage SDK for Java]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage
[Référence du Kit de développement logiciel (SDK) du client Azure Storage]: https://azure.github.io/azure-sdk-for-java/storage.html
[Azure Storage Services REST API]: /rest/api/storageservices/
[Azure Storage Team Blog]: https://techcommunity.microsoft.com/t5/azure-storage/bg-p/AzureStorageBlog
