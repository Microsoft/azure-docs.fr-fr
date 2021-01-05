---
title: Créer des pipelines de données à l’aide du Kit de développement logiciel (SDK) .NET Azure
description: Découvrez comment créer, analyser et gérer par programmation des fabriques de données Azure à l'aide du Kit de développement logiciel (SDK) Data Factory.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/22/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: dd1c7c20441d12600e2101f738904caf6260509e
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96496004"
---
# <a name="create-monitor-and-manage-azure-data-factories-using-azure-data-factory-net-sdk"></a>Créer, surveiller et gérer des fabriques de données Azure à l’aide du Kit de développement logiciel (SDK) Azure Data Factory .NET
> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez le [tutoriel sur l’activité de copie](../quickstart-create-data-factory-dot-net.md). 

## <a name="overview"></a>Vue d’ensemble
Vous pouvez créer, surveiller et gérer des fabriques de données Azure par programmation à l'aide du Kit SDK Data Factory .NET. Cet article contient une procédure pas à pas que vous pouvez suivre pour créer un exemple d'application console .NET qui crée et surveille une fabrique de données. 

> [!NOTE]
> Cet article ne couvre pas toutes les API .NET Data Factory. Consultez [Informations de référence sur l’API .NET Data Factory](/dotnet/api/index?view=azuremgmtdatafactories-4.12.1) pour la documentation complète sur l’API .NET pour Data Factory. 

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Visual Studio 2012, 2013 ou 2015
* Téléchargez et installez le [Kit SDK Azure .NET](https://azure.microsoft.com/downloads/).
* Azure PowerShell. Suivez les instructions de l’article [Installation et configuration d’Azure PowerShell](/powershell/azure/) pour installer Azure PowerShell sur votre ordinateur. Vous utilisez Azure PowerShell pour créer une application Azure Active Directory.

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
    $azureAdApplication = New-AzADApplication -DisplayName "ADFDotNetWalkthroughApp" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.adfdotnetwalkthroughapp.org/example" -Password "Pass@word1"
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
Dans la procédure pas à pas, vous créez une fabrique de données avec un pipeline qui contient une activité de copie. L’activité de copie les données d’un dossier de votre stockage d’objets blob Azure vers un autre dossier dans le même stockage d’objets blob. 

L’activité de copie effectue le déplacement des données dans Azure Data Factory. Elle est mise en œuvre par un service disponible dans le monde entier, capable de copier des données entre différents magasins de données de façon sécurisée, fiable et évolutive. Pour plus d’informations sur l’activité de copie, consultez l’article [Activités de déplacement des données](data-factory-data-movement-activities.md) .

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
4. Remplacez le contenu du fichier **App.config** du projet par le contenu suivant : 
    
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
5. Dans le fichier App.Config, remplacez les valeurs de **&lt;ID d’Application&gt;** , **&lt;Mot de passe&gt;** , **&lt;ID d’abonnement&gt;** , et **&lt;ID client&gt;** par vos propres valeurs.
6. Ajoutez les instructions **using** ci-après dans le fichier **Program.cs** du projet.

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

    //IMPORTANT: specify the name of Azure resource group here
    string resourceGroupName = "ADFTutorialResourceGroup";

    //IMPORTANT: the name of the data factory must be globally unique.
    // Therefore, update this value. For example:APITutorialFactory05122017
    string dataFactoryName = "APITutorialFactory";

    TokenCloudCredentials aadTokenCredentials = new TokenCloudCredentials(
            ConfigurationManager.AppSettings["SubscriptionId"],
            GetAuthorizationHeader().Result);

    Uri resourceManagerUri = new Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

    DataFactoryManagementClient client = new DataFactoryManagementClient(aadTokenCredentials, resourceManagerUri);
    ```

   > [!IMPORTANT]
   > Remplacez la valeur de **resourceGroupName** par le nom de votre groupe de ressources Azure. Vous pouvez créer un groupe de ressources à l’aide du cmdlet [New-AzureResourceGroup](/powershell/module/az.resources/new-azresourcegroup) .
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
9. Ajoutez à la méthode **Main** le code suivant, qui crée des **jeux de données d’entrée et de sortie**.

    Le paramètre **FolderPath** de l’objet blob d’entrée a la valeur **adftutorial/** , où **adftutorial** est le nom du conteneur dans votre stockage des objets blob. Si ce conteneur n'existe pas dans votre stockage d'objets blob Azure, créez un conteneur nommé **adftutorial** et chargez un fichier texte sur celui-ci.

    Le paramètre FolderPath de l’objet blob de sortie est défini sur **adftutorial/apifactoryoutput/{Slice}** où la valeur **Slice** est calculée dynamiquement en fonction de la valeur de **SliceStart** (date/heure de début de chaque tranche).

    ```csharp
    // create input and output datasets
    Console.WriteLine("Creating input and output datasets");
    string Dataset_Source = "DatasetBlobSource";
    string Dataset_Destination = "DatasetBlobDestination";
    
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,
    new DatasetCreateOrUpdateParameters()
    {
        Dataset = new Dataset()
        {
            Name = Dataset_Source,
            Properties = new DatasetProperties()
            {
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
    
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,
    new DatasetCreateOrUpdateParameters()
    {
        Dataset = new Dataset()
        {
            Name = Dataset_Destination,
            Properties = new DatasetProperties()
            {
    
                LinkedServiceName = "AzureStorageLinkedService",
                TypeProperties = new AzureBlobDataset()
                {
                    FolderPath = "adftutorial/apifactoryoutput/{Slice}",
                    PartitionedBy = new Collection<Partition>()
                    {
                        new Partition()
                        {
                            Name = "Slice",
                            Value = new DateTimePartitionValue()
                            {
                                Date = "SliceStart",
                                Format = "yyyyMMdd-HH"
                            }
                        }
                    }
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
10. Ajoutez à la méthode **Main** le code suivant, qui **crée et active un pipeline**. Ce pipeline dispose d’une fonction **CopyActivity** qui accepte **BlobSource** en tant que source et **BlobSink** en tant que récepteur.

    L’activité de copie effectue le déplacement des données dans Azure Data Factory. Elle est mise en œuvre par un service disponible dans le monde entier, capable de copier des données entre différents magasins de données de façon sécurisée, fiable et évolutive. Pour plus d’informations sur l’activité de copie, consultez l’article [Activités de déplacement des données](data-factory-data-movement-activities.md) .

    ```csharp
    // create a pipeline
    Console.WriteLine("Creating a pipeline");
    DateTime PipelineActivePeriodStartTime = new DateTime(2014, 8, 9, 0, 0, 0, 0, DateTimeKind.Utc);
    DateTime PipelineActivePeriodEndTime = PipelineActivePeriodStartTime.AddMinutes(60);
    string PipelineName = "PipelineBlobSample";
    
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
                        Name = "BlobToBlob",
                        Inputs = new List<ActivityInput>()
                        {
                            new ActivityInput()
                {
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
    
                },
            }
        }
    });
    ```
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
13. **(facultatif)** Ajoutez à la méthode **Main** le code suivant pour obtenir les détails d’exécution d’une tranche de données.

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
        });
    
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
14. Ajoutez à la classe **Program** la méthode d’assistance suivante utilisée par la méthode **Main**. Cette méthode affiche une boîte de dialogue qui vous permet de fournir un **nom d’utilisateur** et un **mot de passe** de connexion au portail Azure.

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

15. Dans l’Explorateur de solutions, développez le projet : **DataFactoryAPITestApp**, cliquez avec le bouton droit sur **Références**, puis cliquez sur **Ajouter une référence**. Cochez la case pour l’assembly `System.Configuration` et cliquez sur **OK**.
15. Générez l'application console. Dans le menu, cliquez sur **Générer**, puis sur **Générer la solution**.
16. Vérifiez qu'il existe au moins un fichier dans le conteneur adftutorial de votre stockage d'objets blob Azure. Si ce n'est pas le cas, créez le fichier Emp.txt dans le bloc-notes avec le contenu suivant, puis chargez-le sur le conteneur adftutorial.

    ```
    John, Doe
    Jane, Doe
    ```
17. Exécutez l’exemple en cliquant dans le menu sur **Déboguer** -> **Démarrer le débogage**. Si **Obtention des détails d’exécution d’une tranche de données** s’affiche, patientez quelques minutes, puis appuyez sur **Entrée**.
18. Utilisez le portail Azure pour vérifier que la fabrique de données **APITutorialFactory** est créée avec les artefacts suivants :
    * Service lié : **AzureStorageLinkedService**
    * Jeu de données : **DatasetBlobSource** et **DatasetBlobDestination**.
    * Pipeline : **PipelineBlobSample**
19. Vérifiez qu’un fichier de sortie est créé dans le dossier **apifactoryoutput** du conteneur **adftutorial**.

## <a name="get-a-list-of-failed-data-slices"></a>Obtenir une liste des tranches de données en échec 

```csharp
// Parse the resource path
var ResourceGroupName = "ADFTutorialResourceGroup";
var DataFactoryName = "DataFactoryAPITestApp";

var parameters = new ActivityWindowsByDataFactoryListParameters(ResourceGroupName, DataFactoryName);
parameters.WindowState = "Failed";
var response = dataFactoryManagementClient.ActivityWindows.List(parameters);
do
{
    foreach (var activityWindow in response.ActivityWindowListResponseValue.ActivityWindows)
    {
        var row = string.Join(
            "\t",
            activityWindow.WindowStart.ToString(),
            activityWindow.WindowEnd.ToString(),
            activityWindow.RunStart.ToString(),
            activityWindow.RunEnd.ToString(),
            activityWindow.DataFactoryName,
            activityWindow.PipelineName,
            activityWindow.ActivityName,
            string.Join(",", activityWindow.OutputDatasets));
        Console.WriteLine(row);
    }

    if (response.NextLink != null)
    {
        response = dataFactoryManagementClient.ActivityWindows.ListNext(response.NextLink, parameters);
    }
    else
    {
        response = null;
    }
}
while (response != null);
```

## <a name="next-steps"></a>Étapes suivantes
Consultez l’exemple suivant pour créer un pipeline à l’aide du Kit de développement logiciel .NET qui copie les données d’un stockage d’objets blob Azure dans Azure SQL Database : 

- [Créer un pipeline pour copier des données de stockage d’objets Blob dans SQL Database](data-factory-copy-activity-tutorial-using-dotnet-api.md)
