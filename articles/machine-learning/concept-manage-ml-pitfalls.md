---
title: Éviter le surajustement et les données déséquilibrées avec AutoML
titleSuffix: Azure Machine Learning
description: Identifiez et gérez les pièges courants des modèles ML grâce aux solutions de Machine Learning automatisé d’Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: nibaccam
author: nibaccam
ms.author: nibaccam
ms.date: 04/09/2020
ms.openlocfilehash: ba85bda1e322d3efd467527b48bd4cd90eb7ce8c
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96922624"
---
# <a name="prevent-overfitting-and-imbalanced-data-with-automated-machine-learning"></a>Empêcher le surajustement et les données déséquilibrées avec le Machine Learning automatisé

Les données surajustées et déséquilibrées sont des pièges courants lorsque vous générez des modèles Machine Learning. Par défaut, le Machine Learning automatisé d’Azure Machine Learning fournit des graphiques et des métriques vous permettant d’identifier ces risques et met en œuvre les meilleures pratiques pour les atténuer. 

## <a name="identify-over-fitting"></a>Identifier les surajustements

Le surajustement en Machine Learning se produit quand un modèle ajuste trop bien les données d’entraînement, et est par conséquent dans l’incapacité de prédire avec précision des données de test invisibles. En d’autres termes, le modèle a simplement mémorisé du bruit et des patterns spécifiques dans les données d’entraînement, mais il n’est pas suffisamment flexible pour faire des prédictions sur des données réelles.

Considérez les modèles entraînés suivants et leurs précisions d’entraînement et de test correspondantes.

| Modèle | Précision d’entraînement | Précision de test |
|-------|----------------|---------------|
| Un | 99,9 % | 95 % |
| B | 87 % | 87 % |
| C | 99,9 % | 45 % |

Si l’on considère le modèle **A**, il existe une idée fausse selon laquelle si la précision de test sur des données invisibles est inférieure à la précision d’entraînement, le modèle est surajusté. Toutefois, la précision de test doit toujours être inférieure à la précision d’entraînement, et la distinction entre l’ajustement correct et le surajustement correspond en fait à la *différence* de précision. 

Si l’on compare les modèles **A** et **B**, le modèle **A** est meilleur, car il a une plus grande précision de test, et bien que la précision de test soit légèrement inférieure (95 %), il ne s’agit pas d’une différence significative qui traduit un surajustement. Vous ne choisiriez pas le modèle **B** simplement parce que les précisions d’entraînement et de test sont plus proches.

Le modèle **C** représente un cas clair de surajustement ; la précision d’entraînement est très élevée, mais la précision de test l’est beaucoup moins. Cette distinction est subjective, mais provient de la connaissance de votre problème et de vos données, ainsi que des amplitudes d’erreur acceptables.

## <a name="prevent-over-fitting"></a>Empêcher le surajustement

Dans les cas les plus flagrants, un modèle surajusté suppose que les combinaisons de valeurs de caractéristiques vues lors de l’entraînement produiront toujours exactement la même sortie pour la cible.

La meilleure façon d’empêcher le surajustement consiste à respecter les bonnes pratiques du Machine Learning, notamment :

* Utilisation de données d’entraînement supplémentaires et élimination du biais statistique
* Prévention des fuites de cibles
* Utilisation de moins de fonctionnalités
* **Régularisation et optimisation des hyperparamètres**
* **Limitations de la complexité des modèles**
* **Validation croisée**

Dans le contexte du ML automatisé, les trois premiers éléments ci-dessus sont des **bonnes pratiques que vous implémentez**. Les trois derniers éléments en gras sont **des bonnes pratiques implémentées par le ML automatisé** par défaut afin d’offrir une protection contre le surajustement. Dans les paramètres autres que le ML automatisé, il convient de respecter ces six bonnes pratiques pour éviter le surajustement des modèles.

### <a name="best-practices-you-implement"></a>Bonnes pratiques que vous implémentez

L’utilisation de **davantage** de données est le meilleur moyen (et le plus simple) d’empêcher le surajustement. En plus, cela permet généralement d’augmenter la justesse. Plus vous utilisez de données, plus il devient difficile pour le modèle de mémoriser des patterns exacts, et plus il est contraint de trouver des solutions plus flexibles pour prendre en charge davantage de conditions. Il est également important de reconnaître le **biais statistique** afin de garantir que vos données d’entraînement n’incluent pas de patterns isolés qui n’existeront pas dans les données de prédictions dynamiques. Ce scénario peut être difficile à résoudre, car il se peut qu’il n’y ait pas de surajustement entre vos jeux de test et d’entraînement, mais qu’il y en ait un par comparaison aux données de test dynamiques.

La **fuite de cible** est un problème similaire, où il est possible que le surajustement soit visible non pas entre les jeux d’entraînement et de test, mais au moment de la prédiction. La fuite de cible se produit quand votre modèle « triche » lors de l’entraînement en ayant accès à des données auxquelles il ne devrait normalement pas avoir accès au moment de la prédiction. Par exemple, si votre problème consiste à prédire, le lundi, quel sera le prix d’un produit le vendredi, mais que l’une de vos caractéristiques inclut accidentellement des données du jeudi, il s’agit de données auxquelles le modèle n’aura pas accès au moment de la prédiction, car il ne peut pas voir dans le futur. La fuite de cible est une erreur facile à manquer, mais elle est souvent caractérisée par une justesse anormalement élevée pour votre problème. Si vous tentez de prédire des cours boursiers et que vous avez entraîné un modèle à 95 % de justesse, il est probable qu’il existe des fuites de cible quelque part dans vos caractéristiques.

La **suppression de caractéristiques** peut aussi aider en cas de surajustement, en empêchant que le modèle ait un trop grand nombre de champs à utiliser pour mémoriser des patterns spécifiques, ce qui le rend plus flexible. Cela peut être difficile à mesurer quantitativement, mais si vous pouvez supprimer des caractéristiques et conserver la même justesse, vous avez probablement rendu le modèle plus flexible et vous avez réduit le risque de surajustement.

### <a name="best-practices-automated-ml-implements"></a>Meilleures pratiques implémentées par le ML automatisé

La **régularisation** est le processus qui consiste à minimiser une fonction de coût pour pénaliser les modèles complexes et surajustés. Il existe différents types de fonctions de régularisation, mais en général elles pénalisent toutes la complexité, la variance et la taille de coefficient du modèle. Le ML automatisé utilise L1 (Lasso), L2 (Ridge) et ElasticNet (L1 et L2 simultanément) dans différentes combinaisons avec des paramètres d’hyperparamètres de modèle différents qui contrôlent le surajustement. En termes simples, le ML automatisé peut varier le degré de régulation d’un modèle et choisir le résultat le plus juste.

Le ML automatisé implémente également des **limitations de complexité de modèle** explicites afin d’empêcher le surajustement. Dans la plupart des cas, cette implémentation concerne spécifiquement les algorithmes de forêt ou d’arbres de décision, où la profondeur maximale d’une arborescence donnée est limitée et le nombre total d’arbres utilisés dans les techniques de forêt ou d’ensemble est limité.

La **validation croisée** est le processus qui consiste à prendre de nombreux sous-ensembles de vos données d’entraînement complètes et à entraîner un modèle sur chaque sous-ensemble. L’idée est qu’un modèle pourrait être « chanceux » et avoir une grande justesse avec un sous-ensemble, mais avec l’utilisation de nombreux sous-ensembles le modèle ne présentera pas cette justesse élevée à chaque fois. Avec la validation croisée, vous fournissez un jeu de données d’exclusion de validation, vous spécifiez vos plis de validation croisée (le nombre de sous-ensembles), et le ML automatisé entraîne votre modèle et ajuste les hyperparamètres afin de minimiser les erreurs sur votre jeu de validation. Il est possible qu’un pli de validation croisée présente un surajustement, mais l’utilisation de nombreux plis réduit la probabilité que votre modèle final soit surajusté. L’inconvénient est que la validation croisée entraîne des temps d’entraînement plus longs, et par conséquent un coût plus élevé, car au lieu d’entraîner un modèle une seule fois, vous l’entraînez une fois pour tous les *n* sous-ensembles de validation croisée. 

> [!NOTE]
> La validation croisée n’est pas activée par défaut ; vous devez la configurer dans les paramètres de ML automatisé. Cependant, une fois que la validation croisée est configurée et qu’un jeu de données de validation a été fourni, le processus est automatisé. En savoir plus sur la [configuration de la validation croisée dans Auto ML](how-to-configure-cross-validation-data-splits.md)

<a name="imbalance"></a>

## <a name="identify-models-with-imbalanced-data"></a>Identifier les modèles présentant des données déséquilibrées

Les données déséquilibrées se trouvent généralement dans les scénarios de classification de machine learning. Il s’agit des données dont le taux d’observations dans chaque classe est disproportionné. Ce déséquilibre peut donner une idée faussement positive de la justesse d’un modèle parce que les données d’entrée présentent un biais vers une classe, ce qui influence le modèle entraîné. 

Par ailleurs, les exécutions du ML automatisé génèrent automatiquement les graphiques suivants, qui peuvent vous aider à comprendre la cohérence des classifications de votre modèle et à identifier les modèles potentiellement impactés par des données déséquilibrées.

Graphique| Description
---|---
[Matrice de confusion](how-to-understand-automated-ml.md#confusion-matrix)| Évalue les étiquettes correctement classifiées par rapport aux étiquettes réelles des données. 
[Rappel de précision](how-to-understand-automated-ml.md#precision-recall-curve)| Évalue le ratio d’étiquettes correctes par rapport au ratio d’instances d’étiquettes trouvées dans les données. 
[Courbe ROC](how-to-understand-automated-ml.md#roc-curve)| Évalue le ratio d’étiquettes correctes par rapport au ratio d’étiquettes considérées comme étant des faux positifs.

## <a name="handle-imbalanced-data"></a>Traiter les données déséquilibrées 

Dans l’optique de simplifier le workflow du Machine Learning, le **ML automatisé intègre des fonctionnalités** qui permettent de gérer les données déséquilibrées, à savoir : 

- Une **colonne de pondération** : le ML automatisé peut utiliser une colonne de poids en guise d’entrée. Les lignes de données sont alors pondérées, ce qui peut accentuer ou réduire « l’importance » d’une classe.

- Les algorithmes utilisés par le ML automatisé détectent le déséquilibre quand le nombre d’échantillons dans la classe minoritaire est égal ou inférieur à 20 % du nombre d’échantillons dans la classe majoritaire, où la classe minoritaire fait référence à celle ayant le moins d’échantillons et la classe majoritaire fait référence à celle ayant la plupart des échantillons. Par la suite, AutoML exécutera une expérience avec des données sous-échantillonnées pour vérifier si l’utilisation des poids des classes résoudrait ce problème et améliorerait les performances. S’il conclut à une amélioration des performances par le biais de cette expérience, ce recours est appliqué.

- Utilisation d’une métrique de performances qui gère mieux les données déséquilibrées. Par exemple, AUC_weighted est une métrique principale qui calcule la contribution de chaque classe en fonction du nombre relatif d’échantillons représentant cette classe, d’où une meilleure fiabilité contre le déséquilibre.

Les techniques suivantes sont des options supplémentaires pour traiter les données déséquilibrées **en dehors du ML automatisé**. 

- Rééchantillonnage destiné à niveler le déséquilibre des classes, soit en suréchantillonnant les classes les plus petites soit en sous-échantillonnant les classes les plus grandes. Ces méthodes demandent des compétences techniques pour le traitement et l’analyse.

- Examinez les métriques de performances afin d’identifier les données déséquilibrées. Par exemple, le score F1 est la moyenne harmonique de la précision et du rappel. La précision mesure l’exactitude d’un classifieur, où une précision plus élevée indique moins de faux positifs, tandis que le rappel mesure l’exhaustivité d’un classifieur, où un rappel plus élevé indique moins de faux négatifs.

## <a name="next-steps"></a>Étapes suivantes

Consultez les exemples et découvrez comment générer des modèles à l’aide du Machine Learning automatisé :

+ Suivez le [Didacticiel : Entraîner automatiquement un modèle de régression avec Azure Machine Learning](tutorial-auto-train-models.md)

+ Configurez les paramètres pour l’expérience d’apprentissage automatique :
  + Dans Azure Machine Learning Studio, [suivez ces étapes](how-to-use-automated-ml-for-ml-models.md).
  + Avec le Kit de développement logiciel (SDK) Python, [procédez comme suit](how-to-configure-auto-train.md).


