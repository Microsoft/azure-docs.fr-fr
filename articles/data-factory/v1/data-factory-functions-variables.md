---
title: Variables système et fonctions Data Factory
description: Fournit la liste des variables système et fonctions Azure Data Factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 9b5f91655367e866858a04b941cec4ee61dfe180
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96495648"
---
# <a name="azure-data-factory---functions-and-system-variables"></a>Azure Data Factory - Variables système et fonctions
> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Variables système dans Data Factory](../control-flow-system-variables.md).

Cet article fournit des informations sur les fonctions et variables prises en charge par Azure Data Factory.

## <a name="data-factory-system-variables"></a>Variables système Data Factory

| Nom de la variable | Description | Portée de l’objet | Étendue JSON et cas d’utilisation |
| --- | --- | --- | --- |
| WindowStart |Début de l’intervalle de temps pour l’intervalle d’exécution d’activité en cours |activity |<ol><li>Spécifier des requêtes de sélection de données. Consultez les articles connexes référencés par l’article [Activités de déplacement des données](data-factory-data-movement-activities.md).</li> |
| WindowEnd |Fin de l’intervalle de temps de l’intervalle d’exécution d’activité en cours |activity |identique à WindowStart. |
| SliceStart |Début de l’intervalle de temps pour une tranche de données en cours de génération |activity<br/>dataset |<ol><li>Spécifier les chemins d’accès et les noms de fichiers dynamiques tout en travaillant avec [Blob Azure](data-factory-azure-blob-connector.md) et [Jeux de données du système de fichiers](data-factory-onprem-file-system-connector.md).</li><li>Spécifier les dépendances d’entrée avec les fonctions Data Factory dans la collecte d’activité.</li></ol> |
| SliceEnd |Fin de l’intervalle de temps pour la tranche de données en cours. |activity<br/>dataset |identique à SliceStart. |

> [!NOTE]
> Actuellement, la fabrique de données exige que le calendrier spécifié dans l’activité corresponde exactement à la planification spécifiée dans la disponibilité du jeu de données de sortie. Ainsi, WindowStart, WindowEnd, SliceStart et SliceEnd font toujours correspondre la même période de temps et une tranche de sortie unique.
> 

### <a name="example-for-using-a-system-variable"></a>Exemple d’utilisation d’une variable système
Dans l’exemple suivant, l’année, le mois, le jour et l’heure de **SliceStart** sont extraits dans des variables distinctes qui sont utilisées par les propriétés **folderPath** et **fileName**.

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

## <a name="data-factory-functions"></a>Fonctions Data Factory
Vous pouvez utiliser des fonctions dans Data Factory avec les variables système aux fins suivantes :

1. Spécification de requêtes de sélection de données (consultez les articles connexes référencés par l’article [Activités de déplacement des données](data-factory-data-movement-activities.md) ).
   
   La syntaxe pour appeler une fonction Data Factory est : **$$\<function>** pour les requêtes de sélection de données et d’autres propriétés de l’activité et des jeux de données.  
2. Spécifier les dépendances d’entrée avec les fonctions Data Factory dans la collecte d’entrées d’activité.
   
    $$ n’est pas nécessaire pour spécifier des expressions de dépendance d’entrée.     

Dans l’exemple suivant, la propriété **sqlReaderQuery** d’un fichier JSON est affectée à une valeur renvoyée par la fonction `Text.Format`. Cet exemple utilise également une variable système nommée **WindowStart**, qui représente l’heure de début de la fenêtre d’activité à exécuter.

```json
{
    "Type": "SqlSource",
    "sqlReaderQuery": "$$Text.Format('SELECT * FROM MyTable WHERE StartTime = \\'{0:yyyyMMdd-HH}\\'', WindowStart)"
}
```

Consultez la rubrique [Chaînes de format de date et d’heure personnalisées](/dotnet/standard/base-types/custom-date-and-time-format-strings) , qui décrit les différentes options de formatage que vous pouvez utiliser (par exemple : aa et aaaa). 

### <a name="functions"></a>Fonctions
Les tables qui suivent répertorient toutes les fonctions dans Azure Data Factory :

