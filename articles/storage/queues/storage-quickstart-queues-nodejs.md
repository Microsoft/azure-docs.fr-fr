---
title: 'Démarrage rapide : Bibliothèque de client Stockage File d’attente Azure v12 – JavaScript'
description: Découvrez comment utiliser la bibliothèque de client Stockage File d’attente Azure v12 pour JavaScript, afin de créer une file d’attente et d’y ajouter des messages. Apprenez ensuite à lire et à supprimer des messages de la file d’attente. Vous apprendrez également à supprimer une file d’attente.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 12/13/2019
ms.topic: quickstart
ms.service: storage
ms.subservice: queues
ms.custom: devx-track-js
ms.openlocfilehash: 23c58526ba481a56b371bd077661d8d0bc7d45c7
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97586531"
---
# <a name="quickstart-azure-queue-storage-client-library-v12-for-javascript"></a>Démarrage rapide : Bibliothèque de client Stockage File d’attente Azure v12 pour JavaScript

Familiarisez-vous avec la bibliothèque de client Storage File d’attente Azure v12 pour JavaScript. Le Stockage File d’attente Azure est un service permettant de stocker un grand nombre de messages dans le but de les récupérer et de les traiter plus tard. Suivez les étapes suivantes pour installer le package et essayer un exemple de code pour les tâches de base.

Utilisez la bibliothèque de client Stockage File d’attente Azure v12 pour JavaScript afin d’effectuer les opérations suivantes :

- Créer une file d’attente
- Ajouter des messages à une file d’attente
- Afficher un aperçu des messages d’une file d’attente
- Mettre à jour un message dans une file d’attente
- Réception des messages d'une file d'attente
- Supprimer des messages d’une file d’attente
- Suppression d'une file d'attente

Ressources supplémentaires :

