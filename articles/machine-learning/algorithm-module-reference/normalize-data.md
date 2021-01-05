---
title: 'Normaliser des données : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Apprenez à utiliser le module Normaliser des données d’Azure Machine Learning afin de transformer un jeu de données par le biais de la *normalisation*.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/22/2020
ms.openlocfilehash: de0a23ca9dea210d91fe259b06622226549ba3b2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90890437"
---
# <a name="normalize-data-module"></a>Module Normaliser des données

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez ce module pour transformer un jeu de données par le biais de la *normalisation*.

La normalisation est une technique souvent appliquée dans le cadre de la préparation des données pour le Machine Learning. L'objectif de la normalisation est de modifier les valeurs des colonnes numériques du jeu de données pour utiliser une échelle commune, sans que les différences de plages de valeurs ne soient faussées et sans perte d'informations. Certains algorithmes ont également besoin d'une normalisation pour modéliser correctement les données.

Par exemple, supposons que votre jeu de données d'entrée contienne une colonne avec des valeurs allant de 0 à 1 et une autre colonne avec des valeurs allant de 10 000 à 100 000. La grande différence d'*échelle* des nombres peut poser problème lorsque vous essayez de combiner les valeurs sous forme de fonctions lors de la modélisation.

La *normalisation* permet d'échapper à ce type de problème en créant de nouvelles valeurs qui conservent la même distribution générale et les mêmes ratios que les données sources tout en appliquant la même échelle aux valeurs des différentes colonnes numériques utilisées dans le modèle.

Ce module offre plusieurs options pour transformer des données numériques :

- Vous pouvez modifier toutes les valeurs pour appliquer une échelle de 0 à 1, ou les transformer en les représentant sous forme de rangs centiles plutôt que sous forme de valeurs absolues.
- Vous pouvez appliquer la normalisation à une ou plusieurs colonnes d'un jeu de données.
- Si vous devez réitérer le pipeline ou appliquer les mêmes étapes de normalisation à d'autres données, vous pouvez enregistrer ces étapes sous forme de transformation de normalisation et appliquer celle-ci à d'autres jeux de données dotés du même schéma.

> [!WARNING]
> Avant de former un modèle, certains algorithmes exigent une normalisation des données. D'autres effectuent leur propre mise à l'échelle ou normalisation des données. Par conséquent, lorsque vous choisissez un algorithme de Machine Learning pour un modèle prédictif, examinez bien ses exigences en termes de données avant d'appliquer la normalisation aux données d'apprentissage.

##  <a name="configure-normalize-data"></a>Configurer le module Normaliser des données

Ce module ne vous permet d'appliquer qu'une seule méthode de normalisation à la fois. Par conséquent, la même méthode de normalisation est appliquée à toutes les colonnes que vous sélectionnez. Pour utiliser différentes méthodes de normalisation, utilisez une deuxième instance du module **Normaliser des données**.

1. Ajoutez le module **Normaliser des données** à votre pipeline. Vous le trouverez dans Azure Machine Learning, sous **Transformation de données**, dans la catégorie **Mise à l'échelle et réduction**.

2. Connectez un jeu de données contenant au moins une colonne de tous les nombres.

3. Utilisez le sélecteur de colonne pour choisir les colonnes numériques à normaliser. Si vous ne choisissez pas de colonnes individuelles, **toutes** les colonnes de type numérique de l'entrée sont incluses par défaut, et le même processus de normalisation s'applique à toutes les colonnes sélectionnées. 

    Des résultats étranges peuvent être obtenus si vous incluez des colonnes numériques qui ne devraient pas être normalisées ! Vérifiez toujours les colonnes avec soin.

    Si aucune colonne numérique n'est détectée, vérifiez les métadonnées de la colonne pour vous assurer que le type de données de celle-ci est un type numérique pris en charge.

    > [!TIP]
    > Pour vous assurer que les colonnes d'un type spécifique sont fournies comme entrée, essayez d'utiliser le module [Sélectionner des colonnes dans un jeu de données](./select-columns-in-dataset.md) avant le module **Normaliser des données**.

4. **Utiliser 0 pour les colonnes constantes quand activé** :  sélectionnez cette option lorsqu'une colonne numérique contient une seule valeur immuable. Cette colonne ne sera ainsi pas utilisée lors des opérations de normalisation.

5. Dans la liste déroulante **Méthode de transformation**, choisissez la fonction mathématique à appliquer à toutes les colonnes sélectionnées. 
  
    - **Zscore** : convertit toutes les valeurs en Z-score.
    
      Les valeurs de la colonne sont transformées à l'aide de la formule suivante :  
  
      ![normalisation à l'aide de z-scores](media/module/aml-normalization-z-score.png)
  
      La moyenne et l'écart type sont calculés séparément pour chaque colonne. L'écart type de population est utilisé.
  
    - **MinMax** : le normalisateur Min-Max redimensionne linéairement chaque fonction à l'intervalle [0,1].
    
      Le redimensionnement à l'intervalle [0,1] s'effectue en décalant les valeurs de chaque fonction afin que la valeur minimale soit 0, puis en divisant par la nouvelle valeur maximale (qui correspond à la différence entre les valeurs maximale et minimale d'origine).
      
      Les valeurs de la colonne sont transformées à l'aide de la formule suivante :  
  
      ![normalisation à l'aide de la fonction min&#45;max](media/module/aml-normalization-minmax.png "AML_normalization-minmax")  
  
    - **Logistic** : Les valeurs de la colonne sont transformées à l'aide de la formule suivante :

      ![formule de normalisation par fonction logistique](media/module/aml-normalization-logistic.png "AML_normalization-logistic")  
  
    - **LogNormal** : cette option convertit toutes les valeurs à une échelle log-normale.
  
      Les valeurs de la colonne sont transformées à l'aide de la formule suivante :
  
      ![distribution &#45;normal de la formule](media/module/aml-normalization-lognormal.png "AML_normalization-lognormal")
    
      Ici, μ et σ correspondent aux paramètres de la distribution, calculés empiriquement à partir des données sous forme d'estimations de la probabilité maximale, pour chaque colonne séparément.  
  
    - **TanH** : toutes les valeurs sont converties en tangente hyperbolique.
    
      Les valeurs de la colonne sont transformées à l'aide de la formule suivante :
    
      ![normalisation à l'aide de la fonction tanh](media/module/aml-normalization-tanh.png "AML_normalization-tanh")

6. Envoyez le pipeline ou double-cliquez sur le module **Normaliser des données** et sélectionnez **Exécuter la sélection**. 

## <a name="results"></a>Résultats

Le module **Normaliser des données** génère deux sorties :

- Pour afficher les valeurs transformées, cliquez avec le bouton droit sur le module, puis sélectionnez **Visualiser**.

    Par défaut, les valeurs sont transformées sur place. Si vous souhaitez comparer les valeurs transformées avec les valeurs d'origine, utilisez le module [Ajouter des colonnes](./add-columns.md) pour recombiner les jeux de données et afficher les colonnes côte à côte.

- Pour enregistrer la transformation afin de pouvoir appliquer la même méthode de normalisation à un autre jeu de données, sélectionnez le module, puis **Inscrire le jeu de données** sous l’onglet **Sorties** dans le panneau droit.

    Vous pouvez ensuite charger les transformations enregistrées à partir du groupe **Transformations** du volet de navigation de gauche et les appliquer à un jeu de données doté du même schéma à l’aide de [Appliquer une transformation](apply-transformation.md).  


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 