---
title: Guide de programmabilité du générateur de sortie U-SQL pour Azure Data Lake
description: En savoir plus sur le guide de programmabilité U-SQL UDO - générateur de sortie défini par l’utilisateur.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: 56b104b5cc8f8923445455c71fe2418e39539b8e
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96512431"
---
# <a name="use-user-defined-outputter"></a>Utiliser un générateur de sortie défini par l’utilisateur

## <a name="u-sql-udo-user-defined-outputter"></a>U-SQL UDO : générateur de sortie défini par l’utilisateur
Le générateur de sortie défini par l’utilisateur est un autre UDO U-SQL qui vous permet d’étendre les fonctionnalités U-SQL intégrées. Comme pour l’extracteur, il existe plusieurs générateurs de sortie intégrés.

* *Outputters.Text()*  : écrit des données dans des fichiers texte délimités d’encodages différents.
* *Outputters.csv ()* : écrit des données dans des fichiers de valeurs séparées par des virgules (CSV) d’encodages différents.
* *Outputters.Tsv()*  : écrit des données dans des fichiers de valeur séparées par des tabulations (TSV) d’encodages différents.

Un générateur de sortie personnalisé vous permet d’écrire des données dans un format défini personnalisé. Cela peut être utile pour les tâches suivantes :

* Écrire des données dans des fichiers semi-structurés ou non structurés.
* Écrire des données dans des codages non pris en charge.
* Modifier des données de sortie ou ajouter des attributs personnalisés.

## <a name="how-to-define-and-use-user-defined-outputter"></a>Définition et utilisation d’un générateur de sortie défini par l’utilisateur
Pour définir un générateur de sortie défini par l’utilisateur, nous devons créer l’interface `IOutputter`.

Voici l’implémentation de la classe `IOutputter` de base :

```csharp
public abstract class IOutputter : IUserDefinedOperator
{
    protected IOutputter();

    public virtual void Close();
    public abstract void Output(IRow input, IUnstructuredWriter output);
}
```

Tous les paramètres d’entrée du générateur de sortie, tels que les séparateurs de colonnes/lignes, le codage, et ainsi de suite, doivent être définis dans le constructeur de la classe. L’interface `IOutputter` doit également contenir une définition pour le remplacement de `void Output`. L’attribut `[SqlUserDefinedOutputter(AtomicFileProcessing = true)` peut éventuellement être défini pour le traitement de fichier atomique. Pour plus d’informations, consultez les détails suivants :

```csharp
[SqlUserDefinedOutputter(AtomicFileProcessing = true)]
public class MyOutputter : IOutputter
{

    public MyOutputter(myparam1, myparam2)
    {
      …
    }

    public override void Close()
    {
      …
    }

    public override void Output(IRow row, IUnstructuredWriter output)
    {
      …
    }
}
```

* `Output` est appelé pour chaque ligne d’entrée. Il retourne l’ensemble de lignes `IUnstructuredWriter output`.
* La classe de constructeur est utilisée pour transmettre des paramètres au générateur de sortie défini par l’utilisateur.
* `Close` est utilisé pour un remplacement éventuel pour libérer d’un état coûteux ou déterminer quand la dernière ligne a été écrite.

L’attribut **SqlUserDefinedOutputter** indique que le type doit être inscrit en tant que générateur de sortie défini par l’utilisateur. Cette classe ne peut pas être héritée.

SqlUserDefinedOutputter est un attribut facultatif pour une définition d’un générateur de sortie défini par l’utilisateur. Il est utilisé pour définir la propriété de AtomicFileProcessing.

* bool     AtomicFileProcessing   

* **true** = indique que ce générateur de sortie requiert des fichiers de sortie atomique (JSON, XML, ...)
* **false** = indique que ce générateur de sortie peut traiter des fichiers fractionnés/distribués (CSV, SEQ, ...)

Les principaux objets de programmabilité sont **ligne** et **sortie**. L’objet **row** est utilisé pour énumérer les données de sortie en tant qu’interface `IRow`. **Output** est utilisé pour définir les données de sortie dans le fichier cible.

Les données de sortie sont accessibles via l’interface `IRow`. Les données de sortie sont transmises une ligne à la fois.

Les valeurs individuelles sont énumérées en appelant la méthode Get de l’interface IRow :

```csharp
row.Get<string>("column_name")
```

Les noms des colonnes peuvent être déterminés en appelant `row.Schema` :

```csharp
ISchema schema = row.Schema;
var col = schema[i];
string val = row.Get<string>(col.Name)
```

Cette approche permet de créer un générateur de sortie flexible pour tout schéma de métadonnées.

Les données de sortie sont écrites dans le fichier en utilisant `System.IO.StreamWriter`. Le paramètre stream est défini sur `output.BaseStream` dans `IUnstructuredWriter output`.

Notez qu’il est important de vider le tampon de données du fichier après chaque itération de la ligne. En outre, l’objet `StreamWriter` doit être utilisé avec l’attribut Disposable activé (par défaut) et avec le mot clé **using** :

```csharp
using (StreamWriter streamWriter = new StreamWriter(output.BaseStream, this._encoding))
{
…
}
```

Autrement, appelez explicitement la méthode Flush() après chaque itération. Cela est illustré dans l’exemple suivant.

### <a name="set-headers-and-footers-for-user-defined-outputter"></a>Définir des en-têtes et des pieds de page pour un générateur de sortie défini par l’utilisateur
Pour définir un en-tête, utilisez un flux d’exécution d’itération unique.

