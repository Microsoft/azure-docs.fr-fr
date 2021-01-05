---
title: Qu’est-ce qu’Azure Communication Services ?
description: Découvrez comment Azure Communication Services vous aide à développer de riches expériences utilisateur avec des communications en temps réel.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 07/20/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: d680df0ec5e18cca4d7a42edf45dbd6a50be1472
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94888604"
---
# <a name="what-is-azure-communication-services"></a>Qu’est-ce qu’Azure Communication Services ?

[!INCLUDE [Public Preview Notice](./includes/public-preview-include.md)]

Azure Communication Services vous permet d’ajouter facilement des fonctionnalités multimédias vocales, vidéos et de communication par téléphonie sur IP en temps réel à vos applications. Les bibliothèques de client Communication Services vous permettent également d’ajouter des fonctionnalités de conversation et SMS à vos solutions de communication.

<br>

> [!VIDEO https://www.youtube.com/embed/49oshhgY6UQ]

<br>
<br>

Vous pouvez utiliser Communication Services pour la communication vocale, vidéo, de texte et de données dans divers scénarios :

- Communication navigateur à navigateur, navigateur à application, et application à application
- Interaction entre des utilisateurs et des bots ou autres services
- Interaction entre des utilisateurs et des bots sur le réseau téléphonique commuté

Les scénarios mixtes sont pris en charge. Par exemple, une application Communication Services peut combiner des utilisateurs qui discutent par le biais de navigateurs et d’autres par le biais d’appareils de téléphonie traditionnels. Communication Services peut également être combiné avec Azure Bot Service afin de générer des systèmes de réponse vocale interactive (RVI) pilotés par bot.

## <a name="common-scenarios"></a>Scénarios courants

Les ressources suivantes constituent un excellent point de départ si vous ne connaissez pas Azure Communication Services :
<br>

| Ressource                               |Description                           |
|---                                    |---                                   |
|**[Créer une ressource Communication Services](./quickstarts/create-communication-resource.md)**|Vous pouvez commencer à utiliser Azure Communication Services à l’aide du portail Azure ou de la bibliothèque de client Communication Services Administration afin de provisionner votre première ressource Communication Services. Une fois que vous disposez de la chaîne de connexion de votre ressource Communication Services, vous pouvez provisionner vos premiers jetons d’accès utilisateur.|
|**[Créer votre premier jeton d’accès utilisateur](./quickstarts/access-tokens.md)**|Les jetons d’accès utilisateur servent à authentifier vos services auprès de votre ressource Azure Communication Services. Ces jetons sont provisionnés et réémis à l’aide de la bibliothèque de client Communication Services Administration.|
|**[Obtenir un numéro de téléphone](./quickstarts/telephony-sms/get-phone-number.md)**|Vous pouvez utiliser Azure Communication Services pour provisionner et diffuser des numéros de téléphone. Ces numéros de téléphone peuvent être utilisés pour établir des appels sortants et créer des solutions de communication SMS.|
|**[Envoyer un SMS à partir de votre application](./quickstarts/telephony-sms/send.md)**|La bibliothèque de client Azure Communication Services SMS vous permet d’envoyer et de recevoir des SMS à partir de vos applications .NET et JavaScript.|
|**[Bien démarrer avec l’appel vocal et vidéo](./quickstarts/voice-video-calling/getting-started-with-calling.md)**| Azure Communication Services vous permet d’ajouter des appels vocaux et vidéo à vos applications à l’aide de la bibliothèque de client Calling. Cette bibliothèque tire parti de WebRTC et vous permet d’établir des communications de pair à pair, multimédias et en temps réel au sein de vos applications.|
|**[Bien démarrer avec les conversations](./quickstarts/chat/get-started.md)**|La bibliothèque de client Azure Communication Services Chat peut être utilisée pour intégrer la conversation en temps réel dans vos applications.|


## <a name="samples"></a>Exemples

Les exemples suivants illustrent l’utilisation de bout en bout des bibliothèques de client Azure Communication Services. N’hésitez pas à utiliser ces exemples pour démarrer vos propres solutions Communication Services.
<br>

| Nom d’exemple                               | Description                           |
|---                                    |---                                   |
|**[Exemple de bannière d’appel de groupe](./samples/calling-hero-sample.md)**|Découvrez comment les bibliothèques de client Communication Services peuvent être utilisées pour créer une expérience d’appel de groupe.|
|**[Exemple de bannière de conversation de groupe](./samples/chat-hero-sample.md)**|Découvrez comment les bibliothèques de client Communication Services peuvent être utilisées pour créer une expérience de conversation de groupe.|


## <a name="platforms-and-client-libraries"></a>Plateformes et bibliothèques de client

Les ressources suivantes vous aideront à en savoir plus sur les bibliothèques de client Azure Communication Services :

| Ressource                               | Description                           |
|---                                    |---                                   |
|**[Bibliothèques de client et API REST](./concepts/sdk-options.md)**|Les fonctionnalités Azure Communication Services sont organisées d’un point de vue conceptuel en six domaines, chacun représenté par une bibliothèque de client. Vous pouvez choisir les bibliothèques de client à utiliser en fonction de vos besoins en matière de communication en temps réel.|
|**[Vue d’ensemble de la bibliothèque de client Appel](./concepts/voice-video-calling/calling-sdk-features.md)**|Consultez la vue d’ensemble de la bibliothèque de client Communication Services Calling.|
|**[Vue d’ensemble de la bibliothèque de client Chat](./concepts/chat/sdk-features.md)**|Consultez la vue d’ensemble de la bibliothèque de client Communication Services Chat.|
|**[Vue d’ensemble de la bibliothèque de client SMS](./concepts/telephony-sms/sdk-features.md)**|Consultez la vue d’ensemble de la bibliothèque de client Communication Services SMS.|

## <a name="compare-azure-communication-services"></a>Comparer Azure Communication Services

Vous pouvez envisager d’utiliser deux autres produits de communication Microsoft qui ne sont à l’heure actuelle pas directement interopérables avec Communication Services :

 - Les [API de communication cloud Microsoft Graph](/graph/cloud-communications-concept-overview) permettent aux organisations de créer des expériences de communication liées aux utilisateurs Azure Active Directory disposant de licences M365. C’est idéal pour les applications liées à Azure Active Directory ou lorsque vous souhaitez étendre les expériences de productivité dans Microsoft Teams. Il existe également des API pour créer des applications et une personnalisation au sein de l’[expérience Teams](/microsoftteams/platform/?preserve-view=true&view=msteams-client-js-latest).

 - [Azure PlayFab Party](/gaming/playfab/features/multiplayer/networking/) simplifie l’ajout de communications de conversation et de communication de données à faible latence aux jeux. Même si vous pouvez utiliser la technologie Communication Services pour les systèmes de conversation de jeux et de réseau, PlayFab est une option personnalisée et gratuite sur Xbox.


## <a name="next-steps"></a>Étapes suivantes

 - [Créer une ressource Communication Services](./quickstarts/create-communication-resource.md)