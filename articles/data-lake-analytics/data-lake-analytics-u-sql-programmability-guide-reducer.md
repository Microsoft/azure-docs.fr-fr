---
title: Guide de programmabilité du réducteur U-SQL défini par l’utilisateur pour Azure Data Lake
description: En savoir plus sur le guide de programmabilité U-SQL UDO - réducteur défini par l’utilisateur.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: 52d44685678c3e89dc820042a7925d284500cef8
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96512430"
---
# <a name="use-user-defined-reducer"></a>Utiliser un réducteur défini par l’utilisateur

## <a name="u-sql-udo-user-defined-reducer"></a>U-SQL UDO : réducteur défini par l’utilisateur

U-SQL vous permet d’écrire des réducteurs d’ensemble de lignes personnalisés en C# en utilisant l’infrastructure d’extensibilité d’opérateur définie par l’utilisateur et en implémentant une interface IReducer.

Un réducteur défini par l’utilisateur (ou UDR) permet d’éliminer les lignes inutiles lors de l’extraction de données (importation). Il permet également de manipuler et d’évaluer les lignes et des colonnes. Sur la base d’une logique de programmabilité, il permet également de définir les lignes à extraire.

## <a name="how-to-define-and-use-user-defined-reducer"></a>Définition et utilisation d’un réducteur défini par l’utilisateur
Pour définir une classe UDR, nous devons créer une interface `IReducer` avec un attribut facultatif `SqlUserDefinedReducer`.

Cette interface de classe doit contenir une définition pour le remplacement de l’ensemble de lignes d’interface `IEnumerable`.

```csharp
[SqlUserDefinedReducer]
public class EmptyUserReducer : IReducer
{

    public override IEnumerable<IRow> Reduce(IRowset input, IUpdatableRow output)
    {
        …
    }

}
```

L’attribut **SqlUserDefinedReducer** indique que le type doit être inscrit en tant que réducteur défini par l’utilisateur. Cette classe ne peut pas être héritée.
**SqlUserDefinedReducer** est un attribut facultatif pour une définition de réducteur défini par l’utilisateur. Il est utilisé pour définir la propriété de IsRecursive.

* bool     IsRecursive    
* **true** = indique si le réducteur est associatif et commutatif

Les principaux objets de programmabilité sont l’**entrée** et la **sortie**. L’objet d’entrée permet d’énumérer les lignes d’entrée. La sortie est utilisée pour définir les lignes de sortie suite à l’activité de réduction.

Pour l’énumération des lignes d’entrée, nous utilisons la méthode `Row.Get`.

```csharp
foreach (IRow row in input.Rows)
{
    row.Get<string>("mycolumn");
}
```

Le paramètre pour la méthode `Row.Get` est une colonne transmise dans une classe `PRODUCE` de l’instruction `REDUCE` du script U-SQL de base. Nous devons utiliser le type de données correct ici également.

Pour la sortie, utilisez la méthode `output.Set`.

Il est important de comprendre qu’un réducteur personnalisé ne génère que des valeurs définies avec l’appel de la méthode `output.Set`.

```csharp
output.Set<string>("mycolumn", guid);
```

La sortie réelle du réducteur est déclenchée par l’appel de `yield return output.AsReadOnly();`.

Voici un exemple de réducteur :

```csharp
[SqlUserDefinedReducer]
public class EmptyUserReducer : IReducer
{

    public override IEnumerable<IRow> Reduce(IRowset input, IUpdatableRow output)
    {
    string guid;
    DateTime dt;
    string user;
    string des;

    foreach (IRow row in input.Rows)
    {
        guid = row.Get<string>("guid");
        dt = row.Get<DateTime>("dt");
        user = row.Get<string>("user");
        des = row.Get<string>("des");

        if (user.Length > 0)
        {
        output.Set<string>("guid", guid);
        output.Set<DateTime>("dt", dt);
        output.Set<string>("user", user);
        output.Set<string>("des", des);

        yield return output.AsReadOnly();
        }
    }
    }

}
```

Dans ce scénario d’utilisation, le réducteur ignore les lignes dans lesquelles le nom d’utilisateur est vide. Pour chaque ligne de l’ensemble de lignes, il lit chaque colonne requise, puis évalue la longueur du nom d’utilisateur. Il génère la ligne réelle uniquement si la longueur du nom utilisateur est supérieure à 0.

Le script U-SQL de base utilisant un réducteur personnalisé est le suivant :

```usql
DECLARE @input_file string = @"\usql-programmability\input_file_reducer.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
            guid string,
        dt DateTime,
            user String,
            des String
    FROM @input_file 
    USING Extractors.Tsv();

@rs1 =
    REDUCE @rs0 PRESORT guid
    ON guid  
    PRODUCE guid string, dt DateTime, user String, des String
    USING new USQL_Programmability.EmptyUserReducer();

@rs2 =
    SELECT guid AS start_id,
           dt AS start_time,
           DateTime.Now.ToString("M/d/yyyy") AS Nowdate,
           USQL_Programmability.CustomFunctions.GetFiscalPeriodWithCustomType(dt).ToString() AS start_fiscalperiod,
           user,
           des
    FROM @rs1;

OUTPUT @rs2 
    TO @output_file 
    USING Outputters.Text();
```

## <a name="next-steps"></a>Étapes suivantes
* [Guide de programmabilité U-SQL - vue d’ensemble](data-lake-analytics-u-sql-programmability-guide.md)
* [Guide de programmabilité U-SQL - UDT et UDAGG](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)