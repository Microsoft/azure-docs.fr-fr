---
title: Créer et planifier des pipelines et des activités à la chaîne dans Data Factory
description: Apprenez à créer un pipeline de données dans Azure Data Factory pour déplacer et transformer des données. Créez un flux de travail piloté par les données pour produire des informations prêtes à l’emploi.
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
ms.openlocfilehash: 83b4d14d46677c731b7fb9faae2217492368d4b2
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96496056"
---
# <a name="pipelines-and-activities-in-azure-data-factory"></a>Pipelines et activités dans Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-create-pipelines.md)
> * [Version 2 (version actuelle)](../concepts-pipelines-activities.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Pipelines dans V2](../concepts-pipelines-activities.md).

Cet article vous aide à comprendre les pipelines et les activités dans Azure Data Factory, et à les utiliser dans l’optique de créer des workflows pilotés par les données de bout en bout pour vos scénarios de déplacement des données et de traitement des données.

> [!NOTE]
> Cet article suppose que vous avez parcouru les articles [Présentation du service Azure Data Factory](data-factory-introduction.md). Si vous n’avez pas d’expérience pratique de création de fabriques de données, le [didacticiel relatif à la transformation de données](data-factory-build-your-first-pipeline.md) et/ou [le didacticiel relatif au déplacement de données](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) vous aidera à mieux comprendre cet article.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="overview"></a>Vue d’ensemble
Une fabrique de données peut avoir un ou plusieurs pipelines. Un pipeline constitue un regroupement logique d’activités qui exécutent ensemble une tâche. Les activités d’un pipeline définissent les actions à effectuer sur les données. Par exemple, vous pouvez utiliser une activité de copie pour copier des données d’une base de données SQL Server dans un stockage Blob Azure. Utilisez ensuite une activité Hive qui exécute un script Hive sur un cluster Azure HDInsight pour traiter/transformer les données du stockage Blob afin de produire des données de sortie. Enfin, utilisez une deuxième activité de copie pour copier les données de sortie dans Azure Synapse Analytics sur lequel des solutions de génération de rapports d’aide à la décision sont développées.

Une activité peut inclure zéro ou plusieurs [jeux de données](data-factory-create-datasets.md) d’entrée et produire un ou plusieurs [jeux de données](data-factory-create-datasets.md) de sortie. Le diagramme suivant montre la relation entre le pipeline, l’activité et le jeu de données dans Data Factory :

![Relation entre le pipeline, l’activité et le jeu de données](media/data-factory-create-pipelines/relationship-pipeline-activity-dataset.png)

Un pipeline vous permet de gérer les activités en tant que groupe et non pas individuellement. Par exemple, vous pouvez déployer, planifier, interrompre et reprendre un pipeline, plutôt que de gérer indépendamment des activités dans le pipeline.

Data Factory prend en charge deux types d’activités : les activités de déplacement des données et les activités de transformation des données. Chaque activité peut avoir zéro ou plusieurs [jeux de données](data-factory-create-datasets.md) d’entrée et produire un ou plusieurs jeux de données de sortie.

Un jeu de données d’entrée représente l’entrée d’une activité dans le pipeline, tandis qu’un jeu de données de sortie représente la sortie de l’activité. Les jeux de données identifient les données dans différents magasins de données, par exemple des tables, des fichiers, des dossiers et des documents. Après avoir créé un jeu de données, vous pouvez l’utiliser avec des activités d’un pipeline. Un jeu de données peut, par exemple, constituer un jeu de données d’entrée/sortie d’une activité de copie ou d’une activité HDInsightHive. Pour plus d’informations sur les jeux de données, consultez l’article [Jeux de données dans Azure Data Factory](data-factory-create-datasets.md).

### <a name="data-movement-activities"></a>Activités de déplacement des données
L’activité de copie dans Data Factory permet de copier les données d’un magasin de données source vers un magasin de données récepteur. Data Factory prend en charge les magasins de données suivants. Les données de n’importe quelle source peuvent être écrites dans n’importe quel récepteur. Cliquez sur une banque de données pour découvrir comment copier des données depuis/vers cette banque.

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

