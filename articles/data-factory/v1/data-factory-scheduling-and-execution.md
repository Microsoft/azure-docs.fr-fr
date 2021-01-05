---
title: Planification et exécution avec Data Factory
description: Apprenez à connaître les aspects de planification et d’exécution du modèle d’application Azure Data Factory.
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
ms.openlocfilehash: 60410eb2a9a5f18abf2daf87646943ffdc944402
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96495172"
---
# <a name="data-factory-scheduling-and-execution"></a>Planification et exécution avec Data Factory
> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez l’article [Exécution et déclencheurs du pipeline dans Azure Data Factory](../concepts-pipeline-execution-triggers.md).

Cet article explique les aspects de la planification et de l’exécution du modèle d’application Azure Data Factory. Cet article suppose que vous avez des notions de base sur les concepts de modèle de données Data Factory, dont l’activité, les pipelines, les services connexes et les groupes de données. Pour les concepts de base d’Azure Data Factory, consultez les articles suivants :

* [Présentation de Data Factory](data-factory-introduction.md)
* [Pipelines](data-factory-create-pipelines.md)
* [Groupes de données](data-factory-create-datasets.md) 

## <a name="start-and-end-times-of-pipeline"></a>Heures de début et de fin de pipeline
Un pipeline est actif uniquement entre son heure de **début** et son heure de **fin**. Il n'est pas exécuté avant l'heure de début, ni après l'heure de fin. Lorsque le pipeline est suspendu, il n’est pas exécuté, quelle que soit son heure de début et de fin. Pour qu'un pipeline soit exécuté, il ne doit pas être suspendu. Vous trouvez ces paramètres (début, fin, suspendu) dans la définition du pipeline : 

```json
"start": "2017-04-01T08:00:00Z",
"end": "2017-04-01T11:00:00Z"
"isPaused": false
```

Pour plus d’informations sur ces propriétés, consultez l’article [Créer des pipelines](data-factory-create-pipelines.md). 


## <a name="specify-schedule-for-an-activity"></a>Spécifier la planification d’une activité
Ce n’est pas le pipeline qui est exécuté. Ce sont les activités dans le pipeline qui sont exécutées dans le contexte global du pipeline. Vous pouvez planifier l’activité pour qu’elle s’exécute de façon récurrente grâce à la section **scheduler** (planificateur) de l’activité JSON. Par exemple, vous pouvez planifier l’exécution d’une activité toutes les heures comme suit :  

```json
"scheduler": {
    "frequency": "Hour",
    "interval": 1
},
```

Comme illustré dans le diagramme suivant, la définition d’une planification pour une activité crée une série de fenêtres récurrentes dans les heures de début et de fin du pipeline. Les fenêtres récurrentes sont une série d’intervalles de temps fixes contigus, qui ne se chevauchent pas. Ces fenêtres récurrentes logiques pour une activité sont appelées des **fenêtres d’activité**.

![Exemple de planificateur d’activité](media/data-factory-scheduling-and-execution/scheduler-example.png)

La propriété **scheduler** (planificateur) d’une activité est facultative. Si vous définissez cette propriété, elle doit correspondre à la cadence que vous spécifiez dans la définition du jeu de données de sortie pour l’activité. Le jeu de données de sortie pilote actuellement la planification. Vous devez donc créer un jeu de données de sortie même si l’activité ne génère aucune sortie. 

## <a name="specify-schedule-for-a-dataset"></a>Spécifier la planification d’un jeu de données
Une activité dans un pipeline Data Factory peut inclure zéro ou plusieurs **jeux de données** d’entrée et produire un ou plusieurs jeux de données de sortie. Pour une activité, vous pouvez spécifier la cadence à laquelle les données d’entrée sont disponibles ou les données de sortie sont produites à l’aide de la section **availability** (disponibilité) dans les définitions de jeu de données. 

La **Fréquence** dans la section **availability** (disponibilité) spécifie l’unité de temps. Les valeurs de fréquence autorisées sont : Minute, Hour, Day, Week et Month. La propriété **interval** (intervalle) dans la section availability (disponibilité) spécifie un multiplicateur de fréquence. Par exemple : si la fréquence est définie sur Jour et l’intervalle sur 1 pour un jeu de données de sortie, les données de sortie sont produites chaque jour. Si vous définissez la fréquence en minutes, nous vous recommandons de définir l’intervalle sur une valeur au moins égale à 15. 

Dans l’exemple suivant, les données d’entrée seront disponibles toutes les heures et les données de sortie sont produites toutes les heures (`"frequency": "Hour", "interval": 1`). 

**Jeu de données d'entrée :** 

```json
{
    "name": "AzureSqlInput",
    "properties": {
        "published": false,
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```


**Jeu de données de sortie**

```json
{
    "name": "AzureBlobOutput",
    "properties": {
        "published": false,
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "mypath/{Year}/{Month}/{Day}/{Hour}",
            "format": {
                "type": "TextFormat"
            },
            "partitionedBy": [
                { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
                { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
                { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
                { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "HH" }}
            ]
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

Le **jeu de données de sortie pilote actuellement la planification**. En d’autres termes, la planification spécifiée pour le jeu de données de sortie est utilisée pour exécuter une activité lors de l’exécution. Vous devez donc créer un jeu de données de sortie même si l’activité ne génère aucune sortie. Si l’activité ne prend aucune entrée, vous pouvez ignorer la création du jeu de données d’entrée. 

Dans la définition de pipeline suivante, la propriété **scheduler** (planificateur) est utilisée pour spécifier la planification de l’activité. Cette propriété est facultative. La planification de l’activité doit actuellement correspondre à la planification spécifiée pour le jeu de données de sortie.
 
```json
{
    "name": "SamplePipeline",
    "properties": {
        "description": "copy activity",
        "activities": [
            {
                "type": "Copy",
                "name": "AzureSQLtoBlob",
                "description": "copy activity",
                "typeProperties": {
                    "source": {
                        "type": "SqlSource",
                        "sqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 100000,
                        "writeBatchTimeout": "00:05:00"
                    }
                },
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
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                }
            }
        ],
        "start": "2017-04-01T08:00:00Z",
        "end": "2017-04-01T11:00:00Z"
    }
}
```

Dans cet exemple, l’activité s’exécute toutes les heures entre les heures de début et de fin du pipeline. Les données de sortie sont produites toutes les heures pour des trois fenêtres de temps (8h-9h, 9h-10h et 10h-11h). 

Chaque unité de données consommée ou produite pendant l’exécution d’une activité est appelée **tranche de données**. Le diagramme suivant illustre un exemple d’une activité avec un jeu de données d’entrée et un jeu de données de sortie : 

![Planificateur de disponibilité](./media/data-factory-scheduling-and-execution/availability-scheduler.png)

Les tranches de données recueillies toutes les heures pour le jeu de données d’entrée et de sortie sont affichées dans le diagramme. Le diagramme illustre trois tranches d’entrée qui sont prêtes pour le traitement. L’activité de 10 à 11 h est en cours, et produit la tranche de sortie de 10 à 11 h. 

Vous pouvez accéder à l’intervalle de temps associé à la tranche actuelle dans le jeu de données JSON à l’aide des variables [SliceStart](data-factory-functions-variables.md#data-factory-system-variables) et [SliceEnd](data-factory-functions-variables.md#data-factory-system-variables). De même, vous pouvez accéder à l’intervalle de temps associé à une fenêtre d’activité à l’aide des variables WindowStart et WindowEnd. La planification d’une activité doit correspondre à la planification du jeu de données de sortie pour l’activité. Par conséquent, les valeurs SliceStart et SliceEnd sont identiques aux valeurs WindowStart et WindowEnd, respectivement. Pour plus d’informations sur ces variables, consultez les articles [Variables système et fonctions Data Factory](data-factory-functions-variables.md#data-factory-system-variables).  

Vous pouvez utiliser ces variables à différentes fins dans votre activité JSON. Par exemple, vous pouvez les utiliser pour sélectionner les données à partir des jeux de données d’entrée et de sortie représentant les données de série chronologique (par exemple : entre 8h et 9h). Cet exemple utilise également **WindowStart** et **WindowEnd** pour sélectionner les données pertinentes pour l’exécution d’une activité et les copier sur un objet Blob avec le bon **folderPath**. **folderPath** est paramétré pour avoir un dossier distinct pour chaque heure.  

Dans l’exemple précédent, la planification spécifiée pour les jeux de données d’entrée et de sortie est la même (toutes les heures). Si le jeu de données d’entrée de l’activité est disponible à une fréquence différente, par exemple toutes les 15 minutes, l’activité qui produit ce jeu de données de sortie est toujours exécutée une fois par heure car le jeu de données de sortie pilote la planification de l’activité. Pour plus d’informations, consultez [Modélisation des jeux de données avec des fréquences différentes](#model-datasets-with-different-frequencies).

## <a name="dataset-availability-and-policies"></a>Disponibilité et stratégies du jeu de données
Vous avez vu l’utilisation des propriétés de fréquence et d’intervalle de la section availability (disponibilité) de la définition du jeu de données. D’autres propriétés affectent la planification et l’exécution d’une activité. 

### <a name="dataset-availability"></a>Disponibilité du jeu de données 
Le tableau suivant décrit les propriétés que vous pouvez utiliser dans la section **availability** :

| Propriété | Description | Obligatoire | Default |
| --- | --- | --- | --- |
| frequency |Spécifie l’unité de temps pour la production du segment du jeu de données.<br/><br/><b>Fréquence prise en charge</b> : Minute, Hour, Day, Week, Month |Oui |N/D |
| interval |Spécifie un multiplicateur de fréquence<br/><br/>«Frequency» et «interval» déterminent la fréquence à laquelle la tranche est produite.<br/><br/>Si vous voulez des tranches de jeu de données d’une heure, définissez <b>frequency</b> sur <b>Hour</b> et <b>interval</b> sur <b>1</b>.<br/><br/><b>Remarque</b> : Si vous définissez la fréquence en minutes, nous vous recommandons de définir l’intervalle sur une valeur au moins égale à 15. |Oui |N/D |
| style |Spécifie si le segment doit être généré au début / à la fin de l’intervalle.<ul><li>StartOfInterval</li><li>EndOfInterval</li></ul><br/><br/>Si la fréquence est définie sur Month et le style défini sur EndOfInterval, le segment est généré le dernier jour du mois. Si le style est défini sur StartOfInterval, le segment est généré le premier jour du mois.<br/><br/>Si la fréquence est définie sur Day et le style défini sur EndOfInterval, le segment est généré la dernière heure du jour.<br/><br/>Si la fréquence est définie sur Hour et le style défini sur EndOfInterval, le segment est généré à la fin de l’heure. Par exemple, pour un segment de la période 13 h-14 h, le segment est généré à 14 h. |Non |EndOfInterval |
| anchorDateTime |Définit la position absolue dans le temps utilisée par le planificateur pour calculer les limites de tranche de jeu de données. <br/><br/><b>Remarque</b> : Si AnchorDateTime contient des éléments de date plus précis que la fréquence, ces éléments plus précis sont ignorés. <br/><br/>Par exemple, si <b>interval</b> est <b>hourly</b> (frequency: hour et interval: 1), et que <b>AnchorDateTime</b> contient <b>minutes and seconds</b>, les éléments <b>minutes et seconds</b> d’AnchorDateTime sont ignorés. |Non |01/01/0001 |
| offset |Intervalle de temps marquant le déplacement du début et de la fin de toutes les tranches du jeu de données. <br/><br/><b>Remarque</b> : Si anchorDateTime et offset sont spécifiés, on obtient un décalage combiné. |Non |N/D |

### <a name="offset-example"></a>exemple offset
Par défaut, les tranches quotidiennes (`"frequency": "Day", "interval": 1`) commencent à 0 h UTC (minuit). Si vous souhaitez que l’heure de début soit 6 h UTC, définissez le décalage comme indiqué dans l’extrait suivant : 

```json
"availability":
{
    "frequency": "Day",
    "interval": 1,
    "offset": "06:00:00"
}
```
### <a name="anchordatetime-example"></a>Exemple anchorDateTime
Dans l’exemple suivant, le jeu de données est généré toutes les 23 heures. La première tranche commence à l’heure spécifiée par anchorDateTime, qui est défini sur `2017-04-19T08:00:00` (heure UTC).

```json
"availability":    
{    
    "frequency": "Hour",        
    "interval": 23,    
    "anchorDateTime":"2017-04-19T08:00:00"    
}
```

### <a name="offsetstyle-example"></a>Exemple de décalage/style
Le jeu de données suivant est un jeu de données mensuel et est généré le 3 de chaque mois à 8h00 (`3.08:00:00`) :

```json
"availability": {
    "frequency": "Month",
    "interval": 1,
    "offset": "3.08:00:00", 
    "style": "StartOfInterval"
}
```

### <a name="dataset-policy"></a>Stratégie du jeu de données
Un jeu de données peut avoir une stratégie de validation définie qui spécifie comment les données générées par l’exécution d’une tranche peuvent être validées avant qu’il soit prêt à la consommation. Dans ce cas, une fois que la tranche a terminé l’exécution, l’état de la tranche de sortie devient **En attente** avec un sous-état **Validation**. Une fois les tranches validées, l’état de la tranche passe à **prêt**. Si une tranche de données a été générée mais n’a pas réussi la validation, l’activité s’exécute pour les tranches en aval dépendant de cette tranche qui ne sont pas traitées. [Surveiller et gérer les pipelines](data-factory-monitor-manage-pipelines.md) .

La section **policy** de la définition du jeu de données définit les critères ou la condition que les segments du jeu de données doivent remplir. Le tableau suivant décrit les propriétés que vous pouvez utiliser dans la section **policy** (stratégie) :

| Nom de la stratégie | Description | Appliqué(e) à | Obligatoire | Default |
| --- | --- | --- | --- | --- |
| minimumSizeMB | Valide le fait que les données dans un **objet blob Azure** répondent aux exigences de taille minimale (en mégaoctets). |Objets blob Azure |Non |N/D |
| minimumRows | Valide le fait que les données dans **Azure SQL Database** ou une **table Azure** contiennent le nombre minimal de lignes. |<ul><li>Azure SQL Database</li><li>table Azure</li></ul> |Non |N/D |

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

**minimumRows**

```json
"policy":
{
    "validation":
    {
        "minimumRows": 100
    }
}
```

Pour plus d’informations sur ces propriétés et exemples, consultez l’article [Créer des jeux de données](data-factory-create-datasets.md). 

## <a name="activity-policies"></a>Stratégies d’activité
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

Pour plus d’informations, consultez l’article [Pipelines](data-factory-create-pipelines.md). 

## <a name="parallel-processing-of-data-slices"></a>Traitement en parallèle des tranches de données
Vous pouvez définir la date de début du pipeline dans le passé. Lorsque vous procédez ainsi, Data Factory calcule (remplit postérieurement) automatiquement toutes les tranches de données dans le passé automatiquement et commence à les traiter. Par exemple : si vous créez un pipeline avec la date de début 2017-04-01 et que la date actuelle est 2017-04-10. Si la cadence du jeu de données de sortie est tous les jours, Data Factory commence immédiatement le traitement de toutes les tranches entre le 2017-04-01 et le 2017-04-09, car la date de début se situe dans le passé. La tranche du 2017-04-10 n’est pas encore traitée, car la valeur de la propriété style dans la section availability (disponibilité) est EndOfInterval par défaut. La tranche la plus ancienne est traitée en premier, car la valeur par défaut de executionPriorityOrder est OldestFirst. Pour obtenir une description de la propriété style, consultez la section [Disponibilité du jeu de données](#dataset-availability). Pour obtenir une description de la section executionPriorityOrder, consultez la section [Stratégies d’activité](#activity-policies). 

Vous pouvez configurer des tranches de données pour qu’elles soient traitées en parallèle en définissant la propriété **concurrency** (concurrence) dans la section **policy** (stratégie) de l’activité JSON. Cette propriété détermine le nombre d’exécutions en parallèle de l’activité qui peuvent se produire sur différents segments. La valeur par défaut de la propriété de concurrence est 1. Une tranche est donc traitée à la fois par défaut. La valeur maximale est 10. Lorsqu’un pipeline doit passer par un grand ensemble de données disponibles, une valeur de concurrence plus élevée accélère le traitement des données. 

## <a name="rerun-a-failed-data-slice"></a>Réexécuter une tranche de données ayant échoué
Lorsqu’une erreur se produit pendant le traitement d’une tranche de données, vous pouvez savoir pourquoi le traitement d’une tranche a échoué à l’aide de panneaux du portail Azure ou de l’application Surveiller et gérer. Pour plus d’informations, consultez [Surveillance et gestion des pipelines à l’aide des panneaux du portail Azure](data-factory-monitor-manage-pipelines.md) ou de [l’application Surveillance et gestion](data-factory-monitor-manage-app.md).

Prenons l’exemple suivant, il montre les deux activités. Activity1 et Activity2. Activity1 utilise une tranche de Dataset1 et génère une tranche de Dataset2, qui est utilisé en entrée par Activity2 pour produire une tranche du jeu de données final.

![Tranche de données ayant échoué](./media/data-factory-scheduling-and-execution/failed-slice.png)

Le diagramme montre que, parmi les trois tranches récentes, il y a eu un échec, ce qui a généré une tranche 9 à 10 h pour Dataset2. Data Factory effectue automatiquement le suivi de la dépendance du jeu de données. Par conséquent, il ne lance pas l’activité sur la tranche 9 à 10 h en aval.

Les outils de supervision et de gestion Data Factory vous permettent d’examiner en détail les journaux de diagnostic pour la tranche ayant échoué, et de trouver facilement la cause du problème pour le régler. Une fois le problème résolu, vous pouvez facilement lancer l’exécution de l’activité afin de générer la tranche ayant échoué. Pour plus d’informations sur la façon de lancer les réexécutions et comprendre les transitions d’état des tranches de données, consultez [Surveillance et gestion des pipelines à l’aide des panneaux du portail Azure](data-factory-monitor-manage-pipelines.md) ou [Application de surveillance et gestion](data-factory-monitor-manage-app.md).

Une fois que vous avez relancé l’exécution de la tranche de 9-10 h pour **Dataset2**, Data Factory lance l’exécution de la tranche dépendante 9 à 10 h sur un jeu de données final.

![Réexécuter une tranche de données ayant échoué](./media/data-factory-scheduling-and-execution/rerun-failed-slice.png)

## <a name="multiple-activities-in-a-pipeline"></a>Plusieurs activités à l’intérieur d’un pipeline
Un pipeline peut toutefois contenir plusieurs activités. Si vous avez plusieurs activités dans un pipeline et que la sortie d’une activité n’est pas une entrée dans une autre activité, les activités peuvent s’exécuter en parallèle si les tranches de données d’entrée pour les activités sont prêtes.

Vous pouvez chaîner deux activités (une après l’autre) en configurant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. Les activités peuvent être dans le même pipeline ou dans des pipelines différents. La seconde activité s’exécute uniquement quand la première se termine correctement.

Par exemple, considérez le cas suivant, dans lequel un pipeline a deux activités :

1. L’activité A1 nécessitant le jeu de données d’entrée externe D1 et qui produit le jeu de données de sortie D2.
2. L’activité A2 nécessitant le jeu de données d’entrée D2 et qui produit le jeu de données de sortie D3.

Dans ce scénario, les activités A1 et A2 sont dans le même pipeline. L’activité A1 s’exécute lorsque les données externes seront disponibles et que la fréquence de disponibilité planifiée sera atteinte. L’activité A2 s’exécute lorsque les tranches planifiées de D2 seront disponibles et que la fréquence de disponibilité planifiée sera atteinte. S’il existe une erreur dans l’une des tranches du jeu de données D2, A2 n’est pas exécutée pour cette tranche jusqu’à ce que celle-ci devienne disponible.

La vue de diagramme avec les deux activités dans le même pipeline se présente comme dans le diagramme suivant :

![Chaînage des activités dans le même pipeline](./media/data-factory-scheduling-and-execution/chaining-one-pipeline.png)

Comme mentionné plus tôt, les activités peuvent être dans des pipelines différents. Dans ce scénario, la vue de diagramme se présente comme dans le diagramme suivant :

![Chaînage des activités dans deux pipelines](./media/data-factory-scheduling-and-execution/chaining-two-pipelines.png)

Consultez la section Copier de manière séquentielle de l'annexe pour obtenir un exemple.

## <a name="model-datasets-with-different-frequencies"></a>Modélisation des jeux de données avec des fréquences différentes
Dans les exemples, les fréquences de planification des jeux de données d’entrée et de sortie et l’intervalle d’activité sont les mêmes. Certains scénarios exigent que la fréquence de génération d’une sortie à soit différente de celles d’une ou de plusieurs entrées. Data factory prend en charge la modélisation de ces scénarios.

### <a name="sample-1-produce-a-daily-output-report-for-input-data-that-is-available-every-hour"></a>Exemple 1 : production d’un rapport de sortie quotidien pour des données d’entrée disponibles toutes les heures
Imaginez un scénario dans lequel nous avons entré des données de mesure à partir de capteurs disponibles toutes les heures dans un stockage Azure Blob. Vous voulez générer un rapport d’agrégation quotidien avec des statistiques, comme les valeurs moyenne, maximum et minimum pour la journée avec une [Activité Hive Data Factory](data-factory-hive-activity.md).

Voici ce que vous pouvez modéliser ce scénario avec data factory :

**Jeu de données d'entrée**

Les fichiers d’entrée des heures sont supprimés dans le dossier pour le jour donné. La disponibilité de l’entrée est définie sur **Hour** (frequency: Hour, interval: 1).

```json
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
      "partitionedBy": [
        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "MM"}},
        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "dd"}}
      ],
      "format": {
        "type": "TextFormat"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
**Jeu de données de sortie**

Un fichier de sortie est créé chaque jour dans le dossier pour la journée. La disponibilité de sortie est définie sur **Day** (frequency: Day et interval: 1).

```json
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
      "partitionedBy": [
        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "MM"}},
        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "dd"}}
      ],
      "format": {
        "type": "TextFormat"
      }
    },
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```

**Activité : activité Hive dans un pipeline**

Le script Hive reçoit les informations de *DateTime* en tant que paramètres qui utilisent la variable **WindowStart** comme indiqué dans l’extrait de code suivant. Le script Hive utilise cette variable pour charger les données à partir du dossier correspondant à la journée et exécuter l’agrégation pour générer la sortie.

```json
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2015-01-01T08:00:00",
    "end":"2015-01-01T11:00:00",
    "description":"hive activity",
    "activities": [
        {
            "name": "SampleHiveActivity",
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
            "linkedServiceName": "HDInsightLinkedService",
            "type": "HDInsightHive",
            "typeProperties": {
                "scriptPath": "adftutorial\\hivequery.hql",
                "scriptLinkedService": "StorageLinkedService",
                "defines": {
                    "Year": "$$Text.Format('{0:yyyy}',WindowStart)",
                    "Month": "$$Text.Format('{0:MM}',WindowStart)",
                    "Day": "$$Text.Format('{0:dd}',WindowStart)"
                }
            },
            "scheduler": {
                "frequency": "Day",
                "interval": 1
            },            
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 2,
                "timeout": "01:00:00"
            }
         }
     ]
   }
}
```

Le diagramme suivant illustre le scénario du point de vue de la dépendance des données.

![Dépendance des données](./media/data-factory-scheduling-and-execution/data-dependency.png)

La tranche de sortie dépend des 24 tranches horaires depuis l’ensemble de données en entrée. Data Factory calcule automatiquement ces dépendances en déterminant les tranches de données d’entrée qui tombent dans la même période que la tranche de données à générer. Si une des 24 tranches d’entrée n’est pas disponible, Data Factory attend que la tranche d’entrée soit prête avant de lancer l’exécution d’activité quotidienne.

### <a name="sample-2-specify-dependency-with-expressions-and-data-factory-functions"></a>Exemple 2 : spécifier les dépendances avec des expressions et des fonctions Data Factory
Prenons en compte un autre scénario. Supposez que vous avez une activité Hive qui traite deux jeux de données d’entrée. Un d’eux a de nouvelles données tous les jours, mais l’autre obtient de nouvelles données toutes les semaines. Supposons que vous vouliez faire la jonction entre les deux entrées et générer une sortie quotidiennement.

L’approche simple consistant pour Data Factory à déterminer des tranches d’entrée appropriées à traiter en alignant la période de temps de la tranche de données en sortie ne fonctionne plus.

Vous devez spécifier (pour chaque exécution d’activité que Data Factory doit utiliser) les tranches de données de la semaine précédente pour les jeux de données d’entrée. Vous utilisez les fonctions Azure Data Factory comme indiqué dans l’extrait de code suivant pour implémenter ce comportement.

**Entrée1 : objet blob Azure**

La première entrée est l’objet Azure Blob mis à jour quotidiennement.

```json
{
  "name": "AzureBlobInputDaily",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
      "partitionedBy": [
        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "MM"}},
        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "dd"}}
      ],
      "format": {
        "type": "TextFormat"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```

**Entrée2 : objet blob Azure**

Entrée2 est l’objet Azure Blob mis à jour chaque semaine.

```json
{
  "name": "AzureBlobInputWeekly",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
      "partitionedBy": [
        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "MM"}},
        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "dd"}}
      ],
      "format": {
        "type": "TextFormat"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Day",
      "interval": 7
    }
  }
}
```

**Sortie : objet blob Azure**

Un fichier de sortie est créé chaque jour dans le dossier pour la journée. La disponibilité de sortie est définie sur **day** (frequency: Day, interval: 1).

```json
{
  "name": "AzureBlobOutputDaily",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
      "partitionedBy": [
        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "MM"}},
        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "dd"}}
      ],
      "format": {
        "type": "TextFormat"
      }
    },
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```

**Activité : activité Hive dans un pipeline**

L’activité Hive accepte les deux entrées et génère une tranche de sortie tous les jours. Vous pouvez spécifier la tranche de sortie de tous les jours dépendant de la tranche de semaine précédente pour la sortie hebdomadaire comme suit.

```json
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2015-01-01T08:00:00",
    "end":"2015-01-01T11:00:00",
    "description":"hive activity",
    "activities": [
      {
        "name": "SampleHiveActivity",
        "inputs": [
          {
            "name": "AzureBlobInputDaily"
          },
          {
            "name": "AzureBlobInputWeekly",
            "startTime": "Date.AddDays(SliceStart, - Date.DayOfWeek(SliceStart))",
            "endTime": "Date.AddDays(SliceEnd,  -Date.DayOfWeek(SliceEnd))"  
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutputDaily"
          }
        ],
        "linkedServiceName": "HDInsightLinkedService",
        "type": "HDInsightHive",
        "typeProperties": {
          "scriptPath": "adftutorial\\hivequery.hql",
          "scriptLinkedService": "StorageLinkedService",
          "defines": {
            "Year": "$$Text.Format('{0:yyyy}',WindowStart)",
            "Month": "$$Text.Format('{0:MM}',WindowStart)",
            "Day": "$$Text.Format('{0:dd}',WindowStart)"
          }
        },
        "scheduler": {
          "frequency": "Day",
          "interval": 1
        },            
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 2,  
          "timeout": "01:00:00"
        }
       }
     ]
   }
}
```

Pour obtenir la liste des fonctions et variables système prises en charge par Azure Data Factory, consultez [Variables système et fonctions Data Factory](data-factory-functions-variables.md) .

## <a name="appendix"></a>Annexe

### <a name="example-copy-sequentially"></a>Exemple : Copier de manière séquentielle
Il est possible d’exécuter plusieurs opérations de copie l’une après l’autre, de manière séquentielle/ordonnée. Si, par exemple, vous avez deux activités de copie dans un pipeline : (ActivitédeCopie1 et ActivitédeCopie2) avec les jeux de données de sortie de données d’entrée suivants :   

Activitédecopie1

Entrée : JeudeDonnées. Sortie : JeudeDonnées2.

Activitédecopie2

Entrée : JeudeDonnées2.  Sortie : JeudeDonnées3.

ActivitédeCopie2 s’exécute uniquement si ActivitédeCopie1 s’est exécutée avec succès et que JeudeDonnées2 est disponible.

Voici l’exemple de pipeline JSON :

```json
{
    "name": "ChainActivities",
    "properties": {
        "description": "Run activities in sequence",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "copyBehavior": "PreserveHierarchy",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "Dataset1"
                    }
                ],
                "outputs": [
                    {
                        "name": "Dataset2"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00"
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "CopyFromBlob1ToBlob2",
                "description": "Copy data from a blob to another"
            },
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "Dataset2"
                    }
                ],
                "outputs": [
                    {
                        "name": "Dataset3"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00"
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "CopyFromBlob2ToBlob3",
                "description": "Copy data from a blob to another"
            }
        ],
        "start": "2016-08-25T01:00:00Z",
        "end": "2016-08-25T01:00:00Z",
        "isPaused": false
    }
}
```

Notez que dans l’exemple, le jeu de données de sortie de la première activité de copie (Dataset2) est spécifié en tant qu’entrée pour la deuxième activité. Par conséquent, la deuxième activité s’exécute uniquement lorsque le jeu de données de sortie de la première activité est prêt.  

Dans l’exemple, ActivitédeCopie2 peut avoir une entrée différente, par exemple JeudeDonnées3, mais vous spécifiez JeudeDonnées2 en tant qu’entrée pour ActivitédeCopie2, afin que l’activité ne s’exécute pas avant la fin d’ActivitédeCopie1. Par exemple :

Activitédecopie1

Entrée : JeudeDonnées1. Sortie : JeudeDonnées2.

Activitédecopie2

Entrées : JeudeDonnées3, JeudeDonnées2. Sortie : Jeudedonnées4.

```json
{
    "name": "ChainActivities",
    "properties": {
        "description": "Run activities in sequence",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "copyBehavior": "PreserveHierarchy",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "Dataset1"
                    }
                ],
                "outputs": [
                    {
                        "name": "Dataset2"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00"
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "CopyFromBlobToBlob",
                "description": "Copy data from a blob to another"
            },
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "Dataset3"
                    },
                    {
                        "name": "Dataset2"
                    }
                ],
                "outputs": [
                    {
                        "name": "Dataset4"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00"
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "CopyFromBlob3ToBlob4",
                "description": "Copy data from a blob to another"
            }
        ],
        "start": "2017-04-25T01:00:00Z",
        "end": "2017-04-25T01:00:00Z",
        "isPaused": false
    }
}
```

Notez que dans l’exemple, deux jeux de données d’entrée sont spécifiés pour la deuxième activité de copie. Quand plusieurs entrées sont spécifiées, seul le premier jeu de données d’entrée est utilisé pour copier des données, mais les autres jeux de données sont utilisés en tant que dépendances. L’exécution d’ActivitédeCopie2 démarre uniquement quand les conditions suivantes sont remplies :

* ActivitédeCopie1 s’est terminée avec succès et JeudeDonnées2 est disponible. Ce jeu de données n’est pas utilisé lors de la copie des données vers JeudeDonnées4. Il sert uniquement de dépendance de planification pour ActivitédeCopie2.   
* JeudeDonnées3 est disponible. Ce jeu de données représente les données qui sont copiées vers la destination. 
