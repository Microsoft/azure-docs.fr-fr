---
title: Encoder un élément multimédia avec Media Encoder Standard à l’aide de .NET | Microsoft Docs
description: Cette article explique comment utiliser .NET pour encoder vos éléments multimédias avec Media Encoder Standard.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 03431b64-5518-478a-a1c2-1de345999274
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.reviewer: anilmur
ms.custom: devx-track-csharp
ms.openlocfilehash: 5f49b951473db4b331bc42de3fb72829270822d5
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92014373"
---
# <a name="encode-an-asset-with-media-encoder-standard-using-net"></a>Encoder un élément multimédia avec Media Encoder Standard à l’aide de .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]  

Les tâches d’encodage sont une des opérations de traitement les plus courantes dans Media Services. Vous créez des tâches d’encodage pour convertir des fichiers multimédias d’un encodage à un autre. Lorsque vous les encodez, vous pouvez utiliser l’encodeur multimédia intégré de Media Services. Vous pouvez également utiliser un encodeur fourni par un partenaire Media Services. Ces encodeurs tiers sont disponibles via Place de marché Azure. 

Cette article montre comment utiliser .NET pour encoder vos éléments multimédias avec Media Encoder Standard (MES). Media Encoder Standard se configure à l’aide d’une des présélections d’encodeur décrites [ici](./media-services-mes-presets-overview.md).

Nous vous recommandons de toujours encoder vos fichiers sources sous forme de jeu de fichiers MP4 à débit adaptatif, puis de convertir ce jeu au format souhaité au moyen de l’ [empaquetage dynamique](media-services-dynamic-packaging-overview.md). 

Si votre ressource de sortie est stockée sous forme chiffrée, vous devez configurer une stratégie de remise de ressources. Pour plus d'informations, consultez [Configuration de la stratégie de remise de ressources](media-services-dotnet-configure-asset-delivery-policy.md).

> [!NOTE]
> MES génère un fichier de sortie avec un nom qui contient les 32 premiers caractères du nom du fichier d'entrée. Le nom est basé sur ce qui est spécifié dans le fichier prédéfini. Par exemple, "FileName": "{NomBase}_{Index}{Extension}". {NomBase} est remplacé par les 32 premiers caractères du nom du fichier d'entrée.
> 
> 

### <a name="mes-formats"></a>Formats MES
[Formats et codecs](media-services-media-encoder-standard-formats.md)

### <a name="mes-presets"></a>Présélections MES
Media Encoder Standard se configure à l’aide d’une des présélections d’encodeur décrites [ici](./media-services-mes-presets-overview.md).

### <a name="input-and-output-metadata"></a>Métadonnées d’entrée et de sortie
Quand vous encodez un ou plusieurs éléments multimédias d’entrée à l’aide de MES, vous obtenez un élément multimédia de sortie une fois cette tâche d’encodage terminée. L’élément multimédia de sortie contient la vidéo, l’audio, les miniatures, le manifeste, et ainsi de suite, en fonction des paramètres d’encodage prédéfinis.

Il contient également un fichier avec des métadonnées relatives à l’élément multimédia d’entrée. Le nom du fichier XML de métadonnées a le format suivant : <asset_id>_metadata.xml (par exemple, 41114ad3-eb5e-4c57-8d92-5354e2b7d4a4_metadata.xml), où <asset_id> est la valeur AssetId de l’élément multimédia d’entrée. Le schéma de ce XML de métadonnées d’entrée est décrit [ici](media-services-input-metadata-schema.md).

L’élément multimédia de sortie contient également un fichier avec des métadonnées relatives à l’élément multimédia de sortie. Le nom du fichier XML de métadonnées a le format suivant : <nom_fichier_source>_manifest.xml (par exemple, BigBuckBunny_manifest.xml). Le schéma de ce XML de métadonnées de sortie est décrit [ici](media-services-output-metadata-schema.md).

Si vous souhaitez examiner l’un des deux fichiers de métadonnées, vous pouvez créer un localisateur SAS et télécharger le fichier sur votre ordinateur local. Vous trouverez un exemple illustrant comment créer un localisateur SAS et télécharger un fichier dans la rubrique Utilisation des extensions du SDK Media Services .NET.

