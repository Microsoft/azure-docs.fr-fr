---
author: wolfma61
ms.service: cognitive-services
ms.topic: include
ms.date: 08/30/2019
ms.author: wolfma
ms.openlocfilehash: c7332177795a45331749225e0f3dc3856e28a6a2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001091"
---
Pour créer un projet Visual Studio pour le développement Windows, vous devez créer le projet, configurer Visual Studio pour le développement de bureau .NET, installer le SDK Speech et choisir l’architecture cible.

### <a name="create-the-project-and-add-the-workload"></a>Créer le projet et ajouter la charge de travail

Pour commencer, créez le projet dans Visual Studio et assurez-vous que Visual Studio est configuré pour le développement de bureau .NET :

1. Ouvrez Visual Studio 2019.

1. Dans la fenêtre Démarrer, sélectionnez **Créer un projet**. 

1. Dans la fenêtre **Créer un projet**, choisissez **Application console (.NET Framework)** , puis cliquez sur **Suivant**.

1. Dans la fenêtre **Configurer votre nouveau projet**, entrez *helloworld* dans **Nom du projet**, choisissez ou créez le chemin du répertoire dans **Emplacement**, puis sélectionnez **Créer**.

1. Dans la barre de menus de Visual Studio, sélectionnez **Outils** > **Obtenir les outils et fonctionnalités**, ce qui a pour effet d’ouvrir Visual Studio Installer et d’afficher la boîte de dialogue **Modification**.

1. Vérifiez si la charge de travail **Développement .NET Desktop** est disponible. Si cette charge de travail n’a pas été installée, cochez la case en regard de celle-ci, puis sélectionnez **Modifier** pour démarrer l’installation. Le téléchargement et l’installation peuvent prendre quelques minutes.

   Si la case à cocher à côté de **Développement .NET Desktop** est déjà sélectionnée, sélectionnez **Fermer** pour quitter la boîte de dialogue.

   ![Activer un développement de bureau .NET](../articles/cognitive-services/speech-service/media/sdk/vs-enable-net-desktop-workload.png)

1. Fermez Visual Studio Installer.

### <a name="install-the-speech-sdk"></a>Installer le Kit de développement logiciel (SDK) Speech

L’étape suivante consiste à installer le [package NuGet du SDK Speech](https://aka.ms/csspeech/nuget) afin de pouvoir le référencer dans le code.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **helloworld**, puis sélectionnez **Gérer les packages NuGet** pour afficher le gestionnaire de package NuGet.

   ![Gestionnaire de package NuGet](../articles/cognitive-services/speech-service/media/sdk/vs-nuget-package-manager.png)

1. Dans l’angle supérieur droit, recherchez la zone de liste déroulante **Source du package** et vérifiez que **nuget.org** est sélectionné.

1. Dans l’angle supérieur gauche, sélectionnez **Parcourir.**

1. Dans la zone de recherche, tapez *Microsoft.CognitiveServices.Speech* et appuyez sur la touche **Entrée**.

1. Dans les résultats de la recherche, sélectionnez le package **Microsoft.CognitiveServices.Speech**, puis sélectionnez **Installer** pour installer la dernière version stable.

   ![Installer le package NuGet Microsoft.CognitiveServices.Speech](../articles/cognitive-services/speech-service/media/sdk/qs-csharp-dotnet-windows-03-nuget-install-1.0.0.png)

1. Acceptez tous les contrats et licences pour démarrer l’installation.

   Une fois le package installé, un message de confirmation s’affiche dans la fenêtre **Console du Gestionnaire de package**.

### <a name="choose-the-target-architecture"></a>Choisir l’architecture cible

Maintenant, pour générer et exécuter l’application console, créez une configuration de plateforme correspondant à l’architecture de votre ordinateur.

1. Dans la barre de menus, sélectionnez **Générer** > **Gestionnaire de configurations**. La boîte de dialogue **Gestionnaire de configurations** s’affiche.

   ![Boîte de dialogue Gestionnaire de configurations](../articles/cognitive-services/speech-service/media/sdk/vs-configuration-manager-dialog-box.png)

1. Dans la zone de liste déroulante **Plateforme de la solution active**, sélectionnez **Nouveau**. La boîte de dialogue **Nouvelle plateforme de solution** s’affiche.

1. Dans la zone de liste déroulante **Tapez ou sélectionnez la nouvelle plateforme** :
   - Si vous exécutez Windows 64 bits, sélectionnez **x64**.
   - Si vous exécutez Windows 32 bits, sélectionnez **x86**.

1. Sélectionnez **OK**, puis **Fermer**.
