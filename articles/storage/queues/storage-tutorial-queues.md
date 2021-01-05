---
title: 'Tutoriel : Utiliser des files d’attente Stockage File d’attente Azure dans .NET'
description: Tutoriel sur l’utilisation de Stockage File d’attente Azure pour créer des files d’attente et pour insérer, obtenir et supprimer des messages à l’aide du code .NET.
author: mhopkins-msft
ms.author: mhopkins
ms.reviewer: dineshm
ms.date: 06/09/2020
ms.topic: tutorial
ms.service: storage
ms.subservice: queues
ms.custom: devx-track-csharp
ms.openlocfilehash: 3c41b218ac0d347b2e58931421493755346b13d7
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97591971"
---
# <a name="tutorial-work-with-azure-queue-storage-queues-in-net"></a>Tutoriel : Utiliser des files d’attente Stockage File d’attente Azure dans .NET

Stockage File d’attente Azure implémente des files d’attente cloud pour permettre la communication entre les composants d’une application distribuée. Chaque file d’attente gère une liste de messages qui peuvent être ajoutés par un composant expéditeur et traités par un composant récepteur. Avec une file d’attente, votre application peut être mise à l’échelle immédiatement pour répondre à la demande. Cet article montre les étapes de base pour travailler avec une file d’attente Stockage File d’attente Azure.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> - Créer un compte Stockage Azure
> - Créer l’application
> - Ajouter des bibliothèques clientes Azure
> - Ajouter la prise en charge du code asynchrone
> - Créer une file d’attente
> - Insérer des messages dans une file d’attente
> - Enlever des messages d’une file d’attente
> - Supprimer une file d’attente vide
> - Vérifier les arguments de la ligne de commande
> - Générer et exécuter l’application

## <a name="prerequisites"></a>Prérequis

