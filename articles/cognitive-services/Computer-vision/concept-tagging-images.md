---
title: Étiquettes de contenu - Vision par ordinateur
titleSuffix: Azure Cognitive Services
description: Découvrez les concepts liés à la fonction de balises d’images de l’API Vision par ordinateur.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 02/08/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: a75e55cd51890173b6733ab5f8c0f1bcbfeef0f5
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532973"
---
# <a name="applying-content-tags-to-images"></a>Application de balises de contenu aux images

Vision par ordinateur retourne des balises basées sur des milliers d’objets, d’êtres vivants, de scènes et d’actions reconnaissables. Lorsque les balises sont ambigües ou inhabituelles, la réponse de l’API fournit des « conseils » pour expliquer la signification de la balise dans le contexte des paramètres connus. Les balises ne sont pas organisées sous forme de taxonomie et aucune hiérarchie d’héritage n’existe. La collection de balises de contenu constitue la « description » essentielle d’une image, qui est affichée sous forme de texte lisible par l’homme (phrases complètes). Notez que, à ce stade, seul l’anglais est pris en charge pour la description d’images.

Après le chargement d’une image ou d’une URL d’image, les algorithmes de l’API Vision par ordinateur génèrent des balises basées sur les objets, les êtres vivants et les actions identifiées dans l’image. Le balisage ne se limite pas au sujet principal, comme une personne au premier plan, mais il inclut également le décor (intérieur ou extérieur), le mobilier, les outils, les plantes, les animaux, les accessoires, les gadgets, etc.

## <a name="image-tagging-example"></a>Exemple de balisage d’images

Les réponses JSON suivantes illustrent ce que renvoie Vision par ordinateur au moment du balisage d’éléments visuels détectés dans l’image en exemple.

![Maison bleue et jardin en façade](./Images/house_yard.png).

```json
{
    "tags": [
        {
            "name": "grass",
            "confidence": 0.9999995231628418
        },
        {
            "name": "outdoor",
            "confidence": 0.99992108345031738
        },
        {
            "name": "house",
            "confidence": 0.99685388803482056
        },
        {
            "name": "sky",
            "confidence": 0.99532157182693481
        },
        {
            "name": "building",
            "confidence": 0.99436837434768677
        },
        {
            "name": "tree",
            "confidence": 0.98880356550216675
        },
        {
            "name": "lawn",
            "confidence": 0.788884699344635
        },
        {
            "name": "green",
            "confidence": 0.71250593662261963
        },
        {
            "name": "residential",
            "confidence": 0.70859086513519287
        },
        {
            "name": "grassy",
            "confidence": 0.46624681353569031
        }
    ],
    "requestId": "06f39352-e445-42dc-96fb-0a1288ad9cf1",
    "metadata": {
        "height": 200,
        "width": 300,
        "format": "Jpeg"
    }
}
```

## <a name="use-the-api"></a>Utilisation de l’API

La fonctionnalité de balisage fait partie de l’API [Analyser l’image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b). Vous pouvez appeler cette API via un SDK natif ou via des appels REST. Incluez `Tags` dans le paramètre de requête **visualFeatures**. Ensuite, lorsque vous obtenez la réponse JSON complète, analysez simplement la chaîne de contenu de la section `"tags"`.

* [Démarrage rapide : API REST ou bibliothèques de client Vision par ordinateur](./quickstarts-sdk/client-library.md?pivots=programming-language-csharp)

## <a name="next-steps"></a>Étapes suivantes

Découvrez les concepts associés de [classement des images](concept-categorizing-images.md) et de [description des images](concept-describing-images.md).
