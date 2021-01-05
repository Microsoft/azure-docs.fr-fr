---
title: 'Démarrage rapide : Bibliothèque de client Stockage File d’attente Azure v12 – .NET'
description: Découvrez comment utiliser la bibliothèque de client Stockage File d’attente Azure v12 pour .NET afin de créer une file d’attente et d’ajouter des messages à cette file. Vous apprendrez ensuite à lire et à supprimer des messages de la file d’attente. Vous apprendrez également à supprimer une file d’attente.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/24/2020
ms.topic: quickstart
ms.service: storage
ms.subservice: queues
ms.custom: devx-track-csharp
ms.openlocfilehash: 22038e4145acabc067083177fcf297464972ad58
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97589523"
---
# <a name="quickstart-azure-queue-storage-client-library-v12-for-net"></a>Démarrage rapide : Bibliothèque de client Stockage File d’attente Azure v12 pour .NET

Familiarisez-vous avec la bibliothèque de client Stockage File d’attente Azure version 12 pour .NET. Le Stockage File d’attente Azure est un service permettant de stocker un grand nombre de messages dans le but de les récupérer et de les traiter plus tard. Suivez les étapes suivantes pour installer le package et essayer un exemple de code pour les tâches de base.

Utilisez la bibliothèque de client Stockage File d’attente Azure v12 pour .NET afin d’effectuer les opérations suivantes :

- Créer une file d’attente
- Ajouter des messages à une file d’attente
- Afficher un aperçu des messages d’une file d’attente
- Mettre à jour un message dans une file d’attente
- Réception des messages d'une file d'attente
- Supprimer des messages d’une file d’attente
- Suppression d'une file d'attente

Ressources supplémentaires :

