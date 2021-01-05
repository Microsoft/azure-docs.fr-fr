---
title: Azure Data Factory - Référence de script JSON
description: Fournit des schémas JSON pour les entités Data Factory.
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
ms.openlocfilehash: 41ba08aef7aed761c3c6063f97768f22bffe3a36
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97508490"
---
# <a name="azure-data-factory---json-scripting-reference"></a>Azure Data Factory - Référence de script JSON
> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory.


Cet article fournit des schémas JSON et des exemples pour la définition des entités Azure Data Factory (pipeline, activité, jeu de données et service lié).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="pipeline"></a>Pipeline
La structure de haut niveau d’une définition de pipeline est la suivante :

```json
{
  "name": "SamplePipeline",
  "properties": {
    "description": "Describe what pipeline does",
    "activities": [
    ],
    "start": "2016-07-12T00:00:00",
    "end": "2016-07-13T00:00:00"
  }
}
```

Le tableau suivant décrit les propriétés dans la définition JSON du pipeline :

| Propriété | Description | Obligatoire
-------- | ----------- | --------
| name | Nom du pipeline. Spécifiez un nom qui représente l’action que l’activité ou le pipeline est configuré(e) à exécuter<br/><ul><li>Nombre maximal de caractères : 260</li><li>Doit commencer par une lettre, un chiffre ou un trait de soulignement (\_)</li><li>Les caractères suivants ne sont pas autorisés : « . », « + », « ? », « / », « < », « > », « * », « % », « & », « : », « \\ »</li></ul> |Oui |
| description |Texte décrivant la raison motivant l’activité ou le pipeline. | Non |
| activities | Contient une liste d’activités. | Oui |
| start |Date et heure de début du pipeline. Doit se trouver au [format ISO](https://en.wikipedia.org/wiki/ISO_8601). Par exemple : 2014-10-14T16:32:41. <br/><br/>Il est possible de spécifier une heure locale, par exemple une heure EST. Voici un exemple : `2016-02-27T06:00:00**-05:00`, qui correspond à 6h EST.<br/><br/>Ensemble, les propriétés de début et de fin spécifient la période active du pipeline. Les tranches de sortie sont uniquement générées pendant cette période active. |Non<br/><br/>Si vous spécifiez une valeur pour la propriété end, vous devez en spécifier une pour la propriété start.<br/><br/>Les heures de début et de fin peuvent être toutes les deux non renseignées pour créer un pipeline. Vous devez spécifier les deux valeurs pour définir une période active d’exécution du pipeline. Si vous ne spécifiez pas d’heures de début et de fin lors de la création d’un pipeline, vous pouvez les définir à l’aide de la cmdlet Set-AzDataFactoryPipelineActivePeriod ultérieurement. |
| end |Date et heure de fin du pipeline. Si spécifiée, doit être au format ISO. Par exemple : 2014-10-14T17:32:41 <br/><br/>Il est possible de spécifier une heure locale, par exemple une heure EST. Voici un exemple : `2016-02-27T06:00:00**-05:00`, qui correspond à 6h EST.<br/><br/>Pour exécuter le pipeline indéfiniment, spécifiez 9999-09-09 comme valeur pour la propriété end. |Non <br/><br/>Si vous spécifiez une valeur pour la propriété start, vous devez en spécifier une pour la propriété end.<br/><br/>Consultez les remarques relatives à la propriété **start** . |
| isPaused |Si la valeur est true, le pipeline ne s’exécute pas. Valeur par défaut = false. Vous pouvez utiliser cette propriété pour activer ou désactiver. |Non |
| pipelineMode |La méthode de planification des exécutions pour le pipeline. Les valeurs autorisées sont : scheduled (par défaut) et onetime.<br/><br/>« scheduled » indique que le pipeline s’exécute selon un intervalle de temps spécifié en fonction de sa période active (heure de début et de fin). « Onetime » indique que le pipeline ne s’exécute qu’une seule fois. Une fois créés, les pipelines de type onetime ne peuvent pas être modifiés ni mis à jour pour le moment. Consultez la page [Pipeline onetime](data-factory-create-pipelines.md#onetime-pipeline) pour plus d’informations sur le paramètre onetime. |Non |
| expirationTime |Durée pendant laquelle le pipeline est valide et doit rester configuré. Le pipeline est automatiquement supprimé une fois le délai d’expiration atteint s’il ne contient aucune exécution active, en échec ou en attente. |Non |


## <a name="activity"></a>Activité
La structure de haut niveau pour une activité dans une définition de pipeline (élément d’activités) est la suivante :

```json
{
    "name": "ActivityName",
    "description": "description",
    "type": "<ActivityType>",
    "inputs":  "[]",
    "outputs":  "[]",
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

Le tableau suivant décrit les propriétés dans la définition JSON du pipeline :

| Tag | Description | Obligatoire |
| --- | --- | --- |
| name |Nom de l’activité. Spécifiez un nom qui représente l’action pour laquelle l’activité est configurée<br/><ul><li>Nombre maximal de caractères : 260</li><li>Doit commencer par une lettre, un chiffre ou un trait de soulignement (\_)</li><li>Les caractères suivants ne sont pas autorisés : « . », « + », « ? », « / », « < », « > », « * », « % », « & », « : », « \\ »</li></ul> |Oui |
| description |Texte décrivant la raison motivant l’activité. |Non |
| type |Spécifie le type de l'activité. Consultez les sections [MAGASINS DE DONNÉS](#data-stores) et [ACTIVITÉS DE TRANSFORMATION DES DONNÉES](#data-transformation-activities) pour en savoir plus sur les différents types d’activités. |Oui |
| inputs |Les tables d’entrée utilisées par l’activité<br/><br/>`// one input table`<br/>`"inputs":  [ { "name": "inputtable1"  } ],`<br/><br/>`// two input tables` <br/>`"inputs":  [ { "name": "inputtable1"  }, { "name": "inputtable2"  } ],` |Non pour les activités HDInsightStreaming et SqlServerStoredProcedure <br/> <br/> Oui pour toutes les autres |
| outputs |Les tables de sortie utilisées par l’activité.<br/><br/>`// one output table`<br/>`"outputs":  [ { "name": “outputtable1” } ],`<br/><br/>`//two output tables`<br/>`"outputs":  [ { "name": “outputtable1” }, { "name": “outputtable2” }  ],` |Oui |
| linkedServiceName |Nom du service lié utilisé par l’activité. <br/><br/>Une activité peut nécessiter que vous spécifiiez le service lié à l’environnement de calcul requis. |Oui pour les activités HDInsight, les activités d’Azure Machine Learning Studio (classique) et les activités de procédure stockée. <br/><br/>Non pour toutes les autres |
| typeProperties |Les propriétés de la section typeProperties dépendent du type de l’activité. |Non |
| policy |Stratégies affectant le comportement d’exécution de l’activité. Si aucune valeur n’est spécifiée, les stratégies par défaut sont utilisées. |Non |
| scheduler |La propriété « scheduler » est utilisée pour définir la planification souhaitée pour l’activité. Ses sous-propriétés sont les mêmes que celles de la [propriété de disponibilité dans un jeu de données](data-factory-create-datasets.md#dataset-availability). |Non |

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

### <a name="typeproperties-section"></a>Section typeProperties
La section typeProperties est différente pour chaque activité. Les activités de transformation n’ont que les propriétés du type. Consultez la section [Activités de transformation des données](#data-transformation-activities) dans cet article pour obtenir des exemples JSON qui définissent les activités de transformation dans un pipeline.

L’**Activité de copie** a deux sous-sections dans la section typeProperties : **source** et **sink**. Consultez la section [Magasins de données](#data-stores) de cet article pour obtenir des exemples JSON montrant comment utiliser un magasin de données comme source et/ou récepteur.

### <a name="sample-copy-pipeline"></a>Exemple de pipeline de copie
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
    "start": "2016-07-12T00:00:00",
    "end": "2016-07-13T00:00:00"
  }
}
```

Notez les points suivants :

* Dans la section des activités, il existe une seule activité dont le **type** a la valeur **Copy**.
* L’entrée de l’activité est définie sur **InputDataset** et sa sortie, sur **OutputDataset**.
* Dans la section **typeProperties**, **BlobSource** est spécifié en tant que type de source et **SqlSink**, en tant que type de récepteur.

Consultez la section [Magasins de données](#data-stores) de cet article pour obtenir des exemples JSON montrant comment utiliser un magasin de données comme source et/ou récepteur.

Pour obtenir une description complète de la création de ce pipeline, consultez le [Tutoriel : Copier des données de Stockage Blob vers SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)

### <a name="sample-transformation-pipeline"></a>Exemple de pipeline de transformation
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
        "start": "2016-04-01T00:00:00",
        "end": "2016-04-02T00:00:00",
        "isPaused": false
    }
}
```

Notez les points suivants :

* Dans la section des activités, il existe une seule activité dont le **type** a la valeur **HDInsightHive**.
* Le fichier de script Hive, **partitionweblogs.hql**, est stocké dans le compte de stockage Azure (spécifié par le service scriptLinkedService, appelé **AzureStorageLinkedService**) et dans le dossier **script** du conteneur **adfgetstarted**.
* La section **defines** est utilisée pour spécifier les paramètres d’exécution transmis au script Hive comme valeurs de configuration Hive (par ex. `${hiveconf:inputtable}`, `${hiveconf:partitionedtable}`).

Consultez la section [Activités de transformation des données](#data-transformation-activities) dans cet article pour obtenir des exemples JSON qui définissent les activités de transformation dans un pipeline.

Pour obtenir une description complète de la création de ce pipeline, consultez le [Tutoriel : Générer votre premier pipeline pour traiter les données à l’aide du cluster Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="linked-service"></a>Service lié
La structure de haut niveau d’une définition de service lié est la suivante :

```json
{
    "name": "<name of the linked service>",
    "properties": {
        "type": "<type of the linked service>",
        "typeProperties": {
        }
    }
}
```

Le tableau suivant décrit les propriétés dans la définition JSON du pipeline :

| Propriété | Description | Obligatoire |
| -------- | ----------- | -------- |
| name | Nom du service lié. | Oui |
| propriétés - type | Type du service lié. Par exemple : Stockage Azure, Azure SQL Database. |
| typeProperties | La section typeProperties comporte des éléments qui sont différents pour chaque magasin de données ou environnement de calcul. Consultez la section Magasins de données pour les services liés aux magasins de données et la section [Environnements de calcul](#compute-environments) pour les services liés au calcul. |

## <a name="dataset"></a>Dataset
Un jeu de données dans Azure Data Factory est défini comme suit :

```json
{
    "name": "<name of dataset>",
    "properties": {
        "type": "<type of dataset: AzureBlob, AzureSql etc...>",
        "external": <boolean flag to indicate external data. only for input datasets>,
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
| name | Nom du jeu de données Pour connaître les règles d’affectation des noms, voir [Azure Data Factory - Règles d’affectation des noms](data-factory-naming-rules.md). |Oui |N/D |
| type | Type du jeu de données. Spécifiez l’un des types pris en charge par Azure Data Factory (par exemple : AzureBlob, AzureSqlTable). Consultez la section [Magasins de données](#data-stores) pour obtenir des informations sur les types de magasins de données et jeu de données pris en charge par Azure Data Factory. |
| structure | Schéma du jeu de données. Il contient des colonnes, leurs types, etc. | Non |N/D |
| typeProperties | Propriétés correspondant au type sélectionné. Consultez la section [Magasins de données](#data-stores) pour en savoir plus sur les types pris en charge et leurs propriétés. |Oui |N/D |
| external | Indicateur booléen pour indiquer si un jeu de données est explicitement généré par un pipeline de fabrique de données ou non. |Non |false |
| availability | Définit la fenêtre de traitement ou le modèle de découpage pour la production du jeu de données. Pour plus d’informations sur le modèle de découpage du jeu de données, consultez l’article [Planification et exécution](data-factory-scheduling-and-execution.md) . |Oui |N/D |
| policy |Définit les critères ou la condition que les segments du jeu de données doivent remplir. <br/><br/>Pour plus d'informations, consultez la section Stratégie de jeu de données. |Non |N/D |

Chaque colonne de la section **structure** contient les propriétés suivantes :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| name |Nom de la colonne. |Oui |
| type |Type de données de la colonne.  |Non |
| culture |Culture .NET à utiliser lorsque le type est spécifié et qu’il est de type .NET `Datetime` ou `Datetimeoffset`. La valeur par défaut est `en-us`. |Non |
| format |Chaîne de format à utiliser lorsque le type est spécifié et qu’il est de type .NET `Datetime` ou `Datetimeoffset`. |Non |

Dans l’exemple suivant, le jeu de données contient les trois colonnes `slicetimestamp`, `projectname` et `pageviews`. Leurs types respectifs sont les suivants : String, String et Decimal.

```json
structure:
[
    { "name": "slicetimestamp", "type": "String"},
    { "name": "projectname", "type": "String"},
    { "name": "pageviews", "type": "Decimal"}
]
```

Le tableau suivant décrit les propriétés que vous pouvez utiliser dans la section **availability** :

| Propriété | Description | Obligatoire | Default |
| --- | --- | --- | --- |
| frequency |Spécifie l’unité de temps pour la production du segment du jeu de données.<br/><br/><b>Fréquence prise en charge</b> : Minute, Hour, Day, Week, Month |Oui |N/D |
| interval |Spécifie un multiplicateur de fréquence<br/><br/>«Frequency» et «interval» déterminent la fréquence à laquelle la tranche est produite.<br/><br/>Si vous voulez des tranches de jeu de données d’une heure, définissez <b>frequency</b> sur <b>Hour</b> et <b>interval</b> sur <b>1</b>.<br/><br/><b>Remarque</b> : Si vous définissez la fréquence en minutes, nous vous recommandons de définir l’intervalle sur une valeur au moins égale à 15. |Oui |N/D |
| style |Spécifie si le segment doit être généré au début / à la fin de l’intervalle.<ul><li>StartOfInterval</li><li>EndOfInterval</li></ul><br/><br/>Si la fréquence est définie sur Month et le style défini sur EndOfInterval, le segment est généré le dernier jour du mois. Si le style est défini sur StartOfInterval, le segment est généré le premier jour du mois.<br/><br/>Si la fréquence est définie sur Day et le style défini sur EndOfInterval, le segment est généré la dernière heure du jour.<br/><br/>Si la fréquence est définie sur Hour et le style défini sur EndOfInterval, le segment est généré à la fin de l’heure. Par exemple, pour un segment de la période 13 h-14 h, le segment est généré à 14 h. |Non |EndOfInterval |
| anchorDateTime |Définit la position absolue dans le temps utilisée par le planificateur pour calculer les limites de tranche de jeu de données. <br/><br/><b>Remarque</b> : Si AnchorDateTime contient des éléments de date plus précis que la fréquence, ces éléments plus précis sont ignorés. <br/><br/>Par exemple, si <b>interval</b> est <b>hourly</b> (frequency: hour et interval: 1), et que <b>AnchorDateTime</b> contient <b>minutes and seconds</b>, les éléments <b>minutes et seconds</b> d’AnchorDateTime sont ignorés. |Non |01/01/0001 |
| offset |Intervalle de temps marquant le déplacement du début et de la fin de toutes les tranches du jeu de données. <br/><br/><b>Remarque</b> : Si anchorDateTime et offset sont spécifiés, on obtient un décalage combiné. |Non |N/D |

La section availability suivante spécifie que le jeu de données de sortie est exécuté toutes les heures (ou) que le jeu de données d’entrée est disponible toutes les heures :

```json
"availability":
{
    "frequency": "Hour",
    "interval": 1
}
```

La section **policy** de la définition du jeu de données définit les critères ou la condition que les segments du jeu de données doivent remplir.

| Nom de la stratégie | Description | Appliqué(e) à | Obligatoire | Default |
| --- | --- | --- | --- | --- |
| minimumSizeMB |Valide le fait que les données dans un **objet blob Azure** répondent aux exigences de taille minimale (en mégaoctets). |Objets blob Azure |Non |N/D |
| minimumRows |Valide le fait que les données dans **Azure SQL Database** ou une **table Azure** contiennent le nombre minimal de lignes. |<ul><li>Azure SQL Database</li><li>table Azure</li></ul> |Non |N/D |

**Exemple :**

```json
"policy":

{
    "validation":
    {
        "minimumSizeMB": 10.0
    }
}
```

À moins qu’un jeu de données ne soit généré par Azure Data Factory, il doit être marqué comme **external**(externe). Ce paramètre s’applique généralement aux entrées de la première activité d’un pipeline, à moins que le chaînage des activités ou pipelines ne soit utilisé.

| Nom | Description | Obligatoire | Valeur par défaut |
| --- | --- | --- | --- |
| dataDelay |Durée du délai de la vérification de la disponibilité des données externes pour le segment donné. Par exemple, si les données sont disponibles toutes les heures, il est possible de retarder le test vérifiant si les données externes sont disponibles et si le segment correspondant est prêt à l’aide de dataDelay.<br/><br/>S’applique uniquement à l’heure actuelle.  Par exemple, s’il est 13 h et si cette valeur est de 10 minutes, la validation commence à 13 h 10.<br/><br/>Ce paramètre n’affecte pas les tranches passées (tranches dont la valeur Heure de fin de la tranche + dataDelay < maintenant) qui sont traitées sans délai.<br/><br/>Les heures supérieures à 23:59 doivent être spécifiées en suivant le format `day.hours:minutes:seconds`. Par exemple, pour spécifier 24 heures, n'utilisez pas 24:00:00 ; utilisez plutôt 1.00:00:00. Si vous utilisez 24:00:00, cette valeur est traitée comme 24 jours (24.00:00:00). Pour 1 jour et 4 heures, spécifiez 1:04:00:00. |Non |0 |
| retryInterval |Délai d'attente entre un échec et la nouvelle tentative. Si une tentative échoue, la prochaine tentative est après retryInterval. <br/><br/>S’il est 13 h actuellement, la première tentative commence. Si la durée de la première vérification de validation est de 1 minute et si l’opération a échoué, la tentative suivante aura lieu à 13 h + 1 minute (durée) + 1 minute (intervalle avant nouvelle tentative) = 13 h 02. <br/><br/>Pour les segments dans le passé, il n’y a aucun délai. La nouvelle tentative se fait immédiatement. |Non |00:01:00 (1 minute) |
| retryTimeout |Délai d’attente pour chaque nouvelle tentative.<br/><br/>Si la propriété est définie sur 10 minutes, la validation doit être effectuée en 10 minutes maximum. S’il faut plus de 10 minutes pour effectuer la validation, la nouvelle tentative expire.<br/><br/>Si toutes les tentatives de validation expirent, le segment est marqué comme TimedOut. |Non |00:10:00 (10 minutes) |
| maximumRetry |Nombre de fois où la disponibilité des données externes est vérifiée. La valeur maximale autorisée est de 10. |Non |3 |


## <a name="data-stores"></a>MAGASINS DE DONNÉES
La section [Service lié](#linked-service) fournit des descriptions pour les éléments JSON qui sont communes à tous les types de services liés. Cette section fournit des informations sur les éléments JSON qui sont spécifiques à chaque magasin de données.

La section [Jeu de données](#dataset) fournit des descriptions pour les éléments JSON qui sont communes à tous les types de jeux de données. Cette section fournit des informations sur les éléments JSON qui sont spécifiques à chaque magasin de données.

La section [Activité](#activity) fournit des descriptions pour les éléments JSON qui sont communes à tous les types d’activités. Cette section fournit des informations sur les éléments JSON qui sont spécifiques à chaque magasin de données lorsqu’ils sont utilisés en tant que source/récepteur dans une activité de copie.

Cliquez sur le lien du magasin qui vous intéresse pour afficher les schémas JSON pour les services liés, les jeux de données et la source/récepteur pour l’activité de copie.

| Category | Banque de données
|:--- |:--- |
| **Microsoft Azure** |[stockage d’objets blob Azure](#azure-blob-storage) |
| &nbsp; |Azure Data Lake Store |
| &nbsp; |[Azure Cosmos DB](#azure-cosmos-db) |
| &nbsp; |[Azure SQL Database](#azure-sql-database) |
| &nbsp; |[Azure Synapse Analytics](#azure-synapse-analytics) |
| &nbsp; |[Recherche cognitive Azure](#azure-cognitive-search) |
| &nbsp; |[Stockage Table Azure](#azure-table-storage) |
| **Bases de données** |[Amazon Redshift](#amazon-redshift) |
| &nbsp; |[IBM DB2](#ibm-db2) |
| &nbsp; |[MySQL](#mysql) |
| &nbsp; |[Oracle](#oracle) |
| &nbsp; |[PostgreSQL](#postgresql) |
| &nbsp; |[SAP Business Warehouse](#sap-business-warehouse) |
| &nbsp; |[SAP HANA](#sap-hana) |
| &nbsp; |[SQL Server](#sql-server) |
| &nbsp; |[Sybase](#sybase) |
| &nbsp; |[Teradata](#teradata) |
| **NoSQL** |[Cassandra](#cassandra) |
| &nbsp; |[MongoDB](#mongodb) |
| **File** |[Amazon S3](#amazon-s3) |
| &nbsp; |[Système de fichiers](#file-system) |
| &nbsp; |[FTP](#ftp) |
| &nbsp; |[HDFS](#hdfs) |
| &nbsp; |[SFTP](#sftp) |
| **Autres** |[HTTP](#http) |
| &nbsp; |[OData](#odata) |
| &nbsp; |[ODBC](#odbc) |
| &nbsp; |[Salesforce](#salesforce) |
| &nbsp; |Table web |

## <a name="azure-blob-storage"></a>Stockage Blob Azure

### <a name="linked-service"></a>Service lié
Il existe deux types de services liés : les services liés de stockage Azure et les services liés SAP de stockage Azure.

#### <a name="azure-storage-linked-service"></a>Service lié Stockage Azure
Pour lier votre compte de stockage Azure à une fabrique de données à l’aide de la **clé de compte**, créez un service lié de stockage Azure. Pour définir un stockage Azure lié au service, définissez le **type** du service lié sur **AzureStorage**. Vous pouvez ensuite spécifier les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| connectionString |Spécifier les informations requises pour la connexion au stockage Azure pour la propriété connectionString. |Oui |

##### <a name="example"></a>Exemple

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

#### <a name="azure-storage-sas-linked-service"></a>Service lié SAP de stockage Azure
Le service lié Stockage Azure SAS vous permet de lier un compte de stockage Azure à une fabrique de données Azure à l’aide d’une signature d’accès partagé (SAP). Ainsi, la fabrique de données dispose d’un accès restreint ou limité dans le temps à tout ou partie des ressources (objet blob/conteneur) dans le stockage. Pour lier votre compte de stockage Azure à une fabrique de données à l’aide de la signature d’accès partagé, créez un service lié SAP de stockage Azure. Pour définir un service lié SAP de stockage Azure, définissez le **type** du service lié sur **AzureStorageSas**. Vous pouvez ensuite spécifier les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| sasUri |Spécifiez l’URI de signature d’accès partagé des ressources Stockage Azure, telles qu’un objet blob, un conteneur ou une table. |Oui |

##### <a name="example"></a>Exemple

```json
{
    "name": "StorageSasLinkedService",
    "properties": {
        "type": "AzureStorageSas",
        "typeProperties": {
            "sasUri": "<storageUri>?<sasToken>"
        }
    }
}
```

Pour plus d’informations sur ces services liés, consultez l’article [Connecteur de stockage Blob Azure](data-factory-azure-blob-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données d’objet blob Azure, définissez le **type** du jeu de données sur **AzureBlob**. Ensuite, spécifiez les propriétés spécifiques d’objet blob Azure suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| folderPath |Chemin d'accès au conteneur et au dossier dans le stockage des objets Blobs. Exemple : monconteneurblob\mondossierblob\ |Oui |
| fileName |Le nom de l’objet Blob. fileName est facultatif et sensible à la casse.<br/><br/>Si vous spécifiez un nom de fichier, l’activité (y compris la copie) fonctionne sur l’objet Blob spécifique.<br/><br/>Lorsque fileName n’est pas spécifié, la copie inclut tous les objets Blob dans le paramètre folderPath du jeu de données d’entrée.<br/><br/>Lorsque fileName n’est pas spécifié pour un jeu de données de sortie, le nom du fichier généré aura ce format dans l’exemple suivant : `Data.<Guid>.txt` (par exemple : : Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |Non |
| partitionedBy |partitionedBy est une propriété facultative. Vous pouvez l'utiliser pour spécifier un folderPath dynamique et le nom de fichier pour les données de série chronologique. Par exemple, folderPath peut être paramétré pour toutes les heures de données. |Non |
| format | Les types de formats suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Définissez la propriété **type** située sous Format sur l’une de ces valeurs. Pour en savoir plus, consultez les sections relatives à [format Text](data-factory-supported-file-and-compression-formats.md#text-format), [format Json](data-factory-supported-file-and-compression-formats.md#json-format), [format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si vous souhaitez **copier des fichiers en l’état** entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie. |Non |
| compression | Spécifiez le type et le niveau de compression pour les données. Types pris en charge : **GZip**, **Deflate**, **BZip2** et **ZipDeflate**. Niveaux pris en charge : **Optimal** et **Fastest**. Pour plus d’informations, consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |Non |

#### <a name="example"></a>Exemple

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


Pour plus d’informations, consultez l’article [Azure Blob connector (connecteur d’objet blob Azure)](data-factory-azure-blob-connector.md#dataset-properties).

### <a name="blobsource-in-copy-activity"></a>BlobSource dans l’activité de copie
Si vous copiez des données à partir d’un Stockage Blob Azure, définissez le **type de source** de l’activité de copie sur **BlobSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| recursive |Indique si les données sont lues de manière récursive dans les sous-dossiers ou uniquement dans le dossier spécifié. |True (valeur par défaut), False |Non |

#### <a name="example-blobsource"></a>Exemple : **BlobSource**
```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoSQL",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureSqlOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "SqlSink"
                }
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```
### <a name="blobsink-in-copy-activity"></a>BlobSink dans l’activité de copie
Si vous copiez des données dans un stockage d’objets blob Azure, définissez le **type de récepteur** de l’activité de copie sur **BlobSink** et spécifiez les propriétés suivantes dans la section **sink** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| copyBehavior |Cette propriété définit le comportement de copie lorsque la source est BlobSource ou FileSystem. |<b>PreserveHierarchy</b> : conserve la hiérarchie des fichiers dans le dossier cible. Le chemin d’accès relatif du fichier source vers le dossier source est identique au chemin d’accès relatif du fichier cible vers le dossier cible.<br/><br/><b>FlattenHierarchy</b> : tous les fichiers du dossier source figurent dans le premier niveau du dossier cible. Le nom des fichiers cibles est généré automatiquement. <br/><br/><b>MergeFiles (par défaut)</b> : fusionne tous les fichiers du dossier source dans un même fichier. Si le nom de fichier/d’objet blob est spécifié, le nom de fichier fusionné est le nom spécifié. Dans le cas contraire, le nom de fichier est généré automatiquement. |Non |

#### <a name="example-blobsink"></a>Exemple : BlobSink

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureSQLtoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureSQLInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlSource",
                    "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Pour plus d’informations, consultez l’article [Azure Blob connector (connecteur d’objet blob Azure)](data-factory-azure-blob-connector.md#copy-activity-properties).

## <a name="azure-data-lake-store"></a>Azure Data Lake Store

### <a name="linked-service"></a>Service lié
Pour définir un service lié Azure Data Lake Store, définissez le type du service lié sur **AzureDataLakeStore** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur : **AzureDataLakeStore** | Oui |
| dataLakeStoreUri | Spécifiez des informations à propos du compte Azure Data Lake Store. Il se présente au format suivant : `https://[accountname].azuredatalakestore.net/webhdfs/v1` ou `adl://[accountname].azuredatalakestore.net/`. | Oui |
| subscriptionId | ID d’abonnement Azure auquel appartient Data Lake Store. | Requis pour le récepteur |
| resourceGroupName | Nom du groupe de ressources Azure auquel appartient le magasin Data Lake Store. | Requis pour le récepteur |
| servicePrincipalId | Spécifiez l’ID client de l’application. | Oui (pour l’authentification du principal du service) |
| servicePrincipalKey | Spécifiez la clé de l’application. | Oui (pour l’authentification du principal du service) |
| tenant | Spécifiez les informations de locataire (nom de domaine ou ID de locataire) dans lesquels se trouve votre application. Vous pouvez le récupérer en pointant la souris dans le coin supérieur droit du Portail Azure. | Oui (pour l’authentification du principal du service) |
| autorisation | Cliquez sur le bouton **Autoriser** dans **Data Factory Editor** et saisissez vos informations d’identification, ce qui affecte l’URL d’autorisation générée automatiquement à cette propriété. | Oui (pour l’authentification des informations d’identification utilisateur)|
| sessionID | ID de session OAuth issu de la session d’autorisation OAuth. Chaque ID de session est unique et ne peut être utilisé qu’une seule fois. Ce paramètre est généré automatiquement lorsque vous utilisez Data Factory Editor. | Oui (pour l’authentification des informations d’identification utilisateur) |

#### <a name="example-using-service-principal-authentication"></a>Exemple : utilisation de l’authentification d’un principal du service
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info. Example: microsoft.onmicrosoft.com>"
        }
    }
}
```

#### <a name="example-using-user-credential-authentication"></a>Exemple : utilisation de l’authentification des informations d’identification utilisateur
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "sessionId": "<session ID>",
            "authorization": "<authorization URL>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [Azure Data Lake Store connector (connecteur Azure Data Lake Store)](data-factory-azure-datalake-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données Azure Data Lake Store, définissez le **type** du jeu de données sur **AzureDataLakeStore** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| folderPath |Chemin d’accès au conteneur et au dossier dans le magasin Azure Data Lake |Oui |
| fileName |Le nom du fichier dans le magasin Azure Data Lake. fileName est facultatif et sensible à la casse. <br/><br/>Si vous spécifiez un nom de fichier, l’activité (y compris la copie) fonctionne sur le fichier spécifique.<br/><br/>Lorsque fileName n’est pas spécifié, la copie inclut tous les fichiers dans le paramètre folderPath du jeu de données d’entrée.<br/><br/>Lorsque fileName n’est pas spécifié pour un jeu de données de sortie, le nom du fichier généré aura ce format dans l’exemple suivant : `Data.<Guid>.txt` (par exemple : : Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |Non |
| partitionedBy |partitionedBy est une propriété facultative. Vous pouvez l'utiliser pour spécifier un folderPath dynamique et le nom de fichier pour les données de série chronologique. Par exemple, folderPath peut être paramétré pour toutes les heures de données. |Non |
| format | Les types de formats suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Définissez la propriété **type** située sous Format sur l’une de ces valeurs. Pour en savoir plus, consultez les sections relatives à [format Text](data-factory-supported-file-and-compression-formats.md#text-format), [format Json](data-factory-supported-file-and-compression-formats.md#json-format), [format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si vous souhaitez **copier des fichiers en l’état** entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie. |Non |
| compression | Spécifiez le type et le niveau de compression pour les données. Types pris en charge : **GZip**, **Deflate**, **BZip2** et **ZipDeflate**. Niveaux pris en charge : **Optimal** et **Fastest**. Pour plus d’informations, consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |Non |

#### <a name="example"></a>Exemple
```json
{
    "name": "AzureDataLakeStoreInput",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/input/",
            "fileName": "SearchLog.tsv",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
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

Pour plus d’informations, consultez l’article [Azure Data Lake Store connector (connecteur Azure Data Lake Store)](data-factory-azure-datalake-connector.md#dataset-properties).

### <a name="azure-data-lake-store-source-in-copy-activity"></a>Source Azure Data Lake Store dans l’activité de copie
Si vous copiez des données à partir d’Azure Data Lake Store, définissez le **type de source** de l’activité de copie sur **AzureDataLakeStoreSource** et spécifiez les propriétés suivantes dans la section **source** :

**AzureDataLakeStoreSource** prend en charge les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| recursive |Indique si les données sont lues de manière récursive dans les sous-dossiers ou uniquement dans le dossier spécifié. |True (valeur par défaut), False |Non |

#### <a name="example-azuredatalakestoresource"></a>Exemple : AzureDataLakeStoreSource

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureDakeLaketoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureDataLakeStoreInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "AzureDataLakeStoreSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Pour plus d’informations, consultez l’article [Azure Data Lake Store connector (connecteur Azure Data Lake Store)](data-factory-azure-datalake-connector.md#copy-activity-properties).

### <a name="azure-data-lake-store-sink-in-copy-activity"></a>Récepteur Azure Data Lake Store dans l’activité de copie
Si vous copiez des données dans un Azure Data Lake Store, définissez le **type de récepteur** de l’activité de copie sur **AzureDataLakeStoreSink** et spécifiez les propriétés suivantes dans la section **sink** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| copyBehavior |Spécifie le comportement de copie. |<b>PreserveHierarchy</b> : conserve la hiérarchie des fichiers dans le dossier cible. Le chemin d’accès relatif du fichier source vers le dossier source est identique au chemin d’accès relatif du fichier cible vers le dossier cible.<br/><br/><b>FlattenHierarchy</b> : tous les fichiers du dossier source sont créés dans le premier niveau du dossier cible. Les fichiers cibles sont créés avec le nom généré automatiquement.<br/><br/><b>MergeFiles</b> : fusionne tous les fichiers du dossier source dans un même fichier. Si le nom de fichier/d’objet blob est spécifié, le nom de fichier fusionné est le nom spécifié. Dans le cas contraire, le nom de fichier est généré automatiquement. |Non |

#### <a name="example-azuredatalakestoresink"></a>Exemple : AzureDataLakeStoreSink
```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoDataLake",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureDataLakeStoreOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "AzureDataLakeStoreSink"
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
        }]
    }
}
```

Pour plus d’informations, consultez l’article [Azure Data Lake Store connector (connecteur Azure Data Lake Store)](data-factory-azure-datalake-connector.md#copy-activity-properties).

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

### <a name="linked-service"></a>Service lié
Pour définir un service lié Azure Cosmos DB, réglez le **type** du service lié sur **DocumentDb** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| **Propriété** | **Description** | **Obligatoire** |
| --- | --- | --- |
| connectionString |Spécifiez les informations requises pour se connecter à la base de données Azure Cosmos DB. |Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "CosmosDBLinkedService",
    "properties": {
        "type": "DocumentDb",
        "typeProperties": {
            "connectionString": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
        }
    }
}
```
Pour plus d’informations, consultez l’article sur le [connecteur d’objet blob Azure](data-factory-azure-documentdb-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données Azure Cosmos DB, réglez le **type** du jeu de données sur **DocumentDbCollection** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| **Propriété** | **Description** | **Obligatoire** |
| --- | --- | --- |
| collectionName |Nom de la collection Azure Cosmos DB. |Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "PersonCosmosDBTable",
    "properties": {
        "type": "DocumentDbCollection",
        "linkedServiceName": "CosmosDBLinkedService",
        "typeProperties": {
            "collectionName": "Person"
        },
        "external": true,
        "availability": {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```
Pour plus d’informations, consultez l’article sur le [connecteur d’objet blob Azure](data-factory-azure-documentdb-connector.md#dataset-properties).

### <a name="azure-cosmos-db-collection-source-in-copy-activity"></a>Source de la collection Azure Cosmos DB dans l’activité de copie
Si vous copiez des données à partir d’Azure Cosmos DB, réglez le **type de source** de l’activité de copie sur **DocumentDbCollectionSource** et spécifiez les propriétés suivantes dans la section **source** :


| **Propriété** | **Description** | **Valeurs autorisées** | **Obligatoire** |
| --- | --- | --- | --- |
| query |Spécifier la requête pour lire les données. |Chaîne de requête prise en charge par Azure Cosmos DB. <br/><br/>Exemple : `SELECT c.BusinessEntityID, c.PersonType, c.NameStyle, c.Title, c.Name.First AS FirstName, c.Name.Last AS LastName, c.Suffix, c.EmailPromotion FROM c WHERE c.ModifiedDate > \"2009-01-01T00:00:00\"` |Non <br/><br/>Si non spécifié, l’instruction SQL exécutée : `select <columns defined in structure> from mycollection` |
| nestingSeparator |Caractère spécial pour indiquer que le document est imbriqué. |Tout caractère. <br/><br/>Azure Cosmos DB est une banque NoSQL de documents JSON, où les structures imbriquées sont autorisées. Azure Data Factory permet à l'utilisateur de désigner la hiérarchie via nestingSeparator, qui est « . » dans les exemples ci-dessus. Avec le séparateur, l'activité de copie générera l'objet « Name » avec trois éléments enfants First, Middle et Last, en fonction de « Name.First », « Name.Middle » et « Name.Last » dans la définition de la table. |Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "DocDbToBlobPipeline",
    "properties": {
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "DocumentDbCollectionSource",
                    "query": "SELECT Person.Id, Person.Name.First AS FirstName, Person.Name.Middle as MiddleName, Person.Name.Last AS LastName FROM Person",
                    "nestingSeparator": "."
                },
                "sink": {
                    "type": "BlobSink",
                    "blobWriterAddHeader": true,
                    "writeBatchSize": 1000,
                    "writeBatchTimeout": "00:00:59"
                }
            },
            "inputs": [{
                "name": "PersonCosmosDBTable"
            }],
            "outputs": [{
                "name": "PersonBlobTableOut"
            }],
            "policy": {
                "concurrency": 1
            },
            "name": "CopyFromCosmosDbToBlob"
        }],
        "start": "2016-04-01T00:00:00",
        "end": "2016-04-02T00:00:00"
    }
}
```

### <a name="azure-cosmos-db-collection-sink-in-copy-activity"></a>Récepteur de collection Azure Cosmos DB dans l’activité de copie
Si vous copiez des données dans Azure Cosmos DB, réglez le **type de récepteur** de l’activité de copie sur **DocumentDbCollectionSink** et spécifiez les propriétés suivantes dans la section **sink** :

| **Propriété** | **Description** | **Valeurs autorisées** | **Obligatoire** |
| --- | --- | --- | --- |
| nestingSeparator |Caractère spécial dans le nom de colonne source pour indiquer que le document imbriqué est nécessaire. <br/><br/>Par exemple, ci-dessus : `Name.First` dans la table de sortie produit la structure JSON suivante dans le document Cosmos DB :<br/><br/>"Name": {<br/>    "First": "John"<br/>}, |Caractère utilisé pour séparer les niveaux d'imbrication.<br/><br/>La valeur par défaut est `.` (point). |Caractère utilisé pour séparer les niveaux d'imbrication. <br/><br/>La valeur par défaut est `.` (point). |
| writeBatchSize |Nombre de requêtes parallèles auprès du service Azure Cosmos DB pour créer des documents.<br/><br/>Vous pouvez optimiser les performances lors de la copie des données dans/à partir d’Azure Cosmos DB à l’aide de cette propriété. Vous pouvez obtenir de meilleures performances en augmentant writeBatchSize car davantage de requêtes sont envoyées à Azure Cosmos DB. Toutefois, vous devez éviter les limitations qui peuvent déclencher le message d’erreur : « Le taux de requêtes est élevé ».<br/><br/>Une limitation dépend de divers facteurs, dont la taille des documents, le nombre de termes qu’ils contiennent, la stratégie d’indexation de la collection cible, etc. Pour les opérations de copie, vous pouvez utiliser une meilleure collection (par exemple, S3) pour que le débit disponible soit maximal (2 500 unités de demande par seconde). |Integer |Non (valeur par défaut : 5) |
| writeBatchTimeout |Temps d'attente pour que l'opération soit terminée avant d'expirer. |intervalle de temps<br/><br/> Exemple : “00:30:00” (30 minutes). |Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "BlobToDocDbPipeline",
    "properties": {
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "DocumentDbCollectionSink",
                    "nestingSeparator": ".",
                    "writeBatchSize": 2,
                    "writeBatchTimeout": "00:00:00"
                },
                "translator": {
                    "type": "TabularTranslator",
                    "ColumnMappings": "FirstName: Name.First, MiddleName: Name.Middle, LastName: Name.Last, BusinessEntityID: BusinessEntityID, PersonType: PersonType, NameStyle: NameStyle, Title: Title, Suffix: Suffix"
                }
            },
            "inputs": [{
                "name": "PersonBlobTableIn"
            }],
            "outputs": [{
                "name": "PersonCosmosDbTableOut"
            }],
            "policy": {
                "concurrency": 1
            },
            "name": "CopyFromBlobToCosmosDb"
        }],
        "start": "2016-04-14T00:00:00",
        "end": "2016-04-15T00:00:00"
    }
}
```

Pour plus d’informations, consultez l’article sur le [connecteur d’objet blob Azure](data-factory-azure-documentdb-connector.md#copy-activity-properties).

## <a name="azure-sql-database"></a>Azure SQL Database

### <a name="linked-service"></a>Service lié
Pour définir un service lié Azure SQL Database, définissez le **type** du service lié sur **AzureSqlDatabase** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| connectionString |Spécifier les informations requises pour la connexion à l’instance Azure SQL Database pour la propriété connectionString. |Oui |

#### <a name="example"></a>Exemple
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

Pour plus d’informations, consultez l’article [Azure SQL connector (connecteur Azure SQL)](data-factory-azure-sql-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données Azure SQL Database, définissez le **type** du jeu de données sur **AzureSqlTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table ou de la vue dans l’instance Azure SQL Database à laquelle le service lié fait référence. |Oui |

#### <a name="example"></a>Exemple

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
Pour plus d’informations, consultez l’article [Azure SQL connector (connecteur Azure SQL)](data-factory-azure-sql-connector.md#dataset-properties).

### <a name="sql-source-in-copy-activity"></a>Source SQL dans l’activité de copie
Si vous copiez des données à partir d’Azure SQL Database, définissez le **type de source** de l’activité de copie sur **SqlSource** et spécifiez les propriétés suivantes dans la section **source** :


| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| sqlReaderQuery |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Exemple : `select * from MyTable`. |Non |
| sqlReaderStoredProcedureName |Nom de la procédure stockée qui lit les données de la table source. |Nom de la procédure stockée. |Non |
| storedProcedureParameters |Paramètres de la procédure stockée. |Paires nom/valeur. Les noms et la casse des paramètres doivent correspondre aux noms et à la casse des paramètres de la procédure stockée. |Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureSQLtoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureSQLInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
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
        }]
    }
}
```
Pour plus d’informations, consultez l’article [Azure SQL connector (connecteur Azure SQL)](data-factory-azure-sql-connector.md#copy-activity-properties).

### <a name="sql-sink-in-copy-activity"></a>Récepteur SQL dans l’activité de copie
Si vous copiez des données dans Azure SQL Database, définissez le **type de récepteur** de l’activité de copie sur **SqlSink** et spécifiez les propriétés suivantes dans la section **sink** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| writeBatchTimeout |Temps d’attente pour que l’opération d’insertion de lot soit terminée avant d’expirer. |intervalle de temps<br/><br/> Exemple : “00:30:00” (30 minutes). |Non |
| writeBatchSize |Insère des données dans la table SQL lorsque la taille du tampon atteint writeBatchSize |Nombre entier (nombre de lignes) |Non (valeur par défaut : 10000) |
| sqlWriterCleanupScript |Spécifiez une requête pour exécuter l’activité de copie afin que les données d’un segment spécifique soient nettoyées. |Une instruction de requête. |Non |
| sliceIdentifierColumnName |Spécifiez le nom de la colonne que l’activité de copie doit remplir avec l’identificateur de segment généré automatiquement, qui est utilisé pour nettoyer les données d’un segment spécifique lors de la ré-exécution. |Nom d’une colonne avec le type de données binary(32). |Non |
| sqlWriterStoredProcedureName |Nom de la procédure stockée qui met à jour/insère les données dans la table cible. |Nom de la procédure stockée. |Non |
| storedProcedureParameters |Paramètres de la procédure stockée. |Paires nom/valeur. Les noms et la casse des paramètres doivent correspondre aux noms et à la casse des paramètres de la procédure stockée. |Non |
| sqlWriterTableType |Spécifiez le nom du type de table à utiliser dans la procédure stockée. L’activité de copie place les données déplacées disponibles dans une table temporaire avec ce type de table. Le code de procédure stockée peut ensuite fusionner les données copiées avec les données existantes. |Nom de type de table. |Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoSQL",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureSqlOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource",
                    "blobColumnSeparators": ","
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
        }]
    }
}
```

Pour plus d’informations, consultez l’article [Azure SQL connector (connecteur Azure SQL)](data-factory-azure-sql-connector.md#copy-activity-properties).

## <a name="azure-synapse-analytics"></a>Azure Synapse Analytics

### <a name="linked-service"></a>Service lié
Pour définir un service lié Azure Synapse Analytics, définissez le **type** du service lié sur **AzureSqlDW** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| connectionString |Spécifiez les informations requises pour la connexion à l’instance Azure Synapse Analytics pour la propriété connectionString. |Oui |



#### <a name="example"></a>Exemple

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        }
    }
}
```

Pour plus d’informations, consultez l’article [Connecteur Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données Azure Synapse Analytics, définissez le **type** du jeu de données sur **AzureSqlDWTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table ou de la vue dans la base de données Azure Synapse Analytics à laquelle le service lié fait référence. |Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "AzureSqlDWInput",
    "properties": {
    "type": "AzureSqlDWTable",
        "linkedServiceName": "AzureSqlDWLinkedService",
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

Pour plus d’informations, consultez l’article [Connecteur Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#dataset-properties).

### <a name="azure-synapse-analytics-source-in-copy-activity"></a>Source Azure Synapse Analytics dans l’activité de copie
Si vous copiez des données à partir d’Azure Synapse Analytics, définissez le **type de source** de l’activité de copie sur **SqlDWSource** et spécifiez les propriétés suivantes dans la section **source** :


| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| sqlReaderQuery |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Par exemple : `select * from MyTable`. |Non |
| sqlReaderStoredProcedureName |Nom de la procédure stockée qui lit les données de la table source. |Nom de la procédure stockée. |Non |
| storedProcedureParameters |Paramètres de la procédure stockée. |Paires nom/valeur. Les noms et la casse des paramètres doivent correspondre aux noms et à la casse des paramètres de la procédure stockée. |Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureSQLDWtoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureSqlDWInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlDWSource",
                    "sqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
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
        }]
    }
}
```

Pour plus d’informations, consultez l’article [Connecteur Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#copy-activity-properties).

### <a name="azure-synapse-analytics-sink-in-copy-activity"></a>Récepteur Azure Synapse Analytics dans l’activité de copie
Si vous copiez des données dans Azure Synapse Analytics, définissez le **type de récepteur** de l’activité de copie sur **SqlDWSink** et spécifiez les propriétés suivantes dans la section **sink** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| sqlWriterCleanupScript |Spécifiez une requête pour exécuter l’activité de copie afin que les données d’un segment spécifique soient nettoyées. |Une instruction de requête. |Non |
| allowPolyBase |Indique s’il faut utiliser PolyBase (le cas échéant) au lieu du mécanisme BULKINSERT. <br/><br/> **Utiliser PolyBase est la méthode recommandée pour charger des données dans Azure Synapse Analytics.** |True <br/>False (valeur par défaut) |Non |
| polyBaseSettings |Groupe de propriétés pouvant être spécifié lorsque la propriété **allowPolybase** est définie sur **true**. |&nbsp; |Non |
| rejectValue |Spécifie le nombre ou le pourcentage de lignes pouvant être rejetées avant l’échec de la requête. <br/><br/>Pour en savoir plus sur les options de rejet de PolyBase dans la section **Arguments** de la rubrique [CREATE EXTERNAL TABLE (Transact-SQL)](/sql/t-sql/statements/create-external-table-transact-sql) (Créer une table externe (Transact-SQL)). |0 (par défaut), 1, 2, … |Non |
| rejectType |Spécifie si l’option rejectValue est spécifiée comme une valeur littérale ou un pourcentage. |Value (par défaut), Percentage |Non |
| rejectSampleValue |Détermine le nombre de lignes à extraire avant que PolyBase recalcule le pourcentage de lignes rejetées. |1, 2, … |Oui, si le **rejectType** est **percentage** |
| useTypeDefault |Spécifie comment gérer les valeurs manquantes dans les fichiers texte délimité lorsque PolyBase récupère les données à partir du fichier texte.<br/><br/>Pour plus d’informations sur cette propriété, consultez la section Arguments dans [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](/sql/t-sql/statements/create-external-file-format-transact-sql). |True, False (par défaut) |Non |
| writeBatchSize |Insère des données dans la table SQL lorsque la taille du tampon atteint writeBatchSize |Nombre entier (nombre de lignes) |Non (valeur par défaut : 10000) |
| writeBatchTimeout |Temps d’attente pour que l’opération d’insertion de lot soit terminée avant d’expirer. |intervalle de temps<br/><br/> Exemple : “00:30:00” (30 minutes). |Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoSQLDW",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureSqlDWOutput"
            }],
            "typeProperties": {
                "source": {
                "type": "BlobSource",
                    "blobColumnSeparators": ","
                },
                "sink": {
                    "type": "SqlDWSink",
                    "allowPolyBase": true
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
        }]
    }
}
```

Pour plus d’informations, consultez l’article [Connecteur Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#copy-activity-properties).

## <a name="azure-cognitive-search"></a>Recherche cognitive Azure

### <a name="linked-service"></a>Service lié
Pour définir un service lié Recherche cognitive Azure, définissez son **type** sur **AzureSearch** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| -------- | ----------- | -------- |
| url | URL du service de recherche. | Oui |
| key | Clé d’administration du service de recherche. | Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "AzureSearchLinkedService",
    "properties": {
        "type": "AzureSearch",
        "typeProperties": {
            "url": "https://<service>.search.windows.net",
            "key": "<AdminKey>"
        }
    }
}
```

Pour plus d’informations, voir l’article [Connecteur Recherche cognitive Azure](data-factory-azure-search-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données Recherche cognitive Azure, définissez son **type** sur **AzureSearchIndex** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| -------- | ----------- | -------- |
| type | La propriété de type doit être définie sur **AzureSearchIndex**.| Oui |
| indexName | Nom de l’index de recherche. Data Factory ne crée pas l’index. L’index doit exister dans Recherche cognitive Azure. | Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "AzureSearchIndexDataset",
    "properties": {
        "type": "AzureSearchIndex",
        "linkedServiceName": "AzureSearchLinkedService",
        "typeProperties": {
            "indexName": "products"
        },
        "availability": {
            "frequency": "Minute",
            "interval": 15
        }
    }
}
```

Pour plus d’informations, voir l’article [Connecteur Recherche cognitive Azure](data-factory-azure-search-connector.md#dataset-properties).

### <a name="azure-cognitive-search-index-sink-in-copy-activity"></a>Récepteur de l’index Recherche cognitive Azure dans l’activité de copie
Si vous copiez des données dans un index de recherche, définissez le **type de récepteur** de l’activité de copie sur **AzureSearchIndexSink** et spécifiez les propriétés suivantes dans la section **sink** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| -------- | ----------- | -------------- | -------- |
| WriteBehavior | Indique s’il convient de procéder à une fusion ou à un remplacement lorsqu’un document existe déjà dans l’index. | Merge (par défaut)<br/>Télécharger| Non |
| writeBatchSize | Charge des données dans l’index de recherche lorsque la taille de la mémoire tampon atteint writeBatchSize. | 1 à 1 000. Valeur par défaut : 1 000. | Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "SqlServertoAzureSearchIndex",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": " SqlServerInput"
            }],
            "outputs": [{
                "name": "AzureSearchIndexDataset"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlSource",
                    "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "AzureSearchIndexSink"
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
        }]
    }
}
```

Pour plus d’informations, voir l’article [Connecteur Recherche cognitive Azure](data-factory-azure-search-connector.md#copy-activity-properties).

## <a name="azure-table-storage"></a>Stockage de table Azure

### <a name="linked-service"></a>Service lié
Il existe deux types de services liés : les services liés de stockage Azure et les services liés SAP de stockage Azure.

#### <a name="azure-storage-linked-service"></a>Service lié Stockage Azure
Pour lier votre compte de stockage Azure à une fabrique de données à l’aide de la **clé de compte**, créez un service lié de stockage Azure. Pour définir un stockage Azure lié au service, définissez le **type** du service lié sur **AzureStorage**. Vous pouvez ensuite spécifier les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type |La propriété type doit être définie sur : **AzureStorage** |Oui |
| connectionString |Spécifier les informations requises pour la connexion au stockage Azure pour la propriété connectionString. |Oui |

**Exemple :**

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

#### <a name="azure-storage-sas-linked-service"></a>Service lié SAP de stockage Azure
Le service lié Stockage Azure SAS vous permet de lier un compte de stockage Azure à une fabrique de données Azure à l’aide d’une signature d’accès partagé (SAP). Ainsi, la fabrique de données dispose d’un accès restreint ou limité dans le temps à tout ou partie des ressources (objet blob/conteneur) dans le stockage. Pour lier votre compte de stockage Azure à une fabrique de données à l’aide de la signature d’accès partagé, créez un service lié SAP de stockage Azure. Pour définir un service lié SAP de stockage Azure, définissez le **type** du service lié sur **AzureStorageSas**. Vous pouvez ensuite spécifier les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type |La propriété type doit être définie sur : **AzureStorageSas** |Oui |
| sasUri |Spécifiez l’URI de signature d’accès partagé des ressources Stockage Azure, telles qu’un objet blob, un conteneur ou une table. |Oui |

**Exemple :**

```json
{
    "name": "StorageSasLinkedService",
    "properties": {
        "type": "AzureStorageSas",
        "typeProperties": {
            "sasUri": "<storageUri>?<sasToken>"
        }
    }
}
```

Pour plus d’informations sur ces services liés, consultez l’article [Connecteur de stockage Table Azure](data-factory-azure-table-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données Table Azure, définissez le **type** du jeu de données sur **AzureTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table dans l'instance de base de données Table Azure à laquelle le service lié fait référence. |Oui. Lorsqu’un tableName est spécifié sans azureTableSourceQuery, tous les enregistrements de la table sont copiés vers la destination. Si un azureTableSourceQuery est également spécifié, les enregistrements de la table qui satisfont à la requête sont copiés vers la destination. |

#### <a name="example"></a>Exemple

```json
{
    "name": "AzureTableInput",
    "properties": {
        "type": "AzureTable",
        "linkedServiceName": "StorageLinkedService",
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

Pour plus d’informations sur ces services liés, consultez l’article [Connecteur de stockage Table Azure](data-factory-azure-table-connector.md#dataset-properties).

### <a name="azure-table-source-in-copy-activity"></a>Source Table Azure dans l’activité de copie
Si vous copiez des données à partir d’un stockage de table Azure, définissez le **type de source** de l’activité de copie sur **AzureTableSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| AzureTableSourceQuery |Utilise la requête personnalisée pour lire des données. |Chaîne de requête de table Azure. Consultez les exemples dans la section suivante. |Non. Lorsqu’un tableName est spécifié sans azureTableSourceQuery, tous les enregistrements de la table sont copiés vers la destination. Si un azureTableSourceQuery est également spécifié, les enregistrements de la table qui satisfont à la requête sont copiés vers la destination. |
| azureTableSourceIgnoreTableNotFound |Indiquer si l'exception de la table n'existe pas. |TRUE<br/>FALSE |Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureTabletoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureTableInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "AzureTableSource",
                    "AzureTableSourceQuery": "PartitionKey eq 'DefaultPartitionKey'"
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
        }]
    }
}
```

Pour plus d’informations sur ces services liés, consultez l’article [Connecteur de stockage Table Azure](data-factory-azure-table-connector.md#copy-activity-properties).

### <a name="azure-table-sink-in-copy-activity"></a>Récepteur Table Azure dans l’activité de copie
Si vous copiez des données dans un stockage de table Azure, définissez le **type de récepteur** de l’activité de copie sur **AzureTableSink** et spécifiez les propriétés suivantes dans la section **sink** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| azureTableDefaultPartitionKeyValue |Valeur de clé de partition par défaut qui peut être utilisée par le récepteur. |Valeur de chaîne. |Non |
| azureTablePartitionKeyName |Spécifiez le nom de la colonne dont les valeurs sont utilisées comme clés de partition. Si aucune valeur n'est spécifiée, AzureTableDefaultPartitionKeyValue est utilisée comme clé de partition. |Nom de colonne. |Non |
| azureTableRowKeyName |Spécifiez le nom de la colonne dont les valeurs sont utilisées comme clé de ligne. Si aucune valeur n'est spécifiée, un GUID est utilisé pour chaque ligne. |Nom de colonne. |Non |
| azureTableInsertType |Le mode d’insertion des données dans une table Azure.<br/><br/>Cette propriété détermine le remplacement ou la fusion des valeurs des lignes existantes dans la table de sortie avec des clés de partition et de ligne correspondantes. <br/><br/>Consultez [Insertion ou fusion d’entité](/rest/api/storageservices/Insert-Or-Merge-Entity) et [Insertion ou remplacement d’entité](/rest/api/storageservices/Insert-Or-Replace-Entity) pour en savoir plus sur le fonctionnement de ces paramètres (fusion et remplacement). <br/><br> Ce paramètre s’applique au niveau de la ligne, non au niveau de la table, et aucune option ne supprime des lignes de la table de sortie qui n’existent pas dans l’entrée. |fusionner (par défaut)<br/>remplacer |Non |
| writeBatchSize |Insère des données dans la table Azure lorsque la valeur de writeBatchSize ou writeBatchTimeout est atteinte. |Nombre entier (nombre de lignes) |Non (valeur par défaut : 10000) |
| writeBatchTimeout |Insère des données dans la table Azure lorsque la valeur de writeBatchSize ou writeBatchTimeout est atteinte |intervalle de temps<br/><br/>Exemple : "00:20:00" (20 minutes) |Non (Valeur par défaut du délai d'attente du stockage client par défaut : 90 secondes) |

#### <a name="example"></a>Exemple

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoTable",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureTableOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "AzureTableSink",
                    "writeBatchSize": 100,
                    "writeBatchTimeout": "01:00:00"
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
        }]
    }
}
```
Pour plus d’informations sur ces services liés, consultez l’article [Connecteur de stockage Table Azure](data-factory-azure-table-connector.md#copy-activity-properties).

## <a name="amazon-redshift"></a>Amazon Redshift

### <a name="linked-service"></a>Service lié
Pour définir un service lié Amazon Redshift, définissez le **type** du service lié sur **AmazonRedshift** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| server |Nom d’hôte ou adresse IP du serveur Amazon Redshift. |Oui |
| port |Le numéro du port TCP utilisé par le serveur Amazon Redshift pour écouter les connexions clientes. |Aucune valeur par défaut : 5439 |
| database |Nom de la base de données Amazon Redshift. |Oui |
| username |Nom d’utilisateur ayant accès à la base de données. |Oui |
| mot de passe |Mot de passe du compte d’utilisateur. |Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "AmazonRedshiftLinkedService",
    "properties": {
        "type": "AmazonRedshift",
        "typeProperties": {
            "server": "<Amazon Redshift host name or IP address>",
            "port": 5439,
            "database": "<database name>",
            "username": "user",
            "password": "password"
        }
    }
}
```

Pour plus d'informations, consultez l'article Connecteur Amazon Redshift.

### <a name="dataset"></a>Dataset
Pour définir un jeu de données Amazon Redshift, définissez le **type** du jeu de données sur **RelationalTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table dans l’instance de base de données Amazon Redshift à laquelle le service lié fait référence. |Non (si la **requête** de **RelationalSource** est spécifiée) |


#### <a name="example"></a>Exemple

```json
{
    "name": "AmazonRedshiftInputDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "AmazonRedshiftLinkedService",
        "typeProperties": {
            "tableName": "<Table name>"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
Pour plus d'informations, consultez l'article Connecteur Amazon Redshift.

### <a name="relational-source-in-copy-activity"></a>Source relationnelle dans l’activité de copie
Si vous copiez des données à partir d’Amazon Redshift, définissez le **type de source** de l’activité de copie sur **RelationalSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Par exemple : `select * from MyTable`. |Non (si **tableName** de **dataset** est spécifiée) |

#### <a name="example"></a>Exemple

```json
{
    "name": "CopyAmazonRedshiftToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "AmazonRedshiftInputDataset"
            }],
            "outputs": [{
                "name": "AzureBlobOutputDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "AmazonRedshiftToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```
Pour plus d'informations, consultez l'article Connecteur Amazon Redshift.

## <a name="ibm-db2"></a>IBM DB2

### <a name="linked-service"></a>Service lié
Pour définir un service lié IBM DB2, définissez le **type** du service lié sur **OnPremisesDB2** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| server |Nom du serveur DB2. |Oui |
| database |Nom de la base de données DB2. |Oui |
| schéma |Nom du schéma dans la base de données. Le nom du schéma respecte la casse. |Non |
| authenticationType |Type d'authentification utilisé pour se connecter à la base de données DB2. Les valeurs possibles sont les suivantes : Anonymous, Basic et Windows. |Oui |
| username |Spécifiez le nom d'utilisateur si vous utilisez l'authentification de base ou Windows. |Non |
| mot de passe |Spécifiez le mot de passe du compte d’utilisateur que vous avez spécifié pour le nom d’utilisateur. |Non |
| gatewayName |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter à la base de données DB2 locale. |Oui |

#### <a name="example"></a>Exemple
```json
{
    "name": "OnPremDb2LinkedService",
    "properties": {
        "type": "OnPremisesDb2",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "schema": "<schema>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```
Pour plus d'informations, consultez l'article Connecteur IBM DB2.

### <a name="dataset"></a>Dataset
Pour définir un jeu de données DB2, définissez le **type** du jeu de données sur **RelationalTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table dans l'instance de base de données DB2 à laquelle le service lié fait référence. Le nom de la table respecte la casse. |Non (si la **requête** de **RelationalSource** est spécifiée)

#### <a name="example"></a>Exemple
```json
{
    "name": "Db2DataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremDb2LinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
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

Pour plus d'informations, consultez l'article Connecteur IBM DB2.

### <a name="relational-source-in-copy-activity"></a>Source relationnelle dans l’activité de copie
Si vous copiez des données à partir d’IBM DB2, définissez le **type de source** de l’activité de copie sur **RelationalSource** et spécifiez les propriétés suivantes dans la section **source** :


| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Par exemple : `"query": "select * from "MySchema"."MyTable""`. |Non (si **tableName** de **dataset** est spécifiée) |

#### <a name="example"></a>Exemple
```json
{
    "name": "CopyDb2ToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "select * from \"Orders\""
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "inputs": [{
                "name": "Db2DataSet"
            }],
            "outputs": [{
                "name": "AzureBlobDb2DataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "Db2ToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```
Pour plus d'informations, consultez l'article Connecteur IBM DB2.

## <a name="mysql"></a>MySQL

### <a name="linked-service"></a>Service lié
Pour définir un service lié MySQL, définissez le **type** du service lié sur **OnPremisesMySql** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| server |Nom du serveur MySQL. |Oui |
| database |Nom de la base de données MySQL. |Oui |
| schéma |Nom du schéma dans la base de données. |Non |
| authenticationType |Type d'authentification utilisé pour se connecter à la base de données MySQL. Les valeurs possibles sont les suivantes : `Basic`. |Oui |
| userName |Spécifiez le nom d’utilisateur associé à la connexion à la base de données MySQL. |Oui |
| mot de passe |Spécifiez le mot de passe du compte d’utilisateur que vous avez indiqué. |Oui |
| gatewayName |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter à la base de données MySQL locale. |Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "OnPremMySqlLinkedService",
    "properties": {
        "type": "OnPremisesMySql",
        "typeProperties": {
            "server": "<server name>",
            "database": "<database name>",
            "schema": "<schema name>",
            "authenticationType": "<authentication type>",
            "userName": "<user name>",
            "password": "<password>",
            "gatewayName": "<gateway>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [MySQL connector (connecteur MySQL)](data-factory-onprem-mysql-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données MySQL, définissez le **type** du jeu de données sur **RelationalTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table dans l'instance de base de données MySQL à laquelle le service lié fait référence. |Non (si la **requête** de **RelationalSource** est spécifiée) |

#### <a name="example"></a>Exemple

```json
{
    "name": "MySqlDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremMySqlLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
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
Pour plus d’informations, consultez l’article [MySQL connector (connecteur MySQL)](data-factory-onprem-mysql-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Source relationnelle dans l’activité de copie
Si vous copiez des données à partir d’une base de données MySQL, définissez le **type de source** de l’activité de copie sur **RelationalSource** et spécifiez les propriétés suivantes dans la section **source** :


| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Par exemple : `select * from MyTable`. |Non (si **tableName** de **dataset** est spécifiée) |


#### <a name="example"></a>Exemple
```json
{
    "name": "CopyMySqlToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "MySqlDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobMySqlDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "MySqlToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

Pour plus d’informations, consultez l’article [MySQL connector (connecteur MySQL)](data-factory-onprem-mysql-connector.md#copy-activity-properties).

## <a name="oracle"></a>Oracle

### <a name="linked-service"></a>Service lié
Pour définir un service lié Oracle, définissez le **type** du service lié sur **OnPremisesOracle** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| driverType | Spécifiez le pilote à utiliser pour copier les données à partir de ou vers la base de données Oracle. Valeurs autorisées : **Microsoft** ou **ODP** (par défaut). Pour plus d'informations sur le pilote, consultez la section Version prise en charge et installation. | Non |
| connectionString | Spécifier les informations requises pour la connexion à l’instance de base de données Oracle pour la propriété connectionString. | Oui |
| gatewayName | Nom de la passerelle utilisée pour se connecter au serveur Oracle local |Oui |

#### <a name="example"></a>Exemple
```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "driverType": "Microsoft",
            "connectionString": "Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [Oracle connector (connecteur Oracle)](data-factory-onprem-oracle-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données Oracle, définissez le **type** du jeu de données sur **OracleTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table dans la base de données Oracle à laquelle le service lié fait référence. |Non (si **oracleReaderQuery** de **OracleSource** est spécifié) |

#### <a name="example"></a>Exemple

```json
{
    "name": "OracleInput",
    "properties": {
        "type": "OracleTable",
        "linkedServiceName": "OnPremisesOracleLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "offset": "01:00:00",
            "interval": "1",
            "anchorDateTime": "2016-02-27T12:00:00",
            "frequency": "Hour"
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
Pour plus d’informations, consultez l’article [Oracle connector (connecteur Oracle)](data-factory-onprem-oracle-connector.md#dataset-properties).

### <a name="oracle-source-in-copy-activity"></a>Source Oracle dans l’activité de copie
Si vous copiez des données à partir d’une base de données Oracle, définissez le **type de source** de l’activité de copie sur **OracleSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| oracleReaderQuery |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Par exemple : `select * from MyTable` <br/><br/>Si non spécifié, l’instruction SQL exécutée : `select * from MyTable` |Non (si **tableName** de **dataset** est spécifiée) |

#### <a name="example"></a>Exemple

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "OracletoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": " OracleInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "OracleSource",
                    "oracleReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
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
        }]
    }
}
```

Pour plus d’informations, consultez l’article [Oracle connector (connecteur Oracle)](data-factory-onprem-oracle-connector.md#copy-activity-properties).

### <a name="oracle-sink-in-copy-activity"></a>Récepteur Oracle dans l’activité de copie
Si vous copiez des données dans une base de données Oracle, définissez le **type de récepteur** de l’activité de copie sur **OracleSink** et spécifiez les propriétés suivantes dans la section **sink** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| writeBatchTimeout |Temps d’attente pour que l’opération d’insertion de lot soit terminée avant d’expirer. |intervalle de temps<br/><br/> Exemple : “00:30:00” (30 minutes). |Non |
| writeBatchSize |Insère des données dans la table SQL lorsque la taille du tampon atteint writeBatchSize |Nombre entier (nombre de lignes) |Non (valeur par défaut : 100) |
| sqlWriterCleanupScript |Spécifiez une requête pour exécuter l’activité de copie afin que les données d’un segment spécifique soient nettoyées. |Une instruction de requête. |Non |
| sliceIdentifierColumnName |Spécifiez le nom de la colonne que l’activité de copie doit remplir avec l’identificateur de segment généré automatiquement, et qui est utilisée pour nettoyer les données d’un segment spécifique lors de la réexécution. |Nom d’une colonne avec le type de données binary(32). |Non |

#### <a name="example"></a>Exemple
```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-05T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoOracle",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "OracleOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "OracleSink"
                }
            },
            "scheduler": {
                "frequency": "Day",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```
Pour plus d’informations, consultez l’article [Oracle connector (connecteur Oracle)](data-factory-onprem-oracle-connector.md#copy-activity-properties).

## <a name="postgresql"></a>PostgreSQL

### <a name="linked-service"></a>Service lié
Pour définir un service lié PostgreSQL, définissez le **type** du service lié sur **OnPremisesPostgreSql** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| server |Nom du serveur PostgreSQL. |Oui |
| database |Nom de la base de données PostgreSQL. |Oui |
| schéma |Nom du schéma dans la base de données. Le nom du schéma respecte la casse. |Non |
| authenticationType |Type d'authentification utilisé pour se connecter à la base de données PostgreSQL. Les valeurs possibles sont les suivantes : Anonymous, Basic et Windows. |Oui |
| username |Spécifiez le nom d'utilisateur si vous utilisez l'authentification de base ou Windows. |Non |
| mot de passe |Spécifiez le mot de passe du compte d’utilisateur que vous avez spécifié pour le nom d’utilisateur. |Non |
| gatewayName |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter à la base de données PostgreSQL locale. |Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "OnPremPostgreSqlLinkedService",
    "properties": {
        "type": "OnPremisesPostgreSql",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "schema": "<schema>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```
Pour plus d’informations, consultez l’article [PostgreSQL connector (connecteur PostgreSQL)](data-factory-onprem-postgresql-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données PostgreSQL, définissez le **type** du jeu de données sur **RelationalTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table dans l'instance de base de données PostgreSQL à laquelle le service lié fait référence. Le nom de la table respecte la casse. |Non (si la **requête** de **RelationalSource** est spécifiée) |

#### <a name="example"></a>Exemple
```json
{
    "name": "PostgreSqlDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremPostgreSqlLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
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
Pour plus d’informations, consultez l’article [PostgreSQL connector (connecteur PostgreSQL)](data-factory-onprem-postgresql-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Source relationnelle dans l’activité de copie
Si vous copiez des données à partir d’une base de données PostgreSQL, définissez le **type de source** de l’activité de copie sur **RelationalSource** et spécifiez les propriétés suivantes dans la section **source** :


| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Par exemple : "query": "select * from \"MySchema\".\"MyTable\"". |Non (si **tableName** de **dataset** est spécifiée) |

#### <a name="example"></a>Exemple

```json
{
    "name": "CopyPostgreSqlToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "select * from \"public\".\"usstates\""
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "inputs": [{
                "name": "PostgreSqlDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobPostgreSqlDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "PostgreSqlToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

Pour plus d’informations, consultez l’article [PostgreSQL connector (connecteur PostgreSQL)](data-factory-onprem-postgresql-connector.md#copy-activity-properties).

## <a name="sap-business-warehouse"></a>SAP Business Warehouse


### <a name="linked-service"></a>Service lié
Pour définir un service lié SAP Business Warehouse (BW), définissez le **type** du service lié sur **SapBw** et spécifiez les propriétés suivantes dans la section **typeProperties** :

Propriété | Description | Valeurs autorisées | Obligatoire
-------- | ----------- | -------------- | --------
server | Nom du serveur sur lequel réside l’instance SAP BW. | string | Oui
systemNumber | Numéro de système du système SAP BW. | Nombre décimal à deux chiffres représenté sous forme de chaîne. | Oui
clientId | ID client du client dans le système SAP W. | Nombre décimal à trois chiffres représenté sous forme de chaîne. | Oui
username | Nom de l’utilisateur qui a accès au serveur SAP | string | Oui
mot de passe | Mot de passe pour l’utilisateur. | string | Oui
gatewayName | Nom de la passerelle que le service Data Factory doit utiliser pour se connecter à l’instance SAP BW locale. | string | Oui
encryptedCredential | La chaîne d’informations d’identification chiffrée. | string | Non

#### <a name="example"></a>Exemple

```json
{
    "name": "SapBwLinkedService",
    "properties": {
        "type": "SapBw",
        "typeProperties": {
            "server": "<server name>",
            "systemNumber": "<system number>",
            "clientId": "<client id>",
            "username": "<SAP user>",
            "password": "<Password for SAP user>",
            "gatewayName": "<gateway name>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [SAP Business Warehouse connector (connecteur SAP Business Warehouse)](data-factory-sap-business-warehouse-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données SAP BW, définissez le **type** du jeu de données sur **RelationalTable**. Aucune propriété propre à un type n’est prise en charge pour le type de jeu de données SAP BW **RelationalTable**.

#### <a name="example"></a>Exemple

```json
{
    "name": "SapBwDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "SapBwLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
Pour plus d’informations, consultez l’article [SAP Business Warehouse connector (connecteur SAP Business Warehouse)](data-factory-sap-business-warehouse-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Source relationnelle dans l’activité de copie
Si vous copiez des données à partir de SAP Business Warehouse, définissez le **type de source** de l’activité de copie sur **RelationalSource** et spécifiez les propriétés suivantes dans la section **source** :


| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query | Spécifie la requête MDX pour lire les données de l’instance SAP BW. | Requête MDX. | Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "CopySapBwToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "<MDX query for SAP BW>"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "SapBwDataset"
            }],
            "outputs": [{
                "name": "AzureBlobDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "SapBwToBlob"
        }],
        "start": "2017-03-01T18:00:00",
        "end": "2017-03-01T19:00:00"
    }
}
```

Pour plus d’informations, consultez l’article [SAP Business Warehouse connector (connecteur SAP Business Warehouse)](data-factory-sap-business-warehouse-connector.md#copy-activity-properties).

## <a name="sap-hana"></a>SAP HANA

### <a name="linked-service"></a>Service lié
Pour définir un service lié SAP HANA, définissez le **type** du service lié sur **SapHana** et spécifiez les propriétés suivantes dans la section **typeProperties** :

Propriété | Description | Valeurs autorisées | Obligatoire
-------- | ----------- | -------------- | --------
server | Le nom du serveur sur lequel réside l’instance SAP HANA. Si votre serveur utilise un port personnalisé, spécifiez `server:port`. | string | Oui
authenticationType | Type d'authentification. | chaîne. « Basic » ou « Windows » | Oui
username | Nom de l’utilisateur qui a accès au serveur SAP | string | Oui
mot de passe | Mot de passe pour l’utilisateur. | string | Oui
gatewayName | Nom de la passerelle que le service Data Factory doit utiliser pour se connecter à l’instance SAP HANA locale. | string | Oui
encryptedCredential | La chaîne d’informations d’identification chiffrée. | string | Non

#### <a name="example"></a>Exemple

```json
{
    "name": "SapHanaLinkedService",
    "properties": {
        "type": "SapHana",
        "typeProperties": {
            "server": "<server name>",
            "authenticationType": "<Basic, or Windows>",
            "username": "<SAP user>",
            "password": "<Password for SAP user>",
            "gatewayName": "<gateway name>"
        }
    }
}

```
Pour plus d’informations, consultez l’article [SAP HANA connector (connecteur SAP HANA)](data-factory-sap-hana-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données SAP HANA, définissez le **type** du jeu de données sur **RelationalTable**. Aucune propriété propre à un type n’est prise en charge pour le type de jeu de données SAP HANA **RelationalTable**.

#### <a name="example"></a>Exemple

```json
{
    "name": "SapHanaDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "SapHanaLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
Pour plus d’informations, consultez l’article [SAP HANA connector (connecteur SAP HANA)](data-factory-sap-hana-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Source relationnelle dans l’activité de copie
Si vous copiez des données à partir d’un magasin de données SAP HANA, définissez le **type de source** de l’activité de copie sur **RelationalSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query | Spécifie la requête SQL pour lire les données de l’instance SAP HANA. | Requête SQL. | Oui |


#### <a name="example"></a>Exemple


```json
{
    "name": "CopySapHanaToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "<SQL Query for HANA>"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "SapHanaDataset"
            }],
            "outputs": [{
                "name": "AzureBlobDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "SapHanaToBlob"
        }],
        "start": "2017-03-01T18:00:00",
        "end": "2017-03-01T19:00:00"
    }
}
```

Pour plus d’informations, consultez l’article [SAP HANA connector (connecteur SAP HANA)](data-factory-sap-hana-connector.md#copy-activity-properties).


## <a name="sql-server"></a>SQL Server

### <a name="linked-service"></a>Service lié
Vous créez un service lié de type **OnPremisesSqlServer** pour lier une base de données SQL Server à une fabrique de données. Le tableau suivant fournit la description des éléments JSON spécifiques au service lié SQL Server.

Le tableau suivant fournit la description des éléments JSON spécifiques au service lié SQL Server.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |La propriété de type doit être définie sur : **OnPremisesSqlServer**. |Oui |
| connectionString |Spécifiez les informations connectionString nécessaires pour établir une connexion à la base de données SQL Server à l’aide de l’authentification SQL ou de l’authentification Windows. |Oui |
| gatewayName |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter à la base de données SQL Server. |Oui |
| username |Spécifiez le nom d’utilisateur si vous utilisez l’authentification Windows. Exemple : **domainname\\username**. |Non |
| mot de passe |Spécifiez le mot de passe du compte d’utilisateur que vous avez spécifié pour le nom d’utilisateur. |Non |

Vous pouvez chiffrer les informations d’identification à l’aide de la cmdlet **New-AzDataFactoryEncryptValue**, puis les utiliser dans la chaîne de connexion comme indiqué dans l’exemple suivant (propriété **EncryptedCredential**) :

```json
"connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=True;EncryptedCredential=<encrypted credential>",
```


#### <a name="example-json-for-using-sql-authentication"></a>Exemple : JSON pour utilisation de l’authentification SQL

```json
{
    "name": "MyOnPremisesSQLDB",
    "properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "connectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=False;User ID=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```
#### <a name="example-json-for-using-windows-authentication"></a>Exemple : JSON pour utilisation de l’authentification Windows

Si le nom d’utilisateur et le mot de passe sont spécifiés, la passerelle les utilise pour emprunter l’identité du compte utilisateur spécifié pour la connexion à la base de données SQL Server. Dans le cas contraire, la passerelle se connecte directement à SQL Server avec le contexte de sécurité de la passerelle (son compte de démarrage).

```json
{
    "Name": " MyOnPremisesSQLDB",
    "Properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "ConnectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=True;",
            "username": "<domain\\username>",
            "password": "<password>",
            "gatewayName": "<gateway name>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [SQL Server connector (connecteur SQL Server)](data-factory-sqlserver-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données SQL Server, définissez le **type** du jeu de données sur **SqlServerTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table ou de la vue dans l’instance de base de données SQL Server à laquelle le service lié fait référence. |Oui |

#### <a name="example"></a>Exemple
```json
{
    "name": "SqlServerInput",
    "properties": {
        "type": "SqlServerTable",
        "linkedServiceName": "SqlServerLinkedService",
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

Pour plus d’informations, consultez l’article [SQL Server connector (connecteur SQL Server)](data-factory-sqlserver-connector.md#dataset-properties).

### <a name="sql-source-in-copy-activity"></a>Source SQL dans l’activité de copie
Si vous copiez des données à partir d’une base de données SQL Server, définissez le **type de source** de l’activité de copie sur **SqlSource** et spécifiez les propriétés suivantes dans la section **source** :


| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| sqlReaderQuery |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Par exemple : `select * from MyTable`. Peut faire référence à plusieurs tables de la base de données référencée par le jeu de données d’entrée. S’il n’est pas spécifié, l’instruction SQL est exécutée : select from MyTable. |Non |
| sqlReaderStoredProcedureName |Nom de la procédure stockée qui lit les données de la table source. |Nom de la procédure stockée. |Non |
| storedProcedureParameters |Paramètres de la procédure stockée. |Paires nom/valeur. Les noms et la casse des paramètres doivent correspondre aux noms et à la casse des paramètres de la procédure stockée. |Non |

Si **sqlReaderQuery** est spécifié pour la SqlSource, l'activité de copie exécute cette requête dans la source Azure SQL Database pour obtenir les données.

Vous pouvez également spécifier une procédure stockée en indiquant **sqlReaderStoredProcedureName** et **storedProcedureParameters** (si la procédure stockée accepte des paramètres).

Si vous ne spécifiez pas sqlReaderQuery ou sqlReaderStoredProcedureName, les colonnes définies dans la section structure du code JSON du jeu de données sont utilisées pour créer une requête à exécuter sur Azure SQL Database. Si la définition du jeu de données ne possède pas de structure, toutes les colonnes de la table sont sélectionnées.

> [!NOTE]
> Quand vous utilisez **sqlReaderStoredProcedureName**, vous devez toujours spécifier une valeur pour la propriété **tableName** du code JSON du jeu de données. Cependant, il n’existe aucune validation effectuée pour cette table.


#### <a name="example"></a>Exemple
```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "SqlServertoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": " SqlServerInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
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
        }]
    }
}
```

Dans cet exemple, **sqlReaderQuery** est spécifié pour SqlSource. L'activité de copie exécute cette requête dans la source de base de données SQL server pour obtenir les données. Vous pouvez également spécifier une procédure stockée en indiquant **sqlReaderStoredProcedureName** et **storedProcedureParameters** (si la procédure stockée accepte des paramètres). sqlReaderQuery peut faire référence à plusieurs tables de la base de données référencée par le jeu de données d’entrée. Cela n’est pas limité à la seule table définie en tant que typeProperty tableName du jeu de données.

Si vous ne spécifiez pas sqlReaderQuery ou sqlReaderStoredProcedureName, les colonnes définies dans la section Structure sont utilisées pour sélectionner une requête à exécuter dans l'Azure SQL Database. Si la définition du jeu de données ne possède pas de structure, toutes les colonnes de la table sont sélectionnées.

Pour plus d’informations, consultez l’article [SQL Server connector (connecteur SQL Server)](data-factory-sqlserver-connector.md#copy-activity-properties).

### <a name="sql-sink-in-copy-activity"></a>Récepteur SQL dans l’activité de copie
Si vous copiez des données dans une base de données SQL Server, définissez le **type de récepteur** de l’activité de copie sur **SqlSink** et spécifiez les propriétés suivantes dans la section **sink** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| writeBatchTimeout |Temps d’attente pour que l’opération d’insertion de lot soit terminée avant d’expirer. |intervalle de temps<br/><br/> Exemple : “00:30:00” (30 minutes). |Non |
| writeBatchSize |Insère des données dans la table SQL lorsque la taille du tampon atteint writeBatchSize |Nombre entier (nombre de lignes) |Non (valeur par défaut : 10000) |
| sqlWriterCleanupScript |Spécifiez une requête pour exécuter l'activité de copie de sorte que les données d'un segment spécifique sont nettoyées. Pour plus d'informations, consultez la section consacrée à la répétabilité. |Une instruction de requête. |Non |
| sliceIdentifierColumnName |Spécifiez le nom de la colonne que l’activité de copie doit remplir avec l’identificateur de segment généré automatiquement, et qui est utilisée pour nettoyer les données d’un segment spécifique lors de la réexécution. Pour plus d'informations, consultez la section consacrée à la répétabilité. |Nom d’une colonne avec le type de données binary(32). |Non |
| sqlWriterStoredProcedureName |Nom de la procédure stockée qui met à jour/insère les données dans la table cible. |Nom de la procédure stockée. |Non |
| storedProcedureParameters |Paramètres de la procédure stockée. |Paires nom/valeur. Les noms et la casse des paramètres doivent correspondre aux noms et à la casse des paramètres de la procédure stockée. |Non |
| sqlWriterTableType |Spécifiez le nom du type de table à utiliser dans la procédure stockée. L’activité de copie place les données déplacées disponibles dans une table temporaire avec ce type de table. Le code de procédure stockée peut ensuite fusionner les données copiées avec les données existantes. |Nom de type de table. |Non |

#### <a name="example"></a>Exemple
Le pipeline contient une activité de copie qui est configurée pour utiliser ces jeux de données d'entrée et de sortie, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **BlobSource** et le type **sink** est défini sur **SqlSink**.

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoSQL",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": " SqlServerOutput "
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource",
                    "blobColumnSeparators": ","
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
        }]
    }
}
```

Pour plus d’informations, consultez l’article [SQL Server connector (connecteur SQL Server)](data-factory-sqlserver-connector.md#copy-activity-properties).

## <a name="sybase"></a>Sybase

### <a name="linked-service"></a>Service lié
Pour définir un service lié Sybase, définissez le **type** du service lié sur **OnPremisesSybase** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| server |Nom du serveur Sybase. |Oui |
| database |Nom de la base de données Sybase. |Oui |
| schéma |Nom du schéma dans la base de données. |Non |
| authenticationType |Type d'authentification utilisé pour se connecter à la base de données Sybase. Les valeurs possibles sont les suivantes : Anonymous, Basic et Windows. |Oui |
| username |Spécifiez le nom d'utilisateur si vous utilisez l'authentification de base ou Windows. |Non |
| mot de passe |Spécifiez le mot de passe du compte d’utilisateur que vous avez spécifié pour le nom d’utilisateur. |Non |
| gatewayName |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter à la base de données Sybase locale. |Oui |

#### <a name="example"></a>Exemple
```json
{
    "name": "OnPremSybaseLinkedService",
    "properties": {
        "type": "OnPremisesSybase",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "schema": "<schema>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [Sybase connector (connecteur Sybase)](data-factory-onprem-sybase-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données Sybase, définissez le **type** du jeu de données sur **RelationalTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table dans l'instance de base de données Sybase à laquelle le service lié fait référence. |Non (si la **requête** de **RelationalSource** est spécifiée) |

#### <a name="example"></a>Exemple

```json
{
    "name": "SybaseDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremSybaseLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
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

Pour plus d’informations, consultez l’article [Sybase connector (connecteur Sybase)](data-factory-onprem-sybase-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Source relationnelle dans l’activité de copie
Si vous copiez des données à partir d’une base de données Sybase, définissez le **type de source** de l’activité de copie sur **RelationalSource** et spécifiez les propriétés suivantes dans la section **source** :


| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Par exemple : `select * from MyTable`. |Non (si **tableName** de **dataset** est spécifiée) |

#### <a name="example"></a>Exemple

```json
{
    "name": "CopySybaseToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "select * from DBA.Orders"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "inputs": [{
                "name": "SybaseDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobSybaseDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "SybaseToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

Pour plus d’informations, consultez l’article [Sybase connector (connecteur Sybase)](data-factory-onprem-sybase-connector.md#copy-activity-properties).

## <a name="teradata"></a>Teradata

### <a name="linked-service"></a>Service lié
Pour définir un service lié Teradata, définissez le **type** du service lié sur **OnPremisesTeradata** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| server |Nom du serveur Teradata. |Oui |
| authenticationType |Type d'authentification utilisé pour se connecter à la base de données Teradata. Les valeurs possibles sont les suivantes : Anonymous, Basic et Windows. |Oui |
| username |Spécifiez le nom d'utilisateur si vous utilisez l'authentification de base ou Windows. |Non |
| mot de passe |Spécifiez le mot de passe du compte d’utilisateur que vous avez spécifié pour le nom d’utilisateur. |Non |
| gatewayName |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter à la base de données Teradata locale. |Oui |

#### <a name="example"></a>Exemple
```json
{
    "name": "OnPremTeradataLinkedService",
    "properties": {
        "type": "OnPremisesTeradata",
        "typeProperties": {
            "server": "<server>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [Teradata connector (connecteur Teradata)](data-factory-onprem-teradata-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données d’objet blob Teradata, définissez le **type** du jeu de données sur **RelationalTable**. Il n’existe actuellement aucune propriété type prise en charge pour le jeu de données Teradata.

#### <a name="example"></a>Exemple
```json
{
    "name": "TeradataDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremTeradataLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
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

Pour plus d’informations, consultez l’article [Teradata connector (connecteur Teradata)](data-factory-onprem-teradata-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Source relationnelle dans l’activité de copie
Si vous copiez des données à partir d’une base de données Teradata, définissez le **type de source** de l’activité de copie sur **RelationalSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Par exemple : `select * from MyTable`. |Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "CopyTeradataToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', SliceStart, SliceEnd)"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "TeradataDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobTeradataDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "TeradataToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "isPaused": false
    }
}
```

Pour plus d’informations, consultez l’article [Teradata connector (connecteur Teradata)](data-factory-onprem-teradata-connector.md#copy-activity-properties).

## <a name="cassandra"></a>Cassandra


### <a name="linked-service"></a>Service lié
Pour définir un service lié Cassandra, définissez le **type** du service lié sur **OnPremisesCassandra** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| host |Une ou plusieurs adresses IP ou noms d’hôte de serveurs Cassandra.<br/><br/>Renseignez une liste des adresses IP ou des noms d’hôte séparée par des virgules pour vous connecter simultanément à tous les serveurs. |Oui |
| port |Le port TCP utilisé par le serveur Cassandra pour écouter les connexions clientes. |Aucune valeur par défaut : 9042 |
| authenticationType |Basique ou anonyme |Oui |
| username |Spécifiez le nom d’utilisateur du compte d’utilisateur. |Oui, si authenticationType est défini sur De base. |
| mot de passe |Spécifiez le mot de passe du compte d'utilisateur. |Oui, si authenticationType est défini sur De base. |
| gatewayName |Le nom de la passerelle qui est utilisée pour se connecter à la base de données Cassandra locale. |Oui |
| encryptedCredential |Informations d’identification chiffrées par la passerelle. |Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "CassandraLinkedService",
    "properties": {
        "type": "OnPremisesCassandra",
        "typeProperties": {
            "authenticationType": "Basic",
            "host": "<cassandra server name or IP address>",
            "port": 9042,
            "username": "user",
            "password": "password",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [Cassandra connector (connecteur Cassandra)](data-factory-onprem-cassandra-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données Cassandra, définissez le **type** du jeu de données sur **CassandraTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| espace de clé |Nom de l’espace de clé ou du schéma dans la base de données Cassandra. |Oui (si la **requête** pour **CassandraSource** n’est pas définie). |
| tableName |Nom de la table dans la base de données Cassandra. |Oui (si la **requête** pour **CassandraSource** n’est pas définie). |

#### <a name="example"></a>Exemple

```json
{
    "name": "CassandraInput",
    "properties": {
        "linkedServiceName": "CassandraLinkedService",
        "type": "CassandraTable",
        "typeProperties": {
            "tableName": "mytable",
            "keySpace": "<key space>"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
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

Pour plus d’informations, consultez l’article [Cassandra connector (connecteur Cassandra)](data-factory-onprem-cassandra-connector.md#dataset-properties).

### <a name="cassandra-source-in-copy-activity"></a>Source Cassandra dans l’activité de copie
Si vous copiez des données depuis un système Cassandra, définissez le **type de source** de l’activité de copie sur **CassandraSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Requête SQL-92 ou requête CQL. Reportez-vous à [référence CQL](https://docs.datastax.com/en/cql/3.1/cql/cql_reference/cqlReferenceTOC.html). <br/><br/>Lorsque vous utilisez la requête SQL, indiquez **keyspace name.table name** pour représenter la table que vous souhaitez interroger. |Non (si tableName et keyspace sur le jeu de données sont définis). |
| Niveau de cohérence |Le niveau de cohérence spécifie le nombre de réplicas devant répondre à une demande de lecture avant de renvoyer des données à l’application cliente. Cassandra vérifie le nombre de réplicas spécifié pour permettre aux données de répondre à la demande de lecture. |UN, DEUX, TROIS, QUORUM, TOUT, LOCAL_QUORUM, EACH_QUORUM, LOCAL_ONE. Reportez-vous à [Configuring data consistency (Configuration de la cohérence des données)](https://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_config_consistency_c.html) pour plus d’informations. |Non. La valeur par défaut est UN. |

#### <a name="example"></a>Exemple

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "CassandraToAzureBlob",
            "description": "Copy from Cassandra to an Azure blob",
            "type": "Copy",
            "inputs": [{
                "name": "CassandraInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "CassandraSource",
                    "query": "select id, firstname, lastname from mykeyspace.mytable"
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
        }]
    }
}
```

Pour plus d’informations, consultez l’article [Cassandra connector (connecteur Cassandra)](data-factory-onprem-cassandra-connector.md#copy-activity-properties).

## <a name="mongodb"></a>MongoDB

### <a name="linked-service"></a>Service lié
Pour définir un service lié MongoDB, définissez le **type** du service lié sur **OnPremisesMongoDB** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| server |Nom d’hôte ou adresse IP du serveur MongoDB. |Oui |
| port |Le port TCP utilisé par le serveur MongoDB pour écouter les connexions clientes. |Facultatif, valeur par défaut : 27017 |
| authenticationType |De base ou anonyme. |Oui |
| username |Compte d’utilisateur pour accéder à MongoDB. |Oui (si l’authentification de base est utilisée). |
| mot de passe |Mot de passe pour l’utilisateur. |Oui (si l’authentification de base est utilisée). |
| authSource |Nom de la base de données MongoDB que vous souhaitez utiliser pour vérifier vos informations d’identification pour l’authentification. |Facultatif (si l’authentification de base est utilisée). Par défaut : utilise le compte d’administrateur et la base de données spécifiée à l’aide de la propriété databaseName. |
| databaseName |Nom de la base de données MongoDB à laquelle vous souhaitez accéder. |Oui |
| gatewayName |Nom de la passerelle qui accède au magasin de données. |Oui |
| encryptedCredential |Informations d’identification chiffrées par la passerelle. |Facultatif |

#### <a name="example"></a>Exemple

```json
{
    "name": "OnPremisesMongoDbLinkedService",
    "properties": {
        "type": "OnPremisesMongoDb",
        "typeProperties": {
            "authenticationType": "<Basic or Anonymous>",
            "server": "< The IP address or host name of the MongoDB server >",
            "port": "<The number of the TCP port that the MongoDB server uses to listen for client connections.>",
            "username": "<username>",
            "password": "<password>",
            "authSource": "< The database that you want to use to check your credentials for authentication. >",
            "databaseName": "<database name>",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [MongoDB connector (connecteur MongoDB)](data-factory-on-premises-mongodb-connector.md#linked-service-properties)

### <a name="dataset"></a>Dataset
Pour définir un jeu de données MongoDB, définissez le **type** du jeu de données sur **MongoDbCollection** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| collectionName |Nom de la collection dans la base de données MongoDB. |Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "MongoDbInputDataset",
    "properties": {
        "type": "MongoDbCollection",
        "linkedServiceName": "OnPremisesMongoDbLinkedService",
        "typeProperties": {
            "collectionName": "<Collection name>"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

Pour plus d’informations, consultez l’article [MongoDB connector (connecteur MongoDB)](data-factory-on-premises-mongodb-connector.md#dataset-properties)

#### <a name="mongodb-source-in-copy-activity"></a>Source MongoDB dans l’activité de copie
Si vous copiez des données à partir de MongoDB, définissez le **type de source** de l’activité de copie sur **MongoDbSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL-92. Par exemple : `select * from MyTable`. |Non (si **collectionName** du **jeu de données** est spécifié) |

#### <a name="example"></a>Exemple

```json
{
    "name": "CopyMongoDBToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "MongoDbSource",
                    "query": "select * from MyTable"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "MongoDbInputDataset"
            }],
            "outputs": [{
                "name": "AzureBlobOutputDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "MongoDBToAzureBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

Pour plus d’informations, consultez l’article [MongoDB connector (connecteur MongoDB)](data-factory-on-premises-mongodb-connector.md#copy-activity-properties)

## <a name="amazon-s3"></a>Amazon S3


### <a name="linked-service"></a>Service lié
Pour définir un service lié Amazon S3, définissez le **type** du service lié sur **AwsAccessKey** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| accessKeyID |ID de la clé d’accès secrète. |string |Oui |
| secretAccessKey |La clé d’accès secrète elle-même. |Chaîne secrète chiffrée |Oui |

#### <a name="example"></a>Exemple
```json
{
    "name": "AmazonS3LinkedService",
    "properties": {
        "type": "AwsAccessKey",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": "<secret access key>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [Amazon S3 connector (connecteur Amazon S3)](data-factory-amazon-simple-storage-service-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données Amazon S3, définissez le **type** du jeu de données sur **AmazonS3** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| bucketName |Le nom de compartiment S3. |String |Oui |
| key |La clé d’objet S3. |String |Non |
| prefix |Préfixe de la clé d’objet S3. Les objets dont les clés commencent par ce préfixe sont sélectionnés. S’applique uniquement lorsque la clé est vide. |String |Non |
| version |La version de l’objet S3 si le contrôle de version S3 est activé. |String |Non |
| format | Les types de formats suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Définissez la propriété **type** située sous Format sur l’une de ces valeurs. Pour en savoir plus, consultez les sections relatives à [format Text](data-factory-supported-file-and-compression-formats.md#text-format), [format Json](data-factory-supported-file-and-compression-formats.md#json-format), [format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si vous souhaitez **copier des fichiers en l’état** entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie. |Non | |
| compression | Spécifiez le type et le niveau de compression pour les données. Types pris en charge : **GZip**, **Deflate**, **BZip2** et **ZipDeflate**. Les niveaux pris en charge sont : **Optimal** et **Fastest**. Pour plus d’informations, consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |Non | |


> [!NOTE]
> bucketName + key spécifient l’emplacement de l’objet S3 où le compartiment est le conteneur racine pour les objets S3 et la clé est le chemin d’accès complet à l’objet S3.

#### <a name="example-sample-dataset-with-prefix"></a>Exemple : Exemple de jeu de données avec le préfixe

```json
{
    "name": "dataset-s3",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": "link- testS3",
        "typeProperties": {
            "prefix": "testFolder/test",
            "bucketName": "<S3 bucket name>",
            "format": {
                "type": "OrcFormat"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
#### <a name="example-sample-data-set-with-version"></a>Exemple : Exemple de jeu de données (avec la version)

```json
{
    "name": "dataset-s3",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": "link- testS3",
        "typeProperties": {
            "key": "testFolder/test.orc",
            "bucketName": "<S3 bucket name>",
            "version": "XXXXXXXXXczm0CJajYkHf0_k6LhBmkcL",
            "format": {
                "type": "OrcFormat"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

#### <a name="example-dynamic-paths-for-s3"></a>Exemple : Chemins d’accès dynamiques pour S3
Dans l’exemple, nous avons utilisé des valeurs fixes pour les propriétés key et bucketName dans le jeu de données Amazon S3.

```json
"key": "testFolder/test.orc",
"bucketName": "<S3 bucket name>",
```

Vous pouvez demander à Data Factory de calculer les propriétés key et bucketName dynamiquement au moment de l’exécution à l’aide de variables système telles que SliceStart.

```json
"key": "$$Text.Format('{0:MM}/{0:dd}/test.orc', SliceStart)"
"bucketName": "$$Text.Format('{0:yyyy}', SliceStart)"
```

Vous pouvez faire de même pour la propriété « prefix » d’un jeu de données Amazon S3. Pour obtenir la liste des fonctions et variables prises en charge, consultez [Variables système et fonctions Data Factory](data-factory-functions-variables.md) .

Pour plus d’informations, consultez l’article [Amazon S3 connector (connecteur Amazon S3)](data-factory-amazon-simple-storage-service-connector.md#dataset-properties).

### <a name="file-system-source-in-copy-activity"></a>Source Système de fichiers dans l’activité de copie
Si vous copiez des données à partir d’Amazon S3, définissez le **type de source** de l’activité de copie sur **FileSystemSource** et spécifiez les propriétés suivantes dans la section **source** :


| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| recursive |Spécifie s’il faut répertorier de manière récursive les objets S3 sous le répertoire. |true/false |Non |


#### <a name="example"></a>Exemple


```json
{
    "name": "CopyAmazonS3ToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource",
                    "recursive": true
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "AmazonS3InputDataset"
            }],
            "outputs": [{
                "name": "AzureBlobOutputDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "AmazonS3ToBlob"
        }],
        "start": "2016-08-08T18:00:00",
        "end": "2016-08-08T19:00:00"
    }
}
```

Pour plus d’informations, consultez l’article [Amazon S3 connector (connecteur Amazon S3)](data-factory-amazon-simple-storage-service-connector.md#copy-activity-properties).

## <a name="file-system"></a>Système de fichiers


### <a name="linked-service"></a>Service lié
Vous pouvez lier un système de fichiers local à une fabrique de données Azure grâce au service lié **Serveur de fichiers local**. Le tableau suivant décrit les éléments JSON spécifiques au service lié Serveur de fichiers local.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |Vérifiez que la propriété type est définie sur **OnPremisesFileServer**. |Oui |
| host |Spécifie le chemin d’accès racine du dossier que vous souhaitez copier. Utilisez le caractère d’échappement « \ » pour les caractères spéciaux contenus dans la chaîne. Pour obtenir des exemples, consultez la section Exemples de définitions de jeux de données et de services liés. |Oui |
| userid |Spécifiez l’ID de l’utilisateur qui a accès au serveur. |Non (si vous choisissez encryptedcredential) |
| mot de passe |Spécifiez le mot de passe de l’utilisateur (userid). |Non (si vous choisissez encryptedcredential) |
| encryptedCredential |Spécifiez les informations d’identification chiffrées que vous pouvez obtenir en exécutant la cmdlet New-AzDataFactoryEncryptValue. |Non (si vous choisissez de spécifier un nom d'utilisateur et un mot de passe en texte brut) |
| gatewayName |Spécifie le nom de la passerelle que Data Factory doit utiliser pour se connecter au serveur de fichiers local. |Oui |

#### <a name="sample-folder-path-definitions"></a>Exemples de définitions de chemin d’accès du dossier

| Scénario | Hôte dans la définition du service lié | folderPath dans la définition du jeu de données |
| --- | --- | --- |
| Dossier local sur l’ordinateur passerelle de gestion des données : <br/><br/>Exemples : D:\\\* ou D:\dossier\sous-dossier\\\* |D:\\\\ (pour la passerelle de gestion des données 2.0 et versions ultérieures) <br/><br/> hôte local (pour les versions de la passerelle de gestion des données antérieures à 2.0) |.\\\\ ou dossier\\\\sous-dossier (pour la passerelle de gestion des données 2.0 et versions ultérieures) <br/><br/>D:\\\\ ou D:\\\\dossier\\\\sous-dossier (pour les versions de la passerelle antérieures à 2.0) |
| Dossier partagé distant : <br/><br/>Exemples : \\\\myserver\\share\\\* ou \\\\myserver\\share\\dossier\\sous-dossier\\\* |\\\\\\\\myserver\\\\share |.\\\\ ou dossier\\\\sous-dossier |


#### <a name="example-using-username-and-password-in-plain-text"></a>Exemple : utilisation d’un nom d’utilisateur et d’un mot de passe en texte brut

```json
{
    "Name": "OnPremisesFileServerLinkedService",
    "properties": {
        "type": "OnPremisesFileServer",
        "typeProperties": {
            "host": "\\\\Contosogame-Asia",
            "userid": "Admin",
            "password": "123456",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-using-encryptedcredential"></a>Exemple : Utilisation d’encryptedcredential

```json
{
    "Name": " OnPremisesFileServerLinkedService ",
    "properties": {
        "type": "OnPremisesFileServer",
        "typeProperties": {
            "host": "D:\\",
            "encryptedCredential": "WFuIGlzIGRpc3Rpbmd1aXNoZWQsIG5vdCBvbmx5IGJ5xxxxxxxxxxxxxxxxx",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [File System connector (connecteur Système de fichiers)](data-factory-onprem-file-system-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données de système de fichiers, définissez le **type** du jeu de données sur **FileShare** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| folderPath |Spécifie le sous-chemin vers le dossier. Utilisez le caractère d’échappement « \ » pour les caractères spéciaux contenus dans la chaîne. Pour obtenir des exemples, consultez la section Exemples de définitions de jeux de données et de services liés.<br/><br/>Vous pouvez également effectuer une combinaison avec la propriété **partitionBy** pour que les chemins d’accès de dossier soient basés sur les dates et heures de démarrage et d’arrêt de la tranche. |Oui |
| fileName |Spécifiez le nom du fichier dans l’élément **folderPath** si vous souhaitez que la table se réfère à un fichier spécifique du dossier. Si vous ne spécifiez aucune valeur pour cette propriété, le tableau pointe vers tous les fichiers du dossier.<br/><br/>Lorsque fileName n’est pas spécifié pour un jeu de données de sortie, le nom du fichier généré est au format suivant : <br/><br/>`Data.<Guid>.txt` (Exemple : Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |Non |
| fileFilter |Spécifiez un filtre à utiliser pour sélectionner un sous-ensemble de fichiers dans le folderPath plutôt que tous les fichiers. <br/><br/>Les valeurs autorisées sont : `*` (plusieurs caractères) et `?` (caractère unique).<br/><br/>Exemple 1 : « fileFilter » : « *.log »<br/>Exemple 2 : « fileFilter » : 2016-1-?.txt »<br/><br/>Remarque : fileFilter s’applique à un jeu de données FileShare d’entrée. |Non |
| partitionedBy |partitionedBy peut être utilisé pour spécifier un folderPath/fileName dynamique pour les données de série chronologique. Par exemple, folderPath peut être paramétré pour toutes les heures de données. |Non |
| format | Les types de formats suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Définissez la propriété **type** située sous Format sur l’une de ces valeurs. Pour en savoir plus, consultez les sections relatives à [format Text](data-factory-supported-file-and-compression-formats.md#text-format), [format Json](data-factory-supported-file-and-compression-formats.md#json-format), [format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si vous souhaitez **copier des fichiers en l’état** entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie. |Non |
| compression | Spécifiez le type et le niveau de compression pour les données. Types pris en charge : **GZip**, **Deflate**, **BZip2** et **ZipDeflate** et les niveaux pris en charge sont : **Optimal** et **Fastest**. consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |Non |

> [!NOTE]
> Vous ne pouvez pas utiliser fileName et fileFilter simultanément.

#### <a name="example"></a>Exemple

```json
{
    "name": "OnpremisesFileSystemInput",
    "properties": {
        "type": " FileShare",
        "linkedServiceName": " OnPremisesFileServerLinkedService ",
        "typeProperties": {
            "folderPath": "mysharedfolder/yearno={Year}/monthno={Month}/dayno={Day}",
            "fileName": "{Hour}.csv",
            "partitionedBy": [{
                "name": "Year",
                "value": {
                    "type": "DateTime",
                    "date": "SliceStart",
                        "format": "yyyy"
                }
            }, {
                "name": "Month",
                "value": {
                    "type": "DateTime",
                    "date": "SliceStart",
                    "format": "MM"
                }
            }, {
                "name": "Day",
                "value": {
                    "type": "DateTime",
                    "date": "SliceStart",
                    "format": "dd"
                }
            }, {
                "name": "Hour",
                "value": {
                    "type": "DateTime",
                    "date": "SliceStart",
                    "format": "HH"
                }
            }]
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

Pour plus d’informations, consultez l’article [File System connector (connecteur Système de fichiers)](data-factory-onprem-file-system-connector.md#dataset-properties).

### <a name="file-system-source-in-copy-activity"></a>Source Système de fichiers dans l’activité de copie
Si vous copiez des données à partir d’un système de fichiers, définissez le **type de source** de l’activité de copie sur **FileSystemSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| recursive |Indique si les données sont lues de manière récursive à partir des sous-dossiers ou uniquement du dossier spécifié. |True, False (par défaut) |Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2015-06-01T18:00:00",
        "end": "2015-06-01T19:00:00",
        "description": "Pipeline for copy activity",
        "activities": [{
            "name": "OnpremisesFileSystemtoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "OnpremisesFileSystemInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
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
        }]
    }
}
```
Pour plus d’informations, consultez l’article [File System connector (connecteur Système de fichiers)](data-factory-onprem-file-system-connector.md#copy-activity-properties).

### <a name="file-system-sink-in-copy-activity"></a>Récepteur Système de fichiers dans l’activité de copie
Si vous copiez des données dans un système de fichiers, définissez le **type de récepteur** de l’activité de copie sur **FileSystemSink** et spécifiez les propriétés suivantes dans la section **sink** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| copyBehavior |Cette propriété définit le comportement de copie lorsque la source est BlobSource ou FileSystem. |**PreserveHierarchy :** conserve la hiérarchie des fichiers dans le dossier cible. Le chemin d’accès relatif du fichier source vers le dossier source est identique au chemin d’accès relatif du fichier cible vers le dossier cible.<br/><br/>**FlattenHierarchy :** tous les fichiers du dossier source sont créés au premier niveau du dossier cible. Les fichiers cibles sont créés avec un nom généré automatiquement.<br/><br/>**MergeFiles :** fusionne tous les fichiers du dossier source dans un seul fichier. Si le nom d’objet blob ou le nom de fichier est spécifié, le nom de fichier fusionné est le nom spécifié. Dans le cas contraire, il s’agit d’un nom de fichier généré automatiquement. |Non |

auto-

#### <a name="example"></a>Exemple

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2015-06-01T18:00:00",
        "end": "2015-06-01T20:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureSQLtoOnPremisesFile",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureSQLInput"
            }],
            "outputs": [{
                "name": "OnpremisesFileSystemOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlSource",
                    "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "FileSystemSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 3,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

Pour plus d’informations, consultez l’article [File System connector (connecteur Système de fichiers)](data-factory-onprem-file-system-connector.md#copy-activity-properties).

## <a name="ftp"></a>FTP

### <a name="linked-service"></a>Service lié
Pour définir un service lié FTP, définissez le **type** du service lié sur **FtpServer** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire | Default |
| --- | --- | --- | --- |
| host |Nom ou adresse IP du serveur FTP |Oui |&nbsp; |
| authenticationType |Spécification du type d’authentification |Oui |Basic, anonyme |
| username |L’utilisateur ayant accès au serveur FTP |Non |&nbsp; |
| mot de passe |Mot de passe de l’utilisateur (nom d’utilisateur) |Non |&nbsp; |
| encryptedCredential |Informations d’identification chiffrées pour accéder au serveur FTP |Non |&nbsp; |
| gatewayName |Nom de la passerelle de gestion des données pour se connecter à un serveur FTP local |Non |&nbsp; |
| port |Port sur lequel le serveur FTP écoute |Non |21 |
| enableSsl |Indiquez si vous souhaitez utiliser FTP sur un canal SSL/TLS |Non |true |
| enableServerCertificateValidation |Spécifiez si vous souhaitez activer la validation des certificats TLS/SSL lors de l'utilisation de FTP sur un canal SSL/TLS |Non |true |

#### <a name="example-using-anonymous-authentication"></a>Exemple : Utilisation de l’authentification anonyme

```json
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
            "typeProperties": {
            "authenticationType": "Anonymous",
            "host": "myftpserver.com"
        }
    }
}
```

#### <a name="example-using-username-and-password-in-plain-text-for-basic-authentication"></a>Exemple : Utilisation d’un nom d'utilisateur et d’un mot de passe en texte brut pour l’authentification de base

```json
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "username": "Admin",
            "password": "123456"
        }
    }
}
```

#### <a name="example-using-port-enablessl-enableservercertificatevalidation"></a>Exemple : Utilisation de port, enableSsl, enableServerCertificateValidation

```json
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "username": "Admin",
            "password": "123456",
            "port": "21",
            "enableSsl": true,
            "enableServerCertificateValidation": true
        }
    }
}
```

#### <a name="example-using-encryptedcredential-for-authentication-and-gateway"></a>Exemple : Utilisation d’encryptedCredential pour l’authentification et la passerelle

```json
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "encryptedCredential": "xxxxxxxxxxxxxxxxx",
            "gatewayName": "<onpremgateway>"
        }
      }
}
```

Pour plus d’informations, consultez l’article [FTP connector (connecteur FTP)](data-factory-ftp-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données FTP, définissez le **type** du jeu de données sur **FileShare** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| folderPath |Sous-chemin du dossier. Utilisez le caractère d’échappement « \ » pour les caractères spéciaux contenus dans la chaîne. Pour obtenir des exemples, consultez la section Exemples de définitions de jeux de données et de services liés.<br/><br/>Vous pouvez également effectuer une combinaison avec la propriété **partitionBy** pour que les chemins d’accès de dossier soient basés sur les dates et heures de démarrage et d’arrêt de la tranche. |Oui
| fileName |Spécifiez le nom du fichier dans l’élément **folderPath** si vous souhaitez que la table se réfère à un fichier spécifique du dossier. Si vous ne spécifiez aucune valeur pour cette propriété, le tableau pointe vers tous les fichiers du dossier.<br/><br/>Lorsque fileName n’est pas spécifié pour un jeu de données de sortie, le nom du fichier généré aura ce format dans l’exemple suivant : <br/><br/>`Data.<Guid>.txt` (Exemple : Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |Non |
| fileFilter |Spécifiez un filtre à utiliser pour sélectionner un sous-ensemble de fichiers dans le folderPath plutôt que tous les fichiers.<br/><br/>Les valeurs autorisées sont : `*` (plusieurs caractères) et `?` (caractère unique).<br/><br/>Exemple 1 : `"fileFilter": "*.log"`<br/>Exemple 2 : `"fileFilter": 2016-1-?.txt"`<br/><br/> fileFilter s’applique à un jeu de données FileShare d’entrée. Cette propriété n’est pas prise en charge avec HDFS. |Non |
| partitionedBy |partitionedBy peut être utilisé pour spécifier un folderPath dynamique, fileName pour les données de série chronologique. Par exemple, folderPath peut être paramétré pour toutes les heures de données. |Non |
| format | Les types de formats suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Définissez la propriété **type** située sous Format sur l’une de ces valeurs. Pour en savoir plus, consultez les sections relatives à [format Text](data-factory-supported-file-and-compression-formats.md#text-format), [format Json](data-factory-supported-file-and-compression-formats.md#json-format), [format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si vous souhaitez **copier des fichiers en l’état** entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie. |Non |
| compression | Spécifiez le type et le niveau de compression pour les données. Types pris en charge : **GZip**, **Deflate**, **BZip2** et **ZipDeflate** et les niveaux pris en charge sont : **Optimal** et **Fastest**. Pour plus d’informations, consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |Non |
| useBinaryTransfer |Spécifiez si vous voulez utiliser le mode de transfert binaire. True pour le mode binaire et false pour ASCII. Valeur par défaut : True. Cette propriété peut uniquement être utilisée quand le service lié associé est de type : FtpServer. |Non |

> [!NOTE]
> fileName et fileFilter ne peuvent pas être utilisés simultanément.

#### <a name="example"></a>Exemple

```json
{
    "name": "FTPFileInput",
    "properties": {
        "type": "FileShare",
        "linkedServiceName": "FTPLinkedService",
        "typeProperties": {
            "folderPath": "<path to shared folder>",
            "fileName": "test.csv",
            "useBinaryTransfer": true
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

Pour plus d’informations, consultez l’article [FTP connector (connecteur FTP)](data-factory-ftp-connector.md#dataset-properties).

### <a name="file-system-source-in-copy-activity"></a>Source Système de fichiers dans l’activité de copie
Si vous copiez des données à partir d’un serveur FTP, définissez le **type de source** de l’activité de copie sur **FileSystemSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| recursive |Indique si les données sont lues de manière récursive dans les sous-dossiers ou uniquement dans le dossier spécifié. |True, False (par défaut) |Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "FTPToBlobCopy",
            "inputs": [{
                "name": "FtpFileInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
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
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "00:05:00"
            }
        }],
        "start": "2016-08-24T18:00:00",
        "end": "2016-08-24T19:00:00"
    }
}
```

Pour plus d’informations, consultez l’article [FTP connector (connecteur FTP)](data-factory-ftp-connector.md#copy-activity-properties).


## <a name="hdfs"></a>HDFS

### <a name="linked-service"></a>Service lié
Pour définir un service lié HDFS, définissez le **type** du service lié sur **Hdfs** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |La propriété type doit être définie sur : **Hdfs** |Oui |
| Url |URL vers le système HDFS |Oui |
| authenticationType |Anonyme ou Windows. <br><br> Si vous souhaitez utiliser l'**authentification Kerberos** pour le connecteur HDFS, reportez-vous à cette section afin de configurer votre environnement local en conséquence. |Oui |
| userName |Nom d’utilisateur de l’authentification Windows |Oui (pour l’authentification Windows) |
| mot de passe |Mot de passe de l’authentification Windows |Oui (pour l’authentification Windows) |
| gatewayName |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter au système HDFS. |Oui |
| encryptedCredential |[New-AzDataFactoryEncryptValue](/powershell/module/az.datafactory/new-azdatafactoryencryptvalue) des informations d’accès. |Non |

#### <a name="example-using-anonymous-authentication"></a>Exemple : Utilisation de l’authentification anonyme

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "authenticationType": "Anonymous",
            "userName": "hadoop",
            "url": "http://<machine>:50070/webhdfs/v1/",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-using-windows-authentication"></a>Exemple : Avec l’authentification Windows

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "authenticationType": "Windows",
            "userName": "Administrator",
            "password": "password",
            "url": "http://<machine>:50070/webhdfs/v1/",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

Pour plus d'informations, consultez l'article Connecteur HDFS.

### <a name="dataset"></a>Dataset
Pour définir un jeu de données HDFS, définissez le **type** du jeu de données sur **FileShare** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| folderPath |Chemin d'accès au dossier. Exemple : `myfolder`<br/><br/>Utilisez le caractère d’échappement « \ » pour les caractères spéciaux contenus dans la chaîne. Par exemple : pour dossier\sous-dossier, spécifiez dossier\\\\sous-dossier et pour d:\dossier d’exemple, spécifiez d:\\\\dossier d’exemple.<br/><br/>Vous pouvez également effectuer une combinaison avec la propriété **partitionBy** pour que les chemins d’accès de dossier soient basés sur les dates et heures de démarrage et d’arrêt de la tranche. |Oui |
| fileName |Spécifiez le nom du fichier dans l’élément **folderPath** si vous souhaitez que la table se réfère à un fichier spécifique du dossier. Si vous ne spécifiez aucune valeur pour cette propriété, le tableau pointe vers tous les fichiers du dossier.<br/><br/>Lorsque fileName n’est pas spécifié pour un jeu de données de sortie, le nom du fichier généré aura ce format dans l’exemple suivant : <br/><br/>`Data.<Guid>.txt` (par exemple : : Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |Non |
| partitionedBy |partitionedBy peut être utilisé pour spécifier un folderPath dynamique, fileName pour les données de série chronologique. Exemple : folderPath peut être paramétré pour toutes les heures de données. |Non |
| format | Les types de formats suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Définissez la propriété **type** située sous Format sur l’une de ces valeurs. Pour en savoir plus, consultez les sections relatives à [format Text](data-factory-supported-file-and-compression-formats.md#text-format), [format Json](data-factory-supported-file-and-compression-formats.md#json-format), [format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si vous souhaitez **copier des fichiers en l’état** entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie. |Non |
| compression | Spécifiez le type et le niveau de compression pour les données. Types pris en charge : **GZip**, **Deflate**, **BZip2** et **ZipDeflate**. Niveaux pris en charge : **Optimal** et **Fastest**. Pour plus d’informations, consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |Non |

> [!NOTE]
> fileName et fileFilter ne peuvent pas être utilisés simultanément.

#### <a name="example"></a>Exemple

```json
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
            "interval": 1
        }
    }
}
```

Pour plus d'informations, consultez l'article Connecteur HDFS.

### <a name="file-system-source-in-copy-activity"></a>Source Système de fichiers dans l’activité de copie
Si vous copiez des données à partir de HDFS, définissez le **type de source** de l’activité de copie sur **FileSystemSource** et spécifiez les propriétés suivantes dans la section **source** :

**FileSystemSource** prend en charge les propriétés suivantes :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| recursive |Indique si les données sont lues de manière récursive dans les sous-dossiers ou uniquement dans le dossier spécifié. |True, False (par défaut) |Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "HdfsToBlobCopy",
            "inputs": [{
                "name": "InputDataset"
            }],
            "outputs": [{
                "name": "OutputDataset"
            }],
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "00:05:00"
            }
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

Pour plus d'informations, consultez l'article Connecteur HDFS.

## <a name="sftp"></a>SFTP


### <a name="linked-service"></a>Service lié
Pour définir un service lié SFTP, définissez le **type** du service lié sur **Sftp** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| host | Nom ou adresse IP du serveur SFTP. |Oui |
| port |Port sur lequel le serveur SFTP écoute. La valeur par défaut est : 21 |Non |
| authenticationType |Spécification du type d’authentification. Valeurs autorisées : **Basic**, **SshPublicKey**. <br><br> Reportez-vous aux sections Utilisation de l'authentification de base et [Utilisation de l'authentification par clé publique SSH](#using-ssh-public-key-authentication) contenant respectivement des propriétés supplémentaires et des exemples JSON. |Oui |
| skipHostKeyValidation | Spécifiez s’il faut ignorer la validation de la clé hôte. | Non. valeur par défaut : false |
| hostKeyFingerprint | Spécifiez l’empreinte de la clé hôte. | Oui, si `skipHostKeyValidation` est défini sur false.  |
| gatewayName |Nom de la passerelle de gestion des données pour se connecter à un serveur SFTP local. | Oui en cas de copie de données à partir d’un serveur SFTP local. |
| encryptedCredential | Informations d’identification chiffrées pour accéder au serveur SFTP. Généré automatiquement lorsque vous spécifiez l’authentification de base (nom d’utilisateur + mot de passe) ou l’authentification SshPublicKey (nom d’utilisateur + chemin d’accès ou contenu de la clé privée) dans l’Assistant de copie ou la boîte de dialogue contextuelle ClickOnce. | Non. S’applique uniquement pour la copie de données à partir d’un serveur SFTP local. |

#### <a name="example-using-basic-authentication"></a>Exemple : Utilisation de l’authentification de base

Pour utiliser l’authentification de base, définissez `authenticationType` sur `Basic` et spécifiez les propriétés suivantes en plus des propriétés génériques du connecteur SFTP présentées dans la dernière section :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| username | Utilisateur ayant accès au serveur SFTP. |Oui |
| mot de passe | Mot de passe de l’utilisateur (nom d’utilisateur). | Oui |

```json
{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<SFTP server name or IP address>",
            "port": 22,
            "authenticationType": "Basic",
            "username": "xxx",
            "password": "xxx",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-basic-authentication-with-encrypted-credential"></a>Exemple : **Authentification de base avec informations d’identification chiffrées**

```json
{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<FTP server name or IP address>",
            "port": 22,
            "authenticationType": "Basic",
            "username": "xxx",
            "encryptedCredential": "xxxxxxxxxxxxxxxxx",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="using-ssh-public-key-authentication"></a>**Utilisation de l’authentification par clé publique SSH :**

Pour utiliser l’authentification de base, définissez `authenticationType` sur `SshPublicKey` et spécifiez les propriétés suivantes en plus des propriétés génériques du connecteur SFTP présentées dans la dernière section :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| username |Utilisateur ayant accès au serveur SFTP |Oui |
| privateKeyPath | Spécifiez le chemin absolu au fichier de clé privée auquel la passerelle peut accéder. | Spécifiez soit la propriété `privateKeyPath`, soit la propriété `privateKeyContent`. <br><br> S’applique uniquement pour la copie de données à partir d’un serveur SFTP local. |
| privateKeyContent | Une chaîne sérialisée du contenu de la clé privée. L’Assistant de copie peut lire le fichier de clé privée et extraire le contenu de clé privée automatiquement. Si vous utilisez tout autre outil/SDK, utilisez plutôt la propriété privateKeyPath. | Spécifiez soit la propriété `privateKeyPath`, soit la propriété `privateKeyContent`. |
| passPhrase | Spécifiez la phrase secrète/le mot de passe pour déchiffrer la clé privée si le fichier de clé est protégé par une phrase secrète. | Oui, si le fichier de clé privée est protégé par une phrase secrète. |

```json
{
    "name": "SftpLinkedServiceWithPrivateKeyPath",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<FTP server name or IP address>",
            "port": 22,
            "authenticationType": "SshPublicKey",
            "username": "xxx",
            "privateKeyPath": "D:\\privatekey_openssh",
            "passPhrase": "xxx",
            "skipHostKeyValidation": true,
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-sshpublickey-authentication-using-private-key-content"></a>Exemple : **Authentification SshPublicKey avec un contenu de clé privée**

```json
{
    "name": "SftpLinkedServiceWithPrivateKeyContent",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver.westus.cloudapp.azure.com",
            "port": 22,
            "authenticationType": "SshPublicKey",
            "username": "xxx",
            "privateKeyContent": "<base64 string of the private key content>",
            "passPhrase": "xxx",
            "skipHostKeyValidation": true
        }
    }
}
```

Pour plus d’informations, consultez l’article [SFTP connector (connecteur SFTP)](data-factory-sftp-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données SFTP, définissez le **type** du jeu de données sur **FileShare** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| folderPath |Sous-chemin du dossier. Utilisez le caractère d’échappement « \ » pour les caractères spéciaux contenus dans la chaîne. Pour obtenir des exemples, consultez la section Exemples de définitions de jeux de données et de services liés.<br/><br/>Vous pouvez également effectuer une combinaison avec la propriété **partitionBy** pour que les chemins d’accès de dossier soient basés sur les dates et heures de démarrage et d’arrêt de la tranche. |Oui |
| fileName |Spécifiez le nom du fichier dans l’élément **folderPath** si vous souhaitez que la table se réfère à un fichier spécifique du dossier. Si vous ne spécifiez aucune valeur pour cette propriété, le tableau pointe vers tous les fichiers du dossier.<br/><br/>Lorsque fileName n’est pas spécifié pour un jeu de données de sortie, le nom du fichier généré aura ce format dans l’exemple suivant : <br/><br/>`Data.<Guid>.txt` (Exemple : Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |Non |
| fileFilter |Spécifiez un filtre à utiliser pour sélectionner un sous-ensemble de fichiers dans le folderPath plutôt que tous les fichiers.<br/><br/>Les valeurs autorisées sont : `*` (plusieurs caractères) et `?` (caractère unique).<br/><br/>Exemple 1 : `"fileFilter": "*.log"`<br/>Exemple 2 : `"fileFilter": 2016-1-?.txt"`<br/><br/> fileFilter s’applique à un jeu de données FileShare d’entrée. Cette propriété n’est pas prise en charge avec HDFS. |Non |
| partitionedBy |partitionedBy peut être utilisé pour spécifier un folderPath dynamique, fileName pour les données de série chronologique. Par exemple, folderPath peut être paramétré pour toutes les heures de données. |Non |
| format | Les types de formats suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Définissez la propriété **type** située sous Format sur l’une de ces valeurs. Pour en savoir plus, consultez les sections relatives à [format Text](data-factory-supported-file-and-compression-formats.md#text-format), [format Json](data-factory-supported-file-and-compression-formats.md#json-format), [format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Si vous souhaitez **copier des fichiers en l’état** entre des magasins de fichiers (copie binaire), ignorez la section Format dans les deux définitions de jeu de données d’entrée et de sortie. |Non |
| compression | Spécifiez le type et le niveau de compression pour les données. Types pris en charge : **GZip**, **Deflate**, **BZip2** et **ZipDeflate**. Niveaux pris en charge : **Optimal** et **Fastest**. Pour plus d’informations, consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |Non |
| useBinaryTransfer |Spécifiez si vous voulez utiliser le mode de transfert binaire. True pour le mode binaire et false pour ASCII. Valeur par défaut : True. Cette propriété peut uniquement être utilisée quand le service lié associé est de type : FtpServer. |Non |

> [!NOTE]
> fileName et fileFilter ne peuvent pas être utilisés simultanément.

#### <a name="example"></a>Exemple

```json
{
    "name": "SFTPFileInput",
    "properties": {
        "type": "FileShare",
        "linkedServiceName": "SftpLinkedService",
        "typeProperties": {
            "folderPath": "<path to shared folder>",
            "fileName": "test.csv"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

Pour plus d’informations, consultez l’article [SFTP connector (connecteur SFTP)](data-factory-sftp-connector.md#dataset-properties).

### <a name="file-system-source-in-copy-activity"></a>Source Système de fichiers dans l’activité de copie
Si vous copiez des données à partir d’une source SFTP, définissez le **type de source** de l’activité de copie sur **FileSystemSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| recursive |Indique si les données sont lues de manière récursive dans les sous-dossiers ou uniquement dans le dossier spécifié. |True, False (par défaut) |Non |



#### <a name="example"></a>Exemple

```json
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "SFTPToBlobCopy",
            "inputs": [{
                "name": "SFTPFileInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
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
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "00:05:00"
            }
        }],
        "start": "2017-02-20T18:00:00",
        "end": "2017-02-20T19:00:00"
    }
}
```

Pour plus d’informations, consultez l’article [SFTP connector (connecteur SFTP)](data-factory-sftp-connector.md#copy-activity-properties).


## <a name="http"></a>HTTP

### <a name="linked-service"></a>Service lié
Pour définir un service lié HTTP, définissez le **type** du service lié sur **Http** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| url | URL de base du serveur web | Oui |
| authenticationType | Spécifie le type d’authentification. Les valeurs autorisées sont les suivantes : **Anonymous**, **Basic**, **Digest**, **Windows**, **ClientCertificate**. <br><br> Reportez-vous aux sections suivant ce tableau pour accéder à d’autres propriétés et à des exemples JSON sur ces types d’authentification. | Oui |
| enableServerCertificateValidation | Spécifiez si vous souhaitez activer la validation des certificats TLS/SSL lorsque la source est un serveur web HTTPS | Non, la valeur par défaut est True. |
| gatewayName | Nom de la passerelle de gestion des données pour se connecter à une source HTTP locale. | Oui en cas de copie de données à partir d’une source HTTP locale. |
| encryptedCredential | Informations d’identification chiffrées pour accéder au point de terminaison. Elles sont générées automatiquement lorsque vous configurez les informations d’authentification dans l’Assistant de copie ou la boîte de dialogue contextuelle ClickOnce. | Non. S’applique uniquement pour la copie de données à partir d’un serveur HTTP local. |

#### <a name="example-using-basic-digest-or-windows-authentication"></a>Exemple : Utilisation de l’authentification Basic (De base), Digest ou Windows
Définissez `authenticationType` sur `Basic`, `Digest` ou `Windows` et spécifiez les propriétés suivantes en plus des propriétés génériques du connecteur HTTP présentées ci-dessus :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| username | Nom d’utilisateur pour accéder au point de terminaison HTTP. | Oui |
| mot de passe | Mot de passe de l’utilisateur (nom d’utilisateur). | Oui |

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "Http",
        "typeProperties": {
            "authenticationType": "basic",
            "url": "https://en.wikipedia.org/wiki/",
            "userName": "user name",
            "password": "password"
        }
    }
}
```

#### <a name="example-using-clientcertificate-authentication"></a>Exemple : Utilisation de l’authentification ClientCertificate (Certificat client)

Pour utiliser l’authentification de base, définissez `authenticationType` sur `ClientCertificate` et spécifiez les propriétés suivantes en plus des propriétés génériques du connecteur HTTP présentées ci-dessus :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| embeddedCertData | Contenu codé en Base64 des données binaires du fichier Personal Information Exchange (PFX). | Spécifiez soit la propriété `embeddedCertData`, soit la propriété `certThumbprint`. |
| certThumbprint | Empreinte du certificat qui a été installé dans le magasin de certificats de votre ordinateur de passerelle. S’applique uniquement pour la copie de données à partir d’une source HTTP locale. | Spécifiez soit la propriété `embeddedCertData`, soit la propriété `certThumbprint`. |
| mot de passe | Mot de passe associé au certificat. | Non |

Si vous utilisez `certThumbprint` pour l’authentification et le certificat est installé dans le magasin personnel de l’ordinateur local, vous devez accorder l’autorisation de lecture au service de passerelle :

1. Lancez Microsoft Management Console (MMC). Ajouter le composant logiciel enfichable **Certificats** ciblant l’**ordinateur local**.
2. Développez **Certificats**, **Personnel**, puis cliquez sur **Certificats**.
3. Cliquez avec le bouton droit sur le certificat du magasin personnel, puis sélectionnez **Toutes les tâches**->**Gérer les clés privées...**
3. Dans l’onglet **Sécurité**, ajoutez le compte d’utilisateur sous lequel le service hôte de la passerelle de gestion des données s’exécute avec l’accès en lecture au certificat.

**Exemple : utilisation d’un certificat client :** Ce service lié lie votre fabrique de données à un serveur web HTTP local. Il utilise un certificat client installé sur l’ordinateur doté de la passerelle de gestion des données.

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "Http",
        "typeProperties": {
            "authenticationType": "ClientCertificate",
            "url": "https://en.wikipedia.org/wiki/",
            "certThumbprint": "thumbprint of certificate",
            "gatewayName": "gateway name"
        }
    }
}
```

#### <a name="example-using-client-certificate-in-a-file"></a>Exemple : utilisation d’un certificat client dans un fichier
Ce service lié lie votre fabrique de données à un serveur web HTTP local. Il utilise un fichier de certificat client sur l’ordinateur doté de la passerelle de gestion des données.

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "Http",
        "typeProperties": {
            "authenticationType": "ClientCertificate",
            "url": "https://en.wikipedia.org/wiki/",
            "embeddedCertData": "base64 encoded cert data",
            "password": "password of cert"
        }
    }
}
```

Pour plus d’informations, consultez l’article [HTTP connector (connecteur HTTP)](data-factory-http-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données HTTP, définissez le **type** du jeu de données sur **Http** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| relativeUrl | URL relative de la ressource qui contient les données. Quand le chemin d’accès n’est pas spécifié, seule l’URL spécifiée dans la définition du service lié est utilisée. <br><br> Pour construire une URL dynamique, vous pouvez utiliser [les variables système et les fonctions de Data Factory](data-factory-functions-variables.md), par exemple : `"relativeUrl": "$$Text.Format('/my/report?month={0:yyyy}-{0:MM}&fmt=csv', SliceStart)"`. | Non |
| requestMethod | Méthode HTTP. Les valeurs autorisées sont **GET** ou **POST**. | Non. La valeur par défaut est `GET`. |
| additionalHeaders | En-têtes de requête HTTP supplémentaires. | Non |
| requestBody | Corps de la requête HTTP. | Non |
| format | Si vous souhaitez simplement **récupérer les données du point de terminaison HTTP en l’état**, sans les analyser, ignorez ces paramètres de format. <br><br> Si vous souhaitez analyser le contenu de la réponse HTTP pendant la copie, les types de formats suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Pour en savoir plus, consultez les sections relatives à [format Text](data-factory-supported-file-and-compression-formats.md#text-format), [format Json](data-factory-supported-file-and-compression-formats.md#json-format), [format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). |Non |
| compression | Spécifiez le type et le niveau de compression pour les données. Types pris en charge : **GZip**, **Deflate**, **BZip2** et **ZipDeflate**. Niveaux pris en charge : **Optimal** et **Fastest**. Pour plus d’informations, consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |Non |

#### <a name="example-using-the-get-default-method"></a>Exemple : utilisation de la méthode GET (par défaut)

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
            "relativeUrl": "XXX/test.xml",
            "additionalHeaders": "Connection: keep-alive\nUser-Agent: Mozilla/5.0\n"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

#### <a name="example-using-the-post-method"></a>Exemple : utilisation de la méthode POST

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
            "relativeUrl": "/XXX/test.xml",
            "requestMethod": "Post",
            "requestBody": "body for POST HTTP request"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```
Pour plus d’informations, consultez l’article [HTTP connector (connecteur HTTP)](data-factory-http-connector.md#dataset-properties).

### <a name="http-source-in-copy-activity"></a>Source HTTP dans l’activité de copie
Si vous copiez des données à partir d’une source HTTP, définissez le **type de source** de l’activité de copie sur **HttpSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Obligatoire |
| -------- | ----------- | -------- |
| httpRequestTimeout | Délai d’expiration (TimeSpan) pour l’obtention d’une réponse par la requête HTTP. Il s’agit du délai d’expiration pour l’obtention d’une réponse, et non du délai d’expiration pour la lecture des données de la réponse. | Non. Valeur par défaut : 00:01:40 |


#### <a name="example"></a>Exemple

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "HttpSourceToAzureBlob",
            "description": "Copy from an HTTP source to an Azure blob",
            "type": "Copy",
            "inputs": [{
                "name": "HttpSourceDataInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "HttpSource"
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
        }]
    }
}
```

Pour plus d’informations, consultez l’article [HTTP connector (connecteur HTTP)](data-factory-http-connector.md#copy-activity-properties).

## <a name="odata"></a>OData

### <a name="linked-service"></a>Service lié
Pour définir un service lié OData, définissez le **type** du service lié sur **OData** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| url |URL du service OData. |Oui |
| authenticationType |Type d’authentification utilisé pour se connecter à la source OData. <br/><br/> Pour OData dans le cloud, les valeurs possibles sont Anonyme, De base et OAuth (notez qu’à l’heure actuelle, Azure Data Factory prend en charge uniquement l’authentification OAuth basée sur Azure Active Directory). <br/><br/> Pour OData en local, les valeurs possibles sont Anonyme, De base et Windows. |Oui |
| username |Spécifiez le nom d’utilisateur si vous utilisez l’authentification de base. |Oui (uniquement si vous utilisez l’authentification de base) |
| mot de passe |Spécifiez le mot de passe du compte d’utilisateur que vous avez spécifié pour le nom d’utilisateur. |Oui (uniquement si vous utilisez l’authentification de base) |
| authorizedCredential |Si vous utilisez OAuth, cliquez sur le bouton **Autoriser** de l’Assistant de copie Data Factory ou de l’éditeur et entrez vos informations d’identification. La valeur de cette propriété sera alors générée automatiquement. |Oui (uniquement si vous utilisez l’authentification OAuth) |
| gatewayName |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter au service OData local. Indiquez uniquement si vous copiez des données à partir d’une source OData locale. |Non |

#### <a name="example---using-basic-authentication"></a>Exemple : utilisation de l’authentification de base
```json
{
    "name": "inputLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "https://services.odata.org/OData/OData.svc",
            "authenticationType": "Basic",
            "username": "username",
            "password": "password"
        }
    }
}
```

#### <a name="example---using-anonymous-authentication"></a>Exemple : utilisation de l’authentification anonyme

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "https://services.odata.org/OData/OData.svc",
            "authenticationType": "Anonymous"
        }
    }
}
```

#### <a name="example---using-windows-authentication-accessing-on-premises-odata-source"></a>Exemple : utilisation de l’authentification Windows pour accéder à la source OData locale

```json
{
    "name": "inputLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of on-premises OData source, for example, Dynamics CRM>",
            "authenticationType": "Windows",
            "username": "domain\\user",
            "password": "password",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example---using-oauth-authentication-accessing-cloud-odata-source"></a>Exemple : utilisation de l’authentification OAuth pour accéder à la source OData dans le cloud
```json
{
    "name": "inputLinkedService",
    "properties":
    {
        "type": "OData",
            "typeProperties":
        {
            "url": "<endpoint of cloud OData source, for example, https://<tenant>.crm.dynamics.com/XRMServices/2011/OrganizationData.svc>",
            "authenticationType": "OAuth",
            "authorizedCredential": "<auto generated by clicking the Authorize button on UI>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [OData connector (connecteur OData)](data-factory-odata-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données OData, définissez le **type** du jeu de données sur **ODataResource** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| path |Chemin d'accès à la ressource OData |Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "ODataDataset",
    "properties": {
        "type": "ODataResource",
        "typeProperties": {
            "path": "Products"
        },
        "linkedServiceName": "ODataLinkedService",
        "structure": [],
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "retryInterval": "00:01:00",
            "retryTimeout": "00:10:00",
            "maximumRetry": 3
        }
    }
}
```

Pour plus d’informations, consultez l’article [OData connector (connecteur OData)](data-factory-odata-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Source relationnelle dans l’activité de copie
Si vous copiez des données à partir d’une source OData, définissez le **type de source** de l’activité de copie sur **RelationalSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Exemple | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |"?$select=Name, Description&$top=5" |Non |

#### <a name="example"></a>Exemple

```json
{
    "name": "CopyODataToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "?$select=Name, Description&$top=5"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "ODataDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobODataDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "ODataToBlob"
        }],
        "start": "2017-02-01T18:00:00",
        "end": "2017-02-03T19:00:00"
    }
}
```

Pour plus d’informations, consultez l’article [OData connector (connecteur OData)](data-factory-odata-connector.md#copy-activity-properties).


## <a name="odbc"></a>ODBC


### <a name="linked-service"></a>Service lié
Pour définir un service lié ODBC, définissez le **type** du service lié sur **OnPremisesOdbc** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| connectionString |Partie de la chaîne de connexion ne contenant pas les informations d’accès, avec des informations d’identification chiffrées facultatives. Consultez les exemples dans les sections suivantes. |Oui |
| credential |Partie de la chaîne de connexion contenant les informations d’accès, spécifiée dans un format de valeurs de propriété spécifique au pilote. Exemple : `“Uid=<user ID>;Pwd=<password>;RefreshToken=<secret refresh token>;”.` |Non |
| authenticationType |Type d’authentification utilisé pour se connecter au magasin de données ODBC. Les valeurs possibles sont les suivantes : Anonymous et Basic. |Oui |
| username |Spécifiez le nom d’utilisateur si vous utilisez l’authentification de base. |Non |
| mot de passe |Spécifiez le mot de passe du compte d’utilisateur que vous avez spécifié pour le nom d’utilisateur. |Non |
| gatewayName |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter au magasin de données ODBC. |Oui |

#### <a name="example---using-basic-authentication"></a>Exemple : utilisation de l’authentification de base

```json
{
    "name": "ODBCLinkedService",
    "properties": {
        "type": "OnPremisesOdbc",
        "typeProperties": {
            "authenticationType": "Basic",
            "connectionString": "Driver={SQL Server};Server=Server.database.windows.net; Database=TestDatabase;",
            "userName": "username",
            "password": "password",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```
#### <a name="example---using-basic-authentication-with-encrypted-credentials"></a>Exemple : utilisation de l’authentification de base avec des informations d’identification chiffrées
Vous pouvez chiffrer les informations d'identification via la cmdlet [New-AzDataFactoryEncryptValue](/powershell/module/az.datafactory/new-azdatafactoryencryptvalue).

```json
{
    "name": "ODBCLinkedService",
    "properties": {
        "type": "OnPremisesOdbc",
        "typeProperties": {
            "authenticationType": "Basic",
            "connectionString": "Driver={SQL Server};Server=myserver.database.windows.net; Database=TestDatabase;;EncryptedCredential=eyJDb25uZWN0...........................",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-using-anonymous-authentication"></a>Exemple : Utilisation de l’authentification anonyme

```json
{
    "name": "ODBCLinkedService",
    "properties": {
        "type": "OnPremisesOdbc",
        "typeProperties": {
            "authenticationType": "Anonymous",
            "connectionString": "Driver={SQL Server};Server={servername}.database.windows.net; Database=TestDatabase;",
            "credential": "UID={uid};PWD={pwd}",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [ODBC connector (connecteur ODBC)](data-factory-odbc-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données ODBC, définissez le **type** du jeu de données sur **RelationalTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table dans le magasin de données ODBC. |Oui |


#### <a name="example"></a>Exemple

```json
{
    "name": "ODBCDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "ODBCLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
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

Pour plus d’informations, consultez l’article [ODBC connector (connecteur ODBC)](data-factory-odbc-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Source relationnelle dans l’activité de copie
Si vous copiez des données à partir d’un magasin de données ODBC, définissez le **type de source** de l’activité de copie sur **RelationalSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Par exemple : `select * from MyTable`. |Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "CopyODBCToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "OdbcDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobOdbcDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "OdbcToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

Pour plus d’informations, consultez l’article [ODBC connector (connecteur ODBC)](data-factory-odbc-connector.md#copy-activity-properties).

## <a name="salesforce"></a>Salesforce


### <a name="linked-service"></a>Service lié
Pour définir un service lié Salesforce, définissez le **type** du service lié sur **Salesforce** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| environmentUrl | Spécifiez l’URL de l’instance Salesforce. <br><br> - L'URL par défaut est « https:\//login.salesforce.com ». <br> - Pour copier des données du bac à sable, spécifiez « https://test.salesforce.com  ». <br> - Pour copier des données du domaine personnalisé, spécifiez, par exemple : « https://[domain].my.salesforce.com ». |Non |
| username |Spécifiez un nom d’utilisateur pour le compte d’utilisateur. |Oui |
| mot de passe |Spécifiez le mot de passe du compte d’utilisateur. |Oui |
| securityToken |Spécifiez le jeton de sécurité du compte d’utilisateur. Consultez l’article [Get security token](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm) (Obtenir un jeton de sécurité) pour obtenir des instructions sur la réinitialisation et l’obtention d’un jeton de sécurité. Pour en savoir plus sur les jetons de sécurité, consultez l’article [Security and the API](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_concepts_security.htm)(Sécurité et API). |Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "SalesforceLinkedService",
    "properties": {
        "type": "Salesforce",
        "typeProperties": {
            "username": "<user name>",
            "password": "<password>",
            "securityToken": "<security token>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [Salesforce connector (connecteur Salesforce)](data-factory-salesforce-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données Salesforce, définissez le **type** du jeu de données sur **RelationalTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| tableName |Nom de la table dans Salesforce. |Non (si une **requête** de type **RelationalSource** est spécifiée) |

#### <a name="example"></a>Exemple

```json
{
    "name": "SalesforceInput",
    "properties": {
        "linkedServiceName": "SalesforceLinkedService",
        "type": "RelationalTable",
        "typeProperties": {
            "tableName": "AllDataType__c"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
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

Pour plus d’informations, consultez l’article [Salesforce connector (connecteur Salesforce)](data-factory-salesforce-connector.md#dataset-properties).

### <a name="relational-source-in-copy-activity"></a>Source relationnelle dans l’activité de copie
Si vous copiez des données à partir de Salesforce, définissez le **type de source** de l’activité de copie sur **RelationalSource** et spécifiez les propriétés suivantes dans la section **source** :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| query |Utilise la requête personnalisée pour lire des données. |Une requête SQL-92 ou une requête [SOQL (Salesforce Object Query Language)](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql.htm). Par exemple : `select * from MyTable__c`. |Non (si l’attribut **tableName** de l’élément **dataset** est spécifié) |

#### <a name="example"></a>Exemple



```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "SalesforceToAzureBlob",
            "description": "Copy from Salesforce to an Azure blob",
            "type": "Copy",
            "inputs": [{
                "name": "SalesforceInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "SELECT Id, Col_AutoNumber__c, Col_Checkbox__c, Col_Currency__c, Col_Date__c, Col_DateTime__c, Col_Email__c, Col_Number__c, Col_Percent__c, Col_Phone__c, Col_Picklist__c, Col_Picklist_MultiSelect__c, Col_Text__c, Col_Text_Area__c, Col_Text_AreaLong__c, Col_Text_AreaRich__c, Col_URL__c, Col_Text_Encrypt__c, Col_Lookup__c FROM AllDataType__c"
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
        }]
    }
}
```

> [!IMPORTANT]
> La partie « __c » du nom de l’API est requise pour tout objet personnalisé.

Pour plus d’informations, consultez l’article [Salesforce connector (connecteur Salesforce)](data-factory-salesforce-connector.md#copy-activity-properties).

## <a name="web-data"></a>Données Web

### <a name="linked-service"></a>Service lié
Pour définir un service lié Web, définissez le **type** du service lié sur **Web** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| Url |URL de la source web |Oui |
| authenticationType |Anonyme |Oui |


#### <a name="example"></a>Exemple


```json
{
    "name": "web",
    "properties": {
        "type": "Web",
        "typeProperties": {
            "authenticationType": "Anonymous",
            "url": "https://en.wikipedia.org/wiki/"
        }
    }
}
```

Pour plus d’informations, consultez l’article [Web Table connector (connecteur table web)](data-factory-web-table-connector.md#linked-service-properties).

### <a name="dataset"></a>Dataset
Pour définir un jeu de données Web, définissez le **type** du jeu de données sur **WebTable** et spécifiez les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type |Type du jeu de données. Doit avoir la valeur **WebTable** |Oui |
| path |URL relative de la ressource qui contient la table. |Non. Quand le chemin d’accès n’est pas spécifié, seule l’URL spécifiée dans la définition du service lié est utilisée. |
| index |Index de la table dans la ressource. Pour connaître la marche à suivre pour obtenir l'index d'une table sur une page HTML, consultez la section Obtenir l'index d'une table sur une page HTML. |Oui |

#### <a name="example"></a>Exemple

```json
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "linkedServiceName": "WebLinkedService",
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

Pour plus d’informations, consultez l’article [Web Table connector (connecteur table web)](data-factory-web-table-connector.md#dataset-properties).

### <a name="web-source-in-copy-activity"></a>Source Web dans l’activité de copie
Si vous copiez des données à partir d’une table web, définissez le **type de source** de l’activité de copie sur **WebSource**. Actuellement, lorsque la source de l’activité de copie est de type **WebSource**, aucune propriété supplémentaire n’est prise en charge.

#### <a name="example"></a>Exemple

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "WebTableToAzureBlob",
            "description": "Copy from a Web table to an Azure blob",
            "type": "Copy",
            "inputs": [{
                "name": "WebTableInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "WebSource"
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
        }]
    }
}
```

Pour plus d’informations, consultez l’article [Web Table connector (connecteur table web)](data-factory-web-table-connector.md#copy-activity-properties).

## <a name="compute-environments"></a>ENVIRONNEMENTS DE CALCUL
Le tableau suivant liste les environnements de calcul pris en charge par Azure Data Factory et les activités de transformation qui peuvent s’exécuter sur ces derniers. Cliquez sur le lien concernant le calcul qui vous intéresse pour afficher les schémas JSON pour le service lié, pour le lier à une fabrique de données.

| Environnement de calcul | Activités |
| --- | --- |
| [Cluster HDInsight à la demande](#on-demand-azure-hdinsight-cluster) ou [votre propre cluster HDInsight](#existing-azure-hdinsight-cluster) |[Activité personnalisée .NET](#net-custom-activity), [Activité Hive](#hdinsight-hive-activity), [Activité Pig](#hdinsight-pig-activity), [Activité MapReduce](#hdinsight-mapreduce-activity), Activité de streaming Hadoop, [Activité Spark](#hdinsight-spark-activity) |
| [Azure Batch](#azure-batch) |[Activité personnalisée .NET](#net-custom-activity) |
| [Azure Machine Learning Studio (classique)](#azure-machine-learning-studio-classic) | [Activité d'exécution par lot Azure Machine Learning Studio (classique)](#azure-machine-learning-studio-classic-batch-execution-activity) et [Activité des ressources de mise à jour Azure Machine Learning Studio (classique)](#azure-machine-learning-studio-classic-update-resource-activity) |
| [Service Analytique Azure Data Lake](#azure-data-lake-analytics) |[Langage U-SQL du service Analytique Data Lake](#data-lake-analytics-u-sql-activity) |
| [Azure SQL Database](#azure-sql-database), [Azure Synapse Analytics](#azure-synapse-analytics), [SQL Server](#sql-server-stored-procedure) |[Procédure stockée](#stored-procedure-activity) |

## <a name="on-demand-azure-hdinsight-cluster"></a>Cluster Azure HDInsight à la demande
Le service Azure Data Factory peut automatiquement créer un cluster HDInsight à la demande sous Windows/Linux pour traiter les données. Le cluster est créé dans la même région que celle du compte de stockage (propriété linkedServiceName dans JSON) associé au cluster. Vous pouvez exécuter les activités de transformation suivantes sur ce service lié : [Activité personnalisée .NET](#net-custom-activity), [Activité Hive](#hdinsight-hive-activity), [Activité Pig](#hdinsight-pig-activity), [Activité MapReduce](#hdinsight-mapreduce-activity), Activité de streaming Hadoop, [Activité Spark](#hdinsight-spark-activity).

### <a name="linked-service"></a>Service lié
Le tableau suivant décrit les propriétés utilisées dans la définition JSON Azure d’un service lié HDInsight à la demande.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |La propriété de type doit être définie sur **HDInsightOnDemand**. |Oui |
| clusterSize |Nombre de nœuds worker/données dans le cluster. Le cluster HDInsight est créé avec 2 nœuds principaux et le nombre de nœuds worker que vous spécifiez pour cette propriété. Les nœuds étant de taille Standard_D3 à 4 cœurs, un cluster à 4 nœuds de travail prend 24 cœurs (4\*4 = 16 nœuds pour les nœuds de travail + 2\*4 = 8 cœurs pour les nœuds principaux). Pour plus d’informations sur le niveau Standard_D3, voir [Création de clusters Hadoop basés sur Linux dans HDInsight](../../hdinsight/hdinsight-hadoop-provision-linux-clusters.md). |Oui |
| timetolive |La durée d’inactivité autorisée pour le cluster HDInsight à la demande. Spécifie la durée pendant laquelle le cluster HDInsight à la demande reste actif après l’achèvement d’une exécution d’activité s’il n’existe aucun autre travail actif dans le cluster.<br/><br/>Par exemple, si une exécution d’activité prend 6 minutes et si la propriété TimeToLive est définie sur 5 minutes, le cluster reste actif pendant 5 minutes après les 6 minutes du traitement de l’exécution d’activité. Si une autre exécution d’activité intervient dans la fenêtre de 6 minutes, elle est traitée par le même cluster.<br/><br/>La création d’un cluster HDInsight à la demande étant une opération coûteuse (elle peut prendre du temps), utilisez ce paramètre selon le besoin pour améliorer les performances d’une fabrique de données en réutilisant un cluster HDInsight à la demande.<br/><br/>Si vous définissez la valeur de la propriété TimeToLive sur 0, le cluster est supprimé dès que l’exécution d’activité est traitée. En revanche, si vous définissez une valeur élevée, le cluster peut rester inactif inutilement entraînant des coûts élevés. Par conséquent, il est important de définir la valeur appropriée en fonction de vos besoins.<br/><br/>Plusieurs pipelines peuvent partager la même instance du cluster HDInsight à la demande si la valeur de la propriété TimeToLive est correctement définie. |Oui |
| version |Version du cluster HDInsight. Pour plus d’informations, consultez [Versions de HDInsight prises en charge dans Azure Data Factory](data-factory-compute-linked-services.md#supported-hdinsight-versions-in-azure-data-factory). |Non |
| linkedServiceName |Le service lié Azure Storage utilisé par le cluster à la demande pour le stockage et le traitement des données. <p>Actuellement, vous ne pouvez pas créer un cluster HDInsight à la demande qui utilise un Azure Data Lake Store en guise de stockage. Si vous souhaitez stocker les données de résultat à partir du traitement HDInsight dans un Azure Data Lake Store, utilisez une activité de copie pour copier les données du stockage Blob Azure dans Azure Data Lake Store.</p>  | Oui |
| additionalLinkedServiceNames |Spécifie les comptes de stockage supplémentaires pour le service lié HDInsight afin que le service Data Factory puisse les enregistrer en votre nom. |Non |
| osType |Type de système d'exploitation. Les valeurs autorisées sont les suivantes : Windows (par défaut) et Linux |Non |
| hcatalogLinkedServiceName |Le nom du service lié à SQL Azure pointant vers la base de données HCatalog. Le cluster HDInsight à la demande est créé en utilisant Azure SQL Database en tant que metastore. |Non |

### <a name="json-example"></a>Exemple JSON
Le JSON suivant définit un service lié HDInsight à la demande sous Linux. Le service Data Factory crée automatiquement un cluster HDInsight **sous Linux** lors du traitement d’une tranche de données.

```json
{
    "name": "HDInsightOnDemandLinkedService",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.5",
            "clusterSize": 1,
            "timeToLive": "00:05:00",
            "osType": "Linux",
            "linkedServiceName": "StorageLinkedService"
        }
    }
}
```

Pour plus d’informations, consultez l’article [Compute linked services (services liés de calcul)](data-factory-compute-linked-services.md).

## <a name="existing-azure-hdinsight-cluster"></a>Cluster Azure HDInsight existant
Vous pouvez créer un service lié Azure HDInsight pour inscrire votre propre cluster HDInsight avec Data Factory. Vous pouvez exécuter les activités de transformation de données suivantes sur ce service lié : [Activité personnalisée .NET](#net-custom-activity), [Activité Hive](#hdinsight-hive-activity), [Activité Pig](#hdinsight-pig-activity), [Activité MapReduce](#hdinsight-mapreduce-activity), Activité de streaming Hadoop, [Activité Spark](#hdinsight-spark-activity).

### <a name="linked-service"></a>Service lié
Le tableau suivant décrit les propriétés utilisées dans la définition JSON Azure d’un service lié Azure HDInsight.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |La propriété de type doit être définie sur **HDInsight**. |Oui |
| clusterUri |L'URI du cluster HDInsight. |Oui |
| username |Spécifiez le nom de l'utilisateur à utiliser pour se connecter à un cluster HDInsight existant. |Oui |
| mot de passe |Spécifiez le mot de passe du compte d'utilisateur. |Oui |
| linkedServiceName | Nom du service lié de stockage Azure faisant référence au stockage Blob Azure utilisé par le cluster HDInsight. <p>Actuellement, vous ne pouvez pas spécifier un service lié Azure Data Lake Store pour cette propriété. Vous pouvez accéder aux données d’Azure Data Lake Store à partir de scripts Hive/Pig si le cluster HDInsight a accès à Data Lake Store. </p>  |Oui |

Pour obtenir la liste des versions des clusters HDInsight pris en charge, consultez [Versions de HDInsight prises en charge](data-factory-compute-linked-services.md#supported-hdinsight-versions-in-azure-data-factory).

#### <a name="json-example"></a>Exemple JSON

```json
{
    "name": "HDInsightLinkedService",
    "properties": {
        "type": "HDInsight",
        "typeProperties": {
            "clusterUri": " https://<hdinsightclustername>.azurehdinsight.net/",
            "userName": "admin",
            "password": "<password>",
            "linkedServiceName": "MyHDInsightStoragelinkedService"
        }
    }
}
```

## <a name="azure-batch"></a>Azure Batch
Vous pouvez créer un service lié Azure Batch pour inscrire un pool Batch de machines virtuelles avec une fabrique de données. Vous pouvez exécuter des activités personnalisées .NET à l'aide d'Azure Batch ou Azure HDInsight. Vous pouvez exécuter une [activité personnalisée .NET](#net-custom-activity) sur ce service lié.

### <a name="linked-service"></a>Service lié
Le tableau suivant décrit les propriétés utilisées dans la définition JSON Azure d’un service lié Azure Batch.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |La propriété de type doit être définie sur **AzureBatch**. |Oui |
| accountName |Nom du compte Azure Batch. |Oui |
| accessKey |Clé d'accès du compte Azure Batch. |Oui |
| poolName |Nom du pool de machines virtuelles. |Oui |
| linkedServiceName |Nom du service lié Azure Storage associé à ce service lié Azure Batch. Ce service lié est utilisé pour présenter les fichiers nécessaires à l’exécution de l’activité et stocker les journaux d’activité d’exécution de l’activité. |Oui |


#### <a name="json-example"></a>Exemple JSON

```json
{
    "name": "AzureBatchLinkedService",
    "properties": {
        "type": "AzureBatch",
        "typeProperties": {
            "accountName": "<Azure Batch account name>",
            "accessKey": "<Azure Batch account key>",
            "poolName": "<Azure Batch pool name>",
            "linkedServiceName": "<Specify associated storage linked service reference here>"
        }
    }
}
```

## <a name="azure-machine-learning-studio-classic"></a>Azure Machine Learning Studio (classique)
Vous créez un service lié Azure Machine Learning Studio (classique) pour inscrire un point de terminaison de notation par lot Studio (classique) avec une fabrique de données. Deux activités de transformation des données pouvant être exécutées sur ce service lié : [Activité d'exécution par lot Azure Machine Learning Studio (classique)](#azure-machine-learning-studio-classic-batch-execution-activity) et [Activité des ressources de mise à jour Azure Machine Learning Studio (classique)](#azure-machine-learning-studio-classic-update-resource-activity).

### <a name="linked-service"></a>Service lié
Le tableau suivant décrit les propriétés utilisées dans la définition JSON Azure d’un service lié Studio (classique).

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| Type |La propriété de type doit être définie sur : **AzureML**. |Oui |
| mlEndpoint |L'URL de la notation par lot. |Oui |
| apiKey |L'API du modèle d'espace de travail publié. |Oui |

#### <a name="json-example"></a>Exemple JSON

```json
{
    "name": "AzureMLLinkedService",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://[batch scoring endpoint]/jobs",
            "apiKey": "<apikey>"
        }
    }
}
```

## <a name="azure-data-lake-analytics"></a>Service Analytique Azure Data Lake
Vous créez un service lié **Analytique Azure Data Lake** pour lier un service de calcul Analytique Azure Data Lake Analytics à une fabrique de données Azure avant d’utiliser l’ [activité U-SQL Analytique Data Lake](data-factory-usql-activity.md) dans un pipeline.

### <a name="linked-service"></a>Service lié

Le tableau suivant décrit les propriétés utilisées dans la définition JSON d’un service lié Azure Data Lake Analytics.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| Type |La propriété de type doit être définie sur : **AzureDataLakeAnalytics**. |Oui |
| accountName |Nom du compte du service Analytique Azure Data Lake. |Oui |
| dataLakeAnalyticsUri |URI du service Analytique Azure Data Lake. |Non |
| autorisation |Le code d’autorisation est automatiquement récupéré après un clic sur le bouton **Autoriser** dans l’éditeur de la fabrique de données et une fois la connexion OAuth effectuée. |Oui |
| subscriptionId |ID d’abonnement Azure |Non (si non spécifié, l’abonnement de la fabrique de données est utilisé). |
| resourceGroupName |Nom du groupe de ressources Azure |Non (si non spécifié, le groupe de ressources de la fabrique de données est utilisé). |
| sessionID |ID de session issu de la session d’autorisation OAuth. Chaque ID de session est unique et ne peut être utilisé qu’une seule fois. Voici l’ID de session généré automatiquement lorsque vous utilisez Data Factory. |Oui |


#### <a name="json-example"></a>Exemple JSON
L’exemple suivant présente la définition JSON pour le service lié Analytique Azure Data Lake.

```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "<account name>",
            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
            "authorization": "<authcode>",
            "sessionId": "<session ID>",
            "subscriptionId": "<subscription id>",
            "resourceGroupName": "<resource group name>"
        }
    }
}
```

## <a name="sql-server-stored-procedure"></a>Procédure stockée SQL Server

Créez un service lié à SQL Server et utilisez-le avec l’ [activité de procédure stockée](data-factory-stored-proc-activity.md) pour appeler une procédure stockée à partir d’un pipeline Data Factory.

### <a name="linked-service"></a>Service lié
Vous créez un service lié de type **OnPremisesSqlServer** pour lier une base de données SQL Server à une fabrique de données. Le tableau suivant fournit la description des éléments JSON spécifiques au service lié SQL Server.

Le tableau suivant fournit la description des éléments JSON spécifiques au service lié SQL Server.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |La propriété de type doit être définie sur : **OnPremisesSqlServer**. |Oui |
| connectionString |Spécifiez les informations connectionString nécessaires pour établir une connexion à la base de données SQL Server à l’aide de l’authentification SQL ou de l’authentification Windows. |Oui |
| gatewayName |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter à la base de données SQL Server. |Oui |
| username |Spécifiez le nom d’utilisateur si vous utilisez l’authentification Windows. Exemple : **domainname\\username**. |Non |
| mot de passe |Spécifiez le mot de passe du compte d’utilisateur que vous avez spécifié pour le nom d’utilisateur. |Non |

Vous pouvez chiffrer les informations d’identification à l’aide de la cmdlet **New-AzDataFactoryEncryptValue**, puis les utiliser dans la chaîne de connexion comme indiqué dans l’exemple suivant (propriété **EncryptedCredential**) :

```JSON
"connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=True;EncryptedCredential=<encrypted credential>",
```


#### <a name="example-json-for-using-sql-authentication"></a>Exemple : JSON pour utilisation de l’authentification SQL

```json
{
    "name": "MyOnPremisesSQLDB",
    "properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "connectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=False;User ID=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```
#### <a name="example-json-for-using-windows-authentication"></a>Exemple : JSON pour utilisation de l’authentification Windows

Si le nom d’utilisateur et le mot de passe sont spécifiés, la passerelle les utilise pour emprunter l’identité du compte utilisateur spécifié pour la connexion à la base de données SQL Server. Dans le cas contraire, la passerelle se connecte directement à SQL Server avec le contexte de sécurité de la passerelle (son compte de démarrage).

```json
{
    "Name": " MyOnPremisesSQLDB",
    "Properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "ConnectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=True;",
            "username": "<domain\\username>",
            "password": "<password>",
            "gatewayName": "<gateway name>"
        }
    }
}
```

Pour plus d’informations, consultez l’article [SQL Server connector (connecteur SQL Server)](data-factory-sqlserver-connector.md#linked-service-properties).

## <a name="data-transformation-activities"></a>ACTIVITÉS DE TRANSFORMATION DES DONNÉES

Activité | Description
-------- | -----------
[Activité Hive HDInsight](#hdinsight-hive-activity) | L’activité Hive HDInsight d’un pipeline Data Factory exécute des requêtes Hive sur votre propre cluster ou cluster à la demande HDInsight sous Windows ou Linux.
[Activité Pig HDInsight](#hdinsight-pig-activity) | L’activité Pig HDInsight d’un pipeline Data Factory exécute des requêtes Pig sur votre propre cluster ou cluster à la demande HDInsight sous Windows ou Linux.
[Activité MapReduce HDInsight](#hdinsight-mapreduce-activity) | L’activité MapReduce de HDInsight dans un pipeline Data Factory exécute des programmes MapReduce sur votre cluster HDInsight propre ou à la demande sous Windows ou Linux.
[Activité de diffusion en continu HDInsight](#hdinsight-streaming-activity) | L’activité de diffusion en continu HDInsight dans un pipeline Data Factory exécute des programmes de diffusion en continu Hadoop sur votre cluster HDInsight propre ou à la demande sous Windows ou Linux.
[Activité HDInsight Spark](#hdinsight-spark-activity) | L’activité Spark HDInsight d’un pipeline Data Factory exécute des programmes Spark sur votre propre cluster HDInsight.
[Activité d’exécution par lot Azure Machine Learning Studio (classique)](#azure-machine-learning-studio-classic-batch-execution-activity) | Azure Data Factory vous permet de créer facilement des pipelines qui utilisent un service Web Studio (classique) publié pour l’analyse prédictive. À l’aide de l’activité d’exécution par lot dans un pipeline Azure Data Factory, vous pouvez appeler un service Web Studio (classique) pour effectuer des prédictions sur les données par lot.
[Activité de ressource de mise à jour Azure Machine Learning Studio (classique)](#azure-machine-learning-studio-classic-update-resource-activity) | Au fil du temps, les modèles prédictifs dans les expériences de scoring Azure Machine Learning Studio (classique) doivent être réentraînés à l’aide de nouveaux jeux de données d’entrée. Une fois que vous avez fini la reformation, vous souhaitez mettre à jour le service Web de notation avec le modèle Machine Learning reformé. Vous pouvez utiliser l’activité des ressources de mise à jour pour mettre à jour le service web avec le modèle qui vient d’être formé.
[Activité de procédure stockée](#stored-procedure-activity) | Vous pouvez utiliser l’activité de procédure stockée dans un pipeline Data Factory pour appeler une procédure stockée dans l’un des magasins de données suivants : Azure SQL Database, Azure Synapse Analytics ou SQL Server Database dans votre entreprise ou une machine virtuelle Azure.
[Activité U-SQL Data Lake Analytics](#data-lake-analytics-u-sql-activity) | L’activité U-SQL Data Lake Analytics exécute un script U-SQL sur un cluster Azure Data Lake Analytics.
[Activité personnalisée .NET](#net-custom-activity) | Si vous devez transformer les données d’une manière qui n’est pas prise en charge par Data Factory, créez une activité personnalisée avec votre propre logique de traitement des données et utilisez cette activité dans le pipeline. Vous pouvez configurer l’activité .NET personnalisée pour l’exécuter en utilisant un service Azure Batch ou un cluster Azure HDInsight.


## <a name="hdinsight-hive-activity"></a>Activité Hive HDInsight
Vous pouvez spécifier les propriétés suivantes dans une définition JSON d’activité Hive. La propriété de type de l’activité doit être : **HDInsightHive**. Vous devez d’abord créer un service lié HDInsight, puis spécifier le nom de celui-ci en tant que valeur de la propriété **linkedServiceName**. Les propriétés suivantes sont prises en charge dans la section **typeProperties** lorsque vous définissez le type d’activité sur HDInsightHive :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| script |Spécifier le script en ligne Hive |Non |
| scriptPath |Stockez le script Hive dans un stockage d'objets blob Azure et indiquez le chemin d'accès au fichier. Utilisez la propriété ’script’ ou ’scriptPath’. Les deux propriétés ne peuvent pas être utilisées simultanément. Le nom de fichier respecte la casse. |Non |
| defines |Spécifier les paramètres sous forme de paires clé/valeur pour le référencement au sein du script Hive à l'aide de ’hiveconf’ |Non |

Ces propriétés de type sont spécifiques à l’activité Hive. D’autres propriétés (en dehors de la section typeProperties) sont prises en charge pour toutes les activités.

### <a name="json-example"></a>Exemple JSON
Le JSON suivant définit une Activité Hive HDInsight dans un pipeline.

```json
{
    "name": "Hive Activity",
    "description": "description",
    "type": "HDInsightHive",
    "inputs": [
      {
        "name": "input tables"
      }
    ],
    "outputs": [
      {
        "name": "output tables"
      }
    ],
    "linkedServiceName": "MyHDInsightLinkedService",
    "typeProperties": {
      "script": "Hive script",
      "scriptPath": "<pathtotheHivescriptfileinAzureblobstorage>",
      "defines": {
        "param1": "param1Value"
      }
    },
   "scheduler": {
      "frequency": "Day",
      "interval": 1
    }
}
```

Pour plus d’informations, consultez l’article [Hive Activity (activité Hive)](data-factory-hive-activity.md).

## <a name="hdinsight-pig-activity"></a>Activité Pig HDInsight
Vous pouvez spécifier les propriétés suivantes dans une définition JSON d’activité pig. La propriété de type de l’activité doit être : **HDInsightPig**. Vous devez d’abord créer un service lié HDInsight, puis spécifier le nom de celui-ci en tant que valeur de la propriété **linkedServiceName**. Les propriétés suivantes sont prises en charge dans la section **typeProperties** lorsque vous définissez le type d’activité sur HDInsightPig :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| script |Spécifier le script en ligne pig |Non |
| scriptPath |Stockez le script pig dans un stockage d'objets blob Azure et indiquez le chemin d'accès au fichier. Utilisez la propriété ’script’ ou ’scriptPath’. Les deux propriétés ne peuvent pas être utilisées simultanément. Le nom de fichier respecte la casse. |Non |
| defines |Spécifier les paramètres sous forme de paires clé/valeur pour le référencement au sein du script pig |Non |

Ces propriétés de type sont spécifiques à l’activité pig. D’autres propriétés (en dehors de la section typeProperties) sont prises en charge pour toutes les activités.

### <a name="json-example"></a>Exemple JSON

```json
{
    "name": "HiveActivitySamplePipeline",
      "properties": {
    "activities": [
        {
            "name": "Pig Activity",
            "description": "description",
            "type": "HDInsightPig",
            "inputs": [
                  {
                    "name": "input tables"
                  }
            ],
            "outputs": [
                  {
                    "name": "output tables"
                  }
            ],
            "linkedServiceName": "MyHDInsightLinkedService",
            "typeProperties": {
                  "script": "Pig script",
                  "scriptPath": "<pathtothePigscriptfileinAzureblobstorage>",
                  "defines": {
                    "param1": "param1Value"
                  }
            },
               "scheduler": {
                  "frequency": "Day",
                  "interval": 1
            }
          }
    ]
  }
}
```

Pour plus d'informations, consultez l'article Activité Pig.

## <a name="hdinsight-mapreduce-activity"></a>Activité MapReduce HDInsight
Vous pouvez spécifier les propriétés suivantes dans une définition JSON d’activité MapReduce. La propriété de type de l’activité doit être : **HDInsightMapReduce**. Vous devez d’abord créer un service lié HDInsight, puis spécifier le nom de celui-ci en tant que valeur de la propriété **linkedServiceName**. Les propriétés suivantes sont prises en charge dans la section **typeProperties** lorsque vous définissez le type d’activité sur HDInsightMapReduce :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| jarLinkedService | Nom du service lié pour le stockage Azure qui contient le fichier JAR. | Oui |
| jarFilePath | Chemin d’accès du fichier JAR dans le stockage Azure. | Oui |
| ClassName | Nom de la classe principale dans le fichier JAR. | Oui |
| arguments | Une liste séparée par des virgules des arguments du programme MapReduce. Lors de l’exécution, vous verrez quelques arguments supplémentaires (par exemple : mapreduce.job.tags) à partir de l’infrastructure MapReduce. Pour différencier vos arguments avec les arguments MapReduce, envisagez d’utiliser l’option et la valeur en tant qu’arguments, comme indiqué dans l’exemple suivant (-s, --entrée, --sortie, etc. sont des options immédiatement suivies par leurs valeurs) | Non |

### <a name="json-example"></a>Exemple JSON

```json
{
    "name": "MahoutMapReduceSamplePipeline",
    "properties": {
        "description": "Sample Pipeline to Run a Mahout Custom Map Reduce Jar. This job calculates an Item Similarity Matrix to determine the similarity between two items",
        "activities": [
            {
                "type": "HDInsightMapReduce",
                "typeProperties": {
                    "className": "org.apache.mahout.cf.taste.hadoop.similarity.item.ItemSimilarityJob",
                    "jarFilePath": "adfsamples/Mahout/jars/mahout-examples-0.9.0.2.2.7.1-34.jar",
                    "jarLinkedService": "StorageLinkedService",
                    "arguments": ["-s", "SIMILARITY_LOGLIKELIHOOD", "--input", "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/input", "--output", "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/output/", "--maxSimilaritiesPerItem", "500", "--tempDir", "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/temp/mahout"]
                },
                "inputs": [
                    {
                        "name": "MahoutInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "MahoutOutput"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "retry": 3
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "MahoutActivity",
                "description": "Custom Map Reduce to generate Mahout result",
                "linkedServiceName": "HDInsightLinkedService"
            }
        ],
        "start": "2017-01-03T00:00:00",
        "end": "2017-01-04T00:00:00"
    }
}
```

Pour plus d’informations, consultez l’article [MapReduce Activity (activité MapReduce)](data-factory-map-reduce.md).

## <a name="hdinsight-streaming-activity"></a>Activité de diffusion en continu HDInsight
Vous pouvez spécifier les propriétés suivantes dans une définition JSON d’activité de diffusion en continu Hadoop. La propriété de type de l’activité doit être : **HDInsightStreaming**. Vous devez d’abord créer un service lié HDInsight, puis spécifier le nom de celui-ci en tant que valeur de la propriété **linkedServiceName**. Les propriétés suivantes sont prises en charge dans la section **typeProperties** lorsque vous définissez le type d’activité sur HDInsightStreaming :

| Propriété | Description |
| --- | --- |
| mappeur | Nom du fichier exécutable du mappeur. Dans l’exemple, cat.exe est le fichier exécutable du mappeur.|
| raccord de réduction | Nom du fichier exécutable du raccord de réduction. Dans l’exemple, wc.exe est le fichier exécutable du raccord de réduction. |
| entrée | Fichier d’entrée (emplacement compris) pour le mappeur. Dans l’exemple : `"wasb://adfsample@<account name>.blob.core.windows.net/example/data/gutenberg/davinci.txt"` : adfsample est le conteneur de l’objet blob, example/data/Gutenberg est le dossier et davinci.txt est l’objet blob. |
| sortie | Fichier de sortie (emplacement compris) pour le raccord de réduction. La sortie de la tâche de diffusion en continu Hadoop est écrite à l’emplacement spécifié pour cette propriété. |
| filePaths | Chemins d’accès pour les exécutables du mappeur et du raccord de réduction. Dans l’exemple « adfsample/example/apps/wc.exe », adfsample est le conteneur de l’objet blob, example/apps est le dossier et wc.exe est le fichier exécutable. |
| fileLinkedService | Service lié Stockage Azure qui représente le stockage Azure qui contient les fichiers spécifiés dans la section filePaths. |
| arguments | Une liste séparée par des virgules des arguments du programme MapReduce. Lors de l’exécution, vous verrez quelques arguments supplémentaires (par exemple : mapreduce.job.tags) à partir de l’infrastructure MapReduce. Pour différencier vos arguments avec les arguments MapReduce, envisagez d’utiliser l’option et la valeur en tant qu’arguments, comme indiqué dans l’exemple suivant (-s, --entrée, --sortie, etc. sont des options immédiatement suivies par leurs valeurs) |
| getDebugInfo | Un élément facultatif. Si sa valeur est Failure, les journaux d’activité ne sont téléchargés qu’en cas d’échec. Si sa valeur est All, les journaux d’activité sont toujours téléchargés, quel que soit l’état de l’exécution. |

> [!NOTE]
> Vous devez spécifier un jeu de données de sortie pour l’activité de diffusion en continu Hadoop pour la propriété **outputs** . Il peut s’agir simplement d’un jeu de données factice qui est nécessaire au fonctionnement de la planification (horaire, quotidienne, etc.) du pipeline. Si l’activité n’accepte pas d’entrée, vous pouvez ignorer la spécification d’un jeu de données d’entrée relatif à l’activité pour la propriété **inputs**.

## <a name="json-example"></a>Exemple JSON

```json
{
    "name": "HadoopStreamingPipeline",
    "properties": {
        "description": "Hadoop Streaming Demo",
        "activities": [
            {
                "type": "HDInsightStreaming",
                "typeProperties": {
                    "mapper": "cat.exe",
                    "reducer": "wc.exe",
                    "input": "wasb://<nameofthecluster>@spestore.blob.core.windows.net/example/data/gutenberg/davinci.txt",
                    "output": "wasb://<nameofthecluster>@spestore.blob.core.windows.net/example/data/StreamingOutput/wc.txt",
                    "filePaths": ["<nameofthecluster>/example/apps/wc.exe","<nameofthecluster>/example/apps/cat.exe"],
                    "fileLinkedService": "StorageLinkedService",
                    "getDebugInfo": "Failure"
                },
                "outputs": [
                    {
                        "name": "StreamingOutputDataset"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "RunHadoopStreamingJob",
                "description": "Run a Hadoop streaming job",
                "linkedServiceName": "HDInsightLinkedService"
            }
        ],
        "start": "2014-01-04T00:00:00",
        "end": "2014-01-05T00:00:00"
    }
}
```

Pour plus d’informations, consultez l’article [Hadoop Streaming Activity (activité de diffusion en continu Hadoop)](data-factory-hadoop-streaming-activity.md).

## <a name="hdinsight-spark-activity"></a>Activité Spark HDInsight
Vous pouvez spécifier les propriétés suivantes dans une définition JSON d’activité Spark. La propriété de type de l’activité doit être : **HDInsightSpark**. Vous devez d’abord créer un service lié HDInsight, puis spécifier le nom de celui-ci en tant que valeur de la propriété **linkedServiceName**. Les propriétés suivantes sont prises en charge dans la section **typeProperties** lorsque vous définissez le type d’activité sur HDInsightSpark :

| Propriété | Description | Obligatoire |
| -------- | ----------- | -------- |
| rootPath | Conteneur d’objets blob Azure et dossier contenant le fichier Spark. Le nom de fichier respecte la casse. | Oui |
| entryFilePath | Chemin d’accès relatif au dossier racine du code/package Spark. | Oui |
| ClassName | Classe principale Java/Spark de l’application. | Non |
| arguments | Liste d’arguments de ligne de commande du programme Spark. | Non |
| proxyUser | Identité du compte d’utilisateur à emprunter pour exécuter le programme Spark. | Non |
| sparkConfig | Propriétés de configuration Spark. | Non |
| getDebugInfo | Spécifie quand les fichiers journaux de Spark sont copiés vers le stockage Azure utilisé par le cluster HDInsight (ou) spécifié par sparkJobLinkedService. Valeurs autorisées : None, Always ou Failure. Valeur par défaut : Aucun. | Non |
| sparkJobLinkedService | Service lié de stockage Azure qui contient le fichier de travail, les dépendances et les journaux d’activité Spark.  Si vous ne spécifiez pas de valeur pour cette propriété, le stockage associé au cluster HDInsight est utilisé. | Non |

### <a name="json-example"></a>Exemple JSON

```json
{
    "name": "SparkPipeline",
    "properties": {
        "activities": [
            {
                "type": "HDInsightSpark",
                "typeProperties": {
                    "rootPath": "adfspark\\pyFiles",
                    "entryFilePath": "test.py",
                    "getDebugInfo": "Always"
                },
                "outputs": [
                    {
                        "name": "OutputDataset"
                    }
                ],
                "name": "MySparkActivity",
                "linkedServiceName": "HDInsightLinkedService"
            }
        ],
        "start": "2017-02-05T00:00:00",
        "end": "2017-02-06T00:00:00"
    }
}
```
Notez les points suivants :

- La propriété **type** est définie sur **HDInsightSpark**.
- **rootPath** est définie sur **adfspark\\pyFiles**, où adfspark est le conteneur d’objets Blob Azure contenant le dossier pyFiles. Dans cet exemple, le stockage Blob Azure est celui qui est associé au cluster Spark. Vous pouvez charger le fichier vers un autre stockage Azure. Si vous procédez ainsi, créez un service lié de stockage Azure pour lier ce compte de stockage à la fabrique de données. Ensuite, spécifiez le nom du service lié en tant que valeur pour la propriété **sparkJobLinkedService**. Consultez Propriétés de l'activité Spark pour plus d'informations sur cette propriété et d'autres propriétés prises en charge par l'activité Spark.
- La propriété **entryFilePath** est définie sur **test.py**, c’est-à-dire le fichier python.
- La propriété **getDebugInfo** est définie sur **Always**, ce qui signifie que les fichiers journaux sont toujours générés (succès ou échec).

    > [!IMPORTANT]
    > Nous vous recommandons de ne pas définir cette propriété sur Always dans un environnement de production, sauf si vous dépannez un problème.
- La section **outputs** possède un jeu de données de sortie. Vous devez spécifier un jeu de données de sortie même si le programme Spark ne génère aucune sortie. Le jeu de données de sortie pilote la planification du pipeline (horaire, quotidienne, etc.).

Pour plus d’informations sur l’activité, consultez l’article [Spark Activity (activité Spark)](data-factory-spark.md).

## <a name="azure-machine-learning-studio-classic-batch-execution-activity"></a>Activité d’exécution par lot Azure Machine Learning Studio (classique)
Vous pouvez spécifier les propriétés suivantes dans une définition JSON d’activité d’exécution par lot Azure Machine Learning Studio (classique). La propriété de type de l’activité doit être : **AzureMLBatchExecution**. Vous devez d’abord créer un service lié Studio (classique), puis spécifier le nom de celui-ci en tant que valeur de la propriété **linkedServiceName**. Les propriétés suivantes sont prises en charge dans la section **typeProperties** lorsque vous définissez le type d’activité sur AzureMLBatchExecution :

Propriété | Description | Obligatoire
-------- | ----------- | --------
webServiceInput | Le jeu de données à transmettre en tant qu’entrée pour le service Web Studio (classique). Ce jeu de données doit également être inclus dans les entrées de l’activité. |Utilisez webServiceInput ou webServiceInputs. |
webServiceInputs | Spécifie les jeux de données à transmettre en tant qu’entrées pour le service Web Studio (classique). Si le service web prend plusieurs entrées, utilisez la propriété webServiceInputs au lieu de la propriété webServiceInput. Les jeux de données référencés par **webServiceInputs** doivent également être inclus dans les **entrées** de l’activité. | Utilisez webServiceInput ou webServiceInputs. |
webServiceOutputs | Les jeux de données qui sont attribués en tant que sorties pour le service Web Studio (classique). Le service web renvoie des données de sortie dans ce jeu de données. | Oui |
globalParameters | Spécifie les valeurs pour les paramètres de service web dans cette section. | Non |

### <a name="json-example"></a>Exemple JSON
Dans cet exemple, l’activité a le jeu de données **MLSqlInput** en tant qu’entrée et **MLSqlOutput** en tant que sortie. Le jeu de données **MLSqlInput** est transmis en tant qu’entrée au service web à l’aide de la propriété JSON **webServiceInput**. Le jeu de données **MLSqlOutput** est transmis en tant que sortie au service web à l’aide de la propriété JSON **webServiceOutputs**.

```json
{
   "name": "MLWithSqlReaderSqlWriter",
   "properties": {
      "description": "Azure ML model with sql azure reader/writer",
      "activities": [{
         "name": "MLSqlReaderSqlWriterActivity",
         "type": "AzureMLBatchExecution",
         "description": "test",
         "inputs": [ { "name": "MLSqlInput" }],
         "outputs": [ { "name": "MLSqlOutput" } ],
         "linkedServiceName": "MLSqlReaderSqlWriterDecisionTreeModel",
         "typeProperties":
         {
            "webServiceInput": "MLSqlInput",
            "webServiceOutputs": {
               "output1": "MLSqlOutput"
            },
            "globalParameters": {
               "Database server name": "<myserver>.database.windows.net",
               "Database name": "<database>",
               "Server user account name": "<user name>",
               "Server user account password": "<password>"
            }
         },
         "policy": {
            "concurrency": 1,
            "executionPriorityOrder": "NewestFirst",
            "retry": 1,
            "timeout": "02:00:00"
         }
      }],
      "start": "2016-02-13T00:00:00",
       "end": "2016-02-14T00:00:00"
   }
}
```

Dans l’exemple JSON, le service Web Studio (classique) déployé utilise un module lecteur et un module enregistreur pour lire/écrire des données depuis/vers une base de données Azure SQL Database. Ce service web expose les quatre paramètres suivants :  Database server name, Database name, Server user account name et Server user account password.

> [!NOTE]
> Seules les entrées et sorties de l’activité AzureMLBatchExecution peuvent être transmises en tant que paramètres au service web. Par exemple, dans l’extrait de code JSON ci-dessus, MLSqlInput est une entrée de l’activité AzureMLBatchExecution, qui est transmise comme entrée au service web via le paramètre webServiceInput.

## <a name="azure-machine-learning-studio-classic-update-resource-activity"></a>Activité de ressource de mise à jour Azure Machine Learning Studio (classique)
Vous pouvez spécifier les propriétés suivantes dans une définition JSON d’activité des ressources de mise à jour Azure Machine Learning Studio (classique). La propriété de type de l’activité doit être : **AzureMLUpdateResource**. Vous devez d’abord créer un service lié Studio (classique), puis spécifier le nom de celui-ci en tant que valeur de la propriété **linkedServiceName**. Les propriétés suivantes sont prises en charge dans la section **typeProperties** lorsque vous définissez le type d’activité sur AzureMLUpdateResource :

Propriété | Description | Obligatoire
-------- | ----------- | --------
trainedModelName | Nom du modèle reformé. | Oui |
trainedModelDatasetName | Jeu de données pointant vers le fichier iLearner renvoyé par l’opération de reformation. | Oui |

### <a name="json-example"></a>Exemple JSON
Le pipeline a deux activités : **AzureMLBatchExecution** et **AzureMLUpdateResource**. L’activité d’exécution par lot Studio (classique) prend les données d’apprentissage comme entrée et génère un fichier .iLearner comme sortie. L’activité appelle le service web de formation (expérience de formation exposée comme un service web) avec les données de formation d’entrée et reçoit le fichier iLearner du service web. placeholderBlob est simplement un jeu de données de sortie factice requis par le service Azure Data Factory pour exécuter le pipeline.


```json
{
    "name": "pipeline",
    "properties": {
        "activities": [
            {
                "name": "retraining",
                "type": "AzureMLBatchExecution",
                "inputs": [
                    {
                        "name": "trainingData"
                    }
                ],
                "outputs": [
                    {
                        "name": "trainedModelBlob"
                    }
                ],
                "typeProperties": {
                    "webServiceInput": "trainingData",
                    "webServiceOutputs": {
                        "output1": "trainedModelBlob"
                    }
                 },
                "linkedServiceName": "trainingEndpoint",
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1,
                    "timeout": "02:00:00"
                }
            },
            {
                "type": "AzureMLUpdateResource",
                "typeProperties": {
                    "trainedModelName": "trained model",
                    "trainedModelDatasetName" :  "trainedModelBlob"
                },
                "inputs": [{ "name": "trainedModelBlob" }],
                "outputs": [{ "name": "placeholderBlob" }],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "retry": 3
                },
                "name": "AzureML Update Resource",
                "linkedServiceName": "updatableScoringEndpoint2"
            }
        ],
        "start": "2016-02-13T00:00:00",
        "end": "2016-02-14T00:00:00"
    }
}
```

## <a name="data-lake-analytics-u-sql-activity"></a>Activité U-SQL Data Lake Analytics
Vous pouvez spécifier les propriétés suivantes dans une définition JSON d’activité U-SQL. La propriété de type de l’activité doit être : **DataLakeAnalyticsU-SQL**. Vous devez créer un service lié Azure Data Lake Analytics et spécifier le nom de celui-ci en tant que valeur de la propriété **linkedServiceName**. Les propriétés suivantes sont prises en charge dans la section **typeProperties** lorsque vous définissez le type d’activité sur DataLakeAnalyticsU-SQL :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| scriptPath |Chemin d'accès au dossier qui contient le script SQL-U. Le nom de fichier respecte la casse. |Non (si vous utilisez le script) |
| scriptLinkedService |Service lié qui lie le stockage qui contient le script à la fabrique de données |Non (si vous utilisez le script) |
| script |Spécifiez un script en ligne au lieu de spécifier scriptPath et scriptLinkedService. Par exemple : « script » : « CREATE DATABASE test ». |Non (si vous utilisez scriptPath et scriptLinkedService) |
| degreeOfParallelism |Le nombre maximal de nœuds utilisés simultanément pour exécuter le travail. |Non |
| priority |Détermine les travaux parmi tous ceux qui sont en file d'attente qui doivent être sélectionnés pour s'exécuter en premier. Plus le numéro est faible, plus la priorité est élevée. |Non |
| parameters |Paramètres du script U-SQL |Non |

### <a name="json-example"></a>Exemple JSON

```json
{
    "name": "ComputeEventsByRegionPipeline",
    "properties": {
        "description": "This pipeline computes events for en-gb locale and date less than Feb 19, 2012.",
        "activities":
        [
            {
                "type": "DataLakeAnalyticsU-SQL",
                "typeProperties": {
                    "scriptPath": "scripts\\kona\\SearchLogProcessing.txt",
                    "scriptLinkedService": "StorageLinkedService",
                    "degreeOfParallelism": 3,
                    "priority": 100,
                    "parameters": {
                        "in": "/datalake/input/SearchLog.tsv",
                        "out": "/datalake/output/Result.tsv"
                    }
                },
                "inputs": [
                    {
                        "name": "DataLakeTable"
                    }
                ],
                "outputs":
                [
                    {
                        "name": "EventsByRegionTable"
                    }
                ],
                "policy": {
                    "timeout": "06:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "EventsByRegion",
                "linkedServiceName": "AzureDataLakeAnalyticsLinkedService"
            }
        ],
        "start": "2015-08-08T00:00:00",
        "end": "2015-08-08T01:00:00",
        "isPaused": false
    }
}
```

Pour plus d’informations, consultez [Activité U-SQL Data Lake Analytics](data-factory-usql-activity.md).

## <a name="stored-procedure-activity"></a>Activité de procédure stockée
Vous pouvez spécifier les propriétés suivantes dans une définition JSON d’activité de procédure stockée. La propriété de type de l’activité doit être : **SqlServerStoredProcedure**. Vous devez créer un des services liés suivants et spécifier le nom de celui-ci en tant que valeur de la propriété **linkedServiceName** :

- SQL Server
- Azure SQL Database
- Azure Synapse Analytics

Les propriétés suivantes sont prises en charge dans la section **typeProperties** lorsque vous définissez le type d’activité sur SqlServerStoredProcedure :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| storedProcedureName |Spécifiez le nom de la procédure stockée dans Azure SQL Database ou Azure Synapse Analytics qui est représentée par le service lié utilisé par la table de sortie. |Oui |
| storedProcedureParameters |Spécifiez les valeurs des paramètres de procédure stockée. Si vous avez besoin de passer null pour un paramètre, utilisez la syntaxe : "param1": null (le tout en minuscules). Consultez l’exemple suivant pour en savoir plus sur l’utilisation de cette propriété. |Non |

Si vous spécifiez un jeu de données d’entrée, il doit être disponible (à l’état Prêt) pour l’activité de procédure stockée à exécuter. Les jeux de données d’entrée ne peuvent pas être utilisés dans la procédure stockée en tant que paramètres. Cela sert uniquement à vérifier la dépendance avant de commencer l’activité de procédure stockée. Vous devez spécifier un jeu de données de sortie pour une activité de procédure stockée.

Le jeu de données de sortie spécifie la **planification** pour l’activité de procédure stockée (horaire, hebdomadaire, mensuelle, etc.). Le jeu de données de sortie doit utiliser un **service lié** qui fait référence à une base de données Azure SQL Database, Azure Synapse Analytics ou SQL Server dans laquelle vous souhaitez que la procédure stockée soit exécutée. Le jeu de données de sortie peut être un moyen de passer le résultat de la procédure stockée pour traitement ultérieur par une autre activité ([chaînage des activités](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)) dans le pipeline. Toutefois, Data Factory n’écrit pas automatiquement la sortie d’une procédure stockée pour ce jeu de données. C’est la procédure stockée qui écrit dans une table SQL vers laquelle le jeu de données de sortie pointe. Dans certains cas, le jeu de données de sortie peut être un **jeu de données factice**, qui est utilisé uniquement pour spécifier le calendrier d’exécution de l’activité de procédure stockée.

### <a name="json-example"></a>Exemple JSON

```json
{
    "name": "SprocActivitySamplePipeline",
    "properties": {
        "activities": [
            {
                "type": "SqlServerStoredProcedure",
                "typeProperties": {
                    "storedProcedureName": "usp_sample",
                    "storedProcedureParameters": {
                        "DateTime": "$$Text.Format('{0:yyyy-MM-dd HH:mm:ss}', SliceStart)"
                    }
                },
                "outputs": [{ "name": "sprocsampleout" }],
                "name": "SprocActivitySample"
            }
        ],
         "start": "2016-08-02T00:00:00",
         "end": "2016-08-02T05:00:00",
        "isPaused": false
    }
}
```

Pour plus de détails, consultez l’article [Stored Procedure Activity (activité de procédure stockée)](data-factory-stored-proc-activity.md).

## <a name="net-custom-activity"></a>Activité personnalisée .NET
Vous pouvez spécifier les propriétés suivantes dans une définition JSON d’activité personnalisée .NET. La propriété de type de l’activité doit être : **DotNetActivity**. Vous devez créer un service lié Azure HDInsight ou Azure Batch et spécifier le nom de celui-ci en tant que valeur de la propriété **linkedServiceName**. Les propriétés suivantes sont prises en charge dans la section **typeProperties** lorsque vous définissez le type d’activité sur DotNetActivity :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| AssemblyName | Nom de l’assembly. Dans cet exemple, il s’agit de : **MyDotnetActivity.dll**. | Oui |
| EntryPoint |Nom de la classe qui implémente l’interface IDotNetActivity. Dans cet exemple, il s’agit de : **MyDotNetActivityNS.MyDotNetActivity** où MyDotNetActivityNS est l’espace de noms et MyDotNetActivity est la classe.  | Oui |
| PackageLinkedService | Nom du service lié Stockage Azure qui pointe vers le stockage d’objets blob contenant le fichier .zip de l’activité personnalisée. Dans cet exemple, il s’agit de : **AzureStorageLinkedService**.| Oui |
| PackageFile | Nom du fichier zip. Dans l’exemple, il s’agit de : **customactivitycontainer/MyDotNetActivity.zip**. | Oui |
| extendedProperties | Propriétés étendues que vous pouvez définir et transmettre au code .NET. Dans cet exemple, la variable **SliceStart** est définie sur une valeur basée sur la variable système SliceStart. | Non |

### <a name="json-example"></a>Exemple JSON

```json
{
  "name": "ADFTutorialPipelineCustom",
  "properties": {
    "description": "Use custom activity",
    "activities": [
      {
        "Name": "MyDotNetActivity",
        "Type": "DotNetActivity",
        "Inputs": [
          {
            "Name": "InputDataset"
          }
        ],
        "Outputs": [
          {
            "Name": "OutputDataset"
          }
        ],
        "LinkedServiceName": "AzureBatchLinkedService",
        "typeProperties": {
          "AssemblyName": "MyDotNetActivity.dll",
          "EntryPoint": "MyDotNetActivityNS.MyDotNetActivity",
          "PackageLinkedService": "AzureStorageLinkedService",
          "PackageFile": "customactivitycontainer/MyDotNetActivity.zip",
          "extendedProperties": {
            "SliceStart": "$$Text.Format('{0:yyyyMMddHH-mm}', Time.AddMinutes(SliceStart, 0))"
          }
        },
        "Policy": {
          "Concurrency": 2,
          "ExecutionPriorityOrder": "OldestFirst",
          "Retry": 3,
          "Timeout": "00:30:00",
          "Delay": "00:00:00"
        }
      }
    ],
    "start": "2016-11-16T00:00:00",
    "end": "2016-11-16T05:00:00",
    "isPaused": false
  }
}
```

Pour plus d’informations, consultez l’article [Use custom activities in Data Factory (utiliser des activités personnalisées dans Azure Data Factory)](data-factory-use-custom-activities.md).

## <a name="next-steps"></a>Étapes suivantes
Consultez les didacticiels suivants :

- [Didacticiel : créer un pipeline avec une activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
- [Didacticiel : créer un pipeline avec une activité Hive](data-factory-build-your-first-pipeline.md)