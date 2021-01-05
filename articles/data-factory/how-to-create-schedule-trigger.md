---
title: Créer des déclencheurs de planification dans Azure Data Factory
description: Découvrez comment créer un déclencheur dans Azure Data Factory qui exécute un pipeline selon une planification.
services: data-factory
documentationcenter: ''
author: chez-charlie
ms.author: chez
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 10/30/2020
ms.custom: devx-track-python
ms.openlocfilehash: a6f53d6ce41085b2348857ccb5b45c06132d6a99
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001981"
---
# <a name="create-a-trigger-that-runs-a-pipeline-on-a-schedule"></a>Créer un déclencheur qui exécute un pipeline selon une planification
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article fournit des informations sur le déclencheur de planification et les étapes pour créer, démarrer et surveiller un déclencheur de planification. Pour les autres types de déclencheurs, consultez [Exécution du pipeline et déclencheurs](concepts-pipeline-execution-triggers.md).

Lorsque vous créez un déclencheur de planification, vous spécifiez une planification (date de début, périodicité, date de fin, etc.) pour le déclencheur, et l’associez à un pipeline. Les pipelines et les déclencheurs ont une relation plusieurs-à-plusieurs. Plusieurs déclencheurs peuvent exécuter le même pipeline. Un seul déclencheur peut déclencher plusieurs pipelines.

Les sections suivantes indiquent les étapes pour créer un déclencheur de planification de différentes façons. 

## <a name="data-factory-ui"></a>IU de la fabrique de données
Vous pouvez créer un **déclencheur de planification** afin de planifier l’exécution périodique d’un pipeline (toutes les heures, tous les jours, etc.). 

> [!NOTE]
> Pour la procédure complète de création d’un pipeline et d’un déclencheur de planification, qui associe le déclencheur au pipeline et exécute et surveille le pipeline, consultez [Démarrage rapide : créer une fabrique de données à l’aide de l’interface utilisateur de Data Factory](quickstart-create-data-factory-portal.md).

1. Basculez sur l’onglet **Modifier**, indiqué par un symbole de crayon. 

    ![Basculer vers l’onglet Modifier](./media/how-to-create-schedule-trigger/switch-edit-tab.png)

1. Sélectionnez **Déclencheur** dans le menu, puis sélectionnez **Nouveau/Modifier**. 

    ![Menu de nouveau déclencheur](./media/how-to-create-schedule-trigger/new-trigger-menu.png)

1. Sur la page **Ajouter des déclencheurs**, sélectionnez **Choisir un déclencheur…** , puis **+Nouveau**. 

    ![Ajouter un déclencheur - nouveau déclencheur](./media/how-to-create-schedule-trigger/add-trigger-new-button.png)

1. Sur la page **Nouveau déclencheur**, procédez comme suit : 

    1. Vérifiez que **Planifier** est sélectionné pour **Type**.
    1. Spécifiez la date et l’heure de début du déclencheur pour **Date de début**. Par défaut, elles sont définies sur la date et l’heure actuelles en temps universel coordonné (UTC).
    1. Spécifiez le fuseau horaire dans lequel le déclencheur sera créé. Le paramètre de fuseau horaire s’applique à **Date de début**, **Date de fin** et **Planifier les temps d’exécution** dans les options de périodicité avancée. La modification du paramètre de fuseau horaire ne change pas automatiquement votre date de début. Vérifiez que la date de début est correcte dans le fuseau horaire spécifié. Notez que l’heure d’exécution planifiée du déclencheur sera considérée comme postérieure à la date de début (assurez-vous que la date de début précède d’au moins 1 minute le temps d’exécution, sinon elle déclenchera un pipeline à la prochaine récurrence). 

        > [!NOTE]
        > Pour les fuseaux horaires qui respectent l’heure d’été, l’heure de déclenchement est ajustée automatiquement au changement survenant deux fois par an. Pour désactiver le passage à l’heure d’été, sélectionnez un fuseau horaire qui ne tient pas compte de l’heure d’été, par exemple UTC

    1. Spécifiez la **Périodicité** du déclencheur. Sélectionnez une des valeurs dans la liste déroulante (toutes les minutes, horaire, quotidienne, hebdomadaire et mensuelle). Entrez le multiplicateur dans la zone de texte. Par exemple, si vous souhaitez exécuter le déclencheur toutes les 15 minutes, vous sélectionnez **Toutes les minutes**, puis entrez **15** dans la zone de texte. 
    1. Pour préciser une date et une heure de fin, sélectionnez **Spécifier une date de fin**, puis définissez _Se termine le_ et sélectionnez **OK**. Un coût est associé à chaque exécution du pipeline. Si vous effectuez un test, vous souhaiterez peut-être vous assurer que le pipeline n’est déclenché que deux fois. Toutefois, vérifiez que la durée est suffisante entre l’heure de publication et l’heure de fin pour permettre l’exécution du pipeline. Le déclencheur ne s’applique que lorsque vous avez publié la solution dans Data Factory, et non lorsque vous enregistrez le déclencheur dans l’interface utilisateur.

        ![Paramètres du déclencheur](./media/how-to-create-schedule-trigger/trigger-settings-01.png)

        ![Paramètres du déclencheur pour la date de fin](./media/how-to-create-schedule-trigger/trigger-settings-02.png)

