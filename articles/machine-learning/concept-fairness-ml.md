---
title: Atténuer l’impartialité dans les modèles Machine Learning (préversion)
titleSuffix: Azure Machine Learning
description: Découvrez l’impartialité dans les modèles d’apprentissage automatique et la façon dont le package Python Fairlearn peut vous aider à créer des modèles plus impartiaux.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: luquinta
author: luisquintanilla
ms.date: 08/05/2020
ms.openlocfilehash: 459cb1f7ea6c756b8cf6eba70af5ebabe76cc8b0
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94335756"
---
# <a name="mitigate-fairness-in-machine-learning-models-preview"></a>Atténuer l’impartialité dans les modèles Machine Learning (préversion)

Découvrez l’impartialité dans l’apprentissage automatique et la façon dont le package Python open source [Fairlearn](https://fairlearn.github.io/) peut vous aider à atténuer les problèmes d’impartialité dans les modèles Machine Learning. Si vous ne faites pas l’effort de comprendre les problèmes de partialité et d’évaluer l’impartialité lors de la création de modèles de Machine Learning, vous pourriez créer des modèles qui produisent des résultats injustes.

Le résumé suivant du [guide de l’utilisateur](https://fairlearn.github.io/master/user_guide/index.html) pour le package open source Fairlearn, décrit comment l’utiliser pour évaluer l’impartialité des systèmes IA que vous générez.  Le package open source Fairlearn peut également fournir des options permettant d’atténuer ou de réduire les problèmes de partialité que vous observez.  Pour activer l’évaluation de l’impartialité de systèmes IA pendant la formation sur Azure Machine Learning, consultez [cette page](how-to-machine-learning-fairness-aml.md) et les [notebooks échantillons](https://github.com/Azure/MachineLearningNotebooks/tree/master/contrib/fairness).


## <a name="what-is-fairness-in-machine-learning-models"></a>Qu’est-ce que l’impartialité dans les modèles Machine Learning ?

>[!NOTE]
> L’impartialité est un défi socio-technique. De nombreux aspects de l’impartialité, tels que la justice et l’équité des processus, ne sont pas intégrés dans les métriques quantitatives d’impartialité. De même, de nombreuses métriques quantitatives d’impartialité ne peuvent pas être satisfaites simultanément. L’objectif du package open source Fairlearn est de permettre aux humains d’évaluer les différentes stratégies d’impact et d’atténuation. In fine, il incombe aux utilisateurs humains de créer des intelligence artificielles et des modèles de Machine Learning pour faire des compromis adaptés à leur scénario.

Des systèmes d’intelligence artificielle et d’apprentissage automatique affichent parfois un comportement partial. On peut définir un comportement partial comme un comportement qui nuit à des personnes ou a une incidence sur celles-ci. Les systèmes d’intelligence artificielle peuvent occasionner de nombreux types de préjudices. Pour en savoir plus, consultez le [NeurIPS 2017 Keynote par Kate Crawford](https://www.youtube.com/watch?v=fMym_BKWQzk).

Parmi les types courants de préjudices que peut occasionner l’intelligence artificielle, citons les deux suivants :

- Préjudice d’allocation : Un système d’intelligence artificielle étend ou retient des opportunités, des ressources ou des informations pour certains groupes. Il peut s’agir, par exemple, d’embauche, d’admission dans un établissement d’enseignement ou d’octroi de prêt, où un modèle pourrait être enclin à sélectionner des candidats dans un groupe spécifique de personnes, que dans d’autres groupes.

- Préjudice de qualité de service : un système d’intelligence artificielle ne fonctionne pas aussi bien pour un groupe de personnes que pour un autre. Par exemple, un système de reconnaissance vocale peut ne pas fonctionner aussi bien pour les femmes que pour les hommes.

Pour minimiser les comportements partiaux des systèmes d’intelligence artificielle, vous devez évaluer et atténuer ces préjudices.

## <a name="fairness-assessment-and-mitigation-with-fairlearn"></a>Évaluation et atténuation de la partialité avec Fairlearn

Fairlearn est un package Python open source qui permet aux développeurs de systèmes d’apprentissage automatique d’évaluer l’impartialité de leurs systèmes et d’atténuer les problèmes de partialité observés.

Le package open source Fairlearn comporte deux composants :

- Tableau de bord d’évaluation : widget de bloc-notes Jupyter permettant d’évaluer la façon dont les prévisions d’un modèle affectent différents groupes. Il permet également de comparer plusieurs modèles à l’aide de métriques d’impartialité et de performances.
- Algorithmes d’atténuation : série d’algorithmes permettant d’atténuer la partialité en matière de classification et de régression binaires.

Ensemble, ces composants permettent aux scientifiques des données et aux dirigeants d’explorer divers compromis entre impartialité et performances, et de choisir la stratégie d’atténuation correspondant le mieux à leurs besoins.

## <a name="assess-fairness-in-machine-learning-models"></a>Évaluer l’impartialité dans les modèles Machine Learning

Dans le package open source Fairlearn, l’impartialité est conceptualisée au travers d’une approche connue sous le nom d’ **impartialité de groupe** , qui demande : sur les groupes de personnes exposés à un éventuel préjudice. Les groupes pertinents, également appelés sous-populations, sont définis par des **caractéristiques sensibles** ou attributs sensibles. Les fonctionnalités sensibles sont transmises à un estimateur dans le package open source Fairlearn sous la forme d’un vecteur ou d’une matrice appelée `sensitive_features`. Cette dénomination suggère que le concepteur du système doit être sensible à ces caractéristiques lors de l’évaluation de l’impartialité du groupe. 

Gardez à l’esprit que ces fonctionnalités contiennent des implications en termes de confidentialité en raison de données privées. Le mot « sensible » n’implique cependant pas que ces caractéristiques ne doivent pas être utilisées pour effectuer des prévisions.

>[!NOTE]
> L’évaluation de l’impartialité n’est pas un exercice purement technique.  Le package open source Fairlearn peut vous aider à évaluer l’impartialité d’un modèle, mais il n’effectue pas l’évaluation pour vous.  Ce package open source Fairlearn aide à identifier des métriques quantitatives pour évaluer l’impartialité, mais les développeurs de modèles d’apprentissage automatique doivent également effectuer une analyse qualitative pour évaluer l’impartialité de leurs propres modèles.  Les fonctionnalités sensibles mentionnées ci-dessus sont un exemple de ce type d’analyse qualitative.     

Pendant la phase d’évaluation, l’impartialité est quantifiée à l’aide de métriques de disparité. Ces **métriques de disparité** permettent d’évaluer et de comparer le comportement du modèle dans différents groupes par le biais de ratios ou de différences. Le package open source Fairlearn prend en charge deux classes de métriques de disparité :


- Disparité de performances du modèle : ces ensembles de métriques calculent la disparité (fluctuation) des valeurs d’une métrique de performances sélectionnée entre différents sous-groupes. Voici quelques exemples :

  - disparité de taux d’exactitude
  - disparité de taux d’erreurs
  - disparité de précision
  - disparité de rappel
  - disparité d’erreur absolue moyenne
  - et bien plus

- Disparité de taux de sélection : cette métrique exprime la différence de taux de sélection entre différents sous-groupes. Il s’agit, par exemple, d’une disparité de taux d’approbation de prêt. Le taux de sélection signifie la fraction de points de données dans chaque classe qui sont classifiés comme 1 (dans une classification binaire), ou la distribution de valeurs de prédiction (dans une régression).

## <a name="mitigate-unfairness-in-machine-learning-models"></a>Atténuer l’impartialité dans les modèles Machine Learning

### <a name="parity-constraints"></a>Contraintes de parité

Le package open source Fairlearn comprend un large éventail d’algorithmes d’atténuation des risques de partialité. Ces algorithmes prennent en charge un ensemble de contraintes déterminant le comportement du prédicteur, appelées **contraintes de parité** ou critères. Les contraintes de parité requièrent que certains aspects du comportement du prédicteur soient comparables dans les groupes que des caractéristiques sensibles définissent (par exemple, la nationalité). Les algorithmes d’atténuation dans le package open source Fairlearn utilisent ces contraintes de parité pour atténuer les problèmes de partialité observés.

>[!NOTE]
> L’atténuation de la partialité dans un modèle signifie réduire l’injustice, mais cette atténuation technique ne peut pas éliminer complètement cette partialité.  Les algorithmes d’atténuation des risques de partialité du package open source Fairlearn peuvent fournir des stratégies d’atténuation conseillées pour réduire la partialité d’un modèle de Machine Learning, mais ce ne sont pas des solutions permettant d’éliminer complètement la partialité.  Il peut y avoir d’autres contraintes ou critères de parité qui doivent être pris en compte pour chaque modèle de Machine Learning d’un développeur particulier. Les développeurs qui utilisent Azure Machine Learning doivent déterminer eux-mêmes si l’atténuation élimine suffisamment de partialité dans le cadre de l’utilisation et le déploiement de modèles de Machine Learning qu’ils ont prévus.  

Le package open source Fairlearn prend en charge les types de contraintes de parité suivants : 

|Contrainte de parité  | Objectif  |Tâche d’apprentissage automatique  |
|---------|---------|---------|
|Parité démographique     |  Atténuer les préjudices d’allocation | Classification binaire, régression |
|Chances équilibrées  | Diagnostiquer les préjudice d’allocation et de qualité de service | Classification binaire        |
|Égalité des chances | Diagnostiquer les préjudice d’allocation et de qualité de service | Classification binaire        |
|Perte de groupe limité     |  Atténuer les préjudices de qualité de service | régression ; |



### <a name="mitigation-algorithms"></a>Algorithmes d’atténuation

Le package open source Fairlearn fournit des algorithmes d’atténuation de post-traitement et de réduction de la partialité :

- Réduction : Ces algorithmes prennent un estimateur de Machine Learning de boîte noire standard (par exemple, un modèle LightGBM) et génèrent un ensemble de modèles reformés à l’aide d’une séquence de jeux de données d’apprentissage re-pondérés. Par exemple, les candidats d’un sexe peuvent être pondérés positivement ou négativement pour reformer des modèles et réduire les disparités entre groupes de sexe différent. Les utilisateurs peuvent ensuite choisir un modèle offrant le meilleur compromis entre l’exactitude (ou d’autres métriques de performances) et la disparité, qui devraient généralement reposer sur des règles métier et des calculs de coût.  
- Post-traitement  : ces algorithmes prennent un classifieur existant et la caractéristique sensible comme entrée. Ils dérivent ensuite une transformation de la prédiction du classifieur pour appliquer les contraintes d’impartialité spécifiées. Le principal avantage de l’optimisation de seuil réside dans sa simplicité et sa flexibilité, car elle n’a pas besoin de reformer le modèle. 

| Algorithm | Description | Tâche d’apprentissage automatique | Caractéristiques sensibles | Contraintes de parité prises en charge | Type d’algorithme |
| --- | --- | --- | --- | --- | --- |
| `ExponentiatedGradient` | Approche de la boîte noire pour une classification impartiale décrite dans le document [A Reductions Approach to Fair Classification](https://arxiv.org/abs/1803.02453) | Classification binaire | Par catégorie | [Parité démographique](#parity-constraints), [Chances équilibrées](#parity-constraints) | Réduction |
| `GridSearch` | Approche de la boîte noire décrite dans l’article [A Reductions Approach to Fair Classification](https://arxiv.org/abs/1803.02453)| Classification binaire | Binary | [Parité démographique](#parity-constraints), [Chances équilibrées](#parity-constraints) | Réduction |
| `GridSearch` | Approche de boîte noire qui implémente une variante de formulaire de recherche de la Régression impartiale avec l’algorithme de perte de groupe limité décrit dans le document [Fair Regression: Quantitative Definitions and Reduction-based Algorithms](https://arxiv.org/abs/1905.12843) | régression ; | Binary | [Perte de groupe limité](#parity-constraints) | Réduction |
| `ThresholdOptimizer` | Algorithme de post-traitement basé sur le document [Equality of Opportunity in Supervised Learning](https://arxiv.org/abs/1610.02413). Cette technique prend comme entrée un classifieur et la caractéristique sensible, et dérive une transformation monotone de la prédiction du classifieur pour appliquer les contraintes de parité spécifiées. | Classification binaire | Par catégorie | [Parité démographique](#parity-constraints), [Chances équilibrées](#parity-constraints) | Post-traitement |

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment utiliser les différents composants en consultant le [GitHub](https://github.com/fairlearn/fairlearn/), le [guide de l’utilisateur](https://fairlearn.github.io/master/user_guide/index.html), les [exemples](https://fairlearn.github.io/master/auto_examples/)et les [exemples de notebooks](https://github.com/fairlearn/fairlearn/tree/master/notebooks) de Fairlearn.
- Découvrez [comment](how-to-machine-learning-fairness-aml.md) activer l’évaluation de l’impartialité des modèles de Machine Learning dans Azure Machine Learning.
- Consultez les [exemples de notebooks](https://github.com/Azure/MachineLearningNotebooks/tree/master/contrib/fairness) pour connaître d’autres scénarios d’évaluation de l’impartialité dans Azure Machine Learning. 
