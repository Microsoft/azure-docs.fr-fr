---
title: Guide de programmabilité du processeur défini par l’utilisateur U-SQL pour Azure Data Lake
description: En savoir plus sur le guide de programmabilité U-SQL UDO - générateur de sortie défini par l’utilisateur.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: 6ff45c577e94a8c63bd7cb1e6603e4d5519af5c6
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96512440"
---
# <a name="use-user-defined-processor"></a>Utiliser un processeur défini par l’utilisateur

## <a name="u-sql-udo-user-defined-processor"></a>U-SQL UDO : processeur défini par l’utilisateur
Un processeur défini par l’utilisateur (ou UDP) est un type d’UDO U-SQL qui permet de traiter les lignes entrantes en appliquant des fonctionnalités de programmabilité. Un UDP permet de combiner des colonnes, de modifier des valeurs et d’ajouter des colonnes si nécessaire. En fait, il permet de traiter un ensemble de lignes pour produire des éléments de données requis.

## <a name="how-to-define-and-use-user-defined-processor"></a>Définition et utilisation d’un processeur défini par l’utilisateur
Pour définir un UDP, nous devons créer une interface `IProcessor` avec l’attribut `SqlUserDefinedProcessor`, facultatif pour l’UDP.

Cette interface doit contenir la définition pour le remplacement de l’ensemble de lignes de l’interface `IRow`, comme indiqué dans l’exemple suivant :

```csharp
[SqlUserDefinedProcessor]
public class MyProcessor: IProcessor
{
public override IRow Process(IRow input, IUpdatableRow output)
 {
        …
 }
}
```

**SqlUserDefinedProcessor** indique que le type doit être inscrit en tant que processeur défini par l’utilisateur. Cette classe ne peut pas être héritée.

L’attribut SqlUserDefinedProcessor est **facultatif** pour la définition d’UDP.

Les principaux objets de programmabilité sont l’**entrée** et la **sortie**. L’objet d’entrée permet d’énumérer les colonnes d’entrée et la sortie, et de définir les données de sortie en tant que résultat de l’activité du processeur.

Pour l’énumération des colonnes d’entrée, nous utilisons la méthode `input.Get`.

```csharp
string column_name = input.Get<string>("column_name");
```

Le paramètre pour la méthode `input.Get` est une colonne transmise dans une clause `PRODUCE` de l’instruction `PROCESS` du script U-SQL de base. Nous devons utiliser le type de données ici.

Pour la sortie, utilisez la méthode `output.Set`.

Il est important de noter qu’un producteur personnalisé ne génère que des colonnes et des valeurs qui sont définies avec l’appel de la méthode `output.Set`.

```csharp
output.Set<string>("mycolumn", mycolumn);
```

La sortie réelle du processeur est déclenchée par l’appel de `return output.AsReadOnly();`.

Voici un exemple de processeur :

```csharp
[SqlUserDefinedProcessor]
public class FullDescriptionProcessor : IProcessor
{
public override IRow Process(IRow input, IUpdatableRow output)
 {
     string user = input.Get<string>("user");
     string des = input.Get<string>("des");
     string full_description = user.ToUpper() + "=>" + des;
     output.Set<string>("dt", input.Get<string>("dt"));
     output.Set<string>("full_description", full_description);
     output.Set<Guid>("new_guid", Guid.NewGuid());
     output.Set<Guid>("guid", input.Get<Guid>("guid"));
     return output.AsReadOnly();
 }
}
```

Dans ce scénario d’utilisation, le processeur génère une nouvelle colonne nommée « full_description » en combinant les colonnes existantes, dans ce cas, « user » en majuscules et « des ». Il génère également un GUID et retourne les valeurs de GUID d’origine et nouvelles.

Comme vous pouvez le constater dans l’exemple précédent, vous pouvez appeler des méthodes C# durant l’appel de la méthode `output.Set`.

Vous trouverez ci-dessous un exemple de script U-SQL de base utilisant un processeur personnalisé :

```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
            guid Guid,
        dt String,
            user String,
            des String
    FROM @input_file USING Extractors.Tsv();

@rs1 =
     PROCESS @rs0
     PRODUCE dt String,
             full_description String,
             guid Guid,
             new_guid Guid
     USING new USQL_Programmability.FullDescriptionProcessor();

OUTPUT @rs1 TO @output_file USING Outputters.Text();
```


## <a name="next-steps"></a>Étapes suivantes
* [Guide de programmabilité U-SQL - vue d’ensemble](data-lake-analytics-u-sql-programmability-guide.md)
* [Guide de programmabilité U-SQL - UDT et UDAGG](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)