---
title: Copier des données vers ou à partir d’Azure SQL Database
description: Apprenez à copier des données vers/depuis Azure SQL Database en utilisant Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: 484f735b-8464-40ba-a9fc-820e6553159e
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 83ab9e212e71ad53007a84ad8c10979bfea4516b
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92637392"
---
# <a name="copy-data-to-and-from-azure-sql-database-using-azure-data-factory"></a>Copier des données vers et depuis Azure SQL Database en utilisant Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-azure-sql-connector.md)
> * [Version 2 (version actuelle)](../connector-azure-sql-database.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Connecteur Azure SQL Database dans V2](../connector-azure-sql-database.md).

Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour déplacer des données vers et depuis Azure SQL Database. Il s’appuie sur l’article [Activités de déplacement des données](data-factory-data-movement-activities.md), qui présente une vue d’ensemble du déplacement de données avec l’activité de copie.

## <a name="supported-scenarios"></a>Scénarios pris en charge
Vous pouvez copier des données **depuis Azure SQL Database** vers les magasins de données suivants :

[!INCLUDE [data-factory-supported-sinks](../../../includes/data-factory-supported-sinks.md)]

Vous pouvez copier des données depuis les magasins de données suivants **vers Azure SQL Database**  :

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

## <a name="supported-authentication-type"></a>Type d’authentification pris en charge
Le connecteur Azure SQL Database prend en charge l’authentification de base.

## <a name="getting-started"></a>Prise en main
Vous pouvez créer un pipeline avec une activité de copie qui déplace les données vers/depuis Azure SQL Database à l’aide de différents outils/API.

Le moyen le plus simple de créer un pipeline consiste à utiliser **l’Assistant Copie** . Voir le [tutoriel : Créer un pipeline avec l’activité de copie à l’aide de l’Assistant Copie](data-factory-copy-data-wizard-tutorial.md) pour obtenir une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant Copie de données.

