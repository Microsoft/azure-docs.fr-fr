---
title: Apprentissage par renforcement - Personalizer
titleSuffix: Azure Cognitive Services
description: Personalizer utilise des informations sur les actions et sur le contexte pour faire de meilleures suggestions de classement. Les informations sur ces actions et ce contexte sont des attributs ou des propriétés qui sont appelés « caractéristiques ».
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 05/07/2019
ms.openlocfilehash: 8b97221de4921e06ddfab610618f37683b990181
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87132736"
---
# <a name="what-is-reinforcement-learning"></a>Qu’est-ce que l’apprentissage par renforcement ?

L’apprentissage par renforcement est une approche du machine learning qui apprend des comportements en obtenant un feedback sur son utilisation.
 
L’apprentissage par renforcement fonctionne :

* En fournissant une opportunité ou un degré de liberté pour décréter un comportement, comme prendre des décisions ou faire des choix.
* En fournissant des informations contextuelles sur l’environnement et sur les choix.
* En fournissant un feedback sur la façon dont le comportement atteint un certain objectif.

Bien qu’il existe de nombreux sous-types et styles d’apprentissage par renforcement, voici comment le concept fonctionne dans Personalizer :

* Votre application donne la possibilité de montrer un élément de contenu provenant d’une liste d’alternatives.
* Votre application fournit des informations sur chaque alternative et sur le contexte de l’utilisateur.
* Votre application calcule un _score de récompense_.

Contrairement à certaines approches d’apprentissage par renforcement, Personalizer ne nécessite pas de simulation pour fonctionner. Ses algorithmes d’apprentissage sont conçus pour réagir à un monde extérieur (au lieu de le contrôler) et pour apprendre de chaque point de données, tout en sachant qu’il s’agit d’une opportunité unique dont la création demande du temps et de l’argent, et qu’il y a un regret non nul (perte de récompense possible) dans le cas d’une performance non optimale.

## <a name="what-type-of-reinforcement-learning-algorithms-does-personalizer-use"></a>Quel type d’algorithmes d’apprentissage par renforcement Personalizer utilise ?

La version actuelle de Personalizer utilise l’approche des **bandits contextuels**, qui est une approche de l’apprentissage par renforcement structurée autour de la prise de décisions ou de choix entre des actions discrètes, dans un contexte donné.

La _mémoire des décisions_, le modèle qui a été entraîné pour capturer la meilleure décision possible dans un contexte donné, utilise un ensemble de modèles linéaires. Ceux-ci ont montré à de nombreuses reprises de bons résultats métier et constituent une approche éprouvée, en partie parce qu’ils peuvent apprendre du monde réel très rapidement sans nécessiter un entraînement en plusieurs passes, et en partie parce qu’ils peuvent compléter des modèles d’apprentissage supervisé et des modèles neuronaux profonds.

L’allocation du trafic d’exploration/exploitation est effectuée de façon aléatoire en suivant le pourcentage défini pour l’exploration ; l’algorithme par défaut pour l’exploration est « epsilon-greedy ».

### <a name="history-of-contextual-bandits"></a>Historique des bandits contextuels

John Langford a utilisé le nom de « Contextual Bandits » (Bandits contextuels) (Langford et Zhang [2007]) pour décrire un sous-ensemble maniable d’apprentissage par renforcement et a travaillé sur une demi-douzaine d’articles visant à améliorer notre compréhension de la façon d’apprendre dans ce paradigme :

* Beygelzimer et al. [2011]
* Dudík et al. [2011a,b]
* Agarwal et al. [2014, 2012]
* Beygelzimer and Langford [2009]
* Li et al. [2010]

John a également produit plusieurs tutoriels sur des sujets comme la prédiction mixte (ICML 2015), la théorie des bandits contextuels (NIPS 2013), l’apprentissage actif (ICML 2009) et des exemples de limites de complexité (ICML 2003)

## <a name="what-machine-learning-frameworks-does-personalizer-use"></a>Quelle infrastructure de machine learning Personalizer utilise ?

Personalizer utilise actuellement [Vowpal Wabbit](https://github.com/VowpalWabbit/vowpal_wabbit/wiki) comme base pour le machine learning. Cette infrastructure permet un débit maximal et une latence la moins élevée possible lors des classements de personnalisation et de l’entraînement du modèle avec tous les événements.

## <a name="references"></a>References

* [Making Contextual Decisions with Low Technical Debt](https://arxiv.org/abs/1606.03966)
* [A Reductions Approach to Fair Classification](https://arxiv.org/abs/1803.02453)
* [Efficient Contextual Bandits in Non-stationary Worlds](https://arxiv.org/abs/1708.01799)
* [Residual Loss Prediction: Reinforcement: learning With No Incremental Feedback](https://openreview.net/pdf?id=HJNMYceCW)
* [Mapping Instructions and Visual Observations to Actions with Reinforcement Learning](https://arxiv.org/abs/1704.08795)
* [Learning to Search Better Than Your Teacher](https://arxiv.org/abs/1502.02206)

## <a name="next-steps"></a>Étapes suivantes

[Évaluation hors connexion](concepts-offline-evaluation.md) 