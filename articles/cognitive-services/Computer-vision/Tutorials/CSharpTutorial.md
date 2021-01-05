---
title: 'Exemple : Explorer une application de traitement d’image en C#'
titleSuffix: Azure Cognitive Services
description: Explorez une application Windows de base qui utilise l’API Vision par ordinateur dans Azure Cognitive Services. Effectuez une reconnaissance OCR, créez des miniatures et utilisez des fonctionnalités visuelles dans une image.
services: cognitive-services
author: PatrickFarley
manager: nolachar
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: sample
ms.date: 04/17/2019
ms.author: pafarley
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 522cf801e7c1bfdd3ed1f452c123a2db701c0c42
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008801"
---
# <a name="sample-explore-an-image-processing-app-with-c"></a>Exemple : Explorer une application de traitement d’image avec C#

Explorez une application Windows de base qui utilise l’API Vision par ordinateur pour effectuer une reconnaissance optique des caractères (OCR), créer des miniatures avec un rognage intelligent, mais aussi détecter, classer, baliser et décrire des caractéristiques visuelles, y compris les visages, dans une image. L’exemple ci-dessous vous permet d’envoyer une URL d’image ou un fichier stocké localement. Vous pouvez utiliser cet exemple open source comme modèle afin de générer votre propre application pour Windows avec l’API Vision par ordinateur et Windows Presentation Foundation (WPF), qui fait partie du .NET Framework.

> [!div class="checklist"]
> * Obtenir l’exemple d’application à partir de GitHub
> * Ouvrir et créer l’exemple d’application dans Visual Studio
> * Exécuter l’exemple d’application et interagir pour réaliser différents scénarios
> * Explorer les différents scénarios inclus dans l’exemple d’application

## <a name="prerequisites"></a>Prérequis

Avant d’explorer l’exemple d’application, assurez-vous que vous avez rempli les conditions préalables suivantes :

* Vous devez avoir [Visual Studio 2015](https://visualstudio.microsoft.com/downloads/) ou une version ultérieure.
* Un abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* Une fois que vous avez votre abonnement Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="créez une ressource Vision par ordinateur"  target="_blank">créer une ressource Vision par ordinateur <span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le Portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application au service Vision par ordinateur. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.

## <a name="get-the-sample-app"></a>Obtenir l’exemple d’application

L’exemple d’application Vision par ordinateur est disponible sur GitHub à partir du référentiel `Microsoft/Cognitive-Vision-Windows`. Ce référentiel inclut également le référentiel `Microsoft/Cognitive-Common-Windows` en tant que sous-module Git. Vous pouvez cloner de manière récursive ce référentiel, y compris le sous-module, en utilisant la commande `git clone --recurse-submodules` à partir de la ligne de commande, ou en utilisant le bureau GitHub.

Par exemple, pour cloner de manière récursive le référentiel correspondant à l’exemple d’application Vision par ordinateur à partir d’une invite de commandes, exécutez la commande suivante :

```Console
git clone --recurse-submodules https://github.com/Microsoft/Cognitive-Vision-Windows.git
```

> [!IMPORTANT]
> Ne téléchargez pas ce référentiel au format ZIP. Git n’inclut pas les sous-modules lors du téléchargement des référentiels au format ZIP.

### <a name="get-optional-sample-images"></a>Obtenir des exemples d’image (facultatif)

Vous pouvez éventuellement utiliser les exemples d’image inclus avec l’exemple d’application [Visage](../../Face/Overview.md), disponible sur GitHub dans le référentiel `Microsoft/Cognitive-Face-Windows`. Cet exemple d’application inclut un dossier, `/Data`, qui contient plusieurs images de personnes. Vous pouvez aussi cloner de manière récursive ce référentiel conformément aux méthodes décrites pour l’exemple d’application Vision par ordinateur.

Par exemple, pour cloner de manière récursive le référentiel correspondant à l’exemple d’application Visage à partir d’une invite de commandes, exécutez la commande suivante :

```Console
git clone --recurse-submodules https://github.com/Microsoft/Cognitive-Face-Windows.git
```

## <a name="open-and-build-the-sample-app-in-visual-studio"></a>Ouvrir et créer l’exemple d’application dans Visual Studio

Vous devez tout d’abord créer l’exemple d’application afin que Visual Studio puisse résoudre les dépendances, avant d’exécuter ou d’explorer l’exemple d’application. Pour ouvrir et générer l’exemple d’application, procédez comme suit :

1. Ouvrez le fichier de solution Visual Studio, `/Sample-WPF/VisionAPI-WPF-Samples.sln`, dans Visual Studio.
1. Assurez-vous que la solution Visual Studio contient deux projets :  

   * SampleUserControlLibrary
   * VisionAPI-WPF-Samples  

   Si le projet SampleUserControlLibrary n’est pas disponible, vérifiez que vous avez cloné de manière récursive le référentiel `Microsoft/Cognitive-Vision-Windows`.
1. Dans Visual Studio, appuyez sur Ctrl+Maj+B ou choisissez **Générer** dans le menu du ruban, puis **Générer la solution** pour générer la solution.

## <a name="run-and-interact-with-the-sample-app"></a>Exécuter l’exemple d’application et interagir

Vous pouvez exécuter l’exemple d’application pour vérifier son interaction avec vous et avec la bibliothèque de client Vision par ordinateur lorsque vous effectuez diverses tâches, comme la génération de miniatures ou le balisage d’images. Pour exécuter l’exemple d’application et interagir, procédez comme suit :

1. Une fois la génération terminée, appuyez sur **F5** ou choisissez **Déboguer** dans le menu du ruban, puis sélectionnez **Démarrer le débogage** pour exécuter l’exemple d’application.
1. Lorsque l’exemple d’application s’affiche, choisissez **Subscription Key Management** (Gestion des clés d’abonnement) dans le volet de navigation pour afficher la page de gestion des clés d’abonnement.
   ![Page Subscription Key Management (Gestion des clés d’abonnement)](../Images/Vision_UI_Subscription.PNG)  
1. Entrez votre clé d’abonnement dans **Clé de l’abonnement**.
1. Entrez l’URL du point de terminaison dans **Point de terminaison**.  
   [!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]
1. Si vous ne souhaitez pas entrer votre clé d’abonnement ni l’URL du point de terminaison lors de la prochaine exécution de l’exemple d’application, sélectionnez **Enregistrer le paramètre** pour enregistrer l’URL du point de terminaison et la clé d’abonnement sur votre ordinateur. Si vous souhaitez supprimer l’URL du point de terminaison et la clé d’abonnement précédemment enregistrées, sélectionnez **Supprimer le paramètre**.

   > [!NOTE]
   > L’exemple d’application utilise le stockage isolé, et `System.IO.IsolatedStorage`, pour stocker votre clé d’abonnement et l’URL du point de terminaison.

1. Sous **Sélectionnez un scénario** dans le volet de navigation, sélectionnez l’un des scénarios actuellement inclus dans l’exemple d’application :  

   | Scénario | Description |
   |----------|-------------|
   |Analyser l’image | Utilise l’opération [Analyze Image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa) (Analyser l’image) pour analyser une image locale ou distante. Vous pouvez choisir les caractéristiques visuelles et la langue de l’analyse, et voir à la fois l’image et les résultats.  |
   |Analyser l’image avec le modèle de domaine | Utilise l’opération [List Domain Specific Models](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fd) (Répertorier des modèles spécifiques à un domaine) pour répertorier les modèles de domaine à partir desquels vous pouvez effectuer une sélection, et l’opération [Recognize Domain Specific Content](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e200) (Reconnaître le contenu spécifique à un domaine) pour analyser une image locale ou distante à l’aide du modèle de domaine sélectionné. Vous pouvez également choisir la langue de l’analyse. |
   |Décrire l’image | Utilise l’opération [Describe Image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fe) (Décrire l’image) pour créer la description explicite d’une image locale ou distante. Vous pouvez également choisir la langue de la description. |
   |Générer des balises | Utilise l’opération [Tag Image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1ff) (Baliser l’image) pour baliser les caractéristiques visuelles d’une image locale ou distante. Vous pouvez également choisir la langue des balises. |
   |Reconnaître le texte (OCR) | Utilise l’opération [OCR](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fc) pour reconnaître et extraire le texte imprimé à partir d’une image. Vous pouvez choisir la langue à utiliser, ou laisser Vision par ordinateur détecter automatiquement la langue. |
   |Reconnaître le texte V2 (en anglais) | Utilise les opérations [Reconnaître le texte](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/587f2c6a154055056008f200) et [Get Recognize Text Operation Result](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/587f2cf1154055056008f201) (Obtenir le résultat de l’opération Reconnaître le texte) pour reconnaître de façon asynchrone et extraire le texte imprimé ou manuscrit à partir d’une image. |
   |Générer une miniature | Utilise l’opération [Get Thumbnail](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fb) (Générer une miniature) afin de générer la miniature d’une image locale ou distante. |

   La capture d’écran suivante illustre la page fournie pour le scénario Analyser l’image, après l’analyse d’un exemple d’image.
   ![Capture d’écran de la page Analyser l’image](../Images/Analyze_Image_Example.PNG)

## <a name="explore-the-sample-app"></a>Explorer l’exemple d’application

La solution Visual Studio de l’exemple d’application Vision par ordinateur contient deux projets :

* SampleUserControlLibrary  
  Le projet SampleUserControlLibrary fournit des fonctionnalités partagées par plusieurs exemples Cognitive Services. Le projet contient les fichiers suivants :
  * SampleScenarios  
    UserControl qui fournit une présentation standardisée, comme la barre de titre, le volet de navigation et le volet de contenu, pour les exemples. L’exemple d’application Vision par ordinateur utilise ce contrôle dans la fenêtre MainWindow.xaml pour afficher les pages de scénario et accéder aux informations partagées dans les scénarios, comme la clé d’abonnement et l’URL de point de terminaison.
  * SubscriptionKeyPage  
    Page qui fournit une disposition standardisée permettant d’entrer une clé d’abonnement et une URL de point de terminaison pour l’exemple d’application. L’exemple d’application Vision par ordinateur se sert de cette page pour gérer la clé d’abonnement et l’URL de point de terminaison utilisées par les pages de scénario.
  * VideoResultControl  
    UserControl qui fournit une présentation standardisée pour les informations vidéo. L’exemple d’application Vision par ordinateur n’utilise pas ce contrôle.
* VisionAPI-WPF-Samples  
  Projet principal pour l’exemple d’application Vision par ordinateur. Ce projet contient toutes les fonctionnalités intéressantes pour Vision par ordinateur. Le projet contient les fichiers suivants :
  * AnalyzeInDomainPage.xaml  
    Page de scénario pour le scénario Analyser l’image avec le modèle de domaine.
  * AnalyzeImage.xaml  
    Page de scénario pour le scénario Analyser l’image.
  * DescribePage.xaml  
    Page de scénario pour le scénario Décrire l’image.
  * ImageScenarioPage.cs  
    Classe ImageScenarioPage dont toutes les pages de scénario de l’exemple d’application sont dérivées. Cette classe gère les fonctionnalités, comme la fourniture d’informations d’identification et le formatage de sortie, partagées par toutes les pages de scénario.
  * MainWindow.xaml  
    Fenêtre principale de l’exemple d’application. Elle utilise le contrôle SampleScenarios pour présenter SubscriptionKeyPage et les pages de scénario.
  * OCRPage.xaml  
    Page de scénario pour le scénario Reconnaître le texte (OCR).
  * RecognizeLanguage.cs  
    Classe RecognizeLanguage, qui fournit des informations sur les langues prises en charge par les différentes méthodes dans l’exemple d’application.
  * TagsPage.xaml  
    Page de scénario pour le scénario Générer des balises.
  * TextRecognitionPage.xaml  
    Page de scénario pour le scénario Reconnaître le texte V2 (en anglais).
  * ThumbnailPage.xaml  
    Page de scénario pour le scénario Générer une miniature.

### <a name="explore-the-sample-code"></a>Explorer l’exemple de code

Les parties clés de l’exemple de code sont accompagnées de blocs de commentaires qui commencent par `KEY SAMPLE CODE STARTS HERE` et se terminent par `KEY SAMPLE CODE ENDS HERE`, afin de faciliter l’exploration de l’exemple d’application. Ces parties clés d’exemple de code contiennent le code pertinent pour apprendre à utiliser la bibliothèque de client Vision par ordinateur en vue d’effectuer diverses tâches. Vous pouvez rechercher `KEY SAMPLE CODE STARTS HERE` dans Visual Studio pour passer d’une section pertinente du code à l’autre dans l’exemple d’application Vision par ordinateur. 

Par exemple, la méthode `UploadAndAnalyzeImageAsync`, illustrée ci-après et incluse dans AnalyzePage.xaml, montre comment utiliser la bibliothèque de client pour analyser une image locale en appelant la méthode `ComputerVisionClient.AnalyzeImageInStreamAsync`.

```csharp
private async Task<ImageAnalysis> UploadAndAnalyzeImageAsync(string imageFilePath)
{
    // -----------------------------------------------------------------------
    // KEY SAMPLE CODE STARTS HERE
    // -----------------------------------------------------------------------

    //
    // Create Cognitive Services Vision API Service client.
    //
    using (var client = new ComputerVisionClient(Credentials) { Endpoint = Endpoint })
    {
        Log("ComputerVisionClient is created");

        using (Stream imageFileStream = File.OpenRead(imageFilePath))
        {
            //
            // Analyze the image for all visual features.
            //
            Log("Calling ComputerVisionClient.AnalyzeImageInStreamAsync()...");
            VisualFeatureTypes[] visualFeatures = GetSelectedVisualFeatures();
            string language = (_language.SelectedItem as RecognizeLanguage).ShortCode;
            ImageAnalysis analysisResult = await client.AnalyzeImageInStreamAsync(imageFileStream, visualFeatures, null, language);
            return analysisResult;
        }
    }

    // -----------------------------------------------------------------------
    // KEY SAMPLE CODE ENDS HERE
    // -----------------------------------------------------------------------
}
```

### <a name="explore-the-client-library"></a>Explorer la bibliothèque de client

Cet exemple d’application se sert de la bibliothèque de client Vision par ordinateur, un simple wrapper de client C# pour Vision par ordinateur dans Azure Cognitive Services. La bibliothèque de client NuGet est disponible dans le package [Microsoft.Azure.CognitiveServices.Vision.ComputerVision](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.ComputerVision/). Lorsque vous avez créé l’application Visual Studio, vous avez récupéré la bibliothèque de client à partir de son package NuGet correspondant. Vous pouvez également afficher le code source pour la bibliothèque de client dans le dossier `/ClientLibrary` du référentiel `Microsoft/Cognitive-Vision-Windows`.

La fonctionnalité de la bibliothèque de client se focalise sur la classe `ComputerVisionClient`, dans l’espace de noms `Microsoft.Azure.CognitiveServices.Vision.ComputerVision`, lorsque les modèles utilisés par la classe `ComputerVisionClient` au moment de l’interaction avec Vision par ordinateur se trouvent dans l’espace de noms `Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models`. Les pages de scénario XAML incluses dans l’exemple d’application contiennent les directives `using` suivantes pour ces espaces de noms :

```csharp
// -----------------------------------------------------------------------
// KEY SAMPLE CODE STARTS HERE
// Use the following namespace for ComputerVisionClient.
// -----------------------------------------------------------------------
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
// -----------------------------------------------------------------------
// KEY SAMPLE CODE ENDS HERE
// -----------------------------------------------------------------------
```

Vous en saurez plus sur les méthodes incluses dans la classe `ComputerVisionClient` lorsque vous explorerez les scénarios inclus dans l’exemple d’application Vision par ordinateur.

## <a name="explore-the-analyze-image-scenario"></a>Explorer le scénario Analyser l’image

Ce scénario est géré par la page AnalyzePage.xaml. Vous pouvez choisir les caractéristiques visuelles et la langue de l’analyse, et voir à la fois l’image et les résultats. La page de scénario utilise pour cela l’une des méthodes suivantes, selon la source de l’image :

* UploadAndAnalyzeImageAsync  
  Cette méthode est utilisée pour les images locales. L’image doit être encodée en tant que `Stream` et envoyée à Vision par ordinateur au moyen d’un appel à la méthode `ComputerVisionClient.AnalyzeImageInStreamAsync`.
* AnalyzeUrlAsync  
  Cette méthode est utilisée pour les images distantes. L’URL de l’image doit être envoyée à Vision par ordinateur au moyen d’un appel à la méthode `ComputerVisionClient.AnalyzeImageAsync`.

La méthode `UploadAndAnalyzeImageAsync` crée une instance `ComputerVisionClient` à l’aide de l’URL de point de terminaison et de la clé d’abonnement spécifiées. Étant donné que l’exemple d’application analyse une image locale, elle doit envoyer le contenu de cette image à Vision par ordinateur. Elle ouvre le fichier local spécifié dans `imageFilePath` pour lecture en tant que `Stream`, puis obtient les caractéristiques visuelles et la langue sélectionnées dans la page de scénario. Elle appelle la méthode `ComputerVisionClient.AnalyzeImageInStreamAsync`, en soumettant `Stream` pour le fichier, les caractéristiques visuelles et la langue, puis retourne le résultat sous la forme d’une instance `ImageAnalysis`. Les méthodes héritées de la classe `ImageScenarioPage` présentent les résultats retournés dans la page de scénario.

La méthode `AnalyzeUrlAsync` crée une instance `ComputerVisionClient` à l’aide de l’URL de point de terminaison et de la clé d’abonnement spécifiées. Elle obtient les caractéristiques visuelles et la langue sélectionnées dans la page de scénario. Elle appelle la méthode `ComputerVisionClient.AnalyzeImageInStreamAsync`, en soumettant l’URL de l’image, les caractéristiques visuelles et la langue, puis retourne le résultat sous la forme d’une instance `ImageAnalysis`. Les méthodes héritées de la classe `ImageScenarioPage` présentent les résultats retournés dans la page de scénario.

## <a name="explore-the-analyze-image-with-domain-model-scenario"></a>Explorer le scénario Analyser l’image avec le modèle de domaine

Ce scénario est géré par la page AnalyzeInDomainPage.xaml. Vous pouvez choisir un modèle de domaine, tel que `celebrities` ou `landmarks`, et la langue pour effectuer une analyse d’image spécifique à un domaine, et voir à la fois l’image et les résultats. La page de scénario utilise les méthodes suivantes, selon la source de l’image :

* GetAvailableDomainModelsAsync  
  Cette méthode permet d’obtenir la liste des modèles de domaine disponibles à partir de Vision par ordinateur, et de remplir le contrôle ComboBox `_domainModelComboBox` sur la page, à l’aide de la méthode `ComputerVisionClient.ListModelsAsync`.
* UploadAndAnalyzeInDomainImageAsync  
  Cette méthode est utilisée pour les images locales. L’image doit être encodée en tant que `Stream` et envoyée à Vision par ordinateur au moyen d’un appel à la méthode `ComputerVisionClient.AnalyzeImageByDomainInStreamAsync`.
* AnalyzeInDomainUrlAsync  
  Cette méthode est utilisée pour les images distantes. L’URL de l’image doit être envoyée à Vision par ordinateur au moyen d’un appel à la méthode `ComputerVisionClient.AnalyzeImageByDomainAsync`.

La méthode `UploadAndAnalyzeInDomainImageAsync` crée une instance `ComputerVisionClient` à l’aide de l’URL de point de terminaison et de la clé d’abonnement spécifiées. Étant donné que l’exemple d’application analyse une image locale, elle doit envoyer le contenu de cette image à Vision par ordinateur. Elle ouvre le fichier local spécifié dans `imageFilePath` pour lecture en tant que `Stream`, puis obtient la langue sélectionnée dans la page de scénario. Elle appelle la méthode `ComputerVisionClient.AnalyzeImageByDomainInStreamAsync`, en soumettant `Stream` pour le fichier, le nom du modèle de domaine et la langue, puis retourne le résultat sous la forme d’une instance `DomainModelResults`. Les méthodes héritées de la classe `ImageScenarioPage` présentent les résultats retournés dans la page de scénario.

La méthode `AnalyzeInDomainUrlAsync` crée une instance `ComputerVisionClient` à l’aide de l’URL de point de terminaison et de la clé d’abonnement spécifiées. Elle obtient la langue sélectionnée dans la page de scénario. Elle appelle la méthode `ComputerVisionClient.AnalyzeImageByDomainAsync`, en soumettant l’URL de l’image, les caractéristiques visuelles et la langue, puis retourne le résultat sous la forme d’une instance `DomainModelResults`. Les méthodes héritées de la classe `ImageScenarioPage` présentent les résultats retournés dans la page de scénario.

## <a name="explore-the-describe-image-scenario"></a>Explorer le scénario Décrire l’image

Ce scénario est géré par la page DescribePage.xaml. Vous pouvez choisir une langue pour créer une description explicite de l’image et voir à la fois l’image et les résultats. La page de scénario utilise les méthodes suivantes, selon la source de l’image :

* UploadAndDescribeImageAsync  
  Cette méthode est utilisée pour les images locales. L’image doit être encodée en tant que `Stream` et envoyée à Vision par ordinateur au moyen d’un appel à la méthode `ComputerVisionClient.DescribeImageInStreamAsync`.
* DescribeUrlAsync  
  Cette méthode est utilisée pour les images distantes. L’URL de l’image doit être envoyée à Vision par ordinateur au moyen d’un appel à la méthode `ComputerVisionClient.DescribeImageAsync`.

La méthode `UploadAndDescribeImageAsync` crée une instance `ComputerVisionClient` à l’aide de l’URL de point de terminaison et de la clé d’abonnement spécifiées. Étant donné que l’exemple d’application analyse une image locale, elle doit envoyer le contenu de cette image à Vision par ordinateur. Elle ouvre le fichier local spécifié dans `imageFilePath` pour lecture en tant que `Stream`, puis obtient la langue sélectionnée dans la page de scénario. Elle appelle la méthode `ComputerVisionClient.DescribeImageInStreamAsync`, en soumettant `Stream` pour le fichier, le nombre maximal de candidats (dans le cas présent, 3) et la langue, puis retourne le résultat sous la forme d’une instance `ImageDescription`. Les méthodes héritées de la classe `ImageScenarioPage` présentent les résultats retournés dans la page de scénario.

La méthode `DescribeUrlAsync` crée une instance `ComputerVisionClient` à l’aide de l’URL de point de terminaison et de la clé d’abonnement spécifiées. Elle obtient la langue sélectionnée dans la page de scénario. Elle appelle la méthode `ComputerVisionClient.DescribeImageAsync`, en soumettant l’URL de l’image, le nombre maximal de candidats (dans le cas présent, 3) et la langue, puis retourne le résultat sous la forme d’une instance `ImageDescription`. Les méthodes héritées de la classe `ImageScenarioPage` présentent les résultats retournés dans la page de scénario.

## <a name="explore-the-generate-tags-scenario"></a>Explorer le scénario Générer des balises

Ce scénario est géré par la page TagsPage.xaml. Vous pouvez choisir une langue pour baliser les caractéristiques visuelles d’une image, et voir à la fois l’image et les résultats. La page de scénario utilise les méthodes suivantes, selon la source de l’image :

* UploadAndGetTagsForImageAsync  
  Cette méthode est utilisée pour les images locales. L’image doit être encodée en tant que `Stream` et envoyée à Vision par ordinateur au moyen d’un appel à la méthode `ComputerVisionClient.TagImageInStreamAsync`.
* GenerateTagsForUrlAsync  
  Cette méthode est utilisée pour les images distantes. L’URL de l’image doit être envoyée à Vision par ordinateur au moyen d’un appel à la méthode `ComputerVisionClient.TagImageAsync`.

La méthode `UploadAndGetTagsForImageAsync` crée une instance `ComputerVisionClient` à l’aide de l’URL de point de terminaison et de la clé d’abonnement spécifiées. Étant donné que l’exemple d’application analyse une image locale, elle doit envoyer le contenu de cette image à Vision par ordinateur. Elle ouvre le fichier local spécifié dans `imageFilePath` pour lecture en tant que `Stream`, puis obtient la langue sélectionnée dans la page de scénario. Elle appelle la méthode `ComputerVisionClient.TagImageInStreamAsync`, en soumettant `Stream` pour le fichier et la langue, puis retourne le résultat sous la forme d’une instance `TagResult`. Les méthodes héritées de la classe `ImageScenarioPage` présentent les résultats retournés dans la page de scénario.

La méthode `GenerateTagsForUrlAsync` crée une instance `ComputerVisionClient` à l’aide de l’URL de point de terminaison et de la clé d’abonnement spécifiées. Elle obtient la langue sélectionnée dans la page de scénario. Elle appelle la méthode `ComputerVisionClient.TagImageAsync`, en soumettant l’URL de l’image et la langue, puis retourne le résultat sous la forme d’une instance `TagResult`. Les méthodes héritées de la classe `ImageScenarioPage` présentent les résultats retournés dans la page de scénario.

## <a name="explore-the-recognize-text-ocr-scenario"></a>Explorer le scénario Reconnaître le texte (OCR)

Ce scénario est géré par la page OCRPage.xaml. Vous pouvez choisir une langue pour reconnaître et extraire le texte imprimé d’une image, et voir à la fois l’image et les résultats. La page de scénario utilise les méthodes suivantes, selon la source de l’image :

* UploadAndRecognizeImageAsync  
  Cette méthode est utilisée pour les images locales. L’image doit être encodée en tant que `Stream` et envoyée à Vision par ordinateur au moyen d’un appel à la méthode `ComputerVisionClient.RecognizePrintedTextInStreamAsync`.
* RecognizeUrlAsync  
  Cette méthode est utilisée pour les images distantes. L’URL de l’image doit être envoyée à Vision par ordinateur au moyen d’un appel à la méthode `ComputerVisionClient.RecognizePrintedTextAsync`.

La méthode `UploadAndRecognizeImageAsync` crée une instance `ComputerVisionClient` à l’aide de l’URL de point de terminaison et de la clé d’abonnement spécifiées. Étant donné que l’exemple d’application analyse une image locale, elle doit envoyer le contenu de cette image à Vision par ordinateur. Elle ouvre le fichier local spécifié dans `imageFilePath` pour lecture en tant que `Stream`, puis obtient la langue sélectionnée dans la page de scénario. Elle appelle la méthode `ComputerVisionClient.RecognizePrintedTextInStreamAsync`, en indiquant que l’orientation n’est pas détectée et en soumettant `Stream` pour le fichier et la langue, puis retourne le résultat sous la forme d’une instance `OcrResult`. Les méthodes héritées de la classe `ImageScenarioPage` présentent les résultats retournés dans la page de scénario.

La méthode `RecognizeUrlAsync` crée une instance `ComputerVisionClient` à l’aide de l’URL de point de terminaison et de la clé d’abonnement spécifiées. Elle obtient la langue sélectionnée dans la page de scénario. Elle appelle la méthode `ComputerVisionClient.RecognizePrintedTextAsync`, en indiquant que l’orientation n’est pas détectée et en soumettant l’URL de l’image et la langue, puis retourne le résultat sous la forme d’une instance `OcrResult`. Les méthodes héritées de la classe `ImageScenarioPage` présentent les résultats retournés dans la page de scénario.

## <a name="explore-the-recognize-text-v2-english-scenario"></a>Explorer le scénario Reconnaître le texte V2 (en anglais)

Ce scénario est géré par la page TextRecognitionPage.xaml. Vous pouvez choisir le mode de reconnaissance et une langue afin de reconnaître de façon asynchrone et d’extraire le texte imprimé ou manuscrit à partir d’une image, et voir à la fois l’image et les résultats. La page de scénario utilise les méthodes suivantes, selon la source de l’image :

* UploadAndRecognizeImageAsync  
  Cette méthode est utilisée pour les images locales. L’image doit être encodée en tant que `Stream` et envoyée à Vision par ordinateur au moyen d’un appel à la méthode `RecognizeAsync` et de la soumission d’un délégué paramétrable pour la méthode `ComputerVisionClient.RecognizeTextInStreamAsync`.
* RecognizeUrlAsync  
  Cette méthode est utilisée pour les images distantes. L’URL de l’image doit être envoyée à Vision par ordinateur au moyen d’un appel à la méthode `RecognizeAsync` et de la soumission d’un délégué paramétrable pour la méthode `ComputerVisionClient.RecognizeTextAsync`.
* RecognizeAsync Cette méthode gère l’appel asynchrone pour les méthodes `UploadAndRecognizeImageAsync` et `RecognizeUrlAsync`, ainsi que l’interrogation des résultats par l’appel à la méthode `ComputerVisionClient.GetTextOperationResultAsync`.

Contrairement aux autres scénarios inclus dans l’exemple d’application Vision par ordinateur, ce scénario est asynchrone. La méthode est appelée pour démarrer le processus, mais une autre méthode est appelée pour vérifier l’état et retourner les résultats de ce processus. Le flux logique de ce scénario est un peu différent de celui des autres scénarios.

La méthode `UploadAndRecognizeImageAsync` ouvre le fichier local spécifié dans `imageFilePath` pour lecture en tant que `Stream`, puis appelle la méthode `RecognizeAsync`, en soumettant :

* Expression lambda pour un délégué asynchrone paramétrable de la méthode `ComputerVisionClient.RecognizeTextInStreamAsync`, avec `Stream` pour le fichier et le mode de reconnaissance en tant que paramètres, dans `GetHeadersAsyncFunc`.
* Expression lambda pour qu’un délégué obtienne la valeur d’en-tête de réponse `Operation-Location`, dans `GetOperationUrlFunc`.

La méthode `RecognizeUrlAsync` appelle la méthode `RecognizeAsync`, en soumettant :

* Expression lambda pour un délégué asynchrone paramétrable de la méthode `ComputerVisionClient.RecognizeTextAsync`, avec l’URL de l’image distante et le mode de reconnaissance en tant que paramètres, dans `GetHeadersAsyncFunc`.
* Expression lambda pour qu’un délégué obtienne la valeur d’en-tête de réponse `Operation-Location`, dans `GetOperationUrlFunc`.

Lorsque la méthode `RecognizeAsync` est terminée, les deux méthodes `UploadAndRecognizeImageAsync` et `RecognizeUrlAsync` retournent le résultat sous forme d’instance `TextOperationResult`. Les méthodes héritées de la classe `ImageScenarioPage` présentent les résultats retournés dans la page de scénario.

La méthode `RecognizeAsync` appelle le délégué paramétrable pour la méthode `ComputerVisionClient.RecognizeTextInStreamAsync` ou `ComputerVisionClient.RecognizeTextAsync` soumise dans `GetHeadersAsyncFunc` et attend la réponse. La méthode appelle ensuite le délégué soumis dans `GetOperationUrlFunc` pour obtenir la valeur d’en-tête de réponse `Operation-Location` à partir de la réponse. Cette valeur correspond à l’URL utilisée pour récupérer les résultats de la méthode soumise dans `GetHeadersAsyncFunc` à partir Vision par ordinateur.

La méthode `RecognizeAsync` appelle ensuite la méthode `ComputerVisionClient.GetTextOperationResultAsync`, en soumettant l’URL récupérée à partir de l’en-tête de réponse `Operation-Location`, pour obtenir l’état et le résultat de la méthode soumise dans `GetHeadersAsyncFunc`. Si l’état n’indique pas que la méthode est exécutée, avec succès ou non, la méthode `RecognizeAsync` appelle `ComputerVisionClient.GetTextOperationResultAsync` 3 fois de plus, en attendant 3 secondes entre chaque appel. La méthode `RecognizeAsync` retourne les résultats à la méthode qui l’a appelée.

## <a name="explore-the-get-thumbnail-scenario"></a>Explorez le scénario Générer une miniature

Ce scénario est géré par la page ThumbnailPage.xaml. Vous pouvez indiquer s’il faut utiliser le rognage intelligent et spécifier la hauteur et la largeur souhaitées, afin de générer une miniature à partir d’une image, et voir à la fois l’image et les résultats. La page de scénario utilise les méthodes suivantes, selon la source de l’image :

* UploadAndThumbnailImageAsync  
  Cette méthode est utilisée pour les images locales. L’image doit être encodée en tant que `Stream` et envoyée à Vision par ordinateur au moyen d’un appel à la méthode `ComputerVisionClient.GenerateThumbnailInStreamAsync`.
* ThumbnailUrlAsync  
  Cette méthode est utilisée pour les images distantes. L’URL de l’image doit être envoyée à Vision par ordinateur au moyen d’un appel à la méthode `ComputerVisionClient.GenerateThumbnailAsync`.

La méthode `UploadAndThumbnailImageAsync` crée une instance `ComputerVisionClient` à l’aide de l’URL de point de terminaison et de la clé d’abonnement spécifiées. Étant donné que l’exemple d’application analyse une image locale, elle doit envoyer le contenu de cette image à Vision par ordinateur. Elle ouvre le fichier local spécifié dans `imageFilePath` pour lecture en tant que `Stream`. Elle appelle la méthode `ComputerVisionClient.GenerateThumbnailInStreamAsync`, en soumettant la largeur, la hauteur, `Stream` pour le fichier et l’utilisation éventuelle du rognage intelligent, puis retourne le résultat en tant que `Stream`. Les méthodes héritées de la classe `ImageScenarioPage` présentent les résultats retournés dans la page de scénario.

La méthode `RecognizeUrlAsync` crée une instance `ComputerVisionClient` à l’aide de l’URL de point de terminaison et de la clé d’abonnement spécifiées. Elle appelle la méthode `ComputerVisionClient.GenerateThumbnailAsync`, en soumettant la largeur, la hauteur, l’URL de l’image et l’utilisation éventuelle du rognage intelligent, puis retourne le résultat en tant que `Stream`. Les méthodes héritées de la classe `ImageScenarioPage` présentent les résultats retournés dans la page de scénario.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le dossier dans lequel vous avez cloné le référentiel `Microsoft/Cognitive-Vision-Windows`. Si vous avez choisi d’utiliser les exemples d’image, supprimez également le dossier dans lequel vous avez cloné le référentiel `Microsoft/Cognitive-Face-Windows`.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Bien démarrer avec le service Visage](../../Face/Tutorials/FaceAPIinCSharpTutorial.md)
