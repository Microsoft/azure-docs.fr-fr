---
title: 'Régression logistique à deux classes : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Régression logistique à deux classes dans Azure Machine Learning pour créer un classifieur binaire.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 04/22/2020
ms.openlocfilehash: 2e29a666f4d478e11986f834cff94d9743223f22
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012644"
---
# <a name="two-class-logistic-regression-module"></a>Module de régression logistique à deux classes

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez ce module pour créer un modèle de régression logistique qui peut être utilisé pour prédire seulement deux résultats. 

La régression logistique est une technique statistique connue utilisée pour modéliser de nombreux types de problèmes. Cet algorithme est une méthode d’*apprentissage supervisé*. Par conséquent, vous devez fournir un jeu de données qui contient déjà les résultats pour former le modèle.  

### <a name="about-logistic-regression"></a>À propos de la régression logistique  

La régression logistique est une méthode bien connue en statistiques qui sert à déterminer la probabilité d’un résultat et est particulièrement populaire pour les tâches de classification. L’algorithme prédit la probabilité d’occurrence d’un événement en ajustant les données à une fonction logistique.
  
Dans ce module, l’algorithme de classification est optimisé pour les variables dichotomiques ou binaires. Si vous avez besoin de classer plusieurs résultats, utilisez le module de [régression logistique multiclasse](./multiclass-logistic-regression.md).

##  <a name="how-to-configure"></a>Comment configurer  

Pour entraîner ce modèle, vous devez fournir un jeu de données qui contient une colonne d’étiquette ou de classe. Étant donné que ce module est destiné aux problèmes à deux classes, la colonne d’étiquette ou de classe doit contenir exactement deux valeurs. 

Par exemple, la colonne d’étiquette peut être [Voté], avec les valeurs possibles « Oui » ou « Non ». Ou bien, elle peut être [Risque de crédit], avec les valeurs possibles « Élevé » ou « Faible ». 
  
1.  Ajouter le module **Régression logistique à deux classes** à votre pipeline.  
  
2.  Spécifiez le mode d’apprentissage du modèle en définissant l’option **Créer un mode d’apprentissage**.  
  
    -   **Single Parameter** (Paramètre unique) : si vous savez comment vous voulez configurer le modèle, vous pouvez fournir un ensemble spécifique de valeurs comme arguments.  

    -   **Plage de paramètres** : en cas de doute sur les paramètres à utiliser, utilisez le module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md). Vous fournissez une plage de valeurs, et le mode d'apprentissage effectue une itération sur plusieurs combinaisons de paramètres pour déterminer la combinaison de valeurs qui produit le meilleur résultat.
  
3.  Pour la **tolérance d’optimisation**, spécifiez une valeur de seuil à utiliser lors de l’optimisation du modèle. Si l’amélioration entre les itérations tombe au-dessous du seuil spécifié, on considère que l’algorithme a convergé vers une solution, et l’entraînement cesse.  
  
4.  Pour la **pondération de régularisation L1** et la **pondération de régularisation L2**, saisissez une valeur à utiliser pour les paramètres de régularisation L1 et L2. La définition d’une valeur non nulle est recommandée pour les deux options.  
     La *régularisation* est une méthode qui sert à empêcher le surajustement en pénalisant les modèles avec des valeurs de coefficient extrêmes. La régularisation fonctionne en ajoutant la pénalité associée aux valeurs de coefficient à l’erreur de l’hypothèse. Par conséquent, un modèle précis avec des valeurs de coefficient extrêmes est plus pénalisé, mais un modèle moins précis avec des valeurs plus conservatrices est moins pénalisé.  
  
     Les régularisations L1 et L2 ont différents effets et cas d’utilisation.  
  
    -   La régularisation L1 est applicable aux modèles dispersés, ce qui est utile lorsque vous travaillez avec des données de grande dimension.  
  
    -   En revanche, la régularisation L2 est préférable pour les données qui ne sont pas dispersées.  
  
     Cet algorithme prend en charge une combinaison linéaire de valeurs de régularisation L1 et L2 : autrement dit, si <code>x = L1</code> et <code>y = L2</code>, <code>ax + by = c</code> définit l’étendue linéaire des termes du contrat de régularisation.  
  
    > [!NOTE]
    >  Vous souhaitez en savoir plus sur la régularisation L1 et L2 ? L’article suivant explore les différences entre les régularisations L1 et L2 et comment elles influent sur l’ajustement du modèle, avec des exemples de code pour les modèles de régression logistique et de réseau neuronal :  [Régularisations L1 et L2 pour l’apprentissage automatique](/archive/msdn-magazine/2015/february/test-run-l1-and-l2-regularization-for-machine-learning)  
    >
    > Différentes combinaisons linéaires de termes L1 et L2 ont été imaginées pour les modèles de régression logistique : par exemple, [régularisation de réseau élastique](https://wikipedia.org/wiki/Elastic_net_regularization). Nous vous suggérons de faire référence à ces combinaisons pour définir une combinaison linéaire est efficace dans votre modèle.
      
5.  Pour la **taille de la mémoire pour L-BFGS**, spécifiez la quantité de mémoire à utiliser pour l’optimisation *L-BFGS*.  
  
     L-BFGS est l’acronyme de « limited memory Broyden-Fletcher-Goldfarb-Shanno ». Il s’agit d’un algorithme d’optimisation souvent utilisé pour l’estimation des paramètres. Ce paramètre indique le nombre de positions passées et de gradients à stocker pour le calcul de l’étape suivante.  
  
     Ce paramètre d’optimisation limite la quantité de mémoire utilisée pour calculer l’étape suivante et la direction. Lorsque vous spécifiez moins de mémoire, l’apprentissage est plus rapide mais moins précis.  
  
6.  Pour la **valeur initiale aléatoire**, saisissez une valeur entière. Il est important de définir une valeur de départ si vous souhaitez obtenir des résultats reproductibles sur plusieurs exécutions du même pipeline.  
  
  
8. Ajoutez un jeu de données étiqueté au pipeline, puis entraînez le modèle :

    + Si vous définissez **Create trainer mode** (Créer un mode d’apprentissage) sur **Single Parameter** (Paramètre unique), connectez un jeu de données balisé au module [Entraîner le modèle](train-model.md).  
  
    + Si vous définissez **Créer un mode d’entraînement** sur **Plage de paramètres**, connectez un jeu de données avec balises et entraînez le modèle en utilisant [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md).  
  
    > [!NOTE]
    > 
    > Si vous transmettez une plage de paramètres à [Entraîner le modèle](train-model.md), elle utilise uniquement la valeur par défaut dans la liste de paramètres unique.  
    > 
    > Si vous transmettez un ensemble unique de valeurs de paramètre au module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md), quand il attend une plage de paramètres pour chaque paramètre, il ignore les valeurs et utilise les valeurs par défaut pour l’apprenant.  
    > 
    > Si vous sélectionnez l’option **Plage de paramètres** et que vous entrez une valeur unique pour un paramètre, cette valeur unique que vous avez spécifiée est utilisée tout au long du balayage, même si d’autres paramètres changent dans une plage de valeurs.  
  
9. Envoyez le pipeline.  
  
## <a name="results"></a>Résultats

Une fois l’apprentissage terminé :
 
  
+ Pour effectuer des prévisions sur de nouvelles données, utilisez le modèle entraîné et les nouvelles données en tant qu’entrée pour le module de [notation de modèle](./score-model.md). 


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning.