---
title: SDK REST du serveur de requêtes Phoenix - Azure HDInsight
description: Installer et utiliser le SDK REST pour le serveur de requêtes Phoenix dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 01/01/2020
ms.openlocfilehash: 051d7b37f5f78ce28134fa7c4ee188f3dde81812
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89504840"
---
# <a name="apache-phoenix-query-server-rest-sdk"></a>SDK REST Apache Phoenix Query Server

[Apache Phoenix](https://phoenix.apache.org/) est une couche de base de données relationnelle massivement parallèle open source qui vient s’ajouter à [Apache HBase](apache-hbase-overview.md). Phoenix vous permet d’utiliser des requêtes de type SQL avec HBase via des outils SSH tels que [SQLUltraLite](apache-hbase-query-with-phoenix.md). Phoenix fournit également un serveur HTTP appelé un serveur de requêtes Phoenix (PQS), un client léger qui prend en charge deux mécanismes de transport pour la communication client : JSON et les mémoires tampons de protocole. Les mémoires tampons de protocole constituent le mécanisme par défaut et offrent une communication plus efficace que JSON.

Cet article décrit comment utiliser le Kit de développement logiciel (SDK) REST PQS pour créer des tables, des lignes d’upsert individuellement et en bloc, puis sélectionner des données à l’aide d’instructions SQL. Les exemples utilisent le [pilote Microsoft .NET pour le serveur de requêtes Apache Phoenix](https://www.nuget.org/packages/Microsoft.Phoenix.Client). Ce SDK repose sur des API [Avatica d’Apache Calcite](https://calcite.apache.org/avatica/) qui utilisent exclusivement des mémoires tampons de protocole pour le format de sérialisation.

Pour plus d’informations, consultez [Référence des mémoires tampons de protocole Apache Calcite Avatica](https://calcite.apache.org/avatica/docs/protobuf_reference.html).

## <a name="install-the-sdk"></a>Installer le Kit de développement logiciel (SDK)

Le pilote Microsoft .NET pour le serveur de requêtes Apache Phoenix est fourni sous la forme d’un package NuGet qui peut être installé à partir de la **Console du gestionnaire de package NuGet** de Visual Studio avec la commande suivante :

```console
Install-Package Microsoft.Phoenix.Client
```

## <a name="instantiate-new-phoenixclient-object"></a>Instancier un nouvel objet PhoenixClient

Pour utiliser la bibliothèque, instanciez un nouvel objet `PhoenixClient`, en passant `ClusterCredentials` qui contient le `Uri` à votre cluster, ainsi que le nom d’utilisateur et le mot de passe Apache Hadoop du cluster.

```csharp
var credentials = new ClusterCredentials(new Uri("https://CLUSTERNAME.azurehdinsight.net/"), "USERNAME", "PASSWORD");
client = new PhoenixClient(credentials);
```

Remplacez CLUSTERNAME par le nom de votre cluster HDInsight HBase, ainsi que USERNAME et PASSWORD par les informations d’identification Hadoop spécifiées lors de la création du cluster. Le nom d’utilisateur Hadoop par défaut est **admin**.

## <a name="generate-unique-connection-identifier"></a>Générer un identificateur de connexion unique

Pour envoyer une ou plusieurs requêtes à PQS, vous devez inclure un identificateur de connexion unique afin d’associer la ou les requêtes à la connexion.

```csharp
string connId = Guid.NewGuid().ToString();
```

Dans chaque exemple, un appel de la méthode `OpenConnectionRequestAsync` est tout d’abord effectué, en passant l’identificateur de connexion unique. Ensuite, définissez `ConnectionProperties` et `RequestOptions`, en passant ces objets et l’identificateur de connexion généré à la méthode `ConnectionSyncRequestAsync`. L’objet `ConnectionSyncRequest` du PQS permet de s’assurer que le client et le serveur ont une vue cohérente des propriétés de base de données.

## <a name="connectionsyncrequest-and-its-connectionproperties"></a>ConnectionSyncRequest et ses ConnectionProperties

Pour appeler `ConnectionSyncRequestAsync`, passez dans un objet `ConnectionProperties`.

```csharp
ConnectionProperties connProperties = new ConnectionProperties
{
    HasAutoCommit = true,
    AutoCommit = true,
    HasReadOnly = true,
    ReadOnly = false,
    TransactionIsolation = 0,
    Catalog = "",
    Schema = "",
    IsDirty = true
};
await client.ConnectionSyncRequestAsync(connId, connProperties, options);
```

Voici certaines propriétés intéressantes :

| Propriété | Description |
| -- | -- |
| AutoCommit | Booléen indiquant si `autoCommit` est activé pour les transactions Phoenix. |
| Lecture seule | Booléen indiquant si la connexion est en lecture seule. |
| TransactionIsolation | Entier indiquant le niveau d’isolation des transactions selon la spécification JDBC ; consultez le tableau suivant.|
| Catalogue | Nom du catalogue à utiliser lors de la récupération des propriétés de connexion. |
| schéma | Nom du schéma à utiliser lors de la récupération des propriétés de connexion. |
| IsDirty | Booléen indiquant si les propriétés ont été modifiées. |

Voici les valeurs `TransactionIsolation` :

| Valeur d’isolation | Description |
| -- | -- |
| 0 | Les transactions ne sont pas prises en charge. |
| 1 | Des lectures erronées, des lectures non reproductibles et des lectures fantômes peuvent se produire. |
| 2 | Les lectures erronées sont évitées, mais des lectures non reproductibles et des lectures fantômes peuvent se produire. |
| 4 | Les lectures erronées et les lectures non reproductibles sont évitées, mais des lectures fantômes peuvent se produire. |
| 8 | Les lectures erronées, les lectures non reproductibles et les lectures fantômes sont toutes évitées. |

## <a name="create-a-new-table"></a>Créer une table

HBase, comme n’importe quel autre SGBDR, stocke les données dans des tables. Phoenix utilise des requêtes SQL standard pour créer de nouvelles tables lors de la définition de la clé primaire et des types de colonnes.

Cet exemple et tous les exemples suivants utilisent l’objet `PhoenixClient` instancié tel que défini dans [Instancier un nouvel objet PhoenixClient](#instantiate-new-phoenixclient-object).

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();

// You can set certain request options, such as timeout in milliseconds:
options.TimeoutMillis = 300000;

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
CreateStatementResponse createStatementResponse = null;
OpenConnectionResponse openConnResponse = null;

try
{
    // Opening connection
    var info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);

    // Create the statement
    createStatementResponse = client.CreateStatementRequestAsync(connId, options).Result;
    
    // Create the table if it does not exist
    string sql = "CREATE TABLE IF NOT EXISTS Customers (Id varchar(20) PRIMARY KEY, FirstName varchar(50), " +
        "LastName varchar(100), StateProvince char(2), Email varchar(255), Phone varchar(15))";
    await client.PrepareAndExecuteRequestAsync(connId, sql, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);

    Console.WriteLine($"Table \"Customers\" created.");
}
catch (Exception e)
{
    Console.WriteLine(e);
    throw;
}
finally
{
    if (createStatementResponse != null)
    {
        client.CloseStatementRequestAsync(connId, createStatementResponse.StatementId, options).Wait();
        createStatementResponse = null;
    }

    if (openConnResponse != null)
    {
        client.CloseConnectionRequestAsync(connId, options).Wait();
        openConnResponse = null;
    }
}
```

L’exemple précédent crée une nouvelle table nommée `Customers` à l’aide de l’option `IF NOT EXISTS`. L’appel `CreateStatementRequestAsync` crée une nouvelle instruction sur le serveur Avatica (PQS). Le bloc `finally` ferme le `CreateStatementResponse` retourné et les objets `OpenConnectionResponse`.

## <a name="insert-data-individually"></a>Insérer des données individuellement

Cet exemple montre une insertion de données individuelles, en faisant référence à une collection `List<string>` d’abréviations d’états et de territoires américains :

```csharp
var states = new List<string> { "AL", "AK", "AS", "AZ", "AR", "CA", "CO", "CT", "DE", "DC", "FM", "FL", "GA", "GU", "HI", "ID", "IL", "IN", "IA", "KS", "KY", "LA", "ME", "MH", "MD", "MA", "MI", "MN", "MS", "MO", "MT", "NE", "NV", "NH", "NJ", "NM", "NY", "NC", "ND", "MP", "OH", "OK", "OR", "PW", "PA", "PR", "RI", "SC", "SD", "TN", "TX", "UT", "VT", "VI", "VA", "WA", "WV", "WI", "WY" };
```

La valeur de colonne `StateProvince` de la table sera utilisée dans une opération de sélection ultérieure.

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();
options.TimeoutMillis = 300000;

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
OpenConnectionResponse openConnResponse = null;
StatementHandle statementHandle = null;
try
{
    // Opening connection
    pbc::MapField<string, string> info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);

    string sql = "UPSERT INTO Customers VALUES (?,?,?,?,?,?)";
    PrepareResponse prepareResponse = await client.PrepareRequestAsync(connId, sql, 100, options);
    statementHandle = prepareResponse.Statement;
    
    var r = new Random();

    // Insert 300 rows
    for (int i = 0; i < 300; i++)
    {
        var list = new pbc.RepeatedField<TypedValue>();
        var id = new TypedValue
        {
            StringValue = "id" + i,
            Type = Rep.String
        };
        var firstName = new TypedValue
        {
            StringValue = "first" + i,
            Type = Rep.String
        };
        var lastName = new TypedValue
        {
            StringValue = "last" + i,
            Type = Rep.String
        };
        var state = new TypedValue
        {
            StringValue = states.ElementAt(r.Next(0, 49)),
            Type = Rep.String
        };
        var email = new TypedValue
        {
            StringValue = $"email{1}@junkemail.com",
            Type = Rep.String
        };
        var phone = new TypedValue
        {
            StringValue = $"555-229-341{i.ToString().Substring(0,1)}",
            Type = Rep.String
        };
        list.Add(id);
        list.Add(firstName);
        list.Add(lastName);
        list.Add(state);
        list.Add(email);
        list.Add(phone);

        Console.WriteLine("Inserting customer " + i);

        await client.ExecuteRequestAsync(statementHandle, list, long.MaxValue, true, options);
    }

    await client.CommitRequestAsync(connId, options);

    Console.WriteLine("Upserted customer data");

}
catch (Exception ex)
{

}
finally
{
    if (statementHandle != null)
    {
        await client.CloseStatementRequestAsync(connId, statementHandle.Id, options);
        statementHandle = null;
    }
    if (openConnResponse != null)
    {
        await client.CloseConnectionRequestAsync(connId, options);
        openConnResponse = null;
    }
}
```

La structure d’exécution d’une instruction d’insertion est similaire à la création d’une nouvelle table. À la fin du bloc `try`, la transaction est validée explicitement. Cet exemple répète une transaction d’insertion 300 fois. L’exemple suivant montre un processus d’insertion par lot plus efficace.

## <a name="batch-insert-data"></a>Insertion par lot des données

Le code suivant est presque identique au code d’insertion de données individuellement. Cet exemple utilise l’objet `UpdateBatch` dans un appel à `ExecuteBatchRequestAsync`, plutôt que d’appeler à plusieurs reprises `ExecuteRequestAsync` avec une instruction préparée.

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();
options.TimeoutMillis = 300000;

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
OpenConnectionResponse openConnResponse = null;
CreateStatementResponse createStatementResponse = null;
try
{
    // Opening connection
    pbc::MapField<string, string> info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);

    // Creating statement
    createStatementResponse = await client.CreateStatementRequestAsync(connId, options);

    string sql = "UPSERT INTO Customers VALUES (?,?,?,?,?,?)";
    PrepareResponse prepareResponse = client.PrepareRequestAsync(connId, sql, long.MaxValue, options).Result;
    var statementHandle = prepareResponse.Statement;
    var updates = new pbc.RepeatedField<UpdateBatch>();

    var r = new Random();

    // Insert 300 rows
    for (int i = 300; i < 600; i++)
    {
        var list = new pbc.RepeatedField<TypedValue>();
        var id = new TypedValue
        {
            StringValue = "id" + i,
            Type = Rep.String
        };
        var firstName = new TypedValue
        {
            StringValue = "first" + i,
            Type = Rep.String
        };
        var lastName = new TypedValue
        {
            StringValue = "last" + i,
            Type = Rep.String
        };
        var state = new TypedValue
        {
            StringValue = states.ElementAt(r.Next(0, 49)),
            Type = Rep.String
        };
        var email = new TypedValue
        {
            StringValue = $"email{1}@junkemail.com",
            Type = Rep.String
        };
        var phone = new TypedValue
        {
            StringValue = $"555-229-341{i.ToString().Substring(0, 1)}",
            Type = Rep.String
        };
        list.Add(id);
        list.Add(firstName);
        list.Add(lastName);
        list.Add(state);
        list.Add(email);
        list.Add(phone);

        var batch = new UpdateBatch
        {
            ParameterValues = list
        };
        updates.Add(batch);

        Console.WriteLine($"Added customer {i} to batch");
    }

    var executeBatchResponse = await client.ExecuteBatchRequestAsync(connId, statementHandle.Id, updates, options);

    Console.WriteLine("Batch upserted customer data");

}
catch (Exception ex)
{

}
finally
{
    if (openConnResponse != null)
    {
        await client.CloseConnectionRequestAsync(connId, options);
        openConnResponse = null;
    }
}
```

Dans un environnement de test, l’insertion individuellement de 300 nouveaux enregistrements a nécessité près de 2 minutes. En revanche, l’insertion des 300 enregistrements par lot n’a pris que 6 secondes.

## <a name="select-data"></a>Sélectionner les données

Cet exemple montre comment réutiliser une connexion pour exécuter plusieurs requêtes :

1. Sélectionnez tous les enregistrements, puis extrayez les enregistrements restants lorsque le maximum par défaut de 100 a été retourné.
2. Utilisez une instruction de sélection du nombre total de lignes pour récupérer le résultat scalaire unique.
3. Exécutez une instruction de sélection qui retourne le nombre total de clients par état ou territoire.

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
OpenConnectionResponse openConnResponse = null;
StatementHandle statementHandle = null;

try
{
    // Opening connection
    pbc::MapField<string, string> info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);
    var createStatementResponse = await client.CreateStatementRequestAsync(connId, options);

    string sql = "SELECT * FROM Customers";
    ExecuteResponse executeResponse = await client.PrepareAndExecuteRequestAsync(connId, sql, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);

    pbc::RepeatedField<Row> rows = executeResponse.Results[0].FirstFrame.Rows;
    // Loop through all of the returned rows and display the first two columns
    for (int i = 0; i < rows.Count; i++)
    {
        Row row = rows[i];
        Console.WriteLine(row.Value[0].ScalarValue.StringValue + " " + row.Value[1].ScalarValue.StringValue);
    }
    
    // 100 is hard-coded on the server side as the default firstframe size
    // FetchRequestAsync is called to get any remaining rows
    Console.WriteLine("");
    Console.WriteLine($"Number of rows: {rows.Count}");

    // Fetch remaining rows, offset is not used, simply set to 0
    // When FetchResponse.Frame.Done is true, all rows were fetched
    FetchResponse fetchResponse = await client.FetchRequestAsync(connId, createStatementResponse.StatementId, 0, int.MaxValue, options);
    Console.WriteLine($"Frame row count: {fetchResponse.Frame.Rows.Count}");
    Console.WriteLine($"Fetch response is done: {fetchResponse.Frame.Done}");
    Console.WriteLine("");

    // Running query 2
    string sql2 = "select count(*) from Customers";
    ExecuteResponse countResponse = await client.PrepareAndExecuteRequestAsync(connId, sql2, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);
    long count = countResponse.Results[0].FirstFrame.Rows[0].Value[0].ScalarValue.NumberValue;

    Console.WriteLine($"Total customer records: {count}");
    Console.WriteLine("");

    // Running query 3
    string sql3 = "select StateProvince, count(*) as Number from Customers group by StateProvince order by Number desc";
    ExecuteResponse groupByResponse = await client.PrepareAndExecuteRequestAsync(connId, sql3, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);

    pbc::RepeatedField<Row> stateRows = groupByResponse.Results[0].FirstFrame.Rows;
    for (int i = 0; i < stateRows.Count; i++)
    {
        Row row = stateRows[i];
        Console.WriteLine(row.Value[0].ScalarValue.StringValue + ": " + row.Value[1].ScalarValue.NumberValue);
    }
}
catch (Exception ex)
{

}
finally
{
    if (statementHandle != null)
    {
        await client.CloseStatementRequestAsync(connId, statementHandle.Id, options);
        statementHandle = null;
    }
    if (openConnResponse != null)
    {
        await client.CloseConnectionRequestAsync(connId, options);
        openConnResponse = null;
    }
}
```

La sortie des instructions `select` doit être le résultat suivant :

```output
id0 first0
id1 first1
id10 first10
id100 first100
id101 first101
id102 first102
. . .
id185 first185
id186 first186
id187 first187
id188 first188

Number of rows: 100
Frame row count: 500
Fetch response is done: True

Total customer records: 600

NJ: 21
CA: 19
GU: 17
NC: 16
IN: 16
MA: 16
AZ: 16
ME: 16
IL: 15
OR: 15
. . . 
MO: 10
HI: 10
GA: 10
DC: 9
NM: 9
MD: 9
MP: 9
SC: 7
AR: 7
MH: 6
FM: 5
```

## <a name="next-steps"></a>Étapes suivantes

* [Apache Phoenix dans HDInsight](../hdinsight-phoenix-in-hdinsight.md)
* [Utilisation du SDK REST Apache HBase](apache-hbase-rest-sdk.md)
