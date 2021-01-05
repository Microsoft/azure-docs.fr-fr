---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 10/15/2020
ms.author: erhopf
ms.openlocfilehash: 3f1a24db459cabd65d9ce17b89105c3b9ab8abb7
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96188323"
---
1. Démarrez Visual Studio 2019.

1. Assurez-vous que la charge de travail **Développement multiplateforme .NET Core** est disponible. Choisissez **Outils** > **Obtenir des outils et fonctionnalités** dans la barre de menus de Visual Studio pour ouvrir le programme d’installation de Visual Studio. Si cette charge de travail est déjà activée, fermez la boîte de dialogue.

   ![Capture d’écran du programme d’installation de Visual Studio, avec l’onglet Charges de travail sélectionné](../articles/cognitive-services/Speech-Service/media/sdk/vs-enable-net-core-workload.png)

   Sinon, activez la case à cocher en regard de **Développement multiplateforme .NET Core** et sélectionnez **Modifier** en bas à droite dans la boîte de dialogue. L’installation de la nouvelle fonctionnalité prend quelques instants.

1. Créez une application console .NET Core avec Visual C#. Dans le volet gauche de la boîte de dialogue **Nouveau projet**, développez **Installés** > **Visual C#**  >  **.NET Core**. Ensuite, sélectionnez **Application console (.NET Core)** . Pour le nom du projet, entrez *helloworld*.

   ![Capture d’écran de la boîte de dialogue Nouveau projet](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-dotnetcore-windows-01-new-console-app.png "Créer une application console Visual C# (.NET Core)")

1. Installez et référencez le [package NuGet du Kit de développement logiciel (SDK) Speech](https://aka.ms/csspeech/nuget). Dans l’Explorateur de solutions, cliquez avec le bouton droit sur la solution, puis sélectionnez **Gérer les packages NuGet pour la solution**.

   ![Capture d’écran de l’Explorateur de solutions, avec Gérer les packages NuGet pour la solution mis en surbrillance](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-dotnetcore-windows-02-manage-nuget-packages.png "Gérer les packages NuGet pour la solution")

1. En haut à droite, dans le champ **Source du package**, sélectionnez **nuget.org**. Recherchez le package `Microsoft.CognitiveServices.Speech` et installez-le dans le projet **helloworld**.

   ![Capture d’écran montrant la boîte de dialogue « Gérer les packages de la solution ».](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-dotnetcore-windows-03-nuget-install-1.0.0.png "Installer le package NuGet")

1. Acceptez la licence affichée pour commencer l’installation du package NuGet.

   ![Capture d’écran de la boîte de dialogue Acceptation de la licence](../articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-dotnetcore-windows-04-nuget-license.png "Accepter la licence")

Une fois le package installé, un message de confirmation s’affiche dans la console du Gestionnaire de package.
