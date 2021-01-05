---
title: 'Entraîner le modèle : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module **Entraîner le modèle** dans Azure Machine Learning pour effectuer l’apprentissage d’un modèle de classification ou de régression.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 11/25/2020
ms.openlocfilehash: 7063452d23d2975cf0c26a89e7a08a422de54942
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96751935"
---
# <a name="train-model-module"></a>Module de formation de modèle

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez ce module pour effectuer l’apprentissage d’un modèle de classification ou de régression. La formation a lieu une fois que vous avez défini un modèle et ses paramètres et nécessite des données avec balises. Vous pouvez également utiliser le module **Train Model** (Entraîner le modèle) pour effectuer l’apprentissage d’un modèle existant avec de nouvelles données. 

## <a name="how-the-training-process-works"></a>Fonctionnement du processus de formation

Dans Azure Machine Learning, la création et l’utilisation d’un modèle Machine Learning s’effectuent généralement en trois étapes. 

1. Vous configurez un modèle en choisissant un type d’algorithme particulier et en définissant ses paramètres ou hyperparamètres. Choisissez l’un des types de modèles suivants : 

    + Modèles de **classification**, basée sur les réseaux neuronaux, arbres de décision, forêts de décision et autres algorithmes
    + Modèles de **régression**, qui peuvent inclure une régression linéaire standard ou qui utilisent d’autres algorithmes, notamment les réseaux neuronaux et la régression bayésienne  

2. Fournissez un jeu de données doté d’une étiquette et dont les données sont compatibles avec l’algorithme. Connectez les données et le modèle au module **Train Model** (Entraîner le modèle).

    La formation produit un format binaire spécifique, l’iLearner, qui encapsule les modèles statistiques appris à partir des données. Vous ne pouvez pas modifier ou lire ce format directement. Toutefois, les autres modules peuvent utiliser ce modèle formé. 
    
    Vous pouvez également afficher les propriétés du modèle. Pour plus d’informations, consultez la section Résultats.

3. Une fois la formation terminée, utilisez le modèle formé avec l’un des [modules de notation](./score-model.md) pour effectuer des prédictions sur de nouvelles données.

## <a name="how-to-use-train-model"></a>Comment utiliser le module Effectuer l’apprentissage du modèle 
    
1. Ajoutez le module **Effectuer l'apprentissage du modèle** au pipeline.  Ce module figure dans la catégorie **Machine Learning**. Développez la section **Effectuer l'apprentissage**, puis faites glisser le module **Effectuer l'apprentissage du modèle** dans votre pipeline.
  
1.  Sous l’entrée de gauche, joignez le modèle non formé. Joignez le jeu de données d’apprentissage à l’entrée droite du module **Train Model** (Entraîner le modèle).

    Le jeu de données d’apprentissage doit contenir une colonne d’étiquette. Toutes les lignes sans étiquette sont ignorées.
  
1.  Pour **Étiqueter la colonne**, cliquez sur **Modifier la colonne** dans le volet droit du module, puis choisissez une seule colonne contenant les résultats que le modèle peut utiliser pour la formation.
  
    - Pour les problèmes de classification, la colonne d’étiquette doit contenir des valeurs **catégorielles** ou **discrètes**. Il peut s’agir, par exemple, d’une évaluation oui/non, d’un code ou nom de classification de maladie ou d’un groupe de revenus.  Si vous choisissez une colonne non catégorielle, le module renverra une erreur pendant la formation.
  
    -   Pour les problèmes de régression, la colonne d’étiquette doit contenir des données **numériques** qui représentent la variable de réponse. Dans l’idéal, les données numériques représentent une échelle continue. 
    
    Il peut s’agir, par exemple, de l’évaluation du risque de crédit, du temps entre deux pannes escompté pour un disque dur ou du nombre prévu d’appels à un centre d’appels sur une journée ou à une heure donnée.  Si vous ne choisissez pas une colonne numérique, une erreur peut se produire.
  
    -   Si vous ne spécifiez pas la colonne d’étiquette à utiliser, Azure Machine Learning tente de déduire la colonne appropriée en utilisant les métadonnées du jeu de données. S’il choisit la mauvaise colonne, utilisez le sélecteur de colonne pour résoudre ce problème.
  
    > [!TIP] 
    > Si vous avez des difficultés à utiliser le sélecteur de colonne, consultez l’article [Sélectionner des colonnes dans le jeu de données](./select-columns-in-dataset.md) pour obtenir des conseils. Il décrit quelques scénarios courants et fournit des conseils pour utiliser les options **WITH RULES** (Avec règles) et **BY NAME** (Par nom).
  
1.  Envoyez le pipeline. Si vous avez une grande quantité de données, cela peut prendre un certain temps.

    > [!IMPORTANT] 
    > Si vous avez une colonne d’ID qui correspond à l’ID de chaque ligne, ou une colonne de texte, qui contient trop de valeurs uniques, le module **Entraîner le modèle** peut retourner une erreur telle que « Le nombre de valeurs uniques dans la colonne : « {nom_colonne} » est supérieur à la valeur autorisée ».
    >
    > Cela est dû au fait que la colonne a atteint le seuil de valeurs uniques, ce qui peut entraîner une insuffisance de mémoire. Vous pouvez utiliser [Modifier les métadonnées](edit-metadata.md) pour marquer cette colonne comme **Effacer la caractéristique** afin qu’elle ne soit pas utilisée durant l’entraînement, ou le [module d’extraction des caractéristiques de N-grammes du texte](extract-n-gram-features-from-text.md) pour prétraiter la colonne de texte. Pour plus d’informations sur l’erreur, consultez [Codes d’erreur pour le concepteur](././designer-error-codes.md).

## <a name="results"></a>Résultats

Une fois le modèle formé :


+ Pour utiliser le modèle dans d’autres pipelines, sélectionnez le module et l’icône **Inscrire le jeu de données** sous l’onglet **Sorties** dans le panneau droit. Vous pouvez accéder aux modèles enregistrés dans la palette de modules sous **Jeux de données**.

+ Afin d’utiliser le modèle pour la prédiction de nouvelles valeurs, connectez-le au module [Score Model](./score-model.md) (Noter le modèle) avec les nouvelles données d’entrée.


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 