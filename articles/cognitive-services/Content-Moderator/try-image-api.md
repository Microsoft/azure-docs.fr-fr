---
title: Modérer des images avec la console d’API - Content Moderator
titleSuffix: Azure Cognitive Services
description: Utilisez l’API de modération d’images dans Azure Content Moderator pour lancer des workflows de modération d’analyse et de révision pour le contenu de l’image.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 01/10/2019
ms.author: pafarley
ms.openlocfilehash: 542fadd5e5ab91be7b7113064bf8c998dae08d12
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92912937"
---
# <a name="moderate-images-from-the-api-console"></a>Modérer des images à partir de la console d’API

Utilisez [l’API de modération d’images](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c) dans Azure Content Moderator pour lancer des workflows de modération d’analyse et de révision pour le contenu de l’image. Le travail de modération analyse votre contenu pour détecter les termes vulgaires et le compare à des listes rouges personnalisées et partagées.

## <a name="use-the-api-console"></a>Utiliser la console d’API
Avant de pouvoir tester l’API dans la console en ligne, vous avez besoin de votre clé d’abonnement. Elle se trouve sous l’onglet **Paramètres** , dans la zone **Ocp-Apim-Subscription-Key**. Pour plus d’informations, consultez la [vue d’ensemble ](overview.md) de Content Moderator.

1. Accédez aux [informations de référence sur l’API de modération d’images](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c).

   La page de modération d’images **Image - Évaluer** s’ouvre.

2. Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement. 

   ![Essayer l’image - Sélection de la région sur la page Évaluer](images/test-drive-region.png)
  
   La console d’API **Image - Évaluer** s’ouvre.

3. Dans la zone **Ocp-Apim-abonnement-Key** , entrez votre clé d’abonnement.

   ![Essayer l’image - Clé d’abonnement de la console Évaluer](images/try-image-api-1.PNG)

4. Dans la zone **Corps de la demande** , utilisez l’exemple d’image par défaut ou spécifiez une image à analyser. Vous pouvez envoyer l’image en tant que données de bit binaire, ou spécifier une URL accessible publiquement pour une image. 

   Pour cet exemple, utilisez le chemin d’accès fourni dans la zone **Corps de la demande** , puis sélectionnez **Envoyer**. 

   ![Essayer l’image - Corps de la demande de la console Évaluer](images/try-image-api-2.PNG)

   Il s’agit de l’image à cette URL :

   ![Essayer l’image - Exemple d’image de la console Évaluer](images/sample-image.jpg) 

5. Sélectionnez **Envoyer**.

6. L’API renvoie un score de probabilité pour chaque classification. Elle renvoie également une détermination indiquant si l’image répond aux conditions ( **true** ou **false** ). 

   ![Essayer l’image -  Score de probabilité et détermination de la condition de la console Évaluer](images/try-image-api-3.PNG)

## <a name="face-detection"></a>Détection des visages

Vous pouvez utiliser l’API de modération d’images pour localiser des visages dans une image. Cette option peut être utile lorsque vous avez des problèmes de confidentialité et que vous souhaitez empêcher la publication d’un visage spécifique sur votre plateforme. 

1. Dans la [référence sur l’API de modération d’images](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c), dans le menu de gauche, sous **Image** , sélectionnez **Find Faces** (Rechercher des visages). 

   La page **Image - Find Faces** (Image - Rechercher des visages) s’ouvre.

2. Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement. 

   ![Essayer l’image - Sélection de la région sur la page Find Faces (Rechercher des visages)](images/test-drive-region.png)

   La console d’API **Image - Find Faces** (Image - Rechercher des visages) s’ouvre.

3. Spécifiez une image à analyser. Vous pouvez envoyer l’image en tant que données de bit binaire, ou spécifier une URL accessible publiquement vers une image. Cet exemple associe une image utilisée dans une histoire CNN.

   ![Essayer l’image - Exemple d’image Find Faces (Rechercher des images)](images/try-image-api-face-image.jpg)

   ![Essayer l’image - Exemple de requête Find Faces (Rechercher des images)](images/try-image-api-face-request.png)

4. Sélectionnez **Envoyer**. Dans cet exemple, l’API recherche deux visages et renvoie leurs coordonnées dans l’image.

   ![Essayer l’image - Exemple de zone de contenu de réponse Find Faces (Rechercher des visages)](images/try-image-api-face-response.png)

## <a name="text-detection-via-ocr-capability"></a>Détection de texte via la fonctionnalité d’OCR

Vous pouvez utiliser la fonctionnalité d’OCR de Content Moderator pour détecter du texte dans des images.

1. Dans la [référence sur l’API de modération d’images](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c), dans le menu de gauche, sous **Image** , sélectionnez **OCR**. 

   La page **Image - OCR** s’ouvre.

2. Pour l’option **Open API testing console** (Ouvrir la console de test d’API), sélectionnez la région qui décrit le mieux votre emplacement. 

   ![Image - Sélection d’une région dans la page OCR](images/test-drive-region.png)

   La console d’API **Image - OCR** s’ouvre.

3. Dans la zone **Ocp-Apim-abonnement-Key** , entrez votre clé d’abonnement.

4. Dans la zone **Corps de la demande** , utilisez l’exemple d’image par défaut. Il s’agit de la même image que celle utilisée dans la section précédente.

5. Sélectionnez **Envoyer**. Le texte extrait est affiché au format JSON :

   ![Image - Exemple de zone de contenu de réponse OCR](images/try-image-api-ocr.PNG)

## <a name="next-steps"></a>Étapes suivantes

Utilisez l’API REST dans votre code ou suivez le [Démarrage rapide du kit SDK .NET](./client-libraries.md?pivots=programming-language-csharp%253fpivots%253dprogramming-language-csharp) pour ajouter la modération d’images à votre application.