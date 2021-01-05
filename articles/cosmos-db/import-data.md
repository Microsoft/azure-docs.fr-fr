---
title: 'Tutoriel : Outil de migration de base de données pour Azure Cosmos DB'
description: 'Tutoriel : Découvrez comment utiliser l’outil de migration de données open source Azure Cosmos DB pour importer des données dans Azure Cosmos DB depuis différentes sources, y compris des fichiers MongoDB, SQL Server, Stockage Table, Amazon DynamoDB, CSV et JSON. Conversion CSV vers JSON.'
author: deborahc
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: tutorial
ms.date: 10/23/2020
ms.author: dech
ms.openlocfilehash: 1cee4d2ad1bc7f362a045a5991624ec43521b8d2
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96341646"
---
# <a name="tutorial-use-data-migration-tool-to-migrate-your-data-to-azure-cosmos-db"></a>Tutoriel : Utilisez l’outil de migration de données pour migrer vos données vers Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Ce tutoriel explique comment utiliser l’outil de migration de données Azure Cosmos DB pour importer des données dans des conteneurs Azure Cosmos DB à partir de différentes sources. Vous pouvez importer à partir de fichiers JSON, de fichiers CSV, de SQL, de MongoDB, d’un stockage Table Azure, d’Amazon DynamoDB et même de collections d’API SQL Azure Cosmos DB. Vous migrez ces données vers des collections et des tables en vue de les utiliser avec Azure Cosmos DB. L’outil de migration de données peut également être utilisé pour migrer des données à partir d’une collection à partition unique vers une collection à plusieurs partitions pour l’API SQL.

> [!NOTE]
> L’outil de migration de données Azure Cosmos DB est un outil open source conçu pour les petites migrations. Pour les migrations plus volumineuses, consultez notre [guide de l’ingestion de données](cosmosdb-migrationchoices.md).

