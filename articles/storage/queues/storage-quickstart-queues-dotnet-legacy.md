---
title: 'Démarrage rapide : Utiliser Stockage Azure v11 pour .NET afin de gérer une file d’attente'
description: Ce guide de démarrage rapide explique comment utiliser la bibliothèque de client Stockage Azure pour .NET afin de créer une file d’attente et d’y ajouter des messages. Il explique ensuite comment lire et traiter des messages de la file d’attente.
author: mhopkins-msft
ms.author: mhopkins
ms.reviewer: dineshm
ms.date: 07/24/2020
ms.topic: quickstart
ms.service: storage
ms.subservice: queues
ms.custom: devx-track-csharp
ms.openlocfilehash: 23703dc507aa909aea4711289a4d7d5c5e6a170e
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97588197"
---
# <a name="quickstart-use-the-azure-storage-sdk-v11-for-net-to-manage-a-queue"></a>Démarrage rapide : Utiliser le SDK Stockage Azure v11 pour .NET afin de gérer une file d’attente

Ce guide de démarrage rapide explique comment utiliser la bibliothèque de client Stockage Azure v11 pour .NET afin de créer une file d’attente et d’y ajouter des messages. Il explique ensuite comment lire et traiter des messages de la file d’attente.

> [!NOTE]
> Ce guide de démarrage rapide utilise une version héritée de la bibliothèque de client Stockage File d’attente Azure. Pour démarrer avec la dernière version, consultez [Démarrage rapide : Bibliothèque de client Stockage File d’attente Azure v12 pour .NET](storage-quickstart-queues-dotnet.md).

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

Ensuite, téléchargez et installez .NET Core 2.0 pour votre système d’exploitation. Si vous exécutez Windows, vous pouvez installer Visual Studio et utiliser .NET Framework. Vous pouvez aussi, si vous voulez, installer un éditeur à utiliser avec votre système d’exploitation.

### <a name="windows"></a>Windows

