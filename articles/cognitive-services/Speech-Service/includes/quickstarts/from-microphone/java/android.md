---
title: 'Démarrage rapide : Reconnaître la voix venant d’un microphone, Java (Android) – Service Speech'
titleSuffix: Azure Cognitive Services
description: Découvrir la reconnaissance vocale en Java sur Android à l’aide du SDK Speech
services: cognitive-services
author: fmegen
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 11/05/2019
ms.author: wolfma
ms.openlocfilehash: 3a3799fc1e931993c00ba497765f4cd3e60d3493
ms.sourcegitcommit: d95cab0514dd0956c13b9d64d98fdae2bc3569a0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91377562"
---
## <a name="prerequisites"></a>Prérequis

Avant de commencer :

> [!div class="checklist"]
> * [Créer une ressource Azure Speech](../../../../overview.md#try-the-speech-service-for-free)
> * [Configurer votre environnement de développement](../../../../quickstarts/setup-platform.md?tabs=android&pivots=programming-language-java)
> * Veiller à avoir accès à un microphone pour la capture audio

## <a name="create-a-user-interface"></a>Créer une interface utilisateur

Nous allons maintenant créer une interface utilisateur de base pour l’application. Modifiez la disposition de votre activité principale, `activity_main.xml`. Initialement, la disposition comprend une barre de titre avec le nom de votre application, et un TextView contenant le texte « Hello World! ».

* Sélectionnez l’élément TextView. Remplacez son attribut ID dans le coin supérieur droit par `hello`.

* Dans la palette, en haut à gauche de la fenêtre `activity_main.xml`, faites glisser un bouton dans l’espace vide au-dessus du texte.

* Dans les attributs du bouton à droite, entrez `onSpeechButtonClicked` comme valeur de l’attribut `onClick`. Nous allons écrire une méthode portant ce nom pour gérer l’événement de bouton. Remplacez son attribut ID dans le coin supérieur droit par `button`.

* Utilisez l’icône de baguette magique en haut du concepteur pour calculer les contraintes de disposition.

  ![Capture d’écran de l’icône de baguette magique](~/articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-10-infer-layout-constraints.png)

Le texte et la représentation graphique de votre interface utilisateur doivent maintenant ressembler à ce qui suit :

![Interface utilisateur](~/articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-11-gui.png)

[!code-xml[](~/samples-cognitive-services-speech-sdk/quickstart/java/android/from-microphone/app/src/main/res/layout/activity_main.xml)]

## <a name="add-sample-code"></a>Ajouter un exemple de code

1. Ouvrez le fichier source `MainActivity.java`. Remplacez tout le code de ce fichier par ce qui suit :

   [!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/android/from-microphone/app/src/main/java/com/microsoft/cognitiveservices/speech/samples/quickstart/MainActivity.java#code)]

   * La méthode `onCreate` inclut du code qui demande une autorisation d’accès au microphone et à Internet, et établit la liaison de la plateforme native. La configuration des liaisons à la plateforme native n’est à effectuer qu’une seule fois. Elle doit être réalisée au début de l’initialisation de l’application.

   * La méthode `onSpeechButtonClicked` est, comme indiqué précédemment, le gestionnaire de clic du bouton. Un appui sur le bouton déclenche la transcription de la reconnaissance vocale.

1. Dans le même fichier, remplacez la chaîne `YourSubscriptionKey` par votre clé d’abonnement.

1. De plus, recherchez la chaîne `YourServiceRegion` et remplacez-la par l’**identificateur Région** de la [région](https://aka.ms/speech/sdkregion) associée à votre abonnement.

## <a name="build-and-run-the-app"></a>Générer et exécuter l’application

1. Connectez votre appareil Android à votre PC de développement. Vérifiez que vous avez activé le [mode de développement et le débogage USB](https://developer.android.com/studio/debug/dev-options) sur l’appareil.

1. Pour générer l’application, sélectionnez Ctrl+F9 ou, dans la barre de menus, choisissez **Build (Générer)**  > **Make Project (Créer le projet)** .

1. Pour lancer l’application, sélectionnez Maj+F10, ou sélectionnez **Run (Exécuter)**  > **Run 'app' (Exécutez « app »)** .

1. Dans la fenêtre de la cible de déploiement qui apparaît, sélectionnez votre appareil Android.

   ![Capture d’écran de la fenêtre de sélection de la cible de déploiement](~/articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-12-deploy.png)

Appuyez sur le bouton dans l’application pour commencer une session de reconnaissance vocale. Les 15 secondes suivantes de paroles prononcées en anglais sont envoyées au service Speech et transcrites. Le résultat s’affiche dans l’application Android ainsi que dans la fenêtre logcat d’Android Studio.

![Capture d’écran de l’application Android](~/articles/cognitive-services/Speech-Service/media/sdk/qs-java-android-13-gui-on-device.png)

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [Speech recognition basics](../../speech-to-text-next-steps.md)]
