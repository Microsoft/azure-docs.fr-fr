---
title: 'Arbre de décision optimisé à deux classes : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Arbre de décision optimisé à deux classes dans le concepteur pour créer un classifieur binaire.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 08/24/2020
ms.openlocfilehash: 810878956a89b6e7400be99692cf43925d1cd134
ms.sourcegitcommit: 051908e18ce42b3b5d09822f8cfcac094e1f93c2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94375908"
---
# <a name="two-class-boosted-decision-tree-module"></a>Arbre de décision optimisé à deux classes

Cet article décrit un module dans le concepteur Azure Machine Learning.

Ce module vous permet de créer un modèle Machine Learning reposant sur l’algorithme d’arbres de décision optimisés. 

Un arbre de décision optimisé est une méthode d’apprentissage d’ensemble dans laquelle le second arbre corrige les erreurs du premier arbre, le troisième arbre corrige les erreurs du premier et du deuxième arbres, et ainsi de suite. Les prédictions sont effectuées par l’ensemble d’arbres complet.
  
En règle générale, lorsque les arbres de décision sont correctement configurés, ils représentent le moyen le plus simple d’obtenir des performances optimales sur un large éventail de tâches Machine Learning. Toutefois, ils constituent également l’un des apprentissages qui utilisent le plus de mémoire, et l’implémentation actuelle conserve toutes les données en mémoire. Par conséquent, un modèle d’arbre de décision optimisé n’est pas toujours en mesure de traiter des jeux de données volumineux qui peuvent être gérés par certains apprentissages linéaires.

Ce module est basé sur l’algorithme LightGBM.

## <a name="how-to-configure"></a>Comment configurer

Ce module crée un modèle de classification non entraîné. Étant donné que la classification est une méthode d’apprentissage supervisée, pour effectuer l’apprentissage du modèle, vous devez disposer d’un *jeu de données balisé* qui inclut une colonne d’étiquette avec une valeur pour toutes les lignes.

Vous pouvez effectuer l’apprentissage de ce type de modèle à l’aide de l’article [Entraîner le modèle](././train-model.md). 

1.  Dans Azure Machine Learning, ajoutez le module **Arbre de décision optimisé** à votre pipeline.
  
2.  Spécifiez le mode d’apprentissage du modèle en définissant l’option **Créer un mode d’apprentissage**.
  
    + **Single Parameter** (Paramètre unique) : si vous savez comment vous voulez configurer le modèle, vous pouvez fournir un ensemble spécifique de valeurs comme arguments.
  
    + **Plage de paramètres** : en cas de doute sur les paramètres à utiliser, utilisez le module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md). Vous fournissez une plage de valeurs, et le mode d'apprentissage effectue une itération sur plusieurs combinaisons de paramètres pour déterminer la combinaison de valeurs qui produit le meilleur résultat.
  
3.  Dans le champ **Maximum number of leaves per tree** (Nombre maximal de nœuds terminaux par arbre), indiquez le nombre maximal de nœuds terminaux qui peuvent être créés dans un arbre quelconque.
  
     En augmentant cette valeur, vous augmentez potentiellement la taille de l’arbre et vous bénéficiez d’un surcroît de précision, au risque d’un surajustement et d’un temps d’apprentissage plus long.
  
4.  Dans le champ **Minimum number of samples per leaf node** (Nombre minimal d’échantillons par nœud terminal), indiquez le nombre de cas requis pour la création d’un nœud terminal dans un arbre.  
  
     Plus cette valeur est grande, plus le seuil de création de règles augmente. Par exemple, la valeur par défaut de 1, un seul cas suffit à entraîner la création d’une règle. Si la valeur passe à 5, les données d’apprentissage doivent contenir au moins cinq cas remplissant les mêmes conditions.
  
5.  Dans le champ **Taux d’apprentissage**, saisissez un nombre compris entre 0 et 1 qui définit la taille de pas lors de l’apprentissage.  
  
     Le taux d’apprentissage détermine la vitesse à laquelle l’apprentissage converge vers la solution optimale. Si la taille de pas est trop élevée, vous risquez de passer à côté de la solution optimale. Si la taille de pas est trop faible, l’apprentissage nécessite plus de temps pour converger vers la meilleure solution.
  
6.  Dans le champ **Number of trees constructed** (Nombre d’arbres construits), indiquez le nombre total d’arbres de décision à créer dans l’ensemble. En créant plusieurs arbres de décision, vous pouvez obtenir une meilleure couverture, mais cette opération augmente la durée d’apprentissage.
  
     Si vous définissez la valeur sur 1, un seul arbre est généré (l’arbre avec le jeu de paramètres initial), et aucune autre itération n’est effectuée.
  
7.  Dans le champ **Random number seed** (Valeur de départ aléatoire), saisissez si vous le souhaitez un entier non négatif à utiliser comme valeur initiale aléatoire. La spécification d’une valeur de départ garantit la reproductibilité entre les exécutions qui présentent les mêmes données et paramètres.  
  
     La valeur de départ aléatoire est définie par défaut sur 0, ce qui signifie qu’elle est obtenue à partir de l’horloge système.  Les exécutions successives qui utilisent une valeur de départ aléatoire peuvent avoir des résultats différents.
  

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
   
## <a name="results"></a>Résultats

Une fois l’apprentissage terminé :

+ Pour enregistrer un instantané du modèle entraîné, sélectionnez l’onglet **Sorties** dans le panneau droit du module **Entraîner le modèle**. Sélectionnez l’icône **Inscrire le jeu de données** pour enregistrer le modèle en tant que module réutilisable.

+ Pour utiliser le modèle à des fins de scoring, ajoutez le module **Modèle de scoring** à un pipeline.

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 