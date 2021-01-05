---
title: Exemple de bannière de conversation de groupe
titleSuffix: An Azure Communication Services sample overview
description: Vue d’ensemble de l’exemple de bannière de conversation utilisant Azure Communication Services pour permettre aux développeurs d’en savoir plus sur le fonctionnement interne de l’exemple et sur la manière de le modifier.
author: ddematheu2
manager: nimag
services: azure-communication-services
ms.author: dademath
ms.date: 07/20/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: fc757e1310369c48de24c0cc9253c668ca27495c
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94888570"
---
# <a name="get-started-with-the-group-chat-hero-sample"></a>Bien démarrer avec l’exemple de bannière de conversation de groupe

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

<!----
> [!WARNING]
> links to our Hero Sample repo need to be updated when the sample is publicly available.
---->

> [!IMPORTANT]
> [Cet exemple est disponible sur GitHub.](https://github.com/Azure-Samples/communication-services-web-chat-hero)


L’**exemple de bannière de conversation de groupe** Azure Communication Services illustre comment la bibliothèque de client web Conversation Communication Services peut être utilisée pour créer une expérience d’appel de groupe.

Dans ce guide de démarrage rapide, vous allez découvrir comment fonctionne l’exemple avant de l’exécuter sur votre ordinateur local. Ensuite, vous le déploierez sur Azure à l’aide de vos propres ressources Azure Communication Services.


## <a name="overview"></a>Vue d’ensemble

L’exemple comporte à la fois une application côté client et une application côté serveur. L’**application côté client** est une application web React/Redux qui utilise le framework d’interface utilisateur Fluent de Microsoft. Cette application envoie des requêtes à une **application côté serveur** ASP.NET Core qui aide l’application côté client à se connecter à Azure. 

Voici à quoi ressemble l’exemple :

:::image type="content" source="./media/chat/landing-page.png" alt-text="Capture d’écran montrant la page d’arrivée de l’exemple d’application.":::

Lorsque vous appuyez sur le bouton « Start a Chat » (Démarrer une conversation), l’application web récupère un jeton d’accès utilisateur à partir de l’application côté serveur. Ce jeton est ensuite utilisé pour connecter l’application cliente à Azure Communication Services. Une fois le jeton récupéré, vous êtes invité à spécifier votre nom et l’emoji qui vous représentera durant la conversation. 

:::image type="content" source="./media/chat/pre-chat.png" alt-text="Capture d’écran montrant l’écran de préconversation de l’application.":::

Une fois que vous avez configuré votre nom d’affichage et l’emoji, vous pouvez rejoindre la session de conversation. Vous voyez à présent le canevas de conversation principal où réside l’expérience de conversation de base.

:::image type="content" source="./media/chat/main-app.png" alt-text="Capture d’écran montrant l’écran principal de l’exemple d’application.":::

Composants de l’écran principal de conversation :

- **Zone de conversation principale** : il s’agit de l’expérience de conversation de base où les utilisateurs peuvent envoyer et recevoir des messages. Pour envoyer des messages, vous pouvez utiliser la zone d’entrée et appuyer sur Entrée (ou utiliser le bouton d’envoi). Les messages de conversation reçus sont catégorisés par l’expéditeur avec le nom et l’emoji corrects. Deux types de notifications s’affichent dans la zone de conversation : 1) notifications de frappe lorsqu’un utilisateur frappe au clavier et 2) notifications d’envoi et de lecture pour les messages.
- **En-tête**: c’est ici que l’utilisateur verra le titre du fil de conversation et les commandes permettant de choisir entre les barres latérales des participants et des paramètres. Y figure également un bouton permettant de quitter la session de conversation.
- **Barre latérale** : c’est ici que les informations sur les participants et les paramètres sont affichées, en fonction de l’option choisie à l’aide des commandes de l’en-tête. La barre latérale des participants contient une liste des participants à la conversation et un lien pour inviter des participants à la session de conversation. La barre latérale des paramètres vous permet de configurer le titre du fil de conversation. 

Vous trouverez ci-dessous des informations supplémentaires sur les prérequis et les étapes à suivre pour configurer l’exemple.

## <a name="prerequisites"></a>Prérequis

