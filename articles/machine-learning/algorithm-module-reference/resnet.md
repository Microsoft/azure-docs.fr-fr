---
title: ResNet
titleSuffix: Azure Machine Learning
description: Découvrez comment créer un modèle de classification d’images dans le concepteur Azure Machine Learning à l’aide de l’algorithme ResNet.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 09/26/2020
ms.openlocfilehash: 88a820d0f1fa9515b4f2992a8305a2d1065e0987
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421207"
---
# <a name="resnet"></a>ResNet

Cet article explique comment utiliser le module **ResNet** dans le concepteur Azure Machine Learning pour créer un modèle de classification d’images à l’aide de l’algorithme ResNet.  

Cet algorithme de classification est une méthode d’apprentissage supervisé qui nécessite un jeu de données étiqueté. 
> [!NOTE]
> Ce module ne prend pas en charge le jeu de données étiqueté généré à partir d’*Étiquetage des données* dans le studio ; il ne prend en charge que le répertoire d’images étiqueté généré à partir du module [Convertir en répertoire d’images](convert-to-image-directory.md). 

Pour entraîner un modèle, indiquez ce dernier ainsi qu’un répertoire d’images étiquetées en tant qu’entrées du module [Train Pytorch Model](train-pytorch-model.md). Vous pouvez ensuite utiliser le modèle entraîné pour prédire les valeurs des nouveaux exemples d’entrée à l’aide du module [Score Image Model](score-image-model.md).

### <a name="more-about-resnet"></a>En savoir plus sur ResNet

Pour plus d’informations, consultez [cet article](https://pytorch.org/docs/stable/torchvision/models.html?highlight=resnext101_32x8d#torchvision.models.resnext101_32x8d) sur ResNet.

## <a name="how-to-configure-resnet"></a>Guide pratique pour configurer ResNet

1.  Ajoutez le module **ResNet** à votre pipeline dans le concepteur.  

2.  Pour **Model name** (Nom du modèle), indiquez le nom d’une structure ResNet spécifique que vous pouvez sélectionner parmi les structures resnet prises en charge : « resnet18 », « resnet34 », « resnet50 », « resnet101 », « resnet152 », « resnext50\_32x4d », « resnext101\_32x8d », « wide_resnet50\_2 », « wide_resnet101\_2 ».

3.  Pour **Pretrained** (Préentraîné), indiquez si vous souhaitez utiliser un modèle préentraîné sur ImageNet. Si cette option est sélectionnée, vous pouvez affiner le modèle en fonction du modèle préentraîné sélectionné. Si cette option est désélectionnée, vous pouvez commencer l’entraînement à partir de zéro.

4.  Connectez la sortie du module **DenseNet**, du module de jeu de données d’images d’entraînement et de validation au module [Train Pytorch Model](train-pytorch-model.md). 

5. Envoyez le pipeline.

## <a name="results"></a>Résultats

Une fois l’exécution du pipeline effectuée, si vous souhaitez utiliser le modèle à des fins de scoring, connectez le module [Train Pytorch Model](train-pytorch-model.md) (Entraîner un modèle PyTorch) au module [Score Image Model](score-image-model.md) (Scorer un modèle d’image) pour prédire les valeurs des nouveaux exemples d’entrée.

## <a name="technical-notes"></a>Notes techniques  

###  <a name="module-parameters"></a>Paramètres du module  

| Nom       | Plage | Type    | Default           | Description                              |
| ---------- | ----- | ------- | ----------------- | ---------------------------------------- |
| Nom du modèle | Quelconque   | Mode    | resnext101\_32x8d | Nom d’une structure ResNet spécifique       |
| Pretrained (Préentraîné) | Quelconque   | Boolean | True              | Spécifie si un modèle préentraîné doit être utilisé ou non sur ImageNet |
|            |       |         |                   |                                          |

###  <a name="output"></a>Output  

| Nom            | Type                    | Description                              |
| --------------- | ----------------------- | ---------------------------------------- |
| Untrained model (Modèle non entraîné) | UntrainedModelDirectory | Modèle ResNet non entraîné qui peut être connecté au module Train Pytorch Model. |

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 