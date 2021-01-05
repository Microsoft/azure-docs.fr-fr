---
title: 'Effectuer l’apprentissage du modèle de détection d’anomalie : sur le module Modifier les métadonnées'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Effectuer l’apprentissage du modèle de détection d’anomalie pour créer un modèle de détection d’anomalie formé.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/22/2020
ms.openlocfilehash: edf35fada4233fbe43bc7f859c2414bfb8130714
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90905728"
---
# <a name="train-anomaly-detection-model-module"></a>Module Entraîner le modèle de détection d’anomalie

Cet article explique comment utiliser le module Entraîner le modèle de détection d’anomalie dans le concepteur Azure Machine Learning pour créer un modèle de détection d’anomalie entraîné.

Le module prend comme entrée un ensemble de paramètres pour un modèle de détection d’anomalie et un jeu de données sans étiquettes. Il renvoie un modèle de détection d’anomalie formé, ainsi qu’un jeu d’étiquettes pour les données d’apprentissage.  

Pour plus d’informations sur les algorithmes de détection d’anomalie fournis dans le concepteur, consultez [Détection d’anomalie reposant sur l’ACP](pca-based-anomaly-detection.md).  

## <a name="how-to-configure-train-anomaly-detection-model"></a>Comment configurer le module Effectuer l’apprentissage du modèle de détection d’anomalie 

1.  Ajoutez le module **Effectuer l’apprentissage du modèle de détection d’anomalie** à votre pipeline dans le concepteur. Ce module figure dans la catégorie **Détection d’anomalie**.

2. Connectez l’un des modules conçus pour la détection d’anomalie, par exemple [Détection d’anomalie reposant sur l’ACP](pca-based-anomaly-detection.md).

    Les autres types de modèles ne sont pas pris en charge. Quand vous exécutez le pipeline, vous obtenez l’erreur « Tous les modèles doivent avoir le même type d’apprenant. »  

3.  Configurez le module de détection d’anomalie en choisissant la colonne d’étiquette et en définissant d’autres paramètres spécifiques à l’algorithme.  

4.  Attachez un jeu de données d’entraînement à l’entrée côté droit du module **Entraîner le modèle de détection des anomalies**.  

5.  Envoyez le pipeline.  

## <a name="results"></a>Résultats

Une fois l’apprentissage terminé :

+ Pour afficher les paramètres du modèle, cliquez avec le bouton droit sur le module et sélectionnez **Visualiser**. 

+ Pour créer des prédictions, utilisez le module [Noter le modèle](score-model.md) avec les nouvelles données d’entrée.

+ Pour enregistrer un instantané du modèle entraîné, sélectionnez le module. Sélectionnez ensuite l’icône **Enregistrer le jeu de données** sous l’onglet **Sorties + journaux** dans le volet droit.   

 
## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 

Pour obtenir la liste des erreurs spécifiques aux modules du concepteur, consultez [Exceptions et codes d’erreur du concepteur](designer-error-codes.md).
'