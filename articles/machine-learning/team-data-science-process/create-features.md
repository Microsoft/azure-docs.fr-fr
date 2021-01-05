---
title: Conception de fonctionnalités en science des données - TDSP (Team Data Science Process)
description: En savoir plus sur l’ingénierie de caractéristiques et son rôle dans le processus d’amélioration des données d’apprentissage automatique.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: conceptual
ms.date: 05/14/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath, contperf-fy20q4
ms.openlocfilehash: 7cf97edcb2f2f7a23d5c899194d497deade1f507
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97031859"
---
# <a name="feature-engineering-in-data-science"></a>Ingénierie des caractéristiques dans la science des données

Dans cet article, vous allez découvrir l’ingénierie de caractéristiques et son rôle dans l’amélioration des données d’apprentissage automatique. Découvrez à partir des exemples explicatifs tirés d’expériences [Azure Machine Learning Studio (classique)](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio). 

* **Ingénierie des caractéristiques** : processus de création de nouvelles caractéristiques à partir de données brutes pour augmenter la puissance prédictive de l’algorithme d’apprentissage. Les caractéristiques conçues doivent capturer des informations supplémentaires qui ne sont pas facilement visibles dans l’ensemble de caractéristiques d’origine.
* **Sélection de caractéristiques** : Processus de sélection du sous-ensemble clé de caractéristiques pour réduire la dimensionnalité du problème d’apprentissage.

En général, l’**ingénierie de caractéristiques** s’applique d’abord à la génération de caractéristiques supplémentaires. La **sélection de caractéristiques** est alors effectuée pour éliminer des caractéristiques non pertinentes, redondantes ou fortement corrélées.

L’ingénierie et la sélection de caractéristiques font partie de la [phase de modélisation](lifecycle-modeling.md) du processus TDSP (Team Data science process). Pour en savoir plus sur le processus TDSP et le cycle de vie de la science des données, consultez [Qu’est-ce que le processus TDSP ?](overview.md)

## <a name="what-is-feature-engineering"></a>Qu’est-ce que l’ingénierie des caractéristiques ?

Les données d’apprentissage consistent en une matrice composée de lignes et de colonnes. Chaque ligne de la matrice est une observation ou un enregistrement. Les colonnes de chaque ligne sont les caractéristiques qui décrivent chaque enregistrement. Les caractéristiques spécifiées dans la conception expérimentale devraient caractériser les modèles dans les données.

Bien que la plupart des champs de données brutes peuvent être utilisés directement pour former un modèle, il est souvent nécessaire de créer des caractéristiques supplémentaires (conçues) pour un jeu de données d’apprentissage amélioré.

Les caractéristiques conçues améliorant l’apprentissage offrent des informations qui permettent de mieux différencier les modèles dans les données. Mais ce processus est tout un art. Des décisions réfléchies et productives nécessitent souvent une expertise.

## <a name="example-1-add-temporal-features-for-a-regression-model"></a>Exemple 1 : ajouter des caractéristiques temporelles pour un modèle de régression

