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
ms.openlocfilehash: a76c6467dac69fd3d21aa659c52227046c166938
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94816550"
---
## <a name="prerequisites"></a>Prérequis
Avant de commencer, assurez-vous de :
- Créer un compte Azure avec un abonnement actif. Pour plus d’informations, consultez [Créer un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Installez [Visual Studio](https://visualstudio.microsoft.com/downloads/) 
- Créer une ressource Azure Communication Services. Pour plus d’informations, consultez [Créer une ressource Azure Communication](../../create-communication-resource.md). Vous devrez inscrire le **point de terminaison** de votre ressource pour ce démarrage rapide.
- [Jeton d’accès utilisateur](../../access-tokens.md). Veillez à définir l’étendue sur « chat » (conversation) et notez la chaîne token ainsi que la chaîne userId.

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-c-application"></a>Créer une application C#

Dans une fenêtre de console (par exemple cmd, PowerShell ou Bash), utilisez la commande `dotnet new` pour créer une application console avec le nom `ChatQuickstart`. Cette commande crée un projet C# « Hello World » simple avec un seul fichier source : **Program.cs**.

```console
dotnet new console -o ChatQuickstart
```

Remplacez votre répertoire par le dossier d’application que vous venez de créer et utilisez la commande `dotnet build` pour compiler votre application.

```console
cd ChatQuickstart
dotnet build
```

### <a name="install-the-package"></a>Installer le package

Installer la bibliothèque de client Azure Communication Chat pour .NET

```PowerShell
dotnet add package Azure.Communication.Chat --version 1.0.0-beta.3
``` 

## <a name="object-model"></a>Modèle objet

Les classes suivantes gèrent quelques-unes des principales fonctionnalités de la bibliothèque de client Azure Communication Services Chat pour C#.

| Nom                                  | Description                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| ChatClient | Cette classe est nécessaire à la fonctionnalité de conversation (Chat). Vous l’instanciez avec vos informations d’abonnement et l’utilisez pour créer, obtenir et supprimer des fils de conversation. |
| ChatThreadClient | Cette classe est nécessaire à la fonctionnalité de fil de conversation (Chat Thread). Vous obtenez une instance via ChatClient et l’utilisez pour envoyer/recevoir/mettre à jour/supprimer des messages, ajouter/supprimer/obtenir des utilisateurs, envoyer des notifications de saisie et des accusés de lecture. |

## <a name="create-a-chat-client"></a>Créer un client de conversation

Pour créer un client de conversation, vous allez utiliser votre point de terminaison Communication Services ainsi que le jeton d’accès qui a été généré au cours des étapes prérequises. Vous devez utiliser la classe `CommunicationIdentityClient` de la bibliothèque de client `Administration` pour créer un utilisateur et émettre un jeton à transmettre à votre client de conversation. Apprenez-en davantage sur les [jetons d’accès utilisateur](../../access-tokens.md).

```csharp
using Azure.Communication.Identity;
using Azure.Communication.Chat;

// Your unique Azure Communication service endpoint
Uri endpoint = new Uri("https://<RESOURCE_NAME>.communication.azure.com");

CommunicationUserCredential communicationUserCredential = new CommunicationUserCredential(<Access_Token>);
ChatClient chatClient = new ChatClient(endpoint, communicationUserCredential);
```

## <a name="start-a-chat-thread"></a>Démarrer un fil de conversation

Utilisez la méthode `createChatThread` pour créer un fil de conversation.
- Utilisez `topic` pour attribuer un sujet à cette conversation ; le sujet peut être mis à jour après que le fil de conversation a été créé à l’aide de la fonction `UpdateThread`.
- Utilisez la propriété `members` pour transmettre la liste d’objets `ChatThreadMember` à ajouter au fil de conversation. L’objet `ChatThreadMember` est initialisé avec un objet `CommunicationUser`. Pour obtenir un objet `CommunicationUser`, vous devez transmettre un ID d’accès que vous avez créé en suivant les instructions fournies dans [Créer un utilisateur](../../access-tokens.md#create-an-identity).

La réponse `chatThreadClient` est utilisée dans les opérations effectuées sur le fil de conversation créé : ajout de membres au fil de conversation, envoi d’un message, suppression d’un message, etc. Elle contient l’attribut `Id` qui est l’ID unique du fil de conversation. 

```csharp
var chatThreadMember = new ChatThreadMember(new CommunicationUser("<Access_ID>"))
{
    DisplayName = "UserDisplayName"
};
ChatThreadClient chatThreadClient = await chatClient.CreateChatThreadAsync(topic: "Chat Thread topic C# sdk", members: new[] { chatThreadMember });
string threadId = chatThreadClient.Id;
```

## <a name="get-a-chat-thread-client"></a>Obtenir un client de fil de conversation
La méthode `GetChatThreadClient` retourne un client de fil pour un fil qui existe déjà. Il peut être utilisé dans les opérations effectuées sur le fil créé : ajout de membres, envoi d’un message, etc. threadId est l’ID unique du fil de conversation existant.

```csharp
string threadId = "<THREAD_ID>";
ChatThreadClient chatThreadClient = chatClient.GetChatThreadClient(threadId);
```

## <a name="send-a-message-to-a-chat-thread"></a>Envoyer un message à un fil de conversation

Utilisez la méthode `SendMessage` pour envoyer un message à un fil identifié par threadId.

- Utilisez `content` pour fournir le contenu du message de conversation (obligatoire).
- Utilisez `priority` pour spécifier le niveau de priorité du message, par exemple « Normal » ou « High » (élevé). S’il n’est pas spécifié, « Normal » est utilisé.
- Utilisez `senderDisplayName` pour spécifier le nom d’affichage de l’expéditeur. S’il n’est pas spécifié, un nom vide est utilisé.

`SendChatMessageResult` est la réponse retournée à la suite de l’envoi du message; elle contient un ID, qui est l’ID unique du message.

```csharp
var content = "hello world";
var priority = ChatMessagePriority.Normal;
var senderDisplayName = "sender name";

SendChatMessageResult sendChatMessageResult = await chatThreadClient.SendMessageAsync(content, priority, senderDisplayName);
string messageId = sendChatMessageResult.Id;
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Recevoir les messages de conversation d’un fil de conversation

Vous pouvez récupérer les messages de conversation en interrogeant la méthode `GetMessages` sur le client de fil de conversation selon des intervalles définis.

```csharp
AsyncPageable<ChatMessage> allMessages = chatThreadClient.GetMessagesAsync();
await foreach (ChatMessage message in allMessages)
{
    Console.WriteLine($"{message.Id}:{message.Sender.Id}:{message.Content}");
}
```

`GetMessages` accepte un paramètre `DateTimeOffset` facultatif. Si ce décalage est spécifié, vous recevez les messages qui ont été reçus, mis à jour ou supprimés après cette période. Notez que les messages reçus avant la période de décalage, mais qui ont été modifiés ou supprimés après ladite période sont également retournés.

`GetMessages` retourne la version la plus récente du message, avec les modifications ou les suppressions dont le message a éventuellement fait l’objet via `UpdateMessage` et `DeleteMessage`. Pour les messages supprimés, `chatMessage.DeletedOn` retourne une valeur datetime indiquant à quel moment ce message a été supprimé. Pour les messages modifiés, `chatMessage.EditedOn` retourne une valeur datetime indiquant à quel moment ce message a été modifié. Il est possible d’accéder à l’heure initiale de création du message à l’aide de `chatMessage.CreatedOn` ; elle peut être utilisée à des fins de classement des messages.

`GetMessages` retourne différents types de messages qui peuvent être identifiés par `chatMessage.Type`. Ces types sont les suivants :

- `Text`: Message de conversation ordinaire envoyé par un membre du fil.

- `ThreadActivity/TopicUpdate`: Message système qui indique que le sujet a été mis à jour.

- `ThreadActivity/AddMember`: Message système qui indique qu’un ou plusieurs membres ont été ajoutés au fil de conversation

- `ThreadActivity/DeleteMember`: Message système qui indique qu’un membre a été supprimé du fil de conversation

Pour plus d’informations, consultez [Types de message](../../../concepts/chat/concepts.md#message-types).

## <a name="update-a-message"></a>Mettre à jour un message

Vous pouvez mettre à jour un message qui a déjà été envoyé en appelant `UpdateMessage` sur `ChatThreadClient`.

```csharp
string id = "id-of-message-to-edit";
string content = "updated content";
await chatThreadClient.UpdateMessageAsync(id, content);
```

## <a name="deleting-a-message"></a>Suppression d’un message

Vous pouvez supprimer un message en appelant `DeleteMessage` sur `ChatThreadClient`.

```csharp
string id = "id-of-message-to-delete";
await chatThreadClient.DeleteMessageAsync(id);
```

## <a name="add-a-user-as-member-to-the-chat-thread"></a>Ajouter un utilisateur en tant que membre du fil de conversation

Une fois qu’un fil est créé, vous pouvez y ajouter des utilisateurs ou en supprimer. En ajoutant des utilisateurs, vous leur permettez d’envoyer des messages au fil et d’ajouter/supprimer d’autres membres. Avant d’appeler `AddMembers`, vérifiez que vous avez acquis un nouveau jeton d’accès et une identité pour cet utilisateur. L’utilisateur aura besoin de ce jeton d’accès pour initialiser son client de conversation.

Utilisez la méthode `AddMembers` pour ajouter des membres au fil identifié par threadId.

 - Utilisez `members` pour lister les membres à ajouter au fil de conversation ;
 - `User`, obligatoire, est l’identité que vous obtenez pour ce nouvel utilisateur.
 - `DisplayName`, facultatif, est le nom d’affichage du membre du fil.
 - `ShareHistoryTime`, facultatif, est le moment à partir duquel l’historique de conversation est partagé avec le membre. Pour partager l’historique depuis le début du fil de conversation, définissez-le sur DateTime.MinValue. Pour ne partager aucun historique antérieur au moment où le membre a été ajouté, définissez-le sur l’heure actuelle. Pour partager un historique partiel, définissez-le sur un moment dans le temps compris entre l’heure de création du fil et l’heure actuelle.

```csharp
ChatThreadMember member = new ChatThreadMember(communicationUser);
member.DisplayName = "display name member 1";
member.ShareHistoryTime = DateTime.MinValue; // share all history
await chatThreadClient.AddMembersAsync(members: new[] {member});
```
## <a name="remove-user-from-a-chat-thread"></a>Supprimer un utilisateur d’un fil de conversation

De la même manière que vous pouvez ajouter un utilisateur à un fil, vous pouvez supprimer des utilisateurs d’un fil de conversation. Pour ce faire, vous devez suivre l’identité (CommunicationUser) des membres que vous avez ajoutés.

```csharp
await chatThreadClient.RemoveMemberAsync(communicationUser);
```

## <a name="run-the-code"></a>Exécuter le code

Exécutez l’application à partir de votre répertoire d’application avec la commande `dotnet run`.

```console
dotnet run
```
