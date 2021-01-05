---
title: Éditer les visages avec Azure Media Analytics | Microsoft Docs
description: Azure Media Redactor est un processeur multimédia Azure Media Analytics qui offre une fonctionnalité scalable d’édition des visages dans le cloud. Cet article explique comment éditer des visages avec Azure Media Analytics.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 11/17/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 1a7bd36a6e3f3cc5b785745fc51f0aede3b47b74
ms.sourcegitcommit: 7e97ae405c1c6c8ac63850e1b88cf9c9c82372da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97803304"
---
# <a name="redact-faces-with-azure-media-analytics"></a>Éditer les visages avec Azure Media Analytique

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Vue d’ensemble

**Azure Media Redactor** est un processeur multimédia [Azure Media Analytics](./legacy-components.md) qui offre la rédaction de face évolutive dans le cloud. La rédaction de face vous permet de modifier votre vidéo afin de flouter les visages des individus sélectionnés. Vous souhaitez peut-être utiliser le service de rédaction de face dans des scénarios de média et de sécurité publics. Quelques minutes de séquences vidéo contenant plusieurs visages peuvent nécessiter des heures de traitement manuel, mais avec ce service, le processus de rédaction de face ne nécessitera que quelques étapes simples.

Cet article apporte des précisions sur **Azure Media Redactor** et illustre son utilisation avec le kit SDK Media Services pour .NET.

## <a name="face-redaction-modes"></a>Modes de rédaction de face

La rédaction de face fonctionne en détectant les visages dans chaque image de la vidéo et en suivant l’objet de visage à la fois vers l’avant et l’arrière dans le temps, afin que la même personne puisse être floutée à partir d’autres angles également. Le processus d’édition automatisée est complexe et ne produit pas toujours le résultat souhaité à 100 %, c’est la raison pour laquelle Media Analytics vous fournit deux méthodes pour modifier le résultat final.

En plus d’un mode entièrement automatique, il existe un flux de travail en deux passes qui permet la sélection/désélection des visages trouvés par le biais d’une liste d’ID. En outre, pour rendre arbitraires les réglages par image, le processeur multimédia utilise un fichier de métadonnées au format JSON. Ce flux de travail est divisé en modes **Analyser** et **Rédiger**. Vous pouvez combiner les deux modes en une seule passe qui exécute les deux tâches dans un travail ; ce mode est appelé **Combiné**.

### <a name="combined-mode"></a>Mode Combiné

Un mp4 rédigé automatiquement est généré sans entrée manuelle.

| Étape | Nom de fichier | Notes |
| --- | --- | --- |
| Élément multimédia d’entrée |foo.bar |Vidéo au format WMV, MOV ou MP4 |
| Configuration d’entrée |Job configuration preset |{'version':'1.0', 'options': {'mode':'combined'}} |
| Élément multimédia de sortie |foo_redacted.mp4 |Vidéo avec flou appliqué |

### <a name="analyze-mode"></a>Mode Analyser

La passe **Analyser** du flux de travail en deux passes accepte une entrée vidéo et produit un fichier JSON d’emplacements de visage et des images jpg de chaque visage détecté.

| Étape | Nom de fichier | Notes |
| --- | --- | --- |
| Élément multimédia d’entrée |foo.bar |Vidéo au format WMV, MPV ou MP4 |
| Configuration d’entrée |Job configuration preset |{'version':'1.0', 'options': {'mode':'analyze'}} |
| Élément multimédia de sortie |foo_annotations.json |Données d’annotation des emplacements de visage au format JSON. Cela peut être modifié par l’utilisateur pour changer les cadres de limitation du flou. Voir l’exemple ci-dessous. |
| Élément multimédia de sortie |foo_thumb%06d.jpg [foo_thumb000001.jpg, foo_thumb000002.jpg] |Une image jpg rognée de chaque visage détecté, où le nombre indique l’ID d’étiquette du visage |

#### <a name="output-example"></a>Exemple de sortie

```json
{
  "version": 1,
  "timescale": 24000,
  "offset": 0,
  "framerate": 23.976,
  "width": 1280,
  "height": 720,
  "fragments": [
    {
      "start": 0,
      "duration": 48048,
      "interval": 1001,
      "events": [
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [
          {
            "index": 13,
            "id": 1138,
            "x": 0.29537,
            "y": -0.18987,
            "width": 0.36239,
            "height": 0.80335
          },
          {
            "index": 13,
            "id": 2028,
            "x": 0.60427,
            "y": 0.16098,
            "width": 0.26958,
            "height": 0.57943
          }
        ],

    ... truncated
```

### <a name="redact-mode"></a>Mode Rédiger

La deuxième passe du flux de travail prend un plus grand nombre d’entrées qui doivent être combinées en un seul élément multimédia.

Cela inclut une liste des ID à flouter, la vidéo d’origine et les annotations JSON. Ce mode utilise les annotations pour appliquer le flou sur la vidéo d’entrée.

La sortie de la passe Analyser n’inclut pas la vidéo d’origine. La vidéo doit être chargée dans l’élément multimédia d’entrée pour la tâche en mode Rédiger et sélectionnée comme fichier principal.

| Étape | Nom de fichier | Notes |
| --- | --- | --- |
| Élément multimédia d’entrée |foo.bar |Vidéo au format WMV, MPV ou MP4. Même vidéo que celle de l’étape 1. |
| Élément multimédia d’entrée |foo_annotations.json |Fichier de métadonnées d’annotations de la première phase, avec des modifications facultatives. |
| Élément multimédia d’entrée |foo_IDList.txt (facultatif) |Nouvelle liste facultative séparée par des lignes des ID de visage à traiter. Si ce champ est laissé vide, tous les visages sont floutés. |
| Configuration d’entrée |Job configuration preset |{'version':'1.0', 'options': {'mode':'redact'}} |
| Élément multimédia de sortie |foo_redacted.mp4 |Vidéo avec flou appliqué en fonction des annotations |