Nous allons utiliser l’expérience [Demand forecasting of bikes rentals](https://gallery.azure.ai/Experiment/Regression-Demand-estimation-4) dans Azure Machine Learning Studio (classique) pour montrer comment concevoir des caractéristiques pour une tâche de régression. L’objectif de cette expérience est de prédire la demande de vélos en location pour un mois, un jour ou une heure spécifiques.

### <a name="bike-rental-dataset"></a>Jeu de données de location de vélo

Le jeu de données [Bike Rental UCI](http://archive.ics.uci.edu/ml/datasets/Bike+Sharing+Dataset/) est basé sur des données réelles d’une société de partage de vélos basée aux États-Unis. Il représente le nombre de locations de vélos au cours d’une heure spécifique d’une journée pour les années 2011 et 2012. Il contient 17 379 lignes et 17 colonnes.

L'ensemble des caractéristiques brutes contient des conditions météorologiques (température/humidité/vitesse du vent) et le type de jour (vacances/jour de semaine). Le champ pour la prédiction est le champ « count », qui représente le nombre de locations de vélo au cours d’une heure spécifique. Sa valeur comprise entre 1 et 977.

### <a name="create-a-feature-engineering-experiment"></a>Créer une expérience d’ingénierie de caractéristiques

Afin de construire des caractéristiques efficaces dans les données d'apprentissage, quatre modèles de régression sont générés à l'aide du même algorithme, mais avec quatre jeux de données d'apprentissage différents. Les quatre jeux de données représentent les mêmes données d'entrée brutes, mais avec un nombre croissant de jeux de caractéristiques. Ces caractéristiques sont regroupées en quatre catégories :

1. A = caractéristiques météo + vacances + jour de semaine + week-end pour le jour prévu
2. B = nombre de vélos loués au cours de chacune des 12 dernières heures
3. C = nombre de vélos loués au cours de chacun des 12 derniers jours à la même heure
4. D = nombre de vélos loués au cours de chacune des 12 dernières semaines le même jour à la même heure

Excepté l’ensemble de caractéristiques A, qui existe déjà dans les données brutes d’origine, les trois autres ensembles de caractéristiques sont créés via le processus de conception des caractéristiques. L'ensemble de caractéristiques B capture les toutes dernières demandes de vélos. L'ensemble de caractéristiques C capture la demande de vélos pour une heure en particulier. L'ensemble de caractéristiques D capture la demande de vélos pour une heure particulière et un jour de la semaine particulier. Chacun des quatre jeux de données d'apprentissage inclut respectivement un ensemble de caractéristiques A, A + B, A + B + C et A + B + C + D.

### <a name="feature-engineering-using-studio-classic"></a>ingénierie de caractéristiques à l’aide de Studio (classique)

Dans l’expérience de Studio (classique), ces quatre jeux de données d’apprentissage sont constitués de quatre branches provenant du jeu de données d’entrée traité au préalable. À l’exception de la branche la plus à gauche, chacune de ces branches contient un module [Exécuter le Script R](/azure/machine-learning/studio-module-reference/execute-r-script), dans lequel les caractéristiques dérivées (ensemble de caractéristiques B, C et D) sont construites et ajoutées au jeu de données importé.

La figure suivante montre le script R utilisé pour créer un ensemble de caractéristiques B dans la deuxième branche de gauche.

![création de caractéristiques](./media/create-features/addFeature-Rscripts.png)

### <a name="results"></a>Résultats

Une comparaison des résultats de performance des quatre modèles est résumée dans la table suivante : 

![comparaison des résultats](./media/create-features/result1.png)

Les caractéristiques A + B + C affichent les meilleurs résultats. Le taux d'erreur diminue lorsqu'un ensemble de caractéristiques supplémentaires est inclus dans les données d'apprentissage. Cela confirme l’idée présupposée que les ensembles de caractéristiques B et C fournissent des informations supplémentaires pertinentes pour la tâche de régression. Mais l'ajout de la caractéristique D semble ne pas fournir une réduction du taux d'erreur supplémentaire.

## <a name="example-2-create-features-for-text-mining"></a><a name="example2"></a> Exemple 2 : Créer des caractéristiques pour l’exploration de texte

L’ingénierie de caractéristiques s’applique largement aux tâches liées à l’exploration de texte, telles que la classification de document et l’analyse de sentiments. Comme des éléments individuels de texte brut servent habituellement de données d’entrée, le processus d’ingénierie de caractéristiques est nécessaire pour créer les caractéristiques impliquant des fréquences de mots ou d’expressions.

### <a name="feature-hashing"></a>Hachage des caractéristiques

Pour effectuer cette tâche, une technique appelée [hachage de caractéristiques](/azure/machine-learning/studio-module-reference/feature-hashing) est appliquée pour transformer efficacement les caractéristiques de texte arbitraires en index. Au lieu d’associer chaque caractéristique de texte (mots ou expressions) à un index particulier, cette méthode applique une fonction de hachage aux caractéristiques et utilise directement leurs valeurs de hachage en tant qu’index.

Dans Studio (classique), il existe un module [Hachage des caractéristiques](/azure/machine-learning/studio-module-reference/feature-hashing) qui crée en toute facilité des caractéristiques de mots ou expressions. La figure suivante montre un exemple d'utilisation de ce module. Le jeu de données d'entrée contient deux colonnes : l'évaluation du livre allant de 1 à 5 et le contenu même de la critique. L’objectif de ce module est de récupérer un ensemble de nouvelles caractéristiques qui montrent la fréquence d’occurrence des mots ou expressions correspondants dans cette critique de livre. Pour utiliser ce module, effectuez les étapes suivantes :

* Tout d'abord, sélectionnez la colonne qui contient le texte d'entrée (« Col2 » pour cet exemple).
* Ensuite, définissez le « nombre de bits de hachage » sur 8, ce qui signifie que 2 ^ 8 = 256 caractéristiques seront créées. Le mot ou l'expression sera haché en 256 index dans tout le texte. Le paramètre « hachage du nombre de bits » est compris entre 1 et 31. Les mots ou les phrases sont moins susceptibles d'être hachés dans le même index s'ils sont définis sur un nombre plus grand.
* Enfin, définissez le paramètre « N-grammes » sur 2. Cette valeur permet d’obtenir la fréquence d’occurrence d’unigrammes (une caractéristique pour chaque mot) et de bigrammes (une caractéristique pour chaque paire de mots juxtaposés) à partir du texte d’entrée. Le paramètre « N-grammes » est compris entre 0 et 10, ce qui indique le nombre maximum de mots séquentiels à inclure dans une caractéristique.  

![Module « hachage de caractéristiques »](./media/create-features/feature-Hashing1.png)

La figure suivante montre à quoi ressemblent ces nouvelles caractéristiques.

![Exemple « hachage de caractéristiques »](./media/create-features/feature-Hashing2.png)

## <a name="conclusion"></a>Conclusion
L’ingénierie et la sélection de caractéristiques augmentent l’efficacité du processus d’apprentissage qui tend à extraire les informations essentielles contenues dans les données. Ces processus améliorent également les performances de ces modèles pour classifier les données d'entrée avec précision et prédire les résultats pertinents de façon plus consistante.

L'ingénierie et la sélection de caractéristiques peuvent également être combinées afin de rendre l'apprentissage plus souple d'un point de vue informatique. Cela se fait grâce à l'amélioration puis à la réduction du nombre de caractéristiques nécessaires à l'étalonnage ou l'apprentissage d'un modèle. D’un point de vue mathématique, les caractéristiques sélectionnées sont un ensemble minimum de variables indépendantes qui expliquent les modèles des données, et prédisent correctement des résultats.

Il n’est pas toujours nécessaire d’effectuer une ingénierie de caractéristiques ou une sélection de caractéristiques. Cela dépend des données, de l’algorithme sélectionné et de l’objectif de l’expérience.

## <a name="next-steps"></a>Étapes suivantes

Pour créer des fonctionnalités pour les données dans des environnements spécifiques, consultez les articles suivants :

* [Créer des fonctionnalités pour les données dans SQL Server](create-features-sql-server.md)
* [Création de fonctionnalités pour les données dans un cluster Hadoop à l’aide de requêtes Hive](create-features-hive.md)