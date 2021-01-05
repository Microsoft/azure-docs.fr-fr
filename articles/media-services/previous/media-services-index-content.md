---
title: Indexation de fichiers multimédias avec Azure Media Indexer
description: Azure Media Indexer permet de rendre le contenu de vos fichiers multimédias consultable et de générer une transcription en texte intégral de sous-titrages et de mots-clés. Cette rubrique explique comment utiliser Media Indexer.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 827a56b2-58a5-4044-8d5c-3e5356488271
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 10/21/2020
ms.author: inhenkel
ms.reviewer: johndeu
ms.custom: devx-track-csharp
ms.openlocfilehash: 12a4099a6a3f0d06dfbb40171970c7386c3e4d22
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518838"
---
# <a name="indexing-media-files-with-azure-media-indexer"></a>Indexation de fichiers multimédias avec Azure Media Indexer

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!IMPORTANT]
> Il est recommandé que les clients effectuent une migration de la version 1 d’Indexer à la version 2 à l’aide du [mode de base Media Services V3 AudioAnalyzerPreset](../latest/analyzing-video-audio-files-concept.md). Les processeurs multimédias [Azure Media Indexer](media-services-index-content.md) et [Azure Media Indexer 2 Preview](./legacy-components.md) sont en phase de mise hors service. Pour connaître les dates de mise hors service, consultez la rubrique [Composants hérités](legacy-components.md).

Azure Media Indexer permet de rendre le contenu de vos fichiers multimédias consultable et de générer une transcription en texte intégral de sous-titrages et de mots-clés. Vous pouvez traiter un fichier multimédia ou plusieurs dans un lot.  

Lorsque vous indexez un contenu, veillez à utiliser des fichiers multimédias contenant des paroles nettes (sans musique de fond, bruit, effets ni sifflement de microphone). Voici quelques exemples de contenu approprié : des réunions, des conférences ou des présentations enregistrées. Le contenu suivant peut ne pas convenir à l’indexation : des films, des émissions de télévision, des fichiers avec du son et des effets sonores mélangés, du contenu mal enregistré avec un bruit de fond (sifflement).

Une tâche d’indexation peut générer les sorties suivantes :

* Fichiers de sous-titres codés dans les formats suivants : **TTML** et **WebVTT** .
  
    Les fichiers de sous-titres codés incluent une balise appelée Recognizability, qui note une tâche d’indexation en fonction de la possibilité de reconnaître les mots de la vidéo source.  Vous pouvez utiliser la valeur de Recognizability pour filtrer les fichiers de sortie en fonction de leur usage. Un faible score sous-entend de mauvais résultats d’indexation en raison de la qualité audio.
* Un fichier de mot-clé(XML).

Cet article explique comment créer des travaux d’indexation pour **indexer un élément multimédia** et **indexer plusieurs fichiers** .

## <a name="using-configuration-and-manifest-files-for-indexing-tasks"></a>Utilisation des fichiers de configuration et manifeste pour l’indexation des tâches
Vous pouvez définir plus de détails pour vos tâches d’indexation en utilisant une configuration de tâche. Par exemple, vous pouvez spécifier les métadonnées à utiliser pour votre fichier multimédia. Ces métadonnées sont utilisées par le moteur de langue pour développer son vocabulaire et améliorent considérablement la précision de la reconnaissance vocale.  Vous pouvez également spécifier vos fichiers de sortie souhaités.

Vous pouvez également traiter plusieurs fichiers multimédias à la fois à l’aide d’un fichier manifeste.

Pour plus d’informations, consultez [Présélection de tâches pour Azure Media Indexer](./legacy-components.md).

## <a name="index-an-asset"></a>Indexation d’une ressource
La méthode suivante télécharge un fichier multimédia en tant que ressource et crée une tâche pour indexer la ressource.

Si aucun fichier de configuration n’est spécifié, le fichier multimédia est indexé avec tous les paramètres par défaut.

