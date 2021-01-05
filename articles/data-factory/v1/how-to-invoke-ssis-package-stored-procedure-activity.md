---
title: Appeler un package SSIS à l’aide d’Azure Data Factory – Activité de procédure stockée
description: Cet article décrit comment appeler un package SQL Server Integration Services (SSIS) à partir d’un pipeline Azure Data Factory à l’aide de l’activité de procédure stockée.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: conceptual
ms.date: 01/19/2018
ms.author: jingwang
ms.openlocfilehash: c7a99e7e5f27f8c3503c7fa6124d27cfc4e7f4a4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012831"
---
# <a name="invoke-an-ssis-package-using-stored-procedure-activity-in-azure-data-factory"></a>Appeler un package SSIS à l’aide de l’activité de procédure stockée dans Azure Data Factory
Cet article décrit comment appeler un package SSIS à partir d’un pipeline Azure Data Factory à l’aide d’une activité de procédure stockée. 

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Appeler les packages SSIS à l’aide d’une activité de procédure stockée](../how-to-invoke-ssis-package-stored-procedure-activity.md).

## <a name="prerequisites"></a>Prérequis

### <a name="azure-sql-database"></a>Azure SQL Database 
La procédure pas à pas décrite dans cet article utilise Azure SQL Database. Vous pouvez également utiliser Azure SQL Managed Instance.

### <a name="create-an-azure-ssis-integration-runtime"></a>Créer un runtime d’intégration Azure-SSIS
Créez un runtime d’intégration Azure-SSIS si vous n’en avez pas en suivant les instructions pas à pas fournies dans le [Tutoriel : Déployer des packages SSIS](../tutorial-deploy-ssis-packages-azure.md). Vous ne pouvez pas utiliser Data Factory version 1 pour créer un runtime d’intégration Azure-SSIS. 

## <a name="azure-powershell"></a>Azure PowerShell
Dans cette section, vous utilisez Azure PowerShell pour créer un pipeline Data Factory avec une activité de procédure stockée qui appelle un package SSIS.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Installez les modules Azure PowerShell les plus récents en suivant les instructions décrites dans [Comment installer et configurer Azure PowerShell](/powershell/azure/install-az-ps).

### <a name="create-a-data-factory"></a>Créer une fabrique de données
La procédure suivante décrit les étapes permettant de créer une fabrique de données. Vous créez un pipeline avec une activité de procédure stockée dans cette fabrique de données. L’activité de procédure stockée exécute une procédure stockée dans la base de données SSISDB pour exécuter votre package SSIS.

