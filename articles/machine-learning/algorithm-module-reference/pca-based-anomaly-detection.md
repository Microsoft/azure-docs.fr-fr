---
title: 'Détection d’anomalie reposant sur l’ACP : sur le module Modifier les métadonnées'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Détection d’anomalie reposant sur l’ACP pour créer un modèle de détection d’anomalie basé sur l’analyse en composantes principales (ACP).
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/22/2020
ms.openlocfilehash: fa90fcb9ebc17be9a658b08873234eada98b0fba
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90898460"
---
# <a name="pca-based-anomaly-detection-module"></a>Détection d’anomalie reposant sur l’ACP

Cet article explique comment utiliser le module Détection d’anomalie reposant sur l’ACP dans le concepteur Azure Machine Learning pour créer un modèle de détection d’anomalie basé sur l’analyse en composantes principales (ACP).

Ce module vous aide à créer un modèle dans des scénarios où il est facile d’obtenir des données d’entraînement à partir d’une classe, telles que des transactions valides, mais difficile d’obtenir suffisamment d’exemples d’anomalies ciblées. 

Par exemple, pour détecter des transactions frauduleuses, vous n’avez souvent pas assez d’exemples de fraudes pour effectuer un entraînement. Mais vous pouvez avoir de nombreux exemples de bonnes transactions. Le module Détection d’anomalie reposant sur l’ACP résout le problème en analysant les fonctionnalités disponibles pour déterminer ce qui constitue une classe « normale ». Le module applique ensuite des mesures de distance pour identifier les cas qui constituent des anomalies. Cette approche vous permet d’entraîner un modèle à l’aide de données déséquilibrées existantes.

## <a name="more-about-principal-component-analysis"></a>Informations supplémentaires sur l’analyse en composantes principales

L’ACP est une technique établie dans le domaine du Machine Learning. Elle est fréquemment utilisée dans l’analyse exploratoire des données, car elle révèle la structure interne des données et explique la variance dans les données.

L’ACP fonctionne en analysant les données qui contiennent plusieurs variables. Elle recherche les corrélations entre les variables et détermine la combinaison des valeurs qui capturent le mieux les différences dans les résultats. Ces valeurs de fonctionnalités combinées sont utilisées pour créer un espace de fonctionnalités plus compact appelé les *composantes principales*.

Pour la détection d’anomalie, chaque nouvelle entrée est analysée. L’algorithme de détection d’anomalie calcule sa projection sur les vecteurs propres, ainsi qu’une erreur de reconstruction normalisée. L’erreur normalisée est utilisée comme score d’anomalie. Plus l’erreur est élevée, plus l’instance est anormale.

Pour plus d’informations sur le fonctionnement de l’ACP et sur l’implémentation de la détection d’anomalie, consultez ces articles :

