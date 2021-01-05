---
title: 'Démarrage rapide : Créer une application Xamarin iOS'
description: Dans ce démarrage rapide, vous allez apprendre à générer une application iOS avec Xamarin en utilisant Spatial Anchors.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: quickstart
ms.service: azure-spatial-anchors
ms.openlocfilehash: b62ca4b02a1bd28d150f233adf1dfb621995e1c4
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "96009736"
---
# <a name="quickstart-create-a-xamarin-ios-app-with-azure-spatial-anchors"></a>Démarrage rapide : Créer une application Xamarin iOS avec Azure Spatial Anchors

Ce démarrage rapide explique comment créer une application iOS avec Xamarin en utilisant [Azure Spatial Anchors](../overview.md). Azure Spatial Anchors est un service de développement multiplateforme qui vous permet de créer des expériences de réalité mixte en utilisant des objets qui conservent leur emplacement sur les appareils. Une fois terminé, vous disposez d’une application iOS capable d’enregistrer et de rappeler une ancre spatiale.

Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer un compte Spatial Anchors
> * Configurer l’identificateur et la clé du compte Spatial Anchors
> * Déployer et exécuter sur un appareil iOS

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce guide de démarrage rapide, veillez à avoir :
- Un Mac exécutant macOS High Sierra (10.13) ou une version ultérieure avec :
  - La dernière version du kit de développement logiciel (SDK) Xcode et iOS installée à partir de l’[App Store](https://itunes.apple.com/us/app/xcode/id497799835?mt=12).
  - Une version à jour de <a href="/visualstudio/mac/installation?view=vsmac-2019" target="_blank">Visual Studio pour Mac 8.1+</a>.
  - <a href="https://git-scm.com/download/mac" target="_blank">Git pour macOS</a>.
  - <a href="https://git-lfs.github.com/">Git LFS</a>.

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

## <a name="deploy-the-app-to-your-ios-device"></a>Déployer l’application sur votre appareil iOS

Allumez l’appareil iOS, connectez-vous, puis connectez l’appareil à l’ordinateur au moyen d’un câble USB.

Définissez le projet de démarrage sur **SampleXamarin.iOS**, remplacez la **Configuration de la solution** par **Mise en production**, puis sélectionnez l’appareil sur lequel vous souhaitez effectuer le déploiement dans la liste déroulante de sélection d’appareils.

![Configuration de Visual Studio](./media/get-started-xamarin-iOS/visual-studio-macos-configuration.jpg)

Sélectionnez **Exécuter** > **Démarrer sans débogage** pour déployer et démarrer votre application.

Dans l’application, sélectionnez **De base** pour exécuter la démonstration et suivez les instructions pour placer et rappeler une ancre.

> ![Capture d’écran 1](./media/get-started-xamarin-ios/screenshot-1.jpg)
> ![Capture d’écran 2](./media/get-started-xamarin-ios/screenshot-2.jpg)
> ![Capture d’écran 3](./media/get-started-xamarin-ios/screenshot-3.jpg)

[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

[!INCLUDE [Next steps](../../../includes/spatial-anchors-quickstarts-nextsteps.md)]

> [!div class="nextstepaction"]
> [Tutoriel : Partager Spatial Anchors sur des appareils](../tutorials/tutorial-share-anchors-across-devices.md)