* **[API SQL](./introduction.md)** - Vous pouvez utiliser toutes les options de source fournies dans l’outil de migration de données pour importer des données à petite échelle. [En savoir plus sur les options de migration pour l’importation de données à grande échelle](cosmosdb-migrationchoices.md)
* **[API Table](table-introduction.md)** - Vous pouvez utiliser l’outil de migration de données ou [AzCopy](table-import.md#migrate-data-by-using-azcopy) pour importer les données. Pour plus d’informations, consultez la section [Importer des données à utiliser avec l’API Table Azure Cosmos DB](table-import.md).
* **[API Azure Cosmos DB pour MongoDB](mongodb-introduction.md)** - L’outil de migration de données ne prend pas en charge l’API Azure Cosmos DB pour MongoDB comme source ou cible. Si vous souhaitez migrer les données dans ou en dehors des collections dans Azure Cosmos DB, consultez [Tutoriel : Migrer vos données vers un compte d’API MongoDB Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json) pour obtenir des instructions. Vous pouvez toujours utiliser l’outil de migration de données pour exporter des données de MongoDB vers des collections d’API SQL Azure Cosmos DB en vue d’une utilisation avec l’API SQL.
* **[API Cassandra](graph-introduction.md)** - L’outil de migration de données n’est pas un outil d’importation pris en charge pour les comptes d’API Cassandra. [En savoir plus les options de migration pour l’importation de données dans l’API Cassandra](cosmosdb-migrationchoices.md#azure-cosmos-db-cassandra-api)
* **[API Gremlin](graph-introduction.md)** - Actuellement, l’outil de migration de données n’est pas un outil d’importation pris en charge pour les comptes d’API Gremlin. [En savoir plus sur les options de migration pour l’importation de données dans l’API Gremlin](cosmosdb-migrationchoices.md#other-apis) 

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Installation de l’outil de migration de données
> * Importation de données à partir de différentes sources de données
> * Exportation de données à partir d’Azure Cosmos DB vers JSON

## <a name="prerequisites"></a><a id="Prerequisites"></a>Configuration requise

Avant de suivre les instructions de cet article, veillez à effectuer les étapes suivantes :

* **Installer** [Microsoft .NET Framework 4.51](https://www.microsoft.com/download/developer-tools.aspx) ou version ultérieure.

* **Augmenter le débit :** la durée de la migration des données dépend de la quantité de débit que vous définissez pour une collection spécifique ou un ensemble de collections. Veillez à augmenter le débit pour les migrations de données plus importantes. Une fois que vous avez effectué la migration, diminuez le débit pour réduire les coûts. Pour plus d’informations sur l’augmentation du débit sur le portail Azure, consultez les [niveaux de performances](performance-levels.md) et les [niveaux tarifaires](https://azure.microsoft.com/pricing/details/cosmos-db/) dans Azure Cosmos DB.

* **Créer des ressources Azure Cosmos DB :** avant de commencer à migrer les données, créez au préalable toutes vos collections à partir du portail Azure. Pour migrer vers un compte Azure Cosmos DB qui a un débit de niveau base de données, fournissez une clé de partition au moment de créer les conteneurs Azure Cosmos DB.

> [!IMPORTANT]
> Pour s’assurer que l’outil de migration des données utilise Transport Layer Security (TLS) 1.2 lors de la connexion à vos comptes Azure Cosmos, utilisez .NET Framework version 4.7, ou suivez les instructions données dans [cet article](/dotnet/framework/network-programming/tls).

## <a name="overview"></a><a id="Overviewl"></a>Vue d’ensemble

L’outil de migration de données est une solution open source permettant d’importer des données dans Azure Cosmos DB à partir de différentes sources, notamment :

* Fichiers JSON
* MongoDB
* SQL Server
* Fichiers CSV
* Stockage de tables Azure
* Amazon DynamoDB
* hbase
* Conteneurs Cosmos Azure

L’outil d’importation inclut une interface utilisateur graphique (dtui.exe) et peut aussi être piloté à partir de la ligne de commande (dt.exe). En fait, il existe une option pour générer la commande associée après avoir configuré une importation via l’interface utilisateur. Vous pouvez transformer des données sources tabulaires, comme des fichiers SQL Server ou CSV, pour créer des relations hiérarchiques (sous-documents) pendant l’importation. Poursuivez votre lecture pour en savoir plus sur les options sources, les exemples de commandes pour l’importation depuis chaque source, les options cibles et l’affichage des résultats d’importation.

> [!NOTE]
> Vous devez utiliser l’outil de migration Azure Cosmos DB pour les petites migrations uniquement. Pour les grandes migrations, consultez notre [guide de l’ingestion de données](cosmosdb-migrationchoices.md).

## <a name="installation"></a><a id="Install"></a>Installation

Le code source de l’outil de migration est disponible sur GitHub dans [ce dépôt](https://github.com/azure/azure-documentdb-datamigrationtool). Vous pouvez télécharger et compiler la solution localement ou [télécharger une solution binaire pré-compilée](https://aka.ms/csdmtool), puis exécuter soit :

* **Dtui.exe** : version de l’interface graphique de l’outil
* **Dt.exe** : version en ligne de commande de l’outil

## <a name="select-data-source"></a>Sélectionnez la source de données

Une fois que vous avez installé l’outil, il est temps d’importer vos données. Quel type de données voulez-vous importer ?

* [Fichiers JSON](#JSON)
* [MongoDB](#MongoDB)
* [Fichiers d’exportation MongoDB](#MongoDBExport)
* [SQL Server](#SQL)
* [Fichiers CSV](#CSV)
* [Stockage Table Azure](#AzureTableSource)
* [Amazon DynamoDB](#DynamoDBSource)
* [Objet blob](#BlobImport)
* [Conteneurs Cosmos Azure](#SQLSource)
* [HBase](#HBaseSource)
* [Importation en bloc Azure Cosmos DB](#SQLBulkTarget)
* [Importation d’enregistrement séquentiel Azure Cosmos DB](#SQLSeqTarget)

## <a name="import-json-files"></a><a id="JSON"></a>Importation de fichiers JSON

L’option d’importateur de source de fichiers JSON vous permet d’importer un ou plusieurs fichiers JSON à un seul document ou des fichiers JSON qui contiennent chacun un tableau de documents JSON. Quand vous ajoutez des dossiers qui contiennent des fichiers JSON à importer, vous avez la possibilité d’effectuer une recherche récursive sur les fichiers situés dans des sous-dossiers.

:::image type="content" source="./media/import-data/jsonsource.png" alt-text="Capture d’écran des options de source de fichier JSON - Outils de migration de base de données":::

La chaîne de connexion est au format suivant :

`AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>`

* `<CosmosDB Endpoint>` est l’URI du point de terminaison. Vous pouvez obtenir cette valeur à partir du Portail Azure. Accédez à votre compte Azure Cosmos. Ouvrez le volet **Vue d’ensemble** et copiez la valeur **URI**.
* `<AccountKey>` correspond au « mot de passe » ou à la **clé primaire**. Vous pouvez obtenir cette valeur à partir du Portail Azure. Accédez à votre compte Azure Cosmos. Ouvrez le volet **Chaînes de connexion** ou **Clés** et copiez la valeur de « mot de passe » ou de **clé primaire**.
* `<CosmosDB Database>` est le nom de la base de données CosmosDB.

Exemple : `AccountEndpoint=https://myCosmosDBName.documents.azure.com:443/;AccountKey=wJmFRYna6ttQ79ATmrTMKql8vPri84QBiHTt6oinFkZRvoe7Vv81x9sn6zlVlBY10bEPMgGM982wfYXpWXWB9w==;Database=myDatabaseName`

> [!NOTE]
> Utilisez la commande Verify pour vous assurer que le compte Cosmos DB spécifié dans le champ de la chaîne de connexion est accessible.

Voici quelques exemples de lignes de commande pour importer des fichiers JSON :

```console
#Import a single JSON file
dt.exe /s:JsonFile /s.Files:.\Sessions.json /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Sessions /t.CollectionThroughput:2500

#Import a directory of JSON files
dt.exe /s:JsonFile /s.Files:C:\TESessions\*.json /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Sessions /t.CollectionThroughput:2500

#Import a directory (including sub-directories) of JSON files
dt.exe /s:JsonFile /s.Files:C:\LastFMMusic\**\*.json /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Music /t.CollectionThroughput:2500

#Import a directory (single), directory (recursive), and individual JSON files
dt.exe /s:JsonFile /s.Files:C:\Tweets\*.*;C:\LargeDocs\**\*.*;C:\TESessions\Session48172.json;C:\TESessions\Session48173.json;C:\TESessions\Session48174.json;C:\TESessions\Session48175.json;C:\TESessions\Session48177.json /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:subs /t.CollectionThroughput:2500

#Import a single JSON file and partition the data across 4 collections
dt.exe /s:JsonFile /s.Files:D:\\CompanyData\\Companies.json /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:comp[1-4] /t.PartitionKey:name /t.CollectionThroughput:2500
```

## <a name="import-from-mongodb"></a><a id="MongoDB"></a>Importation à partir de MongoDB

> [!IMPORTANT]
> Si vous importez dans un compte Cosmos configuré avec l’API Azure Cosmos DB pour MongoDB, suivez ces [instructions](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json).

L’option d’importateur de source MongoDB vous permet d’importer à partir d’une collection MongoDB unique, éventuellement de filtrer les documents à l’aide d’une requête et de modifier la structure de document à l’aide d’une projection.  

:::image type="content" source="./media/import-data/mongodbsource.png" alt-text="Capture d’écran des options de source MongoDB":::

La chaîne de connexion est au format MongoDB standard :

`mongodb://<dbuser>:<dbpassword>@<host>:<port>/<database>`

> [!NOTE]
> Utilisez la commande Verify pour vous assurer que l'instance MongoDB spécifiée dans le champ de la chaîne de connexion est accessible.

Saisissez le nom de la collection depuis laquelle les données seront importées. Vous pouvez éventuellement spécifier ou fournir un fichier pour une requête (par exemple, `{pop: {$gt:5000}}`) ou une projection (par exemple, `{loc:0}`) pour à la fois filtrer et mettre en forme les données que vous importez.

Voici quelques exemples de ligne de commande pour l’importation depuis MongoDB :

```console
#Import all documents from a MongoDB collection
dt.exe /s:MongoDB /s.ConnectionString:mongodb://<dbuser>:<dbpassword>@<host>:<port>/<database> /s.Collection:zips /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:BulkZips /t.IdField:_id /t.CollectionThroughput:2500

#Import documents from a MongoDB collection which match the query and exclude the loc field
dt.exe /s:MongoDB /s.ConnectionString:mongodb://<dbuser>:<dbpassword>@<host>:<port>/<database> /s.Collection:zips /s.Query:{pop:{$gt:50000}} /s.Projection:{loc:0} /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:BulkZipsTransform /t.IdField:_id/t.CollectionThroughput:2500
```

## <a name="import-mongodb-export-files"></a><a id="MongoDBExport"></a>Importation de fichiers d'exportation MongoDB

> [!IMPORTANT]
> Si vous importez dans un compte Azure Cosmos DB avec la prise en charge de MongoDB, suivez ces [instructions](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json).

L’option d’importateur source du fichier JSON d’exportation MongoDB vous permet d’importer un ou plusieurs fichiers JSON générés depuis l’utilitaire mongoexport.  

:::image type="content" source="./media/import-data/mongodbexportsource.png" alt-text="Capture d’écran des options de source d’exportation MongoDB":::

Quand vous ajoutez des dossiers qui contiennent des fichiers JSON d’exportation MongoDB à importer, vous avez la possibilité d’effectuer une recherche récursive sur les fichiers situés dans des sous-dossiers.

Voici un exemple de ligne de commande pour importer à partir de fichiers JSON d’exportation MongoDB :

```console
dt.exe /s:MongoDBExport /s.Files:D:\mongoemployees.json /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:employees /t.IdField:_id /t.Dates:Epoch /t.CollectionThroughput:2500
```

## <a name="import-from-sql-server"></a><a id="SQL"></a>Importation depuis SQL Server

L’option d’importateur source SQL vous permet d'importer à partir d'une base de données SQL Server individuelle et de filtrer éventuellement les enregistrements à importer à l'aide d'une requête. De plus, vous pouvez modifier la structure du document en spécifiant un séparateur d'imbrication (plus d’informations dans un instant).  

:::image type="content" source="./media/import-data/sqlexportsource.png" alt-text="Capture d’écran des options de source SQL - Outils de migration de base de données":::

Le format de la chaîne de connexion est le format de chaîne de connexion SQL standard.

> [!NOTE]
> Utilisez la commande Verify pour vous assurer que l'instance SQL Server spécifiée dans le champ de la chaîne de connexion est accessible.

La propriété du séparateur d'imbrication est utilisée pour créer des relations hiérarchiques (sous-documents) lors de l'importation. Examinez la requête SQL suivante :

`select CAST(BusinessEntityID AS varchar) as Id, Name, AddressType as [Address.AddressType], AddressLine1 as [Address.AddressLine1], City as [Address.Location.City], StateProvinceName as [Address.Location.StateProvinceName], PostalCode as [Address.PostalCode], CountryRegionName as [Address.CountryRegionName] from Sales.vStoreWithAddresses WHERE AddressType='Main Office'`

Cette requête retourne les résultats (partiels) suivants :

:::image type="content" source="./media/import-data/sqlqueryresults.png" alt-text="Capture d’écran des résultats d’une requête SQL":::

Notez les alias tels que Address.AddressType et Address.Location.StateProvinceName. En spécifiant un séparateur d’imbrication de « . », l’outil d’importation crée les sous-documents Address et Address.Location lors de l’importation. Voici un exemple de document qui en résulte dans Azure Cosmos DB :

*{ "id": "956", "Name": "Finer Sales and Service", "Address": { "AddressType": "Main Office", "AddressLine1": "#500-75 O’Connor Street", "Location": { "City": "Ottawa", "StateProvinceName": "Ontario" }, "PostalCode": "K4B 1S2", "CountryRegionName": "Canada" } }*

Voici quelques exemples de lignes de commande pour l’importation depuis SQL Server :

```console
#Import records from SQL which match a query
dt.exe /s:SQL /s.ConnectionString:"Data Source=<server>;Initial Catalog=AdventureWorks;User Id=advworks;Password=<password>;" /s.Query:"select CAST(BusinessEntityID AS varchar) as Id, * from Sales.vStoreWithAddresses WHERE AddressType='Main Office'" /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Stores /t.IdField:Id /t.CollectionThroughput:2500

#Import records from sql which match a query and create hierarchical relationships
dt.exe /s:SQL /s.ConnectionString:"Data Source=<server>;Initial Catalog=AdventureWorks;User Id=advworks;Password=<password>;" /s.Query:"select CAST(BusinessEntityID AS varchar) as Id, Name, AddressType as [Address.AddressType], AddressLine1 as [Address.AddressLine1], City as [Address.Location.City], StateProvinceName as [Address.Location.StateProvinceName], PostalCode as [Address.PostalCode], CountryRegionName as [Address.CountryRegionName] from Sales.vStoreWithAddresses WHERE AddressType='Main Office'" /s.NestingSeparator:. /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:StoresSub /t.IdField:Id /t.CollectionThroughput:2500
```

## <a name="import-csv-files-and-convert-csv-to-json"></a><a id="CSV"></a>Importer des fichiers CSV et convertir le format CSV en JSON

L'option d'importateur source du fichier CSV vous permet d'importer un ou plusieurs fichiers CSV. Quand vous ajoutez des dossiers qui contiennent des fichiers CSV à importer, vous avez la possibilité d’effectuer une recherche récursive sur les fichiers situés dans des sous-dossiers.

:::image type="content" source="media/import-data/csvsource.png" alt-text="Capture d’écran des options de source CSV - De CSV en JSON":::

De même que pour la source SQL, la propriété du séparateur d'imbrication peut être utilisée pour créer des relations hiérarchiques (sous-documents) lors de l'importation. Prenez en compte la ligne d'en-tête et les lignes de données du CSV suivant :

:::image type="content" source="./media/import-data/csvsample.png" alt-text="Capture d’écran d’exemples d’enregistrement CSV - De CSV en JSON":::

Notez les alias tels que DomainInfo.Domain_Name et RedirectInfo.Redirecting. En spécifiant un séparateur d’imbrication de « . », l’outil d’importation crée les sous-documents DomainInfo et RedirectInfo lors de l’importation. Voici un exemple de document qui en résulte dans Azure Cosmos DB :

*{ "DomainInfo": { "Domain_Name": "ACUS.GOV", "Domain_Name_Address": "https:\//www.ACUS.GOV" }, "Federal Agency": "Administrative Conference of the United States", "RedirectInfo": { "Redirecting": "0", "Redirect_Destination": "" }, "id": "9cc565c5-ebcd-1c03-ebd3-cc3e2ecd814d" }*

L’outil d’importation essaie de déduire les informations de type pour les valeurs sans guillemets dans les fichiers CSV (les valeurs entre guillemets sont toujours traitées comme des chaînes).  Les types sont identifiés dans l'ordre suivant : nombre, date et heure, valeur booléenne.  

Deux autres points sont à prendre en considération concernant l'importation CSV :

1. Par défaut, les valeurs sans guillemets sont toujours privées de leurs tabulations et espaces, tandis que les valeurs entre guillemets sont conservées telles quelles. Ce comportement peut être remplacé en cochant la case Rogner les valeurs entre guillemets ou à l'aide de l'option de ligne de commande /s.TrimQuoted.
2. Par défaut, une valeur null sans guillemets est considérée comme une valeur null. Ce comportement peut être remplacé (c’est-à-dire, traiter une valeur null sans guillemets comme une chaîne « null ») en cochant la case Traiter les valeurs null sans guillemets comme des chaînes ou à l’aide de l’option de ligne de commande /s.NoUnquotedNulls.

Voici un exemple de ligne de commande pour une importation CSV :

```console
dt.exe /s:CsvFile /s.Files:.\Employees.csv /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Employees /t.IdField:EntityID /t.CollectionThroughput:2500
```

## <a name="import-from-azure-table-storage"></a><a id="AzureTableSource"></a>Importation depuis le stockage de tables Azure

L'option d’importateur source de Stockage Table Azure vous permet d’effectuer l’importation à partir d'une table individuelle de Stockage Table Azure. Vous pouvez aussi filtrer les entités de table à importer.

Vous pouvez sortir des données qui ont été importées de Stockage Table Azure vers des tables et des entités Azure Cosmos DB en vue de les utiliser avec l’API Table. Les données importées peuvent aussi être sorties vers des collections et des documents pour une utilisation avec l’API SQL. Cependant, l’API Table est disponible uniquement en tant que cible dans l’utilitaire de ligne de commande. Vous ne pouvez pas exporter vers l’API Table à partir de l’interface utilisateur de l’outil de migration de données. Pour plus d’informations, consultez la section [Importer des données à utiliser avec l’API Table Azure Cosmos DB](table-import.md).

:::image type="content" source="./media/import-data/azuretablesource.png" alt-text="Capture d’écran des options de source de Stockage Table Azure":::

Le format de la chaîne de connexion de stockage de tables Azure est :

`DefaultEndpointsProtocol=<protocol>;AccountName=<Account Name>;AccountKey=<Account Key>;`

> [!NOTE]
> Utilisez la commande Verify pour vous assurer que l'instance de stockage de tables Azure spécifiée dans le champ de la chaîne de connexion est accessible.

Entrez le nom de la table Azure à partir de laquelle effectuer l’importation. Vous pouvez éventuellement spécifier un [filtre](/visualstudio/azure/vs-azure-tools-table-designer-construct-filter-strings).

L'option d’importateur source de stockage de tables Azure dispose des options supplémentaires suivantes :

1. Inclusion des champs internes
   1. Tous : inclure tous les champs internes (PartitionKey, RowKey et Timestamp)
   2. Aucun : exclure tous les champs internes
   3. RowKey : inclure uniquement le champ RowKey
2. Sélection des colonnes
   1. Les filtres de stockage Table Azure ne prennent pas en charge les projections. Si vous souhaitez importer uniquement des propriétés d'entité de table Azure spécifiques, ajoutez-les à la liste Sélection des colonnes. Toutes les autres propriétés d'entité sont ignorées.

Voici un exemple de ligne de commande pour effectuer l’importation à partir du Stockage Table Azure :

```console
dt.exe /s:AzureTable /s.ConnectionString:"DefaultEndpointsProtocol=https;AccountName=<Account Name>;AccountKey=<Account Key>" /s.Table:metrics /s.InternalFields:All /s.Filter:"PartitionKey eq 'Partition1' and RowKey gt '00001'" /s.Projection:ObjectCount;ObjectSize  /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:metrics /t.CollectionThroughput:2500
```

## <a name="import-from-amazon-dynamodb"></a><a id="DynamoDBSource"></a>Importation à partir d’Amazon DynamoDB

L’option d’importateur de source Amazon DynamoDB vous permet d’importer à partir d’une seule table Amazon DynamoDB. Elle permet de filtrer les entités à importer. Plusieurs modèles sont fournis pour faciliter au maximum la configuration d'une importation.

:::image type="content" source="./media/import-data/dynamodbsource1.png" alt-text="Capture d’écran des options de source Amazon DynamoDB - Outils de migration de base de données.":::

:::image type="content" source="./media/import-data/dynamodbsource2.png" alt-text="Capture d’écran des options de source Amazon DynamoDB avec modèle - Outils de migration de base de données.":::

Le format de la chaîne de connexion Amazon DynamoDB est :

`ServiceURL=<Service Address>;AccessKey=<Access Key>;SecretKey=<Secret Key>;`

> [!NOTE]
> Utilisez la commande Verify pour vous assurer que l'instance Amazon DynamoBD spécifiée dans le champ de la chaîne de connexion est accessible.

Voici un exemple de ligne de commande pour effectuer l’importation à partir d'Amazon DynamoDB :

```console
dt.exe /s:DynamoDB /s.ConnectionString:ServiceURL=https://dynamodb.us-east-1.amazonaws.com;AccessKey=<accessKey>;SecretKey=<secretKey> /s.Request:"{   """TableName""": """ProductCatalog""" }" /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<Azure Cosmos DB Endpoint>;AccountKey=<Azure Cosmos DB Key>;Database=<Azure Cosmos database>;" /t.Collection:catalogCollection /t.CollectionThroughput:2500
```

## <a name="import-from-azure-blob-storage"></a><a id="BlobImport"></a>Importer à partir du Stockage Blob Azure

Les options d’importateur source du fichier JSON, du fichier d'exportation MongoDB et du fichier CSV vous permettent d'importer un ou plusieurs fichiers à partir du stockage d’objets blob Azure. Après avoir spécifié l’URL d’un conteneur d'objets blob et une clé de compte, fournissez une expression régulière pour sélectionner le ou les fichier à importer.

:::image type="content" source="./media/import-data/blobsource.png" alt-text="Capture d’écran des options de source de fichier blob":::

Voici un exemple de ligne de commande pour importer des fichiers JSON à partir du Stockage Blob Azure :

```console
dt.exe /s:JsonFile /s.Files:"blobs://<account key>@account.blob.core.windows.net:443/importcontainer/.*" /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:doctest
```

## <a name="import-from-a-sql-api-collection"></a><a id="SQLSource"></a>Importer à partir d’une collection d’API SQL

L’option de l’importateur source Azure Cosmos DB vous permet d’importer des données à partir d’un ou de plusieurs conteneurs Azure Cosmos DB et de filtrer éventuellement des documents à l’aide d’une requête.  

:::image type="content" source="./media/import-data/documentdbsource.png" alt-text="Capture d’écran des options de source d’Azure Cosmos DB":::

Le format de la chaîne de connexion Azure Cosmos DB est :

`AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;`

Vous pouvez récupérer la chaîne de connexion de compte Azure Cosmos DB à partir de la page Clés du portail Azure, comme décrit dans [Gérer les comptes de base de données dans Azure Cosmos DB](./how-to-manage-database-account.md). Cependant, le nom de la base de données doit être ajouté à la chaîne de connexion dans le format suivant :

`Database=<CosmosDB Database>;`

> [!NOTE]
> Utilisez la commande Verify pour vous assurer que l’instance Azure Cosmos DB spécifiée dans le champ de la chaîne de connexion est accessible.

Pour effectuer l’importation à partir d’un seul conteneur Azure Cosmos DB, entrez le nom de la collection à partir de laquelle importer les données. Pour importer à partir de plusieurs conteneurs Azure Cosmos DB, fournissez une expression régulière correspondant à un ou plusieurs noms de collections (par exemple collection01 | collection02 | collection03). Vous pouvez éventuellement spécifier ou fournir un fichier pour une requête pour à la fois filtrer et mettre en forme les données que vous importez.

> [!NOTE]
> Étant donné que le champ de collection accepte les expressions régulières, si vous importez à partir d’une collection unique dont le nom contient des caractères d’expression régulière, ces caractères doivent être placés en conséquence dans une séquence d’échappement.

L’option d’importateur source Azure Cosmos DB dispose des options avancées suivantes :

1. Inclure des champs internes : spécifie s’il faut inclure ou non dans l’exportation les propriétés système du document Azure Cosmos DB (par exemple, _rid, _ts).
2. Nombre de nouvelles tentatives en cas d’échec : spécifie le nombre de nouvelles tentatives de connexion à Azure Cosmos DB en cas d’échecs temporaires (par exemple, une interruption de connectivité réseau).
3. Intervalle avant nouvelle tentative : spécifie le délai d’attente entre les nouvelles tentatives de connexion à Azure Cosmos DB en cas d’échecs temporaires (par exemple, une interruption de la connectivité réseau).
4. Mode de connexion : cette option indique le mode de connexion à utiliser avec Azure Cosmos DB. Les choix disponibles sont DirectTcp, DirectHttps et la passerelle. Les modes de connexion directs sont plus rapides, tandis que le mode passerelle est mieux adapté au pare-feu car il utilise uniquement le port 443.

:::image type="content" source="./media/import-data/documentdbsourceoptions.png" alt-text="Capture d’écran des options avancées de source d’Azure Cosmos DB":::

> [!TIP]
> L’outil d’importation utilise le mode de connexion DirectTcp par défaut. Si vous rencontrez des problèmes liés au pare-feu, passer au mode de connexion passerelle qui ne nécessite que le port 443.

Voici quelques exemples de lignes de commande pour l’importation à partir d’Azure Cosmos DB :

```console
#Migrate data from one Azure Cosmos container to another Azure Cosmos containers
dt.exe /s:DocumentDB /s.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /s.Collection:TEColl /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:TESessions /t.CollectionThroughput:2500

#Migrate data from more than one Azure Cosmos container to a single Azure Cosmos container
dt.exe /s:DocumentDB /s.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /s.Collection:comp1|comp2|comp3|comp4 /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:singleCollection /t.CollectionThroughput:2500

#Export an Azure Cosmos container to a JSON file
dt.exe /s:DocumentDB /s.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /s.Collection:StoresSub /t:JsonFile /t.File:StoresExport.json /t.Overwrite
```

> [!TIP]
> L’outil d’importation de données Azure Cosmos DB prend également en charge l’importation de données à partir de [l’émulateur Azure Cosmos DB](local-emulator.md). Quand vous importez des données à partir d’un émulateur local, affectez `https://localhost:<port>` comme point de terminaison.

## <a name="import-from-hbase"></a><a id="HBaseSource"></a>Importation à partir de HBase

L’option d’importateur source HBase vous permet d’importer des données à partir d'une table HBase et de filtrer éventuellement les données. Plusieurs modèles sont fournis pour faciliter au maximum la configuration d'une importation.

:::image type="content" source="./media/import-data/hbasesource1.png" alt-text="Capture d’écran des options de source HBase.":::

:::image type="content" source="./media/import-data/hbasesource2.png" alt-text="Capture d’écran des options de source HBase avec le menu contextuel Filtre développé.":::

Le format de la chaîne de connexion HBase Stargate est :

`ServiceURL=<server-address>;Username=<username>;Password=<password>`

> [!NOTE]
> Utilisez la commande Verify pour vous assurer que l'instance HBase spécifiée dans le champ de la chaîne de connexion est accessible.

Voici un exemple de ligne de commande pour l’importation à partir de HBase :

```console
dt.exe /s:HBase /s.ConnectionString:ServiceURL=<server-address>;Username=<username>;Password=<password> /s.Table:Contacts /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:hbaseimport
```

## <a name="import-to-the-sql-api-bulk-import"></a><a id="SQLBulkTarget"></a>Importer vers l’API SQL (importation en bloc)

L’importateur en bloc Azure Cosmos DB vous permet d’importer à partir des options sources disponibles, à l’aide d’une procédure Azure Cosmos DB stockée pour plus d’efficacité. L’outil prend en charge l’importation dans un conteneur Azure Cosmos monopartition. Il prend aussi en charge l’importation partitionnée par laquelle les données sont partitionnées dans plusieurs conteneurs Azure Cosmos DB monopartition. Pour plus d’informations sur le partitionnement de données, consultez [Partitionnement et mise à l’échelle dans Azure Cosmos DB](partitioning-overview.md). L'outil crée, exécute, puis supprime la procédure stockée de la ou les collections cibles.  

:::image type="content" source="./media/import-data/documentdbbulk.png" alt-text="Capture d’écran des options bulk d’Azure Cosmos DB":::

Le format de la chaîne de connexion Azure Cosmos DB est :

`AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;`

La chaîne de connexion de compte Azure Cosmos DB peut être récupérée à partir de la page Clés du portail Azure, comme décrit dans [Guide pratique pour gérer un compte Azure Cosmos DB](./how-to-manage-database-account.md), mais le nom de la base de données doit être ajouté à la chaîne de connexion au format suivant :

`Database=<CosmosDB Database>;`

> [!NOTE]
> Utilisez la commande Verify pour vous assurer que l’instance Azure Cosmos DB spécifiée dans le champ de la chaîne de connexion est accessible.

Pour importer des données dans une seule collection, entrez le nom de la collection cible et cliquez sur le bouton Ajouter. Pour importer dans plusieurs collections, entrez le nom de chaque collection individuellement ou utilisez la syntaxe suivante pour spécifier plusieurs collections : *préfixe_collection*[index de début - index de fin]. Quand vous spécifiez plusieurs collections en utilisant la syntaxe ci-dessus, tenez compte des points suivants :

1. Seuls les modèles de nom de plage de nombres entiers sont pris en charge. Par exemple, la spécification de collection[0-3] crée les collections suivantes : collection0, collection1, collection2, collection3.
2. Vous pouvez utiliser une syntaxe abrégée : collection[3], qui crée le même jeu de collections que celui mentionné à l'étape 1.
3. Plusieurs substitutions peuvent être fournies. Par exemple, collection[0-1] [0-9] génère 20 noms de collection avec des zéros non significatifs (collection01, ..02, ..03).

Une fois que les noms de la collection ont été spécifiés, choisissez le débit souhaité des collections (entre 400 RU et 10 000 RU). Pour de meilleures performances d’importation, choisissez un débit plus élevé. Pour plus d’informations sur les niveaux de performances, consultez les [niveaux de performances d’Azure Cosmos DB](performance-levels.md).

> [!NOTE]
> Le paramètre de débit de performance s’applique uniquement à la création de collections. Si la collection spécifiée existe déjà, son débit n’est pas modifié.

Quand vous importez dans plusieurs collections, l’outil d’importation prend en charge le partitionnement basé sur le hachage. Dans ce scénario, spécifiez la propriété de document que vous souhaitez utiliser comme clé de partition. (Si le champ Clé de partition n’est pas renseigné, les documents sont partitionnés de façon aléatoire parmi les collections cibles.)

Vous pouvez éventuellement spécifier le champ de la source d’importation qui doit être utilisé comme propriété d’ID de document Azure Cosmos DB pendant l’importation. Si les documents ne possèdent pas cette propriété, l’outil d’importation génère un GUID en guise de valeur de la propriété ID.

De nombreuses options avancées sont disponibles lors de l'importation. Tout d'abord, tandis que l'outil inclut une procédure stockée d’importation en bloc par défaut (BulkInsert.js), vous pouvez choisir d’indiquer votre propre procédure stockée d'importation :

 :::image type="content" source="./media/import-data/bulkinsertsp.png" alt-text="Capture d’écran de l’option sproc d’insertion en bloc d’Azure Cosmos DB":::

Par ailleurs, quand vous importez des types de date (par exemple, à partir de SQL Server ou de MongoDB), vous avoir le choix entre trois options d'importation :

 :::image type="content" source="./media/import-data/datetimeoptions.png" alt-text="Capture d’écran des options d’importation de date et d’heure d’Azure Cosmos DB":::

* Chaîne : conserver en tant que valeur de chaîne
* Epoch : conserver en tant que valeur numérique Epoch
* Les deux : conserver les chaînes et les valeurs numériques Epoch Cette option crée un sous-document, par exemple : "date_joined": { "Value": "2013-10-21T21:17:25.2410000Z", "Epoch": 1382390245 }

L’importateur en bloc Azure Cosmos DB dispose des options avancées supplémentaires suivantes :

1. Taille du lot : l’outil adopte par défaut une taille de lot de 50.  Si les documents qui doivent être importés sont volumineux, pensez à réduire la taille du lot. À l’inverse, si les documents qui doivent être importés sont peu volumineux, pensez à augmenter la taille du lot.
2. Taille de script maximale (octets) : l’outil utilise par défaut une taille de script maximale de 512 Ko.
3. Désactivation de la génération automatique d’ID : si tous les documents à importer contiennent un champ ID, le fait de sélectionner cette option peut accroître les performances. Les documents qui n’ont pas de champ ID unique ne sont pas importés.
4. Mise à jour des documents existants : par défaut, l’outil ne remplace pas les documents existants présentant des conflits d’ID. Le fait de sélectionner cette option permet de remplacer les documents existants dont les ID sont identiques. Cette fonctionnalité est utile pour les migrations de données planifiées qui mettent à jour des documents existants.
5. Nombre de nouvelles tentatives en cas d’échec : spécifie la fréquence des nouvelles tentatives de connexion à Azure Cosmos DB en cas d’échec temporaire (par exemple, une interruption de la connectivité réseau).
6. Intervalle avant nouvelle tentative : spécifie le délai d’attente entre les nouvelles tentatives de connexion à Azure Cosmos DB en cas d’échecs temporaires (par exemple, une interruption de la connectivité réseau).
7. Mode de connexion : cette option indique le mode de connexion à utiliser avec Azure Cosmos DB. Les choix disponibles sont DirectTcp, DirectHttps et la passerelle. Les modes de connexion directs sont plus rapides, tandis que le mode passerelle est mieux adapté au pare-feu car il utilise uniquement le port 443.

:::image type="content" source="./media/import-data/docdbbulkoptions.png" alt-text="Capture d’écran des options avancées d’importation en bloc d’Azure Cosmos DB":::

> [!TIP]
> L’outil d’importation utilise le mode de connexion DirectTcp par défaut. Si vous rencontrez des problèmes liés au pare-feu, passer au mode de connexion passerelle qui ne nécessite que le port 443.

## <a name="import-to-the-sql-api-sequential-record-import"></a><a id="SQLSeqTarget"></a>Importer vers l’API SQL (importation d’enregistrement séquentiel)

L’importateur d’enregistrements séquentiel Azure Cosmos DB vous permet d’importer à partir d’une source proposée en option, enregistrement par enregistrement. Vous pouvez choisir cette option si vous importez vers une collection existante ayant atteint son quota de procédures stockées. L’outil prend en charge l’importation dans un seul conteneur (à partition unique ou à plusieurs partitions) Azure Cosmos. Il prend aussi en charge l’importation partitionnée par laquelle les données sont partitionnées dans plusieurs conteneurs Azure Cosmos à partition unique ou à plusieurs partitions. Pour plus d’informations sur le partitionnement de données, consultez [Partitionnement et mise à l’échelle dans Azure Cosmos DB](partitioning-overview.md).

:::image type="content" source="./media/import-data/documentdbsequential.png" alt-text="Capture d’écran des options d’importation d’enregistrement séquentiel d’Azure Cosmos DB":::

Le format de la chaîne de connexion Azure Cosmos DB est :

`AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;`

Vous pouvez récupérer la chaîne de connexion pour le compte Azure Cosmos DB à partir de la page Clés du portail Azure, comme décrit dans [Gérer les comptes de base de données dans Azure Cosmos DB](./how-to-manage-database-account.md). Cependant, le nom de la base de données doit être ajouté à la chaîne de connexion dans le format suivant :

`Database=<Azure Cosmos database>;`

> [!NOTE]
> Utilisez la commande Verify pour vous assurer que l’instance Azure Cosmos DB spécifiée dans le champ de la chaîne de connexion est accessible.

Pour importer dans seule collection, entrez le nom de la collection dans laquelle les données doivent être importées, puis cliquez sur le bouton Ajouter. Pour importer dans plusieurs collections, entrez le nom de chaque collection individuellement. Vous pouvez aussi utiliser la syntaxe suivante pour spécifier plusieurs collections : *préfixe_collection*[index de début - index de fin]. Quand vous spécifiez plusieurs collections en utilisant la syntaxe ci-dessus, tenez compte des points suivants :

1. Seuls les modèles de nom de plage de nombres entiers sont pris en charge. Par exemple, la spécification de collection[0-3] crée les collections suivantes : collection0, collection1, collection2, collection3.
2. Vous pouvez utiliser une syntaxe abrégée : collection[3], qui crée le même jeu de collections que celui mentionné à l'étape 1.
3. Plusieurs substitutions peuvent être fournies. Par exemple, collection[0-1] [0-9] crée 20 noms de collection avec des zéros non significatifs (collection01, ..02, ..03).

Une fois que les noms de la collection ont été spécifiés, choisissez le débit souhaité des collections (entre 400 RU et 250 000 RU). Pour de meilleures performances d’importation, choisissez un débit plus élevé. Pour plus d’informations sur les niveaux de performances, consultez les [niveaux de performances d’Azure Cosmos DB](performance-levels.md). Les importations dans des collections avec un débit > 10 000 RU nécessitent une clé de partition. Si vous choisissez d’avoir plus de 250 000 RU, vous devez envoyer une demande d’augmentation de votre compte dans le portail.

> [!NOTE]
> Le paramètre de débit s’applique uniquement à la création de collections ou de bases de données. Si la collection spécifiée existe déjà, son débit n’est pas modifié.

Quand vous importez dans plusieurs collections, l’outil d’importation prend en charge le partitionnement basé sur le hachage. Dans ce scénario, spécifiez la propriété de document que vous souhaitez utiliser comme clé de partition. (Si le champ Clé de partition n’est pas renseigné, les documents sont partitionnés de façon aléatoire parmi les collections cibles.)

Vous pouvez éventuellement spécifier le champ de la source d’importation qui doit être utilisé comme propriété d’ID de document Azure Cosmos DB pendant l’importation. (Si les documents ne possèdent pas cette propriété, l’outil d’importation génère un GUID en guise de valeur de la propriété ID.)

De nombreuses options avancées sont disponibles lors de l'importation. Tout d’abord, quand vous importez des types de date (par exemple, à partir de SQL Server ou de MongoDB), vous avez le choix entre trois options d'importation :

 :::image type="content" source="./media/import-data/datetimeoptions.png" alt-text="Capture d’écran des options d’importation de date et d’heure d’Azure Cosmos DB":::

* Chaîne : conserver en tant que valeur de chaîne
* Epoch : conserver en tant que valeur numérique Epoch
* Les deux : conserver les chaînes et les valeurs numériques Epoch Cette option crée un sous-document, par exemple : "date_joined": { "Value": "2013-10-21T21:17:25.2410000Z", "Epoch": 1382390245 }

L’importateur d’enregistrement séquentiel Azure Cosmos DB dispose des options avancées supplémentaires suivantes :

1. Nombre de demandes parallèles : l’outil utilise par défaut deux demandes parallèles. Si les documents qui doivent être importés sont peu volumineux, pensez à augmenter le nombre de demandes parallèles. Si ce nombre est trop élevé, l’importation peut rencontrer une limitation.
2. Désactivation de la génération automatique d’ID : si tous les documents à importer contiennent un champ ID, le fait de sélectionner cette option peut accroître les performances. Les documents qui n’ont pas de champ ID unique ne sont pas importés.
3. Mise à jour des documents existants : par défaut, l’outil ne remplace pas les documents existants présentant des conflits d’ID. Le fait de sélectionner cette option permet de remplacer les documents existants dont les ID sont identiques. Cette fonctionnalité est utile pour les migrations de données planifiées qui mettent à jour des documents existants.
4. Nombre de nouvelles tentatives en cas d’échec : spécifie la fréquence des nouvelles tentatives de connexion à Azure Cosmos DB en cas d’échec temporaire (par exemple, une interruption de la connectivité réseau).
5. Intervalle avant nouvelle tentative : spécifie le délai d’attente entre les nouvelles tentatives de connexion à Azure Cosmos DB en cas d’échec temporaire (par exemple, une interruption de la connectivité réseau).
6. Mode de connexion : cette option indique le mode de connexion à utiliser avec Azure Cosmos DB. Les choix disponibles sont DirectTcp, DirectHttps et la passerelle. Les modes de connexion directs sont plus rapides, tandis que le mode passerelle est mieux adapté au pare-feu car il utilise uniquement le port 443.

:::image type="content" source="./media/import-data/documentdbsequentialoptions.png" alt-text="Capture d’écran des options avancées d’importation d’enregistrement séquentiel d’Azure Cosmos DB":::

> [!TIP]
> L’outil d’importation utilise le mode de connexion DirectTcp par défaut. Si vous rencontrez des problèmes liés au pare-feu, passer au mode de connexion passerelle qui ne nécessite que le port 443.

## <a name="specify-an-indexing-policy"></a><a id="IndexingPolicy"></a>Spécifier une stratégie d’indexation

Quand vous autorisez l’outil de migration à créer des collections d’API SQL Azure Cosmos DB pendant l’importation, vous pouvez spécifier la stratégie d’indexation des collections. Dans la section des options d’importation en bloc avancées Azure Cosmos DB et des options d’enregistrement séquentiel Azure Cosmos DB, accédez à la section de la stratégie de l’indexation.

:::image type="content" source="./media/import-data/indexingpolicy1.png" alt-text="Capture d’écran des options avancées de stratégie d’indexation d’Azure Cosmos DB.":::

À l'aide de l’option de stratégie d'indexation avancée, vous pouvez sélectionner un fichier de stratégie d'indexation, entrer manuellement une stratégie d'indexation ou en sélectionner une parmi les différents modèles proposés par défaut (en cliquant avec le bouton droit dans la zone de texte de stratégie d'indexation).

L'outil fournit les modèles de stratégie suivants :

* Par défaut. Cette stratégie est à privilégier quand vous effectuez des requêtes d’égalité sur des chaînes. Elle fonctionne aussi si vous utilisez des requêtes ORDER BY, de plage de données et d’égalité pour les nombres. Cette stratégie dispose d’une surcharge de stockage d'index inférieure à Plage.
* Plage. Cette stratégie est préférable quand vous utilisez des requêtes ORDER BY, de plage de données et d’égalité sur des nombres et des chaînes. Cette stratégie dispose d’une surcharge de stockage d'index supérieure à Par défaut ou Hachage.

:::image type="content" source="./media/import-data/indexingpolicy2.png" alt-text="Capture d’écran des options avancées de stratégie d’indexation d’Azure Cosmos DB spécifiant des informations sur la cible.":::

> [!NOTE]
> Si vous ne spécifiez pas de stratégie d’indexation, la stratégie par défaut est appliquée. Pour plus d’informations sur les stratégies d’indexation, consultez [Stratégies d’indexation d’Azure Cosmos DB](index-policy.md).

## <a name="export-to-json-file"></a>Exportation vers un fichier JSON

L’exportateur JSON Azure Cosmos DB vous permet d’exporter n’importe quelle source parmi celles proposées en option vers un fichier JSON qui contient un tableau de documents JSON. L’outil gère l’exportation automatiquement. Vous pouvez aussi choisir d’afficher la commande de migration résultante et d’exécuter la commande par vous-même. Le fichier JSON résultant peut être stocké localement ou dans le stockage d’objets blob Azure.

:::image type="content" source="./media/import-data/jsontarget.png" alt-text="Capture d’écran des options d’exportation de fichier local JSON d’Azure Cosmos DB":::

:::image type="content" source="./media/import-data/jsontarget2.png" alt-text="Capture d’écran des options d’exportation du stockage Blob Azure JSON d’Azure Cosmos DB":::

Vous pouvez éventuellement choisir d’agrémenter le JSON obtenu. Cette action a pour effet d’accroître la taille du document obtenu tout en rendant le contenu plus lisible.

* Exportation JSON standard

  ```JSON
  [{"id":"Sample","Title":"About Paris","Language":{"Name":"English"},"Author":{"Name":"Don","Location":{"City":"Paris","Country":"France"}},"Content":"Don's document in Azure Cosmos DB is a valid JSON document as defined by the JSON spec.","PageViews":10000,"Topics":[{"Title":"History of Paris"},{"Title":"Places to see in Paris"}]}]
  ```

* Exportation JSON agrémentée

  ```JSON
    [
     {
    "id": "Sample",
    "Title": "About Paris",
    "Language": {
      "Name": "English"
    },
    "Author": {
      "Name": "Don",
      "Location": {
        "City": "Paris",
        "Country": "France"
      }
    },
    "Content": "Don's document in Azure Cosmos DB is a valid JSON document as defined by the JSON spec.",
    "PageViews": 10000,
    "Topics": [
      {
        "Title": "History of Paris"
      },
      {
        "Title": "Places to see in Paris"
      }
    ]
    }]
  ```

Voici un exemple de ligne de commande pour exporter le fichier JSON vers le stockage Blob Azure :

```console
dt.exe /ErrorDetails:All /s:DocumentDB /s.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB database_name>" /s.Collection:<CosmosDB collection_name>
/t:JsonFile /t.File:"blobs://<Storage account key>@<Storage account name>.blob.core.windows.net:443/<Container_name>/<Blob_name>"
/t.Overwrite
```

## <a name="advanced-configuration"></a>Configuration avancée

Dans l'écran Configuration avancée, spécifiez l'emplacement du fichier journal dans lequel écrire toutes les erreurs. Les règles suivantes s'appliquent à cette page :

1. Si un nom de fichier n’est pas fourni, toutes les erreurs sont retournées dans la page Résultats.
2. Si un nom de fichier est fourni sans répertoire, le fichier est créé (ou remplacé) dans le répertoire de l'environnement actuel.
3. Si vous sélectionnez un fichier existant, il est remplacé ; il n’existe pas d’option d’ajout.
4. Choisissez ensuite si vous souhaitez consigner tous les messages d’erreur, uniquement les messages critiques, ou aucun message d'erreur. Enfin, indiquez la fréquence de mise à jour du message de progression du transfert qui s’affiche à l’écran.

   :::image type="content" source="./media/import-data/AdvancedConfiguration.png" alt-text="Capture d’écran de l’écran Configuration avancée":::

## <a name="confirm-import-settings-and-view-command-line"></a>Confirmation des paramètres d'importation et affichage de la ligne de commande

1. Après avoir spécifié les informations de la source et de la cible ainsi que la configuration avancée, vérifiez le récapitulatif de la migration et affichez ou copiez la commande de migration résultante, si vous le souhaitez. (La copie de la commande est utile pour automatiser les opérations d’importation.)

    :::image type="content" source="./media/import-data/summary.png" alt-text="Capture de l’écran récapitulatif.":::

    :::image type="content" source="./media/import-data/summarycommand.png" alt-text="Capture de l’écran récapitulatif avec aperçu de la ligne de commande.":::

2. Une fois que vous êtes satisfait de vos options sources et cibles, cliquez sur **Importer**. Les informations concernant le temps écoulé, la quantité transférée et les échecs (si vous n'avez pas fourni de nom de fichier dans la configuration avancée) sont mises à jour pendant le processus d'importation. Une fois l’importation terminée, vous pouvez exporter les résultats (par exemple, pour gérer les échecs d’importation éventuels).

    :::image type="content" source="./media/import-data/viewresults.png" alt-text="Capture d’écran des options d’exportation JSON d’Azure Cosmos DB.":::

3. Vous pouvez aussi lancer une nouvelle importation en réinitialisant toutes les valeurs ou en conservant les paramètres existants. (Par exemple, vous pouvez choisir de conserver les informations de chaîne de connexion, le choix de la source et de la cible, etc.)

    :::image type="content" source="./media/import-data/newimport.png" alt-text="Capture d’écran des options d’exportation JSON Azure Cosmos DB avec la nouvelle boîte de dialogue de confirmation de l’importation.":::

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez effectué les tâches suivantes :

> [!div class="checklist"]
> * Installation de l’outil de migration de données
> * Importation de données à partir de différentes sources de données
> * Exportation de données à partir d’Azure Cosmos DB vers JSON

Vous pouvez maintenant passer à l’étape suivante du didacticiel et découvrir comment interroger les données à l’aide d’Azure Cosmos DB.

> [!div class="nextstepaction"]
>[Comment interroger les données ?](../cosmos-db/tutorial-query-sql-api.md)