> [!NOTE]
> Les banques de données signalées par un astérisque (*) peuvent être locales ou résider sur une instance Azure IaaS. Elles nécessitent que vous installiez une [passerelle de gestion des données](data-factory-data-management-gateway.md) sur un ordinateur local ou Azure IaaS.

Pour plus d’informations, consultez l’article [Activités de déplacement des données](data-factory-data-movement-activities.md).

### <a name="data-transformation-activities"></a>Activités de transformation des données
[!INCLUDE [data-factory-transformation-activities](../../../includes/data-factory-transformation-activities.md)]

Pour plus d’informations, consultez l’article [Activités de déplacement des données](data-factory-data-transformation-activities.md).

### <a name="custom-net-activities"></a>Activités .NET personnalisées
Si vous devez déplacer des données vers ou à partir d’une banque de données qui n’est pas prise en charge par l’activité de copie, ou transformer des données à l’aide de votre propre logique, créez une **activité .NET personnalisée**. Pour plus d’informations sur la création et l’utilisation d’une activité personnalisée, consultez [Utilisation des activités personnalisées dans un pipeline Azure Data Factory](data-factory-use-custom-activities.md).

## <a name="schedule-pipelines"></a>Planifier des pipelines
Un pipeline est actif uniquement entre son heure de **début** et son heure de **fin**. Il n'est pas exécuté avant l'heure de début, ni après l'heure de fin. Lorsque le pipeline est suspendu, il n’est pas exécuté, quelle que soit son heure de début et de fin. Pour qu'un pipeline soit exécuté, il ne doit pas être suspendu. Consultez [Planification et exécution](data-factory-scheduling-and-execution.md) pour comprendre le fonctionnement de planification et de l’exécution dans Azure Data Factory.

## <a name="pipeline-json"></a>Pipeline JSON
Examinons de plus près la définition d’un pipeline au format JSON. La structure générique d'un pipeline se présente comme suit :

```json
{
    "name": "PipelineName",
    "properties":
    {
        "description" : "pipeline description",
        "activities":
        [

        ],
        "start": "<start date-time>",
        "end": "<end date-time>",
        "isPaused": true/false,
        "pipelineMode": "scheduled/onetime",
        "expirationTime": "15.00:00:00",
        "datasets":
        [
        ]
    }
}
```

