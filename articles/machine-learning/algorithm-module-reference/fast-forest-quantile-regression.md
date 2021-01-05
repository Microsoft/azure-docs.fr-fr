---
title: Régression quantile de forêt rapide : Référence de module
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Fast Forest Quantile Regression permet de créer un modèle de régression capable de prédire des valeurs pour un nombre spécifié de quantiles.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 07/13/2020
ms.openlocfilehash: 6d59d2fabb0b74ef12e33f55467b0ba68e0b7386
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90907953"
---
# <a name="fast-forest-quantile-regression"></a>Régression quantile de forêt rapide

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez ce module pour créer un modèle de régression de quantile de forêt rapide dans un pipeline. Une régression quantile de forêt rapide est utile si vous souhaitez mieux comprendre la distribution de la valeur prédite, au lieu d'obtenir une seule valeur de prédiction moyenne. Cette méthode a de nombreuses applications, notamment :  
  
- Prédiction de prix  
  
- Estimation des performances des étudiants, ou application de graphiques de croissance pour évaluer le développement des enfants  
  
- Découverte de relations prédictives dans les cas où il n'existe qu'une faible relation entre des variables  
  
Cet algorithme de régression est un une méthode d’apprentissage **supervisée**, ce qui signifie qu’il nécessite un jeu de données avec balises incluant une colonne d’étiquette. Étant donné qu’il s’agit d’un algorithme de régression, la colonne d’étiquette ne doit contenir que des valeurs numériques.

## <a name="more-about-quantile-regression"></a>En savoir plus sur la régression quantile

Il existe de nombreux types de régressions différents. En bref, la régression revient à ajuster un modèle à une cible exprimée comme un vecteur numérique. Toutefois, les statisticiens ont développé des méthodes de régression de plus en plus avancées.

Au sens le plus simple du terme, un *quantile* est une valeur qui divise un jeu de données en groupes de taille égale, les valeurs de quantile marquant les limites entre les groupes. Sur le plan statistique, les quantiles sont des valeurs extraites à intervalles réguliers à partir de l'inverse de la fonction de distribution cumulative d'une variable aléatoire.

Tandis que les modèles de régression linéaire tentent de prédire la valeur d'une variable numérique à l'aide d'une simple estimation, la *moyenne*, il faut parfois pouvoir prédire la plage ou la distribution entière de la variable cible. Des techniques telles que la régression bayésienne et la régression quantile ont été développées à cet effet.

La régression quantile vous aide à comprendre la distribution de la valeur prédite. Les modèles de régression quantile basée sur un arbre, tels que celui utilisé dans ce module, présentent l'avantage supplémentaire qu'ils permettent de prédire des distributions non paramétriques.

  
## <a name="how-to-configure-fast-forest-quantile-regression"></a>Configuration de la régression quantile de forêt rapide

1. Ajoutez le module **Régression quantile de forêt rapide** à votre pipeline dans le concepteur. Vous le trouverez sous **Algorithmes de Machine Learning**, dans la catégorie **Régression**.

2. Dans le volet droit du module **Régression quantile de forêt rapide**, spécifiez la façon dont vous souhaitez que le modèle soit formé, en définissant l’option **Créer le mode de formateur**.  
  
    - **Single Parameter** (Paramètre unique) : si vous savez comment vous voulez configurer le modèle, fournissez un ensemble spécifique de valeurs en tant qu’arguments. Lorsque vous formez le modèle, utilisez [Former le modèle](train-model.md).
  
    - **Plage de paramètres** : en cas de doute sur les paramètres à utiliser, changez les paramètres à l’aide du module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md). L’instructeur effectue une itération sur plusieurs valeurs que vous spécifiez pour trouver la configuration optimale.

3. **Nombre d'arbres**, saisissez le nombre maximal d’arbres qui peuvent être créés dans l’ensemble. Un plus grand nombre d'arbres conduit généralement à une plus grande précision, mais au prix d'un temps de formation plus long.  

4. **Nombre de feuilles**, indiquez le nombre maximal de feuilles ou de nœuds terminaux qui peuvent être créés dans n’importe quel arbre.  

5. **Nombre minimal d’instances de formation requises pour former une feuille**, spécifiez le nombre minimal d’exemples requis pour créer un nœud terminal (une feuille) dans un arbre.  
  
     Plus cette valeur est grande, plus le seuil de création de règles augmente. Par exemple, la valeur par défaut de 1, un seul cas suffit à entraîner la création d’une règle. Si vous définissez la valeur sur 5, les données d’apprentissage doivent contenir au moins 5 cas respectant les mêmes conditions.

6. **Fraction d'ensachage**, spécifiez un nombre compris entre 0 et 1 qui représente la fraction des échantillons à utiliser lors de la création de chaque groupe de quantiles. Les exemples sont choisis de façon aléatoire, avec remplacement.

7. **Fraction de division**, spécifiez un nombre compris entre 0 et 1 qui représente la fraction des fonctionnalités à utiliser dans chaque fraction de l’arbre. Les fonctionnalités utilisées sont toujours choisies de façon aléatoire.

8. **Quantiles à estimer**, fournissez une liste séparée par des points-virgules des quantiles sur lesquels vous souhaitez que le modèle apprenne et crée des prédictions.
  
     Par exemple, si vous souhaitez générer un modèle permettant d'estimer des quartiles, tapez `0.25; 0.5; 0.75`.  

9. Éventuellement, saisissez une valeur pour **Random number seed** (Valeur initiale aléatoire) pour amorcer le générateur de nombres aléatoires que le modèle utilise.  La valeur par défaut est 0, ce qui signifie qu'une valeur d'amorce aléatoire est choisie.
  
     Vous devez fournir une valeur si vous devez reproduire les résultats entre les exécutions successives sur les mêmes données.  

10. Connectez le jeu de données d’apprentissage et le modèle non formé à l’un des modules d’apprentissage : 

    - Si vous définissez **Créer un mode d’apprentissage** sur **Paramètre unique**, utilisez le module [Entraîner le du modèle](train-model.md).

    - Si vous définissez **Créer mode de formateur** sur **Plage de paramètres**, utilisez le module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md).

    > [!WARNING]
    > 
    > - Si vous transmettez une plage de paramètres au module [Entraîner le modèle](train-model.md), il utilise uniquement la première valeur dans la liste de plages de paramètres.
    > 
    > - Si vous transmettez un ensemble unique de valeurs de paramètre au module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md), quand il attend une plage de paramètres pour chaque paramètre, il ignore les valeurs et utilise les valeurs par défaut pour l’apprenant.
    > 
    > - Si vous sélectionnez l’option **Plage de paramètres** et que vous entrez une valeur unique pour un paramètre, cette valeur unique que vous avez spécifiée est utilisée tout au long du balayage, même si d’autres paramètres changent sur une plage de valeurs.

11. Envoyez le pipeline.

## <a name="results"></a>Résultats

Une fois l’apprentissage terminé :

+ Pour enregistrer un instantané du modèle entraîné, sélectionnez le module d’entraînement, puis basculez vers l’onglet **Sorties + journaux** dans le panneau droit. Cliquez sur l’icône **Inscrire le jeu de données**.  Vous pouvez trouver le modèle enregistré sous la forme d’un module dans l’arborescence du module.

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning.
