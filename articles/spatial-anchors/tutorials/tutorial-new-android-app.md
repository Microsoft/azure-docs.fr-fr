---
title: 'Tutoriel : Créer une application Android'
description: Dans ce tutoriel, vous allez apprendre à créer une application Android à l’aide d’Azure Spatial Anchors.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: tutorial
ms.service: azure-spatial-anchors
ms.openlocfilehash: af0d01a20728d2332d4a8d71819f73baf68a65a4
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95998382"
---
# <a name="tutorial-step-by-step-instructions-to-create-a-new-android-app-using-azure-spatial-anchors"></a>Tutoriel : Instructions pas à pas pour créer une application Android à l’aide d’Azure Spatial Anchors

Ce tutoriel vous montre comment créer une application Android qui intègre la fonctionnalité ARCore avec Azure Spatial Anchors.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, veillez à disposer des éléments suivants :

- Un ordinateur Windows ou macOS avec <a href="https://developer.android.com/studio/" target="_blank">Android Studio 3.4+</a>.
- Un appareil Android <a href="https://developer.android.com/studio/debug/dev-options" target="_blank">prêt pour le développement</a> et <a href="https://developers.google.com/ar/discover/supported-devices" target="_blank">compatible ARCore</a>.

## <a name="getting-started"></a>Prise en main

Démarrez Android Studio. Dans la fenêtre d’accueil **Welcome to Android Studio**, cliquez sur **Start a new Android Studio project**. Ou bien, si un projet est déjà ouvert, sélectionnez **File**->**New Project**.

Dans la fenêtre **Create New Projets**, sous la section **Phone and Tablet** , choisissez **Empty Activity**, puis cliquez sur **Next**. Ensuite, sous **Minimum API level**, choisissez `API 26: Android 8.0 (Oreo)`, puis vérifiez que **Language** est défini sur `Java`. Vous pouvez éventuellement changer le nom et l’emplacement du projet ainsi que le nom du package. Laissez les autres options telles quelles. Cliquez sur **Terminer**. Le programme d’installation de composants **Component Installer** s’exécute. Une fois qu’il est terminé, cliquez sur **Finish**. Après quelques traitements, Android Studio ouvre l’IDE.

## <a name="trying-it-out"></a>Essai

Pour tester votre nouvelle application, connectez votre appareil prêt pour le développement à votre ordinateur de développement à l’aide d’un câble USB. Cliquez sur **Run**->**Run 'app'** . Dans la fenêtre **Select Deployment Target**, sélectionnez votre appareil, puis cliquez sur **OK**. Android Studio installe l’application sur votre appareil connecté et le démarre. « Hello World! » doit maintenant s’afficher dans l’application qui s’exécute sur votre appareil. Cliquez sur **Run**->**Stop 'app'** .

## <a name="integrating-_arcore_"></a>Intégration d’_ARCore_

<a href="https://developers.google.com/ar/discover/" target="_blank">_ARCore_</a> est la plateforme de Google dédiée à la création d’expériences de réalité augmentée, ce qui permet à votre appareil de suivre sa position au fur et à mesure de ses déplacements et de développer sa propre compréhension du monde réel.

Modifiez `app\manifests\AndroidManifest.xml` pour inclure les entrées suivantes au sein du nœud `<manifest>` racine. Cet extrait de code fait plusieurs choses :

- Il permet à votre application d’accéder à la caméra de votre appareil.
- Il garantit aussi que seuls les appareils qui prennent en charge ARCore peuvent voir votre application sur le Google Play Store.
- Il configure le Google Play Store de façon à télécharger et à installer ARCore, si ce n’est déjà fait, pendant l’installation de votre application.

```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-feature android:name="android.hardware.camera.ar" />

<application>
    ...
    <meta-data android:name="com.google.ar.core" android:value="required" />
    ...
</application>
```

Modifiez `Gradle Scripts\build.gradle (Module: app)` pour y inclure l’entrée ci-dessous. Ce code garantit que votre application cible ARCore version 1.8. Après cette modification, il se peut que vous obteniez une notification de Gradle qui vous invite à effectuer une synchronisation : cliquez sur **Sync now**.

