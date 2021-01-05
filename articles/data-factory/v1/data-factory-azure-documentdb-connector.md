---
title: Déplacer des données vers et à partir d’Azure Cosmos DB
description: Découvrez comment déplacer des données vers et à partir de la collection Azure Cosmos DB à l’aide d’Azure Data Factory.
services: data-factory, cosmosdb
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: c9297b71-1bb4-4b29-ba3c-4cf1f5575fac
ms.service: multiple
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 91f2b338ac9cda96521a5fe1d555de054826e273
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92637630"
---
# <a name="move-data-to-and-from-azure-cosmos-db-using-azure-data-factory"></a>Déplacer des données vers et à partir d’Azure Cosmos DB à l’aide d’Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-azure-documentdb-connector.md)
> * [Version 2 (version actuelle)](../connector-azure-cosmos-db.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Connecteur Azure Cosmos DB dans V2](../connector-azure-cosmos-db.md).

Cet article explique comment utiliser l’activité de copie d’Azure Data Factory pour déplacer des données vers ou à partir d’Azure Cosmos DB (API SQL). Il s’appuie sur l’article [Activités de déplacement des données](data-factory-data-movement-activities.md), qui présente une vue d’ensemble du déplacement de données avec l’activité de copie.

Vous pouvez copier des données du magasin de données source pris en charge de votre choix vers Azure Cosmos DB, ou d’Azure Cosmos DB vers le magasin de données récepteur pris en charge de votre choix. Pour obtenir la liste des magasins de données pris en charge en tant que sources ou récepteurs pour l’activité de copie, consultez le tableau [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats).

> [!IMPORTANT]
> Le connecteur Azure Cosmos DB ne prend en charge que l’API SQL.

