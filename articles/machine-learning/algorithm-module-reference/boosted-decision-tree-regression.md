---
title: 'Régression d’arbre de décision boosté : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Régression d’arbre de décision boosté dans Azure Machine Learning pour créer un ensemble d’arbres de régression à l’aide du boosting.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 08/24/2020
ms.openlocfilehash: 664943fc5535883b3df77b2795383e5c0586a71c
ms.sourcegitcommit: 051908e18ce42b3b5d09822f8cfcac094e1f93c2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94375327"
---
# <a name="boosted-decision-tree-regression-module"></a>Module de régression d’arbre de décision boosté

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez ce module pour créer un ensemble d’arbres de régression à l’aide du boosting. Le terme *boosting* signifie que chaque arbre dépend des arbres précédents. L’algorithme apprend en ajustant les valeurs résiduelles des arbres précédents. Par conséquent, le boosting d’un arbre de décision a tendance à améliorer la précision avec un léger risque de diminution de la couverture.  

Ce module est basé sur l’algorithme LightGBM.
  
Cette méthode de régression est une méthode d’apprentissage supervisé et nécessite donc un *jeu de données étiqueté*. La colonne de libellés doit contenir des valeurs numériques.  

> [!NOTE]
> Utilisez ce module uniquement avec les jeux de données qui contiennent des variables numériques.  

Une fois que vous avez défini le modèle, effectuez l’apprentissage à l’aide du module [Train model (Entraîner le modèle)](./train-model.md).

  
## <a name="more-about-boosted-regression-trees"></a>Plus d’informations sur les arbres de régression boostés  

Le boosting est une méthode classique de création de modèles d’ensemble, de même que le bagging, les forêts aléatoires, etc.  Dans Azure Machine Learning, les arbres de décision boostés utilisent une implémentation efficace de l’algorithme de boosting de gradient MART. Le boosting de gradient est une technique d’apprentissage automatique pour les problèmes de régression. Il génère chaque arbre de régression par étapes, à l’aide d’une fonction de perte prédéfinie pour mesurer l’erreur à chaque étape et la corriger à la suivante. Par conséquent, le modèle de prédiction est en fait un ensemble de modèles de prédiction plus faibles.  
  
Dans les problèmes de régression, le boosting génère une série d’arbres par étapes, puis sélectionne l’arbre optimal à l’aide d’une fonction de perte dérivable arbitraire.  
  
Pour plus d’informations, consultez les articles suivants :  
  
+ [https://wikipedia.org/wiki/Gradient_boosting#Gradient_tree_boosting](https://wikipedia.org/wiki/Gradient_boosting)

    Cet article de Wikipedia sur le boosting de gradient fournit des informations générales sur les arbres boostés. 
  
-  [https://research.microsoft.com/apps/pubs/default.aspx?id=132652](https://research.microsoft.com/apps/pubs/default.aspx?id=132652)  

    Microsoft Research : From RankNet to LambdaRank to LambdaMART: An Overview (De RankNet à LambdaRank et LambdaMART : présentation). De J.C. Burges.

La méthode de boosting de gradient peut également servir pour les problèmes de classification, en les réduisant jusqu’à régression avec une fonction de perte appropriée. Pour plus d’informations sur l’implémentation des arbres boostés pour les tâches de classification, voir [Arbre de décision boosté à deux classes](./two-class-boosted-decision-tree.md).  

## <a name="how-to-configure-boosted-decision-tree-regression"></a>Comment configurer une régression d’arbre de décision boosté

1.  Ajoutez le module **Arbre de décision optimisé** à votre pipeline. Vous le trouverez sous **Machine Learning**, **Initialiser**, dans la catégorie **Régression**. 
  
2.  Spécifiez le mode d’apprentissage du modèle en définissant l’option **Créer un mode d’apprentissage**.  
  
    -   **Single Parameter** (Paramètre unique) : sélectionnez cette option si vous savez comment vous voulez configurer le modèle, et fournissez un ensemble spécifique de valeurs en tant qu’arguments. 
     
    -   **Plage de paramètres** : Sélectionnez cette option si vous n’êtes pas sûr des paramètres à choisir et que vous souhaitez exécuter un balayage des paramètres. Sélectionnez la plage de valeurs sur laquelle vous souhaitez effectuer l’itération. Le module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md) effectue alors une itération sur toutes les combinaisons possibles des paramètres que vous avez fournis pour déterminer les hyperparamètres qui produisent les résultats optimaux.    
   
  
3. **Maximum number of leaves per tree** (Nombre maximal de feuilles par arbre) : indiquer le nombre maximal de nœuds terminaux (feuilles) qui peuvent être créés dans n’importe quel arbre.  

    En augmentant cette valeur, vous augmentez potentiellement la taille de l’arbre et vous bénéficiez d’un surcroît de précision, au risque d’un surajustement et d’un temps d’apprentissage plus long.  

4. **Minimum number of samples per leaf node** (Nombre minimal d’échantillons par nœud terminal) : indiquez le nombre minimal de cas requis pour créer un nœud terminal (feuille) dans un arbre.

    Plus cette valeur est grande, plus le seuil de création de règles augmente. Par exemple, la valeur par défaut de 1, un seul cas suffit à entraîner la création d’une règle. Si vous définissez la valeur sur 5, les données d’apprentissage doivent contenir au moins 5 cas respectant les mêmes conditions.

5. **Taux d’apprentissage** : saisissez un nombre compris entre 0 et 1 qui définit la taille d’étape lors de l’apprentissage. Le taux d’apprentissage détermine la vitesse à laquelle l’apprentissage converge vers la solution optimale. Si la taille d’étape est trop grande, vous risquez de passer à côté de la solution optimale. Si la taille de pas est trop faible, l’apprentissage nécessite plus de temps pour converger vers la meilleure solution.

6. **Number of trees constructed** (Nombre d’arbres construits) : indiquez le nombre total d’arbres de décision à créer dans l’ensemble. Plus il y en a, meilleure peut être la couverture, mais plus le temps d’apprentissage augmente.

    Toutefois, si vous définissez la valeur sur 1, un seul arbre est généré (l’arbre avec le jeu de paramètres initial), et aucune autre itération n’est effectuée.

7. **Random number seed** (Valeur de départ aléatoire) : saisissez un entier non négatif facultatif à utiliser comme la valeur de départ aléatoire. La spécification d’une valeur de départ garantit la reproductibilité entre les exécutions qui présentent les mêmes données et paramètres.

    Par défaut, la valeur de départ aléatoire est définie sur 0, ce qui signifie qu’elle est obtenue à partir de l’horloge système.
  

9. Entraînez le modèle :

    + Si vous définissez **Create trainer mode** (Créer un mode d’apprentissage) sur **Single Parameter** (Paramètre unique), connectez un jeu de données balisé au module [Entraîner le modèle](train-model.md).  
  
    + Si vous définissez **Créer un mode d’entraînement** sur **Plage de paramètres**, connectez un jeu de données avec balises et entraînez le modèle en utilisant [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md).  
  
    > [!NOTE]
    > 
    > Si vous transmettez une plage de paramètres à [Entraîner le modèle](train-model.md), elle utilise uniquement la valeur par défaut dans la liste de paramètres unique.  
    > 
    > Si vous transmettez un ensemble unique de valeurs de paramètre au module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md), quand il attend une plage de paramètres pour chaque paramètre, il ignore les valeurs et utilise les valeurs par défaut pour l’apprenant.  
    > 
    > Si vous sélectionnez l’option **Plage de paramètres** et que vous entrez une valeur unique pour un paramètre, cette valeur unique que vous avez spécifiée est utilisée tout au long du balayage, même si d’autres paramètres changent sur une plage de valeurs.
    

10. Envoyez le pipeline.  
  
## <a name="results"></a>Résultats

Une fois l’apprentissage terminé :

+ Si vous souhaitez utiliser le modèle à des fins de scoring, connectez le module [Train Model](train-model.md) (Entraîner un modèle) au module [Score Model](./score-model.md) (Effectuer le scoring d’un modèle) pour prédire les valeurs des nouveaux exemples d’entrée.

+ Pour enregistrer un instantané du modèle entraîné, sélectionnez l’onglet **Sorties** dans le panneau droit du **modèle entraîné** et cliquez sur l’icône **Inscrire le jeu de données**. La copie du modèle entraîné est enregistrée en tant que module dans l’arborescence du module et ne sera pas mise à jour lors des exécutions successives du pipeline.

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 