1. Dans la fenêtre **Nouveau déclencheur**, sélectionnez **Oui** dans l’option **Activé**, puis sélectionnez **OK**. Vous pouvez utiliser cette case à cocher pour désactiver le déclencheur ultérieurement. 

    ![Paramètres du déclencheur - Bouton Suivant](./media/how-to-create-schedule-trigger/trigger-settings-next.png)

1. Dans la fenêtre **Nouveau déclencheur**, consultez le message d’avertissement, puis sélectionnez **OK**.

    ![Paramètres du déclencheur - Bouton Terminer](./media/how-to-create-schedule-trigger/new-trigger-finish.png)

1. Sélectionnez **Publier tout** pour publier les changements sur Data Factory. Tant que vous n’avez pas publié les modifications sur Data Factory, le déclencheur ne démarre pas le déclenchement des exécutions du pipeline. 

    ![Bouton Publier](./media/how-to-create-schedule-trigger/publish-2.png)

1. Basculez sur l’onglet **Exécutions de pipeline** sur la gauche, puis sélectionnez **Actualiser** pour actualiser la liste. Vous observerez les exécutions du pipeline déclenchées par le déclencheur planifié. Notez les valeurs dans la colonne **Déclenché par**. Si vous utilisez l’option **Déclencher maintenant**, l’exécution du déclencheur manuelle apparaît dans la liste. 

    ![Surveiller les exécutions déclenchées](./media/how-to-create-schedule-trigger/monitor-triggered-runs.png)

1. Passez à l’affichage **Exécutions du déclencheur** \ **Planifier**. 

    ![Surveiller les exécutions du déclencheur](./media/how-to-create-schedule-trigger/monitor-trigger-runs.png)

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Cette section montre comment utiliser Azure PowerShell pour créer, démarrer et surveiller un déclencheur de planification. Pour voir cet exemple en fonctionnement, commencez par suivre le [Guide de démarrage rapide : Créer une fabrique de données à l’aide d’Azure PowerShell](quickstart-create-data-factory-powershell.md). Ensuite, ajoutez le code suivant à la méthode main : il crée et lance un déclencheur de planification qui s’exécute toutes les 15 minutes. Le déclencheur est associé à un pipeline nommé **Adfv2QuickStartPipeline**, créé dans le cadre du guide de démarrage rapide.

1. Créez un fichier JSON nommé **MyTrigger.json** dans le dossier C:\ADFv2QuickStartPSH\ avec le contenu suivant :

    > [!IMPORTANT]
    > Avant d’enregistrer le fichier JSON, affectez l’heure UTC actuelle comme valeur de l’élément **startTime**. Définissez la valeur de l’élément **endTime** sur une (1) heure après l’heure UTC actuelle.

    ```json   
    {
        "properties": {
            "name": "MyTrigger",
            "type": "ScheduleTrigger",
            "typeProperties": {
                "recurrence": {
                    "frequency": "Minute",
                    "interval": 15,
                    "startTime": "2017-12-08T00:00:00Z",
                    "endTime": "2017-12-08T01:00:00Z",
                    "timeZone": "UTC"
                }
            },
            "pipelines": [{
                    "pipelineReference": {
                        "type": "PipelineReference",
                        "referenceName": "Adfv2QuickStartPipeline"
                    },
                    "parameters": {
                        "inputPath": "adftutorial/input",
                        "outputPath": "adftutorial/output"
                    }
                }
            ]
        }
    }
    ```

    Dans l’extrait de code JSON :
    - L’élément **type** du déclencheur est défini sur « ScheduleTrigger ».
    - L’élément **frequency** est défini sur « Minute » et l’élément **interval** sur 15. Par conséquent, le déclencheur exécute le pipeline toutes les 15 minutes entre l’heure de début et l’heure de fin.
    - L’élément **timeZone** spécifie le fuseau horaire dans lequel le déclencheur est créé. Ce paramètre affecte à la fois **startTime** et **endTime**.
    - L’élément **endTime** est une (1) heure après la valeur de l’élément **startTime**. Le déclencheur exécute alors le pipeline 15 minutes, 30 minutes et 45 minutes après l’heure de début. N’oubliez pas de définir l’heure de début sur l’heure UTC actuelle et l’heure de fin sur une (1) heure après l’heure de début. 

        > [!IMPORTANT]
        > Pour le fuseau horaire UTC, les éléments startTime et endTime doivent suivre le format « aaaa-MM-jjTHH:mm:ss **Z** », tandis que pour les autres fuseaux horaires, ils suivent le format « aaaa-MM-jjTHH:mm:ss ». 
        > 
        > Conformément à la norme ISO 8601, le suffixe _Z_ sur l’horodatage marque la date et l’heure du fuseau horaire UTC et rend la présence du champ timeZone inutile. En revanche, l’absence du suffixe _Z_ pour le fuseau horaire UTC génère une erreur à l’_activation_ du déclencheur.

    - Le déclencheur est associé au pipeline **Adfv2QuickStartPipeline**. Pour associer plusieurs pipelines à un déclencheur, ajoutez d’autres sections **pipelineReference**.
    - Le pipeline du guide de démarrage rapide prend deux valeurs de **paramètres** : **inputPath** et **outputPath**. Et vous passez des valeurs pour ces paramètres à partir du déclencheur.

1. Créez un déclencheur avec l’applet de commande **Set-AzDataFactoryV2Trigger** :

    ```powershell
    Set-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger" -DefinitionFile "C:\ADFv2QuickStartPSH\MyTrigger.json"
    ```

1. Vérifiez que l’état du déclencheur est **Stopped** avec l’applet de commande **Get-AzDataFactoryV2Trigger** :

    ```powershell
    Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"
    ```

1. Démarrez le déclencheur avec la cmdlet **Start-AzDataFactoryV2Trigger** :

    ```powershell
    Start-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"
    ```

1. Vérifiez que l’état du déclencheur est **Started** avec l’applet de commande **Get-AzDataFactoryV2Trigger** :

    ```powershell
    Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"
    ```

1.  Récupérez les exécutions du déclencheur dans Azure PowerShell avec l’applet de commande **Get-AzDataFactoryV2TriggerRun**. Pour obtenir les d’informations relatives aux exécutions du déclencheur, exécutez la commande suivante régulièrement. Mettez à jour les valeurs **TriggerRunStartedAfter** et **TriggerRunStartedBefore** pour qu’elles correspondent aux valeurs spécifiées dans la définition du déclencheur :

    ```powershell
    Get-AzDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -TriggerName "MyTrigger" -TriggerRunStartedAfter "2017-12-08T00:00:00" -TriggerRunStartedBefore "2017-12-08T01:00:00"
    ```
    
    > [!NOTE]
    > L’heure de déclenchement des déclencheurs de planification est spécifiée dans l’horodatage UTC. _TriggerRunStartedAfter_ et _TriggerRunStartedBefore_ attendent également l’horodatage UTC

    Pour surveiller les exécutions du déclencheur et du pipeline dans le portail Azure, consultez [Surveiller des exécutions de pipelines](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline).



## <a name="net-sdk"></a>Kit de développement logiciel (SDK) .NET
Cette section montre comment utiliser le Kit de développement logiciel (SDK) .NET pour créer, démarrer et surveiller un déclencheur. Pour voir cet exemple en fonctionnement, commencez par suivre le [Guide de démarrage rapide : Créer une fabrique de données à l’aide du SDK .NET](quickstart-create-data-factory-dot-net.md). Ensuite, ajoutez le code suivant à la méthode main : il crée et lance un déclencheur de planification qui s’exécute toutes les 15 minutes. Le déclencheur est associé à un pipeline nommé **Adfv2QuickStartPipeline**, créé dans le cadre du guide de démarrage rapide.

Pour créer et lancer un déclencheur de planification qui s’exécute toutes les 15 minutes, ajoutez le code suivant à la méthode main :

```csharp
            // Create the trigger
            Console.WriteLine("Creating the trigger");

            // Set the start time to the current UTC time
            DateTime startTime = DateTime.UtcNow;

            // Specify values for the inputPath and outputPath parameters
            Dictionary<string, object> pipelineParameters = new Dictionary<string, object>();
            pipelineParameters.Add("inputPath", "adftutorial/input");
            pipelineParameters.Add("outputPath", "adftutorial/output");

            // Create a schedule trigger
            string triggerName = "MyTrigger";
            ScheduleTrigger myTrigger = new ScheduleTrigger()
            {
                Pipelines = new List<TriggerPipelineReference>()
                {
                    // Associate the Adfv2QuickStartPipeline pipeline with the trigger
                    new TriggerPipelineReference()
                    {
                        PipelineReference = new PipelineReference(pipelineName),
                        Parameters = pipelineParameters,
                    }
                },
                Recurrence = new ScheduleTriggerRecurrence()
                {
                    // Set the start time to the current UTC time and the end time to one hour after the start time
                    StartTime = startTime,
                    TimeZone = "UTC",
                    EndTime = startTime.AddHours(1),
                    Frequency = RecurrenceFrequency.Minute,
                    Interval = 15,
                }
            };

            // Now, create the trigger by invoking the CreateOrUpdate method
            TriggerResource triggerResource = new TriggerResource()
            {
                Properties = myTrigger
            };
            client.Triggers.CreateOrUpdate(resourceGroup, dataFactoryName, triggerName, triggerResource);

            // Start the trigger
            Console.WriteLine("Starting the trigger");
            client.Triggers.Start(resourceGroup, dataFactoryName, triggerName);
```

Pour créer des déclencheurs dans un fuseau horaire différent, autre que UTC, les paramètres suivants sont nécessaires :
```csharp
<<ClientInstance>>.SerializationSettings.DateFormatHandling = Newtonsoft.Json.DateFormatHandling.IsoDateFormat;
<<ClientInstance>>.SerializationSettings.DateTimeZoneHandling = Newtonsoft.Json.DateTimeZoneHandling.Unspecified;
<<ClientInstance>>.SerializationSettings.DateParseHandling = DateParseHandling.None;
<<ClientInstance>>.DeserializationSettings.DateParseHandling = DateParseHandling.None;
<<ClientInstance>>.DeserializationSettings.DateFormatHandling = Newtonsoft.Json.DateFormatHandling.IsoDateFormat;
<<ClientInstance>>.DeserializationSettings.DateTimeZoneHandling = Newtonsoft.Json.DateTimeZoneHandling.Unspecified;
```

Pour surveiller une exécution du déclencheur, ajoutez le code suivant avant la dernière instruction `Console.WriteLine` de l’exemple :

```csharp
            // Check that the trigger runs every 15 minutes
            Console.WriteLine("Trigger runs. You see the output every 15 minutes");

            for (int i = 0; i < 3; i++)
            {
                System.Threading.Thread.Sleep(TimeSpan.FromMinutes(15));
                List<TriggerRun> triggerRuns = client.Triggers.ListRuns(resourceGroup, dataFactoryName, triggerName, DateTime.UtcNow.AddMinutes(-15 * (i + 1)), DateTime.UtcNow.AddMinutes(2)).ToList();
                Console.WriteLine("{0} trigger runs found", triggerRuns.Count);

                foreach (TriggerRun run in triggerRuns)
                {
                    foreach (KeyValuePair<string, string> triggeredPipeline in run.TriggeredPipelines)
                    {
                        PipelineRun triggeredPipelineRun = client.PipelineRuns.Get(resourceGroup, dataFactoryName, triggeredPipeline.Value);
                        Console.WriteLine("Pipeline run ID: {0}, Status: {1}", triggeredPipelineRun.RunId, triggeredPipelineRun.Status);
                        List<ActivityRun> runs = client.ActivityRuns.ListByPipelineRun(resourceGroup, dataFactoryName, triggeredPipelineRun.RunId, run.TriggerRunTimestamp.Value, run.TriggerRunTimestamp.Value.AddMinutes(20)).ToList();
                    }
                }
            }
```

Pour surveiller les exécutions du déclencheur et du pipeline dans le portail Azure, consultez [Surveiller des exécutions de pipelines](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline).


## <a name="python-sdk"></a>Kit de développement logiciel (SDK) Python
Cette section montre comment utiliser le Kit de développement logiciel (SDK) Python pour créer, démarrer et surveiller un déclencheur. Pour voir cet exemple en fonctionnement, commencez par suivre le [Guide de démarrage rapide : Créer une fabrique de données à l’aide du SDK Python](quickstart-create-data-factory-python.md). Ensuite, ajoutez le bloc de code suivant après le bloc de code « surveiller l’exécution du déclencheur » dans le script Python. Ce code crée un déclencheur de planification qui s’exécute toutes les 15 minutes entre l’heure de début et l’heure de fin spécifiées. Définissez la variable **start_time** sur l’heure UTC actuelle et la variable **end_time** sur une (1) heure après l’heure UTC actuelle.

```python
    # Create a trigger
    tr_name = 'mytrigger'
    scheduler_recurrence = ScheduleTriggerRecurrence(frequency='Minute', interval='15',start_time='2017-12-12T04:00:00Z', end_time='2017-12-12T05:00:00Z', time_zone='UTC')
    pipeline_parameters = {'inputPath':'adftutorial/input', 'outputPath':'adftutorial/output'}
    pipelines_to_run = []
    pipeline_reference = PipelineReference('copyPipeline')
    pipelines_to_run.append(TriggerPipelineReference(pipeline_reference, pipeline_parameters))
    tr_properties = ScheduleTrigger(description='My scheduler trigger', pipelines = pipelines_to_run, recurrence=scheduler_recurrence)    
    adf_client.triggers.create_or_update(rg_name, df_name, tr_name, tr_properties)

    # Start the trigger
    adf_client.triggers.start(rg_name, df_name, tr_name)
```

Pour surveiller les exécutions du déclencheur et du pipeline dans le portail Azure, consultez [Surveiller des exécutions de pipelines](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline).

## <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager
Vous pouvez utiliser un modèle Azure Resource Manager pour créer un déclencheur. Pour obtenir des instructions pas à pas, consultez [Créer une fabrique de données Azure à l’aide d’un modèle Resource Manager](quickstart-create-data-factory-resource-manager-template.md).  

## <a name="pass-the-trigger-start-time-to-a-pipeline"></a>Transmettre l’heure de début du déclencheur à un pipeline
Azure Data Factory version 1 prend en charge la lecture ou l’écriture des données partitionnées à l’aide des variables système : **SliceStart**, **SliceEnd**, **WindowStart** et **WindowEnd**. Dans la version actuelle d’Azure Data Factory, vous pouvez obtenir ce comportement à l’aide d’un paramètre de pipeline. L’heure de début et l’heure planifiée du déclencheur sont définies comme étant la valeur du paramètre de pipeline. Dans l’exemple suivant, l’heure planifiée du déclencheur est transmise comme valeur au paramètre **scheduledRunTime** de pipeline :

```json
"parameters": {
    "scheduledRunTime": "@trigger().scheduledTime"
}
```

