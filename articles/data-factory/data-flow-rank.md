---
title: Transformation du classement dans le flux de données de mappage
description: Comment utiliser la transformation de classement du flux de données de mappage d’Azure Data Factory pour générer une colonne de classement
author: dcstwh
ms.author: weetok
ms.reviewer: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 10/05/2020
ms.openlocfilehash: b7adb6bf13cba5f886b442515e8ba5661cfeb8ef
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96490922"
---
# <a name="rank-transformation-in-mapping-data-flow"></a>Transformation du classement dans le flux de données de mappage 

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Utilisez la transformation de classement pour générer un classement ordonné en fonction des conditions de tri spécifiées par l’utilisateur. 

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4GGJo]

## <a name="configuration"></a>Configuration

![Rank settings](media/data-flow/rank-configuration.png "Paramètres de classement")

**Insensible à la casse :** Si une colonne de tri est de type chaîne, la casse sera factorisée dans le classement. 

**Dense :** Si l’option est activée, la colonne de classement sera classée dans un classement dense. Chaque classement sera un nombre consécutif et les valeurs de classement ne seront pas ignorées en cas d’égalité.

**Colonne de classement :** Nom de la colonne de classement générée. Cette colonne sera de type long.

**Conditions de tri :** Choisissez les colonnes sur lesquelles porte le tri et l’ordre dans lequel il se produit. L’ordre détermine la priorité de tri.

La configuration ci-dessus accepte les données de basket-ball entrantes et crée une colonne de classement appelée « pointsRanking ». La ligne ayant la valeur la plus élevée de la colonne *PTS* aura une valeur *pointsRanking* de 1.

## <a name="data-flow-script"></a>Script de flux de données

### <a name="syntax"></a>Syntaxe

```
<incomingStream>
    rank(
        desc(<sortColumn1>),
        asc(<sortColumn2>),
        ...,
        caseInsensitive: { true | false }
        dense: { true | false }
        output(<rankColumn> as long)
    ) ~> <sortTransformationName<>
```

### <a name="example"></a>Exemple

![Rank settings](media/data-flow/rank-configuration.png "Paramètres de classement")

Le script de flux de données correspondant à la configuration de classement ci-dessus est présenté dans l’extrait de code suivant.

```
PruneColumns
    rank(
        desc(PTS, true),
        caseInsensitive: false,
        output(pointsRanking as long),
        dense: false
    ) ~> RankByPoints
```

## <a name="next-steps"></a>Étapes suivantes

Filtrez les lignes en fonction des valeurs de classement à l’aide de la [transformation de filtre](data-flow-filter.md).
