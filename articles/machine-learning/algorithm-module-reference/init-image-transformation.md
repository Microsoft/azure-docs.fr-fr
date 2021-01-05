---
title: Initialiser une transformation d’image – Applique une transformation d’image
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Init Image Transformation dans le concepteur Azure Machine Learning Designer pour initialiser une transformation d’image.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 05/26/2020
ms.openlocfilehash: fc0eb196ed24e413c35d64f0571ff29dc3725032
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421275"
---
# <a name="init-image-transformation"></a>Lancer une transformation d’image

Cet article explique comment utiliser le module **Init Image Transformation** (Initialiser une transformation d’image) dans le concepteur Azure Machine Learning pour initialiser la transformation d’une image afin de spécifier la façon dont vous souhaitez que l’image soit transformée.

## <a name="how-to-configure-init-image-transformation"></a>Comment configurer le module Init Image Transformation

1.  Ajoutez le module **Init Image Transformation** à votre pipeline dans le concepteur. 

2.  Pour **Resize** (Redimensionner), spécifiez s’il faut redimensionner l’image PIL en entrée à la taille donnée. Si vous choisissez « True », vous pouvez spécifier la taille de l’image de sortie souhaitée dans **Size** (Taille). Par défaut, la taille est 256. 

3.  Pour **Center crop** (Rognage au centre), spécifiez s’il faut rogner l’image PIL donnée au centre. Si vous choisissez « True », vous pouvez spécifier la taille d’image de sortie souhaitée du rognage dans **Crop size** (Taille de rognage). Par défaut, la taille est 224.  

4.  Pour **Pad** (Remplir), spécifiez s’il faut remplir l’image PIL donnée sur tous les côtés avec la valeur de remplissage 0. Si vous choisissez « True », vous pouvez spécifier le remplissage (nombre de pixels à ajouter) sur chaque côté dans **Padding** (Remplissage).

5.  Pour **Color jitter** (Gigue des couleurs), indiquez si vous souhaitez modifier de manière aléatoire la luminosité, le contraste et la saturation d’une image.

6.  Pour **Grayscale** (Nuances de gris), indiquez si l’image doit être convertie en nuances de gris.

7.  Pour **Random resized crop** (Rognage redimensionné de façon aléatoire), indiquez si vous souhaitez rogner l’image PIL donnée à une taille et des proportions aléatoires. Un rognage de taille aléatoire (entre 0,08 et 1,0) de la taille d’origine et de proportions aléatoires (entre 3/4 et 4/3) par rapport aux proportions d’origine est effectué. Ce rognage est finalement redimensionné à une taille donnée.
    Il est couramment utilisé pour l’apprentissage des réseaux de départ. Si vous choisissez « True », vous pouvez spécifier la taille de sortie attendue de chaque côté dans **Random size** (Taille aléatoire). Par défaut, la taille est 256.

8.  Pour **Random crop** (Rognage aléatoire), spécifiez s’il faut rogner l’image PIL donnée à un emplacement aléatoire. Si vous choisissez « True », vous pouvez spécifier la taille de sortie souhaitée du rognage dans **Random crop size** (Taille de rognage aléatoire). Par défaut, la taille est 224.

9.  Pour **Random horizontal flip** (retournement horizontal aléatoire), spécifiez s’il faut retourner horizontalement l’image PIL donnée de façon aléatoire avec une probabilité de 0,5.

10.  Pour **Random vertical flip** (retournement vertical aléatoire), spécifiez s’il faut retourner verticalement l’image PIL donnée de façon aléatoire avec une probabilité de 0,5.

11.  Pour **Random rotation** (Rotation aléatoire), spécifiez s’il faut applique à l’image un angle de rotation. Si vous choisissez « True », vous pouvez spécifier la plage de degrés en définissant **Random rotation degrees** (Degrés de rotation aléatoire), ce qui signifie (-degrés, + degrés). Par défaut, la valeur est 0 degré.

12.  Pour **Random affine** (Transformation affine aléatoire), spécifiez s’il faut effectuer une transformation affine aléatoire de l’image en conservant le centre invariant. Si vous choisissez « True », vous pouvez spécifier la plage de degrés à sélectionner dans **Random affine degrees** (Degrés de transformation affine aléatoire), ce qui signifie (-degrés, + degrés). Par défaut, la valeur est 0 degré.

13.  Pour **Random grayscale** (Nuances de gris aléatoires), spécifiez si l’image doit être convertie de façon aléatoire en nuances de gris avec une probabilité de 0,1.