## <a name="json-schema"></a>Schéma JSON
La définition JSON suivante vous montre comment créer un déclencheur de planification avec une planification et une périodicité :

```json
{
  "properties": {
    "type": "ScheduleTrigger",
    "typeProperties": {
      "recurrence": {
        "frequency": <<Minute, Hour, Day, Week, Month>>,
        "interval": <<int>>,             // Optional, specifies how often to fire (default to 1)
        "startTime": <<datetime>>,
        "endTime": <<datetime - optional>>,
        "timeZone": "UTC"
        "schedule": {                    // Optional (advanced scheduling specifics)
          "hours": [<<0-23>>],
          "weekDays": [<<Monday-Sunday>>],
          "minutes": [<<0-59>>],
          "monthDays": [<<1-31>>],
          "monthlyOccurrences": [
               {
                    "day": <<Monday-Sunday>>,
                    "occurrence": <<1-5>>
               }
           ]
        }
      }
    },
   "pipelines": [
            {
                "pipelineReference": {
                    "type": "PipelineReference",
                    "referenceName": "<Name of your pipeline>"
                },
                "parameters": {
                    "<parameter 1 Name>": {
                        "type": "Expression",
                        "value": "<parameter 1 Value>"
                    },
                    "<parameter 2 Name>" : "<parameter 2 Value>"
                }
           }
      ]
  }
}
```

> [!IMPORTANT]
>  La propriété **parameters** est une propriété obligatoire de l’élément **pipelines**. Si votre pipeline n’accepte aucun paramètre, vous devez inclure une définition JSON vide pour la propriété **parameters**.


### <a name="schema-overview"></a>Vue d’ensemble du schéma
Le tableau suivant présente une vue d’ensemble globale des principaux éléments du schéma liés à la périodicité et à la planification d’un déclencheur :

| Propriété JSON | Description |
|:--- |:--- |
| **startTime** | Une valeur date-heure. Pour les planifications simples, la valeur de la propriété **startTime** s’applique à la première occurrence. Pour les planifications complexes, le déclencheur ne démarre pas avant la valeur **startTime** spécifiée. <br> Pour le fuseau horaire UTC, le format est `'yyyy-MM-ddTHH:mm:ssZ'`, pour les autres fuseaux horaires, le format est `'yyyy-MM-ddTHH:mm:ss'`. |
| **endTime** | La date et l’heure de fin du déclencheur. Le déclencheur ne s’exécute pas après la date et l’heure de fin spécifiées. La valeur de la propriété ne peut pas être dans le passé. Cette propriété est facultative.  <br> Pour le fuseau horaire UTC, le format est `'yyyy-MM-ddTHH:mm:ssZ'`, pour les autres fuseaux horaires, le format est `'yyyy-MM-ddTHH:mm:ss'`. |
| **timeZone** | Fuseau horaire dans lequel le déclencheur est créé. Ce paramètre a un impact sur **startTime**, **endTime** et **schedule**. Consultez la [liste des fuseaux horaires pris en charge](#time-zone-option). |
| **recurrence** | Un objet de périodicité qui spécifie les règles de périodicité pour le déclencheur. L’objet de périodicité prend en charge les éléments suivants : **frequency**, **interval**, **endTime**, **count** et **schedule**. Lorsqu’un objet de périodicité est défini, l’élément **frequency** est requis. Les autres éléments de l’objet de périodicité sont facultatifs. |
| **frequency** | L’unité de fréquence à laquelle le déclencheur se répète. Les valeurs prises en charge incluent « minute », « heure », « day », « semaine » et « mois ». |
| **interval** | Un entier positif qui indique l’intervalle de la valeur **frequency**, qui détermine la fréquence d’exécution du déclencheur. Par exemple, si **l’intervalle** est défini sur 3 et la **fréquence** définie sur « semaine », le déclencheur se répète toutes les 3 semaines. |
| **schedule** | La planification périodique du déclencheur. Un déclencheur dont la valeur **frequency** spécifiée modifie sa périodicité selon une planification périodique. Une propriété **schedule** contient des modifications pour la périodicité basées sur des minutes, heures, jours de la semaine, jours du mois et numéro de semaine.

> [!IMPORTANT]
> Pour le fuseau horaire UTC, les éléments startTime et endTime doivent suivre le format « aaaa-MM-jjTHH:mm:ss **Z** », tandis que pour les autres fuseaux horaires, ils suivent le format « aaaa-MM-jjTHH:mm:ss ». 
> 
> Conformément à la norme ISO 8601, le suffixe _Z_ sur l’horodatage marque la date et l’heure du fuseau horaire UTC et rend la présence du champ timeZone inutile. En revanche, l’absence du suffixe _Z_ pour le fuseau horaire UTC génère une erreur à l’_activation_ du déclencheur.

### <a name="schema-defaults-limits-and-examples"></a>Valeurs par défaut, limites et exemples du schéma

| Propriété JSON | Type | Obligatoire | Valeur par défaut | Valeurs valides | Exemple |
|:--- |:--- |:--- |:--- |:--- |:--- |
| **startTime** | String | Oui | None | Dates-Heures ISO-8601 | pour le fuseau horaire UTC `"startTime" : "2013-01-09T09:30:00-08:00Z"` <br> pour les autres fuseaux horaires `"2013-01-09T09:30:00-08:00"` |
| **timeZone** | String | Oui | None | [Valeurs de fuseau horaire](#time-zone-option)  | `"UTC"` |
| **recurrence** | Object | Oui | None | Objet de périodicité | `"recurrence" : { "frequency" : "monthly", "interval" : 1 }` |
| **interval** | Number | Non | 1 | 1 à 1 000 | `"interval":10` |
| **endTime** | String | Oui | None | Une valeur date-heure représentant une heure dans le futur. | pour le fuseau horaire UTC `"endTime" : "2013-02-09T09:30:00-08:00Z"` <br> pour les autres fuseaux horaires `"endTime" : "2013-02-09T09:30:00-08:00"`|
| **schedule** | Object | Non | None | Objet de planification | `"schedule" : { "minute" : [30], "hour" : [8,17] }` |

### <a name="time-zone-option"></a>Option de fuseau horaire

Voici quelques fuseaux horaires pris en charge pour les déclencheurs de planification :

| Time Zone (Fuseau horaire) | Décalage UTC (n’appliquant pas l’heure d’été) | Valeur timeZone | Respect de l’heure d’été | Format horodatage |
| :--- | :--- | :--- | :--- | :--- |
| Temps universel coordonné | 0 | `UTC` | Non | `'yyyy-MM-ddTHH:mm:ssZ'`|
| Pacifique (PT) | -8 | `Pacific Standard Time` | Oui | `'yyyy-MM-ddTHH:mm:ss'` |
| Centre (CT) | -6 | `Central Standard Time` | Oui | `'yyyy-MM-ddTHH:mm:ss'` |
| Est (ET) | -5 | `Eastern Standard Time` | Oui | `'yyyy-MM-ddTHH:mm:ss'` |
| Heure moyenne de Greenwich (GMT) | 0 | `GMT Standard Time` | Oui | `'yyyy-MM-ddTHH:mm:ss'` |
| Heure standard d’Europe centrale | +1 | `W. Europe Standard Time` | Oui | `'yyyy-MM-ddTHH:mm:ss'` |
| Inde (IST) | +5:30 | `India Standard Time` | Non | `'yyyy-MM-ddTHH:mm:ss'` |
| Heure standard de Chine | +8 | `China Standard Time` | Non | `'yyyy-MM-ddTHH:mm:ss'` |

Cette liste est incomplète. Pour obtenir la liste complète des options de fuseau horaire, explorez la [page de création du déclencheur](#data-factory-ui) dans le portail Data Factory

### <a name="starttime-property"></a>propriété startTime
Le tableau suivant vous montre comment la propriété **startTime** contrôle une exécution du déclencheur :

| Valeur startTime | Périodicité sans planification | Périodicité avec planification |
|:--- |:--- |:--- |
| Heure de début dans le passé | Calcule la prochaine exécution initiale après l’heure de début, puis l’exécute à l’heure indiquée.<br/><br/>Lance les exécutions suivantes à partir du calcul effectué lors de la dernière exécution.<br/><br/>Consultez l’exemple qui suit ce tableau. | Le déclencheur _ne démarre pas avant_ l’heure de début spécifiée. La première occurrence dépend de la planification calculée à partir de l’heure de début.<br/><br/>Lance les exécutions suivantes à partir de la planification de périodicité. |
| Heure de début dans le futur ou au moment présent | S’exécute une fois à l’heure de début spécifiée.<br/><br/>Lance les exécutions suivantes à partir du calcul effectué lors de la dernière exécution. | Le déclencheur _ne démarre pas avant_ l’heure de début spécifiée. La première occurrence dépend de la planification calculée à partir de l’heure de début.<br/><br/>Lance les exécutions suivantes à partir de la planification de périodicité. |

Voyons ce qui se passe lorsque l’heure de début se situe dans le passé, avec la valeur de périodicité définie mais pas la valeur de planification. Supposons que l’heure actuelle soit `2017-04-08 13:00`, l’heure de début `2017-04-07 14:00`, et que la périodicité soit tous les deux jours. (La valeur **recurrence** est définie en paramétrant la propriété **frequency** sur « jour » et la propriété **interval** sur 2.) Notez que la valeur **startTime** se situe dans le passé et se produit avant l’heure actuelle.

Dans ces conditions, la première exécution aura lieu le `2017-04-09` à `14:00`. Le moteur d'Azure Scheduler calcule les occurrences de l'exécution à partir de l'heure de début. Toutes les instances dans le passé sont ignorées. Le moteur utilise l'instance suivante qui se produit dans le futur. Dans ce scénario, la date de début étant le `2017-04-07` à `2:00pm`, l’instance suivante surviendra 2 jours après cette date, c’est-à-dire le `2017-04-09` à `2:00pm`.

La première heure d’exécution est la même si la valeur **startTime** est `2017-04-05 14:00` ou `2017-04-01 14:00`. Après la première exécution, les exécutions suivantes sont calculées à l’aide de la planification. Par conséquent, les exécutions suivantes auront lieu le `2017-04-11` à `2:00pm`, puis le `2017-04-13` à `2:00pm`, puis le `2017-04-15` à `2:00pm`, et ainsi de suite.

Enfin, lorsque les heures ou les minutes ne sont pas définies dans la planification d’un déclencheur, les heures ou minutes de la première exécution sont utilisées par défaut.

### <a name="schedule-property"></a>propriété de planification
D’une part, l’utilisation d’une planification peut limiter le nombre d’exécutions du déclencheur. Par exemple, si un déclencheur avec une fréquence mensuelle est planifié pour s’exécuter uniquement le 31, le déclencheur ne s’exécute que durant les mois qui comptent un 31e jour.

Une planification peut, quant à elle, également augmenter le nombre d’exécutions du déclencheur. Par exemple, un déclencheur avec une fréquence mensuelle planifiée pour s’exécuter le 1 et le 2 de chaque mois, s’exécute le 1 et le 2 de chaque mois, plutôt qu’une fois par mois.

Si plusieurs éléments **schedule** sont spécifiés, l’ordre d’évaluation va du plus grand au plus petit paramètre de planification. L’évaluation commence par le numéro de semaine, puis le jour du mois, jour de la semaine, heure et enfin, la minute.

Le tableau suivant décrit les éléments **schedule** en détail :


| Élément JSON | Description | Valeurs valides |
|:--- |:--- |:--- |
| **minutes** | Minutes d’exécution du déclencheur dans l’heure. | <ul><li>Integer</li><li>Tableau d’entiers</li></ul>
| **hours** | Heures d’exécution du déclencheur dans la journée. | <ul><li>Integer</li><li>Tableau d’entiers</li></ul> |
| **weekDays** | Jours d’exécution du déclencheur dans la semaine. La valeur ne peut être spécifiée qu’avec une fréquence hebdomadaire uniquement. | <ul><li>Lundi, mardi, mercredi, jeudi, vendredi, samedi, dimanche</li><li>Tableau des valeurs de jour (la taille maximale du tableau est de 7)</li><li>Les valeurs de jour ne respectent pas la casse</li></ul> |
| **monthlyOccurrences** | Jours d’exécution du déclencheur dans le mois. La valeur ne peut être spécifiée qu’avec une fréquence mensuelle uniquement. | <ul><li>Tableau d’objets **monthlyOccurrence** : `{ "day": day,  "occurrence": occurrence }`.</li><li>L’attribut **day** est le jour de la semaine durant lequel le déclencheur s’exécute. Par exemple, une propriété **monthlyOccurrences** avec une valeur **day** de `{Sunday}` signifie tous les dimanches du mois. L’attribut **day** est requis.</li><li>L’attribut **occurrence** est l’occurrence du **jour** spécifié au cours du mois. Par exemple, une propriété **monthlyOccurrences** avec les valeurs **day** et **occurrence** de `{Sunday, -1}` signifie le dernier dimanche du mois. L’attribut **occurrence** est facultatif.</li></ul> |
| **monthDays** | Jours d’exécution du déclencheur dans le mois. La valeur ne peut être spécifiée qu’avec une fréquence mensuelle uniquement. | <ul><li>Toute valeur <= -1 et >= -31</li><li>Toute valeur >= 1 et <= 31</li><li>Tableau de valeurs</li></ul> |


## <a name="examples-of-trigger-recurrence-schedules"></a>Exemples de planifications de périodicité de déclencheur
Vous trouverez dans cette section plusieurs exemples de planifications de périodicité qui se penchent tout particulièrement sur l’objet **schedule** et ses éléments.

Les exemples supposent que la valeur **interval** est 1 et que la valeur **frequency** est correcte selon la définition de planification. Par exemple, vous ne pouvez pas avoir une valeur **frequency** définie sur « jour » et une modification « monthDays » dans l’objet **schedule**. Ces restrictions sont mentionnées dans le tableau de la section précédente.

|  Exemple | Description |
|:--- |:--- |
| `{"hours":[5]}` | Exécution à 5h00 tous les jours. |
| `{"minutes":[15], "hours":[5]}` | Exécution à 5h15 tous les jours. |
| `{"minutes":[15], "hours":[5,17]}` | Exécution à 5h15 et 17h15 tous les jours. |
| `{"minutes":[15,45], "hours":[5,17]}` | Exécution à 5h15, 5h45, 17h15 et 17h45 tous les jours. |
| `{"minutes":[0,15,30,45]}` | Exécution toutes les 15 minutes. |
| `{hours":[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23]}` | Exécution toutes les heures. Ce déclencheur s’exécute toutes les heures. Les minutes sont contrôlées par la valeur **startTime**, lorsqu’une valeur est spécifiée. Si aucune valeur n’est spécifiée, les minutes sont contrôlées par l’heure de création. Par exemple, si l’heure de début ou l’heure de création (selon le cas) est 00h25, le déclencheur sera exécuté à 00h25, 01h25, 02h25, ..., et 23h25.<br/><br/>Cette planification est équivalente à un déclencheur avec une valeur **frequency** définie sur « heure», une valeur **interval** égale à 1 et aucune **planification**.  Cette planification peut être utilisée avec des valeurs **frequency** et **interval** différentes pour créer d’autres déclencheurs. Par exemple, lorsque la valeur **frequency** est définie sur « mois », la planification ne s’exécute qu’une fois par mois, plutôt que tous les jours, lorsque la valeur **frequency** est définie sur « jour ». |
| `{"minutes":[0]}` | Exécution toutes les heures sur l’heure. Ce déclencheur s’exécute toutes les heures sur l’heure, en commençant à 00h00, 01h00, 02h00, et ainsi de suite.<br/><br/>Cette planification est équivalente à un déclencheur avec une valeur **frequency** définie sur « heure » et une valeur **startTime** de zéro minute, ou aucune **planification** mais une valeur **frequency** définie sur « jour ». Si la valeur **frequency** est définie sur « semaine » ou « mois », la planification s’exécute un jour par semaine ou un jour par mois seulement, respectivement. |
| `{"minutes":[15]}` | Exécution 15 minutes après l’heure, toutes les heures. Ce déclencheur s’exécute toutes les heures 15 minutes après l’heure, en commençant à 00h15, 01h15, 02h15, et ainsi de suite, jusqu’à 23h15. |
| `{"hours":[17], "weekDays":["saturday"]}` | Exécution à 17h le samedi chaque semaine. |
| `{"hours":[17], "weekDays":["monday", "wednesday", "friday"]}` | Exécution à 17h le lundi, le mercredi et le vendredi chaque semaine. |
| `{"minutes":[15,45], "hours":[17], "weekDays":["monday", "wednesday", "friday"]}` | Exécution à 17h15 et 17h45 le lundi, le mercredi et le vendredi chaque semaine. |
| `{"minutes":[0,15,30,45], "weekDays":["monday", "tuesday", "wednesday", "thursday", "friday"]}` | Exécution toutes les 15 minutes les jours de semaine. |
| `{"minutes":[0,15,30,45], "hours": [9, 10, 11, 12, 13, 14, 15, 16] "weekDays":["monday", "tuesday", "wednesday", "thursday", "friday"]}` | Exécution toutes les 15 minutes les jours de semaine entre 9h00 et 16h45. |
| `{"weekDays":["tuesday", "thursday"]}` | Exécution le mardi et le jeudi à l’heure de début spécifiée. |
| `{"minutes":[0], "hours":[6], "monthDays":[28]}` | Exécution à 6h00 le 28e jour de chaque mois (en supposant que la valeur **frequency** est définie sur « mois »). |
| `{"minutes":[0], "hours":[6], "monthDays":[-1]}` | Exécution à 6h00 le dernier jour du mois. Pour exécuter un déclencheur le dernier jour du mois, utilisez -1 au lieu des jours 28, 29, 30 ou 31. |
| `{"minutes":[0], "hours":[6], "monthDays":[1,-1]}` | Exécution à 6h00 le premier et le dernier jour de chaque mois. |
| `{monthDays":[1,14]}` | Exécution le premier et le 14e jour de chaque mois à l’heure de début spécifiée. |
| `{"minutes":[0], "hours":[5], "monthlyOccurrences":[{"day":"friday", "occurrence":1}]}` | Exécution le premier vendredi de chaque mois à 5h00. |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":1}]}` | Exécution le premier vendredi de chaque mois à l’heure de début spécifiée. |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":-3}]}` | Exécutez le troisième vendredi à partir de la fin du mois, chaque mois, à l’heure de début spécifiée. |
| `{"minutes":[15], "hours":[5], "monthlyOccurrences":[{"day":"friday", "occurrence":1},{"day":"friday", "occurrence":-1}]}` | Exécution le premier et le dernier vendredi de chaque mois à 5h15. |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":1},{"day":"friday", "occurrence":-1}]}` | Exécution le premier et le dernier vendredi de chaque mois à l’heure de début spécifiée. |
| `{"monthlyOccurrences":[{"day":"friday", "occurrence":5}]}` | Exécution le cinquième vendredi de chaque mois à l’heure de début spécifiée. S’il n’y a pas de cinquième vendredi dans un mois, le pipeline n’est pas exécuté, dans la mesure où il est planifié pour s’exécuter uniquement le cinquième vendredi du mois. Pour exécuter le déclencheur le dernier vendredi du mois, indiquez -1 au lieu de 5 pour la valeur **occurrence**. |
| `{"minutes":[0,15,30,45], "monthlyOccurrences":[{"day":"friday", "occurrence":-1}]}` | Exécution toutes les 15 minutes le dernier vendredi du mois. |
| `{"minutes":[15,45], "hours":[5,17], "monthlyOccurrences":[{"day":"wednesday", "occurrence":3}]}` | Exécution à 5h15, 5h45, 17h15 et 17h45 le troisième mercredi de chaque mois. |


## <a name="next-steps"></a>Étapes suivantes
Vous trouverez des informations détaillées sur les déclencheurs sur la page [Exécution de pipelines et déclencheurs](concepts-pipeline-execution-triggers.md#trigger-execution).
