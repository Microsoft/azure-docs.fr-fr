---
title: Scorer un modèle Vowpal Wabbit
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Score Vowpal Wabbit Model (Scorer un modèle Vowpal Wabbit) afin de générer des scores pour un ensemble de données d’entrée, à l’aide d’un modèle Vowpal Wabbit entraîné et déjà existant.
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 07/02/2020
ms.openlocfilehash: 23dfee7b78f2606c54525391e1260af69a9b0779
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90898396"
---
# <a name="score-vowpal-wabbit-model"></a>Scorer un modèle Vowpal Wabbit
Cet article explique comment utiliser le module **Score Vowpal Wabbit Model** dans le concepteur Azure Machine Learning afin de générer des scores pour un ensemble de données d’entrée, à l’aide d’un modèle Vowpal Wabbit entraîné et déjà existant.  

Ce module fournit la dernière version du framework Vowpal Wabbit, la version 8.8.1. Utilisez ce module pour scorer les données à l’aide d’un modèle entraîné enregistré au format VW version 8.  

## <a name="how-to-configure-score-vowpal-wabbit-model"></a>Guide pratique pour configurer le module Score Vowpal Wabbit Model

1.  Ajoutez le module **Score Vowpal Wabbit Model** à votre expérience.  
  
2.  Ajoutez un modèle Vowpal Wabbit entraîné, et connectez-le au port d’entrée de gauche. Vous pouvez utiliser un modèle entraîné créé dans la même expérience, ou localiser un modèle enregistré dans la catégorie **Jeux de données** du volet de navigation gauche du concepteur. Toutefois, le modèle doit être disponible dans le concepteur Azure Machine Learning.  
  
    > [!NOTE]
    > Seuls les modèles Vowpal Wabbit 8.8.1 sont pris en charge. Vous ne pouvez pas connecter des modèles enregistrés qui ont été entraînés à l’aide d’autres algorithmes.
  
3.  Ajoutez le jeu de données de test, puis connectez-le au port d’entrée de droite. Si le jeu de données de test est un répertoire qui contient le fichier de données de test, spécifiez le nom du fichier de données de test dans **Name of the test data file** (Nom du fichier de données de test). Si le jeu de données de test correspond à un seul fichier, n’indiquez rien dans **Name of the test data file** (Nom du fichier de données de test).

4. Dans la zone de texte **VW arguments** (Arguments VW), tapez un ensemble d’arguments de ligne de commande valides pour l’exécutable Vowpal Wabbit.  

    Pour plus d’informations sur les arguments Vowpal Wabbit pris en charge et non pris en charge dans Azure Machine Learning, consultez la section [Notes techniques](#technical-notes).  

5.  **Name of the test data file** (Nom du fichier de données de test) : tapez le nom du fichier qui contient les données d’entrée. Cet argument est utilisé uniquement quand le jeu de données de test est un répertoire.

6. **Specify file type** (Spécifier le type de fichier) : indiquez le format utilisé par vos données d’entraînement. Vowpal Wabbit prend en charge les deux formats de fichiers d’entrée suivants :  

   - **VW** représente le format interne utilisé par Vowpal Wabbit. Pour plus d’informations, consultez la [page wiki de Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki/Input-format). 
   - **SVMLight** est un format utilisé par d’autres outils de machine learning. 

7. Sélectionnez l’option **Include an extra column containing labels** (Inclure une colonne supplémentaire contenant des étiquettes), si vous souhaitez afficher des étiquettes avec les scores.  

   En règle générale, durant le traitement des données de texte, Vowpal Wabbit ne nécessite pas d’étiquettes. Il retourne uniquement les scores pour chaque ligne de données.  

8. Sélectionnez l’option **Include an extra column containing raw scores** (Inclure une colonne supplémentaire contenant les scores bruts), si vous souhaitez afficher des scores bruts avec les résultats.  

9. Envoyez le pipeline.

## <a name="results"></a>Résultats

Une fois l’apprentissage terminé :

+ Pour visualiser les résultats, cliquez avec le bouton droit sur la sortie du module [Score Vowpal Wabbit Model](score-vowpal-wabbit-model.md). La sortie indique un score de prédiction normalisé allant de 0 à 1. 

+ Pour évaluer les résultats, le jeu de données de sortie doit contenir des noms de colonnes de score spécifiques, qui répondent aux exigences du module Evaluate Model (Évaluer le modèle).

  + Pour la tâche de régression, le jeu de données à évaluer doit avoir une colonne, nommée `Regression Scored Labels`, qui représente les étiquettes associées à un score.
  + Pour la tâche de classification binaire, le jeu de données à évaluer doit avoir deux colonnes, nommées `Binary Class Scored Labels`,`Binary Class Scored Probabilities`, qui représentent respectivement les étiquettes associées à un score et les probabilités.
  + Pour la tâche de classification multiple, le jeu de données à évaluer doit avoir une colonne, nommée `Multi Class Scored Labels`, qui représente les étiquettes associées à un score.

  Notez que les résultats du module Score Vowpal Wabbit Model ne peuvent pas être évalués directement. Avant l’évaluation, le jeu de données doit être modifié conformément aux exigences ci-dessus.

##  <a name="technical-notes"></a>Notes techniques

Cette section contient des détails, des conseils et des réponses aux questions fréquentes concernant l’implémentation.

### <a name="parameters"></a>Paramètres

Vowpal Wabbit propose de nombreuses options de ligne de commande pour le choix et le réglage des algorithmes. La description complète de ces options n’est pas possible ici. Nous vous recommandons de consulter la [page wiki Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments).  

Les paramètres suivants ne sont pas pris en charge dans Azure Machine Learning Studio (classique).  

-   Les options d’entrée/de sortie spécifiées dans [https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments](https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments)  
  
     Ces propriétés sont déjà configurées automatiquement par le module.  
  
-   En outre, toute option qui génère plusieurs sorties ou accepte plusieurs entrées est interdite. Il s’agit notamment de *`--cbt`* , *`--lda`* et *`--wap`* .  
  
-   Seuls les algorithmes de machine learning supervisé sont pris en charge. Les options suivantes ne sont pas autorisées : *`–active`* , `--rank`, *`--search`* , etc.  

Tous les arguments autres que ceux décrits ci-dessus sont autorisés.

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 