| Category | Fonction | Paramètres | Description |
| --- | --- | --- | --- |
| Temps |AddHours(X,Y) |X: DateTime <br/><br/>Y: int |Ajoute Y heures à l’heure donnée X. <br/><br/>Exemple : `9/5/2013 12:00:00 PM + 2 hours = 9/5/2013 2:00:00 PM` |
| Temps |AddMinutes(X,Y) |X: DateTime <br/><br/>Y: int |Ajoute Y minutes à X.<br/><br/>Exemple : `9/15/2013 12: 00:00 PM + 15 minutes = 9/15/2013 12: 15:00 PM` |
| Temps |StartOfHour(X) |X: Datetime |Obtient l’heure de début de l’heure représentée par le composant heure de X. <br/><br/>Exemple : `StartOfHour of 9/15/2013 05: 10:23 PM is 9/15/2013 05: 00:00 PM` |
| Date |AddDays(X,Y) |X: DateTime<br/><br/>Y: int |Ajoute Y jours à X. <br/><br/>Exemple : 15/9/2013 12:00:00 + 2 jours = 17/9/2013 12:00:00.<br/><br/>Vous pouvez également soustraire les jours en spécifiant Y en tant que nombre négatif.<br/><br/>Exemple : `9/15/2013 12:00:00 PM - 2 days = 9/13/2013 12:00:00 PM`. |
| Date |AddMonths(X,Y) |X: DateTime<br/><br/>Y: int |Ajoute Y mois à X.<br/><br/>`Example: 9/15/2013 12:00:00 PM + 1 month = 10/15/2013 12:00:00 PM`.<br/><br/>Vous pouvez également soustraire les mois en spécifiant Y en tant que nombre négatif.<br/><br/>Exemple : `9/15/2013 12:00:00 PM - 1 month = 8/15/2013 12:00:00 PM`.|
| Date |AddQuarters(X,Y) |X: DateTime <br/><br/>Y: int |Ajoute Y * 3 mois à X.<br/><br/>Exemple : `9/15/2013 12:00:00 PM + 1 quarter = 12/15/2013 12:00:00 PM` |
| Date |AddWeeks(X,Y) |X: DateTime<br/><br/>Y: int |Ajoute Y * 7 jours à X<br/><br/>Exemple : 15/9/2013 12:00:00 + 1 semaine = 22/9/2013 12:00:00<br/><br/>Vous pouvez également soustraire les semaines en spécifiant Y en tant que nombre négatif.<br/><br/>Exemple : `9/15/2013 12:00:00 PM - 1 week = 9/7/2013 12:00:00 PM`. |
| Date |AddYears(X,Y) |X: DateTime<br/><br/>Y: int |Ajoute Y années à X.<br/><br/>`Example: 9/15/2013 12:00:00 PM + 1 year = 9/15/2014 12:00:00 PM`<br/><br/>Vous pouvez également soustraire les années en spécifiant Y en tant que nombre négatif.<br/><br/>Exemple : `9/15/2013 12:00:00 PM - 1 year = 9/15/2012 12:00:00 PM`. |
| Date |Day(X) |X: DateTime |Obtient le composant jour de X.<br/><br/>Exemple : `Day of 9/15/2013 12:00:00 PM is 9`. |
| Date |DayOfWeek(X) |X: DateTime |Obtient le composant semaine de X.<br/><br/>Exemple : `DayOfWeek of 9/15/2013 12:00:00 PM is Sunday`. |
| Date |DayOfYear(X) |X: DateTime |Permet d’obtenir le jour de l’année représenté par le composant année de X.<br/><br/>Exemples :<br/>`12/1/2015: day 335 of 2015`<br/>`12/31/2015: day 365 of 2015`<br/>`12/31/2016: day 366 of 2016 (Leap Year)` |
| Date |DaysInMonth(X) |X: DateTime |Permet d’obtenir les jours du mois représentés par le composant mois du paramètre X.<br/><br/>Exemple : `DaysInMonth of 9/15/2013 are 30 since there are 30 days in the September month`. |
| Date |EndOfDay(X) |X: DateTime |Obtient la valeur date-heure qui représente la fin de la journée (composant jour) de X.<br/><br/>Exemple : `EndOfDay of 9/15/2013 05:10:23 PM is 9/15/2013 11:59:59 PM`. |
| Date |EndOfMonth(X) |X: DateTime |Permet d’obtenir la fin du mois représentée par le composant mois du paramètre X. <br/><br/>Exemple : `EndOfMonth of 9/15/2013 05:10:23 PM is 9/30/2013 11:59:59 PM` (date-heure qui représente la fin du mois de septembre) |
| Date |StartOfDay(X) |X: DateTime |Permet d’obtenir le début de la journée représenté par le composant jour du paramètre X.<br/><br/>Exemple : `StartOfDay of 9/15/2013 05:10:23 PM is 9/15/2013 12:00:00 AM`. |
| DateTime |From(X) |X: String |Analyser la chaîne X à une heure de date. |
| DateTime |Ticks(X) |X: DateTime |Permet d’obtenir la propriété de graduation du paramètre X. Un cycle est égal à 100 nanosecondes. La valeur de cette propriété représente le nombre de graduations écoulées depuis 12:00:00 minuit, le 1er janvier 0001. |
| Texte |Format(X) |X: variable de chaîne |Met en forme le texte (utilisez la combinaison `\\'` pour échapper le caractère `'`).|

> [!IMPORTANT]
> Lorsque vous utilisez une fonction au sein d’une autre fonction, vous n’avez pas besoin d’utiliser le préfixe **$$** de la fonction interne. Par exemple : $$Text.Format('PartitionKey eq \\'my_pkey_filter_value\\' et RowKey ge \\'{0: yyyy-MM-dd HH:mm:ss}\\'', Time.AddHours(SliceStart, -6)). Dans cet exemple, notez que le préfixe **$$** n’est pas utilisé pour la fonction **Time.AddHours**. 

#### <a name="example"></a>Exemple
Dans l’exemple suivant, les paramètres d’entrée et de sortie de l’activité Hive sont déterminés à l’aide de la fonction `Text.Format` et de la variable système SliceStart. 

```json  
{
    "name": "HiveActivitySamplePipeline",
        "properties": {
    "activities": [
            {
            "name": "HiveActivitySample",
            "type": "HDInsightHive",
            "inputs": [
                    {
                    "name": "HiveSampleIn"
                    }
            ],
            "outputs": [
                    {
                    "name": "HiveSampleOut"
                }
            ],
            "linkedServiceName": "HDInsightLinkedService",
            "typeproperties": {
                    "scriptPath": "adfwalkthrough\\scripts\\samplehive.hql",
                    "scriptLinkedService": "StorageLinkedService",
                    "defines": {
                        "Input": "$$Text.Format('wasb://adfwalkthrough@<storageaccountname>.blob.core.windows.net/samplein/yearno={0:yyyy}/monthno={0:MM}/dayno={0:dd}/', SliceStart)",
                        "Output": "$$Text.Format('wasb://adfwalkthrough@<storageaccountname>.blob.core.windows.net/sampleout/yearno={0:yyyy}/monthno={0:MM}/dayno={0:dd}/', SliceStart)"
                    },
                    "scheduler": {
                        "frequency": "Hour",
                        "interval": 1
                }
            }
            }
    ]
    }
}
```

### <a name="example-2"></a>Exemple 2

Dans l’exemple suivant, le paramètre DateTime de l’activité de procédure stockée est déterminé à l’aide de la fonction Text. Format et de la variable SliceStart. 

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
                "outputs": [
                    {
                        "name": "sprocsampleout"
                    }
                ],
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "SprocActivitySample"
            }
        ],
            "start": "2016-08-02T00:00:00Z",
            "end": "2016-08-02T05:00:00Z",
        "isPaused": false
    }
}
```

### <a name="example-3"></a>Exemple 3
Pour lire les données de la veille au lieu du jour représenté par SliceStart, utilisez la fonction AddDays, comme illustré dans l’exemple suivant : 

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-01-01T08:00:00",
        "end": "2017-01-01T11:00:00",
        "description": "hive activity",
        "activities": [
            {
                "name": "SampleHiveActivity",
                "inputs": [
                    {
                        "name": "MyAzureBlobInput",
                        "startTime": "Date.AddDays(SliceStart, -1)",
                        "endTime": "Date.AddDays(SliceEnd, -1)"
                    }
                ],
                "outputs": [
                    {
                        "name": "MyAzureBlobOutput"
                    }
                ],
                "linkedServiceName": "HDInsightLinkedService",
                "type": "HDInsightHive",
                "typeProperties": {
                    "scriptPath": "adftutorial\\hivequery.hql",
                    "scriptLinkedService": "StorageLinkedService",
                    "defines": {
                        "Year": "$$Text.Format('{0:yyyy}',WindowsStart)",
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

Consultez la rubrique [Chaînes de format de date et d’heure personnalisées](/dotnet/standard/base-types/custom-date-and-time-format-strings) , qui décrit les différentes options de formatage que vous pouvez utiliser (par exemple : aa et aaaa).