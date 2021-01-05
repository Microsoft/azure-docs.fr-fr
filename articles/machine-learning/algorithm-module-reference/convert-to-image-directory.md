---
title: Convertir en répertoire d’images
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Convertir en répertoire d’images pour convertir un jeu de données au format de répertoire d’images.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 11/12/2020
ms.openlocfilehash: 1489ce74da2ecff5212feb5a1a2e3c9151b73424
ms.sourcegitcommit: dc342bef86e822358efe2d363958f6075bcfc22a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94555581"
---
# <a name="convert-to-image-directory"></a>Convertir en répertoire d’images

Cet article explique comment utiliser le module Convertir en répertoire d’images pour faciliter la conversion d’un jeu de données d’images en type de données *Répertoire d’images*, qui est un format de données standardisé dans les tâches associées aux images, telles que la classification d’images dans le concepteur Azure Machine Learning.

## <a name="how-to-use-convert-to-image-directory"></a>Utilisation du module Convertir en répertoire d’images  

1. Commencez par préparer votre jeu de données d’images. 

    Pour l’apprentissage supervisé, vous devez spécifier l’étiquette du jeu de données de formation. Le fichier du jeu de données d’images doit se situer dans la structure suivante :
    
    ```
    Your_image_folder_name/Category_1/xxx.png
    Your_image_folder_name/Category_1/xxy.jpg
    Your_image_folder_name/Category_1/xxz.jpeg
    
    Your_image_folder_name/Category_2/123.png
    Your_image_folder_name/Category_2/nsdf3.png
    Your_image_folder_name/Category_2/asd932_.png
    ```
    
    Le dossier du jeu de données d’images contient plusieurs sous-dossiers. Chaque sous-dossier contient des images d’une catégorie, respectivement. Les noms des sous-dossiers sont considérés comme les étiquettes des tâches telles que la classification d’images. Pour plus d’informations, consultez les [jeux de données Torchvision](https://pytorch.org/docs/stable/torchvision/datasets.html#imagefolder).

    > [!WARNING]
    > Les jeux de données actuellement étiquetés exportés à partir de l’étiquetage des données ne sont pas pris en charge dans le concepteur.

    Les images avec ces extensions (en minuscules) sont prises en charge : « .jpg », « .jpeg », « .png », « .ppm », « .bmp », « .pgm », « .tif », « .tiff », « .webp ». Vous pouvez également avoir plusieurs types d’images dans un dossier. Il n’est pas nécessaire d’inclure le même nombre d’images dans chaque dossier de catégorie.

    Vous pouvez utiliser le dossier ou le fichier compressé avec l’extension « .zip », « .tar », « .gz » et « .bz2 ». **Il est recommandé d’utiliser les fichiers compressés pour obtenir de meilleures performances.** 
    
    ![Exemple de jeu de données d’images](./media/module/image-sample-dataset.png)

    > [!NOTE]
    > Pour l’inférence, le dossier du jeu de données d’images doit contenir uniquement des images non classifiées.

1. [Enregistrez le jeu de données d’images en tant que jeu de données de fichiers](../how-to-create-register-datasets.md) dans votre espace de travail, car l’entrée du module Convertir en répertoire d’images doit être un **jeu de données de fichiers**.

1. Ajoutez le jeu de données d’images enregistré au canevas. Vous trouverez votre jeu de données enregistré dans la catégorie **Jeux de données** dans la liste des modules à gauche du canevas. Actuellement, le concepteur ne prend pas en charge la visualisation du jeu de données d’images.

    > [!WARNING]
    > Vous **ne pouvez pas** utiliser le module **Importer des données** pour importer un jeu de données image, car le type de sortie du module **Importer des données** est le répertoire DataFrame, qui contient uniquement la chaîne de chemin de fichier.

1. Ajoutez le module **Convertir en répertoire d’images** au canevas. Vous trouverez ce module dans la catégorie « Computer Vision/Image Data Transformation » (Vision par ordinateur/Transformation de données d’image) de la liste des modules. Connectez-le au jeu de données d’images.
    
3.  Envoyez le pipeline. Ce module peut être exécuté sur le GPU ou l’UC.

## <a name="results"></a>Résultats

La sortie du module **Convertir en répertoire d’images** est au format **Répertoire d’images** et peut être connectée à d’autres modules associés à des images dont le format du port d’entrée est également Répertoire d’images.

![Sortie Convertir en répertoire d’images](./media/module/convert-to-image-directory-output.png)

## <a name="technical-notes"></a>Notes techniques 

###  <a name="expected-inputs"></a>Entrées attendues  

| Nom          | Type                  | Description   |
| ------------- | --------------------- | ------------- |
| Jeu de données d'entrée | AnyDirectory, ZipFile | Jeu de données d'entrée |

###  <a name="output"></a>Output  

| Nom                   | Type           | Description            |
| ---------------------- | -------------- | ---------------------- |
| Répertoire d’images de sortie | ImageDirectory | Répertoire d’images de sortie |

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning.