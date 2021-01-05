---
title: Transformation de colonne dérivée dans le mappage Data Flow
description: Découvrez comment transformer des données à l’échelle dans Azure Data Factory avec la transformation de colonne dérivée du mappage Data Flow.
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 09/14/2020
ms.openlocfilehash: 2e90a8779322cf8967ca9a194c6cc760f7c8b8f5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90532023"
---
# <a name="derived-column-transformation-in-mapping-data-flow"></a>Transformation de colonne dérivée dans le mappage Data Flow

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Utilisez la transformation de colonne dérivée pour générer de nouvelles colonnes dans votre flux de données ou pour modifier des champs existants.

## <a name="create-and-update-columns"></a>Créer et mettre à jour des colonnes

Lors de la création d’une colonne dérivée, vous pouvez soit générer une nouvelle colonne, soit en mettre une à jour. Dans la zone de texte **Colonne**, entrez dans la colonne que vous êtes en train de créer. Pour remplacer une colonne existante dans votre schéma, vous pouvez utiliser la liste déroulante Colonne. Pour générer l’expression de la colonne dérivée, cliquez sur la zone de texte **Entrer une expression**. Vous pouvez soit commencer à entrer votre expression, soit ouvrir le générateur d’expressions pour construire votre logique.

![Paramètres de la colonne dérivée](media/data-flow/create-derive-column.png "Paramètres de la colonne dérivée")

Pour ajouter des colonnes dérivées, cliquez sur **Ajouter** au-dessus de la liste des colonnes ou de l’icône « plus » en regard d’une colonne dérivée existante. Choisissez **Ajouter une colonne** ou **Ajouter un modèle de colonne**.

![Nouvelle sélection de colonne dérivée](media/data-flow/add-derived-column.png "Nouvelle sélection de colonne dérivée")

### <a name="column-patterns"></a>Modèles de colonne

Dans les cas où votre schéma n’est pas explicitement défini ou si vous souhaitez mettre à jour un ensemble de colonnes en bloc, vous pouvez créer des modèles de colonne. Les modèles de colonne vous permettent de faire correspondre les colonnes à l’aide de règles basées sur les métadonnées de colonne et de créer des colonnes dérivées pour chaque colonne correspondante. Pour plus d’informations, consultez [Comment générer des modèles de colonne](concepts-data-flow-column-pattern.md#column-patterns-in-derived-column-and-aggregate) dans la transformation de colonne dérivée.

![Modèles de colonne](media/data-flow/column-pattern-derive.png "Modèles de colonne")

## <a name="building-schemas-using-the-expression-builder"></a>Génération de schémas à l’aide du générateur d’expressions

Lors de l’utilisation du [générateur d’expressions](concepts-data-flow-expression-builder.md) du flux de données de mappage, vous pouvez créer, modifier et gérer vos colonnes dérivées dans la section **Colonnes dérivées**. Toutes les colonnes créées ou modifiées dans la transformation sont répertoriées. Choisissez de manière interactive la colonne ou le modèle que vous modifiez en cliquant sur le nom de la colonne. Pour ajouter une colonne supplémentaire, sélectionnez **Créer** et choisissez si vous souhaitez ajouter une seule colonne ou un modèle.

![Créer une colonne](media/data-flow/derive-add-column.png "Créer une colonne")

Lorsque vous travaillez avec des colonnes complexes, vous pouvez créer des sous-colonnes. Pour ce faire, cliquez sur l’icône « plus » en regard d’une colonne et sélectionnez **Ajouter une sous-colonne**. Pour plus d’informations sur la gestion des types complexes dans le flux de données, consultez [Traitement JSON dans le flux de données de mappage](format-json.md#mapping-data-flow-properties).

![Ajouter une sous-colonne](media/data-flow/derive-add-subcolumn.png "Ajouter une sous-colonne")

Pour plus d’informations sur la gestion des types complexes dans le flux de données, consultez [Traitement JSON dans le flux de données de mappage](format-json.md#mapping-data-flow-properties).

![Ajouter une colonne complexe](media/data-flow/derive-complex-column.png "Ajouter des colonnes")

### <a name="locals"></a>Locals

Si vous partagez la logique entre plusieurs colonnes ou si vous souhaitez compartimenter votre logique, vous pouvez créer une locale dans une transformation de colonne dérivée. Une locale est un ensemble de logique qui n’est pas propagé en aval à la transformation suivante. Vous pouvez créer une locale dans le générateur d’expressions en accédant à **Éléments d’expression** et en sélectionnant **Locales**. Créez-en un en sélectionnant **Créer**.

![Créer une locale](media/data-flow/create-local.png "Créer une locale")

Les locales peuvent référencer n’importe quel élément d’expression d’une colonne dérivée, comme des fonctions, un schéma d’entrée, des paramètres et d’autres locales. Lorsque vous référencez d’autres locales, l’ordre est important, car la locale référencée doit être « au-dessus » de l’objet actuel.

![Créer une locale 2](media/data-flow/create-local-2.png "Créer une locale 2")

Pour faire référence à une locale dans une colonne dérivée, cliquez sur le local à partir de la vue **Éléments d’expression** ou référencez-le avec un signe deux-points devant son nom. Par exemple, une variable locale appelée local1 est référencée par `:local1`. Pour modifier une définition de locale, pointez dessus dans la vue Éléments d’expression, puis cliquez sur l’icône de crayon.

![Utilisation de locales](media/data-flow/using-locals.png "Utilisation de locales")

## <a name="data-flow-script"></a>Script de flux de données

### <a name="syntax"></a>Syntaxe

```
<incomingStream>
    derive(
           <columnName1> = <expression1>,
           <columnName2> = <expression2>,
           each(
                match(matchExpression),
                <metadataColumn1> = <metadataExpression1>,
                <metadataColumn2> = <metadataExpression2>
               )
          ) ~> <deriveTransformationName>
```

### <a name="example"></a>Exemple

L’exemple ci-dessous est une colonne dérivée nommée `CleanData` qui prend un flux entrant `MoviesYear` et crée deux colonnes dérivées. La première colonne dérivée remplace la colonne `Rating` par la valeur de l’évaluation en tant que type entier. La deuxième colonne dérivée est un modèle qui correspond à chaque colonne dont le nom commence par « movies ». Pour chaque colonne correspondante, il crée une colonne `movie` qui est égale à la valeur de la colonne correspondante avec le préfixe « movie_ ». 

Dans l’expérience utilisateur Data Factory, cette transformation se présente comme dans l’image ci-dessous :

![Exemple de dérivation](media/data-flow/derive-script.png "Exemple de dérivation")

Le script de flux de données pour cette transformation se trouve dans l’extrait de code ci-dessous :

```
MoviesYear derive(
                Rating = toInteger(Rating),
                each(
                    match(startsWith(name,'movies')),
                    'movie' = 'movie_' + toString($$)
                )
            ) ~> CleanData
```

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur le [langage d’expression de mappage Data Flow](data-flow-expression-functions.md).
