---
title: Transformation Exists dans le flux de données de mappage
description: Vérifier l’existence de lignes à l’aide de la transformation Exists dans le flux de données de mappage Azure Data Factory
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 05/07/2020
ms.openlocfilehash: 805b51bf4e6d8feab9539f660dfc72ca78b82d5c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82982630"
---
# <a name="exists-transformation-in-mapping-data-flow"></a>Transformation Exists dans le flux de données de mappage

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

La transformation Exists est une transformation de filtrage de lignes qui vérifie si vos données existent dans une autre source ou un autre flux. Le flux de sortie comprend toutes les lignes du flux de gauche qui existent ou non dans le flux de droite. La transformation Exists est similaire à ```SQL WHERE EXISTS``` et ```SQL WHERE NOT EXISTS```.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4vZKz]

## <a name="configuration"></a>Configuration

1. Choisissez le flux de données dont vous vérifiez l’existence dans la liste déroulante **Flux de droite**.
1. Indiquez si vous recherchez les données qui existent ou qui n’existent pas avec le paramètre **Type d’existence**.
1. Indiquez si vous souhaitez ou non une **expression personnalisée**.
1. Choisissez les colonnes clés que vous souhaitez comparer comme conditions d’existence. Par défaut, le flux de données recherche l’équivalence entre une colonne d’un flux et une colonne de l’autre flux. Pour effectuer une comparaison à l’aide d’une valeur calculée, pointez sur la liste déroulante de la colonne, puis sélectionnez **Colonne calculée**.

![Paramètres d’existence](media/data-flow/exists.png "Exists 1")

### <a name="multiple-exists-conditions"></a>Conditions Exists multiples

Pour comparer plusieurs colonnes de chaque flux, ajoutez une nouvelle condition d’existence en cliquant sur l’icône plus en regard d’une ligne existante. Chaque condition supplémentaire est jointe par une instruction « and ». La comparaison de deux colonnes correspond à l’expression suivante :

`source1@column1 == source2@column1 && source1@column2 == source2@column2`

### <a name="custom-expression"></a>Expression personnalisée

Pour créer une expression de forme libre qui contient des opérateurs autres que « and » et « equals to », cochez la case **Expression personnalisée**. Entrez une expression personnalisée par le biais du générateur d’expressions de flux de données en cliquant sur la zone bleue.

![Paramètres personnalisés de transformation Exists](media/data-flow/exists1.png "Transformation Exists personnalisée")

## <a name="broadcast-optimization"></a>Optimisation de la diffusion

![Jonction de diffusion](media/data-flow/broadcast.png "Jonction de diffusion")

Dans les transformations de jointures, de recherches et d’existences, si l’un des flux de données ou les deux tiennent dans la mémoire de nœud Worker, vous pouvez optimiser les performances en activant la **diffusion**. Par défaut, le moteur Spark détermine automatiquement s’il faut diffuser $$$d’un côté. Pour choisir manuellement $$$le côté à diffuser, sélectionnez **$$$Fixe**.

Il n’est pas recommandé de désactiver la diffusion à l’aide de l’option **Désactivé$$$** à moins que vos jointures ne rencontrent des erreurs de délai d’attente.

## <a name="data-flow-script"></a>Script de flux de données

### <a name="syntax"></a>Syntaxe

```
<leftStream>, <rightStream>
    exists(
        <conditionalExpression>,
        negate: { true | false },
        broadcast: { 'auto' | 'left' | 'right' | 'both' | 'off' }
    ) ~> <existsTransformationName>
```

### <a name="example"></a> Exemple

L’exemple ci-dessous illustre une transformation Exists nommée `checkForChanges`, qui utilise le flux de gauche `NameNorm2` et le flux de droite `TypeConversions`.  La condition d’existence est l’expression `NameNorm2@EmpID == TypeConversions@EmpID && NameNorm2@Region == DimEmployees@Region` qui retourne true si les colonnes `EMPID` et `Region` de chaque flux correspondent. Comme nous vérifions l’existence, `negate` a la valeur false. Par ailleurs, comme nous n’autorisons aucune diffusion dans l’onglet Optimiser, `broadcast` a la valeur `'none'`.

Dans l’expérience utilisateur Data Factory, cette transformation se présente comme dans l’image ci-dessous :

![Exemple de transformation Exists](media/data-flow/exists-script.png "Exemple de transformation Exists")

Le script de flux de données pour cette transformation se trouve dans l’extrait de code ci-dessous :

```
NameNorm2, TypeConversions
    exists(
        NameNorm2@EmpID == TypeConversions@EmpID && NameNorm2@Region == DimEmployees@Region,
        negate:false,
        broadcast: 'auto'
    ) ~> checkForChanges
```

## <a name="next-steps"></a>Étapes suivantes

Les transformations [Lookup](data-flow-lookup.md) et [Join](data-flow-join.md) sont similaires.
