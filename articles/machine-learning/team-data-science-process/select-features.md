---
title: Sélection de fonctionnalités dans le processus TDSP (Team Data Science Process)
description: Explique la finalité de la sélection de fonctionnalités et fournit des exemples de son rôle dans le processus d’amélioration des données de l’apprentissage automatique.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 940d4c465acb5e8478d2b204ab5b9b18c79254f0
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93321315"
---
# <a name="feature-selection-in-the-team-data-science-process-tdsp"></a>Sélection de fonctionnalités dans le processus TDSP (Team Data Science Process)
Cet article explique les finalités de la sélection de fonctionnalités et fournit des exemples de son rôle dans le processus d’amélioration des données de l’apprentissage automatique. Ces exemples sont tirés d’Azure Machine Learning Studio.

La conception et la sélection de fonctionnalités constituent une partie du processus TDSP présenté dans l’article [Qu’est ce que le processus TDSP (Team Data Science Process) ?](overview.md). La conception et la sélection de fonctionnalités sont des parties de l’étape de **développement de fonctionnalités** du processus TDSP.

* **Ingénierie des caractéristiques** : ce processus tente de créer des caractéristiques supplémentaires pertinentes à partir de caractéristiques brutes existantes dans les données et d’augmenter la performance de prédiction de l’algorithme d’apprentissage.
* **Sélection de caractéristiques** : ce processus sélectionne le sous-ensemble clé des caractéristiques de données d’origine afin de réduire la dimensionnalité du problème d’apprentissage.

En général, l’ **ingénierie de caractéristiques** s’applique d’abord à la génération de caractéristiques supplémentaires. L’étape de **sélection de caractéristiques** est alors effectuée pour éliminer les caractéristiques inutiles, redondantes ou fortement corrélées.

## <a name="filter-features-from-your-data---feature-selection"></a>Filtrage des caractéristiques à partir de vos données : sélection de caractéristiques
La sélection de caractéristiques peut être utilisée pour les tâches de classification ou de régression. L'objectif est de sélectionner un sous-ensemble des caractéristiques du jeu de données d'origine qui réduit ses dimensions à l'aide d'un ensemble minimal de caractéristiques pour représenter l'écart de quantité maximum dans les données. Ce sous-ensemble de caractéristiques est utilisé pour l’apprentissage du modèle. La sélection de caractéristiques a deux principaux objectifs.

* Tout d'abord, la sélection des caractéristiques augmente souvent la précision de classification en éliminant les caractéristiques non pertinentes, redondantes ou fortement liées.
* De plus, elle réduit le nombre de caractéristiques qui rendent le processus d’apprentissage du modèle plus efficace. L'efficacité est importante pour les apprenants dont l'apprentissage coûte cher, comme les machines à vecteurs de support.

Bien que la sélection des caractéristiques ait pour objet de réduire le nombre de caractéristiques dans le jeu de données utilisé pour l’apprentissage du modèle, celle-ci ne correspond pas au terme de « réduction de la dimensionnalité ». Les méthodes de sélection de caractéristiques extraient un sous-ensemble des caractéristiques d'origine dans les données sans les modifier.  Les méthodes de réduction de la dimensionnalité utilisent l'ingénierie des caractéristiques qui peuvent transformer les caractéristiques d'origine et donc les modifier. Parmi les exemples de méthodes de réduction de la dimensionnalité, on peut noter l'analyse du composant principal, l'analyse canonique des corrélations et la décomposition en valeurs uniques.

Notamment, l’une des méthodes de sélection de caractéristiques de catégorie largement appliquée dans un contexte supervisé est appelée « sélection de caractéristiques basée sur les filtres ». En évaluant la corrélation entre chaque caractéristique et l'attribut cible, ces méthodes appliquent une mesure statistique pour attribuer un score à chaque caractéristique. Les caractéristiques sont ensuite classées suivant le score qui peut être utilisé pour aider à définir le seuil de conservation ou d'élimination d'une caractéristique spécifique. Parmi les exemples de mesures statistiques utilisées dans ces méthodes, on peut noter la corrélation de Pearson, des informations mutuelles et le test de la loi du Khi-deux.