```
dependencies {
    ...
    implementation 'com.google.ar:core:1.11.0'
    ...
}
```

## <a name="integrating-_sceneform_"></a>Intégration de _Sceneform_

[_Sceneform_](https://developers.google.com/sceneform/develop/) simplifie le rendu de scènes 3D réalistes dans les applications de réalité augmentée sans qu’il soit nécessaire d’apprendre OpenGL.

Modifiez `Gradle Scripts\build.gradle (Module: app)` pour y inclure les entrées ci-dessous. Ce code permet à votre application d’utiliser des constructions de langage à partir de Java 8, ce qui est une exigence de `Sceneform`. Il garantit aussi que votre application cible `Sceneform` version 1.8, dans la mesure où elle doit correspondre à la version d’ARCore qu’utilise votre application. Après cette modification, il se peut que vous obteniez une notification de Gradle qui vous invite à effectuer une synchronisation : cliquez sur **Sync now**.

```
android {
    ...

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    ...
    implementation 'com.google.ar.sceneform.ux:sceneform-ux:1.11.0'
    ...
}
```

Ouvrez votre fichier `app\res\layout\activity_main.xml` et remplacez l’élément `<TextView>` Hello World existant par l’ArFragment suivant. Ce code a pour effet d’afficher le flux de caméra sur votre écran, ce qui permet à ARCore de suivre la position de votre appareil au fur et à mesure de ses déplacements.

```xml
<fragment android:name="com.google.ar.sceneform.ux.ArFragment"
    android:id="@+id/ux_fragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

[Redéployez](#trying-it-out) votre application sur votre appareil pour la vérifier une nouvelle fois. Cette fois, vous êtes invité à fournir des autorisations pour la caméra. Après approbation, le rendu du flux de caméra doit s’afficher sur votre écran.

## <a name="place-an-object-in-the-real-world"></a>Placer un objet dans le monde réel

Vous allez maintenant créer et placer un objet à l’aide de votre application. Pour commencer, ajoutez les importations suivantes à `app\java\<PackageName>\MainActivity` :

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=23-33)]

Ajoutez ensuite les variables de membre suivantes dans votre classe `MainActivity` :

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=52-57)]

Ajoutez maintenant le code suivant à la méthode `onCreate()` de votre `app\java\<PackageName>\MainActivity`. Ce code raccorde un écouteur, appelé `handleTap()`, qui détecte les appuis de l’utilisateur sur l’écran de votre appareil. S’il appuie sur une surface du monde réel qui a déjà été reconnue par le suivi d’ARCore, l’écouteur s’exécute.

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=68-74,85&highlight=6-7)]

Enfin, ajoutez la méthode `handleTap()` suivante pour tout lier ensemble. Une sphère est alors créée et placée là où l’utilisateur a appuyé. Au départ, la sphère est noire, car à cet instant, `this.recommendedSessionProgress` est défini sur zéro. Cette valeur sera ajustée par la suite.

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=151-159,171-172,175-183,199-200)]

[Redéployez](#trying-it-out) votre application sur votre appareil pour la vérifier une nouvelle fois. Cette fois, vous pouvez déplacer votre appareil pour permettre à ARCore de commencer à reconnaître votre environnement. Appuyez ensuite sur l’écran pour créer et placer votre sphère noire sur la surface de votre choix.

## <a name="attach-a-local-azure-spatial-anchor"></a>Attacher une ancre spatiale Azure locale

Modifiez `Gradle Scripts\build.gradle (Module: app)` pour y inclure l’entrée ci-dessous. Ce code garantit que votre application cible Azure Spatial Anchors version 2.2.0. Ceci dit, le référencement de n’importe quelle version récente d’Azure Spatial Anchors doit fonctionner. Vous trouverez les notes de publication [ici](https://github.com/Azure/azure-spatial-anchors-samples/releases).

```
dependencies {
    ...
    implementation "com.microsoft.azure.spatialanchors:spatialanchors_jni:[2.2.0]"
    implementation "com.microsoft.azure.spatialanchors:spatialanchors_java:[2.2.0]"
    ...
}
```

Cliquez avec le bouton droit sur `app\java\<PackageName>`->**New**->**Java Class**. Définissez **Name** sur _MyFirstApp_, puis **Superclass** sur _android.app.Application_. Laissez les autres options telles quelles. Cliquez sur **OK**. Un fichier appelé `MyFirstApp.java` est alors créé. Ajoutez-y l’importation suivante :

```java
import com.microsoft.CloudServices;
```

Ajoutez ensuite le code suivant à l’intérieur de la nouvelle classe `MyFirstApp`, qui fait en sorte qu’Azure Spatial Anchors soit initialisé avec le contexte de votre application.

```java
    @Override
    public void onCreate() {
        super.onCreate();
        CloudServices.initialize(this);
    }
```

À présent, modifiez `app\manifests\AndroidManifest.xml` pour inclure l’entrée suivante au sein du nœud `<application>` racine. Ce code raccorde la classe Application que vous avez créée dans votre application.

```xml
    <application
        android:name=".MyFirstApp"
        ...
    </application>
```

De retour dans `app\java\<PackageName>\MainActivity`, ajoutez les importations suivantes :

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=33-40&highlight=2-8)]

Ajoutez ensuite les variables de membre suivantes dans votre classe `MainActivity` :

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=57-60&highlight=3-4)]

Ajoutez maintenant la méthode `initializeSession()` suivante au sein de la classe `mainActivity`. Une fois appelée, elle assurera la création et l’initialisation d’une session Azure Spatial Anchors pendant le démarrage de votre application.

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=89-97,147)]

À présent, raccordez la méthode `initializeSession()` dans la méthode `onCreate()`. De même, veillez à ce que les images issues de votre flux de caméra sont envoyées au kit SDK Azure Spatial Anchors à des fins de traitement.

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=68-85&highlight=9-17)]

Enfin, ajoutez le code suivant à votre méthode `handleTap()`. Ce code a pour effet d’attache une ancre spatiale Azure locale à la sphère noire que vous placez dans le monde réel.

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=151-159,171-183,199-200&highlight=12-13)]

[Redéployez](#trying-it-out) une nouvelle fois votre application. Déplacez votre appareil, appuyez sur l’écran, puis placez une sphère noire. Cette fois, votre code crée une ancre spatiale Azure locale et l’attache à votre sphère.

Avant de poursuivre, vous devez créer un compte Azure Spatial Anchors pour obtenir l’identificateur, la clé et le domaine du compte, si vous ne les avez pas. Pour vous en procurer, suivez les instructions dans la section ci-dessous.

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="upload-your-local-anchor-into-the-cloud"></a>Charger votre ancre locale dans le cloud

Une fois que vous disposez de l’identificateur, de la clé et du domaine du compte Azure Spatial Anchors, revenez dans `app\java\<PackageName>\MainActivity` et ajoutez-y les importations suivantes :

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=40-45&highlight=3-6)]

Ajoutez ensuite les variables de membre suivantes dans votre classe `MainActivity` :

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=60-65&highlight=3-6)]

À présent, ajoutez le code suivant à votre méthode `initializeSession()`. En premier lieu, ce code permet à votre application de surveiller la progression du kit SDK Azure Spatial Anchors à mesure qu’il collecte des images à partir de votre flux de caméra. Pendant ce temps, la couleur de votre sphère commence à passer de sa couleur noire initiale au gris. Ensuite, elle prend la couleur blanche une fois que le nombre d’images collectées est suffisant pour soumettre l’ancre au cloud. En second lieu, ce code fournit les informations d’identification nécessaires pour communiquer avec le back-end cloud. Voici où vous allez configurer votre application pour qu’elle utilise l’identificateur, la clé et le domaine de votre compte. Vous les avez copiés dans un éditeur de texte pendant la [configuration de la ressource Spatial Anchors](#create-a-spatial-anchors-resource).

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=89-120,142-148&highlight=11-37)]

Ensuite, ajoutez la méthode `uploadCloudAnchorAsync()` suivante à l’intérieur de votre classe `mainActivity`. Une fois appelée, cette méthode attend de manière asynchrone que le nombre d’images collectées à partir de votre appareil soit suffisant. Dès lors, elle fait passer la couleur de votre sphère au jaune et commence à charger l’ancre spatiale Azure locale dans le cloud. À l’issue du chargement, le code retourne un identificateur d’ancre.

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?name=uploadCloudAnchorAsync)]

Enfin, raccordez tout ensemble. Dans votre méthode `handleTap()`, ajoutez le code suivant. Il appelle la méthode `uploadCloudAnchorAsync()` aussitôt que la sphère est créée. Une fois la méthode retournée, le code ci-dessous apporte une dernière mise à jour à la sphère en lui donnant la couleur bleue.

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=151-159,171-200&highlight=24-37)]

[Redéployez](#trying-it-out) une nouvelle fois votre application. Déplacez votre appareil, appuyez sur l’écran, puis placez la sphère. Cette fois, la sphère change de couleur, passant du noir à blanc à mesure que les images de la caméra sont collectées. Une fois que le nombre d’images est suffisant, la sphère devient jaune et le chargement cloud débute. À l’issue du chargement, la sphère passe au bleu. Si vous le souhaitez, vous pouvez aussi utiliser la fenêtre `Logcat` dans Android Studio pour analyser les messages de journal envoyés par votre application. Par exemple, ces messages peuvent indiquer la progression de la session pendant les captures d’images ou l’identificateur d’ancre retourné par le cloud à l’issue du chargement.

## <a name="locate-your-cloud-spatial-anchor"></a>Localiser l’ancre spatiale cloud

Une fois que votre ancre est chargée dans le cloud, vous pouvez essayer de la localiser à nouveau. Pour commencer, ajoutez les importations suivantes à votre code.

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?range=45-48&highlight=3-4)]

Ensuite, ajoutez le code suivant à votre méthode `handleTap()`. Ce code a pour effet de :

- Supprimer la sphère bleue existante de l’écran.
- Réinitialiser la session Azure Spatial Anchors. Cette action garantit que l’ancre à localiser provient du cloud et qu’il ne s’agit pas de l’ancre locale que vous avez créée.
- Émettez une requête pour l’ancre que vous avez chargée dans le cloud.

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?name=handleTap&highlight=10-19)]

Maintenant, raccordez le code qui sera appelé dès que l’ancre recherchée sera localisée. À l’intérieur de votre méthode `initializeSession()`, ajoutez le code suivant. Cet extrait de compte crée et place une sphère verte une fois que l’ancre spatiale cloud est localisée. Il réactive également les appuis sur l’écran, ce qui vous permet de répéter une nouvelle fois le scénario dans son intégralité : création, chargement et relocalisation d’une nouvelle ancre.

[!code-java[MainActivity](../../../includes/spatial-anchors-new-android-app-finished.md?name=initializeSession&highlight=34-53)]

Et voilà ! [Redéployez](#trying-it-out) votre application une dernière fois pour essayer l’ensemble du scénario de bout en bout. Déplacez votre appareil, puis placez votre sphère noire. Continuez de déplacer votre appareil pour capturer les images de la caméra jusqu’à ce que la sphère devienne jaune. Votre ancre locale est alors chargée et la sphère devient bleue. Enfin, appuyez une nouvelle fois sur l’écran de façon à supprimer l’ancre locale, puis recherchez son équivalent cloud. Continuez de déplacer votre appareil jusqu’à ce que l’ancre spatiale cloud soit localisée. Une sphère verte doit alors apparaître à l’emplacement approprié, et vous pouvez effacer et répéter une nouvelle fois l’ensemble du scénario.

[!INCLUDE [Share Anchors Sample Prerequisites](../../../includes/spatial-anchors-new-android-app-finished.md)]