1. Définissez une variable pour le nom du groupe de ressources que vous utiliserez ultérieurement dans les commandes PowerShell. Copiez le texte de commande suivant dans PowerShell, spécifiez un nom pour le [groupe de ressources Azure](../../azure-resource-manager/management/overview.md) entre des guillemets doubles, puis exécutez la commande. Par exemple : `"adfrg"`. 
   
     ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup";
    ```

    Si le groupe de ressources existe déjà, vous pouvez ne pas le remplacer. Affectez une valeur différente à la variable `$ResourceGroupName` et exécutez à nouveau la commande
2. Pour créer le groupe de ressources Azure, exécutez la commande suivante : 

    ```powershell
    $ResGrp = New-AzResourceGroup $resourceGroupName -location 'eastus'
    ``` 
    Si le groupe de ressources existe déjà, vous pouvez ne pas le remplacer. Affectez une valeur différente à la variable `$ResourceGroupName` et exécutez à nouveau la commande. 
3. Définissez une variable pour le nom de la fabrique de données. 

    > [!IMPORTANT]
    >  Mettez à jour le nom de la fabrique de données afin qu’il soit globalement unique. 

    ```powershell
    $DataFactoryName = "ADFTutorialFactory";
    ```

5. Pour créer la fabrique de données, exécutez la cmdlet suivante **New-AzDataFactory**, à l’aide des propriétés Location et ResourceGroupName à partir de la variable $ResGrp : 
    
    ```powershell       
    $df = New-AzDataFactory -ResourceGroupName $ResourceGroupName -Name $dataFactoryName -Location "East US"
    ```

Notez les points suivants :

* Le nom de la fabrique de données Azure doit être un nom global unique. Si vous recevez l’erreur suivante, changez le nom, puis réessayez.

    ```
    The specified Data Factory name 'ADFTutorialFactory' is already in use. Data Factory names must be globally unique.
    ```
* Pour créer des instances de fabrique de données, le compte d’utilisateur que vous utilisez pour vous connecter à Azure doit être un membre des rôles **contributeur** ou **propriétaire**, ou un **administrateur** de l’abonnement Azure.

### <a name="create-an-azure-sql-database-linked-service"></a>Créer un service lié Azure SQL Database
Créez un service lié pour lier votre base de données dans Azure SQL Database qui héberge le catalogue SSIS à votre fabrique de données. Data Factory utilise les informations de ce service lié pour se connecter à la base de données SSISDB, et exécute une procédure stockée pour exécuter un package SSIS. 

1. Créez un fichier JSON nommé **AzureSqlDatabaseLinkedService.json** dans le dossier **C:\ADF\RunSSISPackage** avec le contenu suivant : 

    > [!IMPORTANT]
    > Remplacez &lt;servername&gt;, &lt;username&gt;@&lt;servername&gt; et &lt;password&gt; par les valeurs de votre base de données Azure SQL avant d’enregistrer le fichier.

    ```json
    {
        "name": "AzureSqlDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=SSISDB;User ID=<username>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
        }
    ```
2. Dans **Azure PowerShell**, basculez vers le dossier **C:\ADF\RunSSISPackage**.
3. Exécutez l’applet de commande **New-AzDataFactoryLinkedService** pour créer le service lié : **AzureSqlDatabaseLinkedService**. 

    ```powershell
    New-AzDataFactoryLinkedService $df -File ".\AzureSqlDatabaseLinkedService.json"
    ```

### <a name="create-an-output-dataset"></a>Créer un jeu de données de sortie
Ce jeu de données de sortie est un jeu de données factice qui détermine la programmation du pipeline. Notez que la fréquence est définie sur Heure et l’intervalle sur 1. Ainsi, le pipeline s’exécute une fois par heure entre les heures de début et de fin du pipeline. 

1. Créez un fichier OutputDataset.json avec le contenu suivant : 
    
    ```json
    {
        "name": "sprocsampleout",
        "properties": {
            "type": "AzureSqlTable",
            "linkedServiceName": "AzureSqlLinkedService",
            "typeProperties": { },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
    }
    ```
2. Exécutez la cmdlet **New-AzDataFactoryDataset** pour créer un jeu de données. 

    ```powershell
    New-AzDataFactoryDataset $df -File ".\OutputDataset.json"
    ```

### <a name="create-a-pipeline-with-stored-procedure-activity"></a>Créer un pipeline avec une activité de procédure stockée 
Lors de cette étape, vous allez créer un pipeline avec une activité de procédure stockée. L’activité appelle la procédure stockée sp_executesql pour exécuter votre package SSIS. 

1. Créez un fichier JSON nommé **MyPipeline.json** dans le dossier **C:\ADF\RunSSISPackage** avec le contenu suivant :

    > [!IMPORTANT]
    > Remplacez &lt;folder name&gt;, &lt;project name&gt; et &lt;package name&gt; par des noms de dossier, projet et package dans le catalogue SSIS avant d’enregistrer le fichier.

    ```json
    {
        "name": "MyPipeline",
        "properties": {
            "activities": [{
                "name": "SprocActivitySample",
                "type": "SqlServerStoredProcedure",
                "typeProperties": {
                    "storedProcedureName": "sp_executesql",
                    "storedProcedureParameters": {
                        "stmt": "DECLARE @return_value INT, @exe_id BIGINT, @err_msg NVARCHAR(150)    EXEC @return_value=[SSISDB].[catalog].[create_execution] @folder_name=N'<folder name>', @project_name=N'<project name>', @package_name=N'<package name>', @use32bitruntime=0, @runinscaleout=1, @useanyworker=1, @execution_id=@exe_id OUTPUT    EXEC [SSISDB].[catalog].[set_execution_parameter_value] @exe_id, @object_type=50, @parameter_name=N'SYNCHRONIZED', @parameter_value=1    EXEC [SSISDB].[catalog].[start_execution] @execution_id=@exe_id, @retry_count=0    IF(SELECT [status] FROM [SSISDB].[catalog].[executions] WHERE execution_id=@exe_id)<>7 BEGIN SET @err_msg=N'Your package execution did not succeed for execution ID: ' + CAST(@exe_id AS NVARCHAR(20)) RAISERROR(@err_msg,15,1) END"
                    }
                },
                "outputs": [{
                    "name": "sprocsampleout"
                }],
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                }
            }],
            "start": "2017-10-01T00:00:00Z",
            "end": "2017-10-01T05:00:00Z",
            "isPaused": false
        }
    }    
    ```

2. Pour créer le pipeline : **RunSSISPackagePipeline**, exécutez la cmdlet **New-AzDataFactoryPipeline**.

    ```powershell
    $DFPipeLine = New-AzDataFactoryPipeline -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "RunSSISPackagePipeline" -DefinitionFile ".\RunSSISPackagePipeline.json"
    ```

### <a name="monitor-the-pipeline-run"></a>Surveiller l’exécution du pipeline.

1. Exécutez **Get-AzDataFactorySlice** pour obtenir des détails sur toutes les tranches du jeu de données de sortie**, qui est la table de sortie du pipeline.

    ```powershell
    Get-AzDataFactorySlice $df -DatasetName sprocsampleout -StartDateTime 2017-10-01T00:00:00Z
    ```
    Notez que la valeur StartDateTime que vous spécifiez ici est identique à l’heure de début que vous avez spécifiée dans le pipeline de JSON. 
1. Exécutez **Get-AzDataFactoryRun** pour obtenir les détails des exécutions d’activité pour un segment spécifique.

    ```powershell
    Get-AzDataFactoryRun $df -DatasetName sprocsampleout -StartDateTime 2017-10-01T00:00:00Z
    ```

    Vous pouvez continuer d’exécuter cette applet de commande jusqu’à ce que le segment apparaisse dans l’état **Prêt** ou **En échec**. 

    Vous pouvez exécuter la requête suivante dans la base de données SSISDB de votre serveur pour vérifier que le package s'est exécuté. 

    ```sql
    select * from catalog.executions
    ```

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur l’activité de procédure stockée, consultez l’article [Activité de procédure stockée](data-factory-stored-proc-activity.md).