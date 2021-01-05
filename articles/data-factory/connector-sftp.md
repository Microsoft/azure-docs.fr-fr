---
title: Copier des données depuis et vers un serveur SFTP
description: Découvrez comment copier des données depuis et vers un serveur SFTP à l’aide d’Azure Data Factory.
services: data-factory
documentationcenter: ''
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/28/2020
ms.openlocfilehash: f4b78c6cb2af8d18dc761e9bfc78740a845f54fc
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91252933"
---
# <a name="copy-data-from-and-to-the-sftp-server-by-using-azure-data-factory"></a>Copier des données depuis et vers le serveur SFTP à l’aide d’Azure Data Factory

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-sftp-connector.md)
> * [Version actuelle](connector-sftp.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article explique comment copier des données depuis et vers le serveur FTP sécurisé (serveur SFTP). Pour en savoir plus sur Azure Data Factory, lisez l’[article d’introduction](introduction.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Le connecteur SFTP est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)
- [Activité GetMetadata](control-flow-get-metadata-activity.md)
- [Supprimer l’activité](delete-activity.md)

Plus précisément, le connecteur SFTP prend en charge les opérations suivantes :

- Copie de fichiers depuis et vers le serveur SFTP en utilisant l’authentification *De base* ou *SshPublicKey*.
- Copie de fichiers en l’état, ou en analysant ou en générant les fichiers avec les [formats de fichier et codecs de compression pris en charge](supported-file-formats-and-compression-codecs.md).

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

## <a name="get-started"></a>Bien démarrer

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes fournissent des informations sur les propriétés utilisées pour définir les entités Data Factory spécifiques de SFTP.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés suivantes sont prises en charge pour le service lié SFTP :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur *Sftp*. |Oui |
| host | Nom ou adresse IP du serveur SFTP. |Oui |
| port | Port sur lequel le serveur SFTP est à l’écoute.<br/>La valeur autorisée est un entier et la valeur par défaut est *22*. |Non |
| skipHostKeyValidation | Spécifiez s’il faut ignorer la validation de la clé hôte.<br/>Les valeurs autorisées sont *True* et *False* (par défaut).  | Non |
| hostKeyFingerprint | Spécifiez l’empreinte de la clé hôte. | Oui, si la valeur de « skipHostKeyValidation » est définie sur false.  |
| authenticationType | Spécifiez le type d’authentification.<br/>Les valeurs autorisées sont *De base* et *SshPublicKey*. Pour plus d’informations sur les propriétés, consultez la section [Utiliser une authentification de base](#use-basic-authentication). Pour obtenir des exemples JSON, consultez la section [Utiliser l’authentification par clé publique SSH](#use-ssh-public-key-authentication). |Oui |
| connectVia | Le [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Pour en savoir plus, consultez la section [Conditions préalables](#prerequisites). Si le runtime d’intégration n’est pas spécifié, le service utilise le runtime d’intégration Azure par défaut. |Non |

### <a name="use-basic-authentication"></a>Utiliser une authentification de base

Pour utiliser l’authentification de base, définissez la propriété *authenticationType* sur *De base* et spécifiez les propriétés suivantes en plus des propriétés génériques du connecteur SFTP présentées dans la section précédente :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| userName | Utilisateur ayant accès au serveur SFTP. |Oui |
| mot de passe | Mot de passe de l’utilisateur (userName). Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité dans votre fabrique de données, ou [référencez un secret stocké dans un coffre de clés Azure](store-credentials-in-key-vault.md). | Oui |

**Exemple :**

```json
{
    "name": "SftpLinkedService",
    "type": "linkedservices",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<sftp server>",
            "port": 22,
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "authenticationType": "Basic",
            "userName": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of integration runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="use-ssh-public-key-authentication"></a>Utiliser l’authentification par clé publique SSH

Pour utiliser l’authentification par clé publique SSH, définissez la propriété « authenticationType » sur **De base** et spécifiez les propriétés suivantes en plus des propriétés génériques du connecteur SFTP présentées dans la dernière section :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| userName | Utilisateur ayant accès au serveur SFTP. |Oui |
| privateKeyPath | Spécifiez le chemin absolu au fichier de clé privée auquel le runtime d’intégration peut accéder. Cela s’applique uniquement quand le type auto-hébergé du runtime d’intégration est spécifié dans « connectVia ». | Spécifiez `privateKeyPath` ou `privateKeyContent`.  |
| privateKeyContent | Contenu de clé privée SSH encodé en Base64. La clé privée SSH doit être au format OpenSSH. Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité dans votre fabrique de données, ou [référencez un secret stocké dans un coffre de clés Azure](store-credentials-in-key-vault.md). | Spécifiez `privateKeyPath` ou `privateKeyContent`. |
| passPhrase | Spécifiez la phrase secrète ou le mot de passe pour déchiffrer la clé privée si le fichier de clé ou le contenu de clé est protégé par une phrase secrète. Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité dans votre fabrique de données, ou [référencez un secret stocké dans un coffre de clés Azure](store-credentials-in-key-vault.md). | Oui, si le fichier de clé privée ou le contenu de clé est protégé par une phrase secrète. |

> [!NOTE]
> Le connecteur SFTP prend en charge une clé OpenSSH RSA/DSA. Assurez-vous que le contenu de votre fichier de clé commence par « -----BEGIN [RSA/DSA] PRIVATE KEY----- ». Si le fichier de clé privée est un fichier au format PPK, utilisez l’outil PuTTY pour effectuer la conversion du format PPK au format OpenSSH. 

**Exemple 1 : Authentification SshPublicKey avec un chemin de fichier de clé privée**

```json
{
    "name": "SftpLinkedService",
    "type": "Linkedservices",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<sftp server>",
            "port": 22,
            "skipHostKeyValidation": true,
            "authenticationType": "SshPublicKey",
            "userName": "xxx",
            "privateKeyPath": "D:\\privatekey_openssh",
            "passPhrase": {
                "type": "SecureString",
                "value": "<pass phrase>"
            }
        },
        "connectVia": {
            "referenceName": "<name of integration runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Exemple 2 : Authentification SshPublicKey avec un contenu de clé privée**

```json
{
    "name": "SftpLinkedService",
    "type": "Linkedservices",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<sftp server>",
            "port": 22,
            "skipHostKeyValidation": true,
            "authenticationType": "SshPublicKey",
            "userName": "<username>",
            "privateKeyContent": {
                "type": "SecureString",
                "value": "<base64 string of the private key content>"
            },
            "passPhrase": {
                "type": "SecureString",
                "value": "<pass phrase>"
            }
        },
        "connectVia": {
            "referenceName": "<name of integration runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Jeux de données](concepts-datasets-linked-services.md). 

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Les propriétés suivantes sont prises en charge pour SFTP sous les paramètres `location` dans le jeu de données basé sur le format :

| Propriété   | Description                                                  | Obligatoire |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | La propriété *type* sous `location` dans le jeu de données doit être définie sur *SftpLocation*. | Oui      |
| folderPath | Chemin du dossier. Si vous souhaitez utiliser un caractère générique pour filtrer le dossier, ignorez ce paramètre et spécifiez le chemin dans les paramètres de la source de l’activité. | Non       |
| fileName   | Nom de fichier sous le chemin folderPath spécifié. Si vous souhaitez utiliser un caractère générique pour filtrer les fichiers, ignorez ce paramètre et spécifiez le nom de fichier dans les paramètres de la source de l’activité. | Non       |

**Exemple :**

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<SFTP linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "SftpLocation",
                "folderPath": "root/folder/subfolder"
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

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par la source SFTP.

### <a name="sftp-as-source"></a>SFTP en tant que source

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Les propriétés suivantes sont prises en charge pour SFTP sous les paramètres `storeSettings` dans la source de copie basée sur le format :

| Propriété                 | Description                                                  | Obligatoire                                      |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| type                     | La propriété *type* sous `storeSettings` doit être définie sur *SftpReadSettings*. | Oui                                           |
| ***Rechercher les fichiers à copier*** |  |  |
| OPTION 1 : chemin d’accès statique<br> | Copiez à partir du chemin de dossier/fichier spécifié dans le jeu de données. Si vous souhaitez copier tous les fichiers d’un dossier, spécifiez en plus `wildcardFileName` comme `*`. |  |
| OPTION 2 : caractère générique<br>- wildcardFolderPath | Chemin d’accès du dossier avec des caractères génériques pour filtrer les dossiers sources. <br>Les caractères génériques autorisés sont `*` (correspond à zéro, un ou plusieurs caractères) et `?` (correspond à zéro ou un caractère) ; utilisez `^` en guise d’échappement si votre nom de dossier contient effectivement un caractère générique ou ce caractère d’échappement. <br>Pour d’autres exemples, consultez [Exemples de filtres de dossier et de fichier](#folder-and-file-filter-examples). | Non                                            |
| OPTION 2 : caractère générique<br>- wildcardFileName | Nom du fichier avec des caractères génériques situé dans le chemin folderPath/wildcardFolderPath spécifié pour filtrer les fichiers sources. <br>Les caractères génériques autorisés sont `*` (correspond à zéro, un ou plusieurs caractères) et `?` (correspond à zéro ou un caractère) ; utilisez `^` en guise d’échappement si votre nom de dossier contient effectivement un caractère générique ou ce caractère d’échappement.  Pour d’autres exemples, consultez [Exemples de filtres de dossier et de fichier](#folder-and-file-filter-examples). | Oui |
| OPTION 3 : liste de fichiers<br>- fileListPath | Indique de copier un ensemble de fichiers spécifié. Pointez sur un fichier texte qui contient une liste de fichiers que vous voulez copier (un fichier par ligne, avec le chemin relatif au chemin configuré dans le jeu de données).<br/>Lorsque vous utilisez cette option, ne spécifiez pas le nom de fichier dans le jeu de données. Pour plus d’exemples, consultez [Exemples de listes de fichiers](#file-list-examples). |Non |
| ***Paramètres supplémentaires*** |  | |
| recursive | Indique si les données sont lues de manière récursive à partir des sous-dossiers ou uniquement du dossier spécifié. Lorsque l’option « recursive » est définie sur true et que le récepteur est un magasin basé sur un fichier, un dossier vide ou un sous-dossier n’est pas copié ou créé sur le récepteur. <br>Les valeurs autorisées sont *true* (par défaut) et *false*.<br>Cette propriété ne s’applique pas lorsque vous configurez `fileListPath`. |Non |
| deleteFilesAfterCompletion | Indique si les fichiers binaires seront supprimés du magasin source après leur déplacement vers le magasin de destination. La suppression se faisant par fichier, lorsque l’activité de copie échoue, vous pouvez constater que certains fichiers ont déjà été copiés vers la destination et supprimés de la source, tandis que d’autres restent dans le magasin source. <br/>Cette propriété est valide uniquement dans un scénario de copie de fichiers binaires. La valeur par défaut est false. |Non |
| modifiedDatetimeStart    | Les fichiers sont filtrés en fonction de l’attribut *Dernière modification*. <br>Les fichiers sont sélectionnés si leur heure de dernière modification se trouve dans l’intervalle situé entre `modifiedDatetimeStart` et `modifiedDatetimeEnd`. L’heure est appliquée au fuseau horaire UTC au format *2018-12-01T05:00:00Z*. <br> Les propriétés peuvent être NULL, ce qui signifie qu’aucun filtre d’attribut de fichier n’est appliqué au jeu de données.  Lorsque `modifiedDatetimeStart` a une valeur DateHeure, mais que `modifiedDatetimeEnd` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est supérieur ou égal à la valeur DateHeure sont sélectionnés.  Lorsque `modifiedDatetimeEnd` a une valeur DateHeure, mais que `modifiedDatetimeStart` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est inférieur à la valeur DateHeure sont sélectionnés.<br/>Cette propriété ne s’applique pas lorsque vous configurez `fileListPath`. | Non                                            |
| modifiedDatetimeEnd      | Identique à ce qui précède.                                               | Non                                            |
| enablePartitionDiscovery | Pour les fichiers partitionnés, spécifiez s’il faut analyser les partitions à partir du chemin d’accès au fichier et les ajouter en tant que colonnes sources supplémentaires.<br/>Les valeurs autorisées sont **false** (par défaut) et **true**. | Non                                            |
| partitionRootPath | Lorsque la découverte de partition est activée, spécifiez le chemin d’accès racine absolu pour pouvoir lire les dossiers partitionnés en tant que colonnes de données.<br/><br/>S’il n’est pas spécifié, par défaut :<br/>– Quand vous utilisez le chemin d’accès du fichier dans le jeu de données ou la liste des fichiers sur la source, le chemin racine de la partition est le chemin d’accès configuré dans le jeu de données.<br/>– Quand vous utilisez le filtre de dossiers de caractères génériques, le chemin d’accès racine de la partition est le sous-chemin d’accès avant le premier caractère générique.<br/><br/>Par exemple, en supposant que vous configurez le chemin d’accès dans le jeu de données en tant que « root/folder/year=2020/month=08/day=27 » :<br/>– Si vous spécifiez le chemin d’accès racine de la partition en tant que « root/folder/year=2020 », l’activité de copie génère deux colonnes supplémentaires, `month` et `day`, ayant respectivement la valeur « 08 » et « 27 », en plus des colonnes contenues dans les fichiers.<br/>– Si le chemin d’accès racine de la partition n’est pas spécifié, aucune colonne supplémentaire n’est générée. | Non                                            |
| maxConcurrentConnections | Nombre de connexions simultanées possibles au magasin de stockage. Spécifiez une valeur uniquement lorsque vous souhaitez limiter le nombre de connexions simultanées au magasin de données. | Non                                            |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromSFTP",
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
                    "type": "SftpReadSettings",
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

### <a name="sftp-as-a-sink"></a>SFTP en tant que récepteur

[!INCLUDE [data-factory-v2-file-sink-formats](../../includes/data-factory-v2-file-sink-formats.md)]

Les propriétés suivantes sont prises en charge pour SFTP sous les paramètres `storeSettings` dans un récepteur de copie basée sur le format :

| Propriété                 | Description                                                  | Obligatoire |
| ------------------------ | ------------------------------------------------------------ | -------- |
| type                     | La propriété *type* sous `storeSettings` doit être définie sur *SftpWriteSettings*. | Oui      |
| copyBehavior             | Définit le comportement de copie lorsque la source est constituée de fichiers d’une banque de données basée sur un fichier.<br/><br/>Les valeurs autorisées sont les suivantes :<br/><b>- PreserveHierarchy (par défaut)</b> : conserve la hiérarchie des fichiers dans le dossier cible. Le chemin relatif du fichier source vers le dossier source est identique au chemin relatif du fichier cible vers le dossier cible.<br/><b>- FlattenHierarchy</b> : tous les fichiers du dossier source figurent dans le premier niveau du dossier cible. Les noms des fichiers cibles sont générés automatiquement. <br/><b>- MergeFiles</b> : fusionne tous les fichiers du dossier source dans un seul fichier. Si le nom de fichier est spécifié, le nom de fichier fusionné est le nom spécifié. Dans le cas contraire, il s’agit d’un nom de fichier généré automatiquement. | Non       |
| maxConcurrentConnections | Nombre de connexions simultanées possibles au magasin de stockage. Spécifiez une valeur uniquement lorsque vous souhaitez limiter le nombre de connexions simultanées au magasin de données. | Non       |
| useTempFileRename | Indiquez si vous souhaitez effectuer un chargement dans des fichiers temporaires puis les renommer, ou si vous souhaitez écrire directement dans l’emplacement de dossier ou de fichier cible. Par défaut, Azure Data Factory écrit d’abord dans des fichiers temporaires, puis les renomme une fois le chargement terminé. Cette séquence permet (1) d’éviter les conflits susceptibles d’entraîner l’altération d’un fichier si d’autres processus écrivent dans le même fichier, et (2) de garantir l’existence de la version d’origine du fichier pendant le transfert. Si votre serveur SFTP ne prend pas en charge l’opération de renommage, désactivez cette option et vérifiez qu’aucun autre processus d’écriture n’est en cours sur le fichier cible. Pour plus d’informations, consultez le conseil de dépannage fourni après ce tableau. | Non. La valeur par défaut est *true*. |
| operationTimeout | Délai d’attente avant l’expiration de chaque demande d’écriture au serveur SFTP. La valeur par défaut est 60 minutes (01:00:00).|Non |

>[!TIP]
>Si vous recevez l’erreur « UserErrorSftpPathNotFound », « UserErrorSftpPermissionDenied » ou « SftpOperationFail » lorsque vous écrivez des données dans SFTP et que l’utilisateur SFTP que vous utilisez *dispose* des autorisations appropriées, vérifiez que l’opération de renommage du fichier de prise en charge de votre serveur SFTP fonctionne. Si ce n’est pas le cas, désactivez l’option **Charger avec le fichier temporaire** (`useTempFileRename`) et réessayez. Pour en savoir plus sur cette propriété, consultez le tableau précédent. Si vous utilisez un runtime d’intégration auto-hébergé pour l’activité de copie, veillez à utiliser la version 4.6 ou une version ultérieure.

**Exemple :**

```json
"activities":[
    {
        "name": "CopyToSFTP",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
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
                "type": "<source type>"
            },
            "sink": {
                "type": "BinarySink",
                "storeSettings":{
                    "type": "SftpWriteSettings",
                    "copyBehavior": "PreserveHierarchy"
                }
            }
        }
    }
]
```

### <a name="folder-and-file-filter-examples"></a>Exemples de filtres de dossier et de fichier

Cette section décrit le comportement résultant de l’utilisation de filtres de caractères génériques avec des chemins de dossiers et des noms de fichiers.

| folderPath | fileName | recursive | Structure du dossier source et résultat du filtrage (les fichiers en **gras** sont récupérés)|
|:--- |:--- |:--- |:--- |
| `Folder*` | (vide, utiliser la valeur par défaut) | false | DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5.csv<br/>AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |
| `Folder*` | (vide, utiliser la valeur par défaut) | true | DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier5.csv**<br/>AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |
| `Folder*` | `*.csv` | false | DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5.csv<br/>AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |
| `Folder*` | `*.csv` | true | DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier5.csv**<br/>AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |

### <a name="file-list-examples"></a>Exemples de liste de fichiers

Ce tableau décrit le comportement résultant de l’utilisation d’un chemin de liste de fichiers dans la source de l’activité de copie. Il suppose que vous disposez de la structure de dossiers sources suivante et que vous souhaitez copier les fichiers en gras :

| Exemple de structure source                                      | Contenu de FileListToCopy.txt                             | Configuration d’Azure Data Factory                                            |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| root<br/>&nbsp;&nbsp;&nbsp;&nbsp;DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Métadonnées<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FileListToCopy.txt | File1.csv<br>Subfolder1/File3.csv<br>Subfolder1/File5.csv | **Dans le jeu de données :**<br>- chemin d’accès du dossier : `root/FolderA`<br><br>**Dans la source de l’activité de copie :**<br>- chemin d’accès à la liste de fichiers : `root/Metadata/FileListToCopy.txt` <br><br>Le chemin d’accès de la liste de fichiers pointe vers un fichier texte dans le même magasin de données, qui contient la liste de fichiers que vous voulez copier (un fichier par ligne, avec le chemin relatif au chemin configuré dans le jeu de données). |

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour obtenir des informations sur les propriétés de l’activité de recherche (Lookup), consultez [Activité de recherche dans Azure Data Factory](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Propriétés de l’activité GetMetadata

Pour obtenir des informations sur les propriétés de l’activité d’obtention des métadonnées (GetMetadata), consultez [Activité d’obtention des métadonnées dans Azure Data Factory](control-flow-get-metadata-activity.md). 

## <a name="delete-activity-properties"></a>Propriétés de l’activité Delete

Pour obtenir des informations sur les propriétés de l’activité Delete, consultez [Activité Delete dans Azure Data Factory](delete-activity.md).

## <a name="legacy-models"></a>Modèles hérités

>[!NOTE]
>Les modèles suivants sont toujours pris en charge tels quels à des fins de compatibilité descendante. Nous vous recommandons d’utiliser le nouveau modèle abordé précédemment, car l’interface utilisateur de création Azure Data Factory a basculé vers la génération du nouveau modèle.

### <a name="legacy-dataset-model"></a>Modèle de jeu de données hérité

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété *type* du jeu de données doit être définie sur *FileShare*. |Oui |
| folderPath | Chemin du dossier. Un filtre de caractères génériques est pris en charge. Les caractères génériques autorisés sont `*` (correspond à zéro, un ou plusieurs caractères) et `?` (correspond à zéro ou un caractère) ; utilisez `^` en guise d’échappement si votre nom de fichier contient effectivement un caractère générique ou ce caractère d’échappement. <br/><br/>Exemples : dossier_racine/sous-dossier/ ; consultez d’autres exemples dans [Exemples de filtres de dossier et de fichier](#folder-and-file-filter-examples). |Oui |
| fileName |  **Filtre de noms ou de caractères génériques** pour les fichiers sous le chemin « folderPath » spécifié. Si vous ne spécifiez pas de valeur pour cette propriété, le jeu de données pointe vers tous les fichiers du dossier. <br/><br/>Pour le filtre, les caractères génériques autorisés sont `*` (correspond à zéro, un ou plusieurs caractères) et `?` (correspond à zéro ou un caractère).<br/>- Exemple 1 : `"fileName": "*.csv"`<br/>- Exemple 2 : `"fileName": "???20180427.txt"`<br/>Utilisez `^` comme caractère d'échappement si le nom réel de votre dossier contient des caractères génériques ou ce caractère d'échappement. |Non |
| modifiedDatetimeStart | Les fichiers sont filtrés en fonction de l’attribut *Dernière modification*. Les fichiers sont sélectionnés si leur heure de dernière modification se trouve dans l’intervalle situé entre `modifiedDatetimeStart` et `modifiedDatetimeEnd`. L’heure est appliquée au fuseau horaire UTC au format *2018-12-01T05:00:00Z*. <br/><br/> Les performances globales du déplacement des données sont affectées par l’activation de ce paramètre lorsque vous souhaitez filtrer des fichiers parmi un grand nombre de fichiers. <br/><br/> Les propriétés peuvent être NULL, ce qui signifie qu’aucun filtre d’attribut de fichier n’est appliqué au jeu de données.  Lorsque `modifiedDatetimeStart` a une valeur DateHeure, mais que `modifiedDatetimeEnd` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est supérieur ou égal à la valeur DateHeure sont sélectionnés.  Lorsque `modifiedDatetimeEnd` a une valeur DateHeure, mais que `modifiedDatetimeStart` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est inférieur à la valeur DateHeure sont sélectionnés.| Non |
| modifiedDatetimeEnd | Les fichiers sont filtrés en fonction de l’attribut *Dernière modification*. Les fichiers sont sélectionnés si leur heure de dernière modification se trouve dans l’intervalle situé entre `modifiedDatetimeStart` et `modifiedDatetimeEnd`. L’heure est appliquée au fuseau horaire UTC au format *2018-12-01T05:00:00Z*. <br/><br/> Les performances globales du déplacement des données sont affectées par l’activation de ce paramètre lorsque vous souhaitez filtrer des fichiers parmi un grand nombre de fichiers. <br/><br/> Les propriétés peuvent être NULL, ce qui signifie qu’aucun filtre d’attribut de fichier n’est appliqué au jeu de données.  Lorsque `modifiedDatetimeStart` a une valeur DateHeure, mais que `modifiedDatetimeEnd` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est supérieur ou égal à la valeur DateHeure sont sélectionnés.  Lorsque `modifiedDatetimeEnd` a une valeur DateHeure, mais que `modifiedDatetimeStart` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est inférieur à la valeur DateHeure sont sélectionnés.| Non |
| format | Si vous souhaitez copier des fichiers en l’état entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie.<br/><br/>Si vous voulez analyser des fichiers dans un format spécifique, les types de format de fichier suivants sont pris en charge : *TextFormat*, *JsonFormat*, *AvroFormat*, *OrcFormat* et *ParquetFormat*. Définissez la propriété *type* située sous Format sur l’une de ces valeurs. Pour plus d’informations, consultez les sections relatives à [format Text](supported-file-formats-and-compression-codecs-legacy.md#text-format), [format Json](supported-file-formats-and-compression-codecs-legacy.md#json-format), [format Avro](supported-file-formats-and-compression-codecs-legacy.md#avro-format), [format Orc](supported-file-formats-and-compression-codecs-legacy.md#orc-format) et [format Parquet](supported-file-formats-and-compression-codecs-legacy.md#parquet-format). |Non (uniquement pour un scénario de copie binaire) |
| compression | Spécifiez le type et le niveau de compression pour les données. Pour plus d’informations, voir [Formats de fichier et de codecs de compression pris en charge](supported-file-formats-and-compression-codecs-legacy.md#compression-support).<br/>Les types pris en charge sont : *GZip*, *Deflate*, *BZip2* et *ZipDeflate*.<br/>Les niveaux pris en charge sont *Optimal* et *Fastest*. |Non |

>[!TIP]
>Pour copier tous les fichiers d’un dossier, spécifiez *folderPath* uniquement.<br>Pour copier un seul fichier avec un nom donné, spécifiez *folderPath* avec la partie dossier et *fileName* avec le nom du fichier.<br>Pour copier un sous-ensemble de fichiers d’un dossier, spécifiez *folderPath* avec la partie dossier et *fileName* avec le filtre de caractères génériques.

>[!NOTE]
>Si vous avez utilisé la propriété *fileFilter* pour le filtre de fichiers, il est encore pris en charge tel quel, mais nous vous recommandons d’utiliser à l’avenir la nouvelle fonctionnalité de filtre ajoutée à *fileName*.

**Exemple :**

```json
{
    "name": "SFTPDataset",
    "type": "Datasets",
    "properties": {
        "type": "FileShare",
        "linkedServiceName":{
            "referenceName": "<SFTP linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "folder/subfolder/",
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
| type | La propriété *type* de la source d’activité de copie doit être définie sur *FileSystemSource* |Oui |
| recursive | Indique si les données sont lues de manière récursive à partir des sous-dossiers ou uniquement du dossier spécifié. Lorsque l’option « recursive » est définie sur *true* et que le récepteur est un magasin basé sur un fichier, les dossiers et sous-dossiers vides ne sont pas copiés ni créés au niveau du récepteur.<br/>Les valeurs autorisées sont *true* (par défaut) et *false* | Non |
| maxConcurrentConnections | Nombre de connexions simultanées possibles à un magasin de stockage. Spécifiez une valeur uniquement lorsque vous souhaitez limiter le nombre de connexions simultanées au magasin de données. | Non |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromSFTP",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SFTP input dataset name>",
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
                "type": "FileSystemSource",
                "recursive": true
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des banques de données prises en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez les [magasins de données pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).
