---
title: Transformation de filtre dans un flux de données de mappage
description: Filtrer des lignes à l’aide de la transformation de filtre dans un flux de données de mappage Azure Data Factory
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 05/26/2020
ms.openlocfilehash: 8189228d6707812fb943e9925dc2bbf1b6da4972
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84112808"
---
# <a name="filter-transformation-in-mapping-data-flow"></a>Transformation de filtre dans un flux de données de mappage

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Les transformations de filtre permettent de filtrer des lignes en fonction d’une condition. Le flux de sortie comprend toutes les lignes qui correspondent à la condition de filtrage. La transformation de filtre est similaire à une clause WHERE dans SQL.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4xnxN]

## <a name="configuration"></a>Configuration

Utilisez le générateur d’expressions de flux de données afin d’entrer une expression pour la condition de filtre. Pour ouvrir le générateur d’expressions, cliquez sur la zone bleue. La condition de filtre doit être de type booléen. Pour plus d’informations sur la création d’une expression, consultez la documentation sur le [générateur d’expressions](concepts-data-flow-expression-builder.md).

![Transformation de filtre](media/data-flow/filter1.png "Transformation de filtre")

## <a name="data-flow-script"></a>Script de flux de données

### <a name="syntax"></a>Syntaxe

```
<incomingStream>
    filter(
        <conditionalExpression>
    ) ~> <filterTransformationName>
```

### <a name="example"></a>Exemple

L’exemple ci-dessous illustre une transformation de filtre nommée `FilterBefore1960` traitant le flux entrant `CleanData`. La condition de filtre est l’expression `year <= 1960`.

Dans l’expérience utilisateur Data Factory, cette transformation se présente comme dans l’image ci-dessous :

![Transformation de filtre](media/data-flow/filter1.png "Transformation de filtre")

Le script de flux de données pour cette transformation se trouve dans l’extrait de code ci-dessous :

```
CleanData
    filter(
        year <= 1960
    ) ~> FilterBefore1960

```

## <a name="next-steps"></a>Étapes suivantes

Filtrer des colonnes avec la [transformation de sélection](data-flow-select.md)
