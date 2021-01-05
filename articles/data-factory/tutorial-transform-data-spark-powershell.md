---
title: 'Transformer des données avec Spark dans Azure Data Factory '
description: Ce didacticiel fournit des instructions détaillées sur la transformation des données à l’aide d’une activité Spark dans Azure Data Factory.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/22/2018
author: nabhishek
ms.author: abnarain
manager: anandsub
ms.openlocfilehash: bef80cdeab32d14aeaae350adda869a8ea7b05c7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81409083"
---
# <a name="transform-data-in-the-cloud-by-using-spark-activity-in-azure-data-factory"></a>Transformer des données dans le cloud à l’aide d’une activité Spark dans Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Dans ce tutoriel, vous utilisez Azure PowerShell pour créer un pipeline Azure Data Factory qui transforme des données à l’aide d’une activité Spark et d’un service lié HDInsight à la demande. Dans ce tutoriel, vous allez effectuer les étapes suivantes :

> [!div class="checklist"]
> * Créer une fabrique de données. 
> * Créer et déployer des services liés.
> * Créer et déployer un pipeline. 
> * Démarrer une exécution de pipeline.
> * Surveiller l’exécution du pipeline.

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* **Compte Stockage Azure**. Vous créez un script Python et un fichier d’entrée, puis vous les téléchargez vers le stockage Azure. La sortie du programme Spark est stockée dans ce compte de stockage. Le cluster Spark sur demande utilise le même compte de stockage comme stockage principal.  
* **Azure PowerShell**. Suivez les instructions de la page [Installation et configuration d’Azure PowerShell](/powershell/azure/install-Az-ps).


### <a name="upload-python-script-to-your-blob-storage-account"></a>Télécharger un script Python dans votre compte de stockage d’objets Blob
1. Créez un fichier Python nommé **WordCount_Spark.py** avec le contenu suivant : 

    ```python
    import sys
    from operator import add
    
    from pyspark.sql import SparkSession
    
    def main():
        spark = SparkSession\
            .builder\
            .appName("PythonWordCount")\
            .getOrCreate()
            
        lines = spark.read.text("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/inputfiles/minecraftstory.txt").rdd.map(lambda r: r[0])
        counts = lines.flatMap(lambda x: x.split(' ')) \
            .map(lambda x: (x, 1)) \
            .reduceByKey(add)
        counts.saveAsTextFile("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/outputfiles/wordcount")
        
        spark.stop()
    
    if __name__ == "__main__":
        main()
    ```
2. Remplacez **&lt;storageAccountName&gt;** par le nom de votre compte de stockage Azure. Puis enregistrez le fichier. 
3. Dans votre stockage Blob Azure, créez un conteneur nommé **adftutorial** s’il n’existe pas. 
4. Créez un dossier nommé **spark**.
5. Créer un sous-dossier nommé **script** sous le dossier **spark**. 
6. Téléchargez le fichier **WordCount_Spark.py** dans le sous-dossier **script**. 


### <a name="upload-the-input-file"></a>Télécharger le fichier d’entrée
1. Créez un fichier nommé **minecraftstory.txt** avec du texte. Le programme Spark compte le nombre de mots dans ce texte. 
2. Créez un sous-dossier nommé `inputfiles` dans le dossier `spark`. 
3. Téléchargez le fichier `minecraftstory.txt` dans le sous-dossier `inputfiles`. 

## <a name="author-linked-services"></a>Créer des services liés
Vous créez deux services liés dans cette section : 
    
- Un service lié au stockage Azure relie un compte de stockage Azure à la fabrique de données. Ce stockage est utilisé par le cluster HDInsight à la demande. Il contient également le script Spark à exécuter. 
- Un service lié HDInsight à la demande. Azure Data Factory crée automatiquement un cluster HDInsight, exécute le programme Spark, puis supprime le cluster HDInsight à la fin de la période d’inactivité préconfigurée. 

### <a name="azure-storage-linked-service"></a>Service lié Stockage Azure
Créez un fichier JSON à l’aide de votre éditeur favori, copiez la définition JSON suivante d’un service lié au stockage Azure, puis enregistrez le fichier sous **MyStorageLinkedService.json**.  

```json
{
    "name": "MyStorageLinkedService",
    "properties": {
      "type": "AzureStorage",
      "typeProperties": {
        "connectionString": "DefaultEndpointsProtocol=https;AccountName=<storageAccountName>;AccountKey=<storageAccountKey>"
      }
    }
}
```
Mettez à jour les éléments &lt;storageAccountName&gt; et &lt;storageAccountKey&gt; avec le nom et la clé de votre compte de stockage Azure. 


### <a name="on-demand-hdinsight-linked-service"></a>Service lié HDInsight à la demande
Créez un fichier JSON à l’aide de votre éditeur favori, copiez la définition JSON suivante d’un service lié HDInsight Azure et enregistrez le fichier sous **MyOnDemandSparkLinkedService.json**.  