Dans Azure Machine Learning Studio, des modules sont fournis pour la sélection des caractéristiques. Comme indiqué dans la figure suivante, ces modules comprennent une [sélection de caractéristiques par filtrage][filter-based-feature-selection] et une [analyse discriminante linéaire de Fisher][fisher-linear-discriminant-analysis].

![Modules de sélection de caractéristiques](./media/select-features/feature-Selection.png)

Prenons l’exemple de l’utilisation du module de [sélection de caractéristiques par filtrage][filter-based-feature-selection]. Pour plus de commodité, continuez à utiliser l’exemple d’exploration de texte. Supposons que nous voulons créer un modèle de régression après avoir créé un ensemble de 256 caractéristiques via le module de [hachage de caractéristiques][feature-hashing] et que la variable de réponse est le « Col1 » contenant une critique de livre notée de 1 à 5. La « Méthode de notation des caractéristiques » doit être définie en tant que « corrélation de Pearson », la « Colonne cible » en tant que « Col1 » et le « nombre de caractéristiques souhaitées » à 50. Le module de [sélection de caractéristiques par filtrage][filter-based-feature-selection] produit ensuite un jeu de données contenant 50 caractéristiques avec l’attribut cible « Col1 ». La figure suivante montre le flux de cette expérimentation et les paramètres d’entrée :

![Propriétés du module Sélection de caractéristiques par filtrage](./media/select-features/feature-Selection1.png)

La figure suivante montre les jeux de données qui en résultent :

![Jeu de données obtenu pour le module de sélection de caractéristiques par filtrage](./media/select-features/feature-Selection2.png)

Chaque caractéristique obtient un score basé sur la corrélation de Pearson entre elle et l'attribut cible « Col1 ». Les caractéristiques avec les scores les plus élevés sont conservées.

Les scores correspondants des caractéristiques sélectionnées sont indiqués dans la figure suivante :

![Scores pour le module de sélection de caractéristiques par filtrage](./media/select-features/feature-Selection3.png)

En appliquant ce module de [sélection de caractéristiques par filtrage][filter-based-feature-selection], 50 des 256 caractéristiques sont sélectionnées, car elles présentent la corrélation la plus importante avec la variable cible « Col1 » selon la méthode de notation « corrélation de Pearson ».

## <a name="conclusion"></a>Conclusion
La conception et la sélection de fonctionnalités augmentent toutes deux l'efficacité du processus d'apprentissage qui tend à extraire les informations essentielles contenues dans les données. Ces processus améliorent également les performances de ces modèles pour classifier les données d'entrée avec précision et prédire les résultats pertinents de façon plus consistante. L'ingénierie et la sélection de caractéristiques peuvent également être combinées afin de rendre l'apprentissage plus souple d'un point de vue informatique. Cela se fait grâce à l'amélioration puis à la réduction du nombre de caractéristiques nécessaires à l'étalonnage ou l'apprentissage d'un modèle. D'un point de vue mathématique, les caractéristiques sélectionnées pour effectuer l'apprentissage du modèle sont un ensemble minimum de variables indépendantes qui expliquent les modèles des données puis prédisent correctement les résultats.

Il n’est pas toujours nécessaire d’effectuer l’ingénierie de caractéristiques ou la sélection des caractéristiques. Que cela soit nécessaire ou non dépend des données collectées, de l’algorithme sélectionné et des objectifs de l’expérience.

<!-- Module References -->
[feature-hashing]: /azure/machine-learning/studio-module-reference/feature-hashing
[filter-based-feature-selection]: /previous-versions/azure/dn905854(v=azure.100)
[fisher-linear-discriminant-analysis]: /azure/machine-learning/studio-module-reference/fisher-linear-discriminant-analysis