- [Documentation de référence de l’API](/javascript/api/@azure/storage-queue/)
- [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-queue)
- [Package (npm)](https://www.npmjs.com/package/@azure/storage-queue)
- [Exemples](../common/storage-samples-javascript.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json#queue-samples)

## <a name="prerequisites"></a>Prérequis

- Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/)
- Compte de stockage Azure : [créez un compte de stockage](../common/storage-account-create.md)
- [Node.js](https://nodejs.org/en/download/) actuel pour votre système d’exploitation.

## <a name="setting-up"></a>Configuration

Cette section vous guide tout au long de la préparation d’un projet à utiliser avec la bibliothèque de client Stockage File d’attente Azure v12 pour JavaScript.

### <a name="create-the-project"></a>Créer le projet

Créez une application Node.js nommée `queues-quickstart-v12`.

1. Dans une fenêtre de console (telle que cmd, PowerShell ou Bash), créez un nouveau répertoire pour le projet.

    ```console
    mkdir queues-quickstart-v12
    ```

1. Basculez vers le répertoire `queues-quickstart-v12` nouvellement créé.

    ```console
    cd queues-quickstart-v12
    ```

1. Créez un nouveau fichier texte appelé `package.json`. Ce fichier définit le projet Node.js. Enregistrez ce fichier dans le répertoire `queues-quickstart-v12`. Voici le contenu du fichier :

    ```json
    {
        "name": "queues-quickstart-v12",
        "version": "1.0.0",
        "description": "Use the @azure/storage-queue SDK version 12 to interact with Azure Queue storage",
        "main": "queues-quickstart-v12.js",
        "scripts": {
            "start": "node queues-quickstart-v12.js"
        },
        "author": "Your Name",
        "license": "MIT",
        "dependencies": {
            "@azure/storage-queue": "^12.0.0",
            "@types/dotenv": "^4.0.3",
            "dotenv": "^6.0.0"
        }
    }
    ```

    Vous pouvez placer votre propre nom dans pour le champ `author`, si vous le souhaitez.

### <a name="install-the-package"></a>Installer le package

Alors que vous êtes toujours dans le répertoire `queues-quickstart-v12`, installez le package de la bibliothèque de client Stockage File d’attente Azure pour JavaScript au moyen de la commande `npm install`.

```console
npm install
```

Cette commande lit le fichier `package.json` et installe le package de la bibliothèque de client Stockage File d’attente Azure v12 pour JavaScript et toutes les bibliothèques dont elle dépend.

### <a name="set-up-the-app-framework"></a>Configurer le framework d’application

À partir du répertoire de projet :

1. Ouvrez un nouveau fichier texte dans votre éditeur de code
1. Ajouter des appels `require` pour charger des modules Azure et Node.js
1. Créez la structure du programme, y compris la gestion des exceptions de base

    Voici le code :

    ```javascript
    const { QueueClient } = require("@azure/storage-queue");
    const uuidv1 = require("uuid/v1");

    async function main() {
        console.log("Azure Queue Storage client library v12 - JavaScript quickstart sample");
        // Quick start code goes here
    }

    main().then(() => console.log("\nDone")).catch((ex) => console.log(ex.message));

    ```

1. Enregistrez le nouveau fichier sous `queues-quickstart-v12.js` dans le répertoire `queues-quickstart-v12`.

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Modèle objet

Stockage File d’attente Azure est un service permettant de stocker un grand nombre de messages. La taille maximale d’un message de file d’attente est de 64 Ko. Une file d’attente peut contenir des millions de messages, dans la limite de la capacité totale d’un compte de stockage. Les files d’attente sont couramment utilisées pour créer un backlog de travail à traiter de façon asynchrone. Le Stockage File d’attente offre trois types de ressources :

- Le compte de stockage
- Une file d’attente dans le compte de stockage
- Les messages dans la file d’attente

Le diagramme suivant montre la relation entre ces ressources.

![Diagramme de l’architecture du Stockage File d’attente](./media/storage-queues-introduction/queue1.png)

Utilisez les classes JavaScript suivantes pour interagir avec ces ressources :

- [`QueueServiceClient`](/javascript/api/@azure/storage-queue/queueserviceclient) : `QueueServiceClient` vous permet de gérer toutes les files d’attente de votre compte de stockage.
- [`QueueClient`](/javascript/api/@azure/storage-queue/queueclient) : la classe `QueueClient` vous permet de gérer et de manipuler une file d’attente individuelle et ses messages.
- [`QueueMessage`](/javascript/api/@azure/storage-queue/queuemessage) : la classe `QueueMessage` représente les objets individuels retournés lors de l’appel de [`ReceiveMessages`](/javascript/api/@azure/storage-queue/queueclient#receivemessages-queuereceivemessageoptions-) dans une file d’attente.

## <a name="code-examples"></a>Exemples de code

Ces exemples d’extraits de code vous montrent comment effectuer les actions suivantes avec la bibliothèque de client Stockage File d’attente Azure pour JavaScript :

- [Obtenir la chaîne de connexion](#get-the-connection-string)
- [Créer une file d’attente](#create-a-queue)
- [Ajouter des messages à une file d’attente](#add-messages-to-a-queue)
- [Afficher un aperçu des messages d’une file d’attente](#peek-at-messages-in-a-queue)
- [Mettre à jour un message dans une file d’attente](#update-a-message-in-a-queue)
- [Recevoir les messages d’une file d’attente](#receive-messages-from-a-queue)
- [Supprimer des messages d’une file d’attente](#delete-messages-from-a-queue)
- [Supprimer une file d’attente](#delete-a-queue)

### <a name="get-the-connection-string"></a>Obtenir la chaîne de connexion

Le code suivant récupère la chaîne de connexion pour le compte de stockage à partir de la variable d’environnement créée dans la section [Configurer votre chaîne de connexion de stockage](#configure-your-storage-connection-string).

Ajoutez ce code dans la fonction `main` :

```javascript
// Retrieve the connection string for use with the application. The storage
// connection string is stored in an environment variable on the machine
// running the application called AZURE_STORAGE_CONNECTION_STRING. If the
// environment variable is created after the application is launched in a
// console or with Visual Studio, the shell or application needs to be
// closed and reloaded to take the environment variable into account.
const AZURE_STORAGE_CONNECTION_STRING = process.env.AZURE_STORAGE_CONNECTION_STRING;
```

### <a name="create-a-queue"></a>Créer une file d’attente

Choisissez un nom pour la nouvelle file d’attente. Le code suivant ajoute une valeur UUID au nom de la file d’attente pour s’assurer qu’il est unique.

> [!IMPORTANT]
> Les noms de file d’attente peuvent contenir uniquement des lettres minuscules, des chiffres et des traits d’union, et doivent commencer par une lettre ou un nombre. Chaque trait d’union doit être précédé et suivi d’un caractère autre qu’un tiret. Le nom doit avoir entre 3 et 63 caractères. Pour plus d’informations, consultez [Affectation de noms pour les files d’attente et les métadonnées](/rest/api/storageservices/naming-queues-and-metadata).

Créez une instance de la classe [`QueueClient`](/javascript/api/@azure/storage-queue/queueclient). Ensuite, appelez la méthode [`create`](/javascript/api/@azure/storage-queue/queueclient#create-queuecreateoptions-) pour créer la file d’attente dans votre compte de stockage.

Ajoutez ce code à la fin de la fonction `main` :

```javascript
// Create a unique name for the queue
const queueName = "quickstart" + uuidv1();

console.log("\nCreating queue...");
console.log("\t", queueName);

// Instantiate a QueueClient which will be used to create and manipulate a queue
const queueClient = new QueueClient(AZURE_STORAGE_CONNECTION_STRING, queueName);

// Create the queue
const createQueueResponse = await queueClient.create();
console.log("Queue created, requestId:", createQueueResponse.requestId);
```

### <a name="add-messages-to-a-queue"></a>Ajouter des messages à une file d’attente

L’extrait de code suivant ajoute des messages à la file d’attente en appelant la méthode [`sendMessage`](/javascript/api/@azure/storage-queue/queueclient#sendmessage-string--queuesendmessageoptions-). Il enregistre également le [`QueueMessage`](/javascript/api/@azure/storage-queue/queuemessage) retourné par le troisième appel de `sendMessage`. Le `sendMessageResponse` retourné est utilisé pour mettre à jour le contenu du message ultérieurement dans le programme.

Ajoutez ce code à la fin de la fonction `main` :

```javascript
console.log("\nAdding messages to the queue...");

// Send several messages to the queue
await queueClient.sendMessage("First message");
await queueClient.sendMessage("Second message");
const sendMessageResponse = await queueClient.sendMessage("Third message");

console.log("Messages added, requestId:", sendMessageResponse.requestId);
```

### <a name="peek-at-messages-in-a-queue"></a>Afficher un aperçu des messages d’une file d’attente

Affichez un aperçu des messages de la file d’attente en appelant la méthode [`peekMessages`](/javascript/api/@azure/storage-queue/queueclient#peekmessages-queuepeekmessagesoptions-). Cette méthode récupère un ou plusieurs messages du début de la file d’attente, mais ne modifie pas la visibilité du message.

Ajoutez ce code à la fin de la fonction `main` :

```javascript
console.log("\nPeek at the messages in the queue...");

// Peek at messages in the queue
const peekedMessages = await queueClient.peekMessages({ numberOfMessages : 5 });

for (i = 0; i < peekedMessages.peekedMessageItems.length; i++) {
    // Display the peeked message
    console.log("\t", peekedMessages.peekedMessageItems[i].messageText);
}
```

### <a name="update-a-message-in-a-queue"></a>Mettre à jour un message dans une file d’attente

Mettez à jour le contenu d’un message en appelant la méthode [`updateMessage`](/javascript/api/@azure/storage-queue/queueclient#updatemessage-string--string--string--undefined---number--queueupdatemessageoptions-). Cette méthode peut changer le contenu et le délai d’expiration de la visibilité d’un message. Le contenu du message doit être une chaîne encodée en UTF-8 d’une taille maximale de 64 Ko. Avec le nouveau contenu, transmettez `messageId` et `popReceipt` à partir de la réponse qui a été enregistrée dans le code. Les propriétés `sendMessageResponse` identifient le message à mettre à jour.

```javascript
console.log("\nUpdating the third message in the queue...");

// Update a message using the response saved when calling sendMessage earlier
updateMessageResponse = await queueClient.updateMessage(
    sendMessageResponse.messageId,
    sendMessageResponse.popReceipt,
    "Third message has been updated"
);

console.log("Message updated, requestId:", updateMessageResponse.requestId);
```

### <a name="receive-messages-from-a-queue"></a>Réception des messages d'une file d'attente

Téléchargez les messages ajoutés en appelant la méthode [`receiveMessages`](/javascript/api/@azure/storage-queue/queueclient#receivemessages-queuereceivemessageoptions-). Dans le champ `numberOfMessages`, transmettez le nombre maximal de messages à recevoir pour cet appel.

Ajoutez ce code à la fin de la fonction `main` :

```javascript
console.log("\nReceiving messages from the queue...");

// Get messages from the queue
const receivedMessagesResponse = await queueClient.receiveMessages({ numberOfMessages : 5 });

console.log("Messages received, requestId:", receivedMessagesResponse.requestId);
```

### <a name="delete-messages-from-a-queue"></a>Supprimer des messages d’une file d’attente

Supprimez les messages de la file d’attente une fois qu’ils sont reçus et traités. Dans ce cas, le traitement affiche simplement le message sur la console.

Supprimez les messages en appelant la méthode [`deleteMessage`](/javascript/api/@azure/storage-queue/queueclient#deletemessage-string--string--queuedeletemessageoptions-). Tout message qui n’est pas supprimé explicitement sera de nouveau visible dans la file d’attente et pourra donc faire l’objet d’un nouveau traitement.

Ajoutez ce code à la fin de la fonction `main` :

```javascript
// 'Process' and delete messages from the queue
for (i = 0; i < receivedMessagesResponse.receivedMessageItems.length; i++) {
    receivedMessage = receivedMessagesResponse.receivedMessageItems[i];

    // 'Process' the message
    console.log("\tProcessing:", receivedMessage.messageText);

    // Delete the message
    const deleteMessageResponse = await queueClient.deleteMessage(
        receivedMessage.messageId,
        receivedMessage.popReceipt
    );
    console.log("\tMessage deleted, requestId:", deleteMessageResponse.requestId);
}
```

### <a name="delete-a-queue"></a>Suppression d'une file d'attente

Le code suivant nettoie les ressources créées par l’application en supprimant la file d’attente avec la méthode [`delete`](/javascript/api/@azure/storage-queue/queueclient#delete-queuedeleteoptions-).

Ajoutez ce code à la fin de la fonction `main` et enregistrez le fichier :

```javascript
// Delete the queue
console.log("\nDeleting queue...");
const deleteQueueResponse = await queueClient.delete();
console.log("Queue deleted, requestId:", deleteQueueResponse.requestId);
```

## <a name="run-the-code"></a>Exécuter le code

Cette application crée trois messages et les ajoute à une file d’attente Azure. Le code liste les messages dans la file d’attente, puis les récupère et les supprime avant de supprimer la file d’attente.

Dans la fenêtre de votre console, accédez au répertoire contenant le fichier `queues-quickstart-v12.js`, puis utilisez la commande `node` suivante pour exécuter l’application.

```console
node queues-quickstart-v12.js
```

La sortie de l’application ressemble à l’exemple suivant :

```output
Azure Queue Storage client library v12 - JavaScript quickstart sample

Creating queue...
         quickstartc095d120-1d04-11ea-af30-090ee231305f
Queue created, requestId: 5c0bc94c-6003-011b-7c11-b13d06000000

Adding messages to the queue...
Messages added, requestId: a0390321-8003-001e-0311-b18f2c000000

Peek at the messages in the queue...
         First message
         Second message
         Third message

Updating the third message in the queue...
Message updated, requestId: cb172c9a-5003-001c-2911-b18dd6000000

Receiving messages from the queue...
Messages received, requestId: a039036f-8003-001e-4811-b18f2c000000
        Processing: First message
        Message deleted, requestId: 4a65b82b-d003-00a7-5411-b16c22000000
        Processing: Second message
        Message deleted, requestId: 4f0b2958-c003-0030-2a11-b10feb000000
        Processing: Third message has been updated
        Message deleted, requestId: 6c978fcb-5003-00b6-2711-b15b39000000

Deleting queue...
Queue deleted, requestId: 5c0bca05-6003-011b-1e11-b13d06000000

Done
```

Parcourez le code dans le débogueur et vérifiez votre [portail Azure](https://portal.azure.com) tout au long du processus. Vérifiez votre compte de stockage pour vous assurer que les messages de la file d’attente sont créés et supprimés.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer une file d’attente et à y ajouter des messages à l’aide de code JavaScript. Ensuite, vous avez appris à afficher un aperçu des messages, à les récupérer et à les supprimer. Enfin, vous avez appris à supprimer une file d’attente de messages.

Pour obtenir des tutoriels, des exemples, des guides de démarrage rapide et d’autres documents, visitez :

> [!div class="nextstepaction"]
> [Documentation Azure pour JavaScript](/azure/developer/javascript/)

- Pour plus d’informations, consultez la [bibliothèque de client Stockage File d’attente Azure pour JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-queue).
- Pour d’autres exemples d’applications Stockage File d’attente Azure, consultez [Exemples de bibliothèques de client Stockage File d’attente Azure v12 pour JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-queue/samples).