```csharp
    static bool RunIndexingJob(string inputMediaFilePath, string outputFolder, string configurationFile = "")
    {
        // Create an asset and upload the input media file to storage.
        IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
            "My Indexing Input Asset",
            AssetCreationOptions.None);

        // Declare a new job.
        IJob job = _context.Jobs.Create("My Indexing Job");

        // Get a reference to the Azure Media Indexer.
        string MediaProcessorName = "Azure Media Indexer";
        IMediaProcessor processor = GetLatestMediaProcessorByName(MediaProcessorName);

        // Read configuration from file if specified.
        string configuration = string.IsNullOrEmpty(configurationFile) ? "" : File.ReadAllText(configurationFile);

        // Create a task with the encoding details, using a string preset.
        ITask task = job.Tasks.AddNew("My Indexing Task",
            processor,
            configuration,
            TaskOptions.None);

        // Specify the input asset to be indexed.
        task.InputAssets.Add(asset);

        // Add an output asset to contain the results of the job.
        task.OutputAssets.AddNew("My Indexing Output Asset", AssetCreationOptions.None);

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
            Console.WriteLine("Exiting method due to job error.");
            return false;
        }

        // Download the job outputs.
        DownloadAsset(task.OutputAssets.First(), outputFolder);

        return true;
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
```

<!-- __ -->
### <a name="output-files"></a><a id="output_files"></a>Fichiers de sortie
Par défaut, une tâche d’indexation génère les fichiers de sortie suivants. Les fichiers sont stockés dans la première ressource de sortie.

Lorsqu’il existe plusieurs fichiers multimédias d’entrée, l’indexeur génère un fichier manifeste pour les sorties du travail, nommé « JobResult.txt ». Pour chaque fichier multimédia d’entrée, les fichiers TTML, WebVTT et de mots clés qui en résultent sont numérotés de façon séquentielle et nommés avec l’« Alias ».

| Nom de fichier | Description |
| --- | --- |
| **InputFileName.ttml**<br/>**InputFileName.vtt** |Fichiers de sous-titres (CC) aux formats TTML et WebVTT.<br/><br/>Ils permettent de rendre un fichier audio et vidéo accessible aux malentendants.<br/><br/>Les fichiers de sous-titres codés incluent une balise appelée <b>Recognizability</b>, qui note une tâche d’indexation en fonction de la possibilité de reconnaître les mots de la vidéo source.  Vous pouvez utiliser la valeur de <b>Recognizability</b> pour filtrer les fichiers de sortie en fonction de leur usage. Un faible score sous-entend de mauvais résultats d’indexation en raison de la qualité audio. |
| **InputFileName.kw.xml<br/>InputFileName.info** |Fichiers de mots clés et d’informations. <br/><br/>Un fichier de mot-clé est un fichier XML qui contient les mots clés extraits à partir du contenu de la reconnaissance vocale, avec les informations relatives à la fréquence et à la référence. <br/><br/>Un fichier d’informations est un fichier de texte brut qui contient des informations précises sur chaque terme reconnu. La première ligne est spéciale et contient le score Recognizability. Chaque ligne suivante est une liste des données suivantes séparées par des tabulations : heure de début, heure de fin, mot/expression, fiabilité. Les heures sont exprimées en secondes et la fiabilité est exprimée comme un nombre compris entre 0 et 1. <br/><br/>Exemple de ligne : "1.20    1.45    word    0.67" <br/><br/>Ces fichiers peuvent être utilisés à différentes fins, par exemple pour effectuer une analyse vocale ou être présentés à des moteurs de recherche comme Bing, Google ou Microsoft SharePoint pour rendre les fichiers multimédia plus détectables, ou même pour fournir des publicités plus pertinentes. |
| **JobResult.txt** |Manifeste de sortie, présent uniquement en cas d’indexation de plusieurs fichiers, contenant les informations suivantes :<br/><br/><table border="1"><tr><th>InputFile</th><th>Alias</th><th>MediaLength</th><th>Error</th></tr><tr><td>a.mp4</td><td>Media_1</td><td>300</td><td>0</td></tr><tr><td>b.mp4</td><td>Media_2</td><td>0</td><td>3000</td></tr><tr><td>c.mp4</td><td>Media_3</td><td>600</td><td>0</td></tr></table><br/> |

