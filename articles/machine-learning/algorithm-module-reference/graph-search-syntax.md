---
title: Syntaxe des requêtes de recherche Graph
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser la syntaxe de requête de recherche dans le concepteur Azure Machine Learning pour rechercher des nœuds dans le graphique de pipeline.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 8/24/2020
ms.openlocfilehash: 762581ea5b3183d62913e9ea6935bf7e4c4ae67f
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93420765"
---
# <a name="graph-search-query-syntax"></a>Syntaxe des requêtes de recherche Graph

Dans cet article, vous allez découvrir la syntaxe de requête de recherche de graphique dans Azure Machine Learning. La fonctionnalité de recherche de graphique vous permet de rechercher un nœud en fonction de son nom et de ses propriétés. 

 ![Capture d’écran animée montrant un exemple d’expérience de recherche de graphique](media/search/graph-search.gif)

La recherche de graphique prend en charge la recherche par mot clé en texte intégral sur le nom de nœud et les commentaires. Vous pouvez également filtrer la propriété de nœud, par exemple runStatus, duration, computeTarget. La recherche par mot clé est basée sur la requête Lucene. Une requête de recherche complète ressemble à ceci :  

**[requête Lucene | [requête de filtre]** 

Vous pouvez utiliser une requête Lucene ou une requête de filtre. Pour utiliser les deux, servez-vous du séparateur **|** . La syntaxe de la requête de filtre est plus stricte que la requête Lucene. Par conséquent, si l’entrée du client peut être analysée avec les deux, la requête de filtre sera appliquée.

 

## <a name="lucene-query"></a>Requête Lucene

La recherche de graphique utilise la requête simple Lucene comme syntaxe de recherche en texte intégral sur le nœud « name » et « comment ». Les opérateurs Lucene suivants sont pris en charge :

 
- AND/OR
- Caractère générique correspondant à **?** et * *\** _.

### <a name="examples"></a>Exemples

- Recherche simple : `JSON Data`

- AND / OR : `JSON AND Validation`

- AND/OR multiple : `(JSON AND Validation) OR (TSV AND Training)`

 
- Caractère générique correspondant : 
    - `machi?e learning`
    - `mach_ing`
 
>[!NOTE]
> Vous ne pouvez pas démarrer une requête Lucene avec un caractère « * ».

##  <a name="filter-query"></a>Requête de filtre

 
Les requêtes de filtres utilisent le modèle suivant :
 
`**[key1] [operator1] [value1]; [key2] [operator1] [value2];**`

 
Vous pouvez utiliser les propriétés de nœud suivantes comme clés :

- runStatus
- compute
- duration
- reuse

Et vous pouvez utiliser les opérateurs suivants :

- Supérieur ou égal à : `>=`
- Inférieur ou égal à : `<=`
- Supérieur à : `>`
- Inférieur à : `<`
- Égal à : `==`
- Contenir : `=`
- NotEqual : `!=`
- Dans : `in`

 
 

### <a name="example"></a>Exemple

- durée > 100 ;
- état dans {failed, NotStarted}
- calcul dans {GPU-cluster}; runStatus dans {Completed}

## <a name="technical-notes"></a>Notes techniques

- La relation entre plusieurs filtres est « AND »
- Si `>=,  >,  <, or  <=` est choisi, les valeurs sont automatiquement converties en type de nombre. Sinon, les types de chaînes sont utilisés pour la comparaison.
- Pour toutes les valeurs de type chaîne, la casse n’est pas sensible en comparaison.
- L’opérateur « In » attend une collection comme valeur, la syntaxe de la collection est `{name1, name2, name3}`
- L’espace sera ignoré entre les mots clés