#### <a name="example-output"></a>Exemple de sortie

Il s’agit de la sortie à partir d’une liste d’ID avec un ID sélectionné.

Exemple : foo_IDList.txt

```
1
2
3
```

## <a name="blur-types"></a>Types de flou

En mode **Combiné** ou **Rédiger**, 5 modes de flou sont disponibles par le biais de la configuration d'entrée JSON : **Faible**, **Med** (Moyen), **Élevé**, **Box** et **Noir**. Par défaut, **Med** (Moyen) est utilisé.

Vous trouverez des exemples de types de flou ci-dessous.

### <a name="example-json"></a>Exemple JSON

```json
{
    'version':'1.0',
    'options': {
        'Mode': 'Combined',
        'BlurType': 'High'
    }
}
```

#### <a name="low"></a>Faible

![Faible](./media/media-services-face-redaction/blur1.png)

#### <a name="med"></a>Med (Moyen)

![Med (Moyen)](./media/media-services-face-redaction/blur2.png)

#### <a name="high"></a>Élevé

![Élevé](./media/media-services-face-redaction/blur3.png)

#### <a name="box"></a>Box

![Box](./media/media-services-face-redaction/blur4.png)

#### <a name="black"></a>Noir

![Noir](./media/media-services-face-redaction/blur5.png)

## <a name="elements-of-the-output-json-file"></a>Éléments du fichier de sortie JSON

Le processeur multimédia de rédaction permet une détection d’emplacement et un suivi de visage très précis ; il peut détecter jusqu’à 64 visages humains dans une séquence vidéo. Les visages filmés de face donnent les meilleurs résultats ; les visages filmés de côté ou les visages de taille réduite (24 x 24 pixels ou moins) posent plus de problèmes.

[!INCLUDE [media-services-analytics-output-json](../../../includes/media-services-analytics-output-json.md)]

## <a name="net-sample-code"></a>Exemple de code .NET

Le programme suivant montre comment effectuer les tâches suivantes :

1. Créer un élément multimédia et charger un fichier multimédia dans l’élément multimédia.
2. Créer un travail avec une tâche d’édition de visages basée sur un fichier de configuration qui contient la présélection JSON suivante : 

    ```json
            {
                'version':'1.0',
                'options': {
                    'mode':'combined'
                }
            }
    ```

3. Télécharger les fichiers JSON de sortie.

### <a name="create-and-configure-a-visual-studio-project"></a>Créer et configurer un projet Visual Studio

Configurez votre environnement de développement et ajoutez des informations de connexion au fichier app.config selon la procédure décrite dans l’article [Développement Media Services avec .NET](media-services-dotnet-how-to-use.md).

#### <a name="example"></a>Exemple

```csharp
using System;
using System.Configuration;
using System.IO;
using System.Linq;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Threading;
using System.Threading.Tasks;

namespace FaceRedaction
{
    class Program
    {
        // Read values from the App.config file.
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

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            // Run the FaceRedaction job.
            var asset = RunFaceRedactionJob(@"C:\supportFiles\FaceRedaction\SomeFootage.mp4",
                        @"C:\supportFiles\FaceRedaction\config.json");

            // Download the job output asset.
            DownloadAsset(asset, @"C:\supportFiles\FaceRedaction\Output");
        }

        static IAsset RunFaceRedactionJob(string inputMediaFilePath, string configurationFile)
        {
            // Create an asset and upload the input media file to storage.
            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
            "My Face Redaction Input Asset",
            AssetCreationOptions.None);

            // Declare a new job.
            IJob job = _context.Jobs.Create("My Face Redaction Job");

            // Get a reference to Azure Media Redactor.
            string MediaProcessorName = "Azure Media Redactor";

            var processor = GetLatestMediaProcessorByName(MediaProcessorName);

            // Read configuration from the specified file.
            string configuration = File.ReadAllText(configurationFile);

            // Create a task with the encoding details, using a string preset.
            ITask task = job.Tasks.AddNew("My Face Redaction Task",
            processor,
            configuration,
            TaskOptions.None);

            // Specify the input asset.
            task.InputAssets.Add(asset);

            // Add an output asset to contain the results of the job.
            task.OutputAssets.AddNew("My Face Redaction Output Asset", AssetCreationOptions.None);

            // Use the following event handler to check job progress.  
            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);

            // Launch the job.
            job.Submit();

            // Check job execution and wait for job to finish.
            Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);

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
                return null;
            }

            return job.OutputMediaAssets[0];
        }

        static IAsset CreateAssetAndUploadSingleFile(string filePath, string assetName, AssetCreationOptions options)
        {
            IAsset asset = _context.Assets.Create(assetName, options);

            var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
            assetFile.Upload(filePath);

            return asset;
        }

        static void DownloadAsset(IAsset asset, string outputDirectory)
        {
            foreach (IAssetFile file in asset.AssetFiles)
            {
                file.Download(Path.Combine(outputDirectory, file.Name));
            }
        }

        static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
        {
            var processor = _context.MediaProcessors
            .Where(p => p.Name == mediaProcessorName)
            .ToList()
            .OrderBy(p => new Version(p.Version))
            .LastOrDefault();

            if (processor == null)
                throw new ArgumentException(string.Format("Unknown media processor",
                                       mediaProcessorName));

            return processor;
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
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>Liens connexes

[Vue d’ensemble d’Azure Media Services Analytics](./legacy-components.md)

[Démonstrations Azure Media Analytics](http://amslabs.azurewebsites.net/demos/Analytics.html)