- Procurez-vous votre copie gratuite de l’éditeur [Visual Studio Code](https://code.visualstudio.com/download) multiplateforme.
- Téléchargez et installez le [kit SDK .NET Core](https://dotnet.microsoft.com/download) version 3.1 ou ultérieure.
- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="create-an-azure-storage-account"></a>Créer un compte de stockage Azure

Commencez par créer un compte Stockage Azure. Pour obtenir un guide pas à pas de la création d’un compte de stockage, consultez [Créer un compte de stockage](../common/storage-account-create.md?toc=%2Fazure%2Fstorage%2Fqueues%2Ftoc.json). Il s’agit d’une étape distincte que vous effectuez après avoir créé un compte Azure gratuit dans les prérequis.

## <a name="create-the-app"></a>Créer l’application

Créez une application .NET Core nommée `QueueApp`. Par souci pratique, cette application envoie et reçoit des messages via la file d’attente.

1. Dans une fenêtre de console (telle que cmd, PowerShell ou Azure CLI), utilisez la commande `dotnet new` pour créer une application de console avec le nom `QueueApp`. Cette commande crée un projet C# « hello world » simple avec un seul fichier source nommé `Program.cs`.

   ```console
   dotnet new console -n QueueApp
   ```

2. Accédez au dossier `QueueApp` nouvellement créé et générez l’application pour vérifier que tout est en ordre.

   ```console
   cd QueueApp
   ```

   ```console
   dotnet build
   ```

   Vous obtenez normalement des résultats similaires à la sortie suivante :

   ```output
   C:\Tutorials>dotnet new console -n QueueApp
   The template "Console Application" was created successfully.

   Processing post-creation actions...
   Running 'dotnet restore' on QueueApp\QueueApp.csproj...
     Restore completed in 155.63 ms for C:\Tutorials\QueueApp\QueueApp.csproj.

   Restore succeeded.

   C:\Tutorials>cd QueueApp

   C:\Tutorials\QueueApp>dotnet build
   Microsoft (R) Build Engine version 16.0.450+ga8dc7f1d34 for .NET Core
   Copyright (C) Microsoft Corporation. All rights reserved.

     Restore completed in 40.87 ms for C:\Tutorials\QueueApp\QueueApp.csproj.
     QueueApp -> C:\Tutorials\QueueApp\bin\Debug\netcoreapp3.1\QueueApp.dll

   Build succeeded.
       0 Warning(s)
       0 Error(s)

   Time Elapsed 00:00:02.40

   C:\Tutorials\QueueApp>_
   ```

<!-- markdownlint-disable MD023 -->

## <a name="add-the-azure-client-libraries"></a>Ajouter des bibliothèques clientes Azure

1. Ajoutez les bibliothèques clientes du stockage Azure au projet à l’aide de la commande `dotnet add package`.

   # <a name="net-v12"></a>[.NET v12](#tab/dotnet)

   Exécutez la commande suivante à partir du dossier du projet dans la fenêtre de console.

   ```console
   dotnet add package Azure.Storage.Queues
   ```

   # <a name="net-v11"></a>[.NET v11](#tab/dotnetv11)

   Exécutez les commandes suivantes à partir du dossier du projet dans la fenêtre de console.

   ```console
   dotnet add package Microsoft.Azure.Storage.Common
   ```

   ```console
   dotnet add package Microsoft.Azure.Storage.Queue
   ```
   ---

### <a name="add-using-statements"></a>Ajouter des instructions using

1. Sur la ligne de commande dans le répertoire du projet, tapez `code .` pour ouvrir Visual Studio Code dans le répertoire actif. Laissez ouverte la fenêtre de ligne de commande. Il y aura d’autres commandes à exécuter plus tard. Si vous êtes invité à ajouter des ressources C# nécessaires pour générer et déboguer, cliquez sur le bouton **Oui**.

1. Ouvrez le fichier source `Program.cs` et ajoutez les espaces de noms suivants juste après l’instruction `using System;`. Cette application utilise des types provenant de ces espaces de noms pour se connecter à Stockage Azure et pour utiliser des files d’attente.

   # <a name="net-v12"></a>[.NET v12](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_UsingStatements":::

   # <a name="net-v11"></a>[.NET v11](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_UsingStatements":::

1. Enregistrez le fichier `Program.cs`.

## <a name="add-support-for-asynchronous-code"></a>Ajouter la prise en charge du code asynchrone

Comme l’application utilise des ressources cloud, le code s’exécute de façon asynchrone.

1. Mettez à jour la méthode `Main` pour qu’elle s’exécute de façon asynchrone. Remplacez `void` par une valeur de retour `async Task`.

   ```csharp
   static async Task Main(string[] args)
   ```

1. Enregistrez le fichier `Program.cs`.

## <a name="create-a-queue"></a>Créer une file d’attente

Avant d’effectuer des appels dans les API Azure, vous devez obtenir vos informations d’identification à partir du portail Azure.

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

### <a name="add-the-connection-string-to-the-app"></a>Ajouter la chaîne de connexion à l’application

Ajoutez la chaîne de connexion à l’application afin qu’elle puisse accéder au compte de stockage.

1. Revenez à Visual Studio Code.

1. Dans la méthode `Main`, remplacez le code `Console.WriteLine("Hello, World");` par la ligne suivante qui récupère la chaîne de connexion de la variable d’environnement.

   # <a name="net-v12"></a>[.NET v12](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_DeclareConnectionString":::

   # <a name="net-v11"></a>[.NET v11](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_DeclareConnectionString":::

1. Ajoutez le code suivant à `Main` pour créer un objet de file d’attente, qui sera passé plus tard dans les méthodes d’envoi et de réception.

   # <a name="net-v12"></a>[.NET v12](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_CreateQueueClient":::

   # <a name="net-v11"></a>[.NET v11](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_CreateQueueClient":::

1. Enregistrez le fichier .

## <a name="insert-messages-into-the-queue"></a>Insérer des messages dans la file d’attente

Créez une méthode pour envoyer un message dans la file d’attente.

1. Ajoutez la méthode `InsertMessageAsync` suivante dans votre classe `Program`.

   # <a name="net-v12"></a>[.NET v12](#tab/dotnet)

   Une référence de file d’attente est passée à cette méthode. Si elle n’existe pas déjà, une nouvelle file d’attente est créée en appelant [`CreateIfNotExistsAsync`](/dotnet/api/azure.storage.queues.queueclient.createifnotexistsasync). Ensuite, elle ajoute `newMessage` à la file d’attente en appelant [`SendMessageAsync`](/dotnet/api/azure.storage.queues.queueclient.sendmessageasync).

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_InsertMessage":::

   # <a name="net-v11"></a>[.NET v11](#tab/dotnetv11)

   Une référence de file d’attente est passée à cette méthode. Si elle n’existe pas déjà, une nouvelle file d’attente est créée en appelant [`CreateIfNotExistsAsync`](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.createifnotexistsasync). Ensuite, elle ajoute `newMessage` à la file d’attente en appelant [`AddMessageAsync`](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.addmessageasync).

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_InsertMessage":::

1. **Facultatif :** Par défaut, la durée de vie maximale d’un message est définie sur sept jours. Vous pouvez spécifier n’importe quel nombre positif pour la durée de vie du message. L’extrait de code suivant ajoute un message qui n’expire **jamais**.

   # <a name="net-v12"></a>[.NET v12](#tab/dotnet)

    Pour ajouter un message qui n’expire pas, utilisez `Timespan.FromSeconds(-1)` dans votre appel à `SendMessageAsync`.

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Initial.cs" id="snippet_SendNonExpiringMessage":::

   # <a name="net-v11"></a>[.NET v11](#tab/dotnetv11)

    Pour ajouter un message qui n’expire pas, utilisez `Timespan.FromSeconds(-1)` dans votre appel à `AddMessageAsync`.

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Initial.cs" id="snippet_SendNonExpiringMessage":::

1. Enregistrez le fichier .

Le format d’un message de file d’attente doit être compatible avec une demande XML utilisant l’encodage UTF-8. La taille maximale d’un message est de 64 Ko. Si un message contient des données binaires, [encodez au format Base64](/dotnet/api/system.convert.tobase64string) le message.

## <a name="dequeue-messages"></a>Enlever des messages d’une file d’attente

Créez une méthode pour récupérer un message de la file d’attente. Une fois que le message est effectivement reçu, il est important de le supprimer de la file d’attente pour qu’il ne soit pas traité plusieurs fois.

1. Ajoutez une nouvelle méthode appelée `RetrieveNextMessageAsync` à votre classe `Program`.

   # <a name="net-v12"></a>[.NET v12](#tab/dotnet)

   Cette méthode reçoit un message de la file d’attente en appelant [`ReceiveMessagesAsync`](/dotnet/api/azure.storage.queues.queueclient.receivemessagesasync), en passant `1` dans le premier paramètre pour ne récupérer que le message suivant dans la file d’attente. Une fois le message reçu, supprimez-le de la file d’attente en appelant [`DeleteMessageAsync`](/dotnet/api/azure.storage.queues.queueclient.deletemessageasync).

   Quand un message est envoyé à la file d’attente avec une version du SDK antérieure à v12, il est automatiquement encodé en Base64. À compter de la version v12, cette fonctionnalité a été supprimée. Quand vous récupérez un message en utilisant le SDK v12, il n’est pas décodé automatiquement en base64. Vous devez [décoder le contenu en Base64](/dotnet/api/system.convert.frombase64string) vous-même.

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Initial.cs" id="snippet_InitialRetrieveMessage":::

   # <a name="net-v11"></a>[.NET v11](#tab/dotnetv11)

   Cette méthode reçoit un message de la file d’attente en appelant [`GetMessageAsync`](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.getmessageasync). Une fois le message reçu, supprimez-le de la file d’attente en appelant [`DeleteMessageAsync`](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.deletemessageasync).

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Initial.cs" id="snippet_InitialRetrieveMessage":::

1. Enregistrez le fichier .

## <a name="delete-an-empty-queue"></a>Supprimer une file d’attente vide

C’est une bonne pratique à la fin d’un projet que de déterminer si vous avez toujours besoin des ressources que vous avez créées. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Si la file d’attente existe mais qu’est vide, demandez à l’utilisateur s’il veut la supprimer.

1. Développez la méthode `RetrieveNextMessageAsync` pour y inclure une invite de suppression de la file d’attente vide.

   # <a name="net-v12"></a>[.NET v12](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_RetrieveMessage":::

   # <a name="net-v11"></a>[.NET v11](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_RetrieveMessage":::

1. Enregistrez le fichier .

## <a name="check-for-command-line-arguments"></a>Vérifier les arguments de la ligne de commande

Si des arguments de ligne de commande sont passés dans l’application, supposons qu’il s’agisse d’un message à ajouter à la file d’attente. Joignez ensemble les arguments pour en faire une chaîne. Ajoutez cette chaîne à la file d’attente des messages en appelant la méthode `InsertMessageAsync` que nous avons ajoutée précédemment.

S’il n’y a pas d’arguments de ligne de commande, tentez une opération de récupération. Appelez la méthode `RetrieveNextMessageAsync` pour récupérer le message suivant dans la file d’attente.

Enfin, attendez une entrée utilisateur avant de quitter en appelant `Console.ReadLine`.

1. Développez la méthode `Main` pour vérifier les arguments de la ligne de commande et attendre une entrée d’utilisateur.

   # <a name="net-v12"></a>[.NET v12](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_Main":::

   # <a name="net-v11"></a>[.NET v11](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_Main":::

1. Enregistrez le fichier .

## <a name="complete-code"></a>Code complet

Voici le code complet pour ce projet.

   # <a name="net-v12"></a>[.NET v12](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_AllCode":::

   # <a name="net-v11"></a>[.NET v11](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_AllCode":::
   ---

## <a name="build-and-run-the-app"></a>Générer et exécuter l’application

1. À partir de la ligne de commande dans le répertoire du projet, exécutez la commande dotnet suivante pour générer le projet.

   ```console
   dotnet build
   ```

1. Une fois le projet correctement généré, exécutez la commande suivante pour ajouter le premier message à la file d’attente.

   ```console
   dotnet run First queue message
   ```

   Cette sortie doit s’afficher :

   ```output
   C:\Tutorials\QueueApp>dotnet run First queue message
   The queue was created.
   Sent: First queue message
   Press Enter..._
   ```

1. Exécutez l’application sans argument de ligne de commande pour recevoir et supprimer le premier message de la file d’attente.

   ```console
   dotnet run
   ```

1. Continuer à exécuter l’application jusqu’à ce que tous les messages soient supprimés. Si vous l’exécutez une fois de plus, vous obtenez un message indiquant que la file d’attente est vide et une invite pour supprimer la file d’attente.

   ```output
   C:\Tutorials\QueueApp>dotnet run First queue message
   The queue was created.
   Sent: First queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run Second queue message
   Sent: Second queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run Third queue message
   Sent: Third queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run
   Received: First queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run
   Received: Second queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run
   Received: Third queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run
   The queue is empty. Attempt to delete it? (Y/N) Y
   Received: The queue was deleted.
   Press Enter...

   C:\Tutorials\QueueApp>_
   ```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

1. Créer une file d’attente
1. Ajouter et supprimer des messages dans une file d’attente
1. Supprimer une file d’attente Stockage File d’attente Azure

Pour plus d’informations, consultez les guides de démarrage rapide de Stockage File d’attente Azure.

> [!div class="nextstepaction"]
> [Démarrage rapide des files d’attente pour le portail](storage-quickstart-queues-portal.md)

- [Démarrage rapide des files d’attente pour .NET](storage-quickstart-queues-dotnet.md)
- [Démarrage rapide des files d’attente pour Java](storage-quickstart-queues-java.md)
- [Démarrage rapide des files d’attente pour Python](storage-quickstart-queues-python.md)
- [Démarrage rapide des files d’attente pour JavaScript](storage-quickstart-queues-nodejs.md)