14.  Pour **Random perspective** (Perspective aléatoire), spécifiez s’il faut effectuer une transformation de perspective de l’image PIL donnée de façon aléatoire avec une probabilité 0,5.


16.  Connectez-vous au module [Apply Image Transformation](apply-image-transformation.md) (Appliquer une transformation d’image), pour appliquer la transformation spécifiée ci-dessus au jeu de données d’image d’entrée.

17. Envoyez le pipeline.

## <a name="results"></a>Résultats

Une fois la transformation terminée, vous pouvez rechercher les images transformées dans la sortie du module [Apply Image Transformation](apply-image-transformation.md) (Appliquer une transformation d’image).


## <a name="technical-notes"></a>Notes techniques  

Pour plus d’informations sur la transformation d’image, consultez [https://pytorch.org/docs/stable/torchvision/transforms.html](https://pytorch.org/docs/stable/torchvision/transforms.html).

###  <a name="module-parameters"></a>Paramètres du module  

| Nom                    | Plage   | Type    | Default | Description                              |
| ----------------------- | ------- | ------- | ------- | ---------------------------------------- |
| Redimensionner                  | Quelconque     | Boolean | True    | Redimensionner l’image PIL d’entrée à la taille donnée |
| Taille                    | >=1     | Integer | 256     | Spécifier la taille de sortie souhaitée          |
| Center crop (Rognage au centre)             | Quelconque     | Boolean | True    | Rogne l’image PIL donnée au centre  |
| Crop size (Taille de rognage)               | >=1     | Integer | 224     | Spécifier la taille de sortie souhaitée du rognage |
| Remplir                     | Quelconque     | Boolean | False   | Remplir l’image PIL donnée sur tous les côtés avec la valeur « Pad » (remplissage) donnée |
| Remplissage                 | >=0     | Integer | 0       | Remplissage sur chaque côté                   |
| Color jitter (Gigue des couleurs)            | Quelconque     | Boolean | False   | Modifier de manière aléatoire la luminosité, le contraste et la saturation d’une image. |
| Grayscale (Nuances de gris)               | Quelconque     | Boolean | False   | Convertir l’image en nuances de gris               |
| Random resized crop (Rognage redimensionné de façon aléatoire)     | Quelconque     | Boolean | False   | Rogner l’image PIL donnée à une taille et des proportions aléatoires |
| Random size (Taille aléatoire)             | >=1     | Integer | 256     | Taille de sortie attendue de chaque côté        |
| Random crop (Rognage aléatoire)             | Quelconque     | Boolean | False   | Rogner l’image PIL donnée à un emplacement aléatoire |
| Random crop size (Taille de rognage aléatoire)        | >=1     | Integer | 224     | Taille de sortie souhaitée du rognage          |
| Random horizontal flip (Retournement horizontal aléatoire)  | Quelconque     | Boolean | True    | Retourner l’image PIL donnée horizontalement de façon aléatoire avec une probabilité donnée |
| Random vertical flip (Retournement vertical aléatoire)    | Quelconque     | Boolean | False   | Retourner l’image PIL donnée verticalement de façon aléatoire avec une probabilité donnée |
| Random rotation (Rotation aléatoire)         | Quelconque     | Boolean | False   | Appliquer un angle de rotation à l’image                |
| Random rotation degrees (Degrés de rotation aléatoire) | [0,180] | Integer | 0       | Plage de degrés dans laquelle sélectionner          |
| Random affine (Transformation affine aléatoire)           | Quelconque     | Boolean | False   | Transformation affine aléatoire de l’image en conservant le centre invariant |
| Random affine degrees (Degrés de transformation affine aléatoire)   | [0,180] | Integer | 0       | Plage de degrés dans laquelle sélectionner          |
| Random grayscale (Nuances de gris aléatoires)        | Quelconque     | Boolean | False   | Convertir de manière aléatoire l’image en nuances de gris avec une probabilité de 0,1 |
| Random perspective (Perspective aléatoire)      | Quelconque     | Boolean | False   | Effectuer une transformation de perspective de l’image PIL donnée de façon aléatoire avec une probabilité de 0,5 |
| Effacement aléatoire          | Quelconque     | Boolean | False   | Sélectionner de façon aléatoire une zone rectangulaire dans une image et effacer ses pixels avec une probabilité de 0,5 |

###  <a name="output"></a>Output  

| Nom                        | Type                    | Description                              |
| --------------------------- | ----------------------- | ---------------------------------------- |
| Transformation d’image de sortie | TransformationDirectory | Transformation d’image de sortie qui peut être connectée au module **Apply Image Transformation** (Appliquer une transformation d’image). |

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 