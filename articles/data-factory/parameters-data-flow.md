---
title: Paramétrage de flux de données de mappage
description: Découvrez comment paramétrer un flux de données de mappage à partir de pipelines Data Factory
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.date: 05/01/2020
ms.openlocfilehash: 564c7cf6e9627db08d543b964ce476e71bfb473d
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93040742"
---
# <a name="parameterizing-mapping-data-flows"></a>Paramétrage de flux de données de mappage

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)] 

Les flux de données de mappage dans Azure Data Factory prennent en charge l’utilisation de paramètres. Définissez des paramètres dans votre définition de flux de données, et utilisez-les dans vos expressions. Les valeurs de paramètre sont définies par le pipeline appelant via l’activité d’exécution de flux de données. Vous disposez de trois options pour définir les valeurs dans les expressions d’activité de flux de données :

* Utiliser le langage d’expression de flux de contrôle de pipeline pour définir une valeur dynamique
* Utiliser le langage d’expression de flux de données pour définir une valeur dynamique
* Utiliser un langage d’expression pour définir une valeur littérale statique

Utilisez cette fonctionnalité pour rendre vos flux de données polyvalents, flexibles et réutilisables. Vous pouvez définir les paramètres de flux de données et les expressions à l’aide de ces paramètres.

## <a name="create-parameters-in-a-mapping-data-flow"></a>Créer des paramètres dans un flux de données de mappage

Pour ajouter des paramètres à votre flux de données, cliquez sur la partie vide du canevas de celui-ci afin d’accéder aux propriétés générales. Dans le volet des paramètres, vous voyez un onglet nommé **Paramètre**. Sélectionnez **Nouveau** pour générer un nouveau paramètre. Pour chaque paramètre, vous devez attribuer un nom, sélectionner un type et éventuellement définir une valeur par défaut.

![Créer des paramètres de flux de données](media/data-flow/create-params.png "Créer des paramètres de flux de données")

## <a name="use-parameters-in-a-mapping-data-flow"></a>Utiliser des paramètres dans un flux de données de mappage 

Des paramètres peuvent être référencés dans toute expression de flux de données. Les paramètres commencent par $ et sont immuables. Vous trouverez la liste des paramètres disponibles dans le Générateur d’expressions, sous l’onglet **Paramètres**.

![Capture d’écran montrant les paramètres disponibles sous l’onglet Paramètres.](media/data-flow/parameter-expression.png "Expression de paramètre de flux de données")

Vous pouvez ajouter rapidement des paramètres supplémentaires en sélectionnant **Nouveau paramètre** et en spécifiant le nom et le type.

![Capture d’écran montrant les paramètres sous l’onglet Paramètres avec les nouveaux paramètres ajoutés.](media/data-flow/new-parameter-expression.png "Expression de paramètre de flux de données")

## <a name="assign-parameter-values-from-a-pipeline"></a>Assigner des valeurs de paramètre à partir d’un pipeline

Une fois que vous avez créé votre flux de données avec des paramètres, vous pouvez l’exécuter à partir d’un pipeline avec l’activité d’exécution de flux de données. Après avoir ajouté l’activité à votre canevas de pipeline, les paramètres de flux de données disponibles vous sont présentés dans l’onglet **Paramètres** de l’activité.

Quand vous affectez des valeurs de paramètre, vous pouvez utiliser le [langage d’expression de pipeline](control-flow-expression-language-functions.md) ou le [langage d’expression de flux de données](data-flow-expression-functions.md) en fonction des types Spark. Chaque flux de données de mappage peut comporter n’importe quelle combinaison de paramètres d’expression de pipeline et de flux de données.

![Capture d’écran montrant l’onglet Paramètres avec l’expression de flux de données sélectionnée pour la valeur de myparam.](media/data-flow/parameter-assign.png "Définition d’un paramètre de flux de données")

### <a name="pipeline-expression-parameters"></a>Paramètres d’expression de pipeline

Les paramètres d’expression de pipeline vous permettent de référencer des variables système, des fonctions, des paramètres de pipeline et des variables similaires à d’autres activités de pipeline. Quand vous cliquez sur **Pipeline expression** (Expression de pipeline), un volet de navigation s’ouvre sur le côté pour vous permettre d’entrer une expression à l’aide du Générateur d’expressions.

![Capture d’écran montrant le volet du générateur d’expressions.](media/data-flow/parameter-pipeline.png "Définition d’un paramètre de flux de données")

Une fois référencés, les paramètres de pipeline sont évalués, puis leur valeur est utilisée dans le langage d’expression de flux de données. Le type d’expression de pipeline n’a pas besoin de correspondre au type de paramètre de flux de données. 

#### <a name="string-literals-vs-expressions"></a>Littéraux de chaîne et expressions

