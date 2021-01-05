---
title: One-vs-All Multiclass
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module One-vs-All Multiclass dans le concepteur Azure Machine Learning pour créer un ensemble de modèles de classification binaire.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 11/13/2020
ms.openlocfilehash: 4dfe284a00052cbd1915d62355e1d7772f3712ab
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94591867"
---
# <a name="one-vs-all-multiclass"></a>One-vs-All Multiclass

Cet article décrit comment utiliser le module One-vs-All Multiclass dans le concepteur Azure Machine Learning. L’objectif est de créer un modèle de classification capable de prédire plusieurs classes, en utilisant l’approche *One-versus-All* (un contre tous).

Ce module est utile pour créer des modèles qui prédisent trois résultats possibles, lorsque le résultat dépend de variables de prédiction continues ou catégoriques. Cette méthode vous permet également d’utiliser des méthodes de classification binaire pour les problèmes qui requièrent plusieurs classes de sortie.

### <a name="more-about-one-versus-all-models"></a>En savoir plus sur les modèles One-versus-All

Certains algorithmes de classification permettent d’utiliser plus de deux classes par conception. D’autres limitent les résultats possibles à l’une des deux valeurs (modèle binaire ou à deux classes). Toutefois, même les algorithmes de classification binaire peuvent être adaptés aux tâches de classification multiclasse via diverses stratégies. 

Ce module implémente la méthode One-versus-All, dans laquelle un modèle binaire est créé pour chacune des multiples classes de sortie. Le module évalue chacun de ces modèles binaires pour les classes individuelles par rapport à son complément (toutes les autres classes du modèle) comme s’il s’agissait d’un problème de classification binaire. Outre son efficacité de calcul (seuls les classifieurs `n_classes` sont nécessaires), l’un des avantages de cette approche est son interprétabilité. Étant donné que chaque classe est représentée par un seul et unique classifieur, il est possible d’obtenir des informations sur la classe en inspectant son classifieur correspondant. Il s’agit de la stratégie la plus couramment utilisée pour la classification multiclasse. C’est un choix par défaut équitable. Le module effectue ensuite une prédiction en exécutant ces classifieurs binaires et en choisissant la prédiction dotée du plus haut score de confiance. 

En résumé, le module crée un ensemble de modèles individuels et fusionne ensuite les résultats afin de créer un modèle unique qui prédit toutes les classes. Tout classifieur binaire peut être utilisé comme base pour un modèle One-versus-All.  

Par exemple, supposons que vous configurez un modèle de [Machine à vecteurs de support à deux classes](two-class-support-vector-machine.md) et que vous le fournissez en entrée dans le module One-vs-All Multiclass. Le module crée des modèles de machine à vecteurs de support à deux classes pour tous les membres de la classe de sortie. Il applique ensuite la méthode One-versus-All pour combiner les résultats de toutes les classes.  

Le module utilise OneVsRestClassifier de sklearn. Pour plus d’informations, cliquez [ici](https://scikit-learn.org/stable/modules/generated/sklearn.multiclass.OneVsRestClassifier.html).

## <a name="how-to-configure-the-one-vs-all-multiclass-classifier"></a>Comment configurer le classifieur One-vs-All Multiclass  

Ce module crée un ensemble de modèles de classification binaire pour analyser plusieurs classes. Pour utiliser ce module, vous devez tout d’abord configurer et entraîner un modèle de *classification binaire*. 

Vous connectez le modèle binaire au module One-vs-All Multiclass. Ensuite, vous entraînez l’ensemble des modèles en utilisant [Entraîner le modèle](train-model.md) avec un jeu de données d’entraînement étiqueté.

Lorsque vous combinez les modèles, le module One-vs-All Multiclass crée plusieurs modèles de classification binaire, optimise l’algorithme pour chaque classe, puis fusionne les modèles. Le module effectue ces tâches même si le jeu de données d’entraînement peut avoir plusieurs valeurs de classe.

1. Ajoutez le module One-vs-All Multiclass à votre pipeline dans le concepteur. Vous pouvez trouver ce module sous **Machine Learning - Initialiser**, dans la catégorie **Classification**.

   Le classifieur One-vs-All Multiclass ne comprend pas de paramètres configurables. Toutes les personnalisations doivent être effectuées dans le modèle de classification binaire fourni comme entrée.

2. Ajoutez un modèle de classification binaire au pipeline et configurez ce modèle. Par exemple, vous pouvez utiliser [Machine à vecteurs de support à deux classes](two-class-support-vector-machine.md) ou [Arbre de décision optimisé à deux classes](two-class-boosted-decision-tree.md).

3. Ajoutez le module [Entraîner le modèle](train-model.md) à votre pipeline. Connectez le classifieur non entraîné qui correspond à la sortie du module One-vs-All Multiclass.

4. Sur l’autre entrée du module [Entraîner le modèle](train-model.md), connectez un jeu de données d’entraînement étiqueté qui comprend plusieurs valeurs de classe.

5. Envoyez le pipeline.

## <a name="results"></a>Résultats

Une fois l’entraînement terminé, vous pouvez utiliser le modèle pour effectuer des prédictions multiclasses.

Vous pouvez également transmettre le classifieur non entraîné au module [Modèle de validation croisée](cross-validate-model.md) pour une validation croisée par rapport à un jeu de données de validation étiqueté.


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 
