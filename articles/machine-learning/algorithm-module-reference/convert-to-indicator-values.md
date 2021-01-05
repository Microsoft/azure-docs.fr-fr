---
title: Convertir en valeurs d’indicateur
titleSuffix: Azure Machine Learning
description: Utilisez le module Convertir en valeurs d’indicateur dans le concepteur Azure Machine Learning pour convertir des colonnes catégorielles en une série de colonnes d’indicateurs binaires.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/11/2020
ms.openlocfilehash: 81b3c113f46428327842c1555fdd1934e9ae8762
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93420850"
---
# <a name="convert-to-indicator-values"></a>Convertir en valeurs d’indicateur
Cet article décrit un module du concepteur Azure Machine Learning.

Utilisez le module **Convertir en valeurs d’indicateur** dans le concepteur Azure Machine Learning pour convertir des colonnes qui contiennent des valeurs catégorielles en une série de colonnes d'indicateurs binaires.  

Ce module génère également une définition de la transformation utilisée pour effectuer une conversion en valeurs d’indicateur. Vous pouvez réutiliser cette transformation sur d’autres jeux de données qui ont le même schéma, à l’aide du module [Appliquer une transformation](apply-transformation.md).

## <a name="how-to-configure-convert-to-indicator-values"></a>Comment configurer le module Convertir en valeurs d’indicateur

1.  Recherchez le module **Convertir en valeurs d’indicateur**, puis faites-le glisser vers votre brouillon de pipeline. Ce module est disponible sous la catégorie **Transformation des données**.
    > [!NOTE]
    > Vous pouvez utiliser le module [Modifier des métadonnées](edit-metadata.md) avant de convertir le module **Convertir en valeurs d’indicateur** pour marquer la ou les colonnes cibles comme catégorielles.

1. Connectez le module **Convertir en valeurs d’indicateur** en un jeu de données contenant les colonnes que vous souhaitez convertir. 

1. Sélectionnez **Modifier la colonne** pour choisir une ou plusieurs colonnes catégorielles.

1. Sélectionnez l’option **Remplacer les colonnes catégorielles** si vous souhaitez obtenir en sortie **uniquement** les nouvelles colonnes booléennes. Par défaut, cette option est désactivée.
    

    > [!TIP]
    >  Si vous choisissez l’option de remplacement, la colonne source n’est pas réellement supprimée ou modifiée. Au lieu de cela, les nouvelles colonnes sont générées et présentées dans le jeu de données de sortie, et la colonne source reste disponible dans l’espace de travail. Si vous avez besoin de voir les données d’origine, vous pouvez utiliser le module [Ajouter des colonnes](add-columns.md) à tout moment pour rajouter la colonne source.

1. Envoyez le pipeline.

## <a name="results"></a>Résultats

Supposons que vous avez une colonne contenant des scores qui indiquent si un serveur a une probabilité de défaillance élevée, moyenne ou faible.  

| ID du serveur | Score de défaillance |
| --------- | ------------- |
| 10301     | Faible           |
| 10302     | Moyenne        |
| 10303     | Élevé          |

Quand vous appliquez le module **Convertir en valeurs d’indicateur**, le concepteur convertit une seule colonne d’étiquettes en plusieurs colonnes contenant des valeurs booléennes :  

| ID du serveur | Probabilité de défaillance - Faible | Probabilité de défaillance - Moyenne | Probabilité de défaillance - Élevée |
| --------- | ------------------- | ---------------------- | -------------------- |
| 10301     | 1                   | 0                      | 0                    |
| 10302     | 0                   | 1                      | 0                    |
| 10303     | 0                   | 0                      | 1                    |

Voici comment la conversion fonctionne :  

-   Dans la colonne **Probabilité de défaillance** qui décrit le risque, il n’y a que trois valeurs possibles (Élevée, Moyenne et Faible), et aucune valeur manquante. Par conséquent, exactement trois colonnes sont créées.  

-   Les nouvelles colonnes d’indicateurs sont nommées en fonction des en-têtes de colonne et des valeurs de la colonne source, selon le modèle *\<source column>- \<data value>* .  

-   Il doit y avoir un 1 dans exactement une colonne d’indicateur, et 0 dans toutes les autres colonnes d’indicateur, puisque chaque serveur ne peut avoir qu’une seule évaluation du risque.  