- [A randomized algorithm for principal component analysis](https://arxiv.org/abs/0809.2274), par Rokhlin, Szlan et Tygert

- [Finding Structure with Randomness: Probabilistic Algorithms for Constructing Approximate Matrix Decompositions](http://users.cms.caltech.edu/~jtropp/papers/HMT11-Finding-Structure-SIREV.pdf) (téléchargement PDF), par Halko, Martinsson et Tropp

## <a name="how-to-configure-pca-based-anomaly-detection"></a>Comment configurer la détection d’anomalie reposant sur l’ACP

1. Ajoutez le module **Détection d’anomalie reposant sur l’ACP** à votre pipeline dans le concepteur. Ce module figure dans la catégorie **Détection d’anomalie**.

2. Dans le volet droit du module, sélectionnez l’option**Mode d’entraînement**. Indiquez si vous souhaitez entraîner le modèle à l’aide d’un ensemble spécifique de paramètres ou utiliser un balayage de paramètre pour rechercher les meilleurs paramètres.

    Si vous savez comment vous voulez configurer le modèle, sélectionnez l’option **Paramètre unique** et fournissez un ensemble spécifique de valeurs en tant qu’arguments.

3. Pour **Nombre de composants à utiliser dans l’ACP**, spécifiez le nombre souhaité de fonctionnalités de sortie ou de composantes.

    Le choix du nombre de composantes à inclure est une partie importante de la conception d’expérimentation qui utilise l’ACP. D’une manière générale, vous ne devez pas inclure le même nombre de composantes ACP que de variables. Au lieu de cela, vous devez commencer avec un plus petit nombre de composantes et les augmenter jusqu’à ce qu’un certain critère soit rempli.

    Les meilleurs résultats sont obtenus lorsque le nombre de composantes de sortie est *inférieur* au nombre de colonnes de fonctionnalités disponibles dans le jeu de données.

4. Spécifiez le nombre de suréchantillonnages à effectuer lors de la formation de l’ACP aléatoire. Dans les problèmes de détection d’anomalie, les données déséquilibrées compliquent l’application des techniques ACP standard. En spécifiant une quantité de suréchantillonnage, vous pouvez augmenter le nombre d’instances cibles.

    Si vous spécifiez **1**, aucun suréchantillonnage n’est effectué. Si vous spécifiez une valeur supérieure à **1**, des échantillons supplémentaires sont générés pour être utilisés dans l’entraînement du modèle.

    Il existe deux options, selon que vous utilisez ou non un balayage des paramètres :

    - **Paramètre de suréchantillonnage pour l’ACP aléatoire** : Saisissez un nombre entier unique qui représente le rapport entre le suréchantillonnage de la classe minoritaire et la classe normale. (Cette option est disponible lors de l’utilisation de la méthode d’entraînement **Paramètre unique**.)

    > [!NOTE]
    > Vous ne pouvez pas afficher le jeu de données suréchantillonné. Pour plus d’informations sur l’utilisation du suréchantillonnage avec l’ACP, consultez les [notes techniques](#technical-notes).

5. Sélectionnez l’option **Activer la normalisation des moyennes des fonctionnalités d’entrée** pour normaliser toutes les fonctionnalités d’entrée avec une moyenne de zéro. La normalisation ou la mise à l’échelle sur zéro est généralement recommandée pour l’ACP, car l’objectif de cette dernière est d’optimiser l’écart entre les variables.

    Cette option est activée par défaut. Désélectionnez cette option si les valeurs ont déjà été normalisées à l’aide d’une méthode ou mise à l’échelle différente.

6. Connectez un jeu de données d’entraînement balisé à l’un des modules d’entraînement.

   Si vous définissez l’option **Créer un mode d’apprentissage** sur **Paramètre unique**, utilisez le module [Entraîner le modèle de détection d’anomalie](train-anomaly-detection-model.md).

7. Envoyez le pipeline.

## <a name="results"></a>Résultats

Une fois l’entraînement terminé, vous pouvez enregistrer le modèle entraîné. Ou vous pouvez le connecter au module [Noter le modèle](score-model.md) pour prédire les scores d’anomalie.

Pour évaluer les résultats d’un modèle de détection d’anomalie :

1. Vérifiez qu’une colonne de score est disponible dans les deux jeux de données.

    Si vous essayez d’évaluer un modèle de détection d’anomalie et que vous obtenez l’erreur « Il n’y a pas de colonne de score dans le jeu de données noté à comparer », vous utilisez un jeu de données d’évaluation classique qui contient une colonne d’étiquette, mais pas de score de probabilité. Choisissez un jeu de données qui correspond à la sortie de schéma pour les modèles de détection d’anomalie, qui comprend les colonnes **Étiquettes notées** et **Probabilités notées**.

2. Vérifiez que les colonnes d’étiquette sont marquées.

    Parfois, les métadonnées associées à la colonne d’étiquette sont supprimées dans le graphique de pipeline. Dans ce cas, quand vous utilisez le module [Évaluer le modèle](evaluate-model.md) pour comparer les résultats de deux modèles de détection d’anomalie, vous pouvez obtenir l’erreur « Aucune colonne d’étiquette dans le jeu de données noté ». Vous pouvez également obtenir l’erreur « Il n’y a pas de colonne d’étiquette dans le jeu de données noté à comparer ».

    Vous pouvez éviter ces erreurs en ajoutant le module [Modifier les métadonnées](edit-metadata.md) avant le module [Évaluer le modèle](evaluate-model.md). Utilisez le sélecteur de colonne pour choisir la colonne de classe, puis dans la liste **Champs**, sélectionnez **Étiquette**.

3. Utilisez le module [Exécuter le script Python](execute-python-script.md) pour ajuster les catégories de colonne d’étiquette à **1 (positif, normal)** et **0 (négatif, anormal)** .

    ````
    label_column_name = 'XXX'
    anomaly_label_category = YY
    dataframe1[label_column_name] = dataframe1[label_column_name].apply(lambda x: 0 if x == anomaly_label_category else 1)
    ````

    
## <a name="technical-notes"></a>Notes techniques

Cet algorithme utilise l’ACP pour se rapprocher du sous-espace contenant la classe normale. Le sous-espace est fractionné par vecteurs propres associés au valeurs propres supérieures de la matrice de covariance des données. 

Pour chaque nouvelle entrée, le détecteur d’anomalies calcule d’abord sa projection sur les vecteurs propres avant de calculer l’erreur de reconstruction normalisée. Cette erreur représente le score d’anomalie. Plus l’erreur est élevée, plus l’instance est anormale. Pour plus d’informations sur la façon dont l’espace normal est calculé, consultez la page Wikipédia : [Analyse en composantes principales](https://wikipedia.org/wiki/Principal_component_analysis). 


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 

Pour obtenir la liste des erreurs spécifiques aux modules du concepteur, consultez [Exceptions et codes d’erreur du concepteur](designer-error-codes.md).