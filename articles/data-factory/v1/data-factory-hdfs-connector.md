---
title: Déplacer des données à partir d’un HDFS local
description: Découvrez comment transférer des données à partir d’un HDFS en local à l’aide d’Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: 3215b82d-291a-46db-8478-eac1a3219614
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: c4393ebeb8b1e287bd881233418a902fc523f7f5
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97589612"
---
# <a name="move-data-from-on-premises-hdfs-using-azure-data-factory"></a>Transfert de données à partir d’un HDFS local à l’aide d’Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-hdfs-connector.md)
> * [Version 2 (version actuelle)](../connector-hdfs.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Connecteur HDFS dans V2](../connector-hdfs.md).

Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour déplacer des données à partir d’un HDFS local. Il s’appuie sur l’article [Activités de déplacement des données](data-factory-data-movement-activities.md), qui présente une vue d’ensemble du déplacement de données avec l’activité de copie.

Vous pouvez copier les données depuis HDFS vers tout magasin de données récepteur pris en charge. Consultez le tableau [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) pour obtenir la liste des magasins de données pris en charge en tant que récepteurs par l’activité de copie. Actuellement, Data Factory prend uniquement en charge le déplacement de données de HDFS en local vers d’autres magasins de données, mais pas l’inverse.

> [!NOTE]
> L’activité de copie ne supprime pas le fichier source une fois qu’il est copié sur la destination. Si vous devez supprimer le fichier source une fois qu’il est copié, créez une activité personnalisée pour supprimer le fichier et utilisez l’activité dans le pipeline. 

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="enabling-connectivity"></a>Activation de la connectivité
Le service Data Factory prend en charge la connexion à des sources HDFS locales à l’aide de la passerelle de gestion des données. Consultez l’article [Déplacement de données entre des emplacements locaux et le cloud](data-factory-move-data-between-onprem-and-cloud.md) pour en savoir plus sur la passerelle de gestion des données et obtenir des instructions détaillées sur la configuration de la passerelle. Utilisez la passerelle pour vous connecter à HDFS même si elle est hébergée sur des machines virtuelles IaaS Azure.

> [!NOTE]
> Assurez-vous que la passerelle de gestion des données peut accéder à **TOUS** les éléments [serveur du nœud de nom]:[port du nœud de nom] et [serveurs du nœud de données]:[port du nœud de données] du cluster Hadoop. Le [port du nœud de nom] par défaut est 50070 et le [port du nœud de données] par défaut est 50075.

Bien qu’il soit possible d’installer la passerelle sur le même ordinateur local ou la même machine virtuelle Azure que le HDFS, nous vous recommandons de l’installer sur un ordinateur ou une machine virtuelle IaaS Azure distinct(e). Disposer d’une passerelle sur un ordinateur distinct réduit les conflits de ressources et améliore les performances. Lorsque vous installez la passerelle sur un ordinateur distinct, l’ordinateur doit être en mesure d’accéder à l’ordinateur qui exécute le système HDFS.

## <a name="getting-started"></a>Prise en main
Vous pouvez créer un pipeline avec une activité de copie qui déplace les données d’une source HDFS à l’aide de différents outils/API.

Le moyen le plus simple de créer un pipeline consiste à utiliser **l’Assistant Copie**. Voir le [tutoriel : Créer un pipeline avec l’activité de copie à l’aide de l’Assistant Copie](data-factory-copy-data-wizard-tutorial.md) pour obtenir une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant Copie de données.

