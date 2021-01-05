---
title: Guide de programmabilité de l’applicateur défini par l’utilisateur U-SQL pour Azure Data Lake
description: En savoir plus sur le guide de programmabilité U-SQL UDO - applicateur défini par l’utilisateur.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: 0842a2cfa021ef8ea45c19ec885c7dec371730de
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96512432"
---
# <a name="use-user-defined-applier"></a>Utiliser un applicateur défini par l’utilisateur 

## <a name="u-sql-udo-user-defined-applier"></a>U-SQL UDO : applicateur défini par l’utilisateur
Un applicateur U-SQL défini par l’utilisateur vous permet d’appeler une fonction C# personnalisée pour chaque ligne retournée par une expression de la table externe d’une requête. L’entrée appropriée est évaluée pour chaque ligne à partir de l’entrée de gauche et les lignes qui sont produites sont combinées pour la sortie finale. La liste des colonnes produites par l’opérateur APPLY est la combinaison des jeux de colonnes dans les entrées de gauche et de droite.


## <a name="how-to-define-and-use-user-defined-applier"></a>Définition et utilisation d’un applicateur défini par l’utilisateur
Un applicateur défini par l’utilisateur est appelé dans l’expression U-SQL SELECT.

L’appel standard à l’applicateur défini par l’utilisateur se présente comme suit :

```usql
SELECT …
FROM …
CROSS APPLYis used to pass parameters
new MyScript.MyApplier(param1, param2) AS alias(output_param1 string, …);
```

Pour plus d’informations sur l’utilisation d’applicateurs dans une expression SELECT, voir [U-SQL SELECT Selecting from CROSS APPLY and OUTER APPLY](/u-sql/statements-and-expressions/select/from/select-selecting-from-cross-apply-and-outer-apply) (Sélection d’U-SQL SELECT à partir de CROSS APPLY et de OUTER APPLY).

La définition de classe de base d’applicateur défini par l’utilisateur est la suivante :

```csharp
public abstract class IApplier : IUserDefinedOperator
{
protected IApplier();

public abstract IEnumerable<IRow> Apply(IRow input, IUpdatableRow output);
}
```

Pour définir un applicateur défini par l’utilisateur, nous devons créer l’interface `IApplier` avec l’attribut [`SqlUserDefinedApplier`] qui est facultatif pour une définition d’applicateur défini par l’utilisateur.

```csharp
[SqlUserDefinedApplier]
public class ParserApplier : IApplier
{
    public ParserApplier()
    {
        …
    }

    public override IEnumerable<IRow> Apply(IRow input, IUpdatableRow output)
    {
        …
    }
}
```

* Apply est appelé pour chaque ligne de la table externe. Il retourne l’ensemble de lignes de sortie `IUpdatableRow`.
* La classe de constructeur est utilisée pour transmettre des paramètres à l’applicateur défini par l’utilisateur.

**SqlUserDefinedApplier** indique que le type doit être inscrit en tant qu’un applicateur défini par l’utilisateur. Cette classe ne peut pas être héritée.

**SqlUserDefinedApplier** est **facultatif** pour une définition d’applicateur défini par l’utilisateur.


Les principaux objets de programmabilité sont les suivants :

```csharp
public override IEnumerable<IRow> Apply(IRow input, IUpdatableRow output)
```

Les ensembles de lignes d’entrée sont transmis en tant qu’entrée `IRow`. Les lignes de sortie sont générées en tant qu’interface de sortie `IUpdatableRow`.

Les noms des colonnes peuvent être déterminés en appelant la méthode de schéma `IRow`.

```csharp
ISchema schema = row.Schema;
var col = schema[i];
string val = row.Get<string>(col.Name)
```

Pour obtenir les valeurs de données réelles du `IRow` entrant, nous utilisons la méthode Get() de l’interface `IRow`.

```csharp
mycolumn = row.Get<int>("mycolumn")
```

Ou bien, nous utilisons le nom de colonne de schéma :

```csharp
row.Get<int>(row.Schema[0].Name)
```

Les valeurs de sortie doivent être définies avec la sortie `IUpdatableRow` :

```csharp
output.Set<int>("mycolumn", mycolumn)
```

Il est important de comprendre qu’un applicateur personnalisé génère uniquement des colonnes et des valeurs qui sont définies avec l’appel de la méthode `output.Set`.

La sortie réelle est déclenchée par l’appel de `yield return output.AsReadOnly();`.

Les paramètres de l’applicateur défini par l’utilisateur peuvent être transmis au constructeur. Un applicateur peut retourner un nombre variable de colonnes, qui doit être défini durant l’appel de l’applicateur dans le script U-SQL de base.

```csharp
new USQL_Programmability.ParserApplier ("all") AS properties(make string, model string, year string, type string, millage int);
```

Voici l’exemple d’applicateur défini par l’utilisateur :

```csharp
[SqlUserDefinedApplier]
public class ParserApplier : IApplier
{
private string parsingPart;

public ParserApplier(string parsingPart)
{
    if (parsingPart.ToUpper().Contains("ALL")
    || parsingPart.ToUpper().Contains("MAKE")
    || parsingPart.ToUpper().Contains("MODEL")
    || parsingPart.ToUpper().Contains("YEAR")
    || parsingPart.ToUpper().Contains("TYPE")
    || parsingPart.ToUpper().Contains("MILLAGE")
    )
    {
    this.parsingPart = parsingPart;
    }
    else
    {
    throw new ArgumentException("Incorrect parameter. Please use: 'ALL[MAKE|MODEL|TYPE|MILLAGE]'");
    }
}

public override IEnumerable<IRow> Apply(IRow input, IUpdatableRow output)
{

    string[] properties = input.Get<string>("properties").Split(',');

    //  only process with correct number of properties
    if (properties.Count() == 5)
    {

    string make = properties[0];
    string model = properties[1];
    string year = properties[2];
    string type = properties[3];
    int millage = -1;

    // Only return millage if it is number, otherwise, -1
    if (!int.TryParse(properties[4], out millage))
    {
        millage = -1;
    }

    if (parsingPart.ToUpper().Contains("MAKE") || parsingPart.ToUpper().Contains("ALL")) output.Set<string>("make", make);
    if (parsingPart.ToUpper().Contains("MODEL") || parsingPart.ToUpper().Contains("ALL")) output.Set<string>("model", model);
    if (parsingPart.ToUpper().Contains("YEAR") || parsingPart.ToUpper().Contains("ALL")) output.Set<string>("year", year);
    if (parsingPart.ToUpper().Contains("TYPE") || parsingPart.ToUpper().Contains("ALL")) output.Set<string>("type", type);
    if (parsingPart.ToUpper().Contains("MILLAGE") || parsingPart.ToUpper().Contains("ALL")) output.Set<int>("millage", millage);
    }
    yield return output.AsReadOnly();            
}
}
```

Le script U-SQL de base pour cet applicateur défini par l’utilisateur est le suivant :

```usql
DECLARE @input_file string = @"c:\usql-programmability\car_fleet.tsv";
DECLARE @output_file string = @"c:\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
        stocknumber int,
        vin String,
        properties String
    FROM @input_file USING Extractors.Tsv();

@rs1 =
    SELECT
        r.stocknumber,
        r.vin,
        properties.make,
        properties.model,
        properties.year,
        properties.type,
        properties.millage
    FROM @rs0 AS r
    CROSS APPLY
    new USQL_Programmability.ParserApplier ("all") AS properties(make string, model string, year string, type string, millage int);

OUTPUT @rs1 TO @output_file USING Outputters.Text();
```

Dans ce scénario d’utilisation, l’applicateur défini par l’utilisateur agit comme un analyseur de valeurs délimitées par des virgules pour les propriétés d’une flotte de voitures. Les lignes du fichier d’entrée ressemblent à ceci :

```text
103 Z1AB2CD123XY45889   Ford,Explorer,2005,SUV,152345
303 Y0AB2CD34XY458890   Chevrolet,Cruise,2010,4Dr,32455
210 X5AB2CD45XY458893   Nissan,Altima,2011,4Dr,74000
```

Il s’agit d’un fichier TSV standard de valeurs délimitées par des tabulations avec une colonne Propriétés contenant des propriétés de voitures telles que la marque et le modèle. Ces propriétés doivent être analysées dans les colonnes de table. L’applicateur fourni vous permet également de générer un nombre dynamique de propriétés dans l’ensemble de lignes de résultat, en fonction du paramètre transmis. Vous pouvez générer soit toutes les propriétés, soit un ensemble spécifique de propriétés.

```text
...USQL_Programmability.ParserApplier ("all")
...USQL_Programmability.ParserApplier ("make")
...USQL_Programmability.ParserApplier ("make&model")
```

L’applicateur défini par l’utilisateur peut être appelé en tant que nouvelle instance de l’objet applicateur :

```usql
CROSS APPLY new MyNameSpace.MyApplier (parameter: "value") AS alias([columns types]…);
```

Ou par l’appel d’une méthode de fabrique de wrapper :

```csharp
    CROSS APPLY MyNameSpace.MyApplier (parameter: "value") AS alias([columns types]…);
```


## <a name="next-steps"></a>Étapes suivantes
* [Guide de programmabilité U-SQL - vue d’ensemble](data-lake-analytics-u-sql-programmability-guide.md)
* [Guide de programmabilité U-SQL - UDT et UDAGG](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)