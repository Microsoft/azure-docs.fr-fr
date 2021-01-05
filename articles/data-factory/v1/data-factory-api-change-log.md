---
title: Data Factory – Journal des modifications de l’API .NET
description: Décrit les changements cassants, les ajouts de fonctionnalités, les correctifs de bogues, etc. d’une version spécifique de l’API .NET pour Azure Data Factory.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
robots: noindex
ms.date: 01/22/2018
ms.openlocfilehash: 70df35409d1c84efb996bb40f4e39bde6ad7d5a8
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96496498"
---
# <a name="azure-data-factory---net-api-change-log"></a>Azure Data Factory : Journal des modifications de l’API .NET
> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. 

Cet article fournit des informations sur les modifications apportées au SDK Azure Data Factory dans une version spécifique. Vous trouverez le dernier package NuGet pour Azure Data Factory [ici](https://www.nuget.org/packages/Microsoft.Azure.Management.DataFactories)

## <a name="version-4110"></a>Version 4.11.0
Ajouts de fonctionnalités :

* Les types de services liés suivants ont été ajoutés :
  * [OnPremisesMongoDbLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.onpremisesmongodblinkedservice)
  * [AmazonRedshiftLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.amazonredshiftlinkedservice)
  * [AwsAccessKeyLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.awsaccesskeylinkedservice)
* Les types de jeux de données suivants ont été ajoutés :
  * [MongoDbCollectionDataset](/dotnet/api/microsoft.azure.management.datafactories.models.mongodbcollectiondataset)
  * [AmazonS3Dataset](/dotnet/api/microsoft.azure.management.datafactories.models.amazons3dataset)
* Les types de sources de copie suivants ont été ajoutés :
  * [MongoDbSource](/dotnet/api/microsoft.azure.management.datafactories.models.mongodbsource)

## <a name="version-4100"></a>Version 4.10.0
* Les propriétés facultatives suivantes ont été ajoutées à TextFormat :
  * [SkipLineCount](/dotnet/api/microsoft.azure.management.datafactories.models.textformat)
  * [FirstRowAsHeader](/dotnet/api/microsoft.azure.management.datafactories.models.textformat)
  * [TreatEmptyAsNull](/dotnet/api/microsoft.azure.management.datafactories.models.textformat)
* Les types de services liés suivants ont été ajoutés :
  * [OnPremisesCassandraLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.onpremisescassandralinkedservice)
  * [SalesforceLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.salesforcelinkedservice)
* Les types de jeux de données suivants ont été ajoutés :
  * [OnPremisesCassandraTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.onpremisescassandratabledataset)
* Les types de sources de copie suivants ont été ajoutés :
  * [CassandraSource](/dotnet/api/microsoft.azure.management.datafactories.models.cassandrasource)
* Ajout de la propriété [WebServiceInputs](/dotnet/api/microsoft.azure.management.datafactories.models.azuremlbatchexecutionactivity) à AzureMLBatchExecutionActivity
  * Autorisation du transfert de plusieurs entrées de service web dans une expérience Azure Machine Learning

## <a name="version-491"></a>Version 4.9.1
### <a name="bug-fix"></a>Correctif de bogue
* L’authentification basée sur des API Web pour [WebLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.weblinkedservice)est déconseillée.

## <a name="version-490"></a>Version 4.9.0
### <a name="feature-additions"></a>Ajouts de fonctionnalités
* Ajout des propriétés [EnableStaging](/dotnet/api/microsoft.azure.management.datafactories.models.copyactivity) et [StagingSettings](/dotnet/api/microsoft.azure.management.datafactories.models.stagingsettings) à CopyActivity. Consultez la page [Copie intermédiaire](data-factory-copy-activity-performance.md#staged-copy) pour plus d’informations sur la fonctionnalité.

### <a name="bug-fix"></a>Correctif de bogue
* Introduction d’une surcharge de méthode [ActivityWindowOperationExtensions.List](/dotnet/api/microsoft.azure.management.datafactories.activitywindowoperationsextensions), qui nécessite une instance de [ActivityWindowsByActivityListParameters](/dotnet/api/microsoft.azure.management.datafactories.models.activitywindowsbyactivitylistparameters).
* Marquage de [WriteBatchSize](/dotnet/api/microsoft.azure.management.datafactories.models.copysink) et [WriteBatchTimeout](/dotnet/api/microsoft.azure.management.datafactories.models.copysink) comme facultatifs dans CopySink.

## <a name="version-480"></a>Version 4.8.0
### <a name="feature-additions"></a>Ajouts de fonctionnalités
* Les propriétés facultatives suivantes ont été ajoutées au type d’activité Copie pour une optimisation des performances de copie :
  * [ParallelCopies](/dotnet/api/microsoft.azure.management.datafactories.models.copyactivity)
  * [CloudDataMovementUnits](/dotnet/api/microsoft.azure.management.datafactories.models.copyactivity)

## <a name="version-470"></a>Version 4.7.0
### <a name="feature-additions"></a>Ajouts de fonctionnalités
* Ajout du nouveau type StorageFormat [OrcFormat](/dotnet/api/microsoft.azure.management.datafactories.models.orcformat) pour copier des fichiers au format ORC (Optimized Row Columnar).
* Ajout des propriétés [AllowPolyBase](/dotnet/api/microsoft.azure.management.datafactories.models.sqldwsink) et PolyBaseSettings à SqlDWSink.
  * Permet l’utilisation de PolyBase pour copier des données dans Azure Synapse Analytics.

## <a name="version-461"></a>Version 4.6.1
### <a name="bug-fixes"></a>Correctifs de bogues
* Résout les requêtes HTTP pour les listes de fenêtres d’activité.
  * Supprime le nom du groupe de ressources et le nom de la fabrique de données de la charge utile des requêtes.

## <a name="version-460"></a>Version 4.6.0
### <a name="feature-additions"></a>Ajouts de fonctionnalités
* Les propriétés suivantes ont été ajoutées à [PipelineProperties](/dotnet/api/microsoft.azure.management.datafactories.models.pipelineproperties):
  * [PipelineMode](/dotnet/api/microsoft.azure.management.datafactories.models.pipelineproperties)
  * [ExpirationTime](/dotnet/api/microsoft.azure.management.datafactories.models.pipelineproperties)
  * [Groupes de données](/dotnet/api/microsoft.azure.management.datafactories.models.pipelineproperties)
* Les propriétés suivantes ont été ajoutées à [PipelineRuntimeInfo](/dotnet/api/microsoft.azure.management.datafactories.common.models.pipelineruntimeinfo):
  * [PipelineState](/dotnet/api/microsoft.azure.management.datafactories.common.models.pipelineruntimeinfo)
* Ajout des types [StorageFormat](/dotnet/api/microsoft.azure.management.datafactories.models.storageformat) et [JsonFormat](/dotnet/api/microsoft.azure.management.datafactories.models.jsonformat) pour définir les jeux de données dont les données sont au format JSON.

## <a name="version-450"></a>Version 4.5.0
### <a name="feature-additions"></a>Ajouts de fonctionnalités
* Ajout d’une [liste des opérations pour la fenêtre d’activité](/dotnet/api/microsoft.azure.management.datafactories.activitywindowoperationsextensions).
  * Ajout de méthodes pour récupérer les fenêtres d’activité avec des filtres en fonction des type d’entités (fabriques de données, jeux de données, pipelines et activités).
* Les types de services liés suivants ont été ajoutés :
  * [ODataLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.odatalinkedservice), [WebLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.weblinkedservice)
* Les types de jeux de données suivants ont été ajoutés :
  * [ODataResourceDataset](/dotnet/api/microsoft.azure.management.datafactories.models.odataresourcedataset), [WebTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.webtabledataset)
* Les types de sources de copie suivants ont été ajoutés :     
  * [WebSource](/dotnet/api/microsoft.azure.management.datafactories.models.websource)

## <a name="version-440"></a>Version 4.4.0
### <a name="feature-additions"></a>Ajouts de fonctionnalités
* Le type de service lié suivant a été ajouté en tant que source de données et récepteur pour les activités de copie :
  * [AzureStorageSasLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.azurestoragesaslinkedservice). Consultez la section [Service lié SAP Azure Storage](data-factory-azure-blob-connector.md#azure-storage-sas-linked-service) pour obtenir des informations conceptuelles et des exemples.

## <a name="version-430"></a>Version 4.3.0
### <a name="feature-additions"></a>Ajouts de fonctionnalités
* Les types de service lié suivants ont été ajoutés en tant que sources de données pour les activités de copie :
  * [HdfsLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.hdfslinkedservice). Consultez la section [Déplacer des données de HDFS à l’aide de Data Factory](data-factory-hdfs-connector.md) pour obtenir des informations conceptuelles et des exemples.
  * [OnPremisesOdbcLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.onpremisesodbclinkedservice). Consultez la section [Transfert de données à partir de magasins de données ODBC à l’aide d’Azure Data Factory](data-factory-odbc-connector.md) pour obtenir des informations conceptuelles et des exemples.

## <a name="version-420"></a>Version 4.2.0
### <a name="feature-additions"></a>Ajouts de fonctionnalités
* Le type d’activité suivant a été ajouté : [AzureMLUpdateResourceActivity](/dotnet/api/microsoft.azure.management.datafactories.models.azuremlupdateresourceactivity). Pour plus d’informations sur l’activité, consultez [Mettre à jour les modèles Azure ML à l’aide de l’activité de mise à jour des ressources](data-factory-azure-ml-batch-execution-activity.md).
* Une nouvelle propriété facultative [updateResourceEndpoint](/dotnet/api/microsoft.azure.management.datafactories.models.azuremllinkedservice) a été ajoutée à la [classe AzureMLLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.azuremllinkedservice).
* Les propriétés [LongRunningOperationInitialTimeout](/dotnet/api/microsoft.azure.management.datafactories.datafactorymanagementclient) et [LongRunningOperationRetryTimeout](/dotnet/api/microsoft.azure.management.datafactories.datafactorymanagementclient) ont été ajoutées à la classe [DataFactoryManagementClient](/dotnet/api/microsoft.azure.management.datafactories.datafactorymanagementclient).
* Elles permettent de configurer les délais d’attente pour les appels du client vers le service Data Factory.

## <a name="version-410"></a>Version 4.1.0
### <a name="feature-additions"></a>Ajouts de fonctionnalités
* Les types de services liés suivants ont été ajoutés :
  * [AzureDataLakeStoreLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice)
  * [AzureDataLakeAnalyticsLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice)
* Les types d’activités suivants ont été ajoutés :
  * [DataLakeAnalyticsUSQLActivity](/dotnet/api/microsoft.azure.management.datafactories.models.datalakeanalyticsusqlactivity)
* Les types de jeux de données suivants ont été ajoutés :
  * [AzureDataLakeStoreDataset](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestoredataset)
* Les types de source et de récepteur suivants pour l’activité de copie ont été ajoutés :
  * [AzureDataLakeStoreSource](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestoresource)
  * [AzureDataLakeStoreSink](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestoresink)

## <a name="version-401"></a>Version 4.0.1
### <a name="breaking-changes"></a>Changements cassants
Les classes suivantes ont été renommées. Les nouveaux noms des classes sont les noms antérieurs à la version 4.0.0.

| Nom dans 4.0.0 | Nom dans 4.0.1 |
|:--- |:--- |
| AzureSqlDataWarehouseDataset |[AzureSqlDataWarehouseTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.azuresqldatawarehousetabledataset) |
| AzureSqlDataset |[AzureSqlTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.azuresqltabledataset) |
| AzureDataset |[AzureTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.azuretabledataset) |
| OracleDataset |[OracleTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.oracletabledataset) |
| RelationalDataset |[RelationalTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.relationaltabledataset) |
| SqlServerDataset |[SqlServerTableDataset](/dotnet/api/microsoft.azure.management.datafactories.models.sqlservertabledataset) |

## <a name="version-400"></a>Version 4.0.0
### <a name="breaking-changes"></a>Changements cassants
* Les classes/interfaces suivantes ont été renommées.

| Ancien nom | Nouveau nom |
|:--- |:--- |
| ITableOperations |[IDatasetOperations](/dotnet/api/microsoft.azure.management.datafactories.idatasetoperations) |
| Table de charge de travail |[Dataset](/dotnet/api/microsoft.azure.management.datafactories.models.dataset) |
| TableProperties |[DatasetProperties](/dotnet/api/microsoft.azure.management.datafactories.models.datasetproperties) |
| TableTypeProprerties |[DatasetTypeProperties](/dotnet/api/microsoft.azure.management.datafactories.models.datasettypeproperties) |
| TableCreateOrUpdateParameters |[DatasetCreateOrUpdateParameters](/dotnet/api/microsoft.azure.management.datafactories.models.datasetcreateorupdateparameters) |
| TableCreateOrUpdateResponse |[DatasetCreateOrUpdateResponse](/dotnet/api/microsoft.azure.management.datafactories.models.datasetcreateorupdateresponse) |
| TableGetResponse |[DatasetGetResponse](/dotnet/api/microsoft.azure.management.datafactories.models.datasetgetresponse) |
| TableListResponse |[DatasetListResponse](/dotnet/api/microsoft.azure.management.datafactories.models.datasetlistresponse) |
| CreateOrUpdateWithRawJsonContentParameters |[DatasetCreateOrUpdateWithRawJsonContentParameters](/dotnet/api/microsoft.azure.management.datafactories.models.datasetcreateorupdatewithrawjsoncontentparameters) |

* Les méthodes **Liste** renvoient désormais des résultats paginés. Si la réponse contient une propriété **NextLink** non vide, l’application cliente doit continuer à récupérer la page suivante jusqu’à ce que toutes les pages soient renvoyées.  Voici un exemple :

    ```csharp
    PipelineListResponse response = client.Pipelines.List("ResourceGroupName", "DataFactoryName");
    var pipelines = new List<Pipeline>(response.Pipelines);

    string nextLink = response.NextLink;
    while (!string.IsNullOrEmpty(nextLink))
    {
        PipelineListResponse nextResponse = client.Pipelines.ListNext(nextLink);
        pipelines.AddRange(nextResponse.Pipelines);

        nextLink = nextResponse.NextLink;
    }
    ```
* **Listee** renvoie uniquement le résumé d’un pipeline au lieu des détails complets. Par exemple, les activités d’un résumé de pipeline ne contiennent que le nom et le type.

### <a name="feature-additions"></a>Ajouts de fonctionnalités
* La classe [SqlDWSink](/dotnet/api/microsoft.azure.management.datafactories.models.sqldwsink) prend en charge deux nouvelles propriétés, **SliceIdentifierColumnName** et **SqlWriterCleanupScript**, pour prendre en charge la copie idempotente vers Azure Synapse Analytics. Pour plus d’informations sur ces propriétés, consultez l’article relatif à [Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md).
* Nous prenons désormais en charge l’exécution d’une procédure stockée sur des sources Azure SQL Database et Azure Synapse Analytics dans le cadre de l’activité de copie. Les classes [SqlSource](/dotnet/api/microsoft.azure.management.datafactories.models.sqlsource) et [SqlDWSource](/dotnet/api/microsoft.azure.management.datafactories.models.sqldwsource) comportent les propriétés suivantes : **SqlReaderStoredProcedureName** et **StoredProcedureParameters**. Pour plus d’informations sur ces propriétés, consultez les articles relatifs à [Azure SQL Database](data-factory-azure-sql-connector.md#sqlsource) et [Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#sqldwsource) sur Azure.com.