- [Documentation de référence de l’API](/dotnet/api/azure.storage.queues)
- [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Queues)
- [Package (NuGet)](https://www.nuget.org/packages/Azure.Storage.Queues/12.0.0)
- [Exemples](../common/storage-samples-dotnet.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json#queue-samples)

## <a name="prerequisites"></a>Prérequis

- Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/)
- Compte de stockage Azure : [créez un compte de stockage](../common/storage-account-create.md)
- Dernière version du [Kit SDK .NET Core](https://dotnet.microsoft.com/download/dotnet-core) pour votre système d’exploitation. Veillez à disposer du Kit de développement logiciel (SDK), et non du runtime.

## <a name="setting-up"></a>Configuration

Cette section vous guide tout au long de la préparation d’un projet à utiliser avec la bibliothèque de client Stockage File d’attente Azure v12 pour .NET.

### <a name="create-the-project"></a>Créer le projet

Créez une application .NET Core nommée `QueuesQuickstartV12`.

1. Dans une fenêtre de console (par exemple cmd, PowerShell ou Bash), utilisez la commande `dotnet new` pour créer une application console avec le nom `QueuesQuickstartV12`. Cette commande crée un projet C# « hello world » simple avec un seul fichier source nommé `Program.cs`.

   ```console
   dotnet new console -n QueuesQuickstartV12
   ```

1. Basculez vers le répertoire `QueuesQuickstartV12` créé.

   ```console
   cd QueuesQuickstartV12
   ```

### <a name="install-the-package"></a>Installer le package

Alors que vous êtes toujours dans le répertoire de l’application, installez le package de la bibliothèque de client Stockage File d’attente Azure pour .NET à l’aide de la commande `dotnet add package`.

```console
dotnet add package Azure.Storage.Queues
```

### <a name="set-up-the-app-framework"></a>Configurer le framework d’application

À partir du répertoire de projet :

1. Ouvrez le fichier `Program.cs` dans votre éditeur.
1. Supprimez l'instruction `Console.WriteLine("Hello, World");`.
1. Ajoutez des directives `using`.
1. Mettez à jour la déclaration de méthode `Main` pour [prendre en charge le code asynchrone](/dotnet/csharp/whats-new/csharp-7#async-main)

Voici le code :

```csharp
using Azure;
using Azure.Storage.Queues;
using Azure.Storage.Queues.Models;
using System;
using System.Threading.Tasks;

namespace QueuesQuickstartV12
{
    class Program
    {
        static async Task Main(string[] args)
        {
        }
    }
}
```

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Modèle objet

Stockage File d’attente Azure est un service permettant de stocker un grand nombre de messages. La taille maximale d’un message de file d’attente est de 64 Ko. Une file d’attente peut contenir des millions de messages, dans la limite de la capacité totale d’un compte de stockage. Les files d’attente sont couramment utilisées pour créer un backlog de travail à traiter de façon asynchrone. Le Stockage File d’attente offre trois types de ressources :

- Le compte de stockage
- Une file d’attente dans le compte de stockage
- Les messages dans la file d’attente

Le diagramme suivant montre la relation entre ces ressources.

![Diagramme de l’architecture du Stockage File d’attente](./media/storage-queues-introduction/queue1.png)

Utilisez les classes .NET suivantes pour interagir avec ces ressources :

- [`QueueServiceClient`](/dotnet/api/azure.storage.queues.queueserviceclient) : `QueueServiceClient` vous permet de gérer toutes les files d’attente de votre compte de stockage.
- [`QueueClient`](/dotnet/api/azure.storage.queues.queueclient) : la classe `QueueClient` vous permet de gérer et de manipuler une file d’attente individuelle et ses messages.
- [`QueueMessage`](/dotnet/api/azure.storage.queues.models.queuemessage) : La classe `QueueMessage` représente les objets individuels retournés lors de l’appel de [`ReceiveMessages`](/dotnet/api/azure.storage.queues.queueclient.receivemessages) dans une file d’attente.

## <a name="code-examples"></a>Exemples de code

Ces exemples d’extraits de code vous montrent comment effectuer les actions suivantes avec la bibliothèque de client Stockage File d’attente Azure pour .NET :

- [Obtenir la chaîne de connexion](#get-the-connection-string)
- [Créer une file d’attente](#create-a-queue)
- [Ajouter des messages à une file d’attente](#add-messages-to-a-queue)
- [Afficher un aperçu des messages d’une file d’attente](#peek-at-messages-in-a-queue)
- [Mettre à jour un message dans une file d’attente](#update-a-message-in-a-queue)
- [Recevoir les messages d’une file d’attente](#receive-messages-from-a-queue)
- [Supprimer des messages d’une file d’attente](#delete-messages-from-a-queue)
- [Supprimer une file d’attente](#delete-a-queue)

### <a name="get-the-connection-string"></a>Obtenir la chaîne de connexion

Le code suivant récupère la chaîne de connexion du compte de stockage. La chaîne de connexion est stockée dans la variable d’environnement créée dans la section [Configurer votre chaîne de connexion de stockage](#configure-your-storage-connection-string).

Ajoutez ce code dans la méthode `Main` :

```csharp
Console.WriteLine("Azure Queue Storage client library v12 - .NET quickstart sample\n");

// Retrieve the connection string for use with the application. The storage
// connection string is stored in an environment variable called
// AZURE_STORAGE_CONNECTION_STRING on the machine running the application.
// If the environment variable is created after the application is launched
// in a console or with Visual Studio, the shell or application needs to be
// closed and reloaded to take the environment variable into account.
string connectionString = Environment.GetEnvironmentVariable("AZURE_STORAGE_CONNECTION_STRING");
```

### <a name="create-a-queue"></a>Créer une file d’attente

Choisissez un nom pour la nouvelle file d’attente. Le code suivant ajoute une valeur GUID au nom de la file d’attente pour s’assurer qu’il est unique.

> [!IMPORTANT]
> Les noms de file d’attente peuvent contenir uniquement des lettres minuscules, des chiffres et des traits d’union, et doivent commencer par une lettre ou un nombre. Chaque trait d’union doit être précédé et suivi d’un caractère autre qu’un tiret. Le nom doit avoir entre 3 et 63 caractères. Pour plus d’informations, consultez [Affectation de noms pour les files d’attente et les métadonnées](/rest/api/storageservices/naming-queues-and-metadata).

Créez une instance de la classe [`QueueClient`](/dotnet/api/azure.storage.queues.queueclient). Ensuite, appelez la méthode [`CreateAsync`](/dotnet/api/azure.storage.queues.queueclient.createasync) pour créer la file d’attente dans votre compte de stockage.

Ajoutez ce code à la fin de la méthode `Main` :

```csharp
// Create a unique name for the queue
string queueName = "quickstartqueues-" + Guid.NewGuid().ToString();

Console.WriteLine($"Creating queue: {queueName}");

// Instantiate a QueueClient which will be
// used to create and manipulate the queue
QueueClient queueClient = new QueueClient(connectionString, queueName);

// Create the queue
await queueClient.CreateAsync();
```

### <a name="add-messages-to-a-queue"></a>Ajouter des messages à une file d’attente

L’extrait de code suivant ajoute de façon asynchrone des messages à la file d’attente en appelant la méthode [`SendMessageAsync`](/dotnet/api/azure.storage.queues.queueclient.sendmessageasync). Il enregistre également un [`SendReceipt`](/dotnet/api/azure.storage.queues.models.sendreceipt) retourné à partir d’un appel de `SendMessageAsync`. La réception est utilisée pour mettre à jour le message ultérieurement dans le programme.

Ajoutez ce code à la fin de la méthode `Main` :

```csharp
Console.WriteLine("\nAdding messages to the queue...");

// Send several messages to the queue
await queueClient.SendMessageAsync("First message");
await queueClient.SendMessageAsync("Second message");

// Save the receipt so we can update this message later
SendReceipt receipt = await queueClient.SendMessageAsync("Third message");
```

### <a name="peek-at-messages-in-a-queue"></a>Afficher un aperçu des messages d’une file d’attente

Affichez un aperçu des messages de la file d’attente en appelant la méthode [`PeekMessagesAsync`](/dotnet/api/azure.storage.queues.queueclient.peekmessagesasync). Cette méthode récupère un ou plusieurs messages du début de la file d’attente, mais ne modifie pas la visibilité du message.

Ajoutez ce code à la fin de la méthode `Main` :

```csharp
Console.WriteLine("\nPeek at the messages in the queue...");

// Peek at messages in the queue
PeekedMessage[] peekedMessages = await queueClient.PeekMessagesAsync(maxMessages: 10);

foreach (PeekedMessage peekedMessage in peekedMessages)
{
    // Display the message
    Console.WriteLine($"Message: {peekedMessage.MessageText}");
}
```

### <a name="update-a-message-in-a-queue"></a>Mettre à jour un message dans une file d’attente

Mettez à jour le contenu d’un message en appelant la méthode [`UpdateMessageAsync`](/dotnet/api/azure.storage.queues.queueclient.updatemessageasync). Cette méthode peut changer le contenu et le délai d’expiration de la visibilité d’un message. Le contenu du message doit être une chaîne encodée en UTF-8 d’une taille maximale de 64 Ko. Avec le nouveau contenu du message, transmettez les valeurs du `SendReceipt` qui a été enregistré dans le code. Les valeurs `SendReceipt` identifient le message à mettre à jour.

```csharp
Console.WriteLine("\nUpdating the third message in the queue...");

// Update a message using the saved receipt from sending the message
await queueClient.UpdateMessageAsync(receipt.MessageId, receipt.PopReceipt, "Third message has been updated");
```

### <a name="receive-messages-from-a-queue"></a>Réception des messages d'une file d'attente

Téléchargez les messages ajoutés en appelant la méthode [`ReceiveMessagesAsync`](/dotnet/api/azure.storage.queues.queueclient.receivemessagesasync).

Ajoutez ce code à la fin de la méthode `Main` :

```csharp
Console.WriteLine("\nReceiving messages from the queue...");

// Get messages from the queue
QueueMessage[] messages = await queueClient.ReceiveMessagesAsync(maxMessages: 10);
```

### <a name="delete-messages-from-a-queue"></a>Supprimer des messages d’une file d’attente

Supprimez les messages de la file d’attente une fois qu’ils ont été traités. Dans ce cas, le traitement affiche simplement le message sur la console.

Avant de traiter et de supprimer les messages, l’application s’interrompt dans l’attente d’une entrée de l’utilisateur en appelant `Console.ReadLine`. Vérifiez dans votre [portail Azure](https://portal.azure.com) que les ressources ont été créées correctement avant d’être supprimées. Tout message qui n’est pas supprimé explicitement sera de nouveau visible dans la file d’attente et pourra donc faire l’objet d’un nouveau traitement.

Ajoutez ce code à la fin de la méthode `Main` :

```csharp
Console.WriteLine("\nPress Enter key to 'process' messages and delete them from the queue...");
Console.ReadLine();

// Process and delete messages from the queue
foreach (QueueMessage message in messages)
{
    // "Process" the message
    Console.WriteLine($"Message: {message.MessageText}");

    // Let the service know we're finished with
    // the message and it can be safely deleted.
    await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
}
```

### <a name="delete-a-queue"></a>Suppression d'une file d'attente

Le code suivant nettoie les ressources créées par l’application en supprimant la file d’attente avec la méthode [`DeleteAsync`](/dotnet/api/azure.storage.queues.queueclient.deleteasync).

Ajoutez ce code à la fin de la méthode `Main` :

```csharp
Console.WriteLine("\nPress Enter key to delete the queue...");
Console.ReadLine();

// Clean up
Console.WriteLine($"Deleting queue: {queueClient.Name}");
await queueClient.DeleteAsync();

Console.WriteLine("Done");
```

## <a name="run-the-code"></a>Exécuter le code

Cette application crée trois messages et les ajoute à une file d’attente Azure. Le code liste les messages dans la file d’attente, puis les récupère et les supprime avant de supprimer la file d’attente.

Dans la fenêtre de votre console, accédez au répertoire de l’application, puis générez et exécutez l’application.

```console
dotnet build
```

```console
dotnet run
```

La sortie de l’application ressemble à l’exemple suivant :

```output
Azure Queue Storage client library v12 - .NET quickstart sample

Creating queue: quickstartqueues-5c72da2c-30cc-4f09-b05c-a95d9da52af2

Adding messages to the queue...

Peek at the messages in the queue...
Message: First message
Message: Second message
Message: Third message

Updating the third message in the queue...

Receiving messages from the queue...

Press Enter key to 'process' messages and delete them from the queue...

Message: First message
Message: Second message
Message: Third message has been updated

Press Enter key to delete the queue...

Deleting queue: quickstartqueues-5c72da2c-30cc-4f09-b05c-a95d9da52af2
Done
```

Quand l’application s’interrompt avant de recevoir des messages, vérifiez votre compte de stockage dans le [portail Azure](https://portal.azure.com). Vérifiez que les messages se trouvent dans la file d’attente.

Appuyez sur la touche `Enter` pour recevoir et supprimer les messages. Quand vous y êtes invité, réappuyez sur la touche `Enter` pour supprimer la file d’attente et terminer la démonstration.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer une file d’attente et à y ajouter des messages à l’aide de code .NET asynchrone. Ensuite, vous avez appris à afficher un aperçu des messages, à les récupérer et à les supprimer. Enfin, vous avez appris à supprimer une file d’attente de messages.

Pour obtenir des tutoriels, des exemples, des guides de démarrage rapide et autres documentations, visitez :

> [!div class="nextstepaction"]
> [Azure pour les développeurs .NET et .NET Core](/dotnet/azure/)

- Pour plus d’informations, consultez les [bibliothèques Stockage Azure pour .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage).
- Pour d’autres exemples d’applications Stockage File d’attente Azure, consultez [Exemples de bibliothèques de client Stockage File d’attente Azure pour .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Queues/samples).
- Pour en savoir plus sur .NET Core, consultez [Prise en main de .NET en 10 minutes](https://www.microsoft.com/net/learn/get-started/).
