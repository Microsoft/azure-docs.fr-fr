---
title: 'Démarrage rapide : Reconnaître la voix venant d’un microphone, C# (Unity) – Service Speech'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.custom: devx-track-csharp
ms.topic: include
ms.date: 04/02/2020
ms.author: erhopf
ms.openlocfilehash: 974f147492877f481616b43d67146a03814ada16
ms.sourcegitcommit: d95cab0514dd0956c13b9d64d98fdae2bc3569a0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91376777"
---
> [!NOTE]
> Le SDK Speech pour Unity prend en charge Windows Desktop (x86 et x64), ou la plateforme Windows universelle (x86, x64, ARM/ARM64), Android (x86, ARM32/64) et iOS (simulateur x64, ARM32 et ARM64).

## <a name="prerequisites"></a>Prérequis

Avant de commencer :

> [!div class="checklist"]
> * [Créer une ressource Azure Speech](../../../../overview.md#try-the-speech-service-for-free)
> * [Configurer votre environnement de développement et créer un projet vide](../../../../quickstarts/setup-platform.md?tabs=unity&pivots=programming-language-csharp)
> * Veiller à avoir accès à un microphone pour la capture audio

Si vous l’avez déjà fait, c’est parfait. Poursuivons.

## <a name="create-a-unity-project"></a>Créer un projet Unity

1. Ouvrez Unity. Si vous utilisez Unity pour la première fois, la fenêtre **Unity Hub** *<version number>* s’affiche. (Vous pouvez également ouvrir Unity Hub directement pour accéder à cette fenêtre.)

   [![Fenêtre Unity Hub](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-hub.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-hub.png#lightbox)
1. Sélectionnez **Nouveau**. La fenêtre **Create a new project with Unity** *<version number>* s’affiche.

   [![Créer un projet dans Unity Hub](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-create-a-new-project.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-create-a-new-project.png#lightbox)
1. Sous **Project Name**, entrez **csharp-unity**.
1. Sous **Templates**, sélectionnez **3D** si ce n’est déjà fait.
1. Sous **Location**, sélectionnez ou créez un dossier dans lequel le projet sera enregistré.
1. Sélectionnez **Create** (Créer).

Après quelques instants, la fenêtre de l’éditeur Unity s’affiche.



## <a name="add-ui"></a>Ajouter une interface utilisateur

À présent, nous allons ajouter une interface utilisateur minimale à notre scène. Cette interface utilisateur comprend un bouton pour déclencher la reconnaissance vocale et un champ de texte pour afficher le résultat. La [fenêtre **Hierarchy**](https://docs.unity3d.com/Manual/Hierarchy.html) présente un exemple de scène créée par Unity avec le nouveau projet.

1. En haut de la fenêtre **Hierarchy**, sélectionnez **Create** > **UI** > **Button**.

   Cette action crée trois objets de jeu que vous pouvez voir dans la fenêtre **Hierarchy** : un objet **Button**, un objet **Canvas** contenant le bouton et un objet **EventSystem**.

   [![Environnement de l’éditeur Unity](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-editor-window.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-editor-window.png#lightbox)

1. [Naviguez dans la vue **Scene**](https://docs.unity3d.com/Manual/SceneViewNavigation.html) pour avoir une bonne vue du canevas et du bouton de la [vue **Scene**](https://docs.unity3d.com/Manual/UsingTheSceneView.html).

1. Dans la [fenêtre **Inspector**](https://docs.unity3d.com/Manual/UsingTheInspector.html) (à droite par défaut), définissez les propriétés **Pos X** et **Pos Y** avec la valeur **0** pour centrer le bouton au milieu du canevas.

1. Dans la fenêtre **Hierarchy**, sélectionnez **Create** > **UI** > **Text** pour créer un objet **Text**.

1. Dans la fenêtre **Inspector**, définissez les propriétés **Pos X** et **Pos Y** avec les valeurs **0** et **120**, puis définissez les propriétés **Width** et **Height** avec les valeurs **240** et **120**. Ces valeurs garantissent que le champ de texte et le bouton ne se chevauchent pas.

Quand vous avez terminé, la vue **Scene** doit ressembler à cette capture d’écran :

[![Vue Scene dans l’éditeur Unity](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-02-ui-inline.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-02-ui-inline.png#lightbox)

## <a name="add-the-sample-code"></a>Ajouter l’exemple de code

Pour ajouter l’exemple de code de script au projet Unity, procédez comme suit :

1. Dans la [fenêtre Project](https://docs.unity3d.com/Manual/ProjectView.html), sélectionnez **Create** > **C# script** pour ajouter un nouveau script C#.

   [![Fenêtre Project dans l’éditeur Unity](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-project-window.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-project-window.png#lightbox)
1. Nommez le script `HelloWorld`.

1. Double-cliquez sur `HelloWorld` pour modifier le script nouvellement créé.

   > [!NOTE]
   > Pour configurer l’éditeur de code qui sera utilisé par Unity pour la modification, sélectionnez**Edit** > **Preferences**, puis accédez aux préférences **External Tools**. Pour plus d’informations, consultez le [manuel de l’utilisateur Unity](https://docs.unity3d.com/Manual/Preferences.html).

1. Remplacez le script existant par le code suivant :

   [!code-csharp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/unity/from-microphone/Assets/Scripts/HelloWorld.cs#code)]

1. Recherchez et remplacez la chaîne `YourSubscriptionKey` par votre clé d’abonnement au service Speech.

1. Recherchez la chaîne `YourServiceRegion` et remplacez-la par l’**identificateur Région** de la [région](https://aka.ms/speech/sdkregion) associée à votre abonnement.

1. Enregistrez les changements apportés au script.

À présent, revenez dans l’éditeur Unity et ajoutez le script en tant que composant à l’un de vos objets de jeu :

1. Dans la fenêtre **Hierarchy**, sélectionnez l’objet **Canvas**.

1. Dans la fenêtre **Inspector**, sélectionnez le bouton **Add Component**.

   [![Fenêtre Inspector dans l’éditeur Unity](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-inspector-window.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-inspector-window.png#lightbox)

1. Dans la liste déroulante, recherchez le script `HelloWorld` que nous avons créé précédemment, puis ajoutez-le. Une section **Hello World (Script)** s’affiche dans la fenêtre **Inspector**. Elle présente deux propriétés non initialisées : **Output Text** et **Start Reco Button**. Ces propriétés de composant Unity correspondent aux propriétés publiques de la classe `HelloWorld`.

1. Cliquez sur le sélecteur d’objet de la propriété **Start Reco Button** (petite icône en forme de cercle à droite de la propriété), puis choisissez l’objet **Button** que vous avez créé précédemment.

1. Cliquez sur le sélecteur d’objet de la propriété **Output Text**, puis choisissez l’objet **Text** que vous avez créé précédemment.

   > [!NOTE]
   > Le bouton comporte également un objet Text imbriqué. Veillez à ne pas le choisir accidentellement pour la sortie de texte (ou renommez l’un des objets de texte à l’aide du champ **Name** dans la fenêtre **Inspector** pour éviter toute confusion).

## <a name="run-the-application-in-the-unity-editor"></a>Exécuter l’application dans l’éditeur Unity

Vous êtes maintenant prêt à exécuter l’application dans l’éditeur Unity.

1. Dans la barre d’outils de l’éditeur Unity (sous la barre de menus), sélectionnez le bouton **Play** (triangle pointant vers la droite) .

1. Accédez à la [vue **Game**](https://docs.unity3d.com/Manual/GameView.html) et attendez que l’objet **Text** affiche **Click button to recognize speech**. (Il affiche **New Text** si l’application n’a pas démarré ou n’est pas prête à répondre.)

1. Cliquez sur le bouton et prononcez une phrase en anglais dans le microphone de votre ordinateur. Vos paroles sont transmises au service Speech et transcrites en texte, qui apparaît dans la vue **Game**.

   [![Vue Game dans l’éditeur Unity](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-03-output-inline.png)](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-unity-03-output-inline.png#lightbox)

1. Consultez la [fenêtre **Console**](https://docs.unity3d.com/Manual/Console.html) pour voir les messages de débogage. Si la fenêtre **Console** ne s’affiche pas, accédez à la barre de menus et sélectionnez **Window** > **General** > **Console** pour l’afficher.

1. Une fois la reconnaissance vocale terminée, cliquez sur le bouton **Play** dans la barre d’outils de l’éditeur Unity pour arrêter l’application.

## <a name="additional-options-to-run-this-application"></a>Options supplémentaires pour exécuter cette application

Vous pouvez également déployer cette application en tant qu’application Android, Windows autonome ou UWP.
Pour plus d’informations, consultez le [référentiel d’exemples](https://aka.ms/csspeech/samples). Le dossier `quickstart/csharp-unity` décrit la configuration de ces cibles supplémentaires.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [Speech recognition basics](../../speech-to-text-next-steps.md)]

