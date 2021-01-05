---
title: Utiliser le SDK .NET HBase - Azure HDInsight
description: Utilisez le SDK .NET HBase pour créer et supprimer des tables et pour lire et écrire des données.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 12/02/2019
ms.openlocfilehash: 9b5693ddef5e512b0a95c87a700fd12acd4b5fae
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97654640"
---
# <a name="use-the-net-sdk-for-apache-hbase"></a>Utiliser le SDK .NET pour Apache HBase

[Apache HBase](apache-hbase-overview.md) fournit deux principales options pour utiliser vos données : [les requêtes Apache Hive et des appels à l’API RESTful de HBase](apache-hbase-tutorial-get-started-linux.md). Vous pouvez travailler directement avec l’API REST à l’aide de la commande `curl` ou d’un utilitaire similaire.

Pour les applications C# et .NET, la [bibliothèque de client Microsoft HBase REST pour .NET](https://www.nuget.org/packages/Microsoft.HBase.Client/) fournit une bibliothèque de client sur l’API REST HBase.

## <a name="install-the-sdk"></a>Installer le Kit de développement logiciel (SDK)

Le SDK HBase .NET est fourni sous la forme d’un package NuGet qui peut être installé à partir de la **Console du gestionnaire de package NuGet** de Visual Studio avec la commande suivante :

```console
Install-Package Microsoft.HBase.Client
```

## <a name="instantiate-a-new-hbaseclient-object"></a>Instancier un nouvel objet HBaseClient

Pour utiliser le SDK, instanciez un nouvel objet `HBaseClient`, en passant `ClusterCredentials` contenant le `Uri` dans votre cluster, ainsi que le nom d’utilisateur et le mot de passe Hadoop.

```csharp
var credentials = new ClusterCredentials(new Uri("https://CLUSTERNAME.azurehdinsight.net"), "USERNAME", "PASSWORD");
client = new HBaseClient(credentials);
```

Remplacez CLUSTERNAME par le nom de votre cluster HDInsight HBase, puis remplacez USERNAME et PASSWORD par les informations d’identification Apache Hadoop spécifiées lors de la création du cluster. Le nom d’utilisateur Hadoop par défaut est **admin**.

## <a name="create-a-new-table"></a>Créer une table

HBase stocke des données dans des tables. Une table se compose de *Rowkey*, la clé primaire, et d’un ou de plusieurs groupes de colonnes appelées *familles de colonnes*. Les données de chaque table sont distribuées horizontalement par une plage Rowkey dans des *régions*. Chaque région dispose d’une clé de début et de fin. Une table peut comporter une ou plusieurs régions. À mesure que les données de la table augmentent, HBase fractionne les grandes régions en régions plus petites. Les régions sont stockées dans *des serveurs de région*, où un même serveur de région peut stocker plusieurs régions.

Les données sont stockées physiquement dans *HFiles*. Un seul HFile contient des données pour une table, une région et une famille de colonnes. Les lignes du HFile sont stockées et triées sur Rowkey. Chaque HFile a un index *d’arbre B+* pour la récupération rapide des lignes.

Pour créer une table, spécifiez un `TableSchema` et des colonnes. Le code suivant vérifie si la table « RestSDKTable » existe déjà. Dans le cas contraire, la table est créée.

```csharp
if (!client.ListTablesAsync().Result.name.Contains("RestSDKTable"))
{
    // Create the table
    var newTableSchema = new TableSchema {name = "RestSDKTable" };
    newTableSchema.columns.Add(new ColumnSchema() {name = "t1"});
    newTableSchema.columns.Add(new ColumnSchema() {name = "t2"});

    await client.CreateTableAsync(newTableSchema);
}
```

