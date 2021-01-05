---
title: Déplacer des données vers et à partir de Stockage Blob Azure
description: 'Découvrez comment copier des données d’objets blob dans Azure Data Factory. Utilisez notre exemple : Guide pratique pour copier des données vers et à partir de Stockage Blob Azure et Azure SQL Database.'
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: bec8160f-5e07-47e4-8ee1-ebb14cfb805d
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/05/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: fa6e19fd9759d6e489d0945b5521a2e0ae3881e0
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462647"
---
# <a name="copy-data-to-or-from-azure-blob-storage-using-azure-data-factory"></a>Échanger des données avec le Stockage Blob Azure à l’aide d’Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-azure-blob-connector.md)
> * [Version 2 (version actuelle)](../connector-azure-blob-storage.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Connecteur Stockage Blob Azure dans V2](../connector-azure-blob-storage.md).


Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour échanger des données avec le Stockage Blob Azure. Il s’appuie sur l’article [Activités de déplacement des données](data-factory-data-movement-activities.md), qui présente une vue d’ensemble du déplacement de données avec l’activité de copie.

## <a name="overview"></a>Vue d’ensemble
Vous pouvez copier et coller les données à partir de tout magasin de données source pris en charge vers le Stockage Blob Azure, ou entre ce dernier et tout magasin de données récepteur pris en charge. Le tableau ci-dessous contient la liste des banques de données prises en charge en tant que sources ou récepteurs par l’activité de copie. Par exemple, vous pouvez déplacer des données **depuis** une base de données SQL Server ou une base de données dans Azure SQL Database **vers** un stockage Blob Azure. Et vous pouvez copier des données **depuis** le stockage Blob Azure **vers** Azure Synapse Analytics ou une collection Azure Cosmos DB.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="supported-scenarios"></a>Scénarios pris en charge
Vous pouvez copier les données **à partir du stockage Blob Azure** dans les magasins de données suivants :

[!INCLUDE [data-factory-supported-sink](../../../includes/data-factory-supported-sinks.md)]

Vous pouvez copier les données des magasins de données suivants **dans le stockage Blob Azure** :

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

> [!IMPORTANT]
> L’activité de copie prend en charge l’échange de données avec des comptes de stockage Azure à usage général et le Stockage Blob à chaud/froid. L’activité prend en charge la **lecture à partir d’objets blob de blocs, d’annexe ou de page**, mais l’**écriture vers des objets blob de blocs** uniquement. Le stockage Azure Premium n’est pas pris en charge en tant que récepteur, car il repose sur des objets blob de pages.
>
> L’activité de copie ne supprime pas les données de la source une fois celles-ci copiées vers la destination. Si vous devez supprimer les données une fois la copie effectuée, créez une [activité personnalisée](data-factory-use-custom-activities.md) pour supprimer les données, et utilisez l’activité dans le pipeline. Pour obtenir un exemple, consultez [Exemple de suppression d’objet blob ou de dossier sur GitHub](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/DeleteBlobFileFolderCustomActivity).

## <a name="get-started"></a>Bien démarrer
Vous pouvez créer un pipeline avec une activité de copie qui déplace les données vers/depuis un stockage blob Azure à l’aide de différents outils/API.

