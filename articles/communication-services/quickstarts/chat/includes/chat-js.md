---
title: Fichier include
description: Fichier include
services: azure-communication-services
author: mikben
manager: mikben
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 9/1/2020
ms.topic: include
ms.custom: include file
ms.author: mikben
ms.openlocfilehash: d0754ea2d7e8f8f59ec475be8e27fcffd058c11f
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91376399"
---
## <a name="prerequisites"></a>Prérequis
Avant de commencer, assurez-vous de :

- Créer un compte Azure avec un abonnement actif. Pour plus d’informations, consultez [Créer un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Installer les versions Active LTS et Maintenance LTS de [Node.js](https://nodejs.org/en/download/) (versions 8.11.1 et 10.14.1 recommandées).
- Créer une ressource Azure Communication Services. Pour plus d’informations, consultez [Créer une ressource Azure Communication](../../create-communication-resource.md). Vous devrez inscrire le **point de terminaison** de votre ressource pour ce démarrage rapide.
- [Jeton d’accès utilisateur](../../access-tokens.md). Veillez à définir l’étendue sur « chat » (conversation) et notez la chaîne token ainsi que la chaîne userId.

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-web-application"></a>Créer une application web

Pour commencer, ouvrez votre terminal ou votre fenêtre Commande pour créer un répertoire pour votre application, puis accédez-y.

```console
mkdir chat-quickstart && cd chat-quickstart
```
   
Exécutez `npm init -y` pour créer un fichier **package.json** avec les paramètres par défaut.

```console
npm init -y
```

Utilisez un éditeur de texte pour créer un fichier appelé **start-chat.js** dans le répertoire racine du projet. Dans les sections suivantes, vous ajouterez à ce fichier l’ensemble du code source de ce démarrage rapide.

### <a name="install-the-packages"></a>Installer les packages

Utilisez la commande `npm install` pour installer les bibliothèques de client Communication Services pour JavaScript ci-dessous.

```console
npm install @azure/communication-common --save

npm install @azure/communication-administration --save

npm install @azure/communication-signaling --save

npm install @azure/communication-chat --save

```

L’option `--save` liste la bibliothèque comme dépendance dans votre fichier **package.json**.

### <a name="set-up-the-app-framework"></a>Configurer le framework d’application

Ce démarrage rapide utilise webpack pour regrouper les ressources de l’application. Exécutez la commande suivante pour installer les packages npm webpack, webpack-cli et webpack-dev-server, puis listez-les comme dépendances de développement dans votre **package.json** :

```console
npm install webpack webpack-cli webpack-dev-server --save-dev
```

Créez un fichier **index.html** dans le répertoire racine de votre projet. Nous nous servirons de ce fichier de modèle pour ajouter la fonctionnalité de conversation (chat) avec la bibliothèque de client Azure Communication Chat pour JavaScript.

Voici le code :

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Communication Client - Chat Sample</title>
  </head>
  <body>
    <h4>Azure Communication Services</h4>
    <h1>Chat Quickstart</h1>
    <script src="./bundle.js"></script>
  </body>
</html>
```
Créez un fichier dans le répertoire racine de votre projet sous le nom **client.js** qui contiendra la logique d’application pour ce démarrage rapide. 

### <a name="create-a-chat-client"></a>Créer un client de conversation

Pour créer un client de conversation dans votre application web, vous allez utiliser le point de terminaison Communication Services ainsi que le jeton d’accès qui a été généré au cours des étapes prérequises. Les jetons d’accès utilisateur vous permettent de créer des applications clientes qui s’authentifient directement auprès d’Azure Communication Services. Une fois que vous avez généré ces jetons sur votre serveur, transmettez-les en retour à un appareil client. Vous devez utiliser la classe `AzureCommunicationUserCredential` de la `Common client library` pour transmettre le jeton à votre client de conversation.

Créez un fichier **client.js** dans le répertoire racine de votre projet. Nous nous servirons de ce fichier pour ajouter la fonctionnalité de conversation (chat) à l’aide de la bibliothèque de client Azure Communication Chat pour JavaScript.

```JavaScript

import { ChatClient } from '@azure/communication-chat';
import { AzureCommunicationUserCredential } from '@azure/communication-common';

// Your unique Azure Communication service endpoint
let endpointUrl = 'https://<RESOURCE_NAME>.communication.azure.com';
let userAccessToken = '<USER_ACCESS_TOKEN>';

let chatClient = new ChatClient(endpointUrl, new AzureCommunicationUserCredential(userAccessToken));
console.log('Azure Communication Chat client created!');
```
Remplacez **ENDPOINT** par le point de terminaison qui a créé précédemment selon les instructions contenues dans la documentation [Créer une ressource Azure Communication](../../create-communication-resource.md).
Remplacez **USER_ACCESS_TOKEN** par un jeton émis selon les instructions contenues dans la documentation [Jeton d’accès utilisateur](../../access-tokens.md).
Ajoutez ce code au fichier **client.js**.


### <a name="run-the-code"></a>Exécuter le code
Utilisez `webpack-dev-server` pour créer et exécuter votre application. Exécutez la commande suivante pour regrouper l’hôte d’application sur un serveur web local :
```console
npx webpack-dev-server --entry ./client.js --output bundle.js --debug --devtool inline-source-map
```
Ouvrez votre navigateur et accédez à http://localhost:8080/.
Dans la console d’outils de développement de votre navigateur, vous devez voir les informations suivantes :

```console
Azure Communication Chat client created!
```

## <a name="object-model"></a>Modèle objet 
Les classes et interfaces suivantes gèrent quelques-unes des principales fonctionnalités de la bibliothèque de client Azure Communication Services Chat pour JavaScript.

| Nom                                   | Description                                                                                                                                                                           |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ChatClient | Cette classe est nécessaire à la fonctionnalité de conversation (Chat). Vous l’instanciez avec vos informations d’abonnement et l’utilisez pour créer, obtenir et supprimer des fils de conversation. |
| ChatThreadClient | Cette classe est nécessaire à la fonctionnalité de fil de conversation (Chat Thread). Vous obtenez une instance via ChatClient et l’utilisez pour envoyer/recevoir/mettre à jour/supprimer des messages, ajouter/supprimer/obtenir des utilisateurs, envoyer des notifications de saisie et des accusés de lecture, souscrire à des événements de conversation. |


## <a name="start-a-chat-thread"></a>Démarrer un fil de conversation

Utilisez la méthode `createThread` pour créer un fil de conversation.

`createThreadRequest` est utilisé pour décrire la demande de fil :

- Utilisez `topic` pour attribuer un sujet à cette conversation ; le sujet peut être mis à jour après que le fil de conversation a été créé à l’aide de la fonction `UpdateThread`. 
- Utilisez `members` pour lister les membres à ajouter au fil de conversation ;

Une fois résolue, la méthode `createChatThread` retourne `threadId` qui est utilisé dans les opérations effectuées sur le fil de conversation nouvellement créé, comme l’ajout de membres au fil de conversation, l’envoi d’un message, la suppression d’un message, etc.

```Javascript
async function createChatThread() {
   let createThreadRequest = {
       topic: 'Preparation for London conference',
       members: [{
                   user: { communicationUserId: '<USER_ID_FOR_JACK>' },
                   displayName: 'Jack'
               }, {
                   user: { communicationUserId: '<USER_ID_FOR_GEETA>' },
                   displayName: 'Geeta'
               }]
   };
   let chatThreadClient= await chatClient.createChatThread(createThreadRequest);
   let threadId = chatThreadClient.threadId;
   return threadId;
}

createChatThread().then(async threadId => {
   console.log(`Thread created:${threadId}`);
   // PLACEHOLDERS
   // <CREATE CHAT THREAD CLIENT>
   // <RECEIVE A CHAT MESSAGE FROM A CHAT THREAD>
   // <SEND MESSAGE TO A CHAT THREAD>
   // <LIST MESSAGES IN A CHAT THREAD>
   // <ADD NEW MEMBER TO THREAD>
   // <LIST MEMBERS IN A THREAD>
   // <REMOVE MEMBER FROM THREAD>
});
```

Remplacez **USER_ID_FOR_JACK** et **USER_ID_FOR_GEETA** par les ID d’utilisateurs obtenus à l’étape précédente (Créer des utilisateurs et émettre des [jetons d’accès utilisateur](../../access-tokens.md)).

Quand vous actualisez l’onglet du navigateur, la console doit présenter les éléments suivants
```console
Thread created: <threadId>
```

## <a name="get-a-chat-thread-client"></a>Obtenir un client de fil de conversation

La méthode `getChatThreadClient` retourne un `chatThreadClient` pour un fil qui existe déjà. Il peut être utilisé dans les opérations effectuées sur le fil créé : ajout de membres, envoi d’un message, etc. threadId est l’ID unique du fil de conversation existant.

```JavaScript

let chatThreadClient = await chatClient.getChatThreadClient(threadId);
console.log(`Chat Thread client for threadId:${chatThreadClient.threadId}`);

```
Ajoutez ce code à la place du commentaire `<CREATE CHAT THREAD CLIENT>` dans **client.js**, actualisez l’onglet du navigateur et examinez la console, qui doit présenter les éléments suivants :
```console
Chat Thread client for threadId: <threadId>
```

## <a name="send-a-message-to-a-chat-thread"></a>Envoyer un message à un fil de conversation

Utilisez la méthode `sendMessage` pour envoyer un message de conversation au fil que vous venez de créer, identifié par threadId.

`sendMessageRequest` décrit les champs obligatoires de la demande de message de conversation :

- Utilisez `content` pour fournir le contenu du message de conversation ;

`sendMessageOptions` décrit les champs facultatifs de la demande de message de conversation :

- Utilisez `priority` pour spécifier le niveau de priorité du message de conversation, par exemple « Normal » ou « High » (Élevé) ; cette propriété peut être utilisée pour que l’indicateur d’interface utilisateur attire l’attention de l’utilisateur destinataire sur le message ou pour exécuter une logique métier personnalisée.   
- Utilisez `senderDisplayName` pour spécifier le nom d’affichage de l’expéditeur ;

La réponse `sendChatMessageResult` contient un « ID », qui est l’ID unique de ce message.

```JavaScript

let sendMessageRequest =
{
    content: 'Hello Geeta! Can you share the deck for the conference?'
};
let sendMessageOptions =
{
    priority: 'Normal',
    senderDisplayName : 'Jack'
};
let sendChatMessageResult = await chatThreadClient.sendMessage(sendMessageRequest, sendMessageOptions);
let messageId = sendChatMessageResult.id;
console.log(`Message sent!, message id:${messageId}`);

```
Ajoutez ce code à la place du commentaire `<SEND MESSAGE TO A CHAT THREAD>` dans **client.js**, actualisez l’onglet du navigateur et examinez la console.
```console
Message sent!, message id:<number>
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Recevoir les messages de conversation d’un fil de conversation

Avec la signalisation en temps réel, vous pouvez vous abonner pour écouter les nouveaux messages entrants et mettre à jour en conséquence les messages actuellement en mémoire. Azure Communication Services prend en charge une [liste d’événements auxquels vous pouvez vous abonner](../../../concepts/chat/concepts.md#real-time-signaling).

```JavaScript
// open notifications channel
await chatClient.startRealtimeNotifications();
// subscribe to new notification
chatClient.on("chatMessageReceived", (e) => {
    console.log("Notification chatMessageReceived!");
    // your code here
});

```
Ajoutez ce code à la place du commentaire `<RECEIVE A CHAT MESSAGE FROM A CHAT THREAD>` dans **client.js**.
Si vous actualisez l’onglet du navigateur, la console doit présenter un message `Notification chatMessageReceived` ;

Vous pouvez aussi récupérer les messages de conversation en interrogeant la méthode `listMessages` selon des intervalles définis. 

```JavaScript

let pagedAsyncIterableIterator = await chatThreadClient.listMessages();
let nextMessage = await pagedAsyncIterableIterator.next();
 while (!nextMessage.done) {
     let chatMessage = nextMessage.value;
     console.log(`Message :${chatMessage.content}`);
     // your code here
     nextMessage = await pagedAsyncIterableIterator.next();
 }

```
Ajoutez ce code à la place du commentaire `<LIST MESSAGES IN A CHAT THREAD>` dans **client.js**.
Si vous actualisez l’onglet, la liste des messages envoyés dans ce fil de conversation doit figurer dans la console.


`listMessages` retourne la version la plus récente du message, avec les modifications ou les suppressions dont le message a éventuellement fait l’objet via `updateMessage` et `deleteMessage`.
Pour les messages supprimés, `chatMessage.deletedOn` retourne une valeur datetime indiquant à quel moment ce message a été supprimé. Pour les messages modifiés, `chatMessage.editedOn` retourne une valeur datetime indiquant à quel moment ce message a été modifié. Il est possible d’accéder à l’heure initiale de création du message à l’aide de `chatMessage.createdOn` qui peut être utilisée à des fins de classement des messages.

`listMessages` retourne différents types de messages qui peuvent être identifiés par `chatMessage.type`. Ces types sont les suivants :

- `Text`: Message de conversation ordinaire envoyé par un membre du fil.

- `ThreadActivity/TopicUpdate`: Message système qui indique que le sujet a été mis à jour.

- `ThreadActivity/AddMember`: Message système qui indique qu’un ou plusieurs membres ont été ajoutés au fil de conversation.

- `ThreadActivity/RemoveMember`: Message système qui indique qu’un membre a été supprimé du fil de conversation.

Pour plus d’informations, consultez [Types de messages](../../../concepts/chat/concepts.md#message-types).

## <a name="add-a-user-as-member-to-the-chat-thread"></a>Ajouter un utilisateur en tant que membre du fil de conversation

Une fois qu’un fil de conversation est créé, vous pouvez y ajouter des utilisateurs et en supprimer. En ajoutant des utilisateurs, vous leur permettez d’envoyer des messages au fil de conversation et d’ajouter/supprimer d’autres membres. Avant d’appeler la méthode `addMembers`, vérifiez que vous avez acquis un nouveau jeton d’accès et une identité pour cet utilisateur. L’utilisateur aura besoin de ce jeton d’accès pour initialiser son client de conversation.

`addMembersRequest` décrit l’objet de demande dans lequel `members` liste les membres à ajouter au fil de conversation ;
- `user`, obligatoire, est l’utilisateur de communication à ajouter au fil de conversation.
- `displayName`, facultatif, est le nom d’affichage du membre du fil.
- `shareHistoryTime`, facultatif, est le moment à partir duquel l’historique de conversation est partagé avec le membre. Pour partager l’historique depuis le début du fil de conversation, attribuez à cette propriété une date égale ou antérieure à la date de création du fil. Pour ne partager aucun historique antérieur au moment où le membre a été ajouté, attribuez-lui l’heure actuelle. Pour partager l’historique partiel, attribuez-lui la date de votre choix.

```JavaScript

let addMembersRequest =
{
    members: [
        {
            user: { communicationUserId: '<NEW_MEMBER_USER_ID>' },
            displayName: 'Jane'
        }
    ]
};

await chatThreadClient.addMembers(addMembersRequest);

```
Remplacez **NEW_MEMBER_USER_ID** par un [nouvel ID d’utilisateur](../../access-tokens.md). Ajoutez ce code à la place du commentaire `<ADD NEW MEMBER TO THREAD>` dans **client.js**.

## <a name="list-users-in-a-chat-thread"></a>Lister les utilisateurs dans un fil de conversation
```JavaScript
async function listThreadMembers() {
   let pagedAsyncIterableIterator = await chatThreadClient.listMembers();
   let next = await pagedAsyncIterableIterator.next();
   while (!next.done) {
      let user = next.value;
      console.log(`User :${user.displayName}`);
      next = await pagedAsyncIterableIterator.next();
   }
}
await listThreadMembers();
```
Ajoutez ce code à la place du commentaire `<LIST MEMBERS IN A THREAD>` dans **client.js**, actualisez l’onglet du navigateur et examinez la console, qui doit afficher des informations sur les utilisateurs présents dans un fil.

## <a name="remove-user-from-a-chat-thread"></a>Supprimer un utilisateur d’un fil de conversation

De la même manière que vous pouvez ajouter un membre, vous pouvez supprimer des membres d’un fil de conversation. Pour ce faire, vous devez suivre les ID des membres que vous avez ajoutés.

Utilisez la méthode `removeMember` où `member` est l’utilisateur de communication à supprimer du fil.

```JavaScript

await chatThreadClient.removeMember({ communicationUserId: <MEMBER_ID> });
await listThreadMembers();
```
Remplacez **MEMBER_ID** par un ID d’utilisateur utilisé à l’étape précédente (<NEW_MEMBER_USER_ID>).
Ajoutez ce code à la place du commentaire `<REMOVE MEMBER FROM THREAD>` dans **client.js**,
