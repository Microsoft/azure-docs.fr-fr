---
title: 'Forêt d’arbres de décision multiclasse : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Forêt d’arbres de décision multiclasse dans Azure Machine Learning pour créer un modèle Machine Learning basé sur l’algorithme de *forêt d’arbres de décision*.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 04/22/2020
ms.openlocfilehash: 1be66bdd8a1cf25a32ad3102d770078c904c4b6c
ms.sourcegitcommit: 051908e18ce42b3b5d09822f8cfcac094e1f93c2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94376823"
---
# <a name="multiclass-decision-forest-module"></a>Module Forêt d’arbres de décision multiclasse

Cet article décrit un module dans le concepteur Azure Machine Learning.

Ce module vous permet de créer un modèle Machine Learning basé sur l’algorithme de *forêt d’arbres de décision*. Une forêt d’arbres de décision est un modèle d’ensemble qui génère rapidement une série d’arbres de décision, tout en apprenant grâce à des données avec balises.

## <a name="more-about-decision-forests"></a>En savoir plus sur la forêt d’arbres de décision

L’algorithme de forêt d’arbres de décision est une méthode d’apprentissage d’ensemble pour la classification. Pour fonctionner, l’algorithme crée plusieurs arbres de décision, puis il *vote* pour sélectionner la classe de sortie la plus populaire. Le vote est une forme d’agrégation dans laquelle chaque arbre d’une forêt d’arbres de décision génère un histogramme de fréquence non normalisé pour les étiquettes. Le processus d’agrégation additionne ces histogrammes et normalise le résultat pour obtenir les « probabilités » de chaque étiquette. Les arbres présentant un niveau de confiance élevé en matière de prédiction influent davantage sur la décision finale de l’ensemble.

En général, les arbres de décision sont des modèles non paramétriques, ce qui signifie qu’ils prennent en charge les données avec des distributions variées. Dans chaque arbre, une séquence de tests simples est exécutée pour chaque classe, augmentant les niveaux d’une arborescente jusqu’à atteindre un nœud terminal (une décision).

Les arbres de décision présentent plusieurs avantages :

+ Ils peuvent représenter des limites de décisions non linéaires.
+ Ils sont efficaces en matière de calcul et d’utilisation de la mémoire lors de la formation et de la prédiction.
+ Ils sélectionnent et classifient les fonctionnalités intégrées.
+ Ils sont résilients en cas de fonctionnalités bruyantes.

Le classifieur de forêt d’arbres de décision dans Azure Machine Learning est un ensemble d’arbres de décision. En général, les modèles ensemblistes offrent une meilleure couverture et une précision plus élevée que les arbres de décision uniques. Pour plus d’informations, consultez [Arbres de décision](https://go.microsoft.com/fwlink/?LinkId=403677).

## <a name="how-to-configure-multiclass-decision-forest"></a>Configuration du module Forêt d’arbres de décision multiclasse

1. Ajoutez le module **Forêt d’arbres de décision multiclasse** à votre pipeline dans le concepteur. Vous pouvez trouver ce module sous **Machine Learning**, **Initialiser le modèle** et **Classification**.

2. Double-cliquez sur le module pour ouvrir le volet **Propriétés**.

3. Pour **Méthode de rééchantillonnage**, choisissez la méthode utilisée pour créer les arbres individuels :  Vous pouvez choisir l’ensachage ou la réplication.

    + **Bagging** : le bagging, également appelé *agrégation boostrap*, est une méthode selon laquelle chaque arbre est développé sur un nouvel échantillon, créé en échantillonnant le jeu de données d’origine de manière aléatoire à l’aide d’un ensemble de remplacement jusqu’à obtenir un jeu de données de taille identique à l’original. Les sorties des modèles sont combinées grâce à un *vote* qui est une forme d’agrégation. Pour plus d’informations, voir l’entrée Wikipedia Bootstrap aggregating (en anglais).

    + **Réplication** : avec la réplication, l’apprentissage de chacun des arbres est effectué sur les mêmes données d’entrée. Le prédicat fractionné utilisé pour chaque nœud d’arbre est toujours déterminé de manière aléatoire, ce qui crée des arbres variés.

   

4. Spécifiez le mode d’apprentissage du modèle en définissant l’option **Créer un mode d’apprentissage**.

    + **Single Parameter** (Paramètre unique) : sélectionnez cette option si vous savez comment vous voulez configurer le modèle et vous fournissez un ensemble spécifique de valeurs en tant qu’arguments.

    + **Plage de paramètres** : Sélectionnez cette option si vous n’êtes pas sûr des paramètres à choisir et que vous souhaitez exécuter un balayage des paramètres. Sélectionnez une plage de valeurs à itérer, et le module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md) itère toutes les combinaisons possibles des paramètres que vous avez fournis pour déterminer les hyperparamètres qui produisent les résultats optimaux.   

5. **Nombre d’arbres de décision** : saisissez le nombre maximal d’arbres de décision qui peuvent être créés dans l’ensemble. En créant plusieurs arbres de décision, vous pouvez obtenir une meilleure couverture, mais cette opération peut augmenter la durée de la formation.

    Toutefois, si vous définissez la valeur sur 1, cela signifie qu’un seul arbre peut être généré (celui qui inclut le jeu de paramètres initial) et qu’aucune autre itération supplémentaire n’est effectuée.

6. **Profondeur maximale des arbres de décision** : saisissez un nombre destiné à limiter la profondeur maximale de n’importe quel arbre de décision. Le fait d’augmenter la profondeur de l’arbre est susceptible d’améliorer la précision, au prix d’un risque de surajustement et d’augmentation du temps d’apprentissage.

7. **Nombre de fractionnements aléatoires par nœud** : saisissez le nombre de fractionnements à utiliser lors de la génération de chaque nœud de l’arbre. Le *fractionnement* consiste à diviser de manière aléatoire les fonctionnalités à chaque niveau de l’arbre (nœud).

8. **Minimum number of samples per leaf node** (Nombre minimal d’échantillons par nœud terminal) : indiquez le nombre minimal de cas requis pour créer un nœud terminal (feuille) dans un arbre. Plus cette valeur est grande, plus le seuil de création de règles augmente.

    Par exemple, la valeur par défaut de 1, un seul cas suffit à entraîner la création d’une règle. Si la valeur passe à 5, les données d’apprentissage doivent contenir au moins cinq cas remplissant les mêmes conditions.



10. Connectez un jeu de données étiqueté et entraînez le modèle :

    + Si vous définissez **Create trainer mode** (Créer un mode d’apprentissage) sur **Single Parameter** (Paramètre unique), connectez un jeu de données balisé au module [Entraîner le modèle](train-model.md).  
  
    + Si vous définissez **Créer un mode d’entraînement** sur **Plage de paramètres**, connectez un jeu de données avec balises et entraînez le modèle en utilisant [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md).  
  
    > [!NOTE]
    > 
    > Si vous transmettez une plage de paramètres à [Entraîner le modèle](train-model.md), elle utilise uniquement la valeur par défaut dans la liste de paramètres unique.  
    > 
    > Si vous transmettez un ensemble unique de valeurs de paramètre au module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md), quand il attend une plage de paramètres pour chaque paramètre, il ignore les valeurs et utilise les valeurs par défaut pour l’apprenant.  
    > 
    > Si vous sélectionnez l’option **Plage de paramètres** et que vous entrez une valeur unique pour un paramètre, cette valeur unique que vous avez spécifiée est utilisée tout au long du balayage, même si d’autres paramètres changent sur une plage de valeurs.

11. Envoyez le pipeline.



## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 