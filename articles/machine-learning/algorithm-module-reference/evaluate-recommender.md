---
title: 'Évaluer le générateur de recommandations : sur le module Modifier les métadonnées'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Évaluer le générateur de recommandations dans Azure Machine Learning pour évaluer la précision de prédictions du modèle de générateur de recommandations.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 10/10/2019
ms.openlocfilehash: 0890e13acbba8dae31de28d7c78a81bd9b516853
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76312258"
---
# <a name="evaluate-recommender"></a>Évaluer le générateur de recommandations

Cet article décrit comment utiliser le module Évaluer le générateur de recommandations dans le concepteur Azure Machine Learning. L’objectif est de mesurer la justesse des prédictions effectuées par un modèle de recommandation. Grâce à ce module, vous pouvez évaluer différents types de recommandations :  
  
-   Notations prédites pour un utilisateur et un élément    
-   Éléments recommandés pour un utilisateur  
  
Lorsque vous créez des prédictions à l’aide d’un modèle de recommandation, des résultats légèrement différents sont retournés pour chacun de ces types de prédictions pris en charge. Le module Évaluer le générateur de recommandations déduit le type de prédiction du format de colonne du jeu de données noté. Par exemple, le jeu de données noté peut contenir :

- Des triplets utilisateur-élément-notation
- Des utilisateurs et leurs éléments recommandés

Le module applique également les métriques de performances appropriées, en fonction du type de prédiction effectué. 

  
## <a name="how-to-configure-evaluate-recommender"></a>Comment configurer Évaluer le générateur de recommandations

Le module Évaluer le générateur de recommandations compare les résultats de prédiction en utilisant un modèle de recommandation avec les données correspondantes de la « réalité du terrain ». Par exemple, le module [Noter le générateur de recommandations SVD](score-svd-recommender.md) produit des jeux de données notés que vous pouvez analyser à l’aide du module Évaluer le générateur de recommandations.

### <a name="requirements"></a>Spécifications

Le module Évaluer le générateur de recommandations requiert les jeux de données suivants en entrée. 
  
#### <a name="test-dataset"></a>Jeu de données de test

Le jeu de données de test contient les données de la « réalité du terrain » sous la forme de triplets utilisateur-élément-notation.  

#### <a name="scored-dataset"></a>Jeu de données noté

Le jeu de données noté contient les prédictions générées par le modèle de recommandation.  
  
Les colonnes de ce second jeu de données dépendent du type de prédiction que vous avez effectué pendant le processus de scoring. Par exemple, le jeu de données noté peut contenir l’un des éléments suivants :

- Des utilisateurs, des éléments et les notations que l’utilisateur voudra probablement donner pour l’élément
- Liste des utilisateurs et des éléments recommandés pour ceux-ci 

### <a name="metrics"></a>Mesures

Les métriques de performances du modèle sont générées en fonction du type d’entrée. Les sections suivantes fournissent des détails.

## <a name="evaluate-predicted-ratings"></a>Évaluer les notations prédites  

Lorsque vous évaluez des notations prédites, le jeu de données noté (la deuxième entrée du module Évaluer le générateur de recommandations) doit contenir des triplets utilisateur-élément-notation répondant aux exigences suivantes :
  
-   La première colonne du jeu de données contient les identificateurs d’utilisateur.    
-   La deuxième colonne contient les identificateurs d’élément.  
-   La troisième colonne contient les notations utilisateur-élément correspondantes.  
  
> [!IMPORTANT] 
> Pour que l’évaluation aboutisse, les noms de colonnes doivent être `User`, `Item` et `Rating`, respectivement.  
  
Le module Évaluer le générateur de recommandations compare les notations du jeu de données de la « réalité du terrain » aux notations prédites du jeu de données noté. Il calcule ensuite l’erreur absolue moyenne (MAE) et l’erreur quadratique moyenne (RMSE).



## <a name="evaluate-item-recommendations"></a>Évaluer les recommandations d’éléments

Lorsque vous évaluez des recommandations d’élément, utilisez un jeu de données noté incluant les éléments recommandés pour chaque utilisateur :
  
-   La première colonne du jeu de données contient l’identificateur d’utilisateur.    
-   Toutes les colonnes suivantes doivent contenir les identificateurs d’éléments recommandés correspondants, classés selon la pertinence d’un élément par rapport à l’utilisateur. 

Avant que vous connectiez ce jeu de données, nous vous recommandons de le trier afin que les éléments les plus pertinents passent en premier.  

> [!IMPORTANT] 
> Pour que le module Évaluer le générateur de recommandations fonctionne, les noms de colonnes doivent être `User`, `Item 1`, `Item 2`, `Item 3`, et ainsi de suite.  
  
Le module Évaluer le générateur de recommandations calcule le gain cumulé réduit normalisé moyen (NDCG) et le retourne dans le jeu de données de sortie.  
  
Comme il est impossible de connaître la « réalité du terrain » pour les éléments recommandés, le module Évaluer le générateur de recommandations utilise les notations utilisateur-élément dans le jeu de données de test en tant que gains dans le calcul du NDCG. Pour réaliser l’évaluation, le module de scoring du générateur de recommandations doit uniquement produire des recommandations pour les éléments dotés de notations de « réalité du terrain » (dans le jeu de données de test).  
  

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 
