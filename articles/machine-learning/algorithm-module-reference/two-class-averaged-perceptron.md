---
title: 'Perceptron moyenné à deux classes : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Perceptron avec moyenné à deux classes dans le concepteur pour créer un classifieur binaire.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 04/22/2020
ms.openlocfilehash: f1161dc99b84026bcc830c08b63b95d0bf3b0994
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421139"
---
# <a name="two-class-averaged-perceptron-module"></a>Module Perceptron moyenné à deux classes

Cet article décrit un module dans le concepteur Azure Machine Learning.

Ce module vous permet de créer un modèle Machine Learning reposant sur l’algorithme perceptron moyenné.  
  
Cet algorithme de classification constitue une méthode d’apprentissage supervisée et nécessite un *jeu de données avec balises*, qui inclut une colonne d’étiquette. Vous pouvez effectuer l’apprentissage du modèle en fournissant un jeu de données avec balises et le modèle en tant qu’entrée pour [Train Model](./train-model.md) (Entraîner le modèle). Le modèle formé peut ensuite être utilisé pour prédire des valeurs pour les nouveaux exemples d’entrées.  

### <a name="about-averaged-perceptron-models"></a>À propos des modèles Perceptron moyenné à deux classes

La *méthode Perceptron moyenné* est une version anticipée et simplifiée d’un réseau neuronal. Dans cette approche, les entrées sont classées en plusieurs sorties possibles selon une fonction linéaire, puis combinées avec un ensemble de poids dérivés du vecteur de fonctionnalité, d’où le nom « perceptron ».

Les modèles Perceptron les plus simples sont adaptés à l’apprentissage des modèles séparables de façon linéaire, tandis que les réseaux neuronaux (surtout les réseaux neuronaux profonds) peuvent modéliser des limites entre les classes plus complexes. Toutefois, les modules Perceptron sont plus rapides, et étant donné qu’ils traitent les cas en série, ils peuvent être utilisés avec une formation continue.

## <a name="how-to-configure-two-class-averaged-perceptron"></a>Comment configurer Perceptron moyenné à deux classes

1.  Ajoutez le module **Perceptron moyenné à deux classes** à votre pipeline.  

2.  Spécifiez le mode d’apprentissage du modèle en définissant l’option **Créer un mode d’apprentissage**.  
  
    -   **Single Parameter** (Paramètre unique) : si vous savez comment vous voulez configurer le modèle, fournissez un ensemble spécifique de valeurs en tant qu’arguments.

    -   **Plage de paramètres** : Sélectionnez cette option si vous n’êtes pas sûr des paramètres à choisir et que vous souhaitez exécuter un balayage des paramètres. Sélectionnez la plage de valeurs sur laquelle vous souhaitez effectuer l’itération. Le module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md) effectue alors une itération sur toutes les combinaisons possibles des paramètres que vous avez fournis pour déterminer les hyperparamètres qui produisent les résultats optimaux.  
  
3.  Pour **Taux d’apprentissage**, spécifiez une valeur pour le *taux d’apprentissage*. Les valeurs de taux d’apprentissage contrôlent la taille de l’étape utilisée par SGD (Stochastic Gradient Descent) chaque fois que le modèle est testé et corrigé.
  
     En réduisant ce taux, vous testez le modèle plus souvent, avec le risque de rester bloqué dans un plateau local. En l’augmentant, vous pouvez converger plus rapidement, au risque de dépasser les minima réels.
  
4.  Pour **Nombre maximal d’itérations**, entrez le nombre de fois où vous voulez que l’algorithme examine les données de formation.  
  
     L’arrêt précoce offre une meilleure généralisation, la plupart du temps. L’augmentation du nombre d’itérations améliore l’ajustement, mais présente un risque de surajustement.
  
5.  Pour **Valeur de départ numérique aléatoire**, vous pouvez éventuellement saisir un entier comme valeur initiale. L’utilisation d’un seed est recommandée si vous souhaitez garantir la reproductibilité du pipeline au cours des exécutions.  
  
1.  Connectez un jeu de données d’entraînement et entraînez le modèle :

    + Si vous définissez **Create trainer mode** (Créer un mode d’apprentissage) sur **Single Parameter** (Paramètre unique), connectez un jeu de données balisé au module [Entraîner le modèle](train-model.md).  
  
    + Si vous définissez **Créer un mode d’entraînement** sur **Plage de paramètres**, connectez un jeu de données avec balises et entraînez le modèle en utilisant [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md).  
  
    > [!NOTE]
    > 
    > Si vous transmettez une plage de paramètres à [Entraîner le modèle](train-model.md), elle utilise uniquement la valeur par défaut dans la liste de paramètres unique.  
    > 
    > Si vous transmettez un ensemble unique de valeurs de paramètre au module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md), quand il attend une plage de paramètres pour chaque paramètre, il ignore les valeurs et utilise les valeurs par défaut pour l’apprenant.  
    > 
    > Si vous sélectionnez l’option **Plage de paramètres** et que vous entrez une valeur unique pour un paramètre, cette valeur unique que vous avez spécifiée est utilisée tout au long du balayage, même si d’autres paramètres changent dans une plage de valeurs.




## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 