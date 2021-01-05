---
title: Déplacer des données à partir de magasins de données ODBC
description: Apprenez à transférer des données à partir de magasins de données ODBC à l’aide d’Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: ad70a598-c031-4339-a883-c6125403cb76
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 11/19/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 3c68b1f4d76a1899ce473c57f3a6d5de1eab71c6
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92636865"
---
# <a name="move-data-from-odbc-data-stores-using-azure-data-factory"></a>Transfert de données à partir de magasins de données ODBC à l’aide d’Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-odbc-connector.md)
> * [Version 2 (version actuelle)](../connector-odbc.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Connecteur ODBC dans V2](../connector-odbc.md).


Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour déplacer des données à partir d’un magasin de données ODBC local. Il s’appuie sur l’article [Activités de déplacement des données](data-factory-data-movement-activities.md), qui présente une vue d’ensemble du déplacement de données avec l’activité de copie.

Vous pouvez copier et coller les données d’un magasin de données ODBC dans tout magasin de données récepteur pris en charge. Consultez le tableau [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) pour obtenir la liste des magasins de données pris en charge en tant que récepteurs par l’activité de copie. Actuellement, Data Factory prend uniquement en charge le déplacement de données d’un magasin de données ODBC vers d’autres magasins de données, mais non l’inverse.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="enabling-connectivity"></a>Activation de la connectivité
Le service Data Factory prend en charge la connexion à des sources ODBC locales à l’aide de la passerelle de gestion des données. Consultez l’article [Déplacement de données entre des emplacements locaux et le cloud](data-factory-move-data-between-onprem-and-cloud.md) pour en savoir plus sur la passerelle de gestion des données et obtenir des instructions détaillées sur la configuration de la passerelle. Utilisez la passerelle pour vous connecter à un magasin de données ODBC même si elle est hébergée sur une machine virtuelle IaaS Azure.

Vous pouvez installer la passerelle sur le même ordinateur local ou la machine virtuelle Azure comme magasin de données ODBC. Toutefois, nous vous recommandons d’installer la passerelle sur un ordinateur/une machine virtuelle IaaS Azure distinct(e) afin d’éviter les conflits de ressources, ainsi que pour obtenir de meilleures performances. Lorsque vous installez la passerelle sur un ordinateur distinct, l’ordinateur doit être en mesure d’accéder à l’ordinateur sur lequel réside le magasin de données ODBC.

En dehors de la passerelle de gestion des données, vous devez également installer le pilote ODBC pour le magasin de données de l’ordinateur de passerelle.

> [!NOTE]
> Consultez [Résolution des problèmes de passerelle](data-factory-data-management-gateway.md#troubleshooting-gateway-issues) pour obtenir des conseils sur la résolution des problèmes de connexion/passerelle.

## <a name="getting-started"></a>Prise en main
Vous pouvez créer un pipeline avec une activité de copie qui déplace les données d’un magasin de données ODBC local à l’aide de différents outils/API.

Le moyen le plus simple de créer un pipeline consiste à utiliser **l’Assistant Copie** . Voir le [tutoriel : Créer un pipeline avec l’activité de copie à l’aide de l’Assistant Copie](data-factory-copy-data-wizard-tutorial.md) pour obtenir une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant Copie de données.

Vous pouvez également utiliser les outils suivants pour créer un pipeline : **Visual Studio** , **Azure PowerShell** , **modèle Azure Resource Manager** , **.NET API** et **API REST** . Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [didacticiel sur l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Que vous utilisiez des outils ou des API, la création d’un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur implique les étapes suivantes :

1. Création de **services liés** pour lier les magasins de données d’entrée et de sortie à votre fabrique de données.
2. Création de **jeux de données** pour représenter les données d’entrée et de sortie de l’opération de copie.
3. Création d’un **pipeline** avec une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie.

Lorsque vous utilisez l’Assistant, les définitions JSON de ces entités Data Factory (services liés, jeux de données et pipeline) sont automatiquement créées pour vous. Lorsque vous utilisez des outils/API (à l’exception de l’API .NET), vous devez définir ces entités Data Factory au format JSON. Pour un exemple contenant des définitions JSON pour les entités Data Factory servant à copier des données à partir d’un magasin de données ODBC, consultez la section [Exemple JSON : copier des données d’un magasin de données ODBC vers Stockage Blob Azure](#json-example-copy-data-from-odbc-data-store-to-azure-blob) de cet article.

Les sections suivantes contiennent des informations détaillées sur les propriétés JSON utilisées pour définir les entités Data Factory propres au magasin de données ODBC :

## <a name="linked-service-properties"></a>Propriétés du service lié
Le tableau suivant fournit la description des éléments JSON spécifiques au service lié ODBC.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |La propriété type doit être définie sur : **OnPremisesOdbc** |Oui |
| connectionString |Partie de la chaîne de connexion ne contenant pas les informations d’accès, avec des informations d’identification chiffrées facultatives. Consultez les exemples dans les sections suivantes. <br/><br/>Vous pouvez spécifier la chaîne de connexion avec un modèle comme `"Driver={SQL Server};Server=Server.database.windows.net; Database=TestDatabase;"`, ou utiliser le système DSN (Data Source Name) que vous avez configuré sur l’ordinateur passerelle avec `"DSN=<name of the DSN>;"` (vous devez toujours spécifier la partie informations d’identification dans le service lié comme il convient). |Oui |
| credential |Partie de la chaîne de connexion contenant les informations d’accès, spécifiée dans un format de valeurs de propriété spécifique au pilote. Exemple : `"Uid=<user ID>;Pwd=<password>;RefreshToken=<secret refresh token>;"`. |Non |
| authenticationType |Type d’authentification utilisé pour se connecter au magasin de données ODBC. Les valeurs possibles sont les suivantes : Anonymous et Basic. |Oui |
| userName |Spécifiez le nom d’utilisateur si vous utilisez l’authentification de base. |Non |
| mot de passe |Indiquez le mot de passe du compte d’utilisateur défini pour userName. |Non |
| gatewayName |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter au magasin de données ODBC. |Oui |

### <a name="using-basic-authentication"></a>Utilisation de l’authentification de base

```json
{
    "name": "odbc",
    "properties":
    {
        "type": "OnPremisesOdbc",
        "typeProperties":
        {
            "authenticationType": "Basic",
            "connectionString": "Driver={SQL Server};Server=Server.database.windows.net; Database=TestDatabase;",
            "userName": "username",
            "password": "password",
            "gatewayName": "mygateway"
        }
    }
}
```
### <a name="using-basic-authentication-with-encrypted-credentials"></a>Utilisation de l’authentification de base avec des informations d’identification chiffrées
Vous pouvez chiffrer les informations d’identification à l’aide de l’applet de commande [New-AzDataFactoryEncryptValue](/powershell/module/az.datafactory/new-azdatafactoryencryptvalue) (version 1.0 d’Azure PowerShell) ou [New-AzureDataFactoryEncryptValue](/previous-versions/azure/dn834940(v=azure.100)) (version 0.9 ou antérieure d’Azure PowerShell).

```json
{
    "name": "odbc",
    "properties":
    {
        "type": "OnPremisesOdbc",
        "typeProperties":
        {
            "authenticationType": "Basic",
            "connectionString": "Driver={SQL Server};Server=myserver.database.windows.net; Database=TestDatabase;;EncryptedCredential=eyJDb25uZWN0...........................",
            "gatewayName": "mygateway"
        }
    }
}
```

### <a name="using-anonymous-authentication"></a>Utilisation de l’authentification anonyme

```json
{
    "name": "odbc",
    "properties":
    {
        "type": "OnPremisesOdbc",
        "typeProperties":
        {
            "authenticationType": "Anonymous",
            "connectionString": "Driver={SQL Server};Server={servername}.database.windows.net; Database=TestDatabase;",
            "credential": "UID={uid};PWD={pwd}",
            "gatewayName": "mygateway"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données
Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md). Les sections comme la structure, la disponibilité et la stratégie d'un jeu de données JSON sont similaires pour tous les types de jeux de données (SQL Azure, Azure Blob, Azure Table, etc.).

La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations sur l’emplacement des données dans le magasin de données. La section typeProperties du jeu de données de type **RelationalTable** (qui inclut le jeu de données ODBC) présente les propriétés suivantes

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table dans le magasin de données ODBC. |Oui |

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie
Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les tables d'entrée et de sortie et les différentes stratégies sont disponibles pour tous les types d'activités.

En revanche, les propriétés disponibles dans la section **typeProperties** de l'activité varient pour chaque type d'activité. Pour l’activité de copie, elles dépendent des types de sources et récepteurs.

Dans l’activité de copie, quand la source est de type **RelationalSource** (ce qui inclut ODBC), les propriétés suivantes sont disponibles dans la section typeProperties :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Par exemple : select * from MyTable. |Oui |


## <a name="json-example-copy-data-from-odbc-data-store-to-azure-blob"></a>Exemple JSON : copier des données d’un magasin de données ODBC vers Stockage Blob Azure
Cet exemple présente des définitions JSON que vous pouvez utiliser pour créer un pipeline à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou d’[Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Il indique comment copier des données depuis une source ODBC vers un système de stockage Blob Azure. Toutefois, les données peuvent être copiées vers l’un des récepteurs indiqués [ici](data-factory-data-movement-activities.md#supported-data-stores-and-formats) , via l’activité de copie d’Azure Data Factory.

L’exemple contient les entités de fabrique de données suivantes :

1. Un service lié de type [OnPremisesOdbc](#linked-service-properties).
2. Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [RelationalTable](#dataset-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
5. Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [RelationalSource](#copy-activity-properties) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

L’exemple copie toutes les heures les données de résultat d’une requête d’un magasin de données ODBC en local vers un objet blob. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

Dans un premier temps, configurez la passerelle de gestion des données. Les instructions se trouvent dans l’article [Déplacement de données entre des emplacements locaux et le cloud](data-factory-move-data-between-onprem-and-cloud.md) .

**Service lié de HDFS** : cet exemple utilise l’authentification de base. Consultez la section [Service lié ODBC](#linked-service-properties) pour connaître les différents types d’authentification que vous pouvez utiliser.

```json
{
    "name": "OnPremOdbcLinkedService",
    "properties":
    {
        "type": "OnPremisesOdbc",
        "typeProperties":
        {
            "authenticationType": "Basic",
            "connectionString": "Driver={SQL Server};Server=Server.database.windows.net; Database=TestDatabase;",
            "userName": "username",
            "password": "password",
            "gatewayName": "mygateway"
        }
    }
}
```

**Service lié Azure Storage**

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

**Jeu de données d’entrée ODBC**

L’exemple suppose que vous avez créé une table « MyTable » dans une base de données ODBC et qu’elle contient une colonne appelée « timestampcolumn » pour les données de série chronologique.

La définition de « external » : « true» informe le service Data Factory qu’il s’agit d’un jeu de données qui est externe à Data Factory et non produit par une activité dans Data Factory.

```json
{
    "name": "ODBCDataSet",
    "properties": {
        "published": false,
        "type": "RelationalTable",
        "linkedServiceName": "OnPremOdbcLinkedService",
        "typeProperties": {},
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

**Jeu de données de sortie d'objet Blob Azure**

Les données sont écrites dans un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1). Le chemin d’accès du dossier pour l’objet blob est évalué dynamiquement en fonction de l’heure de début du segment en cours de traitement. Le chemin d'accès du dossier utilise l'année, le mois, le jour et l'heure de l'heure de début.

```json
{
    "name": "AzureBlobOdbcDataSet",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/odbc/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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

**Activité de copie dans un pipeline, avec une source ODBC (RelationalSource) et un récepteur blob (BlobSink)**

Le pipeline contient une activité de copie qui est configurée pour utiliser ces jeux de données d'entrée et de sortie, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **RelationalSource** et le type **sink** est défini sur **BlobSink** . La requête SQL spécifiée pour la propriété **query** sélectionne les données de la dernière heure à copier.

```json
{
    "name": "CopyODBCToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "RelationalSource",
                        "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "OdbcDataSet"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOdbcDataSet"
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
                "name": "OdbcToBlob"
            }
        ],
        "start": "2016-06-01T18:00:00Z",
        "end": "2016-06-01T19:00:00Z"
    }
}
```
### <a name="type-mapping-for-odbc"></a>Mappage de type pour ODBC
Comme mentionné dans l’article consacré aux [activités de déplacement de données](data-factory-data-movement-activities.md) , l’activité de copie convertit automatiquement des types source en types récepteur à l’aide de l’approche en deux étapes suivante :

1. Conversion de types natifs source en types .NET
2. Conversion à partir du type .NET en type de récepteur natif

Lors du déplacement de données à partir de magasins de données ODBC, les types de données ODBC sont mappés aux types .NET, comme indiqué dans la rubrique [Mappages de types de données ODBC](/dotnet/framework/data/adonet/odbc-data-type-mappings) .

## <a name="map-source-to-sink-columns"></a>Mapper les colonnes source aux colonnes du récepteur
Pour en savoir plus sur le mappage de colonnes du jeu de données source à des colonnes du jeu de données récepteur, voir [Mappage des colonnes d’un jeu de données dans Azure Data Factory](data-factory-map-columns.md).

## <a name="repeatable-read-from-relational-sources"></a>Lecture renouvelée de sources relationnelles
Lorsque vous copiez des données à partir de magasins de données relationnels, gardez à l’esprit la répétabilité de l’opération, afin d’éviter des résultats imprévus. Dans Azure Data Factory, vous pouvez réexécuter une tranche manuellement. Vous pouvez également configurer une stratégie de nouvelles tentatives pour un jeu de données, afin qu’une tranche soit réexécutée en cas de défaillance. Lorsqu’une tranche est réexécutée d’une manière ou d’une autre, vous devez vous assurer que les mêmes données sont lues et ce, quel que soit le nombre d’exécutions de la tranche. Voir [Lecture renouvelée de sources relationnelles](data-factory-repeatable-copy.md#repeatable-read-from-relational-sources).

## <a name="troubleshoot-connectivity-issues"></a>Résoudre les problèmes de connectivité
Pour résoudre les problèmes de connexion, utilisez l’onglet **Diagnostics** du **Gestionnaire de configuration de la passerelle de gestion des données** .

1. Lancez le **Gestionnaire de configuration de la passerelle de gestion des données** . Vous pouvez exécuter directement « C:\Program Files\Microsoft Data Management Gateway\1.0\Shared\ConfigManager.exe » (ou) rechercher **Passerelle** pour trouver un lien vers l’application **Passerelle de gestion des données de Microsoft** , comme l’illustre l’image suivante.

    ![Rechercher la passerelle](./media/data-factory-odbc-connector/search-gateway.png)
2. Basculez vers l’onglet **Diagnostics** .

    ![Diagnostics de la passerelle](./media/data-factory-odbc-connector/data-factory-gateway-diagnostics.png)
3. Sélectionnez le **type** de magasin de données (service lié).
4. Spécifiez **l’authentification** et entrez les **informations d’identification** (ou) entrez la **chaîne de connexion** utilisée pour vous connecter au magasin de données.
5. Cliquez sur **Tester la connexion** pour tester la connexion au magasin de données.

## <a name="performance-and-tuning"></a>Performances et réglage
Consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser.