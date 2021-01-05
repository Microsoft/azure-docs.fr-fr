---
title: Filtrer des données à l’aide de l’accélération des requêtes d’Azure Data Lake Storage | Microsoft Docs
description: Utilisez l’accélération des requêtes pour récupérer un sous-ensemble de données à partir de votre compte de stockage.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 09/09/2020
ms.author: normesta
ms.reviewer: jamsbak
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: b02f5a7c390c5594a7c5692798a0691c8d9a42d0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95912771"
---
# <a name="filter-data-by-using-azure-data-lake-storage-query-acceleration"></a>Filtrer des données à l’aide de l’accélération des requêtes d’Azure Data Lake Storage

Cet article explique comment utiliser l’accélération des requêtes pour récupérer un sous-ensemble de données à partir de votre compte de stockage. 

L’accélération des requêtes est une nouvelle capacité d’Azure Data Lake Storage qui permet aux applications et aux infrastructures d’analytique d’optimiser considérablement le traitement des données en extrayant uniquement les données requises pour effectuer une opération donnée. Pour en savoir plus, consultez [Accélération des requêtes Azure Data Lake Storage](data-lake-storage-query-acceleration.md).

## <a name="prerequisites"></a>Prérequis

- Pour accéder à Stockage Azure, vous avez besoin d’un abonnement Azure. Si vous n’avez pas d’abonnement, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

- Un compte de stockage **universel v2**. Voir [Créer un compte de stockage](../common/storage-account-create.md).

