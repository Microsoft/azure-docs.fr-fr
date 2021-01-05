---
title: 'Tutoriel : Créer un pipeline avec l’activité de copie à l’aide de Visual Studio '
description: Dans ce didacticiel, vous allez créer un pipeline Azure Data Factory avec une activité de copie à l’aide de Visual Studio.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: 1751185b-ce0a-4ab2-a9c3-e37b4d149ca3
ms.service: data-factory
ms.workload: data-services
ms.custom: vs-azure
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: c5a4a6e896d0cee22424274d845ccc24463833cc
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97608763"
---
# <a name="tutorial-create-a-pipeline-with-copy-activity-using-visual-studio"></a>Tutoriel : Créer un pipeline avec l’activité de copie à l’aide de Visual Studio
> [!div class="op_single_selector"]
> * [Vue d’ensemble et composants requis](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Assistant de copie](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Modèle Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)
> 
> 

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez le [tutoriel sur l’activité de copie](../quickstart-create-data-factory-dot-net.md). 

Dans cet article, vous allez apprendre à utiliser Microsoft Visual Studio pour créer une fabrique de données avec un pipeline qui copie les données d’un stockage Blob Azure dans Azure SQL Database. Si vous débutez avec Azure Data Factory, lisez l’article [Présentation d’Azure Data Factory](data-factory-introduction.md) avant de suivre ce didacticiel.   

Dans ce tutoriel, vous créez un pipeline contenant une seule activité : une activité de copie. L’activité de copie copie les données d’un magasin de données pris en charge vers un magasin de données de récepteur pris en charge. Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs, consultez [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Elle est mise en œuvre par un service disponible dans le monde entier, capable de copier des données entre différents magasins de données de façon sécurisée, fiable et évolutive. Pour plus d’informations sur l’activité de copie, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md).

Un pipeline peut contenir plusieurs activités. En outre, vous pouvez chaîner deux activités (une après l’autre) en configurant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. Pour plus d’informations, consultez [Plusieurs activités dans un pipeline](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).

> [!NOTE] 
> Dans ce didacticiel, le pipeline de données copie les données d’un magasin de données source vers un magasin de données de destination. Pour suivre un tutoriel sur la transformation des données à l’aide d’Azure Data Factory, consultez [Tutoriel : Créer un pipeline pour transformer des données à l’aide d’un cluster Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

1. Lisez l’article [Vue d’ensemble du didacticiel](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) et effectuez les **étapes préalables requises** .       
2. Pour créer des instances Data Factory, vous devez avoir un rôle de [collaborateur de fabrique de données](../../role-based-access-control/built-in-roles.md#data-factory-contributor) au niveau de l’abonnement/du groupe de ressources.
3. Les composants suivants doivent être installés sur votre ordinateur : 
   * Visual Studio 2013 ou Visual Studio 2015
   * Téléchargez le Kit de développement logiciel (SDK) Azure pour Visual Studio 2013 ou Visual Studio 2015. Accédez à la [page de téléchargement d’Azure](https://azure.microsoft.com/downloads/), puis cliquez sur **VS 2013** ou **VS 2015** dans la section  **.NET**.
   * Téléchargez le dernier plug-in Azure Data Factory pour Visual Studio : [VS 2013](https://visualstudiogallery.msdn.microsoft.com/754d998c-8f92-4aa7-835b-e89c8c954aa5) ou [VS 2015](https://visualstudiogallery.msdn.microsoft.com/371a4cf9-0093-40fa-b7dd-be3c74f49005). Vous pouvez également mettre à jour le plug-in en effectuant les étapes suivantes : Dans le menu, cliquez sur **Outils** -> **Extensions et mises à jour** -> **En ligne** -> **Galerie Visual Studio** -> **Outils Microsoft Azure Data Factory pour Visual Studio** -> **Mettre à jour**.

## <a name="steps"></a>Étapes

Voici les étapes à effectuer dans le cadre de ce didacticiel :

1. Créez des **services liés** dans la fabrique de données. Dans cette étape, vous créez deux services liés des types suivants : Stockage Azure et Azure SQL Database. 

    AzureStorageLinkedService relie votre compte de stockage Azure à la fabrique de données. Vous avez créé un conteneur et chargé des données dans ce compte de stockage en remplissant les [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).   

    AzureSqlLinkedService lie Azure SQL Database à la fabrique de données. Les données copiées à partir du stockage Blob sont stockées dans cette base de données. Vous avez créé une table SQL dans cette base de données en remplissant les [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).     

2. Créez des **jeux de données** d’entrée et de sortie dans la fabrique de données.  

    Le service lié Stockage Azure spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à votre compte de stockage Azure. Le jeu de données blob d’entrée spécifie quant à lui le conteneur et le dossier qui contient les données d’entrée.  

    De même, le service lié Azure SQL Database spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à Azure SQL Database. Et le jeu de données de la table SQL de sortie spécifie la table de la base de données dans laquelle les données du stockage Blob sont copiées.

3. Créez un **pipeline** dans la fabrique de données. Dans cette étape, vous allez créer un pipeline avec une activité de copie.   

    Cette activité copie les données d’un objet blob du stockage Blob Azure dans une table Azure SQL Database. Vous pouvez utiliser une activité de copie dans un pipeline pour copier les données d’une source prise en charge dans une destination prise en charge. Pour obtenir la liste des magasins de données pris en charge, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md#supported-data-stores-and-formats).

4. Créez une **fabrique de données** Azure lors du déploiement des entités Data Factory (services liés, jeux de données/tables et pipelines). 

## <a name="create-visual-studio-project"></a>Création d’un projet Visual Studio

1. Lancez **Visual Studio 2015**. Cliquez sur **Fichier**, pointez le curseur de la souris sur **Nouveau**, puis cliquez sur **Projet**. La boîte de dialogue **Nouveau projet** doit s’afficher.  

2. Dans la boîte de dialogue **Nouveau projet**, sélectionnez le modèle **DataFactory**, puis cliquez sur **Projet Data Factory vide**.  

    ![Boîte de dialogue Nouveau projet](./media/data-factory-copy-activity-tutorial-using-visual-studio/new-project-dialog.png)

3. Spécifiez le nom du projet, l’emplacement de la solution et le nom de la solution, puis cliquez sur **OK**.

    ![Explorateur de solutions](./media/data-factory-copy-activity-tutorial-using-visual-studio/solution-explorer.png)    

## <a name="create-linked-services"></a>Créez des services liés

Vous allez créer des services liés dans une fabrique de données pour lier vos magasins de données et vos services de calcul à la fabrique de données. Dans ce didacticiel, vous n’allez pas utiliser n’importe quel service de calcul comme Azure HDInsight ou Azure Data Lake Analytics. Vous utilisez deux magasins de données de type Stockage Azure (source) et Azure SQL Database (destination). 

Par conséquent, vous créez deux services liés des types suivants : AzureStorage et AzureSqlDatabase.  

Le service lié Stockage Azure relie votre compte de stockage Azure à la fabrique de données. Ce compte de stockage est celui dans lequel vous avez créé un conteneur et chargé les données en remplissant les [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).   

Le service lié Azure SQL lie Azure SQL Database à la fabrique de données. Les données copiées à partir du stockage Blob sont stockées dans cette base de données. Vous avez créé une table emp dans cette base de données dans le cadre des [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Les services liés se chargent de lier des magasins de données ou des services de calcul à une fabrique de données Azure. Pour connaître l’ensemble des sources et des récepteurs pris en charge par l’activité de copie, consultez [Banques de données et formats pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) . Pour obtenir la liste des services de calcul pris en charge par Data Factory, consultez [Services liés de calcul](data-factory-compute-linked-services.md) . Dans ce didacticiel, aucun service de calcul n’est utilisé. 

### <a name="create-the-azure-storage-linked-service"></a>Créer le service lié Azure Storage

1. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur **Services liés**, pointez sur **Ajouter** puis cliquez sur **Nouvel élément**.      

2. Dans la boîte de dialogue **Ajouter un nouvel élément**, sélectionnez **Service lié Azure Storage** dans la liste, puis cliquez sur **Ajouter**. 

    ![Nouveau service lié](./media/data-factory-copy-activity-tutorial-using-visual-studio/new-linked-service-dialog.png)

3. Remplacez `<accountname>` et `<accountkey>`* par le nom de votre compte de stockage Azure et par sa clé. 

    ![Service lié Stockage Azure](./media/data-factory-copy-activity-tutorial-using-visual-studio/azure-storage-linked-service.png)

4. Enregistrez le fichier **AzureStorageLinkedService1.json** .

    Pour plus d’informations sur les propriétés JSON dans la définition de service lié, consultez l’article [Connecteur de stockage Blob Azure](data-factory-azure-blob-connector.md#linked-service-properties).

### <a name="create-the-azure-sql-linked-service"></a>Créer le service lié SQL Azure

1. Cliquez de nouveau avec le bouton droit sur le nœud **Services liés** dans l’**Explorateur de solutions**, pointez sur **Ajouter**, puis cliquez sur **Nouvel élément**. 
2. Cette fois, sélectionnez **Service lié SQL Azure**, puis cliquez sur **Ajouter**. 
3. Dans le **fichier AzureSqlLinkedService1.json**, remplacez `<servername>`, `<databasename>`, `<username@servername>` et `<password>` par le nom du compte d’utilisateur, de la base de données et de votre serveur, et par le mot de passe associé.    
4. Enregistrez le fichier **AzureSqlLinkedService1.json** . 

    Pour plus d’informations sur ces propriétés JSON, consultez [Connecteur Azure SQL Database](data-factory-azure-sql-connector.md#linked-service-properties).

## <a name="create-datasets"></a>Créez les jeux de données

Dans l’étape précédente, vous avez créé des services pour lier votre compte de stockage Azure et Azure SQL Database à votre fabrique de données. Dans cette étape, vous définissez deux jeux de données nommés InputDataset et OutputDataset, qui représentent les données d’entrée/sortie stockées dans les banques de données référencées par AzureStorageLinkedService1 et AzureSqlLinkedService1, respectivement.

Le service lié Stockage Azure spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à votre compte de stockage Azure. Le jeu de données blob d’entrée (InputDataset) spécifie quant à lui le conteneur et le dossier qui contient les données d’entrée.  

De même, le service lié Azure SQL Database spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à Azure SQL Database. Et le jeu de données de la table SQL de sortie (OututDataset) spécifie la table de la base de données dans laquelle les données du stockage Blob sont copiées. 

### <a name="create-input-dataset"></a>Créer le jeu de données d’entrée

Dans cette étape, vous créez un jeu de données nommé InputDataset qui pointe vers un fichier blob (emp.txt) dans le dossier racine d’un conteneur d’objets blob (adftutorial) du stockage Azure représenté par le service lié AzureStorageLinkedService1. Si vous ne spécifiez pas de valeur pour fileName (ou si vous ignorez ce paramètre), les données de tous les objets blob du dossier d’entrée sont copiées dans la destination. Dans ce didacticiel, vous spécifiez une valeur pour fileName. 

Ici, vous utilisez le terme « tables » plutôt que « jeux de données ». Une table est un jeu de données rectangulaire. C’est le seul type de jeu de données pris en charge pour l’instant. 

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur **Tables**, pointez sur **Ajouter**, puis cliquez sur **Nouvel élément**.
2. Dans la boîte de dialogue **Ajouter un nouvel élément**, sélectionnez **Objet blob Azure**, puis cliquez sur **Ajouter**.   
3. Remplacez le texte JSON par le texte suivant, puis enregistrez le fichier **AzureBlobLocation1.json** . 

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
      "linkedServiceName": "AzureStorageLinkedService1",
      "typeProperties": {
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
    | type | La propriété type est définie sur **AzureBlob**, car les données se trouvent dans le stockage d’objets blob Azure. |
    | linkedServiceName | Fait référence au service **AzureStorageLinkedService** que vous avez créé précédemment. |
    | folderPath | Spécifie le **conteneur** d’objets blob et le **dossier** contenant les objets blob d’entrée. Dans ce tutoriel, adftutorial est le conteneur d’objets blob et folder le dossier racine. | 
    | fileName | Cette propriété est facultative. Si vous omettez cette propriété, tous les fichiers spécifiés dans folderPath sont récupérés. Dans ce tutoriel, **emp.txt** est spécifié pour fileName, si bien que seul ce fichier est récupéré pour le traitement. |
    | format -> type |Le fichier d’entrée étant au format texte, nous utilisons **TextFormat**. |
    | columnDelimiter | Les colonnes du fichier d’entrée sont délimitées par une **virgule (`,`)** . |
    | frequency/interval | La fréquence est définie sur **Heure** et l’intervalle est **1**, ce qui signifie que les segments d’entrée sont disponibles **toutes les heures**. En d’autres termes, le service Data Factory recherche les données d’entrée toutes les heures dans le dossier racine du conteneur d’objets blob (**adftutorial**) que vous avez spécifié. Il recherche des données entre les heures de début et de fin du pipeline, pas avant ni après.  |
    | external | Cette propriété a la valeur **true** si les données ne sont pas générées par ce pipeline. Dans ce tutoriel, les données d’entrée sont dans le fichier emp.txt, qui n’est pas généré par ce pipeline. Nous définissons donc cette propriété sur true. |

    Pour plus d’informations sur ces propriétés JSON, voir [Connecteur de stockage Blob Azure](data-factory-azure-blob-connector.md#dataset-properties).   

### <a name="create-output-dataset"></a>Créer un jeu de données de sortie

Dans cette étape, vous créez un jeu de données de sortie nommé **OutputDataset**. Ce jeu de données pointe vers une table SQL dans Azure SQL Database représentée par **AzureSqlLinkedService1**. 

1. Dans l’**Explorateur de solutions**, cliquez de nouveau avec le bouton droit sur **Tables**, pointez sur **Ajouter**, puis cliquez sur **Nouvel élément**.
2. Dans la boîte de dialogue **Ajouter un nouvel élément**, sélectionnez **Azure SQL**, puis cliquez sur **Ajouter**. 
3. Remplacez le texte JSON par le texte JSON suivant, puis enregistrez le fichier **AzureSqlTableLocation1.json** .

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
            "linkedServiceName": "AzureSqlLinkedService1",
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
    | type | La propriété du type est définie sur **AzureSqlTable** parce que les données sont copiées dans une table dans Azure SQL Database. |
    | linkedServiceName | Fait référence au service **AzureSqlLinkedService** que vous avez créé précédemment. |
    | tableName | Spécifie la **table** dans laquelle les données sont copiées. | 
    | frequency/interval | La fréquence est définie sur **Heure** et l’intervalle sur **1**, ce qui signifie que les tranches de sortie sont produites **toutes les heures** entre les heures de début et de fin du pipeline, pas avant ni après.  |

    La table emp de la base de données contient trois colonnes : **ID**, **FirstName** et **LastName**. ID étant une colonne d’identité, il vous suffit de spécifier **FirstName** et **LastName**.

    Pour plus d’informations sur ces propriétés JSON, consultez l’article [Connecteur SQL Azure](data-factory-azure-sql-connector.md#dataset-properties).

## <a name="create-pipeline"></a>Création d’un pipeline

Dans cette étape, vous créez un pipeline avec une **activité de copie** qui utilise **InputDataset** en entrée et **OutputDataset** en sortie.

Le jeu de données de sortie pilote actuellement la planification. Dans ce didacticiel, le jeu de données de sortie est configuré pour produire une tranche par heure. Les heures de début et de fin du pipeline sont distantes d’une journée, soit 24 heures. Par conséquent, 24 tranches de jeu de données de sortie sont générées par le pipeline. 

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur **Pipelines**, pointez sur **Ajouter**, puis cliquez sur **Nouvel élément**.  
2. Sélectionnez **Pipeline de copie de données** dans la boîte de dialogue **Ajouter un nouvel élément**, puis cliquez sur **Ajouter**. 
3. Remplacez le texte JSON par le texte JSON suivant, puis enregistrez le fichier **CopyActivity1.json** .

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
           "style": "StartOfInterval",
           "retry": 0,
           "timeout": "01:00:00"
         }
       }
     ],
     "start": "2017-05-11T00:00:00Z",
     "end": "2017-05-12T00:00:00Z",
     "isPaused": false
   }
  }
    ```   
   - Dans la section des activités, il existe une seule activité dont le **type** a la valeur **Copy**. Pour plus d’informations sur l’activité de copie, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md). Dans les solutions Data Factory, vous pouvez également utiliser [Activités de transformation des données](data-factory-data-transformation-activities.md).
   - L’entrée de l’activité est définie sur **InputDataset** et sa sortie, sur **OutputDataset**. 
   - Dans la section **typeProperties**, **BlobSource** est spécifié en tant que type de source et **SqlSink**, en tant que type de récepteur. Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs pour l’activité de copie, consultez [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Pour apprendre à utiliser un magasin de données pris en charge spécifique en tant que source/récepteur, cliquez sur le lien dans le tableau.  
     
     Remplacez la valeur de la propriété **start** par le jour actuel et la valeur **end**, par le jour suivant. Si vous le souhaitez, spécifiez uniquement la date et ignorez l'heure. Par exemple, « 2016-02-03 », qui équivaut à « 2016-02-03T00:00:00Z ».
     
     Les dates/heures de début et de fin doivent toutes deux être au [format ISO](https://en.wikipedia.org/wiki/ISO_8601). Par exemple : 2016-10-14T16:32:41Z. L’heure de fin ( **end** ) est facultative, mais nous allons l’utiliser dans ce didacticiel. 
     
     Si vous ne spécifiez aucune valeur pour la propriété **end**, cette dernière est calculée comme suit : « **start + 48 heures** ». Pour exécuter le pipeline indéfiniment, spécifiez **9999-09-09** comme valeur pour la propriété **end**.
     
     Dans l’exemple ci-dessus, il existe 24 tranches de données, car une tranche de données est générée toutes les heures.

     Pour obtenir une description des propriétés JSON dans une définition de pipeline, consultez l’article [Créer des pipelines](data-factory-create-pipelines.md). Pour obtenir une description des propriétés JSON dans une définition d’activité de copie, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md). Pour obtenir une description des propriétés JSON prises en charge par BlobSource, consultez l’article [Connecteur de stockage Blob Azure](data-factory-azure-blob-connector.md). Pour obtenir une description des propriétés JSON prises en charge par SqlSink, consultez l’article [Connecteur Azure SQL Database](data-factory-azure-sql-connector.md).

## <a name="publishdeploy-data-factory-entities"></a>Publier/Déployer des entités Data Factory
Dans cette étape, vous publiez les entités Data Factory (services liés, jeux de données et pipeline) que vous avez créées précédemment. Vous spécifiez également le nom de la fabrique de données à créer pour contenir ces entités.  

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet, puis cliquez sur **Publier**. 
2. Si la boîte de dialogue **Connectez-vous à votre compte Microsoft** s’affiche, saisissez vos informations d’identification pour le compte associé à l’abonnement Azure, puis cliquez sur **Se connecter**.
3. La boîte de dialogue suivante doit s’afficher :
   
   ![Boîte de dialogue Publier](./media/data-factory-copy-activity-tutorial-using-visual-studio/publish.png)
4. Dans la page Configurer une fabrique de données, procédez comme suit : 
   
   1. Sélectionnez l'option **Créer une fabrique de données** .
   2. Saisissez **VSTutorialFactory** dans le champ **Nom**.  
      
      > [!IMPORTANT]
      > Le nom de la fabrique de données Azure doit être un nom global unique. Si vous obtenez une erreur concernant le nom de la fabrique de données pendant la publication, modifiez ce nom (par exemple, votrenomVSTutorialFactory) et relancez la publication. Consultez la rubrique [Data Factory - Règles d’affectation des noms](data-factory-naming-rules.md) pour savoir comment nommer les artefacts Data Factory.        
      > 
      > 
   3. Sélectionnez votre abonnement Azure pour le champ **Abonnement** .
      
      > [!IMPORTANT]
      > Si vous ne voyez pas les abonnements, vérifiez que vous êtes connecté à l’aide d’un compte administrateur ou coadministrateur de l’abonnement.  
      > 
      > 
   4. Sélectionnez le **groupe de ressources** pour la fabrique de données à créer. 
   5. Sélectionnez la **région** pour la fabrique de données. Seules les régions prises en charge par le service Data Factory sont affichées dans la liste déroulante.
   6. Cliquez sur **Suivant** pour basculer vers la page **Publier des éléments**.
      
       ![Page Configurer une fabrique de données](media/data-factory-copy-activity-tutorial-using-visual-studio/configure-data-factory-page.png)   
5. Dans la page **Publier des éléments**, vérifiez que toutes les entités de fabriques de données sont sélectionnées, puis cliquez sur **Suivant** pour basculer vers la page **Résumé**.
   
   ![Page Publier des éléments](media/data-factory-copy-activity-tutorial-using-visual-studio/publish-items-page.png)     
6. Passez en revue le résumé, puis cliquez sur **Suivant** pour démarrer le processus de déploiement et afficher l’**état du déploiement**.
   
   ![Page Résumé de la publication](media/data-factory-copy-activity-tutorial-using-visual-studio/publish-summary-page.png)
7. Dans la page **État du déploiement** , vous devez voir l’état du processus de déploiement. Une fois le déploiement terminé, cliquez sur Terminer.
 
   ![Page État du déploiement](media/data-factory-copy-activity-tutorial-using-visual-studio/deployment-status.png)

Notez les points suivants : 

* Si vous recevez le message d’erreur : « L’abonnement n’est pas inscrit pour utiliser l’espace de noms Microsoft.DataFactory », effectuez l’une des opérations suivantes et essayez de relancer la publication : 
  
  * Dans Azure PowerShell, exécutez la commande suivante pour enregistrer le fournisseur Data Factory. 

  ```powershell    
  Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```
  Vous pouvez exécuter la commande suivante pour confirmer que le fournisseur Data Factory est bien enregistré. 
    
  ```powershell
  Get-AzResourceProvider
    ```
  * Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide de l’abonnement Azure et accédez à un panneau Data Factory (ou) créez une fabrique de données dans le portail Azure. Cette action enregistre automatiquement le fournisseur.
* Le nom de la fabrique de données pourra être enregistré en tant que nom DNS et devenir ainsi visible publiquement.

> [!IMPORTANT]
> Pour créer des instances Data Factory, vous devez être administrateur ou co-administrateur de l’abonnement Azure.

## <a name="monitor-pipeline"></a>Surveillance d’un pipeline
Accédez à la page d’accueil de votre fabrique de données :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Cliquez sur **Plus de services** sur le menu de gauche, puis sur **Fabriques de données**.

  ![Parcourir les fabriques de données](media/data-factory-copy-activity-tutorial-using-visual-studio/browse-data-factories.png)
3. Commencez à taper le nom de votre fabrique de données.

  ![Nom de la fabrique de données](media/data-factory-copy-activity-tutorial-using-visual-studio/enter-data-factory-name.png) 
4. Cliquez sur votre fabrique de données dans la liste des résultats pour afficher la page d’accueil de votre fabrique de données.

  ![Page d’accueil Data Factory](media/data-factory-copy-activity-tutorial-using-visual-studio/data-factory-home-page.png)
5. Suivez les instructions dans [Surveiller les jeux de données et le pipeline](data-factory-monitor-manage-pipelines.md) pour surveiller le pipeline et les jeux de données que vous avez créés dans ce didacticiel. Pour le moment, Visual Studio ne prend pas en charge la surveillance des pipelines Data Factory. 

## <a name="summary"></a>Résumé
Dans ce tutoriel, vous avez créé une fabrique de données Azure pour copier des données d’objet blob Azure vers Azure SQL Database. Vous avez utilisé Visual Studio pour créer la fabrique de données, les services liés, les jeux de données et un pipeline. Voici les étapes de premier niveau que vous avez effectuées dans ce didacticiel :  

1. Création d’une **fabrique de données** Azure.
2. Création de **services liés**:
   1. Un service lié **Stockage Azure** pour lier votre compte Stockage Azure contenant des données d’entrée.     
   2. Un service lié **Azure SQL** pour lier votre base de données contenant les données de sortie. 
3. Création des **jeux de données** qui décrivent les données d’entrée et de sortie des pipelines.
4. Création d’un **pipeline** avec une **activité de copie** avec **BlobSource** en tant que source et **SqlSink** en tant que récepteur. 

Pour voir comment utiliser une activité Hive HDInsight visant à transformer des données à l’aide d’un cluster Azure HDInsight, consultez [Tutoriel : Générer votre premier pipeline pour transformer les données à l’aide du cluster Hadoop](data-factory-build-your-first-pipeline.md).

Vous pouvez chaîner deux activités (une après l’autre) en configurant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. Pour plus d’informations, voir [Planification et exécution dans Data Factory](data-factory-scheduling-and-execution.md). 

## <a name="view-all-data-factories-in-server-explorer"></a>Afficher toutes les fabriques de données dans l’Explorateur de serveurs
Cette section explique comment utiliser l’Explorateur de serveurs dans Visual Studio pour afficher toutes les fabriques de données dans votre abonnement Azure et créer un projet Visual Studio basé sur une fabrique de données existante. 

1. Dans **Visual Studio**, cliquez sur **Affichage** dans le menu, puis sur **Explorateur de serveurs**.
2. Dans la fenêtre Explorateur de serveurs, développez **Azure**, puis **Data Factory**. Si la boîte de dialogue **Connectez-vous à Visual Studio** s’affiche, saisissez le **compte** associé à votre abonnement Azure, puis cliquez sur **Continuer**. Saisissez le **mot de passe**, puis cliquez sur **Se connecter**. Visual Studio essaie d’obtenir des informations sur toutes les fabriques de données Azure contenues dans votre abonnement. L’état de cette opération s’affiche dans la fenêtre **Liste des tâches de Data Factory** .

    ![Explorateur de serveurs](./media/data-factory-copy-activity-tutorial-using-visual-studio/server-explorer.png)

## <a name="create-a-visual-studio-project-for-an-existing-data-factory"></a>Créer un projet Visual Studio pour une fabrique de données existante

- Cliquez avec le bouton droit sur une fabrique de données dans l’Explorateur de serveurs, puis sélectionnez **Exporter la fabrique de données vers le nouveau projet** pour créer un projet Visual Studio basé sur une fabrique de données existante.

  ![Exporter la fabrique de données vers un projet Visual Studio](./media/data-factory-copy-activity-tutorial-using-visual-studio/export-data-factory-menu.png)  

## <a name="update-data-factory-tools-for-visual-studio"></a>Mettre à jour des outils Data Factory pour Visual Studio
Pour mettre à jour des outils Azure Data Factory pour Visual Studio, procédez comme suit :

1. Dans le menu, cliquez sur **Outils**, puis sélectionnez **Extensions et mises à jour**. 
2. Dans le volet de gauche, sélectionnez **Mises à jour**, puis **Galerie Visual Studio**.
3. Sélectionnez **Outils Azure Data Factory pour Visual Studio**, puis cliquez sur **Mettre à jour**. Si cette entrée n’est pas affichée, c’est que vous possédez déjà la dernière version de ces outils. 

## <a name="use-configuration-files"></a>Utiliser des fichiers de configuration
Vous pouvez utiliser des fichiers de configuration dans Visual Studio pour configurer les propriétés des services/tableaux/pipelines liés différemment pour chaque environnement.

Examinez la définition JSON suivante pour un service lié Azure Storage. Spécifiez **connectionString** avec différentes valeurs pour accountname et accountkey, en fonction de l’environnement (dév./test/production) sur lequel vous déployez des entités Data Factory. Vous pouvez parvenir à ce comportement en utilisant un fichier de configuration distinct pour chaque environnement.

```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "description": "",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

### <a name="add-a-configuration-file"></a>Ajouter un fichier de configuration
Ajoutez un fichier de configuration pour chaque environnement en effectuant les opérations suivantes :   

1. Cliquez avec le bouton droit de la souris sur le projet Data Factory dans votre solution Visual Studio, pointez sur **Ajouter**, puis cliquez sur **Nouvel élément**.
2. Sélectionnez **Config** dans la liste des modèles installés sur la gauche, choisissez **Fichier de configuration**, entrez un **nom** pour ce fichier, puis cliquez sur **Ajouter**.

    ![Ajouter un fichier de configuration](./media/data-factory-build-your-first-pipeline-using-vs/add-config-file.png)
3. Ajoutez les paramètres de configuration et leurs valeurs au format suivant :

  ```json
  {
      "$schema": "http://datafactories.schema.management.azure.com/vsschemas/V1/Microsoft.DataFactory.Config.json",
      "AzureStorageLinkedService1": [
          {
              "name": "$.properties.typeProperties.connectionString",
              "value": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
          }
      ],
      "AzureSqlLinkedService1": [
          {
              "name": "$.properties.typeProperties.connectionString",
              "value":  "Server=tcp:<logical SQL server name>.database.windows.net,1433;Database=<Azure SQL datbase>;User ID=<Username>;Password=<Password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
          }
      ]
  }
    ```

    Cet exemple configure la propriété connectionString d’un service lié Azure Storage et d’un service lié SQL Azure. Notez que la syntaxe de spécification du nom est [JsonPath](https://goessner.net/articles/JsonPath/).   

    Si JSON est doté d’une propriété ayant un tableau de valeurs comme indiqué dans le code suivant :  

  ```json
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
    ```

    Configurez les propriétés comme indiqué dans le fichier de configuration suivant (utilisez indexation de base zéro) :

  ```json
    {
        "name": "$.properties.structure[0].name",
        "value": "FirstName"
    }
    {
        "name": "$.properties.structure[0].type",
        "value": "String"
    }
    {
        "name": "$.properties.structure[1].name",
        "value": "LastName"
    }
    {
        "name": "$.properties.structure[1].type",
        "value": "String"
    }
    ```

### <a name="property-names-with-spaces"></a>Noms de propriétés avec des espaces
Si un nom de propriété comporte des espaces, utilisez des crochets comme indiqué dans l’exemple suivant (nom de serveur de base de données) :

```json
 {
     "name": "$.properties.activities[1].typeProperties.webServiceParameters.['Database server name']",
     "value": "MyAsqlServer.database.windows.net"
 }
```

### <a name="deploy-solution-using-a-configuration"></a>Déployer une solution à l’aide d’une configuration
Lorsque vous publiez des entités Azure Data Factory dans Visual Studio, vous pouvez spécifier la configuration que vous souhaitez utiliser pour cette opération de publication.

Pour publier des entités dans un projet Azure Data Factory à l’aide d’un fichier de configuration :   

1. Cliquez avec le bouton droit sur le projet Data Factory, puis cliquez sur **Publier** pour afficher la boîte de dialogue **Publier des éléments**.
2. Dans la page **Configurer une fabrique de données**, sélectionnez une fabrique de données existante ou spécifiez les valeurs pour en créer une, puis cliquez sur **Suivant**.   
3. La page **Publier des éléments** contient une liste déroulante avec les configurations disponibles pour le champ **Sélectionner une configuration de déploiement**.

    ![Sélectionner un fichier de config](./media/data-factory-build-your-first-pipeline-using-vs/select-config-file.png)
4. Sélectionnez le **fichier de configuration** que vous souhaitez utiliser, puis cliquez sur **Suivant**.
5. Vérifiez que vous voyez bien le nom du fichier JSON dans la page **Résumé**, puis cliquez sur **Suivant**.
6. Une fois l’opération de déploiement terminée, cliquez sur **Terminer** .

Au cours du déploiement, les valeurs du fichier de configuration sont utilisées pour définir celles des propriétés des fichiers JSON avant que les entités ne soient déployées sur le service Azure Data Factory.   

## <a name="use-azure-key-vault"></a>Utiliser Azure Key Vault
Il n’est pas recommandé et souvent déconseillé vis-à-vis de la stratégie de sécurité pour valider des données sensibles telles que des chaînes de connexion au référentiel de code. Consultez l’exemple [ADF Secure Publish](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/ADFSecurePublish) sur GitHub pour en savoir plus sur le stockage d’informations sensibles dans Azure Key Vault et son utilisation lors de la publication des entités Data Factory. L’extension Secure Publish pour Visual Studio permet de stocker les secrets dans Key Vault, et seules les références à ceux-ci sont spécifiés dans des services / configurations de déploiement liés. Ces références sont résolues lorsque vous publiez des entités Data Factory dans Azure. Ces fichiers peuvent ensuite être validés sur le référentiel source sans exposer les secrets.


## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez utilisé le stockage Blob Azure comme magasin de données source et Azure SQL Database comme banque de données de destination dans une opération de copie. Le tableau ci-dessous contient la liste des magasins de données pris en charge en tant que sources et destinations par l’activité de copie : 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Pour découvrir comment copier des données vers/depuis un magasin de données, cliquez sur le lien du magasin de données dans le tableau.
