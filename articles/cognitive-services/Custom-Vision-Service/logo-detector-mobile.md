---
title: 'Tutoriel : Utiliser le détecteur de logo personnalisé pour reconnaître les services Azure - Vision personnalisée'
titleSuffix: Azure Cognitive Services
description: Dans ce tutoriel, vous allez examiner un exemple d’application qui utilise Custom Vision dans le cadre d’un scénario de détection de logo. Découvrez comment le service Vision personnalisée est utilisé avec d’autres composants pour fournir une application de bout en bout.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: tutorial
ms.date: 11/23/2020
ms.author: pafarley
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: e5e344aa84bfc6912266720083f0cd15f575883c
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95740248"
---
# <a name="tutorial-recognize-azure-service-logos-in-camera-pictures"></a>Tutoriel : Reconnaître les logos des services Azure dans les images de l’appareil photo

Dans ce tutoriel, vous allez examiner un exemple d’application qui utilise Custom Vision dans le cadre d’un scénario plus vaste. L’application AI Visual Provision, une application Xamarin.Forms pour plateformes mobiles, analyse les images d’appareil photo des logos des services Azure, puis déploie les services eux-mêmes sur le compte Azure de l’utilisateur. Ici, vous verrez comment elle utilise Custom Vision en coordination avec d’autres composants pour fournir une application de bout en bout utile. Vous pouvez exécuter l’intégralité du scénario de l’application pour vous-même, ou effectuer uniquement la partie Custom Vision de l’installation et examiner l’usage qu’en fait l’application.

Ce didacticiel vous explique comment :

> [!div class="checklist"]
> - Créer un détecteur d’objets personnalisé pour reconnaître les logos des services Azure.
> - Connecter votre application aux services Azure Vision par ordinateur et Custom Vision.
> - Créer un compte de principal de service Azure pour déployer des services Azure à partir de l’application.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer. 

## <a name="prerequisites"></a>Prérequis

