---
title: 'Entraîner un modèle de clustering : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Train Clustering Model (Entraîner un modèle de clustering) dans Azure Machine Learning pour entraîner des modèles de clustering.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 11/19/2019
ms.openlocfilehash: 0dfde6fef7cc91edf2101095018bc031d392c4ed
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90898172"
---
# <a name="train-clustering-model"></a>Entraîner un modèle de clustering

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez ce module pour entraîner un modèle de clustering.

Le module prend un modèle de clustering non entraîné que vous avez déjà configuré à l’aide du module [Clustering k-moyennes](k-means-clustering.md) et l’entraîne à l’aide d’un jeu de données étiqueté ou sans étiquette. Le module crée à la fois un modèle entraîné que vous pouvez utiliser à des fins de prédiction et un ensemble d’affectations de cluster pour chaque cas dans les données d’apprentissage.

> [!NOTE]
> Il est impossible d’entraîner un modèle de clustering à l’aide du module [Train Model](train-model.md) (Entraîner le modèle), qui est le module générique pour l’entraînement des modèles d’apprentissage automatique. Cela est dû au fait que le module [Train Model](train-model.md) (Entraîner le modèle) fonctionne uniquement avec les algorithmes d’apprentissage supervisé. L’algorithme de clustering k-moyennes et les autres algorithmes de clustering permettent un apprentissage non supervisé, ce qui signifie que l’algorithme peut apprendre à partir de données sans étiquette.  
  
## <a name="how-to-use-train-clustering-model"></a>Entraînement d’un modèle de clustering  

1.  Ajoutez le module **Train Clustering Model** (Entraîner un modèle de clustering) à votre pipeline dans le concepteur. Vous trouverez ce module sous **Modules Machine Learning**, dans la catégorie **Entraîner**.  
  
2. Ajouter le module [Clustering k-moyennes](k-means-clustering.md) ou un autre module personnalisé qui crée un modèle de clustering compatible, puis définissez les paramètres du modèle de clustering.  
    
3.  Joignez un jeu de données d’entraînement à l’entrée à droite du module **Train Clustering Model** (Entraîner un modèle de clustering).
  
5.  Dans **Jeu de colonnes**, sélectionnez les colonnes du jeu de données à utiliser lors de la création de clusters. Veillez à sélectionner les colonnes qui constituent de bonnes fonctionnalités : par exemple, évitez d’utiliser les ID ou d’autres colonnes qui ont des valeurs uniques ou des colonnes qui ont toutes les mêmes valeurs.

    Si une étiquette est disponible, vous pouvez l’utiliser comme une fonctionnalité ou l’ignorer.  
  
6. Sélectionnez l’option **Check for Append or Uncheck for Result Only** (Cocher pour ajouter ou décocher pour les résultats uniquement) si vous souhaitez exporter les données d’apprentissage avec la nouvelle étiquette de cluster.

    Si vous désélectionnez cette option, seuls les affectations de cluster sont générées. 

7. Envoyez le pipeline ou cliquez sur le module **Effectuer l’apprentissage un modèle de clustering** et sélectionnez **Exécuter la sélection**.  
  
### <a name="results"></a>Résultats

Une fois l’apprentissage terminé :

+ Pour enregistrer un instantané du modèle entraîné, sélectionnez l’onglet **Sorties** dans le panneau droit du module **Entraîner le modèle**. Sélectionnez l’icône **Inscrire le jeu de données** pour enregistrer le modèle en tant que module réutilisable.

+ Pour générer des scores à partir du modèle, utilisez l’option [Attribuer des données à des clusters](assign-data-to-clusters.md).

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 