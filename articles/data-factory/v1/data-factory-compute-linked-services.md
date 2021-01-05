---
title: Environnements Compute pris en charge par Azure Data Factory version 1
description: Découvrez les environnements de calcul que vous pouvez utiliser dans les pipelines Azure Data Factory (tels qu’Azure HDInsight) pour transformer ou traiter les données.
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
ms.openlocfilehash: 9f970b66bd30ef2faf705d2fb41825bb81bbb8c9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96496090"
---
# <a name="compute-environments-supported-by-azure-data-factory-version-1"></a>Environnements Compute pris en charge par Azure Data Factory version 1
> [!NOTE]
> Cet article s’applique à la version 1 d’Azure Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Services liés de calcul](../compute-linked-services.md).

Cet article décrit les environnements de calcul que vous pouvez utiliser pour traiter ou transformer des données. Il fournit également des détails sur les différentes configurations (à la demande ou de type « apporter votre propre configuration ») prises en charge par Data Factory lorsque vous configurez des services liés qui relient ces environnements de calcul à une fabrique de données Azure.

Le tableau suivant fournit une liste d’environnements de calcul qui sont pris en charge par Data Factory et les activités qui peuvent s’exécuter sur ces derniers. 

| Environnement de calcul                      | Activités                               |
| ---------------------------------------- | ---------------------------------------- |
| [Cluster Azure HDInsight à la demande](#azure-hdinsight-on-demand-linked-service) ou [votre propre cluster HDInsight](#azure-hdinsight-linked-service) | [DotNet](data-factory-use-custom-activities.md), [Hive](data-factory-hive-activity.md), [Pig](data-factory-pig-activity.md), [MapReduce](data-factory-map-reduce.md), [Diffusion en continu Hadoop](data-factory-hadoop-streaming-activity.md) |
| [Azure Batch](#azure-batch-linked-service) | [DotNet](data-factory-use-custom-activities.md) |
| [Azure Machine Learning Studio (classique)](#azure-machine-learning-studio-classic-linked-service) | [Activités Studio (classique) : exécution par lot et ressource de mise à jour](data-factory-azure-ml-batch-execution-activity.md) |
| [Service Analytique Azure Data Lake](#azure-data-lake-analytics-linked-service) | [Langage U-SQL du service Analytique Data Lake](data-factory-usql-activity.md) |
| [Azure SQL](#azure-sql-linked-service), [Azure Synapse Analytics](#azure-synapse-analytics-linked-service), [SQL Server](#sql-server-linked-service) | [Activité de procédure stockée](data-factory-stored-proc-activity.md) |

## <a name="hdinsight-versions-supported-in-data-factory"></a><a name="supported-hdinsight-versions-in-azure-data-factory"></a>Versions de HDInsight prises en charge dans Data Factory
Azure HDInsight prend en charge plusieurs versions de cluster Hadoop que vous pouvez déployer à tout moment. Chaque version prise en charge crée une version spécifique de la distribution de la plateforme de données Hortonworks (HDP) et un ensemble de composants dans la distribution. 

Microsoft met à jour la liste des versions de HDInsight prises en charge avec les correctifs et composants les plus récents de l’écosystème Hadoop. Pour plus d’informations, consultez [Versions de HDInsight prises en charge](../../hdinsight/hdinsight-component-versioning.md#supported-hdinsight-versions).

> [!IMPORTANT]
> La version 3.3 de HDInsight basé sur Linux a été mise hors service le 31 juillet 2017. Les clients des services liés HDInsight à la demande de Data Factory version 1 ont jusqu’au 15 décembre 2017 pour procéder aux tests et à la mise à niveau vers une version ultérieure de HDInsight. HDInsight basé sur Windows sera mis hors service le 31 juillet 2018.
>
> 

### <a name="after-the-retirement-date"></a>Après la date de mise hors service 

Après le 15 décembre 2017 :

- Vous ne pouvez plus créer de clusters HDInsight version 3.3 basé sur Linux (ni des versions antérieures) à l’aide d’un service lié HDInsight à la demande dans Data Factory version 1. 
- Si les propriétés [**osType** et **Version**](#azure-hdinsight-on-demand-linked-service) ne sont pas spécifiées explicitement dans la définition JSON d’un service lié HDInsight à la demande Data Factory version 1, la valeur par défaut **Version=3.1, osType=Windows** est remplacée par **Version=\<latest HDI default version\>(https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning) , osType=Linux**.

Après le 31 juillet 2018 :

- Vous ne pouvez plus créer de clusters HDInsight basé sur Windows à l’aide d’un service lié HDInsight à la demande dans Data Factory version 1. 

### <a name="recommended-actions"></a>Actions recommandées

- Afin de vous assurer de pouvoir utiliser les tout derniers correctifs et composants de l’écosystème Hadoop, mettez à jour les propriétés [**osType** et **Version**](#azure-hdinsight-on-demand-linked-service) des définitions du service lié HDInsight à la demande d’Azure Data Factory version 1 vers de nouvelles versions de HDInsight basé sur Linux (HDInsight 3.6). 
- Avant le 15 décembre 2017, testez les activités de diffusion en continu Hive, Pig, MapReduce et Hadoop de Data Factory version 1 faisant référence au service lié concerné. Vérifiez qu’elles sont compatibles avec les nouvelles valeurs **osType** et **Version** par défaut (**Version=3.6**, **osType=Linux**) ou avec la version HDInsight explicite et le type de système d’exploitation vers lequel vous mettez à niveau. 
  Pour plus d’informations sur la compatibilité, consultez [Effectuer la migration d’un cluster HDInsight Windows vers un cluster Linux](../../hdinsight/index.yml) et [Quels sont les composants et versions Hadoop disponibles avec HDInsight ?](../../hdinsight/hdinsight-component-versioning.md). 
- Pour continuer à utiliser un service lié HDInsight à la demande d’Azure Data Factory version 1 afin de créer des clusters HDInsight basé sur Windows, définissez explicitement **osType** to **Windows** avant le 15 décembre 2017. Nous vous recommandons de procéder à la migration vers les clusters HDInsight basé sur Linux avant le 31 juillet 2018. 
- Si vous utilisez un service lié HDInsight à la demande pour exécuter une activité personnalisée DotNet d’Azure Data Factory version 1, mettez à jour la définition JSON de l’activité personnalisée DotNet pour utiliser plutôt un service lié Azure Batch. Pour plus d’informations, consultez [Utilisation des activités personnalisées dans un pipeline Data Factory](./data-factory-use-custom-activities.md). 

> [!Note]
> Si vous utilisez votre propre appareil lié HDInsight de cluster dans Data Factory version 1 ou votre propre service lié HDInsight à la demande dans Azure Data Factory, aucune action n’est requise. Dans ces scénarios, la dernière stratégie de prise en charge de version des clusters HDInsight est déjà appliquée. 
>
> 


## <a name="on-demand-compute-environment"></a>Environnement de calcul à la demande
Dans une configuration à la demande, Data Factory gère totalement l’environnement de calcul. Data Factory crée automatiquement l’environnement de calcul avant l’envoi d’un travail de traitement des données. Lorsque le travail est terminé, Data Factory supprime l’environnement de calcul. 

Vous pouvez créer un service lié pour un environnement de calcul à la demande. Utilisez le service lié pour configurer l’environnement de calcul, et pour contrôler les paramètres granulaires pour l’exécution de la tâche, la gestion du cluster et les actions d’amorçage.

> [!NOTE]
> Actuellement, la configuration à la demande est prise en charge uniquement pour les clusters HDInsight.
> 

## <a name="azure-hdinsight-on-demand-linked-service"></a>Service lié à la demande Azure HDInsight
Data Factory peut automatiquement créer un cluster HDInsight à la demande basé sur Windows ou Linux pour traiter les données. Le cluster est créé dans la même région que celle du compte de stockage qui est associé au cluster. Utilisez la propriété **linkedServiceName** JSON pour créer le cluster.

Notez les points *clés* suivants sur le service lié HDInsight à la demande :

* Le cluster HDInsight à la demande n’apparaît pas dans votre abonnement Azure. Le service Data Factory gère le cluster HDInsight à la demande à votre place.
* Les journaux d’activité des tâches exécutées sur un cluster HDInsight à la demande sont copiés dans le compte de stockage qui est associé au cluster HDInsight. Pour accéder à ces journaux d’activité, dans le portail Azure, accédez au volet **Détails sur l’exécution d’activité**. Pour plus d’informations, consultez [Surveiller et gérer les pipelines](data-factory-monitor-manage-pipelines.md).
* Vous êtes facturé uniquement lorsque le cluster HDInsight est actif et exécute des tâches.

> [!IMPORTANT]
> Il faut généralement au moins *20 minutes* pour mettre en service un cluster HDInsight à la demande.
>
> 

### <a name="example"></a>Exemple
Le JSON suivant définit un service lié HDInsight à la demande sous Linux. Data Factory crée automatiquement un cluster HDInsight *basé sur Linux* lorsqu’il traite une tranche de données. 

```json
{
    "name": "HDInsightOnDemandLinkedService",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.6",
            "osType": "Linux",
            "clusterSize": 1,
            "timeToLive": "00:05:00",            
            "linkedServiceName": "AzureStorageLinkedService"
        }
    }
}
```

> [!IMPORTANT]
> Le cluster HDInsight crée un *conteneur par défaut* dans le stockage d’objets blob Azure que vous spécifiez dans la propriété **linkedServiceName** JSON. De par sa conception, HDInsight ne supprime pas ce conteneur lorsque le cluster est supprimé. Dans un service lié HDInsight à la demande, un cluster HDInsight est créé dès qu’une tranche doit être traitée, à moins qu’il n’existe un cluster actif (**timeToLive**). Ce cluster est automatiquement supprimé lorsque le traitement est terminé. 
>
> À mesure que le nombre de tranches traitées augmente, les conteneurs se multiplient dans votre stockage d’objets blob. Si vous n’avez pas besoin des conteneurs pour dépanner les travaux, vous pouvez les supprimer afin de réduire les frais de stockage. Les noms de ces conteneurs sont conformes au modèle suivant : `adf<your Data Factory name>-<linked service name>-<date and time>`. Vous pouvez utiliser un outil tel que l’[Explorateur Stockage Microsoft Azure](https://storageexplorer.com/) pour supprimer des conteneurs du Stockage Blob.
>
> 

### <a name="properties"></a>Propriétés
| Propriété                     | Description                              | Obligatoire |
| ---------------------------- | ---------------------------------------- | -------- |
| type                         | Définissez la propriété de type sur **HDInsightOnDemand**. | Oui      |
| clusterSize                  | Le nombre de nœuds worker et de données dans le cluster. Le cluster HDInsight est créé avec 2 nœuds principaux, en plus du nombre de nœuds worker que vous spécifiez pour cette propriété. Les nœuds sont de taille Standard_D3 à 4 cœurs. Un cluster à 4 nœuds worker prend 24 cœurs (4\*4 = 16 nœuds pour les nœuds worker + 2\*4 = 8 cœurs pour les nœuds principaux). Pour plus d’informations sur le niveau Standard_D3, consultez [Création de clusters Hadoop basés sur Linux dans HDInsight](../../hdinsight/hdinsight-hadoop-provision-linux-clusters.md). | Oui      |
| timeToLive                   | La durée d’inactivité autorisée pour le cluster HDInsight à la demande. Spécifie la durée pendant laquelle le cluster HDInsight à la demande reste actif lorsqu’une exécution d’activité est terminée s’il n’existe aucun autre travail actif dans le cluster.<br /><br />Par exemple, si une exécution d’activité prend 6 minutes et si la propriété **timeToLive** est définie sur 5 minutes, le cluster reste actif pendant 5 minutes après les 6 minutes du traitement de l’exécution d’activité. Si une autre exécution d’activité intervient dans la fenêtre de 6 minutes, elle est traitée par le même cluster.<br /><br />La création d’un cluster HDInsight à la demande est une opération coûteuse (elle peut prendre un certain temps). Utilisez ce paramètre si nécessaire pour améliorer les performances d’une fabrique de données en réutilisant un cluster HDInsight à la demande.<br /><br />Si vous définissez la valeur de la propriété **timeToLive** sur **0**, le cluster est supprimé dès que l’exécution d’activité se termine. Toutefois, si vous définissez une valeur élevée, le cluster peut rester inactif inutilement, entraînant des coûts élevés. Il est important de définir la valeur appropriée en fonction de vos besoins.<br /><br />Plusieurs pipelines peuvent partager l’instance du cluster HDInsight à la demande si la valeur de la propriété **timeToLive** est correctement définie. | Oui      |
| version                      | La version du cluster HDInsight. Pour connaître les versions de HDInsight autorisées, consultez [Versions de HDInsight prises en charge](../../hdinsight/hdinsight-component-versioning.md#supported-hdinsight-versions). Si cette valeur n’est pas spécifiée, la [dernière version HDI par défaut](../../hdinsight/hdinsight-component-versioning.md) est utilisée. | Non       |
| linkedServiceName            | Le service lié Stockage Azure utilisé par le cluster à la demande pour le stockage et le traitement des données. Le cluster HDInsight est créé dans la même région que ce compte de stockage.<p>Actuellement, vous ne pouvez pas créer un cluster HDInsight à la demande qui utilise un Azure Data Lake Store en guise de stockage. Si vous souhaitez stocker les données de résultat à partir du traitement HDInsight dans un Data Lake Store, utilisez l’activité de copie pour copier les données du stockage blob dans Data Lake Store. </p> | Oui      |
| additionalLinkedServiceNames | Spécifie des comptes de stockage supplémentaires pour le service lié HDInsight. Data Factory inscrit les comptes de stockage en votre nom. Ces comptes de stockage doivent se trouver dans la même région que le cluster HDInsight. Le cluster HDInsight est créé dans la même région que celle du compte de stockage qui est spécifié par la propriété **linkedServiceName**. | Non       |
| osType                       | Le type de système d’exploitation. Valeurs autorisées : **Linux** et **Windows**. Si cette valeur n’est pas spécifiée, **Linux** est utilisé.  <br /><br />Nous vous recommandons vivement d’utiliser des clusters HDInsight basés sur Linux. La date de mise hors service de HDInsight sur Windows est le 31 juillet 2018. | Non       |
| hcatalogLinkedServiceName    | Le nom du service lié SQL Azure pointant vers la base de données HCatalog. Le cluster HDInsight à la demande est créé en utilisant la base de données SQL en tant que metastore. | Non       |

#### <a name="example-linkedservicenames-json"></a>Exemple : JSON LinkedServiceNames

```json
"additionalLinkedServiceNames": [
    "otherLinkedServiceName1",
    "otherLinkedServiceName2"
  ]
```

### <a name="advanced-properties"></a>Propriétés avancées
Pour la configuration granulaire du cluster HDInsight à la demande, vous pouvez spécifier les propriétés suivantes :

| Propriété               | Description                              | Obligatoire |
| :--------------------- | :--------------------------------------- | :------- |
| coreConfiguration      | Spécifie les paramètres de configuration de base (core-site.xml) pour le cluster HDInsight à créer. | Non       |
| hBaseConfiguration     | Spécifie les paramètres de configuration HBase (hbase-site.xml) pour le cluster HDInsight. | Non       |
| hdfsConfiguration      | Spécifie les paramètres de configuration HDFS (hdfs-site.xml) pour le cluster HDInsight. | Non       |
| hiveConfiguration      | Spécifie les paramètres de configuration Hive (hive-site.xml) pour le cluster HDInsight. | Non       |
| mapReduceConfiguration | Spécifie les paramètres de configuration MapReduce (mapred-site.xml) pour le cluster HDInsight. | Non       |
| oozieConfiguration     | Spécifie les paramètres de configuration Oozie (oozie-site.xml) pour le cluster HDInsight. | Non       |
| stormConfiguration     | Spécifie les paramètres de configuration Storm (storm-site.xml) pour le cluster HDInsight. | Non       |
| yarnConfiguration      | Spécifie les paramètres de configuration YARN (yarn-site.xml) pour le cluster HDInsight. | Non       |

#### <a name="example-on-demand-hdinsight-cluster-configuration-with-advanced-properties"></a>Exemple : Configuration de cluster HDInsight à la demande avec les propriétés avancées

```json
{
  "name": " HDInsightOnDemandLinkedService",
  "properties": {
    "type": "HDInsightOnDemand",
    "typeProperties": {
      "version": "3.6",
      "osType": "Linux",
      "clusterSize": 16,
      "timeToLive": "01:30:00",
      "linkedServiceName": "adfods1",
      "coreConfiguration": {
        "templeton.mapper.memory.mb": "5000"
      },
      "hiveConfiguration": {
        "templeton.mapper.memory.mb": "5000"
      },
      "mapReduceConfiguration": {
        "mapreduce.reduce.java.opts": "-Xmx4000m",
        "mapreduce.map.java.opts": "-Xmx4000m",
        "mapreduce.map.memory.mb": "5000",
        "mapreduce.reduce.memory.mb": "5000",
        "mapreduce.job.reduce.slowstart.completedmaps": "0.8"
      },
      "yarnConfiguration": {
        "yarn.app.mapreduce.am.resource.mb": "5000",
        "mapreduce.map.memory.mb": "5000"
      },
      "additionalLinkedServiceNames": [
        "datafeeds",
        "adobedatafeed"
      ]
    }
  }
}
```

### <a name="node-sizes"></a>Tailles de nœuds
Pour spécifier les tailles du nœud principal, du nœud de données et du nœud ZooKeeper, utilisez les propriétés suivantes : 

| Propriété          | Description                              | Obligatoire |
| :---------------- | :--------------------------------------- | :------- |
| headNodeSize      | Définit la taille du nœud principal. La valeur par défaut est **Standard_D3**. Pour plus d’informations, consultez [Spécifier des tailles de nœuds](#specify-node-sizes). | Non       |
| dataNodeSize      | Définit la taille du nœud de données. La valeur par défaut est **Standard_D3**. | Non       |
| zookeeperNodeSize | Définit la taille du nœud ZooKeeper. La valeur par défaut est **Standard_D3**. | Non       |

#### <a name="specify-node-sizes"></a>Spécifier des tailles de nœuds
Pour les valeurs de chaîne que vous devez spécifier pour les propriétés décrites dans la section précédente, consultez [Tailles des machines virtuelles](../../virtual-machines/sizes.md). Les valeurs doivent être conformes aux applets de commande et aux API référencées dans [Tailles des machines virtuelles](../../virtual-machines/sizes.md). La taille du nœud de données Grande taille (valeur par défaut) inclut 7 Go de mémoire. Cela peut ne pas être suffisant pour votre scénario. 

Si vous voulez créer des nœuds principaux et des nœuds worker de taille D4, spécifiez la valeur **Standard_D4** pour les propriétés **headNodeSize** et **dataNodeSize** : 

```json
"headNodeSize": "Standard_D4",    
"dataNodeSize": "Standard_D4",
```

Si vous définissez une valeur incorrecte pour ces propriétés, le message suivant peut s’afficher :

  Failed to create cluster. (Impossible de créer le cluster.) Exception : Impossible de terminer l’opération de création du cluster. Operation failed with code ’400’. Cluster left behind state (État du cluster abandonné) : 'Error' (« Error »). Message : 'PreClusterCreationValidationFailure'. 
  
Si vous voyez ce message, vérifiez que vous utilisez l’applet de commande et les noms d’API du tableau dans [Tailles des machines virtuelles](../../virtual-machines/sizes.md).  

> [!NOTE]
> Actuellement, Data Factory ne prend pas en charge les clusters HDInsight qui utilisent Data Lake Store comme magasin principal. Utilisez Stockage Azure comme magasin principal pour les clusters HDInsight. 
>
> 


## <a name="bring-your-own-compute-environment"></a>Apportez votre propre environnement de calcul
Vous pouvez inscrire un environnement de calcul existant en tant que service lié dans Data Factory. Vous gérez l’environnement de calcul. Le service Data Factory utilise l’environnement de calcul pour exécuter des activités.

Ce type de configuration est pris en charge pour les environnements de calcul suivants :

* Azure HDInsight
* Azure Batch
* Azure Machine Learning Studio (classique)
* Service Analytique Azure Data Lake
* Azure SQL Database, Azure Synapse Analytics, SQL Server

## <a name="azure-hdinsight-linked-service"></a>Service lié Azure HDInsight
Vous pouvez créer un service lié HDInsight pour inscrire votre propre cluster HDInsight avec Data Factory.

### <a name="example"></a>Exemple

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

### <a name="properties"></a>Propriétés
| Propriété          | Description                              | Obligatoire |
| ----------------- | ---------------------------------------- | -------- |
| type              | Définissez la propriété de type sur **HDInsight**. | Oui      |
| clusterUri        | L'URI du cluster HDInsight.        | Oui      |
| username          | Le nom du compte d’utilisateur à utiliser pour se connecter à un cluster HDInsight existant. | Oui      |
| mot de passe          | Mot de passe du compte d’utilisateur.   | Oui      |
| linkedServiceName | Le nom du service lié de stockage faisant référence au stockage blob utilisé par le cluster HDInsight. <p>Actuellement, vous ne pouvez pas spécifier un service lié Data Lake Store pour cette propriété. Si le cluster HDInsight a accès à Data Lake Store, vous pouvez accéder aux données de Data Lake Store à partir de scripts Hive ou Pig. </p> | Oui      |

## <a name="azure-batch-linked-service"></a>Service lié Azure Batch
Vous pouvez créer un service lié Batch pour inscrire un pool de machines virtuelles (VM) Batch à une fabrique de données. Vous pouvez exécuter des activités personnalisées Microsoft .NET à l’aide de Batch ou de HDInsight.

Si vous ne savez pas utiliser le service Batch :

* En savoir plus sur les [concepts de base d’Azure Batch](../../azure-sql/database/sql-database-paas-overview.md).
* En savoir plus sur l’applet de commande [New-AzureBatchAccount](/previous-versions/azure/mt125880(v=azure.100)). Utilisez cette applet de commande pour créer un compte Batch. Vous pouvez également créer un compte Batch à l’aide du [portail Azure](../../batch/batch-account-create-portal.md). Pour obtenir des instructions détaillées sur l’utilisation de l’applet de commande, consultez [Utilisation de PowerShell pour gérer un compte Batch](/archive/blogs/windowshpc/using-azure-powershell-to-manage-azure-batch-account).
* En savoir plus sur l’applet de commande [New-AzureBatchPool](/previous-versions/azure/mt125936(v=azure.100)). Utilisez cette applet de commande pour créer un pool Batch.

### <a name="example"></a>Exemple

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

Pour la propriété **accountName**, ajoutez **.\<region name\>** au nom de votre compte Batch. Par exemple :

```json
"accountName": "mybatchaccount.eastus"
```

Une autre option consiste à fournir le point de terminaison **batchUri**. Par exemple :

```json
"accountName": "adfteam",
"batchUri": "https://eastus.batch.azure.com",
```

### <a name="properties"></a>Propriétés
| Propriété          | Description                              | Obligatoire |
| ----------------- | ---------------------------------------- | -------- |
| type              | Définissez la propriété de type sur **AzureBatch**. | Oui      |
| accountName       | Le nom du compte Batch.         | Oui      |
| accessKey         | La clé d’accès du compte Batch.  | Oui      |
| poolName          | Le nom du pool de machines virtuelles.    | Oui      |
| linkedServiceName | Le nom du service lié de stockage qui est associé à ce service lié Batch. Ce service lié est utilisé pour présenter les fichiers nécessaires à l’exécution de l’activité et pour stocker les journaux d’activité d’exécution de l’activité. | Oui      |

## <a name="azure-machine-learning-studio-classic-linked-service"></a>Service lié Azure Machine Learning Studio (classique)
Vous pouvez créer un service lié Azure Machine Learning Studio (classique) pour inscrire un point de terminaison de notation par lot Studio (classique) auprès d’une fabrique de données.

### <a name="example"></a>Exemple

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

### <a name="properties"></a>Propriétés
| Propriété   | Description                              | Obligatoire |
| ---------- | ---------------------------------------- | -------- |
| Type       | Définissez la propriété de type sur **AzureML**. | Oui      |
| mlEndpoint | L'URL de la notation par lot.                   | Oui      |
| apiKey     | L'API du modèle d'espace de travail publié.     | Oui      |

## <a name="azure-data-lake-analytics-linked-service"></a>Service lié Azure Data Lake Analytics
Vous créez un service lié Data Lake Analytics pour lier un service de calcul Data Lake Analytics à une fabrique de données Azure. L’activité U-SQL Analytique Data Lake dans le pipeline fait référence à ce service lié. 

Le tableau suivant décrit les propriétés génériques utilisées dans la définition JSON :

| Propriété                 | Description                              | Obligatoire                                 |
| ------------------------ | ---------------------------------------- | ---------------------------------------- |
| type                 | Définissez la propriété de type sur **AzureDataLakeAnalytics**. | Oui                                      |
| accountName          | Le nom du compte Data Lake Analytics.  | Oui                                      |
| dataLakeAnalyticsUri | L’URI de Data Lake Analytics.           | Non                                       |
| subscriptionId       | L’ID d’abonnement Azure.                    | Non<br /><br />(Si non spécifié, l’abonnement de la fabrique de données est utilisé.) |
| resourceGroupName    | Le nom du groupe de ressources Azure.                | Non<br /><br /> (Si non spécifié, le groupe de ressources de la fabrique de données est utilisé.) |

### <a name="authentication-options"></a>Options d’authentification
Pour votre service lié Data Lake Analytics, vous avez le choix entre l’authentification à l’aide d’un principal du service ou d’informations d’identification utilisateur.

#### <a name="service-principal-authentication-recommended"></a>Authentification d’un principal du service (recommandée)
Pour utiliser une authentification du principal du service, inscrivez une entité d’application dans Azure Active Directory (Azure AD). Accordez ensuite à Azure AD l’accès à Data Lake Store. Consultez la page [Authentification de service à service](../../data-lake-store/data-lake-store-service-to-service-authenticate-using-active-directory.md) pour des instructions détaillées. Prenez note des valeurs suivantes, qui vous permettent de définir le service lié :
* ID de l'application
* Clé de l'application 
* ID client

Utilisez l’authentification par principal de service en spécifiant les propriétés suivantes :

| Propriété                | Description                              | Obligatoire |
| :---------------------- | :--------------------------------------- | :------- |
| servicePrincipalId  | L’ID client de l’application.     | Oui      |
| servicePrincipalKey | La clé de l’application.           | Oui      |
| tenant              | Les informations sur le locataire (nom de domaine ou ID de locataire) dans lesquels votre application se trouve. Pour obtenir ces informations, passez la souris sur le coin supérieur droit du portail Azure. | Oui      |

**Exemple : Authentification d’un principal de service**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

#### <a name="user-credential-authentication"></a>Authentification des informations d’identification utilisateur
Pour l’authentification par informations d’identification utilisateur pour Data Lake Analytics, spécifiez les propriétés suivantes :

| Propriété          | Description                              | Obligatoire |
| :---------------- | :--------------------------------------- | :------- |
| autorisation | Dans l’éditeur Data Factory, sélectionnez le bouton **Autoriser**. Entrez les informations d’identification qui associent l’URL d’autorisation générée automatiquement à cette propriété. | Oui      |
| sessionID     | L’ID de session OAuth issu de la session d’autorisation OAuth. Chaque ID de session est unique et ne peut être utilisé qu’une seule fois. Ce paramètre est généré automatiquement lorsque vous utilisez Data Factory Editor. | Oui      |

**Exemple : Authentification des informations d’identification utilisateur**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
            "authorization": "<authcode>",
            "sessionId": "<session ID>", 
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

#### <a name="token-expiration"></a>Expiration du jeton
Le code d’autorisation que vous avez généré en sélectionnant le bouton **Autoriser** expire au bout d’un intervalle défini. 

Vous pouvez rencontrer le message d’erreur suivant lorsque le jeton d’authentification expire : 

  Erreur de l’opération des informations d’identification : invalid_grant - AADSTS70002 : Erreur de validation des informations d’identification. AADSTS70008 : L’autorisation d’accès fournie a expiré ou est révoquée. ID de trace : d18629e8-af88-43c5-88e3-d8419eb1fca1 ID de corrélation : fac30a0c-6be6-4e02-8d69-a776d2ffefd7 Horodatage : 2015-12-15 21:09:31Z

Le tableau suivant présente les délais d’expiration par type de compte d’utilisateur : 

| Type d’utilisateur                                | Expire après                            |
| :--------------------------------------- | :--------------------------------------- |
| Comptes d’utilisateur qui sont *pas* gérés par Azure AD (Hotmail, Live, etc.) | 12 heures.                                 |
| Comptes d’utilisateurs qui *sont* gérés par Azure AD | 14 jours après la dernière exécution de tranche de données. <br /><br />90 jours, si une tranche basée sur un service lié OAuth est exécutée au moins une fois tous les 14 jours. |

Pour éviter ou résoudre cette erreur, accordez une nouvelle autorisation en sélectionnant le bouton **Autoriser** au moment de l’expiration du jeton. Ensuite, redéployez le service lié. Vous pouvez également générer des valeurs pour les propriétés **sessionId** et **authorization** à l’aide du code suivant :

```csharp
if (linkedService.Properties.TypeProperties is AzureDataLakeStoreLinkedService ||
    linkedService.Properties.TypeProperties is AzureDataLakeAnalyticsLinkedService)
{
    AuthorizationSessionGetResponse authorizationSession = this.Client.OAuth.Get(this.ResourceGroupName, this.DataFactoryName, linkedService.Properties.Type);

    WindowsFormsWebAuthenticationDialog authenticationDialog = new WindowsFormsWebAuthenticationDialog(null);
    string authorization = authenticationDialog.AuthenticateAAD(authorizationSession.AuthorizationSession.Endpoint, new Uri("urn:ietf:wg:oauth:2.0:oob"));

    AzureDataLakeStoreLinkedService azureDataLakeStoreProperties = linkedService.Properties.TypeProperties as AzureDataLakeStoreLinkedService;
    if (azureDataLakeStoreProperties != null)
    {
        azureDataLakeStoreProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeStoreProperties.Authorization = authorization;
    }

    AzureDataLakeAnalyticsLinkedService azureDataLakeAnalyticsProperties = linkedService.Properties.TypeProperties as AzureDataLakeAnalyticsLinkedService;
    if (azureDataLakeAnalyticsProperties != null)
    {
        azureDataLakeAnalyticsProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeAnalyticsProperties.Authorization = authorization;
    }
}
```

Pour plus d’informations sur les classes Data Factory utilisées dans cet exemple de code, consultez :
* [Classe AzureDataLakeStoreLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice)
* [Classe AzureDataLakeAnalyticsLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice)
* [Classe AuthorizationSessionGetResponse](/dotnet/api/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse)

Ajoutez une référence à Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll pour la classe **WindowsFormsWebAuthenticationDialog**. 

## <a name="azure-sql-linked-service"></a>Service lié SQL Azure
Vous pouvez créer un service lié SQL et l’utiliser avec l’[activité de procédure stockée](data-factory-stored-proc-activity.md) pour appeler une procédure stockée à partir d’un pipeline Data Factory. Pour plus d’informations, consultez [Azure SQL connector](data-factory-azure-sql-connector.md#linked-service-properties) (Connecteur Azure SQL).

## <a name="azure-synapse-analytics-linked-service"></a>Service lié Azure Synapse Analytics
Vous pouvez créer un service lié Azure Synapse Analytics et l’utiliser avec l’[activité de procédure stockée](data-factory-stored-proc-activity.md) pour appeler une procédure stockée à partir d’un pipeline Data Factory. Pour plus d’informations, consultez [Connecteur Azure Synapse Analytics](data-factory-azure-sql-data-warehouse-connector.md#linked-service-properties).

## <a name="sql-server-linked-service"></a>Service lié SQL Server
Vous pouvez créer un service lié SQL Server et l’utiliser avec l’[activité de procédure stockée](data-factory-stored-proc-activity.md) pour appeler une procédure stockée à partir d’un pipeline Data Factory. Pour plus d’informations, consultez [SQL Server connector](data-factory-sqlserver-connector.md#linked-service-properties) (Connecteur SQL Server).