Vous pouvez maintenant utiliser les trois colonnes d’indicateur comme caractéristiques dans un modèle Machine Learning.

Le module renvoie deux sorties :

- **Results dataset** (Jeu de données de résultats) : jeu de données avec des colonnes de valeurs d’indicateur converties. Les colonnes non sélectionnées pour le nettoyage sont également « transmises ».
- **Transformation de valeurs d’indicateur** : transformation de données utilisée pour la conversion en valeurs d’indicateur, et qui peut être enregistrée dans votre espace de travail et appliquée ultérieurement à de nouvelles données.

## <a name="apply-a-saved-indicator-values-operation-to-new-data"></a>Appliquer une opération de valeurs d’indicateur enregistrée à de nouvelles données

Si vous devez répéter souvent des opérations de valeurs d’indicateur, vous pouvez enregistrer vos étapes de manipulation des données en tant que *transformation* afin de la réutiliser avec le même jeu de données. C’est utile si vous devez fréquemment réimporter puis nettoyer des données qui ont le même schéma.

1. Ajoutez le module [Appliquer une transformation](apply-transformation.md) à votre pipeline.

1. Ajoutez le jeu de données que vous souhaitez nettoyer et connectez le jeu de données au port d’entrée droit.

1. Développez le groupe **Transformation des données** dans le volet gauche du concepteur. Recherchez la transformation enregistrée et faites-la glisser vers le pipeline.

1. Connectez la transformation enregistrée au port d’entrée gauche de [Appliquer une transformation](apply-transformation.md).

   Quand vous appliquez une transformation enregistrée, vous ne pouvez pas sélectionner les colonnes à transformer. Cela est dû au fait que la transformation a été définie et qu’elle s’applique automatiquement aux types de données spécifiés dans l’opération d’origine.

1. Envoyez le pipeline.
 
## <a name="technical-notes"></a>Notes techniques  

Cette section contient des détails, des conseils et des réponses aux questions fréquentes concernant l’implémentation.

### <a name="usage-tips"></a>Conseils d’utilisation

-   Seules les colonnes qui sont marquées comme catégorielles peuvent être converties en colonnes d’indicateurs. Si l’erreur suivante s’affiche, l’une des colonnes que vous avez sélectionnées n’est probablement pas catégorielle :  

     Erreur 0056 : La colonne portant le nom \<column name> n’appartient pas à une catégorie autorisée.  

     Par défaut, la plupart des colonnes de chaîne sont gérées comme des caractéristiques de chaîne. Vous devez donc les marquer explicitement comme catégorielles à l’aide de l’option [Modifier des métadonnées](edit-metadata.md).  

-   Le nombre de colonnes que vous pouvez convertir en colonnes d’indicateur n’est pas limité. Toutefois, étant donné que chaque colonne de valeurs peut générer plusieurs colonnes d’indicateur, il est possible de convertir et de consulter uniquement quelques colonnes à la fois.  

-   Si des valeurs sont manquantes dans la colonne, une colonne d’indicateurs distincte est créée pour la catégorie manquante sous le nom *\<source column>- Missing*  

-   Si la colonne que vous convertissez en valeurs d’indicateur contient des nombres, ceux-ci doivent être marqués comme étant catégoriels comme toute autre colonne de caractéristique. Cela étant fait, les nombres sont traités comme des valeurs discrètes. Par exemple, si vous avez une colonne numérique contenant des valeurs de consommation de carburant (en MPG, miles par gallon) comprises entre 25 et 30, une nouvelle colonne d’indicateur est créée pour chaque valeur discrète :  

    | Marque       | MPG sur autoroute -25 | MPG sur autoroute -26 | MPG sur autoroute -27 | MPG sur autoroute -28 | MPG sur autoroute -29 | MPG sur autoroute -30 |
    | ---------- | --------------- | --------------- | --------------- | --------------- | --------------- | --------------- |
    | Voitures Contoso | 0               | 0               | 0               | 0               | 0               | 1               |

- Pour éviter d’ajouter un trop grand nombre de dimensions à votre jeu de données, nous vous recommandons de vérifier d’abord le nombre de valeurs présentes dans la colonne, puis de placer les données dans des conteneurs ou de les quantifier de manière appropriée.  


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 
