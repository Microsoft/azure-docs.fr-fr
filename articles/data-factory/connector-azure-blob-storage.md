---
title: Copier et transformer des données dans Stockage Blob Azure
description: Découvrez comment copier des données vers et depuis un stockage Blob, et comment transformer les données dans un stockage Blob en utilisant Data Factory.
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 12/08/2020
ms.openlocfilehash: 327ab63a048e08328cd5b1334b0a697a61ced6c8
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97346328"
---
# <a name="copy-and-transform-data-in-azure-blob-storage-by-using-azure-data-factory"></a>Copier et transformer des données dans un stockage Azure Blob à l’aide d’Azure Data Factory

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-azure-blob-connector.md)
> * [Version actuelle](connector-azure-blob-storage.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article décrit comment utiliser l’activité Copy dans Azure Data Factory pour copier des données d’un stockage d’objets Blob Azure ou vers un stockage d’objets blob Azure. Il explique également comment utiliser l’activité Data Flow pour transformer des données en stockage d’objets blob Azure. Pour en savoir plus sur Azure Data Factory, lisez l’[article d’introduction](introduction.md).

>[!TIP]
>Pour découvrir un scénario de migration de lac de données ou d’entrepôt de données, consultez [Utiliser Azure Data Factory pour migrer des données d’un lac de données ou d’un entrepôt de données vers Azure](data-migration-guidance-overview.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur de stockage d’objets blob Azure est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Mappage de flux de données](concepts-data-flow-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)
- [Activité GetMetadata](control-flow-get-metadata-activity.md)
- [Supprimer l’activité](delete-activity.md)

Pour l’activité Copy, ce connecteur de stockage d’objets blob prend en charge les opérations suivantes :

- La copie d’objets blob vers et depuis des comptes de stockage Azure à usage général et un stockage d’objets blob à chaud ou à froid. 
- Copie d’objets blob à l’aide d’une clé de compte, d’une signature d’accès partagé (SAP) de service ou d’identités managées pour des authentifications de ressources Azure.
- La copie d’objets blob à partir d’objets blob de blocs, d’ajout ou de page, et la copie de données uniquement vers des objets blob de blocs.
- Copie d’objets blob en l’état, ou analyse/génération d’objets blob avec des [formats de fichier et codecs de compression pris en charge](supported-file-formats-and-compression-codecs.md).
- [Conservation des métadonnées de fichier lors de la copie](#preserving-metadata-during-copy).

## <a name="get-started"></a>Bien démarrer

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes fournissent des informations détaillées sur les propriétés utilisées pour définir les entités Azure Data Factory spécifiques au stockage Blob.

## <a name="linked-service-properties"></a>Propriétés du service lié

Ce connecteur de stockage d’objets blob prend en charge les types d’authentification suivants. Pour plus d’informations, consultez les sections correspondantes.

- [Authentification par clé de compte](#account-key-authentication)
- [Authentification avec une signature d’accès partagé](#shared-access-signature-authentication)
- [Authentification d’un principal du service](#service-principal-authentication)
- [Identités managées pour l’authentification des ressources Azure](#managed-identity)

>[!NOTE]
>- Si vous utilisez le runtime d’intégration Azure public pour vous connecter à votre Stockage Blob avec l’option **Autoriser les services Microsoft approuvés à accéder à ce compte de stockage** activée sur le pare-feu Stockage Azure, vous devez recourir à [l’authentification par identité managée](#managed-identity).
>- Si vous utilisez PolyBase ou l’instruction COPY pour charger des données dans Azure Synapse Analytics et que votre Stockage Blob source ou de préproduction est configuré avec un point de terminaison de réseau virtuel Azure, vous devez utiliser l’authentification par identité managée comme l’exige Synapse. Pour en savoir plus sur la configuration requise, consultez la section sur [Authentification par identité managée](#managed-identity).

>[!NOTE]
>Les activités Azure HDInsight et Azure Machine Learning prennent en charge uniquement l’authentification utilisant des clés de compte de stockage d’objets blob Azure.

### <a name="account-key-authentication"></a>Authentification par clé de compte

Pour l’authentification par clé de compte de stockage, Data Factory prend en charge les propriétés suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** doit être définie sur **AzureBlobStorage** (recommandé) ou **AzureStorage** (voir les remarques ci-dessous). |Oui |
| connectionString | Pour vous connecter au Stockage, pour la propriété **connectionString**, spécifiez les informations requises. <br/> Vous pouvez également définir une clé de compte dans Azure Key Vault et extraire la configuration `accountKey` de la chaîne de connexion. Pour plus d’informations, consultez les exemples suivants et l’article [Stocker les informations d’identification dans Azure Key Vault](store-credentials-in-key-vault.md). |Oui |
| connectVia | Le [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Vous pouvez utiliser le runtime d’intégration Azure ou un runtime d’intégration auto-hébergé (si votre banque de données se trouve sur un réseau privé). Si cette propriété n’est pas spécifiée, le service utilise le runtime d’intégration Azure par défaut. |Non |

>[!NOTE]
>Un point de terminaison de service BLOB secondaire n’est pas pris en charge lors de l’utilisation de l’authentification par clé de compte. Vous pouvez utiliser d’autres types d’authentification.

>[!NOTE]
>Si vous utilisez le service lié de type « AzureStorage », il est toujours pris en charge tel quel. Toutefois, nous vous suggérons d’utiliser désormais le nouveau type de service lié « AzureBlobStorage ».

**Exemple :**

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Exemple : stockage de la clé de compte dans Azure Key Vault**

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;",
            "accountKey": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }            
    }
}
```

### <a name="shared-access-signature-authentication"></a>Authentification avec une signature d’accès partagé

Une signature d'accès partagé fournit un accès délégué aux ressources de votre compte de stockage. Vous pouvez utiliser une signature d’accès partagé pour octroyer à un client des autorisations d’accès limité à des objets de votre compte de stockage pendant une période donnée. 

Vous n’êtes pas obligé de partager vos clés d’accès de compte. La signature d’accès partagé est un URI qui englobe dans ses paramètres de requête toutes les informations nécessaires pour obtenir un accès authentifié à une ressource de stockage. Pour accéder aux ressources de stockage avec la signature d’accès partagé, il suffit au client de transmettre cette dernière à la méthode ou au constructeur approprié. 

Pour plus d’informations sur les signatures d’accès partagé, consultez [Signatures d’accès partagé : Comprendre le modèle de signature d’accès partagé](../storage/common/storage-sas-overview.md).

> [!NOTE]
>- Azure Data Factory prend désormais en charge les *signatures d’accès partagé de service* et les *signatures d’accès partagé de compte*. Pour plus d’informations sur les signatures d’accès partagé, consultez [Accorder un accès limité aux ressources du Stockage Azure à l’aide des signatures d’accès partagé](../storage/common/storage-sas-overview.md).
>- Dans les configurations de jeu de données ultérieures, le chemin du dossier est le chemin absolu commençant au niveau du conteneur. Vous devez en configurer un qui soit aligné avec le chemin dans votre URI SAS.

Pour l’authentification par signature d’accès partagé, Data Factory prend en charge les propriétés suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** doit être définie sur **AzureBlobStorage** (recommandé) ou **AzureStorage** (voir la remarque ci-dessous). |Oui |
| sasUri | Spécifiez l’URI de signature d’accès partagé des ressources de stockage, telles qu’un objet blob ou un conteneur. <br/>Marquez ce champ comme **SecureString** pour le stocker en toute sécurité dans Data Factory. Vous pouvez également placer le jeton SAP dans Azure Key Vault pour utiliser la rotation automatique et supprimer la portion jeton. Pour plus d’informations, consultez les exemples suivants et [Stocker des informations d’identification dans Azure Key Vault](store-credentials-in-key-vault.md). |Oui |
| connectVia | Le [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Vous pouvez utiliser le runtime d’intégration Azure ou un runtime d’intégration auto-hébergé (si votre banque de données se trouve sur un réseau privé). Si cette propriété n’est pas spécifiée, le service utilise le runtime d’intégration Azure par défaut. |Non |

>[!NOTE]
>Si vous utilisez le service lié de type « AzureStorage », il est toujours pris en charge tel quel. Toutefois, nous vous suggérons d’utiliser désormais le nouveau type de service lié « AzureBlobStorage ».

**Exemple :**

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {
            "sasUri": {
                "type": "SecureString",
                "value": "<SAS URI of the Azure Storage resource e.g. https://<accountname>.blob.core.windows.net/?sv=<storage version>&st=<start time>&se=<expire time>&sr=<resource>&sp=<permissions>&sip=<ip range>&spr=<protocol>&sig=<signature>>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Exemple : stockage de la clé de compte dans Azure Key Vault**

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {
            "sasUri": {
                "type": "SecureString",
                "value": "<SAS URI of the Azure Storage resource without token e.g. https://<accountname>.blob.core.windows.net/>"
            },
            "sasToken": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName with value of SAS token e.g. ?sv=<storage version>&st=<start time>&se=<expire time>&sr=<resource>&sp=<permissions>&sip=<ip range>&spr=<protocol>&sig=<signature>>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

Lorsque vous créez un URI de signature d’accès partagé, prenez en compte les points suivants :

- Définissez des autorisations de lecture/écriture appropriées sur les objets en fonction de l’utilisation du service lié (lecture, écriture, lecture/écriture) dans votre fabrique de données.
- Définissez le paramètre **Heure d’expiration** correctement. Assurez-vous que l’accès aux objets du stockage n’expire pas pendant la période active du pipeline.
- L’URI doit être créé au niveau du blob ou du conteneur appropriés en fonction des besoins. Un URI de signature d’accès partagé à un objet blob permet à Azure Data Factory d’accéder à cet objet blob particulier. Un URI de signature d’accès partagé à un conteneur de stockage Blob permet à Azure Data Factory d’itérer via des objets blob dans ce conteneur. Pour fournir l’accès à plus ou moins d’objets ultérieurement ou mettre à jour l’URI de signature d’accès partagé, rappelez-vous de mettre à jour le service lié avec le nouvel URI.

### <a name="service-principal-authentication"></a>Authentification d’un principal du service

Pour des informations générales sur l’authentification du principal de service du Stockage Azure, consultez [Authentifier l’accès au Stockage Azure à l’aide d’Azure Active Directory](../storage/common/storage-auth-aad.md).

Pour l’authentification de principal de service, effectuez les étapes suivantes :

1. Inscrivez une entité d’application dans Azure Active Directory (Azure AD) en suivant les instructions de la section [Inscrire votre application à un locataire Azure AD](../storage/common/storage-auth-aad-app.md#register-your-application-with-an-azure-ad-tenant). Prenez note des valeurs suivantes qui vous permettent de définir le service lié :

    - ID de l'application
    - Clé de l'application
    - ID client

2. Accordez l’autorisation appropriée au principal de service dans le Stockage Blob Azure : Pour plus d’informations sur les rôles, consultez [Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](../storage/common/storage-auth-aad-rbac-portal.md).

    - **En tant que source**, dans **Contrôle d’accès (IAM)** , accordez au moins le rôle **Lecteur des données blob du stockage**.
    - **En tant que récepteur**, dans **Contrôle d’accès (IAM)** , accordez au moins le rôle **Contributeur aux données Blob du stockage**.

Les propriétés prises en charge pour un service lié de Stockage Blob Azure sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** doit être définie sur **AzureBlobStorage**. |Oui |
| serviceEndpoint | Spécifiez le point de terminaison du service Stockage Blob Azure à l’aide du modèle suivant : `https://<accountName>.blob.core.windows.net/`. |Oui |
| accountKind | Spécifiez le type de votre compte de stockage. Les valeurs autorisées sont les suivantes : **Stockage** (v1 à usage général), **StorageV2** (v2 à usage général), **BlobStorage** ou **BlockBlobStorage**. <br/> Lorsque le service lié Blob Azure est utilisé dans un flux de données, l’authentification par identité managée ou par principal de service n’est pas prise en charge si le type de compte est vide ou « Stockage ». Spécifiez le type de compte approprié, choisissez une autre authentification ou mettez à niveau votre compte de stockage vers la version v2 à usage général. |Non |
| servicePrincipalId | Spécifiez l’ID client de l’application. | Oui |
| servicePrincipalKey | Spécifiez la clé de l’application. Marquez ce champ en tant que **SecureString** afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui |
| tenant | Spécifiez les informations de locataire (nom de domaine ou ID de locataire) dans lesquels se trouve votre application. Récupérez-les en pointant dans l’angle supérieur droit du portail Azure. | Oui |
| azureCloudType | Pour l’authentification du principal du service, spécifiez le type d’environnement cloud Azure auprès duquel votre application Azure Active Directory est inscrite. <br/> Les valeurs autorisées sont **AzurePublic**, **AzureChina**, **AzureUsGovernment** et **AzureGermany**. Par défaut, l’environnement cloud de la fabrique de données est utilisé. | Non |
| connectVia | Le [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Vous pouvez utiliser le runtime d’intégration Azure ou un runtime d’intégration auto-hébergé (si votre banque de données se trouve sur un réseau privé). Si cette propriété n’est pas spécifiée, le service utilise le runtime d’intégration Azure par défaut. |Non |

>[!NOTE]
>Si votre compte d’objet blob active la [suppression réversible](../storage/blobs/soft-delete-blob-overview.md), l’authentification de principal du service n’est pas prise en charge dans Data Flow.

>[!NOTE]
>L’authentification du principal du service n’est prise en charge que par le service lié de type « AzureBlobStorage », non par le service lié de type « AzureStorage » précédent.

**Exemple :**

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {            
            "serviceEndpoint": "https://<accountName>.blob.core.windows.net/",
            "accountKind": "StorageV2",
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

### <a name="managed-identities-for-azure-resource-authentication"></a><a name="managed-identity"></a> Identités managées pour l’authentification des ressources Azure

Une fabrique de données peut être associée à une [identité managée pour les ressources Azure](data-factory-service-identity.md), laquelle représente cette même fabrique de données. Vous pouvez utiliser directement cette identité managée pour l’authentification de stockage d’objets blob, ce qui revient à utiliser votre propre principal de service. Cela permet à la fabrique désignée d’accéder aux données et de les copier à partir de votre stockage d’objets blob.

Pour des informations générales sur l’authentification de Stockage Azure, consultez [Authentifier l’accès au Stockage Azure à l’aide d’Azure Active Directory](../storage/common/storage-auth-aad.md). Pour utiliser des identités managées afin d’authentifier des ressources Azure, procédez comme suit :

1. [Récupérez les informations d’identité managée de Data Factory](data-factory-service-identity.md#retrieve-managed-identity) en copiant la valeur de l’ID d’objet d’identité managée générée en même temps que votre fabrique.

2. Accordez l’autorisation d’identité managée dans le stockage d’objets blob Azure. Pour plus d’informations sur les rôles, consultez [Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](../storage/common/storage-auth-aad-rbac-portal.md).

    - **En tant que source**, dans **Contrôle d’accès (IAM)** , accordez au moins le rôle **Lecteur des données blob du stockage**.
    - **En tant que récepteur**, dans **Contrôle d’accès (IAM)** , accordez au moins le rôle **Contributeur aux données Blob du stockage**.

>[!IMPORTANT]
>Si vous utilisez PolyBase ou l’instruction COPY pour charger des données du Stockage Blob (comme source ou emplacement de préproduction) dans Azure Synapse Analytics et que vous avez recours à l’authentification par identité managée pour le Stockage Blob, veillez également à suivre les étapes 1 à 3 de [cette aide](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage). Ces étapes inscrivent votre serveur auprès d’Azure AD et attribuent le rôle de contributeur aux données de l’objet blob de stockage. Data Factory gère le reste. Si vous configurez le Stockage Blob avec un point de terminaison de réseau virtuel Azure, vous devez également activer **Autoriser les services Microsoft approuvés à accéder à ce compte de stockage** dans le menu des paramètres **Pare-feu et réseaux virtuels** du compte Stockage Azure, comme l’exige Synapse.

Les propriétés prises en charge pour un service lié de Stockage Blob Azure sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** doit être définie sur **AzureBlobStorage**. |Oui |
| serviceEndpoint | Spécifiez le point de terminaison du service Stockage Blob Azure à l’aide du modèle suivant : `https://<accountName>.blob.core.windows.net/`. |Oui |
| accountKind | Spécifiez le type de votre compte de stockage. Les valeurs autorisées sont les suivantes : **Stockage** (v1 à usage général), **StorageV2** (v2 à usage général), **BlobStorage** ou **BlockBlobStorage**. <br/> Lorsque le service lié Blob Azure est utilisé dans un flux de données, l’authentification par identité managée ou par principal de service n’est pas prise en charge si le type de compte est vide ou « Stockage ». Spécifiez le type de compte approprié, choisissez une autre authentification ou mettez à niveau votre compte de stockage vers la version v2 à usage général. |Non |
| connectVia | Le [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Vous pouvez utiliser le runtime d’intégration Azure ou un runtime d’intégration auto-hébergé (si votre banque de données se trouve sur un réseau privé). Si cette propriété n’est pas spécifiée, le service utilise le runtime d’intégration Azure par défaut. |Non |

> [!NOTE]
> Si votre compte d’objet blob active la [suppression réversible](../storage/blobs/soft-delete-blob-overview.md), l’authentification d’identité managée n’est pas prise en charge dans Data Flow.

> [!NOTE]
> Les identités managées pour l’authentification des ressources Azure ne sont prises en charge que par le service lié de type « AzureBlobStorage », non par le service lié de type « AzureStorage » précédent.

**Exemple :**

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {            
            "serviceEndpoint": "https://<accountName>.blob.core.windows.net/",
            "accountKind": "StorageV2" 
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Jeux de données](concepts-datasets-linked-services.md). 

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Les propriétés suivantes sont prises en charge pour le stockage d’objets blob Azure sous les paramètres `location` dans un jeu de données basé sur un format :

| Propriété   | Description                                                  | Obligatoire |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | La propriété **type** de l’emplacement dans le jeu de données doit être définie sur **AzureBlobStorageLocation**. | Oui      |
| conteneur  | Le conteneur d’objets blob.                                          | Oui      |
| folderPath | Chemin d’accès au dossier sous le conteneur donné. Si vous souhaitez utiliser un caractère générique pour filtrer le dossier, ignorez ce paramètre et spécifiez-le dans les paramètres de la source de l’activité. | Non       |
| fileName   | Le nom de fichier sous le conteneur et le chemin d’accès du dossier donnés. Si vous souhaitez utiliser un caractère générique pour filtrer les fichiers, ignorez ce paramètre et spécifiez cela dans les paramètres de la source de l’activité. | Non       |

**Exemple :**

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
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

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés que la source et le récepteur du stockage d’objets blob prennent en charge.

### <a name="blob-storage-as-a-source-type"></a>Stockage Blob en tant que type de source

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Les propriétés suivantes sont prises en charge pour le stockage d’objets blob Azure sous les paramètres `storeSettings` dans une source de copie basée sur un format :

| Propriété                 | Description                                                  | Obligatoire                                      |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| type                     | La propriété **type** sous `storeSettings` doit être définie sur **AzureBlobStorageReadSettings**. | Oui                                           |
| **_Rechercher les fichiers à copier :_* _ |  |  |
| OPTION 1 : chemin d’accès statique<br> | Copie à partir du conteneur donné ou du chemin d’accès au dossier/fichier spécifié dans le jeu de données. Si vous souhaitez copier tous les blobs d’un conteneur ou d’un dossier, spécifiez en plus `wildcardFileName` comme `_`. |  |
| OPTION 2 : préfixe blob<br>- prefix | Préfixe du nom d’objet blob sous le conteneur donné configuré dans un jeu de données pour filtrer les objets blob sources. Les blobs dont le nom commence par `container_in_dataset/this_prefix` sont sélectionnés. Il utilise le filtre côté service pour le stockage d’objets blob, qui offre de meilleures performances qu’un filtre de caractères génériques.<br><br>Quand vous utilisez le préfixe et que vous choisissez de copier le récepteur basé sur un fichier avec conservation de la hiérarchie, notez que le sous-chemin après le dernier signe « / » dans le préfixe est conservé. Par exemple, si vous avez la source `container/folder/subfolder/file.txt` et que vous configurez le préfixe sous la forme `folder/sub`, le chemin du fichier conservé est `subfolder/file.txt`. | Non                                                          |
| OPTION 3 : caractère générique<br>- wildcardFolderPath | Chemin d’accès du dossier avec des caractères génériques sous le conteneur donné configuré dans un jeu de données pour filtrer les dossiers sources. <br>Les caractères génériques autorisés sont les suivants : `*` (correspond à zéro caractère ou plusieurs) et `?` (correspond à zéro ou un caractère). Utilisez `^` comme caractère d’échappement si le nom de votre dossier contient des caractères génériques ou ce caractère d’échappement. <br>Consultez d’autres exemples dans les [exemples de filtre de dossier et de fichier](#folder-and-file-filter-examples). | Non                                            |
| OPTION 3 : caractère générique<br>- wildcardFileName | Nom de fichier avec caractères génériques sous le conteneur donné et chemin d’accès du dossier (ou chemin d’accès du dossier en caractères génériques) pour filtrer les fichiers sources. <br>Les caractères génériques autorisés sont les suivants : `*` (correspond à zéro caractère ou plusieurs) et `?` (correspond à zéro ou un caractère). Utilisez `^` comme caractère d’échappement si le nom de votre fichier contient un caractère générique ou ce caractère d’échappement. Consultez d’autres exemples dans les [exemples de filtre de dossier et de fichier](#folder-and-file-filter-examples). | Oui |
| OPTION 4 : liste de fichiers<br>- fileListPath | Indique de copier un ensemble de fichiers donné. Pointez vers un fichier texte contenant la liste des fichiers que vous voulez copier, un fichier par ligne indiquant le chemin d’accès relatif configuré dans le jeu de données.<br/>Lorsque vous utilisez cette option, ne spécifiez pas de nom de fichier dans le jeu de données. Pour plus d’exemples, consultez [Exemples de listes de fichiers](#file-list-examples). |Non |
| ***Paramètres supplémentaires :** _ |  | |
| recursive | Indique si les données sont lues de manière récursive à partir des sous-dossiers ou uniquement du dossier spécifié. Notez que lorsque l’option _ *recursive** est définie sur **true** et que le récepteur est un magasin basé sur un fichier, un dossier ou un sous-dossier vide n’est pas copié ou créé sur le récepteur. <br>Les valeurs autorisées sont **true** (par défaut) et **false**.<br>Cette propriété ne s’applique pas lorsque vous configurez `fileListPath`. |Non |
| deleteFilesAfterCompletion | Indique si les fichiers binaires seront supprimés du magasin source après leur déplacement vers le magasin de destination. La suppression se faisant par fichier, lorsque l’activité de copie échoue, vous pouvez constater que certains fichiers ont déjà été copiés vers la destination et supprimés de la source, tandis que d’autres restent dans le magasin source. <br/>Cette propriété est valide uniquement dans un scénario de copie de fichiers binaires. La valeur par défaut est false. |Non |
| modifiedDatetimeStart    | Les fichiers sont filtrés en fonction de l’attribut de dernière modification. <br>Les fichiers seront sélectionnés si l’heure de leur dernière modification d’inscrit dans l’intervalle de temps compris entre `modifiedDatetimeStart` et `modifiedDatetimeEnd`. L’heure est appliquée à un fuseau horaire UTC au format « 2018-12-01T05:00:00Z ». <br> Les propriétés peuvent avoir la valeur **NULL**, ce qui a pour effet qu’aucun filtre d’attribut de fichier n’est appliqué au jeu de données.  Quand `modifiedDatetimeStart` a une valeur de DateHeure, mais que la valeur de `modifiedDatetimeEnd` est **NULL**, les fichiers dont l’attribut de dernière modification a une valeur supérieure ou égale à la valeur de DateHeure sont sélectionnés.  Quand `modifiedDatetimeEnd` a une valeur de DateHeure, mais que la valeur de `modifiedDatetimeStart` est **NULL**, les fichiers dont l’attribut de dernière modification a une valeur inférieure à la valeur de DateHeure sont sélectionnés.<br/>Cette propriété ne s’applique pas lorsque vous configurez `fileListPath`. | Non                                            |
| modifiedDatetimeEnd      | Identique à ce qui précède.                                               | Non                                            |
| enablePartitionDiscovery | Pour les fichiers partitionnés, spécifiez s’il faut analyser les partitions à partir du chemin d’accès au fichier et les ajouter en tant que colonnes sources supplémentaires.<br/>Les valeurs autorisées sont **false** (par défaut) et **true**. | Non                                            |
| partitionRootPath | Lorsque la découverte de partition est activée, spécifiez le chemin d’accès racine absolu pour pouvoir lire les dossiers partitionnés en tant que colonnes de données.<br/><br/>S’il n’est pas spécifié, par défaut :<br/>– Quand vous utilisez le chemin d’accès du fichier dans le jeu de données ou la liste des fichiers sur la source, le chemin racine de la partition est le chemin d’accès configuré dans le jeu de données.<br/>– Quand vous utilisez le filtre de dossiers de caractères génériques, le chemin d’accès racine de la partition est le sous-chemin d’accès avant le premier caractère générique.<br/>– Quand vous utilisez le préfixe, le chemin d’accès racine de la partition est le sous-chemin d’accès avant le dernier « / ». <br/><br/>Par exemple, en supposant que vous configurez le chemin d’accès dans le jeu de données en tant que « root/folder/year=2020/month=08/day=27 » :<br/>– Si vous spécifiez le chemin d’accès racine de la partition en tant que « root/folder/year=2020 », l’activité de copie génère deux colonnes supplémentaires, `month` et `day`, ayant respectivement la valeur « 08 » et « 27 », en plus des colonnes contenues dans les fichiers.<br/>– Si le chemin d’accès racine de la partition n’est pas spécifié, aucune colonne supplémentaire n’est générée. | Non                                            |
| maxConcurrentConnections | Nombre de connexions simultanées au stockage. Ne le spécifiez que si vous souhaitez limiter les connexions simultanées au magasin de données. | Non                                            |

> [!NOTE]
> Pour les formats Parquet et de texte délimité, le type **BlobSource** pour la source de l’activité Copy mentionnée dans la section suivante est toujours pris en charge à des fins de compatibilité descendante. Nous vous suggérons d’utiliser le nouveau modèle jusqu’à ce que l’interface utilisateur de création de Data Factory ait basculé vers la génération de ces nouveaux types.

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromBlob",
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
                    "type": "AzureBlobStorageReadSettings",
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

### <a name="blob-storage-as-a-sink-type"></a>Stockage Blob en tant que type de récepteur

[!INCLUDE [data-factory-v2-file-sink-formats](../../includes/data-factory-v2-file-sink-formats.md)] 

Les propriétés suivantes sont prises en charge pour le stockage d’objets blob Azure sous les paramètres `storeSettings` dans un récepteur de copie basé sur un format :

| Propriété                 | Description                                                  | Obligatoire |
| ------------------------ | ------------------------------------------------------------ | -------- |
| type                     | La propriété **type** sous `storeSettings` doit être définie sur **AzureBlobStorageWriteSettings**. | Oui      |
| copyBehavior             | Définit le comportement de copie lorsque la source est constituée de fichiers d’une banque de données basée sur un fichier.<br/><br/>Les valeurs autorisées sont les suivantes :<br/><b>- PreserveHierarchy (par défaut)</b> : conserve la hiérarchie des fichiers dans le dossier cible. Le chemin relatif du fichier source vers le dossier source est identique au chemin relatif du fichier cible vers le dossier cible.<br/><b>- FlattenHierarchy</b> : tous les fichiers du dossier source figurent dans le premier niveau du dossier cible. Les noms des fichiers cibles sont générés automatiquement. <br/><b>- MergeFiles</b> : fusionne tous les fichiers du dossier source dans un seul fichier. Si le nom d’objet blob ou de fichier est spécifié, le nom de fichier fusionné est le nom spécifié. Dans le cas contraire, il s’agit d’un nom de fichier généré automatiquement. | Non       |
| blockSizeInMB | Spécifiez la taille du bloc, en Mo, qui est utilisée pour écrire des données dans des objets blobs de blocs. En savoir plus sur les [objets blobs de blocs](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-block-blobs). <br/>Les valeurs valides sont *comprises entre 4 et 100 Mo*. <br/>Par défaut, Data Factory détermine automatiquement la taille de bloc en fonction du type et des données de votre magasin source. Pour une copie non binaire dans un stockage d’objets blob, la taille de bloc par défaut est de 100 Mo, ce qui permet de stocker jusqu’à 4,95 To de données. Cela peut ne pas être optimal si vos données ne sont pas volumineuses, en particulier si vous utilisez le runtime d’intégration auto-hébergé avec des connexions réseau médiocres qui entraînent des problèmes de délai d’expiration d’opération ou de performances. Vous pouvez spécifier explicitement une taille de bloc, tout en veillant à ce que `blockSizeInMB*50000` soit suffisamment grand pour stocker les données. Dans le cas contraire, l’exécution de l’activité Copy échoue. | Non |
| maxConcurrentConnections | Nombre de connexions simultanées au stockage. Ne le spécifiez que si vous souhaitez limiter les connexions simultanées au magasin de données. | Non       |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromBlob",
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
                    "type": "AzureBlobStorageWriteSettings",
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
| `container/Folder*` | (vide, utiliser la valeur par défaut) | false | conteneur<br/>&nbsp;&nbsp;&nbsp;&nbsp;DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |
| `container/Folder*` | (vide, utiliser la valeur par défaut) | true | conteneur<br/>&nbsp;&nbsp;&nbsp;&nbsp;DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |
| `container/Folder*` | `*.csv` | false | conteneur<br/>&nbsp;&nbsp;&nbsp;&nbsp;DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |
| `container/Folder*` | `*.csv` | true | conteneur<br/>&nbsp;&nbsp;&nbsp;&nbsp;DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |

### <a name="file-list-examples"></a>Exemples de liste de fichiers

Cette section décrit le comportement résultant de l’utilisation d’un chemin d’accès de liste de fichiers dans la source de l’activité Copy.

Supposons que vous disposez de la structure de dossiers sources suivante et que vous souhaitez copier les fichiers en gras :

| Exemple de structure source                                      | Contenu de FileListToCopy.txt                             | Configuration de Data Factory                                            |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| conteneur<br/>&nbsp;&nbsp;&nbsp;&nbsp;DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Métadonnées<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FileListToCopy.txt | File1.csv<br>Subfolder1/File3.csv<br>Subfolder1/File5.csv | **Dans le jeu de données :**<br>- Conteneur : `container`<br>- chemin d’accès du dossier : `FolderA`<br><br>**Dans la source de l’activité Copy :**<br>- chemin d’accès à la liste de fichiers : `container/Metadata/FileListToCopy.txt` <br><br>Le chemin d’accès de la liste de fichiers pointe vers un fichier texte dans le même magasin de données, qui contient la liste de fichiers que vous voulez copier, un fichier par ligne indiquant le chemin d’accès relatif configuré dans le jeu de données. |

### <a name="some-recursive-and-copybehavior-examples"></a>Quelques exemples de valeurs recursive et copyBehavior

Cette section décrit le comportement résultant de l’opération de copie pour différentes combinaisons de valeurs **recursive** et **copyBehavior**.

| recursive | copyBehavior | Structure du dossier source | Cible obtenue |
|:--- |:--- |:--- |:--- |
| true |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 | Le dossier cible, Dossier1, est créé avec la même structure que la source :<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 |
| true |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 | Le dossier cible, Dossier1, est créé avec la structure suivante : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier5 |
| true |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 | Le dossier cible, Dossier1, est créé avec la structure suivante : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Le contenu de Fichier1 + Fichier2 + Fichier3 + Fichier4 + Fichier5 est fusionné dans un fichier avec un nom de fichier généré automatiquement. |
| false |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 | Le dossier cible, Dossier1, est créé avec la structure suivante : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |
| false |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 | Le dossier cible, Dossier1, est créé avec la structure suivante : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier2<br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |
| false |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5 | Le dossier cible, Dossier1, est créé avec la structure suivante :<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Le contenu de Fichier1 + Fichier2 est fusionné dans un fichier avec un nom de fichier généré automatiquement. nom généré automatiquement pour Fichier1<br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |

## <a name="preserving-metadata-during-copy"></a>Conservation des métadonnées lors de la copie

Lorsque vous copiez des fichiers à partir d’Amazon S3, d’un stockage d’objets blob Azure ou d’Azure Data Lake Storage Gen2 vers Azure Data Lake Storage Gen2 ou un stockage d’objets blob Azure, vous pouvez choisir de conserver les métadonnées des fichiers avec les données. Pour plus d’informations, consultez [Conserver les métadonnées](copy-activity-preserve-metadata.md#preserve-metadata).

## <a name="mapping-data-flow-properties"></a>Propriétés du mappage de flux de données

Lorsque vous transformez des données en flux de données de mappage, vous pouvez lire et écrire des fichiers à partir du stockage d’objets blob Azure aux formats suivants :
* [Avro](format-avro.md#mapping-data-flow-properties)
* [Texte délimité](format-delimited-text.md#mapping-data-flow-properties)
* [Delta](format-delta.md#mapping-data-flow-properties)
* [Excel](format-excel.md#mapping-data-flow-properties)
* [JSON](format-json.md#mapping-data-flow-properties)
* [Parquet](format-parquet.md#mapping-data-flow-properties)

Les paramètres spécifiques du format se trouvent dans la documentation de ce format. Pour plus d’informations, consultez [Transformation de source en flux de données de mappage](data-flow-source.md) et [Transformation de récepteur en flux de données de mappage](data-flow-sink.md).

### <a name="source-transformation"></a>Transformation de la source

Dans une transformation de source, vous pouvez lire à partir d’un conteneur, d’un dossier ou d’un fichier individuel dans un stockage d’objets blob Azure. Utilisez l’onglet **Options de la source** pour gérer la façon dont les fichiers sont lus. 

![Options de la source](media/data-flow/sourceOptions1.png "Options de la source")

**Chemin avec des caractères génériques :** L’utilisation d’un modèle à caractères génériques donne pour instruction à Data Factory de lire en boucle chaque dossier et fichier correspondant dans une même transformation de source. Il s’agit d’un moyen efficace de traiter plusieurs fichiers dans un seul et même flux. Ajoutez plusieurs modèles de correspondance à caractères génériques avec le signe plus qui apparaît quand vous pointez sur votre modèle à caractères génériques existant.

Dans le conteneur source, choisissez une série de fichiers qui correspondent à un modèle. Seul un conteneur peut être spécifié dans le jeu de données. Votre chemin contenant des caractères génériques doit donc également inclure le chemin de votre dossier à partir du dossier racine.

Exemples de caractères génériques :

* ```*``` Représente un jeu de caractères quelconque.
* ```**``` Représente une imbrication de répertoires récursifs.
* ```?``` Remplace un caractère.
* ```[]``` Cherche une correspondance avec le ou les caractères entre crochets.

* ```/data/sales/**/*.csv``` Obtient tous les fichiers .csv se trouvant sous /data/sales.
* ```/data/sales/20??/**/``` Obtient tous les fichiers datés du XXe siècle.
* ```/data/sales/*/*/*.csv``` Obtient les fichiers .csv à deux niveaux sous /data/sales.
* ```/data/sales/2004/*/12/[XY]1?.csv``` Obtient tous les fichiers .csv datés de décembre 2004, commençant par X ou Y et ayant comme préfixe un nombre à deux chiffres.

**Chemin racine de la partition :** Si vous avez partitionné des dossiers dans votre source de fichier avec un format ```key=value``` (par exemple, `year=2019`), vous pouvez attribuer le niveau supérieur de cette arborescence de dossiers de partitions à un nom de colonne dans votre flux de données.

Tout d’abord, définissez un caractère générique pour inclure tous les chemins d’accès des dossiers partitionnés, ainsi que des fichiers de nœud terminal que vous souhaitez lire.

![Paramètres du fichier source de partition](media/data-flow/partfile2.png "Paramètre du fichier de partition")

Utilisez le paramètre **Chemin racine de la partition** pour définir le niveau supérieur de la structure de dossiers. Quand vous affichez le contenu de vos données via un aperçu des données, vous voyez que Data Factory ajoute les partitions résolues trouvées dans chacun de vos niveaux de dossiers.

![Chemin racine de la partition](media/data-flow/partfile1.png "Aperçu du chemin racine de la partition")

**Liste de fichiers :** Il s’agit d’un ensemble de fichiers. Créez un fichier texte qui inclut une liste de fichiers avec chemin relatif à traiter. Pointez sur ce fichier texte.

**Colonne où stocker le nom du fichier :** Stockez le nom du fichier source dans une colonne de vos données. Entrez un nouveau nom de colonne pour stocker la chaîne de nom de fichier.

**Après l’exécution :** après l’exécution du flux de données, choisissez de ne rien faire avec le fichier source, de le supprimer ou de le déplacer. Pour le déplacement, les chemins sont des chemins relatifs.

Pour déplacer les fichiers sources vers un autre emplacement de post-traitement, sélectionnez tout d’abord « Déplacer » comme opération de fichier. Définissez ensuite le répertoire de provenance (« from »). Si vous n’utilisez pas de caractères génériques pour votre chemin, le paramètre « from » sera le même dossier que votre dossier source.

Si vous avez un chemin d’accès source contenant un caractère générique, votre syntaxe se présente comme suit :

```/data/sales/20??/**/*.csv```

Vous pouvez spécifier « from » comme suit :

```/data/sales```

Et vous pouvez spécifier « to » comme suit :

```/backup/priorSales```

Dans le cas présent, tous les fichiers qui provenaient de /data/sales sont déplacés dans /backup/priorSales.

> [!NOTE]
> Les opérations de fichier s’exécutent uniquement quand vous démarrez le flux de données à partir d’une exécution de pipeline (débogage ou exécution) qui utilise l’activité Exécuter le flux de données dans un pipeline. Les opérations de fichiers ne s’exécutent *pas* en mode de débogage de flux de données.

**Filtrer par date de dernière modification :** Vous pouvez filtrer les fichiers traités en spécifiant une plage de dates sur laquelle les fichiers ont été modifiés pour la dernière fois. Toutes les valeurs de DateHeure sont exprimées en temps universel coordonné (UTC). 

### <a name="sink-properties"></a>Propriétés du récepteur

Dans la transformation de récepteur, vous pouvez écrire dans un conteneur ou un dossier dans le stockage d’objets blob Azure. L’onglet **Paramètres** vous permet de gérer la façon dont les fichiers sont écrits.

![Options du récepteur](media/data-flow/file-sink-settings.png "Options du récepteur")

**Effacer le contenu du dossier :** Détermine si le contenu du dossier de destination doit être effacé avant l’écriture des données.

**Option de nom de fichier :** Détermine la façon dont les fichiers de destination sont nommés dans le dossier de destination. Les options de nom de fichier sont les suivantes :
   * **Par défaut** : Autorisez Spark à nommer les fichiers en fonction des valeurs par défaut de la partition.
   * **Modèle** : Entrez un modèle qui énumère vos fichiers de sortie par partition. Par exemple, **loans[n].csv** crée loans1.csv, loans2.csv, etc.
   * **Par partition** : Entrez un nom de fichier pour chaque partition.
   * **Comme les données de la colonne** : Définissez le fichier de sortie sur la valeur d’une colonne. Le chemin est relatif au conteneur du jeu de données et non pas au dossier de destination. Si vous avez un chemin de dossier dans votre jeu de données, il sera remplacé.
   * **Sortie d’un seul fichier** : Combinez les fichiers de sortie partitionnés en un seul fichier nommé. Le chemin est relatif au dossier du jeu de données. Sachez que cette opération de fusion peut échouer en raison de la taille du nœud. Nous ne recommandons pas cette option pour les jeux de données volumineux.

**Tout mettre entre guillemets :** Détermine si toutes les valeurs doivent être placées entre guillemets.

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour en savoir plus sur les propriétés, consultez [Activité Lookup](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Propriétés de l’activité GetMetadata

Pour en savoir plus sur les propriétés, consultez [Activité GetMetadata](control-flow-get-metadata-activity.md). 

## <a name="delete-activity-properties"></a>Propriétés de l’activité Delete

Pour en savoir plus sur les propriétés, consultez [Activité Delete](delete-activity.md).

## <a name="legacy-models"></a>Modèles hérités

>[!NOTE]
>Les modèles suivants sont toujours pris en charge tels quels à des fins de compatibilité descendante. Nous vous suggérons d’utiliser le nouveau modèle mentionné précédemment. L’interface utilisateur de création de Data Factory a basculé vers la génération du nouveau modèle.

### <a name="legacy-dataset-model"></a>Modèle de jeu de données hérité

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** du jeu de données doit être définie sur **AzureBlob**. |Oui |
| folderPath | Chemin d’accès au conteneur et au dossier dans le stockage d’objets blob. <br/><br/>Le filtre de caractères génériques est pris en charge pour le chemin d’accès, à l’exclusion du nom du conteneur. Les caractères génériques autorisés sont les suivants : `*` (correspond à zéro caractère ou plusieurs) et `?` (correspond à zéro ou un caractère). Utilisez `^` comme caractère d’échappement si le nom de votre dossier contient un caractère générique ou ce caractère d’échappement. <br/><br/>Un exemple est myblobcontainer/myblobfolder/. Consultez d’autres exemples dans les [exemples de filtre de dossier et de fichier](#folder-and-file-filter-examples). |Oui pour l’activité Copy ou Lookup, non pour l’activité GetMetadata |
| fileName | Filtre de nom ou de caractères génériques pour les blobs sous la valeur **folderPath** spécifiée. Si vous ne spécifiez pas de valeur pour cette propriété, le jeu de données pointe vers tous les objets blob du dossier. <br/><br/>Pour le filtre, les caractères génériques autorisés sont les suivants : `*` (correspond à zéro caractère ou plus) et `?` (correspond à zéro ou un caractère).<br/>- Exemple 1 : `"fileName": "*.csv"`<br/>- Exemple 2 : `"fileName": "???20180427.txt"`<br/>Utilisez `^` comme caractère d’échappement si le nom de votre fichier contient un caractère générique ou ce caractère d’échappement.<br/><br/>Lorsque **fileName** n’est pas spécifié pour un jeu de données de sortie et que **preserveHierarchy** n’est pas spécifié dans le récepteur d’activité, l’activité Copy génère automatiquement le nom d’objet blob selon le modèle suivant : « *Data.[GUID d’exécution d’activité].[GUID si FlattenHierarchy].[format si configuré].[compression si configurée]*  ». Par exemple : « Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt.gz ». <br/><br/>Si vous effectuez la copie à partir d’une source tabulaire à l’aide d’un nom de tableau au lieu d’une requête, le modèle du nom est «  *[nom_tableau].[format].[compression si configurée]*  ». Par exemple : « MyTable.csv ». |Non |
| modifiedDatetimeStart | Les fichiers sont filtrés en fonction de l’attribut de dernière modification. Les fichiers seront sélectionnés si l’heure de leur dernière modification d’inscrit dans l’intervalle de temps compris entre `modifiedDatetimeStart` et `modifiedDatetimeEnd`. L’heure est appliquée au fuseau horaire UTC au format « 2018-12-01T05:00:00Z ». <br/><br/> N’oubliez pas que l’activation de ce paramètre affecte les performances globales du déplacement de données lorsque vous souhaitez filtrer d’énormes quantités de fichiers. <br/><br/> Les propriétés peuvent avoir la valeur **NULL**, ce qui a pour effet qu’aucun filtre d’attribut de fichier n’est appliqué au jeu de données.  Quand `modifiedDatetimeStart` a une valeur de DateHeure, mais que la valeur de `modifiedDatetimeEnd` est **NULL**, les fichiers dont l’attribut de dernière modification a une valeur supérieure ou égale à la valeur de DateHeure sont sélectionnés.  Quand `modifiedDatetimeEnd` a une valeur de DateHeure, mais que la valeur de `modifiedDatetimeStart` est **NULL**, les fichiers dont l’attribut de dernière modification a une valeur inférieure à la valeur de DateHeure sont sélectionnés.| Non |
| modifiedDatetimeEnd | Les fichiers sont filtrés en fonction de l’attribut de dernière modification. Les fichiers seront sélectionnés si l’heure de leur dernière modification d’inscrit dans l’intervalle de temps compris entre `modifiedDatetimeStart` et `modifiedDatetimeEnd`. L’heure est appliquée au fuseau horaire UTC au format « 2018-12-01T05:00:00Z ». <br/><br/> N’oubliez pas que l’activation de ce paramètre affecte les performances globales du déplacement de données lorsque vous souhaitez filtrer d’énormes quantités de fichiers. <br/><br/> Les propriétés peuvent avoir la valeur **NULL**, ce qui a pour effet qu’aucun filtre d’attribut de fichier n’est appliqué au jeu de données.  Quand `modifiedDatetimeStart` a une valeur de DateHeure, mais que la valeur de `modifiedDatetimeEnd` est **NULL**, les fichiers dont l’attribut de dernière modification a une valeur supérieure ou égale à la valeur de DateHeure sont sélectionnés.  Quand `modifiedDatetimeEnd` a une valeur de DateHeure, mais que la valeur de `modifiedDatetimeStart` est **NULL**, les fichiers dont l’attribut de dernière modification a une valeur inférieure à la valeur de DateHeure sont sélectionnés.| Non |
| format | Si vous souhaitez copier des fichiers en l’état entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie.<br/><br/>Si vous souhaitez analyser ou générer des fichiers dans un format spécifique, les types de format de fichier suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Définissez la propriété **type** située sous **Format** sur l’une de ces valeurs. Pour en savoir plus, voir les sections [Format Text](supported-file-formats-and-compression-codecs-legacy.md#text-format), [Format JSON](supported-file-formats-and-compression-codecs-legacy.md#json-format), [Format Avro](supported-file-formats-and-compression-codecs-legacy.md#avro-format), [Format Orc](supported-file-formats-and-compression-codecs-legacy.md#orc-format) et [Format Parquet](supported-file-formats-and-compression-codecs-legacy.md#parquet-format). |Non (uniquement pour un scénario de copie binaire) |
| compression | Spécifiez le type et le niveau de compression pour les données. Pour plus d’informations, voir [Formats de fichier et de codecs de compression pris en charge](supported-file-formats-and-compression-codecs-legacy.md#compression-support).<br/>Les types pris en charge sont : **GZip**, **Deflate**, **BZip2** et **ZipDeflate**.<br/>Les niveaux pris en charge sont **Optimal** et **Fastest**. |Non |

>[!TIP]
>Pour copier tous les objets blob d’un dossier, spécifiez **folderPath** uniquement.<br>Pour copier un seul objet blob avec un nom donné, spécifiez **folderPath** pour la partie dossier et **fileName** pour le nom du fichier.<br>Pour copier un sous-ensemble d’objets blob d’un dossier, spécifiez **folderPath** avec la partie dossier et **fileName** avec un filtre de caractères génériques. 

**Exemple :**

```json
{
    "name": "AzureBlobDataset",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": {
            "referenceName": "<Azure Blob storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "mycontainer/myfolder",
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

### <a name="legacy-source-model-for-the-copy-activity"></a>Modèle source hérité pour l’activité Copy

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** de la source de l’activité Copy doit être définie sur **BlobSource**. |Oui |
| recursive | Indique si les données sont lues de manière récursive à partir des sous-dossiers ou uniquement du dossier spécifié. Notez que lorsque l’option **recursive** est définie sur **true** et que le récepteur est un magasin basé sur un fichier, un dossier ou un sous-dossier vide n’est pas copié ou créé sur le récepteur.<br/>Les valeurs autorisées sont **true** (par défaut) et **false**. | Non |
| maxConcurrentConnections | Nombre de connexions simultanées au stockage. Ne le spécifiez que si vous souhaitez limiter les connexions simultanées au magasin de données. | Non |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromBlob",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure Blob input dataset name>",
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
                "type": "BlobSource",
                "recursive": true
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="legacy-sink-model-for-the-copy-activity"></a>Modèle de récepteur hérité pour l’activité Copy

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** du récepteur de l’activité Copy doit être définie sur **BlobSink**. |Oui |
| copyBehavior | Définit le comportement de copie lorsque la source est constituée de fichiers d’une banque de données basée sur un fichier.<br/><br/>Les valeurs autorisées sont les suivantes :<br/><b>- PreserveHierarchy (par défaut)</b> : conserve la hiérarchie des fichiers dans le dossier cible. Le chemin d’accès relatif du fichier source vers le dossier source est identique au chemin d’accès relatif du fichier cible vers le dossier cible.<br/><b>- FlattenHierarchy</b> : tous les fichiers du dossier source figurent dans le premier niveau du dossier cible. Les noms des fichiers cibles sont générés automatiquement. <br/><b>- MergeFiles</b> : fusionne tous les fichiers du dossier source dans un seul fichier. Si le nom d’objet blob ou de fichier est spécifié, le nom de fichier fusionné est le nom spécifié. Dans le cas contraire, il s’agit d’un nom de fichier généré automatiquement. | Non |
| maxConcurrentConnections | Nombre de connexions simultanées au stockage. Ne le spécifiez que si vous souhaitez limiter les connexions simultanées au magasin de données. | Non |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyToBlob",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Azure Blob output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "BlobSink",
                "copyBehavior": "PreserveHierarchy"
            }
        }
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir la liste des magasins de données pris en charge par l’activité Copy dans Data Factory en tant que sources et récepteurs, consultez [Magasins de données pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).