Pour copier des données en l’état vers ou à partir de fichiers JSON ou une autre collection Cosmos DB, consultez la page [Import/Export de documents JSON](#importexport-json-documents).

## <a name="getting-started"></a>Prise en main
Vous pouvez créer un pipeline avec une activité de copie qui déplace les données vers ou à partir d’Azure Cosmos DB à l’aide de différents outils et API.

Le moyen le plus simple de créer un pipeline consiste à utiliser **l’Assistant Copie** . Voir le [tutoriel : Créer un pipeline avec l’activité de copie à l’aide de l’Assistant Copie](data-factory-copy-data-wizard-tutorial.md) pour obtenir une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant Copie de données.

Vous pouvez également utiliser les outils suivants pour créer un pipeline : **Visual Studio** , **Azure PowerShell** , **modèle Azure Resource Manager** , **.NET API** et **API REST** . Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [didacticiel sur l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Que vous utilisiez des outils ou des API, la création d’un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur implique les étapes suivantes :

1. Création de **services liés** pour lier les magasins de données d’entrée et de sortie à votre fabrique de données.
2. Création de **jeux de données** pour représenter les données d’entrée et de sortie de l’opération de copie.
3. Création d’un **pipeline** avec une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie.

Lorsque vous utilisez l’Assistant, les définitions JSON de ces entités Data Factory (services liés, jeux de données et pipeline) sont automatiquement créées pour vous. Lorsque vous utilisez des outils/API (à l’exception de l’API .NET), vous devez définir ces entités Data Factory au format JSON. Pour obtenir des exemples comportant des définitions JSON d’entités Data Factory utilisées pour copier les données vers ou à partir de Cosmos DB, consultez la section [Exemples JSON](#json-examples) de cet article.

Les sections suivantes fournissent des informations sur les propriétés JSON utilisées pour définir des entités Data Factory propres à Cosmos DB :

## <a name="linked-service-properties"></a>Propriétés du service lié
Le tableau suivant fournit une description des éléments JSON propres au service lié Azure Cosmos DB.

| **Propriété** | **Description** | **Obligatoire** |
| --- | --- | --- |
| type |La propriété type doit être définie sur : **DocumentDb** |Oui |
| connectionString |Spécifiez les informations requises pour se connecter à la base de données Azure Cosmos DB. |Oui |

Exemple :

```JSON
{
  "name": "CosmosDbLinkedService",
  "properties": {
    "type": "DocumentDb",
    "typeProperties": {
      "connectionString": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
    }
  }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données
Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l'article [Création de jeux de données](data-factory-create-datasets.md). Les sections comme la structure, la disponibilité et la stratégie d'un jeu de données JSON sont similaires pour tous les types de jeux de données (SQL Azure, Azure Blob, Azure Table, etc.).

La section typeProperties est différente pour chaque type de jeu de données et fournit des informations sur l'emplacement des données dans le magasin de données. La section typeProperties pour le jeu de données de type **DocumentDbCollection** a les propriétés suivantes.

| **Propriété** | **Description** | **Obligatoire** |
| --- | --- | --- |
| collectionName |Nom de la collection de documents Cosmos DB. |Oui |

Exemple :

```JSON
{
  "name": "PersonCosmosDbTable",
  "properties": {
    "type": "DocumentDbCollection",
    "linkedServiceName": "CosmosDbLinkedService",
    "typeProperties": {
      "collectionName": "Person"
    },
    "external": true,
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```
### <a name="schema-by-data-factory"></a>Schéma par Data Factory
Pour les magasins de données sans schéma comme Azure Cosmos DB, le service Data Factory déduit le schéma de l’une des manières suivantes :

1. Si vous spécifiez la structure des données à l’aide de la propriété **structure** dans la définition du jeu de données, le service Data Factory respecte cette structure en tant que schéma. Dans ce cas, si une ligne ne contient pas de valeur pour une colonne, une valeur null est fournie pour celle-ci.
2. Si vous ne spécifiez pas la structure des données à l’aide de la propriété **structure** dans la définition du jeu de données, le service Data Factory déduit le schéma à l’aide de la première ligne dans les données. Dans ce cas, si la première ligne ne contient pas le schéma complet, certaines colonnes ne seront pas incluses dans le résultat de l’opération de copie.

Par conséquent, pour les sources de données sans schéma, la meilleure pratique consiste à définir la structure des données à l’aide de la propriété **structure** .

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie
Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les tables d’entrée et de sortie et la stratégie sont disponibles pour tous les types d’activités.

> [!NOTE]
> L'activité de copie accepte uniquement une entrée et produit une seule sortie.

Par contre, les propriétés disponibles dans la section typeProperties de l'activité varient avec chaque type d'activité et dans le cas de l'activité de copie, elles varient selon les types de sources et de récepteurs.

Dans le cas d’une activité de copie, quand la source est de type **DocumentDbCollectionSource** , les propriétés suivantes sont disponibles dans la section **typeProperties**  :

| **Propriété** | **Description** | **Valeurs autorisées** | **Obligatoire** |
| --- | --- | --- | --- |
| query |Spécifier la requête pour lire les données. |Chaîne de requête prise en charge par Azure Cosmos DB. <br/><br/>Exemple : `SELECT c.BusinessEntityID, c.PersonType, c.NameStyle, c.Title, c.Name.First AS FirstName, c.Name.Last AS LastName, c.Suffix, c.EmailPromotion FROM c WHERE c.ModifiedDate > \"2009-01-01T00:00:00\"` |Non <br/><br/>Si non spécifié, l’instruction SQL exécutée : `select <columns defined in structure> from mycollection` |
| nestingSeparator |Caractère spécial pour indiquer que le document est imbriqué. |Tout caractère. <br/><br/>Azure Cosmos DB est une banque NoSQL de documents JSON, où les structures imbriquées sont autorisées. Azure Data Factory permet à l'utilisateur de désigner la hiérarchie via nestingSeparator, qui est « . » dans les exemples ci-dessus. Avec le séparateur, l'activité de copie générera l'objet « Name » avec trois éléments enfants First, Middle et Last, en fonction de « Name.First », « Name.Middle » et « Name.Last » dans la définition de la table. |Non |

**DocumentDbCollectionSink** prend en charge les propriétés suivantes :

| **Propriété** | **Description** | **Valeurs autorisées** | **Obligatoire** |
| --- | --- | --- | --- |
| nestingSeparator |Caractère spécial dans le nom de colonne source pour indiquer que le document imbriqué est nécessaire. <br/><br/>Par exemple, ci-dessus : `Name.First` dans la table de sortie produit la structure JSON suivante dans le document Cosmos DB :<br/><br/>"Name": {<br/>    "First": "John"<br/>}, |Caractère utilisé pour séparer les niveaux d'imbrication.<br/><br/>La valeur par défaut est `.` (point). |Caractère utilisé pour séparer les niveaux d'imbrication. <br/><br/>La valeur par défaut est `.` (point). |
| writeBatchSize |Nombre de requêtes parallèles auprès du service Azure Cosmos DB pour créer des documents.<br/><br/>Vous pouvez optimiser les performances lors de la copie des données vers ou à partir de Cosmos DB à l’aide de cette propriété. Vous pouvez obtenir de meilleures performances en augmentant writeBatchSize, c’est-à-dire le nombre de requêtes parallèles envoyées à Cosmos DB. Toutefois, vous devez éviter les limitations qui peuvent déclencher le message d’erreur : « Le taux de requêtes est élevé ».<br/><br/>Une limitation dépend de divers facteurs, dont la taille des documents, le nombre de termes qu’ils contiennent, la stratégie d’indexation de la collection cible, etc. Pour les opérations de copie, vous pouvez utiliser une meilleure collection (par exemple, S3) pour que le débit disponible soit maximal (2 500 unités de requêtes par seconde). |Integer |Non (valeur par défaut : 5) |
| writeBatchTimeout |Temps d'attente pour que l'opération soit terminée avant d'expirer. |intervalle de temps<br/><br/> Exemple : “00:30:00” (30 minutes). |Non |

## <a name="importexport-json-documents"></a>Importation/exportation de documents JSON
À l’aide de ce connecteur Cosmos DB, vous pouvez facilement :

* Importer des documents JSON de différentes sources dans Cosmos DB, notamment le Stockage Blob Azure, Azure Data Lake, un système de fichiers local ou d’autres banques basées sur des fichiers prises en charge par Azure Data Factory.
* Exporter des documents JSON d’une collection Cosmos DB vers différentes banques basées sur des fichiers.
* Migrer des données entre deux collections Cosmos DB en l’état.

Pour obtenir une copie de ce type, indépendante du schéma :
* Si vous utilisez l’Assistant Copie, cochez l’option **« Exporter en l’état dans des fichiers JSON ou une collection Cosmos DB »** .
* Si vous utilisez l’édition JSON, ne spécifiez pas la section « structure » dans le ou les jeux de données Cosmos DB, ni la propriété « nestingSeparator » dans la source / le récepteur Cosmos DB de l’activité de copie. Pour importer à partir de/exporter dans des fichiers JSON, dans le jeu de données du magasin de fichiers, spécifiez le type de format « JsonFormat », configurez « filePattern » et ignorez les paramètres de format rest. Consultez la section [Format JSON](data-factory-supported-file-and-compression-formats.md#json-format) pour plus de détails.

## <a name="json-examples"></a>Exemples JSON
Les exemples suivants présentent des exemples de définitions JSON que vous pouvez utiliser pour créer un pipeline à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou d’[Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Ils indiquent comment copier des données vers et à partir du Stockage Blob Azure et d’Azure Cosmos DB. Toutefois, les données peuvent être copiées **directement** vers l’un des récepteurs indiqués [ici](data-factory-data-movement-activities.md#supported-data-stores-and-formats) , via l’activité de copie de Microsoft Azure Data Factory.

## <a name="example-copy-data-from-azure-cosmos-db-to-azure-blob"></a>Exemple : copier des données d’Azure Cosmos DB vers un objet Blob Azure
L’exemple ci-dessous présente les éléments suivants :

1. Un service lié de type [DocumentDb](#linked-service-properties).
2. Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [DocumentDbCollection](#dataset-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
5. Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [DocumentDbCollectionSource](#copy-activity-properties) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

L'exemple copie des données d’Azure Cosmos DB vers un objet Blob Azure. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

**Service lié Azure Cosmos DB :**

```JSON
{
  "name": "CosmosDbLinkedService",
  "properties": {
    "type": "DocumentDb",
    "typeProperties": {
      "connectionString": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
    }
  }
}
```
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
**Jeu de données d'entrée Document DB Azure :**

L’exemple suppose que vous avez une collection nommée **Person** dans une base de données Azure Cosmos DB.

La définition de « external » : « true » et la spécification de la stratégie externalData informent le service Azure Data Factory qu'il s'agit d'une table qui est externe à la Data Factory et non produite par une activité dans la Data Factory.

```JSON
{
  "name": "PersonCosmosDbTable",
  "properties": {
    "type": "DocumentDbCollection",
    "linkedServiceName": "CosmosDbLinkedService",
    "typeProperties": {
      "collectionName": "Person"
    },
    "external": true,
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```

**Jeu de données de sortie d'objet Blob Azure :**

Les données sont copiées vers un nouvel objet blob toutes les heures. Le chemin d'accès à l'objet blob reflète la valeur date-heure spécifique avec une granularité horaire.

```JSON
{
  "name": "PersonBlobTableOut",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "docdb",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "nullValue": "NULL"
      }
    },
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```
Exemple de document JSON dans la collection Person d’une base de données Cosmos DB :

```JSON
{
  "PersonId": 2,
  "Name": {
    "First": "Jane",
    "Middle": "",
    "Last": "Doe"
  }
}
```
Cosmos DB prend en charge l'interrogation de documents à l'aide d'une syntaxe de type SQL sur des documents JSON hiérarchiques.

Exemple :

```sql
SELECT Person.PersonId, Person.Name.First AS FirstName, Person.Name.Middle as MiddleName, Person.Name.Last AS LastName FROM Person
```

Le pipeline suivant copie les données de la collection Person de la base de données Cosmos DB vers un objet Blob Azure. Dans le cadre de l'activité de copie, les jeux de données d'entrée et de sortie ont été spécifiés.

```JSON
{
  "name": "DocDbToBlobPipeline",
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "DocumentDbCollectionSource",
            "query": "SELECT Person.Id, Person.Name.First AS FirstName, Person.Name.Middle as MiddleName, Person.Name.Last AS LastName FROM Person",
            "nestingSeparator": "."
          },
          "sink": {
            "type": "BlobSink",
            "blobWriterAddHeader": true,
            "writeBatchSize": 1000,
            "writeBatchTimeout": "00:00:59"
          }
        },
        "inputs": [
          {
            "name": "PersonCosmosDbTable"
          }
        ],
        "outputs": [
          {
            "name": "PersonBlobTableOut"
          }
        ],
        "policy": {
          "concurrency": 1
        },
        "name": "CopyFromDocDbToBlob"
      }
    ],
    "start": "2015-04-01T00:00:00Z",
    "end": "2015-04-02T00:00:00Z"
  }
}
```
## <a name="example-copy-data-from-azure-blob-to-azure-cosmos-db"></a>Exemple : copier des données d’un objet Blob Azure vers Azure Cosmos DB
L’exemple ci-dessous présente les éléments suivants :

1. Un service lié de type DocumentDb.
2. Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. un [jeu de données](data-factory-create-datasets.md) d'entrée de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type DocumentDbCollection.
5. Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) et DocumentDbCollectionSink.

L'exemple copie des données d’un objet Blob Azure vers Azure Cosmos DB. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

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
**Service lié Azure Cosmos DB :**

```JSON
{
  "name": "CosmosDbLinkedService",
  "properties": {
    "type": "DocumentDb",
    "typeProperties": {
      "connectionString": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
    }
  }
}
```
**Jeu de données d'entrée d'objet Blob Azure :**

```JSON
{
  "name": "PersonBlobTableIn",
  "properties": {
    "structure": [
      {
        "name": "Id",
        "type": "Int"
      },
      {
        "name": "FirstName",
        "type": "String"
      },
      {
        "name": "MiddleName",
        "type": "String"
      },
      {
        "name": "LastName",
        "type": "String"
      }
    ],
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "fileName": "input.csv",
      "folderPath": "docdb",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "nullValue": "NULL"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```
**Jeu de données de sortie Azure Cosmos DB :**

L'exemple copie des données vers une collection nommée « Person ».

```JSON
{
  "name": "PersonCosmosDbTableOut",
  "properties": {
    "structure": [
      {
        "name": "Id",
        "type": "Int"
      },
      {
        "name": "Name.First",
        "type": "String"
      },
      {
        "name": "Name.Middle",
        "type": "String"
      },
      {
        "name": "Name.Last",
        "type": "String"
      }
    ],
    "type": "DocumentDbCollection",
    "linkedServiceName": "CosmosDbLinkedService",
    "typeProperties": {
      "collectionName": "Person"
    },
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```
Le pipeline suivant copie les données d’un objet Blob Azure vers la collection Person de Cosmos DB. Dans le cadre de l'activité de copie, les jeux de données d'entrée et de sortie ont été spécifiés.

```JSON
{
  "name": "BlobToDocDbPipeline",
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "DocumentDbCollectionSink",
            "nestingSeparator": ".",
            "writeBatchSize": 2,
            "writeBatchTimeout": "00:00:00"
          },
          "translator": {
              "type": "TabularTranslator",
              "ColumnMappings": "FirstName: Name.First, MiddleName: Name.Middle, LastName: Name.Last, BusinessEntityID: BusinessEntityID, PersonType: PersonType, NameStyle: NameStyle, Title: Title, Suffix: Suffix, EmailPromotion: EmailPromotion, rowguid: rowguid, ModifiedDate: ModifiedDate"
          }
        },
        "inputs": [
          {
            "name": "PersonBlobTableIn"
          }
        ],
        "outputs": [
          {
            "name": "PersonCosmosDbTableOut"
          }
        ],
        "policy": {
          "concurrency": 1
        },
        "name": "CopyFromBlobToDocDb"
      }
    ],
    "start": "2015-04-14T00:00:00Z",
    "end": "2015-04-15T00:00:00Z"
  }
}
```
Si l'entré de l'objet blob d'exemple est

```
1,John,,Doe
```
Alors, la sortie JSON dans Cosmos DB sera :

```JSON
{
  "Id": 1,
  "Name": {
    "First": "John",
    "Middle": null,
    "Last": "Doe"
  },
  "id": "a5e8595c-62ec-4554-a118-3940f4ff70b6"
}
```
Azure Cosmos DB est une banque NoSQL de documents JSON, où les structures imbriquées sont autorisées. Azure Data Factory permet à l’utilisateur de désigner la hiérarchie via **nestingSeparator** , qui est « . » dans cet exemple. Avec le séparateur, l'activité de copie générera l'objet « Name » avec trois éléments enfants First, Middle et Last, en fonction de « Name.First », « Name.Middle » et « Name.Last » dans la définition de la table.

## <a name="appendix"></a>Annexe
1. **Question :** L’activité de copie prend-elle en charge la mise à jour d’enregistrements existants ?

    **Réponse :** Non.
2. **Question :** Comment une nouvelle tentative de copie vers Azure Cosmos DB gère-t-elle les enregistrements déjà copiés ?

    **Réponse :** Si les enregistrements ont un champ « ID » et que l’opération de copie tente d’insérer un enregistrement avec le même ID, l’opération de copie génère une erreur.
3. **Question :** Data Factory prend-il en charge le [partitionnement de données basé sur un intervalle ou sur le hachage](../../cosmos-db/partitioning-overview.md) ?

    **Réponse :** Non.
4. **Question :** Puis-je indiquer plusieurs collections Azure Cosmos DB pour une table ?

    **Réponse :** Non. Il n’est possible d’indiquer qu’une collection pour le moment.

## <a name="performance-and-tuning"></a>Performances et réglage
Consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser.