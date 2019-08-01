---
title: Copier des données dans et à partir d’Azure SQL Data Warehouse à l’aide d’Azure Data Factory | Microsoft Docs
description: Découvrez comment utiliser Azure Data Factory pour copier des données de magasins de données sources pris en charge vers SQL Data Warehouse ou de SQL Data Warehouse vers des magasins de données récepteurs pris en charge.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 05/24/2019
ms.author: jingwang
ms.openlocfilehash: 24ee419e5c6eb4b8c148c61c232d2ab7ab07c74b
ms.sourcegitcommit: f56b267b11f23ac8f6284bb662b38c7a8336e99b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/28/2019
ms.locfileid: "67449596"
---
# <a name="copy-data-to-or-from-azure-sql-data-warehouse-by-using-azure-data-factory"></a>Copier des données depuis/vers Azure SQL Data Warehouse à l’aide d’Azure Data Factory 
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version1](v1/data-factory-azure-sql-data-warehouse-connector.md)
> * [Version actuelle](connector-azure-sql-data-warehouse.md)

Cet article explique comment copier des données vers et depuis Azure SQL Data Warehouse. Pour en savoir plus sur Azure Data Factory, lisez l’[article d’introduction](introduction.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur Blob Azure est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec tableau de [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Mappage de flux de données](concepts-data-flow-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)
- [Activité GetMetadata](control-flow-get-metadata-activity.md)

Plus précisément, ce connecteur Azure SQL Data Warehouse prend en charge les fonctions suivantes :

- Copie de données à l’aide de l’authentification SQL et de l’authentification du jeton de l’application Azure Active Directory (Azure AD) avec un principal de service ou l’identité managée pour les ressources Azure.
- En tant que source, récupération de données à l’aide d’une requête SQL ou d’une procédure stockée.
- En tant que récepteur, chargement des données à l’aide de PolyBase ou d’une insertion en bloc. Nous recommandons PolyBase pour améliorer les performances de copie.

> [!IMPORTANT]
> Si vous copiez des données à l’aide d’Azure Data Factory Integration Runtime, configurez un [pare-feu de serveur Azure SQL](https://msdn.microsoft.com/library/azure/ee621782.aspx#ConnectingFromAzure) de façon à ce que les services Azure puissent accéder au serveur.
> Si vous copiez des données à l’aide d’un runtime d’intégration auto-hébergé, configurez le pare-feu du serveur Azure SQL pour autoriser la plage IP appropriée. Cette plage inclut l’adresse IP de l’ordinateur qui est utilisé pour se connecter à Azure SQL Database.

## <a name="get-started"></a>Prise en main

> [!TIP]
> Pour obtenir les meilleures performances, utilisez PolyBase pour charger des données dans Azure SQL Data Warehouse. Pour plus de détails, consultez [Utiliser PolyBase pour charger des données dans Azure SQL Data Warehouse](#use-polybase-to-load-data-into-azure-sql-data-warehouse) . Consultez [Charger 1 To dans Azure SQL Data Warehouse en moins de 15 minutes avec Azure Data Factory](load-azure-sql-data-warehouse.md) pour obtenir une procédure pas à pas avec un cas d’utilisation.

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes fournissent des informations détaillées sur les propriétés qui définissent les entités Data Factory propres à un connecteur Azure SQL Data Warehouse.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié Azure SQL Data Warehouse sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété de type doit être définie sur **AzureSqlDW** | OUI |
| connectionString | Spécifiez les informations requises pour la connexion à l’instance Azure SQL Data Warehouse pour la propriété **connectionString**. <br/>Marquez ce champ comme SecureString pour le stocker de façon sécurisée dans Data Factory. Vous pouvez également stocker un mot de passe/une clé de principal de service dans Azure Key Vault et, en cas d'authentification SQL, extraire la configuration `password` de la chaîne de connexion. Pour plus d'informations, reportez-vous à l'exemple JSON décrit sous le tableau et à l'article [Stocker des informations d'identification dans Azure Key Vault](store-credentials-in-key-vault.md). | OUI |
| servicePrincipalId | Spécifiez l’ID client de l’application. | Oui, quand vous utilisez l’authentification Azure AD avec le principal de service. |
| servicePrincipalKey | Spécifiez la clé de l’application. Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui, quand vous utilisez l’authentification Azure AD avec le principal de service. |
| locataire | Spécifiez les informations de locataire (nom de domaine ou ID de locataire) dans lesquels se trouve votre application. Vous pouvez le récupérer en pointant la souris dans le coin supérieur droit du Portail Azure. | Oui, quand vous utilisez l’authentification Azure AD avec le principal de service. |
| connectVia | Le [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Vous pouvez utiliser Azure Integration Runtime ou un runtime d’intégration auto-hébergé (si votre magasin de données se trouve sur un réseau privé). À défaut de spécification, le runtime d’intégration Azure par défaut est utilisé. | Non |

Pour en savoir plus sur les autres types d’authentification, consultez les sections suivantes sur les prérequis et les exemples JSON, respectivement :

- [Authentification SQL](#sql-authentication)
- Authentification par jeton d’application Azure AD : [Principal du service](#service-principal-authentication)
- Authentification par jeton d’application Azure AD : [Identités managées pour les ressources Azure](#managed-identity)

>[!TIP]
>Si vous rencontrez une erreur avec le code d’erreur « UserErrorFailedToConnectToSqlServer » et un message tel que « La limite de session pour la base de données est XXX et a été atteinte. », ajoutez `Pooling=false` à votre chaîne de connexion, puis réessayez.

### <a name="sql-authentication"></a>Authentification SQL

#### <a name="linked-service-example-that-uses-sql-authentication"></a>Exemple de service lié qui utilise l’authentification SQL

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Mot de passe dans Azure Key Vault :**

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            },
            "password": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="service-principal-authentication"></a>Authentification d’un principal du service

Pour utiliser l’authentification du jeton d’application Azure AD basée sur le principal de service, effectuez les étapes suivantes :

1. **[Créez une application Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#create-an-azure-active-directory-application)** dans le portail Azure. Prenez note du nom de l’application et des valeurs suivantes qui définissent le service lié :

    - ID de l'application
    - Clé de l'application
    - ID client

2. **[Provisionnez un administrateur Azure Active Directory](../sql-database/sql-database-aad-authentication-configure.md#provision-an-azure-active-directory-administrator-for-your-azure-sql-database-server)** pour votre serveur Azure SQL sur le portail Azure, si ce n’est pas déjà fait. L’administrateur Azure AD peut être un utilisateur Azure AD ou un groupe Azure AD. Si vous accordez au groupe avec identité managée un rôle d’administrateur, ignorez les étapes 3 et 4. L’administrateur aura un accès complet à la base de données.

3. **[Créez des utilisateurs de base de données autonome](../sql-database/sql-database-aad-authentication-configure.md#create-contained-database-users-in-your-database-mapped-to-azure-ad-identities)** pour le principal de service. Connectez-vous à l’entrepôt de données vers lequel ou à partir duquel vous souhaitez copier des données à l’aide d’outils tels que SSMS, avec une identité Azure AD qui a au moins l’autorisation ALTER ANY USER. Exécutez le code T-SQL suivant :
  
    ```sql
    CREATE USER [your application name] FROM EXTERNAL PROVIDER;
    ```

4. **Accordez les autorisations requises par le principal de service** comme vous le feriez d’habitude pour des utilisateurs SQL ou autres. Exécutez le code suivant, ou consultez davantage d’options [ici](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql?view=sql-server-2017). Si vous souhaitez utiliser PolyBase pour charger les données, consultez l’[autorisation de base de données requise](#required-database-permission).

    ```sql
    EXEC sp_addrolemember db_owner, [your application name];
    ```

5. **Dans Azure Data Factory, configurez un service lié Azure SQL Data Warehouse.**


#### <a name="linked-service-example-that-uses-service-principal-authentication"></a>Exemple de service lié qui utilise l’authentification du principal de service

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;Connection Timeout=30"
            },
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="managed-identity"></a> Identités managées pour authentifier les ressources Azure

Une fabrique de données peut être associée à une [identité managée pour les ressources Azure](data-factory-service-identity.md), laquelle représente cette même fabrique. Vous pouvez utiliser cette identité managée pour l’authentification Azure SQL Data Warehouse. La fabrique désignée peut accéder et copier des données vers ou à partir de votre entrepôt de données à l’aide de cette identité.

Pour utiliser l’authentification par identité managée, effectuez les étapes suivantes :

1. **[Provisionnez un administrateur Azure Active Directory](../sql-database/sql-database-aad-authentication-configure.md#provision-an-azure-active-directory-administrator-for-your-azure-sql-database-server)** pour votre serveur Azure SQL sur le portail Azure, si ce n’est pas déjà fait. L’administrateur Azure AD peut être un utilisateur Azure AD ou un groupe Azure AD. Si vous accordez au groupe avec identité managée un rôle d’administrateur, ignorez les étapes 3 et 4. L’administrateur aura un accès complet à la base de données.

2. **[Créer des utilisateurs de base de données autonome](../sql-database/sql-database-aad-authentication-configure.md#create-contained-database-users-in-your-database-mapped-to-azure-ad-identities)** pour l’identité managée de Data Factory. Connectez-vous à l’entrepôt de données vers lequel ou à partir duquel vous souhaitez copier des données à l’aide d’outils tels que SSMS, avec une identité Azure AD qui a au moins l’autorisation ALTER ANY USER. Exécutez le code T-SQL suivant. 
  
    ```sql
    CREATE USER [your Data Factory name] FROM EXTERNAL PROVIDER;
    ```

3. **Accordez les autorisations requises par l’identité managée Data Factory** comme vous le feriez d’habitude pour des utilisateurs SQL et autres. Exécutez le code suivant, ou consultez davantage d’options [ici](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql?view=sql-server-2017). Si vous souhaitez utiliser PolyBase pour charger les données, consultez l’[autorisation de base de données requise](#required-database-permission).

    ```sql
    EXEC sp_addrolemember db_owner, [your Data Factory name];
    ```

5. **Dans Azure Data Factory, configurez un service lié Azure SQL Data Warehouse.**

**Exemple :**

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;Connection Timeout=30"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Jeux de données](concepts-datasets-linked-services.md). Cette section fournit la liste des propriétés prises en charge par le jeu de données Azure SQL Data Warehouse.

Pour copier des données depuis ou vers Azure SQL Data Warehouse, les propriétés suivantes sont prises en charge :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| Type | La propriété **type** du jeu de données doit être définie sur **AzureSqlDWTable**. | OUI |
| tableName | Nom de la table ou de la vue dans l’instance Azure SQL Data Warehouse à laquelle le service lié fait référence. | Non pour Source, Oui pour Récepteur |

#### <a name="dataset-properties-example"></a>Exemple de propriétés du jeu de données

```json
{
    "name": "AzureSQLDWDataset",
    "properties":
    {
        "type": "AzureSqlDWTable",
        "linkedServiceName": {
            "referenceName": "<Azure SQL Data Warehouse linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "typeProperties": {
            "tableName": "MyTable"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par Azure SQL Data Warehouse en tant que source et récepteur.

### <a name="azure-sql-data-warehouse-as-the-source"></a>Azure SQL Data Warehouse en tant que source

Pour copier des données d’Azure SQL Data Warehouse, affectez la valeur **SqlDWSource** à la propriété **type** dans l’activité de copie. Les propriétés prises en charge dans la section **source** de l’activité de copie sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| Type | La propriété **type** de la source d’activité de copie doit être définie sur **SqlDWSource**. | OUI |
| sqlReaderQuery | Utiliser la requête SQL personnalisée pour lire les données. Exemple : `select * from MyTable`. | Non |
| sqlReaderStoredProcedureName | Nom de la procédure stockée qui lit les données de la table source. La dernière instruction SQL doit être une instruction SELECT dans la procédure stockée. | Non |
| storedProcedureParameters | Paramètres de la procédure stockée.<br/>Les valeurs autorisées sont des paires de noms ou de valeurs. Les noms et la casse des paramètres doivent correspondre aux noms et à la casse des paramètres de la procédure stockée. | Non |

### <a name="points-to-note"></a>Points à noter

- Si **sqlReaderQuery** est spécifié pour **SqlSource**, l’activité de copie exécute cette requête sur la source Azure SQL Data Warehouse pour obtenir les données. Vous pouvez aussi spécifier une procédure stockée. Spécifiez **sqlReaderStoredProcedureName** et **storedProcedureParameters** si la procédure stockée accepte des paramètres.
- Si vous ne spécifiez pas **sqlReaderQuery** ou **sqlReaderStoredProcedureName**, les colonnes définies dans la section **structure** du JSON de jeu de données sont utilisées pour créer une requête. `select column1, column2 from mytable` est exécuté sur Azure SQL Data Warehouse. Si la définition du jeu de données n’a pas de **structure**, toutes les colonnes de la table sont sélectionnées.

#### <a name="sql-query-example"></a>Exemple de requête SQL

```json
"activities":[
    {
        "name": "CopyFromAzureSQLDW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure SQL DW input dataset name>",
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
                "type": "SqlDWSource",
                "sqlReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="stored-procedure-example"></a>Exemple de procédure stockée

```json
"activities":[
    {
        "name": "CopyFromAzureSQLDW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure SQL DW input dataset name>",
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
                "type": "SqlDWSource",
                "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
                "storedProcedureParameters": {
                    "stringData": { "value": "str3" },
                    "identifier": { "value": "$$Text.Format('{0:yyyy}', <datetime parameter>)", "type": "Int"}
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="stored-procedure-definition"></a>Définition de la procédure stockée

```sql
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

### <a name="azure-sql-data-warehouse-as-sink"></a> Azure SQL Data Warehouse en tant que récepteur

Pour copier des données vers Azure SQL Data Warehouse, définissez **SqlDWSink** comme type de récepteur dans l’activité de copie. Les propriétés suivantes sont prises en charge dans la section **sink** de l’activité de copie :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| Type | La propriété **type** du récepteur d’activité de copie doit être définie sur **SqlDWSink**. | OUI |
| allowPolyBase | Indique s’il faut utiliser PolyBase (le cas échéant) au lieu du mécanisme BULKINSERT. <br/><br/> Nous vous recommandons d’utiliser PolyBase pour charger des données dans SQL Data Warehouse. Pour connaître les contraintes et les détails, consultez la section [Utiliser PolyBase pour charger des données dans Azure SQL Data Warehouse](#use-polybase-to-load-data-into-azure-sql-data-warehouse).<br/><br/>Les valeurs autorisées sont **True** et **False** (par défaut).  | Non |
| polyBaseSettings | Groupe de propriétés pouvant être spécifié lorsque la propriété **allowPolybase** est définie sur **true**. | Non |
| rejectValue | Spécifie le nombre ou le pourcentage de lignes pouvant être rejetées avant l’échec de la requête.<br/><br/>Découvrez-en plus sur les options de rejet de PolyBase dans la section Arguments de [CREATE EXTERNAL TABLE (Transact-SQL)](https://msdn.microsoft.com/library/dn935021.aspx). <br/><br/>Les valeurs autorisées sont : 0 (par défaut), 1, 2, et ainsi de suite. |Non |
| rejectType | Indique si l’option **rejectValue** est une valeur littérale ou un pourcentage.<br/><br/>Les valeurs autorisées sont **Value** (par défaut) et **Percentage**. | Non |
| rejectSampleValue | Détermine le nombre de lignes à extraire avant que PolyBase recalcule le pourcentage de lignes rejetées.<br/><br/>Les valeurs autorisées sont 1, 2, et ainsi de suite. | Oui, si **rejectType** est **percentage** |
| useTypeDefault | Spécifie comment gérer les valeurs manquantes dans les fichiers texte délimités lorsque PolyBase extrait des données à partir du fichier texte.<br/><br/>Pour plus d’informations sur cette propriété, consultez la section Arguments dans [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](https://msdn.microsoft.com/library/dn935026.aspx).<br/><br/>Les valeurs autorisées sont **True** et **False** (par défaut).<br><br>**Consultez les [étapes de résolution des problèmes](#polybase-troubleshooting) relatives à ce paramètre.** | Non |
| writeBatchSize | Nombre de lignes à insérer dans le tableau SQL **par lot**. S’applique uniquement quand PolyBase n’est pas utilisé.<br/><br/>La valeur autorisée est **integer** (nombre de lignes). Par défaut, Data Factory détermine de façon dynamique la taille de lot appropriée selon la taille de ligne. | Non |
| writeBatchTimeout | Temps d’attente pour que l’opération d’insertion de lot soit terminée avant d’expirer. S’applique uniquement quand PolyBase n’est pas utilisé.<br/><br/>La valeur autorisée est **timespan**. Exemple : “00:30:00” (30 minutes). | Non |
| preCopyScript | Spécifiez une requête SQL pour l’activité de copie à exécuter avant l’écriture de données dans Azure SQL Data Warehouse à chaque exécution. Utilisez cette propriété pour nettoyer les données préchargées. | Non |

#### <a name="sql-data-warehouse-sink-example"></a>Exemple de récepteur SQL Data Warehouse

```json
"sink": {
    "type": "SqlDWSink",
    "allowPolyBase": true,
    "polyBaseSettings":
    {
        "rejectType": "percentage",
        "rejectValue": 10.0,
        "rejectSampleValue": 100,
        "useTypeDefault": true
    }
}
```

Apprenez-en davantage sur la façon d’utiliser PolyBase pour charger efficacement dans SQL Data Warehouse en lisant la section suivante.

## <a name="use-polybase-to-load-data-into-azure-sql-data-warehouse"></a>Utiliser PolyBase pour charger des données dans Azure SQL Data Warehouse

L’utilisation de [PolyBase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide) est un moyen efficace de charger de grandes quantités de données dans Azure SQL Data Warehouse avec un débit élevé. Vous profiterez d’un gain important de débit en utilisant PolyBase au lieu du mécanisme BULKINSERT par défaut. Consultez [Référence sur les performances](copy-activity-performance.md#performance-reference) pour obtenir une comparaison détaillée. Pour obtenir une procédure pas à pas avec un cas d’utilisation, consultez [Charger 1 To dans Azure SQL Data Warehouse](v1/data-factory-load-sql-data-warehouse.md).

* Si votre source de données se trouve dans **Blob Azure, Azure Data Lake Storage Gen1 ou Azure Data Lake Storage Gen2**et que le **format est compatible à PolyBase**, vous pouvez utiliser l’activité de copie pour appeler directement PolyBase et permettre à Azure SQL Data Warehouse d’extraire les données à partir de la source. Pour plus d’informations, consultez **[Copie directe à l’aide de PolyBase](#direct-copy-by-using-polybase)** .
* Si votre magasin de données source et son format ne sont pas pris en charge à l’origine par PolyBase, utilisez plutôt la fonctionnalité **[Copie intermédiaire avec PolyBase](#staged-copy-by-using-polybase)** . La fonctionnalité de copie intermédiaire offre également un meilleur débit. Elle convertit automatiquement les données dans un format compatible avec PolyBase. Et elle stocke les données dans Stockage Blob Azure. Elle charge ensuite les données dans SQL Data Warehouse.

>[!TIP]
>Apprenez-en plus sur les [Bonnes pratiques d’utilisation de PolyBase](#best-practices-for-using-polybase).

### <a name="direct-copy-by-using-polybase"></a>Copie directe à l’aide de PolyBase

SQL Data Warehouse PolyBase prend directement en charge les objets Blob Azure, Azure Data Lake Storage Gen1 et Azure Data Lake Storage Gen2. Si vos données sources répondent aux critères décrits dans cette section, utilisez PolyBase pour copier directement à partir du magasin de données source vers Azure SQL Data Warehouse. Sinon, utilisez la méthode [Copie intermédiaire à l’aide de PolyBase](#staged-copy-by-using-polybase).

> [!TIP]
> Pour copier des données vers SQL Data Warehouse efficacement, consultez [Azure Data Factory makes it even easier and convenient to uncover insights from data when using Data Lake Store with SQL Data Warehouse](https://blogs.msdn.microsoft.com/azuredatalake/2017/04/08/azure-data-factory-makes-it-even-easier-and-convenient-to-uncover-insights-from-data-when-using-data-lake-store-with-sql-data-warehouse/) (Azure Data Factory facilite et rend plus pratique la découverte d’informations à partir de données lors de l’utilisation de Data Lake Store avec SQL Data Warehouse).

Si les critères ne sont pas remplis, Azure Data Factory contrôle les paramètres et rétablit automatiquement le mécanisme BULKINSERT pour le déplacement des données.

1. Le **service lié source** contient les types et méthodes d’authentification suivants :

    | Type de magasin de données source pris en charge | Type d’authentification source pris en charge |
    |:--- |:--- |
    | [Blob Azure](connector-azure-blob-storage.md) | Authentification par clé de compte, l’authentification d’une identité gérée |
    | [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md) | Authentification d’un principal du service |
    | [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md) | Authentification par clé de compte, l’authentification d’une identité gérée |

    >[!IMPORTANT]
    >Si votre stockage Azure est configuré avec le point de terminaison de service réseau virtuel, vous devez utiliser l’authentification d’identité gérée. Consultez [Impact de l’utilisation des points de terminaison de service de réseau virtuel avec le stockage Azure](../sql-database/sql-database-vnet-service-endpoint-rule-overview.md#impact-of-using-vnet-service-endpoints-with-azure-storage). Découvrez les configurations requises dans Data Factory dans les sections [Blob Azure - authentification de l’identité gérée](connector-azure-blob-storage.md#managed-identity) et [Azure Data Lake Storage Gen2 - authentification de l’identité gérée](connector-azure-data-lake-storage.md#managed-identity).

2. Le **format de données source** est **Parquet**, **ORC**, ou **Texte délimité**, avec les configurations suivantes :

   1. Le chemin d’accès au dossier ne contient pas de filtre de caractères génériques.
   2. Le nom de fichier pointe vers un fichier unique ou est `*` ou `*.*`.
   3. `rowDelimiter` doit être **\n**.
   4. `nullValue` est soit défini sur **une chaîne vide** («») ou conserve sa valeur par défaut, et `treatEmptyAsNull` conserve également sa valeur par défaut ou n’est pas définie sur false.
   5. `encodingName` est **utf-8**, qui est la valeur par défaut.
   6. `quoteChar`, `escapeChar` et `skipLineCount` ne sont pas spécifiés. PolyBase est capable d’ignorer la ligne d’en-tête ; cela peut être paramétré en tant que `firstRowAsHeader` dans ADF.
   7. `compression` peut être **aucune compression**, **GZip** ou **Deflate**.

```json
"activities":[
    {
        "name": "CopyFromAzureBlobToSQLDataWarehouseViaPolyBase",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "BlobDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "BlobSource",
            },
            "sink": {
                "type": "SqlDWSink",
                "allowPolyBase": true
            }
        }
    }
]
```

### <a name="staged-copy-by-using-polybase"></a>Copie intermédiaire à l’aide de PolyBase

Quand vos données sources ne remplissent pas les critères présentés dans la section précédente, activez la copie des données par le biais d’une instance de stockage Blob Azure intermédiaire. Il ne doit pas s’agir du Stockage Premium Azure. Dans ce cas, Azure Data Factory exécute automatiquement des transformations sur les données pour répondre aux exigences de format de données de PolyBase. Ensuite, il utilise PolyBase pour charger des données dans SQL Data Warehouse. Pour finir, il nettoie vos données temporaires du stockage Blob. Pour plus d’informations sur la copie des données par le biais d’une instance de Stockage Blob Azure, consultez [Copie intermédiaire](copy-activity-performance.md#staged-copy).

Pour utiliser cette fonctionnalité, créez un [service lié au Stockage Blob Azure](connector-azure-blob-storage.md#linked-service-properties) qui fait référence au compte de stockage Azure avec le stockage Blob intermédiaire. Spécifiez ensuite les propriétés `enableStaging` et `stagingSettings` de l’activité de copie comme indiqué dans le code suivant :

>[!IMPORTANT]
>Si votre stockage Azure est configuré avec le point de terminaison de service réseau virtuel, vous devez utiliser l’authentification d’identité gérée. Consultez [Impact de l’utilisation des points de terminaison de service de réseau virtuel avec le stockage Azure](../sql-database/sql-database-vnet-service-endpoint-rule-overview.md#impact-of-using-vnet-service-endpoints-with-azure-storage). Découvrez les configurations requises dans Data Factory à partir de [Blob Azure - authentification de l’identité gérée](connector-azure-blob-storage.md#managed-identity).

```json
"activities":[
    {
        "name": "CopyFromSQLServerToSQLDataWarehouseViaPolyBase",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "SQLServerDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlSource",
            },
            "sink": {
                "type": "SqlDWSink",
                "allowPolyBase": true
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingBlob",
                    "type": "LinkedServiceReference"
                }
            }
        }
    }
]
```

## <a name="best-practices-for-using-polybase"></a>Bonnes pratiques d’utilisation de PolyBase

Les sections suivantes contiennent des bonnes pratiques qui s’ajoutent à celles mentionnées dans [Bonnes pratiques pour Azure SQL Data Warehouse](../sql-data-warehouse/sql-data-warehouse-best-practices.md).

### <a name="required-database-permission"></a>Autorisation de base de données requise

Pour utiliser PolyBase, l’utilisateur qui charge des données dans SQL Data Warehouse doit avoir [l’autorisation « CONTROL »](https://msdn.microsoft.com/library/ms191291.aspx) sur la base de données cible. Vous pouvez pour cela l’ajouter en tant que membre du rôle **db_owner**. Découvrez comment procéder dans la [Présentation de SQL Data Warehouse](../sql-data-warehouse/sql-data-warehouse-overview-manage-security.md#authorization).

### <a name="row-size-and-data-type-limits"></a>Limitations en matière de taille de ligne et de type de données

Les chargements PolyBase sont limités aux lignes inférieures à 1 Mo. Vous ne pouvez pas l’utiliser pour charger VARCHR(MAX), NVARCHAR(MAX) ou VARBINARY(MAX). Pour en savoir plus, consultez [Limites de la capacité de SQL Data Warehouse](../sql-data-warehouse/sql-data-warehouse-service-capacity-limits.md#loads).

Quand votre source de données comporte des lignes supérieures à 1 Mo, vous pouvez fractionner verticalement les tables sources en plusieurs petites tables. Vérifiez que la taille maximale de chaque ligne ne dépasse pas la limite. Vous pouvez ensuite charger les tables plus petites à l’aide de PolyBase et les fusionner dans Azure SQL Data Warehouse.

Autrement, pour les données avec des colonnes aussi larges, vous pouvez utiliser non-PolyBase pour charger les données à l’aide d’ADF, en désactivant le paramètre « Autoriser PolyBase ».

### <a name="polybase-troubleshooting"></a>Résolution des problèmes liés à PolyBase

**Chargement de la colonne décimale**

Si votre source de données est au format texte ou dans d’autres magasins non compatibles avec PolyBase (avec copie intermédiaire et PolyBase), et qu’elle contient une valeur vide à charger dans une colonne décimale de SQL Data Warehouse, il se peut que vous rencontriez l’erreur suivante :

```
ErrorCode=FailedDbOperation, ......HadoopSqlException: Error converting data type VARCHAR to DECIMAL.....Detailed Message=Empty string can't be converted to DECIMAL.....
```

La solution consiste à désélectionner l’option « **Utiliser l’option de type par défaut** » (false) dans Récepteur d’activité de copie -> Paramètres de PolyBase. « [USE_TYPE_DEFAULT](https://docs.microsoft.com/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest#arguments
) » est une configuration native PolyBase qui spécifie comment gérer les valeurs manquantes dans les fichiers texte délimités lorsque PolyBase extrait des données à partir du fichier texte. 

**Autres**

Pour en apprendre plus sur les problèmes connus relatifs à PolyBase, consultez [Résolution des problèmes liés à la charge Polybase pour Azure SQL Data Warehouse](../sql-data-warehouse/sql-data-warehouse-troubleshoot.md#polybase).

### <a name="sql-data-warehouse-resource-class"></a>Classe de ressources SQL Data Warehouse

Pour obtenir le meilleur débit possible, attribuez une classe de ressources plus volumineuse à l’utilisateur qui charge des données dans SQL Data Warehouse par le biais de PolyBase.

### <a name="tablename-in-azure-sql-data-warehouse"></a>**tableName** dans Azure SQL Data Warehouse

Le tableau suivant donne des exemples montrant comment spécifier la propriété **tableName** dans le jeu de données JSON. Il montre plusieurs combinaisons de noms de schéma et de table.

| Schéma BD | Nom de la table | Propriété JSON **tableName** |
| --- | --- | --- |
| dbo | MyTable | MyTable ou dbo.MyTable ou [dbo].[MyTable] |
| dbo1 | MyTable | dbo1.MyTable ou [dbo1].[MyTable] |
| dbo | My.Table | [My.Table] ou [dbo].[My.Table] |
| dbo1 | My.Table | [dbo1].[My.Table] |

Si vous voyez l’erreur suivante, il peut s’agir d’un problème avec la valeur spécifiée pour la propriété **tableName**. Consultez le tableau précédent pour savoir comment spécifier des valeurs pour la propriété JSON **tableName**.

```
Type=System.Data.SqlClient.SqlException,Message=Invalid object name 'stg.Account_test'.,Source=.Net SqlClient Data Provider
```

### <a name="columns-with-default-values"></a>Colonnes avec des valeurs par défaut

Actuellement, la fonctionnalité PolyBase dans Data Factory accepte seulement le même nombre de colonnes que dans la table cible. Par exemple, vous avez une table avec quatre colonnes dont l’une est définie avec une valeur par défaut. Les données d’entrée doivent toujours avoir quatre colonnes. Un jeu de données d’entrée de trois colonnes produit une erreur semblable au message suivant :

```
All columns of the table must be specified in the INSERT BULK statement.
```

La valeur NULL est une forme spéciale de la valeur par défaut. Si la colonne est nullable, les données d’entrée dans l’objet Blob pour cette colonne peuvent être vides. En revanche, elles ne peuvent pas être manquantes dans le jeu de données d’entrée. PolyBase insère NULL pour les données manquantes dans Azure SQL Data Warehouse.

## <a name="mapping-data-flow-properties"></a>Propriétés du mappage de flux de données

Découvrez plus de détails sur la [transformation de la source](data-flow-source.md) et la [transformation de réception](data-flow-sink.md) dans Mappage de flux de données.

## <a name="data-type-mapping-for-azure-sql-data-warehouse"></a>Mappage de type de données pour Azure SQL Data Warehouse

Quand vous copiez des données vers ou à partir d’Azure SQL Data Warehouse, les mappages suivants sont utilisés entre les types de données Azure SQL Data Warehouse et les types de données intermédiaires d’Azure Data Factory. Pour découvrir comment l’activité de copie mappe le schéma et le type de données sources au récepteur, consultez [Mappages de schémas et de types de données](copy-activity-schema-and-type-mapping.md).

>[!TIP]
>Reportez-vous à l’article [Types de tables de données dans Azure SQL Data Warehouse](../sql-data-warehouse/sql-data-warehouse-tables-data-types.md) sur les types de données pris en charge par SQL Data Warehouse et leurs solutions de contournement.

| Type de données Azure SQL Data Warehouse | Type de données intermédiaires d’Azure Data Factory |
|:--- |:--- |
| bigint | Int64 |
| binary | Byte[] |
| bit | Boolean |
| char | String, Char[] |
| date | Datetime |
| Datetime | Datetime |
| datetime2 | Datetime |
| Datetimeoffset | DateTimeOffset |
| Decimal | Decimal |
| FILESTREAM attribute (varbinary(max)) | Byte[] |
| Float | Double |
| image | Byte[] |
| int | Int32 |
| money | Decimal |
| nchar | String, Char[] |
| numeric | Decimal |
| nvarchar | String, Char[] |
| real | Single |
| rowversion | Byte[] |
| smalldatetime | Datetime |
| smallint | Int16 |
| smallmoney | Decimal |
| time | TimeSpan |
| tinyint | Byte |
| uniqueidentifier | Guid |
| varbinary | Byte[] |
| varchar | String, Char[] |

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez le tableau [Magasins de données pris en charge](copy-activity-overview.md##supported-data-stores-and-formats).
