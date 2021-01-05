---
title: Transformation de fractionnement conditionnel dans un flux de données de mappage
description: Fractionnez les données en différents flux à l’aide de la transformation de fractionnement conditionnel dans un flux de données de mappage Azure Data Factory
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 05/21/2020
ms.openlocfilehash: eece6f97e82f3800d4f59ac1849b34c2a1e4635b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83800085"
---
# <a name="conditional-split-transformation-in-mapping-data-flow"></a>Transformation de fractionnement conditionnel dans un flux de données de mappage

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

La transformation de fractionnement conditionnel route les lignes de données vers différents flux en fonction de conditions de correspondance. La transformation de fractionnement conditionnel est similaire à une structure de décision CASE dans un langage de programmation. La transformation évalue les expressions, et selon les résultats, dirige la ligne de données vers le flux spécifié.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4wKCX]

## <a name="configuration"></a>Configuration

Le paramètre **SplitOn** détermine si la ligne de données est acheminée vers le premier flux correspondant ou vers chaque flux auquel elle correspond.

Utilisez le générateur d’expressions de flux de données afin d’entrer une expression pour la condition de fractionnement. Pour ajouter une nouvelle condition, cliquez sur l’icône plus dans une ligne existante. Vous pouvez également ajouter un flux par défaut pour les lignes qui ne correspondent à aucune condition.

![fractionnement conditionnel](media/data-flow/conditionalsplit1.png "options de fractionnement conditionnel")

## <a name="data-flow-script"></a>Script de flux de données

### <a name="syntax"></a>Syntaxe

```
<incomingStream>
    split(
        <conditionalExpression1>
        <conditionalExpression2>
        ...
        disjoint: {true | false}
    ) ~> <splitTx>@(stream1, stream2, ..., <defaultStream>)
```

### <a name="example"></a>Exemple

L’exemple ci-dessous illustre une transformation de fractionnement conditionnel nommée `SplitByYear` traitant le flux entrant `CleanData`. Cette transformation a deux conditions de fractionnement : `year < 1960` et `year > 1980`. `disjoint` a la valeur false, car les données sont acheminées suivant la première condition correspondante. Chaque ligne correspondant à la première condition est acheminée vers le flux de sortie `moviesBefore1960`. Toutes les lignes restantes correspondant à la deuxième condition sont acheminées vers le flux de sortie `moviesAFter1980`. Toutes les autres lignes sont acheminées vers le flux par défaut `AllOtherMovies`.

Dans l’expérience utilisateur Data Factory, cette transformation se présente comme dans l’image ci-dessous :

![fractionnement conditionnel](media/data-flow/conditionalsplit1.png "options de fractionnement conditionnel")

Le script de flux de données pour cette transformation se trouve dans l’extrait de code ci-dessous :

```
CleanData
    split(
        year < 1960,
        year > 1980,
        disjoint: false
    ) ~> SplitByYear@(moviesBefore1960, moviesAfter1980, AllOtherMovies)
```

## <a name="next-steps"></a>Étapes suivantes

Les transformations de flux de données courantes utilisées avec le fractionnement conditionnel sont la [transformation de jointure](data-flow-join.md), la [transformation de recherche](data-flow-lookup.md) et la [transformation de sélection](data-flow-select.md).
