---
title: Copier et transformer des données dans Azure Data Lake Storage Gen2
description: Découvrez comment copier des données vers et depuis Azure Data Lake Storage Gen2 et y transformer des données avec Azure Data Factory.
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 10/28/2020
ms.openlocfilehash: c0f7df8db79f549dfeca15e6411ae72cbe66d2bd
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97346280"
---
# <a name="copy-and-transform-data-in-azure-data-lake-storage-gen2-using-azure-data-factory"></a>Copier et transformer des données dans Data Lake Storage Gen2 avec Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Azure Data Lake Storage Gen2 (ADLS Gen2) est un ensemble de fonctionnalités dédiées à l'analytique du Big Data et intégrées au service [Stockage Blob Azure](../storage/blobs/storage-blobs-introduction.md). Il vous permet d’interagir avec vos données selon les deux paradigmes que sont le système de fichiers et le stockage d’objets.

Cet article indique comment utiliser l’activité de copie dans Azure Data Factory pour copier des données depuis et vers Azure Data Lake Storage Gen2 et utiliser Data Flow pour transformer les données dans Azure Data Lake Storage Gen2. Pour en savoir plus sur Azure Data Factory, lisez l’[article d’introduction](introduction.md).

>[!TIP]
>Pour plus d’informations sur le scénario de migration de lac de données ou d’entrepôt de données, consultez [Migration des données dans Azure à partir d’un lac de données ou d’un entrepôt de données avec Azure Data Factory](data-migration-guidance-overview.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur Azure Data Lake Storage Gen2 est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Mappage de flux de données](concepts-data-flow-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)
- [Activité GetMetadata](control-flow-get-metadata-activity.md)
- [Supprimer l’activité](delete-activity.md)

Pour l’activité de copie, avec ce connecteur, vous pouvez effectuer les opérations suivantes :

- Copier des données depuis/vers Azure Data Lake Storage Gen2 avec une clé de compte, un principal de service ou des identités managées pour les authentifications de ressources Azure.
- Copier des fichiers tels quels, ou analyser ou générer des fichiers avec les [formats de fichier et codecs de compression pris en charge](supported-file-formats-and-compression-codecs.md).
- [La conservation des métadonnées de fichier lors de la copie](#preserve-metadata-during-copy).
- [Conserver les listes de contrôle d’accès](#preserve-acls) lors de la copie à partir d’Azure Data Lake Storage Gen1/Gen2.

## <a name="get-started"></a>Bien démarrer

>[!TIP]
>Pour une procédure pas à pas d’utilisation du connecteur Data Lake Storage Gen2, consultez [Charger des données dans Azure Data Lake Storage Gen2](load-azure-data-lake-storage-gen2.md).

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes fournissent des informations sur les propriétés utilisées pour définir les entités Data Factory spécifiques de Data Lake Storage Gen2.

## <a name="linked-service-properties"></a>Propriétés du service lié

Le connecteur Azure Data Lake Storage Gen2 prend en charge les types d’authentification suivants. Consultez les sections correspondantes pour plus d’informations :

- [Authentification par clé de compte](#account-key-authentication)
- [Authentification d’un principal du service](#service-principal-authentication)
- [Identités managées pour authentifier les ressources Azure](#managed-identity)

>[!NOTE]
>- Si vous utilisez le runtime d’intégration Azure public pour vous connecter à Data Lake Storage Gen2 avec l’option **Autoriser les services Microsoft approuvés à accéder à ce compte de stockage** activée sur le pare-feu Stockage Azure, vous devez recourir à [l’authentification par identité managée](#managed-identity).
>- Si vous utilisez PolyBase ou l’instruction COPY pour charger des données dans Azure Synapse Analytics et que votre Data Lake Storage Gen2 source ou de préproduction est configuré avec un point de terminaison de réseau virtuel Azure, vous devez utiliser l’authentification par identité managée comme l’exige Synapse. Pour en savoir plus sur la configuration requise, voir la section sur l’[authentification par identité managée](#managed-identity).

### <a name="account-key-authentication"></a>Authentification par clé de compte

Pour l’authentification par clé de compte de stockage, les propriétés suivantes sont prises en charge :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur **AzureBlobFS**. |Oui |
| url | Point de terminaison pour Data Lake Storage Gen2 avec le modèle de `https://<accountname>.dfs.core.windows.net`. | Oui |
| accountKey | Clé de compte pour Data Lake Storage Gen2. Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). |Oui |
| connectVia | Le [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Vous pouvez utiliser le runtime d'intégration Azure ou un runtime d’intégration auto-hébergé si votre banque de données se trouve sur un réseau privé. Si cette propriété n’est pas spécifiée, le runtime d’intégration Azure par défaut est utilisé. |Non |

>[!NOTE]
>Le point de terminaison du système de fichiers ADLS secondaire n’est pas pris en charge lors de l’utilisation de l’authentification de clé du compte. Vous pouvez utiliser d’autres types d’authentification.

**Exemple :**

```json
{
    "name": "AzureDataLakeStorageGen2LinkedService",
    "properties": {
        "type": "AzureBlobFS",
        "typeProperties": {
            "url": "https://<accountname>.dfs.core.windows.net", 
            "accountkey": { 
                "type": "SecureString", 
                "value": "<accountkey>" 
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

Pour l’authentification de principal de service, effectuez les étapes suivantes.

1. Inscrivez une entité d’application dans Azure Active Directory (Azure AD) en suivant les instructions de la section [Inscrire votre application à un locataire Azure AD](../storage/common/storage-auth-aad-app.md#register-your-application-with-an-azure-ad-tenant). Prenez note des valeurs suivantes, qui vous permettent de définir le service lié :

    - ID de l'application
    - Clé de l'application
    - ID client

2. Accordez l’autorisation nécessaire au principal de service. Pour obtenir des exemples sur le fonctionnement des autorisations dans Data Lake Storage Gen2, consultez [Listes de contrôle d’accès sur les fichiers et répertoires](../storage/blobs/data-lake-storage-access-control.md#access-control-lists-on-files-and-directories).

    - **En tant que source** : Dans l’Explorateur Stockage, accordez au moins l’autorisation **Exécution** à l’ensemble des dossiers en amont et au système de fichiers et l’autorisation **Lecture** pour les fichiers à copier. Dans le contrôle d’accès (IAM), vous pouvez également accorder au moins un rôle **Lecteur des données Blob du stockage** .
    - **En tant que récepteur** : Dans l’Explorateur Stockage, accordez au moins l’autorisation **Exécution** à l’ensemble des dossiers en amont et au système de fichiers et l’autorisation **Écriture** pour le dossier récepteur. Dans le contrôle d’accès (IAM), vous pouvez également accorder au moins un rôle **Contributeur aux données Blob du stockage**.

>[!NOTE]
>Si vous utilisez l’interface utilisateur de Data Factory pour la création et que le principal de service n’est pas défini avec le rôle « Lecteur des données Blob du stockage/Contributeur aux données Blob du stockage » dans IAM, quand vous effectuez une connexion de test ou accédez à des dossiers, choisissez « Tester la connexion au chemin du fichier » ou « Parcourir à partir du chemin spécifié », puis spécifiez un chemin avec l’autorisation **Lecture + Exécution** pour continuer.

Ces propriétés sont prises en charge pour le service lié :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur **AzureBlobFS**. |Oui |
| url | Point de terminaison pour Data Lake Storage Gen2 avec le modèle de `https://<accountname>.dfs.core.windows.net`. | Oui |
| servicePrincipalId | Spécifiez l’ID client de l’application. | Oui |
| servicePrincipalCredentialType | Type d’informations d’identification à utiliser pour l’authentification de principal du service. Les valeurs autorisées sont **ServicePrincipalKey** et **ServicePrincipalCert**. | Oui |
| servicePrincipalCredential | Informations d’identification du principal du service. <br/> Quand vous utilisez **ServicePrincipalKey** comme type d’informations d’identification, spécifiez la clé de l’application. Marquez ce champ en tant que **SecureString** afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). <br/> Quand vous utilisez **ServicePrincipalCert** comme informations d’identification, référencez un certificat dans Azure Key Vault. | Oui |
| servicePrincipalKey | Spécifiez la clé de l’application. Marquez ce champ en tant que **SecureString** afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). <br/> Cette propriété est toujours prise en charge telle quelle pour `servicePrincipalId` + `servicePrincipalKey`. Comme ADF ajoute une nouvelle authentification par certificat du principal du service, le nouveau modèle pour l’authentification de principal du service est `servicePrincipalId` + `servicePrincipalCredentialType` + `servicePrincipalCredential`. | Non |
| tenant | Spécifiez les informations de locataire (nom de domaine ou ID de locataire) dans lesquels se trouve votre application. Récupérez-les en pointant la souris dans le coin supérieur droit du Portail Azure. | Oui |
| azureCloudType | Pour l’authentification du principal du service, spécifiez le type d’environnement cloud Azure auprès duquel votre application Azure Active Directory est inscrite. <br/> Les valeurs autorisées sont **AzurePublic**, **AzureChina**, **AzureUsGovernment** et **AzureGermany**. Par défaut, l’environnement cloud de la fabrique de données est utilisé. | Non |
| connectVia | Le [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Vous pouvez utiliser le runtime d'intégration Azure ou un runtime d’intégration auto-hébergé si votre banque de données se trouve sur un réseau privé. À défaut de spécification, l’Azure Integration Runtime par défaut est utilisé. |Non |

**Exemple : utilisation de l’authentification de la clé du principal de service**

Vous pouvez également stocker la clé du principal du service dans Azure Key Vault.

```json
{
    "name": "AzureDataLakeStorageGen2LinkedService",
    "properties": {
        "type": "AzureBlobFS",
        "typeProperties": {
            "url": "https://<accountname>.dfs.core.windows.net", 
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalCredentialType": "ServicePrincipalKey",
            "servicePrincipalCredential": {
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

**Exemple : utilisation de l’authentification par certificat du principal du service**
```json
{
    "name": "AzureDataLakeStorageGen2LinkedService",
    "properties": {
        "type": "AzureBlobFS",
        "typeProperties": {
            "url": "https://<accountname>.dfs.core.windows.net", 
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalCredentialType": "ServicePrincipalCert",
            "servicePrincipalCredential": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<AKV reference>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<certificate name in AKV>" 
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

### <a name="managed-identities-for-azure-resources-authentication"></a><a name="managed-identity"></a> Identités managées pour authentifier les ressources Azure

Une fabrique de données peut être associée à une [identité managée pour les ressources Azure](data-factory-service-identity.md), laquelle représente cette même fabrique de données. Vous pouvez utiliser directement cette identité managée pour l’authentification Azure Data Lake Storage Gen2, ce qui revient à utiliser votre propre principal de service. Cela permet à la fabrique désignée d’accéder aux données et de les copier depuis ou vers votre Data Lake Storage Gen2.

Pour utiliser les identités managées afin d’authentifier les ressources Azure, procédez comme suit.

1. [Récupérez les informations d’identité managée de Data Factory](data-factory-service-identity.md#retrieve-managed-identity) en copiant la valeur de l’**ID d’objet d’identité managée** générée en même temps que votre fabrique.

2. Accordez l’autorisation nécessaire à l’identité managée. Pour obtenir des exemples sur le fonctionnement des autorisations dans Data Lake Storage Gen2, consultez [Listes de contrôle d’accès sur les fichiers et répertoires](../storage/blobs/data-lake-storage-access-control.md#access-control-lists-on-files-and-directories).

    - **En tant que source** : Dans l’Explorateur Stockage, accordez au moins l’autorisation **Exécution** à l’ensemble des dossiers en amont et au système de fichiers et l’autorisation **Lecture** pour les fichiers à copier. Dans le contrôle d’accès (IAM), vous pouvez également accorder au moins un rôle **Lecteur des données Blob du stockage** .
    - **En tant que récepteur** : Dans l’Explorateur Stockage, accordez au moins l’autorisation **Exécution** à l’ensemble des dossiers en amont et au système de fichiers et l’autorisation **Écriture** pour le dossier récepteur. Dans le contrôle d’accès (IAM), vous pouvez également accorder au moins un rôle **Contributeur aux données Blob du stockage**.

>[!NOTE]
>Si vous utilisez l’interface utilisateur de Data Factory pour la création et que l’identité managée n’est pas définie avec le rôle « Lecteur des données Blob du stockage/Contributeur aux données Blob du stockage » dans IAM, quand vous effectuez une connexion de test ou accédez à des dossiers, choisissez « Tester la connexion au chemin du fichier » ou « Parcourir à partir du chemin spécifié », puis spécifiez un chemin avec l’autorisation **Lecture + Exécution** pour continuer.

>[!IMPORTANT]
>Si vous utilisez PolyBase ou l’instruction COPY pour charger des données de Data Lake Storage Gen2 dans Azure Synapse Analytics et que vous avez recours à l’authentification par identité managée pour Data Lake Storage Gen2, veillez également à suivre les étapes 1 à 3 de [cette aide](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage). Ces étapes inscrivent votre serveur auprès d’Azure AD et attribuent le rôle de contributeur aux données de l’objet blob de stockage. Data Factory gère le reste. Si vous configurez le Stockage Blob avec un point de terminaison de réseau virtuel Azure, vous devez également activer **Autoriser les services Microsoft approuvés à accéder à ce compte de stockage** dans le menu des paramètres **Pare-feu et réseaux virtuels** du compte Stockage Azure, comme l’exige Synapse.

Ces propriétés sont prises en charge pour le service lié :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur **AzureBlobFS**. |Oui |
| url | Point de terminaison pour Data Lake Storage Gen2 avec le modèle de `https://<accountname>.dfs.core.windows.net`. | Oui |
| connectVia | Le [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Vous pouvez utiliser le runtime d'intégration Azure ou un runtime d’intégration auto-hébergé si votre banque de données se trouve sur un réseau privé. À défaut de spécification, l’Azure Integration Runtime par défaut est utilisé. |Non |

**Exemple :**

```json
{
    "name": "AzureDataLakeStorageGen2LinkedService",
    "properties": {
        "type": "AzureBlobFS",
        "typeProperties": {
            "url": "https://<accountname>.dfs.core.windows.net", 
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article sur les [jeux de données](concepts-datasets-linked-services.md).

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Les propriétés suivantes sont prises en charge pour Data Lake Storage Gen2 dans les paramètres `location` du jeu de données basé sur le format :

| Propriété   | Description                                                  | Obligatoire |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | La propriété de type sous `location` dans le jeu de données doit être définie sur **AzureBlobFSLocation**. | Oui      |
| fileSystem | Nom du système de fichiers Data Lake Storage Gen2.                              | Non       |
| folderPath | Chemin d’accès à un dossier sous le système de fichiers donné. Si vous souhaitez utiliser un caractère générique pour filtrer les dossiers, ignorez ce paramètre et spécifiez-le dans les paramètres de la source de l’activité. | Non       |
| fileName   | Nom de fichier dans le chemin d’accès fileSystem +folderPath donné. Si vous souhaitez utiliser un caractère générique pour filtrer les fichiers, ignorez ce paramètre et spécifiez-le dans les paramètres de la source de l’activité. | Non       |

**Exemple :**

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<Data Lake Storage Gen2 linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureBlobFSLocation",
                "fileSystem": "filesystemname",
                "folderPath": "folder/subfolder"
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez les articles [Configurations des activités de copie](copy-activity-overview.md#configuration) et [Pipelines et activités](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par la source et le récepteur Data Lake Storage Gen2.

### <a name="azure-data-lake-storage-gen2-as-a-source-type"></a>Azure Data Lake Storage Gen2 comme type de source

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Vous avez plusieurs options pour copier des données à partir de ADLS Gen2 :

- Copier à partir du chemin d’accès spécifié dans le jeu de données.
- Filtre de caractères génériques sur le chemin d’accès du dossier ou le nom du fichier, consultez `wildcardFolderPath` et `wildcardFileName`.
- Copier les fichiers définis dans un fichier texte donné en tant que jeu de fichiers, consultez `fileListPath`.

Les propriétés suivantes sont prises en charge pour Data Lake Storage Gen2 dans les paramètres `storeSettings` de la source de copie basée sur le format :

| Propriété                 | Description                                                  | Obligatoire                                      |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| type                     | La propriété type sous `storeSettings` doit être définie sur **AzureBlobFSReadSettings**. | Oui                                           |
| **_Rechercher les fichiers à copier :_* _ |  |  |
| OPTION 1 : chemin d’accès statique<br> | Copie à partir du système de fichiers donné ou du chemin d’accès au dossier/fichier spécifié dans le jeu de données. Si vous souhaitez copier tous les fichiers d’un système de fichiers/dossier, spécifiez en plus `wildcardFileName` comme `_`. |  |
| OPTION 2 : caractère générique<br>- wildcardFolderPath | Chemin d’accès du dossier avec des caractères génériques sous le système de fichiers configuré dans le jeu de données pour filtrer les dossiers sources. <br>Les caractères génériques autorisés sont : `*` (correspond à zéro ou plusieurs caractères) et `?` (correspond à zéro ou un caractère) ; utilisez `^` en guise d’échappement si votre nom de dossier contient effectivement ce caractère d’échappement ou générique. <br>Consultez d’autres exemples dans les [exemples de filtre de dossier et de fichier](#folder-and-file-filter-examples). | Non                                            |
| OPTION 2 : caractère générique<br>- wildcardFileName | Nom du fichier avec des caractères génériques situé dans le chemin d’accès système de fichiers + folderPath/wildcardFolderPath donné pour filtrer les fichiers sources. <br>Les caractères génériques autorisés sont : `*` (correspond à zéro ou plusieurs caractères) et `?` (correspond à zéro ou un caractère) ; utilisez `^` en guise d’échappement si votre nom de fichier contient effectivement ce caractère d’échappement ou générique.  Consultez d’autres exemples dans les [exemples de filtre de dossier et de fichier](#folder-and-file-filter-examples). | Oui |
| OPTION 3 : liste de fichiers<br>- fileListPath | Indique de copier un ensemble de fichiers donné. Pointez vers un fichier texte contenant la liste des fichiers que vous voulez copier, un fichier par ligne indiquant le chemin d’accès relatif configuré dans le jeu de données.<br/>Si vous utilisez cette option, ne spécifiez pas de nom de fichier dans le jeu de données. Pour plus d’exemples, consultez [Exemples de listes de fichiers](#file-list-examples). |Non |
| ***Paramètres supplémentaires :** _ |  | |
| recursive | Indique si les données sont lues de manière récursive à partir des sous-dossiers ou uniquement du dossier spécifié. Notez que lorsque l’option « recursive » est définie sur true et que le récepteur est un magasin basé sur un fichier, un dossier vide ou un sous-dossier n’est pas copié ou créé sur le récepteur. <br>Les valeurs autorisées sont _ *true** (par défaut) et **false**.<br>Cette propriété ne s’applique pas lorsque vous configurez `fileListPath`. |Non |
| deleteFilesAfterCompletion | Indique si les fichiers binaires seront supprimés du magasin source après leur déplacement vers le magasin de destination. La suppression se faisant par fichier, lorsque l’activité de copie échoue, vous pouvez constater que certains fichiers ont déjà été copiés vers la destination et supprimés de la source, tandis que d’autres restent dans le magasin source. <br/>Cette propriété est valide uniquement dans un scénario de copie de fichiers binaires. La valeur par défaut est false. |Non |
| modifiedDatetimeStart    | Filtre de fichiers en fonction de l’attribut : Dernière modification. <br>Les fichiers seront sélectionnés si l’heure de leur dernière modification d’inscrit dans l’intervalle de temps compris entre `modifiedDatetimeStart` et `modifiedDatetimeEnd`. L’heure est appliquée au fuseau horaire UTC au format « 2018-12-01T05:00:00Z ». <br> Les propriétés peuvent avoir la valeur Null, ce qui a pour effet qu’aucun filtre d’attribut de fichier n’est appliqué au jeu de données.  Lorsque `modifiedDatetimeStart` a une valeur DateHeure, mais que `modifiedDatetimeEnd` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est supérieur ou égal à la valeur DateHeure sont sélectionnés.  Lorsque `modifiedDatetimeEnd` a une valeur DateHeure, mais que `modifiedDatetimeStart` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est inférieur à la valeur DateHeure sont sélectionnés.<br/>Cette propriété ne s’applique pas lorsque vous configurez `fileListPath`. | Non                                            |
| modifiedDatetimeEnd      | Identique à ce qui précède.                                               | Non                                            |
| enablePartitionDiscovery | Pour les fichiers partitionnés, spécifiez s’il faut analyser les partitions à partir du chemin d’accès au fichier et les ajouter en tant que colonnes sources supplémentaires.<br/>Les valeurs autorisées sont **false** (par défaut) et **true**. | Non                                            |
| partitionRootPath | Lorsque la découverte de partition est activée, spécifiez le chemin d’accès racine absolu pour pouvoir lire les dossiers partitionnés en tant que colonnes de données.<br/><br/>S’il n’est pas spécifié, par défaut :<br/>– Quand vous utilisez le chemin d’accès du fichier dans le jeu de données ou la liste des fichiers sur la source, le chemin racine de la partition est le chemin d’accès configuré dans le jeu de données.<br/>– Quand vous utilisez le filtre de dossiers de caractères génériques, le chemin d’accès racine de la partition est le sous-chemin d’accès avant le premier caractère générique.<br/><br/>Par exemple, en supposant que vous configurez le chemin d’accès dans le jeu de données en tant que « root/folder/year=2020/month=08/day=27 » :<br/>– Si vous spécifiez le chemin d’accès racine de la partition en tant que « root/folder/year=2020 », l’activité de copie génère deux colonnes supplémentaires, `month` et `day`, ayant respectivement la valeur « 08 » et « 27 », en plus des colonnes contenues dans les fichiers.<br/>– Si le chemin d’accès racine de la partition n’est pas spécifié, aucune colonne supplémentaire n’est générée. | Non                                            |
| maxConcurrentConnections | Nombre de connexions simultanées au magasin de stockage. Spécifiez-le uniquement lorsque vous souhaitez limiter les connexions simultanées au magasin de données. | Non                                            |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromADLSGen2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delimited text input dataset name>",
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
                "type": "DelimitedTextSource",
                "formatSettings":{
                    "type": "DelimitedTextReadSettings",
                    "skipLineCount": 10
                },
                "storeSettings":{
                    "type": "AzureBlobFSReadSettings",
                    "recursive": true,
                    "wildcardFolderPath": "myfolder*A",
                    "wildcardFileName": "*.csv"
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="azure-data-lake-storage-gen2-as-a-sink-type"></a>Azure Data Lake Storage Gen2 comme type de récepteur

[!INCLUDE [data-factory-v2-file-sink-formats](../../includes/data-factory-v2-file-sink-formats.md)]

Les propriétés suivantes sont prises en charge pour Data Lake Storage Gen2 dans les paramètres `storeSettings` du récepteur de copie basée sur le format :

| Propriété                 | Description                                                  | Obligatoire |
| ------------------------ | ------------------------------------------------------------ | -------- |
| type                     | La propriété type sous `storeSettings` doit être définie sur **AzureBlobFSWriteSettings**. | Oui      |
| copyBehavior             | Définit le comportement de copie lorsque la source est constituée de fichiers d’une banque de données basée sur un fichier.<br/><br/>Les valeurs autorisées sont les suivantes :<br/><b>- PreserveHierarchy (par défaut)</b> : conserve la hiérarchie des fichiers dans le dossier cible. Le chemin relatif du fichier source vers le dossier source est identique au chemin relatif du fichier cible vers le dossier cible.<br/><b>- FlattenHierarchy</b> : tous les fichiers du dossier source figurent dans le premier niveau du dossier cible. Les noms des fichiers cibles sont générés automatiquement. <br/><b>- MergeFiles</b> : fusionne tous les fichiers du dossier source dans un seul fichier. Si le nom de fichier est spécifié, le nom de fichier fusionné est le nom spécifié. Dans le cas contraire, il s’agit d’un nom de fichier généré automatiquement. | Non       |
| blockSizeInMB | Spécifiez la taille du bloc (en Mo) qui est utilisée pour écrire des données dans ADLS Gen2. En savoir plus sur les [objets blobs de blocs](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-block-blobs). <br/>Les valeurs autorisées sont comprises **entre 4 et 100 Mo**. <br/>Par défaut, ADF détermine automatiquement la taille du bloc en fonction du type et des données de votre magasin source. Pour une copie non binaire dans ADLS Gen2, la taille de bloc par défaut est de 100 Mo, ce qui permet le stockage de 4,95 To de données au maximum. Cela peut ne pas être optimal si vos données ne sont pas volumineuses, en particulier si vous utilisez des runtimes d’intégration auto-hébergés avec un réseau insuffisant qui entraîne l’expiration des opérations ou un problème de performances. Vous pouvez spécifier explicitement une taille de bloc, tout en veillant à ce que blockSizeInMB*50000 soit suffisant pour stocker les données. Si ce n’est pas le cas, l’exécution de l’activité de copie échouera. | Non |
| maxConcurrentConnections | Nombre de connexions simultanées au magasin de données. Spécifiez-le uniquement lorsque vous souhaitez limiter les connexions simultanées au magasin de données. | Non       |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyToADLSGen2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Parquet output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "ParquetSink",
                "storeSettings":{
                    "type": "AzureBlobFSWriteSettings",
                    "copyBehavior": "PreserveHierarchy"
                }
            }
        }
    }
]
```

### <a name="folder-and-file-filter-examples"></a>Exemples de filtres de dossier et de fichier

Cette section décrit le comportement résultant de l’utilisation de filtres de caractères génériques dans les noms de fichier et les chemins de dossier.

| folderPath | fileName | recursive | Structure du dossier source et résultat du filtrage (les fichiers en **gras** sont récupérés)|
|:--- |:--- |:--- |:--- |
| `Folder*` | (vide, utiliser la valeur par défaut) | false | DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5.csv<br/>AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |
| `Folder*` | (vide, utiliser la valeur par défaut) | true | DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier5.csv**<br/>AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |
| `Folder*` | `*.csv` | false | DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5.csv<br/>AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |
| `Folder*` | `*.csv` | true | DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier5.csv**<br/>AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |

### <a name="file-list-examples"></a>Exemples de liste de fichiers

Cette section décrit le comportement résultant de l’utilisation du chemin d’accès à la liste de fichiers dans la source de l’activité de copie.

En supposant que vous disposez de la structure de dossiers source suivante et que vous souhaitez copier les fichiers en gras :

| Exemple de structure source                                      | Contenu de FileListToCopy.txt                             | Configuration ADF                                            |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| filesystem<br/>&nbsp;&nbsp;&nbsp;&nbsp;DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Métadonnées<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FileListToCopy.txt | File1.csv<br>Subfolder1/File3.csv<br>Subfolder1/File5.csv | **Dans le jeu de données :**<br>- système de fichiers : `filesystem`<br>- chemin d’accès du dossier : `FolderA`<br><br>**Dans la source de l’activité de copie :**<br>- chemin d’accès à la liste de fichiers : `filesystem/Metadata/FileListToCopy.txt` <br><br>Le chemin d’accès à la liste de fichiers pointe vers un fichier texte dans le même magasin de données qui contient la liste de fichiers que vous voulez copier, un fichier par ligne étant le chemin d’accès relatif au chemin d’accès configuré dans le jeu de données. |


### <a name="some-recursive-and-copybehavior-examples"></a>Quelques exemples de valeurs recursive et copyBehavior

Cette section décrit le comportement résultant de l’opération de copie pour différentes combinaisons de valeurs recursive et copyBehavior.

| recursive | copyBehavior | Structure du dossier source | Cible obtenue |
|:--- |:--- |:--- |:--- |
| true |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 | Le Dossier1 cible est créé et structuré de la même manière que la source :<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 |
| true |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 | Le dossier cible Dossier1 est créé et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier5 |
| true |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 | Le dossier cible Dossier1 est créé et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Le contenu de Fichier1 + Fichier2 + Fichier3 + Fichier4 + Fichier5 est fusionné dans un fichier avec un nom de fichier généré automatiquement. |
| false |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 | Le dossier cible Dossier1 est créé et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |
| false |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 | Le dossier cible Dossier1 est créé et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier2<br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |
| false |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 | Le dossier cible Dossier1 est créé et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Le contenu de Fichier1 + Fichier2 est fusionné dans un fichier avec un nom de fichier généré automatiquement. nom généré automatiquement pour Fichier1<br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |

## <a name="preserve-metadata-during-copy"></a>Conserver les métadonnées lors de la copie

Lorsque vous copiez des fichiers depuis Amazon S3/Azure Blob/Azure Data Lake Storage Gen2 vers Azure Data Lake Storage Gen2/Azure Blob, vous pouvez choisir de conserver les métadonnées des fichiers avec les données. Pour plus d’informations, consultez [Conserver les métadonnées](copy-activity-preserve-metadata.md#preserve-metadata).

## <a name="preserve-acls-from-data-lake-storage-gen1gen2"></a><a name="preserve-acls"></a> Conserver les listes de contrôle d’accès de Data Lake Storage Gen1/Gen2

Lorsque vous copiez des fichiers à partir d’Azure Data Lake Storage Gen1/Gen2 vers Gen2, vous pouvez choisir de conserver les listes de contrôle d’accès (ACL) POSIX, ainsi que des données. Pour plus d’informations, consultez [Conserver les listes de contrôle d’accès entre Data Lake Storage Gen1/Gen2 et Gen2](copy-activity-preserve-metadata.md#preserve-acls).

>[!TIP]
>Pour copier des données à partir d’Azure Data Lake Storage Gen1 dans Gen2 en général, consultez l’article [Copier les données depuis Azure Data Lake Storage Gen1 vers Gen2 avec Azure Data Factory](load-azure-data-lake-storage-gen2-from-gen1.md) pour obtenir une procédure pas à pas et découvrir les meilleures pratiques.

## <a name="mapping-data-flow-properties"></a>Propriétés du mappage de flux de données

Lorsque vous transformez des données en flux de données de mappage, vous pouvez lire et écrire des fichiers à partir d’Azure Data Lake Storage Gen2 aux formats suivants :
* [Avro](format-avro.md#mapping-data-flow-properties)
* [Common Data Model (préversion)](format-common-data-model.md#mapping-data-flow-properties)
* [Texte délimité](format-delimited-text.md#mapping-data-flow-properties)
* [Delta](format-delta.md#mapping-data-flow-properties)
* [Excel](format-excel.md#mapping-data-flow-properties)
* [JSON](format-json.md#mapping-data-flow-properties)
* [Parquet](format-parquet.md#mapping-data-flow-properties)

Les paramètres spécifiques du format se trouvent dans la documentation de ce format. Pour plus d’informations, consultez [Transformation de source en flux de données de mappage](data-flow-source.md) et [Transformation de récepteur en flux de données de mappage](data-flow-sink.md).

### <a name="source-transformation"></a>Transformation de la source

Dans la transformation de la source, vous pouvez lire à partir d’un conteneur, d’un dossier ou d’un fichier individuel dans Azure Data Lake Storage Gen2. L’onglet **Options de la source** vous permet de gérer la façon dont les fichiers sont lus. 

![Options de la source](media/data-flow/sourceOptions1.png "Options de la source")

**Chemin contenant des caractères génériques :** Si vous utilisez un modèle à caractères génériques, le système demande à ADF de lire chaque dossier et fichier correspondants en boucle dans une même transformation de source. Il s’agit d’un moyen efficace de traiter plusieurs fichiers dans un seul et même flux. Ajoutez plusieurs modèles de correspondance à caractères génériques avec le signe + qui apparaît quand vous placez le pointeur sur votre modèle existant.

Dans le conteneur source, choisissez une série de fichiers qui correspondent à un modèle. Seul le conteneur peut être spécifié dans le jeu de données. Votre chemin contenant des caractères génériques doit donc également inclure le chemin de votre dossier à partir du dossier racine.

Exemples de caractères génériques :

* ```*``` Représente n’importe quel ensemble de caractères
* ```**``` Représente une imbrication de répertoires récursifs
* ```?``` Remplace un caractère
* ```[]``` Met en correspondance un ou plusieurs caractères indiqués entre crochets

* ```/data/sales/**/*.csv``` Obtient tous les fichiers CSV se trouvant sous /data/sales
* ```/data/sales/20??/**/``` Obtient tous les fichiers datés du 20ème siècle
* ```/data/sales/*/*/*.csv``` Obtient les fichiers CSV à deux niveaux sous /data/sales
* ```/data/sales/2004/*/12/[XY]1?.csv``` Obtient tous les fichiers CSV datés de décembre 2004, commençant par X ou Y et ayant comme préfixe un nombre à deux chiffres

**Chemin racine de la partition :** Si vous avez partitionné des dossiers dans votre source de fichiers avec un format ```key=value``` (par exemple, année = 2019), vous pouvez attribuer le niveau supérieur de cette arborescence de dossiers de partitions à un nom de colonne dans votre flux de données.

Tout d’abord, définissez un caractère générique pour inclure tous les chemins d’accès aux dossiers partitionnés, ainsi qu’aux fichiers feuilles que vous souhaitez lire.

![Paramètres du fichier source de partition](media/data-flow/partfile2.png "Paramètre du fichier de partition")

Utilisez le paramètre Chemin racine de la partition pour définir le niveau supérieur de la structure de dossiers. Quand vous affichez le contenu de vos données à l’aide d’un aperçu des données, vous voyez qu’ADF ajoute les partitions résolues trouvées dans chacun de vos niveaux de dossiers.

![Chemin racine de la partition](media/data-flow/partfile1.png "Aperçu du chemin racine de la partition")

**Liste de fichiers :** Il s’agit d’un ensemble de fichiers. Créez un fichier texte qui inclut une liste de fichiers avec chemin relatif à traiter. Pointez sur ce fichier texte.

**Colonne où stocker le nom du fichier :** Stockez le nom du fichier source dans une colonne de vos données. Entrez un nouveau nom de colonne pour stocker la chaîne de nom de fichier.

**Après l’exécution :** après l’exécution du flux de données, choisissez de ne rien faire avec le fichier source, de le supprimer ou de le déplacer. Pour le déplacement, les chemins sont des chemins relatifs.

Pour déplacer les fichiers sources vers un autre emplacement de post-traitement, sélectionnez tout d’abord « Déplacer » comme opération de fichier. Définissez ensuite le répertoire de provenance (« from »). Si vous n’utilisez pas de caractères génériques pour votre chemin, le paramètre « from » sera le même dossier que votre dossier source.

Si vous avez un chemin d’accès source avec caractère générique, votre syntaxe se présente comme suit :

```/data/sales/20??/**/*.csv```

vous pouvez spécifier «from» sous la forme

```/data/sales```

et « to » sous la forme

```/backup/priorSales```

Dans le cas présent, tous les fichiers qui provenaient de /data/sales sont déplacés dans /backup/priorSales.

> [!NOTE]
> Les opérations de fichier s’exécutent uniquement quand vous démarrez le flux de données à partir d’une exécution de pipeline (débogage ou exécution) qui utilise l’activité Exécuter le flux de données dans un pipeline. Les opérations de fichiers ne s’exécutent *pas* en mode de débogage de flux de données.

**Filtrer par date de dernière modification :** Vous pouvez filtrer les fichiers traités en spécifiant une plage de dates sur laquelle les fichiers ont été modifiés pour la dernière fois. Toutes les dates et heures sont exprimées en UTC. 

### <a name="sink-properties"></a>Propriétés du récepteur

Dans la transformation du récepteur, vous pouvez écrire dans un conteneur ou un dossier dans Azure Data Lake Storage Gen2. L’onglet **Paramètres** vous permet de gérer la façon dont les fichiers sont écrits.

![Options du récepteur](media/data-flow/file-sink-settings.png "Options du récepteur")

**Effacer le contenu du dossier :** Détermine si le contenu du dossier de destination doit être effacé avant l’écriture des données.

**Option de nom de fichier :** Détermine la façon dont les fichiers de destination sont nommés dans le dossier de destination. Les options de nom de fichier sont les suivantes :
   * **Par défaut** : Autorisez Spark à nommer les fichiers en fonction des valeurs par défaut de la partition.
   * **Modèle** : Entrez un modèle qui énumère vos fichiers de sortie par partition. Par exemple, **loans[n].csv** crée loans1.csv, loans2.csv, etc.
   * **Par partition** : Entrez un nom de fichier pour chaque partition.
   * **Comme les données de la colonne** : Définissez le fichier de sortie sur la valeur d’une colonne. Le chemin est relatif au conteneur du jeu de données et non pas au dossier de destination. Si vous avez un chemin de dossier dans votre jeu de données, il sera remplacé.
   * **Sortie d’un seul fichier** : Combinez les fichiers de sortie partitionnés en un seul fichier nommé. Le chemin est relatif au dossier du jeu de données. Sachez que cette opération de fusion peut échouer en fonction de la taille du nœud. Cette option n’est pas recommandée pour des jeux de données volumineux.

**Tout mettre entre guillemets :** Détermine si toutes les valeurs doivent être placées entre guillemets

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour en savoir plus sur les propriétés, consultez [Activité Lookup](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Propriétés de l’activité GetMetadata

Pour en savoir plus sur les propriétés, consultez [Activité GetMetadata](control-flow-get-metadata-activity.md). 

## <a name="delete-activity-properties"></a>Propriétés de l’activité Delete

Pour en savoir plus sur les propriétés, consultez [Activité Delete](delete-activity.md).

## <a name="legacy-models"></a>Modèles hérités

>[!NOTE]
>Les Modèles suivants sont toujours pris en charge tels quels à des fins de compatibilité descendante. Il est recommandé d’utiliser le nouveau Modèle mentionné dans les sections ci-dessus à partir de maintenant. L’interface utilisateur de création ADF peut désormais générer ce nouveau Modèle.

### <a name="legacy-dataset-model"></a>Modèle de jeu de données hérité

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type du jeu de données doit être définie sur **AzureBlobFSFile**. |Oui |
| folderPath | Chemin vers le dossier dans Data Lake Storage Gen2. Si non spécifié, il pointe vers la racine. <br/><br/>Le filtre de caractères génériques est pris en charge. Les caractères génériques autorisés sont les suivants : `*` (correspond à zéro ou plusieurs caractères) et `?` (correspond à zéro ou un caractère). Utilisez `^` comme caractère d'échappement si le nom réel de votre dossier contient des caractères génériques ou ce caractère d'échappement. <br/><br/>Exemples : système de fichiers/dossier/. Consultez d’autres exemples dans les [exemples de filtre de dossier et de fichier](#folder-and-file-filter-examples). |Non |
| fileName | Filtre de nom ou de caractère générique pour les fichiers sous le « folderPath » spécifié. Si vous ne spécifiez pas de valeur pour cette propriété, le jeu de données pointe vers tous les fichiers du dossier. <br/><br/>Dans le filtre, les caractères génériques autorisés sont `*` (correspond à zéro caractère ou plus) et `?` (correspond à zéro ou un caractère).<br/>- Exemple 1 : `"fileName": "*.csv"`<br/>- Exemple 2 : `"fileName": "???20180427.txt"`<br/>Utilisez `^` comme caractère d'échappement si le nom réel de votre fichier contient des caractères génériques ou ce caractère d'échappement.<br/><br/>Lorsque fileName n’est pas spécifié pour un jeu de données de sortie et que **preserveHierarchy** n’est pas spécifié dans le récepteur d’activité, l’activité de copie génère automatiquement le nom de fichier suivant ce modèle : « *Data.[ID GUID d’exécution d’activité].[GUID si FlattenHierarchy].[format si configuré].[compression si configurée]*  », par exemple, « Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt.gz ». Si vous effectuez la copie à partir d’une source tabulaire à l’aide d’un nom de table au lieu d’une requête, le modèle du nom est «  *[nom_table].[format].[compression si configurée]*  », par exemple « MyTable.csv ». |Non |
| modifiedDatetimeStart | Filtre de fichiers en fonction de l’attribut Dernière modification. Les fichiers sont sélectionnés si leur heure de dernière modification se trouve dans l’intervalle de temps situé entre `modifiedDatetimeStart` et `modifiedDatetimeEnd`. L’heure est appliquée au fuseau horaire UTC au format « 2018-12-01T05:00:00Z ». <br/><br/> Les performances globales du déplacement des données sont influencées par l’activation de ce paramètre lorsque vous souhaitez appliquer un filtre sur de grandes quantités de fichiers. <br/><br/> Les propriétés peuvent être NULL, ce qui signifie qu’aucun filtre d’attribut de fichier n’est appliqué au jeu de données. Lorsque `modifiedDatetimeStart` a une valeur DateHeure, mais que `modifiedDatetimeEnd` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est supérieur ou égal à la valeur DateHeure sont sélectionnés. Lorsque `modifiedDatetimeEnd` a une valeur DateHeure, mais que `modifiedDatetimeStart` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est inférieur à la valeur DateHeure sont sélectionnés.| Non |
| modifiedDatetimeEnd | Filtre de fichiers en fonction de l’attribut Dernière modification. Les fichiers sont sélectionnés si leur heure de dernière modification se trouve dans l’intervalle de temps situé entre `modifiedDatetimeStart` et `modifiedDatetimeEnd`. L’heure est appliquée au fuseau horaire UTC au format « 2018-12-01T05:00:00Z ». <br/><br/> Les performances globales du déplacement des données sont influencées par l’activation de ce paramètre lorsque vous souhaitez appliquer un filtre sur de grandes quantités de fichiers. <br/><br/> Les propriétés peuvent être NULL, ce qui signifie qu’aucun filtre d’attribut de fichier n’est appliqué au jeu de données. Lorsque `modifiedDatetimeStart` a une valeur DateHeure, mais que `modifiedDatetimeEnd` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est supérieur ou égal à la valeur DateHeure sont sélectionnés. Lorsque `modifiedDatetimeEnd` a une valeur DateHeure, mais que `modifiedDatetimeStart` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est inférieur à la valeur DateHeure sont sélectionnés.| Non |
| format | Si vous souhaitez copier des fichiers en l’état entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie.<br/><br/>Si vous souhaitez analyser ou générer des fichiers dans un format spécifique, les types de format de fichier suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Définissez la propriété **type** située sous **Format** sur l’une de ces valeurs. Pour en savoir plus, voir les sections [Format Text](supported-file-formats-and-compression-codecs-legacy.md#text-format), [Format JSON](supported-file-formats-and-compression-codecs-legacy.md#json-format), [Format Avro](supported-file-formats-and-compression-codecs-legacy.md#avro-format), [Format ORC](supported-file-formats-and-compression-codecs-legacy.md#orc-format) et [Format Parquet](supported-file-formats-and-compression-codecs-legacy.md#parquet-format). |Non (uniquement pour un scénario de copie binaire) |
| compression | Spécifiez le type et le niveau de compression pour les données. Pour plus d’informations, voir [Formats de fichier et de codecs de compression pris en charge](supported-file-formats-and-compression-codecs-legacy.md#compression-support).<br/>Les types pris en charge sont : **GZip**, **Deflate**, **BZip2** et **ZipDeflate**.<br/>Les niveaux pris en charge sont **Optimal** et **Fastest**. |Non |

>[!TIP]
>Pour copier tous les fichiers d’un dossier, spécifiez **folderPath** uniquement.<br>Pour copier un seul fichier avec un nom donné, spécifiez **folderPath** avec la partie dossier et **fileName** avec un nom de fichier.<br>Pour copier un sous-ensemble de fichiers d’un dossier, spécifiez **folderPath** avec la partie dossier et **fileName** avec un filtre de caractères génériques. 

**Exemple :**

```json
{
    "name": "ADLSGen2Dataset",
    "properties": {
        "type": "AzureBlobFSFile",
        "linkedServiceName": {
            "referenceName": "<Azure Data Lake Storage Gen2 linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "myfilesystem/myfolder",
            "fileName": "*",
            "modifiedDatetimeStart": "2018-12-01T05:00:00Z",
            "modifiedDatetimeEnd": "2018-12-01T06:00:00Z",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

### <a name="legacy-copy-activity-source-model"></a>Modèle hérité de la source d’activité de copie

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type de la source d’activité de copie doit être définie sur **AzureBlobFSSource**. |Oui |
| recursive | Indique si les données sont lues de manière récursive à partir des sous-dossiers ou uniquement du dossier spécifié. Lorsque l’option « recursive » est définie sur true et que le récepteur est un magasin basé sur un fichier, un dossier vide ou un sous-dossier n’est pas copié ou créé sur le récepteur.<br/>Les valeurs autorisées sont **true** (par défaut) et **false**. | Non |
| maxConcurrentConnections | Nombre de connexions simultanées au magasin de données. Spécifiez-le uniquement lorsque vous souhaitez limiter les connexions simultanées au magasin de données. | Non |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromADLSGen2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<ADLS Gen2 input dataset name>",
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
                "type": "AzureBlobFSSource",
                "recursive": true
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="legacy-copy-activity-sink-model"></a>Modèle hérité du récepteur d’activité de copie

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type du récepteur d’activité de copie doit être définie sur **AzureBlobFSSink**. |Oui |
| copyBehavior | Définit le comportement de copie lorsque la source est constituée de fichiers d’une banque de données basée sur un fichier.<br/><br/>Les valeurs autorisées sont les suivantes :<br/><b>- PreserveHierarchy (par défaut)</b> : conserve la hiérarchie des fichiers dans le dossier cible. Le chemin relatif du fichier source vers le dossier source est identique au chemin relatif du fichier cible vers le dossier cible.<br/><b>- FlattenHierarchy</b> : tous les fichiers du dossier source figurent dans le premier niveau du dossier cible. Les noms des fichiers cibles sont générés automatiquement. <br/><b>- MergeFiles</b> : fusionne tous les fichiers du dossier source dans un seul fichier. Si le nom de fichier est spécifié, le nom de fichier fusionné est le nom spécifié. Dans le cas contraire, il s’agit d’un nom de fichier généré automatiquement. | Non |
| maxConcurrentConnections | Nombre de connexions simultanées au magasin de données. Spécifiez-le uniquement lorsque vous souhaitez limiter les connexions simultanées au magasin de données. | Non |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyToADLSGen2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<ADLS Gen2 output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureBlobFSSink",
                "copyBehavior": "PreserveHierarchy"
            }
        }
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir la liste des banques de données prises en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez le tableau [Banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).