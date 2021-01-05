---
title: 'Régression de Poisson : sur le module Modifier les métadonnées'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Régression de Poisson dans le concepteur Azure Machine Learning pour créer un modèle de régression de Poisson.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 07/13/2020
ms.openlocfilehash: 2dfd8b3d919f9eeb3e183135ef543f417c878977
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93420697"
---
# <a name="poisson-regression"></a>Régression de Poisson

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez ce module pour créer un modèle de régression de Poisson dans un pipeline. La régression de Poisson est destinée à prédire des valeurs numériques, en général des nombres. Par conséquent, vous devez utiliser ce module pour créer votre modèle de régression uniquement si les valeurs que vous essayez de prédire répondent aux conditions suivantes :

- La variable de la réponse suit une [distribution de Poisson](https://en.wikipedia.org/wiki/Poisson_distribution).  

- Les nombres ne peuvent pas être négatifs. La méthode échoue totalement si vous essayez de l'utiliser avec des étiquettes négatives.

- Une distribution de Poisson étant une distribution discrète, il n'y a donc pas lieu d'utiliser cette méthode avec des nombres non entiers.

> [!TIP]
> Si votre cible n'est pas un nombre, la régression de Poisson n'est probablement pas une méthode appropriée. Essayez [d’autres modules de régression dans le concepteur](./module-reference.md#machine-learning-algorithms). 

Une fois que vous avez configuré la méthode de régression, vous devez effectuer l’apprentissage du modèle à l’aide d’un jeu de données contenant des exemples de la valeur que vous souhaitez prédire. Le modèle ainsi formé permet ensuite d’effectuer des prédictions.

## <a name="more-about-poisson-regression"></a>En savoir plus sur la régression de Poisson

La régression de Poisson est un type d'analyse de régression généralement utilisée pour les nombres de modèles. Par exemple, la régression de Poisson est utile dans les cas suivants :

- Modélisation du nombre de rhinites associées aux vols en avion

- Estimation du nombre d’appels de service d’urgence pendant un événement

- Projection du nombre de demandes de clients après une promotion

- Création de tables d'urgence

Étant donné que la variable de réponse a une distribution de Poisson, le modèle fait différentes hypothèses concernant les données et sa distribution de probabilités, par exemple, la régression des moindres carrés. Par conséquent, les modèles de Poisson doivent être interprétés différemment des autres modèles de régression.

## <a name="how-to-configure-poisson-regression"></a>Comment configurer la régression de Poisson

1. Ajoutez le module **Régression de poisson** à votre pipeline dans le Concepteur. Vous le trouverez sous **Algorithmes de Machine Learning**, dans la catégorie **Régression**.

2. Ajoutez un jeu de données qui contient les données d’apprentissage du bon type. 

    Il est recommandé d'utiliser [Normaliser les données](normalize-data.md) pour normaliser le jeu de données d'entrée avant de l'utiliser pour former le régresseur.

3. Dans le volet droit du module **Régression de Poisson**, spécifiez la façon dont vous souhaitez que le modèle soit formé, en définissant l’option **Créer le mode de formateur**.  
  
    - **Single Parameter** (Paramètre unique) : si vous savez comment vous voulez configurer le modèle, fournissez un ensemble spécifique de valeurs en tant qu’arguments.
  
    - **Plage de paramètres** : en cas de doute sur les paramètres à utiliser, changez les paramètres à l’aide du module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md). L’instructeur effectue une itération sur plusieurs valeurs que vous spécifiez pour trouver la configuration optimale.
  
4. **Tolérance d'optimisation** : Tapez une valeur qui définit l'intervalle de tolérance lors de l'optimisation. Plus la valeur est faible, plus l'ajustement est lent et précis.

5. **L1 regularization weight** (Pondération de régularisation L1) et **L2 regularization weight** (L2 regularization weight) : Tapez les valeurs à utiliser pour la régularisation L1 et L2. La *régularisation* ajoute des contraintes à l'algorithme en ce qui concerne les aspects du modèle qui sont indépendants des données d'apprentissage. La régularisation permet également d'éviter le surajustement. 

    - La régularisation L1 est utile si l'objectif est de disposer d'un modèle aussi épars que possible.

        La régularisation L1 est effectuée en soustrayant le poids L1 du vecteur de poids de l'expression de perte que l'apprenant essaie de minimiser. La norme L1 est une bonne approximation de la norme L0, qui correspond au nombre de coordonnées différentes de zéro.

    - La régularisation L2 empêche que les coordonnées uniques du vecteur de poids ne croissent trop en grandeur. La régularisation L2 est utile si l'objectif est de disposer d'un modèle avec des poids globaux faibles.

    Dans ce module, vous pouvez appliquer une combinaison de régularisations L1 et L2. En associant les régularisations L1 et L2, vous pouvez imposer une pénalité sur l'amplitude des valeurs de paramètre. L'apprenant essaie de réduire la pénalité, en faisant un compromis avec la réduction de la perte.

    Pour une bonne présentation des régularisations L1 et L2, consultez [Régularisations L1 et L2 pour l’apprentissage automatique](/archive/msdn-magazine/2015/february/test-run-l1-and-l2-regularization-for-machine-learning).

6. **Taille de la mémoire pour L-BFGS** : Spécifiez la quantité de mémoire à réserver pour l’ajustement et l’optimisation du modèle.

     L-BFGS est une méthode spécifique pour l’optimisation, basée sur l’algorithme Broyden-Fletcher – Goldfarb – Shanno (BFGS). La méthode utilise une quantité limitée de mémoire (L) pour calculer la direction de l’étape suivante.

     En modifiant ce paramètre, vous pouvez affecter le nombre de positions passées et de gradients pour stocker le calcul de l'étape suivante.

7. Connectez le jeu de données d’apprentissage et le modèle non formé à l’un des modules d’apprentissage : 

    - Si vous définissez **Créer un mode d’apprentissage** sur **Paramètre unique**, utilisez le module [Entraîner le du modèle](train-model.md).

    - Si vous définissez **Créer mode de formateur** sur **Plage de paramètres**, utilisez le module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md).

    > [!WARNING]
    > 
    > - Si vous transmettez une plage de paramètres au module [Entraîner le modèle](train-model.md), il utilise uniquement la première valeur dans la liste de plages de paramètres.
    > 
    > - Si vous transmettez un ensemble unique de valeurs de paramètre au module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md), quand il attend une plage de paramètres pour chaque paramètre, il ignore les valeurs et utilise les valeurs par défaut pour l’apprenant.
    > 
    > - Si vous sélectionnez l’option **Plage de paramètres** et que vous entrez une valeur unique pour un paramètre, cette valeur unique que vous avez spécifiée est utilisée tout au long du balayage, même si d’autres paramètres changent sur une plage de valeurs.

8.  Envoyez le pipeline.

## <a name="results"></a>Résultats

Une fois l’apprentissage terminé :

+ Pour enregistrer un instantané du modèle entraîné, sélectionnez le module d’entraînement, puis basculez vers l’onglet **Sorties + journaux** dans le panneau droit. Cliquez sur l’icône **Inscrire le jeu de données**.  Vous pouvez trouver le modèle enregistré sous la forme d’un module dans l’arborescence du module. 

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning.