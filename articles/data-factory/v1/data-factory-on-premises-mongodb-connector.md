---
title: Déplacer des données à partir de MongoDB
description: Découvrez comment déplacer des données depuis une base de données MongoDB à l’aide d’Azure Data Factory.
services: data-factory
author: linda33wj
ms.author: jingwang
manager: shwang
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 04/13/2018
ms.openlocfilehash: edddd100bddab1d642a8169353298a2d20620274
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "79236349"
---
# <a name="move-data-from-mongodb-using-azure-data-factory"></a>Déplacer des données depuis MongoDB à l’aide d’Azure Data Factory

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-on-premises-mongodb-connector.md)
> * [Version 2 (version actuelle)](../connector-mongodb.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Connecteur MongoDB dans V2](../connector-mongodb.md).


Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour déplacer des données à partir d’une base de données MongoDB locale. Il s’appuie sur l’article [Activités de déplacement des données](data-factory-data-movement-activities.md), qui présente une vue d’ensemble du déplacement de données avec l’activité de copie.

Vous pouvez copier les données d’un magasin de données MongoDB local dans tout magasin de données récepteur pris en charge. Consultez le tableau [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) pour obtenir la liste des magasins de données pris en charge en tant que récepteurs par l’activité de copie. Actuellement, les fabriques de données ne prennent en charge que le déplacement des données d’un magasin de données MongoDB vers d’autres magasins de données, mais pas l’inverse.

## <a name="prerequisites"></a>Prérequis
Pour permettre au service Azure Data Factory de se connecter à votre base de données MongoDB locale, vous devez installer les composants suivants :

- Versions MongoDB prises en charge : 2.4, 2.6, 3.0, 3.2, 3.4 et 3.6.
- Une passerelle de gestion de données sur l’ordinateur qui héberge la base de données ou sur un autre ordinateur afin d’éviter toute mise en concurrence avec la base de données pour les ressources. La passerelle de gestion de données est un logiciel qui connecte des sources de données locales à des services cloud de manière gérée et sécurisée. Consultez l’article [Passerelle de gestion des données](data-factory-data-management-gateway.md) pour obtenir des informations détaillées sur la passerelle de gestion des données. Consultez l’article [Déplacement de données entre des sources locales et le cloud à l’aide de la passerelle de gestion des données](data-factory-move-data-between-onprem-and-cloud.md) pour obtenir des instructions détaillées sur la configuration de la passerelle, un pipeline de données, pour déplacer des données.

    L’installation de la passerelle engendre automatiquement l’installation d’un pilote Microsoft ODBC MongoDB, utilisé pour se connecter à MongoDB.

    > [!NOTE]
    > Vous devez utiliser la passerelle pour vous connecter à MongoDB même si elle est hébergée sur des machines virtuelles IaaS Azure. Si vous essayez de vous connecter à une instance MongoDB hébergée dans le cloud, vous pouvez également installer l’instance de la passerelle dans la machine virtuelle IaaS.

## <a name="getting-started"></a>Prise en main
Vous pouvez créer un pipeline avec une activité de copie qui déplace les données d’un magasin de données MongoDB local à l’aide de différents outils/API.

Le moyen le plus simple de créer un pipeline consiste à utiliser **l’Assistant Copie**. Voir le [tutoriel : Créer un pipeline avec l’activité de copie à l’aide de l’Assistant Copie](data-factory-copy-data-wizard-tutorial.md) pour obtenir une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant Copie de données.

