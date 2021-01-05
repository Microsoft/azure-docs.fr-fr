---
author: msftradford
ms.service: spatial-anchors
ms.topic: include
ms.date: 11/20/2020
ms.author: parkerra
ms.openlocfilehash: 9554e04c82299016076f09f85f604af10a94ab4a
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185410"
---
## <a name="android"></a>[Android](#tab/Android)

L’exemple Android Java prend en charge le partage entre appareils.

Dans Android Studio, ouvrez le fichier *SharedActivity.java* à partir du dossier d’exemples. 

Entrez l’URL que vous avez copiée à l’étape précédente (à partir du déploiement Azure de votre application web ASP.NET) comme valeur de `SharingAnchorsServiceUrl` dans le fichier *SharedActivity.java*. 

Remplacez `index.html` dans l’URL par `api/anchors`. Ce que vous obtenez doit ressembler à ceci : `https://<app_name>.azurewebsites.net/api/anchors`.

[!INCLUDE [Run shared sample](spatial-anchors-deploy-sample.md)]

[!INCLUDE [Run shared sample](spatial-anchors-run-sample.md)]

## <a name="ios"></a>[iOS](#tab/iOS)

L’exemple iOS Objective-C prend en charge le partage entre appareils.

Ouvrez le fichier *SharedDemoViewController.m* dans le dossier d’exemples. 

Entrez l’URL que vous avez obtenue à l’étape précédente (à partir du déploiement Azure de votre application web ASP.NET) comme valeur de `SharingAnchorsServiceUrl` dans le fichier *SharedDemoViewController.m*. 

Remplacez `index.html` dans l’URL par `api/anchors`. Ce que vous obtenez doit ressembler à ceci : `https://<app_name>.azurewebsites.net/api/anchors`.

Déployer l’application sur votre appareil 

Une fois l’application démarrée, sélectionnez l’option **Tap to start Shared Demo** (Appuyez pour démarrer la démonstration partagée), puis suivez les instructions dans l’application. Vous pouvez sélectionner **Appuyez pour localiser l’ancre par son numéro** ou **Appuyez pour créer l’ancre et l’enregistrer dans le service**.

[!INCLUDE [Run shared sample](spatial-anchors-run-sample.md)]

## <a name="xamarin"></a>[Xamarin](#tab/Xamarin)

Les exemples Xamarin Android et iOS prennent en charge le partage entre appareils.

Ouvrez le fichier *AccountDetails.cs* dans le dossier d’exemples. 

Entrez l’URL que vous avez obtenue à l’étape précédente (à partir du déploiement Azure de votre application web ASP.NET) comme valeur de `AnchorSharingServiceUrl` dans le fichier *AccountDetails.cs*. 

Remplacez `index.html` dans l’URL par `api/anchors`. Ce que vous obtenez doit ressembler à ceci : `https://<app_name>.azurewebsites.net/api/anchors`.

[!INCLUDE [Run shared sample](spatial-anchors-deploy-sample.md)]

[!INCLUDE [Run shared sample](spatial-anchors-run-sample.md)]

## <a name="unity"></a>[Unity](#tab/Unity)

[!INCLUDE [Open Unity Project](spatial-anchors-open-unity-project.md)]

### <a name="set-up-an-android-device"></a>Configurer un appareil Android

[!INCLUDE [Android Unity Build Settings](spatial-anchors-unity-android-build-settings.md)]

### <a name="set-up-an-ios-device"></a>Configurer un appareil iOS

[!INCLUDE [iOS Unity Build Settings](spatial-anchors-unity-ios-build-settings.md)]

[!INCLUDE [Configure Unity Scene](spatial-anchors-unity-configure-scene.md)]

Dans le volet **Project** (Projet), accédez à `Assets\AzureSpatialAnchors.Examples\Resources`. 

Sélectionnez **SpatialAnchorSamplesConfig**. Ensuite, dans le volet **Inspector** (Inspecteur), entrez l’URL `Sharing Anchors Service` (à partir du déploiement Azure de votre application web ASP.NET) comme valeur pour `Base Sharing Url`. Remplacez `index.html` par `api/anchors`. Ce que vous obtenez doit ressembler à ceci : `https://<app_name>.azurewebsites.net/api/anchors`.

Enregistrez la scène en sélectionnant **File** > **Save**.

## <a name="deploy-to-your-device"></a>Déployer sur votre appareil

### <a name="deploy-to-an-android-device"></a>Déployer sur un appareil Android

Connectez-vous à votre appareil Android et reliez-le à votre ordinateur à l’aide d’un câble USB.

Ouvrez **Build Settings** (Paramètres de build) en sélectionnant **File** (Fichier) > **Build Settings** (Paramètres de build).

Sous **Scenes In Build** (Scènes dans la build), vérifiez que toutes les scènes présentent une coche en regard.

Vérifiez que la case **Export Project** (Exporter le projet) n’est pas cochée. Sélectionnez **Build And Run** (Générer et exécuter). Vous êtes invité à enregistrer votre fichier *.apk*. Vous pouvez choisir n’importe quel nom pour celui-ci.

[!INCLUDE [Run shared sample](spatial-anchors-run-sample.md)]

### <a name="deploy-to-an-ios-device"></a>Déployer sur un appareil iOS

Ouvrez **Build Settings** (Paramètres de build) en sélectionnant **File** (Fichier) > **Build Settings** (Paramètres de build).

Sous **Scenes In Build** (Scènes dans la build), vérifiez que toutes les scènes présentent une coche en regard.

[!INCLUDE [Configure Xcode](spatial-anchors-unity-ios-xcode.md)]

[!INCLUDE [Run shared sample](spatial-anchors-run-sample.md)]

Dans Xcode, arrêtez l’application en sélectionnant **Stop**.