- Installez [.NET Core pour Windows](https://www.microsoft.com/net/download/windows) ou [.NET Framework](https://www.microsoft.com/net/download/windows) (inclus avec Visual Studio pour Windows)
- Installez [Visual Studio pour Windows](https://www.visualstudio.com/). Si vous utilisez .NET Core, l’installation de Visual Studio est facultative.

Pour plus d’informations sur le choix entre .NET Core et .NET Framework, consultez [Choix entre .NET Core et .NET Framework pour les applications serveur](/dotnet/standard/choosing-core-framework-server).

### <a name="linux"></a>Linux

- Installer [.NET Core pour Linux](https://www.microsoft.com/net/download/linux)
- Installer [Visual Studio Code](https://www.visualstudio.com/) et l’[extension C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) (optionnel)

### <a name="macos"></a>macOS

- Installer [.NET Core pour macOS](https://www.microsoft.com/net/download/macos)
- Installer [Visual Studio pour Mac](https://www.visualstudio.com/vs/visual-studio-mac/) (optionnel)

## <a name="download-the-sample-application"></a>Téléchargement de l'exemple d'application

L’exemple d’application utilisé dans ce guide de démarrage rapide est une application console de base. Vous pouvez explorer l’exemple d’application sur [GitHub](https://github.com/Azure-Samples/storage-queues-dotnet-quickstart).

Utilisez [Git](https://git-scm.com/) pour télécharger une copie de l’application dans votre environnement de développement.

```bash
git clone https://github.com/Azure-Samples/storage-queues-dotnet-quickstart.git
```

Cette commande clone le dépôt dans votre dossier Git local. Pour ouvrir la solution Visual Studio, recherchez le dossier `storage-queues-dotnet-quickstart`, ouvrez-le et double-cliquez sur `storage-queues-dotnet-quickstart.sln`.

[!INCLUDE [storage-copy-connection-string-portal](../../../includes/storage-copy-connection-string-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Configurer votre chaîne de connexion de stockage

Pour exécuter l’application, vous devez fournir la chaîne de connexion de votre compte de stockage. L’exemple d’application lit la chaîne de connexion à partir d’une variable d’environnement et l’utilise pour autoriser les requêtes dans Stockage Azure.

Après avoir copié votre chaîne de connexion, écrivez-la dans une variable d’environnement sur l’ordinateur local exécutant l’application. Pour définir la variable d’environnement, ouvrez une fenêtre de console et suivez les instructions pour votre système d’exploitation. Remplacez `<yourconnectionstring>` par votre chaîne de connexion :

### <a name="windows"></a>Windows

```cmd
setx storageconnectionstring "<yourconnectionstring>"
```

Après avoir ajouté la variable d’environnement, vous devrez peut-être redémarrer tous les programmes en cours d’exécution qui devront la lire, y compris la fenêtre de console. Par exemple, si vous utilisez Visual Studio comme éditeur, redémarrez Visual Studio avant d’exécuter l’exemple.

### <a name="linux"></a>Linux

```bash
export storageconnectionstring=<yourconnectionstring>
```

Après avoir ajouté la variable d’environnement, exécutez `source ~/.bashrc` depuis la fenêtre de console pour appliquer les changements.

### <a name="macos"></a>macOS

Modifiez votre profil bash_profile, et ajoutez la variable d’environnement :

```bash
export STORAGE_CONNECTION_STRING=<yourconnectionstring>
```

Après avoir ajouté la variable d’environnement, exécutez `source .bash_profile` depuis la fenêtre de console pour appliquer les changements.

## <a name="run-the-sample"></a>Exécution de l'exemple

L’exemple d’application crée une file d’attente et y ajoute un message. L’application commence par lire le message sans le supprimer de la file d’attente, puis elle récupère le message et le supprime de la file d’attente.

### <a name="windows"></a>Windows

Si vous utilisez Visual Studio comme éditeur, vous pouvez appuyer sur `F5` pour exécuter.

Sinon, accédez au répertoire de l’application, puis exécutez l’application avec la commande `dotnet run`.

```
dotnet run
```

### <a name="linux"></a>Linux

Accédez au répertoire de l’application, puis exécutez l’application avec la commande `dotnet run`.

```
dotnet run
```

### <a name="macos"></a>macOS

Accédez au répertoire de l’application, puis exécutez l’application avec la commande `dotnet run`.

```
dotnet run
```

Le résultat de l’exemple d’application ressemble à ce qui suit :

```
Azure Queues - .NET Quickstart sample

Created queue 'quickstartqueues-3136fe9a-fa52-4b19-a447-8999a847da52'

Added message 'aa8fa95f-07ea-4df7-bf86-82b3f7debfb7' to queue 'quickstartqueues-3136fe9a-fa52-4b19-a447-8999a847da52'
Message insertion time: 2/7/2019 4:30:46 AM +00:00
Message expiration time: 2/14/2019 4:30:46 AM +00:00

Contents of peeked message 'aa8fa95f-07ea-4df7-bf86-82b3f7debfb7': Hello, World

Message 'aa8fa95f-07ea-4df7-bf86-82b3f7debfb7' becomes visible again at 2/7/2019 4:31:16 AM +00:00

Processed and deleted message 'aa8fa95f-07ea-4df7-bf86-82b3f7debfb7'

Press any key to delete the sample queue.
```

## <a name="understand-the-sample-code"></a>Comprendre l’exemple de code

Ensuite, explorez l’exemple de code pour comprendre son fonctionnement.

### <a name="try-parsing-the-connection-string"></a>Essayez d’analyser la chaîne de connexion.

L’exemple vérifie d’abord si la variable d’environnement contient une chaîne de connexion à analyser pour créer un objet [`CloudStorageAccount`](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount) pointant vers le compte de stockage. Pour vérifier si la chaîne de connexion est valide, l’exemple utilise la méthode [`TryParse`](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount.tryparse). Si la méthode `TryParse` réussit, elle initialise la variable `storageAccount` et retourne `true`.

```csharp
// Retrieve the connection string for use with the application. The storage connection string is stored
// in an environment variable called storageconnectionstring, on the machine where the application is running.
// If the environment variable is created after the application is launched in a console or with Visual
// Studio, the shell needs to be closed and reloaded to take the environment variable into account.
string storageConnectionString = Environment.GetEnvironmentVariable("storageconnectionstring");

// Check whether the connection string can be parsed.
if (CloudStorageAccount.TryParse(storageConnectionString, out storageAccount))
{
    // If the connection string is valid, proceed with calls to Azure Queues here.
    ...
}
else
{
    Console.WriteLine(
        "A connection string has not been defined in the system environment variables. " +
        "Add an environment variable named 'storageconnectionstring' with your storage " +
        "connection string as a value.");
}
```

### <a name="create-the-queue"></a>Créer la file d’attente

Tout d’abord, l’exemple crée une file d’attente et y ajoute un message.

```csharp
// Create a queue called 'quickstartqueues' and append a GUID value so that the queue name
// is unique in your storage account.
queue = cloudQueueClient.GetQueueReference("quickstartqueues-" + Guid.NewGuid().ToString());
await queue.CreateAsync();

Console.WriteLine("Created queue '{0}'", queue.Name);
Console.WriteLine();
```

### <a name="add-a-message"></a>Ajouter un message

Ensuite, l’exemple ajoute un message en bas de la file d’attente.

Un message doit être dans un format utilisable dans une requête XML avec encodage UTF-8 et avoir une taille maximale de 64 Ko. Si un message contient des données binaires, nous recommandons de l’encoder au format Base64.

Par défaut, un message peut être conservé pendant une durée maximale de 7 jours. Vous pouvez spécifier n’importe quel nombre positif pour la durée de vie du message.

```csharp
// Create a message and add it to the queue. Set expiration time to 14 days.
CloudQueueMessage message = new CloudQueueMessage("Hello, World");
await queue.AddMessageAsync(message, new TimeSpan(14,0,0,0), null, null, null);
Console.WriteLine("Added message '{0}' to queue '{1}'", message.Id, queue.Name);
Console.WriteLine("Message insertion time: {0}", message.InsertionTime.ToString());
Console.WriteLine("Message expiration time: {0}", message.ExpirationTime.ToString());
Console.WriteLine();
```

Pour ajouter un message qui n’expire pas, utilisez `Timespan.FromSeconds(-1)` dans votre appel à [`AddMessageAsync`](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.addmessageasync).

```csharp
await queue.AddMessageAsync(message, TimeSpan.FromSeconds(-1), null, null, null);
```

### <a name="peek-a-message-from-the-queue"></a>Afficher l’aperçu d’un message de la file d’attente

L’exemple montre comment afficher l’aperçu d’un message dans une file d’attente. Quand vous affichez l’aperçu d’un message, vous pouvez lire le contenu du message. Toutefois, le message reste visible sur les autres clients afin qu’un autre client puisse après récupérer et traiter le même message.

```csharp
// Peek at the message at the front of the queue. Peeking does not alter the message's
// visibility, so that another client can still retrieve and process it.
CloudQueueMessage peekedMessage = await queue.PeekMessageAsync();

// Display the ID and contents of the peeked message.
Console.WriteLine("Contents of peeked message '{0}': {1}", peekedMessage.Id, peekedMessage.AsString);
Console.WriteLine();
```

### <a name="dequeue-a-message"></a>Enlever un message de la file d’attente

L’exemple montre aussi comment enlever un message de la file d’attente. Quand vous enlevez un message de la file d’attente, vous récupérez le message du haut de la file d’attente et vous le masquez temporairement sur les autres clients. Par défaut, un message reste masqué pendant 30 secondes. Pendant ce temps, votre code peut traiter le message. Pour terminer l’enlèvement du message, vous supprimez le message immédiatement après son traitement, afin qu’un autre client ne puisse pas enlever le même message de la file d’attente.

Si votre code ne parvient pas à traiter un message en raison d’une défaillance matérielle ou logicielle, le message redevient visible au terme du délai de masquage. Un autre client peut alors récupérer le même message et réessayer l’opération.

```csharp
// Retrieve the message at the front of the queue. The message becomes invisible for
// a specified interval, during which the client attempts to process it.
CloudQueueMessage retrievedMessage = await queue.GetMessageAsync();

// Display the time at which the message will become visible again if it is not deleted.
Console.WriteLine("Message '{0}' becomes visible again at {1}", retrievedMessage.Id, retrievedMessage.NextVisibleTime);
Console.WriteLine();

//Process and delete the message within the period of invisibility.
await queue.DeleteMessageAsync(retrievedMessage);
Console.WriteLine("Processed and deleted message '{0}'", retrievedMessage.Id);
Console.WriteLine();
```

### <a name="clean-up-resources"></a>Nettoyer les ressources

L’exemple nettoie les ressources qu’il a créées en supprimant la file d’attente. La suppression de la file d’attente supprime également tous les messages qu’elle contient.

```csharp
Console.WriteLine("Press any key to delete the sample queue.");
Console.ReadLine();
Console.WriteLine("Deleting the queue and any messages it contains...");
Console.WriteLine();
if (queue != null)
{
    await queue.DeleteIfExistsAsync();
}
```

## <a name="resources-for-developing-net-applications-with-queues"></a>Ressources sur le développement d’applications .NET avec des files d’attente

Consultez ces ressources supplémentaires sur le développement .NET avec le Stockage File d’attente Azure :

### <a name="binaries-and-source-code"></a>Fichiers binaires et code source

- Téléchargez les packages NuGet pour obtenir la dernière version de la [bibliothèque de client Stockage Azure pour .NET](/dotnet/api/overview/azure/storage)
  - [Commun](https://www.nuget.org/packages/microsoft.azure.storage.common/)
  - [Files d’attente](https://www.nuget.org/packages/Azure.Storage.Queues/)
- Consultez le [code source de la bibliothèque de client .NET](https://github.com/Azure/azure-storage-net) sur GitHub.

### <a name="azure-storage-client-library-reference-and-samples"></a>Référence et exemples de la bibliothèque de client Stockage Azure

- Pour plus d’informations sur les bibliothèques de client .NET, consultez la [Référence sur les bibliothèques de client Stockage Azure pour .NET](/dotnet/api/overview/azure/storage).
- Explorez les [Exemples de Stockage File d’attente](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=dotnet&term=queues) conçus à l’aide de la bibliothèque de client .NET.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez découvert comment ajouter des messages à une file d’attente, afficher un aperçu des messages d’une file d’attente, et enlever et traiter des messages de la file d’attente à l’aide de .NET.

> [!div class="nextstepaction"]
> [Communiquer entre applications avec le Stockage File d’attente Azure](/learn/modules/communicate-between-apps-with-azure-queue-storage/index)

- Pour en savoir plus sur .NET Core, consultez [Prise en main de .NET en 10 minutes](https://www.microsoft.com/net/learn/get-started/).
