---
title: Exemple de bannière d’appel de groupe
titleSuffix: An Azure Communication Services sample overview
description: Vue d’ensemble de l’exemple de bannière d’appel utilisant Azure Communication Services pour permettre aux développeurs d’en savoir plus sur le fonctionnement interne de l’exemple.
author: ddematheu2
manager: nimag
services: azure-communication-services
ms.author: dademath
ms.date: 07/20/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: c872c2a2799eee3ae9e39653b0046b70ca5d7e8b
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94887533"
---
# <a name="get-started-with-the-group-calling-hero-sample"></a>Bien démarrer avec l’exemple de bannière d’appel de groupe

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

<!----
> [!WARNING]
> Add links to our Hero Sample repo when the sample is publicly available.
---->

> [!IMPORTANT]
> [Cet exemple est disponible sur GitHub.](https://github.com/Azure-Samples/communication-services-web-calling-hero)

L’**exemple de bannière d’appel de groupe** Azure Communication Services illustre comment la bibliothèque de client web Appel Communication Services peut être utilisée pour créer une expérience d’appel de groupe.

Dans ce guide de démarrage rapide, vous allez découvrir comment fonctionne l’exemple avant de l’exécuter sur votre ordinateur local. Ensuite, vous le déploierez sur Azure à l’aide de vos propres ressources Azure Communication Services.

## <a name="overview"></a>Vue d’ensemble

L’exemple comporte à la fois une application côté client et une application côté serveur. L’**application côté client** est une application web React/Redux qui utilise le framework d’interface utilisateur Fluent de Microsoft. Cette application envoie des requêtes à une **application côté serveur** ASP.NET Core qui aide l’application côté client à se connecter à Azure. 

Voici à quoi ressemble l’exemple :

:::image type="content" source="./media/calling/landing-page.png" alt-text="Capture d’écran montrant la page d’arrivée de l’exemple d’application.":::

Lorsque vous appuyez sur le bouton « Start a call » (Démarrer un appel), l’application web récupère un jeton d’accès utilisateur à partir de l’application côté serveur. Ce jeton est ensuite utilisé pour connecter l’application cliente à Azure Communication Services. Une fois le jeton récupéré, vous êtes invité à spécifier la caméra et le microphone que vous souhaitez utiliser. Vous pouvez désactiver/activer vos appareils avec des bascules :

:::image type="content" source="./media/calling/pre-call.png" alt-text="Capture d’écran montrant l’écran de pré-appel de l’exemple d’application.":::

Une fois que vous avez configuré votre nom d’affichage et vos appareils, vous pouvez rejoindre la session d’appel. Vous voyez à présent le canevas d’appel principal où réside l’expérience d’appel de base.

:::image type="content" source="./media/calling/main-app.png" alt-text="Capture d’écran montrant l’écran principal de l’exemple d’application.":::

Composants de l’écran principal d’appel :

1. **Galerie multimédia** : scène principale montrant les participants. Si une caméra est activée pour un participant, son flux vidéo est affiché ici. Chaque participant a une vignette individuelle qui comporte son nom d’affichage et son flux vidéo (le cas échéant).
2. **En-tête**: il s’agit de l’emplacement où se trouvent les principales commandes d’appel pour basculer entre les barres latérales de paramètres et de participants, activer/désactiver la vidéo et le micro, partager l’écran et quitter l’appel.
3. **Barre latérale** : c’est ici que les informations sur les participants et les paramètres sont affichées, en fonction de l’option choisie à l’aide des commandes de l’en-tête. Le composant peut être fermé à l’aide du « X » situé en haut à droite. La barre latérale des participants affiche une liste des participants et un lien pour inviter davantage d’utilisateurs à discuter. La barre latérale des paramètres vous permet de configurer les paramètres du microphone et de la caméra.

Vous trouverez ci-dessous des informations supplémentaires sur les prérequis et les étapes à suivre pour configurer l’exemple.

## <a name="prerequisites"></a>Prérequis

- Créer un compte Azure avec un abonnement actif. Pour plus d’informations, consultez [Créer un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- [Node.js (12.18.4 et versions ultérieures)](https://nodejs.org/en/download/)
- [Visual Studio (2019 et versions ultérieures)](https://visualstudio.microsoft.com/vs/)
- [.NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1) (Veillez à installer la version qui correspond à votre instance de Visual Studio, 32 ou 64 bits.)
- Créer une ressource Azure Communication Services. Pour plus d’informations, consultez [Créer des ressources Azure Communication Services](../quickstarts/create-communication-resource.md). Vous devez noter la **chaîne de connexion** de votre ressource pour ce guide de démarrage rapide

## <a name="locally-deploy-the-service--client-applications"></a>Déploiement local du service et des applications clientes

L’exemple d’appel de groupe est essentiellement composé deux applications : l’application ClientApp et l’application Service.NET.

Lorsque nous souhaitons le déployer localement, nous devons démarrer les deux applications. Lorsque l’application serveur est sollicitée à partir du navigateur, elle utilise l’application ClientApp déployée localement pour l’expérience utilisateur.

Vous pouvez tester l’exemple localement en ouvrant plusieurs sessions de navigateur avec l’URL de votre appel afin de simuler un appel multi-utilisateur.

## <a name="before-running-the-sample-for-the-first-time"></a>Avant d’exécuter l’exemple pour la première fois

1. Ouvrez une instance de PowerShell, de Terminal Windows, une invite de commandes ou équivalent, puis accédez au répertoire dans lequel vous souhaitez cloner l’exemple.
2. `git clone https://github.com/Azure-Samples/communication-services-web-calling-hero.git`
3. Obtenez la `Connection String` à partir du portail Azure. Pour plus d’informations sur les chaînes de connexion, consultez [Créer des ressources Azure Communication Services](../quickstarts/create-communication-resource.md).
4. Une fois que vous avez obtenu la `Connection String`, ajoutez-la au fichier **Calling/appsetting.json** qui se trouve sous le dossier Service .NET. Entrez votre chaîne de connexion dans la variable : `ResourceConnectionString`.

### <a name="local-run"></a>Exécution locale

1. Accédez au dossier Calling et ouvrez la solution `Calling.csproj` dans Visual Studio.
2. Exécutez le projet `Calling`. Le navigateur s’ouvre à localhost:5001.

#### <a name="troubleshooting"></a>Dépannage

- La solution n’est pas générée, elle lève des erreurs lors de l’installation/génération NPM

   Essayez de nettoyer/regénérer les projets.

## <a name="publish-the-sample-to-azure"></a>Publier l’exemple sur Azure

1. Cliquez avec le bouton droit sur le projet `Calling` et sélectionnez Publier.
2. Créez un profil de publication et sélectionnez votre abonnement Azure.
3. Avant la publication, ajoutez votre chaîne de connexion avec `Edit App Service Settings` et renseignez `ResourceConnectionString` comme clé, et spécifiez votre chaîne de connexion (copiée à partir d’appsettings.json) comme valeur.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous voulez nettoyer et supprimer un abonnement Communication Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées. Découvrez-en plus sur le [nettoyage des ressources](../quickstarts/create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Étapes suivantes

>[!div class="nextstepaction"] 
>[Télécharger l’exemple à partir de GitHub](https://github.com/Azure-Samples/communication-services-web-calling-hero)

Pour plus d’informations, consultez les articles suivants :

- Familiarisez-vous avec la [bibliothèque de client d’appel](../quickstarts/voice-video-calling/calling-client-samples.md)
- Apprenez-en davantage sur le [fonctionnement des appels](../concepts/voice-video-calling/about-call-types.md)

## <a name="additional-reading"></a>Documentation supplémentaire

- [Azure Communication GitHub](https://github.com/Azure/communication) : Autres exemples et informations dans la page GitHub officielle
- [Redux](https://redux.js.org/) - Gestion de l’état côté client
- [FluentUI](https://aka.ms/fluent-ui) - Bibliothèque d’interface utilisateur, Technologie Microsoft
- [React](https://reactjs.org/) - Bibliothèque pour la création d’interfaces utilisateur
- [ASP.NET Core](/aspnet/core/introduction-to-aspnet-core?preserve-view=true&view=aspnetcore-3.1) - Framework pour la création d’applications web