---
title: Utiliser un modèle ONNX avec Windows ML - Service Custom Vision
titleSuffix: Azure Cognitive Services
description: Découvrez comment créer une application UWP Windows qui utilise un modèle ONNX exporté à partir d’Azure Cognitive Services.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 04/29/2020
ms.author: pafarley
ms.openlocfilehash: 58ced0c45d66223ac3e40112126e92a4539db32d
ms.sourcegitcommit: 9706bee6962f673f14c2dc9366fde59012549649
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94616074"
---
# <a name="use-an-onnx-model-from-custom-vision-with-windows-ml-preview"></a>Utiliser un modèle ONNX à partir de Custom Vision avec Windows ML (préversion)

Découvrez comment utiliser un modèle ONNX exporté à partir du service Custom Vision avec Windows ML (préversion).

Dans ce guide, vous allez découvrir comment utiliser un fichier ONNX exporté à partir du service Custom Vision avec Windows ML. Vous allez utiliser l’exemple d’application UWP avec votre propre classifieur d’images entraîné.

## <a name="prerequisites"></a>Prérequis

* Windows 10 version 1809 ou ultérieure
* Microsoft Windows SDK pour build 17763 ou ultérieure
* Visual Studio 2017 version 15.7 ou ultérieure avec la charge de travail __Développement pour la plateforme Windows universelle__ activée.
* Mode Développeur activé sur votre PC. Pour plus d’informations, consultez [Activer votre appareil pour le développement](/windows/uwp/get-started/enable-your-device-for-development).

## <a name="about-the-example-app"></a>À propos de l’exemple d’application

L’application incluse est une application UWP Windows générique. Elle vous permet de sélectionner une image à partir de votre ordinateur, laquelle est ensuite traitée par un modèle de classification stocké localement. Les balises et les scores renvoyés par le modèle sont affichés en regard de l’image.

## <a name="get-the-example-code"></a>Obtenir l’exemple de code

L’exemple d’application est disponible dans le dépôt [Exemple Custom Vision de modèle ONNX de Cognitive Services](https://github.com/Azure-Samples/cognitive-services-onnx-customvision-sample) sur GitHub. Clonez-le sur votre machine locale, puis ouvrez *SampleOnnxEvaluationApp.sln* dans Visual Studio.

## <a name="test-the-application"></a>Test de l’application

1. Utilisez la touche `F5` pour démarrer l’application à partir de Visual Studio. Il peut vous être demandé d’activer le mode Développeur.
1. Lorsque l’application démarre, utilisez le bouton pour sélectionner une image pour le scoring. Le modèle ONNX par défaut est entraîné pour classifier différents types de plancton.

## <a name="use-your-own-model"></a>Utiliser votre propre modèle

Pour utiliser votre propre modèle de classifieur d’images, effectuez les étapes suivantes :

1. Créez et entraînez un classifieur avec le service Custom Vision. Pour obtenir des instructions sur la façon de procéder, consultez [Créer et entraîner un classifieur](./getting-started-build-a-classifier.md). Utilisez l’un des domaines **compacts**, par exemple **General (compact)** (Général (compact)). 
   * Si vous disposez déjà d’un classifieur qui utilise un autre domaine, vous pouvez le convertir en **compact** dans les paramètres du projet. Réentraînez ensuite votre projet avant de poursuivre.
1. Exportez votre modèle. Basculez vers l’onglet Performances, puis sélectionnez une itération qui a été entraînée avec un domaine **compact**. Sélectionnez ensuite le bouton **Exporter** qui s’affiche. Sélectionnez ensuite **ONNX**, puis **Exporter**. Une fois que le fichier est prêt, sélectionnez le bouton **Télécharger**. Pour plus d’informations sur les options d’exportation, consultez [Exporter votre modèle](./export-your-model.md).
1. Ouvrez le fichier  *.zip* téléchargé, puis extrayez le fichier *model.onnx* de celui-ci. Ce fichier contient votre modèle de classifieur.
1. Dans l’Explorateur de solutions de Visual Studio, cliquez avec le bouton droit sur le dossier **Ressources**, puis sélectionnez __Ajouter un élément existant__. Sélectionnez votre fichier ONNX.
1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le fichier ONNX, puis sélectionnez **Propriétés**. Modifiez les propriétés suivantes du fichier :
   * __Action de génération__ -> __Contenu__
   * __Copier dans le répertoire de sortie__ -> __Copier si plus récent__
1. Ouvrez ensuite _MainPage.xaml.cs_, puis remplacez la valeur de `_ourOnnxFileName` par le nom de votre fichier ONNX.
1. Utilisez `F5` pour générer et exécuter le projet.
1. Cliquez sur le bouton pour sélectionner l’image à évaluer.

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir d’autres façons d’exporter et d’utiliser un modèle Custom Vision, consultez les documents suivants :

* [Exporter votre modèle](./export-your-model.md)
* [Utiliser le modèle Tensorflow exporté dans une application Android](https://github.com/Azure-Samples/cognitive-services-android-customvision-sample)
* [Utiliser le modèle CoreML exporté dans une application Swift iOS](https://go.microsoft.com/fwlink/?linkid=857726)
* [Utiliser le modèle CoreML exporté dans une application iOS avec Xamarin](https://github.com/xamarin/ios-samples/tree/master/ios11/CoreMLAzureModel)

Pour plus d’informations sur l’utilisation de modèles ONNX avec Windows ML, consultez [Intégrer un modèle à votre application avec Windows ML](/windows/ai/windows-ml/integrate-model).