Si tous les fichiers multimédias d’entrée ne sont pas correctement indexés, le travail d’indexation échoue avec le code d’erreur 4000. Pour plus d’informations, consultez les [codes d’erreur](#error_codes).

## <a name="index-multiple-files"></a>indexer plusieurs fichiers
La méthode suivante télécharge plusieurs fichiers multimédias en tant que ressource et crée une tâche pour indexer tous ces fichiers en lot.

Un fichier manifeste avec l’extension .lst est créé et chargé dans la ressource. Le fichier manifeste contient la liste de tous les fichiers de ressources. Pour plus d’informations, consultez [Présélection de tâches pour Azure Media Indexer](./legacy-components.md).

```csharp
    static bool RunBatchIndexingJob(string[] inputMediaFiles, string outputFolder)
    {
        // Create an asset and upload to storage.
        IAsset asset = CreateAssetAndUploadMultipleFiles(inputMediaFiles,
            "My Indexing Input Asset - Batch Mode",
            AssetCreationOptions.None);

        // Create a manifest file that contains all the asset file names and upload to storage.
        string manifestFile = "input.lst";            
        File.WriteAllLines(manifestFile, asset.AssetFiles.Select(f => f.Name).ToArray());
        var assetFile = asset.AssetFiles.Create(Path.GetFileName(manifestFile));
        assetFile.Upload(manifestFile);

        // Declare a new job.
        IJob job = _context.Jobs.Create("My Indexing Job - Batch Mode");

        // Get a reference to the Azure Media Indexer.
        string MediaProcessorName = "Azure Media Indexer";
        IMediaProcessor processor = GetLatestMediaProcessorByName(MediaProcessorName);

        // Read configuration.
        string configuration = File.ReadAllText("batch.config");

        // Create a task with the encoding details, using a string preset.
        ITask task = job.Tasks.AddNew("My Indexing Task - Batch Mode",
            processor,
            configuration,
            TaskOptions.None);

        // Specify the input asset to be indexed.
        task.InputAssets.Add(asset);

        // Add an output asset to contain the results of the job.
        task.OutputAssets.AddNew("My Indexing Output Asset - Batch Mode", AssetCreationOptions.None);

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
            Console.WriteLine("Exiting method due to job error.");
            return false;
        }

        // Download the job outputs.
        DownloadAsset(task.OutputAssets.First(), outputFolder);

        return true;
    }

    private static IAsset CreateAssetAndUploadMultipleFiles(string[] filePaths, string assetName, AssetCreationOptions options)
    {
        IAsset asset = _context.Assets.Create(assetName, options);

        foreach (string filePath in filePaths)
        {
            var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
            assetFile.Upload(filePath);
        }

        return asset;
    }
```

### <a name="partially-succeeded-job"></a>Tâche partiellement réussie
Si tous les fichiers multimédias d’entrée ne sont pas correctement indexés, la tâche d’indexation échoue avec le code d’erreur 4000. Pour plus d’informations, consultez les [codes d’erreur](#error_codes).

Les mêmes sorties que pour des tâches réussies sont générées. Vous pouvez consulter le fichier manifeste de sortie pour savoir quels fichiers d’entrée ont échoué, en fonction des valeurs de la colonne d’erreur. Pour les fichiers d’entrée ayant échoué, les fichiers TTML, WebVTT et de mots clés NE sont PAS générés.

### <a name="task-preset-for-azure-media-indexer"></a><a id="preset"></a> Présélection de tâches pour l’Indexeur multimédia Azure
Le traitement de l’Indexeur multimédia Azure peut être personnalisé par une tâche facultative prédéfinie à côté de la tâche.  La section suivante décrit le format de ce fichier xml de configuration.

| Nom | Require | Description |
| --- | --- | --- |
| **input** |false |Fichiers de ressources que vous souhaitez indexer.</p><p>Azure Media Indexer prend en charge les formats de fichiers multimédias suivants : MP4, WMV, MP3, M4A, WMA, AAC, WAV.</p><p>Vous pouvez spécifier les noms de fichier dans l’attribut **name** ou **list** attribut de l’élément **d’entrée** (comme indiqué ci-dessous). Si vous ne spécifiez pas de fichier de ressource à indexer, le fichier principal est sélectionné. Si aucun fichier principal n’est défini, le premier fichier de ressource d’entrée est indexé.</p><p>Pour spécifier explicitement le nom du fichier multimédia, effectuez ceci :<br/>`<input name="TestFile.wmv">`<br/><br/>Vous pouvez également indexer plusieurs fichiers de ressources à la fois (jusqu'à 10 fichiers). Pour ce faire :<br/><br/><ol class="ordered"><li><p>Créez un fichier texte (fichier manifeste) et affectez-lui une extension .lst. </p></li><li><p>Ajoutez une liste de tous les noms de fichier d’élément multimédia à ce fichier de manifeste. </p></li><li><p>Ajoutez (chargez) le fichier manifeste dans l’élément multimédia.  </p></li><li><p>Spécifiez le nom du fichier manifeste dans l’attribut list de l’entrée.<br/>`<input list="input.lst">`</li></ol><br/><br/>Remarque : si vous ajoutez plus de 10 fichiers dans le fichier manifeste, la tâche d’indexation échoue avec le code d’erreur 2006. |
| **métadonnées** |false |Métadonnées pour les fichiers de ressources actives utilisés pour l’Adaptation du vocabulaire.  Il est utile préparer l’indexeur à reconnaître les mots de vocabulaire non standard tels que des noms propres.<br/>`<metadata key="..." value="..."/>` <br/><br/>Vous pouvez fournir des **valeurs** pour les **clés** prédéfinies. Les clés actuellement prises en charge sont les suivantes :<br/><br/>« titre » et « description ». Elles servent à l’adaptation du vocabulaire pour sélectionner le modèle de langue pour votre travail et améliorer la précision de la reconnaissance vocale.  Les valeurs amorcent des recherches sur Internet pour rechercher des documents pertinents du point de vue contextuel en utilisant le contenu pour étoffer le dictionnaire interne pour la durée de la tâche d’indexation.<br/>`<metadata key="title" value="[Title of the media file]" />`<br/>`<metadata key="description" value="[Description of the media file] />"` |
| **fonctionnalités** <br/><br/> ajoutées dans la version 1.2. Actuellement, la seule fonctionnalité prise en charge est la reconnaissance vocale (« ASR »). |false |La fonctionnalité de reconnaissance vocale a les clés de paramètres suivantes :<table><tr><th><p>Clé</p></th>        <th><p>Description</p></th><th><p>Valeur d'exemple</p></th></tr><tr><td><p>Language</p></td><td><p>La langue naturelle à reconnaître dans le fichier multimédia.</p></td><td><p>Anglais, espagnol</p></td></tr><tr><td><p>CaptionFormats</p></td><td><p>une liste délimitée par des points-virgules des formats de sous-titre souhaités (le cas échéant)</p></td><td><p>ttml;webvtt</p></td></tr><tr><td><p></p></td><td><p> </p></td><td><p>True; False</p></td></tr><tr><td><p>GenerateKeywords</p></td><td><p>Indicateur booléen indiquant si un fichier XML de mot-clé est requis ou non.</p></td><td><p>True; False. </p></td></tr><tr><td><p>ForceFullCaption</p></td><td><p>Indicateur booléen spécifiant s’il faut forcer ou non les sous-titres complets (quel que soit le niveau de confiance).  </p><p>La valeur par défaut est false, auquel cas les mots et expressions dont le niveau de confiance est inférieur ou égal à 50 % sont omis dans les sous-titres finaux et remplacés par des points de suspension («... »).  Les points de suspension sont utiles pour le contrôle de la qualité et l’audit des sous-titres.</p></td><td><p>True; False. </p></td></tr></table> |

### <a name="error-codes"></a><a id="error_codes"></a>Codes d’erreur
En cas d’erreur, Azure Media indexeur doit signaler un des codes d’erreur suivants :

| Code | Nom | Causes possibles |
| --- | --- | --- |
| 2000 |Configuration non valide |Configuration non valide |
| 2001 |Ressources d’entrée non valides |Ressources d’entrée manquantes ou vides. |
| 2002 |Manifeste non valide |Le manifeste est vide ou contient des éléments non valides. |
| 2003 |Impossible de télécharger le fichier multimédia |L’URL dans le fichier manifeste n’est pas valide. |
| 2004 |Protocole non pris en charge |Le protocole de l'URL du média n’est pas pris en charge. |
| 2005 |Type de fichier non pris en charge |Le type de fichier multimédia d’entrée n’est pas pris en charge. |
| 2006 |Trop de fichiers d’entrée |Le manifeste d’entrée comporte plus de 10 fichiers. |
| 3000 |Impossible de décoder le fichier multimédia |Codec de média non pris en charge  <br/>or<br/> Fichier multimédia endommagé <br/>or<br/> Il n’y a aucun flux audio dans le fichier d’entrée. |
| 4000 |Indexation en lot partiellement réussie |Impossible d'indexer certains des fichiers multimédias d’entrée. Pour plus d’informations, consultez <a href="#output_files">Fichiers de sortie</a>. |
| other |Erreurs internes |Veuillez contacter l’équipe du support technique. indexer@microsoft.com |

## <a name="supported-languages"></a><a id="supported_languages"></a>Langues prises en charge
Les langues prises en charge pour le moment sont l’anglais et l’espagnol.  

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>Liens connexes
[Vue d’ensemble d’Azure Media Services Analytics](./legacy-components.md)

[Indexation de fichiers multimédias avec Azure Media Indexer 2 Preview](./legacy-components.md)