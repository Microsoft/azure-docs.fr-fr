---
title: 'Forêt d’arbres décisionnels à deux classes : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Forêt d’arbres décisionnels à deux classes dans Azure Machine Learning pour créer un modèle Machine Learning basé sur l’algorithme de forêt d’arbres décisionnels.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 09/03/2020
ms.openlocfilehash: c4c303a7640454ba0cb6622b21fd161354266068
ms.sourcegitcommit: 051908e18ce42b3b5d09822f8cfcac094e1f93c2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94375923"
---
# <a name="two-class-decision-forest-module"></a>Module Forêt d’arbres décisionnels à deux classes

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez ce module pour créer un modèle Machine Learning basé sur l’algorithme de forêt d’arbres décisionnels.  

Les forêts d’arbres décisionnels sont des modèles ensemblistes rapides et supervisés. Ce module est une bonne solution pour prédire une cible présentant un maximum de deux résultats. 

## <a name="understanding-decision-forests"></a>Bien comprendre les forêts d’arbres décisionnels

L’algorithme de forêt d’arbres décisionnels est une méthode d’apprentissage ensembliste destinée à des tâches de classification. Les méthodes ensemblistes sont fondées sur le principe général suivant : plutôt que de s’appuyer sur un modèle unique, on peut obtenir de meilleurs résultats et un modèle plus généralisé en créant plusieurs modèles liés et en les combinant d’une certaine façon. En général, les modèles ensemblistes offrent une meilleure couverture et une précision plus élevée que les arbres de décision uniques. 

Il existe de nombreuses façons de créer des modèles individuels et de les combiner pour former un ensemble. Cette implémentation particulière de la forêt d’arbres décisionnels consiste à créer plusieurs arbres de décision, puis à **voter** pour sélectionner la classe de sortie la plus populaire. Le vote est l’un des moyens les plus connus de générer des résultats dans un modèle ensembliste. 

+ De nombreux arbres de classification individuels sont créés, avec l’ensemble du jeu de données, mais différents points de départ (généralement randomisés). Cette approche est différente de celle de la forêt aléatoire, selon laquelle les arbres de décision individuels peuvent n’utiliser qu’une partie randomisée des données ou des fonctionnalités.
+ Chacun des arbres de la forêt d’arbres décisionnels donne en sortie un histogramme à fréquence non normalisée d’étiquettes. 
+ Le processus d’agrégation additionne ces histogrammes et normalise le résultat pour obtenir les « probabilités » de chaque étiquette. 
+ Les arbres présentant un niveau de confiance élevé en matière de prédiction pèsent davantage dans la décision finale de l’ensemble.

Les arbres de décision présentent en général de nombreux avantages pour les tâches de classification :
  
- Ils peuvent capturer des limites de décisions non linéaires.
- Il est possible d’effectuer un apprentissage et d’établir des prédictions sur de gros volumes de données, car ils sont efficaces dans l’utilisation du processeur et de la mémoire.
- La sélection de fonctionnalités est intégrée aux processus d’apprentissage et de classification.  
- Les arbres peuvent prendre en charge des données bruitées et de nombreuses fonctionnalités.  
- Ce sont des modèles non paramétriques, ce qui signifie qu’ils peuvent gérer des données présentant des distributions variées. 

Toutefois, les arbres de décision simples présentent un risque de surajustement par rapport aux données et sont moins généralisables que les ensembles d’arbres.

Pour plus d’informations, voir [Arbres de décision](https://go.microsoft.com/fwlink/?LinkId=403677).  

## <a name="how-to-configure"></a>Comment configurer
  
1.  Ajoutez le module **Forêt d’arbres décisionnels à deux classes** à votre pipeline dans Azure Machine Learning, puis ouvrez le volet **Propriétés** du module. 

    Vous trouverez le module sous **Machine Learning**. Développez **Initialiser**, puis **Classification**.  
  
2.  Pour **Méthode de rééchantillonnage**, choisissez la méthode utilisée pour créer les arbres individuels :  **Bagging** ou **Réplication**.  
  
    -   **Bagging** : le bagging, également appelé *agrégation boostrap*, est une méthode selon laquelle chaque arbre est développé sur un nouvel échantillon, créé en échantillonnant le jeu de données d’origine de manière aléatoire à l’aide d’un ensemble de remplacement jusqu’à obtenir un jeu de données de taille identique à l’original.  
  
         Les sorties des modèles sont combinées grâce à un *vote* qui est une forme d’agrégation. Chacun des arbres de la forêt d’arbres décisionnels de classification donne en sortie un histogramme à fréquence non normalisée d’étiquettes. L’agrégation consiste à additionner et à normaliser ces histogrammes pour obtenir les « probabilités » de chaque étiquette. Ainsi, les arbres présentant un niveau de confiance élevé en matière de prédiction pèseront davantage dans la décision finale de l’ensemble.  
  
         Pour plus d’informations, voir l’entrée Wikipedia Bootstrap aggregating (en anglais).  
  
    -   **Réplication** : avec la réplication, l’apprentissage de chacun des arbres est effectué sur les mêmes données d’entrée. Le prédicat fractionné utilisé pour chaque nœud d’arbre est toujours déterminé de manière aléatoire ; les arbres sont variés.   
  
3.  Spécifiez le mode d’apprentissage du modèle en définissant l’option **Créer un mode d’apprentissage**.  
  
    -   **Single Parameter** (Paramètre unique) : si vous savez comment vous voulez configurer le modèle, vous pouvez fournir un ensemble spécifique de valeurs comme arguments.

    -   **Plage de paramètres** : en cas de doute sur les paramètres à utiliser, utilisez le module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md). Vous fournissez une plage de valeurs, et le mode d'apprentissage effectue une itération sur plusieurs combinaisons de paramètres pour déterminer la combinaison de valeurs qui produit le meilleur résultat.
  
4.  Dans le champ **Nombre d’arbres de décision**, saisissez le nombre maximal d’arbres de décision qui peuvent être créés dans l’ensemble. Plus il y en a, meilleure peut être la couverture, mais plus le temps d’apprentissage augmente.  
  
    > [!NOTE]
    >  Si vous définissez la valeur sur 1. Toutefois, un seul arbre sera produit (l’arbre avec le jeu de paramètres initial) sans qu’aucune autre itération soit effectuée.
  
5.  Dans le champ **Profondeur maximale des arbres de décision**, tapez un nombre pour limiter la profondeur de chacun des arbres de décision. Le fait d’augmenter la profondeur de l’arbre est susceptible d’améliorer la précision, au prix d’un risque de surajustement et d’augmentation du temps d’apprentissage.
  
  
7.  Dans le champ **Nombre minimal d’échantillons par nœud terminal**, indiquez le nombre minimal de cas requis pour la création d’un nœud terminal dans un arbre.
  
     Plus cette valeur est grande, plus le seuil de création de règles augmente. Par exemple, la valeur par défaut de 1, un seul cas suffit à entraîner la création d’une règle. Si la valeur passe à 5, les données d’apprentissage doivent contenir au moins cinq cas remplissant les mêmes conditions.  
  
8.  Sélectionnez l’option **Autoriser les valeurs inconnues pour les fonctionnalités catégorielles** afin de créer un groupe pour les valeurs inconnues des jeux d’apprentissage ou de validation. Le modèle risque d’être moins précis pour les valeurs connues, mais il pourra fournir de meilleures prédictions pour les nouvelles valeurs (inconnues). 

     Si cette option est désélectionnée, le modèle n’accepte que les valeurs contenues dans les données d’apprentissage.
  
9. Attachez un jeu de données étiqueté et entraînez le modèle :

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