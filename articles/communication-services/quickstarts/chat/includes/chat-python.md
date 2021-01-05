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
ms.openlocfilehash: 2b7d00335253772683b867acf0765b77fc493e79
ms.sourcegitcommit: 4bee52a3601b226cfc4e6eac71c1cb3b4b0eafe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94523700"
---
## <a name="prerequisites"></a>Prérequis
Avant de commencer, assurez-vous de :

- Créer un compte Azure avec un abonnement actif. Pour plus d’informations, consultez [Créer un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Installez [Python](https://www.python.org/downloads/)
- Créer une ressource Azure Communication Services. Pour plus d’informations, consultez [Créer une ressource Azure Communication](../../create-communication-resource.md). Vous devrez inscrire le **point de terminaison** de votre ressource pour ce démarrage rapide.
- [Jeton d’accès utilisateur](../../access-tokens.md). Veillez à définir l’étendue sur « chat » (conversation) et notez la chaîne token ainsi que la chaîne userId.

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-python-application"></a>Créer une application Python

Ouvrez votre fenêtre de terminal ou de commande, créez un répertoire pour votre application, puis accédez-y.

```console
mkdir chat-quickstart && cd chat-quickstart
```

Utilisez un éditeur de texte pour créer un fichier appelé **send-chat.py** dans le répertoire racine du projet, puis ajoutez la structure du programme, notamment la gestion des exceptions de base. Dans les sections suivantes, vous ajouterez à ce fichier l’ensemble du code source de ce démarrage rapide.

```python
import os
# Add required client library components from quickstart here

try:
    # Quickstart code goes here
except Exception as ex:
    print('Exception:')
    print(ex)
```

### <a name="install-client-library"></a>Installer la bibliothèque de client

```console

pip install azure-communication-chat

```

## <a name="object-model"></a>Modèle objet

Les classes et interfaces suivantes gèrent quelques-unes des principales fonctionnalités de la bibliothèque de client Azure Communication Services Chat pour Python.

| Nom                                  | Description                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| ChatClient | Cette classe est nécessaire à la fonctionnalité de conversation (Chat). Vous l’instanciez avec vos informations d’abonnement et l’utilisez pour créer, obtenir et supprimer des fils de conversation. |
| ChatThreadClient | Cette classe est nécessaire à la fonctionnalité de fil de conversation (Chat Thread). Vous obtenez une instance via ChatClient et l’utilisez pour envoyer/recevoir/mettre à jour/supprimer des messages, ajouter/supprimer/obtenir des utilisateurs, envoyer des notifications de saisie et des accusés de lecture. |

## <a name="create-a-chat-client"></a>Créer un client de conversation

Pour créer un client de conversation, vous allez utiliser le point de terminaison Communication Services ainsi que le `Access Token` qui a été généré au cours des étapes prérequises. Apprenez-en davantage sur les [jetons d’accès utilisateur](../../access-tokens.md).

```console
pip install azure-communication-administration
```

```python
from azure.communication.chat import ChatClient, CommunicationUserCredential

endpoint = "https://<RESOURCE_NAME>.communication.azure.com"
chat_client = ChatClient(endpoint, CommunicationUserCredential(<Access Token>))
```

## <a name="start-a-chat-thread"></a>Démarrer un fil de conversation

Utilisez la méthode `create_chat_thread` pour créer un fil de conversation.

- Utilisez `topic` pour attribuer un sujet au fil ; le sujet peut être mis à jour après que le fil de conversation a été créé à l’aide de la fonction `update_thread`.
- Utilisez `members` pour lister les `ChatThreadMember` à ajouter au fil de conversation. `ChatThreadMember` prend le type `CommunicationUser` comme `user`, que vous avez obtenu à l’issue du processus de création décrit dans [Créer un utilisateur](../../access-tokens.md#create-an-identity)

La réponse `chat_thread_client` est utilisée dans les opérations effectuées sur le fil de conversation nouvellement créé : ajout de membres au fil de conversation, envoi d’un message, suppression d’un message, etc. Elle contient une propriété `thread_id` qui est l’ID unique du fil de conversation.

```python
from datetime import datetime
from azure.communication.chat import ChatThreadMember

topic="test topic"
thread_members=[ChatThreadMember(
    user=user,
    display_name='name',
    share_history_time=datetime.utcnow()
)]
chat_thread_client = chat_client.create_chat_thread(topic, thread_members)
```

## <a name="get-a-chat-thread-client"></a>Obtenir un client de fil de conversation
La méthode get_chat_thread_client retourne un client de fil pour un fil qui existe déjà. Il peut être utilisé dans les opérations effectuées sur le fil créé : ajout de membres, envoi d’un message, etc. thread_id est l’ID unique du fil de conversation existant.

```python
thread_id = 'id'
chat_thread_client = chat_client.get_chat_thread_client(thread_id)
```

## <a name="send-a-message-to-a-chat-thread"></a>Envoyer un message à un fil de conversation

Utilisez la méthode `send_message` pour envoyer un message au fil de conversation que vous venez de créer, identifié par threadId.

- Utilisez `content` pour fournir le contenu du message de conversation ;
- Utilisez `priority` pour spécifier le niveau de priorité du message, par exemple « Normal » ou « High » (Élevé) ; cette propriété peut être utilisée pour que l’indicateur d’interface utilisateur attire l’attention de l’utilisateur destinataire sur le message ou pour exécuter une logique métier personnalisée.
- Utilisez `senderDisplayName` pour spécifier le nom d’affichage de l’expéditeur ;

La réponse `SendChatMessageResult` contient un « ID », qui est l’ID unique de ce message.

```python
from azure.communication.chat import ChatMessagePriority

content='hello world'
priority=ChatMessagePriority.NORMAL
sender_display_name='sender name'

send_message_result = chat_thread_client.send_message(content, priority=priority, sender_display_name=sender_display_name)
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Recevoir les messages de conversation d’un fil de conversation

Vous pouvez récupérer les messages de conversation en interrogeant la méthode `list_messages` selon des intervalles définis.

```python
chat_messages = chat_thread_client.list_messages()
```
`list_messages` retourne la version la plus récente du message, avec les modifications ou les suppressions dont le message a éventuellement fait l’objet via `update_message` et `delete_message`. Pour les messages supprimés, `ChatMessage.deleted_on` retourne une valeur datetime indiquant à quel moment ce message a été supprimé. Pour les messages modifiés, `ChatMessage.edited_on` retourne une valeur datetime indiquant à quel moment ce message a été modifié. Il est possible d’accéder à l’heure initiale de création du message à l’aide de `ChatMessage.created_on` qui peut être utilisée à des fins de classement des messages.

`list_messages` retourne différents types de messages qui peuvent être identifiés par `ChatMessage.type`. Ces types sont les suivants :

- `Text`: Message de conversation ordinaire envoyé par un membre du fil.

- `ThreadActivity/TopicUpdate`: Message système qui indique que le sujet a été mis à jour.

- `ThreadActivity/AddMember`: Message système qui indique qu’un ou plusieurs membres ont été ajoutés au fil de conversation.

- `ThreadActivity/DeleteMember`: Message système qui indique qu’un membre a été supprimé du fil de conversation.

Pour plus d’informations, consultez [Types de messages](../../../concepts/chat/concepts.md#message-types).

## <a name="add-a-user-as-member-to-the-chat-thread"></a>Ajouter un utilisateur en tant que membre du fil de conversation

Une fois qu’un fil de conversation est créé, vous pouvez y ajouter des utilisateurs et en supprimer. En ajoutant des utilisateurs, vous leur permettez d’envoyer des messages au fil et d’ajouter/supprimer d’autres membres. Avant d’appeler la méthode `add_members`, vérifiez que vous avez acquis un nouveau jeton d’accès et une identité pour cet utilisateur. L’utilisateur aura besoin de ce jeton d’accès pour initialiser son client de conversation.

Utilisez la méthode `add_members` pour ajouter des membres au fil identifié par threadId.

- Utilisez `members` pour lister les membres à ajouter au fil de conversation ;
- `user`, obligatoire, est le `CommunicationUser` que vous avez créé avec `CommunicationIdentityClient` à l’étape [Créer un utilisateur](../../access-tokens.md#create-an-identity).
- `display_name`, facultatif, est le nom d’affichage du membre du fil.
- `share_history_time`, facultatif, est le moment à partir duquel l’historique de conversation est partagé avec le membre. Pour partager l’historique depuis le début du fil de conversation, attribuez à cette propriété une date égale ou antérieure à la date de création du fil. Pour ne partager aucun historique antérieur au moment où le membre a été ajouté, attribuez-lui l’heure actuelle. Pour partager un historique partiel, attribuez-lui une date intermédiaire.

```python
new_user = identity_client.create_user()

from azure.communication.chat import ChatThreadMember
from datetime import datetime
member = ChatThreadMember(
    user=new_user,
    display_name='name',
    share_history_time=datetime.utcnow())
thread_members = [member]
chat_thread_client.add_members(thread_members)
```

## <a name="remove-user-from-a-chat-thread"></a>Supprimer un utilisateur d’un fil de conversation

De la même manière que vous pouvez ajouter un membre, vous pouvez aussi supprimer des membres d’un fil. Pour ce faire, vous devez suivre les ID des membres que vous avez ajoutés.

Utilisez la méthode `remove_member` pour supprimer des membres du fil identifié par threadId.
- `user` est le CommunicationUser à supprimer du fil.

```python
chat_thread_client.remove_member(user)
```

## <a name="run-the-code"></a>Exécuter le code

Exécutez l’application à partir de votre répertoire d’application avec la commande `python`.

```console
python start-chat.py
```
