---
title: 'Tutoriel : Créer un pipeline pour déplacer les données à l’aide d’Azure PowerShell '
description: Dans ce didacticiel, vous créez un pipeline Azure Data Factory avec une activité de copie à l’aide d’Azure PowerShell.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: ''
editor: ''
ms.assetid: 71087349-9365-4e95-9847-170658216ed8
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: aa97fd2ac12ca73d1a317fb09e03e49d0056ceb1
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97608627"
---
# <a name="tutorial-create-a-data-factory-pipeline-that-moves-data-by-using-azure-powershell"></a>Tutoriel : créer un pipeline Data Factory qui déplace les données à l’aide d’Azure PowerShell
> [!div class="op_single_selector"]
> * [Vue d’ensemble et composants requis](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Assistant de copie](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Modèle Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez le [tutoriel sur l’activité de copie](../quickstart-create-data-factory-powershell.md). 

Dans cet article, vous allez apprendre à utiliser PowerShell pour créer une fabrique de données avec un pipeline qui copie les données d’un stockage Blob Azure dans Azure SQL Database. Si vous débutez avec Azure Data Factory, lisez l’article [Présentation d’Azure Data Factory](data-factory-introduction.md) avant de suivre ce didacticiel.   

Dans ce tutoriel, vous créez un pipeline contenant une seule activité : une activité de copie. L’activité de copie copie les données d’un magasin de données pris en charge vers un magasin de données de récepteur pris en charge. Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs, consultez [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Elle est mise en œuvre par un service disponible dans le monde entier, capable de copier des données entre différents magasins de données de façon sécurisée, fiable et évolutive. Pour plus d’informations sur l’activité de copie, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md).

Un pipeline peut contenir plusieurs activités. En outre, vous pouvez chaîner deux activités (une après l’autre) en configurant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. Pour plus d’informations, consultez [Plusieurs activités dans un pipeline](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).

> [!NOTE]
> cet article ne couvre pas toutes les applets de commande Data Factory. Consultez la [Référence des applets de commande Data Factory](/powershell/module/az.datafactory) pour obtenir une documentation complète sur ces applets de commande.
> 
> Dans ce didacticiel, le pipeline de données copie les données d’un magasin de données source vers un magasin de données de destination. Pour suivre un tutoriel sur la transformation des données à l’aide d’Azure Data Factory, consultez [Tutoriel : Créer un pipeline pour transformer des données à l’aide d’un cluster Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

- Assurez-vous que vous respectez la configuration requise décrite dans l’article [Configuration requise pour le didacticiel](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
- Installez **Azure PowerShell**. Suivez les instructions de la page [Installation et configuration d’Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="steps"></a>Étapes
Voici les étapes à effectuer dans le cadre de ce didacticiel :

1. Créez une **fabrique de données** Azure. Dans cette étape, vous créez une fabrique de données nommée ADFTutorialDataFactoryPSH. 
1. Créez des **services liés** dans la fabrique de données. Dans cette étape, vous créez deux services liés des types suivants : Stockage Azure et Azure SQL Database. 

    AzureStorageLinkedService relie votre compte de stockage Azure à la fabrique de données. Vous avez créé un conteneur et chargé des données dans ce compte de stockage en remplissant les [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).   

    AzureSqlLinkedService lie Azure SQL Database à la fabrique de données. Les données copiées à partir du stockage Blob sont stockées dans cette base de données. Vous avez créé une table SQL dans cette base de données en remplissant les [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).   
1. Créez des **jeux de données** d’entrée et de sortie dans la fabrique de données.  

    Le service lié Stockage Azure spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à votre compte de stockage Azure. Le jeu de données blob d’entrée spécifie quant à lui le conteneur et le dossier qui contient les données d’entrée.  

    De même, le service lié Azure SQL Database spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à votre base de données. Et le jeu de données de la table SQL de sortie spécifie la table de la base de données dans laquelle les données du stockage Blob sont copiées.
1. Créez un **pipeline** dans la fabrique de données. Dans cette étape, vous allez créer un pipeline avec une activité de copie.   

    Cette activité copie les données d’un objet blob du stockage Blob Azure dans une table Azure SQL Database. Vous pouvez utiliser une activité de copie dans un pipeline pour copier les données d’une source prise en charge dans une destination prise en charge. Pour obtenir la liste des magasins de données pris en charge, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md#supported-data-stores-and-formats). 
1. Surveillez le pipeline. Dans cette étape, vous allez **surveiller** les tranches de jeux de données d’entrée et de sortie à l’aide de PowerShell.

## <a name="create-a-data-factory"></a>Créer une fabrique de données
> [!IMPORTANT]
> Assurez-vous de remplir les [conditions préalables à l’exécution du didacticiel](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) si ce n’est déjà fait.   

Une fabrique de données peut avoir un ou plusieurs pipelines. Un pipeline peut contenir une ou plusieurs activités. Par exemple, une activité de copie pour copier des données d’une source vers un magasin de données de destination, et une activité Hive HDInsight pour exécuter un script Hive pour transformer des données d’entrée et produire des données de sortie. Commençons par la création de la fabrique de données dans cette étape.

1. Lancez **PowerShell**. Conservez Azure PowerShell ouvert jusqu’à la fin de ce tutoriel. Si vous fermez puis rouvrez Azure PowerShell, vous devez réexécuter ces commandes.

    Exécutez la commande suivante, puis saisissez le nom d’utilisateur et le mot de passe que vous avez utilisés pour la connexion au portail Azure :

    ```powershell
    Connect-AzAccount
    ```   

    Exécutez la commande suivante pour afficher tous les abonnements de ce compte :

    ```powershell
    Get-AzSubscription
    ```

    Exécutez la commande suivante pour sélectionner l’abonnement que vous souhaitez utiliser. Remplacez **&lt;NameOfAzureSubscription**&gt; par le nom de votre abonnement Azure :

    ```powershell
    Get-AzSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzContext
    ```
1. Créez un groupe de ressources Azure nommé **ADFTutorialResourceGroup** en exécutant la commande suivante :

    ```powershell
    New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```

    Certaines étapes de ce didacticiel supposent que vous utilisez le groupe de ressources nommé **ADFTutorialResourceGroup**. Si vous utilisez un autre groupe de ressources, vous devrez l’utiliser à la place d’ADFTutorialResourceGroup dans ce didacticiel.
1. Exécutez l’applet de commande **New-AzDataFactory** pour créer une fabrique de données nommée **ADFTutorialDataFactoryPSH** :  

    ```powershell
    $df=New-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH –Location "West US"
    ```
    Ce nom peut avoir déjà été utilisé. Par conséquent, utilisez un nom de fabrique de données unique en ajoutant un préfixe ou suffixe (par exemple : ADFTutorialDataFactoryPSH05152017), puis réexécutez la commande.  

Notez les points suivants :

* Le nom de la fabrique de données Azure doit être un nom global unique. Si l’erreur suivante s’affiche, changez le nom (par exemple, votrenomADFTutorialDataFactoryPSH). Utilisez ce nom à la place d’ADFTutorialFactoryPSH quand vous effectuez les étapes de ce didacticiel. Consultez la rubrique [Data Factory – Règles d’affectation des noms](data-factory-naming-rules.md) pour les artefacts Data Factory.

    ```
    Data factory name “ADFTutorialDataFactoryPSH” is not available
    ```
* Pour créer des instances de fabrique de données, vous devez avoir le statut d’administrateur/collaborateur de l’abonnement Azure.
* Le nom de la fabrique de données pourra être enregistré en tant que nom DNS et devenir ainsi visible publiquement.
* Vous risquez de recevoir l’erreur suivante : « **Cet abonnement n’est pas inscrit pour utiliser l’espace de noms Microsoft.DataFactory.**  » Effectuez l’une des opérations suivantes, puis essayez de publier à nouveau :

  * Dans Azure PowerShell, exécutez la commande suivante pour enregistrer le fournisseur Data Factory :

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```

    Exécutez la commande suivante pour confirmer l’enregistrement du fournisseur Data Factory :

    ```powershell
    Get-AzResourceProvider
    ```
  * Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide de votre abonnement Azure. Accédez à un panneau Data Factory, ou créez une fabrique de données dans le portail Azure. Cette action enregistre automatiquement le fournisseur.

## <a name="create-linked-services"></a>Créez des services liés
Vous allez créer des services liés dans une fabrique de données pour lier vos magasins de données et vos services de calcul à la fabrique de données. Dans ce didacticiel, vous n’allez pas utiliser n’importe quel service de calcul comme Azure HDInsight ou Azure Data Lake Analytics. Vous utilisez deux magasins de données de type Stockage Azure (source) et Azure SQL Database (destination). 

Vous créez ainsi deux services liés nommés AzureStorageLinkedService et AzureSqlLinkedService de types : AzureStorage et AzureSqlDatabase.  

AzureStorageLinkedService relie votre compte de stockage Azure à la fabrique de données. Ce compte de stockage est celui dans lequel vous avez créé un conteneur et chargé les données en remplissant les [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).   

AzureSqlLinkedService lie Azure SQL Database à la fabrique de données. Les données copiées à partir du stockage Blob sont stockées dans cette base de données. Vous avez créé une table emp dans cette base de données dans le cadre des [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md). 

### <a name="create-a-linked-service-for-an-azure-storage-account"></a>Créer un service lié pour un compte de stockage Azure
Dans cette étape, vous liez votre compte de stockage Azure à votre fabrique de données.

1. Créez un fichier JSON nommé **AzureStorageLinkedService.json** dans le dossier **C:\ADFGetStartedPSH** avec le contenu suivant : (Créez le dossier ADFGetStartedPSH s’il n’existe pas déjà.)

    > [!IMPORTANT]
    > Remplacez &lt;accountname&gt; et &lt;accountkey&gt; par le nom et la clé de votre compte de stockage Azure avant d’enregistrer le fichier. 

    ```json
    {
        "name": "AzureStorageLinkedService",
        "properties": {
            "type": "AzureStorage",
            "typeProperties": {
                "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
            }
        }
     }
    ``` 
1. Dans **Azure PowerShell**, basculez vers le dossier **ADFGetStartedPSH**.
1. Exécutez l’applet de commande **New-AzDataFactoryLinkedService** pour créer le service lié : **AzureStorageLinkedService**. Cette applet de commande et d’autres applets de commande Data Factory que vous utilisez dans ce didacticiel vous obligent à transmettre des valeurs aux paramètres **ResourceGroupName** et **DataFactoryName**. Vous pouvez également passer l’objet DataFactory retourné par l’applet de commande New-AzDataFactory sans avoir à taper ResourceGroupName et DataFactoryName chaque fois que vous exécutez une applet de commande. 

    ```powershell
    New-AzDataFactoryLinkedService $df -File .\AzureStorageLinkedService.json
    ```
    Voici l'exemple de sortie :

    ```
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.LinkedServiceProperties
    ProvisioningState : Succeeded
    ``` 

    Vous pouvez aussi créer ce service lié en spécifiant le nom du groupe de ressources et le nom de la fabrique de données au lieu de l’objet DataFactory.  

    ```powershell
    New-AzDataFactoryLinkedService -ResourceGroupName ADFTutorialResourceGroup -DataFactoryName <Name of your data factory> -File .\AzureStorageLinkedService.json
    ```

### <a name="create-a-linked-service-for-azure-sql-database"></a>Créer un service lié pour Azure SQL Database
Dans cette étape, vous liez Azure SQL Database à votre fabrique de données.

1. Créez un fichier JSON nommé AzureSqlLinkedService.json dans C:\ADFGetStartedPSH avec le contenu suivant :

    > [!IMPORTANT]
    > Remplacez &lt;servername&gt;, &lt;databasename&gt;, &lt;username@servername&gt;, et &lt;password&gt; par les noms de votre serveur, de la base de données, du compte d’utilisateur et par le mot de passe.

    ```json
    {
        "name": "AzureSqlLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server=tcp:<server>.database.windows.net,1433;Database=<databasename>;User ID=<user>@<server>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
     }
    ```
1. Exécutez la commande suivante pour créer un service lié :

    ```powershell
    New-AzDataFactoryLinkedService $df -File .\AzureSqlLinkedService.json
    ```

    Voici l'exemple de sortie :

    ```
    LinkedServiceName : AzureSqlLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.LinkedServiceProperties
    ProvisioningState : Succeeded
    ```

   Vérifiez que le paramètre **Autoriser l’accès aux services Azure** est activé sur votre serveur. Pour vérifier et l’activer, procédez comme suit :

    1. Connectez-vous au [portail Azure](https://portal.azure.com)
    1. Cliquez **Plus de services** à gauche, puis sur **Serveurs SQL** dans la catégorie **BASES DE DONNÉES**.
    1. Sélectionnez votre serveur dans la liste des serveurs SQL.
    1. Dans le panneau du serveur SQL, cliquez sur le lien **Afficher les paramètres de pare-feu**.
    1. Dans le panneau **Paramètres de pare-feu**, cliquez sur **ACTIVER** pour **Autoriser l’accès aux services Azure**.
    1. Cliquez sur **Save** dans la barre d'outils. 

## <a name="create-datasets"></a>Créez les jeux de données
Dans l’étape précédente, vous avez créé des services liés pour lier votre compte de stockage Azure et Azure SQL Database à votre fabrique de données. Dans cette étape, vous définissez deux jeux de données nommés InputDataset et OutputDataset, qui représentent les données d’entrée/sortie stockées dans les banques de données référencées par AzureStorageLinkedService et AzureSqlLinkedService, respectivement.

Le service lié Stockage Azure spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à votre compte de stockage Azure. Le jeu de données blob d’entrée (InputDataset) spécifie quant à lui le conteneur et le dossier qui contient les données d’entrée.  

De même, le service lié Azure SQL Database spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à votre base de données. Et le jeu de données de la table SQL de sortie (OututDataset) spécifie la table de la base de données dans laquelle les données du stockage Blob sont copiées. 

### <a name="create-an-input-dataset"></a>Créer un jeu de données d’entrée
Dans cette étape, vous créez un jeu de données nommé InputDataset qui pointe vers un fichier blob (emp.txt) dans le dossier racine d’un conteneur d’objets blob (adftutorial) du stockage Azure représenté par le service lié AzureStorageLinkedService. Si vous ne spécifiez pas de valeur pour fileName (ou si vous ignorez ce paramètre), les données de tous les objets blob du dossier d’entrée sont copiées dans la destination. Dans ce didacticiel, vous spécifiez une valeur pour fileName.  

1. Créez un fichier JSON nommé **InputDataset.json** dans le dossier **C:\ADFGetStartedPSH** avec le contenu suivant :

    ```json
    {
        "name": "InputDataset",
        "properties": {
            "structure": [
                {
                    "name": "FirstName",
                    "type": "String"
                },
                {
                    "name": "LastName",
                    "type": "String"
                }
            ],
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "fileName": "emp.txt",
                "folderPath": "adftutorial/",
                "format": {
                    "type": "TextFormat",
                    "columnDelimiter": ","
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

    Le tableau suivant décrit les propriétés JSON utilisées dans l'extrait de code :

    | Propriété | Description |
    |:--- |:--- |
    | type | La propriété du type est définie sur **AzureBlob**, car les données se trouvent dans le stockage blob Azure. |
    | linkedServiceName | Fait référence au service **AzureStorageLinkedService** que vous avez créé précédemment. |
    | folderPath | Spécifie le **conteneur** d’objets blob et le **dossier** contenant les objets blob d’entrée. Dans ce didacticiel, adftutorial est le conteneur d’objets blob et folder est le dossier racine. | 
    | fileName | Cette propriété est facultative. Si vous omettez cette propriété, tous les fichiers spécifiés dans le paramètre folderPath sont récupérés. Dans ce didacticiel, **emp.txt** est spécifié pour le paramètre fileName, si bien que seul ce fichier est récupéré pour le traitement. |
    | format -> type |Le fichier d’entrée étant au format texte, nous utilisons **TextFormat**. |
    | columnDelimiter | Les colonnes du fichier d’entrée sont délimitées par une **virgule (`,`)** . |
    | frequency/interval | La fréquence est définie sur **Heure** et l’intervalle est **1**, ce qui signifie que les segments d’entrée sont disponibles **toutes les heures**. En d’autres termes, le service Data Factory recherche les données d’entrée toutes les heures dans le dossier racine du conteneur d’objets blob (**adftutorial**) que vous avez spécifié. Il recherche des données entre les heures de début et de fin du pipeline, pas avant ni après.  |
    | external | Cette propriété a la valeur **true** si les données ne sont pas générées par ce pipeline. Dans ce didacticiel, les données d’entrée sont dans le fichier emp.txt, qui n’est pas généré par ce pipeline. Nous définissons donc cette propriété sur true. |

    Pour plus d’informations sur ces propriétés JSON, voir [Connecteur de stockage Blob Azure](data-factory-azure-blob-connector.md#dataset-properties).
1. Exécutez la commande suivante pour créer le jeu de données Data Factory :

    ```powershell  
    New-AzDataFactoryDataset $df -File .\InputDataset.json
    ```
    Voici l'exemple de sortie :

    ```
    DatasetName       : InputDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Availability      : Microsoft.Azure.Management.DataFactories.Common.Models.Availability
    Location          : Microsoft.Azure.Management.DataFactories.Models.AzureBlobDataset
    Policy            : Microsoft.Azure.Management.DataFactories.Common.Models.Policy
    Structure         : {FirstName, LastName}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DatasetProperties
    ProvisioningState : Succeeded
    ```

### <a name="create-an-output-dataset"></a>Créer un jeu de données de sortie
Dans cette partie de l’étape, vous créez un jeu de données de sortie nommé **OutputDataset**. Ce jeu de données pointe vers une table SQL dans Azure SQL Database représentée par **AzureSqlLinkedService**. 

1. Créez un fichier JSON nommé **OutputDataset.json** dans le dossier **C:\ADFGetStartedPSH** avec le contenu suivant :

    ```json
    {
        "name": "OutputDataset",
        "properties": {
            "structure": [
                {
                    "name": "FirstName",
                    "type": "String"
                },
                {
                    "name": "LastName",
                    "type": "String"
                }
            ],
            "type": "AzureSqlTable",
            "linkedServiceName": "AzureSqlLinkedService",
            "typeProperties": {
                "tableName": "emp"
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
    }
    ```

    Le tableau suivant décrit les propriétés JSON utilisées dans l'extrait de code :

    | Propriété | Description |
    |:--- |:--- |
    | type | La propriété du type est définie sur **AzureSqlTable** car les données sont copiées dans une table dans Azure SQL Database. |
    | linkedServiceName | Fait référence au service **AzureSqlLinkedService** que vous avez créé précédemment. |
    | tableName | Spécifie la **table** dans laquelle les données sont copiées. | 
    | frequency/interval | La fréquence est définie sur **Heure** et l’intervalle sur **1**, ce qui signifie que les tranches de sortie sont produites **toutes les heures** entre les heures de début et de fin du pipeline, pas avant ni après.  |

    La table emp de la base de données contient trois colonnes : **ID**, **FirstName** et **LastName**. ID étant une colonne d’identité, il vous suffit de spécifier **FirstName** et **LastName**.

    Pour plus d’informations sur ces propriétés JSON, consultez l’article [Connecteur SQL Azure](data-factory-azure-sql-connector.md#dataset-properties).
1. Exécutez la commande suivante pour créer le jeu de données de fabrique de données.

    ```powershell   
    New-AzDataFactoryDataset $df -File .\OutputDataset.json
    ```

    Voici l'exemple de sortie :

    ```
    DatasetName       : OutputDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Availability      : Microsoft.Azure.Management.DataFactories.Common.Models.Availability
    Location          : Microsoft.Azure.Management.DataFactories.Models.AzureSqlTableDataset
    Policy            :
    Structure         : {FirstName, LastName}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DatasetProperties
    ProvisioningState : Succeeded
    ```

## <a name="create-a-pipeline"></a>Créer un pipeline
Dans cette étape, vous créez un pipeline avec une **activité de copie** qui utilise **InputDataset** en entrée et **OutputDataset** en sortie.

Le jeu de données de sortie pilote actuellement la planification. Dans ce didacticiel, le jeu de données de sortie est configuré pour produire une tranche par heure. Les heures de début et de fin du pipeline sont distantes d’une journée, soit 24 heures. Par conséquent, 24 tranches de jeu de données de sortie sont générées par le pipeline. 

1. Créez un fichier JSON nommé **ADFTutorialPipeline.json** dans le dossier **C:\ADFGetStartedPSH** avec le contenu suivant :

    ```json   
    {
      "name": "ADFTutorialPipeline",
      "properties": {
        "description": "Copy data from a blob to Azure SQL table",
        "activities": [
          {
            "name": "CopyFromBlobToSQL",
            "type": "Copy",
            "inputs": [
              {
                "name": "InputDataset"
              }
            ],
            "outputs": [
              {
                "name": "OutputDataset"
              }
            ],
            "typeProperties": {
              "source": {
                "type": "BlobSource"
              },
              "sink": {
                "type": "SqlSink",
                "writeBatchSize": 10000,
                "writeBatchTimeout": "60:00:00"
              }
            },
            "Policy": {
              "concurrency": 1,
              "executionPriorityOrder": "NewestFirst",
              "retry": 0,
              "timeout": "01:00:00"
            }
          }
        ],
        "start": "2017-05-11T00:00:00Z",
        "end": "2017-05-12T00:00:00Z"
      }
    } 
    ```
    Notez les points suivants :

   - Dans la section des activités, il existe une seule activité dont le **type** a la valeur **Copy**. Pour plus d’informations sur l’activité de copie, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md). Dans les solutions Data Factory, vous pouvez également utiliser [Activités de transformation des données](data-factory-data-transformation-activities.md).
   - L’entrée de l’activité est définie sur **InputDataset** et sa sortie, sur **OutputDataset**. 
   - Dans la section **typeProperties**, **BlobSource** est spécifié en tant que type de source et **SqlSink**, en tant que type de récepteur. Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs pour l’activité de copie, consultez [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Pour apprendre à utiliser un magasin de données pris en charge spécifique en tant que source/récepteur, cliquez sur le lien dans le tableau.  

     Remplacez la valeur de la propriété **start** par le jour actuel et la valeur **end**, par le jour suivant. Si vous le souhaitez, spécifiez uniquement la date et ignorez l'heure. Par exemple, « 2016-02-03 », qui équivaut à « 2016-02-03T00:00:00Z ».

     Les dates/heures de début et de fin doivent toutes deux être au [format ISO](https://en.wikipedia.org/wiki/ISO_8601). Par exemple : 2016-10-14T16:32:41Z. L’heure de fin ( **end** ) est facultative, mais nous allons l’utiliser dans ce didacticiel. 

     Si vous ne spécifiez aucune valeur pour la propriété **end**, cette dernière est calculée comme suit : « **start + 48 heures** ». Pour exécuter le pipeline indéfiniment, spécifiez **9999-09-09** comme valeur pour la propriété **end**.

     Dans l’exemple ci-dessus, il existe 24 tranches de données, car une tranche de données est générée toutes les heures.

     Pour obtenir une description des propriétés JSON dans une définition de pipeline, consultez l’article [Créer des pipelines](data-factory-create-pipelines.md). Pour obtenir une description des propriétés JSON dans une définition d’activité de copie, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md). Pour obtenir une description des propriétés JSON prises en charge par BlobSource, consultez l’article [Connecteur de stockage Blob Azure](data-factory-azure-blob-connector.md). Pour obtenir une description des propriétés JSON prises en charge par SqlSink, consultez l’article [Connecteur Azure SQL Database](data-factory-azure-sql-connector.md).
1. Exécutez la commande suivante pour créer la table de fabrique de données.

    ```powershell   
    New-AzDataFactoryPipeline $df -File .\ADFTutorialPipeline.json
    ```

    Voici l'exemple de sortie : 

    ```
    PipelineName      : ADFTutorialPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.PipelinePropertie
    ProvisioningState : Succeeded
    ```

**Félicitations !** Vous avez créé une fabrique de données Azure, avec un pipeline permettant de copier les données d’un stockage Blob Azure vers Azure SQL Database. 

## <a name="monitor-the-pipeline"></a>Surveiller le pipeline
Au cours de cette étape, vous utilisez Azure PowerShell pour surveiller ce qui se passe dans une fabrique de données Azure.

1. Remplacez &lt;DataFactoryName&gt; par le nom de votre fabrique de données, exécutez **Get-AzDataFactory** et affectez la sortie à une variable $df.

    ```powershell  
    $df=Get-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name <DataFactoryName>
    ```

    Par exemple :
    ```powershell
    $df=Get-AzDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH0516
    ```

    Ensuite, exécutez le contenu de $df pour afficher la sortie suivante : 

    ```
    PS C:\ADFGetStartedPSH> $df

    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DataFactoryId     : 6f194b34-03b3-49ab-8f03-9f8a7b9d3e30
    ResourceGroupName : ADFTutorialResourceGroup
    Location          : West US
    Tags              : {}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DataFactoryProperties
    ProvisioningState : Succeeded
    ```
1. Exécutez **Get-AzDataFactorySlice** pour obtenir des détails sur toutes les tranches de **OutputDatabase**, qui correspond au jeu de données de sortie du pipeline.  

    ```powershell   
    Get-AzDataFactorySlice $df -DatasetName OutputDataset -StartDateTime 2017-05-11T00:00:00Z
    ```

   Ce paramètre doit correspondre à la valeur **Start** dans le pipeline JSON. Vous devez voir 24 tranches, une pour chaque heure entre 12: 00 du jour actuel et 12: 00 le lendemain.

   Voici trois exemples de tranches de la sortie : 

    ``` 
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 11:00:00 PM
    End               : 5/12/2017 12:00:00 AM
    RetryCount        : 0
    State             : Ready
    SubState          :
    LatencyStatus     :
    LongRetryCount    : 0

    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 9:00:00 PM
    End               : 5/11/2017 10:00:00 PM
    RetryCount        : 0
    State             : InProgress
    SubState          :
    LatencyStatus     :
    LongRetryCount    : 0

    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 8:00:00 PM
    End               : 5/11/2017 9:00:00 PM
    RetryCount        : 0
    State             : Waiting
    SubState          : ConcurrencyLimit
    LatencyStatus     :
    LongRetryCount    : 0
    ```
1. Exécutez **Get-AzDataFactoryRun** pour obtenir les détails des exécutions d’activité pour un segment **spécifique**. Copiez la valeur date-heure à partir de la sortie de la commande précédente pour spécifier la valeur du paramètre StartDateTime. 

    ```powershell  
    Get-AzDataFactoryRun $df -DatasetName OutputDataset -StartDateTime "5/11/2017 09:00:00 PM"
    ```

   Voici l'exemple de sortie : 

    ```
    Id                  : c0ddbd75-d0c7-4816-a775-704bbd7c7eab_636301332000000000_636301368000000000_OutputDataset
    ResourceGroupName   : ADFTutorialResourceGroup
    DataFactoryName     : ADFTutorialDataFactoryPSH0516
    DatasetName         : OutputDataset
    ProcessingStartTime : 5/16/2017 8:00:33 PM
    ProcessingEndTime   : 5/16/2017 8:01:36 PM
    PercentComplete     : 100
    DataSliceStart      : 5/11/2017 9:00:00 PM
    DataSliceEnd        : 5/11/2017 10:00:00 PM
    Status              : Succeeded
    Timestamp           : 5/16/2017 8:00:33 PM
    RetryAttempt        : 0
    Properties          : {}
    ErrorMessage        :
    ActivityName        : CopyFromBlobToSQL
    PipelineName        : ADFTutorialPipeline
    Type                : Copy  
    ```

Pour obtenir une documentation complète sur les applets de commande Data Factory, consultez la [Référence des applets de commande Data Factory](/powershell/module/az.datafactory).

## <a name="summary"></a>Résumé
Dans ce tutoriel, vous avez créé une fabrique de données Azure pour copier des données d’objet blob Azure vers Azure SQL Database. Vous avez utilisé PowerShell pour créer la fabrique de données, les services liés, les jeux de données et un pipeline. Voici les étapes de premier niveau que vous avez effectuées dans ce didacticiel :  

1. Création d’une **fabrique de données** Azure.
1. Création de **services liés**:

   a. Un service lié **Stockage Azure** pour lier votre compte Stockage Azure contenant des données d’entrée.     
   b. Un service lié **Azure SQL** pour lier votre base de données SQL contenant les données de sortie.
1. Création de **jeux de données** qui décrivent les données d’entrée et de sortie des pipelines.
1. Création d’un **pipeline** avec une **activité de copie** avec **BlobSource** en tant que source et **SqlSink** en tant que récepteur.

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez utilisé le stockage Blob Azure comme magasin de données source et Azure SQL Database comme banque de données de destination dans une opération de copie. Le tableau ci-dessous contient la liste des magasins de données pris en charge en tant que sources et destinations par l’activité de copie : 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Pour découvrir comment copier des données vers/depuis un magasin de données, cliquez sur le lien du magasin de données dans le tableau. 

