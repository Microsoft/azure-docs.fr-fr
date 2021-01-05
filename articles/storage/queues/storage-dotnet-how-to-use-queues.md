---
title: Prise en main du Stockage File d’attente Azure à l’aide de .NET - Stockage Azure
description: Les files d’attente Azure fournissent une messagerie asynchrone fiable entre les composants d’application. La messagerie cloud permet de mettre à l’échelle vos composants d’application indépendamment.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 10/08/2020
ms.service: storage
ms.subservice: queues
ms.topic: how-to
ms.reviewer: dineshm
ms.custom: devx-track-csharp
ms.openlocfilehash: b59c7d2af33efd82f27af9e13fc9c1f36ca788ee
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93348591"
---
# <a name="get-started-with-azure-queue-storage-using-net"></a>Prise en main du stockage de files d’attente Azure à l’aide de .NET

[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

## <a name="overview"></a>Vue d’ensemble

Le Stockage File d’attente Azure fournit une messagerie cloud entre les composants d’application. Lors de la conception d’applications pour la mise à l’échelle, des composants d’application sont souvent découplés, de sorte qu’ils peuvent être mis à l’échelle indépendamment. Le Stockage File d’attente offre une messagerie asynchrone entre les composants d’application, qu’ils soient exécutés dans le cloud, sur le bureau, sur un serveur local ou sur un appareil mobile. Le stockage de files d’attente prend également en charge la gestion des tâches asynchrones et la création des flux de travail de processus.

### <a name="about-this-tutorial"></a>À propos de ce didacticiel

Ce didacticiel montre comment écrire du code .NET pour des scénarios courants d’utilisation du stockage de files d’attente Azure. Les scénarios traités sont les suivants : création et suppression de files d’attente, et ajout, lecture et suppression des messages de file d’attente.

**Durée estimée :** 45 minutes

### <a name="prerequisites"></a>Prérequis

- [Microsoft Visual Studio](https://www.visualstudio.com/downloads/)
- Un [compte de stockage Azure](../common/storage-account-create.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="set-up-your-development-environment"></a>Configurer l''environnement de développement

Ensuite, configurez votre environnement de développement dans Visual Studio afin d’être prêt pour essayer les exemples de code fournis dans ce guide.

### <a name="create-a-windows-console-application-project"></a>Créer un projet d’application de console Windows

Dans Visual Studio, créez une application de console Windows. Les étapes suivantes vous montrent comment créer une application console dans Visual Studio 2019. Les étapes sont semblables pour d’autres versions de Visual Studio.

1. Sélectionnez **Fichier** > **Nouveau** > **Projet**
2. Sélectionnez **Plateforme** > **Windows**
3. Sélectionnez **Application console (.NET Framework)**
4. Sélectionnez **Suivant**.
5. Saisissez un nom pour votre application dans le champ **Nom du projet**.
6. Sélectionnez **Créer**

Tous les exemples de code figurant dans ce didacticiel peuvent être ajoutés à la méthode **Main()** dans le fichier **Program.cs** de votre application console.

Vous pouvez utiliser les bibliothèques de client du Stockage Azure dans n’importe quelle application .NET, y compris un service cloud Azure, une application web, ou une application de bureau ou mobile. Dans ce guide, nous utilisons une application console pour plus de simplicité.

### <a name="use-nuget-to-install-the-required-packages"></a>Utiliser NuGet pour installer les packages requis

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Pour terminer ce didacticiel, vous devez référencer les quatre packages suivants dans votre projet :

- [Bibliothèque Azure Core pour .NET](https://www.nuget.org/packages/Azure.Core/) : Ce package fournit des primitives, des abstractions et des applications auxiliaires partagées pour les bibliothèques clientes modernes du SDK Azure .NET.
- [Bibliothèque cliente commune Stockage Azure pour .NET](https://www.nuget.org/packages/Azure.Storage.Common/) : Ce package fournit une infrastructure partagée par les autres bibliothèques clientes du stockage Azure.
- [Bibliothèque du service de File d’attente Stockage Azure pour .NET](https://www.nuget.org/packages/Azure.Storage.Queues/) : Ce package permet d’utiliser le service de File d’attente Stockage Azure pour stocker les messages qui sont accessibles par un client.
- [Bibliothèque Configuration Manager pour .NET](https://www.nuget.org/packages/System.Configuration.ConfigurationManager/) : Ce package fournit un accès aux fichiers de configuration pour les applications clientes.

Vous pouvez utiliser NuGet pour obtenir ces packages. Procédez comme suit :

1. Cliquez avec le bouton droit sur votre projet dans **l’Explorateur de solutions** , puis sélectionnez **Gérer les packages NuGet**.
1. Sélectionnez **Parcourir**.
1. Recherchez « Azure.Storage.Queues » en ligne, puis sélectionnez **Installer** pour installer la bibliothèque cliente Stockage Azure et ses dépendances. Cela aura également pour effet d’installer les bibliothèques Azure.Storage.Common et Azure.Core, qui sont des dépendances de la bibliothèque de file d’attente.
1. Recherchez « System.Configuration.ConfigurationManager » en ligne, puis sélectionnez **Installer** pour installer Configuration Manager.

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Pour terminer ce didacticiel, vous devez référencer les trois packages suivants dans votre projet :

- [Bibliothèque de client commune du Stockage Microsoft Azure pour .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/) : ce package fournit un accès programmatique aux ressources de données de votre compte de stockage.
- [Bibliothèque du Stockage File d’attente Azure pour .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Queue/) : cette bibliothèque de client permet d’utiliser le service Stockage File d’attente Azure pour stocker les messages qui sont accessibles par un client.
- [Bibliothèque Microsoft Azure Configuration Manager pour .NET](https://www.nuget.org/packages/Microsoft.Azure.ConfigurationManager/) : ce package fournit une classe pour l’analyse d’une chaîne de connexion à partir d’un fichier config, quel que soit l’emplacement d’exécution de votre application.

Vous pouvez utiliser NuGet pour obtenir ces packages. Procédez comme suit :

1. Cliquez avec le bouton droit sur votre projet dans **l’Explorateur de solutions** , puis sélectionnez **Gérer les packages NuGet**.
1. Sélectionnez **Parcourir**.
1. Recherchez « Microsoft.Azure.Storage.Queue » en ligne, puis sélectionnez **Installer** pour installer la bibliothèque de client du Stockage Azure et ses dépendances. Cela installera également la bibliothèque Microsoft.Azure.Storage.Common, qui est une dépendance de la bibliothèque de file d’attente.
1. Recherchez « Microsoft.Azure.ConfigurationManager » en ligne, puis sélectionnez **Installer** pour installer Azure Configuration Manager.

---

### <a name="determine-your-target-environment"></a>Déterminer votre environnement cible

Vous avez le choix entre deux environnements pour exécuter les exemples de ce guide :

- Vous pouvez exécuter votre code sur un compte Azure Storage dans le cloud.
- Vous pouvez exécuter votre code sur l’émulateur de stockage Azurite. L’émulateur de stockage Azurite est un environnement local qui émule un compte Stockage Azure dans le cloud. Azurite est une option gratuite permettant de tester et déboguer votre code lors du développement de votre application. L’émulateur utilise un compte et une clé connus. Pour plus d’informations, consultez [Utilisation de l’émulateur Azurite pour le développement et le test de Stockage Azure local](../common/storage-use-azurite.md).

> [!NOTE]
> Vous pouvez cibler l’émulateur de stockage pour éviter les frais liés à l’utilisation des services de stockage Azure. Toutefois, si vous choisissez de cibler un compte de stockage Azure situé dans le cloud, les frais associés à l’utilisation de ce didacticiel seront négligeables.

## <a name="get-your-storage-connection-string"></a>Obtenir votre chaîne de connexion de stockage

Les bibliothèques de client du Stockage Azure pour .NET prennent en charge l’utilisation d’une chaîne de connexion de stockage pour la configuration de points de terminaison et d’informations d’identification permettant d’accéder aux services de stockage. Pour plus d’informations, consultez [Gérer les clés d’accès au compte de stockage](../common/storage-account-keys-manage.md).

### <a name="copy-your-credentials-from-the-azure-portal"></a>Copier vos informations d’identification depuis le portail Azure

L’exemple de code a besoin d’autoriser l’accès à votre compte de stockage. Pour l’autorisation, vous devez fournir vos informations d’identification du compte de stockage à l’application sous la forme d’une chaîne de connexion. Pour afficher les informations d’identification de votre compte de stockage :

1. Accédez au [portail Azure](https://portal.azure.com).
2. Recherchez votre compte de stockage.
3. Dans la section **Paramètres** de la présentation du compte de stockage, sélectionnez **Clés d’accès**. Vos clés d’accès au compte s’affichent, ainsi que la chaîne de connexion complète de chaque clé.
4. Recherchez la valeur de **Chaîne de connexion** sous **clé1** , puis cliquez sur le bouton **Copier** pour copier la chaîne de connexion. Vous allez ajouter la valeur de chaîne de connexion dans une variable d’environnement à l’étape suivante.

    ![Capture d’écran montrant comment copier une chaîne de connexion à partir du portail Azure](media/storage-dotnet-how-to-use-queues/portal-connection-string.png)

Pour plus d’informations sur les chaînes de connexion, voir [Configuration d’une chaîne de connexion dans Stockage Azure](../common/storage-configure-connection-string.md).

> [!NOTE]
> Votre clé de compte de stockage est similaire au mot de passe racine pour votre compte de stockage. Veillez toujours à protéger votre clé de compte de stockage. Évitez de la communiquer à d’autres utilisateurs, de la coder en dur ou de l’enregistrer dans un fichier texte brut accessible à d’autres personnes. Régénérez votre clé à l’aide du portail Azure si vous pensez que sa confidentialité est compromise.

La meilleure façon de conserver votre chaîne de connexion de stockage est dans un fichier de configuration. Pour configurer votre chaîne de connexion, ouvrez le fichier *app.config* depuis l’Explorateur de solutions de Visual Studio. Ajoutez le contenu de l’élément `\<appSettings\>` indiqué ci-dessous. Remplacez *connection-string* par la valeur copiée depuis votre compte de stockage dans le portail :

```xml
<configuration>
    <startup>
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.7.2" />
    </startup>
    <appSettings>
        <add key="StorageConnectionString" value="connection-string" />
    </appSettings>
</configuration>
```

Par exemple, votre paramètre de configuration est semblable à :

```xml
<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=storagesample;AccountKey=GMuzNHjlB3S9itqZJHHCnRkrokLkcSyW7yK9BRbGp0ENePunLPwBgpxV1Z/pVo9zpem/2xSHXkMqTHHLcx8XRA==EndpointSuffix=core.windows.net" />
```

Pour cibler l’émulateur de stockage Azurite, vous pouvez utiliser un raccourci qui correspond à la clé et au nom de compte connus. Dans ce cas, le paramètre de votre chaîne de connexion est :

```xml
<add key="StorageConnectionString" value="UseDevelopmentStorage=true" />
```

### <a name="add-using-directives"></a>Ajouter des directives d’utilisation

Ajoutez les directives `using` suivantes au fichier `Program.cs` :

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_UsingStatements":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
using System; // Namespace for Console output
using Microsoft.Azure; // Namespace for CloudConfigurationManager
using Microsoft.Azure.Storage; // Namespace for CloudStorageAccount
using Microsoft.Azure.Storage.Queue; // Namespace for Queue storage types
```

---

### <a name="create-the-queue-service-client"></a>Création du client du service Queue

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

La classe [QueueClient](/dotnet/api/azure.storage.queues.queueclient) vous permet de récupérer des files d’attente stockées dans Stockage File d’attente. Voici un moyen de créer le client du service :

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_CreateClient":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

La classe [CloudQueueClient](/dotnet/api/microsoft.azure.storage.queue.cloudqueueclient?view=azure-dotnet-legacy&preserve-view=true) vous permet de récupérer des files d’attente stockées dans Queue Storage. Voici un moyen de créer le client du service :

```csharp
// Retrieve storage account from connection string
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
```

---

Vous êtes maintenant prêt à écrire du code qui lit et écrit des données dans le Queue Storage.

## <a name="create-a-queue"></a>Créer une file d’attente

Cet exemple montre comment créer une file d’attente :

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_CreateQueue":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
// Retrieve storage account from connection string
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Create the queue if it doesn't already exist
queue.CreateIfNotExists();
```

---

## <a name="insert-a-message-into-a-queue"></a>Insertion d'un message dans une file d'attente

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Pour insérer un message dans une file d’attente existante, appelez la méthode [SendMessage](/dotnet/api/azure.storage.queues.queueclient.sendmessage). Un message peut être un `string` (au format UTF-8) ou un tableau `byte`. Le code suivant crée une file d’attente (si elle n’existe pas) et insère un message :

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_InsertMessage":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Pour insérer un message dans une file d'attente existante, commencez par créer un [CloudQueueMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage?view=azure-dotnet-legacy&preserve-view=true). Appelez ensuite la méthode [AddMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.addmessage?view=azure-dotnet-legacy&preserve-view=true) . Un `CloudQueueMessage` peut être créé à partir d’un `string` (au format UTF-8) ou d’un tableau `byte`. Voici le code qui crée une file d’attente (si elle n’existe pas) et insère le message « Hello, World » :

```csharp
// Retrieve storage account from connection string
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Create the queue if it doesn't already exist
queue.CreateIfNotExists();

// Create a message and add it to the queue
CloudQueueMessage message = new CloudQueueMessage("Hello, World");
queue.AddMessage(message);
```

---

## <a name="peek-at-the-next-message"></a>Lecture furtive du message suivant

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Vous pouvez lire furtivement le message dans la file d’attente sans l’enlever de la file d’attente en appelant la méthode [PeekMessages](/dotnet/api/azure.storage.queues.queueclient.peekmessages). Si vous ne transmettez pas de valeur pour le paramètre *maxMessages* , la valeur par défaut consiste à afficher un aperçu d’un message.

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_PeekMessage":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Vous pouvez lire furtivement le message au début de la file d'attente sans l'enlever de la file d'attente en appelant la méthode [PeekMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.peekmessage?view=azure-dotnet-legacy&preserve-view=true) .

```csharp
// Retrieve storage account from connection string
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Peek at the next message
CloudQueueMessage peekedMessage = queue.PeekMessage();

// Display message.
Console.WriteLine(peekedMessage.AsString);
```

---

## <a name="change-the-contents-of-a-queued-message"></a>Modification du contenu d'un message en file d'attente

Vous pouvez modifier le contenu d'un message placé dans la file d'attente. Si le message représente une tâche, vous pouvez utiliser cette fonctionnalité pour mettre à jour l'état de la tâche. Le code suivant met à jour le message de la file d'attente avec un nouveau contenu et ajoute 60 secondes au délai d'expiration de la visibilité. Cette opération enregistre l'état de la tâche associée au message et accorde une minute supplémentaire au client pour traiter le message. Vous pouvez utiliser cette technique pour suivre des flux de travail à plusieurs étapes sur les messages de file d'attente, sans devoir reprendre du début si une étape du traitement échoue à cause d'une défaillance matérielle ou logicielle. Normalement, vous conservez aussi un nombre de nouvelles tentatives et si le message est retenté plus de *n* fois, vous le supprimez. Cela protège du déclenchement d'une erreur d'application par un message chaque fois qu'il est traité.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_UpdateMessage":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Get the message from the queue and update the message contents.
CloudQueueMessage message = queue.GetMessage();
message.SetMessageContent2("Updated contents.", false);
queue.UpdateMessage(message,
    TimeSpan.FromSeconds(60.0),  // Make it invisible for another 60 seconds.
    MessageUpdateFields.Content | MessageUpdateFields.Visibility);
```

---

## <a name="de-queue-the-next-message"></a>Enlèvement du message suivant de la file d'attente

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Supprimez un message d’une file d’attente en deux étapes. Lorsque vous appelez [ReceiveMessages](/dotnet/api/azure.storage.queues.queueclient.receivemessages), vous obtenez le message suivant dans une file d’attente. Un message renvoyé par `ReceiveMessages` devient invisible par les autres codes lisant les messages de cette file d'attente. Par défaut, ce message reste invisible pendant 30 secondes. Pour finaliser la suppression du message de la file d'attente, vous devez aussi appeler [DeleteMessage](/dotnet/api/azure.storage.queues.queueclient.deletemessage). Ce processus de suppression d'un message en deux étapes garantit que, si votre code ne parvient pas à traiter un message à cause d'une défaillance matérielle ou logicielle, une autre instance de votre code peut obtenir le même message et réessayer. Votre code appelle `DeleteMessage` juste après le traitement du message.

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_DequeueMessage":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Votre code enlève un message d'une file d'attente en deux étapes. Lorsque vous appelez [GetMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.getmessage?view=azure-dotnet-legacy&preserve-view=true), vous obtenez le message suivant dans une file d'attente. Un message renvoyé par `GetMessage` devient invisible par les autres codes lisant les messages de cette file d'attente. Par défaut, ce message reste invisible pendant 30 secondes. Pour finaliser la suppression du message de la file d'attente, vous devez aussi appeler [DeleteMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.deletemessage?view=azure-dotnet-legacy&preserve-view=true). Ce processus de suppression d'un message en deux étapes garantit que, si votre code ne parvient pas à traiter un message à cause d'une défaillance matérielle ou logicielle, une autre instance de votre code peut obtenir le même message et réessayer. Votre code appelle `DeleteMessage` juste après le traitement du message.

```csharp
// Retrieve storage account from connection string
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Get the next message
CloudQueueMessage retrievedMessage = queue.GetMessage();

//Process the message in less than 30 seconds, and then delete the message
queue.DeleteMessage(retrievedMessage);
```

---

## <a name="use-async-await-pattern-with-common-queue-storage-apis"></a>Utiliser le modèle Async-Await avec les API de stockage de files d’attente communes

Cet exemple décrit comment utiliser le modèle Async-Await avec les API de stockage de files d’attente communes. L’exemple appelle la version asynchrone de chacune des méthodes spécifiées, comme l’indique le suffixe *Async* de chaque méthode. Quand une méthode asynchrone est utilisée, le modèle Async-Await suspend l’exécution locale jusqu’à la fin de l’appel. Ce comportement permet au thread actuel d’effectuer d’autres tâches afin d’éviter les goulots d’étranglement au niveau des performances et d’améliorer la réactivité globale de votre application. Pour plus d’informations sur l’utilisation du modèle Async-Await dans .NET, consultez l’article [Programmation asynchrone avec Async et Await (C# et Visual Basic)](/previous-versions/hh191443(v=vs.140))

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_AsyncQueue":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
// Create the queue if it doesn't already exist
if(await queue.CreateIfNotExistsAsync())
{
    Console.WriteLine("Queue '{0}' Created", queue.Name);
}
else
{
    Console.WriteLine("Queue '{0}' Exists", queue.Name);
}

// Create a message to put in the queue
CloudQueueMessage cloudQueueMessage = new CloudQueueMessage("My message");

// Async enqueue the message
await queue.AddMessageAsync(cloudQueueMessage);
Console.WriteLine("Message added");

// Async dequeue the message
CloudQueueMessage retrievedMessage = await queue.GetMessageAsync();
Console.WriteLine("Retrieved message with content '{0}'", retrievedMessage.AsString);

// Async delete the message
await queue.DeleteMessageAsync(retrievedMessage);
Console.WriteLine("Deleted message");
```

---

## <a name="leverage-additional-options-for-de-queuing-messages"></a>Utilisation d’options supplémentaires pour l’enlèvement des messages

Il existe deux façons de personnaliser la récupération des messages à partir d'une file d'attente. Premièrement, vous pouvez obtenir un lot de messages (jusqu'à 32). Deuxièmement, vous pouvez définir un délai d'expiration de l'invisibilité plus long ou plus court afin d'accorder à votre code plus ou moins de temps pour traiter complètement chaque message.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

L’exemple de code suivant utilise la méthode [ReceiveMessages](/dotnet/api/azure.storage.queues.queueclient.receivemessages) pour obtenir 20 messages en un appel. Ensuite, il traite chaque message à l'aide d'une boucle `foreach`. Il définit également le délai d'expiration de l'invisibilité sur cinq minutes pour chaque message. Notez que le délai de 5 minutes démarre en même temps pour tous les messages, donc une fois les 5 minutes écoulées après l’appel de `ReceiveMessages`, tous les messages n’ayant pas été supprimés redeviennent visibles.

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_DequeueMessages":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

L'exemple de code suivant utilise la méthode [GetMessages](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.getmessages?view=azure-dotnet-legacy&preserve-view=true) pour obtenir 20 messages en un appel. Ensuite, il traite chaque message à l'aide d'une boucle `foreach`. Il définit également le délai d'expiration de l'invisibilité sur cinq minutes pour chaque message. Notez que le délai de 5 minutes démarre en même temps pour tous les messages, donc une fois les 5 minutes écoulées après l’appel de `GetMessages`, tous les messages n’ayant pas été supprimés redeviennent visibles.

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

foreach (CloudQueueMessage message in queue.GetMessages(20, TimeSpan.FromMinutes(5)))
{
    // Process all messages in less than 5 minutes, deleting each message after processing.
    queue.DeleteMessage(message);
}
```

---

## <a name="get-the-queue-length"></a>Obtention de la longueur de la file d'attente

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Vous pouvez obtenir une estimation du nombre de messages dans une file d'attente. La méthode [GetProperties](/dotnet/api/azure.storage.queues.queueclient.getproperties) demande au service de File d’attente d’extraire les propriétés de la file d’attente, y compris le nombre de messages. La propriété [ApproximateMessagesCount](/dotnet/api/azure.storage.queues.models.queueproperties.approximatemessagescount) contient le nombre approximatif de messages dans la file d’attente. Ce nombre n’est pas inférieur au nombre réel de messages dans la file d’attente, mais peut être supérieur.

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_GetQueueLength":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Vous pouvez obtenir une estimation du nombre de messages dans une file d'attente. La méthode [FetchAttributes](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.fetchattributes?view=azure-dotnet-legacy&preserve-view=true) demande au service de files d'attente d'extraire les attributs de la file d'attente, y compris le nombre de messages. La propriété [ApproximateMessageCount](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.approximatemessagecount?view=azure-dotnet-legacy&preserve-view=true) renvoie la dernière valeur extraite par la méthode `FetchAttributes`, sans appeler le service de File d’attente.

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Fetch the queue attributes.
queue.FetchAttributes();

// Retrieve the cached approximate message count.
int cachedMessageCount = queue.ApproximateMessageCount;

// Display number of messages.
Console.WriteLine("Number of messages in queue: " + cachedMessageCount);
```

---

## <a name="delete-a-queue"></a>Suppression d'une file d'attente

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Pour supprimer une file d'attente et tous les messages qu'elle contient, appelez la méthode [Delete](/dotnet/api/azure.storage.queues.queueclient.delete) sur l'objet file d'attente.

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_DeleteQueue":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Pour supprimer une file d'attente et tous les messages qu'elle contient, appelez la méthode [Delete](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.delete?view=azure-dotnet-legacy&preserve-view=true) sur l'objet file d'attente.

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Delete the queue.
queue.Delete();
```

---

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous connaissez les bases du stockage des files d'attente, consultez les liens suivants pour apprendre à exécuter les tâches de stockage plus complexes.

- Pour plus d'informations sur les API disponibles, consultez la documentation de référence des services de files d'attente :
  - [Référence de la bibliothèque cliente de stockage pour .NET](/dotnet/api/overview/azure/storage)
  - [Référence d’API REST](/rest/api/storageservices/)
- Pour plus d’informations sur les autres options de stockage de données dans Azure, consultez d’autres guides de fonctionnalités.
  - [Prise en main du stockage de tables Azure à l’aide de .NET](../../cosmos-db/tutorial-develop-table-dotnet.md) pour le stockage de données structurées.
  - [Prise en main d’Azure Blob Storage à l’aide de .NET](../blobs/storage-quickstart-blobs-dotnet.md) pour le stockage de données non structurées.
  - [Connexion à SQL Database à l’aide de .NET (C#)](../../azure-sql/database/connect-query-dotnet-core.md) pour stocker des données relationnelles.
- Découvrez comment simplifier le code que vous écrivez avec Azure Storage, à l’aide du [Kit de développement logiciel (SDK) Azure WebJobs](https://github.com/Azure/azure-webjobs-sdk/wiki).
