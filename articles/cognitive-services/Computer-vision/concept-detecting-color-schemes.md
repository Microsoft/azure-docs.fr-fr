---
title: Détection des modèles de couleurs - Vision par ordinateur
titleSuffix: Azure Cognitive Services
description: Concepts liés à la détection des jeux de couleurs dans les images à l’aide de l’API Vision par ordinateur.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 02/08/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: 4e7774d2cd100931f92ff80066ebea4463c6f65c
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532650"
---
# <a name="detect-color-schemes-in-images"></a>Détecter les jeux de couleurs dans les images

La Vision par ordinateur analyse les couleurs d’une image pour identifier trois attributs différents : la couleur de premier plan dominante, la couleur d’arrière-plan dominante et le jeu de couleurs dominantes de l’image dans sa globalité. Les couleurs retournées appartiennent au jeu : noir, bleu, marron, gris, vert, orange, rose, violet, rouge, bleu canard, blanc et jaune. 

La Vision par ordinateur extrait également une couleur d’accentuation, qui représente la couleur la plus éclatante de l’image, à partir d’une combinaison de couleurs dominantes et de saturation. La couleur d’accentuation est retournée sous la forme d’un code de couleur HTML hexadécimal. 

La Vision par ordinateur retourne également une valeur booléenne qui indique si l’image est en noir et blanc.

## <a name="color-scheme-detection-examples"></a>Exemples de détection de jeux de couleurs

L’exemple suivant illustre la réponse JSON retournée par Vision par ordinateur lors de la détection du jeu de couleurs de l’image d’exemple. Dans ce cas, l’exemple d’image n’est pas en noir et blanc, mais les couleurs de premier plan et d’arrière-plan dominantes sont le noir, tandis que les couleurs dominantes de l’image dans son ensemble sont le noir et le blanc.

![Montagne en extérieur au coucher du soleil, avec la silhouette d’une personne](./Images/mountain_vista.png)

```json
{
    "color": {
        "dominantColorForeground": "Black",
        "dominantColorBackground": "Black",
        "dominantColors": ["Black", "White"],
        "accentColor": "BB6D10",
        "isBwImg": false
    },
    "requestId": "0dc394bf-db50-4871-bdcc-13707d9405ea",
    "metadata": {
        "height": 202,
        "width": 300,
        "format": "Jpeg"
    }
}
```

### <a name="dominant-color-examples"></a>Exemples de couleur dominante

Le tableau suivant présente la couleur de premier plan, la couleur d’arrière-plan et la couleur de l’image retournées pour chaque exemple d’image.

| Image | Couleurs dominantes |
|-------|-----------------|
|![Fleur blanche sur fond vert](./Images/flower.png)| Premier plan : Noir<br/>Arrière-plan : White<br/>Couleurs : Noir, blanc, vert|
![Train traversant une gare](./Images/train_station.png) | Premier plan : Noir<br/>Arrière-plan : Noir<br/>Couleurs : Noir |

### <a name="accent-color-examples"></a>Exemples de couleur d’accentuation

 Le tableau suivant présente la couleur d’accentuation retournée, sous la forme d’une valeur de couleur HTML hexadécimale, pour chaque exemple d’image.

| Image | Couleur d’accentuation |
|-------|--------------|
|![Personne se tenant sur un rocher montagneux au coucher du soleil](./Images/mountain_vista.png) | #BB6D10 |
|![Fleur blanche sur fond vert](./Images/flower.png) | #C6A205 |
|![Train traversant une gare](./Images/train_station.png) | #474A84 |

### <a name="black--white-detection-examples"></a>Exemples de détection pour le noir et blanc

Le tableau suivant présente l’évaluation en noir et blanc de la Vision par ordinateur dans les exemples d’images.

| Image | Noir et blanc ? |
|-------|----------------|
|![Image noir et blanc représentant des immeubles de Manhattan](./Images/bw_buildings.png) | true |
|![Maison bleue et jardin en façade](./Images/house_yard.png) | false |

## <a name="use-the-api"></a>Utilisation de l’API

La fonctionnalité de détection des modèles de couleurs fait partie de l’API [Analyser l’image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b). Vous pouvez appeler cette API via un SDK natif ou via des appels REST. Incluez `Color` dans le paramètre de requête **visualFeatures**. Ensuite, lorsque vous obtenez la réponse JSON complète, analysez simplement la chaîne de contenu de la section `"color"`.

* [Démarrage rapide : API REST ou bibliothèques de client Vision par ordinateur](./quickstarts-sdk/client-library.md?pivots=programming-language-csharp)
