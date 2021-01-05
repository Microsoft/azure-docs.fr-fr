---
title: 'Démarrage rapide : Configuration de la plateforme du Kit de développement logiciel (SDK) Speech pour .NET Framework (Windows) - Service de reconnaissance vocale'
titleSuffix: Azure Cognitive Services
description: Utilisez ce guide pour configurer votre plateforme pour C# sous .NET Framework pour Windows avec le kit SDK du service Speech.
services: cognitive-services
author: markamos
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 10/15/2020
ms.author: erhopf
ms.custom: devx-track-dotnet
ms.openlocfilehash: f63c97fd23d56ff508de8d5953eefac6b15ae1aa
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96188296"
---
Ce guide explique comment installer le [Kit de développement logiciel (SDK) Speech](~/articles/cognitive-services/speech-service/speech-sdk.md) pour .NET Framework (Windows). Si vous voulez simplement le nom du package pour commencer seul, exécutez `Install-Package Microsoft.CognitiveServices.Speech` dans la console NuGet.

[!INCLUDE [License Notice](~/includes/cognitive-services-speech-service-license-notice.md)]

## <a name="prerequisites"></a>Prérequis

Ce démarrage rapide nécessite :

* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/)

## <a name="create-a-visual-studio-project-and-install-the-speech-sdk"></a>Créer un projet Visual Studio et installer le Kit de développement logiciel (SDK) Speech

Vous devrez installer le [package NuGet du SDK Speech](https://aka.ms/csspeech/nuget) afin de pouvoir le référencer dans votre code. Pour cela, vous devrez peut-être d’abord créer un projet **helloworld**. Si vous disposez déjà d’un projet avec la charge de travail **Développement .NET Desktop**, vous pouvez utiliser ce projet et passer à [Utiliser le gestionnaire de package NuGet pour installer le Kit de développement logiciel (SDK) Speech](#use-nuget-package-manager-to-install-the-speech-sdk).

### <a name="create-helloworld-project"></a>Créer un projet helloworld

1. Ouvrez Visual Studio 2019.

1. Dans la fenêtre Démarrer, sélectionnez **Créer un projet**. 

1. Dans la fenêtre **Créer un projet**, choisissez **Application console (.NET Framework)** , puis cliquez sur **Suivant**.

1. Dans la fenêtre **Configurer votre nouveau projet**, entrez *helloworld* dans **Nom du projet**, choisissez ou créez le chemin du répertoire dans **Emplacement**, puis sélectionnez **Créer**.

1. Dans la barre de menus de Visual Studio, sélectionnez **Outils** > **Obtenir les outils et fonctionnalités**, ce qui a pour effet d’ouvrir Visual Studio Installer et d’afficher la boîte de dialogue **Modification**.

1. Vérifiez si la charge de travail **Développement .NET Desktop** est disponible. Si cette charge de travail n’a pas été installée, cochez la case en regard de celle-ci, puis sélectionnez **Modifier** pour démarrer l’installation. Le téléchargement et l’installation peuvent prendre quelques minutes.

   Si la case à cocher à côté de **Développement .NET Desktop** est déjà sélectionnée, sélectionnez **Fermer** pour quitter la boîte de dialogue.

   ![Activer un développement de bureau .NET](~/articles/cognitive-services/speech-service/media/sdk/vs-enable-net-desktop-workload.png)

1. Fermez Visual Studio Installer.

### <a name="use-nuget-package-manager-to-install-the-speech-sdk"></a>Utiliser le gestionnaire de package NuGet pour installer le Kit de développement logiciel (SDK) Speech

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **helloworld**, puis sélectionnez **Gérer les packages NuGet** pour afficher le gestionnaire de package NuGet.

   ![Gestionnaire de package NuGet](~/articles/cognitive-services/speech-service/media/sdk/vs-nuget-package-manager.png)

1. Dans l’angle supérieur droit, recherchez la zone de liste déroulante **Source du package** et vérifiez que **`nuget.org`** est sélectionné.

1. Dans l’angle supérieur gauche, sélectionnez **Parcourir.**

1. Dans la zone de recherche, tapez *Microsoft.CognitiveServices.Speech* et appuyez sur la touche **Entrée**.

1. Dans les résultats de la recherche, sélectionnez le package **Microsoft.CognitiveServices.Speech**, puis sélectionnez **Installer** pour installer la dernière version stable.

   ![Installer le package NuGet Microsoft.CognitiveServices.Speech](~/articles/cognitive-services/speech-service/media/sdk/qs-csharp-dotnet-windows-03-nuget-install-1.0.0.png)

1. Acceptez tous les contrats et licences pour démarrer l’installation.

   Une fois le package installé, un message de confirmation s’affiche dans la fenêtre **Console du Gestionnaire de package**.

### <a name="choose-target-architecture"></a>Choisir l’architecture cible

Pour générer et exécuter l’application console, créez une configuration de plateforme correspondant à l’architecture de votre ordinateur.

1. Dans la barre de menus, sélectionnez **Générer** > **Gestionnaire de configurations**. La boîte de dialogue **Gestionnaire de configurations** s’affiche.

   ![Boîte de dialogue Gestionnaire de configurations](~/articles/cognitive-services/speech-service/media/sdk/vs-configuration-manager-dialog-box.png)

1. Dans la zone de liste déroulante **Plateforme de la solution active**, sélectionnez **Nouveau**. La boîte de dialogue **Nouvelle plateforme de solution** s’affiche.

1. Dans la zone de liste déroulante **Tapez ou sélectionnez la nouvelle plateforme** :
   - Si vous exécutez Windows 64 bits, sélectionnez **x64**.
   - Si vous exécutez Windows 32 bits, sélectionnez **x86**.

1. Sélectionnez **OK**, puis **Fermer**.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [windows](../quickstart-list.md)]
