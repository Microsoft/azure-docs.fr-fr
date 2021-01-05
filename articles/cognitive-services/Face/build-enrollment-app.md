---
title: Créer une application d’inscription pour Android avec React
titleSuffix: Azure Cognitive Services
description: Découvrez comment configurer votre environnement de développement et déployer une application d’inscription Visage pour obtenir le consentement des clients.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 11/17/2020
ms.author: pafarley
ms.openlocfilehash: 085dd18214f795566669fb862bba63b67eb9115d
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350362"
---
# <a name="build-an-enrollment-app-for-android-with-react"></a>Créer une application d’inscription pour Android avec React

Ce guide vous montre comment prendre en main l’exemple d’application d’inscription Visage. L’application illustre les bonnes pratiques en matière d’obtention du consentement explicite permettant d’inscrire des utilisateurs à un service de reconnaissance faciale et d’acquérir des données de visage de haute précision. Un système intégré peut utiliser une application d’inscription comme celle-ci pour proposer un contrôle d’accès sans contact, une vérification de l’identité, un suivi de l’assiduité ou une borne de personnalisation, en fonction de ses données de visage.

Une fois lancée, l’application présente un écran de consentement détaillé aux utilisateurs. Si l’utilisateur donne son consentement, l’application l’invite à entrer un nom d’utilisateur et un mot de passe, puis capture une image de haute qualité de son visage à l’aide de la caméra de l’appareil.

L’exemple d’application d’inscription est écrit à l’aide de JavaScript et du framework React Native. Il peut actuellement être déployé sur des appareils Android. D’autres options de déploiement seront disponibles prochainement.

## <a name="prerequisites"></a>Prérequis 

