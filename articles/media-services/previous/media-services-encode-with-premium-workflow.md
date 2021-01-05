---
title: Encodage avancé avec Media Encoder Premium Workflow | Microsoft Docs
description: Découvrez comment encoder avec Media Encoder Premium Workflow. Les exemples de code sont écrits en C# et utilisent le Kit de développement logiciel (SDK) Media Services pour .NET.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 7f3efa23ad0b5e79e84a2436aaf56b67a805d05f
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92017231"
---
# <a name="advanced-encoding-with-media-encoder-premium-workflow"></a>Encodage avancé avec Media Encoder Premium Workflow

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Le processeur multimédia Media Encoder Premium Workflow présenté dans cet article n’est pas disponible en Chine.
>
>

## <a name="overview"></a>Vue d’ensemble
Microsoft Azure Media Services lance le processeur multimédia **Media Encoder Premium Workflow** . Ce processeur offre des fonctionnalités d’encodage avancées pour vos flux de travail à la demande premium.

Vous trouverez dans les rubriques suivantes des détails concernant **Media Encoder Premium Workflow**:

* [Formats pris en charge par le flux de travail Media Encoder Premium Workflow](./media-services-encode-asset.md) : présente les formats et les codecs pris en charge par **Media Encoder Premium Workflow**.
* [Vue d’ensemble et comparaison d’encodeurs multimédia à la demande Azure](media-services-encode-asset.md) compare les fonctionnalités d’encodage de **Media Encoder Premium Workflow** et de **Media Encoder Standard**.

Cet article montre comment encoder avec **Media Encoder Premium Workflow** à l’aide de .NET.

Les tâches d’encodage pour **Media Encoder Premium Workflow** nécessitent un fichier de configuration distinct appelé fichier de flux de travail. Ces fichiers ont une extension .workflow et sont créés à l’aide de l’outil [Concepteur de workflow](media-services-workflow-designer.md)

