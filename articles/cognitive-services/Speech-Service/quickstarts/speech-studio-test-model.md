---
title: Tester un modèle avec des fichiers audio - Speech Studio
titleSuffix: Azure Cognitive Services
description: Dans ce guide pratique, vous utilisez Speech Studio pour tester la reconnaissance de la voix dans un fichier audio.
services: cognitive-services
author: v-demjoh
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 09/04/2020
ms.author: v-demjoh
ms.openlocfilehash: 572b3b3459e1d837130f3c987d45ee45629f37ad
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96485074"
---
# <a name="test-a-model-using-an-audio-file-in-speech-studio"></a>Tester un modèle à l’aide d’un fichier audio dans Speech Studio

Dans ce guide pratique, vous utilisez Speech Studio pour convertir la voix d’un fichier audio en texte. Speech Studio vous permet de tester, de comparer, d’améliorer et de déployer des modèles de reconnaissance vocale à l’aide des textes associés, d’audio avec des transcriptions manuelles et des conseils de prononciation que vous fournissez.

## <a name="prerequisites"></a>Prérequis

Avant d’utiliser le portail Speech, [suivez ces instructions pour créer un compte Azure et vous abonner au service Speech](../custom-speech-overview.md#set-up-your-azure-account). Cet abonnement unifié vous donne accès à la reconnaissance vocale, à la synthèse vocale, à la traduction vocale et au portail Custom Speech.

## <a name="download-an-audio-file"></a>Télécharger un fichier audio

Procédez comme suit pour télécharger un fichier audio qui contient la reconnaissance vocale et l’empaqueter dans un fichier zip.

1. Téléchargez l’ **[exemple de fichier wav](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-speech-sdk/f9807b1079f3a85f07cbb6d762c6b5449d536027/samples/cpp/windows/console/samples/whatstheweatherlike.wav)** en cliquant avec le bouton droit sur le lien et en sélectionnant **Enregistrer la cible du lien sous**. Cliquez sur **Enregistrer** pour télécharger le fichier `whatstheweatherlike.wav`.
2. À l’aide d’un explorateur de fichiers ou d’une fenêtre de terminal doté d’un outil zip, créez un fichier zip nommé `whatstheweatherlike.zip` contenant le fichier `whatstheweatherlike.wav` que vous avez téléchargé. Dans Windows, ouvrez l’Explorateur Windows, accédez au dossier `Downloads`, cliquez avec le bouton droit sur `whatstheweatherliike.wav`, cliquez sur **Envoyer vers**, cliquez sur **Dossier compressé**, puis appuyez sur Entrée pour accepter le nom de fichier par défaut.

## <a name="create-a-project-in-the-custom-speech-portal"></a>Créer un projet dans le portail Custom Speech

Procédez comme suit pour créer un projet qui contient le fichier zip d’un fichier audio.

1. Ouvrez [Speech Studio](https://speech.microsoft.com/), puis cliquez sur **Nouveau projet**. Tapez le nom de ce projet, puis cliquez sur **Créer**. Votre projet s’affiche dans la liste Custom Speech.
2. Cliquez sur le nom de votre projet. Dans l’onglet Données, cliquez sur **Charger des données**.
3. Par défaut, le type de données Speech est défini sur **Audio uniquement**. Par conséquent, cliquez sur **Suivant**.
4. Nommez votre nouveau jeu de données de reconnaissance vocale `MyZipOfAudio`, puis cliquez sur **Suivant**.
5. Cliquez sur **Parcourir les fichiers...** , accédez à votre fichier `whatstheweatherlike.zip`, puis cliquez sur **Ouvrir**.
6. Cliquez sur le bouton **Télécharger** . Le navigateur charge votre fichier zip dans Speech Studio qui traite le contenu.

## <a name="test-a-model"></a>Tester un modèle

Une fois que Speech Studio a traité le contenu de votre fichier zip, vous pouvez lire l’audio source tout en examinant la transcription pour rechercher des erreurs ou des omissions. Procédez comme suit pour examiner la qualité des transcriptions dans le navigateur.

1. Cliquez sur l’onglet **Test**, puis sur **Ajouter un test**.
2. Dans ce test, inspectez la qualité des données audio uniquement. Par conséquent, cliquez sur **Suivant** pour accepter ce type de test.
3. Nommez ce test `MyModelTest`, puis cliquez sur **Suivant**.
4. Cliquez sur la case d’option située à gauche de `MyZipOfAudio`, puis cliquez sur **Suivant**.
5. Comme la liste déroulante **Modèle 1** est par défaut le dernier modèle de reconnaissance, cliquez sur **Créer**. Après le traitement du contenu de votre jeu de données audio, l’état du test passe à **Réussite**.
6. Cliquez sur **MyModelTest**. Les résultats de la reconnaissance vocale s’affichent. Cliquez sur le triangle pointant vers la droite dans le cercle pour écouter l’audio et comparez ce que vous entendez avec le texte à côté du cercle.

## <a name="download-detailed-results"></a>Télécharger les résultats détaillés

Vous pouvez télécharger des fichiers qui décrivent les transcriptions de manière plus détaillée. Les fichiers incluent une forme lexicale des paroles dans vos fichiers audio, ainsi que des fichiers JSON qui contiennent des détails sur l’offset, la durée et le niveau de confiance de la transcription pour chaque mot. Pour afficher ces fichiers, procédez comme suit.

1. Cliquez sur **Télécharger**.
2. Dans la boîte de dialogue de téléchargement, désélectionnez **Audio**, puis cliquez sur **Télécharger**.
3. Décompressez le fichier zip téléchargé, puis examinez les fichiers extraits.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur l’amélioration de la précision de la reconnaissance vocale en [entraînant un modèle personnalisé](../how-to-custom-speech-test-and-train.md).