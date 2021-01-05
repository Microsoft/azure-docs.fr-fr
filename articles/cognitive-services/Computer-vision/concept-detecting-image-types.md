---
title: Détection des types d’images - Vision par ordinateur
titleSuffix: Azure Cognitive Services
description: Concepts liés à la fonctionnalité de détection du type d’image de l'API Vision par ordinateur.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 03/11/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: 6d2ed00f3fc6f5b52a9a13a96f1e1659e30f02d5
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532599"
---
# <a name="detecting-image-types-with-computer-vision"></a>Détection des types d’images avec Vision par ordinateur

Avec l’API [Analyze Image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b), Vision par ordinateur analyse le type de contenu des images, indiquant si une image est de type clipart ou un dessin au trait.

## <a name="detecting-clip-art"></a>Détection des images clipart

Vision par ordinateur analyse une image et évalue sa probabilité d’être une image clipart sur une échelle de 0 à 3, comme décrit dans le tableau suivant.

| Valeur | Signification |
|-------|---------|
| 0 | Non-clip-art |
| 1 | Ambigu |
| 2 | Image clipart normale |
| 3 | 3 Image clipart de bonne qualité |

### <a name="clip-art-detection-examples"></a>Exemples de détection des images clipart

Les réponses JSON suivantes illustrent ce que renvoie Vision par ordinateur au moment d’évaluer la probabilité pour des images d’être des images clipart.

![Image clipart d’une tranche de fromage](./Images/cheese_clipart.png)

```json
{
    "imageType": {
        "clipArtType": 3,
        "lineDrawingType": 0
    },
    "requestId": "88c48d8c-80f3-449f-878f-6947f3b35a27",
    "metadata": {
        "height": 225,
        "width": 300,
        "format": "Jpeg"
    }
}
```

![Maison bleue et jardin en façade](./Images/house_yard.png)

```json
{
    "imageType": {
        "clipArtType": 0,
        "lineDrawingType": 0
    },
    "requestId": "a9c8490a-2740-4e04-923b-e8f4830d0e47",
    "metadata": {
        "height": 200,
        "width": 300,
        "format": "Jpeg"
    }
}
```

## <a name="detecting-line-drawings"></a>Détection des dessins au trait

Vision par ordinateur analyse une image et renvoie une valeur booléenne indiquant si l’image est un dessin au trait.

### <a name="line-drawing-detection-examples"></a>Exemples de détection de dessin au trait

Les réponses JSON suivantes illustrent ce que Vision par ordinateur renvoie si les images fournies à titre d’exemple sont des dessins au trait.

![Image de dessin au trait d’un lion](./Images/lion_drawing.png)

```json
{
    "imageType": {
        "clipArtType": 2,
        "lineDrawingType": 1
    },
    "requestId": "6442dc22-476a-41c4-aa3d-9ceb15172f01",
    "metadata": {
        "height": 268,
        "width": 300,
        "format": "Jpeg"
    }
}
```

![Fleur blanche sur fond vert](./Images/flower.png)

```json
{
    "imageType": {
        "clipArtType": 0,
        "lineDrawingType": 0
    },
    "requestId": "98437d65-1b05-4ab7-b439-7098b5dfdcbf",
    "metadata": {
        "height": 200,
        "width": 300,
        "format": "Jpeg"
    }
}
```

## <a name="use-the-api"></a>Utilisation de l’API

La fonctionnalité de détection de type d’image fait partie de l’API [Analyser l’image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b). Vous pouvez appeler cette API via un SDK natif ou via des appels REST. Incluez `ImageType` dans le paramètre de requête **visualFeatures**. Ensuite, lorsque vous obtenez la réponse JSON complète, analysez simplement la chaîne de contenu de la section `"imageType"`.

* [Démarrage rapide : API REST ou bibliothèques de client Vision par ordinateur](./quickstarts-sdk/client-library.md?pivots=programming-language-csharp)
