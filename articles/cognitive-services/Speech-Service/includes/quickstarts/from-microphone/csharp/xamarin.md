---
title: 'Démarrage rapide : Reconnaître la voix venant d’un microphone, C# (Xamarin) – Service Speech'
titleSuffix: Azure Cognitive Services
description: Dans cet article, vous créez une application C# Xamarin multiplateforme pour la plateforme Windows universelle (UWP), Android et iOS en utilisant le SDK Speech de Cognitive Services. Vous transcrivez la reconnaissance vocale en temps réel à partir du microphone de votre appareil ou simulateur. L’application est créée avec le package NuGet du SDK Speech et Microsoft Visual Studio 2019.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 04/02/2020
ms.author: erhopf
ms.custom: devx-track-csharp
ms.openlocfilehash: f89bdbfd144fb327c1daae020a1e371c00045859
ms.sourcegitcommit: d95cab0514dd0956c13b9d64d98fdae2bc3569a0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91376527"
---
## <a name="prerequisites"></a>Prérequis

Avant de commencer :

> [!div class="checklist"]
> * [Créer une ressource Azure Speech](../../../../overview.md#try-the-speech-service-for-free)
> * [Configurer votre environnement de développement et créer un projet vide](../../../../quickstarts/setup-platform.md?tabs=xamarin&pivots=programming-language-csharp)
> * Veiller à avoir accès à un microphone pour la capture audio

Si vous l’avez déjà fait, c’est parfait. Poursuivons.

## <a name="add-sample-code-for-the-common-helloworld-project"></a>Ajouter un exemple de code pour le projet helloworld commun

Le projet helloworld commun contient des implémentations indépendantes des plateformes pour votre application multiplateforme. Ajoutez maintenant le code XAML qui définit l’interface utilisateur de l’application, puis ajoutez le code C# derrière l’implémentation.

1. Dans l’**Explorateur de solutions**, sous le projet helloworld commun, ouvrez `MainPage.xaml`.

1. Dans la vue XAML du concepteur, insérez l’extrait de code XAML suivant dans la balise **Grid** entre `<StackLayout>` et `</StackLayout>` :

   [!code-xml[UI elements](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld/MainPage.xaml)]

1. Dans l’**Explorateur de solutions**, ouvrez le fichier source code-behind `MainPage.xaml.cs`. Il se trouve sous `MainPage.xaml`.

1. Remplacez tout le code de ce fichier par l’extrait de code suivant :

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld/MainPage.xaml.cs)]

1. Dans le gestionnaire `OnRecognitionButtonClicked` du fichier source, recherchez la chaîne `YourSubscriptionKey`, puis remplacez-la par la clé de votre abonnement.


1. Dans le gestionnaire `OnRecognitionButtonClicked`, recherchez la chaîne `YourServiceregion`, puis remplacez-la par l’**identificateur Région** de la [région](https://aka.ms/speech/sdkregion) associée à votre abonnement. 

1. Ensuite, vous devez créer un [service Xamarin](https://docs.microsoft.com/xamarin/android/app-fundamentals/services/creating-a-service/), qui est utilisé pour interroger les autorisations du microphone à partir de différents projets de plateforme, comme UWP, Android et iOS. Pour cela, ajoutez un nouveau dossier nommé *Services* sous le projet helloworld, puis créez un fichier source C# sous celui-ci. Vous pouvez cliquer avec le bouton droit sur le dossier *Services*, puis sélectionner **Ajouter** > **Nouvel élément** > **Fichier de code**. Renommez le fichier `IMicrophoneService.cs` et placez tout le code de l’extrait de code suivant dans ce fichier :

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld/Services/IMicrophoneService.cs)]

#### <a name="android"></a>[Android](#tab/x-android)
## <a name="add-sample-code-for-the-helloworldandroid-project"></a>Ajouter un exemple de code pour le projet `helloworld.Android`

Ajoutez maintenant le code C# qui définit la partie de l’application spécifique à Android.

1. Dans l’**Explorateur de solutions**, sous le projet helloworld.Android, ouvrez `MainActivity.cs`.

1. Remplacez tout le code de ce fichier par l’extrait de code suivant :

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld.Android/MainActivity.cs)]

1. Ensuite, ajoutez l’implémentation spécifique à Android pour `MicrophoneService` en créant un dossier *Services* sous le projet helloworld.Android. Après cela, créez un fichier source C# sous celui-ci. Renommez le fichier `MicrophoneService.cs`. Copiez et collez l’extrait de code suivant dans ce fichier :

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld.Android/Services/MicrophoneService.cs)]

1. Ensuite, ouvrez `AndroidManifest.xml` sous le dossier *Properties*. Ajoutez le paramètre uses-permission suivant pour le microphone entre `<manifest>` et `</manifest>` :

   ```xml
   <uses-permission android:name="android.permission.RECORD_AUDIO" />
   ```
   
#### <a name="ios"></a>[iOS](#tab/ios)
## <a name="add-sample-code-for-the-helloworldios-project"></a>Ajouter un exemple de code pour le projet `helloworld.iOS`

Ajoutez maintenant le code C# qui définit la partie de l’application spécifique à iOS. Créez également des configurations spécifiques aux appareils Apple dans le projet helloworld.iOS.

