---
title: Diviser un répertoire d’images
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Diviser un répertoire d’images dans le concepteur pour diviser les images d’un répertoire d’images en deux jeux distincts.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 09/26/2020
ms.openlocfilehash: 3ee4dd9b2e344ecb3e1a6424ce7310270e7cd076
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421190"
---
# <a name="split-image-directory"></a>Diviser un répertoire d’images

Cette rubrique explique comment utiliser le module Diviser un répertoire d’images dans Azure Machine Learning Designer pour diviser les images d’un répertoire d’images en deux jeux distincts.

Ce module est particulièrement utile quand vous devez séparer des données d’image en jeux de formation et de test. 

## <a name="how-to-configure-split-image-directory"></a>Comment configurer le module Diviser un répertoire d’images

1. Ajoutez le module **Diviser un répertoire d’images** à votre pipeline. Vous trouverez ce module dans la catégorie 'Computer Vision/Image Data Transformation' (Vision par ordinateur/Transformation de données d’image).

2. Connectez-le au module dont la sortie est le répertoire d’images.

3. Entrez **Fraction d’images dans la première sortie** pour spécifier le pourcentage de données à placer dans le fractionnement de gauche, par défaut 0.9. Si le résultat de la fraction n’est pas un entier, le module utilise le plus petit entier proche.


## <a name="technical-notes"></a>Notes techniques

### <a name="expected-inputs"></a>Entrées attendues

| Nom                  | Type           | Description              |
| --------------------- | -------------- | ------------------------ |
| Répertoire d’images d’entrée | ImageDirectory | Répertoire d’images à fractionner |

### <a name="module-parameters"></a>Paramètres du module

| Nom                                   | Type  | Plage | Facultatif | Description                            | Default |
| -------------------------------------- | ----- | ----- | -------- | -------------------------------------- | ------- |
| Fraction d’images dans la première sortie | Float | 0-1   | Obligatoire | Fraction d’images dans la première sortie | 0.9     |

### <a name="outputs"></a>Sorties

| Nom                    | Type           | Description                              |
| ----------------------- | -------------- | ---------------------------------------- |
| Répertoire d’images de sortie 1 | ImageDirectory | Répertoire d’images qui contient les images sélectionnées |
| Répertoire d’images de sortie 2 | ImageDirectory | Répertoire d’images qui contient toutes les autres images |

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 