- [Visual Studio 2017 ou version ultérieure](https://www.visualstudio.com/downloads/)
- La charge de travail Xamarin pour Visual Studio (voir [Installation de Xamarin](/xamarin/cross-platform/get-started/installation/windows))
- Un émulateur iOS ou Android pour Visual Studio
- [Azure CLI](/cli/azure/install-azure-cli-windows?view=azure-cli-latest) (facultatif)

## <a name="get-the-source-code"></a>Obtenir le code source

Pour utiliser l’application web fournie, clonez ou téléchargez le code source de l’application à partir du dépôt [AI Visual Provision](https://github.com/Microsoft/AIVisualProvision) sur GitHub. Ouvrez le fichier *Source/VisualProvision.sln* dans Visual Studio. Par la suite, vous allez modifier certains des fichiers du projet afin de pouvoir exécuter l’application.

## <a name="create-an-object-detector"></a>Créer un détecteur d’objet

Connectez-vous au [site web du service Vision personnalisée](https://customvision.ai/) et créez un nouveau projet. Spécifiez un projet de détection d’objets et utilisez le domaine Logo ; le service utilisera ainsi un algorithme optimisé pour la détection des logos. 

![Fenêtre Nouveau projet sur le site web du service Custom Vision dans le navigateur Chrome](media/azure-logo-tutorial/new-project.png)

## <a name="upload-and-tag-images"></a>Charger et étiqueter des images

Ensuite, entraînez l’algorithme de détection de logos en chargeant les images des logos des services Azure et en les marquant manuellement. Le dépôt AIVisualProvision inclut un ensemble d’images d’entraînement que vous pouvez utiliser. Sur le site web, sélectionnez le bouton **Ajouter des images** sous l’onglet **Images d’entraînement**. Accédez ensuite au dossier **Documents/Images/Training_DataSet** du dépôt. Vous devez marquer manuellement les logos dans chaque image. Ainsi, si vous testez seulement ce projet, vous pouvez ne charger qu’un sous-ensemble des images. Chargez au moins 15 instances de chaque balise que vous prévoyez d’utiliser.

Une fois que vous avez chargé les images d’entraînement, sélectionnez la première affichée. La fenêtre d’étiquetage s’affiche. Dessinez des rectangles et assignez des balises pour chaque logo dans chaque image. 

![Marquage des logos sur le site web de Custom Vision](media/azure-logo-tutorial/tag-logos.png)

L’application est configurée pour fonctionner avec des chaînes de balises spécifiques. Vous trouverez les définitions dans le fichier *Source\VisualProvision\Services\Recognition\RecognitionService.cs* :

[!code-csharp[Tag definitions](~/AIVisualProvision/Source/VisualProvision/Services/Recognition/RecognitionService.cs?name=snippet_constants)]

Après avoir marqué une image, passez à droite pour marquer la suivante. Fermez la fenêtre de marquage quand vous avez terminé.

## <a name="train-the-object-detector"></a>Entraîner le détecteur d’objet

Dans le volet gauche, affectez au commutateur **Balises** la valeur **Marqué** pour afficher vos images. Ensuite, sélectionnez le bouton vert en haut de la page pour entraîner le modèle. L’algorithme s’entraîne à reconnaître les mêmes étiquettes dans de nouvelles images. Cela teste également le modèle sur certaines de vos images existantes pour générer des scores de précision.

![Site web Custom Vision sous l’onglet Images d’entraînement. Dans cette capture d’écran, le bouton Entraîner est mis en surbrillance.](media/azure-logo-tutorial/train-model.png)

## <a name="get-the-prediction-url"></a>Obtenir l’URL de prédiction

Une fois votre modèle entraîné, vous êtes prêt à l’intégrer dans votre application. Vous devez obtenir l’URL de point de terminaison (l’adresse de votre modèle, que l’application va interroger) et la clé de prédiction (pour permettre à l’application d’accéder aux demandes de prédiction). Sous l’onglet **Performances**, sélectionnez le bouton **URL de prédiction** en haut de la page.

![Site web Custom Vision, montrant une fenêtre de l’API de prédiction qui présente une adresse URL et une clé API](media/azure-logo-tutorial/cusvis-endpoint.png)

Copiez l’URL du point de terminaison et la valeur **Prediction-Key** dans les champs appropriés du fichier *Source\VisualProvision\AppSettings.cs* :

[!code-csharp[Custom Vision fields](~/AIVisualProvision/Source/VisualProvision/AppSettings.cs?name=snippet_cusvis_keys)]

## <a name="examine-custom-vision-usage"></a>Examiner l’utilisation du service Vision personnalisée

Ouvrez le fichier *Source/VisualProvision/Services/Recognition/CustomVisionService.cs* pour voir comment l’application utilise votre clé Custom Vision et votre URL de point de terminaison. La méthode **PredictImageContentsAsync** prend un flux d’octets d’un fichier image et un jeton d’annulation (pour la gestion des tâches asynchrones), appelle l’API de prédiction du service Vision personnalisée et retourne le résultat de la prédiction. 

[!code-csharp[Custom Vision fields](~/AIVisualProvision/Source/VisualProvision/Services/Recognition/CustomVisionService.cs?name=snippet_prediction)]

Ce résultat prend la forme d’une instance **PredictionResult**, qui contient elle-même une liste d’instances **Prediction**. Une **prédiction** contient une balise détectée et l’emplacement de son cadre englobant dans l’image.

[!code-csharp[Custom Vision fields](~/AIVisualProvision/Source/VisualProvision/Services/Recognition/Prediction.cs?name=snippet_prediction_class)]

Pour en savoir plus sur la façon dont l’application gère ces données, démarrez avec la méthode **GetResourcesAsync**. Cette méthode est définie dans le fichier *Source/VisualProvision/Services/Recognition/RecognitionService.cs*.  

## <a name="add-computer-vision"></a>Ajouter le service Vision par ordinateur

La partie Custom Vision du tutoriel est terminée. Si vous voulez exécuter l’application, vous devez également intégrer le service Vision par ordinateur. L’application utilise la fonctionnalité de reconnaissance de texte de Vision par ordinateur pour compléter le processus de détection de logos. Un logo Azure est reconnaissable à son aspect *ou* au texte apposé en regard. Contrairement aux modèles Custom Vision, le service Vision par ordinateur est déjà entraîné pour effectuer certaines opérations sur les images ou vidéos.

Abonnez-vous au service Vision par ordinateur pour obtenir une clé et une URL de point de terminaison. Pour obtenir de l’aide sur cette étape, consultez [Comment obtenir des clés d’abonnement](../cognitive-services-apis-create-account.md?tabs=singleservice%2Cwindows).

![Service Vision par ordinateur dans le portail Azure. Menu Démarrage rapide sélectionné. Un lien est entouré pour les clés, ainsi que l’URL de point de terminaison de l’API](media/azure-logo-tutorial/comvis-keys.png)

Ensuite, ouvrez le fichier *Source\VisualProvision\AppSettings.cs* et affectez des valeurs correctes aux variables `ComputerVisionEndpoint` et `ComputerVisionKey`.

[!code-csharp[Computer Vision fields](~/AIVisualProvision/Source/VisualProvision/AppSettings.cs?name=snippet_comvis_keys)]

## <a name="create-a-service-principal"></a>Créer un principal du service

L’application nécessite un compte de principal de service Azure pour déployer des services dans le cadre de votre abonnement Azure. Un principal de service vous permet de déléguer des autorisations spécifiques à une application à l’aide du contrôle d’accès en fonction du rôle Azure. Pour en savoir plus, consultez le [guide des principaux du service](/azure-stack/operator/azure-stack-create-service-principals).

Vous pouvez créer un principal du service à l’aide d’Azure Cloud Shell ou de l’interface de ligne de commande Azure, comme indiqué ici. Pour commencer, connectez-vous et sélectionnez l’abonnement à utiliser.

```azurecli
az login
az account list
az account set --subscription "<subscription name or subscription id>"
```

Créez ensuite votre principal du service. (Ce processus peut être relativement long.)

```azurecli
az ad sp create-for-rbac --name <servicePrincipalName> --password <yourSPStrongPassword>
```

Après cela, vous devriez voir la sortie JSON suivante contenant les informations d’identification nécessaires.

```json
{
  "clientId": "(...)",
  "clientSecret": "(...)",
  "subscriptionId": "(...)",
  "tenantId": "(...)",
  ...
}
```

Notez les valeurs `clientId` et `tenantId`. Ajoutez-les dans les champs appropriés, dans le fichier *Source\VisualProvision\AppSettings.cs*.

[!code-csharp[Computer Vision fields](~/AIVisualProvision/Source/VisualProvision/AppSettings.cs?name=snippet_serviceprincipal)]

## <a name="run-the-app"></a>Exécuter l’application

À ce stade, vous avez donné à l’application l’accès :

- à un modèle Custom Vision entraîné
- au service Vision par ordinateur
- à un compte de principal du service

Pour exécuter l’application, procédez comme suit :

1. Dans l’Explorateur de solutions Visual Studio, sélectionnez le projet **VisualProvision.Android** ou le **VisualProvision.iOS**. Choisissez un émulateur correspondant ou un appareil mobile connecté dans le menu déroulant de la barre d’outils principale. Ensuite, exécutez l’application.

    > [!NOTE]
    > Vous avez besoin d’un appareil MacOS pour exécuter un émulateur iOS.

1. Dans le premier écran, entrez votre ID client de principal du service, votre ID de locataire et votre mot de passe. Sélectionnez le bouton de **connexion**.

    > [!NOTE]
    > Sur certains émulateurs, le bouton de **connexion** peut ne pas être activé à ce stade. Le cas échéant, arrêtez l’application, ouvrez le fichier *Source/VisualProvision/Pages/LoginPage.xaml*, recherchez l’élément `Button` étiqueté **LOGIN BUTTON**, supprimez la ligne suivante, puis réexécutez l’application.
    >  ```xaml
    >  IsEnabled="{Binding IsValid}"
    >  ```
    
    ![Écran de l’application avec des champs pour les informations d’identification du principal du service](media/azure-logo-tutorial/app-credentials.png)

1. Dans l’écran suivant, sélectionnez votre abonnement Azure dans le menu déroulant. (Ce menu doit contenir tous les abonnements auxquels votre principal du service a accès.) Sélectionnez le bouton **Continuer**. À ce stade, l’application peut vous demander d’octroyer l’accès à l’appareil photo et au stockage de photos de l’appareil. Accordez ces autorisations d’accès.

    ![Écran de l’application avec un champ de liste déroulante pour l’abonnement Azure cible](media/azure-logo-tutorial/app-az-subscription.png)


1. L’appareil photo est activé sur votre appareil. Prenez une photo de l’un des logos des services Azure que vous avez utilisés pour l’entraînement. Une fenêtre de déploiement doit vous inviter à sélectionner une région et un groupe de ressources pour les nouveaux services (comme vous le feriez si vous les déployiez à partir du portail Azure). 

    ![Écran de l’appareil photo du smartphone avec deux découpages papier des logos Azure](media/azure-logo-tutorial/app-camera-capture.png)

    ![Écran de l’application montrant les champs pour la région de déploiement et le groupe de ressources](media/azure-logo-tutorial/app-deployment-options.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez suivi toutes les étapes de ce scénario et utilisé l’application pour déployer des services Azure sur votre compte, accédez au [portail Azure](https://ms.portal.azure.com/). Là, annulez les services que vous ne voulez pas utiliser.

Si vous envisagez de créer votre propre projet de détection d’objets avec Custom Vision, vous pouvez supprimer le projet de détection de logos que vous avez créé dans ce tutoriel. Un abonnement gratuit à Custom Vision permet d’utiliser deux projets uniquement. Pour supprimer le projet de détection de logos, sur le [site web Custom Vision](https://customvision.ai), ouvrez **Projets**, puis sélectionnez l’icône de corbeille sous **Mon nouveau projet**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez configuré et exploré une application Xamarin.Forms complète qui a recours au service Custom Vision pour détecter des logos dans les images de l’appareil photo d’un téléphone portable. Découvrez ensuite les bonnes pratiques de création d’un modèle Custom Vision pour en créer un efficace et précis pour votre propre application.

> [!div class="nextstepaction"]
> [Guide pratique pour améliorer votre classifieur](getting-started-improving-your-classifier.md)