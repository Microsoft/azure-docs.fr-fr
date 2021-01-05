---
title: 'Sélectionner des colonnes dans le jeu de données : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Sélectionner des colonnes dans le jeu de données d’Azure Machine Learning pour choisir un sous-ensemble de colonnes à utiliser dans des opérations en aval.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 10/22/2019
ms.openlocfilehash: 7926d5dd33df5538713f8de152dbbace2bc29864
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90890296"
---
# <a name="select-columns-in-dataset-module"></a>Module Sélectionner des colonnes dans le jeu de données

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez ce module pour choisir un sous-ensemble de colonnes à utiliser dans des opérations en aval. Le module ne supprime pas physiquement les colonnes de jeu de données source. Au lieu de cela, il crée un sous-ensemble de colonnes, à l’instar d’une *vue* ou *projection* de base de données.

Ce module est utile lorsque vous devez limiter les colonnes disponibles pour une opération en aval, ou si vous souhaitez réduire la taille du jeu de données en supprimant les colonnes inutiles.

Les colonnes dans le jeu de données sont générées dans le même ordre que dans les données d’origine, même si vous les spécifiez dans un ordre différent.

## <a name="how-to-use"></a>Utilisation

Ce module n’a aucun paramètre. Vous utilisez le sélecteur de colonne pour choisir les colonnes à inclure ou exclure.

### <a name="choose-columns-by-name"></a>Choisir des colonnes par nom

Plusieurs options du module permettent de choisir des colonnes par nom : 

+ Filtrer et rechercher

    Cliquez sur l’option **PAR NOM**.

    Si vous avez connecté un jeu de données déjà rempli, une liste des colonnes disponibles doit apparaître. Si aucune colonne n’apparaît, vous devrez peut-être exécuter des modules en amont pour afficher la liste des colonnes.

    Pour filtrer la liste, saisissez dans la zone de recherche. Par exemple, si vous saisissez la lettre `w` dans la zone de recherche, la liste est filtrée pour afficher les noms des colonnes qui contiennent la lettre `w`.

    Sélectionnez les colonnes et cliquez sur le bouton de flèche droite pour déplacer les colonnes sélectionnées dans la liste du volet de droite.

    + Pour sélectionner une plage continue de noms de colonnes, appuyez sur **Maj + clic**.
    + Pour ajouter des colonnes individuelles à la sélection, appuyez sur **Ctrl + clic**.

    Cliquez sur la coche pour enregistrer le fichier et fermer.

+ Utiliser des noms en combinaison avec d’autres règles

    Cliquez sur l’option **WITH RULES** (AVEC RÈGLES).
    
    Choisissez une règle, par exemple l’affichage des colonnes d’un type de données spécifique.

    Ensuite, cliquez sur les colonnes individuelles de ce type par nom, pour les ajouter à la liste de sélection.

+ Saisir ou coller une liste de noms de colonnes séparée par des virgules

    Si votre jeu de données est volumineux, il peut être plus facile d’utiliser des index ou des listes de noms générées plutôt que de sélectionner des colonnes individuellement. Supposons que vous avez préparé la liste à l’avance :

    1. Cliquez sur l’option **WITH RULES** (AVEC RÈGLES). 
    2. Sélectionnez **Aucune colonne**, sélectionnez **Inclure**, puis cliquez dans la zone de texte avec le point d’exclamation rouge. 
    3. Collez ou saisissez une liste de noms de colonnes validés précédemment séparée par des virgules. Vous ne pouvez pas enregistrer le module si une colonne a un nom non valide. Veillez donc à vérifier les noms au préalable.
    
    Vous pouvez également utiliser cette méthode pour spécifier une liste de colonnes à l’aide de leurs valeurs d’index. 

### <a name="choose-by-type"></a>Choisir par type

Si vous utilisez l’option **WITH RULES** (AVEC RÈGLES), vous pouvez appliquer plusieurs conditions aux sélections de colonnes. Par exemple, vous devrez peut-être obtenir les colonnes de fonctionnalités d’un type de données numériques uniquement.

L’option **COMMENCER PAR** détermine votre point de départ et est importante pour la compréhension des résultats. 

+ Si vous sélectionnez l’option **TOUTES LES COLONNES**, toutes les colonnes sont ajoutées à la liste. Vous devez ensuite utiliser l’option **Exclure** pour *supprimer* les colonnes correspondant à certaines conditions. 

    Par exemple, vous pouvez commencer avec toutes les colonnes puis supprimer des colonnes par nom ou par type.

+ Si vous sélectionnez l’option **AUCUNE COLONNE**, la liste de colonnes initiale est vide. Vous spécifiez ensuite des conditions pour *ajouter* des colonnes à la liste. 

    Si vous appliquez plusieurs règles, chaque condition est **additive**. Par exemple, supposons que vous commenciez avec aucune colonne, puis que vous ajoutez une règle pour obtenir toutes les colonnes numériques. Dans le jeu de données de prix Automobile, ceci génère 16 colonnes. Vous cliquez ensuite sur le signe **+** pour ajouter une nouvelle condition, puis sélectionnez **Include all features** (Inclure toutes les fonctionnalités). Le jeu de données obtenu inclut toutes les colonnes numériques, ainsi que toutes les colonnes de fonctionnalités, y compris certaines colonnes de fonctionnalités de chaîne.

### <a name="choose-by-column-index"></a>Choisir par index de colonne

L’index de colonne fait référence à l’ordre de la colonne dans le jeu de données d’origine.

+ Les colonnes sont numérotées de manière séquentielle à partir de 1.  
+ Pour obtenir une plage de colonnes, utilisez un trait d’union. 
+ Les spécifications ouvertes telles que `1-` ou `-3` ne sont pas autorisées.
+ Les valeurs d’index (ou noms de colonnes) en double ne sont pas autorisées et peuvent entraîner une erreur.

Par exemple, en supposant que votre jeu de données contient au moins huit colonnes, vous pouvez coller dans l’un des exemples suivants pour retourner plusieurs colonnes non contiguës : 

+ `8,1-4,6`
+ `1,3-8`
+ `1,3-6,4` 

le dernier exemple n’entraîne pas d’erreur. Il retourne toutefois une instance unique de la colonne `4`.



### <a name="change-order-of-columns"></a>Modifier l’ordre des colonnes

L’option **Allow duplicates and preserve column order in selection** (Autoriser les doublons et conserver l’ordre des colonnes dans la sélection) donne initialement une liste vide et ajoute des colonnes que vous spécifiez par nom ou par index. Contrairement à d’autres options, qui retournent toujours des colonnes dans leur « ordre naturel », cette option affiche les colonnes dans l’ordre dans lequel vous les nommez ou répertoriez. 

Par exemple, dans un jeu de données avec les colonnes Col1, Col2, Col3 et Col4, vous pouvez inverser l’ordre des colonnes et omettre la colonne 2 en spécifiant une des listes suivantes :

+ `Col4, Col3, Col1`
+ `4,3,1`


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 