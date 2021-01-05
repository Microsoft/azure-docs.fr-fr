---
title: Modération d’image - Content Moderator
titleSuffix: Azure Cognitive Services
description: Utilisez la modération d'image assistée par ordinateur et l'outil de révision à intervention humaine de Content Moderator pour modérer les images incluant du contenu pour adultes et osé.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: pafarley
ms.openlocfilehash: fe76e32bfd9b1734f3c84a400f897b7af7e3168b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85800993"
---
# <a name="learn-image-moderation-concepts"></a>Familiarisez-vous avec les concepts de modération d'image

Utilisez la modération de l’image assistée par ordinateur de Content Moderator et l’[outil de révision](Review-Tool-User-Guide/human-in-the-loop.md) pour modérer les images incluant du contenu pour adultes et osé. Recherchez du contenu textuel dans les images, extrayez le texte et détectez les visages. Vous pouvez comparer des images à des listes personnalisées et prendre des mesures complémentaires.

## <a name="evaluating-for-adult-and-racy-content"></a>Évaluer du contenu pour adultes et osé

L’opération **Évaluer** renvoie un score de confiance entre 0 et 1. Il renvoie également des données booléennes égales à true ou false. Ces valeurs prédisent si l’image contient un contenu potentiellement pour adultes ou osé. Lorsque vous appelez l’API avec votre image (fichier ou URL), la réponse renvoyée inclut les informations suivantes :

```json
"ImageModeration": {
    .............
    "adultClassificationScore": 0.019196987152099609,
    "isImageAdultClassified": false,
    "racyClassificationScore": 0.032390203326940536,
    "isImageRacyClassified": false,
    ............
    ],
```

> [!NOTE]
> 
> - `isImageAdultClassified` représente la présence d’images pouvant être considérées comme sexuellement explicites ou réservées aux adultes dans certaines situations.
> - `isImageRacyClassified` représente la présence d’images pouvant être considérées comme sexuellement suggestives ou réservées aux adultes dans certaines situations.
> - Les scores sont compris entre 0 et 1. Plus le score est élevé, plus le modèle prédit que la catégorie peut être applicable. Cette préversion s’appuie sur un modèle statistique plutôt que sur des résultats codés manuellement. Nous vous recommandons d’effectuer le test avec votre propre contenu afin de déterminer comment chaque catégorie correspond à vos besoins.
> - Les valeurs booléennes sont true ou false, selon les seuils de score internes. Les clients doivent évaluer l’opportunité d’utiliser cette valeur ou choisir des seuils personnalisés en fonction de leurs stratégies de contenu.

## <a name="detecting-text-with-optical-character-recognition-ocr"></a>Détecter le texte à l’aide de la reconnaissance optique de caractères (OCR)

La **reconnaissance optique de caractères (OCR)** prédit la présence de texte dans une image et l’extrait pour modération du texte, entre autres. Vous pouvez spécifier la langue. Si vous ne spécifiez pas de langue, la détection utilise l’anglais par défaut.

La réponse inclut les informations suivantes :
- Le texte d’origine.
- Les éléments de texte détectés et leur score de confiance.

Exemple d’extrait :

```json
"TextDetection": {
    "status": {
        "code": 3000.0,
        "description": "OK",
        "exception": null
    },
    .........
    "language": "eng",
    "text": "IF WE DID \r\nALL \r\nTHE THINGS \r\nWE ARE \r\nCAPABLE \r\nOF DOING, \r\nWE WOULD \r\nLITERALLY \r\nASTOUND \r\nOURSELVE \r\n",
    "candidates": []
},
```

## <a name="detecting-faces"></a>Détection des visages

La détection des visages permet de détecter des données personnelles dans les images, par exemple les visages. Vous détectez les visages potentiels et le nombre de visages potentiels dans chaque image.

Les réponses incluent ces informations :

- Nombre de visages
- Liste des emplacements où des visages ont été détectés

Exemple d’extrait :

```json
"FaceDetection": {
    ......
    "result": true,
    "count": 2,
    "advancedInfo": [
        .....
    ],
    "faces": [
        {
            "bottom": 598,
            "left": 44,
            "right": 268,
            "top": 374
        },
        {
            "bottom": 620,
            "left": 308,
            "right": 532,
            "top": 396
        }
    ]
}
```

## <a name="creating-and-managing-custom-lists"></a>Création et gestion de listes personnalisées

Dans de nombreuses communautés en ligne, lorsque les utilisateurs téléchargent des images ou un autre type de contenu, des éléments offensants peuvent être partagés plusieurs fois les jours, semaines et mois suivants. Les coûts répétés de l’analyse et du filtrage d’une même image ou de versions légèrement modifiées de l’image à partir de plusieurs emplacements peuvent être élevés et ces actions peuvent être sources d’erreurs.

Au lieu de modérer plusieurs fois la même image, vous ajoutez les images offensantes à votre liste personnalisée de contenu bloqué. De cette façon, votre système de modération du contenu compare les images entrantes à vos listes personnalisées et arrête tout traitement supplémentaire.

> [!NOTE]
> Il existe une limite maximale de **5 listes d’images**, chaque liste ne devant **pas dépasser 10 000 images**.
>

Content Moderator fournit une [API Gestion de liste d’images](try-image-list-api.md) complète, incluant les opérations de gestion des listes d’images personnalisées. Commencez avec la [Console d’API de listes d’images personnalisées](try-image-list-api.md) et utilisez les exemples de code API REST. Consultez également le [démarrage rapide .NET des listes d’images](image-lists-quickstart-dotnet.md) si vous connaissez déjà Visual Studio et C#.

## <a name="matching-against-your-custom-lists"></a>Mise en correspondance par rapport à vos listes personnalisées

L’opération de mise en correspondance permet une correspondance approximative des images entrantes par rapport à l’une de vos listes personnalisées, créées et gérées à l’aide des opérations de liste.

Si une correspondance est trouvée, l’opération renvoie l’identificateur et les balises de modération de l’image concernée. La réponse inclut ces informations :

- Score de correspondance (entre 0 et 1)
- Image concernée
- Balises d’image (attribuées au cours d’une précédente modération)
- Étiquettes d’image

Exemple d’extrait :

```json
{
    ..............,
    "IsMatch": true,
    "Matches": [
        {
            "Score": 1.0,
            "MatchId": 169490,
            "Source": "169642",
            "Tags": [],
            "Label": "Sports"
        }
    ],
    ....
}
```

## <a name="review-tool"></a>Outil de révision

Pour des cas plus subtils, utilisez l'[outil de révision](Review-Tool-User-Guide/human-in-the-loop.md) Content Moderator et son API pour faire apparaître les résultats de la modération et le contenu dans la révision à l'intention de vos opérateurs humains. Ils passent en revue les balises affectées par l’ordinateur et donnent leur décision finale.

![Révision d’images pour les modérateurs humains](images/moderation-reviews-quickstart-dotnet.PNG)

## <a name="next-steps"></a>Étapes suivantes

Évaluez la [Console d’API Modération de l’image](try-image-api.md) et utilisez les exemples de code API REST. Consultez également [Révisions, flux de travail et travaux](./review-api.md) pour en savoir plus sur le mode de configuration des révisions humaines.