## <a name="download-sample"></a>Charger l’exemple
Vous pouvez obtenir et exécuter un exemple qui montre comment encoder avec MES [ici](https://azure.microsoft.com/documentation/samples/media-services-dotnet-on-demand-encoding-with-media-encoder-standard/).

## <a name="net-sample-code"></a>Exemple de code .NET

Le code suivant utilise le Kit de développement logiciel (SDK) .NET de Media Services pour effectuer les tâches suivantes :

* Création d’une tâche d’encodage.
* Obtention d’une référence à l’encodeur Media Encoder Standard.
* Spécifiez l’utilisation de la présélection [Diffusion en continu adaptative](media-services-autogen-bitrate-ladder-with-mes.md). 
* Ajout d’une tâche d’encodage unique. 
* Spécification de l’élément multimédia d’entrée à encoder.
* Création d’un élément multimédia de sortie qui contient l’élément multimédia encodé.
* Ajout d’un gestionnaire d’événements pour vérifier la progression de la tâche.
* Envoyez le travail.

#### <a name="create-and-configure-a-visual-studio-project"></a>Créer et configurer un projet Visual Studio

Configurez votre environnement de développement et ajoutez des informations de connexion au fichier app.config selon la procédure décrite dans l’article [Développement Media Services avec .NET](media-services-dotnet-how-to-use.md). 

#### <a name="example"></a>Exemple 

```csharp
using System;
using System.Linq;
using System.Configuration;
using System.IO;
using System.Threading;
using Microsoft.WindowsAzure.MediaServices.Client;

namespace MediaEncoderStandardSample
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

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);


            // Get an uploaded asset.
            var asset = _context.Assets.FirstOrDefault();

            // Encode and generate the output using the "Adaptive Streaming" preset.
            EncodeToAdaptiveBitrateMP4Set(asset);

            Console.ReadLine();
        }

        static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset asset)
        {
            // Declare a new job.
            IJob job = _context.Jobs.Create("Media Encoder Standard Job");
            // Get a media processor reference, and pass to it the name of the 
            // processor to use for the specific task.
            IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

            // Create a task with the encoding details, using a string preset.
            // In this case "Adaptive Streaming" preset is used.
            ITask task = job.Tasks.AddNew("My encoding task",
                processor,
                "Adaptive Streaming",
                TaskOptions.None);

            // Specify the input asset to be encoded.
            task.InputAssets.Add(asset);
            // Add an output asset to contain the results of the job. 
            // This output is specified as AssetCreationOptions.None, which 
            // means the output asset is not encrypted. 
            task.OutputAssets.AddNew("Output asset",
                AssetCreationOptions.None);

            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
            job.Submit();
            job.GetExecutionProgressTask(CancellationToken.None).Wait();

            return job.OutputMediaAssets[0];
        }

        private static void JobStateChanged(object sender, JobStateChangedEventArgs e)
        {
            Console.WriteLine("Job state changed event:");
            Console.WriteLine("  Previous state: " + e.PreviousState);
            Console.WriteLine("  Current state: " + e.CurrentState);
            switch (e.CurrentState)
            {
                case JobState.Finished:
                    Console.WriteLine();
                    Console.WriteLine("Job is finished. Please wait while local tasks or downloads complete...");
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

## <a name="advanced-encoding-features-to-explore"></a>Fonctionnalités d’encodage avancées à découvrir
* [Procédure de génération de miniatures](media-services-dotnet-generate-thumbnail-with-mes.md)
* [Génération de miniatures pendant l’encodage](media-services-dotnet-generate-thumbnail-with-mes.md#example-of-generating-a-thumbnail-while-encoding)
* [Rognage de vidéos pendant l’encodage](media-services-crop-video.md)
* [Personnalisation des présélections d’encodage](media-services-custom-mes-presets-with-dotnet.md)
* [Superposition ou filigrane d’une vidéo avec une image](media-services-advanced-encoding-with-mes.md#overlay)

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Étapes suivantes
[Comment générer une miniature à l’aide de Media Encoder Standard avec .NET](media-services-dotnet-generate-thumbnail-with-mes.md)
[Vue d’ensemble du codage Media Services](media-services-encode-asset.md)