Quand vous affectez un paramètre d’expression de pipeline de type chaîne, par défaut, des guillemets sont ajoutés, et la valeur est évaluée en tant que littéral. Pour lire la valeur de paramètre en tant qu’expression de flux de données, cochez la zone d’expression à côté du paramètre.

![Capture d’écran montrant le volet Paramètres du flux de données avec l’option Expression sélectionnée pour un paramètre.](media/data-flow/string-parameter.png "Définition d’un paramètre de flux de données")

Si le paramètre de flux de données `stringParam` référence un paramètre de pipeline avec la valeur `upper(column1)`. 

- Si l’option expression est cochée, `$stringParam` prend la valeur de column1 en majuscules.
- Si l’option expression n’est pas cochée (comportement par défaut), `$stringParam` prend la valeur de `'upper(column1)'`

#### <a name="passing-in-timestamps"></a>Passage d’horodatages

Dans le langage d’expression de pipeline, les variables système telles que `pipeline().TriggerTime` et les fonctions telles que `utcNow()` retournent les horodatages sous forme de chaînes au format « aaaa-MM-jj\'T\'HH:mm:ss.SSSSSSZ ». Pour les convertir en paramètres de flux de données de type horodatage, utilisez l’interpolation de chaîne afin d’inclure l’horodatage souhaité dans une fonction `toTimestamp()`. Par exemple, pour convertir l’heure de déclenchement du pipeline en paramètre de flux de données, vous pouvez utiliser `toTimestamp(left('@{pipeline().TriggerTime}', 23), 'yyyy-MM-dd\'T\'HH:mm:ss.SSS')`. 

![Capture d’écran montrant l’onglet Paramètres dans lequel vous pouvez entrer une heure de déclenchement.](media/data-flow/parameter-timestamp.png "Définition d’un paramètre de flux de données")

> [!NOTE]
> Les flux de données peuvent prendre en charge 3 millisecondes au maximum. La fonction `left()` est utilisée pour supprimer les chiffres supplémentaires.

#### <a name="pipeline-parameter-example"></a>Exemple de paramètre de pipeline

Vous avez un paramètre entier `intParam` qui référence un paramètre de pipeline de type String, `@pipeline.parameters.pipelineParam`. 

![Capture d’écran montrant l’onglet Paramètres avec les paramètres nommés stringParam et intParam.](media/data-flow/parameter-pipeline-2.png "Définition d’un paramètre de flux de données")

`@pipeline.parameters.pipelineParam` se voit affecter la valeur `abs(1)` au moment de l’exécution.

![Capture d’écran montrant l’onglet Paramètres avec la valeur abs(1) sélectionnée.](media/data-flow/parameter-pipeline-4.png "Définition d’un paramètre de flux de données")

Quand `$intParam` est référencé dans une expression telle qu’une colonne dérivée, il évalue que `abs(1)` retourne `1`. 

![Capture d’écran montrant la valeur des colonnes.](media/data-flow/parameter-pipeline-3.png "Définition d’un paramètre de flux de données")

### <a name="data-flow-expression-parameters"></a>Paramètres d’expression de flux de données

Sélectionnez **Data flow expression** (Expression de flux de données) pour ouvrir le Générateur d’expressions de flux de données. Vous pourrez référencer des fonctions, d’autres paramètres et toute colonne de schéma définie dans votre flux de données. Cette expression est évaluée telle quelle quand elle est référencée.

> [!NOTE]
> Si vous passez une expression non valide ou si vous référencez une colonne de schéma qui n’existe pas dans cette transformation, le paramètre prend une valeur null.


### <a name="passing-in-a-column-name-as-a-parameter"></a>Transmission d’un nom de colonne en tant que paramètre

Un modèle courant consiste à passer un nom de colonne en tant que valeur de paramètre. Si la colonne est définie dans le schéma de flux de données, vous pouvez la référencer directement en tant qu’expression de chaîne. Si la colonne n’est pas définie dans le schéma, utilisez la fonction `byName()`. Veillez à effectuer un cast de la colonne vers son type approprié à l’aide d’une fonction de conversion telle que `toString()`.

Par exemple, si vous souhaitez mapper une colonne de chaîne basée sur un paramètre `columnName`, vous pouvez ajouter une transformation de colonne dérivée égale à `toString(byName($columnName))`.

![Transmission d’un nom de colonne en tant que paramètre](media/data-flow/parameterize-column-name.png "Transmission d’un nom de colonne en tant que paramètre")

## <a name="next-steps"></a>Étapes suivantes
* [Exécuter une activité de flux de données](control-flow-execute-data-flow-activity.md)
* [Expressions de flux de contrôle](control-flow-expression-language-functions.md)
