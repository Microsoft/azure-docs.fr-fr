---
title: Entraîner un modèle Pytorch
titleSuffix: Azure Machine Learning
description: Utilisez le module Entraîner un modèle Pytorch dans le concepteur Azure Machine Learning pour effectuer l’apprentissage de modèles à partir de zéro ou pour ajuster les modèles existants.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 09/26/2020
ms.openlocfilehash: 2d88069f33995bdbe9dd479afe9a4e72ab9939b6
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93420663"
---
# <a name="train-pytorch-model"></a>Entraîner un modèle Pytorch

Cet article explique comment utiliser le module **Train Pytorch Model** (Entraîner un modèle PyTorch) dans le concepteur Azure Machine Learning pour entraîner des modèles PyTorch, tels que DenseNet. L’entraînement a lieu une fois que vous avez défini un modèle et ses paramètres. De plus, il nécessite des données étiquetées. 

## <a name="how-to-use-train-pytorch-model"></a>Comment entraîner un modèle PyTorch 

1. Ajoutez le module [DenseNet](densenet.md) ou [ResNet](resnet.md) à votre brouillon de pipeline dans le concepteur.

2. Ajoutez le module **Train Pytorch Model** à votre pipeline. Vous trouverez ce module dans la catégorie **Model Training** (Entraînement de modèle). Développez **Train** (Entraîner), puis faites glisser le module **Train Pytorch Model** vers votre pipeline.

   > [!NOTE]
   > Il est préférable d’exécuter le module **Train Pytorch Model** sur une cible de calcul de type **GPU** pour les jeux de données volumineux, à défaut de quoi votre pipeline échouera. Vous pouvez sélectionner le calcul du module spécifique dans le volet droit du module en définissant **Use other compute target** (Utiliser une autre cible de calcul).

3.  Dans l’entrée gauche, attachez un modèle non entraîné. Attachez le jeu de données d’entraînement et le jeu de données de validation à l’entrée du milieu et de droite du module **Train Pytorch Model**.

    Pour le modèle non entraîné, il doit s’agir d’un modèle PyTorch tel que DenseNet. Sinon, une exception « InvalidModelDirectoryError » est levée.

    Pour le jeu de données, le jeu de données d’entraînement doit être un répertoire d’images étiquetées. Consultez **Convertir en répertoire d’images** pour savoir comment obtenir un répertoire d’images étiquetées. En l’absence d’étiquettes, une exception « NotLabeledDatasetError » est levée.

    Le jeu de données d’entraînement et le jeu de données de validation ont les mêmes catégories d’étiquette. Sinon une exception InvalidDatasetError est levée.

4.  Pour **Epochs** (Époques), spécifiez le nombre d’époques à entraîner. L’ensemble du jeu de données est itéré à chaque époque. La valeur par défaut est 5.

5.  Pour **Batch size** (Taille du lot), spécifiez le nombre d’instances à entraîner dans un lot. La valeur par défaut est 16.

6.  Pour **Taux d’apprentissage**, spécifiez une valeur pour le *taux d’apprentissage*. La valeur du taux d’apprentissage contrôle la taille de l’étape utilisée dans l’optimiseur, par exemple le SGD (descente du gradient stochastique) chaque fois que le modèle est testé et corrigé.

    En réduisant ce taux, vous testez le modèle plus souvent, avec le risque de rester bloqué dans un plateau local. En l’augmentant, vous pouvez converger plus rapidement, au risque de dépasser les minima réels. 0\.001 par défaut.

7.  Pour **Random seed** (Valeur initiale aléatoire), tapez éventuellement une valeur entière à utiliser en tant que valeur initiale. L’utilisation d’un seed est recommandée si vous souhaitez garantir la reproductibilité de l’essai au cours des exécutions.

8.  Pour **Patience**, spécifiez le nombre d’époques où l’entraînement doit être arrêté de manière anticipée si la perte de validation ne diminue pas de manière consécutive. 3 par défaut.

9.  Envoyez le pipeline. Si votre jeu de données a une taille supérieure, cela prendra du temps et le calcul GPU est recommandé.

## <a name="results"></a>Résultats

Une fois l’exécution du pipeline effectuée, si vous souhaitez utiliser le modèle à des fins de scoring, connectez le module [Train Pytorch Model](train-pytorch-model.md) (Entraîner un modèle PyTorch) au module [Score Image Model](score-image-model.md) (Scorer un modèle d’image) pour prédire les valeurs des nouveaux exemples d’entrée.

## <a name="technical-notes"></a>Notes techniques
###  <a name="expected-inputs"></a>Entrées attendues  

| Nom               | Type                    | Description                              |
| ------------------ | ----------------------- | ---------------------------------------- |
| Untrained model (Modèle non entraîné)    | UntrainedModelDirectory | Modèle non entraîné, nécessite PyTorch         |
| Jeu de données d’entraînement   | ImageDirectory          | Jeu de données d’entraînement                         |
| Jeu de données de validation | ImageDirectory          | Jeu de données de validation pour l’évaluation de chaque époque |

###  <a name="module-parameters"></a>Paramètres du module  

| Nom          | Plage            | Type    | Default | Description                              |
| ------------- | ---------------- | ------- | ------- | ---------------------------------------- |
| Époques        | >0               | Integer | 5       | Sélectionnez la colonne qui contient le libellé ou la colonne de résultat |
| Taille du lot    | >0               | Integer | 16      | Nombre d’instances à entraîner dans un lot   |
| Taux d’apprentissage | >=double.Epsilon | Float   | 0.001   | Taux d’apprentissage initial pour l’optimiseur de descente de gradient stochastique. |
| Valeur initiale aléatoire   | Quelconque              | Integer | 1       | Valeur initiale pour le générateur de nombres aléatoires utilisé par le modèle. |
| Patience      | >0               | Integer | 3       | Nombre d’époques où l’entraînement doit être arrêté de manière anticipée   |

###  <a name="outputs"></a>Sorties  

| Nom          | Type           | Description   |
| ------------- | -------------- | ------------- |
| Modèle entraîné | ModelDirectory | Modèle entraîné |

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 