Vous pouvez également utiliser les outils suivants pour créer un pipeline : **Visual Studio** , **Azure PowerShell** , **modèle Azure Resource Manager** , **.NET API** et **API REST** . Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [didacticiel sur l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Que vous utilisiez des outils ou des API, la création d’un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur implique les étapes suivantes :

1. Création d'une **fabrique de données** . Une fabrique de données peut contenir un ou plusieurs pipelines.
2. Création de **services liés** pour lier les magasins de données d’entrée et de sortie à votre fabrique de données. Par exemple, si vous copiez des données depuis un stockage d’objets blob Azure vers une base de données Azure SQL Database, vous créez deux services liés pour lier votre compte de stockage Azure et votre base de données Azure SQL Database à votre fabrique de données. Pour les propriétés du service lié qui sont spécifiques à Azure SQL Database, consultez la section [propriétés du service lié](#linked-service-properties).
3. Création de **jeux de données** pour représenter les données d’entrée et de sortie de l’opération de copie. Dans l’exemple mentionné dans la dernière étape, vous créez un jeu de données pour spécifier le conteneur d’objets blob et le dossier qui contient les données d’entrée. Vous créez aussi un autre jeu de données pour spécifier la table SQL dans Azure SQL Database qui contient les données copiées depuis le stockage d’objets blob. Pour les propriétés du jeu de données qui sont spécifiques à Azure Data Lake Store, consultez la section [propriétés du jeu de données](#dataset-properties).
4. Création d’un **pipeline** avec une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie. Dans l’exemple mentionné plus haut, vous utilisez BlobSource comme source et SqlSink comme récepteur pour l’activité de copie. De même, si vous copiez depuis Azure SQL Database vers Stockage Blob Azure, vous utilisez SqlSource et BlobSink dans l’activité de copie. Pour les propriétés d’activité de copie qui sont spécifiques à Azure SQL Database, consultez la section [propriétés de l’activité de copie](#copy-activity-properties). Pour plus d’informations sur l’utilisation d’un magasin de données comme source ou comme récepteur, cliquez sur le lien de la section précédente de votre magasin de données.

Lorsque vous utilisez l’Assistant, les définitions JSON de ces entités Data Factory (services liés, jeux de données et pipeline) sont automatiquement créées pour vous. Lorsque vous utilisez des outils/API (à l’exception de l’API .NET), vous devez définir ces entités Data Factory au format JSON. Pour obtenir des exemples comportant des définitions JSON pour les entités Data Factory utilisées pour copier les données vers ou à partir d’Azure SQL Database, consultez la section [Exemples JSON](#json-examples-for-copying-data-to-and-from-sql-database) de cet article.

Les sections suivantes offrent des informations détaillées sur les propriétés JSON utilisées pour définir les entités Data Factory propres à Azure SQL Database :

## <a name="linked-service-properties"></a>Propriétés du service lié
Un service lié Azure SQL lie Azure SQL Database à votre fabrique de données. Le tableau suivant fournit la description des éléments JSON spécifiques au service lié SQL Azure.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |La propriété type doit être définie sur : **AzureSqlDatabase** |Oui |
| connectionString |Spécifier les informations requises pour la connexion à l’instance Azure SQL Database pour la propriété connectionString. Seule l’authentification de base est prise en charge. |Oui |

> [!IMPORTANT]
> Configurez le [pare-feu Azure SQL Database](/previous-versions/azure/ee621782(v=azure.100)#ConnectingFromAzure) et le serveur de base de données pour [autoriser les services Azure à accéder au serveur](/previous-versions/azure/ee621782(v=azure.100)#ConnectingFromAzure). En outre, si vous copiez des données vers Azure SQL Database à partir d’un emplacement situé en dehors d’Azure, y compris à partir de sources de données locales avec la passerelle de la fabrique de données, configurez la plage d’adresses IP appropriée pour l’ordinateur qui envoie des données à Azure SQL Database.

## <a name="dataset-properties"></a>Propriétés du jeu de données
Pour spécifier un jeu de données afin de représenter les données d’entrée ou de sortie dans Azure SQL Database, vous devez définir la propriété de type du jeu de données sur : **AzureSqlTable** . Définissez la propriété **linkedServiceName** du jeu de données sur le nom du service lié Azure SQL.

Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md). Les sections comme la structure, la disponibilité et la stratégie d'un jeu de données JSON sont similaires pour tous les types de jeux de données (SQL Azure, Azure Blob, Azure Table, etc.).

La section typeProperties est différente pour chaque type de jeu de données et fournit des informations sur l'emplacement des données dans le magasin de données. La section **typeProperties** pour le jeu de données de type **AzureSqlTable** a les propriétés suivantes :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table ou de la vue dans l’instance Azure SQL Database à laquelle le service lié fait référence. |Oui |

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie
Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les tables d’entrée et de sortie et la stratégie sont disponibles pour tous les types d’activités.

> [!NOTE]
> L'activité de copie accepte uniquement une entrée et produit une seule sortie.

En revanche, les propriétés disponibles dans la section **typeProperties** de l’activité varient pour chaque type d’activité. Pour l’activité de copie, elles dépendent des types de sources et récepteurs.

Si vous déplacez des données à partir d’Azure SQL Database, vous définissez le type de source dans l’activité de copie sur **SqlSource** . De même, si vous déplacez des données vers Azure SQL Database, vous définissez le type de récepteur dans l’activité de copie sur **SqlSink** . Cette section fournit une liste de propriétés prises en charge par SqlSource et SqlSink.

### <a name="sqlsource"></a>SqlSource
Dans le cas d’une activité de copie, quand la source est de type **SqlSource** , les propriétés suivantes sont disponibles dans la section **typeProperties**  :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| sqlReaderQuery |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Exemple : `select * from MyTable`. |Non |
| sqlReaderStoredProcedureName |Nom de la procédure stockée qui lit les données de la table source. |Nom de la procédure stockée. La dernière instruction SQL doit être une instruction SELECT dans la procédure stockée. |Non |
| storedProcedureParameters |Paramètres de la procédure stockée. |Paires nom/valeur. Les noms et la casse des paramètres doivent correspondre aux noms et à la casse des paramètres de la procédure stockée. |Non |

Si **sqlReaderQuery** est spécifié pour SqlSource, l'activité de copie exécute cette requête sur la source Azure SQL Database pour obtenir les données. Vous pouvez également spécifier une procédure stockée en indiquant **sqlReaderStoredProcedureName** et **storedProcedureParameters** (si la procédure stockée accepte des paramètres).

Si vous ne spécifiez pas sqlReaderQuery ou sqlReaderStoredProcedureName, les colonnes définies dans la section Structure du jeu de données JSON sont utilisées pour créer une requête (`select column1, column2 from mytable`) à exécuter sur la base de données Azure SQL. Si la définition du jeu de données ne possède pas de structure, toutes les colonnes de la table sont sélectionnées.

> [!NOTE]
> Quand vous utilisez **sqlReaderStoredProcedureName** , vous devez toujours spécifier une valeur pour la propriété **tableName** du code JSON du jeu de données. Cependant, il n’existe aucune validation effectuée pour cette table.
>
>

### <a name="sqlsource-example"></a>Exemple SqlSource

```JSON
"source": {
    "type": "SqlSource",
    "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
    "storedProcedureParameters": {
        "stringData": { "value": "str3" },
        "identifier": { "value": "$$Text.Format('{0:yyyy}', SliceStart)", "type": "Int"}
    }
}
```

**Définition de la procédure stockée :**

```SQL
CREATE PROCEDURE CopyTestSrcStoredProcedureWithParameters
(
    @stringData varchar(20),
    @identifier int
)
AS
SET NOCOUNT ON;
BEGIN
    select *
    from dbo.UnitTestSrcTable
    where dbo.UnitTestSrcTable.stringData != stringData
    and dbo.UnitTestSrcTable.identifier != identifier
END
GO
```

### <a name="sqlsink"></a>SqlSink
**SqlSink** prend en charge les propriétés suivantes :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| writeBatchTimeout |Temps d’attente pour que l’opération d’insertion de lot soit terminée avant d’expirer. |intervalle de temps<br/><br/> Exemple : “00:30:00” (30 minutes). |Non |
| writeBatchSize |Insère des données dans la table SQL lorsque la taille du tampon atteint writeBatchSize |Nombre entier (nombre de lignes) |Non (valeur par défaut : 10000) |
| sqlWriterCleanupScript |Spécifiez une requête pour exécuter l’activité de copie afin que les données d’un segment spécifique soient nettoyées. Pour en savoir plus, voir [Copie renouvelée](#repeatable-copy). |Une instruction de requête. |Non |
| sliceIdentifierColumnName |Spécifiez le nom de la colonne que l’activité de copie doit remplir avec l’identificateur de segment généré automatiquement, qui est utilisé pour nettoyer les données d’un segment spécifique lors de la ré-exécution. Pour en savoir plus, voir [Copie renouvelée](#repeatable-copy). |Nom d’une colonne avec le type de données binary(32). |Non |
| sqlWriterStoredProcedureName |Nom de la procédure stockée qui définit comment appliquer les données sources dans la table cible, par exemple pour effectuer des upserts ou des transformations à l’aide de votre propre logique métier. <br/><br/>Notez que cette procédure stockée sera **appelée par lot** . Si vous souhaitez effectuer une opération qui ne s’exécute qu’une seule fois et n’a rien à faire avec les données sources, par exemple supprimer/tronquer, utilisez la propriété `sqlWriterCleanupScript`. |Nom de la procédure stockée. |Non |
| storedProcedureParameters |Paramètres de la procédure stockée. |Paires nom/valeur. Les noms et la casse des paramètres doivent correspondre aux noms et à la casse des paramètres de la procédure stockée. |Non |
| sqlWriterTableType |Spécifiez le nom du type de table à utiliser dans la procédure stockée. L’activité de copie place les données déplacées disponibles dans une table temporaire avec ce type de table. Le code de procédure stockée peut ensuite fusionner les données copiées avec les données existantes. |Nom de type de table. |Non |

#### <a name="sqlsink-example"></a>Exemple SqlSink

```JSON
"sink": {
    "type": "SqlSink",
    "writeBatchSize": 1000000,
    "writeBatchTimeout": "00:05:00",
    "sqlWriterStoredProcedureName": "CopyTestStoredProcedureWithParameters",
    "sqlWriterTableType": "CopyTestTableType",
    "storedProcedureParameters": {
        "identifier": { "value": "1", "type": "Int" },
        "stringData": { "value": "str1" },
        "decimalData": { "value": "1", "type": "Decimal" }
    }
}
```

## <a name="json-examples-for-copying-data-to-and-from-sql-database"></a>Exemples JSON pour copier des données vers et depuis SQL Database
Les exemples suivants présentent des exemples de définitions JSON que vous pouvez utiliser pour créer un pipeline à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou d’[Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Ils indiquent comment copier des données vers et depuis une base de données Azure SQL et Azure Blob Storage. Toutefois, les données peuvent être copiées **directement** vers l’un des récepteurs indiqués [ici](data-factory-data-movement-activities.md#supported-data-stores-and-formats) , via l’activité de copie de Microsoft Azure Data Factory.

### <a name="example-copy-data-from-azure-sql-database-to-azure-blob"></a>Exemple : copier des données d’Azure SQL Database vers Stockage Blob Azure
L’exemple définit les entités de fabrique de données suivantes :

1. Un service lié de type [AzureSqlDatabase](#linked-service-properties).
2. Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Un [jeu de données](data-factory-create-datasets.md) d'entrée de type [AzureSqlTable](#dataset-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type [Azure Blob](data-factory-azure-blob-connector.md#dataset-properties).
5. Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [SqlSource](#copy-activity-properties) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

L’exemple copie toutes les heures les données de série chronologique (horaire, journalière, etc.) d’une table d’une base dans Azure SQL Database vers un objet blob. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

**Service lié Azure SQL Database :**

```JSON
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
Consultez la section Service lié SQL Azure pour obtenir la liste des propriétés prises en charge par ce service lié.

**Service lié Azure Blob Storage :**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Consultez l’article [Objets blob Azure](data-factory-azure-blob-connector.md#azure-storage-linked-service) pour obtenir la liste des propriétés prises en charge par ce service lié.


**Jeu de données d'entrée SQL Azure :**

L'exemple suppose que vous avez créé une table « MyTable » dans SQL Azure et qu'elle contient une colonne appelée « timestampcolumn » pour les données de série chronologique.

La définition de « external » : « true» informe le service Azure Data Factory que le jeu de données est externe à la fabrique de données et n’est pas produit par une activité dans la fabrique de données.

```JSON
{
  "name": "AzureSqlInput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

Consultez la section Propriétés de type du jeu de données SQL Azure pour obtenir la liste des propriétés prises en charge par ce type de jeu de données.

**Jeu de données de sortie d'objet Blob Azure :**

Les données sont écrites dans un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1). Le chemin d’accès du dossier pour l’objet blob est évalué dynamiquement en fonction de l’heure de début du segment en cours de traitement. Le chemin d'accès du dossier utilise l'année, le mois, le jour et l'heure de l'heure de début.

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}/",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
Consultez la section [Propriétés de type du jeu de données d’objets blob Azure](data-factory-azure-blob-connector.md#dataset-properties) pour obtenir la liste des propriétés prises en charge par ce type de jeu de données.

**Activité de copie dans un pipeline avec une source SQL et un récepteur blob :**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d'entrée et de sortie, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **SqlSource** et le type **sink** est défini sur **BlobSink** . La requête SQL spécifiée pour la propriété **SqlReaderQuery** sélectionne les données de la dernière heure à copier.

```JSON
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline for copy activity",
    "activities":[
      {
        "name": "AzureSQLtoBlob",
        "description": "copy activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureSQLInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "SqlSource",
            "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
        "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ]
  }
}
```
Dans l’exemple, **sqlReaderQuery** est spécifié pour SqlSource. L'activité de copie exécute cette requête sur la source Azure SQL Database pour obtenir les données. Vous pouvez également spécifier une procédure stockée en indiquant **sqlReaderStoredProcedureName** et **storedProcedureParameters** (si la procédure stockée accepte des paramètres).

Si vous ne spécifiez pas sqlReaderQuery ou sqlReaderStoredProcedureName, les colonnes définies dans la section Structure du jeu de données JSON sont utilisées pour créer une requête à exécuter sur Azure SQL Database. Par exemple : `select column1, column2 from mytable`. Si la définition du jeu de données ne possède pas de structure, toutes les colonnes de la table sont sélectionnées.

Consultez la section [Sql Source](#sqlsource) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) pour obtenir la liste des propriétés prises en charge par SqlSource et BlobSink.

### <a name="example-copy-data-from-azure-blob-to-azure-sql-database"></a>Exemple : copier des données de Stockage Blob Azure vers Azure SQL Database
L’exemple définit les entités Data Factory suivantes :

1. Un service lié de type [AzureSqlDatabase](#linked-service-properties).
2. Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. un [jeu de données](data-factory-create-datasets.md) d'entrée de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureSqlTable](#dataset-properties).
5. Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) et [SqlSink](#copy-activity-properties).

L’exemple copie toutes les heures les données temporelles (horaire, journalière, etc.) d’un objet blob Azure vers une table dans Azure SQL Database. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

**Service lié SQL Azure :**

```JSON
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
Consultez la section Service lié SQL Azure pour obtenir la liste des propriétés prises en charge par ce service lié.

**Service lié Azure Blob Storage :**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Consultez l’article [Objets blob Azure](data-factory-azure-blob-connector.md#azure-storage-linked-service) pour obtenir la liste des propriétés prises en charge par ce service lié.


**Jeu de données d'entrée d'objet Blob Azure :**

Les données sont récupérées à partir d’un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1). Le nom du chemin d'accès et du fichier de dossier pour l'objet blob sont évalués dynamiquement en fonction de l'heure de début du segment en cours de traitement. Le chemin d’accès du dossier utilise l’année, le mois et le jour de début et le nom de fichier utilise l’heure de début. Le paramètre « external » : « true » informe le service Data Factory que cette table est externe à la fabrique de données et n’est pas produite par une activité dans la fabrique de données.

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/",
      "fileName": "{Hour}.csv",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": "\n"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```
Consultez la section [Propriétés de type du jeu de données d’objets blob Azure](data-factory-azure-blob-connector.md#dataset-properties) pour obtenir la liste des propriétés prises en charge par ce type de jeu de données.

**Jeu de données de sortie Azure SQL Database :**

L'exemple copie les données dans une table nommée « MyTable » dans SQL Azure. Créez la table dans SQL Azure avec le même nombre de colonnes que le fichier CSV d’objets blob doit en contenir. De nouvelles lignes sont ajoutées à la table toutes les heures.

```JSON
{
  "name": "AzureSqlOutput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
Consultez la section Propriétés de type du jeu de données SQL Azure pour obtenir la liste des propriétés prises en charge par ce type de jeu de données.

**Activité de copie dans un pipeline avec une source blob et un récepteur SQL :**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d'entrée et de sortie, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **BlobSource** et le type **sink** est défini sur **SqlSink** .

```JSON
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[
      {
        "name": "AzureBlobtoSQL",
        "description": "Copy Activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSqlOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource",
            "blobColumnSeparators": ","
          },
          "sink": {
            "type": "SqlSink"
          }
        },
        "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ]
  }
}
```
Consultez la section [Sql Sink](#sqlsink) et [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) pour obtenir la liste des propriétés prises en charge par SqlSink et BlobSource.

## <a name="identity-columns-in-the-target-database"></a>Colonnes d’identité dans la base de données cible
Cette section fournit un exemple pour copier des données d’une table source sans colonne d’identité vers une table de destination avec une colonne d’identité.

**Table source :**

```SQL
create table dbo.SourceTbl
(
    name varchar(100),
    age int
)
```
**Table de destination :**

```SQL
create table dbo.TargetTbl
(
    identifier int identity(1,1),
    name varchar(100),
    age int
)
```
Notez que la table cible possède une colonne d’identité.

**Définition du jeu de données JSON source**

```JSON
{
    "name": "SampleSource",
    "properties": {
        "type": " SqlServerTable",
        "linkedServiceName": "TestIdentitySQL",
        "typeProperties": {
            "tableName": "SourceTbl"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```
**Définition de jeu de données JSON de destination**

```JSON
{
    "name": "SampleTarget",
    "properties": {
        "structure": [
            { "name": "name" },
            { "name": "age" }
        ],
        "type": "AzureSqlTable",
        "linkedServiceName": "TestIdentitySQLSource",
        "typeProperties": {
            "tableName": "TargetTbl"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": false,
        "policy": {}
    }
}
```

Notez que vos tables source et cible ont des schémas différents (la cible possède une colonne supplémentaire avec identité). Dans ce scénario, vous devez spécifier la propriété **structure** dans la définition du jeu de données cible, qui n’inclut pas la colonne d’identité.

## <a name="invoke-stored-procedure-from-sql-sink"></a>Appel d’une procédure stockée pour un récepteur SQL
Pour obtenir un exemple d’appel d’une procédure stockée à partir d’un récepteur SQL dans l’activité de copie d’un pipeline, consultez l’article [Appeler une procédure stockée pour un récepteur SQL dans l’activité de copie](data-factory-invoke-stored-procedure-from-copy-activity.md).

## <a name="type-mapping-for-azure-sql-database"></a>Mappage de type pour Azure SQL Database
Comme mentionné dans l’article consacré aux [activités de déplacement des données](data-factory-data-movement-activities.md) , l’activité de copie convertit automatiquement les types source en types récepteur à l’aide de l’approche en 2 étapes suivante :

1. Conversion de types natifs source en types .NET
2. Conversion à partir du type .NET en type de récepteur natif

Lors du déplacement des données vers et à partir d’Azure SQL Database, les mappages suivants sont utilisés depuis le type SQL vers le type .NET, et vice-versa. Le mappage est identique au mappage du type de données SQL Server pour ADO.NET.

| Type de moteur de base de données SQL Server | Type .NET Framework |
| --- | --- |
| bigint |Int64 |
| binary |Byte[] |
| bit |Boolean |
| char |String, Char[] |
| Date |DateTime |
| Datetime |DateTime |
| datetime2 |DateTime |
| Datetimeoffset |DateTimeOffset |
| Decimal |Decimal |
| FILESTREAM attribute (varbinary(max)) |Byte[] |
| Float |Double |
| image |Byte[] |
| int |Int32 |
| money |Decimal |
| NCHAR |String, Char[] |
| ntext |String, Char[] |
| numeric |Decimal |
| NVARCHAR |String, Char[] |
| real |Unique |
| rowversion |Byte[] |
| smalldatetime |DateTime |
| SMALLINT |Int16 |
| SMALLMONEY |Decimal |
| sql_variant |Object * |
| text |String, Char[] |
| time |TimeSpan |
| timestamp |Byte[] |
| TINYINT |Byte |
| UNIQUEIDENTIFIER |Guid |
| varbinary |Byte[] |
| varchar |String, Char[] |
| Xml |Xml |

## <a name="map-source-to-sink-columns"></a>Mapper les colonnes source aux colonnes du récepteur
Pour en savoir plus sur le mappage de colonnes du jeu de données source à des colonnes du jeu de données récepteur, voir [Mappage des colonnes d’un jeu de données dans Azure Data Factory](data-factory-map-columns.md).

## <a name="repeatable-copy"></a>Copie renouvelée
Lors de la copie de données sur une base de données SQL Server, l’activité de copie ajoute des données à la table de récepteur par défaut. Pour effectuer une opération UPSERT à la place, consultez l’article [Écriture renouvelée sur SqlSink](data-factory-repeatable-copy.md#repeatable-write-to-sqlsink).

Lorsque vous copiez des données à partir de magasins de données relationnels, gardez à l’esprit la répétabilité de l’opération, afin d’éviter des résultats imprévus. Dans Azure Data Factory, vous pouvez réexécuter une tranche manuellement. Vous pouvez également configurer une stratégie de nouvelles tentatives pour un jeu de données, afin qu’une tranche soit réexécutée en cas de défaillance. Lorsqu’une tranche est réexécutée d’une manière ou d’une autre, vous devez vous assurer que les mêmes données sont lues et ce, quel que soit le nombre d’exécutions de la tranche. Voir [Lecture renouvelée de sources relationnelles](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="performance-and-tuning"></a>Performances et réglage
Consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser.