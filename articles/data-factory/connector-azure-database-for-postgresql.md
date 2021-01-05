---
title: Copier des données depuis et vers Azure Database pour PostgreSQL
description: Découvrez comment copier des données vers et à partir d’Azure Database pour PostgreSQL à l’aide d’une activité de copie dans un pipeline Azure Data Factory.
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 11/26/2020
ms.openlocfilehash: 11e0d3336f085ccae9a7fb83ed050d69a15ce42b
ms.sourcegitcommit: 192f9233ba42e3cdda2794f4307e6620adba3ff2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96296503"
---
# <a name="copy-data-to-and-from-azure-database-for-postgresql-by-using-azure-data-factory"></a>Copier des données vers et depuis Azure Database pour PostgreSQL avec Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article explique comment utiliser la fonctionnalité d'activité de copie dans Azure Data Factory pour copier des données d’Azure Database pour PostgreSQL. Il s’appuie sur l’article [Activité de copie dans Azure Data Factory](copy-activity-overview.md), qui constitue une présentation de l’activité de copie.

Ce connecteur est spécialisé pour le [service Azure Database pour PostgreSQL](../postgresql/overview.md). Pour copier des données à partir d’une base de données PostgreSQL générique locale ou située dans le cloud, utilisez le [connecteur PostgreSQL](connector-postgresql.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur Azure Database pour PostgreSQL est pris en charge pour les activités suivantes :

- [Activité de copie](copy-activity-overview.md) avec [prise en charge de la matrice source/récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)

Vous pouvez copier des données d’Azure Database pour PostgreSQL vers n’importe quel magasin de données récepteur pris en charge. Vous pouvez également copier des données à partir de tout magasin de données source pris en charge vers Azure Database pour PostgreSQL. Pour obtenir la liste des magasins de données prises en charge par l'activité de copie en tant que sources et récepteurs, consultez la table [Magasins de données pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Azure Data Factory fournit un pilote intégré pour permettre la connectivité. Vous n'êtes donc pas tenu d'installer manuellement un pilote pour utiliser ce connecteur.

## <a name="getting-started"></a>Prise en main

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes détaillent les propriétés utilisées pour définir des entités Data Factory propres au connecteur Azure Database pour PostgreSQL.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés suivantes sont prises en charge pour le service lié Azure Database pour PostgreSQL :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur : **AzurePostgreSql**. | Oui |
| connectionString | Chaîne de connexion ODBC permettant de se connecter à Azure Database pour PostgreSQL.<br/>Vous pouvez également définir un mot de passe dans Azure Key Vault et extraire la configuration `password` de la chaîne de connexion. Pour plus d’informations, consultez les exemples suivants et [Stocker des informations d’identification dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui |
| connectVia | Cette propriété correspond au [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour la connexion à la banque de données. Vous pouvez utiliser runtime d’intégration Azure ou un runtime d’intégration auto-hébergé (si votre banque de données se trouve dans un réseau privé). À défaut de spécification, le runtime d’intégration Azure par défaut est utilisé. |Non |

Voici un exemple de chaîne de connexion typique : `Server=<server>.postgres.database.azure.com;Database=<database>;Port=<port>;UID=<username>;Password=<Password>`. Selon votre cas de figure, vous pouvez définir d’autres propriétés :

| Propriété | Description | Options | Obligatoire |
|:--- |:--- |:--- |:--- |
| EncryptionMethod (EM)| La méthode utilisée par le pilote pour chiffrer les données envoyées entre le pilote et le serveur de base de données. Par exemple, `EncryptionMethod=<0/1/6>;`| 0 (aucun chiffrement) **(par défaut)** / 1 (SSL) / 6 (RequestSSL) | Non |
| ValidateServerCertificate (VSC) | Détermine si le pilote valide le certificat envoyé par le serveur de base de données lorsque le chiffrement SSL est activé (EncryptionMethod = 1). Par exemple, `ValidateServerCertificate=<0/1>;`| 0 (désactivé) **(par défaut)** / 1 (activé) | Non |

**Exemple** :

```json
{
    "name": "AzurePostgreSqlLinkedService",
    "properties": {
        "type": "AzurePostgreSql",
        "typeProperties": {
            "connectionString": "Server=<server>.postgres.database.azure.com;Database=<database>;Port=<port>;UID=<username>;Password=<Password>"
        }
    }
}
```

**Exemple** :

**_Stockage du mot de passe dans Azure Key Vault_* _

```json
{
    "name": "AzurePostgreSqlLinkedService",
    "properties": {
        "type": "AzurePostgreSql",
        "typeProperties": {
            "connectionString": "Server=<server>.postgres.database.azure.com;Database=<database>;Port=<port>;UID=<username>;",
            "password": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName>" 
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez [Jeux de données dans Azure Data Factory](concepts-datasets-linked-services.md). Cette section fournit la liste des propriétés prises en charge par le jeu de données Azure Database pour PostgreSQL.

Pour copier des données d’Azure Database pour PostgreSQL, affectez la valeur _*AzurePostgreSqlTable** à la propriété type du jeu de données. Les propriétés prises en charge sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type du jeu de données doit être définie sur **AzurePostgreSqlTable** | Oui |
| tableName | Nom de la table | Non (si « query » dans la source de l’activité est spécifié) |

**Exemple** :

```json
{
    "name": "AzurePostgreSqlDataset",
    "properties": {
        "type": "AzurePostgreSqlTable",
        "linkedServiceName": {
            "referenceName": "<AzurePostgreSql linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez [Pipelines et activités dans Azure Data Factory](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par une source Azure Database pour PostgreSQL.

### <a name="azure-database-for-postgresql-as-source"></a>Azure Database pour PostgreSQL en tant que source

Pour copier des données d’Azure Database pour PostgreSQL, affectez la valeur **AzurePostgreSqlSource** au type source de l’activité de copie. Les propriétés prises en charge dans la section **source** de l’activité de copie sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type de la source de l’activité de copie doit être définie sur **AzurePostgreSqlSource** | Oui |
| query | Utiliser la requête SQL personnalisée pour lire les données. Par exemple, `SELECT * FROM mytable` ou `SELECT * FROM "MyTable"`. Notez que dans PostgreSQL, le nom de l’entité est traité comme insensible à la casse s’il n’est pas placé entre guillemets. | Non (si la propriété tableName du jeu de données est spécifiée) |

**Exemple** :

```json
"activities":[
    {
        "name": "CopyFromAzurePostgreSql",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<AzurePostgreSql input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "AzurePostgreSqlSource",
                "query": "<custom query e.g. SELECT * FROM mytable>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="azure-database-for-postgresql-as-sink"></a>Azure Database pour PostgreSQL en tant que récepteur

Pour copier des données vers Azure Database pour PostgreSQL, les propriétés suivantes sont prises en charge dans la section **sink** de l’activité de copie :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type du récepteur d’activité de copie doit être définie sur **AzurePostgreSQLSink**. | Oui |
| preCopyScript | Spécifiez une requête SQL pour l’activité de copie à exécuter avant d'écrire des données dans Azure Database pour PostgreSQL à chaque exécution. Vous pouvez utiliser cette propriété pour nettoyer des données préchargées. | Non |
| writeBatchSize | Insère des données dans la table Azure Database pour PostgreSQL quand la taille de la mémoire tampon atteint writeBatchSize.<br>La valeur autorisée est un entier qui représente le nombre de lignes. | Non (valeur par défaut : 10 000) |
| writeBatchTimeout | Temps d’attente pour que l’opération d’insertion de lot soit terminée avant d’expirer.<br>Les valeurs autorisées sont des intervalles de temps. Exemple : 00:30:00 (30 minutes). | Non (valeur par défaut : 00:00:30) |

**Exemple** :

```json
"activities":[
    {
        "name": "CopyToAzureDatabaseForPostgreSQL",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Azure PostgreSQL output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzurePostgreSQLSink",
                "preCopyScript": "<custom SQL script>",
                "writeBatchSize": 100000
            }
        }
    }
]
```

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour plus d’informations sur les propriétés, consultez [Activité Lookup dans Azure Data Factory](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs par l'activité de copie d'Azure Data Factory, consultez le tableau [Magasins de données pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).
