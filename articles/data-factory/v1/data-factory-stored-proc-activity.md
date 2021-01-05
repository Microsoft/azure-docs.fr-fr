---
title: Activité de procédure stockée SQL Server
description: Découvrez comment utiliser l’activité de procédure stockée SQL Server pour appeler une procédure stockée dans Azure SQL Database ou Azure Synapse Analytics à partir d’un pipeline Data Factory.
services: data-factory
documentationcenter: ''
ms.assetid: 1c46ed69-4049-44ec-9b46-e90e964a4a8e
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
author: nabhishek
ms.author: abnarain
manager: anandsub
robots: noindex
ms.openlocfilehash: f702633f6311d4ae4a79452e9b1c9176125c56f9
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97508397"
---
# <a name="sql-server-stored-procedure-activity"></a>Activité de procédure stockée SQL Server
> [!div class="op_single_selector" title1="Activités de transformation"]
> * [Activité Hive](data-factory-hive-activity.md)
> * [Activité pig](data-factory-pig-activity.md)
> * [Activité MapReduce](data-factory-map-reduce.md)
> * [Activité de diffusion en continu Hadoop](data-factory-hadoop-streaming-activity.md)
> * [Activité Spark](data-factory-spark.md)
> * [Activité Exécution par lots Azure Machine Learning studio (classique)](data-factory-azure-ml-batch-execution-activity.md)
> * [Activité Mettre à jour une ressource Azure Machine Learning studio (classique)](data-factory-azure-ml-update-resource-activity.md)
> * [Activité de procédure stockée](data-factory-stored-proc-activity.md)
> * [Activité U-SQL Data Lake Analytics](data-factory-usql-activity.md)
> * [Activité personnalisée .NET](data-factory-use-custom-activities.md)

> [!NOTE]
> Cet article s’applique à la version 1 d’Azure Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Transformer des données à l’aide d’une activité de procédure stockée dans Data Factory](../transform-data-using-stored-procedure.md).

## <a name="overview"></a>Vue d’ensemble
Vous utilisez des activités de transformation dans un [pipeline](data-factory-create-pipelines.md) Data Factory pour transformer et traiter des données brutes en prévisions et en analyses. L’activité de procédure stockée est l’une des activités de transformation prises en charge par Data Factory. Cet article s'appuie sur l'article [Activités de transformation des données](data-factory-data-transformation-activities.md) qui présente une vue d'ensemble de la transformation des données et les activités de transformation prises en charge dans Data Factory.

Vous pouvez utiliser l’activité de procédure stockée pour appeler une procédure stockée dans l’une des banques de données suivantes dans votre entreprise ou sur une machine virtuelle Azure :

- Azure SQL Database
- Azure Synapse Analytics
- Base de données SQL Server Si vous utilisez SQL Server, installez la passerelle de gestion des données sur l’ordinateur qui héberge la base de données ou sur un autre ordinateur ayant accès à la base de données. La passerelle de gestion des données est un composant qui connecte des sources de données locales ou se trouvant sur une machine virtuelle Azure à des services cloud de manière gérée et sécurisée. Consultez l’article [Passerelle de gestion des données](data-factory-data-management-gateway.md) pour plus d’informations sur la passerelle.