1. Dans l’**Explorateur de solutions**, sous le projet helloworld.iOS, ouvrez `AppDelegate.cs`.

1. Remplacez tout le code de ce fichier par l’extrait de code suivant :

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld.iOS/AppDelegate.cs)]

1. Ensuite, ajoutez l’implémentation spécifique à iOS pour `MicrophoneService` en créant un dossier *Services* sous le projet helloworld.iOS. Après cela, créez un fichier source C# sous celui-ci. Renommez le fichier `MicrophoneService.cs`. Copiez et collez l’extrait de code suivant dans ce fichier :

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld.iOS/Services/MicrophoneService.cs)]

1. Ouvrez `Info.plist` sous le projet helloworld.iOS dans l’éditeur de texte. Ajoutez la paire de valeurs de clé suivante sous la section dict :

   <key>NSMicrophoneUsageDescription</key>
   <string>Cet exemple d’application nécessite un accès au microphone</string>

   > [!NOTE]
   > Si vous générez une application pour un appareil iPhone, vérifiez que `Bundle Identifier` correspond à l’ID d’application du profil de provisionnement de votre appareil. Sinon, la build échoue. Avec iPhoneSimulator, vous pouvez le laisser tel quel.

1. Si vous effectuez une génération sur un PC Windows, établissez une connexion à un appareil Mac pour la génération via **Outils** > **iOS** > **Coupler au Mac**. Suivez l’Assistant d’instructions fourni par Visual Studio pour activer la connexion à l’appareil Mac.

#### <a name="uwp"></a>[UWP](#tab/helloworlduwp)
## <a name="add-sample-code-for-the-helloworlduwp-project"></a>Ajouter un exemple de code pour le projet `helloworld.UWP`

## <a name="add-sample-code-for-the-helloworlduwp-project"></a>Ajouter un exemple de code pour le projet helloworld.UWP

Ajoutez maintenant le code C# qui définit la partie de l’application spécifique à UWP.

1. Dans l’**Explorateur de solutions**, sous le projet helloworld.UWP, ouvrez `MainPage.xaml.cs`.

1. Remplacez tout le code de ce fichier par l’extrait de code suivant :

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld.UWP/MainPage.xaml.cs)]

1. Ensuite, ajoutez l’implémentation spécifique à UWP pour `MicrophoneService` en créant un dossier *Services* sous le projet helloworld.UWP. Après cela, créez un fichier source C# sous celui-ci. Renommez le fichier `MicrophoneService.cs`. Copiez et collez l’extrait de code suivant dans ce fichier :

   [!code-csharp[Quickstart code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/xamarin/helloworld/helloworld.UWP/Services/MicrophoneService.cs)]

1. Ensuite, double-cliquez sur le fichier `Package.appxmanifest` sous le projet helloworld.UWP dans Visual Studio. Sous **Capacités**, vérifiez que **Microphone** est sélectionné, puis enregistrez le fichier.

1. Ensuite, double-cliquez sur le fichier `Package.appxmanifest` sous le projet `helloworld.UWP` dans Visual Studio, puis sous **Fonctionnalités**, assurez-vous que **Microphone** est coché et enregistrez le fichier.
   > Remarque : Si vous voyez l’avertissement suivant : Le fichier de certificat n’existe pas : helloworld.UWP_TemporaryKey.pfx, consultez l’exemple de [reconnaissance vocale](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-csharp&tabs=uwp) pour plus d’informations.

1. Dans la barre de menus, sélectionnez **Fichier** > **Enregistrer tout** pour enregistrer vos modifications.

## <a name="build-and-run-the-uwp-application"></a>Générer et exécuter l’application UWP

1. Définissez helloworld.UWP comme projet de démarrage. Cliquez avec le bouton droit sur le projet helloworld.UWP, puis sélectionnez **Générer** pour générer l’application.

1. Sélectionnez **Déboguer** > **Démarrer le débogage** (ou sélectionnez **F5**) pour démarrer l’application. La fenêtre **helloworld** s’affiche.

   ![Exemple d’application de reconnaissance vocale UWP en C# - Démarrage rapide](../../../../media/sdk/qs-csharp-xamarin-helloworld-uwp-window.png)

1. Sélectionnez **Activer le microphone**. Quand la demande d’autorisation d’accès apparaît, sélectionnez **Oui**.

   ![Demande d’autorisation d’accès au microphone](../../../../media/sdk/qs-csharp-xamarin-uwp-access-prompt.png)

1. Sélectionnez **Démarrer la reconnaissance vocale** et prononcez une phrase ou quelques mots en anglais dans le micro de votre appareil. Votre production orale est transmise au service Speech, et transcrite en texte qui apparaît dans la fenêtre.

   ![Interface utilisateur de la reconnaissance vocale](../../../../media/sdk/qs-csharp-xamarin-uwp-ui-result.png)
* * *

## <a name="build-and-run-the-android-and-ios-applications"></a>Générer et exécuter les applications Android et iOS

La génération et l’exécution des applications Android et iOS dans l’appareil ou le simulateur se font de façon similaire à UWP. Vérifiez que tous les SDK sont installés correctement, comme indiqué dans la section « Prérequis » de cet article.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [Speech recognition basics](../../speech-to-text-next-steps.md)]
