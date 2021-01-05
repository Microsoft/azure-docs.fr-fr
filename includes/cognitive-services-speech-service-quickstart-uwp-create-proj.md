---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 10/15/2020
ms.author: erhopf
ms.openlocfilehash: df9dbdf61dbc7d83f1cf74871ddc864488ed9d47
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96188223"
---
Pour créer un projet Visual Studio pour le développement UWP (Plateforme Windows universelle), vous devez configurer les options de développement de Visual Studio, créer le projet, sélectionner l’architecture cible, configurer la capture audio et installer le SDK Speech.

### <a name="set-up-visual-studio-development-options"></a>Configurer les options de développement de Visual Studio

Pour commencer, vérifiez que Visual Studio est configuré correctement pour le développement UWP :

1. Ouvrez Visual Studio 2019 pour afficher la fenêtre **Démarrer**.

   ![Capture d’écran montrant la fenêtre « Démarrer » avec l’action « Continuer sans code » en surbrillance.](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-uwp-start-window.png)

1. Sélectionnez **Continuer sans code** pour accéder à l’IDE Visual Studio.

1. Dans la barre de menus de Visual Studio, sélectionnez **Outils** > **Obtenir les outils et fonctionnalités** pour ouvrir Visual Studio Installer et afficher la boîte de dialogue **Modification**.

   ![Capture d’écran montrant l’onglet « Charges de travail » de la boîte de dialogue « Modification » avec l’option « Activer le développement pour la plateforme Windows universelle » en surbrillance.](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-uwp-workload.png)

1. Sous l’onglet **Charges de travail**, sous **Windows**, recherchez la charge de travail **Développement pour la plateforme Windows universelle**. Si la case à cocher située en regard de cette charge de travail est déjà sélectionnée, fermez la boîte de dialogue **Modification** et passez à l’étape 6.

1. Cochez la case **Développement pour la plateforme Windows universelle**, sélectionnez **Modifier** puis, dans la boîte de dialogue **Avant que nous commencions**, sélectionnez **Continuer** pour installer la charge de travail de développement UWP. L’installation de la nouvelle fonctionnalité peut prendre quelques instants.

1. Fermez Visual Studio Installer.

### <a name="create-the-project-and-select-the-target-architecture"></a>Créer le projet et sélectionner l’architecture cible

Ensuite, créez votre projet :

1. Dans la barre de menus de Visual Studio, choisissez **Fichier** > **Nouveau** > **Projet** pour afficher la fenêtre **Créer un projet**.

   ![Capture d’écran montrant la fenêtre « Créer un projet » avec l’option « Application vide (Windows universel) » sélectionnée et le bouton « Suivant » en surbrillance.](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-uwp-create-new-project.png)

1. Recherchez et sélectionnez **Application vide (Windows universelle)** . Veillez à sélectionner la version C# de ce type de projet (et non pas la version Visual Basic).

1. Sélectionnez **Suivant** pour afficher l’écran **Configurer votre nouveau projet**.

   ![Capture d’écran montrant l’écran « Configurer votre nouveau projet » avec les champs « Nom du projet » et « Emplacement », ainsi que le bouton « Créer » en surbrillance.](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-uwp-configure-your-new-project.png)

1. Dans **Nom du projet**, entrez `helloworld`.

1. Dans **Emplacement**, accédez au dossier où enregistrer votre projet et sélectionnez-le, ou bien créez ce dossier.

1. Sélectionnez **Créer** pour accéder à la fenêtre **Nouveau projet de plateforme Windows universelle**.

   ![Capture d’écran montrant la boîte de dialogue « Nouveau projet de plateforme Windows universelle ».](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-02-new-uwp-project.png)

1. Dans **Version minimale** (la deuxième zone de liste déroulante), choisissez **Windows 10 Fall Creators Update (10.0 ; Build 16299)** , qui est le minimum nécessaire pour le SDK Speech.

1. Dans **Version cible** (la première zone de liste déroulante), choisissez une valeur identique ou ultérieure à la valeur figurant dans **Version minimale**.

1. Sélectionnez **OK**. Vous êtes redirigé vers l’IDE Visual Studio, avec le nouveau projet créé et visible dans le volet **Explorateur de solutions**.

   ![Projet helloworld - Visual Studio](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-uwp-helloworld.png)

Sélectionnez maintenant l’architecture de votre plateforme cible. Dans la barre d’outils de Visual Studio, recherchez la zone de liste déroulante **Plateformes Solution**. (Si vous ne la voyez pas, choisissez **Afficher** > **Barres d’outils** > **Standard** pour afficher la barre d’outils contenant **Plateformes Solution**.) Si vous exécutez Windows 64 bits, choisissez **x64** dans la zone de liste déroulante. Windows 64 bits peut aussi exécuter les applications 32 bits : vous pouvez donc choisir **x86** si vous préférez.

> [!NOTE]
> Le SDK Speech prend en charge tous les processeurs compatibles Intel, mais **uniquement les versions x64** des processeurs ARM.

### <a name="set-up-audio-capture"></a>Configurer la capture audio

Autorisez le projet à capturer l’entrée audio :

1. Dans l’**Explorateur de solutions**, double-cliquez sur **Package.appxmanifest** pour ouvrir le manifeste de l’application de package.

1. Sélectionnez l’onglet **Fonctionnalités**.

   ![Onglet Fonctionnalités, Manifeste d’application de package - Visual Studio](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-07-capabilities.png)

1. Cochez la case pour la fonctionnalité **Microphone**.

1. Dans la barre de menus, choisissez **Fichier** > **Enregistrer Package.appxmanifest** pour enregistrer vos modifications.

### <a name="install-the-speech-sdk"></a>Installer le Kit de développement logiciel (SDK) Speech

Enfin, installez le [package NuGet du SDK Speech](https://aka.ms/csspeech/nuget) et référencez le SDK Speech dans votre projet :

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur la solution, puis choisissez **Gérer les packages NuGet pour la solution** pour accéder à la fenêtre **NuGet - Solution**.

1. Sélectionnez **Parcourir**.

   ![Capture d’écran montrant la boîte de dialogue « Gérer les packages de la solution » avec l’onglet « Parcourir », la zone « Rechercher » et l’option « Source du package » en surbrillance.](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-uwp-nuget-solution-browse.png)

1. Dans **Source du package**, choisissez **nuget.org**.

1. Dans la zone **Rechercher**, entrez `Microsoft.CognitiveServices.Speech`, puis choisissez ce package une fois qu’il apparaît dans les résultats de la recherche.

   ![Capture d’écran montrant « Microsoft.CognitiveServices.Speech » sélectionné, avec le projet et le bouton « Installer » en surbrillance.](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-05-nuget-install-1.0.0.png)

1. Dans le volet d’état du package en regard des résultats de la recherche, sélectionnez votre projet **helloworld**.

1. Sélectionnez **Installer**.

1. Dans la boîte de dialogue **Aperçu des modifications**, sélectionnez **OK**.

1. Dans la boîte de dialogue **Acceptation de la licence**, affichez la licence, puis sélectionnez **J’accepte**. L’installation du package commence et, quand l’installation est terminée, le volet **Sortie** affiche un message similaire au texte suivant : `Successfully installed 'Microsoft.CognitiveServices.Speech 1.13.0' to helloworld`.