```csharp
public override void Output(IRow row, IUnstructuredWriter output)
{
 …
if (isHeaderRow)
{
    …                
}

 …
if (isHeaderRow)
{
    isHeaderRow = false;
}
 …
}
}
```

Le code du premier bloc `if (isHeaderRow)` n’est exécuté qu’une seule fois.

Pour le pied de page, utilisez la référence à l’instance de l’objet `System.IO.Stream` (`output.BaseStream`). Écrivez le pied de page dans la méthode Close() de l’interface `IOutputter`.  (Pour plus d’informations, consultez les exemples suivants.)

Voici un exemple d’un générateur de sortie défini par l’utilisateur :

```csharp
[SqlUserDefinedOutputter(AtomicFileProcessing = true)]
public class HTMLOutputter : IOutputter
{
    // Local variables initialization
    private string row_delimiter;
    private char col_delimiter;
    private bool isHeaderRow;
    private Encoding encoding;
    private bool IsTableHeader = true;
    private Stream g_writer;

    // Parameters definition            
    public HTMLOutputter(bool isHeader = false, Encoding encoding = null)
    {
    this.isHeaderRow = isHeader;
    this.encoding = ((encoding == null) ? Encoding.UTF8 : encoding);
    }

    // The Close method is used to write the footer to the file. It's executed only once, after all rows
    public override void Close()
    {
    //Reference to IO.Stream object - g_writer
    StreamWriter streamWriter = new StreamWriter(g_writer, this.encoding);
    streamWriter.Write("</table>");
    streamWriter.Flush();
    streamWriter.Close();
    }

    public override void Output(IRow row, IUnstructuredWriter output)
    {
    System.IO.StreamWriter streamWriter = new StreamWriter(output.BaseStream, this.encoding);

    // Metadata schema initialization to enumerate column names
    ISchema schema = row.Schema;

    // This is a data-independent header--HTML table definition
    if (IsTableHeader)
    {
        streamWriter.Write("<table border=1>");
        IsTableHeader = false;
    }

    // HTML table attributes
    string header_wrapper_on = "<th>";
    string header_wrapper_off = "</th>";
    string data_wrapper_on = "<td>";
    string data_wrapper_off = "</td>";

    // Header row output--runs only once
    if (isHeaderRow)
    {
        streamWriter.Write("<tr>");
        for (int i = 0; i < schema.Count(); i++)
        {
        var col = schema[i];
        streamWriter.Write(header_wrapper_on + col.Name + header_wrapper_off);
        }
        streamWriter.Write("</tr>");
    }

    // Data row output
    streamWriter.Write("<tr>");                
    for (int i = 0; i < schema.Count(); i++)
    {
        var col = schema[i];
        string val = "";
        try
        {
        // Data type enumeration--required to match the distinct list of types from OUTPUT statement
        switch (col.Type.Name.ToString().ToLower())
        {
            case "string": val = row.Get<string>(col.Name).ToString(); break;
            case "guid": val = row.Get<Guid>(col.Name).ToString(); break;
            default: break;
        }
        }
        // Handling NULL values--keeping them empty
        catch (System.NullReferenceException)
        {
        }
        streamWriter.Write(data_wrapper_on + val + data_wrapper_off);
    }
    streamWriter.Write("</tr>");

    if (isHeaderRow)
    {
        isHeaderRow = false;
    }
    // Reference to the instance of the IO.Stream object for footer generation
    g_writer = output.BaseStream;
    streamWriter.Flush();
    }
}

// Define the factory classes
public static class Factory
{
    public static HTMLOutputter HTMLOutputter(bool isHeader = false, Encoding encoding = null)
    {
    return new HTMLOutputter(isHeader, encoding);
    }
}
```

Et le script U-SQL de base :

```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.html";

@rs0 =
    EXTRACT
            guid Guid,
        dt String,
            user String,
            des String
         FROM @input_file
         USING new USQL_Programmability.FullDescriptionExtractor(Encoding.UTF8);

OUTPUT @rs0 
    TO @output_file 
    USING new USQL_Programmability.HTMLOutputter(isHeader: true);
```

Il s’agit d’un générateur de sortie HTML qui crée un fichier HTML avec les données de la table.

### <a name="call-outputter-from-u-sql-base-script"></a>Appeler un générateur de sortie à partir du script U-SQL de base
Pour appeler un générateur de sortie personnalisé à partir du script U-SQL de base, la nouvelle instance de l’objet générateur de sortie doit être créée.

```sql
OUTPUT @rs0 TO @output_file USING new USQL_Programmability.HTMLOutputter(isHeader: true);
```

Pour éviter de créer une instance de l’objet dans le script de base, nous pouvons créer un wrapper de fonction, comme indiqué dans l’exemple précédent :

```csharp
        // Define the factory classes
        public static class Factory
        {
            public static HTMLOutputter HTMLOutputter(bool isHeader = false, Encoding encoding = null)
            {
                return new HTMLOutputter(isHeader, encoding);
            }
        }
```

Dans ce cas, l’appel d’origine ressemble à ceci :

```usql
OUTPUT @rs0 
TO @output_file 
USING USQL_Programmability.Factory.HTMLOutputter(isHeader: true);
```

## <a name="next-steps"></a>Étapes suivantes
* [Guide de programmabilité U-SQL - vue d’ensemble](data-lake-analytics-u-sql-programmability-guide.md)
* [Guide de programmabilité U-SQL - UDT et UDAGG](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)