---
title: Copier des données depuis HDFS à l’aide d’Azure Data Factory
description: Découvrez comment utiliser l’activité de copie dans un pipeline Azure Data Factory pour copier des données d’une source HDFS dans le cloud ou en local vers des banques de données réceptrices prises en charge.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 12/18/2020
ms.author: jingwang
ms.openlocfilehash: 6670d6dc676ebefa149815253d5ce65c8a9b1abe
ms.sourcegitcommit: 66b0caafd915544f1c658c131eaf4695daba74c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97680939"
---
# <a name="copy-data-from-the-hdfs-server-by-using-azure-data-factory"></a>Copier des données à partir d’un serveur HDFS à l’aide d’Azure Data Factory

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-hdfs-connector.md)
> * [Version actuelle](connector-hdfs.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article décrit comment copier des données à partir du serveur Hadoop Distributed File System (HDFS). Pour en savoir plus sur Azure Data Factory, lisez l’[article d’introduction](introduction.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Le connecteur HDFS est pris en charge pour les activités suivantes :

- [Activité de copie](copy-activity-overview.md) avec [prise en charge de la matrice de source/récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)
- [Supprimer l’activité](delete-activity.md)

Plus précisément, le connecteur HDFS prend en charge ce qui suit :

- Copie de fichiers en utilisant une authentification *Windows* (Kerberos) ou *Anonyme*.
- Copie de fichiers en utilisant le protocole *webhdfs* ou la prise en charge de *DistCp intégré*.
- Copie de fichiers en l’état, ou analyse ou génération de fichiers avec les [formats de fichier et codecs de compression pris en charge](supported-file-formats-and-compression-codecs.md).

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

> [!NOTE]
> Assurez-vous que le runtime d'intégration peut accéder à *tous* les éléments [serveur du nœud de nom]:[port du nœud de nom] et [serveurs du nœud de données]:[port du nœud de données] du cluster Hadoop. Le [port du nœud de nom] par défaut est 50070 et le [port du nœud de données] par défaut est 50075.

## <a name="get-started"></a>Bien démarrer

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes fournissent des informations sur les propriétés utilisées pour définir les entités Data Factory spécifiques de HDFS :

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié HDFS sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété *type* doit être définie sur *Hdfs*. | Oui |
| url |URL vers le système HDFS |Oui |
| authenticationType | Valeurs autorisées : *Anonyme* ou *Windows*. <br><br> Pour configurer votre environnement local, consultez la section [Utiliser l’authentification Kerberos pour le connecteur HDFS](#use-kerberos-authentication-for-the-hdfs-connector). |Oui |
| userName |Nom d’utilisateur de l’authentification Windows. Pour l’authentification Kerberos, spécifiez **\<username>@\<domain>.com**. |Oui (pour l’authentification Windows) |
| mot de passe |Mot de passe de l’authentification Windows. Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité dans votre fabrique de données, ou [référencez un secret stocké dans le coffre de clés Azure](store-credentials-in-key-vault.md). |Oui (pour l’authentification Windows) |
| connectVia | Le [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Pour en savoir plus, consultez la section [Conditions préalables](#prerequisites). Si le runtime d’intégration n’est pas spécifié, le service utilise le runtime d’intégration Azure par défaut. |Non |

**Exemple : utilisation d’une authentification anonyme**

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "authenticationType": "Anonymous",
            "userName": "hadoop"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Exemple : utilisation d’une authentification Windows**

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "authenticationType": "Windows",
            "userName": "<username>@<domain>.com (for Kerberos auth)",
            "password": {
                "type": "SecureString",
                "value": "<password>"
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

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez [Jeux de données dans Azure Data Factory](concepts-datasets-linked-services.md). 

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Les propriétés suivantes sont prises en charge pour HDFS sous les paramètres `location` dans le jeu de données basé sur le format :

| Propriété   | Description                                                  | Obligatoire |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | La propriété de *type* sous `location` dans le jeu de données doit être définie sur *HdfsLocation*. | Oui      |
| folderPath | Chemin du dossier. Si vous souhaitez utiliser un caractère générique pour filtrer le dossier, ignorez ce paramètre et spécifiez le chemin dans les paramètres de la source de l’activité. | Non       |
| fileName   | Nom de fichier sous le chemin folderPath spécifié. Si vous souhaitez utiliser un caractère générique pour filtrer les fichiers, ignorez ce paramètre et spécifiez le nom de fichier dans les paramètres de la source de l’activité. | Non       |

**Exemple :**

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<HDFS linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "HdfsLocation",
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

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez [Pipelines et activités dans Azure Data Factory](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par la source HDFS.

### <a name="hdfs-as-source"></a>HDFS en tant que source

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Les propriétés suivantes sont prises en charge pour HDFS sous les paramètres `storeSettings` dans la source de la copie basée sur le format :

| Propriété                 | Description                                                  | Obligatoire                                      |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| type                     | La propriété *type* sous `storeSettings` doit être définie sur **HdfsReadSettings**. | Oui                                           |
| **_Rechercher les fichiers à copier_* _ |  |  |
| OPTION 1 : chemin d’accès statique<br> | Copiez à partir du chemin d’accès au dossier ou au fichier spécifié dans le jeu de données. Si vous souhaitez copier tous les fichiers d’un dossier, spécifiez en plus `wildcardFileName` comme `_`. |  |
| OPTION 2 : caractère générique<br>- wildcardFolderPath | Chemin d’accès du dossier avec des caractères génériques pour filtrer les dossiers sources. <br>Les caractères génériques autorisés sont les suivants : `*` (correspond à zéro caractère ou plusieurs) et `?` (correspond à zéro ou un caractère). Utilisez `^` comme caractère d’échappement si le nom réel de votre dossier contient un caractère générique ou ce caractère d’échappement. <br>Pour d’autres exemples, consultez dans [Exemples de filtre de dossier et de fichier](#folder-and-file-filter-examples). | Non                                            |
| OPTION 2 : caractère générique<br>- wildcardFileName | Nom du fichier avec des caractères génériques situé dans le chemin d’accès folderPath/wildcardFolderPath spécifié pour filtrer les fichiers sources. <br>Les caractères génériques autorisés sont `*` (correspond à zéro ou plusieurs caractères) et `?` (correspond à zéro ou un caractère) ; utilisez `^` en guise d’échappement si votre nom de fichier contient effectivement ce caractère d’échappement ou générique.  Pour d’autres exemples, consultez dans [Exemples de filtre de dossier et de fichier](#folder-and-file-filter-examples). | Oui |
| OPTION 3 : liste de fichiers<br>- fileListPath | Indique de copier un ensemble de fichiers spécifié. Pointez vers un fichier texte qui contient une liste de fichiers que vous voulez copier (un fichier par ligne étant le chemin d’accès relatif au chemin d’accès configuré dans le jeu de données).<br/>Lorsque vous utilisez cette option, ne spécifiez pas de nom de fichier dans le jeu de données. Pour plus d’exemples, consultez [Exemples de listes de fichiers](#file-list-examples). |Non |
| ***Paramètres supplémentaires :** _ |  | |
| recursive | Indique si les données sont lues de manière récursive à partir des sous-dossiers ou uniquement du dossier spécifié. Lorsque l’option `recursive` est définie sur _true* et que le récepteur est un magasin basé sur un fichier, aucun dossier ou sous-dossier vide n’est copié ou créé au niveau du récepteur. <br>Les valeurs autorisées sont *true* (par défaut) et *false*.<br>Cette propriété ne s’applique pas lorsque vous configurez `fileListPath`. |Non |
| deleteFilesAfterCompletion | Indique si les fichiers binaires seront supprimés du magasin source après leur déplacement vers le magasin de destination. La suppression se faisant par fichier, lorsque l’activité de copie échoue, vous pouvez constater que certains fichiers ont déjà été copiés vers la destination et supprimés de la source, tandis que d’autres restent dans le magasin source. <br/>Cette propriété est valide uniquement dans un scénario de copie de fichiers binaires. La valeur par défaut est false. |Non |
| modifiedDatetimeStart    | Les fichiers sont filtrés en fonction de l’attribut *Dernière modification*. <br>Les fichiers sont sélectionnés si leur heure de dernière modification se trouve dans l’intervalle situé entre `modifiedDatetimeStart` et `modifiedDatetimeEnd`. L’heure est appliquée au fuseau horaire UTC au format *2018-12-01T05:00:00Z*. <br> Les propriétés peuvent être NULL, ce qui signifie qu’aucun filtre d’attribut de fichier n’est appliqué au jeu de données.  Lorsque `modifiedDatetimeStart` a une valeur DateHeure, mais que `modifiedDatetimeEnd` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est supérieur ou égal à la valeur DateHeure sont sélectionnés.  Lorsque `modifiedDatetimeEnd` a une valeur DateHeure, mais que `modifiedDatetimeStart` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est inférieur à la valeur DateHeure sont sélectionnés.<br/>Cette propriété ne s’applique pas lorsque vous configurez `fileListPath`. | Non                                            |
| modifiedDatetimeEnd      | Identique à ce qui précède.  
| enablePartitionDiscovery | Pour les fichiers partitionnés, spécifiez s’il faut analyser les partitions à partir du chemin d’accès au fichier et les ajouter en tant que colonnes sources supplémentaires.<br/>Les valeurs autorisées sont **false** (par défaut) et **true**. | Non                                            |
| partitionRootPath | Lorsque la découverte de partition est activée, spécifiez le chemin d’accès racine absolu pour pouvoir lire les dossiers partitionnés en tant que colonnes de données.<br/><br/>S’il n’est pas spécifié, par défaut :<br/>– Quand vous utilisez le chemin d’accès du fichier dans le jeu de données ou la liste des fichiers sur la source, le chemin racine de la partition est le chemin d’accès configuré dans le jeu de données.<br/>– Quand vous utilisez le filtre de dossiers de caractères génériques, le chemin d’accès racine de la partition est le sous-chemin d’accès avant le premier caractère générique.<br/><br/>Par exemple, en supposant que vous configurez le chemin d’accès dans le jeu de données en tant que « root/folder/year=2020/month=08/day=27 » :<br/>– Si vous spécifiez le chemin d’accès racine de la partition en tant que « root/folder/year=2020 », l’activité de copie génère deux colonnes supplémentaires, `month` et `day`, ayant respectivement la valeur « 08 » et « 27 », en plus des colonnes contenues dans les fichiers.<br/>– Si le chemin d’accès racine de la partition n’est pas spécifié, aucune colonne supplémentaire n’est générée. | Non                                            |
| maxConcurrentConnections | Nombre de connexions simultanées possibles au magasin de stockage. Spécifiez une valeur uniquement lorsque vous souhaitez limiter le nombre de connexions simultanées au magasin de données. | Non                                            |
| **_Paramètres DistCp_* _ |  | |
| distcpSettings | Groupe de propriétés à utiliser lorsque vous utilisez DistCp HDFS. | Non |
| resourceManagerEndpoint | Point de terminaison YARN (Yet Another Resource Negotiator) | Oui, en cas d’utilisation de DistCp |
| tempScriptPath | Chemin d’accès du dossier utilisé pour stocker le script de commande DistCp temporaire. Le fichier de script est généré par Data Factory et supprimé une fois le travail de copie terminé. | Oui, en cas d’utilisation de DistCp |
| distcpOptions | Options supplémentaires fournies à la commande DistCp. | Non |

_ *Exemple :* *

```json
"activities":[
    {
        "name": "CopyFromHDFS",
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
                    "type": "HdfsReadSettings",
                    "recursive": true,
                    "distcpSettings": {
                        "resourceManagerEndpoint": "resourcemanagerendpoint:8088",
                        "tempScriptPath": "/usr/hadoop/tempscript",
                        "distcpOptions": "-m 100"
                    }
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="folder-and-file-filter-examples"></a>Exemples de filtres de dossier et de fichier

Cette section décrit le comportement résultant de l’utilisation de filtres de caractères génériques dans les noms de fichier et les chemins de dossier.

| folderPath | fileName             | recursive | Structure du dossier source et résultat du filtrage (les fichiers en **gras** sont récupérés) |
| :--------- | :------------------- | :-------- | :----------------------------------------------------------- |
| `Folder*`  | (vide, utiliser la valeur par défaut) | false     | DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5.csv<br/>AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |
| `Folder*`  | (vide, utiliser la valeur par défaut) | true      | DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier5.csv**<br/>AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |
| `Folder*`  | `*.csv`              | false     | DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5.csv<br/>AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |
| `Folder*`  | `*.csv`              | true      | DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier5.csv**<br/>AutreDossierB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier6.csv |

### <a name="file-list-examples"></a>Exemples de liste de fichiers

Cette section décrit le comportement résultant de l’utilisation d’un chemin d’accès de liste de fichiers dans la source de l’activité Copy. Supposons que vous disposez de la structure de dossiers sources suivante et que vous souhaitez copier les fichiers en gras :

| Exemple de structure source                                      | Contenu de FileListToCopy.txt                             | Configuration d’Azure Data Factory                                            |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| root<br/>&nbsp;&nbsp;&nbsp;&nbsp;DossierA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Fichier5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Métadonnées<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FileListToCopy.txt | File1.csv<br>Subfolder1/File3.csv<br>Subfolder1/File5.csv | **Dans le jeu de données :**<br>- chemin d’accès du dossier : `root/FolderA`<br><br>**Dans la source de l’activité de copie :**<br>- chemin d’accès à la liste de fichiers : `root/Metadata/FileListToCopy.txt` <br><br>Le chemin d’accès à la liste de fichiers pointe vers un fichier texte dans le même magasin de données qui contient la liste de fichiers que vous voulez copier (un fichier par ligne étant le chemin d’accès relatif au chemin d’accès configuré dans le jeu de données). |

## <a name="use-distcp-to-copy-data-from-hdfs"></a>Utiliser DistCp pour copier des données de HDFS

[DistCp](https://hadoop.apache.org/docs/current3/hadoop-distcp/DistCp.html) est un outil en ligne de commande Hadoop natif permettant d’effectuer une copie distribuée dans un cluster Hadoop. Quand vous exécutez une commande dans l’outil DistCp, il répertorie tout d’abord tous les fichiers à copier, puis crée plusieurs tâches de mappage dans le cluster Hadoop. Chaque tâche de mappage effectue une copie binaire de la source vers le récepteur.

L’activité de copie prend en charge l’utilisation de DistCp pour copier des fichiers en l’état dans le stockage d’objets Azure Blob (y compris [copie intermédiaire](copy-activity-performance.md)) ou le stockage de lac de données Azure. Dans ce cas, DistCp peut tirer parti de la puissance de votre cluster au lieu de s’exécuter sur le runtime d’intégration auto-hébergé. Utiliser DistCp permet que le débit de la copie soit ainsi amélioré, en particulier si votre cluster est très puissant. Selon la configuration de votre fabrique de données, l’activité de copie construit automatiquement une commande DistCp, l’envoie à votre cluster Hadoop et surveille l’état de la copie.

### <a name="prerequisites"></a>Prérequis

Pour utiliser DistCp pour copier des fichiers en l’état de HDFS vers le stockage de blob Azure (y compris une copie intermédiaire) ou du lac de données Azure, assurez-vous que votre cluster Hadoop présente la configuration requise suivante :

* Les services MapReduce et YARN sont activés.  
* YARN version 2.5 ou ultérieure.  
* Le serveur HDFS est intégré à votre banque de données cible : **Stockage Blob Azure** ou **Azure Data Lake Store (ADLS Gen1)**  : 

    - Le système de fichiers d’objet blob Azure est pris en charge en mode natif depuis Hadoop 2.7. Vous devez uniquement spécifier le chemin d’accès JAR dans la configuration de l’environnement Hadoop.
    - Le système de fichiers d’Azure Data Lake Store est empaqueté depuis Hadoop 3.0.0-alpha1. Si la version de votre cluster Hadoop est antérieure à cette version, vous devez importer manuellement les packages JAR associés à Azure Data Lake Store (azure-datalake-store.jar) dans le cluster à partir d’[ici](https://hadoop.apache.org/releases.html) et spécifier le chemin d’accès du fichier JAR dans la configuration de l’environnement Hadoop.

* Préparez un dossier temporaire dans HDFS. Ce dossier temporaire est utilisé pour stocker le script d'interpréteur de commandes DistCp afin qu’il occupe un espace exprimable en Ko.
* Assurez-vous que le compte d’utilisateur fourni dans le service lié HDFS a l’autorisation d’effectuer les opérations suivantes :
   * Soumettez une application dans YARN.
   * Créez un sous-dossier et écrivez/lisez les fichiers du dossier Temp.

### <a name="configurations"></a>Configurations

Pour voir les exemples et les configurations liés à DistCp, consultez la section [HDFS en tant que source](#hdfs-as-source).

## <a name="use-kerberos-authentication-for-the-hdfs-connector"></a>Utilisation de l’authentification Kerberos pour le connecteur HDFS

Il existe deux options pour configurer l’environnement local afin d’utiliser l’authentification Kerberos avec le connecteur HDFS. Vous pouvez choisir celle qui correspond mieux à votre situation.
* Option 1 : [Joindre un ordinateur exécutant le runtime d’intégration auto-hébergé dans le domaine Kerberos](#kerberos-join-realm)
* Option n°2 : [Activer l’approbation mutuelle entre le domaine Windows et le domaine Kerberos](#kerberos-mutual-trust)

Quelle que soit l’option, veillez à activer webhdfs pour le cluster Hadoop :

1. Créez le principal HTTP et le keytab pour webhdfs.

    > [!IMPORTANT]
    > Le principal HTTP Kerberos doit commencer par « **HTTP/**  » conformément à la spécification HTTP SPNEGO de Kerberos. Pour plus d’informations, cliquez [ici](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#HDFS_Configuration_Options).

    ```bash
    Kadmin> addprinc -randkey HTTP/<namenode hostname>@<REALM.COM>
    Kadmin> ktadd -k /etc/security/keytab/spnego.service.keytab HTTP/<namenode hostname>@<REALM.COM>
    ```

2. Options de configuration HDFS : ajoutez les trois propriétés suivantes dans `hdfs-site.xml`.
    ```xml
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.web.authentication.kerberos.principal</name>
        <value>HTTP/_HOST@<REALM.COM></value>
    </property>
    <property>
        <name>dfs.web.authentication.kerberos.keytab</name>
        <value>/etc/security/keytab/spnego.service.keytab</value>
    </property>
    ```

### <a name="option-1-join-a-self-hosted-integration-runtime-machine-in-the-kerberos-realm"></a><a name="kerberos-join-realm"></a>Option 1 : Joindre un ordinateur exécutant le runtime d’intégration auto-hébergé dans le domaine Kerberos

#### <a name="requirements"></a>Spécifications

* L’ordinateur exécutant le runtime d’intégration auto-hébergé doit joindre le domaine Kerberos et ne peut pas joindre un domaine Windows.

#### <a name="how-to-configure"></a>Comment configurer

**Sur le serveur du centre de distribution de clés Kerberos :**

Créez un principal pour Azure Data Factory et spécifiez le mot de passe.

> [!IMPORTANT]
> Le nom d’utilisateur ne doit pas contenir le nom d’hôte.

```bash
Kadmin> addprinc <username>@<REALM.COM>
```

**Sur un ordinateur exécutant le runtime d’intégration auto-hébergé :**

1.  Exécutez l’utilitaire Ksetup pour configurer le serveur et le domaine du centre de distribution de clés Kerberos.

    L’ordinateur doit être configuré en tant que membre d’un groupe de travail, car un domaine Kerberos est différent d’un domaine Windows. Vous pouvez obtenir cette configuration en définissant le domaine Kerberos et en ajoutant un serveur KDC en exécutant les commandes suivantes. Remplacez *REALM.COM* par votre propre nom de domaine.

    ```cmd
    C:> Ksetup /setdomain REALM.COM
    C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
    ```

    Après avoir exécuté ces commandes, redémarrez l’ordinateur.

2.  Vérifiez la configuration avec la commande `Ksetup`. La sortie doit être semblable à :

    ```cmd
    C:> Ksetup
    default realm = REALM.COM (external)
    REALM.com:
        kdc = <your_kdc_server_address>
    ```

**Dans votre fabrique de données :**

* Configurez le connecteur HDFS à l’aide de l’authentification Windows avec votre nom principal Kerberos et le mot de passe pour vous connecter à la source de données HDFS. Vérifiez les détails de configuration dans la section [Propriétés du service lié HDFS](#linked-service-properties).

### <a name="option-2-enable-mutual-trust-between-the-windows-domain-and-the-kerberos-realm"></a><a name="kerberos-mutual-trust"></a>Option 2 : Activer l’approbation mutuelle entre le domaine Windows et le domaine Kerberos

#### <a name="requirements"></a>Spécifications

*   L’ordinateur exécutant le runtime d’intégration auto-hébergé doit joindre un domaine Windows.
*   Vous avez besoin d’autorisations pour mettre à jour les paramètres du contrôleur de domaine.

#### <a name="how-to-configure"></a>Comment configurer

> [!NOTE]
> Remplacez REALM.COM et AD.COM dans le didacticiel suivant par votre propre nom de domaine et de contrôleur de domaine.

**Sur le serveur du centre de distribution de clés Kerberos :**

1. Modifiez la configuration KDC dans le fichier *krb5.conf* afin de permettre au KDC d’approuver le domaine Windows faisant référence au modèle de configuration suivant. Par défaut, la configuration se trouve dans */etc/krb5.conf*.

   ```config
   [logging]
    default = FILE:/var/log/krb5libs.log
    kdc = FILE:/var/log/krb5kdc.log
    admin_server = FILE:/var/log/kadmind.log
            
   [libdefaults]
    default_realm = REALM.COM
    dns_lookup_realm = false
    dns_lookup_kdc = false
    ticket_lifetime = 24h
    renew_lifetime = 7d
    forwardable = true
            
   [realms]
    REALM.COM = {
     kdc = node.REALM.COM
     admin_server = node.REALM.COM
    }
   AD.COM = {
    kdc = windc.ad.com
    admin_server = windc.ad.com
   }
            
   [domain_realm]
    .REALM.COM = REALM.COM
    REALM.COM = REALM.COM
    .ad.com = AD.COM
    ad.com = AD.COM
            
   [capaths]
    AD.COM = {
     REALM.COM = .
    }
    ```

   Après avoir configuré le fichier, redémarrez le service KDC.

2. Préparez un fichier principal nommé *krbtgt/REALM.COM\@AD.COM* dans le serveur KDC avec la commande suivante :

    ```cmd
    Kadmin> addprinc krbtgt/REALM.COM@AD.COM
    ```

3. Dans le fichier de configuration du service HDFS *hadoop.security.auth_to_local*, ajoutez `RULE:[1:$1@$0](.*\@AD.COM)s/\@.*//`.

**Sur le contrôleur de domaine :**

1.  Exécutez les commandes `Ksetup` suivantes pour ajouter une entrée de domaine :

    ```cmd
    C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
    C:> ksetup /addhosttorealmmap HDFS-service-FQDN REALM.COM
    ```

2.  Établissez l’approbation du domaine Windows pour le domaine Kerberos. [password] correspond au mot de passe pour le principal *krbtgt/REALM.COM\@AD.COM*.

    ```cmd
    C:> netdom trust REALM.COM /Domain: AD.COM /add /realm /password:[password]
    ```

3.  Sélectionnez l’algorithme de chiffrement utilisé dans Kerberos.

    a. Sélectionnez **Gestionnaire de serveur** > **Gestion des stratégies de groupe** > **Domaine** > **Objets de stratégie de groupe** > **Stratégie de domaine par défaut ou actif**, puis sélectionnez **Modifier**.

    b. Dans le volet **Éditeur de gestion des stratégies de groupe**, accédez à **Configuration de l’ordinateur** > **Stratégies** > **Paramètres Windows** > **Paramètres de sécurité** > **Stratégies locales** > **Options de sécurité**, puis configurez **Sécurité réseau : Configurer les types de chiffrement autorisés pour Kerberos**.

    c. Sélectionnez l’algorithme de chiffrement à utiliser lorsque vous vous connectez au serveur KDC. Vous pouvez sélectionner toutes les options.

    ![Capture d’écran du volet « Sécurité réseau : Configurer les types de chiffrement autorisés pour Kerberos »](media/connector-hdfs/config-encryption-types-for-kerberos.png)

    d. Utilisez la commande `Ksetup` pour spécifier l’algorithme de chiffrement à utiliser dans le domaine spécifié.

    ```cmd
    C:> ksetup /SetEncTypeAttr REALM.COM DES-CBC-CRC DES-CBC-MD5 RC4-HMAC-MD5 AES128-CTS-HMAC-SHA1-96 AES256-CTS-HMAC-SHA1-96
    ```

4.  Créez le mappage entre le compte de domaine et le principal Kerberos pour utiliser le principal Kerberos dans le domaine Windows.

    a. Sélectionnez **Outils d’administration** > **Utilisateurs et ordinateurs Active Directory**.

    b. Configurez les fonctionnalités avancées en sélectionnant **Afficher** > **Fonctionnalités avancées**.

    c. Dans le volet **Fonctionnalités avancées**, faites un clic droit sur le compte sur lequel vous souhaitez créer des mappages et, dans le volet **Mappages des noms**, sélectionnez l’onglet **Noms Kerberos**.

    d. Ajoutez un principal provenant du domaine.

       ![Le volet « Mappage des identités de sécurité »](media/connector-hdfs/map-security-identity.png)

**Sur un ordinateur exécutant le runtime d’intégration auto-hébergé :**

* Exécutez les commandes `Ksetup` suivantes pour ajouter une entrée de domaine.

   ```cmd
   C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
   C:> ksetup /addhosttorealmmap HDFS-service-FQDN REALM.COM
   ```

**Dans votre fabrique de données :**

* Configurez le connecteur HDFS à l’aide de l’authentification Windows avec votre compte de domaine ou le principal Kerberos pour vous connecter à la source de données HDFS. Pour les détails de configuration, consultez la section [Propriétés du service lié HDFS](#linked-service-properties).

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour obtenir des informations sur les propriétés de l’activité de recherche (Lookup), consultez [Activité de recherche dans Azure Data Factory](control-flow-lookup-activity.md).

## <a name="delete-activity-properties"></a>Propriétés de l’activité Delete

Pour obtenir des informations sur les propriétés de l’activité Delete, consultez [Activité Delete dans Azure Data Factory](delete-activity.md).

## <a name="legacy-models"></a>Modèles hérités

>[!NOTE]
>Les modèles suivants sont toujours pris en charge tels quels à des fins de compatibilité descendante. Nous vous recommandons d’utiliser le nouveau modèle abordé précédemment, car l’interface utilisateur de création Azure Data Factory a basculé vers la génération du nouveau modèle.

### <a name="legacy-dataset-model"></a>Modèle de jeu de données hérité

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété *type* du jeu de données doit être définie sur *FileShare* |Oui |
| folderPath | Chemin du dossier. Un filtre de caractères génériques est pris en charge. Les caractères génériques autorisés sont `*` (correspond à zéro ou plusieurs caractères) et `?` (correspond à zéro ou un caractère) ; utilisez `^` en guise d’échappement si votre nom de fichier contient effectivement ce caractère d’échappement ou générique. <br/><br/>Exemples : dossier_racine/sous-dossier/ ; consultez d’autres exemples dans [Exemples de filtres de dossier et de fichier](#folder-and-file-filter-examples). |Oui |
| fileName |  Filtre de noms ou de caractères génériques pour les fichiers sous le « folderPath » spécifié. Si vous ne spécifiez pas de valeur pour cette propriété, le jeu de données pointe vers tous les fichiers du dossier. <br/><br/>Dans le filtre, les caractères génériques autorisés sont `*` (correspond à zéro caractère ou plus) et `?` (correspond à zéro ou un caractère).<br/>- Exemple 1 : `"fileName": "*.csv"`<br/>- Exemple 2 : `"fileName": "???20180427.txt"`<br/>Utilisez `^` comme caractère d’échappement si le nom réel de votre dossier contient un caractère générique ou ce caractère d’échappement. |Non |
| modifiedDatetimeStart | Les fichiers sont filtrés en fonction de l’attribut *Dernière modification*. Les fichiers sont sélectionnés si leur heure de dernière modification se trouve dans l’intervalle situé entre `modifiedDatetimeStart` et `modifiedDatetimeEnd`. L’heure est appliquée au fuseau horaire UTC au format *2018-12-01T05:00:00Z*. <br/><br/> Sachez que les performances globales du déplacement des données sont affectées par l’activation de ce paramètre lorsque vous souhaitez filtrer des fichiers parmi de grandes quantités de fichiers. <br/><br/> Les propriétés peuvent être NULL, ce qui signifie qu’aucun filtre d’attribut de fichier n’est appliqué au jeu de données.  Lorsque `modifiedDatetimeStart` a une valeur DateHeure, mais que `modifiedDatetimeEnd` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est supérieur ou égal à la valeur DateHeure sont sélectionnés.  Lorsque `modifiedDatetimeEnd` a une valeur DateHeure, mais que `modifiedDatetimeStart` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est inférieur à la valeur DateHeure sont sélectionnés.| Non |
| modifiedDatetimeEnd | Les fichiers sont filtrés en fonction de l’attribut *Dernière modification*. Les fichiers sont sélectionnés si leur heure de dernière modification se trouve dans l’intervalle situé entre `modifiedDatetimeStart` et `modifiedDatetimeEnd`. L’heure est appliquée au fuseau horaire UTC au format *2018-12-01T05:00:00Z*. <br/><br/> Sachez que les performances globales du déplacement des données sont affectées par l’activation de ce paramètre lorsque vous souhaitez filtrer des fichiers parmi de grandes quantités de fichiers. <br/><br/> Les propriétés peuvent être NULL, ce qui signifie qu’aucun filtre d’attribut de fichier n’est appliqué au jeu de données.  Lorsque `modifiedDatetimeStart` a une valeur DateHeure, mais que `modifiedDatetimeEnd` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est supérieur ou égal à la valeur DateHeure sont sélectionnés.  Lorsque `modifiedDatetimeEnd` a une valeur DateHeure, mais que `modifiedDatetimeStart` est NULL, cela signifie que les fichiers dont l’attribut de dernière modification est inférieur à la valeur DateHeure sont sélectionnés.| Non |
| format | Si vous souhaitez copier des fichiers en l’état entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie.<br/><br/>Si vous voulez analyser des fichiers dans un format spécifique, les types de format de fichier suivants sont pris en charge : *TextFormat*, *JsonFormat*, *AvroFormat*, *OrcFormat* et *ParquetFormat*. Définissez la propriété *type* située sous Format sur l’une de ces valeurs. Pour en savoir plus, voir les sections [Format Text](supported-file-formats-and-compression-codecs-legacy.md#text-format), [Format JSON](supported-file-formats-and-compression-codecs-legacy.md#json-format), [Format Avro](supported-file-formats-and-compression-codecs-legacy.md#avro-format), [Format ORC](supported-file-formats-and-compression-codecs-legacy.md#orc-format) et [Format Parquet](supported-file-formats-and-compression-codecs-legacy.md#parquet-format). |Non (uniquement pour un scénario de copie binaire) |
| compression | Spécifiez le type et le niveau de compression pour les données. Pour plus d’informations, voir [Formats de fichier et de codecs de compression pris en charge](supported-file-formats-and-compression-codecs-legacy.md#compression-support).<br/>Types pris en charge : *GZip*, *Deflate*, *BZip2* et *ZipDeflate*.<br/>Niveaux pris en charge : *Optimal* et *Fastest*. |Non |

>[!TIP]
>Pour copier tous les fichiers d’un dossier, spécifiez **folderPath** uniquement.<br>Pour copier un seul fichier avec un nom donné, spécifiez **folderPath** avec la partie dossier et **fileName** avec le nom du fichier.<br>Pour copier un sous-ensemble de fichiers d’un dossier, spécifiez **folderPath** avec la partie dossier et **fileName** avec le filtre de caractères génériques.

**Exemple :**

```json
{
    "name": "HDFSDataset",
    "properties": {
        "type": "FileShare",
        "linkedServiceName":{
            "referenceName": "<HDFS linked service name>",
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
| type | La propriété *type* de la source de l’activité de copie doit être définie sur *HdfsSource*. |Oui |
| recursive | Indique si les données sont lues de manière récursive à partir des sous-dossiers ou uniquement du dossier spécifié. Lorsque l’option « recursive » est définie sur *true* et que le récepteur est un magasin basé sur un fichier, un dossier vide ou un sous-dossier n’est pas copié ou créé sur le récepteur.<br/>Les valeurs autorisées sont *true* (par défaut) et *false*. | Non |
| distcpSettings | Groupe de propriétés lorsque vous utilisez HDFS DistCp. | Non |
| resourceManagerEndpoint | Point de terminaison de YARN Resource Manager | Oui, en cas d’utilisation de DistCp |
| tempScriptPath | Chemin d’accès du dossier utilisé pour stocker le script de commande DistCp temporaire. Le fichier de script est généré par Data Factory et supprimé une fois le travail de copie terminé. | Oui, en cas d’utilisation de DistCp |
| distcpOptions | Options supplémentaires fournies à la commande DistCp. | Non |
| maxConcurrentConnections | Nombre de connexions simultanées possibles au magasin de stockage. Spécifiez une valeur uniquement lorsque vous souhaitez limiter le nombre de connexions simultanées au magasin de données. | Non |

**Exemple : Source HDFS dans une activité de copie avec DistCp**

```json
"source": {
    "type": "HdfsSource",
    "distcpSettings": {
        "resourceManagerEndpoint": "resourcemanagerendpoint:8088",
        "tempScriptPath": "/usr/hadoop/tempscript",
        "distcpOptions": "-m 100"
    }
}
```

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des banques de données prises en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez les [magasins de données pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).
