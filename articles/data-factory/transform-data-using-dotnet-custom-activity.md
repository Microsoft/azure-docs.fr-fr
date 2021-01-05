---
title: Utiliser des activités personnalisées dans un pipeline
description: Découvrez comment créer des activités personnalisées à l’aide de .NET, puis utiliser ces activités dans un pipeline Azure Data Factory.
services: data-factory
ms.service: data-factory
author: nabhishek
ms.author: abnarain
manager: anandsub
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 11/26/2018
ms.openlocfilehash: e84f7a2ee8c2f7a57ce1734ad3392a217d6de5fe
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92632105"
---
# <a name="use-custom-activities-in-an-azure-data-factory-pipeline"></a>Utilisation des activités personnalisées dans un pipeline Azure Data Factory

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-use-custom-activities.md)
> * [Version actuelle](transform-data-using-dotnet-custom-activity.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Vous pouvez utiliser deux types d’activités dans un pipeline Azure Data Factory.

- Les [activités de déplacement de données](copy-activity-overview.md) permettent de transférer des données entre les [magasins de données source et récepteur pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).
- Les [activités de transformation de données](transform-data.md) permettent de transformer des données à l’aide de services de calcul, comme Azure HDInsight, Azure Batch et Azure Machine Learning.

Pour déplacer des données vers ou depuis un magasin de données que Data Factory ne prend pas en charge, ou pour transformer et traiter les données d’une manière qui n’est pas prise en charge par Data Factory, créez une **Activité personnalisée** avec votre propre logique de déplacement ou de transformation des données, et utilisez cette activité dans un pipeline. L’activité personnalisée exécute votre logique de code personnalisé sur un pool de machines virtuelles **Azure Batch**.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Consultez les articles suivants si vous ne connaissez pas le service Azure Batch :

* [Présentation de base d’Azure Batch](../batch/batch-technical-overview.md) pour une vue d’ensemble du service Azure Batch.
* Applet de commande [New-AzBatchAccount](/powershell/module/az.batch/New-azBatchAccount) pour créer un compte Azure Batch (ou) [Portail Azure](../batch/batch-account-create-portal.md) pour créer le compte Azure Batch à l’aide du portail Azure. Pour obtenir des instructions détaillées sur l’utilisation de la cmdlet, voir [Utilisation de PowerShell pour gérer un compte Azure Batch](/archive/blogs/windowshpc/using-azure-powershell-to-manage-azure-batch-account).
* Applet de commande [New-AzBatchPool](/powershell/module/az.batch/New-AzBatchPool) pour créer un pool Azure Batch.

## <a name="azure-batch-linked-service"></a>Service lié Azure Batch

L’extrait de code JSON suivant définit un exemple de service lié Azure Batch. Pour plus de détails, consultez [Environnements de calcul pris en charge par Azure Data Factory](compute-linked-services.md).

```json
{
    "name": "AzureBatchLinkedService",
    "properties": {
        "type": "AzureBatch",
        "typeProperties": {
            "accountName": "batchaccount",
            "accessKey": {
                "type": "SecureString",
                "value": "access key"
            },
            "batchUri": "https://batchaccount.region.batch.azure.com",
            "poolName": "poolname",
            "linkedServiceName": {
                "referenceName": "StorageLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }
}
```

 Pour approfondir votre connaissance du service lié Azure Batch, consultez l’article [Services liés de calcul](compute-linked-services.md).

## <a name="custom-activity"></a>Activité personnalisée

L’extrait de code JSON suivant définit un pipeline avec une activité personnalisée simple. La définition de l’activité comporte une référence au service lié Azure Batch.

```json
{
  "name": "MyCustomActivityPipeline",
  "properties": {
    "description": "Custom activity sample",
    "activities": [{
      "type": "Custom",
      "name": "MyCustomActivity",
      "linkedServiceName": {
        "referenceName": "AzureBatchLinkedService",
        "type": "LinkedServiceReference"
      },
      "typeProperties": {
        "command": "helloworld.exe",
        "folderPath": "customactv2/helloworld",
        "resourceLinkedService": {
          "referenceName": "StorageLinkedService",
          "type": "LinkedServiceReference"
        }
      }
    }]
  }
}
```

Dans cet exemple, le fichier helloworld.exe est une application personnalisée qui est stockée dans le dossier customactv2/helloworld du compte Stockage Azure utilisé dans le resourceLinkedService. L’activité personnalisée soumet cette application personnalisée pour qu’elle soit exécutée sur Azure Batch. Vous pouvez remplacer la commande par n’importe quelle application par défaut pouvant être exécutée sur le système d’exploitation cible des nœuds du pool Azure Batch.

Le tableau suivant indique les noms et les descriptions des propriétés qui sont spécifiques à cette activité.

| Propriété              | Description                              | Obligatoire |
| :-------------------- | :--------------------------------------- | :------- |
| name                  | Nom de l’activité dans le pipeline     | Oui      |
| description           | Texte décrivant l’activité.  | Non       |
| type                  | Pour une activité personnalisée, le type d’activité est **Custom**. | Oui      |
| linkedServiceName     | Service lié sur Azure Batch. Pour en savoir plus sur ce service lié, consultez l’article [Services liés de calcul](compute-linked-services.md).  | Oui      |
| command               | Commande de l’application personnalisée à exécuter. Si l’application est déjà disponible sur le nœud du pool Azure Batch, resourceLinkedService et folderPath peuvent être ignorés. Par exemple, vous pouvez spécifier la commande pour qu’elle soit `cmd /c dir`, ce qui est pris en charge en mode natif par le nœud du pool Windows Batch. | Oui      |
| resourceLinkedService | Le service lié Stockage Azure sur le compte de stockage où l’application personnalisée est stockée. | Non &#42;       |
| folderPath            | Chemin du dossier de l’application personnalisée et de toutes ses dépendances.<br/><br/>Si vous avez des dépendances stockées dans les sous-dossiers (autrement dit, dans une structure de dossiers hiérarchique sous *folderPath* ),-la structure de dossiers est aplatie lorsque les fichiers sont copiés vers Azure Batch. Autrement dit, tous les fichiers sont copiés dans un dossier unique, sans sous-dossier. Pour contourner ce problème, envisagez de compresser les fichiers, de copier le fichier compressé, puis de le décompresser avec du code personnalisé à l’emplacement souhaité. | Non &#42;       |
| referenceObjects      | Tableau des services liés et des jeux de données existants. Les services liés et les jeux de données référencés sont passés à l’application personnalisée au format JSON, votre code personnalisé peut ainsi référencer des ressources de la fabrique de données. | Non       |
| extendedProperties    | Propriétés définies par l’utilisateur qui peuvent être passées à l’application personnalisée au format JSON, votre code personnalisé peut ainsi référencer des propriétés supplémentaires. | Non       |
| retentionTimeInDays | Durée de rétention pour les fichiers soumis pour une activité personnalisée. La valeur par défaut est de 30 jours. | Non |

&#42; Les propriétés `resourceLinkedService` et `folderPath` doivent être toutes deux spécifiées ou omises.

> [!NOTE]
> Si vous transmettez des services liés en tant que referenceObjects dans une activité personnalisée, il est recommandé de transmettre un service lié Azure Key Vault (dans la mesure où il ne contient pas de chaînes sécurisées), et de récupérer les informations d’identification à l’aide du nom du secret directement à partir de Key Vault issu du code. Vous trouverez un exemple [ici](https://github.com/nabhishek/customactivity_sample/tree/linkedservice) qui fait référence au service lié AKV, qui récupère les informations d’identification de Key Vault, puis accède au stockage dans le code.

## <a name="custom-activity-permissions"></a>Autorisations d’activité personnalisée

L’activité personnalisée définit le compte d’utilisateur automatique Azure Batch sur *Accès non-administrateur avec étendue de la tâche* (spécification utilisateur automatique par défaut). Vous ne pouvez pas modifier le niveau d’autorisation du compte d’utilisateur automatique. Pour plus d’informations, voir [Exécuter des tâches sous des comptes d’utilisateur dans Azure Batch | Comptes d’utilisateur automatique](../batch/batch-user-accounts.md#auto-user-accounts).

## <a name="executing-commands"></a>Exécution de commandes

Vous pouvez exécuter directement une commande à l’aide d’une activité personnalisée. Dans l’exemple suivant, la commande « echo hello world » est exécutée sur les nœuds de pool Azure Batch cibles, et la sortie est envoyée vers stdout.

```json
{
  "name": "MyCustomActivity",
  "properties": {
    "description": "Custom activity sample",
    "activities": [{
      "type": "Custom",
      "name": "MyCustomActivity",
      "linkedServiceName": {
        "referenceName": "AzureBatchLinkedService",
        "type": "LinkedServiceReference"
      },
      "typeProperties": {
        "command": "cmd /c echo hello world"
      }
    }]
  }
}
```

## <a name="passing-objects-and-properties"></a>Passage des objets et des propriétés

Cet exemple montre comment vous pouvez utiliser les referenceObjects et les extendedProperties pour passer des objets Data Factory et des propriétés définies par l’utilisateur sur votre application personnalisée.

```json
{
  "name": "MyCustomActivityPipeline",
  "properties": {
    "description": "Custom activity sample",
    "activities": [{
      "type": "Custom",
      "name": "MyCustomActivity",
      "linkedServiceName": {
        "referenceName": "AzureBatchLinkedService",
        "type": "LinkedServiceReference"
      },
      "typeProperties": {
        "command": "SampleApp.exe",
        "folderPath": "customactv2/SampleApp",
        "resourceLinkedService": {
          "referenceName": "StorageLinkedService",
          "type": "LinkedServiceReference"
        },
        "referenceObjects": {
          "linkedServices": [{
            "referenceName": "AzureBatchLinkedService",
            "type": "LinkedServiceReference"
          }]
        },
        "extendedProperties": {          
          "connectionString": {
            "type": "SecureString",
            "value": "aSampleSecureString"
          },
          "PropertyBagPropertyName1": "PropertyBagValue1",
          "propertyBagPropertyName2": "PropertyBagValue2",
          "dateTime1": "2015-04-12T12:13:14Z"
        }
      }
    }]
  }
}
```

Lorsque l’activité est exécutée, les éléments referenceObjects et extendedProperties sont stockés dans les fichiers suivants qui sont déployés vers le même dossier d’exécution de l’application SampleApp.exe :

- `activity.json`

  Stocke extendedProperties et les propriétés de l’activité personnalisée.

- `linkedServices.json`

  Stocke un tableau des services liés définis dans la propriété referenceObjects.

- `datasets.json`

  Stocke un tableau de jeux de données défini dans la propriété referenceObjects.

L’exemple de code suivant montre comment SampleApp.exe peut accéder aux informations nécessaires à partir de fichiers JSON :

```csharp
using Newtonsoft.Json;
using System;
using System.IO;

namespace SampleApp
{
    class Program
    {
        static void Main(string[] args)
        {
            //From Extend Properties
            dynamic activity = JsonConvert.DeserializeObject(File.ReadAllText("activity.json"));
            Console.WriteLine(activity.typeProperties.extendedProperties.connectionString.value);

            // From LinkedServices
            dynamic linkedServices = JsonConvert.DeserializeObject(File.ReadAllText("linkedServices.json"));
            Console.WriteLine(linkedServices[0].properties.typeProperties.accountName);
        }
    }
}
```

## <a name="retrieve-execution-outputs"></a>Récupérer les sorties de l’exécution

Vous pouvez démarrer une exécution de pipeline à l’aide de la commande PowerShell suivante :

```powershell
$runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName $pipelineName
```

Quand le pipeline s’exécute, vous pouvez vérifier la sortie de l’exécution à l’aide des commandes suivantes :

```powershell
while ($True) {
    $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)

    if(!$result) {
        Write-Host "Waiting for pipeline to start..." -foregroundcolor "Yellow"
    }
    elseif (($result | Where-Object { $_.Status -eq "InProgress" } | Measure-Object).count -ne 0) {
        Write-Host "Pipeline run status: In Progress" -foregroundcolor "Yellow"
    }
    else {
        Write-Host "Pipeline '"$pipelineName"' run finished. Result:" -foregroundcolor "Yellow"
        $result
        break
    }
    ($result | Format-List | Out-String)
    Start-Sleep -Seconds 15
}

Write-Host "Activity `Output` section:" -foregroundcolor "Yellow"
$result.Output -join "`r`n"

Write-Host "Activity `Error` section:" -foregroundcolor "Yellow"
$result.Error -join "`r`n"
```

Les éléments **stdout** et **stderr** de votre application personnalisée sont enregistrés dans le conteneur **adfjobs** du service lié Stockage Azure que vous avez défini lors de la création du service lié Azure Batch à l’aide d’un GUID de la tâche. Vous pouvez obtenir le chemin détaillé à partir de la sortie de l’exécution de l’activité, comme indiqué dans l’extrait de code suivant :

```
Pipeline ' MyCustomActivity' run finished. Result:

ResourceGroupName : resourcegroupname
DataFactoryName   : datafactoryname
ActivityName      : MyCustomActivity
PipelineRunId     : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
PipelineName      : MyCustomActivity
Input             : {command}
Output            : {exitcode, outputs, effectiveIntegrationRuntime}
LinkedServiceName :
ActivityRunStart  : 10/5/2017 3:33:06 PM
ActivityRunEnd    : 10/5/2017 3:33:28 PM
DurationInMs      : 21203
Status            : Succeeded
Error             : {errorCode, message, failureType, target}

Activity Output section:
"exitcode": 0
"outputs": [
  "https://<container>.blob.core.windows.net/adfjobs/<GUID>/output/stdout.txt",
  "https://<container>.blob.core.windows.net/adfjobs/<GUID>/output/stderr.txt"
]
"effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)"
Activity Error section:
"errorCode": ""
"message": ""
"failureType": ""
"target": "MyCustomActivity"
```

Si vous souhaitez consommer le contenu de stdout.txt dans des activités en aval, vous pouvez obtenir le chemin du fichier stdout.txt dans l’expression « \@activity(’MyCustomActivity’).output.outputs[0] ».

> [!IMPORTANT]
> - Les fichiers activity.json, linkedServices.json et datasets.json sont stockés dans le dossier d’exécution de la tâche de traitement par lots. Pour cet exemple, les fichiers activity.json, linkedServices.json et datasets.json sont stockés dans le chemin d’accès `"https://adfv2storage.blob.core.windows.net/adfjobs/\<GUID>/runtime/"`. Si nécessaire, vous devez les nettoyer séparément.
> - Pour les services liés qui utilisent le runtime d’intégration auto-hébergé, les informations sensibles comme les clés ou les mots de passe sont chiffrées par le runtime d’intégration auto-hébergé pour faire en sorte que les informations d’identification restent dans l’environnement de réseau privé défini par le client. Certains champs sensibles peuvent manquer lorsqu’ils sont référencés par votre code d’application personnalisé de cette façon. Au besoin, utilisez SecureString dans extendedProperties au lieu d’utiliser une référence de service lié.

## <a name="pass-outputs-to-another-activity"></a>Passer les sorties à une autre activité

Vous pouvez renvoyer les valeurs personnalisées figurant dans le code d’une activité personnalisée à Azure Data Factory. Pour cela, vous devez les écrire dans le fichier `outputs.json` de votre application. Data Factory copie le contenu de `outputs.json` et l’ajoute à la sortie d’activité comme valeur de la propriété `customOutput`. (La taille limite est de 2 Mo.) Si vous souhaitez utiliser le contenu de `outputs.json` dans des activités en aval, vous pouvez obtenir la valeur à l’aide de l’expression `@activity('<MyCustomActivity>').output.customOutput`.

## <a name="retrieve-securestring-outputs"></a>Récupérer les sorties SecureString

Les valeurs de propriété sensibles désignées en tant que type *SecureString* , comme illustré dans certains exemples de cet article, sont masqués dans l’onglet Surveillance de l’interface utilisateur de Data Factory.  Lors de l’exécution réelle du pipeline, cependant, une propriété *SecureString* est sérialisée au format JSON dans le fichier `activity.json` en tant que texte brut. Par exemple :

```json
"extendedProperties": {
  "connectionString": {
    "type": "SecureString",
    "value": "aSampleSecureString"
  }
}
```

Cette sérialisation n’est pas véritablement sécurisée et n’est pas destinée à être sécurisée. L’objectif est d’indiquer à Data Factory de masquer la valeur dans l’onglet Surveillance.

Pour accéder aux propriétés de type *SecureString* à partir d’une activité personnalisée, lisez le fichier `activity.json`, placé dans le même dossier que le fichier EXE, désérialisez le code JSON, puis accédez à la propriété JSON (extendedProperties => [propertyName] => valeur).

## <a name="compare-v2-custom-activity-and-version-1-custom-dotnet-activity"></a><a name="compare-v2-v1"></a> Comparer les activités personnalisées de la version 2 et les activités DotNet (personnalisées) de la version 1

Dans la version 1 d’Azure Data Factory, pour implémenter une activité DotNet (personnalisée), on crée un projet de bibliothèque de classes .NET avec une classe qui implémente la méthode `Execute` de l’interface `IDotNetActivity`. Les services liés, les jeux de données et les propriétés étendues de la charge utile JSON d’une activité DotNet (personnalisée) sont transmis à la méthode d’exécution sous forme d’objets fortement typés. Pour plus d’informations sur le comportement de la version 1, consultez la page [DotNet (personnalisé) dans la version 1](v1/data-factory-use-custom-activities.md). À cause de cette implémentation, le code de votre activité DotNet de la version 1 doit cibler .NET Framework 4.5.2. L’activité DotNet de la version 1 doit également être exécutée sur des nœuds de pools Azure Batch Windows.

Dans une activité personnalisée de la version 2 d’Azure Data Factory, il n’est pas obligatoire d’implémenter une interface .NET. Vous pouvez maintenant exécuter directement des commandes, des scripts et votre propre code compilé sous forme d’exécutable. Pour configurer cette implémentation, spécifiez la propriété `Command` conjointement avec la propriété `folderPath`. L’activité personnalisée charge l’exécutable et ses dépendances sur `folderpath` et exécute la commande automatiquement.

Les services liés, les jeux de données (définis dans referenceObjects) et les propriétés étendues définis dans la charge utile JSON d’une activité personnalisée de la version 2 de Data Factory sont accessibles par le biais de l’exécutable sous forme de fichiers JSON. Vous pouvez accéder aux propriétés requises à l’aide du sérialiseur JSON, comme dans l’exemple de code SampleApp.exe précédent.

Grâce aux modifications introduites dans l’activité personnalisée de la version 2 d’Azure Data Factory, vous pouvez écrire votre logique de code personnalisée dans le langage de votre choix et l’exécuter sur les systèmes d’exploitation Windows et Linux pris en charge par Azure Batch.

Le tableau suivant décrit les différences qui existent entre l’activité personnalisée de la version 2 de Data Factory et l’activité DotNet (personnalisée) de la version 1 de Data Factory :

|Différences      | Activité personnalisée      | Activité DotNet (personnalisée) de la version 1      |
| ---- | ---- | ---- |
|Mode de définition de la logique personnalisée      |En fournissant un exécutable      |En implémentant une DLL .NET      |
|Environnement d’exécution de la logique personnalisée      |Windows ou Linux      |Windows (.NET Framework 4.5.2)      |
|Exécution des scripts      |Prend en charge l’exécution directe de scripts (par exemple « cmd /c echo hello world » sur une machine virtuelle Windows)      |Nécessite une implémentation dans la DLL .NET      |
|Jeu de données nécessaire      |Facultatif      |Nécessaire pour chaîner des activités et passer des informations      |
|Passer des informations entre l’activité et la logique personnalisée      |Via ReferenceObjects (LinkedServices et Datasets) et ExtendedProperties (propriétés personnalisées)      |et via ExtendedProperties (propriétés personnalisées), Input et Output Datasets      |
|Récupération des informations dans la logique personnalisée      |Analyse activity.json, linkedServices.json et datasets.json, stockés dans le même dossier que l’exécutable      |Via .NET SDK (.NET Frame 4.5.2)      |
|Journalisation      |Écrit directement dans STDOUT      |Implémentation de l’enregistreur d’événements dans la DLL .NET      |

S’il vous reste du code .NET écrit pour une activité DotNet (personnalisée) de la version 1, vous devez le modifier de sorte qu’il fonctionne avec la version actuelle de l’activité personnalisée. Pour mettre à jour votre code, suivez ces instructions générales :

  - Changez le projet pour qu’il s’agisse non plus d’une bibliothèque de classes .Net, mais d’une application console.
  - Démarrez votre application avec la méthode `Main`. La méthode `Execute` de l’interface `IDotNetActivity` n’est plus nécessaire.
  - Lisez et analyser les services liés, les jeux de données et l’activité avec un sérialiseur JSON et non en tant qu’objets fortement typés. Transmettez les valeurs des propriétés requises à la logique du code personnalisé principal. Voir l’exemple de code SampleApp.exe plus haut.
  - L’objet Enregistreur d’événements n’est plus pris en charge. La sortie de l’exécutable peut être imprimée sur la console ; elle est enregistrée dans stdout.txt.
  - Le package NuGet Microsoft.Azure.Management.DataFactories n’est plus nécessaire.
  - Compilez votre code, chargez l’exécutable et ses dépendances dans le Stockage Azure et définissez le chemin d’accès dans la propriété `folderPath`.

Vous trouverez un exemple complet de réécriture de l’exemple de DLL et de pipeline de bout en bout décrit dans l’article sur la version 1 de Data Factory, [Utiliser des activités personnalisées dans un pipeline Azure Data Factory](./v1/data-factory-use-custom-activities.md), en tant qu’activité personnalisée Data Factory dans la page [Exemple d’activité personnalisée Data Factory](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/ADFv2CustomActivitySample).

## <a name="auto-scaling-of-azure-batch"></a>Mise à l’échelle automatique d’Azure Batch

Vous pouvez aussi créer un pool Azure Batch avec la fonctionnalité **autoscale** . Par exemple, vous pouvez créer un pool Azure Batch avec 0 machine virtuelle dédiée et une formule de mise à l’échelle automatique en fonction du nombre de tâches en attente.

L’exemple de formule fourni ici entraîne le comportement suivant : Lors de sa création, le pool ne contient qu’une seule machine virtuelle. La métrique $PendingTasks définit le nombre de tâches dans l’état En cours d’exécution + Actif (en file d’attente). Cette formule recherche le nombre moyen de tâches en attente au cours des 180 dernières secondes et définit TargetDedicated en conséquence. Elle garantit que TargetDedicated ne va jamais au-delà de 25 machines virtuelles. Par conséquent, à mesure que de nouvelles tâches sont envoyées, le pool s’accroît automatiquement et, au fil de la réalisation des tâches, les machines virtuelles se libèrent une à une et la mise à l’échelle automatique réduit ces machines virtuelles. Vous pouvez ajuster startingNumberOfVMs et maxNumberofVMs selon vos besoins.

Formule de mise à l’échelle automatique :

```
startingNumberOfVMs = 1;
maxNumberofVMs = 25;
pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(180 * TimeInterval_Second);
pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(180 * TimeInterval_Second));
$TargetDedicated=min(maxNumberofVMs,pendingTaskSamples);
```

Pour plus d’informations, consultez [Mettre automatiquement à l’échelle les nœuds de calcul dans un pool Azure Batch](../batch/batch-automatic-scaling.md) .

Si le pool utilise la valeur par défaut du paramètre [autoScaleEvaluationInterval](/rest/api/batchservice/pool/enableautoscale), le service Batch peut mettre 15 à 30 minutes à préparer la machine virtuelle avant d’exécuter l’activité personnalisée. Si le pool utilise une autre valeur pour autoScaleEvaluationInterval, le service Batch peut prendre la durée d’autoScaleEvaluationInterval + 10 minutes.

## <a name="next-steps"></a>Étapes suivantes
Consultez les articles suivants qui expliquent comment transformer des données par d’autres moyens :

* [Activité U-SQL](transform-data-using-data-lake-analytics.md)
* [Activité Hive](transform-data-using-hadoop-hive.md)
* [Activité pig](transform-data-using-hadoop-pig.md)
* [Activité MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Activité de diffusion en continu Hadoop](transform-data-using-hadoop-streaming.md)
* [Activité Spark](transform-data-using-spark.md)
* [Activité Exécution par lots Azure Machine Learning Studio (classique)](transform-data-using-machine-learning.md)
* [Activité de procédure stockée](transform-data-using-stored-procedure.md)