| Tag | Description | Obligatoire |
| --- | --- | --- |
| name |Nom du pipeline. Spécifiez un nom qui représente l’action effectuée par le pipeline. <br/><ul><li>Nombre maximal de caractères : 260</li><li>Doit commencer par une lettre, un chiffre ou un trait de soulignement (\_)</li><li>Les caractères suivants ne sont pas autorisés : « . », « + », « ? », « / », « < », « > », « \* », « % », « & », « : », « \\ »</li></ul> |Oui |
| description | Spécifiez le texte décrivant la raison motivant l’utilisation du pipeline. |Oui |
| activities | La section **Activités** peut contenir une ou plusieurs activités définies. Consultez la section suivante pour plus d’informations sur l’élément JSON des activités. | Oui |
| start | Date et heure de début du pipeline. Doit se trouver au [format ISO](https://en.wikipedia.org/wiki/ISO_8601). Par exemple : `2016-10-14T16:32:41Z`. <br/><br/>Il est possible de spécifier une heure locale, par exemple une heure EST. Voici un exemple : `2016-02-27T06:00:00-05:00`, qui correspond à 6h EST.<br/><br/>Ensemble, les propriétés de début et de fin spécifient la période active du pipeline. Les tranches de sortie sont uniquement générées pendant cette période active. |Non<br/><br/>Si vous spécifiez une valeur pour la propriété end, vous devez en spécifier une pour la propriété start.<br/><br/>Les heures de début et de fin peuvent être toutes les deux non renseignées pour créer un pipeline. Vous devez spécifier les deux valeurs pour définir une période active d’exécution du pipeline. Si vous ne spécifiez pas d’heures de début et de fin lors de la création d’un pipeline, vous pouvez les définir à l’aide de la cmdlet Set-AzDataFactoryPipelineActivePeriod ultérieurement. |
| end | Date et heure de fin du pipeline. Si spécifiée, doit être au format ISO. Par exemple : `2016-10-14T17:32:41Z` <br/><br/>Il est possible de spécifier une heure locale, par exemple une heure EST. Voici un exemple : `2016-02-27T06:00:00-05:00`, qui correspond à 6h EST.<br/><br/>Pour exécuter le pipeline indéfiniment, spécifiez 9999-09-09 comme valeur pour la propriété end. <br/><br/> Un pipeline est actif uniquement entre son heure de début et son heure de fin. Il n'est pas exécuté avant l'heure de début, ni après l'heure de fin. Lorsque le pipeline est suspendu, il n’est pas exécuté, quelle que soit son heure de début et de fin. Pour qu'un pipeline soit exécuté, il ne doit pas être suspendu. Consultez [Planification et exécution](data-factory-scheduling-and-execution.md) pour comprendre le fonctionnement de planification et de l’exécution dans Azure Data Factory. |Non <br/><br/>Si vous spécifiez une valeur pour la propriété start, vous devez en spécifier une pour la propriété end.<br/><br/>Consultez les remarques relatives à la propriété **start** . |
| isPaused | Si la valeur est true, le pipeline ne s’exécute pas. Il est à l’état de pause. Valeur par défaut = false. Vous pouvez utiliser cette propriété pour activer ou désactiver un pipeline. |Non |
| pipelineMode | La méthode de planification des exécutions pour le pipeline. Les valeurs autorisées sont : scheduled (par défaut) et onetime.<br/><br/>« scheduled » indique que le pipeline s’exécute selon un intervalle de temps spécifié en fonction de sa période active (heure de début et de fin). « Onetime » indique que le pipeline ne s’exécute qu’une seule fois. Une fois créés, les pipelines de type onetime ne peuvent pas être modifiés ni mis à jour pour le moment. Consultez la page [Pipeline onetime](#onetime-pipeline) pour plus d’informations sur le paramètre onetime. |Non |
| expirationTime | Durée pendant laquelle le [pipeline à usage unique](#onetime-pipeline) est valide et doit rester configuré. Le pipeline est automatiquement supprimé une fois le délai d’expiration atteint s’il ne contient aucune exécution active, en échec ou en attente. La valeur par défaut est : `"expirationTime": "3.00:00:00"`|Non |
| jeux de données |Liste des jeux de données à utiliser par les activités définies dans le pipeline. Cette propriété permet de définir des jeux de données spécifiques à ce pipeline et non définis dans la fabrique de données. Les jeux de données définis dans ce pipeline ne peuvent être utilisés que par ce pipeline et ne peuvent pas être partagés. Pour plus d’informations, consultez la page [Étendue des jeux de données](data-factory-create-datasets.md#scoped-datasets) . |Non |

## <a name="activity-json"></a>Activité JSON
La section **Activités** peut contenir une ou plusieurs activités définies. Chaque activité possède la structure de niveau supérieur suivante :

```json
{
    "name": "ActivityName",
    "description": "description",
    "type": "<ActivityType>",
    "inputs": "[]",
    "outputs": "[]",
    "linkedServiceName": "MyLinkedService",
    "typeProperties":
    {

    },
    "policy":
    {
    },
    "scheduler":
    {
    }
}
```

Le tableau suivant décrit des propriétés de la définition JSON de l’activité :

| Tag | Description | Obligatoire |
| --- | --- | --- |
| name | Nom de l’activité. Spécifiez un nom qui représente l’action effectuée par l’activité. <br/><ul><li>Nombre maximal de caractères : 260</li><li>Doit commencer par une lettre, un chiffre ou un trait de soulignement (\_)</li><li>Les caractères suivants ne sont pas autorisés : « . », « + », « ? », « / », « < », « > », « * », « % », « & », « : », « \\ »</li></ul> |Oui |
| description | Texte décrivant la raison motivant l’activité ou son utilisation |Oui |
| type | Type de l’activité. Consultez les sections [Activités de déplacement des données](#data-movement-activities) et [Activités de transformation des données](#data-transformation-activities) pour en savoir plus sur les différents types d’activités. |Oui |
| inputs |Les tables d’entrée utilisées par l’activité<br/><br/>`// one input table`<br/>`"inputs":  [ { "name": "inputtable1"  } ],`<br/><br/>`// two input tables` <br/>`"inputs":  [ { "name": "inputtable1"  }, { "name": "inputtable2"  } ],` |Oui |
| outputs |Les tables de sortie utilisées par l’activité.<br/><br/>`// one output table`<br/>`"outputs":  [ { "name": "outputtable1" } ],`<br/><br/>`//two output tables`<br/>`"outputs":  [ { "name": "outputtable1" }, { "name": "outputtable2" }  ],` |Oui |
| linkedServiceName |Nom du service lié utilisé par l’activité. <br/><br/>Une activité peut nécessiter que vous spécifiiez le service lié à l’environnement de calcul requis. |Oui pour l’activité HDInsight et l’activité de scoring par lots Azure Machine Learning <br/><br/>Non pour toutes les autres |
| typeProperties |Les propriétés de la section **typeProperties** dépendent du type de l’activité. Pour afficher les propriétés de type d’une activité, cliquez sur les liens vers l’activité dans la section précédente. | Non |
| policy |Stratégies affectant le comportement d’exécution de l’activité. Si aucune valeur n’est spécifiée, les stratégies par défaut sont utilisées. |Non |
| scheduler | La propriété « scheduler » est utilisée pour définir la planification souhaitée pour l’activité. Ses sous-propriétés sont les mêmes que celles de la [propriété de disponibilité dans un jeu de données](data-factory-create-datasets.md#dataset-availability). |Non |

### <a name="policies"></a>Stratégies
Les stratégies affectent le comportement d'exécution d'une activité, en particulier lors du traitement du segment d'une table. Le tableau suivant fournit les détails.

| Propriété | Valeurs autorisées | Valeur par défaut | Description |
| --- | --- | --- | --- |
| accès concurrentiel |Integer <br/><br/>Valeur maximale : 10 |1 |Nombre d’exécutions simultanées de l’activité.<br/><br/>Il détermine le nombre d’exécutions en parallèle de l’activité qui peuvent se produire sur différents segments. Par exemple, si une activité doit passer par un grand ensemble de données disponibles, une valeur de concurrence plus élevée accélère le traitement des données. |
| executionPriorityOrder |NewestFirst<br/><br/>OldestFirst |OldestFirst |Détermine l’ordre des segments de données qui sont traités.<br/><br/>Par exemple, si vous avez 2 segments (l’un se produisant à 16 heures et l’autre à 17 heures) et que les deux sont en attente d’exécution. Si vous définissez executionPriorityOrder sur NewestFirst, le segment à 17 h est traité en premier. De même, si vous définissez executionPriorityOrder sur OldestFIrst, le segment à 16 h est traité en premier. |
| retry |Integer<br/><br/>La valeur max peut être 10 |0 |Nombre de nouvelles tentatives avant que le traitement des données du segment ne soit marqué comme un échec. L'exécution de l'activité pour un segment de données est répétée jusqu'au nombre de tentatives spécifié. La nouvelle tentative est effectuée dès que possible après l'échec. |
| délai d'expiration |TimeSpan |00:00:00 |Délai d'expiration de l'activité. Exemple : 00:10:00 (implique un délai d’expiration de 10 minutes)<br/><br/>Si une valeur n’est pas spécifiée ou est égale à 0, le délai d’expiration est infini.<br/><br/>Si le temps de traitement des données sur un segment dépasse la valeur du délai d’expiration, il est annulé et le système tente de réexécuter le traitement. Le nombre de nouvelles tentatives dépend de la propriété « retry ». Quand le délai d’expiration est atteint, l’état est défini sur TimedOut. |
| delay |TimeSpan |00:00:00 |Spécifie le délai avant le début du traitement des données du segment.<br/><br/>L’exécution d’activité pour une tranche de données est démarrée une fois que le délai a dépassé l’heure d’exécution prévue.<br/><br/>Exemple : 00:10:00 (implique un délai de 10 minutes) |
| longRetry |Integer<br/><br/>Valeur maximale : 10 |1 |Le nombre de nouvelles tentatives longues avant l’échec de l’exécution du segment.<br/><br/>Les tentatives longRetry sont espacées par longRetryInterval. Par conséquent, si vous devez spécifier un délai entre chaque tentative, utilisez longRetry. Si les valeurs Retry et longRetry sont spécifiées, chaque tentative longRetry inclut des tentatives Retry et le nombre maximal de tentatives sera égal à Retry * longRetry.<br/><br/>Par exemple, si nous avons les paramètres suivants dans la stratégie de l’activité :<br/>Retry : 3<br/>longRetry : 2<br/>longRetryInterval : 01:00:00<br/><br/>Supposons qu’il existe un seul segment à exécuter (dont l’état est Waiting) et que l’exécution de l’activité échoue à chaque fois. Au départ, il y aurait 3 tentatives consécutives d'exécution. Après chaque tentative, l’état du segment serait Retry. Une fois les 3 premières tentatives terminées, l’état du segment serait LongRetry.<br/><br/>Après une heure (c’est-à-dire la valeur de longRetryInterval), il y aurait un autre ensemble de 3 tentatives consécutives d’exécution. Ensuite, l'état du segment serait Failed et aucune autre tentative ne serait exécutée. Par conséquent, 6 tentatives ont été exécutées.<br/><br/>Si une exécution réussit, l’état de la tranche est Ready et aucune nouvelle tentative n’est tentée.<br/><br/>La valeur longRetry peut être utilisée dans les situations où les données dépendantes arrivent à des moments non déterministes ou lorsque l’environnement global où le traitement des données se produit est douteux. Dans ces cas, l’exécution de nouvelles tentatives l’une après l’autre peut ne pas être utile et procéder ainsi après un intervalle de temps précis produit la sortie désirée.<br/><br/>Mise en garde : ne définissez pas de valeurs élevées pour longRetry ou longRetryInterval. En règle générale, des valeurs plus élevées impliquent d’autres problèmes systémiques. |
| longRetryInterval |TimeSpan |00:00:00 |Le délai entre les nouvelles tentatives longues |

## <a name="sample-copy-pipeline"></a>Exemple de pipeline de copie
Dans l’exemple de pipeline suivant, il existe une activité de type **Copy** in the **d’activités** . Dans cet exemple, l’[activité de copie](data-factory-data-movement-activities.md) duplique des données d’un stockage Blob Azure vers une base de données Azure SQL.

```json
{
  "name": "CopyPipeline",
  "properties": {
    "description": "Copy data from a blob to Azure SQL table",
    "activities": [
      {
        "name": "CopyFromBlobToSQL",
        "type": "Copy",
        "inputs": [
          {
            "name": "InputDataset"
          }
        ],
        "outputs": [
          {
            "name": "OutputDataset"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink",
            "writeBatchSize": 10000,
            "writeBatchTimeout": "60:00:00"
          }
        },
        "Policy": {
          "concurrency": 1,
          "executionPriorityOrder": "NewestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ],
    "start": "2016-07-12T00:00:00Z",
    "end": "2016-07-13T00:00:00Z"
  }
}
```

Notez les points suivants :

* Dans la section des activités, il existe une seule activité dont le **type** a la valeur **Copy**.
* L’entrée de l’activité est définie sur **InputDataset** et sa sortie, sur **OutputDataset**. Consultez l’article [Jeux de données](data-factory-create-datasets.md) pour en savoir plus sur la définition de jeux de données dans JSON.
* Dans la section **typeProperties**, **BlobSource** est spécifié en tant que type de source et **SqlSink**, en tant que type de récepteur. Dans la section [Activités de déplacement des données](#data-movement-activities), cliquez sur le magasin de données que vous souhaitez utiliser comme source ou récepteur pour en savoir plus sur le déplacement des données vers/depuis ce magasin de données.

Pour obtenir une description complète de la création de ce pipeline, consultez le [Tutoriel : Copier des données de Stockage Blob vers SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)

## <a name="sample-transformation-pipeline"></a>Exemple de pipeline de transformation
Dans l’exemple de pipeline suivant, il existe une activité de type **HDInsightHive** in the **d’activités** . Dans cet exemple, [l’activité Hive HDInsight](data-factory-hive-activity.md) transforme les données d’un stockage blob Azure en exécutant un fichier de script Hive sur un cluster Hadoop Azure HDInsight.

```json
{
    "name": "TransformPipeline",
    "properties": {
        "description": "My first Azure Data Factory pipeline",
        "activities": [
            {
                "type": "HDInsightHive",
                "typeProperties": {
                    "scriptPath": "adfgetstarted/script/partitionweblogs.hql",
                    "scriptLinkedService": "AzureStorageLinkedService",
                    "defines": {
                        "inputtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/inputdata",
                        "partitionedtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/partitioneddata"
                    }
                },
                "inputs": [
                    {
                        "name": "AzureBlobInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "policy": {
                    "concurrency": 1,
                    "retry": 3
                },
                "scheduler": {
                    "frequency": "Month",
                    "interval": 1
                },
                "name": "RunSampleHiveActivity",
                "linkedServiceName": "HDInsightOnDemandLinkedService"
            }
        ],
        "start": "2016-04-01T00:00:00Z",
        "end": "2016-04-02T00:00:00Z",
        "isPaused": false
    }
}
```

Notez les points suivants :

* Dans la section des activités, il existe une seule activité dont le **type** a la valeur **HDInsightHive**.
* Le fichier de script Hive, **partitionweblogs.hql**, est stocké dans le compte de stockage Azure (spécifié par le service scriptLinkedService, appelé **AzureStorageLinkedService**) et dans le dossier **script** du conteneur **adfgetstarted**.
* La section `defines` est utilisée pour spécifier les paramètres d’exécution transmis au script Hive comme valeurs de configuration Hive (par ex. `${hiveconf:inputtable}`, `${hiveconf:partitionedtable}`).

La section **typeProperties** est différente pour chaque activité de transformation. Pour en savoir plus sur les propriétés de type prises en charge pour une activité de transformation, cliquez sur l’activité de transformation dans la table [Activités de transformation des données](#data-transformation-activities).

Pour obtenir une description complète de la création de ce pipeline, consultez le [Tutoriel : Générer votre premier pipeline pour traiter les données à l’aide du cluster Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="multiple-activities-in-a-pipeline"></a>Plusieurs activités à l’intérieur d’un pipeline
Les deux exemples de pipelines précédents ne contiennent qu’une seule activité. Un pipeline peut toutefois contenir plusieurs activités.

Si vous avez plusieurs activités dans un pipeline et que la sortie d’une activité n’est pas une entrée dans une autre activité, les activités peuvent s’exécuter en parallèle si les segments de données d’entrée pour les activités sont prêts.

Vous pouvez chaîner deux activités en utilisant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. La seconde activité s’exécute uniquement quand la première se termine correctement.

![Chaînage des activités dans le même pipeline](./media/data-factory-create-pipelines/chaining-one-pipeline.png)

Dans cet exemple, le pipeline contient deux activités : Activity1 et Activity2. Activity1 utilise Dataset1 comme entrée et produit une sortie Dataset2. L’activité utilise Dataset2 comme entrée et produit une sortie Dataset3. La sortie d’Activity1 (Dataset2) étant l’entrée d’Activity2, Activity2 n’est exécutée que lorsque l’activité se termine avec succès et produit la tranche Dataset2. Si Activity1 échoue pour une quelconque raison et ne produit pas la tranche Dataset2, Activity2 n’est pas exécutée pour cette tranche (par exemple : entre 9 h et 10 h).

Vous pouvez également chaîner des activités se trouvant dans des pipelines différents.

![Chaînage des activités dans deux pipelines](./media/data-factory-create-pipelines/chaining-two-pipelines.png)

Dans cet exemple, Pipeline1 ne contient qu’une seule activité qui utilise Dataset1 comme entrée et produit Dataset2 comme sortie. Pipeline2 ne contient également qu’une seule activité qui utilise Dataset2 comme entrée et produit Dataset3 comme sortie.

Pour plus d’informations, consultez [Planification et exécution](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).
## <a name="create-and-monitor-pipelines"></a>Créer et surveiller des pipelines
Vous pouvez créer des pipelines à l’aide de l’un de ces outils ou kits de développement logiciel (SDK).

- Assistant de copie
- Visual Studio
- Azure PowerShell
- Modèle Azure Resource Manager
- API REST
- API .NET

Consultez les didacticiels suivants pour obtenir des instructions pas à pas sur la création de pipelines à l’aide de l’un de ces outils ou kits de développement logiciel (SDK).

- [Créer un pipeline avec une activité de transformation des données](data-factory-build-your-first-pipeline.md)
- [Créer un pipeline avec une activité de déplacement des données](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)

Une fois qu’un pipeline est créé/déployé, vous pouvez gérer et surveiller les pipelines à l’aide des panneaux du portail Azure et de l’application Surveiller et gérer. Pour des instructions pas à pas, consultez les rubriques suivantes.

- [Surveillez et gérez les pipelines à l’aide des panneaux du portail Azure](data-factory-monitor-manage-pipelines.md).
- [Surveiller et gérer les pipelines à l’aide de l’application Surveiller et gérer](data-factory-monitor-manage-app.md)

## <a name="onetime-pipeline"></a>Pipeline onetime
Vous pouvez créer et planifier un pipeline pour qu’il s’exécute périodiquement (par exemple  toutes les heures ou tous les jours) en fonction de l’heure de début et de l’heure de fin que vous spécifiez dans la définition du pipeline. Pour plus d'informations, consultez Planification des activités. Vous pouvez également créer un pipeline qui ne s’exécute qu’une seule fois. Pour ce faire, vous définissez la propriété **pipelineMode** dans la définition du pipeline sur la valeur **onetime**, comme indiqué dans l’exemple JSON suivant. La valeur par défaut de cette propriété est **scheduled**(planifié).

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
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataset"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataset"
                    }
                ],
                "name": "CopyActivity-0"
            }
        ],
        "pipelineMode": "OneTime"
    }
}
```

Notez les points suivants :

* Les heures de **début** et de **fin** ne sont pas spécifiées.
* La **disponibilité** des jeux de données d’entrée et de sortie est spécifiée (**fréquence** et **intervalle**) même si les valeurs ne sont pas utilisées par Data Factory.
* La vue schématique n’affiche pas les pipelines à usage unique (onetime). Ce comportement est normal.
* Les pipelines à usage unique ne peuvent pas être mis à jour. Vous pouvez cloner un pipeline à usage unique, le renommer, mettre à jour ses propriétés et le déployer pour en créer un autre.

## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur les jeux de données, consultez l’article [Créer des jeux de données](data-factory-create-datasets.md).
- Pour plus d’informations sur la façon dont les pipelines sont planifiés et exécutés, consultez l’article [Planification et exécution dans Azure Data Factory](data-factory-scheduling-and-execution.md).
