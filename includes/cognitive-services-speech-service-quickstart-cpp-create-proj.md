---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 10/15/2020
ms.author: erhopf
ms.openlocfilehash: 967861363d5056a5a466aa88840cda2aafbeb86b
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96188316"
---
Pour créer un projet Visual Studio pour le développement Desktop en C++, vous devez configurer les options de développement de Visual Studio, créer le projet, sélectionner l’architecture cible et installer le SDK Speech.

### <a name="set-up-visual-studio-development-options"></a>Configurer les options de développement de Visual Studio

Pour commencer, vérifiez que Visual Studio est configuré correctement pour le développement Desktop en C++ :

1. Ouvrez Visual Studio 2019 pour afficher la fenêtre **Démarrer**.

   ![Capture d’écran montrant la fenêtre « Démarrer » de Visual Studio 2019.](../articles/cognitive-services/Speech-Service/media/sdk/vs-start-window.png)

1. Sélectionnez **Continuer sans code** pour accéder à l’IDE Visual Studio.

1. Dans la barre de menus de Visual Studio, sélectionnez **Outils** > **Obtenir les outils et fonctionnalités** pour ouvrir Visual Studio Installer et afficher la boîte de dialogue **Modification**.

   ![Capture d’écran montrant l’onglet « Charges de travail » de la boîte de dialogue « Modification » pour « Visual Studio Installer ».](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-cpp-workload.png)

1. Sous l’onglet **Charges de travail**, sous **Windows**, recherchez la charge de travail **Développement Desktop en C++** . Si la case située en regard de cette charge de travail n’est pas déjà cochée, cochez-la.

1. Sous l’onglet **Composants individuels**, recherchez la case à cocher **Gestionnaire de package NuGet**. Si la case n’est pas déjà cochée, cochez-la.

1. Sélectionnez le bouton dans le coin avec le libellé **Fermer** ou **Modifier**. (Le nom du bouton varie selon que vous avez sélectionné ou non des fonctionnalités pour l’installation.) Si vous sélectionnez **Modifier**, l’installation commence ; elle peut prendre un certain temps.

1. Fermez Visual Studio Installer.

### <a name="create-the-project-and-select-the-target-architecture"></a>Créer le projet et sélectionner l’architecture cible

Ensuite, créez votre projet :

1. Dans la barre de menus de Visual Studio, choisissez **Fichier** > **Nouveau** > **Projet** pour afficher la fenêtre **Créer un projet**.

   ![Créer un projet, C++ - Visual Studio](../articles/cognitive-services/Speech-Service/media/sdk/qs-cpp-windows-01-new-console-app.png)

1. Recherchez et sélectionnez **Application console**. Veillez à sélectionner la version C++ de ce type de projet (et non pas la version C# ou Visual Basic).

1. Sélectionnez **Suivant** pour afficher l’écran **Configurer votre nouveau projet**.

   ![Configurer votre nouveau projet, C++ - Visual Studio](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-cpp-configure-your-new-project.png)

1. Dans **Nom du projet**, entrez `helloworld`.

1. Dans **Emplacement**, accédez au dossier où enregistrer votre projet et sélectionnez-le, ou bien créez ce dossier.

Sélectionnez maintenant l’architecture de votre plateforme cible. Dans la barre d’outils de Visual Studio, recherchez la zone de liste déroulante **Plateformes Solution**. (Si vous ne la voyez pas, choisissez **Afficher** > **Barres d’outils** > **Standard** pour afficher la barre d’outils contenant **Plateformes Solution**.) Si vous exécutez Windows 64 bits, choisissez **x64** dans la zone de liste déroulante. Windows 64 bits peut aussi exécuter les applications 32 bits : vous pouvez donc choisir **x86** si vous préférez.

### <a name="install-the-speech-sdk"></a>Installer le Kit de développement logiciel (SDK) Speech

Enfin, installez le [package NuGet du SDK Speech](https://aka.ms/csspeech/nuget) et référencez le SDK Speech dans votre projet :

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur la solution, puis choisissez **Gérer les packages NuGet pour la solution** pour accéder à la fenêtre **NuGet - Solution**.

1. Sélectionnez **Parcourir**.

   ![NuGet - Onglet Solution, Visual Studio](../articles/cognitive-services/Speech-Service/media/sdk/qs-cpp-windows-03-manage-nuget-packages.png)

1. Dans **Source du package**, choisissez **nuget.org**.

1. Dans la zone **Rechercher**, entrez `Microsoft.CognitiveServices.Speech`, puis choisissez ce package une fois qu’il apparaît dans les résultats de la recherche.

   ![Installation du package C++ Microsoft.CognitiveServices.Speech](../articles/cognitive-services/Speech-Service/media/sdk/qs-cpp-windows-04-nuget-install-1.0.0.png)

1. Dans le volet d’état du package en regard des résultats de la recherche, sélectionnez votre projet **helloworld**.

1. Sélectionnez **Installer**.

1. Dans la boîte de dialogue **Aperçu des modifications**, sélectionnez **OK**.

1. Dans la boîte de dialogue **Acceptation de la licence**, affichez la licence, puis sélectionnez **J’accepte**. L’installation du package commence et, quand l’installation est terminée, le volet **Sortie** affiche un message similaire au texte suivant : `Successfully installed 'Microsoft.CognitiveServices.Speech 1.13.0' to helloworld`.