> [!IMPORTANT]
> Lorsque vous copiez des données dans Azure SQL Database ou SQL Server, vous pouvez configurer l’élément **SqlSink** dans l’activité de copie pour appeler une procédure stockée en utilisant la propriété **sqlWriterStoredProcedureName**. Pour plus de détails, consultez l’article [Appeler une procédure stockée à partir d’une activité de copie](data-factory-invoke-stored-procedure-from-copy-activity.md). Pour plus d’informations sur la propriété, consultez les articles suivants sur les connecteurs : [Azure SQL Database](data-factory-azure-sql-connector.md#copy-activity-properties), [SQL Server](data-factory-sqlserver-connector.md#copy-activity-properties). L’appel d’une procédure stockée lors de la copie de données dans Azure Synapse Analytics à l’aide d’une activité de copie n’est pas pris en charge. Toutefois, vous pouvez utiliser l’activité de procédure stockée pour appeler une procédure stockée dans Azure Synapse Analytics.
>
> Lors de la copie de données à partir d’Azure SQL Database, de SQL Server ou d’Azure Synapse Analytics, vous pouvez configurer **SqlSource** dans l’activité de copie pour appeler une procédure stockée afin de lire les données à partir de la base de données source en utilisant la propriété **sqlReaderStoredProcedureName**. Pour plus d’informations, consultez les articles suivants sur les connecteurs : [Azure SQL Database](data-factory-azure-sql-connector.md#copy-activity-properties), [SQL Server](data-factory-sqlserver-connector.md#copy-activity-properties), [Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#copy-activity-properties)

La procédure pas à pas suivante utilise l’activité de procédure stockée dans un pipeline pour appeler une procédure stockée dans Azure SQL Database.

## <a name="walkthrough"></a>Procédure pas à pas
### <a name="sample-table-and-stored-procedure"></a>Exemple de table et de procédure stockée
1. Créez la **table** suivante dans votre base de données Azure SQL à l’aide de SQL Server Management Studio ou d’un autre outil que vous maîtrisez. La colonne datetimestamp affiche la date et l’heure auxquelles l’ID correspondant est généré.

    ```SQL
    CREATE TABLE dbo.sampletable
    (
        Id uniqueidentifier,
        datetimestamp nvarchar(127)
    )
    GO

    CREATE CLUSTERED INDEX ClusteredID ON dbo.sampletable(Id);
    GO
    ```
    La colonne ID affiche l’identifiant unique et la colonne datetimestamp affiche la date et l’heure auxquelles l’ID correspondant est généré.
    
    ![Exemples de données](./media/data-factory-stored-proc-activity/sample-data.png)

    Dans cet exemple, la procédure stockée est dans Azure SQL Database. Si la procédure stockée est dans Azure Synapse Analytics et SQL Server Database, l’approche est similaire. Pour une base de données SQL Server, vous devez installer une [passerelle de gestion des données](data-factory-data-management-gateway.md).
2. Créez la **procédure stockée** suivante qui insère des données dans la table **sampletable**.

    ```SQL
    CREATE PROCEDURE usp_sample @DateTime nvarchar(127)
    AS

    BEGIN
        INSERT INTO [sampletable]
        VALUES (newid(), @DateTime)
    END
    ```

   > [!IMPORTANT]
   > Le **nom** et la **casse** du paramètre (DateTime dans cet exemple) doivent correspondre à ceux du paramètre spécifié dans le script JSON de l’activité/du pipeline. Dans la définition de procédure stockée, vérifiez que **\@** est utilisé en tant que préfixe pour le paramètre.

### <a name="create-a-data-factory"></a>Créer une fabrique de données
1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Cliquez sur **NOUVEAU** dans le menu de gauche, puis sur **Intelligence + analyse** et sur **Data Factory**.

    ![Nouvelle fabrique de données 1](media/data-factory-stored-proc-activity/new-data-factory.png)
3. Dans le panneau **Nouvelle fabrique de données**, entrez **SProcDF** dans le champ Nom. Les noms Azure Data Factory sont **globalement uniques**. Vous devez faire précéder le nom de la fabrique de données par votre nom, pour activer la création de la fabrique.

   ![Nouvelle fabrique de données 2](media/data-factory-stored-proc-activity/new-data-factory-blade.png)
4. Sélectionnez votre **abonnement Azure**.
5. Pour **Groupe de ressources**, effectuez l’une des opérations suivantes :
   1. Cliquez sur **Créer un nouveau** et entrez un nom pour le groupe de ressources.
   2. Cliquez sur **Utiliser l’existant** et sélectionnez un groupe de ressources existant.
6. Sélectionnez **l’emplacement** de la fabrique de données.
7. Sélectionnez **Épingler au tableau de bord** pour afficher la fabrique de données dans le tableau de bord la prochaine fois que vous vous connectez.
8. Cliquez sur **Créer** dans le panneau **Nouvelle fabrique de données**.
9. La fabrique de données apparaît comme étant en cours de création dans le **tableau de bord** du portail Azure. Une fois la fabrique de données créée, la page correspondante s’affiche et indique son contenu.

   ![Page d’accueil Data Factory](media/data-factory-stored-proc-activity/data-factory-home-page.png)

### <a name="create-an-azure-sql-linked-service"></a>Créer un service lié Azure SQL
Après avoir créé la fabrique de données, vous créez un service lié Azure SQL reliant votre base de données dans Azure SQL Database, qui contient la table sampletable et la procédure stockée usp_sample, à votre fabrique de données.

1. Dans le panneau **Fabrique de données**, cliquez sur **Créer et déployer** pour que **SProcDF** lance l’éditeur de la fabrique de données.
2. Cliquez sur **Nouvelle banque de données** dans la barre de commandes et choisissez **Azure SQL Database**. Le script JSON de création d’un service lié Azure SQL doit apparaître dans l’éditeur.

   ![Nouveau magasin de données 1](media/data-factory-stored-proc-activity/new-data-store.png)
3. Dans le script JSON, apportez les modifications suivantes :

   1. Remplacez `<servername>` par le nom de votre serveur.
   2. Remplacez `<databasename>` par la base de données dans laquelle vous avez créé la table et la procédure stockée.
   3. Remplacez `<username@servername>` par le compte d’utilisateur qui a accès à la base de données.
   4. Remplacez `<password>` par le mot de passe du compte d’utilisateur.

      ![Nouveau magasin de données 2](media/data-factory-stored-proc-activity/azure-sql-linked-service.png)
4. Pour déployer le service lié, cliquez sur **Déployer** dans la barre de commandes. Vérifiez que AzureSqlLinkedService apparaît dans l’arborescence à gauche.

    ![arborescence avec service lié 1](media/data-factory-stored-proc-activity/tree-view.png)

### <a name="create-an-output-dataset"></a>Créer un jeu de données de sortie
Vous devez spécifier un jeu de données de sortie pour une activité de procédure stockée même si la procédure stockée ne génère aucune donnée. C’est parce qu’il s’agit du jeu de données de sortie qui pilote le calendrier de l’activité (fréquence d’exécution de l’activité : horaire, quotidienne, etc.). Le jeu de données de sortie doit utiliser un **service lié** qui fait référence à une base de données Azure SQL Database, Azure Synapse Analytics ou SQL Server dans laquelle vous souhaitez que la procédure stockée soit exécutée. Le jeu de données de sortie peut être un moyen de passer le résultat de la procédure stockée pour traitement ultérieur par une autre activité ([chaînage des activités](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)) dans le pipeline. Toutefois, Data Factory n’écrit pas automatiquement la sortie d’une procédure stockée pour ce jeu de données. C’est la procédure stockée qui écrit dans une table SQL vers laquelle le jeu de données de sortie pointe. Dans certains cas, le jeu de données de sortie peut être un **jeu de données factice** (un jeu de données qui pointe vers une table qui ne possède pas vraiment de sortie de la procédure stockée). Ce jeu de données factice est utilisé uniquement pour spécifier le calendrier d’exécution de l’activité de procédure stockée.

1. Si ce bouton n'est pas affiché dans la barre d'outils, cliquez sur **... Plus** dans la barre d’outils, sur **Nouveau jeu de données**, puis sur **SQL Azure**. Cliquez sur **Nouveau jeu de données** dans la barre de commandes et sélectionnez **SQL Azure**.

    ![arborescence avec service lié 2](media/data-factory-stored-proc-activity/new-dataset.png)
2. Copiez-collez le script JSON suivant dans l’éditeur JSON.

    ```JSON
    {
        "name": "sprocsampleout",
        "properties": {
            "type": "AzureSqlTable",
            "linkedServiceName": "AzureSqlLinkedService",
            "typeProperties": {
                "tableName": "sampletable"
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
    }
    ```
3. Pour déployer le jeu de données, cliquez sur **Déployer** dans la barre de commandes. Vérifiez que le jeu de données apparaît dans l’arborescence.

    ![arborescence avec services liés](media/data-factory-stored-proc-activity/tree-view-2.png)

### <a name="create-a-pipeline-with-sqlserverstoredprocedure-activity"></a>Créer un pipeline avec une activité SqlServerStoredProcedure
Nous allons maintenant créer un pipeline avec une activité de procédure stockée.

Notez les propriétés suivantes :

- La propriété **type** doit être définie sur **SqlServerStoredProcedure**.
- Le paramètre **storedProcedureName** des propriétés type est défini sur **usp_sample** (nom de la procédure stockée).
- La section **storedProcedureParameters** contient un paramètre nommé **DateTime**. Le nom et la casse du paramètre dans JSON doivent correspondre à ceux du paramètre dans la définition de procédure stockée. Si vous avez besoin d’utiliser la valeur null pour un paramètre, utilisez la syntaxe : `"param1": null` (tout en minuscules).

1. Si ce bouton n'est pas affiché dans la barre d'outils, cliquez sur **... Plus** dans la barre de commandes et sur **Nouveau pipeline**.
2. Copiez-collez l’extrait de code JSON suivant :

    ```JSON
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
            "start": "2017-04-02T00:00:00Z",
            "end": "2017-04-02T05:00:00Z",
            "isPaused": false
        }
    }
    ```
3. Pour déployer le pipeline, cliquez sur **Déployer** dans la barre d’outils.

### <a name="monitor-the-pipeline"></a>Surveiller le pipeline
1. Cliquez sur **X** pour fermer les panneaux de Data Factory Editor et revenir au panneau Data Factory, puis cliquez sur **Diagramme**.

    ![vignette de diagramme 1](media/data-factory-stored-proc-activity/data-factory-diagram-tile.png)
2. Dans la **Vue de diagramme**, une vue d’ensemble des pipelines et des jeux de données utilisés dans ce didacticiel s’affiche.

    ![vignette de diagramme 2](media/data-factory-stored-proc-activity/data-factory-diagram-view.png)
3. Dans la vue de diagramme, double-cliquez sur le jeu de données `sprocsampleout`. Les tranches s’affichent avec l’état Prêt. Il doit y avoir cinq tranches, car une tranche est produite pour chaque heure entre l’heure de début et l’heure de fin dans le JSON.

    ![vignette de diagramme 3](media/data-factory-stored-proc-activity/data-factory-slices.png)
4. Quand une tranche est à l’état **Prêt**, exécutez une requête `select * from sampletable` sur la base de données pour vérifier que les données ont été insérées dans la table par la procédure stockée.

   ![Données de sortie](./media/data-factory-stored-proc-activity/output.png)

   Consultez [Surveiller le pipeline](data-factory-monitor-manage-pipelines.md) pour plus d’informations sur la surveillance des pipelines Azure Data Factory.

## <a name="specify-an-input-dataset"></a>Spécifier un jeu de données d’entrée
Dans la procédure pas à pas, l’activité de procédure stockée n’a pas de jeux de données d’entrée. Si vous spécifiez un jeu de données d’entrée, l’activité de procédure stockée ne s’exécute pas tant que la tranche du jeu de données d’entrée n’est pas disponible (à l’état Prêt). Le jeu de données peut être un jeu de données externe (non généré par une autre activité dans le même pipeline) ou un jeu de données interne généré par une activité en amont (l’activité qui s’exécute avant cette activité). Vous pouvez spécifier plusieurs jeux de données d’entrée pour l’activité de procédure stockée. Si vous procédez ainsi, l’activité de procédure stockée ne s’exécute que lorsque toutes les tranches du jeu de données d’entrée sont disponibles (à l’état Prêt). Les jeux de données d’entrée ne peuvent pas être utilisés dans la procédure stockée en tant que paramètres. Cela sert uniquement à vérifier la dépendance avant de commencer l’activité de procédure stockée.

## <a name="chaining-with-other-activities"></a>Chaînage avec d’autres activités
Si vous souhaitez chaîner une activité en amont avec cette activité, spécifiez la sortie de l’activité en amont en tant qu’entrée de cette activité. Lorsque vous procédez ainsi, l’activité de procédure stockée ne s’exécute pas tant que l’activité en amont n’est pas terminée et que le jeu de données de sortie de l’activité en amont n’est pas disponible (à l’état Prêt). Vous pouvez spécifier des jeux de données de sortie de plusieurs activités en amont comme jeux de données d’entrée de l’activité de procédure stockée. Lorsque vous procédez ainsi, l’activité de procédure stockée ne s’exécute que lorsque toutes les tranches du jeu de données d’entrée sont disponibles.

Dans l’exemple suivant, la sortie de l’activité de copie est : OutputDataset, qui est une entrée de l’activité de procédure stockée. Par conséquent, l’activité de procédure stockée ne s’exécute pas tant que l’activité de copie n’est pas disponible et que la tranche OutputDataset n’est pas disponible (à l’état Prêt). Si vous spécifiez plusieurs jeux de données d’entrée, l’activité de procédure stockée ne s’exécute pas tant que toutes les tranches du jeu de données d’entrée ne sont pas disponibles (à l’état Prêt). Les jeux de données d’entrée ne peuvent pas servir directement de paramètres pour l’activité de procédure stockée.

Pour plus d’informations sur le chaînage des activités, consultez [Plusieurs activités dans un pipeline](data-factory-create-pipelines.md#multiple-activities-in-a-pipeline)

```json
{
    "name": "ADFTutorialPipeline",
    "properties": {
        "description": "Copy data from a blob to blob",
        "activities": [
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
                "inputs": [ { "name": "InputDataset" } ],
                "outputs": [ { "name": "OutputDataset" } ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst"
                },
                "name": "CopyFromBlobToSQL"
            },
            {
                "type": "SqlServerStoredProcedure",
                "typeProperties": {
                    "storedProcedureName": "SPSproc"
                },
                "inputs": [ { "name": "OutputDataset" } ],
                "outputs": [ { "name": "SQLOutputDataset" } ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "retry": 3
                },
                "name": "RunStoredProcedure"
            }
        ],
        "start": "2017-04-12T00:00:00Z",
        "end": "2017-04-13T00:00:00Z",
        "isPaused": false,
    }
}
```

De même, pour lier l’activité de procédure stockée avec des **activités en aval** (activités qui s’exécutent une fois l’activité de procédure stockée terminée), spécifiez le jeu de données de sortie de l’activité de procédure stockée en tant qu’entrée de l’activité en aval dans le pipeline.

> [!IMPORTANT]
> Lorsque vous copiez des données dans Azure SQL Database ou SQL Server, vous pouvez configurer l’élément **SqlSink** dans l’activité de copie pour appeler une procédure stockée en utilisant la propriété **sqlWriterStoredProcedureName**. Pour plus de détails, consultez l’article [Appeler une procédure stockée à partir d’une activité de copie](data-factory-invoke-stored-procedure-from-copy-activity.md). Pour plus d’informations sur la propriété, consultez les articles suivants sur les connecteurs : [Azure SQL Database](data-factory-azure-sql-connector.md#copy-activity-properties), [SQL Server](data-factory-sqlserver-connector.md#copy-activity-properties).
> 
> Lors de la copie de données à partir d’Azure SQL Database, de SQL Server ou d’Azure Synapse Analytics, vous pouvez configurer **SqlSource** dans l’activité de copie pour appeler une procédure stockée afin de lire les données à partir de la base de données source en utilisant la propriété **sqlReaderStoredProcedureName**. Pour plus d’informations, consultez les articles suivants sur les connecteurs : [Azure SQL Database](data-factory-azure-sql-connector.md#copy-activity-properties), [SQL Server](data-factory-sqlserver-connector.md#copy-activity-properties), [Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#copy-activity-properties)

## <a name="json-format"></a>Format JSON
Voici le format JSON pour la définition d’une activité de procédure stockée :

```JSON
{
    "name": "SQLSPROCActivity",
    "description": "description",
    "type": "SqlServerStoredProcedure",
    "inputs": [ { "name": "inputtable" } ],
    "outputs": [ { "name": "outputtable" } ],
    "typeProperties":
    {
        "storedProcedureName": "<name of the stored procedure>",
        "storedProcedureParameters":
        {
            "param1": "param1Value"
            …
        }
    }
}
```

Le tableau suivant décrit ces paramètres JSON :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| name | Nom de l’activité |Oui |
| description |Texte décrivant la raison motivant l’activité. |Non |
| type | Doit être défini sur : **SqlServerStoredProcedure** | Oui |
| inputs | facultatif. Si vous spécifiez un jeu de données d’entrée, il doit être disponible (à l’état Prêt) pour l’activité de procédure stockée à exécuter. Les jeux de données d’entrée ne peuvent pas être utilisés dans la procédure stockée en tant que paramètres. Cela sert uniquement à vérifier la dépendance avant de commencer l’activité de procédure stockée. |Non |
| outputs | Vous devez spécifier un jeu de données de sortie pour une activité de procédure stockée. Le jeu de données de sortie spécifie la **planification** pour l’activité de procédure stockée (horaire, hebdomadaire, mensuelle, etc.). <br/><br/>Le jeu de données de sortie doit utiliser un **service lié** qui fait référence à une base de données Azure SQL Database, Azure Synapse Analytics ou SQL Server dans laquelle vous souhaitez que la procédure stockée soit exécutée. <br/><br/>Le jeu de données de sortie peut être un moyen de passer le résultat de la procédure stockée pour traitement ultérieur par une autre activité ([chaînage des activités](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)) dans le pipeline. Toutefois, Data Factory n’écrit pas automatiquement la sortie d’une procédure stockée pour ce jeu de données. C’est la procédure stockée qui écrit dans une table SQL vers laquelle le jeu de données de sortie pointe. <br/><br/>Dans certains cas, le jeu de données de sortie peut être un **jeu de données factice**, qui est utilisé uniquement pour spécifier le calendrier d’exécution de l’activité de procédure stockée. |Oui |
| storedProcedureName |Spécifiez le nom de la procédure stockée dans Azure SQL Database, Azure Synapse Analytics ou SQL Server, qui est représentée par le service lié utilisé par la table de sortie. |Oui |
| storedProcedureParameters |Spécifiez les valeurs des paramètres de procédure stockée. Si vous avez besoin de passer null pour un paramètre, utilisez la syntaxe : "param1": null (le tout en minuscules). Consultez l’exemple suivant pour en savoir plus sur l’utilisation de cette propriété. |Non |

## <a name="passing-a-static-value"></a>Transmission d’une valeur statique
À présent, ajoutons une autre colonne nommée « Scénario » dans la table contenant une valeur statique appelée « Exemple de document ».

![Exemple de données 2](./media/data-factory-stored-proc-activity/sample-data-2.png)

**Table :**

```SQL
CREATE TABLE dbo.sampletable2
(
    Id uniqueidentifier,
    datetimestamp nvarchar(127),
    scenario nvarchar(127)
)
GO

CREATE CLUSTERED INDEX ClusteredID ON dbo.sampletable2(Id);
```

**Procédure stockée**

```SQL
CREATE PROCEDURE usp_sample2 @DateTime nvarchar(127) , @Scenario nvarchar(127)

AS

BEGIN
    INSERT INTO [sampletable2]
    VALUES (newid(), @DateTime, @Scenario)
END
```

Maintenant, transmettez le paramètre **Scénario** et la valeur de l’activité de procédure stockée. La section **typeProperties** de l’exemple précédent ressemble à l’extrait de code suivant :

```JSON
"typeProperties":
{
    "storedProcedureName": "usp_sample",
    "storedProcedureParameters":
    {
        "DateTime": "$$Text.Format('{0:yyyy-MM-dd HH:mm:ss}', SliceStart)",
        "Scenario": "Document sample"
    }
}
```

**Jeu de données Data Factory :**

```JSON
{
    "name": "sprocsampleout2",
    "properties": {
        "published": false,
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties": {
            "tableName": "sampletable2"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Pipeline Data Factory**

```JSON
{
    "name": "SprocActivitySamplePipeline2",
    "properties": {
        "activities": [
            {
                "type": "SqlServerStoredProcedure",
                "typeProperties": {
                    "storedProcedureName": "usp_sample2",
                    "storedProcedureParameters": {
                        "DateTime": "$$Text.Format('{0:yyyy-MM-dd HH:mm:ss}', SliceStart)",
                        "Scenario": "Document sample"
                    }
                },
                "outputs": [
                    {
                        "name": "sprocsampleout2"
                    }
                ],
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "SprocActivitySample"
            }
        ],
        "start": "2016-10-02T00:00:00Z",
        "end": "2016-10-02T05:00:00Z"
    }
}
```