Vous pouvez aussi vous procurer les fichiers de flux de travail par défaut [ici](https://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/MediaEncoderPremiumWorkfows). Le dossier contient aussi la description de ces fichiers.

Les fichiers de flux de travail doivent être téléchargés vers votre compte Media Services sous forme de ressource, qui doit être transmise à la tâche d’encodage.

## <a name="create-and-configure-a-visual-studio-project"></a>Créer et configurer un projet Visual Studio

Configurez votre environnement de développement et ajoutez des informations de connexion au fichier app.config selon la procédure décrite dans l’article [Développement Media Services avec .NET](media-services-dotnet-how-to-use.md). 

## <a name="encoding-example"></a>Exemple d’encodage

L’exemple suivant montre comment encoder avec **Media Encoder Premium Workflow**.

Les étapes à exécuter sont les suivantes :

1. Créez une ressource et téléchargez un fichier de flux de travail.
2. Créez une ressource et téléchargez un fichier multimédia source.
3. Procurez-vous le processeur multimédia « Media Encoder Premium Workflow ».
4. Créez un travail et une tâche.

    Dans la plupart des cas, la chaîne de configuration de la tâche est vide (comme dans l'exemple suivant). Il existe des scénarios avancés (qui nécessitent de définir dynamiquement les propriétés d'exécution) auquel cas, vous fournissez une chaîne XML à la tâche d'encodage. La création d'une superposition, l'assemblage parallèle ou séquentiel multimédia, le sous-titrage sont des exemples de ces scénarios.
5. Ajoutez deux ressources d’entrée à la tâche.

   1. En premier lieu, la ressource de flux de travail.
   2. En second lieu, la ressource vidéo.

      >[!NOTE]
      >La ressource de flux de travail doit être ajoutée à la tâche avant la ressource multimédia.
      La chaîne de configuration de cette tâche doit être vide.
   
6. Soumettez le travail d’encodage.

```csharp
using System;
using System.Linq;
using System.Configuration;
using System.IO;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.MediaServices.Client;

namespace MediaEncoderPremiumWorkflowSample
{
    class Program
    {
        private static readonly string _AADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _RESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];

        // Field for service context.
        private static CloudMediaContext _context = null;

        private static readonly string _supportFiles =
            Path.GetFullPath(@"../..\Media");

        private static readonly string _workflowFilePath =
            Path.GetFullPath(_supportFiles + @"\H264 Progressive Download MP4.workflow");

        private static readonly string _singleMP4InputFilePath =
            Path.GetFullPath(_supportFiles + @"\BigBuckBunny.mp4");

        static void Main(string[] args)
        {

            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            var workflowAsset = CreateAssetAndUploadSingleFile(_workflowFilePath);
            var videoAsset = CreateAssetAndUploadSingleFile(_singleMP4InputFilePath);
            IAsset outputAsset = CreateEncodingJob(workflowAsset, videoAsset);

        }

        static public IAsset CreateAssetAndUploadSingleFile(string singleFilePath)
        {
            var assetName = "UploadSingleFile_" + DateTime.UtcNow.ToString();
            var asset = _context.Assets.Create(assetName, AssetCreationOptions.None);

            var fileName = Path.GetFileName(singleFilePath);

            var assetFile = asset.AssetFiles.Create(fileName);

            Console.WriteLine("Created assetFile {0}", assetFile.Name);

            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading {0}", assetFile.Name);

            return asset;
        }

        static public IAsset CreateEncodingJob(IAsset workflow, IAsset video)
        {
            // Declare a new job.
            IJob job = _context.Jobs.Create("Premium Workflow encoding job");
            // Get a media processor reference, and pass to it the name of the
            // processor to use for the specific task.
            IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Premium Workflow");

            // Create a task with the encoding details, using a string preset.
            ITask task = job.Tasks.AddNew("Premium Workflow encoding task",
                processor,
                "",
                TaskOptions.None);

            // Specify the input asset to be encoded.
            task.InputAssets.Add(workflow);
            task.InputAssets.Add(video); // we add one asset
                                         // Add an output asset to contain the results of the job.
                                         // This output is specified as AssetCreationOptions.None, which
                                         // means the output asset is not encrypted.
            task.OutputAssets.AddNew("Output asset",
                AssetCreationOptions.None);

            // Use the following event handler to check job progress.  
            job.StateChanged += new
                    EventHandler<JobStateChangedEventArgs>(StateChanged);

            // Launch the job.
            job.Submit();

            // Check job execution and wait for job to finish.
            Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
            progressJobTask.Wait();

            // If job state is Error the event handling
            // method for job progress should log errors.  Here we check
            // for error state and exit if needed.
            if (job.State == JobState.Error)
            {
                throw new Exception("\nExiting method due to job error.");
            }

            return job.OutputMediaAssets[0];
        }

        static private void StateChanged(object sender, JobStateChangedEventArgs e)
        {
            Console.WriteLine("Job state changed event:");
            Console.WriteLine("  Previous state: " + e.PreviousState);
            Console.WriteLine("  Current state: " + e.CurrentState);

            switch (e.CurrentState)
            {
                case JobState.Finished:
                    Console.WriteLine();
                    Console.WriteLine("Job is finished.");
                    Console.WriteLine();
                    break;
                case JobState.Canceling:
                case JobState.Queued:
                case JobState.Scheduled:
                case JobState.Processing:
                    Console.WriteLine("Please wait...\n");
                    break;
                case JobState.Canceled:
                case JobState.Error:
                    // Cast sender as a job.
                    IJob job = (IJob)sender;
                    // Display or log error details as needed.
                    // LogJobStop(job.Id);
                    break;
                default:
                    break;
            }
        }
        private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
        {
            var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
            ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

            if (processor == null)
                throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

            return processor;
        }
    }
}
```

## <a name="need-help"></a>Vous avez besoin d’aide ?

Vous pouvez ouvrir un ticket de support en accédant à [Nouvelle demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]