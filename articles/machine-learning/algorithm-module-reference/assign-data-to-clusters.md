---
title: 'Attribuer des données à des clusters : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Attribuer des données à des clusters dans Azure Machine Learning pour évaluer les modèles de clustering.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 11/19/2019
ms.openlocfilehash: e618395a2a4a11b4afb311d612cf5d0e27503dc6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90898858"
---
# <a name="module-assign-data-to-clusters"></a>Module : Attribuer des données à des clusters

Cet article décrit comment utiliser le module *Attribuer des données à des clusters* dans le concepteur Azure Machine Learning. Le module génère des prédictions via un modèle de clustering qui a été formé avec l’algorithme de *clustering k-moyennes*.

Le module Attribuer des données à des clusters retourne un jeu de données contenant les attributions probables de chaque nouveau point de données. 

## <a name="how-to-use-assign-data-to-clusters"></a>Comment utiliser le module Attribuer des données à des clusters
  
1. Dans le concepteur Azure Machine Learning, recherchez un modèle de clustering entraîné précédemment. Vous pouvez créer et former un modèle de clustering à l’aide d’une des méthodes suivantes :  
  
    - Configurer l’algorithme de clustering K-moyennes à l’aide du module [Clustering k-moyennes](k-means-clustering.md) et former le modèle à l’aide d’un jeu de données et du module Former le modèle de clustering (cet article).  
  
    - Vous pouvez également ajouter un modèle de clustering formé existant à partir du groupe **Modèles enregistrés** dans votre espace de travail.

2. Attacher le modèle formé au port d’entrée gauche du module **Attribuer des données à des clusters**.  

3. Attachez un nouveau jeu de données en tant qu’entrée. 

   Dans ce jeu de données, les étiquettes sont facultatives. En règle générale, le clustering est une méthode d’apprentissage non supervisée. Vous n’êtes pas censé connaître les catégories en avance. Toutefois, les colonnes d’entrée doivent être identiques aux colonnes utilisées lors de la formation du modèle de clustering. Dans le cas contraire, une erreur se produit.

    > [!TIP]
    > Pour réduire le nombre de colonnes rédigées dans le concepteur à partir des prédictions de cluster, utilisez [Sélectionner les colonnes dans le jeu de données](select-columns-in-dataset.md), puis sélectionnez un sous-ensemble de colonnes. 
    
4. Cochez la case **Cocher pour ajouter ou décocher pour résultats uniquement** si vous voulez que les résultats comprennent le jeu de données d’entrée entier, y compris une colonne qui affiche les résultats (attributions de cluster).
  
    Si vous décochez cette case, seuls les résultats sont retournés. Cette option peut être utile lorsque vous créez des prédictions dans le cadre d’un service web.
  
5.  Envoyez le pipeline.  
  
### <a name="results"></a>Résultats

+  Pour afficher les valeurs dans le jeu de données, cliquez avec le bouton droit sur le module, puis sélectionnez **Visualiser**. Vous pouvez aussi sélectionner le module et basculer vers l’onglet **Sorties** dans le panneau droit, cliquer sur l’icône d’histogramme dans **Port outputs** (Sorties de port) pour visualiser le résultat.

