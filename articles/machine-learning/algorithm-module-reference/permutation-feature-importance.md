---
title: 'Importance de la fonctionnalité de permutation : sur le module Modifier les métadonnées'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Importance de la fonctionnalité de permutation dans le concepteur pour calculer les scores d’importance des fonctionnalités de permutation des variables de fonctionnalité.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/24/2020
ms.openlocfilehash: 8ae1e79922cc0f34e8b2d1f253fce5078df286d2
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421241"
---
# <a name="permutation-feature-importance"></a>Importance de la fonctionnalité de permutation

Cet article explique comment utiliser le module Importance de la fonctionnalité de permutation dans le concepteur Azure Machine Learning, afin de calculer un ensemble de scores d’importance de fonctionnalité pour votre jeu de données. Vous utilisez ces scores pour vous aider à déterminer les meilleures fonctionnalités à utiliser dans un modèle.

Dans ce module, les valeurs des fonctionnalités sont mélangées aléatoirement, une colonne à la fois. Les performances du modèle sont mesurées avant et après. Vous pouvez choisir l’une des métriques standard pour mesurer les performances.

Les scores retournés par le module représentent la *modification* dans les performances d’un modèle formé, après permutation. Les fonctionnalités importantes sont généralement plus sensibles au processus de mélange. Elles génèrent donc des scores d’importance plus élevés. 

Cet article fournit une vue d’ensemble de la fonctionnalité de permutation, de sa base théorique et de ses applications dans le machine learning : [Importance de la fonctionnalité de permutation](/archive/blogs/machinelearning/permutation-feature-importance).  

## <a name="how-to-use-permutation-feature-importance"></a>Comment utiliser l’importance de la fonctionnalité de permutation

La génération d’un ensemble de scores de fonctionnalités exige que vous disposiez d’un modèle déjà entraîné, ainsi que d’un jeu de données de test.  

1.  Ajoutez le module Importance de la fonctionnalité de permutation à votre pipeline. Ce module figure dans la catégorie **Sélection des caractéristiques**. 

2.  Connectez un modèle formé à l’entrée gauche. Le modèle doit être un modèle de régression ou un modèle de classification.  

3.  Sur l’entrée droite, connectez un jeu de données. De préférence, choisissez-en un qui soit différent du jeu de données utilisé pour l’entraînement du modèle. Ce jeu de données est utilisé pour réaliser un scoring basé sur le modèle entraîné. Il est également utilisé pour évaluer le modèle une fois que les valeurs des fonctionnalités ont changé.  

4.  Pour **Random seed** (Valeur de départ aléatoire), entrez une valeur à utiliser comme valeur de départ pour la répartition aléatoire. Si vous spécifiez 0 (valeur par défaut), un nombre est généré en fonction de l’horloge système.

     Une valeur de départ est facultative, mais vous devez fournir une valeur si vous souhaitez que la reproductibilité s’effectue entre les exécutions du même pipeline.  

5.  Pour **Métrique pour la mesure des performances**, sélectionnez une métrique unique à utiliser lors du calcul de la qualité du modèle après la permutation.  

     Le concepteur Azure Machine Learning prend en charge les métriques suivantes, selon que vous évaluez un modèle de classification ou de régression :  

    -   **Classification**

        Exactitude, Précision, Rappel  

    -   **Régression**

        Précision, Rappel, Erreur d’absolue moyenne, Erreur carrée moyenne racine, Erreur d’absolue relative, Erreur carrée relative, Coefficient de détermination  

     Pour obtenir une description plus détaillée de ces métriques d’évaluation et de la façon dont elles sont calculées, consultez [Évaluer le modèle](evaluate-model.md).  

6.  Envoyez le pipeline.  

7.  Le module génère une liste de colonnes de fonctionnalités et les scores qui leur sont associés. Cette liste est classée dans l’ordre décroissant des scores.  


##  <a name="technical-notes"></a>Notes techniques

L’importance de la fonctionnalité de permutation consiste à modifier de façon aléatoire les valeurs de chaque colonne de fonctionnalité, une colonne à la fois. Elle évalue ensuite le modèle. 

Les classements fournis par le module sont souvent différents de ceux que vous obtenez du module [Sélection de caractéristiques par filtrage](filter-based-feature-selection.md). La sélection de caractéristiques par filtrage calcule les scores *avant* la création d’un modèle. 

La raison de cette différence tient au fait que le module Importance de la fonctionnalité de permutation ne mesure pas l’association entre une caractéristique et une valeur cible. Au lieu de cela, il capture l’influence de chaque caractéristique sur les prédictions du modèle.
  
## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning.