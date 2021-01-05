---
title: 'Démarrage rapide : Créer une application Xamarin Android'
description: Dans ce démarrage rapide, vous allez apprendre à générer une application Android avec Xamarin en utilisant Spatial Anchors.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: quickstart
ms.service: azure-spatial-anchors
ms.openlocfilehash: 1626dfa739df6b7cf971aa570f14d36ac6c532e7
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "96022613"
---
# <a name="quickstart-create-a-xamarin-android-app-with-azure-spatial-anchors"></a>Démarrage rapide : Créer une application Android Xamarin avec Azure Spatial Anchors

Ce démarrage rapide explique comment créer une application Android avec Xamarin en utilisant [Azure Spatial Anchors](../overview.md). Azure Spatial Anchors est un service de développement multiplateforme qui vous permet de créer des expériences de réalité mixte en utilisant des objets qui conservent leur emplacement sur les appareils. Quand vous aurez terminé, vous aurez une application Android capable d’enregistrer et de rappeler une ancre spatiale.

Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer un compte Spatial Anchors
> * Configurer l’identificateur et la clé du compte Spatial Anchors
> * Déployer et exécuter sur un appareil Android

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce guide de démarrage rapide, veillez à avoir :
- Un ordinateur Windows ou macOS :
  - Si vous utilisez Windows :
    - Une version à jour de <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2019 16.2+</a>.
    - <a href="https://git-scm.com/download/win" target="_blank">Git pour Windows</a>.
    - <a href="https://git-lfs.github.com/">Git LFS</a>.
  - Si vous utilisez macOS :
    - Une version à jour de <a href="/visualstudio/mac/installation?view=vsmac-2019" target="_blank">Visual Studio pour Mac 8.1+</a>.
    - <a href="https://git-scm.com/download/mac" target="_blank">Git pour macOS</a>.
    - <a href="https://git-lfs.github.com/">Git LFS</a>.
- La dernière version de Xamarin.Android installée et en cours d’exécution sur la plateforme de votre choix. Pour obtenir un guide d’installation de Xamarin.Android, reportez-vous aux guides d’[Installation de Xamarin.Android](/xamarin/android/get-started/installation/index).
- Un appareil Android <a href="https://developer.android.com/studio/debug/dev-options" target="_blank">prêt pour le développement</a> et <a href="https://developers.google.com/ar/discover/supported-devices" target="_blank">compatible ARCore</a>.
  - Des pilotes d’appareils supplémentaires peuvent être nécessaires pour que votre ordinateur puisse communiquer avec votre appareil Android. Vous pourrez trouver plus d’informations [ici](https://developer.android.com/studio/run/device.html).
- Votre application doit cibler ARCore **1.8**.

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="open-the-sample-project"></a>Ouvrir l’exemple de projet

[!INCLUDE [Clone Sample Repo](../../../includes/spatial-anchors-clone-sample-repository.md)]

Ouvrez `Xamarin/SampleXamarin.sln` dans Visual Studio.

## <a name="configure-account-identifier-and-key"></a>Configurer l’identificateur et la clé du compte

L’étape suivante configure l’application pour qu’elle utilise votre identificateur et votre clé de compte. Vous les avez copiés dans un éditeur de texte pendant la [configuration de la ressource Spatial Anchors](#create-a-spatial-anchors-resource).

Ouvrez `Xamarin/SampleXamarin.Common/AccountDetails.cs`.

Localisez le champ `SpatialAnchorsAccountKey` et remplacez `Set me` par la clé du compte.

Localisez le champ `SpatialAnchorsAccountId` et remplacez `Set me` par l’identificateur du compte.

Recherchez le champ `SpatialAnchorsAccountDomain` et remplacez `Set me` par le domaine du compte.

## <a name="deploy-the-app-to-your-android-device"></a>Déployer l’application sur votre appareil Android

Allumez l’appareil Android, connectez-vous, puis connectez l’appareil à l’ordinateur au moyen d’un câble USB.

Définissez le projet de démarrage sur **SampleXamarin.Android**, remplacez la **Configuration de la solution** par **Mise en production**, puis sélectionnez l’appareil sur lequel vous souhaitez effectuer le déploiement dans la liste déroulante de sélection d’appareils.

# <a name="windows"></a>[Windows](#tab/deploy-windows)

![Capture d’écran montrant le menu permettant de sélectionner le projet et l’appareil dans Windows.](./media/get-started-xamarin-android/visual-studio-windows-configuration.png)

Sélectionnez **Déboguer** > **Démarrer le débogage** pour déployer et démarrer votre application.

# <a name="macos"></a>[macOS](#tab/deploy-macos)

![Configuration de Visual Studio](./media/get-started-xamarin-android/visual-studio-macos-configuration.jpg)

Sélectionnez **Exécuter** > **Démarrer sans débogage** pour déployer et démarrer votre application.

---

Dans l’application, sélectionnez **De base** pour exécuter la démonstration et suivez les instructions pour placer et rappeler une ancre.

> ![Capture d’écran 1](./media/get-started-xamarin-android/screenshot-1.jpg)
> ![Capture d’écran 2](./media/get-started-xamarin-android/screenshot-2.jpg)
> ![Capture d’écran 3](./media/get-started-xamarin-android/screenshot-3.jpg)

[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

[!INCLUDE [Next steps](../../../includes/spatial-anchors-quickstarts-nextsteps.md)]

> [!div class="nextstepaction"]
> [Tutoriel : Partager Spatial Anchors sur des appareils](../tutorials/tutorial-share-anchors-across-devices.md)