---
title: 'Entraîner le générateur de recommandations SVD : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Entraîner le générateur de recommandations SVD dans Azure Machine Learning pour former un conseiller bayésien à l’aide de l’algorithme SVD.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/22/2020
ms.openlocfilehash: a5740e851fbd8f7ba82e179f7e5299d6c7090596
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90890237"
---
# <a name="train-svd-recommender"></a>Entraîner le générateur de recommandations SVD

Cet article décrit comment utiliser le module Entraîner le générateur de recommandations SVD dans le concepteur Azure Machine Learning. Utilisez ce module pour entraîner un modèle de recommandation basé sur l’algorithme SVD (décomposition à valeur unique).  

Le module Entraîner le générateur de recommandations SVD lit un jeu de données de triplets utilisateur-élément-notation. Il retourne un générateur de recommandations SVD formé. Vous pouvez ensuite utiliser le modèle formé pour prédire des évaluations ou générer des recommandations à l’aide du module [Noter le générateur de recommandations SVD](score-svd-recommender.md).  


  
## <a name="more-about-recommendation-models-and-the-svd-recommender"></a>En savoir plus sur les modèles de recommandation et le générateur de recommandations SVD  

L’objectif principal d’un système de recommandation est de recommander un ou plusieurs *éléments* aux *utilisateurs* du système. Un film, un restaurant, un livre ou une chanson peuvent être des exemples d’un élément. Un utilisateur peut être une personne, un groupe de personnes ou une autre entité avec des préférences d’élément.  

Il existe deux approches principales pour les systèmes générateurs de recommandations : 

+ Une approche **basée sur le contenu** utilise des fonctionnalités à la fois pour les utilisateurs et les éléments. Les utilisateurs peuvent être décrits par des propriétés telles que l’âge et le sexe. Les éléments peuvent être décrits par des propriétés telles que l’auteur et le fabricant. Vous trouverez des exemples typiques de systèmes générateurs de recommandations basés sur le contenu sur les réseaux sociaux. 
+ Le **filtrage collaboratif** utilise uniquement les identificateurs des utilisateurs et des éléments. Il obtient des informations implicites sur ces entités à partir d’une matrice (fragmentée) de notations données aux éléments par les utilisateurs. Nous pouvons en apprendre plus sur un utilisateur à partir des éléments qu’il a évalués et des autres utilisateurs qui ont évalué les mêmes éléments.  

Le générateur de recommandations SVD utilise les identificateurs des utilisateurs et des éléments, ainsi qu’une matrice d’évaluations données par les utilisateurs aux éléments. Il s’agit d’un *générateur de recommandations collaboratif*. 

Pour plus d’informations sur le générateur de recommandations SVD, consultez le document de recherche associé : [Techniques de factorisation de matrice pour les systèmes de recommandation](https://datajobs.com/data-science-repo/Recommender-Systems-[Netflix].pdf).


## <a name="how-to-configure-train-svd-recommender"></a>Comment configurer le module Entraîner le générateur de recommandations SVD  

### <a name="prepare-data"></a>Préparer les données

Avant d’utiliser le module, vos données d’entrée doivent être au format attendu par le modèle de recommandation. Un jeu de données d’entraînement composé de triplets utilisateur-élément-notation est requis.

+ La première colonne contient les identificateurs des utilisateurs.
+ La deuxième colonne contient les identificateurs des éléments.
+ La troisième colonne contient l’évaluation de la paire utilisateur-élément. Les valeurs d’évaluation doivent être de type numérique.  

Le jeu de données **Notations de films** dans le concepteur Azure Machine Learning (sélectionnez **Jeux de données**, puis **Exemples**) illustre le format attendu :

![Notations de films](media/module/movie-ratings-dataset.png)

À partir de cet exemple, vous pouvez voir qu’un utilisateur unique a évalué plusieurs films. 

### <a name="train-the-model"></a>Effectuer l’apprentissage du modèle

1.  Ajoutez le module Entraîner le générateur de recommandations SVD à votre pipeline dans le concepteur et connectez-le aux données d’entraînement.  
   
2.  Pour **Nombre de facteurs**, spécifiez le nombre de facteurs à utiliser avec le générateur de recommandations.  
    
    Chaque facteur mesure le degré de relation entre l’utilisateur et l’élément. Le nombre de facteurs est également la dimensionnalité de l’espace de facteurs latent. Quand les nombres d’utilisateurs et d’éléments augmentent, il est préférable de définir un plus grand nombre de facteurs. Toutefois, si ce nombre est trop important, les performances peuvent chuter.
    
3.  **Number of recommendation algorithm iterations** (Nombre d’itérations de l’algorithme de recommandation) indique le nombre de fois que l’algorithme doit traiter les données d’entrée. Plus ce nombre est élevé, plus les prédictions sont justes. Toutefois, un nombre plus élevé signifie un entraînement plus lent. La valeur par défaut est 30.

4.  Pour **Learning rate** (Taux d’apprentissage), entrez un nombre compris entre 0,0 et 2,0 qui définira la taille de pas pour l’apprentissage.

    Le taux d’apprentissage détermine la taille de pas à chaque itération. Si la taille de pas est trop élevée, vous risquez de passer à côté de la solution optimale. Si la taille de pas est trop faible, l’entraînement nécessite plus de temps pour trouver la meilleure solution. 
  
5.  Envoyez le pipeline.  


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 
