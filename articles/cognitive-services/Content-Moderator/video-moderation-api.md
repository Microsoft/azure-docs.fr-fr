---
title: Analyser la présence de matériau répréhensible dans du contenu vidéo en C# - Content Moderator
titleSuffix: Azure Cognitive Services
description: Guide pratique pour analyser la présence de matériau répréhensible dans du contenu vidéo avec le Kit SDK Content Moderator pour .NET.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: how-to
ms.date: 05/18/2020
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 4f98eac4305333ec7225c90da2777b7e02f050a0
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96853530"
---
# <a name="analyze-video-content-for-objectionable-material-in-c"></a>Analyser la présence de matériau répréhensible dans du contenu vidéo en C#

Cet article donne des informations et des exemples de code qui vous aideront à bien démarrer avec le [Kit de développement logiciel (SDK) Content Moderator pour .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) afin d’analyser la présence de contenu suggestif ou pour adultes dans du contenu vidéo.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer. 

## <a name="prerequisites"></a>Conditions préalables requises
- N’importe quelle édition de [Visual Studio 2015 ou 2017](https://www.visualstudio.com/downloads/)

## <a name="set-up-azure-resources"></a>Configurer les ressources Azure

Les fonctionnalités de modération de vidéos de Content Moderator sont disponibles gratuitement sous forme de **processeur multimédia** en préversion publique dans Azure Media Services (AMS). Azure Media Services est un service Azure spécialisé dans le stockage et la diffusion en continu de contenu vidéo. 

### <a name="create-an-azure-media-services-account"></a>Créer un compte Azure Media Services

Suivez les instructions de [Créer un compte Azure Media Services](../../media-services/previous/media-services-portal-create-account.md) pour vous abonner à AMS et créer un compte de stockage Azure associé. Dans ce compte de stockage, créez un conteneur de stockage Blob.

### <a name="create-an-azure-active-directory-application"></a>Créer une application Azure Active Directory

Accédez à votre nouvel abonnement AMS sur le Portail Azure et sélectionnez **Accès à l’API** dans le menu latéral. Sélectionnez **Se connecter à Azure Media Services avec le principal de service**. Notez la valeur du champ **Point de terminaison d’API REST** ; vous en aurez besoin plus tard.

Dans la section **Application Azure AD**, sélectionnez **Créer** et donnez un nom à votre nouvelle inscription d’application Azure AD (par exemple, « VideoModADApp »). Cliquez sur **Enregistrer** et patientez quelques minutes, le temps que l’application se configure. Votre nouvelle inscription d’application devrait alors apparaître sous la section **Application Azure AD** de la page.

Sélectionnez votre inscription d’application et cliquez sur le bouton **Gérer l’application** qui se trouve en dessous. Notez la valeur du champ **ID d’application** ; vous en aurez besoin plus tard. Sélectionnez **Paramètres** > **Clés**, puis entrez la description de la nouvelle clé (par exemple, « VideoModKey »). Cliquez sur **Enregistrer**, puis copiez la nouvelle valeur de clé et enregistrez-la dans un endroit sûr.

Pour une présentation plus détaillée de la procédure ci-dessus, voir [Bien démarrer avec l’authentification Azure AD](../../media-services/previous/media-services-portal-get-started-with-aad.md).

Vous pourrez ensuite utiliser le processeur multimédia de modération de vidéos de deux manières différentes.

## <a name="use-azure-media-services-explorer"></a>Utiliser Azure Media Services Explorer

L’explorateur Azure Media Services Explorer est un front-end convivial pour AMS. Grâce à lui, vous pouvez parcourir votre compte AMS, charger des vidéos et analyser du contenu avec le processeur multimédia Content Moderator. Téléchargez-le sur [GitHub](https://github.com/Azure/Azure-Media-Services-Explorer/releases) et installez-le. Pour plus d’informations, voir le [billet de blog Explorateur Azure Media Services](https://azure.microsoft.com/blog/managing-media-workflows-with-the-new-azure-media-services-explorer-tool/).

![Explorateur Azure Media Services avec Content Moderator](images/ams-explorer-content-moderator.PNG)

## <a name="create-the-visual-studio-project"></a>Créer le projet Visual Studio

1. Dans Visual Studio, créez un projet **Application de console (.NET Framework)** et nommez-le **VideoModeration**. 
1. Si votre solution comporte d’autres projets, sélectionnez celui-ci comme projet de démarrage unique.
1. Récupérez les packages NuGet requis. Cliquez avec le bouton droit sur votre projet dans l’Explorateur de solutions et sélectionnez **Gérer les packages NuGet** ; ensuite, recherchez et installez les packages suivants :
    - windowsazure.mediaservices
    - windowsazure.mediaservices.extensions

## <a name="add-video-moderation-code"></a>Ajouter du code de modération de vidéos

Vous allez maintenant copier et coller le code de ce guide dans votre projet pour implémenter un scénario de base de modération du contenu.

### <a name="update-the-programs-using-statements"></a>Mettre à jour les instructions using du programme

Ajoutez les instructions `using` suivantes en haut de votre fichier _Program.cs_.

```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.IO;
using System.Threading;
using Microsoft.WindowsAzure.Storage.Blob;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Auth;
using System.Collections.Generic;
```

### <a name="set-up-resource-references"></a>Configurer les références aux ressources

Ajoutez les champs statiques suivants à la classe **Program** dans _Program.cs_. Ces champs contiennent les informations nécessaires pour se connecter à l’abonnement AMS. Remplissez-les avec les valeurs obtenues aux étapes ci-dessus. Notez que `CLIENT_ID` est la valeur **ID d’application** de votre application Azure AD, et `CLIENT_SECRET` la valeur de « VideoModKey » que vous avez créée pour cette application.

```csharp
// declare constants and globals
private static CloudMediaContext _context = null;
private static CloudStorageAccount _StorageAccount = null;

// Azure Media Services (AMS) associated Storage Account, Key, and the Container that has 
// a list of Blobs to be processed.
static string STORAGE_NAME = "YOUR AMS ASSOCIATED BLOB STORAGE NAME";
static string STORAGE_KEY = "YOUR AMS ASSOCIATED BLOB STORAGE KEY";
static string STORAGE_CONTAINER_NAME = "YOUR BLOB CONTAINER FOR VIDEO FILES";

private static StorageCredentials _StorageCredentials = null;

// Azure Media Services authentication. 
private const string AZURE_AD_TENANT_NAME = "microsoft.onmicrosoft.com";
private const string CLIENT_ID = "YOUR CLIENT ID";
private const string CLIENT_SECRET = "YOUR CLIENT SECRET";

// REST API endpoint, for example "https://accountname.restv2.westcentralus.media.azure.net/API".      
private const string REST_API_ENDPOINT = "YOUR API ENDPOINT";

// Content Moderator Media Processor Nam
private const string MEDIA_PROCESSOR = "Azure Media Content Moderator";

// Input and Output files in the current directory of the executable
private const string INPUT_FILE = "VIDEO FILE NAME";
private const string OUTPUT_FOLDER = "";

// JSON settings file
private static readonly string CONTENT_MODERATOR_PRESET_FILE = "preset.json";

```

Si vous souhaitez utiliser un fichier vidéo local (cas le plus simple), ajoutez-le au projet et entrez son chemin d’accès sous forme de valeur `INPUT_FILE` (chemins d’accès relatifs au répertoire d’exécution).

Vous devrez également créer le fichier _preset.json_ dans le répertoire actif et l’utiliser pour spécifier un numéro de version. Par exemple :

```JSON
{
    "version": "2.0"
}
```

### <a name="load-the-input-videos"></a>Charger la ou les vidéos d’entrée

La méthode **Main** de la classe **Program** crée un contexte Azure Media, puis un contexte Stockage Azure (au cas où les vidéos se trouveraient dans un Stockage Blob). Le code restant analyse une vidéo à partir d’un dossier local, ou d’un ou plusieurs objets blob dans un conteneur de stockage Azure. Vous pouvez essayer toutes les options en commentant les autres lignes de code.

```csharp
// Create Azure Media Context
CreateMediaContext();

// Create Storage Context
CreateStorageContext();

// Use a file as the input.
IAsset asset = CreateAssetfromFile();

// -- OR ---

// Or a blob as the input
// IAsset asset = CreateAssetfromBlob((CloudBlockBlob)GetBlobsList().First());

// Then submit the asset to Content Moderator
RunContentModeratorJob(asset);

//-- OR ----

// Just run the content moderator on all blobs in a list (from a Blob Container)
// RunContentModeratorJobOnBlobs();
```

### <a name="create-an-azure-media-context"></a>Créer un contexte Azure Media

Ajoutez la méthode suivante à la classe **Program**. Vos informations d’identification AMS sont nécessaires pour permettre la communication avec AMS.

```csharp
// Creates a media context from azure credentials
static void CreateMediaContext()
{
    // Get Azure AD credentials
    var tokenCredentials = new AzureAdTokenCredentials(AZURE_AD_TENANT_NAME,
        new AzureAdClientSymmetricKey(CLIENT_ID, CLIENT_SECRET),
        AzureEnvironments.AzureCloudEnvironment);

    // Initialize an Azure AD token
    var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

    // Create a media context
    _context = new CloudMediaContext(new Uri(REST_API_ENDPOINT), tokenProvider);
}
```

### <a name="add-the-code-to-create-an-azure-storage-context"></a>Ajouter le code pour créer un contexte Stockage Azure

Ajoutez la méthode suivante à la classe **Program**. Le contexte Stockage, créé à partir de vos informations d’identification de stockage, vous permet d’accéder à votre Stockage Blob.

```csharp
// Creates a storage context from the AMS associated storage name and key
static void CreateStorageContext()
{
    // Get a reference to the storage account associated with a Media Services account. 
    if (_StorageCredentials == null)
    {
        _StorageCredentials = new StorageCredentials(STORAGE_NAME, STORAGE_KEY);
    }
    _StorageAccount = new CloudStorageAccount(_StorageCredentials, false);
}
```

### <a name="add-the-code-to-create-azure-media-assets-from-local-file-and-blob"></a>Ajouter le code pour créer des actifs multimédias Azure à partir d’un fichier local et d’un objet blob

Le processeur multimédia Content Moderator exécute des travaux sur des **Actifs** au sein de la plateforme Azure Media Services.
Ces méthodes créent les Actifs à partir d’un fichier local ou d’un objet blob associé.

```csharp
// Creates an Azure Media Services Asset from the video file
static IAsset CreateAssetfromFile()
{
    return _context.Assets.CreateFromFile(INPUT_FILE, AssetCreationOptions.None); ;
}

// Creates an Azure Media Services asset from your blog storage
static IAsset CreateAssetfromBlob(CloudBlockBlob Blob)
{
    // Create asset from the FIRST blob in the list and return it
    return _context.Assets.CreateFromBlob(Blob, _StorageCredentials, AssetCreationOptions.None);
}
```

### <a name="add-the-code-to-scan-a-collection-of-videos-as-blobs-within-a-container"></a>Ajouter le code pour analyser une collection de vidéos (en tant qu’objets blob) au sein d’un conteneur

```csharp
// Runs the Content Moderator Job on all Blobs in a given container name
static void RunContentModeratorJobOnBlobs()
{
    // Get the reference to the list of Blobs. See the following method.
    var blobList = GetBlobsList();

    // Iterate over the Blob list items or work on specific ones as needed
    foreach (var sourceBlob in blobList)
    {
        // Create an Asset
        IAsset asset = _context.Assets.CreateFromBlob((CloudBlockBlob)sourceBlob,
                            _StorageCredentials, AssetCreationOptions.None);
        asset.Update();

        // Submit to Content Moderator
        RunContentModeratorJob(asset);
    }
}

// Get all blobs in your container
static IEnumerable<IListBlobItem> GetBlobsList()
{
    // Get a reference to the Container within the Storage Account
    // that has the files (blobs) for moderation
    CloudBlobClient CloudBlobClient = _StorageAccount.CreateCloudBlobClient();
    CloudBlobContainer MediaBlobContainer = CloudBlobClient.GetContainerReference(STORAGE_CONTAINER_NAME);

    // Get the reference to the list of Blobs 
    var blobList = MediaBlobContainer.ListBlobs();
    return blobList;
}
```

### <a name="add-the-method-to-run-the-content-moderator-job"></a>Ajouter la méthode pour exécuter le travail de Content Moderator

```csharp
// Run the Content Moderator job on the designated Asset from local file or blob storage
static void RunContentModeratorJob(IAsset asset)
{
    // Grab the presets
    string configuration = File.ReadAllText(CONTENT_MODERATOR_PRESET_FILE);

    // grab instance of Azure Media Content Moderator MP
    IMediaProcessor mp = _context.MediaProcessors.GetLatestMediaProcessorByName(MEDIA_PROCESSOR);

    // create Job with Content Moderator task
    IJob job = _context.Jobs.Create(String.Format("Content Moderator {0}",
            asset.AssetFiles.First() + "_" + Guid.NewGuid()));

    ITask contentModeratorTask = job.Tasks.AddNew("Adult and racy classifier task",
            mp, configuration,
            TaskOptions.None);
    contentModeratorTask.InputAssets.Add(asset);
    contentModeratorTask.OutputAssets.AddNew("Adult and racy classifier output",
        AssetCreationOptions.None);

    job.Submit();


    // Create progress printing and querying tasks
    Task progressPrintTask = new Task(() =>
    {
        IJob jobQuery = null;
        do
        {
            var progressContext = _context;
            jobQuery = progressContext.Jobs
            .Where(j => j.Id == job.Id)
                .First();
                Console.WriteLine(string.Format("{0}\t{1}",
                DateTime.Now,
                jobQuery.State));
                Thread.Sleep(10000);
            }
            while (jobQuery.State != JobState.Finished &&
            jobQuery.State != JobState.Error &&
            jobQuery.State != JobState.Canceled);
    });
    progressPrintTask.Start();

    Task progressJobTask = job.GetExecutionProgressTask(
    CancellationToken.None);
    progressJobTask.Wait();

    // If job state is Error, the event handling 
    // method for job progress should log errors.  Here we check 
    // for error state and exit if needed.
    if (job.State == JobState.Error)
    {
        ErrorDetail error = job.Tasks.First().ErrorDetails.First();
        Console.WriteLine(string.Format("Error: {0}. {1}",
        error.Code,
        error.Message));
    }

    DownloadAsset(job.OutputMediaAssets.First(), OUTPUT_FOLDER);
}
```

### <a name="add-helper-functions"></a>Ajouter des fonctions d’assistance

Ces méthodes téléchargent le fichier de sortie (JSON) de Content Moderator à partir de l’actif Azure Media Services, et facilitent le suivi de l’état du travail de modération afin que le programme puisse journaliser un état d’exécution sur la console.

```csharp
static void DownloadAsset(IAsset asset, string outputDirectory)
{
    foreach (IAssetFile file in asset.AssetFiles)
    {
        file.Download(Path.Combine(outputDirectory, file.Name));
    }
}

// event handler for Job State
static void StateChanged(object sender, JobStateChangedEventArgs e)
{
    Console.WriteLine("Job state changed event:");
    Console.WriteLine("  Previous state: " + e.PreviousState);
    Console.WriteLine("  Current state: " + e.CurrentState);
    switch (e.CurrentState)
    {
        case JobState.Finished:
            Console.WriteLine();
            Console.WriteLine("Job finished.");
            break;
        case JobState.Canceling:
        case JobState.Queued:
        case JobState.Scheduled:
        case JobState.Processing:
            Console.WriteLine("Please wait...\n");
            break;
        case JobState.Canceled:
            Console.WriteLine("Job is canceled.\n");
            break;
        case JobState.Error:
            Console.WriteLine("Job failed.\n");
            break;
        default:
            break;
    }
}
```

### <a name="run-the-program-and-review-the-output"></a>Exécuter le programme et examiner la sortie

Une fois le travail de modération du contenu accompli, analysez la réponse JSON. Elle se compose des éléments suivants :

- Résumé des informations sur la vidéo
- **Captures** en tant que « **fragments** »
- **Images clés** en tant qu’« **événements** » avec un indicateur **reviewRecommended" (= true or false)"** basé sur les scores **Adult** et **Racy**
- **start**, **duration**, **totalDuration** et **timestamp** sont exprimés en « tics ». Divisez par la valeur **timestamp** pour obtenir le nombre de secondes.
 
> [!NOTE]
> - `adultScore` représente la présence potentielle et le score de prédiction du contenu pouvant être considéré comme sexuellement explicite ou pour adultes dans certaines situations.
> - `racyScore` représente la présence potentielle et le score de prédiction du contenu pouvant être considéré comme sexuellement suggestif ou pour adultes dans certaines situations.
> - `adultScore` et `racyScore` ont des valeurs de 0 à 1. Plus le score est élevé, plus le modèle prédit que la catégorie peut être applicable. Cette préversion s’appuie sur un modèle statistique plutôt que sur des résultats codés manuellement. Nous vous recommandons d’effectuer le test avec votre propre contenu afin de déterminer comment chaque catégorie correspond à vos besoins.
> - La valeur de `reviewRecommended` est true ou false, selon les seuils de score internes. Les clients doivent évaluer l’opportunité d’utiliser cette valeur ou choisir des seuils personnalisés en fonction de leurs stratégies de contenu.

```json
{
"version": 2,
"timescale": 90000,
"offset": 0,
"framerate": 50,
"width": 1280,
"height": 720,
"totalDuration": 18696321,
"fragments": [
{
    "start": 0,
    "duration": 18000
},
{
    "start": 18000,
    "duration": 3600,
    "interval": 3600,
    "events": [
    [
        {
        "reviewRecommended": false,
        "adultScore": 0.00001,
        "racyScore": 0.03077,
        "index": 5,
        "timestamp": 18000,
        "shotIndex": 0
        }
    ]
    ]
},
{
    "start": 18386372,
    "duration": 119149,
    "interval": 119149,
    "events": [
    [
        {
        "reviewRecommended": true,
        "adultScore": 0.00000,
        "racyScore": 0.91902,
        "index": 5085,
        "timestamp": 18386372,
        "shotIndex": 62
        }
    ]
    ]
}
]
}
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment générer des [analyses vidéo](video-reviews-quickstart-dotnet.md) à partir du résultat de votre modération.

[Téléchargez la solution Visual Studio](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator) pour ce guide de démarrage rapide et d’autres guides de démarrage rapide Content Moderator pour .NET.