Cette nouvelle table a deux familles de colonnes : t1 et t2. Étant donné que les familles de colonnes sont stockées séparément dans des HFiles différents, il est judicieux d’avoir une famille de colonnes distincte pour les données fréquemment interrogées. Dans l’exemple [Insérer des données](#insert-data) suivant, les colonnes sont ajoutées à la famille de colonnes t1.

## <a name="delete-a-table"></a>Suppression d’une table

Pour supprimer une table :

```csharp
await client.DeleteTableAsync("RestSDKTable");
```

## <a name="insert-data"></a>Insertion des données

Pour insérer des données, vous spécifiez une clé de ligne unique en tant qu’identificateur de ligne. Toutes les données sont stockées dans un tableau `byte[]`. Le code suivant définit et ajoute les colonnes `title`, `director` et `release_date` à la famille de colonnes t1, car il s’agit des colonnes les plus souvent utilisées. Les colonnes `description` et `tagline` sont ajoutées à la famille de colonnes t2. Vous pouvez partitionner vos données dans des familles de colonnes en fonction des besoins.

```csharp
var key = "fifth_element";
var row = new CellSet.Row { key = Encoding.UTF8.GetBytes(key) };
var value = new Cell
{
    column = Encoding.UTF8.GetBytes("t1:title"),
    data = Encoding.UTF8.GetBytes("The Fifth Element")
};
row.values.Add(value);
value = new Cell
{
    column = Encoding.UTF8.GetBytes("t1:director"),
    data = Encoding.UTF8.GetBytes("Luc Besson")
};
row.values.Add(value);
value = new Cell
{
    column = Encoding.UTF8.GetBytes("t1:release_date"),
    data = Encoding.UTF8.GetBytes("1997")
};
row.values.Add(value);
value = new Cell
{
    column = Encoding.UTF8.GetBytes("t2:description"),
    data = Encoding.UTF8.GetBytes("In the colorful future, a cab driver unwittingly becomes the central figure in the search for a legendary cosmic weapon to keep Evil and Mr Zorg at bay.")
};
row.values.Add(value);
value = new Cell
{
    column = Encoding.UTF8.GetBytes("t2:tagline"),
    data = Encoding.UTF8.GetBytes("The Fifth is life")
};
row.values.Add(value);

var set = new CellSet();
set.rows.Add(row);

await client.StoreCellsAsync("RestSDKTable", set);
```

HBase implémente [Cloud BigTable](https://cloud.google.com/bigtable/) afin que le format de données ressemble à l’image qui suit :

![Exemple de sortie de données Apache HBase](./media/apache-hbase-rest-sdk/hdinsight-table-roles.png)

## <a name="select-data"></a>Sélectionner les données

Pour lire des données à partir d’une table HBase, passez le nom de table et la clé de ligne à la méthode `GetCellsAsync` pour retourner le `CellSet`.

```csharp
var key = "fifth_element";

var cells = await client.GetCellsAsync("RestSDKTable", key);
// Get the first value from the row.
Console.WriteLine(Encoding.UTF8.GetString(cells.rows[0].values[0].data));
// Get the value for t1:title
Console.WriteLine(Encoding.UTF8.GetString(cells.rows[0].values
    .Find(c => Encoding.UTF8.GetString(c.column) == "t1:title").data));
// With the previous insert, it should yield: "The Fifth Element"
```

Dans ce cas, le code retourne uniquement la première ligne correspondante, car il ne doit y avoir qu’une seule ligne pour une clé unique. La valeur retournée est convertie au format `string` à partir du tableau `byte[]`. Vous pouvez également convertir la valeur en d’autres types, comme un entier pour la date de sortie du projet :

```csharp
var releaseDateField = cells.rows[0].values
    .Find(c => Encoding.UTF8.GetString(c.column) == "t1:release_date");
int releaseDate = 0;

if (releaseDateField != null)
{
    releaseDate = Convert.ToInt32(Encoding.UTF8.GetString(releaseDateField.data));
}
Console.WriteLine(releaseDate);
// Should return 1997
```

## <a name="scan-over-rows"></a>Analyser les lignes

HBase utilise `scan` pour récupérer une ou plusieurs lignes. Cet exemple demande plusieurs lignes par lots de 10 et récupère les données dont les valeurs de clés sont comprises entre 25 et 35. Après avoir récupéré toutes les lignes, supprimez l’analyseur pour nettoyer les ressources.

```csharp
var tableName = "mytablename";

// Assume the table has integer keys and we want data between keys 25 and 35
var scanSettings = new Scanner()
{
    batch = 10,
    startRow = BitConverter.GetBytes(25),
    endRow = BitConverter.GetBytes(35)
};
RequestOptions scanOptions = RequestOptions.GetDefaultOptions();
scanOptions.AlternativeEndpoint = "hbaserest0/";
ScannerInformation scannerInfo = null;
try
{
    scannerInfo = await client.CreateScannerAsync(tableName, scanSettings, scanOptions);
    CellSet next = null;
    while ((next = client.ScannerGetNextAsync(scannerInfo, scanOptions).Result) != null)
    {
        foreach (var row in next.rows)
        {
            // ... read the rows
        }
    }
}
finally
{
    if (scannerInfo != null)
    {
        await client.DeleteScannerAsync(tableName, scannerInfo, scanOptions);
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Prise en main d’un exemple Apache HBase dans HDInsight](apache-hbase-tutorial-get-started-linux.md)
* Créer une application de bout en bout avec l’[analyse des sentiments Twitter en temps réel avec Apache HBase](./apache-hbase-tutorial-get-started-linux.md)