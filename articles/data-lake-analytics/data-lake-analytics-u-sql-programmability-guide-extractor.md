---
title: Guide de programmabilité de l’extracteur défini par l’utilisateur U-SQL pour Azure Data Lake
description: En savoir plus sur le guide de programmabilité U-SQL UDO - extracteur défini par l’utilisateur.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: ad7f6336753903533771033de21aec8262425a61
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97608015"
---
# <a name="use-user-defined-extractor"></a>Utiliser un extracteur défini par l’utilisateur

## <a name="u-sql-udo-user-defined-extractor"></a>U-SQL UDO : extracteur défini par l’utilisateur
U-SQL vous permet d’importer des données externes en utilisant une instruction EXTRACT. Une instruction EXTRACT peut utiliser des extracteurs UDO intégrés :  

* *Extractors.Text()*  : produit une extraction à partir de fichiers texte délimités d’encodages différents.

* *Extractors.Csv()*  : produit une extraction à partir de fichiers de valeurs séparées par des virgules (CSV) d’encodages différents.

* *Extractors.Tsv()*  : produit une extraction à partir de fichiers de valeurs séparées par des tabulations (TSV) d’encodages différents.

Il peut être utile de développer un extracteur personnalisé. C’est le cas lors de l’importation de données si vous souhaitez effectuer l’une des tâches suivantes :

* Modifier des données d’entrée en fractionnant des colonnes et en modifiant des valeurs individuelles. Il est préférable de combiner des colonnes à l’aide de la fonctionnalité PROCESSOR.
* Analyser des données non structurées telles que des pages web et des messages électroniques, ou des données semi-structurées telles que des fichiers XML/JSON.
* Analyser des données dans un codage non pris en charge.

## <a name="how-to-define-and-use-user-defined-extractor"></a>Définition et utilisation d’un extracteur défini par l’utilisateur
Pour définir un extracteur défini par l’utilisateur (ou UDE), nous devons créer une interface `IExtractor`. Tous les paramètres d’entrée de l’extracteur, tels que les séparateurs de colonnes/lignes et le codage, doivent être définis dans le constructeur de la classe. L’interface `IExtractor` doit également contenir une définition pour le remplacement de `IEnumerable<IRow>` comme suit :

```csharp
[SqlUserDefinedExtractor]
public class SampleExtractor : IExtractor
{
     public SampleExtractor(string row_delimiter, char col_delimiter)
     { … }

     public override IEnumerable<IRow> Extract(IUnstructuredReader input, IUpdatableRow output)
     { … }
}
```

L’attribut **SqlUserDefinedExtractor** indique que le type doit être inscrit en tant qu’extracteur défini par l’utilisateur. Cette classe ne peut pas être héritée.

SqlUserDefinedExtractor est un attribut facultatif pour la définition d’UDE. Il est utilisé pour définir la propriété AtomicFileProcessing pour l’objet UDE.

* bool     AtomicFileProcessing   

* **true** = indique que cet extracteur requiert des fichiers d’entrée atomique (JSON, XML, ...)
* **false** = indique que cet extracteur peut traiter des fichiers fractionnés/distribués (CSV, SEQ, ...)

Les principaux objets de programmabilité d’UDE sont l’**entrée** et la **sortie**. L’objet d’entrée permet d’énumérer les données d’entrée en tant que `IUnstructuredReader`. L’objet de sortie permet de définir les données de sortie en tant que résultat de l’activité de l’extracteur.

Les données d’entrée sont accessibles via `System.IO.Stream` et `System.IO.StreamReader`.

Pour l’énumération des colonnes d’entrée, nous fractionnons d’abord le flux d’entrée en utilisant un séparateur de lignes.

```csharp
foreach (Stream current in input.Split(my_row_delimiter))
{
…
}
```

Ensuite, nous fractionnons la ligne d’entrée en parties de colonne.

```csharp
foreach (Stream current in input.Split(my_row_delimiter))
{
…
    string[] parts = line.Split(my_column_delimiter);
    foreach (string part in parts)
    { … }
}
```

Pour définir les données de sortie, nous utilisons la méthode `output.Set`.

Il est important de comprendre que l’extracteur personnalisé ne génère que des colonnes et des valeurs qui sont définies avec la sortie. Définissez l’appel de méthode.

```csharp
output.Set<string>(count, part);
```

La sortie réelle de l’extracteur est déclenchée par l’appel de `yield return output.AsReadOnly();`.

Voici l’exemple d’extracteur :

```csharp
[SqlUserDefinedExtractor(AtomicFileProcessing = true)]
public class FullDescriptionExtractor : IExtractor
{
     private Encoding _encoding;
     private byte[] _row_delim;
     private char _col_delim;

    public FullDescriptionExtractor(Encoding encoding, string row_delim = "\r\n", char col_delim = '\t')
    {
         this._encoding = ((encoding == null) ? Encoding.UTF8 : encoding);
         this._row_delim = this._encoding.GetBytes(row_delim);
         this._col_delim = col_delim;

    }

    public override IEnumerable<IRow> Extract(IUnstructuredReader input, IUpdatableRow output)
    {
         string line;
         //Read the input line by line
         foreach (Stream current in input.Split(_encoding.GetBytes("\r\n")))
         {
        using (System.IO.StreamReader streamReader = new StreamReader(current, this._encoding))
         {
             line = streamReader.ReadToEnd().Trim();
             //Split the input by the column delimiter
             string[] parts = line.Split(this._col_delim);
             int count = 0; // start with first column
             foreach (string part in parts)
             {
    if (count == 0)
             {  // for column “guid”, re-generated guid
                 Guid new_guid = Guid.NewGuid();
                 output.Set<Guid>(count, new_guid);
             }
             else if (count == 2)
             {
                 // for column “user”, convert to UPPER case
                 output.Set<string>(count, part.ToUpper());

             }
             else
             {
                 // keep the rest of the columns as-is
                 output.Set<string>(count, part);
             }
             count += 1;
             }

         }
         yield return output.AsReadOnly();
         }
         yield break;
     }
}
```

Dans ce scénario d’utilisation, l’extracteur régénère le GUID pour la colonne « guid », et convertit les valeurs de la colonne « utilisateur » en majuscules. Des extracteurs personnalisés peuvent produire des résultats plus complexes en analysant et manipulant des données d’entrée.

Le script U-SQL de base utilisant un extracteur personnalisé est le suivant :

```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
        guid Guid,
        dt String,
        user String,
        des String
    FROM @input_file
        USING new USQL_Programmability.FullDescriptionExtractor(Encoding.UTF8);

OUTPUT @rs0 TO @output_file USING Outputters.Text();
```


## <a name="next-steps"></a>Étapes suivantes
* [Guide de programmabilité U-SQL - vue d’ensemble](data-lake-analytics-u-sql-programmability-guide.md)
* [Guide de programmabilité U-SQL - UDT et UDAGG](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)