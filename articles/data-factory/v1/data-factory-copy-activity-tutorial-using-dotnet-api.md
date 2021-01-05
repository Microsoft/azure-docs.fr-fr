---
title: 'Tutoriel : créer un pipeline avec l’activité de copie à l’aide de l’API .NET '
description: Dans ce didacticiel, vous allez créer un pipeline Azure Data Factory avec une activité de copie à l’aide de l’API .NET.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: 58fc4007-b46d-4c8e-a279-cb9e479b3e2b
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: d047a45d678918541eb3c2d2c45e4519a34bdd57
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97608678"
---
# <a name="tutorial-create-a-pipeline-with-copy-activity-using-net-api"></a>Tutoriel : créer un pipeline avec l’activité de copie à l’aide de l’API .NET
> [!div class="op_single_selector"]
> * [Vue d’ensemble et composants requis](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Assistant de copie](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Modèle Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez le [tutoriel sur l’activité de copie](../quickstart-create-data-factory-dot-net.md). 

Dans cet article, vous allez apprendre à utiliser l’[API .NET](https://portal.azure.com) pour créer une fabrique de données avec un pipeline qui copie les données d’un stockage Blob Azure dans Azure SQL Database. Si vous débutez avec Azure Data Factory, lisez l’article [Présentation d’Azure Data Factory](data-factory-introduction.md) avant de suivre ce didacticiel.   

Dans ce tutoriel, vous créez un pipeline contenant une seule activité : une activité de copie. L’activité de copie copie les données d’un magasin de données pris en charge vers un magasin de données de récepteur pris en charge. Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs, consultez [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Elle est mise en œuvre par un service disponible dans le monde entier, capable de copier des données entre différents magasins de données de façon sécurisée, fiable et évolutive. Pour plus d’informations sur l’activité de copie, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md).

Un pipeline peut contenir plusieurs activités. En outre, vous pouvez chaîner deux activités (une après l’autre) en configurant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. Pour plus d’informations, consultez [Plusieurs activités dans un pipeline](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline). 

> [!NOTE] 
> Pour la documentation complète sur l’API .NET pour Data Factory, consultez [Informations de référence sur l’API .NET Data Factory](/dotnet/api/index?view=azuremgmtdatafactories-4.12.1).
> 
> Dans ce didacticiel, le pipeline de données copie les données d’un magasin de données source vers un magasin de données de destination. Pour suivre un tutoriel sur la transformation des données à l’aide d’Azure Data Factory, consultez [Tutoriel : Créer un pipeline pour transformer des données à l’aide d’un cluster Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Consultez [Vue d’ensemble et étapes préalables requises](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) pour obtenir une vue d’ensemble du didacticiel et effectuer les **étapes préalables requises** .
* Visual Studio 2012, 2013 ou 2015
* Téléchargez et installez le [Kit de développement logiciel (SDK) Azure .NET](https://azure.microsoft.com/downloads/)
* Azure PowerShell. Suivez les instructions de l’article [Installation et configuration d’Azure PowerShell](/powershell/azure/install-Az-ps) pour installer Azure PowerShell sur votre ordinateur. Vous utilisez Azure PowerShell pour créer une application Azure Active Directory.

### <a name="create-an-application-in-azure-active-directory"></a>Créer une application dans Azure Active Directory
Créez une application Azure Active Directory, créez un principal de service pour l’application et attribuez-lui le rôle **Contributeurs de Data Factory** .

1. Lancez **PowerShell**.
2. Exécutez la commande suivante, puis saisissez le nom d’utilisateur et le mot de passe que vous avez utilisés pour la connexion au portail Azure.

    ```powershell
    Connect-AzAccount
    ```
3. Exécutez la commande suivante pour afficher tous les abonnements de ce compte.

    ```powershell
    Get-AzSubscription
    ```
4. Exécutez la commande suivante pour sélectionner l’abonnement que vous souhaitez utiliser. Remplacez **&lt;NameOfAzureSubscription**&gt; par le nom de votre abonnement Azure.

    ```powershell
    Get-AzSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzContext
    ```

   > [!IMPORTANT]
   > Notez les éléments **SubscriptionId** et **TenantId** dans la sortie de cette commande.

5. Créez un groupe de ressources Azure nommé **ADFTutorialResourceGroup** en exécutant la commande suivante dans PowerShell.

    ```powershell
    New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```

    Si le groupe de ressources existe, indiquez s’il faut le mettre à jour (Y) ou le conserver tel quel (N).

    Si vous utilisez un autre groupe de ressources, vous devez remplacer ADFTutorialResourceGroup par le nom de votre groupe de ressources dans ce didacticiel.
6. Créez une application Azure Active Directory.

    ```powershell
    $azureAdApplication = New-AzADApplication -DisplayName "ADFCopyTutotiralApp" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.adfcopytutorialapp.org/example" -Password "Pass@word1"
    ```

    Si vous obtenez l’erreur suivante, spécifiez une autre URL et relancez la commande.
    
    ```powershell
    Another object with the same value for property identifierUris already exists.
    ```
7. Créez le principal du service AD.

    ```powershell
    New-AzADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId
    ```
8. Ajoutez le principal du service au rôle **Contributeurs de Data Factory** .

    ```powershell
    New-AzRoleAssignment -RoleDefinitionName "Data Factory Contributor" -ServicePrincipalName $azureAdApplication.ApplicationId.Guid
    ```
9. Récupérez l’ID de l’application.

    ```powershell
    $azureAdApplication
    ```
    Notez l’ID d’application (applicationID) dans la sortie.

Vous devez avoir les quatre valeurs suivantes après ces étapes :

* ID client
* Identifiant d’abonnement
* ID de l'application
* Mot de passe (spécifié dans la première commande)

## <a name="walkthrough"></a>Procédure pas à pas
1. À l’aide de Visual Studio 2012/2013/2015, créez une application console C# .NET.
   1. Lancez **Visual Studio** 2012/2013/2015.
   2. Cliquez sur **Fichier**, pointez le curseur de la souris sur **Nouveau**, puis cliquez sur **Projet**.
   3. Développez **Modèles**, puis sélectionnez **Visual C#** . Dans cette procédure pas à pas, vous utilisez C#, mais vous pouvez utiliser un autre langage .NET.
   4. Sélectionnez **Application console** dans la liste des types de projet située sur la droite.
   5. Entrez **DataFactoryAPITestApp** dans le champ Nom.
   6. Sélectionnez **C:\ADFGetStarted** dans le champ Emplacement.
   7. Cliquez sur **OK** pour créer le projet.
2. Cliquez sur **Outils**, pointez le curseur de la souris sur **Gestionnaire de package NuGet**, puis cliquez sur **Console du gestionnaire de package**.
3. Dans la **Console du Gestionnaire de package**, effectuez les étapes suivantes :
   1. Exécutez la commande suivante pour installer le package Data Factory : `Install-Package Microsoft.Azure.Management.DataFactories`
   2. Exécutez la commande suivante pour installer le package Azure Active Directory (vous utilisez l’API Active Directory dans le code) : `Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.19.208020213`
4. Ajoutez la section **appSettings** suivante au fichier **App.config**. Ces paramètres sont utilisés par la méthode d’assistance : **GetAuthorizationHeader**.

    Remplacez les valeurs de **&lt;ID d’Application&gt;** , **&lt;Mot de passe&gt;** , **&lt;ID d’abonnement&gt;** , et **&lt;ID client&gt;** par vos propres valeurs.

    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
        <appSettings>
            <add key="ActiveDirectoryEndpoint" value="https://login.microsoftonline.com/" />
            <add key="ResourceManagerEndpoint" value="https://management.azure.com/" />
            <add key="WindowsManagementUri" value="https://management.core.windows.net/" />

            <add key="ApplicationId" value="your application ID" />
            <add key="Password" value="Password you used while creating the AAD application" />
            <add key="SubscriptionId" value= "Subscription ID" />
            <add key="ActiveDirectoryTenantId" value="Tenant ID" />
        </appSettings>
    </configuration>
    ```

5. Ajoutez les instructions **using** suivantes au fichier source (Program.cs) dans le projet.

    ```csharp
    using System.Configuration;
    using System.Collections.ObjectModel;
    using System.Threading;
    using System.Threading.Tasks;

    using Microsoft.Azure;
    using Microsoft.Azure.Management.DataFactories;
    using Microsoft.Azure.Management.DataFactories.Models;
    using Microsoft.Azure.Management.DataFactories.Common.Models;

    using Microsoft.IdentityModel.Clients.ActiveDirectory;

    ```

6. Ajoutez à la méthode **Main** le code suivant, qui crée une instance de la classe **DataPipelineManagementClient**. Cet objet vous permet de créer une fabrique de données, un service lié, des jeux de données d’entrée et de sortie, ainsi qu’un pipeline. Il vous permet également d’analyser les tranches d’un jeu de données lors de l’exécution.

    ```csharp
    // create data factory management client
    string resourceGroupName = "ADFTutorialResourceGroup";
    string dataFactoryName = "APITutorialFactory";

    TokenCloudCredentials aadTokenCredentials = new TokenCloudCredentials(
            ConfigurationManager.AppSettings["SubscriptionId"],
            GetAuthorizationHeader().Result);

    Uri resourceManagerUri = new Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

    DataFactoryManagementClient client = new DataFactoryManagementClient(aadTokenCredentials, resourceManagerUri);
    ```

   > [!IMPORTANT]
   > Remplacez la valeur de **resourceGroupName** par le nom de votre groupe de ressources Azure.
   >
   > Mettez à jour le nom de la fabrique de données (dataFactoryName) pour le rendre unique. Le nom de la fabrique de données doit être un nom global unique. Consultez la rubrique [Data Factory - Règles d’affectation des noms](data-factory-naming-rules.md) pour savoir comment nommer les artefacts Data Factory.

7. Ajoutez à la méthode **Main** le code suivant, qui crée une **fabrique de données**.

    ```csharp
    // create a data factory
    Console.WriteLine("Creating a data factory");
    client.DataFactories.CreateOrUpdate(resourceGroupName,
        new DataFactoryCreateOrUpdateParameters()
        {
            DataFactory = new DataFactory()
            {
                Name = dataFactoryName,
                Location = "westus",
                Properties = new DataFactoryProperties()
            }
        }
    );
    ```

    Une fabrique de données peut avoir un ou plusieurs pipelines. Un pipeline peut contenir une ou plusieurs activités. Par exemple, une activité de copie pour copier des données d’une source vers un magasin de données de destination, et une activité Hive HDInsight pour exécuter un script Hive pour transformer des données d’entrée et produire des données de sortie. Commençons par la création de la fabrique de données dans cette étape.
8. Ajoutez le code suivant pour créer un **service lié Azure Storage** dans la méthode **Main**.

   > [!IMPORTANT]
   > Remplacez **storageaccountname** et **accountkey** par le nom et la clé de votre compte Azure Storage.

    ```csharp
    // create a linked service for input data store: Azure Storage
    Console.WriteLine("Creating Azure Storage linked service");
    client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
        new LinkedServiceCreateOrUpdateParameters()
        {
            LinkedService = new LinkedService()
            {
                Name = "AzureStorageLinkedService",
                Properties = new LinkedServiceProperties
                (
                    new AzureStorageLinkedService("DefaultEndpointsProtocol=https;AccountName=<storageaccountname>;AccountKey=<accountkey>")
                )
            }
        }
    );
    ```

    Vous allez créer des services liés dans une fabrique de données pour lier vos magasins de données et vos services de calcul à la fabrique de données. Dans ce didacticiel, vous n’allez pas utiliser n’importe quel service de calcul comme Azure HDInsight ou Azure Data Lake Analytics. Vous utilisez deux magasins de données de type Stockage Azure (source) et Azure SQL Database (destination). 

    Vous créez ainsi deux services liés nommés AzureStorageLinkedService et AzureSqlLinkedService de types : AzureStorage et AzureSqlDatabase.  

    AzureStorageLinkedService relie votre compte de stockage Azure à la fabrique de données. Ce compte de stockage est celui dans lequel vous avez créé un conteneur et chargé les données en remplissant les [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
9. Ajoutez le code suivant pour créer un **service lié Azure SQL** dans la méthode **Main**.

   > [!IMPORTANT]
   > Remplacez **servername**, **databasename**, **username** et **password** par le nom de votre serveur, le nom de la base de données, le nom de l’utilisateur et le mot de passe.

    ```csharp
    // create a linked service for output data store: Azure SQL Database
    Console.WriteLine("Creating Azure SQL Database linked service");
    client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
        new LinkedServiceCreateOrUpdateParameters()
        {
            LinkedService = new LinkedService()
            {
                Name = "AzureSqlLinkedService",
                Properties = new LinkedServiceProperties
                (
                    new AzureSqlDatabaseLinkedService("Data Source=tcp:<servername>.database.windows.net,1433;Initial Catalog=<databasename>;User ID=<username>;Password=<password>;Integrated Security=False;Encrypt=True;Connect Timeout=30")
                )
            }
        }
    );
    ```

    AzureSqlLinkedService lie Azure SQL Database à la fabrique de données. Les données copiées à partir du stockage Blob sont stockées dans cette base de données. Vous avez créé une table emp dans cette base de données dans le cadre des [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
10. Ajoutez à la méthode **Main** le code suivant, qui crée des **jeux de données d’entrée et de sortie**.

    ```csharp
    // create input and output datasets
    Console.WriteLine("Creating input and output datasets");
    string Dataset_Source = "InputDataset";
    string Dataset_Destination = "OutputDataset";

    Console.WriteLine("Creating input dataset of type: Azure Blob");
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,

    new DatasetCreateOrUpdateParameters()
    {
        Dataset = new Dataset()
        {
            Name = Dataset_Source,
            Properties = new DatasetProperties()
            {
                Structure = new List<DataElement>()
                {
                    new DataElement() { Name = "FirstName", Type = "String" },
                    new DataElement() { Name = "LastName", Type = "String" }
                },
                LinkedServiceName = "AzureStorageLinkedService",
                TypeProperties = new AzureBlobDataset()
                {
                    FolderPath = "adftutorial/",
                    FileName = "emp.txt"
                },
                External = true,
                Availability = new Availability()
                {
                    Frequency = SchedulePeriod.Hour,
                    Interval = 1,
                },

                Policy = new Policy()
                {
                    Validation = new ValidationPolicy()
                    {
                        MinimumRows = 1
                    }
                }
            }
        }
    });

    Console.WriteLine("Creating output dataset of type: Azure SQL");
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,
        new DatasetCreateOrUpdateParameters()
        {
            Dataset = new Dataset()
            {
                Name = Dataset_Destination,
                Properties = new DatasetProperties()
                {
                    Structure = new List<DataElement>()
                    {
                        new DataElement() { Name = "FirstName", Type = "String" },
                        new DataElement() { Name = "LastName", Type = "String" }
                    },
                    LinkedServiceName = "AzureSqlLinkedService",
                    TypeProperties = new AzureSqlTableDataset()
                    {
                        TableName = "emp"
                    },
                    Availability = new Availability()
                    {
                        Frequency = SchedulePeriod.Hour,
                        Interval = 1,
                    },
                }
            }
        });
    ```
    
    Dans l’étape précédente, vous avez créé des services liés pour lier votre compte de stockage Azure et Azure SQL Database à votre fabrique de données. Dans cette étape, vous définissez deux jeux de données nommés InputDataset et OutputDataset, qui représentent les données d’entrée/sortie stockées dans les banques de données référencées par AzureStorageLinkedService et AzureSqlLinkedService, respectivement.

    Le service lié Stockage Azure spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à votre compte de stockage Azure. Le jeu de données blob d’entrée (InputDataset) spécifie quant à lui le conteneur et le dossier qui contient les données d’entrée.  

    De même, le service lié Azure SQL Database spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à votre base de données. Et le jeu de données de la table SQL de sortie (OututDataset) spécifie la table de la base de données dans laquelle les données du stockage Blob sont copiées.

    Dans cette étape, vous créez un jeu de données nommé InputDataset qui pointe vers un fichier blob (emp.txt) dans le dossier racine d’un conteneur d’objets blob (adftutorial) du stockage Azure représenté par le service lié AzureStorageLinkedService. Si vous ne spécifiez pas de valeur pour fileName (ou si vous ignorez ce paramètre), les données de tous les objets blob du dossier d’entrée sont copiées dans la destination. Dans ce didacticiel, vous spécifiez une valeur pour fileName.    

    Dans cette étape, vous créez un jeu de données de sortie nommé **OutputDataset**. Ce jeu de données pointe vers une table SQL de la base de données représentée par **AzureSqlLinkedService**.
11. Ajoutez à la méthode **Main** le code suivant, qui **crée et active un pipeline**. Dans cette étape, vous créez un pipeline avec une **activité de copie** qui utilise **InputDataset** en entrée et **OutputDataset** en sortie.

    ```csharp
    // create a pipeline
    Console.WriteLine("Creating a pipeline");
    DateTime PipelineActivePeriodStartTime = new DateTime(2017, 5, 11, 0, 0, 0, 0, DateTimeKind.Utc);
    DateTime PipelineActivePeriodEndTime = new DateTime(2017, 5, 12, 0, 0, 0, 0, DateTimeKind.Utc);
    string PipelineName = "ADFTutorialPipeline";

    client.Pipelines.CreateOrUpdate(resourceGroupName, dataFactoryName,
        new PipelineCreateOrUpdateParameters()
        {
            Pipeline = new Pipeline()
            {
                Name = PipelineName,
                Properties = new PipelineProperties()
                {
                    Description = "Demo Pipeline for data transfer between blobs",

                    // Initial value for pipeline's active period. With this, you won't need to set slice status
                    Start = PipelineActivePeriodStartTime,
                    End = PipelineActivePeriodEndTime,

                    Activities = new List<Activity>()
                    {
                        new Activity()
                        {
                            Name = "BlobToAzureSql",
                            Inputs = new List<ActivityInput>()
                            {
                                new ActivityInput() {
                                    Name = Dataset_Source
                                }
                            },
                            Outputs = new List<ActivityOutput>()
                            {
                                new ActivityOutput()
                                {
                                    Name = Dataset_Destination
                                }
                            },
                            TypeProperties = new CopyActivity()
                            {
                                Source = new BlobSource(),
                                Sink = new BlobSink()
                                {
                                    WriteBatchSize = 10000,
                                    WriteBatchTimeout = TimeSpan.FromMinutes(10)
                                }
                            }
                        }
                    }
                }
            }
        });
    ```

    Notez les points suivants :
   
    - Dans la section des activités, il existe une seule activité dont le **type** a la valeur **Copy**. Pour plus d’informations sur l’activité de copie, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md). Dans les solutions Data Factory, vous pouvez également utiliser [Activités de transformation des données](data-factory-data-transformation-activities.md).
    - L’entrée de l’activité est définie sur **InputDataset** et sa sortie, sur **OutputDataset**. 
    - Dans la section **typeProperties**, **BlobSource** est spécifié en tant que type de source et **SqlSink**, en tant que type de récepteur. Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs pour l’activité de copie, consultez [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Pour apprendre à utiliser un magasin de données pris en charge spécifique en tant que source/récepteur, cliquez sur le lien dans le tableau.  
   
    Le jeu de données de sortie pilote actuellement la planification. Dans ce didacticiel, le jeu de données de sortie est configuré pour produire une tranche par heure. Les heures de début et de fin du pipeline sont distantes d’une journée, soit 24 heures. Par conséquent, 24 tranches de jeu de données de sortie sont générées par le pipeline.
12. Ajoutez le code suivant à la méthode **Main** pour obtenir l’état d’une tranche de données du jeu de données de sortie. Une seule tranche est attendue dans cet exemple.

    ```csharp
    // Pulling status within a timeout threshold
    DateTime start = DateTime.Now;
    bool done = false;

    while (DateTime.Now - start < TimeSpan.FromMinutes(5) && !done)
    {
        Console.WriteLine("Pulling the slice status");        
        // wait before the next status check
        Thread.Sleep(1000 * 12);

        var datalistResponse = client.DataSlices.List(resourceGroupName, dataFactoryName, Dataset_Destination,
            new DataSliceListParameters()
            {
                DataSliceRangeStartTime = PipelineActivePeriodStartTime.ConvertToISO8601DateTimeString(),
                DataSliceRangeEndTime = PipelineActivePeriodEndTime.ConvertToISO8601DateTimeString()
            });

        foreach (DataSlice slice in datalistResponse.DataSlices)
        {
            if (slice.State == DataSliceState.Failed || slice.State == DataSliceState.Ready)
            {
                Console.WriteLine("Slice execution is done with status: {0}", slice.State);
                done = true;
                break;
            }
            else
            {
                Console.WriteLine("Slice status is: {0}", slice.State);
            }
        }
    }
    ```

13. Ajoutez à la méthode **Main** le code suivant pour obtenir les détails d’exécution d’une tranche de données.

    ```csharp
    Console.WriteLine("Getting run details of a data slice");

    // give it a few minutes for the output slice to be ready
    Console.WriteLine("\nGive it a few minutes for the output slice to be ready and press any key.");
    Console.ReadKey();

    var datasliceRunListResponse = client.DataSliceRuns.List(
            resourceGroupName,
            dataFactoryName,
            Dataset_Destination,
            new DataSliceRunListParameters()
            {
                DataSliceStartTime = PipelineActivePeriodStartTime.ConvertToISO8601DateTimeString()
            }
        );

    foreach (DataSliceRun run in datasliceRunListResponse.DataSliceRuns)
    {
        Console.WriteLine("Status: \t\t{0}", run.Status);
        Console.WriteLine("DataSliceStart: \t{0}", run.DataSliceStart);
        Console.WriteLine("DataSliceEnd: \t\t{0}", run.DataSliceEnd);
        Console.WriteLine("ActivityId: \t\t{0}", run.ActivityName);
        Console.WriteLine("ProcessingStartTime: \t{0}", run.ProcessingStartTime);
        Console.WriteLine("ProcessingEndTime: \t{0}", run.ProcessingEndTime);
        Console.WriteLine("ErrorMessage: \t{0}", run.ErrorMessage);
    }

    Console.WriteLine("\nPress any key to exit.");
    Console.ReadKey();
    ```

14. Ajoutez à la classe **Program** la méthode d’assistance suivante utilisée par la méthode **Main**.

    > [!NOTE] 
    > Lorsque vous copiez et collez le code suivant, assurez-vous que le code copié se trouve au même niveau que la méthode Main.

    ```csharp
    public static async Task<string> GetAuthorizationHeader()
    {
        AuthenticationContext context = new AuthenticationContext(ConfigurationManager.AppSettings["ActiveDirectoryEndpoint"] + ConfigurationManager.AppSettings["ActiveDirectoryTenantId"]);
        ClientCredential credential = new ClientCredential(
            ConfigurationManager.AppSettings["ApplicationId"],
            ConfigurationManager.AppSettings["Password"]);
        AuthenticationResult result = await context.AcquireTokenAsync(
            resource: ConfigurationManager.AppSettings["WindowsManagementUri"],
            clientCredential: credential);

        if (result != null)
            return result.AccessToken;

        throw new InvalidOperationException("Failed to acquire token");
    }
    ```

15. Dans l’Explorateur de solutions, développez le projet (DataFactoryAPITestApp), cliquez avec le bouton droit sur **Références**, puis cliquez sur **Ajouter une référence**. Cochez la case de l’assemblage **System.Configuration**. et cliquez sur **OK**.
16. Générez l'application console. Dans le menu, cliquez sur **Générer**, puis sur **Générer la solution**.
17. Vérifiez qu’il existe au moins un fichier dans le conteneur **adftutorial** de votre stockage d’objets blob Azure. Si tel n’est pas le cas, créez le fichier **Emp.txt** dans le bloc-notes avec le contenu suivant, puis chargez-le dans le conteneur adftutorial.

    ```
    John, Doe
    Jane, Doe
    ```
18. Exécutez l’exemple en cliquant dans le menu sur **Déboguer** -> **Démarrer le débogage**. Si **Obtention des détails d’exécution d’une tranche de données** s’affiche, patientez quelques minutes, puis appuyez sur **Entrée**.
19. Utilisez le portail Azure pour vérifier que la fabrique de données **APITutorialFactory** est créée avec les artefacts suivants :
    * Service lié : **LinkedService_AzureStorage**
    * Jeu de données : **InputDataset** et **OutputDataset**.
    * Pipeline : **PipelineBlobSample**
20. Vérifiez que les deux enregistrements d’employés sont créés dans la table **emp** de la base de données spécifiée.

## <a name="next-steps"></a>Étapes suivantes
Pour la documentation complète sur l’API .NET pour Data Factory, consultez [Informations de référence sur l’API .NET Data Factory](/dotnet/api/index?view=azuremgmtdatafactories-4.12.1).

Dans ce tutoriel, vous avez utilisé le stockage Blob Azure comme magasin de données source et Azure SQL Database comme banque de données de destination dans une opération de copie. Le tableau ci-dessous contient la liste des magasins de données pris en charge en tant que sources et destinations par l’activité de copie : 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Pour découvrir comment copier des données vers/depuis un magasin de données, cliquez sur le lien du magasin de données dans le tableau.

