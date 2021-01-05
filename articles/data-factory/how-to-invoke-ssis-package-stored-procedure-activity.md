---
title: Exécuter le package SSIS avec l’activité de procédure stockée - Azure
description: Cet article décrit comment exécuter un package SQL Server Integration Services (SSIS) dans un pipeline Azure Data Factory à l’aide de l’activité de procédure stockée.
services: data-factory
documentationcenter: ''
author: swinarko
manager: anandsub
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: sawinark
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6cb3bf53db14c6c01290abea7a5c48c332a07632
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92634876"
---
# <a name="run-an-ssis-package-with-the-stored-procedure-activity-in-azure-data-factory"></a>Exécuter un package SSIS avec l’activité de procédure stockée dans Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article décrit comment exécuter un package SSIS dans un pipeline Azure Data Factory à l’aide d’une activité de procédure stockée. 

## <a name="prerequisites"></a>Prérequis

### <a name="azure-sql-database"></a>Azure SQL Database 
Dans le cadre de la procédure pas à pas de cet article, Azure SQL Database est utilisé pour héberger le catalogue SSIS. Vous pouvez également utiliser Azure SQL Managed Instance.

## <a name="create-an-azure-ssis-integration-runtime"></a>Créer un runtime d’intégration Azure-SSIS
Créez un runtime d’intégration Azure-SSIS si vous n’en avez pas en suivant les instructions pas à pas fournies dans le [Tutoriel : Déployer des packages SSIS](./tutorial-deploy-ssis-packages-azure.md).

## <a name="data-factory-ui-azure-portal"></a>Interface utilisateur de Data Factory (portail Azure)
Dans cette section, vous utilisez l’interface utilisateur de Data Factory pour créer un pipeline Data Factory avec une activité de procédure stockée qui appelle un package SSIS.

### <a name="create-a-data-factory"></a>Créer une fabrique de données
La première étape consiste à créer une fabrique de données à l’aide du portail Azure. 

1. Lancez le navigateur web **Microsoft Edge** ou **Google Chrome** . L’interface utilisateur de Data Factory n’est actuellement prise en charge que par les navigateurs web Microsoft Edge et Google Chrome.
2. Accédez au [portail Azure](https://portal.azure.com). 
3. Cliquez sur **Nouveau** dans le menu de gauche, puis sur **Données + analyse** et sur **Data Factory** . 
   
   ![Nouveau -> DataFactory](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-azure-data-factory-menu.png)
2. Dans la page **Nouvelle fabrique de données** , entrez **ADFTutorialDataFactory** comme **nom** . 
      
     ![Page Nouvelle fabrique de données](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-azure-data-factory.png)
 
   Le nom de la fabrique de données Azure doit être un nom **global unique** . Si l’erreur suivante s’affiche pour le champ du nom, changez le nom de la fabrique de données (par exemple, votrenomADFTutorialDataFactory). Consultez l’article [Data Factory - Règles d’affectation des noms](naming-rules.md) pour savoir comment nommer les artefacts Data Factory.
  
     ![Nom indisponible - erreur](./media/how-to-invoke-ssis-package-stored-procedure-activity/name-not-available-error.png)
3. Sélectionnez l’ **abonnement** Azure dans lequel vous voulez créer la fabrique de données. 
4. Pour le **groupe de ressources** , effectuez l’une des opérations suivantes :
     
   - Sélectionnez **Utiliser l’existant** , puis sélectionnez un groupe de ressources existant dans la liste déroulante. 
   - Sélectionnez **Créer** , puis entrez le nom d’un groupe de ressources.   
         
     Pour plus d’informations sur les groupes de ressources, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/overview.md).  
4. Sélectionnez **V2** pour la **version** .
5. Sélectionnez **l’emplacement** de la fabrique de données. Seuls les emplacements pris en charge par Data Factory sont affichés dans la liste déroulante. Les magasins de données (Stockage Azure, Azure SQL Database, etc.) et les services de calcul (HDInsight, etc.) utilisés par la fabrique de données peuvent se trouver dans d’autres emplacements.
6. Sélectionnez **Épingler au tableau de bord** .     
7. Cliquez sur **Créer** .
8. Sur le tableau de bord, vous voyez la vignette suivante avec l’état : **Déploiement de Data Factory** . 

     ![mosaïque déploiement de fabrique de données](media//how-to-invoke-ssis-package-stored-procedure-activity/deploying-data-factory.png)
9. Une fois la création terminée, la page **Data Factory** s’affiche comme sur l’image.
   
     ![Page d’accueil Data Factory](./media/how-to-invoke-ssis-package-stored-procedure-activity/data-factory-home-page.png)
10. Cliquez sur la vignette **Créer et surveiller** pour lancer l’application d’interface utilisateur (IU) d’Azure Data Factory dans un onglet séparé. 

### <a name="create-a-pipeline-with-stored-procedure-activity"></a>Créer un pipeline avec une activité de procédure stockée
Lors de cette étape, vous utilisez l’interface utilisateur de Data Factory pour créer un pipeline. Vous ajoutez une activité de procédure stockée au pipeline et vous le configurez pour exécuter le package SSIS à l’aide de la procédure stockée sp_executesql. 

1. Dans la page de prise en main, cliquez sur **Créer un pipeline** : 

    ![Page de prise en main](./media/how-to-invoke-ssis-package-stored-procedure-activity/get-started-page.png)
2. Dans la boîte à outils **Activités** , développez **Général** et glissez-déplacez l’activité **Procédure stockée** vers la surface du concepteur de pipeline. 

    ![Glisser-déplacer l’activité de procédure stockée](./media/how-to-invoke-ssis-package-stored-procedure-activity/drag-drop-sproc-activity.png)
3. Dans la fenêtre de propriétés de l’activité de procédure stockée, basculez vers l’onglet **Compte SQL** , puis cliquez sur **+ Nouveau** . Vous créez une connexion à la base de données dans Azure SQL Database qui héberge le catalogue SSIS (base de données SSIDB). 
   
    ![Bouton de nouveau service lié](./media/how-to-invoke-ssis-package-stored-procedure-activity/new-linked-service-button.png)
4. Dans la fenêtre **Nouveau service lié** , procédez comme suit : 

    1. Sélectionnez **Azure SQL Database** comme **Type** .
    2. Sélectionnez le runtime d’intégration Azure **Par défaut** pour vous connecter à la base de données Azure SQL qui héberge la base de données `SSISDB`.
    3. Dans le champ **Nom du serveur** , sélectionnez la base de données Azure SQL qui héberge la base de données SSISDB.
    4. Sélectionnez **SSISDB** comme **Nom de la base de données** .
    5. Dans **Nom d’utilisateur** , entrez le nom de l’utilisateur qui a accès à la base de données.
    6. Dans **Mot de passe** , entrez le mot de passe de l’utilisateur. 
    7. Testez la connexion à la base de données en cliquant sur le bouton **Tester la connexion** .
    8. Enregistrez le service lié en cliquant sur le bouton **Enregistrer** . 

        ![Capture d’écran montrant le processus d’ajout d’un nouveau service lié.](./media/how-to-invoke-ssis-package-stored-procedure-activity/azure-sql-database-linked-service-settings.png)
5. Dans la fenêtre de propriétés, basculez vers l’onglet **Procédure stockée** à partir de l’onglet **Compte SQL** et effectuez les étapes suivantes : 

    1. Sélectionnez **Modifier** . 
    2. Dans le champ **Nom de la procédure stockée** , entrez `sp_executesql`. 
    3. Cliquez sur **+ Nouveau** dans la section **Paramètres de procédure stockée** . 
    4. Comme **Nom** du paramètre, entrez **stmt** . 
    5. Comme **Type** de paramètre, entrez **String** . 
    6. Comme **Valeur** du paramètre, entrez la requête SQL suivante :

        Dans la requête SQL, spécifiez les valeurs correctes pour les paramètres **folder_name** , **project_name** et **package_name** . 

        ```sql
        DECLARE @return_value INT, @exe_id BIGINT, @err_msg NVARCHAR(150)    EXEC @return_value=[SSISDB].[catalog].[create_execution] @folder_name=N'<FOLDER name in SSIS Catalog>', @project_name=N'<PROJECT name in SSIS Catalog>', @package_name=N'<PACKAGE name>.dtsx', @use32bitruntime=0, @runinscaleout=1, @useanyworker=1, @execution_id=@exe_id OUTPUT    EXEC [SSISDB].[catalog].[set_execution_parameter_value] @exe_id, @object_type=50, @parameter_name=N'SYNCHRONIZED', @parameter_value=1    EXEC [SSISDB].[catalog].[start_execution] @execution_id=@exe_id, @retry_count=0    IF(SELECT [status] FROM [SSISDB].[catalog].[executions] WHERE execution_id=@exe_id)<>7 BEGIN SET @err_msg=N'Your package execution did not succeed for execution ID: ' + CAST(@exe_id AS NVARCHAR(20)) RAISERROR(@err_msg,15,1) END
        ```

        ![Service lié Azure SQL Database](./media/how-to-invoke-ssis-package-stored-procedure-activity/stored-procedure-settings.png)
6. Pour valider la configuration du pipeline, cliquez sur **Valider** dans la barre d’outils. Pour fermer le **Rapport de validation de pipeline** , cliquez sur **>>** .

    ![Valider le pipeline](./media/how-to-invoke-ssis-package-stored-procedure-activity/validate-pipeline.png)
7. Publiez le pipeline dans Data Factory en cliquant sur le bouton **Publier tout** . 

    ![Publish](./media/how-to-invoke-ssis-package-stored-procedure-activity/publish-all-button.png)    

### <a name="run-and-monitor-the-pipeline"></a>Exécuter et surveiller le pipeline
Dans cette section, vous déclenchez une exécution du pipeline puis vous la surveillez. 

1. Pour déclencher une exécution de pipeline, cliquez sur **Déclencher** dans la barre d’outils, puis sur **Déclencher maintenant** . 

    ![Déclencher maintenant](media/how-to-invoke-ssis-package-stored-procedure-activity/trigger-now.png)

2. Dans la fenêtre **Exécution du pipeline** , sélectionnez **Terminer** . 
3. Basculez vers l’onglet **Surveiller** sur la gauche. Vous voyez l’exécution de pipeline et son état, ainsi que d’autres informations (telles que l’heure de début d’exécution). Pour actualiser la vue, cliquez sur **Actualiser** .

    ![Exécutions de pipeline](./media/how-to-invoke-ssis-package-stored-procedure-activity/pipeline-runs.png)

3. Cliquez sur le lien **Afficher les exécutions d’activités** dans la colonne **Actions** . Une seule exécution d’activité est affichée, étant donnée que le pipeline n’a qu’une seule activité (activité de procédure stockée).

    ![Exécutions d’activités](./media/how-to-invoke-ssis-package-stored-procedure-activity/activity-runs.png)

4. Vous pouvez exécuter la **requête** suivante sur la base de données SSISDB dans SQL Database pour vérifier que le package s’est exécuté. 

    ```sql
    select * from catalog.executions
    ```

    ![Vérifier les exécutions de package](./media/how-to-invoke-ssis-package-stored-procedure-activity/verify-package-executions.png)


> [!NOTE]
> Vous pouvez également créer un déclencheur planifié pour votre pipeline afin que le pipeline s’exécute selon une planification (horaire, quotidienne, et ainsi de suite). Pour obtenir un exemple, consultez [Créer une fabrique de données - Interface utilisateur de Data Factory](quickstart-create-data-factory-portal.md#trigger-the-pipeline-on-a-schedule).

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Dans cette section, vous utilisez Azure PowerShell pour créer un pipeline Data Factory avec une activité de procédure stockée qui appelle un package SSIS. 

Installez les modules Azure PowerShell les plus récents en suivant les instructions décrites dans [Comment installer et configurer Azure PowerShell](/powershell/azure/install-az-ps). 

### <a name="create-a-data-factory"></a>Créer une fabrique de données
Vous pouvez utiliser la fabrique de données qui a le runtime d’intégration Azure-SSIS ou créer une fabrique de données distincte. La procédure suivante décrit les étapes permettant de créer une fabrique de données. Vous créez un pipeline avec une activité de procédure stockée dans cette fabrique de données. L’activité de procédure stockée exécute une procédure stockée dans la base de données SSISDB pour exécuter votre package SSIS. 

1. Définissez une variable pour le nom du groupe de ressources que vous utiliserez ultérieurement dans les commandes PowerShell. Copiez le texte de commande suivant dans PowerShell, spécifiez un nom pour le [groupe de ressources Azure](../azure-resource-manager/management/overview.md) entre des guillemets doubles, puis exécutez la commande. Par exemple : `"adfrg"`. 
   
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

5. Pour créer la fabrique de données, exécutez l’applet de commande suivante **Set-AzDataFactoryV2** , à l’aide des propriétés Location et ResourceGroupName à partir de la variable $ResGrp : 
    
    ```powershell       
    $DataFactory = Set-AzDataFactoryV2 -ResourceGroupName $ResGrp.ResourceGroupName -Location $ResGrp.Location -Name $dataFactoryName 
    ```

Notez les points suivants :

* Le nom de la fabrique de données Azure doit être un nom global unique. Si vous recevez l’erreur suivante, changez le nom, puis réessayez.

    ```
    The specified Data Factory name 'ADFv2QuickStartDataFactory' is already in use. Data Factory names must be globally unique.
    ```
* Pour créer des instances de fabrique de données, le compte d’utilisateur que vous utilisez pour vous connecter à Azure doit être un membre des rôles **contributeur** ou **propriétaire** , ou un **administrateur** de l’abonnement Azure.
* Pour obtenir la liste des régions Azure dans lesquelles Data Factory est actuellement disponible, sélectionnez les régions qui vous intéressent dans la page suivante, puis développez **Analytique** pour localiser **Data Factory**  : [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/). Les magasins de données (Stockage Azure, Azure SQL Database, etc.) et les services de calcul (HDInsight, etc.) utilisés par la fabrique de données peuvent être proposés dans d’autres régions.

### <a name="create-an-azure-sql-database-linked-service"></a>Créer un service lié Azure SQL Database
Créez un service lié pour lier votre base de données qui héberge le catalogue SSIS à votre fabrique de données. Data Factory utilise les informations de ce service lié pour se connecter à la base de données SSISDB, et exécute une procédure stockée pour exécuter un package SSIS. 

1. Créez un fichier JSON nommé **AzureSqlDatabaseLinkedService.json** dans le dossier **C:\ADF\RunSSISPackage** avec le contenu suivant : 

    > [!IMPORTANT]
    > Remplacez &lt;servername&gt;, &lt;username&gt; et &lt;password&gt; par les valeurs de votre base de données Azure SQL avant d’enregistrer le fichier.

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

2. Dans **Azure PowerShell** , basculez vers le dossier **C:\ADF\RunSSISPackage** .

3. Exécutez la cmdlet **Set-AzDataFactoryV2LinkedService** pour créer le service lié : **AzureSqlDatabaseLinkedService** . 

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "AzureSqlDatabaseLinkedService" -File ".\AzureSqlDatabaseLinkedService.json"
    ```

### <a name="create-a-pipeline-with-stored-procedure-activity"></a>Créer un pipeline avec une activité de procédure stockée 
Lors de cette étape, vous allez créer un pipeline avec une activité de procédure stockée. L’activité appelle la procédure stockée sp_executesql pour exécuter votre package SSIS. 

1. Créez un fichier JSON nommé **RunSSISPackagePipeline.json** dans le dossier **C:\ADF\RunSSISPackage** avec le contenu suivant :

    > [!IMPORTANT]
    > Remplacez &lt;FOLDER NAME&gt;, &lt;PROJECT NAME&gt; et &lt;PACKAGE NAME&gt; par des noms de dossier, projet et package dans le catalogue SSIS avant d’enregistrer le fichier. 

    ```json
    {
        "name": "RunSSISPackagePipeline",
        "properties": {
            "activities": [
                {
                    "name": "My SProc Activity",
                    "description":"Runs an SSIS package",
                    "type": "SqlServerStoredProcedure",
                    "linkedServiceName": {
                        "referenceName": "AzureSqlDatabaseLinkedService",
                        "type": "LinkedServiceReference"
                    },
                    "typeProperties": {
                        "storedProcedureName": "sp_executesql",
                        "storedProcedureParameters": {
                            "stmt": {
                                "value": "DECLARE @return_value INT, @exe_id BIGINT, @err_msg NVARCHAR(150)    EXEC @return_value=[SSISDB].[catalog].[create_execution] @folder_name=N'<FOLDER NAME>', @project_name=N'<PROJECT NAME>', @package_name=N'<PACKAGE NAME>', @use32bitruntime=0, @runinscaleout=1, @useanyworker=1, @execution_id=@exe_id OUTPUT    EXEC [SSISDB].[catalog].[set_execution_parameter_value] @exe_id, @object_type=50, @parameter_name=N'SYNCHRONIZED', @parameter_value=1    EXEC [SSISDB].[catalog].[start_execution] @execution_id=@exe_id, @retry_count=0    IF(SELECT [status] FROM [SSISDB].[catalog].[executions] WHERE execution_id=@exe_id)<>7 BEGIN SET @err_msg=N'Your package execution did not succeed for execution ID: ' + CAST(@exe_id AS NVARCHAR(20)) RAISERROR(@err_msg,15,1) END"
                            }
                        }
                    }
                }
            ]
        }
    }
    ```

2. Pour créer le pipeline : **RunSSISPackagePipeline** , exécutez la cmdlet **Set-AzDataFactoryV2Pipeline** .

    ```powershell
    $DFPipeLine = Set-AzDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -Name "RunSSISPackagePipeline" -DefinitionFile ".\RunSSISPackagePipeline.json"
    ```

    Voici l'exemple de sortie :

    ```
    PipelineName      : Adfv2QuickStartPipeline
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {CopyFromBlobToBlob}
    Parameters        : {[inputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification], [outputPath, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
    ```

### <a name="create-a-pipeline-run"></a>Créer une exécution du pipeline
Utilisez la cmdlet **Invoke-AzDataFactoryV2Pipeline** pour exécuter le pipeline. L’applet de commande renvoie l’ID d’exécution du pipeline pour permettre une surveillance ultérieure.

```powershell
$RunId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $DataFactory.DataFactoryName -ResourceGroupName $ResGrp.ResourceGroupName -PipelineName $DFPipeLine.Name
```

### <a name="monitor-the-pipeline-run"></a>Surveiller l’exécution du pipeline.

Exécutez le script PowerShell suivant afin de vérifier continuellement l’état de l’exécution du pipeline jusqu’à la fin de la copie des données. Copiez/collez le script suivant dans la fenêtre PowerShell et appuyez sur ENTRÉE. 

```powershell
while ($True) {
    $Run = Get-AzDataFactoryV2PipelineRun -ResourceGroupName $ResGrp.ResourceGroupName -DataFactoryName $DataFactory.DataFactoryName -PipelineRunId $RunId

    if ($Run) {
        if ($run.Status -ne 'InProgress') {
            Write-Output ("Pipeline run finished. The status is: " +  $Run.Status)
            $Run
            break
        }
        Write-Output  "Pipeline is running...status: InProgress"
    }

    Start-Sleep -Seconds 10
}   
```

### <a name="create-a-trigger"></a>Créer un déclencheur
À l’étape précédente, vous avez appelé le pipeline à la demande. Vous pouvez également créer un déclencheur de planification pour exécuter le pipeline d’après une planification (horaire, quotidienne, etc.).

1. Créez un fichier JSON nommé **MyTrigger.json** dans le dossier **C:\ADF\RunSSISPackage** avec le contenu suivant : 

    ```json
    {
        "properties": {
            "name": "MyTrigger",
            "type": "ScheduleTrigger",
            "typeProperties": {
                "recurrence": {
                    "frequency": "Hour",
                    "interval": 1,
                    "startTime": "2017-12-07T00:00:00-08:00",
                    "endTime": "2017-12-08T00:00:00-08:00"
                }
            },
            "pipelines": [{
                    "pipelineReference": {
                        "type": "PipelineReference",
                        "referenceName": "RunSSISPackagePipeline"
                    },
                    "parameters": {}
                }
            ]
        }
    }    
    ```
2. Dans **Azure PowerShell** , basculez vers le dossier **C:\ADF\RunSSISPackage** .
3. Exécutez la cmdlet **Set-AzDataFactoryV2Trigger** pour créer le déclencheur. 

    ```powershell
    Set-AzDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName -DataFactoryName $DataFactory.DataFactoryName -Name "MyTrigger" -DefinitionFile ".\MyTrigger.json"
    ```
4. Par défaut, le déclencheur est arrêté. Démarrez le déclencheur en exécutant la cmdlet **Start-AzDataFactoryV2Trigger** . 

    ```powershell
    Start-AzDataFactoryV2Trigger -ResourceGroupName $ResGrp.ResourceGroupName -DataFactoryName $DataFactory.DataFactoryName -Name "MyTrigger" 
    ```
5. Vérifiez que le déclencheur est démarré en exécutant la cmdlet **Get-AzDataFactoryV2Trigger** . 

    ```powershell
    Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name "MyTrigger"     
    ```    
6. Exécutez la commande ci-dessous après l’heure suivante. Par exemple, si l’heure actuelle est 15h25 UTC, exécutez la commande à 16h00 UTC. 
    
    ```powershell
    Get-AzDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -TriggerName "MyTrigger" -TriggerRunStartedAfter "2017-12-06" -TriggerRunStartedBefore "2017-12-09"
    ```

    Vous pouvez exécuter la requête suivante sur la base de données SSISDB dans SQL Database pour vérifier que le package s’est exécuté. 

    ```sql
    select * from catalog.executions
    ```


## <a name="next-steps"></a>Étapes suivantes
Vous pouvez également surveiller le pipeline à l’aide du portail Azure. Pour obtenir des instructions pas à pas, consultez [Surveiller le pipeline](quickstart-create-data-factory-resource-manager-template.md#monitor-the-pipeline).