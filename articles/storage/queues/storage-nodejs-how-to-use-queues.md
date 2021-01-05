---
title: Comment utiliser Stockage File d’attente Azure à partir de Node.js – Stockage Azure
description: Apprenez à utiliser le service de File d’attente Azure pour créer et supprimer des files d’attente. En savoir plus sur l’insertion, l’extraction et la suppression de messages à l’aide de Node.js.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 08/31/2020
ms.service: storage
ms.subservice: queues
ms.topic: how-to
ms.reviewer: dineshm
ms.custom: seo-javascript-september2019, devx-track-js
ms.openlocfilehash: c5a9fb1a179164d24c84213762ee7e2332a1aa25
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93345939"
---
# <a name="how-to-use-azure-queue-storage-from-nodejs"></a>Guide pratique pour utiliser Stockage Files d’attente à partir de Node.js

[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

## <a name="overview"></a>Vue d’ensemble

Ce guide décrit le déroulement de scénarios courants dans le cadre de l’utilisation du service de File d’attente Microsoft Azure. Les exemples sont écrits en utilisant l'API Node.js. Les scénarios traités comprennent l’insertion, la consultation rapide, la récupération et la suppression de messages de la file d’attente. Découvrez également comment créer et supprimer des files d’attente.

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-nodejs-application"></a>Création d’une application Node.js

Pour créer une application Node.js vierge, consultez [Créer une application web Node.js dans Azure App Service][Create a Node.js web app in Azure App Service], [Création et déploiement d’une application Node.js dans un service cloud Azure][Build and deploy a Node.js application to an Azure Cloud Service] à l’aide de Windows PowerShell ou [Visual Studio Code][Visual Studio Code].

## <a name="configure-your-application-to-access-storage"></a>Configuration de votre application pour accéder au stockage

La [bibliothèque de client Stockage Azure pour JavaScript][Azure Storage client library for JavaScript] comprend un ensemble de bibliothèques pratiques qui communiquent avec les services REST de stockage.

### <a name="use-node-package-manager-npm-to-obtain-the-package"></a>Utilisation de Node Package Manager (NPM) pour obtenir le package

1. Utilisez une interface de ligne de commande telle que PowerShell (Windows), Terminal (Mac) ou Bash (Unix) pour accéder au dossier dans lequel vous avez créé votre exemple d’application.

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

1. Saisissez **npm install \@azure/storage-queue** dans la fenêtre de commande.

1. Vérifiez qu’un dossier **node\_modules** a été créé. Dans ce dossier, vous trouverez le package **\@azure/storage-queue** , qui contient la bibliothèque de client dont vous avez besoin pour accéder au stockage.

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

1. Tapez **npm install azure-storage** dans la fenêtre de commande.

1. Vérifiez qu’un dossier **node\_modules** a été créé. Dans ce dossier, vous trouverez le package **azure-storage** , qui contient les bibliothèques dont vous avez besoin pour accéder au stockage.

---

### <a name="import-the-package"></a>Importation du package

À l’aide de votre éditeur de code, ajoutez le code suivant au début du fichier JavaScript dans lequel vous avez l’intention d’utiliser des files d’attente.

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_ImportStatements":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

```javascript
var azure = require('azure-storage');
```

---

## <a name="how-to-create-a-queue"></a>Création d’une file d’attente

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

Le code suivant obtient la valeur d’une variable d’environnement appelée `AZURE_STORAGE_CONNECTION_STRING` et l’utilise pour créer un objet [QueueServiceClient](/javascript/api/@azure/storage-queue/queueserviceclient). L’objet **QueueServiceClient** est ensuite utilisé pour créer un objet [QueueClient](/javascript/api/@azure/storage-queue/queueclient). L’objet **QueueClient** vous permet d’utiliser une file d’attente spécifique.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_CreateQueue":::

Si la file d’attente existe déjà, une exception est levée.

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Le module Azure lit les variables d’environnement `AZURE_STORAGE_ACCOUNT` et `AZURE_STORAGE_ACCESS_KEY` ou `AZURE_STORAGE_CONNECTION_STRING` pour obtenir les informations nécessaires à la connexion à votre compte de stockage Azure. Si ces variables d’environnement ne sont pas définies, vous devez spécifier les informations de compte lors de l’appel de **createQueueService**.

Le code suivant crée un objet **QueueService** , ce qui vous permet d'utiliser les files d'attente.

```javascript
var queueSvc = azure.createQueueService();
```

Appelez la méthode **createQueueIfNotExists** pour créer une nouvelle file d’attente avec le nom spécifié ou pour retourner la file d’attente si elle existe déjà.

```javascript
queueSvc.createQueueIfNotExists('myqueue', function(error, results, response){
  if(!error){
    // Queue created or exists
  }
});
```

Si la file d’attente est créée, `result.created` a la valeur true. Si la file d’attente existe, `result.created` a la valeur false.

---

## <a name="how-to-insert-a-message-into-a-queue"></a>Insertion d'un message dans une file d'attente

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

Pour ajouter un message à une file d’attente, appelez la méthode [sendMessage](/javascript/api/@azure/storage-queue/queueclient#sendmessage-string--queuesendmessageoptions-).

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_AddMessage":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Pour insérer un message dans une file d’attente, appelez la méthode **createMessage** afin de créer un message et de l’ajouter à la file d’attente.

```javascript
queueSvc.createMessage('myqueue', "Hello world!", function(error, results, response){
  if(!error){
    // Message inserted
  }
});
```

---

## <a name="how-to-peek-at-the-next-message"></a>Lecture furtive du message suivant

Vous pouvez lire furtivement les messages dans la file d’attente sans les supprimer de la file d’attente en appelant la méthode **PeekMessages**.

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

Par défaut, [peekMessages](/javascript/api/@azure/storage-queue/queueclient#peekmessages-queuepeekmessagesoptions-) lit un seul message. L’exemple suivant lit les cinq premiers messages de la file d’attente. Si moins de cinq messages sont visibles, seuls les messages visibles sont retournés.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_PeekMessage":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Par défaut, **peekMessages** lit un seul message.

```javascript
queueSvc.peekMessages('myqueue', function(error, results, response){
  if(!error){
    // Message text is in results[0].messageText
  }
});
```

`result` contient le message.

---

L’appel de **peekMessages** lorsqu’il n’y a aucun message dans la file d’attente ne retourne pas d’erreur. Toutefois, aucun message n’est retourné.

## <a name="how-to-change-the-contents-of-a-queued-message"></a>Modification du contenu d’un message en file d’attente

L’exemple suivant met à jour le texte d’un message.

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

Modifiez le contenu d’un message qui se trouve dans la file d’attente en utilisant [updateMessage](/javascript/api/@azure/storage-queue/queueclient#updatemessage-string--string--string--number--queueupdatemessageoptions-).

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_UpdateMessage":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Modifiez le contenu d’un message qui se trouve dans la file d’attente en utilisant **updateMessage**.

```javascript
queueSvc.getMessages('myqueue', function(error, getResults, getResponse){
  if(!error){
    // Got the message
    var message = getResults[0];
    queueSvc.updateMessage('myqueue', message.messageId, message.popReceipt, 10, {messageText: 'new text'}, function(error, updateResults, updateResponse){
      if(!error){
        // Message updated successfully
      }
    });
  }
});
```

---

## <a name="how-to-dequeue-a-message"></a>Retrait des messages de la file d’attente

La suppression d’un message de la file d’attente se fait en deux étapes :

1. Récupérer le message.

1. Supprimer le message.

L’exemple suivant récupère un message, puis le supprime.

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

Pour récupérer un message, appelez la méthode [receiveMessages](/javascript/api/@azure/storage-queue/queueclient#receivemessages-queuereceivemessageoptions-). Cet appel rend les messages invisibles dans la file d’attente, et aucun autre client ne peut les traiter. Lorsque votre application a traité un message, appelez [deleteMessage](/javascript/api/@azure/storage-queue/queueclient#deletemessage-string--string--queuedeletemessageoptions-) pour supprimer le message de la file d’attente.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_DequeueMessage":::

Par défaut, un message est masqué uniquement pendant 30 secondes. Après 30 secondes, il est visible par les autres clients. Vous pouvez spécifier une valeur différente en définissant [options.visibilityTimeout](/javascript/api/@azure/storage-queue/queuereceivemessageoptions#visibilitytimeout) lorsque vous appelez **receiveMessages**.

L’appel de **receiveMessages** lorsqu’il n’y a aucun message dans la file d’attente ne retourne pas d’erreur. Toutefois, aucun message n’est retourné.

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Pour récupérer un message, appelez la méthode **getMessages**. Cet appel rend les messages invisibles dans la file d’attente, et aucun autre client ne peut les traiter. Lorsque votre application a traité un message, appelez **deleteMessage** pour supprimer le message de la file d’attente.

```javascript
queueSvc.getMessages('myqueue', function(error, results, response){
  if(!error){
    // Message text is in results[0].messageText
    var message = results[0];
    queueSvc.deleteMessage('myqueue', message.messageId, message.popReceipt, function(error, response){
      if(!error){
        //message deleted
      }
    });
  }
});
```

Par défaut, un message est masqué uniquement pendant 30 secondes. Après 30 secondes, il est visible par les autres clients. Vous pouvez indiquer une autre valeur en utilisant `options.visibilityTimeout` avec **getMessages**.

L’utilisation de **getMessages** lorsqu’il n’y a aucun message dans la file d’attente ne retourne pas d’erreur. Toutefois, aucun message n’est retourné.

---

## <a name="additional-options-for-dequeuing-messages"></a>Options supplémentaires pour la suppression des messages dans la file d'attente

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

Il existe deux méthodes pour personnaliser l'extraction d'un message d'une file d'attente :

- [options.numberOfMessages](/javascript/api/@azure/storage-queue/queuereceivemessageoptions#numberofmessages) : récupérer un lot de messages (jusqu’à 32).
- [options.visibilityTimeout](/javascript/api/@azure/storage-queue/queuereceivemessageoptions#visibilitytimeout) : définir une durée d’invisibilité plus longue ou plus courte.

L’exemple suivant utilise la méthode **receiveMessages** pour obtenir cinq messages en un appel. Ensuite, il traite chaque message à l'aide d'une boucle `for`. La durée d'invisibilité est passée à cinq minutes pour tous les messages renvoyés par cette méthode.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_DequeueMessages":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Il existe deux méthodes pour personnaliser l'extraction d'un message d'une file d'attente :

- `options.numOfMessages` - Extraire un lot de messages (jusqu’à 32).
- `options.visibilityTimeout` - Définir une durée d’invisibilité plus longue ou plus courte.

L'exemple suivant utilise la méthode **getMessages** pour obtenir 15 messages dans un appel. Ensuite, il traite chaque message à l'aide d'une boucle `for`. La durée d'invisibilité est passée à cinq minutes pour tous les messages renvoyés par cette méthode.

```javascript
queueSvc.getMessages('myqueue', {numOfMessages: 15, visibilityTimeout: 5 * 60}, function(error, results, getResponse){
  if(!error){
    // Messages retrieved
    for(var index in results){
      // text is available in result[index].messageText
      var message = results[index];
      queueSvc.deleteMessage(queueName, message.messageId, message.popReceipt, function(error, deleteResponse){
        if(!error){
          // Message deleted
        }
      });
    }
  }
});
```

---

## <a name="how-to-get-the-queue-length"></a>Obtention de la longueur de la file d’attente

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

La méthode [getProperties](/javascript/api/@azure/storage-queue/queueclient#getproperties-queuegetpropertiesoptions-) renvoie des métadonnées sur la file d’attente, notamment le nombre approximatif de messages en attente dans la file d’attente.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_QueueLength":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

La méthode **getQueueMetadata** renvoie des métadonnées sur la file d’attente, notamment le nombre approximatif de messages en attente dans la file d’attente.

```javascript
queueSvc.getQueueMetadata('myqueue', function(error, results, response){
  if(!error){
    // Queue length is available in results.approximateMessageCount
  }
});
```

---

## <a name="how-to-list-queues"></a>Liste des files d’attente

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

Pour récupérer une liste de files d’attente, appelez [QueueServiceClient.listQueues](). Pour récupérer une liste filtrée à l’aide d’un préfixe spécifique, définissez [options.prefix](/javascript/api/@azure/storage-queue/servicelistqueuesoptions#prefix) dans votre appel à **listQueues**.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_ListQueues":::

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Pour extraire une liste de files d'attente, utilisez **listQueuesSegmented**. Pour extraire une liste filtrée par un certain préfixe, utilisez **listQueuesSegmentedWithPrefix**.

```javascript
queueSvc.listQueuesSegmented(null, function(error, results, response){
  if(!error){
    // results.entries contains the list of queues
  }
});
```

Si aucune file d’attente ne peut être renvoyée, transmettez `result.continuationToken` comme premier paramètre de **listQueuesSegmented** ou second paramètre de **listQueuesSegmentedWithPrefix** pour récupérer plus de résultats.

---

## <a name="how-to-delete-a-queue"></a>Suppression d’une file d’attente

# <a name="javascript-v12"></a>[JavaScript v12](#tab/javascript)

Pour supprimer une file d’attente et tous les messages qu’elle contient, appelez la méthode [deleteQueue](/javascript/api/@azure/storage-queue/queueclient#delete-queuedeleteoptions-) sur l’objet **QueueClient**.

:::code language="javascript" source="~/azure-storage-snippets/queues/howto/JavaScript/JavaScript-v12/javascript-queues-v12.js" id="Snippet_DeleteQueue":::

Pour effacer tous les messages d’une file d’attente sans supprimer cette dernière, appelez [clearMessages](/javascript/api/@azure/storage-queue/queueclient#clearmessages-queueclearmessagesoptions-).

# <a name="javascript-v2"></a>[JavaScript v2](#tab/javascript2)

Pour supprimer une file d'attente et tous les messages qu'elle contient, appelez la méthode **deleteQueue** sur l'objet file d'attente.

```javascript
queueSvc.deleteQueue(queueName, function(error, response){
  if(!error){
    // Queue has been deleted
  }
});
```

Pour effacer tous les messages d’une file d’attente sans supprimer cette dernière, appelez **clearMessages**.

---

[!INCLUDE [storage-check-out-samples-all](../../../includes/storage-check-out-samples-all.md)]

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous connaissez les bases du stockage des files d'attente, consultez les liens suivants pour apprendre à exécuter les tâches de stockage plus complexes.

- Consultez le [blog de l’équipe Azure Storage][Azure Storage Team Blog] pour découvrir les nouveautés.
- Consultez le référentiel relatif à la [bibliothèque de client Stockage Azure pour JavaScript][Azure Storage client library for JavaScript] sur GitHub.

[Azure Storage client library for JavaScript]: https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage#azure-storage-client-library-for-javascript
[Azure Storage Team Blog]: https://techcommunity.microsoft.com/t5/azure-storage/bg-p/AzureStorageBlog
[Build and deploy a Node.js application to an Azure Cloud Service]: ../../cloud-services/cloud-services-nodejs-develop-deploy-app.md
[Create a Node.js web app in Azure App Service]: ../../app-service/quickstart-nodejs.md
[Visual Studio Code]: https://code.visualstudio.com/docs/nodejs/nodejs-tutorial
