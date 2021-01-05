---
title: 'Modèle de validation croisée : sur le module Modifier les métadonnées'
titleSuffix: Azure Machine Learning
description: Utilisez le module Modèle de validation croisée dans le concepteur Azure Machine Learning afin d’effectuer une validation croisée des estimations de paramètre pour les modèles de classification ou de régression.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/11/2020
ms.openlocfilehash: d4099ecf6e6bcc6654391e54292878393fb22914
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421343"
---
# <a name="cross-validate-model"></a>Modèle de validation croisée

Cet article décrit comment utiliser le module Modèle de validation croisée dans le concepteur Azure Machine Learning. La *validation croisée* est une technique souvent utilisée en machine learning pour évaluer la variabilité d’un jeu de données et la fiabilité de tout modèle entraîné avec ces données.  

Le module Modèle de validation croisée prend comme entrée un jeu de données étiqueté ainsi qu’un modèle de classification ou de régression non entraîné. Il divise le jeu de données en un certain nombre de sous-ensembles (*échantillons*), crée un modèle sur chaque échantillon, puis retourne un jeu de statistiques de justesse pour chaque échantillon. En comparant les statistiques de justesse pour tous les replis, vous pouvez interpréter la qualité du jeu de données. Vous pouvez alors déterminer si le modèle est sensible aux variations des données.  

Le modèle de validation croisée retourne également des résultats prédits et des probabilités pour le jeu de données, afin que vous puissiez évaluer la fiabilité des prédictions.  

### <a name="how-cross-validation-works"></a>Fonctionnement de la validation croisée

1. La validation croisée divise de façon aléatoire les données d’entraînement en replis. 

   L’algorithme se base par défaut sur 10 échantillons si vous n’avez pas déjà partitionné le jeu de données. Pour diviser le jeu de données en un nombre différent d’échantillons, vous pouvez utiliser le module [Partition et échantillon](partition-and-sample.md) et indiquer le nombre d’échantillons à utiliser.  

2.  Le module met de côté les données du repli 1 à utiliser pour la validation. (Ce repli est parfois appelé *repli d’exclusion*.) Le module utilise les replis restants pour entraîner un modèle. 

    Par exemple, si vous créez cinq replis, le module génère cinq modèles au cours de la validation croisée. Le module entraîne chaque modèle en utilisant quatre cinquièmes des données. Il teste chaque modèle sur le cinquième restant.  

3.  Lors du test du modèle pour chaque repli, le module évalue plusieurs statistiques de justesse. Les statistiques que le module utilise dépendent du type de modèle en cours d’évaluation. Différentes statistiques sont utilisées pour évaluer les modèles de classification par rapport aux modèles de régression.  

4.  Quand le processus de génération et d’évaluation est terminé pour tous les replis, le modèle de validation croisée génère un ensemble de métriques de performances et de résultats notés pour toutes les données. Passez en revue ces métriques pour déterminer si le moindre repli a une justesse élevée ou faible. 

### <a name="advantages-of-cross-validation"></a>Avantages de la validation croisée

Une méthode différente et courante d’évaluation d’un modèle consiste à diviser les données en un jeu d’entraînement et de test en utilisant [Fractionner les données](split-data.md), puis à valider le modèle sur les données d’entraînement. Toutefois, la validation croisée offre certains avantages :  

-   La validation croisée utilise davantage de données de test.

    La validation croisée mesure les performances du modèle avec les paramètres spécifiés dans un espace de données plus grand. Autrement dit, la validation croisée utilise la totalité du jeu de données d’entraînement pour l’entraînement et l’évaluation, au lieu d’une partie seulement. En revanche, si vous validez un modèle à l’aide de données générées à partir d’un découpage aléatoire, vous évaluez généralement le modèle sur seulement 30 % ou moins des données disponibles.  

    Toutefois, comme la validation croisée entraîne et valide le modèle plusieurs fois sur un jeu de données plus grand, elle est beaucoup plus gourmande en ressources de calcul. Elle prend beaucoup plus de temps que la validation sur un découpage aléatoire.  

-   La validation croisée évalue le jeu de données et le modèle.

    La validation croisée ne mesure pas simplement la justesse d’un modèle. Elle donne également une idée du degré de représentativité du jeu de données et de la sensibilité du modèle aux variations des données.  

## <a name="how-to-use-cross-validate-model"></a>Comment utiliser le modèle de validation croisée

L’exécution de la validation croisée peut prendre beaucoup de temps si votre jeu de données est volumineux.  Par conséquent, vous pouvez utiliser le modèle de validation croisée dans la phase initiale de création et de test de votre modèle. Dans cette phase, vous pouvez évaluer l’adéquation des paramètres du modèle (en supposant que le temps de calcul est acceptable). Vous pouvez ensuite entraîner et évaluer votre modèle en utilisant les paramètres établis avec les modules [Entraîner un modèle](train-model.md) et [Évaluer un modèle](evaluate-model.md).

Dans ce scénario, vous entraînez et testez le modèle à l’aide du modèle de validation croisée.

1. Ajoutez le module Modèle de validation croisée à votre pipeline. Vous pouvez le trouver dans le concepteur Azure Machine Learning, dans la catégorie **Scoring & évaluation de modèle**. 

2. Connectez la sortie de n’importe quel modèle de classification ou de régression. 

    Par exemple, si vous utilisez **Arbre de décision optimisé à deux classes** pour la classification, configurez le modèle avec les paramètres de votre choix. Ensuite, faites glisser un connecteur du port **Modèle non entraîné** du classifieur vers le port correspondant du modèle de validation croisée. 

    > [!TIP] 
    > Vous n’avez pas besoin d’entraîner le modèle, car le modèle de validation croisée l’entraîne automatiquement dans le cadre de l’évaluation.  
3.  Sur le port **Jeu de données** du modèle de validation croisée, connectez tout jeu de données d’entraînement étiqueté.  

4.  Dans le volet droit de Cross Validate Model, cliquez sur **Modifier la colonne**. Sélectionnez la colonne unique qui contient l’étiquette de classe ou la valeur prévisible. 

5. Définissez une valeur pour le paramètre **Random seed** (Valeur de départ aléatoire) si vous souhaitez répéter les résultats de la validation croisée entre des exécutions successives sur les mêmes données.  

6. Envoyez le pipeline.

7. Pour obtenir une description des rapports, consultez la section [Résultats](#results).

## <a name="results"></a>Résultats

Une fois toutes les itérations terminées, le modèle de validation croisée crée des scores pour le jeu de données complet. Il crée également des métriques de performances que vous pouvez utiliser pour évaluer la qualité du modèle.

### <a name="scored-results"></a>Résultats notés

La première sortie du module fournit les données sources pour chaque ligne ainsi que des valeurs prédites et des probabilités associées. 

Pour voir les résultats, dans le pipeline, cliquez avec le bouton droit sur le module Modèle de validation croisée. Sélectionnez **Visualize Scored results** (Visualiser les résultats notés).

| Nouveau nom de colonne      | Description                              |
| -------------------- | ---------------------------------------- |
| Étiquettes notées        | Cette colonne est ajoutée à la fin du jeu de données. Elle contient la valeur prédite pour chaque ligne. |
| Probabilités notées | Cette colonne est ajoutée à la fin du jeu de données. Elle indique la probabilité estimée de la valeur dans **Scored Labels** (Étiquettes notées). |
| Nombre d’échantillons          | Indique l’index de base 0 du repli auquel chaque ligne de données a été attribuée lors de la validation croisée. |

 ### <a name="evaluation-results"></a>Evaluation results

Le deuxième rapport est regroupé par échantillons. N’oubliez pas que, pendant l’exécution, le modèle de validation croisée divise de façon aléatoire les données d’entraînement en *n* replis (par défaut, 10). Dans chaque itération sur le jeu de données, le modèle de validation croisée utilise un repli comme jeu de données de validation. Il utilise les *n-1* replis restants pour entraîner un modèle. Chacun des *n* modèles est testé par rapport aux données de tous les autres échantillons.

Dans ce rapport, les échantillons sont listés par valeur d’index, dans l’ordre croissant.  Pour effectuer un tri sur une autre colonne, vous pouvez enregistrer les résultats sous la forme d’un jeu de données.

Pour voir les résultats, dans le pipeline, cliquez avec le bouton droit sur le module Modèle de validation croisée. Sélectionnez **Visualize Evaluation results by fold** (Visualiser les résultats de l’évaluation par échantillon).


|Nom de la colonne| Description|
|----|----|
|Nombre d’échantillons| Identificateur pour chaque échantillon. Si vous avez créé cinq replis, vous devez avoir cinq sous-ensembles de données, numérotés de 0 à 4.
|Nombre d’exemples dans l’échantillon|Nombre de lignes affectées à chaque échantillon. Ils doivent être à peu près égaux. |


Le module inclut aussi les métriques suivantes pour chaque repli, en fonction du type de modèle que vous évaluez : 

+ **Modèles de classification** : précision, rappel, F-score, AUC, justesse  

+ **Modèles de régression** : erreur d’absolue moyenne, erreur carrée moyenne racine, erreur d’absolue relative, erreur carrée relative et coefficient de détermination


## <a name="technical-notes"></a>Notes techniques  

+ Une bonne pratique consiste à normaliser les jeux de données avant de les utiliser pour la validation croisée. 

+ Le modèle de validation croisée est beaucoup plus gourmand en calculs et prend plus de temps que si vous validiez le modèle à l’aide d’un jeu de données divisé de façon aléatoire. Cela est dû au fait que le modèle de validation croisée entraîne et valide le modèle plusieurs fois.

+ Il n’est pas nécessaire de diviser le jeu de données en jeux d’entraînement et de test quand vous utilisez la validation croisée pour mesurer la justesse du modèle. 


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 

