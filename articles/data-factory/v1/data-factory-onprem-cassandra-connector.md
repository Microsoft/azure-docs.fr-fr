---
title: Déplacer des données à partir de Cassandra avec Data Factory
description: Découvrez comment déplacer des données depuis une base de données Cassandra locale à l’aide d’Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: 085cc312-42ca-4f43-aa35-535b35a102d5
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 06/07/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 0f96680f1ea91434c84d6606e3637c68c1cb5a84
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019631"
---
# <a name="move-data-from-an-on-premises-cassandra-database-using-azure-data-factory"></a>Déplacer des données depuis une base de données Cassandra locale à l’aide d’Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-onprem-cassandra-connector.md)
> * [Version 2 (version actuelle)](../connector-cassandra.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Connecteur Cassandra dans V2](../connector-cassandra.md).

Cet article explique comment utiliser l’activité de copie dans Azure Data Factory, afin de déplacer des données à partir d’une base de données Cassandra locale. Il s’appuie sur l’article [Activités de déplacement des données](data-factory-data-movement-activities.md), qui présente une vue d’ensemble du déplacement de données avec l’activité de copie.

Vous pouvez copier et coller les données d’un magasin de données Cassandra local vers tout magasin de données récepteur pris en charge. Consultez le tableau [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) pour obtenir la liste des magasins de données pris en charge en tant que récepteurs par l’activité de copie. Actuellement, Data Factory prend uniquement en charge le déplacement de données d’un magasin de données Cassandra vers d’autres magasins de données, mais non l’inverse.

## <a name="supported-versions"></a>Versions prises en charge
Le connecteur Cassandra prend en charge les versions suivantes de Cassandra : 2.x et 3.x. Pour une activité exécutée sur le runtime d’intégration auto-hébergé, Cassandra 3.x est pris en charge à partir d’Integration Runtime version 3.7 ou supérieure.

## <a name="prerequisites"></a>Prérequis
Pour que le service Azure Data Factory puisse se connecter à la base de données Cassandra locale, vous devez installer une passerelle de gestion de données sur l’ordinateur qui héberge la base de données ou sur un autre ordinateur, afin d’éviter toute mise en concurrence avec la base de données pour les ressources. La passerelle de gestion de données est un composant qui connecte des sources de données locales à des services cloud de manière gérée et sécurisée. Consultez l’article [Passerelle de gestion des données](data-factory-data-management-gateway.md) pour obtenir des informations détaillées sur la passerelle de gestion des données. Consultez l’article [Déplacement de données entre des sources locales et le cloud à l’aide de la passerelle de gestion des données](data-factory-move-data-between-onprem-and-cloud.md) pour obtenir des instructions détaillées sur la configuration de la passerelle, un pipeline de données, pour déplacer des données.

Vous devez utiliser la passerelle pour vous connecter à une base de données Cassandra, même si elle est hébergée dans le cloud, par exemple sur une machine virtuelle IaaS Azure. Vous pouvez installer la passerelle sur la même machine virtuelle que le magasin de données ou sur une autre machine virtuelle, tant que la passerelle peut se connecter à la base de données.

L’installation de la passerelle engendre automatiquement l’installation d’un pilote Microsoft ODBC Cassandra, utilisé pour se connecter à la base de données Cassandra. Par conséquent, vous n’avez pas besoin d’installer manuellement un pilote sur l’ordinateur de passerelle lors de la copie des données à partir de la base de données Cassandra.

