---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 10/15/2020
ms.author: erhopf
ms.openlocfilehash: 6d20df031633df4642ce9fb5cbbc469fd7f0a5da
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96188326"
---
Pour créer un projet Visual Studio pour le développement .NET d’applications mobiles multiplateformes avec Xamarin, vous devez :
- Configurer les options de développement de Visual Studio.
- Créer le projet et sélectionner l’architecture cible. 
- Installer le kit SDK Speech.

### <a name="set-up-visual-studio-development-options"></a>Configurer les options de développement de Visual Studio

Pour commencer, vérifiez que la configuration de Visual Studio est correcte pour le développement mobile multiplateforme avec .NET :

1. Ouvrez Visual Studio 2019.

1. Dans la barre de menus de Visual Studio, sélectionnez **Outils** > **Obtenir les outils et fonctionnalités** pour ouvrir Visual Studio Installer et afficher la boîte de dialogue **Modification**.

   ![Capture d’écran montrant l’onglet Charges de travail, la boîte de dialogue Modification et Visual Studio Installer](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-xamarin-workload.png)

1. Sous l’onglet **Charges de travail**, sous **Windows**, recherchez la charge de travail **Développement mobile en .NET**. Si la case à côté de cette charge de travail est déjà cochée, fermez la boîte de dialogue **Modification** et passez à l’étape 5.

1. Cochez la case **Développement mobile en .NET**, puis sélectionnez **Modifier**. Dans la boîte de dialogue **Avant de commencer**, sélectionnez **Continuer** pour installer la charge de travail Développement mobile en .NET. L’installation de la nouvelle fonctionnalité peut prendre quelques instants.

1. Fermez Visual Studio Installer.

### <a name="create-the-project"></a>Créer le projet

1. Dans la barre de menus Visual Studio, sélectionnez **Fichier** > **Nouveau** > **Projet** pour afficher la fenêtre **Créer un projet**.

   ![Capture d’écran montrant comment créer un projet dans Visual Studio.](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-xamarin-create-new-project.png)

1. Recherchez et sélectionnez **Application mobile (Xamarin.Forms)** .

1. Sélectionnez **Suivant** pour afficher l’écran **Configurer votre nouveau projet**.

   ![Capture d’écran montrant comment configurer votre nouveau projet dans Visual Studio.](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-xamarin-configure-your-new-project.png)

1. Dans **Nom du projet**, entrez *helloworld*.

1. Dans **Emplacement**, accédez au dossier où enregistrer votre projet et sélectionnez-le, ou bien créez ce dossier.

1. Sélectionnez **Créer** pour accéder à la fenêtre **Nouveau projet Xamarin.Forms Mobile App**.

   ![Capture d’écran montrant la boîte de dialogue Nouveau projet Xamarin.Forms Mobile App dans Visual Studio.](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-xamarin-new-xamarin-project.png)

1. Sélectionnez le modèle **Vide**.

1. Dans **Plateforme**, cochez les cases **Android**, **iOS** et **Windows (UWP)** .

1. Sélectionnez **OK**. Vous êtes redirigé vers l’IDE Visual Studio, avec le nouveau projet créé et visible dans le volet **Explorateur de solutions**.

   ![Projet helloworld - Visual Studio](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-xamarin-helloworld.png)

Sélectionnez maintenant l’architecture de votre plateforme cible et votre projet de démarrage. Dans la barre d’outils Visual Studio, recherchez la zone de liste déroulante **Plateformes Solution**. (Si vous ne la voyez pas, sélectionnez **Afficher** > **Barres d’outils** > **Standard** pour afficher la barre d’outils qui contient **Plateformes Solution**.) Si vous exécutez Windows 64 bits, sélectionnez **x64** dans la zone de liste déroulante. Vous pouvez sélectionner **x86** si vous le souhaitez, car Windows 64 bits peut également exécuter des applications 32 bits. Dans la zone de liste déroulante **Projets de démarrage**, définissez **helloworld.UWP (Windows universel)** .

### <a name="install-the-speech-sdk"></a>Installer le Kit de développement logiciel (SDK) Speech

Installez le [package NuGet du SDK Speech](https://aka.ms/csspeech/nuget), puis référencez le SDK Speech dans votre projet.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur votre solution. Sélectionnez **Gérer les packages NuGet pour la solution** pour accéder à la fenêtre **NuGet - Solution**.

1. Sélectionnez **Parcourir**.

   ![Capture d’écran de la boîte de dialogue Gérer les packages de la solution lors de l’installation du SDK Speech.](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-uwp-nuget-solution-browse.png)

1. Dans **Source du package**, sélectionnez nuget.org.

1. Dans la zone de **recherche**, entrez *Microsoft.CognitiveServices.Speech*. Sélectionnez ensuite ce package une fois qu’il apparaît dans les résultats de la recherche.

   ![Capture d’écran avec mise en évidence du package Microsoft.CognitiveServices.Speech.](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-xamarin-nuget-install.png)

   > [!NOTE] 
   > Bitcode n’est pas activé pour la bibliothèque iOS à l’intérieur du package NuGet `Microsoft.CognitiveServices.Speech`. Si vous avez besoin d’une bibliothèque avec Bitcode activé pour votre application, utilisez le package NuGet `Microsoft.CognitiveServices.Speech.Xamarin.iOS` spécifiquement pour le projet iOS.

1. Dans le volet d’état du package en regard des résultats de la recherche, sélectionnez tous les projets : **helloworld**, **helloworld.Android**, **helloworld.iOS** et **helloworld.UWP**.

1. Sélectionnez **Installer**.

1. Dans la boîte de dialogue **Aperçu des modifications**, sélectionnez **OK**.

1. Dans la boîte de dialogue **Acceptation de la licence**, affichez la licence, puis sélectionnez **J’accepte**. Installez la référence du package du SDK Speech à tous les projets. Une fois l’installation terminée, l’avertissement suivant peut s’afficher pour helloworld.iOS. Il s’agit d’un problème connu qui ne doit normalement pas avoir d’impact sur les fonctionnalités de votre application.

   > Impossible de résoudre la référence « C:\Users\Default\.nuget\packages\microsoft.cognitiveservices.speech\1.7.0\build\Xamarin.iOS\libMicrosoft.CognitiveServices.Speech.core.a ». Si cette référence est nécessaire à votre code, des erreurs de compilation peuvent se produire.