Le moyen le plus simple de créer un pipeline consiste à utiliser **l’Assistant Copie**. Cet article contient une [procédure pas à pas](#walkthrough-use-copy-wizard-to-copy-data-tofrom-blob-storage) pour la création d’un pipeline pour copier des données d’un emplacement de stockage Blob Azure vers un autre emplacement de stockage Blob Azure. Pour suivre un tutoriel permettant de créer un pipeline pour copier des données de Stockage Blob Azure vers Azure SQL Database, consultez [Tutoriel : Créer un pipeline à l’aide de l’Assistant de copie](data-factory-copy-data-wizard-tutorial.md).

Vous pouvez également utiliser les outils suivants pour créer un pipeline : **Visual Studio**, **Azure PowerShell**, **modèle Azure Resource Manager**, **.NET API** et **API REST**. Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [didacticiel sur l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Que vous utilisiez des outils ou des API, la création d’un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur implique les étapes suivantes :

1. Création d'une **fabrique de données**. Une fabrique de données peut contenir un ou plusieurs pipelines.
2. Création de **services liés** pour lier les magasins de données d’entrée et de sortie à votre fabrique de données. Par exemple, si vous copiez des données depuis un stockage d’objets blob Azure vers une base de données Azure SQL Database, vous créez deux services liés pour lier votre compte de stockage Azure et votre base de données Azure SQL Database à votre fabrique de données. Pour plus d’informations sur les propriétés de service lié qui sont spécifiques au stockage Blob Azure, consultez la section [Propriétés du service lié](#linked-service-properties).
2. Création de **jeux de données** pour représenter les données d’entrée et de sortie de l’opération de copie. Dans l’exemple mentionné dans la dernière étape, vous créez un jeu de données pour spécifier le conteneur d’objets blob et le dossier qui contient les données d’entrée. Vous créez aussi un autre jeu de données pour spécifier la table SQL dans Azure SQL Database qui contient les données copiées depuis le stockage d’objets blob. Pour plus d’informations sur les propriétés de jeu de données qui sont spécifiques au stockage Blob Azure, consultez la section [Propriétés du jeu de données](#dataset-properties).
3. Création d’un **pipeline** avec une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie. Dans l’exemple mentionné plus haut, vous utilisez BlobSource comme source et SqlSink comme récepteur pour l’activité de copie. De même, si vous copiez depuis Azure SQL Database vers Stockage Blob Azure, vous utilisez SqlSource et BlobSink dans l’activité de copie. Pour plus d’informations sur les propriétés de l’activité de copie qui sont spécifiques au stockage Blob Azure, consultez la section [Propriétés de l’activité de copie](#copy-activity-properties). Pour plus d’informations sur l’utilisation d’un magasin de données comme source ou comme récepteur, cliquez sur le lien de la section précédente de votre magasin de données.

Lorsque vous utilisez l’Assistant, les définitions JSON de ces entités Data Factory (services liés, jeux de données et pipeline) sont automatiquement créées pour vous. Lorsque vous utilisez des outils/API (à l’exception de l’API .NET), vous devez définir ces entités Data Factory au format JSON.  Pour obtenir des exemples comportant des définitions JSON pour les entités Data Factory utilisées pour copier les données vers ou à partir du Stockage Blob Azure, consultez la section [Exemples JSON](#json-examples-for-copying-data-to-and-from-blob-storage  ) de cet article.

Les sections suivantes offrent des informations détaillées sur les propriétés JSON utilisées pour définir les entités Data Factory propres au Stockage Blob Azure.

## <a name="linked-service-properties"></a>Propriétés du service lié
Deux types de services liés permettent de lier un stockage Azure à une fabrique de données Azure. Il s'agit de : service lié **AzureStorage** et service lié **AzureStorageSas**. Le service lié Azure Storage fournit à la fabrique de données un accès global à Azure Storage. Tandis que le service lié Azure de stockage SAP (signature d'accès partagé) fournit à la fabrique de données un accès restreint/limité dans le temps à Azure Storage. Il n'existe aucune différence entre ces deux services liés. Choisissez le service lié qui répond à vos besoins. Les sections suivantes expliquent plus en détail ces deux services liés.

[!INCLUDE [data-factory-azure-storage-linked-services](../../../includes/data-factory-azure-storage-linked-services.md)]

## <a name="dataset-properties"></a>Propriétés du jeu de données
Pour spécifier un jeu de données afin de représenter les données d’entrée ou de sortie dans un Stockage Blob Azure, vous devez définir la propriété de type du jeu de données sur **AzureBlob**. Définissez la propriété **linkedServiceName** du jeu de données sur le nom du service lié du Stockage Azure ou SAP Azure Storage.  Les propriétés de type du jeu de données spécifient le **conteneur d’objets blob** et le **dossier** dans le Stockage Blob.

Pour obtenir une liste complète des sections et propriétés JSON disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md). Les sections comme la structure, la disponibilité et la stratégie d'un jeu de données JSON sont similaires pour tous les types de jeux de données (SQL Azure, Azure Blob, Azure Table, etc.).

La fabrique de données prend en charge les valeurs de type .NET conformes CLS suivantes pour fournir des informations de type dans la section « structure » du schéma dans les sources de données de lecture telles qu’un objet blob Azure : Int16, Int32, Int64, Single, Double, Decimal, Byte[], Bool, String, Guid, Datetime, Datetimeoffset, Timespan. Data Factory effectue automatiquement les conversions de type lorsque vous déplacez des données d’un magasin de données source vers un magasin de données récepteur.

La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations sur l'emplacement, le format, etc. des données dans le magasin de données. La section typeProperties pour le jeu de données de type **AzureBlob** a les propriétés suivantes :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| folderPath |Chemin d'accès au conteneur et au dossier dans le stockage des objets Blobs. Exemple : monconteneurblob\mondossierblob\ |Oui |
| fileName |Le nom de l’objet Blob. fileName est facultatif et sensible à la casse.<br/><br/>Si vous spécifiez un nom de fichier, l’activité (y compris la copie) fonctionne sur l’objet Blob spécifique.<br/><br/>Lorsque fileName n’est pas spécifié, la copie inclut tous les objets Blob dans le paramètre folderPath du jeu de données d’entrée.<br/><br/>Quand **fileName** n’est pas spécifié pour un jeu de données de sortie et que **preserveHierarchy** n’est pas spécifié dans le récepteur d’activité, le nom du fichier généré est au format suivant : `Data.<Guid>.txt` (par exemple : : Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |Non |
| partitionedBy |partitionedBy est une propriété facultative. Vous pouvez l'utiliser pour spécifier un folderPath dynamique et le nom de fichier pour les données de série chronologique. Par exemple, folderPath peut être paramétré pour toutes les heures de données. Consultez [Utilisation de la section propriété partitionedBy](#using-partitionedby-property) pour obtenir plus d’informations et des exemples. |Non |
| format | Les types de formats suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Définissez la propriété **type** située sous Format sur l’une de ces valeurs. Pour en savoir plus, consultez les sections relatives à [format Text](data-factory-supported-file-and-compression-formats.md#text-format), [format Json](data-factory-supported-file-and-compression-formats.md#json-format), [format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si vous souhaitez **copier des fichiers en l’état** entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie. |Non |
| compression | Spécifiez le type et le niveau de compression pour les données. Types pris en charge : **GZip**, **Deflate**, **BZip2** et **ZipDeflate**. Niveaux pris en charge : **Optimal** et **Fastest**. Pour plus d’informations, consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |Non |

### <a name="using-partitionedby-property"></a>Utilisation de la propriété partitionedBy
Comme mentionné dans la section précédente, vous pouvez spécifier un folderPath et un fileName dynamiques pour les données de série chronologique avec la propriété **partitionedBy**, [les fonctions Data Factory et les variables système](data-factory-functions-variables.md).

Pour obtenir plus d’informations sur les jeux de données de série chronologique, la planification et les segments, consultez les articles [Création de jeux de données](data-factory-create-datasets.md) et [Planification et exécution](data-factory-scheduling-and-execution.md).

#### <a name="sample-1"></a>Exemple 1

```json
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```

Dans cet exemple, {Slice} est remplacé par la valeur de la variable système Data Factory SliceStart au format (AAAAMMJJHH) spécifié. SliceStart fait référence à l'heure de début du segment. folderPath est différent pour chaque segment. Par exemple : wikidatagateway/wikisampledataout/2014100103 ou wikidatagateway/wikisampledataout/2014100104.

#### <a name="sample-2"></a>Exemple 2

```json
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
Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les jeux de données d’entrée et de sortie et les stratégies sont disponibles pour tous les types d’activités. En revanche, les propriétés disponibles dans la section **typeProperties** de l’activité varient pour chaque type d’activité. Pour l’activité de copie, elles dépendent des types de sources et récepteurs. Si vous déplacez des données à partir d’un stockage blob Azure, vous définissez le type de source dans l’activité de copie sur **BlobSource**. De même, si vous déplacez des données vers un stockage blob Azure, vous définissez le type de récepteur dans l’activité de copie sur **BlobSink**. Cette section fournit une liste de propriétés prises en charge par BlobSource et BlobSink.

**BlobSource** prend en charge les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| recursive |Indique si les données sont lues de manière récursive dans les sous-dossiers ou uniquement dans le dossier spécifié. |True (valeur par défaut), False |Non |

**BlobSink** prend en charge les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| copyBehavior |Cette propriété définit le comportement de copie lorsque la source est BlobSource ou FileSystem. |<b>PreserveHierarchy</b> : conserve la hiérarchie des fichiers dans le dossier cible. Le chemin d’accès relatif du fichier source vers le dossier source est identique au chemin d’accès relatif du fichier cible vers le dossier cible.<br/><br/><b>FlattenHierarchy</b> : tous les fichiers du dossier source figurent dans le premier niveau du dossier cible. Le nom des fichiers cibles est généré automatiquement. <br/><br/><b>MergeFiles</b> : fusionne tous les fichiers du dossier source dans un même fichier. Si le nom de fichier/d’objet blob est spécifié, le nom de fichier fusionné est le nom spécifié. Dans le cas contraire, le nom de fichier est généré automatiquement. |Non |

**BlobSource** prend également en charge ces deux propriétés pour la compatibilité descendante.

* **treatEmptyAsNull** : spécifie s’il faut traiter une chaîne null ou vide en tant que valeur null.
* **skipHeaderLineCount** : spécifie le nombre de lignes à ignorer. Applicable uniquement quand le jeu de données d’entrée utilise TextFormat.

De même, **BlobSink** prend en charge la propriété suivante pour la compatibilité descendante.

* **blobWriterAddHeader** : spécifie s’il faut ajouter un en-tête de définitions de colonne lors de l’écriture dans un jeu de données de sortie.

Les jeux de données prennent désormais en charge les propriétés suivantes qui implémentent la même fonctionnalité : **treatEmptyAsNull**, **skipLineCount**, **firstRowAsHeader**.

Le tableau suivant fournit des recommandations sur l’utilisation de nouvelles propriétés de jeu de données pour remplacer ces propriétés de BlobSink et BlobSource.

| Propriété de l’activité de copie | Propriété du jeu de données |
|:--- |:--- |
| skipHeaderLineCount sur BlobSource |skipLineCount et firstRowAsHeader. Les lignes sont d’abord ignorées, puis la première ligne est lue en tant qu’en-tête. |
| treatEmptyAsNull sur BlobSource |treatEmptyAsNull sur le jeu de données d’entrée |
| blobWriterAddHeader sur BlobSink |firstRowAsHeader sur le jeu de données de sortie |

Consultez la section [Définition de TextFormat](data-factory-supported-file-and-compression-formats.md#text-format) pour plus d’informations sur ces propriétés.

### <a name="recursive-and-copybehavior-examples"></a>exemples de valeurs recursive et copyBehavior
Cette section décrit le comportement résultant de l’opération de copie pour différentes combinaisons de valeurs recursive et copyBehavior.

| recursive | copyBehavior | Comportement résultant |
| --- | --- | --- |
| true |preserveHierarchy |Pour un dossier source nommé Dossier1 et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible nommé Dossier1 est créé et structuré de la même manière que la source<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5. |
| true |flattenHierarchy |Pour un dossier source nommé Dossier1 et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible Dossier1 est créé et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier5 |
| true |mergeFiles |Pour un dossier source nommé Dossier1 et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible Dossier1 est créé et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Le contenu de Fichier1 + Fichier2 + Fichier3 + Fichier4 + Fichier5 est fusionné dans un fichier avec le nom de fichier généré automatiquement |
| false |preserveHierarchy |Pour un dossier source nommé Dossier1 et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible nommé Dossier1 est créé et structuré comme suit<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/><br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |
| false |flattenHierarchy |Pour un dossier source nommé Dossier1 et structuré comme suit :<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible nommé Dossier1 est créé et structuré comme suit<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier2<br/><br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |
| false |mergeFiles |Pour un dossier source nommé Dossier1 et structuré comme suit :<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible nommé Dossier1 est créé et structuré comme suit<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Le contenu de Fichier1 + Fichier2 est fusionné dans un fichier avec le nom de fichier généré automatiquement. nom généré automatiquement pour Fichier1<br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |

## <a name="walkthrough-use-copy-wizard-to-copy-data-tofrom-blob-storage"></a>Procédure pas à pas : Utiliser l’Assistant Copie pour copier des données vers/à partir de Stockage Blob
Examinons comment copier rapidement des données vers/depuis un stockage Blob Azure. Dans cette procédure pas à pas, les banques de données source et de destination sont du type Stockage Blob Azure. Le pipeline de cette procédure pas à pas copie des données entre les dossiers d’un même conteneur d’objets blob. Cette procédure pas à pas est volontairement simple pour vous montrer les paramètres ou les propriétés lorsque vous utilisez le Stockage Blob en tant que source ou récepteur.

### <a name="prerequisites"></a>Prérequis
1. Créez un **compte de stockage Azure** à usage général si vous n’en avez pas encore un. Vous utilisez le stockage d’objets blob comme banque de données **source** et de **destination** dans cette procédure pas à pas. Si vous n’avez pas de compte de stockage Azure, consultez l’article [Créer un compte de stockage](../../storage/common/storage-account-create.md) pour découvrir comment en créer un.
2. Créez un conteneur d’objets blob nommé **adfblobconnector** dans le compte de stockage.
4. Créez un dossier nommé **input** dans le conteneur **adfblobconnector**.
5. Créez un fichier nommé **emp.txt** avec le contenu suivant et chargez-le dans le dossier **input** à l’aide d’outils tels [qu’Azure Storage Explorer (Explorateur du Stockage Azure)](https://azurestorageexplorer.codeplex.com/).
    ```json
    John, Doe
    Jane, Doe
    ```

### <a name="create-the-data-factory"></a>Création de la fabrique de données
1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Cliquez sur **Créer une ressource** en haut à gauche, cliquez sur **Intelligence + analyse**, puis sur **Data Factory**.
3. Dans le volet **Nouvelle fabrique de données** :  
    1. Entrez **ADFBlobConnectorDF** comme **nom**. Le nom de la fabrique de données Azure doit être un nom global unique. Si l’erreur `*Data factory name “ADFBlobConnectorDF” is not available` s’affiche, changez le nom de la fabrique de données (par exemple, votrenomADFTutorialDataFactory), puis tentez de la recréer. Consultez la rubrique [Data Factory - Règles d’affectation des noms](data-factory-naming-rules.md) pour savoir comment nommer les artefacts Data Factory.
    2. Sélectionnez votre **abonnement** Azure.
    3. Pour le groupe de ressources, sélectionnez **Utiliser l’existant** pour sélectionner un groupe de ressources existant (ou) sélectionnez **Créer un nouveau** pour entrer le nom d’un groupe de ressources.
    4. Sélectionnez un **emplacement** pour la fabrique de données.
    5. Sélectionnez la case à cocher **Épingler au tableau de bord** en bas du panneau.
    6. Cliquez sur **Créer**.
3. Une fois la création terminée, le panneau **Data Factory** s’affiche comme sur l’image suivante :  ![Page d’accueil Data Factory](./media/data-factory-azure-blob-connector/data-factory-home-page.png)

### <a name="copy-wizard"></a>Assistant de copie
1. Dans la page d’accueil Data Factory, cliquez sur la vignette **Copier les données** pour lancer **l’Assistant Copier des données** dans un onglet distinct.  

    > [!NOTE]
    > Si vous voyez que le navigateur web est bloqué au niveau « Autorisation... », désactivez/décochez l’option **Block third party cookies and site data** (Bloquer les cookies et les données de site tiers) (ou) laissez cette option activée et créez une exception pour **login.microsoftonline.com**, puis essayez de relancer l’Assistant.
2. Dans la page **Propriétés** :
    1. Entrez **CopyPipeline** comme **Nom de la tâche**. Le nom de la tâche correspond au nom du pipeline de votre fabrique de données.
    2. Entrez une **description** pour la tâche (facultative).
    3. Pour **Task cadence or Task schedule (Cadence ou planification des tâches)** , conservez l’option **Run regularly on schedule (Exécuter régulièrement selon la planification)** . Si vous souhaitez exécuter cette tâche une seule fois au lieu de l’exécuter à plusieurs reprises selon une planification, sélectionnez **Run once now (Exécuter une seule fois maintenant)** . Si vous sélectionnez l’option **Run once now (Exécuter une seule fois maintenant)** , un [pipeline à usage unique](data-factory-create-pipelines.md#onetime-pipeline) est créé.
    4. Conservez les paramètres pour le **Modèle récurrent**. Cette tâche s’exécute tous les jours entre les heures de début et de fin que vous spécifiez à l’étape suivante.
    5. Modifiez la **Date et heure de début** et définissez-la sur **21/04/2017**.
    6. Modifiez la **Date et heure de fin** et définissez-la sur **25/04/2017**. Vous souhaiterez peut-être taper la date au lieu de parcourir le calendrier.
    8. Cliquez sur **Suivant**.
        ![Outil de copie - Page Propriétés](./media/data-factory-azure-blob-connector/copy-tool-properties-page.png)
3. Dans la page **Source data store** (Magasin de données source), cliquez sur la vignette **Stockage d’objets blob Azure**. Cette page sert à spécifier le magasin de données source pour la tâche de copie. Vous pouvez utiliser un service lié de magasin de données existant (ou) spécifier un nouveau magasin de données. Pour utiliser un service lié existant, vous devez sélectionner **FROM EXISTING LINKED SERVICES** (À PARTIR DES SERVICES LIÉS EXISTANTS) et le service lié approprié.
    ![Outil de copie - Page Banque de données sources](./media/data-factory-azure-blob-connector/copy-tool-source-data-store-page.png)
4. Dans la page **Specify the Azure Blob storage account** (Spécifier le compte de stockage d’objets blob Azure) :
    1. Conservez le nom généré automatiquement comme **Nom de la connexion**. Le nom de la connexion est le nom du service lié de type Stockage Azure.
    2. Vérifiez que l’option **À partir des abonnements** est sélectionnée pour **Account selection method** (Méthode de sélection du compte).
    3. Sélectionnez votre abonnement Azure ou conservez **Sélectionner tout** pour l’option **Abonnement Azure**.
    4. Sélectionnez un **compte de stockage Azure** dans la liste des comptes de stockage Azure disponibles dans l’abonnement sélectionné. Vous pouvez également choisir d’entrer manuellement les paramètres du compte de stockage en sélectionnant l’option **Entrer manuellement** dans **Account selection method** (Méthode de sélection de compte).
    5. Cliquez sur **Suivant**.  
        ![Outil de copie - Spécifier le compte de stockage Blob Azure](./media/data-factory-azure-blob-connector/copy-tool-specify-azure-blob-storage-account.png)
5. Dans la page **Choose the input file or folder** (Choisir le fichier ou le dossier d’entrée) :
    1. Double-cliquez sur **adfblobcontainer**.
    2. Sélectionnez **input**, puis cliquez sur **Choisir**. Dans cette procédure pas à pas, vous sélectionnez le dossier d’entrée. Vous pouvez également sélectionner à la place le fichier emp.txt dans le dossier.
        ![Outil de copie - Choisir le fichier ou le dossier d’entrée 1](./media/data-factory-azure-blob-connector/copy-tool-choose-input-file-or-folder.png)
6. Dans la page **Choose the input file or folder** (Choisir le fichier ou le dossier d’entrée) :
    1. Vérifiez que le **fichier ou dossier** est défini sur **adfblobconnector/input**. Si les fichiers se trouvent dans des sous-dossiers, par exemple, 01/04/2017, 02/04/2017 et ainsi de suite, entrez adfblobconnector/input /{jour}/{mois}/{année} pour le fichier ou le dossier. Si vous appuyez sur TAB hors de la zone de texte, trois listes déroulantes s’affichent. Elles permettent de sélectionner les formats d’année (aaaa), de mois (MM) et de jour (jj).
    2. Ne définissez pas **Copy file recursively (Copier le fichier de façon récursive)** . Sélectionnez cette option pour rechercher de manière récursive dans les dossiers les fichiers à copier vers la destination.
    3. Ne sélectionnez pas l’option **Copie binaire**. Sélectionnez cette option pour effectuer une copie binaire du fichier source vers la destination. Ne sélectionnez pas cette option pour la procédure pas à pas afin de pouvoir afficher d’autres options dans les pages suivantes.
    4. Vérifiez que l’option **Type de compression** est définie sur **Aucune**. Sélectionnez une valeur pour cette option si vos fichiers sources sont compressés dans l’un des formats pris en charge.
    5. Cliquez sur **Suivant**.
    ![Outil de copie - Choisir le fichier ou le dossier d’entrée 2](./media/data-factory-azure-blob-connector/chose-input-file-folder.png)
7. Dans la page **File format settings** (Paramètres de format de fichier), vous pouvez voir les délimiteurs et le schéma qui sont détectés automatiquement par l’Assistant en analysant le fichier.
    1. Vérifiez les options suivantes :  
        a. L’option **Format de fichier** est définie sur **Format du texte**. La liste déroulante contient tous les formats pris en charge. Par exemple : JSON, Avro, ORC, Parquet.
       b. L’option **Séparateur de colonnes** est définie sur `Comma (,)`. Vous pouvez voir les autres séparateurs de colonnes pris en charge par Data Factory dans la liste déroulante. Vous pouvez également spécifier un séparateur personnalisé.
       c. L’option **Séparateur de lignes** est définie sur `Carriage Return + Line feed (\r\n)`. Vous pouvez voir les autres séparateurs de lignes pris en charge par Data Factory dans la liste déroulante. Vous pouvez également spécifier un séparateur personnalisé.
       d. L’option **Skip line count** (Nombre de lignes à ignorer) est définie sur **0**. Entrez ici le nombre de lignes que vous souhaitez ignorer en haut du fichier.
       e. L’option **First data row contains column names** (La première ligne de données contient des noms de colonne) n’est pas cochée. Si les fichiers sources contiennent des noms de colonne dans la première ligne, sélectionnez cette option.
       f. L’option **Treat empty column value as null** (Traiter une colonne vide comme valeur Null) est cochée.
    2. Développez **Paramètres avancés** pour afficher les options avancées disponibles.
    3. En bas de la page, affichez **l’aperçu** des données du fichier emp.txt.
    4. Cliquez sur l’onglet **SCHÉMA** en bas de la page pour afficher le schéma déduit par l’Assistant Copie en examinant les données du fichier source.
    5. Une fois que vous avez vérifié les délimiteurs et afficher un aperçu des données, cliquez sur **Suivant**.
    ![Outil de copie - Paramètres de format de fichier](./media/data-factory-azure-blob-connector/copy-tool-file-format-settings.png)
8. Dans la **page de la banque de données de destination**, sélectionnez **Stockage Blob Azure**, puis cliquez sur **Suivant**. Vous utilisez le Stockage Blob Azure comme banques de données source et de destination dans cette procédure pas à pas.  
    ![Outil de copie - Sélectionner la banque de données de destination](media/data-factory-azure-blob-connector/select-destination-data-store.png)
9. Dans la page **Specify the Azure Blob storage account** (Spécifier le compte de stockage Blob Azure) :  
    1. Entrez **AzureStorageLinkedService** dans le champ **Nom de la connexion**.
    2. Vérifiez que l’option **À partir des abonnements** est sélectionnée pour **Account selection method** (Méthode de sélection du compte).
    3. Sélectionnez votre **abonnement** Azure.
    4. Sélectionnez votre compte de stockage Azure.
    5. Cliquez sur **Suivant**.
10. Dans la page **Choose the output file or folder** (Choisir le fichier ou le dossier de sortie) :  
    1. spécifiez le **chemin d’accès du dossier** **adfblobconnector/output/{année}/{mois}/{jour}** . Entrez **TAB**.
    1. Pour **l’année**, sélectionnez **aaaa**.
    1. Pour le **mois**, vérifiez que le paramètre est défini sur **MM**.
    1. Pour le **jour**, vérifiez que le paramètre est défini sur **jj**.
    1. Vérifiez que l’option **Type de compression** est définie sur **Aucune**.
    1. Vérifiez que l’option **Copy behavior (Comportement de copie)** est définie sur **Merge files (Fusionner les fichiers)** . Si un fichier de sortie du même nom existe déjà, le nouveau contenu est ajouté à la fin du même fichier.
    1. Cliquez sur **Suivant**.
       ![Outil de copie - Choisir le fichier ou le dossier de sortie](media/data-factory-azure-blob-connector/choose-the-output-file-or-folder.png)
11. Dans la page **Paramètres de format de fichier**, passez en revue les paramètres, puis cliquez sur **Suivant**. L’une des options supplémentaires ici consiste à ajouter un en-tête au fichier de sortie. Si vous sélectionnez cette option, une ligne d’en-tête est ajoutée avec les noms des colonnes du schéma de la source. Vous pouvez renommer les noms de colonne par défaut lorsque vous affichez le schéma de la source. Par exemple, vous pouvez modifier les première et deuxième colonnes et les définir respectivement sur Prénom et Nom. Le fichier de sortie est alors généré avec un en-tête contenant ces noms comme noms de colonne.
    ![Outil de copie - Paramètres de format de fichier pour la destination](media/data-factory-azure-blob-connector/file-format-destination.png)
12. Dans la page **Paramètres de performances**, vérifiez que les paramètres **cloud units (Unités cloud)** et **parallel copies (Copies parallèles)** sont définis sur **Auto**, puis cliquez sur Suivant. Pour plus d’informations sur ces paramètres, consultez le [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md#parallel-copy).
    ![Outil de copie - Paramètres de performances](media/data-factory-azure-blob-connector/copy-performance-settings.png)
14. Dans la page **Résumé**, passez en revue tous les paramètres (propriétés de tâche, paramètres de source et de destination, et paramètres de copie), puis cliquez sur **Suivant**.
    ![Outil de copie - Page Résumé](media/data-factory-azure-blob-connector/copy-tool-summary-page.png)
15. Passez en revue les informations contenues dans la page **Résumé**, puis cliquez sur **Terminer**. L’Assistant crée deux services liés, deux jeux de données (entrée et sortie) et un pipeline dans la fabrique de données (d’où vous avez lancé l’Assistant Copie).
    ![Outil de copie - Page Déploiement](media/data-factory-azure-blob-connector/copy-tool-deployment-page.png)

### <a name="monitor-the-pipeline-copy-task"></a>Surveiller le pipeline (tâche de copie)

1. Cliquez sur le lien `Click here to monitor copy pipeline` dans la page **Déploiement**.
2. **L’application Surveiller et Gérer** doit apparaître dans un onglet distinct.  ![Application Surveiller et gérer](media/data-factory-azure-blob-connector/monitor-manage-app.png)
3. Modifiez l’heure de **début** indiquée en haut sur `04/19/2017` et l’heure de **fin** sur `04/27/2017`, puis cliquez sur **Appliquer**.
4. La liste **FENÊTRES D’ACTIVITÉ** doit contenir cinq fenêtres d’activité. Les heures **WindowStart** doivent couvrir tous les jours, de l’heure de début du pipeline à son heure de fin.
5. Cliquez plusieurs fois sur le bouton **Actualiser** correspondant à la liste **FENÊTRES D’ACTIVITÉ** jusqu’à ce que le statut de toutes les fenêtres d’activité soit défini sur Prêt.
6. À présent, vérifiez que les fichiers de sortie sont générés dans le dossier de sortie du conteneur adfblobconnector. Vous devez voir la structure des dossiers suivante dans le dossier de sortie :
    ```
    2017/04/21
    2017/04/22
    2017/04/23
    2017/04/24
    2017/04/25
    ```
   Pour plus d’informations sur la surveillance et la gestion des fabriques de données, consultez l’article [Monitor and manage Data Factory pipeline (Surveiller et gérer le pipeline Data Factory)](data-factory-monitor-manage-app.md).

### <a name="data-factory-entities"></a>Entités Data Factory
À présent, revenez dans l’onglet contenant la page d’accueil Data Factory. Notez qu’il existe désormais deux services liés, deux jeux de données et un pipeline dans votre fabrique de données.

![Page d’accueil Data Factory comportant des entités](media/data-factory-azure-blob-connector/data-factory-home-page-with-numbers.png)

Cliquez sur **Créer et déployer** pour démarrer Data Factory Editor.

![Data Factory Editor](media/data-factory-azure-blob-connector/data-factory-editor.png)

Votre fabrique de données doit contenir les entités Data Factory suivantes :

- Deux services liés, l’un pour la source et l’autre pour la destination. Les deux services liés désignent le même compte de stockage Azure dans cette procédure pas à pas.
- Deux jeux de données, un jeu de données d’entrée et un jeu de données de sortie. Dans cette procédure pas à pas, les deux utilisent le même conteneur d’objets blob, mais désignent des dossiers différents (d’entrée et de sortie).
- Un pipeline. Le pipeline contient une activité de copie qui utilise une source d’objet blob et un récepteur d’objet blob pour copier les données d’un emplacement d’objet blob Azure vers un autre emplacement d’objet blob Azure.

Les sections qui suivent fournissent des informations supplémentaires sur ces entités.

#### <a name="linked-services"></a>Services liés
Vous devez voir deux services liés, l’un pour la source et l’autre pour la destination. Dans cette procédure pas à pas, les deux définitions sont identiques, à l’exception de leurs noms. Le **type** du service lié est défini sur **AzureStorage**. **connectionString** est la propriété la plus importante de la définition de service lié. Elle est utilisée par Data Factory pour se connecter à votre compte de stockage Azure lors de l’exécution. Ignorez la propriété hubName dans la définition.

##### <a name="source-blob-storage-linked-service"></a>Service lié du stockage d’objets blob source
```json
{
    "name": "Source-BlobStorage-z4y",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=mystorageaccount;AccountKey=**********"
        }
    }
}
```

##### <a name="destination-blob-storage-linked-service"></a>Service lié du stockage d’objets blob de destination

```json
{
    "name": "Destination-BlobStorage-z4y",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=mystorageaccount;AccountKey=**********"
        }
    }
}
```

Pour plus d’informations sur le service lié Stockage Azure, consultez la section [Propriétés du service lié](#linked-service-properties).

#### <a name="datasets"></a>Groupes de données
Il existe deux jeux de données : un jeu de données d’entrée et un jeu de données de sortie. Le type des deux jeux de données est défini sur **AzureBlob**.

Le jeu de données d’entrée pointe vers le dossier **input** du conteneur d’objets blob **adfblobconnector**. La propriété **externe** est définie sur **true** pour ce jeu de données, car les données ne sont pas produites par le pipeline avec l’activité de copie qui utilise ce jeu de données en tant qu’entrée.

Le jeu de données de sortie pointe vers le dossier **output** du même conteneur d’objets blob. Le jeu de données de sortie utilise également l’année, le mois et le jour de la variable système **SliceStart** pour évaluer dynamiquement le chemin d’accès du fichier de sortie. Pour obtenir la liste des fonctions et variables système prises en charge par Data Factory, consultez [Variables système et fonctions Data Factory](data-factory-functions-variables.md). La propriété **externe** est définie sur **false** (valeur par défaut), car ce jeu de données est produit par le pipeline.

Pour plus d’informations sur les propriétés prises en charge par le jeu de données d’objets blob Azure, consultez la section [Propriétés du jeu de données](#dataset-properties).

##### <a name="input-dataset"></a>Jeu de données d'entrée

```json
{
    "name": "InputDataset-z4y",
    "properties": {
        "structure": [
            { "name": "Prop_0", "type": "String" },
            { "name": "Prop_1", "type": "String" }
        ],
        "type": "AzureBlob",
        "linkedServiceName": "Source-BlobStorage-z4y",
        "typeProperties": {
            "folderPath": "adfblobconnector/input/",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```

##### <a name="output-dataset"></a>Jeu de données de sortie

```json
{
    "name": "OutputDataset-z4y",
    "properties": {
        "structure": [
            { "name": "Prop_0", "type": "String" },
            { "name": "Prop_1", "type": "String" }
        ],
        "type": "AzureBlob",
        "linkedServiceName": "Destination-BlobStorage-z4y",
        "typeProperties": {
            "folderPath": "adfblobconnector/output/{year}/{month}/{day}",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            },
            "partitionedBy": [
                { "name": "year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
                { "name": "month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
                { "name": "day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } }
            ]
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        },
        "external": false,
        "policy": {}
    }
}
```

#### <a name="pipeline"></a>Pipeline
Le pipeline contient une seule activité. Définissez le **type** de l’activité sur **Copier**. Dans les propriétés de type de l’activité, il existe deux sections, l’une pour la source et l’autre pour le récepteur. Le type de source est défini sur **BlobSource**, car l’activité copie les données d’un stockage d’objets blob. Le type de récepteur est défini sur **BlobSink**, car l’activité copie les données dans un stockage d’objets blob. L’activité de copie utilise InputDataset-z4y comme entrée et OutputDataset-z4y comme sortie.

Pour plus d’informations sur les propriétés prises en charge par BlobSource et BlobSink, consultez la section [Propriétés de l’activité de copie](#copy-activity-properties).

```json
{
    "name": "CopyPipeline",
    "properties": {
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource",
                        "recursive": false
                    },
                    "sink": {
                        "type": "BlobSink",
                        "copyBehavior": "MergeFiles",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataset-z4y"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataset-z4y"
                    }
                ],
                "policy": {
                    "timeout": "1.00:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "style": "StartOfInterval",
                    "retry": 3,
                    "longRetry": 0,
                    "longRetryInterval": "00:00:00"
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "Activity-0-Blob path_ adfblobconnector_input_->OutputDataset-z4y"
            }
        ],
        "start": "2017-04-21T22:34:00Z",
        "end": "2017-04-25T05:00:00Z",
        "isPaused": false,
        "pipelineMode": "Scheduled"
    }
}
```

## <a name="json-examples-for-copying-data-to-and-from-blob-storage"></a>Exemples JSON pour copier des données vers et depuis le stockage Blob
Les exemples suivants présentent des exemples de définitions JSON que vous pouvez utiliser pour créer un pipeline à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou d’[Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Ils indiquent comment copier des données vers et depuis le stockage Blob Azure et Azure SQL Database. Toutefois, les données peuvent être copiées **directement** vers l’un des récepteurs indiqués [ici](data-factory-data-movement-activities.md#supported-data-stores-and-formats) , via l’activité de copie de Microsoft Azure Data Factory.

### <a name="json-example-copy-data-from-blob-storage-to-sql-database"></a>Exemple JSON : Copier des données de Stockage Blob vers SQL Database
L’exemple suivant montre :

1. Un service lié de type [AzureSqlDatabase](data-factory-azure-sql-connector.md#linked-service-properties).
2. Un service lié de type [AzureStorage](#linked-service-properties).
3. un [jeu de données](data-factory-create-datasets.md) d'entrée de type [AzureBlob](#dataset-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureSqlTable](data-factory-azure-sql-connector.md#dataset-properties).
5. Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [BlobSource](#copy-activity-properties) et [SqlSink](data-factory-azure-sql-connector.md#copy-activity-properties).

L’exemple copie des données de série horaire à partir d’un objet blob Azure vers une table SQL Azure, toutes les heures. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

**Service lié SQL Azure :**

```json
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
**Service lié Azure Storage :**

```json
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Azure Data Factory prend en charge deux types de service lié Stockage Azure : **AzureStorage** et **AzureStorageSas**. Pour le premier, vous spécifiez la chaîne de connexion qui inclut la clé de compte, et pour le second, vous spécifiez l'Uri de signature d’accès partagé (SAP). Pour plus d’informations, consultez la section [Services liés](#linked-service-properties) .

**Jeu de données d'entrée d'objet Blob Azure :**

Les données sont récupérées à partir d’un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1). Le nom du chemin d'accès et du fichier de dossier pour l'objet blob sont évalués dynamiquement en fonction de l'heure de début du segment en cours de traitement. Le chemin d’accès du dossier utilise l’année, le mois et le jour de début et le nom de fichier utilise l’heure de début. Le paramètre « external » : « true » informe Data Factory que la table est externe à la Data Factory et non produite par une activité dans la Data Factory.

```json
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/",
      "fileName": "{Hour}.csv",
      "partitionedBy": [
        { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
        { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
        { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
        { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "HH" } }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": "\n"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
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
**Jeu de données de sortie SQL Azure :**

L'exemple copie les données dans une table nommée « MyTable » dans Azure SQL Database. Créez la table dans votre base de données SQL avec le même nombre de colonnes que le fichier CSV d'objet Blob doit contenir. De nouvelles lignes sont ajoutées à la table toutes les heures.

```json
{
  "name": "AzureSqlOutput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
**Activité de copie dans un pipeline avec une source blob et un récepteur SQL :**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d'entrée et de sortie, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **BlobSource** et le type **sink** est défini sur **SqlSink**.

```json
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[
      {
        "name": "AzureBlobtoSQL",
        "description": "Copy Activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSqlOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink"
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
### <a name="json-example-copy-data-from-azure-sql-to-azure-blob"></a>Exemple JSON : Copier des données de SQL Azure vers Stockage Blob Azure
L’exemple suivant montre :

1. Un service lié de type [AzureSqlDatabase](data-factory-azure-sql-connector.md#linked-service-properties).
2. Un service lié de type [AzureStorage](#linked-service-properties).
3. Un [jeu de données](data-factory-create-datasets.md) d'entrée de type [AzureSqlTable](data-factory-azure-sql-connector.md#dataset-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](#dataset-properties).
5. Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [SqlSource](data-factory-azure-sql-connector.md#copy-activity-properties) et [BlobSink](#copy-activity-properties).

L’exemple copie des données de série horaire à partir d’une table SQL Azure vers un objet blob Azure, toutes les heures. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

**Service lié SQL Azure :**

```json
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
**Service lié Azure Storage :**

```json
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Azure Data Factory prend en charge deux types de service lié Stockage Azure : **AzureStorage** et **AzureStorageSas**. Pour le premier, vous spécifiez la chaîne de connexion qui inclut la clé de compte, et pour le second, vous spécifiez l'Uri de signature d’accès partagé (SAP). Pour plus d’informations, consultez la section [Services liés](#linked-service-properties) .

**Jeu de données d'entrée SQL Azure :**

L'exemple suppose que vous avez créé une table « MyTable » dans SQL Azure et qu'elle contient une colonne appelée « timestampcolumn » pour les données de série chronologique.

La définition de external sur true informe le service Data Factory que la table est externe à la Data Factory et non produite par une activité dans la Data Factory.

```json
{
  "name": "AzureSqlInput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
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

**Jeu de données de sortie d'objet Blob Azure :**

Les données sont écrites dans un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1). Le chemin d’accès du dossier pour l’objet blob est évalué dynamiquement en fonction de l’heure de début du segment en cours de traitement. Le chemin d'accès du dossier utilise l'année, le mois, le jour et l'heure de l'heure de début.

```json
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}/",
      "partitionedBy": [
        {
          "name": "Year",
          "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
        { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
        { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
        { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "HH" } }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**Activité de copie dans un pipeline avec une source SQL et un récepteur blob :**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d'entrée et de sortie, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **SqlSource** et le type **sink** est défini sur **BlobSink**. La requête SQL spécifiée pour la propriété **SqlReaderQuery** sélectionne les données de la dernière heure à copier.

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[
            {
                "name": "AzureSQLtoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureSQLInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "SqlSource",
                        "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
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

> [!NOTE]
> Pour savoir comment mapper des colonnes d’un jeu de données source sur des colonnes d’un jeu de données récepteur, consultez [Mappage de colonnes des jeux de données dans Azure Data Factory](data-factory-map-columns.md).

## <a name="performance-and-tuning"></a>Performances et réglage
Consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser.
