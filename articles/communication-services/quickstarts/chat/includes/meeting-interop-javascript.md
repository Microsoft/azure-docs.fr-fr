---
title: Démarrage rapide - Participer à une réunion Teams
author: askaur
ms.author: askaur
ms.date: 12/08/2020
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 0c41771af81989ff965098a762338216db54fd27
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97578013"
---
## <a name="join-the-meeting-chat"></a>Rejoindre la conversation d’une réunion 

Une fois l’interopérabilité de Teams activée, un utilisateur Communication Services peut rejoindre l’appel Teams en tant qu’utilisateur invité à l’aide de la bibliothèque de client d’appel. En rejoignant l’appel, il est également ajouté comme participant à la conversation de la réunion, où il peut échanger des messages avec d’autres utilisateurs lors de l’appel. L’utilisateur n’aura pas accès aux messages de conversation qui ont été envoyés avant qu’il ne rejoigne l’appel. 

## <a name="get-a-teams-meeting-chat-thread-for-a-communication-services-user"></a>Obtenir le fil de conversation d’une réunion Teams pour un utilisateur Communication Services

Tout d’abord, instanciez `ChatThreadClient` pour le fil de conversation de la réunion. Analysez le lien de réunion ou utilisez les API Graph avec l’ID de réunion pour récupérer l’ID de fil de conversation. 

- Un lien de réunion Teams ressemble à ceci : `https://teams.microsoft.com/l/meetup-join/meeting_chat_thread_id/1606337455313?context=some_context_here`. L’ID de fil de conversation correspond à l’emplacement de `meeting_chat_thread_id` dans ce lien. 
- Si vous disposez de l’ID de réunion, vous pouvez utiliser l’[API Graph](https://docs.microsoft.com/graph/api/onlinemeeting-createorget?tabs=http&view=graph-rest-beta) pour obtenir l’ID de fil de conversation. La réponse [GET API](https://docs.microsoft.com/graph/api/onlinemeeting-get?view=graph-rest-beta&tabs=http%22%20%5C) présentera un objet `chatInfo` contenant `threadID`. 

Une fois que vous disposez de l’ID de fil de conversation, vous pouvez récupérer le client de fil de conversation à l’aide de la bibliothèque de client de conversation JavaScript : 

```javascript
let chatThreadClient = await chatClient.getChatThreadClient(threadId); 

console.log(`Chat Thread client for threadId:${chatThreadClient.threadId}`); 
```
  
`chatThreadClient` peut être utilisé pour lister les membres dans le fil de conversation, recevoir l’historique de conversation et envoyer des messages.  

## <a name="send-and-receive-messages"></a>Envoyer et recevoir des messages  

Utilisez `SendMessage` pour envoyer un message à la conversation de réunion. Pour recevoir des messages entrants, la possibilité de s’abonner à des événements comme `chatMessageReceived` n’est pas prise en charge, car la signalisation en temps réel n’est pas encore activée pour ce scénario. Pour récupérer les messages les plus récents, vous pouvez interroger l’API `ListMessages`. Pour le scénario d’interopérabilité, l’API `ListMessages` prend désormais en charge le retour de trois nouveaux types de messages :
- `RichText/HTML`
- `ThreadActivity/MemberJoined`
- `ThreadActivity/MemberLeft` </br>

Pour plus d’informations sur les types de messages, rendez-vous [ici](../../../concepts/chat/concepts.md). 

**Remarque** Actuellement, seuls l’envoi et la réception de messages sont pris en charge pour les scénarios d’interopérabilité avec Teams. D’autres fonctionnalités, telles que les indicateurs de saisie et les utilisateurs Communication Services qui ajoutent ou suppriment d’autres utilisateurs dans la réunion Teams, ne sont pas prises en charge pour l’instant.  

 