- Choisissez un onglet pour afficher la configuration requise spécifique au Kit de développement logiciel (SDK).

  ### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

  Non applicable

  ### <a name="net"></a>[.NET](#tab/dotnet)

  [Kit de développement logiciel (SDK) .NET](https://dotnet.microsoft.com/download) 

  ### <a name="java"></a>[Java](#tab/java)

  - [Kit de développement Java (JDK)](/java/azure/jdk/?view=azure-java-stable&preserve-view=true), version 8 ou ultérieure

  - [Apache Maven](https://maven.apache.org/download.cgi) 

    > [!NOTE] 
    > Cet article suppose que vous avez créé un projet Java à l’aide d’Apache Maven. Pour obtenir un exemple de création de projet à l’aide d’Apache Maven, consultez [Configuration](storage-quickstart-blobs-java.md#setting-up).
  
  ### <a name="python"></a>[Python](#tab/python)

  [Python](https://www.python.org/downloads/) 3.8 ou version ultérieure.

  ### <a name="nodejs"></a>[Node.JS](#tab/nodejs)

  Aucune configuration supplémentaire n’est requise pour utiliser le Kit de développement logiciel (SDK) Node.js.

---

## <a name="enable-query-acceleration"></a>Activer l’accélération des requêtes

Pour utiliser l’accélération des requêtes, vous devez inscrire la fonctionnalité d’accélération des requêtes sur votre abonnement. Une fois que vous avez vérifié que la fonctionnalité est inscrite, vous devez inscrire le fournisseur de ressources Stockage Azure. 

### <a name="step-1-register-the-query-acceleration-feature"></a>Étape 1 : Inscrire la fonctionnalité d’accélération des requêtes

Pour utiliser l’accélération des requêtes, vous devez d’abord inscrire la fonctionnalité d’accélération des requêtes sur votre abonnement. 

#### <a name="powershell"></a>[PowerShell](#tab/powershell)

1. Ouvrez une fenêtre Commande Windows PowerShell.

1. Connectez-vous à votre abonnement Azure avec la commande `Connect-AzAccount` et suivez les instructions à l’écran.

   ```powershell
   Connect-AzAccount
   ```

2. Si votre identité est associée à plusieurs abonnements, définissez votre abonnement actif.

   ```powershell
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

   Remplacez la valeur d’espace réservé `<subscription-id>` par l’ID de votre abonnement.

3. Inscrivez la fonctionnalité d’accélération des requêtes à l’aide de la commande [Register-AzProviderFeature](/powershell/module/az.resources/register-azproviderfeature).

   ```powershell
   Register-AzProviderFeature -ProviderNamespace Microsoft.Storage -FeatureName BlobQuery
   ```

#### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Ouvrez [Azure Cloud Shell](../../cloud-shell/overview.md) ou, si vous avez [installé](/cli/azure/install-azure-cli) Azure CLI localement, ouvrez une application console de commandes telle que Windows PowerShell.

2. Si votre identité est associée à plusieurs abonnements, définissez l’abonnement actif comme abonnement du compte de stockage.

   ```azurecli-interactive
   az account set --subscription <subscription-id>
   ```

   Remplacez la valeur d’espace réservé `<subscription-id>` par l’ID de votre abonnement.

3. Inscrivez la fonctionnalité d’accélération des requêtes à l’aide de la commande [az feature register](/cli/azure/feature#az-feature-register).

   ```azurecli
   az feature register --namespace Microsoft.Storage --name BlobQuery
   ```

---

### <a name="step-2-verify-that-the-feature-is-registered"></a>Étape 2 : Vérifier que l’appareil est enregistré

#### <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour vérifier que l’inscription est terminée, utilisez la commande [Get-AzProviderFeature](/powershell/module/az.resources/get-azproviderfeature).

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage -FeatureName BlobQuery
```

#### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour vérifier que l’inscription est terminée, utilisez la commande [az feature](/cli/azure/feature#az-feature-show).

```azurecli
az feature show --namespace Microsoft.Storage --name BlobQuery
```

---

### <a name="step-3-register-the-azure-storage-resource-provider"></a>Étape 3 : Réinscrire le fournisseur de ressources Stockage Azure

Une fois votre inscription approuvée, vous devez réinscrire le fournisseur de ressources Stockage Azure. 

#### <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour réinscrire le fournisseur de ressources, utilisez la commande [Register-AzResourceProvider](/powershell/module/az.resources/register-azresourceprovider).

```powershell
Register-AzResourceProvider -ProviderNamespace 'Microsoft.Storage'
```

#### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour réinscrire le fournisseur de ressources, utilisez la commande [az provider register](/cli/azure/provider#az-provider-register).

```azurecli
az provider register --namespace 'Microsoft.Storage'
```

---

## <a name="set-up-your-environment"></a>Configurer votre environnement

### <a name="step-1-install-packages"></a>Étape 1 : Installer des packages 

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Installez la version 4.6.0 ou ultérieure du module Az.

```powershell
Install-Module -Name Az -Repository PSGallery -Force
```

Pour mettre à jour à partir d’une version antérieure d’Az, exécutez la commande suivante :

```powershell
Update-Module -Name Az
```

#### <a name="net"></a>[.NET](#tab/dotnet)

1. Ouvrez une invite de commandes et changez de répertoire (`cd`) dans votre dossier de projet. Par exemple :

   ```console
   cd myProject
   ```

2. Installez la version`12.5.0-preview.6` du package de la bibliothèque de client Stockage Blob Azure pour .NET en utilisant la commande `dotnet add package`. 

   ```console
   dotnet add package Azure.Storage.Blobs -v 12.6.0
   ```

3. Les exemples qui apparaissent dans cet article analysent un fichier CSV à l’aide de la bibliothèque [CsvHelper](https://www.nuget.org/packages/CsvHelper/). Pour utiliser cette bibliothèque, utilisez la commande suivante.

   ```console
   dotnet add package CsvHelper
   ```

#### <a name="java"></a>[Java](#tab/java)

1. Ouvrez le fichier *pom.xml* de votre projet dans un éditeur de texte. Ajoutez les éléments de dépendance suivants au groupe de dépendances. 

   ```xml
   <!-- Request static dependencies from Maven -->
   <dependency>
       <groupId>com.azure</groupId>
       <artifactId>azure-core</artifactId>
       <version>1.6.0</version>
   </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-csv</artifactId>
        <version>1.8</version>
    </dependency>    
    <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-storage-blob</artifactId>
      <version>12.8.0-beta.1</version>
    </dependency>
   ```

#### <a name="python"></a>[Python](#tab/python)

Installez la bibliothèque cliente Azure Data Lake Storage pour Python à l’aide de [pip](https://pypi.org/project/pip/).

```
pip install azure-storage-blob==12.4.0
```

#### <a name="nodejs"></a>[Node.JS](#tab/nodejs)

Installez la bibliothèque de client Data Lake pour JavaScript en ouvrant une fenêtre de terminal, puis en entrant la commande suivante.

```javascript
    npm install @azure/storage-blob
    npm install @fast-csv/parse
```

---

### <a name="step-2-add-statements"></a>Étape 2 : Ajouter des instructions

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Non applicable

#### <a name="net"></a>[.NET](#tab/dotnet)

Ajoutez les instructions `using` suivantes au début de votre fichier de code.

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
using Azure.Storage.Blobs.Specialized;
```

L’accélération des requêtes récupère les données au format CSV et JSON. Par conséquent, veillez à ajouter des instructions using pour les bibliothèques d’analyse CSV ou JSON que vous choisissez d’utiliser. Les exemples qui apparaissent dans cet article analysent un fichier CSV à l’aide de la bibliothèque [CsvHelper](https://www.nuget.org/packages/CsvHelper/) qui est disponible sur le site de NuGet. Par conséquent, nous ajouterons ces instructions `using` au début du fichier de code.

```csharp
using CsvHelper;
using CsvHelper.Configuration;
```

Pour compiler des exemples présentés dans cet article, vous devrez également ajouter ces instructions `using`.

```csharp
using System.Threading.Tasks;
using System.IO;
using System.Globalization;
```

#### <a name="java"></a>[Java](#tab/java)

Ajoutez les instructions `import` suivantes au début de votre fichier de code.

```java
import com.azure.storage.blob.*;
import com.azure.storage.blob.options.*;
import com.azure.storage.blob.models.*;
import com.azure.storage.common.*;
import java.io.*;
import java.util.function.Consumer;
import org.apache.commons.csv.*;
```

#### <a name="python"></a>[Python](#tab/python)

Ajoutez les instructions d’importation suivantes au début de votre fichier de code.

```python
import sys, csv
from azure.storage.blob import BlobServiceClient, ContainerClient, BlobClient, DelimitedTextDialect, BlobQueryError
```

### <a name="nodejs"></a>[Node.JS](#tab/nodejs)

Incluez le module `storage-blob` en plaçant cette instruction en haut de votre fichier de code. 

```javascript
const { BlobServiceClient } = require("@azure/storage-blob");
```

L’accélération des requêtes récupère les données au format CSV et JSON. Par conséquent, veillez à ajouter des instructions pour tous les modules d’analyse CSV ou JSON que vous choisissez d’utiliser. Les exemples qui apparaissent dans cet article analysent un fichier CSV à l’aide du module [fast-csv](https://www.npmjs.com/package/fast-csv). Par conséquent, nous ajoutons cette instruction au début du fichier de code.

```javascript
const csv = require('@fast-csv/parse');
```

---

## <a name="retrieve-data-by-using-a-filter"></a>Récupérer des données à l’aide d’un filtre

Vous pouvez utiliser SQL pour spécifier les prédicats de filtrage de lignes et les projections de colonne dans une demande d’accélération des requêtes. Le code suivant interroge un fichier CSV dans le stockage et retourne toutes les lignes de données où la troisième colonne correspond à la valeur `Hemingway, Ernest`. 

- Dans la requête SQL, le mot clé `BlobStorage` est utilisé pour indiquer le fichier qui est interrogé.

- Les références de colonnes sont spécifiées par `_N` où la première colonne est `_1`. Si le fichier source contient une ligne d’en-tête, vous pouvez faire référence aux colonnes par le nom spécifié dans la ligne d’en-tête. 

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
Function Get-QueryCsv($ctx, $container, $blob, $query, $hasheaders) {
    $tempfile = New-TemporaryFile
    $informat = New-AzStorageBlobQueryConfig -AsCsv -HasHeader:$hasheaders
    Get-AzStorageBlobQueryResult -Context $ctx -Container $container -Blob $blob -InputTextConfiguration $informat -OutputTextConfiguration (New-AzStorageBlobQueryConfig -AsCsv -HasHeader) -ResultFile $tempfile.FullName -QueryString $query -Force
    Get-Content $tempfile.FullName
}

$container = "data"
$blob = "csv/csv-general/seattle-library.csv"
Get-QueryCsv $ctx $container $blob "SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'" $false

```

### <a name="net"></a>[.NET](#tab/dotnet)

La méthode asynchrone `BlobQuickQueryClient.QueryAsync` envoie la requête à l’API d’accélération des requêtes, puis transmet en continu les résultats à l’application sous la forme d’un objet [Stream](/dotnet/api/system.io.stream).

```cs
static async Task QueryHemingway(BlockBlobClient blob)
{
    string query = @"SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'";
    await DumpQueryCsv(blob, query, false);
}

private static async Task DumpQueryCsv(BlockBlobClient blob, string query, bool headers)
{
    try
    {
        var options = new BlobQueryOptions() {
            InputTextConfiguration = new BlobQueryCsvTextOptions() { HasHeaders = headers },
            OutputTextConfiguration = new BlobQueryCsvTextOptions() { HasHeaders = true },
            ProgressHandler = new Progress<long>((finishedBytes) => Console.Error.WriteLine($"Data read: {finishedBytes}"))
        };
        options.ErrorHandler += (BlobQueryError err) => {
            Console.ForegroundColor = ConsoleColor.Red;
            Console.Error.WriteLine($"Error: {err.Position}:{err.Name}:{err.Description}");
            Console.ResetColor();
        };
        // BlobDownloadInfo exposes a Stream that will make results available when received rather than blocking for the entire response.
        using (var reader = new StreamReader((await blob.QueryAsync(
                query,
                options)).Value.Content))
        {
            using (var parser = new CsvReader(reader, new CsvConfiguration(CultureInfo.CurrentCulture) { HasHeaderRecord = true }))
            {
                while (await parser.ReadAsync())
                {
                    Console.Out.WriteLine(String.Join(" ", parser.Context.Record));
                }
            }
        }
    }
    catch (Exception ex)
    {
        Console.Error.WriteLine("Exception: " + ex.ToString());
    }
}

```

### <a name="java"></a>[Java](#tab/java)

La méthode `BlobQuickQueryClient.openInputStream()` envoie la requête à l’API d’accélération des requêtes, puis transmet en continu les résultats à l’application sous la forme d’un objet `InputStream` qui peut être lu comme tout autre objet InputStream.

```java
static void QueryHemingway(BlobClient blobClient) {
    String expression = "SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'";
    DumpQueryCsv(blobClient, expression, true);
}

static void DumpQueryCsv(BlobClient blobClient, String query, Boolean headers) {
    try {
        BlobQuerySerialization input = new BlobQueryDelimitedSerialization()
            .setRecordSeparator('\n')
            .setColumnSeparator(',')
            .setHeadersPresent(headers)
            .setFieldQuote('\0')
            .setEscapeChar('\\');
        BlobQuerySerialization output = new BlobQueryDelimitedSerialization()
            .setRecordSeparator('\n')
            .setColumnSeparator(',')
            .setHeadersPresent(true)
            .setFieldQuote('\0')
            .setEscapeChar('\n');
        Consumer<BlobQueryError> errorConsumer = System.out::println;
        Consumer<BlobQueryProgress> progressConsumer = progress -> System.out.println("total bytes read: " + progress.getBytesScanned());
        BlobQueryOptions queryOptions = new BlobQueryOptions(query)
            .setInputSerialization(input)
            .setOutputSerialization(output)
            .setErrorConsumer(errorConsumer)
            .setProgressConsumer(progressConsumer);            

        /* Open the query input stream. */
        InputStream stream = blobClient.openQueryInputStream(queryOptions).getValue();
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(stream))) {
            /* Read from stream like you normally would. */
            for (CSVRecord record : CSVParser.parse(reader, CSVFormat.EXCEL.withHeader())) {
                System.out.println(record.toString());
            }
        }
    } catch (Exception e) {
        System.err.println("Exception: " + e.toString());
        e.printStackTrace(System.err);
    }
}
```

### <a name="python"></a>[Python](#tab/python)

```python
def query_hemingway(blob: BlobClient):
    query = "SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'"
    dump_query_csv(blob, query, False)

def dump_query_csv(blob: BlobClient, query: str, headers: bool):
    qa_reader = blob.query_blob(query, blob_format=DelimitedTextDialect(has_header=headers), on_error=report_error, encoding='utf-8')
    # records() returns a generator that will stream results as received. It will not block pending all results.
    csv_reader = csv.reader(qa_reader.records())
    for row in csv_reader:
        print("*".join(row))
```

### <a name="nodejs"></a>[Node.JS](#tab/nodejs)

Cet exemple envoie la requête à l’API d’accélération des requêtes, puis retourne les résultats.

```javascript
async function queryHemingway(blob)
{
    const query = "SELECT * FROM BlobStorage WHERE _3 = 'Hemingway, Ernest, 1899-1961'";
    await dumpQueryCsv(blob, query, false);
}

async function dumpQueryCsv(blob, query, headers)
{
    var response = await blob.query(query, {
        inputTextConfiguration: {
            kind: "csv",
            recordSeparator: '\n',
            hasHeaders: headers
        },
        outputTextConfiguration: {
            kind: "csv",
            recordSeparator: '\n',
            hasHeaders: true
        },
        onProgress: (progress) => console.log(`Data read: ${progress.loadedBytes}`),
        onError: (err) => console.error(`Error: ${err.position}:${err.name}:${err.description}`)});
    return new Promise(
        function (resolve, reject) {
            csv.parseStream(response.readableStreamBody)
                .on('data', row => console.log(row))
                .on('error', error => {
                    console.error(error);
                    reject(error);
                })
                .on('end', rowCount => resolve());
    });
}
```

---

## <a name="retrieve-specific-columns"></a>Récupérer des colonnes spécifiques

Vous pouvez étendre vos résultats à un sous-ensemble de colonnes. De cette façon, vous récupérez uniquement les colonnes nécessaires pour effectuer un calcul donné. Cela permet d’améliorer les performances de l’application et de réduire les coûts, car moins de données sont transférées via le réseau. 

Ce code récupère uniquement la colonne `BibNum` pour tous les livres du jeu de données. Il utilise également les informations de la ligne d’en-tête dans le fichier source pour référencer des colonnes dans la requête.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
Function Get-QueryCsv($ctx, $container, $blob, $query, $hasheaders) {
    $tempfile = New-TemporaryFile
    $informat = New-AzStorageBlobQueryConfig -AsCsv -HasHeader:$hasheaders
    Get-AzStorageBlobQueryResult -Context $ctx -Container $container -Blob $blob -InputTextConfiguration $informat -OutputTextConfiguration (New-AzStorageBlobQueryConfig -AsCsv -HasHeader) -ResultFile $tempfile.FullName -QueryString $query -Force
    Get-Content $tempfile.FullName
}

$container = "data"
$blob = "csv/csv-general/seattle-library-with-headers.csv"
Get-QueryCsv $ctx $container $blob "SELECT BibNum FROM BlobStorage" $true

```

### <a name="net"></a>[.NET](#tab/dotnet)

```cs
static async Task QueryBibNum(BlockBlobClient blob)
{
    string query = @"SELECT BibNum FROM BlobStorage";
    await DumpQueryCsv(blob, query, true);
}
```

### <a name="java"></a>[Java](#tab/java)

```java
static void QueryBibNum(BlobClient blobClient)
{
    String expression = "SELECT BibNum FROM BlobStorage";
    DumpQueryCsv(blobClient, expression, true);
}
```

### <a name="python"></a>[Python](#tab/python)

```python
def query_bibnum(blob: BlobClient):
    query = "SELECT BibNum FROM BlobStorage"
    dump_query_csv(blob, query, True)
```

### <a name="nodejs"></a>[Node.JS](#tab/nodejs)

```javascript
async function queryBibNum(blob)
{
    const query = "SELECT BibNum FROM BlobStorage";
    await dumpQueryCsv(blob, query, true);
}
```

---

Le code suivant combine le filtrage de lignes et les projections de colonnes dans la même requête. 

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
Get-QueryCsv $ctx $container $blob $query $true

Function Get-QueryCsv($ctx, $container, $blob, $query, $hasheaders) {
    $tempfile = New-TemporaryFile
    $informat = New-AzStorageBlobQueryConfig -AsCsv -HasHeader:$hasheaders
    Get-AzStorageBlobQueryResult -Context $ctx -Container $container -Blob $blob -InputTextConfiguration $informat -OutputTextConfiguration (New-AzStorageBlobQueryConfig -AsCsv -HasHeader) -ResultFile $tempfile.FullName -QueryString $query -Force
    Get-Content $tempfile.FullName
}

$container = "data"
$query = "SELECT BibNum, Title, Author, ISBN, Publisher, ItemType 
            FROM BlobStorage 
            WHERE ItemType IN 
                ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')"

```

### <a name="net"></a>[.NET](#tab/dotnet)

```cs
static async Task QueryDvds(BlockBlobClient blob)
{
    string query = @"SELECT BibNum, Title, Author, ISBN, Publisher, ItemType 
        FROM BlobStorage 
        WHERE ItemType IN 
            ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')";
    await DumpQueryCsv(blob, query, true);
}
```

### <a name="java"></a>[Java](#tab/java)

```java
static void QueryDvds(BlobClient blobClient)
{
    String expression = "SELECT BibNum, Title, Author, ISBN, Publisher, ItemType " +
                        "FROM BlobStorage " +
                        "WHERE ItemType IN " +
                        "   ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')";
    DumpQueryCsv(blobClient, expression, true);
}
```

### <a name="python"></a>[Python](#tab/python)

```python
def query_dvds(blob: BlobClient):
    query = "SELECT BibNum, Title, Author, ISBN, Publisher, ItemType "\
        "FROM BlobStorage "\
        "WHERE ItemType IN "\
        "   ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')"
    dump_query_csv(blob, query, True)
```

### <a name="nodejs"></a>[Node.JS](#tab/nodejs)

```javascript
async function queryDvds(blob)
{
    const query = "SELECT BibNum, Title, Author, ISBN, Publisher, ItemType " +
                  "FROM BlobStorage " +
                  "WHERE ItemType IN " + 
                  " ('acdvd', 'cadvd', 'cadvdnf', 'calndvd', 'ccdvd', 'ccdvdnf', 'jcdvd', 'nadvd', 'nadvdnf', 'nalndvd', 'ncdvd', 'ncdvdnf')";
    await dumpQueryCsv(blob, query, true);
}
```

---

## <a name="next-steps"></a>Étapes suivantes

- [Accélération des requêtes Azure Data Lake Storage](data-lake-storage-query-acceleration.md)
- [Informations de référence sur l’accélération des requêtes en langage SQL](query-acceleration-sql-reference.md)