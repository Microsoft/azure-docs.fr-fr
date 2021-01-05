---
title: 'Démarrage rapide : Créer une application Android'
description: Dans ce démarrage rapide, vous allez apprendre à générer une application Android à l’aide de Spatial Anchors.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: quickstart
ms.service: azure-spatial-anchors
ms.openlocfilehash: b7be4e257fc884c655fe380f69b08797a907fbbb
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "96022647"
---
# <a name="quickstart-create-an-android-app-with-azure-spatial-anchors"></a>Démarrage rapide : Créer une application Android avec Azure Spatial Anchors

Ce démarrage rapide explique comment créer une application Android à l’aide d’[Azure Spatial Anchors](../overview.md) en Java ou C++/NDK. Azure Spatial Anchors est un service de développement multiplateforme qui vous permet de créer des expériences de réalité mixte en utilisant des objets qui conservent leur emplacement sur les appareils. Quand vous aurez terminé, vous aurez une application Android ARCore capable d’enregistrer et de rappeler une ancre spatiale.

Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer un compte Spatial Anchors
> * Configurer l’identificateur et la clé du compte Spatial Anchors
> * Déployer et exécuter sur un appareil Android

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce guide de démarrage rapide, veillez à avoir :

- Un ordinateur Windows ou macOS avec <a href="https://developer.android.com/studio/" target="_blank">Android Studio 3.4+</a>.
  - Si vous exécutez Windows, vous avez aussi besoin de <a href="https://git-scm.com/download/win" target="_blank">Git pour Windows</a> et <a href="https://git-lfs.github.com/">Git LFS</a>.
  - Si vous exécutez macOS, installez Git via HomeBrew. Entrez la commande suivante sur une seule ligne du Terminal : `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`. Puis exécutez `brew install git` et `brew install git-lfs`.
  - Pour générer l’exemple NDK, vous devez aussi installer le NDK et SDK Tools CMake 3.6 ou version supérieure dans Android Studio.
- Un appareil Android <a href="https://developer.android.com/studio/debug/dev-options" target="_blank">prêt pour le développement</a> et <a href="https://developers.google.com/ar/discover/supported-devices" target="_blank">compatible ARCore</a>.
  - Des pilotes d’appareils supplémentaires peuvent être nécessaires pour que votre ordinateur puisse communiquer avec votre appareil Android. Cliquez [ici](https://developer.android.com/studio/run/device.html) pour obtenir des informations et instructions supplémentaires.
- Votre application doit cibler ARCore **1.11.0**.

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="open-the-sample-project"></a>Ouvrir l’exemple de projet

# <a name="java"></a>[Java](#tab/openproject-java)

[!INCLUDE [Clone Sample Repo](../../../includes/spatial-anchors-clone-sample-repository.md)]

# <a name="ndk"></a>[NDK](#tab/openproject-ndk)

[!INCLUDE [Clone Sample Repo](../../../includes/spatial-anchors-clone-sample-repository.md)]

Téléchargez `arcore_c_api.h` à partir d’[ici](https://raw.githubusercontent.com/google-ar/arcore-android-sdk/v1.11.0/libraries/include/arcore_c_api.h) et placez-le dans `Android\NDK\libraries\include`.

À partir du référentiel nouvellement cloné, initialisez des sous-modules en exécutant la commande suivante :

```console
git submodule update --init --recursive
```

---

Ouvrez Android Studio.

# <a name="java"></a>[Java](#tab/openproject-java)

Sélectionnez **Open an existing Android Studio project**, puis le projet situé à l’emplacement `Android/Java/`.

# <a name="ndk"></a>[NDK](#tab/openproject-ndk)

Sélectionnez **Open an existing Android Studio project**, puis le projet situé à l’emplacement `Android/NDK/`.

---

## <a name="configure-account-identifier-and-key"></a>Configurer l’identificateur et la clé du compte

L’étape suivante configure l’application pour qu’elle utilise votre identificateur et votre clé de compte. Vous les avez copiés dans un éditeur de texte pendant la [configuration de la ressource Spatial Anchors](#create-a-spatial-anchors-resource).

# <a name="java"></a>[Java](#tab/openproject-java)

Ouvrez `Android/Java/app/src/main/java/com/microsoft/sampleandroid/AzureSpatialAnchorsManager.java`.

Localisez le champ `SpatialAnchorsAccountKey` et remplacez `Set me` par la clé du compte.

Localisez le champ `SpatialAnchorsAccountId` et remplacez `Set me` par l’identificateur du compte.

Recherchez le champ `SpatialAnchorsAccountDomain` et remplacez `Set me` par le domaine du compte.

# <a name="ndk"></a>[NDK](#tab/openproject-ndk)

Ouvrez `Android/NDK/app/src/main/cpp/AzureSpatialAnchorsApplication.cpp`.

Localisez le champ `SpatialAnchorsAccountKey` et remplacez `Set me` par la clé du compte.

Localisez le champ `SpatialAnchorsAccountId` et remplacez `Set me` par l’identificateur du compte.

Recherchez le champ `SpatialAnchorsAccountDomain` et remplacez `Set me` par le domaine du compte.

---

## <a name="deploy-the-app-to-your-android-device"></a>Déployer l’application sur votre appareil Android

Allumez l’appareil Android, connectez-vous, puis connectez l’appareil au PC au moyen d’un câble USB.

Sélectionnez **Run** dans la barre d’outils Android Studio.

![Déployer et exécuter dans Android Studio](./media/get-started-android/android-studio-deploy-run.png)

Sélectionnez l’appareil Android dans la boîte de dialogue **Select Deployment Target** et sélectionnez **OK** pour exécuter l’application sur l’appareil Android.

Suivez les instructions dans l’application pour placer et rappeler une ancre.

Arrêtez l’application en sélectionnant **Stop** dans la barre d’outils Android Studio.

![Bouton Stop dans Android Studio](./media/get-started-android/android-studio-stop.png)

[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

[!INCLUDE [Next steps](../../../includes/spatial-anchors-quickstarts-nextsteps.md)]

> [!div class="nextstepaction"]
> [Tutoriel : Partager Spatial Anchors sur des appareils](../tutorials/tutorial-share-anchors-across-devices.md)
