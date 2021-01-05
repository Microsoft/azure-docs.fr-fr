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
ms.openlocfilehash: edf48bc75817b3510264d852eb9cc717ed022f33
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94915423"
---
## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Java Development Kit (JDK)](/java/azure/jdk/?preserve-view=true&view=azure-java-stable) version 8 ou ultérieure.
- [Apache Maven](https://maven.apache.org/download.cgi).
- Chaîne de connexion et ressource Communication Services déployée. [Créez une ressource Communication Services](../../create-communication-resource.md).
- [Jeton d’accès utilisateur](../../access-tokens.md). Veillez à définir l’étendue sur « chat » (conversation) et notez la chaîne token ainsi que la chaîne userId.


## <a name="setting-up"></a>Configuration

### <a name="create-a-new-java-application"></a>Créer une application Java

Ouvrez votre terminal ou votre fenêtre Commande, puis accédez au répertoire dans lequel vous souhaitez créer votre application Java. Exécutez la commande ci-dessous pour générer le projet Java à partir du modèle maven-archetype-quickstart.

```console
mvn archetype:generate -DgroupId=com.communication.quickstart -DartifactId=communication-quickstart -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

Vous remarquerez que l’objectif « generate » a créé un répertoire portant le même nom que l’artifactId. Sous ce répertoire, `src/main/java directory` contient le code source du projet, le répertoire `src/test/java` contient la source de test et le fichier pom.xml est le modèle objet du projet, ou [POM](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html).

Mettez à jour le fichier POM de votre application pour utiliser Java 8 ou version ultérieure :

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

### <a name="add-the-package-references-for-the-chat-client-library"></a>Ajouter les références de package pour la bibliothèque de client de conversation

Dans votre fichier POM, référencez le package `azure-communication-chat` avec les API de conversation (Chat) :

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-chat</artifactId>
    <version>1.0.0-beta.3</version> 
</dependency>
```

Pour l’authentification, votre client doit référencer le package `azure-communication-common` :

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-common</artifactId>
    <version>1.0.0-beta.3</version> 
</dependency>

```

## <a name="object-model"></a>Modèle objet

Les classes et interfaces suivantes gèrent quelques-unes des principales fonctionnalités de la bibliothèque de client Azure Communication Services Chat pour Java.

| Nom                                  | Description                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| ChatClient | Cette classe est nécessaire à la fonctionnalité de conversation (Chat). Vous l’instanciez avec vos informations d’abonnement et l’utilisez pour créer, obtenir et supprimer des fils de conversation. |
| ChatAsyncClient | Cette classe est nécessaire à la fonctionnalité de conversation (Chat) asynchrone. Vous l’instanciez avec vos informations d’abonnement et l’utilisez pour créer, obtenir et supprimer des fils de conversation. |
| ChatThreadClient | Cette classe est nécessaire à la fonctionnalité de fil de conversation (Chat Thread). Vous obtenez une instance via ChatClient et l’utilisez pour envoyer/recevoir/mettre à jour/supprimer des messages, ajouter/supprimer/obtenir des utilisateurs, envoyer des notifications de saisie et des accusés de lecture. |
| ChatThreadAsyncClient | Cette classe est nécessaire à la fonctionnalité de fil de conversation (Chat Thread) asynchrone. Vous obtenez une instance via ChatAsyncClient et l’utilisez pour envoyer/recevoir/mettre à jour/supprimer des messages, ajouter/supprimer/obtenir des utilisateurs, envoyer des notifications de saisie et des accusés de lecture. |

## <a name="create-a-chat-client"></a>Créer un client de conversation
Pour créer un client de conversation, vous allez utiliser le point de terminaison Communication Services ainsi que le jeton d’accès qui a été généré au cours des étapes prérequises. Les jetons d’accès utilisateur vous permettent de créer des applications clientes qui s’authentifient directement auprès d’Azure Communication Services. Une fois que vous avez généré ces jetons sur votre serveur, transmettez-les en retour à un appareil client. Vous devez utiliser la classe CommunicationUserCredential de la bibliothèque de client commune pour transmettre le jeton à votre client de conversation. 

Si vous devez ajouter des instructions d’importation, veillez à ajouter uniquement des importations des espaces de noms com.azure.communication.chat et com.azure.communication.chat.models, et non de l’espace de noms com.azure.communication.chat.implementation. Dans le fichier App.java qui a été généré via Maven, vous pouvez commencer avec le code suivant :

```Java
import com.azure.communication.chat.*;
import com.azure.communication.chat.models.*;
import com.azure.communication.common.*;
import com.azure.core.http.HttpClient;

import java.io.*;

public class App
{
    public static void main( String[] args ) throws IOException
    {
        System.out.println("Azure Communication Services - Chat Quickstart");
        
        // Your unique Azure Communication service endpoint
        String endpoint = "https://<RESOURCE_NAME>.communication.azure.com";

        // Create an HttpClient builder of your choice and customize it
        // Use com.azure.core.http.netty.NettyAsyncHttpClientBuilder if that suits your needs
        NettyAsyncHttpClientBuilder yourHttpClientBuilder = new NettyAsyncHttpClientBuilder();
        HttpClient httpClient = yourHttpClientBuilder.build();

        // User access token fetched from your trusted service
        String userAccessToken = "<USER_ACCESS_TOKEN>";

        // Create a CommunicationUserCredential with the given access token, which is only valid until the token is valid
        CommunicationUserCredential userCredential = new CommunicationUserCredential(userAccessToken);

        // Initialize the chat client
        final ChatClientBuilder builder = new ChatClientBuilder();
        builder.endpoint(endpoint)
            .credential(userCredential)
            .httpClient(httpClient);
        ChatClient chatClient = builder.buildClient();
    }
}
```


## <a name="start-a-chat-thread"></a>Démarrer un fil de conversation

Utilisez la méthode `createChatThread` pour créer un fil de conversation.
`createChatThreadOptions` est utilisé pour décrire la demande de fil.

- Utilisez `topic` pour attribuer un sujet à cette conversation ; le sujet peut être mis à jour après que le fil de conversation a été créé à l’aide de la fonction `UpdateThread`.
- Utilisez `members` pour lister les membres de fil à ajouter à la conversation. `ChatThreadMember` prend l’utilisateur que vous avez créé dans le démarrage rapide [Jeton d’accès utilisateur](../../access-tokens.md).

La réponse `chatThreadClient` est utilisée dans les opérations effectuées sur le fil de conversation créé : ajout de membres au fil de conversation, envoi d’un message, suppression d’un message, etc. Elle contient une propriété `chatThreadId` qui est l’ID unique du fil de conversation. La propriété est accessible par la méthode publique .getChatThreadId().

```Java
List<ChatThreadMember> members = new ArrayList<ChatThreadMember>();

ChatThreadMember firstThreadMember = new ChatThreadMember()
    .setUser(firstUser)
    .setDisplayName("Member Display Name 1");
    
ChatThreadMember secondThreadMember = new ChatThreadMember()
    .setUser(secondUser)
    .setDisplayName("Member Display Name 2");

members.add(firstThreadMember);
members.add(secondThreadMember);

CreateChatThreadOptions createChatThreadOptions = new CreateChatThreadOptions()
    .setTopic("Topic")
    .setMembers(members);
ChatThreadClient chatThreadClient = chatClient.createChatThread(createChatThreadOptions);
String chatThreadId = chatThreadClient.getChatThreadId();
```

## <a name="send-a-message-to-a-chat-thread"></a>Envoyer un message à un fil de conversation

Utilisez la méthode `sendMessage` pour envoyer un message au fil que vous venez de créer, identifié par chatThreadId.
`sendChatMessageOptions` est utilisé pour décrire la demande de message de conversation.

- Utilisez `content` pour fournir le contenu du message de conversation.
- Utilisez `priority` pour spécifier le niveau de priorité du message de conversation, par exemple « Normal » ou « High » (Élevé) ; cette propriété peut être utilisée pour que l’indicateur d’interface utilisateur attire l’attention de l’utilisateur destinataire sur le message ou pour exécuter une logique métier personnalisée.
- Utilisez `senderDisplayName` pour spécifier le nom d’affichage de l’expéditeur.

La réponse `sendChatMessageResult` contient un `id`, qui est l’ID unique du message.

```Java
SendChatMessageOptions sendChatMessageOptions = new SendChatMessageOptions()
    .setContent("Message content")
    .setPriority(ChatMessagePriority.NORMAL)
    .setSenderDisplayName("Sender Display Name");

SendChatMessageResult sendChatMessageResult = chatThreadClient.sendMessage(sendChatMessageOptions);
String chatMessageId = sendChatMessageResult.getId();
```


## <a name="get-a-chat-thread-client"></a>Obtenir un client de fil de conversation

La méthode `getChatThreadClient` retourne un client de fil pour un fil qui existe déjà. Il peut être utilisé dans les opérations effectuées sur le fil créé : ajout de membres, envoi d’un message, etc. `chatThreadId` est l’ID unique du fil de conversation existant.

```Java
String chatThreadId = "Id";
ChatThread chatThread = chatClient.getChatThread(chatThreadId);
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Recevoir les messages de conversation d’un fil de conversation

Vous pouvez récupérer les messages de conversation en interrogeant la méthode `listMessages` sur le client de fil de conversation selon des intervalles définis.

```Java
chatThreadClient.listMessages().iterableByPage().forEach(resp -> {
    System.out.printf("Response headers are %s. Url %s  and status code %d %n", resp.getHeaders(),
        resp.getRequest().getUrl(), resp.getStatusCode());
    resp.getItems().forEach(message -> {
        System.out.printf("Message id is %s.", message.getId());
    });
});
```

`listMessages` retourne la version la plus récente du message, avec les modifications ou les suppressions dont le message a éventuellement fait l’objet via .editMessage() et .deleteMessage(). Pour les messages supprimés, `chatMessage.getDeletedOn()` retourne une valeur datetime indiquant à quel moment ce message a été supprimé. Pour les messages modifiés, `chatMessage.getEditedOn()` retourne une valeur datetime indiquant à quel moment ce message a été modifié. Il est possible d’accéder à l’heure initiale de création du message à l’aide de `chatMessage.getCreatedOn()` ; elle peut être utilisée à des fins de classement des messages.

`listMessages` retourne différents types de messages qui peuvent être identifiés par `chatMessage.getType()`. Ces types sont les suivants :

- `Text`: Message de conversation ordinaire envoyé par un membre du fil.

- `ThreadActivity/TopicUpdate`: Message système qui indique que le sujet a été mis à jour.

- `ThreadActivity/AddMember`: Message système qui indique qu’un ou plusieurs membres ont été ajoutés au fil de conversation.

- `ThreadActivity/DeleteMember`: Message système qui indique qu’un membre a été supprimé du fil de conversation.

Pour plus d’informations, consultez [Types de messages](../../../concepts/chat/concepts.md#message-types).

## <a name="add-a-user-as-member-to-the-chat-thread"></a>Ajouter un utilisateur en tant que membre du fil de conversation

Une fois qu’un fil de conversation est créé, vous pouvez y ajouter des utilisateurs et en supprimer. En ajoutant des utilisateurs, vous leur permettez d’envoyer des messages au fil de conversation et d’ajouter/supprimer d’autres membres. Vous devez commencer par obtenir un nouveau jeton d’accès et une identité pour cet utilisateur. Avant d’appeler la méthode addMembers, vérifiez que vous avez acquis un nouveau jeton d’accès et une identité pour cet utilisateur. L’utilisateur aura besoin de ce jeton d’accès pour initialiser son client de conversation.

Utilisez la méthode `addMembers` pour ajouter des membres au fil identifié par threadId.

- Utilisez `members` pour lister les membres à ajouter au fil de conversation.
- `user`, obligatoire, est le CommunicationUser que vous avez créé avec le CommunicationIdentityClient dans le démarrage rapide [Jeton d’accès utilisateur](../../access-tokens.md).
- `display_name`, facultatif, est le nom d’affichage du membre du fil.
- `share_history_time`, facultatif, est le moment à partir duquel l’historique de conversation est partagé avec le membre. Pour partager l’historique depuis le début du fil de conversation, attribuez à cette propriété une date égale ou antérieure à la date de création du fil. Pour ne partager aucun historique antérieur au moment où le membre a été ajouté, attribuez-lui l’heure actuelle. Pour partager un historique partiel, attribuez-lui la date de votre choix.

```Java
List<ChatThreadMember> members = new ArrayList<ChatThreadMember>();

ChatThreadMember firstThreadMember = new ChatThreadMember()
    .setUser(user1)
    .setDisplayName("Display Name 1");

ChatThreadMember secondThreadMember = new ChatThreadMember()
    .setUser(user2)
    .setDisplayName("Display Name 2");

members.add(firstThreadMember);
members.add(secondThreadMember);

AddChatThreadMembersOptions addChatThreadMembersOptions = new AddChatThreadMembersOptions()
    .setMembers(members);
chatThreadClient.addMembers(addChatThreadMembersOptions);
```

## <a name="remove-user-from-a-chat-thread"></a>Supprimer un utilisateur d’un fil de conversation

De la même manière que vous pouvez ajouter un utilisateur à un fil, vous pouvez supprimer des utilisateurs d’un fil de conversation. Pour ce faire, vous devez suivre les identités utilisateur des membres que vous avez ajoutés.

Utilisez `removeMember`, où `user` est le CommunicationUser que vous avez créé.

```Java
chatThreadClient.removeMember(user);
```

## <a name="run-the-code"></a>Exécuter le code

Accédez au répertoire contenant le fichier *pom.xml*, puis compilez le projet à l’aide de la commande `mvn` suivante.

```console
mvn compile
```

Ensuite, générez le package.

```console
mvn package
```

Exécutez la commande `mvn` suivante pour exécuter l’application.

```console
mvn exec:java -Dexec.mainClass="com.communication.quickstart.App" -Dexec.cleanupDaemonThreads=false
```