Vous pouvez également utiliser les outils suivants pour créer un pipeline : **portail Azure**, **Visual Studio**, **Azure PowerShell**, **modèle Azure Resource Manager**, **API .NET** et **API REST**. Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [didacticiel sur l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Que vous utilisiez des outils ou des API, la création d’un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur implique les étapes suivantes :

1. Création de **services liés** pour lier les magasins de données d’entrée et de sortie à votre fabrique de données.
2. Création de **jeux de données** pour représenter les données d’entrée et de sortie de l’opération de copie.
3. Création d’un **pipeline** avec une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie.

Lorsque vous utilisez l’Assistant, les définitions JSON de ces entités Data Factory (services liés, jeux de données et pipeline) sont automatiquement créées pour vous. Lorsque vous utilisez des outils/API (à l’exception de l’API .NET), vous devez définir ces entités Data Factory au format JSON.  Pour un exemple contenant des définitions JSON pour les entités Data Factory servant à copier des données à partir d’un magasin de données HDFS, consultez la section [Exemple JSON : Copier des données d’une source HDFS vers Stockage Blob Azure](#json-example-copy-data-from-on-premises-hdfs-to-azure-blob) de cet article.

Les sections suivantes fournissent des informations sur les propriétés JSON utilisées pour définir les entités Data Factory spécifiques à HDFS :

## <a name="linked-service-properties"></a>Propriétés du service lié
Un service lié lie un magasin de données à une fabrique de données. Vous créez un service lié de type **HDFS** pour lier un système HDFS local à votre fabrique de données. Le tableau suivant fournit la description des éléments JSON spécifiques au service lié HDFS.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |La propriété type doit être définie sur : **Hdfs** |Oui |
| url |URL vers le système HDFS |Oui |
| authenticationType |Anonyme ou Windows. <br><br> Pour utiliser l’**authentification Kerberos** pour le connecteur HDFS, reportez-vous à [cette section](#use-kerberos-authentication-for-hdfs-connector) pour configurer votre environnement local en conséquence. |Oui |
| userName |Nom d’utilisateur de l’authentification Windows Pour l’authentification Kerberos, spécifiez `<username>@<domain>.com`. |Oui (pour l’authentification Windows) |
| mot de passe |Mot de passe de l’authentification Windows |Oui (pour l’authentification Windows) |
| gatewayName |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter au système HDFS. |Oui |
| encryptedCredential |[New-AzDataFactoryEncryptValue](/powershell/module/az.datafactory/new-azdatafactoryencryptvalue) des informations d’accès. |Non |

### <a name="using-anonymous-authentication"></a>Utilisation de l’authentification anonyme

```JSON
{
    "name": "hdfs",
    "properties":
    {
        "type": "Hdfs",
        "typeProperties":
        {
            "authenticationType": "Anonymous",
            "userName": "hadoop",
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "gatewayName": "mygateway"
        }
    }
}
```

### <a name="using-windows-authentication"></a>Avec l’authentification Windows

```JSON
{
    "name": "hdfs",
    "properties":
    {
        "type": "Hdfs",
        "typeProperties":
        {
            "authenticationType": "Windows",
            "userName": "<username>@<domain>.com (for Kerberos auth)",
            "password": "password",
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "gatewayName": "mygateway"
        }
    }
}
```
## <a name="dataset-properties"></a>Propriétés du jeu de données
Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md). Les sections comme la structure, la disponibilité et la stratégie d'un jeu de données JSON sont similaires pour tous les types de jeux de données (SQL Azure, Azure Blob, Azure Table, etc.).

La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations sur l’emplacement des données dans le magasin de données. La section typeProperties du jeu de données de type **FileShare** (comprenant le jeu de données HDFS) présente les propriétés suivantes

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| folderPath |Chemin d'accès au dossier. Exemple : `myfolder`<br/><br/>Utilisez le caractère d’échappement « \ » pour les caractères spéciaux contenus dans la chaîne. Par exemple : pour dossier\sous-dossier, spécifiez dossier\\\\sous-dossier et pour d:\dossier d’exemple, spécifiez d:\\\\dossier d’exemple.<br/><br/>Vous pouvez également effectuer une combinaison avec la propriété **partitionBy** pour que les chemins d’accès de dossier soient basés sur les dates et heures de démarrage et d’arrêt de la tranche. |Oui |
| fileName |Spécifiez le nom du fichier dans l’élément **folderPath** si vous souhaitez que la table se réfère à un fichier spécifique du dossier. Si vous ne spécifiez aucune valeur pour cette propriété, le tableau pointe vers tous les fichiers du dossier.<br/><br/>Lorsque fileName n’est pas spécifié pour un jeu de données de sortie, le nom du fichier généré aura ce format dans l’exemple suivant : <br/><br/>`Data.<Guid>.txt` (par exemple : : Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |Non |
| partitionedBy |partitionedBy peut être utilisé pour spécifier un folderPath dynamique, fileName pour les données de série chronologique. Exemple : folderPath peut être paramétré pour toutes les heures de données. |Non |
| format | Les types de formats suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Définissez la propriété **type** située sous Format sur l’une de ces valeurs. Pour en savoir plus, consultez les sections relatives à [format Text](data-factory-supported-file-and-compression-formats.md#text-format), [format Json](data-factory-supported-file-and-compression-formats.md#json-format), [format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si vous souhaitez **copier des fichiers en l’état** entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie. |Non |
| compression | Spécifiez le type et le niveau de compression pour les données. Types pris en charge : **GZip**, **Deflate**, **BZip2** et **ZipDeflate**. Niveaux pris en charge : **Optimal** et **Fastest**. Pour plus d’informations, consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |Non |

> [!NOTE]
> fileName et fileFilter ne peuvent pas être utilisés simultanément.

### <a name="using-partionedby-property"></a>Utilisation de la propriété partitionedBy
Comme mentionné dans la section précédente, vous pouvez spécifier un folderPath et un fileName dynamiques pour les données de série chronologique avec la propriété **partitionedBy**, [les fonctions Data Factory et les variables système](data-factory-functions-variables.md).

Consultez les articles [Création de jeux de données](data-factory-create-datasets.md), [Planification et exécution](data-factory-scheduling-and-execution.md) et [Création de pipelines](data-factory-create-pipelines.md) pour en savoir plus sur les jeux de données de série chronologique, la planification et les segments.

#### <a name="sample-1"></a>Exemple 1 :

```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```
Dans cet exemple, {Slice} est remplacé par la valeur de la variable système Data Factory SliceStart au format (AAAAMMJJHH) spécifié. SliceStart fait référence à l'heure de début du segment. folderPath est différent pour chaque segment. Par exemple : wikidatagateway/wikisampledataout/2014100103 ou wikidatagateway/wikisampledataout/2014100104.

#### <a name="sample-2"></a>Exemple 2 :

```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
 [
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```
Dans cet exemple, l'année, le mois, le jour et l'heure de SliceStart sont extraits dans des variables distinctes qui sont utilisées par les propriétés folderPath et fileName.

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie
Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les tables d'entrée et de sortie et les différentes stratégies sont disponibles pour tous les types d'activités.

En revanche, les propriétés disponibles dans la section typeProperties de l’activité varient pour chaque type d'activité. Pour l’activité de copie, elles dépendent des types de sources et récepteurs.

Pour une activité de copie, quand la source est de type **FileSystemSource** , les propriétés suivantes sont disponibles dans la section typeProperties :

**FileSystemSource** prend en charge les propriétés suivantes :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| recursive |Indique si les données sont lues de manière récursive dans les sous-dossiers ou uniquement dans le dossier spécifié. |True, False (par défaut) |Non |

## <a name="supported-file-and-compression-formats"></a>Formats de fichier et de compression pris en charge
Pour plus d’informations, voir [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md).

## <a name="json-example-copy-data-from-on-premises-hdfs-to-azure-blob"></a>Exemple JSON : Copier des données depuis HDFS local vers Stockage Blob Azure
Cet exemple indique comment copier des données depuis un système HDFS local vers un système Blob Storage Microsoft Azure. Toutefois, les données peuvent être copiées **directement** vers l’un des récepteurs indiqués [ici](data-factory-data-movement-activities.md#supported-data-stores-and-formats) , via l’activité de copie d’Azure Data Factory.  

Cet exemple fournit des définitions JSON pour les entités Data Factory suivantes. Vous pouvez utiliser ces définitions pour créer un pipeline afin de copier des données de HDFS vers un Stockage Blob Azure à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou d’[Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md).

1. Un service lié de type [OnPremisesHdfs](#linked-service-properties).
2. Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [FileShare](#dataset-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
5. Un [pipeline](data-factory-create-pipelines.md) avec activité de copie qui utilise [FileSystemSource](#copy-activity-properties) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

L’exemple copie toutes les heures les données d’un système HDFS en local vers un objet Blob Azure. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

Dans un premier temps, configurez la passerelle de gestion des données. Les instructions se trouvent dans l’article [Déplacement de données entre des emplacements locaux et le cloud](data-factory-move-data-between-onprem-and-cloud.md) .

**Service lié HDFS :** Cet exemple utilise l’authentification Windows. Consultez la section [Service lié HDFS](#linked-service-properties) pour connaître les différents types d’authentification que vous pouvez utiliser.

```JSON
{
    "name": "HDFSLinkedService",
    "properties":
    {
        "type": "Hdfs",
        "typeProperties":
        {
            "authenticationType": "Windows",
            "userName": "Administrator",
            "password": "password",
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "gatewayName": "mygateway"
        }
    }
}
```

**Service lié Azure Storage :**

```JSON
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

**Jeu de données d’entrée HDFS :** Ce jeu de données fait référence au dossier HDFS DataTransfer/UnitTest/. Le pipeline copie tous les fichiers de son dossier vers la destination.

La définition de « external » : « true» informe le service Data Factory qu’il s’agit d’un jeu de données qui est externe à Data Factory et non produit par une activité dans Data Factory.

```JSON
{
    "name": "InputDataset",
    "properties": {
        "type": "FileShare",
        "linkedServiceName": "HDFSLinkedService",
        "typeProperties": {
            "folderPath": "DataTransfer/UnitTest/"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}
```

**Jeu de données de sortie d'objet Blob Azure :**

Les données sont écrites dans un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1). Le chemin d’accès du dossier pour l’objet blob est évalué dynamiquement en fonction de l’heure de début du segment en cours de traitement. Le chemin d'accès du dossier utilise l'année, le mois, le jour et l'heure de l'heure de début.

```JSON
{
    "name": "OutputDataset",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/hdfs/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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

**Activité de copie dans un pipeline avec une source Système de fichiers et un récepteur blob :**

Le pipeline contient une activité de copie qui est configurée pour utiliser ces jeux de données d'entrée et de sortie, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **FileSystemSource** et le type **sink** est défini sur **BlobSink**. La requête SQL spécifiée pour la propriété **query** sélectionne les données de la dernière heure à copier.

```JSON
{
    "name": "pipeline",
    "properties":
    {
        "activities":
        [
            {
                "name": "HdfsToBlobCopy",
                "inputs": [ {"name": "InputDataset"} ],
                "outputs": [ {"name": "OutputDataset"} ],
                "type": "Copy",
                "typeProperties":
                {
                    "source":
                    {
                        "type": "FileSystemSource"
                    },
                    "sink":
                    {
                        "type": "BlobSink"
                    }
                },
                "policy":
                {
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1,
                    "timeout": "00:05:00"
                }
            }
        ],
        "start": "2014-06-01T18:00:00Z",
        "end": "2014-06-01T19:00:00Z"
    }
}
```

## <a name="use-kerberos-authentication-for-hdfs-connector"></a>Utilisation de l’authentification Kerberos pour le connecteur HDFS
Il existe deux options de configuration de l’environnement local afin d’utiliser l’authentification Kerberos dans le connecteur HDFS. Vous pouvez choisir celle qui correspond mieux à votre situation.
* Option 1 : [Joindre l’ordinateur de la passerelle au domaine Kerberos](#kerberos-join-realm)
* Option n°2 : [Activer l’approbation mutuelle entre le domaine Windows et le domaine Kerberos](#kerberos-mutual-trust)

### <a name="option-1-join-gateway-machine-in-kerberos-realm"></a><a name="kerberos-join-realm"></a>Option 1 : Joindre l’ordinateur de la passerelle au domaine Kerberos

#### <a name="requirement"></a>Condition :

* L’ordinateur de passerelle doit rejoindre le domaine Kerberos et ne peut rejoindre aucun domaine Windows.

#### <a name="how-to-configure"></a>Pour effectuer la configuration :

**Sur l’ordinateur de la passerelle :**

1.  Exécutez l’utilitaire **Ksetup** pour configurer le serveur Kerberos KDC et le domaine.

    L’ordinateur doit être configuré en tant que membre d’un groupe de travail, car un domaine Kerberos est différent d’un domaine Windows. Pour ce faire, définissez le domaine Kerberos et ajoutez un serveur KDC comme suit. Remplacez *REALM.COM* par votre propre domaine respectif en fonction des besoins.

    ```cmd
    C:> Ksetup /setdomain REALM.COM
    C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
    ```

    **Redémarrez** l’ordinateur après avoir exécuté ces 2 commandes.

2.  Vérifiez la configuration avec la commande **Ksetup**. La sortie doit être semblable à :

    ```cmd
    C:> Ksetup
    default realm = REALM.COM (external)
    REALM.com:
        kdc = <your_kdc_server_address>
    ```

**Dans Azure Data Factory :**

* Configurez le connecteur HDFS à l’aide de l’**authentification Windows** avec votre nom principal Kerberos et le mot de passe pour vous connecter à la source de données HDFS. Vérifiez les détails de configuration dans la section sur les [propriétés du service lié HDFS](#linked-service-properties).

### <a name="option-2-enable-mutual-trust-between-windows-domain-and-kerberos-realm"></a><a name="kerberos-mutual-trust"></a>Option 2 : Activer l’approbation mutuelle entre le domaine Windows et le domaine Kerberos

#### <a name="requirement"></a>Condition :
*   L’ordinateur de passerelle doit rejoindre un domaine Windows.
*   Vous avez besoin d’autorisations pour mettre à jour les paramètres du contrôleur de domaine.

#### <a name="how-to-configure"></a>Pour effectuer la configuration :

> [!NOTE]
> Remplacez REALM.COM et AD.COM dans le didacticiel suivant par votre propre domaine et contrôleur de domaine respectifs en fonction des besoins.

**Sur le serveur KDC :**

1. Modifiez la configuration KDC dans le fichier **krb5.conf** afin de permettre au KDC d’approuver le domaine Windows faisant référence au modèle de configuration suivant. Par défaut, la configuration se trouve dans **/etc/krb5.conf**.

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

   **Redémarrez** le service KDC après la configuration.

2. Préparez un fichier principal nommé **krbtgt/REALM.COM\@AD.COM** dans le serveur KDC avec la commande suivante :

   ```cmd
   Kadmin> addprinc krbtgt/REALM.COM@AD.COM
   ```

3. Dans le fichier de configuration du service HDFS **hadoop.security.auth_to_local**, ajoutez `RULE:[1:$1@$0](.*\@AD.COM)s/\@.*//`.

**Sur le contrôleur de domaine :**

1.  Exécutez les commandes **Ksetup** suivantes pour ajouter une entrée de domaine :

    ```cmd
    C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
    C:> ksetup /addhosttorealmmap HDFS-service-FQDN REALM.COM
    ```

2.  Établir l’approbation entre le domaine Windows et le domaine Kerberos. [password] correspond au mot de passe pour le principal **krbtgt/REALM.COM\@AD.COM**.

    ```cmd
    C:> netdom trust REALM.COM /Domain: AD.COM /add /realm /passwordt:[password]
    ```

3.  Sélectionnez l’algorithme de chiffrement utilisé dans Kerberos.

    1. Accédez à Gestionnaire de serveur > Gestion des stratégies de groupe > Domaine > Objets de stratégie de groupe > Stratégie de domaine par défaut ou actif, puis Modifier.

    2. Dans la fenêtre contextuelle **Éditeur de gestion des stratégies de groupe**, accédez à Configuration ordinateur > Stratégies > Paramètres Windows > Paramètres de sécurité > Stratégies locales > Options de sécurité, puis configurez **Sécurité réseau : Configurer les types de chiffrement autorisés pour Kerberos**.

    3. Sélectionnez l’algorithme de chiffrement à utiliser pour vous connecter à KDC. En règle générale, vous pouvez simplement sélectionner toutes les options.

        ![Configuration des types de chiffrement pour Kerberos](media/data-factory-hdfs-connector/config-encryption-types-for-kerberos.png)

    4. Utilisez la commande **Ksetup** pour spécifier l’algorithme de chiffrement à utiliser dans le domaine spécifique.

       ```cmd
       C:> ksetup /SetEncTypeAttr REALM.COM DES-CBC-CRC DES-CBC-MD5 RC4-HMAC-MD5 AES128-CTS-HMAC-SHA1-96 AES256-CTS-HMAC-SHA1-96
       ```

4.  Créez le mappage entre le compte de domaine et le principal Kerberos afin de pouvoir utiliser le principal Kerberos dans un domaine Windows.

    1. Démarrez Outils d’administration > **Utilisateurs et ordinateurs Active Directory**.

    2. Pour configurer les fonctionnalités avancées, cliquez sur **Affichage** > **Fonctionnalités avancées**.

    3. Recherchez le compte pour lequel vous souhaitez créer des mappages, cliquez avec le bouton droit pour afficher **Mappages des noms** > cliquez sur l’onglet **Noms Kerberos**.

    4. Ajoutez un principal provenant du domaine.

        ![Mappage des identités de sécurité](media/data-factory-hdfs-connector/map-security-identity.png)

**Sur l’ordinateur de la passerelle :**

* Exécutez les commandes **Ksetup** suivantes pour ajouter une entrée de domaine.

   ```cmd
   C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
   C:> ksetup /addhosttorealmmap HDFS-service-FQDN REALM.COM
   ```

**Dans Azure Data Factory :**

* Configurez le connecteur HDFS à l’aide de l’**authentification Windows** avec votre compte de domaine ou le principal Kerberos pour vous connecter à la source de données HDFS. Vérifiez les détails de configuration dans la section sur les [propriétés du service lié HDFS](#linked-service-properties).

> [!NOTE]
> Pour savoir comment mapper des colonnes d’un jeu de données source sur des colonnes d’un jeu de données récepteur, consultez [Mappage de colonnes des jeux de données dans Azure Data Factory](data-factory-map-columns.md).


## <a name="performance-and-tuning"></a>Performances et réglage
Consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser.