* Un abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/cognitive-services/).  
* Une fois que vous avez votre abonnement Azure, [créez une ressource Visage](https://portal.azure.com/#create/Microsoft.CognitiveServicesFace) dans le portail Azure pour obtenir votre clé et votre point de terminaison. À la fin du déploiement, sélectionnez **Accéder à la ressource**.  
  * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous avez créée pour connecter votre application à l’API Visage.  
  * À des fins de développement et de test locaux, vous pouvez coller la clé API et le point de terminaison dans le fichier de configuration. Pour le déploiement final, stockez la clé API dans un emplacement sécurisé, mais jamais dans le code.  

> [!IMPORTANT]
> Ces clés d’abonnement sont utilisées pour accéder à votre API Cognitive Service. Ne partagez pas vos clés. Stockez-les en toute sécurité, par exemple, à l’aide de Azure Key Vault. Nous vous recommandons également de régénérer ces clés régulièrement. Une seule clé est nécessaire pour effectuer un appel d’API. Lors de la régénération de la première clé, vous pouvez utiliser la deuxième clé pour un accès continu au service.

## <a name="set-up-the-development-environment"></a>Configuration de l’environnement de développement

1. Clonez le dépôt git pour l’[exemple d’application d’inscription](https://github.com/azure-samples/cognitive-services-FaceAPIEnrollmentSample).
1. Pour configurer votre environnement de développement, suivez la <a href="https://reactnative.dev/docs/environment-setup"  title="documentation React Native"  target="_blank">documentation React Native <span class="docon docon-navigate-external x-hidden-focus"></span></a>. Sélectionnez **React Native CLI Quickstart** (Démarrage rapide avec CLI React Native) comme système d’exploitation de développement, puis **Android** comme système d’exploitation cible. Suivez les indications des sections **Installing dependencies** (Installation des dépendances) et **Android development environment** (Environnement de développement Android).
1. Ouvrez le fichier env.json dans l’éditeur de texte de votre choix, par exemple [Visual Studio Code](https://code.visualstudio.com/), puis ajoutez votre point de terminaison et votre clé. Vous pouvez accéder à votre point de terminaison et à votre clé dans le portail Azure sous l’onglet **Vue d’ensemble** de votre ressource. Cette étape sert uniquement à des fins de test local : n’archivez pas votre clé API Visage dans votre référentiel distant.
1. Exécutez l’application à l’aide de l’émulateur d’appareil virtuel Android à partir d’Android Studio ou de votre propre appareil Android. Pour tester votre application sur un appareil physique, suivez la <a href="https://reactnative.dev/docs/running-on-device"  title="documentation React Native"  target="_blank">documentation React Native <span class="docon docon-navigate-external x-hidden-focus"></span></a> correspondante.  


## <a name="create-an-enrollment-experience"></a>Créer une expérience d’inscription  

Maintenant que vous avez configuré l’exemple d’application d’inscription, vous pouvez l’adapter aux besoins de votre propre expérience d’inscription.

Par exemple, vous pouvez éventuellement ajouter des informations propres à votre situation dans votre page de consentement :

> [!div class="mx-imgBorder"]
> ![page de consentement de l’application](./media/enrollment-app/1-consent-1.jpg)

Le service propose des contrôles de la qualité d’image pour vous aider à déterminer si l’image est d’une qualité suffisante pour inscrire le client ou tenter une reconnaissance faciale. Cette application montre comment accéder aux images issues de la caméra de l’appareil, sélectionner celles dont la qualité est optimale et inscrire le visage détecté dans le service API Visage. 

De nombreux problèmes de reconnaissance faciale sont dus à des images de référence de faible qualité. Voici des facteurs pouvant dégrader les performances du modèle :
* Taille du visage (visages éloignés de la caméra)
* Orientation du visage (visages tournés ou inclinés ne regardant pas la caméra)
* Conditions d’éclairage médiocres (faible éclairage ou contre-jour) qui produisent une image sous-exposée ou bruitée
* Obstruction (visages partiellement cachés ou recouverts), notamment avec des accessoires comme des chapeaux ou des lunettes à monture épaisse)
* Flou (par exemple à cause d’un mouvement de visage rapide au moment où la photo a été prise) 

> [!div class="mx-imgBorder"]
> ![Page d’instructions sur la capture d’image dans l’application](./media/enrollment-app/4-instruction.jpg)

Remarquez que l’application offre également des fonctionnalités permettant de supprimer l’inscription de l’utilisateur et de le réinscrire.

> [!div class="mx-imgBorder"]
> ![page de gestion des profils](./media/enrollment-app/10-manage-2.jpg)

Pour étendre les fonctionnalités de l’application afin de couvrir toute l’expérience d’inscription, consultez la [vue d’ensemble](enrollment-overview.md) pour connaître les autres fonctionnalités à implémenter et les bonnes pratiques.

## <a name="deploy-the-enrollment-app"></a>Déployer l’application d’inscription

### <a name="android"></a>Android

Pour commencer, vérifiez que votre application est prête pour le déploiement en production : supprimez les clés ou les secrets du code de l’application et vérifiez que vous avez suivi les [bonnes pratiques en matière de sécurité](../cognitive-services-security.md?tabs=command-line%2ccsharp).

Quand vous êtes prêt à publier votre application en production, vous générez un fichier APK prêt pour la mise en production, qui correspond au format de fichier de package pour les applications Android. Ce fichier APK doit être signé avec une clé privée. Avec cette build, vous pouvez commencer à distribuer l’application directement à vos appareils. 

Pour savoir comment générer une clé privée, signer votre application et générer un fichier APK de mise en production, suivez la documentation <a href="https://developer.android.com/studio/publish/preparing#publishing-build"  title="Prepare for release"  target="_blank">Prepare for release <span class="docon docon-navigate-external x-hidden-focus"></span></a> (Préparer la mise en production).  

Une fois que vous avez créé un fichier APK signé, consultez la documentation <a href="https://developer.android.com/studio/publish"  title="Publish your app"  target="_blank">Publish your app<span class="docon docon-navigate-external x-hidden-focus"></span></a> (Publier votre application) pour en savoir plus sur la publication de votre application.

## <a name="next-steps"></a>Étapes suivantes  

Dans ce guide, vous avez appris à configurer votre environnement de développement et à bien démarrer avec l’exemple d’application d’inscription. Si vous ne connaissez pas bien React Native, vous pouvez lire sa [documentation de prise en main](https://reactnative.dev/docs/getting-started) pour obtenir plus d’informations générales. Il peut également s’avérer utile de vous familiariser avec l’[API Visage](Overview.md). Lisez les autres sections de la documentation sur l’application d’inscription avant de commencer le développement.