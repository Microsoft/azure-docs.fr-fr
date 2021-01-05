---
title: Créer des jeux de données dans Azure Data Factory
description: Apprenez à créer des jeux de données dans Azure Data Factory, en vous appuyant sur des exemples qui utilisent des propriétés de type offset et anchorDateTime.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 9bf6ff2971de57338dc299d48e24f6ffebd4b6b5
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96495937"
---
# <a name="datasets-in-azure-data-factory-version-1"></a>Jeux de données dans Azure Data Factory (version 1)
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-create-datasets.md)
> * [Version 2 (version actuelle)](../concepts-datasets-linked-services.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Jeux de données dans V2](../concepts-datasets-linked-services.md).

Cet article décrit les jeux de données, comment ils sont définis au format JSON et comment ils sont utilisés dans les pipelines d’Azure Data Factory. Il fournit des détails sur chaque section (par exemple, structure, disponibilité et stratégie) dans la définition JSON du jeu de données. Cet article fournit également des exemples d’utilisation des propriétés **offset**, **anchorDateTime** et **style** dans une définition JSON de jeu de données.

> [!NOTE]
> Si vous débutez avec Data Factory, consultez [Présentation d’Azure Data Factory](data-factory-introduction.md) pour obtenir une vue d’ensemble. Si vous n’avez pas d’expérience pratique de la création de fabriques de données, vous pouvez obtenir une meilleure compréhension en lisant le [tutoriel de transformation des données](data-factory-build-your-first-pipeline.md) et le [tutoriel de déplacement des données](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

## <a name="overview"></a>Vue d’ensemble
Une fabrique de données peut avoir un ou plusieurs pipelines. Un **pipeline** constitue un regroupement logique d’**activités** qui exécutent ensemble une tâche. Les activités d’un pipeline définissent les actions à effectuer sur les données. Par exemple, vous pouvez utiliser une activité de copie pour copier des données d’une base de données SQL Server vers un stockage Blob Azure. Ensuite, vous pouvez utiliser une activité Hive qui exécute un script Hive sur un cluster Azure HDInsight pour traiter les données du stockage Blob afin de produire des données de sortie. Enfin, vous pouvez utiliser une deuxième activité de copie pour copier les données de sortie dans Azure Synapse Analytics sur lequel des solutions de génération de rapports d’aide à la décision sont développées. Pour plus d’informations sur les pipelines et les activités, voir [Pipelines et activités dans Azure Data Factory](data-factory-create-pipelines.md).

Une activité peut inclure zéro ou plusieurs **jeux de données** d’entrée et produire un ou plusieurs jeux de données de sortie. Un jeu de données d’entrée représente l’entrée d’une activité dans le pipeline, tandis qu’un jeu de données de sortie représente la sortie de l’activité. Les jeux de données identifient les données dans différents magasins de données, par exemple des tables, des fichiers, des dossiers et des documents. Par exemple, un jeu de données d’objets Blob Azure spécifie le conteneur et le dossier du stockage Blob à partir duquel le pipeline doit lire les données.

Avant de créer un jeu de données, créez un **service lié** pour lier votre magasin de données à la fabrique de données. Les services liés ressemblent à des chaînes de connexion. Ils définissent les informations de connexion nécessaires à Data Factory pour se connecter à des ressources externes. Les jeux de données identifient données dans les magasins de données liés, par exemple des tables SQL, des fichiers, des dossiers et des documents. Par exemple, un service lié de stockage Azure relie un compte de stockage à la fabrique de données. Un jeu de données blob Azure représente le conteneur d’objets blob et le dossier qui contient les objets blob d’entrée à traiter.

Voici un exemple de scénario. Pour copier des données du stockage Blob vers une base de données SQL Database, vous devez créer deux services liés : Stockage Azure et Azure SQL Database. Créez ensuite deux jeux de données : le jeu de données d’objets blob Azure (qui fait référence au service lié Stockage Azure) et le jeu de données de table SQL Azure (qui fait référence au service lié Azure SQL Database). Les services liés Stockage Azure et Azure SQL Database contiennent des chaînes de connexion utilisées par la fabrique de données pendant l’exécution pour se connecter à votre instance Stockage Azure et Azure SQL Database, respectivement. Le jeu de données d’objets blob Azure spécifie le conteneur d’objets blob et le dossier d’objets blob qui contient les objets blob d’entrée dans votre stockage Blob. Le jeu de données de table SQL Azure spécifie la table SQL dans votre base de données SQL dans laquelle les données doivent être copiées.

Le diagramme suivant montre la relation entre le pipeline, l’activité, le jeu de données et le service lié dans la fabrique de données :

![Relation entre le pipeline, l’activité, le jeu de données et les services liés](media/data-factory-create-datasets/relationship-between-data-factory-entities.png)

## <a name="dataset-json"></a>Jeu de données JSON
Un jeu de données dans la fabrique de données est défini au format JSON comme suit :

```json
{
    "name": "<name of dataset>",
    "properties": {
        "type": "<type of dataset: AzureBlob, AzureSql etc...>",
        "external": "<boolean flag to indicate external data. only for input datasets>",
        "linkedServiceName": "<Name of the linked service that refers to a data store.>",
        "structure": [
            {
                "name": "<Name of the column>",
                "type": "<Name of the type>"
            }
        ],
        "typeProperties": {
            "<type specific property>": "<value>",
            "<type specific property 2>": "<value 2>",
        },
        "availability": {
            "frequency": "<Specifies the time unit for data slice production. Supported frequency: Minute, Hour, Day, Week, Month>",
            "interval": "<Specifies the interval within the defined frequency. For example, frequency set to 'Hour' and interval set to 1 indicates that new data slices should be produced hourly>"
        },
        "policy":
        {
        }
    }
}
```

La table suivante décrit les propriétés dans le JSON ci-dessus :

| Propriété | Description | Obligatoire | Default |
| --- | --- | --- | --- |
| name |Nom du jeu de données Pour connaître les règles d’affectation des noms, voir [Azure Data Factory - Règles d’affectation des noms](data-factory-naming-rules.md). |Oui |N/D |
| type |Type du jeu de données. Spécifiez l’un des types pris en charge par la fabrique de données (par exemple : AzureBlob, AzureSqlTable). <br/><br/>Pour plus d’informations, consultez [Type du jeu de données](#Type). |Oui |N/D |
| structure |Schéma du jeu de données.<br/><br/>Pour plus d’informations, consultez [Structure d’un jeu de données](#Structure). |Non |N/D |
| typeProperties | Les propriétés de type sont différentes pour chaque type (par exemple : objet blob Azure, table SQL Azure). Pour plus d’informations sur les types pris en charge et leurs propriétés, consultez [Type du jeu de données](#Type). |Oui |N/D |
| external | Indicateur booléen pour indiquer si un jeu de données est explicitement généré par un pipeline de fabrique de données ou non. Si le jeu de données d’entrée d’une activité n’est pas produit par le pipeline actuel, définissez cet indicateur sur true. Définissez cet indicateur sur true pour le jeu de données d’entrée de la première activité dans le pipeline.  |Non |false |
| availability | Définit la fenêtre de traitement (par exemple, horaire ou journalier) ou le modèle de découpage pour la production du jeu de données. Chaque unité de données consommée et produite pendant l’exécution d’une activité est appelée tranche de données. Si la disponibilité d’un jeu de données de sortie est journalière (fréquence : Day, intervalle : 1), une tranche est produite chaque jour. <br/><br/>Pour plus d’informations, consultez Disponibilité du jeu de données. <br/><br/>Pour plus d’informations sur le modèle de découpage du jeu de données, consultez l’article [Planification et exécution](data-factory-scheduling-and-execution.md). |Oui |N/D |
| policy |Définit les critères ou la condition que les segments du jeu de données doivent remplir. <br/><br/>Pour plus d’informations, consultez la section [Stratégie du jeu de données](#Policy). |Non |N/D |

## <a name="dataset-example"></a>Exemple de jeu de données
Dans l’exemple suivant, le jeu de données représente une table nommée **MyTable** dans une base de données SQL.

```json
{
    "name": "DatasetSample",
    "properties": {
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties":
        {
            "tableName": "MyTable"
        },
        "availability":
        {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

Notez les points suivants :

* **type** est défini sur AzureSQLTable.
* La propriété de type **tableName** (propre au type AzureSqlTable) est définie sur MyTable.
* **linkedServiceName** fait référence à un service lié de type AzureSqlDatabase, qui est défini dans l’extrait de code JSON suivant.
* **fréquence de disponibilité** est défini sur Jour, et **intervalle** est défini sur 1. Cela signifie que la tranche de jeu de données est générée tous les jours.

**AzureSqlLinkedService** est défini comme suit :

```json
{
    "name": "AzureSqlLinkedService",
    "properties": {
        "type": "AzureSqlDatabase",
        "description": "",
        "typeProperties": {
            "connectionString": "Data Source=tcp:<servername>.database.windows.net,1433;Initial Catalog=<databasename>;User ID=<username>@<servername>;Password=<password>;Integrated Security=False;Encrypt=True;Connect Timeout=30"
        }
    }
}
```

Dans l’extrait de code JSON précédent :

* **type** est défini sur AzureSqlDatabase.
* La propriété de type **connectionString** spécifie les informations nécessaires pour vous connecter à une base de données SQL.

Comme vous pouvez le voir, le service lié définit la façon de se connecter à une base de données SQL. Le jeu de données définit quelle table est utilisée comme entrée et sortie de l’activité dans un pipeline.

> [!IMPORTANT]
> À moins qu’un jeu de données ne soit généré par le pipeline, il doit être marqué comme **external** (externe). Ce paramètre s’applique généralement aux entrées de la première activité d’un pipeline.

## <a name="dataset-type"></a><a name="Type"></a> Type du jeu de données
Le type du jeu de données dépend du magasin de données que vous utilisez. Consultez le tableau suivant pour obtenir la liste des magasins de données pris en charge par Data Factory. Cliquez sur un magasin de données pour savoir comment créer un service lié et un jeu de données pour ce magasin de données.

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

> [!NOTE]
> Les magasins de données avec * peuvent être locaux ou sur l’infrastructure Azure en tant que service (IaaS). Ces magasins de données vous obligent à installer la [passerelle de gestion des données](data-factory-data-management-gateway.md).

Dans l’exemple de la section précédente, le type du jeu de données est défini sur **AzureSqlTable**. De même, pour un jeu de données d’objets blob Azure, le type du jeu de données est défini sur **AzureBlob**, comme indiqué dans le JSON suivant :

```json
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "fileName": "input.log",
            "folderPath": "adfgetstarted/inputdata",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```

## <a name="dataset-structure"></a><a name="Structure"></a>Structure d’un jeu de données
La section **structure** est facultative. Elle définit le schéma du jeu de données en contenant une collection de noms et de types de données de colonnes. Vous utilisez la section structure pour fournir des informations sur le type utilisé pour convertir les types et colonnes de mappage depuis la source vers la destination. Dans l’exemple suivant, le jeu de données comporte trois colonnes : `slicetimestamp`, `projectname`, et `pageviews`. Elles sont respectivement de type String, String et Decimal.

```json
structure:
[
    { "name": "slicetimestamp", "type": "String"},
    { "name": "projectname", "type": "String"},
    { "name": "pageviews", "type": "Decimal"}
]
```

Chaque colonne de la structure contient les propriétés suivantes :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| name |Nom de la colonne. |Oui |
| type |Type de données de la colonne.  |Non |
| culture |Culture .NET à utiliser lorsque le type est un type .NET : `Datetime` ou `Datetimeoffset`. Par défaut, il s’agit de `en-us`. |Non |
| format |Chaîne de format à utiliser lorsque le type est un type .NET : `Datetime` ou `Datetimeoffset`. |Non |

Les recommandations suivantes vous permettent de déterminer le moment auquel inclure les informations de structure et pour connaître les éléments à inclure dans la section **structure**.

* **Pour les sources de données structurées**, spécifiez la section structure uniquement si vous souhaitez mapper les colonnes sources aux colonnes du récepteur et que leurs noms ne sont pas identiques. Ce type de source de données structurées stocke les informations de schéma et de type de données, ainsi que les données elles-mêmes. SQL Server, Oracle et la table Azure sont des exemples de sources de données structurées.
  
    Comme les informations de type sont déjà disponibles pour les sources de données structurées, vous ne devez pas les inclure lorsque vous incluez la section structure.
* **Pour un schéma des sources de données de lecture (en particulier le stockage Blob)** , vous pouvez choisir de stocker des données sans les informations de type ou de schéma. Pour ces types de sources de données, incluez la structure lorsque vous souhaitez mapper les colonnes sources aux colonnes de récepteur. Incluez également la structure lorsque le jeu de données est une entrée d’une activité de copie et que les types de données du jeu de données source doivent être convertis en types natifs pour le récepteur.
    
    Data Factory prend en charge les valeurs suivantes pour fournir des informations de type dans la structure : **Int16, Int32, Int64, Single, Double, Decimal, Byte[], Boolean, String, Guid, Datetime, Datetimeoffset et Timespan**. Ces valeurs sont des valeurs de type basées sur .NET compatibles avec la norme CLS (Common Language Specification).

Data Factory effectue automatiquement les conversions de type lorsque vous déplacez des données d’un magasin de données source vers un magasin de données récepteur.

## <a name="dataset-availability"></a>Disponibilité du jeu de données
La section **availability** dans un jeu de données définit la fenêtre de traitement (par exemple, horaire, journalier ou hebdomadaire) du jeu de données. Pour plus d’informations sur les fenêtres d’activité, voir [Planification et exécution](data-factory-scheduling-and-execution.md).

La section availability suivante spécifie que le jeu de données de sortie est exécuté toutes les heures ou que le jeu de données d’entrée est disponible toutes les heures :

```json
"availability":
{
    "frequency": "Hour",
    "interval": 1
}
```

Si le pipeline présente les heures de début et de fin suivantes :

```json
    "start": "2016-08-25T00:00:00Z",
    "end": "2016-08-25T05:00:00Z",
```

Le jeu de données de sortie est généré toutes les heures dans la plage des heures de début et de fin du pipeline. Par conséquent, cinq tranches de jeu de données sont générées par ce pipeline, une pour chaque fenêtre d’activité (0 h - 1 h, 1 h - 2 h, 2 h - 3 h, 3 h - 4 h, 4 h - 5 h).

Le tableau suivant décrit les propriétés que vous pouvez utiliser dans la section availability :

| Propriété | Description | Obligatoire | Default |
| --- | --- | --- | --- |
| frequency |Spécifie l’unité de temps pour la production du segment du jeu de données.<br/><br/><b>Fréquence prise en charge</b> : Minute, Hour, Day, Week, Month |Oui |N/D |
| interval |Spécifie un multiplicateur de fréquence.<br/><br/>« Frequency x interval » déterminent la fréquence à laquelle la tranche est produite. Par exemple, si vous voulez des tranches de jeu de données d’une heure, définissez <b>frequency</b> sur <b>Hour</b> et <b>interval</b> sur <b>1</b>.<br/><br/>Notez que si vous spécifiez **frequency** sur **Minute**, vous devez définir l’intervalle sur une valeur au minimum égale à 15. |Oui |N/D |
| style |Spécifie si le segment doit être généré au début ou à la fin de l’intervalle.<ul><li>StartOfInterval</li><li>EndOfInterval</li></ul>Si **frequency** est défini sur **Month** et **style** défini sur **EndOfInterval**, le segment est généré le dernier jour du mois. Si **style** est défini sur **StartOfInterval**, le segment est généré le premier jour du mois.<br/><br/>Si **frequency** est défini sur **Day** et **style** est défini sur **EndOfInterval**, le segment est généré la dernière heure du jour.<br/><br/>Si **frequency** est défini sur **Hour** et **style** défini sur **EndOfInterval**, le segment est généré à la fin de l’heure. Par exemple, pour un segment de la période 13 h-14 h, le segment est généré à 14 h. |Non |EndOfInterval |
| anchorDateTime |Définit la position absolue dans le temps utilisée par le planificateur pour calculer les limites de tranche de jeu de données. <br/><br/>Notez que si cette propriété comporte des éléments de date plus précis que la fréquence spécifiée, les éléments plus précis sont ignorés. Par exemple, si **interval** est **hourly** (frequency: hour et interval: 1), et que **anchorDateTime** contient **minutes and seconds**, les éléments minutes et seconds d’**anchorDateTime** sont ignorés. |Non |01/01/0001 |
| offset |Intervalle de temps marquant le déplacement du début et de la fin de toutes les tranches du jeu de données. <br/><br/>Notez que si **anchorDateTime** et **offset** sont spécifiés, un décalage combiné est obtenu. |Non |N/D |

### <a name="offset-example"></a>exemple offset
Par défaut, les tranches quotidiennes (`"frequency": "Day", "interval": 1`) commencent à 12:00 (minuit), temps universel coordonné (UTC). Si vous souhaitez que l’heure de début soit 6 h UTC, définissez le décalage comme indiqué dans l’extrait suivant :

```json
"availability":
{
    "frequency": "Day",
    "interval": 1,
    "offset": "06:00:00"
}
```
### <a name="anchordatetime-example"></a>Exemple anchorDateTime
Dans l’exemple suivant, le jeu de données est généré toutes les 23 heures. La première tranche commence à l’heure spécifiée par **anchorDateTime**, qui est définie sur `2017-04-19T08:00:00` (UTC).

```json
"availability":
{
    "frequency": "Hour",
    "interval": 23,
    "anchorDateTime":"2017-04-19T08:00:00"
}
```

### <a name="offsetstyle-example"></a>Exemple de décalage/style
Le jeu de données suivant est mensuel et est généré le 3 de chaque mois à 8h00 (`3.08:00:00`) :

```json
"availability": {
    "frequency": "Month",
    "interval": 1,
    "offset": "3.08:00:00", 
    "style": "StartOfInterval"
}
```

## <a name="dataset-policy"></a><a name="Policy"></a>Stratégie du jeu de données
La section **policy** dans la définition du jeu de données définit les critères ou la condition que les tranches du jeu de données doivent remplir.

### <a name="validation-policies"></a>Stratégies de validation
| Nom de stratégie | Description | Appliqué(e) à | Obligatoire | Default |
| --- | --- | --- | --- | --- |
| minimumSizeMB |Valide le fait que les données dans un **stockage blob Azure** répondent aux exigences de taille minimale (en mégaoctets). |Stockage Blob Azure |Non |N/D |
| minimumRows |Valide le fait que les données dans une **base de données Azure SQL** ou une **table Azure** contiennent le nombre minimal de lignes. |<ul><li>Azure SQL Database</li><li>Table Azure</li></ul> |Non |N/D |

#### <a name="examples"></a>Exemples
**minimumSizeMB :**

```json
"policy":

{
    "validation":
    {
        "minimumSizeMB": 10.0
    }
}
```

**minimumRows :**

```json
"policy":
{
    "validation":
    {
        "minimumRows": 100
    }
}
```

### <a name="external-datasets"></a>Jeux de données externes
Les jeux de données externes sont ceux qui ne sont pas générés par un pipeline en cours d’exécution dans la fabrique de données. Si le jeu de données est marqué comme étant **external**, la stratégie **ExternalData** peut être définie de manière à influencer le comportement de disponibilité du segment du jeu de données.

À moins qu’un jeu de données ne soit généré par la fabrique de données, il doit être marqué comme **external**(externe). Ce paramètre s’applique généralement aux entrées de la première activité d’un pipeline, à moins que le chaînage des activités ou pipelines ne soit utilisé.

| Nom | Description | Obligatoire | Valeur par défaut |
| --- | --- | --- | --- |
| dataDelay |Durée du délai de la vérification de la disponibilité des données externes pour la tranche donnée. Par exemple, vous pouvez retarder une vérification de toutes les heures à l’aide de ce paramètre.<br/><br/>Le paramètre s’applique uniquement à l’heure actuelle. Par exemple, s’il est 13 h et si cette valeur est de 10 minutes, la validation commence à 13 h 10.<br/><br/>Notez que ce paramètre n’affecte pas les tranches du passé. Les tranches avec **Slice End Time** + **dataDelay** < **Now** sont traitées sans aucun délai.<br/><br/>Les heures supérieures à 23:59 doivent être spécifiées à l’aide du format `day.hours:minutes:seconds`. Par exemple, pour spécifier 24 heures, n’utilisez pas 24:00:00. Utilisez plutôt 1.00:00:00. Si vous utilisez 24:00:00, cette valeur est traitée comme 24 jours (24.00:00:00). Pour 1 jour et 4 heures, spécifiez 1:04:00:00. |Non |0 |
| retryInterval |Délai d'attente entre un échec et la tentative suivante. Ce paramètre s’applique à l’heure actuelle. Si la tentative précédente a échoué, la tentative suivante est effectuée après la période **retryInterval**. <br/><br/>S’il est 13 h actuellement, la première tentative commence. Si la durée de la première vérification de validation est de 1 minute et si l’opération a échoué, la tentative suivante aura lieu à 13h + 1 min (durée) + 1 minute (intervalle avant nouvelle tentative) = 13 h 02. <br/><br/>Pour les segments dans le passé, il n’y a aucun délai. La nouvelle tentative se fait immédiatement. |Non |00:01:00 (1 minute) |
| retryTimeout |Délai d’attente pour chaque nouvelle tentative.<br/><br/>Si la propriété est définie sur 10 minutes, la validation doit être effectuée en 10 minutes maximum. S’il faut plus de 10 minutes pour effectuer la validation, la nouvelle tentative expire.<br/><br/>Si toutes les tentatives de validation expirent, la tranche est marquée comme **TimedOut**. |Non |00:10:00 (10 minutes) |
| maximumRetry |Nombre de fois où la disponibilité des données externes est vérifiée. La valeur maximale autorisée correspond à 10. |Non |3 |


## <a name="create-datasets"></a>Créez les jeux de données
Vous pouvez créer des jeux de données à l’aide de l’un de ces outils ou kits de développement logiciel (SDK) :

- Assistant de copie
- Visual Studio
- PowerShell
- Modèle Azure Resource Manager
- API REST
- API .NET

Consultez les tutoriels suivants pour obtenir des instructions pas à pas sur la création de pipelines et de jeux de données à l’aide de l’un de ces outils ou kits de développement logiciel (SDK) :

- [Créer un pipeline avec une activité de transformation des données](data-factory-build-your-first-pipeline.md)
- [Créer un pipeline avec une activité de déplacement des données](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)

Une fois qu’un pipeline est créé et déployé, vous pouvez gérer et surveiller les pipelines à l’aide des panneaux du portail Azure ou de l’application de surveillance et de gestion. Pour des instructions pas à pas, consultez les rubriques suivantes :

- [Surveiller et gérer les pipelines à l’aide des panneaux du portail Azure](data-factory-monitor-manage-pipelines.md)
- [Surveiller et gérer les pipelines à l’aide de l’application de surveillance et gestion](data-factory-monitor-manage-app.md)

## <a name="scoped-datasets"></a>Étendue des jeux de données
Vous pouvez créer des jeux de données étendus jusqu’à un pipeline à l’aide de la propriété **datasets** . Ces jeux de données peuvent uniquement être utilisés par les activités dans ce pipeline, et non pas par les activités d’autres pipelines. L’exemple suivant définit un pipeline avec deux jeux de données à utiliser dans le pipeline (InputDataset-rdc et OutputDataset-rdc).

> [!IMPORTANT]
> Les étendues des jeux de données sont uniquement prises en charge avec les pipelines à usage unique (où **pipelineMode** défini sur **OneTime**). Pour plus d’informations, consultez [Pipeline onetime](data-factory-create-pipelines.md#onetime-pipeline) .
>
>

```json
{
    "name": "CopyPipeline-rdc",
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
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataset-rdc"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataset-rdc"
                    }
                ],
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1,
                    "style": "StartOfInterval"
                },
                "name": "CopyActivity-0"
            }
        ],
        "start": "2016-02-28T00:00:00Z",
        "end": "2016-02-28T00:00:00Z",
        "isPaused": false,
        "pipelineMode": "OneTime",
        "expirationTime": "15.00:00:00",
        "datasets": [
            {
                "name": "InputDataset-rdc",
                "properties": {
                    "type": "AzureBlob",
                    "linkedServiceName": "InputLinkedService-rdc",
                    "typeProperties": {
                        "fileName": "emp.txt",
                        "folderPath": "adftutorial/input",
                        "format": {
                            "type": "TextFormat",
                            "rowDelimiter": "\n",
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
            },
            {
                "name": "OutputDataset-rdc",
                "properties": {
                    "type": "AzureBlob",
                    "linkedServiceName": "OutputLinkedService-rdc",
                    "typeProperties": {
                        "fileName": "emp.txt",
                        "folderPath": "adftutorial/output",
                        "format": {
                            "type": "TextFormat",
                            "rowDelimiter": "\n",
                            "columnDelimiter": ","
                        }
                    },
                    "availability": {
                        "frequency": "Day",
                        "interval": 1
                    },
                    "external": false,
                    "policy": {}
                }
            }
        ]
    }
}
```

## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur les pipelines, voir [Créer des pipelines](data-factory-create-pipelines.md).
- Pour plus d’informations sur la façon dont les pipelines sont planifiés et exécutés, voir [Planification et exécution dans Azure Data Factory](data-factory-scheduling-and-execution.md).
