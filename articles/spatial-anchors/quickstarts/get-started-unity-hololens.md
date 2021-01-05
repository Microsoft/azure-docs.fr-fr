---
title: 'Démarrage rapide : Créer une application HoloLens avec Unity'
description: Dans ce démarrage rapide, vous allez apprendre à générer une application HoloLens avec Unity en utilisant Spatial Anchors.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: quickstart
ms.service: azure-spatial-anchors
ms.openlocfilehash: 11d8e8d918b408881f211605ce6c713615f0aa93
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "96022630"
---
# <a name="quickstart-create-a-unity-hololens-app-that-uses-azure-spatial-anchors"></a>Démarrage rapide : Créer une application HoloLens Unity qui utilise Azure Spatial Anchors

Ce guide de démarrage rapide explique comment créer une application HoloLens Unity qui utilise [Azure Spatial Anchors](../overview.md). Spatial Anchors est un service de développement multiplateforme qui vous permet de créer des expériences de réalité mixte avec des objets qui conservent leur emplacement sur les appareils. Quand vous aurez terminé, vous aurez une application HoloLens générée avec Unity capable d’enregistrer et de rappeler une ancre spatiale.

Vous découvrirez comment effectuer les actions suivantes :

- Créer un compte Spatial Anchors.
- Préparer les paramètres de build Unity.
- Configurer l’identificateur et la clé du compte Spatial Anchors.
- Exporter le projet HoloLens Visual Studio.
- Déployer l’application et l’exécuter sur un appareil HoloLens.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce guide de démarrage rapide :

- Vous avez besoin d’un ordinateur Windows sur lequel <a href="https://unity3d.com/get-unity/download" target="_blank">Unity 2019.4 (LTS)</a> et <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2019</a> ou ultérieur sont installés. Votre installation de Visual Studio doit comprendre la charge de travail **Développement pour la plateforme Windows universelle** et le **Kit de développement logiciel (SDK) Windows 10 SDK (10.0.18362.0 ou plus récent)** . Vous devez aussi installer <a href="https://git-scm.com/download/win" target="_blank">Git pour Windows</a> et <a href="https://git-lfs.github.com/">Git LFS</a>.
- Vous avez besoin d’un appareil HoloLens avec le [mode développeur](/windows/mixed-reality/using-visual-studio) activé. La [mise à jour Windows 10 de mai 2020](/windows/mixed-reality/whats-new/release-notes-may-2020) doit être installée sur l’appareil. Pour mettre à jour votre appareil HoloLens avec la dernière version, ouvrez l’application **Paramètres**, accédez à **Mise à jour et sécurité**, puis sélectionnez **Rechercher les mises à jour**.
- Dans votre application, vous devez activer la fonctionnalité **SpatialPerception**. Ce paramètre se trouve dans **Build Settings (Paramètres de build)**  > **Player Settings (Paramètres du lecteur)**  > **Publishing Settings (Paramètres de publication)**  > **Capabilities (Fonctionnalités)** .
- Dans votre application, vous devez activer **Virtual Reality Supported** (Réalité virtuelle prise en charge) avec le **SDK Windows Mixed Reality**. Ce paramètre se trouve dans **Build Settings (Paramètres de build)**  > **Player Settings (Paramètres du lecteur)**  > **XR Settings (Paramètres XR)** .

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="download-and-open-the-unity-sample-project"></a>Télécharger et ouvrir l’exemple de projet Unity

[!INCLUDE [Clone Sample Repo](../../../includes/spatial-anchors-clone-sample-repository.md)]

[!INCLUDE [Open Unity Project](../../../includes/spatial-anchors-open-unity-project.md)]

Ouvrez **Build Settings** (Paramètres de build) en sélectionnant **File** (Fichier) > **Build Settings** (Paramètres de build).

Dans la section **Platform** (Plateforme), sélectionnez **Universal Windows Platform** (Plateforme Windows universelle). Remplacez **Target Device** (Appareil cible) par **HoloLens**.

Sélectionnez **Switch Platform** (Changer de plateforme) pour passer à la plateforme **Universal Windows Platform** (Plateforme Windows universelle). Unity peut vous demander d’installer des composants de prise en charge UWP s’ils sont manquants.

![Fenêtre des paramètres de build dans Unity](./media/get-started-unity-hololens/unity-build-settings.png)

Fermez la fenêtre **Build Settings**.

[!INCLUDE [Configure Unity Scene](../../../includes/spatial-anchors-unity-configure-scene.md)]

## <a name="export-the-hololens-visual-studio-project"></a>Exporter le projet HoloLens Visual Studio

[!INCLUDE [Export Unity Project](../../../includes/spatial-anchors-unity-export-project-snip.md)]

Sélectionnez **Build**. Dans la boîte de dialogue, sélectionnez un dossier où exporter le projet HoloLens Visual Studio.

À l’issue de l’exportation, un dossier avec le projet HoloLens exporté s’affiche.

## <a name="deploy-the-hololens-application"></a>Déployer l’application HoloLens

Dans le dossier, double-cliquez sur **HelloAR U3D.sln** pour ouvrir le projet dans Visual Studio.

Remplacez **Configuration de la solution** par **Mise en production**, **Plateforme de solution** par **x86**, puis sélectionnez **Appareil** dans les options de cible de déploiement.

Si vous utilisez HoloLens 2, utilisez **ARM64** comme **plateforme de solution** au lieu de **x86**.

   ![Configuration de Visual Studio](./media/get-started-unity-hololens/visual-studio-configuration.png)

Allumez l’appareil HoloLens, connectez-vous, puis connectez l’appareil au PC au moyen d’un câble USB.

Sélectionnez **Déboguer** > **Démarrer le débogage** pour déployer votre application et commencer le débogage.

Dans l’application, sélectionnez **BasicDemo** à l’aide des flèches, puis appuyez sur le bouton **Go!** pour exécuter la démo. Suivez les instructions dans l’application pour placer et rappeler une ancre.

![Capture d’écran 1](./media/get-started-unity-hololens/screenshot-1.jpg)
![Capture d’écran 2](./media/get-started-unity-hololens/screenshot-2.jpg)
![Capture d’écran 3](./media/get-started-unity-hololens/screenshot-3.jpg)
![Capture d’écran 4](./media/get-started-unity-hololens/screenshot-4.jpg)

Dans Visual Studio, arrêtez l’application en sélectionnant **Arrêter le débogage** ou en appuyant sur Maj+F5.

[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

[!INCLUDE [Next steps](../../../includes/spatial-anchors-quickstarts-nextsteps.md)]

> [!div class="nextstepaction"]
> [Tutoriel : Partager des ancres spatiales sur des appareils](../tutorials/tutorial-share-anchors-across-devices.md)

> [!div class="nextstepaction"]
> [Guide pratique pour configurer Azure Spatial Anchors dans un projet Unity](../how-tos/setup-unity-project.md)