---
title: 'Supprimer les lignes en double : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Supprimer les lignes en double dans Azure Machine Learning pour supprimer les doublons potentiels d’un jeu de données.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 10/22/2019
ms.openlocfilehash: bf35d08128aa8a3e8f545ed7184866694219f2cb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90905224"
---
# <a name="remove-duplicate-rows-module"></a>Module Supprimer les lignes en double

Cet article décrit un module dans le concepteur Azure Machine Learning.

Ce module permet de supprimer les doublons potentiels d’un jeu de données.

Par exemple, supposons que vos données se présentent comme suit et représentent plusieurs dossiers de patients. 

| PatientID | Initials| Sexe|Age|Admis|
|----|----|----|----|----|
|1|F.M.| M| 53| Jan|
|2| F.A.M.| M| 53| Jan|
|3| F.A.M.| M| 24| Jan|
|3| F.M.| M| 24| Fév|
|4| F.M.| M| 23| Fév|
| | F.M.| M| 23| |
|5| F.A.M.| M| 53| |
|6| F.A.M.| M| NaN| |
|7| F.A.M.| M| NaN| |

Clairement, cet exemple comporte plusieurs colonnes avec des données potentiellement en double. Le fait de savoir s’il s’agit réellement de doublons dépend de votre connaissance des données. 

+ Par exemple, vous savez peut-être que plusieurs patients ont le même nom. Vous n’élimineriez pas les doublons à l’aide des colonnes de nom, mais uniquement de la colonne **ID**. De cette façon, seules les lignes avec des valeurs d’ID en double sont supprimées, que les patients portent le même nom ou pas.

+ Vous pouvez également choisir d’autoriser les doublons dans le champ ID et utiliser une autre combinaison de fichiers pour rechercher des dossiers uniques, comme le prénom, le nom, l’âge et le sexe.  

Pour définir les critères déterminant si une ligne est en double ou non, vous spécifiez une seule colonne ou un ensemble de colonnes à utiliser comme **clés**. Deux lignes sont considérées comme des doublons uniquement lorsque les valeurs dans **toutes**  les colonnes de clés sont identiques. Si une ligne ne comporte pas de valeur pour les **clés**, elle n’est pas considérée comme un doublon. Par exemple, si l’âge et le sexe sont définis comme clés dans la table ci-dessus, les lignes 6 et 7 ne sont pas des doublons, étant donné que la valeur d’âge est manquante.

Lorsque vous exécutez le module, il crée un jeu de données candidat et envoie un ensemble de lignes sans doublons dans le jeu de colonnes spécifié.

> [!IMPORTANT]
> Le jeu de données source n’est pas modifié. Ce module crée un autre jeu de données qui est filtré pour exclure les doublons, selon les critères que vous spécifiez.

## <a name="how-to-use-remove-duplicate-rows"></a>Suppression des lignes en double

1. Ajouter le module à votre pipeline. Le module **Supprimer les lignes en double** se trouve sous **Transformation de données**, **Manipulation**.  

2. Connectez le jeu de données dans lequel rechercher les lignes en double.

3. Dans le volet **Propriétés**, sous **Key column selection filter expression** (Expression de filtre pour la sélection de colonne de clé), cliquez sur **Launch column selector** (Lancer le sélecteur de colonnes) pour choisir les colonnes qui serviront à identifier les doublons.

    Dans ce contexte, **clé** n’a pas le sens d’identificateur unique. Toutes les colonnes que vous sélectionnez à l’aide du sélecteur de colonnes sont désignées comme **colonnes de clés**, ce que ne sont pas toutes les colonnes non sélectionnées. La combinaison de colonnes choisies comme clés détermine l’unicité des enregistrements. (Considérez cela comme une instruction SQL qui utilise plusieurs jointures d’égalités.)

    Exemples :

    + « Je veux m’assurer que les ID sont uniques » : choisissez uniquement la colonne ID.
    + « Je veux m’assurer que la combinaison de prénom, nom et ID est unique » : sélectionnez les trois colonnes.

4. Utilisez la case à cocher **Retain first duplicate row** (Conserver la première ligne en double) pour indiquer les lignes à renvoyer lorsque des doublons sont détectés :

    + Si cette option est sélectionnée, la première ligne est renvoyée, et les autres sont ignorées. 
    + Si vous désélectionnez cette option, la dernière ligne en double est conservée dans les résultats, et les autres sont ignorées. 

5. Envoyez le pipeline.

6. Pour examiner les résultats, cliquez avec le bouton droit sur le module, puis sélectionnez **Visualiser**. 

> [!TIP]
> Si les résultats sont difficiles à comprendre, ou si vous souhaitez exclure certaines colonnes, vous pouvez supprimer des colonnes à l’aide du module [Select Columns in Dataset](./select-columns-in-dataset.md) (Sélectionner des colonnes dans un jeu de données).

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 