> [!NOTE]
> Consultez [Résolution des problèmes de passerelle](data-factory-data-management-gateway.md#troubleshooting-gateway-issues) pour obtenir des conseils sur la résolution des problèmes de connexion/passerelle.

## <a name="getting-started"></a>Prise en main
Vous pouvez créer un pipeline avec une activité de copie qui déplace les données d’un magasin de données Cassandra local à l’aide de différents outils/API.

- Le moyen le plus simple de créer un pipeline consiste à utiliser **l’Assistant Copie**. Voir le [tutoriel : Créer un pipeline avec l’activité de copie à l’aide de l’Assistant Copie](data-factory-copy-data-wizard-tutorial.md) pour obtenir une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant Copie de données.
- Vous pouvez également utiliser les outils suivants pour créer un pipeline : **Visual Studio**, **Azure PowerShell**, **modèle Azure Resource Manager**, **.NET API** et **API REST**. Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [didacticiel sur l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Que vous utilisiez des outils ou des API, la création d’un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur implique les étapes suivantes :

1. Création de **services liés** pour lier les magasins de données d’entrée et de sortie à votre fabrique de données.
2. Création de **jeux de données** pour représenter les données d’entrée et de sortie de l’opération de copie.
3. Création d’un **pipeline** avec une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie.

Lorsque vous utilisez l’Assistant, les définitions JSON de ces entités Data Factory (services liés, jeux de données et pipeline) sont automatiquement créées pour vous. Lorsque vous utilisez des outils/API (à l’exception de l’API .NET), vous devez définir ces entités Data Factory au format JSON. Pour un exemple contenant des définitions JSON pour les entités Data Factory servant à copier des données à partir d’un magasin de données Cassandra local, consultez la section [Exemple JSON : copier des données de Cassandra vers Stockage Blob Azure](#json-example-copy-data-from-cassandra-to-azure-blob) de cet article.

Les sections suivantes contiennent des informations détaillées sur les propriétés JSON utilisées pour définir les entités Data Factory propres à un magasin de données Cassandra :

## <a name="linked-service-properties"></a>Propriétés du service lié
Le tableau suivant fournit la description des éléments JSON spécifiques au service lié Cassandra.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |La propriété type doit être définie sur : **OnPremisesCassandra** |Oui |
| host |Une ou plusieurs adresses IP ou noms d’hôte de serveurs Cassandra.<br/><br/>Renseignez une liste des adresses IP ou des noms d’hôte séparée par des virgules pour vous connecter simultanément à tous les serveurs. |Oui |
| port |Le port TCP utilisé par le serveur Cassandra pour écouter les connexions clientes. |Aucune valeur par défaut : 9042 |
| authenticationType |Basique ou anonyme |Oui |
| username |Spécifiez le nom d’utilisateur du compte d’utilisateur. |Oui, si authenticationType est défini sur De base. |
| mot de passe |Spécifiez le mot de passe du compte d'utilisateur. |Oui, si authenticationType est défini sur De base. |
| gatewayName |Le nom de la passerelle qui est utilisée pour se connecter à la base de données Cassandra locale. |Oui |
| encryptedCredential |Informations d’identification chiffrées par la passerelle. |Non |

>[!NOTE]
>Actuellement, la connexion à Cassandra à l’aide de TLS n’est pas prise en charge.

## <a name="dataset-properties"></a>Propriétés du jeu de données
Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md). Les sections comme la structure, la disponibilité et la stratégie d'un jeu de données JSON sont similaires pour tous les types de jeux de données (SQL Azure, Azure Blob, Azure Table, etc.).

La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations sur l’emplacement des données dans le magasin de données. La section typeProperties pour le jeu de données de type **CassandraTable** a les propriétés suivantes

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| espace de clé |Nom de l’espace de clé ou du schéma dans la base de données Cassandra. |Oui (si la **requête** pour **CassandraSource** n’est pas définie). |
| tableName |Nom de la table dans la base de données Cassandra. |Oui (si la **requête** pour **CassandraSource** n’est pas définie). |

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie
Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les tables d’entrée et de sortie et la stratégie sont disponibles pour tous les types d’activités.

En revanche, les propriétés disponibles dans la section typeProperties de l’activité varient pour chaque type d'activité. Pour l’activité de copie, elles dépendent des types de sources et récepteurs.

Lorsque la source est de type **CassandraSource**, les propriétés suivantes sont disponibles dans la section typeProperties :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Requête SQL-92 ou requête CQL. Reportez-vous à [référence CQL](https://docs.datastax.com/en/cql/3.1/cql/cql_reference/cqlReferenceTOC.html). <br/><br/>Lorsque vous utilisez la requête SQL, indiquez **keyspace name.table name** pour représenter la table que vous souhaitez interroger. |Non (si tableName et keyspace sur le jeu de données sont définis). |
| Niveau de cohérence |Le niveau de cohérence spécifie le nombre de réplicas devant répondre à une demande de lecture avant de renvoyer des données à l’application cliente. Cassandra vérifie le nombre de réplicas spécifié pour permettre aux données de répondre à la demande de lecture. |UN, DEUX, TROIS, QUORUM, TOUT, LOCAL_QUORUM, EACH_QUORUM, LOCAL_ONE. Reportez-vous à [Configuring data consistency (Configuration de la cohérence des données)](https://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_config_consistency_c.html) pour plus d’informations. |Non. La valeur par défaut est UN. |

## <a name="json-example-copy-data-from-cassandra-to-azure-blob"></a>Exemple JSON : copier des données de Cassandra vers Stockage Blob Azure
Cet exemple présente des exemples de définition JSON que vous pouvez utiliser pour créer un pipeline à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou d’[Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Il indique comment copier des données à partir d’une base de données Cassandra locale vers un système de Stockage Blob Azure. Toutefois, les données peuvent être copiées vers l’un des récepteurs indiqués [ici](data-factory-data-movement-activities.md#supported-data-stores-and-formats) , via l’activité de copie d’Azure Data Factory.

> [!IMPORTANT]
> Cet exemple fournit des extraits de code JSON. Il n’inclut pas d’instructions détaillées pour la création de la fabrique de données. Les instructions se trouvent dans l’article [Déplacement de données entre des emplacements locaux et le cloud](data-factory-move-data-between-onprem-and-cloud.md) .

L’exemple contient les entités de fabrique de données suivantes :

* Un service lié de type [OnPremisesCassandra](#linked-service-properties).
* Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [CassandraTable](#dataset-properties).
* Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [CassandraSource](#copy-activity-properties) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

**Service lié Cassandra :**

Cet exemple utilise le service lié **Cassandra** . Consultez la section [Service lié Cassandra](#linked-service-properties) pour visualiser les propriétés prises en charge par ce service lié.

```json
{
    "name": "CassandraLinkedService",
    "properties":
    {
        "type": "OnPremisesCassandra",
        "typeProperties":
        {
            "authenticationType": "Basic",
            "host": "mycassandraserver",
            "port": 9042,
            "username": "user",
            "password": "password",
            "gatewayName": "mygateway"
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

**Jeu de données d’entrée Cassandra :**

```json
{
    "name": "CassandraInput",
    "properties": {
        "linkedServiceName": "CassandraLinkedService",
        "type": "CassandraTable",
        "typeProperties": {
            "tableName": "mytable",
            "keySpace": "mykeyspace"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
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

La définition de **external** sur **true** informe le service Data Factory qu’il s’agit d’un jeu de données qui est externe à la Data Factory et non produit par une activité dans la Data Factory.

**Jeu de données de sortie d'objet Blob Azure :**

Les données sont écrites dans un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1).

```json
{
    "name": "AzureBlobOutput",
    "properties":
    {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties":
        {
            "folderPath": "adfgetstarted/fromcassandra"
        },
        "availability":
        {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Activité de copie dans un pipeline avec une source Cassandra et un récepteur blob :**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d'entrée et de sortie, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **CassandraSource** et le type **sink** est défini sur **BlobSink**.

Pour obtenir la liste des propriétés prises en charge par RelationalSource, consultez [propriétés du type RelationalSource](#copy-activity-properties) .

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2016-06-01T18:00:00",
        "end":"2016-06-01T19:00:00",
        "description":"pipeline with copy activity",
        "activities":[
        {
            "name": "CassandraToAzureBlob",
            "description": "Copy from Cassandra to an Azure blob",
            "type": "Copy",
            "inputs": [
            {
                "name": "CassandraInput"
            }
            ],
            "outputs": [
            {
                "name": "AzureBlobOutput"
            }
            ],
            "typeProperties": {
                "source": {
                    "type": "CassandraSource",
                    "query": "select id, firstname, lastname from mykeyspace.mytable"

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

### <a name="type-mapping-for-cassandra"></a>Mappage de type pour Cassandra
| Type Cassandra | Type basé sur .NET |
| --- | --- |
| ASCII |String |
| bigint |Int64 |
| BLOB |Byte[] |
| BOOLEAN |Boolean |
| DECIMAL |Decimal |
| DOUBLE |Double |
| FLOAT |Unique |
| INET |String |
| INT |Int32 |
| TEXT |String |
| timestamp |DateTime |
| TIMEUUID |Guid |
| UUID |Guid |
| VARCHAR |String |
| VARINT |Decimal |

> [!NOTE]
> Pour les types de collections (mappages, ensembles, listes, etc.), reportez-vous à la section [Work with Cassandra collection types using virtual table (Travailler avec les types de collections Cassandra à l’aide d’une table virtuelle)](#work-with-collections-using-virtual-table) .
>
> Les types définis par l’utilisateur ne sont pas pris en charge.
>
> La longueur des colonnes binaires et des colonnes de chaîne ne peut pas être supérieure à 4 000.
>
>

## <a name="work-with-collections-using-virtual-table"></a>Travailler avec des collections à l’aide d’une table virtuelle
Azure Data Factory utilise un pilote ODBC intégré pour assurer la connexion à votre base de données Cassandra et copier des données à partir de cette dernière. Pour les types de collection, notamment les cartes, ensembles et listes, le pilote renormalise les données dans des tables virtuelles correspondantes. En particulier, si une table contient des colonnes de n’importe quelle collection, le pilote génère les tables virtuelles suivantes :

* Une **table de base**, qui contient les mêmes données que la table réelle, à l’exception des colonnes de collection. La table de base utilise le même nom que la table réelle qu’elle représente.
* Une **table virtuelle** pour chaque colonne de collection, qui étend les données imbriquées. Le nom des tables virtuelles qui représentent des collections est composé du nom de la table réelle, du séparateur «*vt*» et du nom de la colonne.

Les tables virtuelles font référence aux données présentées dans la table réelle, de manière à permettre au pilote d’accéder aux données dénormalisées. Consultez la section Exemple pour plus d’informations. Vous pouvez accéder au contenu des collections Cassandra en interrogeant et en joignant les tables virtuelles.

Vous pouvez utiliser l’[Assistant de copie](data-factory-data-movement-activities.md#create-a-pipeline-with-copy-activity) afin d’afficher de manière intuitive la liste des tables dans la base de données Cassandra, y compris les tables virtuelles, et de prévisualiser les données qui s’y trouvent. Vous pouvez également construire une requête dans l’Assistant de copie et valider pour voir le résultat.

### <a name="example"></a>Exemple
Par exemple, « ExampleTable » ci-après est une table de base de données Cassandra qui contient une colonne clé primaire entière nommée « pk_int », une colonne de texte nommée « value », une colonne de liste, une colonne de mappage et une colonne de jeu (nommée « StringSet »).

| pk_int | Valeur | List | Mappage | StringSet |
| --- | --- | --- | --- | --- |
| 1 |« exemple de valeur 1 » |[« 1 », « 2 », « 3 »] |{« S1 » : « a », « S2 » : « b »} |{« A », « B », « C »} |
| 3 |« exemple de valeur 3 » |[« 100 », « 101 », « 102 », « 105 »] |{« S1 » : « t »} |{« A », « E »} |

Le pilote génère plusieurs tables virtuelles pour représenter cette table. Les colonnes de clés étrangères dans les tables virtuelles font référence aux colonnes de clés primaires dans la table réelle, et indiquent à quelles lignes de la table réelle les lignes de la table virtuelle correspondent.

La première table virtuelle est la table de base nommée « ExampleTable » affichée dans le tableau suivant. La table de base contient les mêmes données que la table de base de données d’origine, à l’exception des collections, qui sont omises de cette table et développées dans d’autres tables virtuelles.

| pk_int | Valeur |
| --- | --- |
| 1 |« exemple de valeur 1 » |
| 3 |« exemple de valeur 3 » |

Les tableaux suivants montrent les tables virtuelles qui renormalisent les données des colonnes Liste, Mappage et StringSet. Les colonnes portant des noms se terminant par « _index » ou « _key » indiquent la position des données dans la liste ou le mappage d’origine. Les colonnes portant des noms se terminant par « _value » contiennent les données étendues de la collection.

#### <a name="table-exampletable_vt_list"></a>Table « ExampleTable_vt_List » :
| pk_int | List_index | List_value |
| --- | --- | --- |
| 1 |0 |1 |
| 1 |1 |2 |
| 1 |2 |3 |
| 3 |0 |100 |
| 3 |1 |101 |
| 3 |2 |102 |
| 3 |3 |103 |

#### <a name="table-exampletable_vt_map"></a>Table « ExampleTable_vt_List » :
| pk_int | Map_key | Map_value |
| --- | --- | --- |
| 1 |S1 |Un |
| 1 |S2 |b |
| 3 |S1 |t |

#### <a name="table-exampletable_vt_stringset"></a>Table « ExampleTable_vt_List » :
| pk_int | StringSet_value |
| --- | --- |
| 1 |Un |
| 1 |B |
| 1 |C |
| 3 |Un |
| 3 |E |

## <a name="map-source-to-sink-columns"></a>Mapper les colonnes source aux colonnes du récepteur
Pour en savoir plus sur le mappage de colonnes du jeu de données source à des colonnes du jeu de données récepteur, voir [Mappage des colonnes d’un jeu de données dans Azure Data Factory](data-factory-map-columns.md).

## <a name="repeatable-read-from-relational-sources"></a>Lecture renouvelée de sources relationnelles
Lorsque vous copiez des données à partir de magasins de données relationnels, gardez à l’esprit la répétabilité de l’opération, afin d’éviter des résultats imprévus. Dans Azure Data Factory, vous pouvez réexécuter une tranche manuellement. Vous pouvez également configurer une stratégie de nouvelles tentatives pour un jeu de données, afin qu’une tranche soit réexécutée en cas de défaillance. Lorsqu’une tranche est réexécutée d’une manière ou d’une autre, vous devez vous assurer que les mêmes données sont lues et ce, quel que soit le nombre d’exécutions de la tranche. Voir [Lecture renouvelée de sources relationnelles](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="performance-and-tuning"></a>Performances et réglage
Consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser.