```json
{
    "name": "MyOnDemandSparkLinkedService",
    "properties": {
      "type": "HDInsightOnDemand",
      "typeProperties": {
        "clusterSize": 2,
        "clusterType": "spark",
        "timeToLive": "00:15:00",
        "hostSubscriptionId": "<subscriptionID> ",
        "servicePrincipalId": "<servicePrincipalID>",
        "servicePrincipalKey": {
          "value": "<servicePrincipalKey>",
          "type": "SecureString"
        },
        "tenant": "<tenant ID>",
        "clusterResourceGroup": "<resourceGroupofHDICluster>",
        "version": "3.6",
        "osType": "Linux",
        "clusterNamePrefix":"ADFSparkSample",
        "linkedServiceName": {
          "referenceName": "MyStorageLinkedService",
          "type": "LinkedServiceReference"
        }
      }
    }
}
```
Mettez à jour les valeurs des propriétés suivantes dans la définition de service lié : 

- **hostSubscriptionId**. Remplacez &lt;subscriptionID&gt; par l’ID de votre abonnement Azure. Le cluster HDInsight à la demande est créé dans cet abonnement. 
- **tenant** Remplacez &lt;tenantID&gt; par l’ID de votre client Azure. 
- **servicePrincipalId**, **servicePrincipalKey**. Remplacez &lt;servicePrincipalID&gt; et &lt;servicePrincipalKey&gt; par l’ID et la clé de votre principal de service dans Azure Active Directory. Ce principal de service doit être membre du rôle Contributeur de l’abonnement ou du groupe de ressources dans lequel le cluster est créé. Pour plus de détails, reportez-vous à l’article relatif à la [création de l’application Azure Active Directory et du principal du service à l’aide du portail](../active-directory/develop/howto-create-service-principal-portal.md). L’**ID de principal de service** est équivalent à l’*ID d’application* et une **clé de principal de service** est équivalente à la valeur d’un *secret client*.
- **clusterResourceGroup**. Remplacez &lt;resourceGroupOfHDICluster&gt; par le nom du groupe de ressources dans lequel le cluster HDInsight doit être créé. 

> [!NOTE]
> Azure HDInsight présente une limite relative au nombre total de cœurs que vous pouvez utiliser dans chaque région Azure prise en charge. Pour le service lié HDInsight à la demande, le cluster HDInsight sera créé au même emplacement de stockage Azure que celui utilisé comme stockage principal. Vérifiez que vous disposez des quotas de cœurs suffisants pour pouvoir créer le cluster avec succès. Pour plus d’informations, reportez-vous à l’article [Configurer des clusters dans HDInsight avec Hadoop, Spark, Kafka et bien plus encore](../hdinsight/hdinsight-hadoop-provision-linux-clusters.md). 


## <a name="author-a-pipeline"></a>Créer un pipeline 
Dans cette étape, vous allez créer un pipeline avec une activité Spark. L’activité utilise l’exemple **word count**. Téléchargez le contenu à partir de cet emplacement si vous ne l’avez pas encore fait.

Créez un fichier JSON dans votre éditeur favori, copiez la définition JSON suivante d’une définition de pipeline et enregistrez-la sous **MySparkOnDemandPipeline.json**. 

```json
{
  "name": "MySparkOnDemandPipeline",
  "properties": {
    "activities": [
      {
        "name": "MySparkActivity",
        "type": "HDInsightSpark",
        "linkedServiceName": {
            "referenceName": "MyOnDemandSparkLinkedService",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
          "rootPath": "adftutorial/spark",
          "entryFilePath": "script/WordCount_Spark.py",
          "getDebugInfo": "Failure",
          "sparkJobLinkedService": {
            "referenceName": "MyStorageLinkedService",
            "type": "LinkedServiceReference"
          }
        }
      }
    ]
  }
}
```

Notez les points suivants : 

- rootPath pointe vers le dossier Spark du conteneur adftutorial. 
- entryFilePath pointe vers le fichier WordCount_Spark.py dans le sous-dossier de script du dossier Spark. 


## <a name="create-a-data-factory"></a>Créer une fabrique de données 
Vous avez créé des définitions de service lié et de pipeline dans des fichiers JSON. Maintenant, nous allons créer une fabrique de données et déployer le service lié et les fichiers JSON de pipeline à l’aide des cmdlets PowerShell. Exécutez une par une les commandes PowerShell suivantes : 

1. Définissez les variables une par une.

    **Nom du groupe de ressources**
    ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup" 
    ```

    **Nom de Data Factory. Doit être globalement unique** 
    ```powershell
    $dataFactoryName = "MyDataFactory09102017"
    ```
    
    **Nom du pipeline**
    ```powershell
    $pipelineName = "MySparkOnDemandPipeline" # Name of the pipeline
    ```
2. Lancez **PowerShell**. Gardez Azure PowerShell ouvert jusqu’à la fin de ce guide de démarrage rapide. Si vous fermez puis rouvrez Azure PowerShell, vous devez réexécuter ces commandes. Pour obtenir la liste des régions Azure dans lesquelles Data Factory est actuellement disponible, sélectionnez les régions qui vous intéressent dans la page suivante, puis développez **Analytique** pour localiser **Data Factory** : [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/). Les magasins de données (Stockage Azure, Azure SQL Database, etc.) et les services de calcul (HDInsight, etc.) utilisés par la fabrique de données peuvent être proposés dans d’autres régions.

    Exécutez la commande suivante, puis saisissez le nom d’utilisateur et le mot de passe que vous avez utilisés pour la connexion au portail Azure :
        
    ```powershell
    Connect-AzAccount
    ```        
    Exécutez la commande suivante pour afficher tous les abonnements de ce compte :

    ```powershell
    Get-AzSubscription
    ```
    Exécutez la commande suivante pour sélectionner l’abonnement que vous souhaitez utiliser. Remplacez **SubscriptionId** par l’ID de votre abonnement Azure :

    ```powershell
    Select-AzSubscription -SubscriptionId "<SubscriptionId>"    
    ```  
3. Créez le groupe de ressources : ADFTutorialResourceGroup. 

    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location "East Us" 
    ```
4. Créez la fabrique de données. 

    ```powershell
     $df = Set-AzDataFactoryV2 -Location EastUS -Name $dataFactoryName -ResourceGroupName $resourceGroupName
    ```

    Exécutez la commande suivante pour afficher la sortie : 

    ```powershell
    $df
    ```
5. Basculez vers le dossier où vous avez créé des fichiers JSON et exécutez la commande suivante pour déployer un service lié au stockage Azure : 
       
    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "MyStorageLinkedService" -File "MyStorageLinkedService.json"
    ```
6. Exécutez la commande suivante pour déployer un service lié Spark à la demande : 
       
    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "MyOnDemandSparkLinkedService" -File "MyOnDemandSparkLinkedService.json"
    ```
7. Exécutez la commande suivante pour déployer un pipeline : 
       
    ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name $pipelineName -File "MySparkOnDemandPipeline.json"
    ```
    
## <a name="start-and-monitor-a-pipeline-run"></a>Démarrer et surveiller l’exécution d’un pipeline  

1. Démarrer une exécution de pipeline. Cette opération capture également l’ID d’exécution du pipeline pour une surveillance ultérieure.

    ```powershell
    $runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName $pipelineName  
    ```
2. Exécutez le script suivant pour vérifier en permanence l’état de l’exécution du pipeline jusqu’à la fin.

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
3. Voici la sortie de l’exemple d’exécution : 

    ```
    Pipeline run status: In Progress
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : 
    ActivityName      : MySparkActivity
    PipelineRunId     : 94e71d08-a6fa-4191-b7d1-cf8c71cb4794
    PipelineName      : MySparkOnDemandPipeline
    Input             : {rootPath, entryFilePath, getDebugInfo, sparkJobLinkedService}
    Output            : 
    LinkedServiceName : 
    ActivityRunStart  : 9/20/2017 6:33:47 AM
    ActivityRunEnd    : 
    DurationInMs      : 
    Status            : InProgress
    Error             :
    …
    
    Pipeline ' MySparkOnDemandPipeline' run finished. Result:
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : MyDataFactory09102017
    ActivityName      : MySparkActivity
    PipelineRunId     : 94e71d08-a6fa-4191-b7d1-cf8c71cb4794
    PipelineName      : MySparkOnDemandPipeline
    Input             : {rootPath, entryFilePath, getDebugInfo, sparkJobLinkedService}
    Output            : {clusterInUse, jobId, ExecutionProgress, effectiveIntegrationRuntime}
    LinkedServiceName : 
    ActivityRunStart  : 9/20/2017 6:33:47 AM
    ActivityRunEnd    : 9/20/2017 6:46:30 AM
    DurationInMs      : 763466
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    
    Activity Output section:
    "clusterInUse": "https://ADFSparkSamplexxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.azurehdinsight.net/"
    "jobId": "0"
    "ExecutionProgress": "Succeeded"
    "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)"
    Activity Error section:
    "errorCode": ""
    "message": ""
    "failureType": ""
    "target": "MySparkActivity"
    ```
4. Confirmez qu’un dossier nommé `outputfiles` est créé dans le dossier `spark` du conteneur adftutorial avec la sortie du programme Spark. 


## <a name="next-steps"></a>Étapes suivantes
Dans cet exemple, le pipeline copie les données d’un emplacement vers un autre dans un stockage Blob Azure. Vous avez appris à : 

> [!div class="checklist"]
> * Créer une fabrique de données. 
> * Créer et déployer des services liés.
> * Créer et déployer un pipeline. 
> * Démarrer une exécution de pipeline.
> * Surveiller l’exécution du pipeline.

Passez au didacticiel suivant pour découvrir comment transformer des données en exécutant un script Hive sur un cluster Azure HDInsight qui se trouve dans un réseau virtuel. 

> [!div class="nextstepaction"]
> [Didacticiel : transformer des données à l’aide de Hive dans un réseau virtuel Azure](tutorial-transform-data-hive-virtual-network.md).