- Créer un compte Azure avec un abonnement actif. Pour plus d’informations, consultez [Créer un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- [Node.js (8.11.2 et versions ultérieures)](https://nodejs.org/en/download/)
- [Visual Studio (2017 et versions ultérieures)](https://visualstudio.microsoft.com/vs/)
- [.NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1) (Veillez à installer la version qui correspond à votre instance de Visual Studio, 32 ou 64 bits.)
- Créer une ressource Azure Communication Services. Pour plus d’informations, consultez [Créer des ressources Azure Communication Services](../quickstarts/create-communication-resource.md). Vous devez noter la **chaîne de connexion** de votre ressource pour ce guide de démarrage rapide

## <a name="locally-deploying-the-service--client-app"></a>Déploiement local du service et de l’application cliente

L’exemple de conversation monofil est en fait constitué de deux « applications » : une application cliente et une application serveur.

Ouvrez chat.csproj dans Visual Studio et exécutez-le en mode Débogage pour démarrer le service front-end de conversation. Lorsque l’application serveur est sollicitée à partir du navigateur, elle redirige le trafic vers le service front-end de conversation déployé localement.

Vous pouvez tester l’exemple localement en ouvrant plusieurs sessions de navigateur avec l’URL de votre conversation afin de simuler une conversation multi-utilisateur.

## <a name="before-running-the-sample-for-the-first-time"></a>Avant d’exécuter l’exemple pour la première fois

1. Ouvrez une instance de PowerShell, de Terminal Windows, une invite de commandes ou équivalent, puis accédez au répertoire dans lequel vous souhaitez cloner l’exemple.
2. `git clone https://github.com/Azure-Samples/communication-services-web-chat-hero.git`
3. Obtenez la `Connection String` à partir du portail Azure. Pour plus d’informations sur les chaînes de connexion, consultez [Créer des ressources Azure Communication Services](../quickstarts/create-communication-resource.md).
4. Une fois la `Connection String` obtenue, ajoutez la chaîne de connexion au fichier **Chat/appsettings.json** qui se trouve dans le dossier Chat. Entrez votre chaîne de connexion dans la variable : `ResourceConnectionString`.

### <a name="local-run"></a>Exécution locale

1. Accédez au dossier Chat et ouvrez la solution `Chat.csproj` dans Visual Studio.
2. Exécutez le projet. Le navigateur s’ouvre à localhost:5000.

#### <a name="troubleshooting"></a>Dépannage

- La solution n’est pas générée, elle lève des erreurs lors de l’installation/génération NPM

   Nettoyez/regénérez la solution C#.

## <a name="publish-the-sample-to-azure"></a>Publier l’exemple sur Azure

1. Cliquez avec le bouton droit sur le projet `Chat` et sélectionnez Publier.
2. Créez un profil de publication et sélectionnez votre abonnement Azure.
3. Avant la publication, ajoutez votre chaîne de connexion avec `Edit App Service Settings` et renseignez `ResourceConnectionString` comme clé, et spécifiez votre chaîne de connexion (copiée à partir d’appsettings.json) comme valeur.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous voulez nettoyer et supprimer un abonnement Communication Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées. Découvrez-en plus sur le [nettoyage des ressources](../quickstarts/create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Étapes suivantes

>[!div class="nextstepaction"] 
>[Télécharger l’exemple à partir de GitHub](https://github.com/Azure-Samples/communication-services-web-chat-hero)

Pour plus d’informations, consultez les articles suivants :

- Apprenez-en davantage sur les [concepts relatifs aux conversations](../concepts/chat/concepts.md)
- Familiarisez-vous avec notre [bibliothèque de client de conversation](../concepts/chat/sdk-features.md)

## <a name="additional-reading"></a>Documentation supplémentaire

- [Azure Communication GitHub](https://github.com/Azure/communication) : Autres exemples et informations dans la page GitHub officielle
- [Redux](https://redux.js.org/) - Gestion de l’état côté client
- [FluentUI](https://aka.ms/fluent-ui) - Bibliothèque d’interface utilisateur, Technologie Microsoft
- [React](https://reactjs.org/) - Bibliothèque pour la création d’interfaces utilisateur
- [ASP.NET Core](/aspnet/core/introduction-to-aspnet-core?preserve-view=true&view=aspnetcore-3.1) - Framework pour la création d’applications web