Vous pouvez également utiliser les outils suivants pour créer un pipeline : **Visual Studio**, **Azure PowerShell**, **modèle Azure Resource Manager**, **.NET API** et **API REST**. Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [didacticiel sur l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Que vous utilisiez des outils ou des API, la création d’un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur implique les étapes suivantes :

1. Création de **services liés** pour lier les magasins de données d’entrée et de sortie à votre fabrique de données.
2. Création de **jeux de données** pour représenter les données d’entrée et de sortie de l’opération de copie.
3. Création d’un **pipeline** avec une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie.

Lorsque vous utilisez l’Assistant, les définitions JSON de ces entités Data Factory (services liés, jeux de données et pipeline) sont automatiquement créées pour vous. Lorsque vous utilisez des outils/API (à l’exception de l’API .NET), vous devez définir ces entités Data Factory au format JSON.  Pour un exemple contenant des définitions JSON pour les entités Data Factory servant à copier des données à partir d’un magasin de données MongoDB local, consultez la section [Exemple JSON : copier des données de MongoDB vers Stockage Blob Azure](#json-example-copy-data-from-mongodb-to-azure-blob) de cet article.

Les sections suivantes contiennent des informations détaillées sur les propriétés JSON utilisées pour définir les entités Data Factory propres à la source MongoDB :

## <a name="linked-service-properties"></a>Propriétés du service lié
La table suivante fournit une description des éléments JSON spécifiques au service lié **OnPremisesMongoDB** .

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |La propriété type doit être définie sur : **OnPremisesMongoDb** |Oui |
| server |Nom d’hôte ou adresse IP du serveur MongoDB. |Oui |
| port |Le port TCP utilisé par le serveur MongoDB pour écouter les connexions clientes. |Facultatif, valeur par défaut : 27017 |
| authenticationType |De base ou anonyme. |Oui |
| username |Compte d’utilisateur pour accéder à MongoDB. |Oui (si l’authentification de base est utilisée). |
| mot de passe |Mot de passe pour l’utilisateur. |Oui (si l’authentification de base est utilisée). |
| authSource |Nom de la base de données MongoDB que vous souhaitez utiliser pour vérifier vos informations d’identification pour l’authentification. |Facultatif (si l’authentification de base est utilisée). Par défaut : utilise le compte d’administrateur et la base de données spécifiée à l’aide de la propriété databaseName. |
| databaseName |Nom de la base de données MongoDB à laquelle vous souhaitez accéder. |Oui |
| gatewayName |Nom de la passerelle qui accède au magasin de données. |Oui |
| encryptedCredential |Informations d’identification chiffrées par la passerelle. |Facultatif |

## <a name="dataset-properties"></a>Propriétés du jeu de données
Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md). Les sections comme la structure, la disponibilité et la stratégie d'un jeu de données JSON sont similaires pour tous les types de jeux de données (SQL Azure, Azure Blob, Azure Table, etc.).

La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations sur l’emplacement des données dans le magasin de données. La section typeProperties pour le jeu de données de type **MongoDbCollection** a les propriétés suivantes :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| collectionName |Nom de la collection dans la base de données MongoDB. |Oui |

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie
Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les tables d’entrée et de sortie et la stratégie sont disponibles pour tous les types d’activités.

En revanche, les propriétés disponibles dans la section **typeProperties** de l'activité varient pour chaque type d'activité. Pour l’activité de copie, elles dépendent des types de sources et récepteurs.

Lorsque la source est de type **MongoDbSource** , les propriétés suivantes sont disponibles dans la section typeProperties :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL-92. Par exemple : select * from MyTable. |Non (si **collectionName** du **jeu de données** est spécifié) |



## <a name="json-example-copy-data-from-mongodb-to-azure-blob"></a>Exemple JSON : copier des données de MongoDB vers Stockage Blob Azure
Cet exemple présente des exemples de définition JSON que vous pouvez utiliser pour créer un pipeline à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou d’[Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Il indique comment copier des données d’un magasin de données MongoDB local vers Stockage Blob Azure. Toutefois, les données peuvent être copiées vers l’un des récepteurs indiqués [ici](data-factory-data-movement-activities.md#supported-data-stores-and-formats) , via l’activité de copie d’Azure Data Factory.

L’exemple contient les entités de fabrique de données suivantes :

1. Un service lié de type [OnPremisesMongoDb](#linked-service-properties).
2. Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [MongoDbCollection](#dataset-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
5. Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [MongoDbSource](#copy-activity-properties) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

L’exemple copie toutes les heures les données de résultat d’une requête de base de données MongoDB vers un objet blob. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

Dans un premier temps, configurez la passerelle de gestion des données en suivant les instructions de l’article [Passerelle de gestion des données](data-factory-data-management-gateway.md) .

**Service lié MongoDB :**

```json
{
    "name": "OnPremisesMongoDbLinkedService",
    "properties":
    {
        "type": "OnPremisesMongoDb",
        "typeProperties":
        {
            "authenticationType": "<Basic or Anonymous>",
            "server": "< The IP address or host name of the MongoDB server >",
            "port": "<The number of the TCP port that the MongoDB server uses to listen for client connections.>",
            "username": "<username>",
            "password": "<password>",
            "authSource": "< The database that you want to use to check your credentials for authentication. >",
            "databaseName": "<database name>",
            "gatewayName": "<mygateway>"
        }
    }
}
```

**Service lié Azure Storage :**

```json
{
  "name": "AzureStorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

**Jeu de données d’entrée MongoDB :** La définition de « external » : « true » sur informe le service Data Factory qu’il s’agit d’un jeu de données qui est externe à la Data Factory et non produit par une activité dans la Data Factory.

```json
{
    "name": "MongoDbInputDataset",
    "properties": {
        "type": "MongoDbCollection",
        "linkedServiceName": "OnPremisesMongoDbLinkedService",
        "typeProperties": {
            "collectionName": "<Collection name>"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

**Jeu de données de sortie d'objet Blob Azure :**

Les données sont écrites dans un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1). Le chemin d’accès du dossier pour l’objet blob est évalué dynamiquement en fonction de l’heure de début du segment en cours de traitement. Le chemin d'accès du dossier utilise l'année, le mois, le jour et l'heure de l'heure de début.

```json
{
    "name": "AzureBlobOutputDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/frommongodb/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            },
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
            ]
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Activité de copie dans un pipeline avec une source MongoDB et un récepteur d’objets blob :**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d'entrée et de sortie ci-dessus, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **MongoDbSource** et le type **sink** est défini sur **BlobSink**. La requête SQL spécifiée pour la propriété **query** sélectionne les données de la dernière heure à copier.

```json
{
    "name": "CopyMongoDBToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "MongoDbSource",
                        "query": "$$Text.Format('select * from MyTable where LastModifiedDate >= {{ts\'{0:yyyy-MM-dd HH:mm:ss}\'}} AND LastModifiedDate < {{ts\'{1:yyyy-MM-dd HH:mm:ss}\'}}', WindowStart, WindowEnd)"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "MongoDbInputDataset"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutputDataSet"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "MongoDBToAzureBlob"
            }
        ],
        "start": "2016-06-01T18:00:00Z",
        "end": "2016-06-01T19:00:00Z"
    }
}
```


## <a name="schema-by-data-factory"></a>Schéma par Data Factory
Le service Azure Data Factory déduit le schéma à partir d’une collection MongoDB à l’aide des 100 derniers documents dans la collection. Si ces 100 documents ne contiennent pas de schéma complet, certaines colonnes peuvent être ignorées lors de l’opération de copie.

## <a name="type-mapping-for-mongodb"></a>Mappage de type pour MongoDB
Comme mentionné dans l’article consacré aux [activités de déplacement des données](data-factory-data-movement-activities.md) , l’activité de copie convertit automatiquement les types source en types récepteur à l’aide de l’approche en 2 étapes suivante :

1. Conversion de types natifs source en types .NET
2. Conversion à partir du type .NET en type de récepteur natif

Lors du déplacement de données vers MongoDB, les mappages suivants sont utilisés pour passer des types MongoDB aux types .NET.

| Type MongoDB | Type .NET Framework |
| --- | --- |
| Binary |Byte[] |
| Boolean |Boolean |
| Date |DateTime |
| NumberDouble |Double |
| NumberInt |Int32 |
| NumberLong |Int64 |
| ObjectID |String |
| String |String |
| UUID |Guid |
| Object |Renormalisé dans des colonnes aplaties avec « _ » comme séparateur imbriqué |

> [!NOTE]
> Pour en savoir plus sur la prise en charge des tableaux à l’aide de tables virtuelles, reportez-vous à la section [Prise en charge des types complexes à l’aide de tables virtuelles](#support-for-complex-types-using-virtual-tables) ci-dessous.

Les types de données MongoDB suivants ne sont pas pris en charge pour le moment : DBPointer, JavaScript, clé max./min., expression régulière, symbole, horodateur, non défini.

## <a name="support-for-complex-types-using-virtual-tables"></a>Prise en charge des types complexes à l’aide de tables virtuelles
Azure Data Factory utilise un pilote ODBC intégré pour assurer la connexion à votre base de données MongoDB et copier des données à partir de cette dernière. Pour les types complexes tels que des tableaux ou des objets avec des types différents entre les documents, le pilote normalise de nouveau les données dans les tables virtuelles correspondantes. En particulier, si une table contient de telles colonnes, le pilote génère les tables virtuelles suivantes :

* Une **table de base**, qui contient les mêmes données que la table réelle, à l’exception des colonnes de type complexe. La table de base utilise le même nom que la table réelle qu’elle représente.
* Une **table virtuelle** pour chaque colonne de type complexe, qui étend les données imbriquées. Le nom des tables virtuelles est composé du nom de la table réelle, d’un séparateur « _ » et du nom du tableau ou de l’objet.

Les tables virtuelles font référence aux données présentées dans la table réelle, de manière à permettre au pilote d’accéder aux données dénormalisées. Consultez la section Exemple ci-dessous pour plus d’informations. Vous pouvez accéder au contenu des tableaux MongoDB en interrogeant et en joignant les tables virtuelles.

Vous pouvez utiliser [l’Assistant de copie](data-factory-data-movement-activities.md#create-a-pipeline-with-copy-activity) afin d’afficher de manière intuitive la liste des tables dans la base de données MongoDB, y compris les tables virtuelles, et de prévisualiser les données qui s’y trouvent. Vous pouvez également construire une requête dans l’Assistant de copie et valider pour voir le résultat.

### <a name="example"></a>Exemple
Par exemple, « ExampleTable » ci-dessous est une table MongoDB qui dispose d’une colonne avec un tableau d’objets dans chaque cellule (Factures) et d’une colonne avec un tableau de types scalaires (Évaluations).

| _id | Nom du client | Factures | Niveau de service | Évaluations |
| --- | --- | --- | --- | --- |
| 1111 |ABC |[{invoice_id:”123”, item:”toaster”, price:”456”, discount:”0.2”}, {invoice_id:”124”, item:”oven”, price: ”1235”, discount: ”0.2”}] |Argent |[5,6] |
| 2222 |XYZ |[{invoice_id:”135”, item:”fridge”, price: ”12543”, discount: ”0.0”}] |Or |[1,2] |

Le pilote génère plusieurs tables virtuelles pour représenter cette table. La première table virtuelle est la table de base ci-dessous nommée « ExampleTable ». La table de base contient toutes les données de la table d’origine, mais les données dans les tableaux ont été omises et sont développées dans les tables virtuelles.

| _id | Nom du client | Niveau de service |
| --- | --- | --- |
| 1111 |ABC |Argent |
| 2222 |XYZ |Or |

Les tables suivantes montrent les tables virtuelles qui représentent les tableaux d’origine dans l’exemple. Ces tables contiennent les éléments suivants :

* Une référence à la colonne de clé primaire d’origine correspondant à la ligne du tableau d’origine (via la colonne _id)
* Une indication de la position des données dans le tableau d’origine
* Les données développées pour chaque élément du tableau

Table « ExampleTable_Invoices » :

| _id | ExampleTable_Invoices_dim1_idx | invoice_id | item | price | Remise |
| --- | --- | --- | --- | --- | --- |
| 1111 |0 |123 |grille-pain |456 |0.2 |
| 1111 |1 |124 |four |1235 |0.2 |
| 2222 |0 |135 |réfrigérateur |12543 |0.0 |

Table « ExampleTable_Ratings » :

| _id | ExampleTable_Ratings_dim1_idx | ExampleTable_Ratings |
| --- | --- | --- |
| 1111 |0 |5 |
| 1111 |1 |6 |
| 2222 |0 |1 |
| 2222 |1 |2 |

## <a name="map-source-to-sink-columns"></a>Mapper les colonnes source aux colonnes du récepteur
Pour en savoir plus sur le mappage de colonnes du jeu de données source à des colonnes du jeu de données récepteur, voir [Mappage des colonnes d’un jeu de données dans Azure Data Factory](data-factory-map-columns.md).

## <a name="repeatable-read-from-relational-sources"></a>Lecture renouvelée de sources relationnelles
Lorsque vous copiez des données à partir de magasins de données relationnels, gardez à l’esprit la répétabilité de l’opération, afin d’éviter des résultats imprévus. Dans Azure Data Factory, vous pouvez réexécuter une tranche manuellement. Vous pouvez également configurer une stratégie de nouvelles tentatives pour un jeu de données, afin qu’une tranche soit réexécutée en cas de défaillance. Lorsqu’une tranche est réexécutée d’une manière ou d’une autre, vous devez vous assurer que les mêmes données sont lues et ce, quel que soit le nombre d’exécutions de la tranche. Voir [Lecture renouvelée de sources relationnelles](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="performance-and-tuning"></a>Performances et réglage
Consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser.

## <a name="next-steps"></a>Étapes suivantes
Consultez l’article [Déplacement de données entre des sources locales et le cloud à l’aide de la passerelle de gestion des données](data-factory-move-data-between-onprem-and-cloud.md) pour obtenir des instructions détaillées sur la création d’un pipeline de données qui déplace les données à partir d’un magasin de données local vers un magasin de données Azure.
