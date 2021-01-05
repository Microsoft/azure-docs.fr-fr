---
title: Utilisation de Media Services avec .NET | Microsoft Docs
description: Ce didacticiel vous présente les étapes d’implémentation d’une application de diffusion de contenu à la demande avec Azure Media Services pour .NET.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 388b8928-9aa9-46b1-b60a-a918da75bd7b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: bbbf924ea950c5ba281be430190370ae77734b7a
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184353"
---
# <a name="get-started-with-delivering-content-on-demand-using-net-sdk"></a>Prendre en main la diffusion de contenus à la demande à l’aide du Kit de développement logiciel (SDK) .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]  

[!INCLUDE [media-services-selector-get-started](../../../includes/media-services-selector-get-started.md)]

Ce didacticiel explique comment implémenter un service de base de diffusion de contenu vidéo à la demande (VoD) avec l’application Azure Media Services (AMS) à l’aide du Kit de développement logiciel (SDK) .NET Azure Media Services.

## <a name="prerequisites"></a>Prérequis

Les éléments suivants sont requis pour suivre le didacticiel :

* Un compte Azure. Pour plus d’informations, consultez [Essai gratuit Azure](https://azure.microsoft.com/pricing/free-trial/).
* Un compte Media Services. Pour créer un compte Media Services, consultez [Création d’un compte Media Services](media-services-portal-create-account.md).
* .NET Framework 4.0 ou version ultérieure.
* Visual Studio.

Ce didacticiel comprend les tâches suivantes :

1. Démarrer les points de terminaison de streaming (à l’aide du portail Azure).
2. Créer et configurer un projet Visual Studio
3. Se connecter au compte Media Services.
2. Télécharger un fichier vidéo.
3. Encoder le fichier source en un ensemble de fichiers MP4 à débit adaptatif.
4. Publier l'élément multimédia et obtenir les URL de diffusion et de téléchargement progressif.  
5. Lire votre contenu.

## <a name="overview"></a>Vue d’ensemble
Ce didacticiel vous présente les étapes d'implémentation d'une application de diffusion de contenu vidéo à la demande (VoD) avec le Kit de développement logiciel (SDK) Azure Media Services (AMS) pour .NET.

Il présente le workflow Media Services de base et les objets et tâches de programmation les plus courants requis pour le développement Media Services. À la fin de ce didacticiel, vous pourrez lire en continu ou télécharger de façon progressive un exemple de fichier multimédia que vous aurez chargé, encodé et téléchargé.

### <a name="ams-model"></a>Modèle d’AMS

L’image suivante illustre certains des objets couramment utilisés lors du développement d’applications VoD par rapport au modèle OData de Media Services.

Cliquez sur l’image pour l’afficher en plein écran.  

[![Diagramme montrant certains des objets les plus couramment utilisés dans le modèle de données d’objet Azure Media Services pour le développement d’applications de vidéo à la demande.](./media/media-services-dotnet-get-started/media-services-overview-object-model-small.png)](./media/media-services-dotnet-get-started/media-services-overview-object-model.png#lightbox)

Vous pouvez afficher l’ensemble du modèle [ici](https://media.windows.net/API/$metadata?api-version=2.15).  

## <a name="start-streaming-endpoints-using-the-azure-portal"></a>Démarrer les points de terminaison de streaming à l’aide du portail Azure

Lorsque vous utilisez Azure Media Services, la diffusion de vidéos en continu à débit binaire adaptatif constitue l’un des scénarios les plus courants. Media Services assure l’empaquetage dynamique, qui vous permet de distribuer juste-à-temps un contenu encodé en MP4 à débit adaptatif dans un format de diffusion en continu pris en charge par Media Services (MPEG DASH, HLS, Smooth Streaming), sans qu’il vous soit nécessaire de stocker des versions pré-empaquetées de chacun de ces formats.

>[!NOTE]
>Une fois votre compte AMS créé, un point de terminaison de diffusion continue **par défaut** est ajouté à l’état **Arrêté**. Pour démarrer la diffusion en continu de votre contenu et tirer parti de l’empaquetage et du chiffrement dynamiques, le point de terminaison de streaming à partir duquel vous souhaitez diffuser du contenu doit se trouver à l’état **En cours d’exécution**.

Pour démarrer le point de terminaison de streaming, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans la fenêtre Paramètres, cliquez sur Points de terminaison de streaming.
3. Cliquez sur le point de terminaison de diffusion en continu par défaut.

    La fenêtre DEFAULT STREAMING ENDPOINT DETAILS (DÉTAILS DU POINT DE TERMINAISON DE STREAMING PAR DÉFAUT) s’affiche.

4. Cliquez sur l’icône de démarrage.
5. Cliquez sur le bouton Enregistrer pour enregistrer vos modifications.

## <a name="create-and-configure-a-visual-studio-project"></a>Créer et configurer un projet Visual Studio

1. Configurez votre environnement de développement et ajoutez des informations de connexion au fichier app.config selon la procédure décrite dans l’article [Développement Media Services avec .NET](media-services-dotnet-how-to-use.md). 
2. Créez un dossier (à tout emplacement sur votre disque local) et copiez-y le fichier .mp4 à encoder et à diffuser en continu ou à télécharger. Dans cet exemple, le chemin d'accès « C:\VideoFiles » est utilisé.

## <a name="connect-to-the-media-services-account"></a>Se connecter au compte Media Services

Lorsque vous utilisez Media Services avec .NET, vous devez utiliser la classe **CloudMediaContext** pour la plupart des tâches de programmation Media Services : connexion au compte Media Services, création, mise à jour, accès et suppression des objets suivants : éléments multimédia, fichiers multimédias, travaux, stratégies d'accès, localisateurs, etc.

Remplacez la classe Program par défaut avec le code suivant : Le code montre comment lire les valeurs de connexion à partir du fichier App.config et comment créer l’objet **CloudMediaContext** pour se connecter à Media Services. Pour plus d’informations, consultez [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md).

Veillez à mettre à jour le nom du fichier et le chemin d’accès où se trouve votre fichier multimédia.

La fonction **Main** appelle des méthodes qui seront définies ultérieurement dans cette section.

> [!NOTE]
> Vous obtiendrez des erreurs de compilation tant que vous n’aurez pas ajouté de définitions pour toutes les fonctions qui sont définies plus loin dans cet article.

```csharp
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

        private static CloudMediaContext _context = null;

        static void Main(string[] args)
        {
        try
        {
            AzureAdTokenCredentials tokenCredentials = 
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            // Add calls to methods defined in this section.
            // Make sure to update the file name and path to where you have your media file.
            IAsset inputAsset =
            UploadFile(@"C:\VideoFiles\BigBuckBunny.mp4", AssetCreationOptions.None);

            IAsset encodedAsset =
            EncodeToAdaptiveBitrateMP4s(inputAsset, AssetCreationOptions.None);

            PublishAssetGetURLs(encodedAsset);
        }
        catch (Exception exception)
        {
            // Parse the XML error message in the Media Services response and create a new
            // exception with its content.
            exception = MediaServicesExceptionParser.Parse(exception);

            Console.Error.WriteLine(exception.Message);
        }
        finally
        {
            Console.ReadLine();
        }
        }
```

## <a name="create-a-new-asset-and-upload-a-video-file"></a>Créer un nouvel élément et charger un fichier vidéo

Dans Media Services, vous téléchargez (ou réceptionnez) vos fichiers numériques dans un élément multimédia. L’entité **Asset** peut contenir des fichiers vidéo et audio, des images, des collections de miniatures, des pistes textuelles et des légendes (et les métadonnées concernant ces fichiers).  Une fois les fichiers téléchargés, votre contenu est stocké en toute sécurité dans le cloud et peut faire l’objet d’un traitement et d’une diffusion en continu. Les fichiers de l'élément multimédia sont appelés **fichiers d'élément multimédia**.

La méthode **UploadFile** définie ci-dessous appelle **CreateFromFile** (défini dans les extensions du Kit de développement logiciel (SDK) .NET). **CreateFromFile** crée un nouvel élément multimédia dans lequel le fichier source spécifié est téléchargé.

La méthode **CreateFromFile** prend **AssetCreationOptions**, ce qui vous permet de spécifier une des options de création de ressource suivantes :

* **None** : aucun chiffrement. Il s’agit de la valeur par défaut. À noter que quand vous utilisez cette option, votre contenu n'est pas protégé pendant le transit ou le repos dans le stockage.
  Si vous prévoyez de fournir un MP4 sous forme de téléchargement progressif, utilisez cette option.
* **StorageEncrypted** : utilisez cette option pour chiffrer votre contenu localement à l’aide du chiffrement Advanced Encryption Standard (AES) 256 bits, qui est ensuite chargé sur Azure Storage, où il est stocké au repos sous forme chiffrée. Les éléments multimédias protégés par le chiffrement de stockage sont automatiquement déchiffrés et placés dans un système de fichiers chiffré avant d’être encodés, puis éventuellement rechiffrés avant d’être rechargés sous la forme d’un nouvel élément multimédia de sortie. Le principal cas d'utilisation du chiffrement de stockage concerne la sécurisation de fichiers multimédias d'entrée de haute qualité avec un chiffrement renforcé au repos sur le disque.
* **CommonEncryptionProtected** : utilisez cette option quand vous chargez du contenu qui a déjà été chiffré et protégé avec la DRM Common Encryption ou PlayReady (par exemple Smooth Streaming protégé avec la DRM PlayReady).
* **EnvelopeEncryptionProtected** : utilisez cette option lorsque vous téléchargez un contenu au format TLS chiffré avec AES. Notez que les fichiers doivent avoir été encodés et chiffrés par le gestionnaire de transformation Transform Manager.

La méthode **CreateFromFile** vous permet également de spécifier un rappel pour indiquer la progression du chargement du fichier.

Dans l’exemple suivant, nous spécifions **None** pour les options de l’élément multimédia.

Ajoutez la méthode suivante à la classe Program.

```csharp
    static public IAsset UploadFile(string fileName, AssetCreationOptions options)
    {
        IAsset inputAsset = _context.Assets.CreateFromFile(
            fileName,
            options,
            (af, p) =>
            {
                Console.WriteLine("Uploading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
            });

        Console.WriteLine("Asset {0} created.", inputAsset.Id);

        return inputAsset;
    }
```

## <a name="encode-the-source-file-into-a-set-of-adaptive-bitrate-mp4-files"></a>Encoder le fichier source en un ensemble de fichiers MP4 à débit adaptatif
Après avoir reçu des éléments multimédias dans Media Services, vous pouvez encoder un média, modifier le format de ce dernier, lui appliquer un filigrane, etc. avant de le livrer à des clients. Afin de garantir des performances et une disponibilité optimales, ces activités sont planifiées et exécutées dans de nombreuses instances de rôle en arrière-plan. Ces activités s’appellent des travaux et chaque Travail se compose de tâches atomiques qui effectuent le travail à proprement parler sur le fichier de ressource.

Comme mentionné précédemment, lorsque vous travaillez avec Azure Media Services, un des scénarios les plus courants est la diffusion de contenu à débit adaptatif à vos clients. Media Services peut empaqueter de façon dynamique un ensemble de fichiers MP4 à débit adaptatif dans l’un des formats suivants : HTTP Live Streaming (HLS), Smooth Streaming et MPEG DASH.

Pour tirer parti de l’empaquetage dynamique, vous devez encoder ou transcoder votre fichier mezzanine (source) en un ensemble de fichiers MP4 à débit adaptatif ou de fichiers Smooth Streaming à débit adaptatif.  

Le code suivant vous explique comment effectuer envoyer une tâche d'encodage. Le travail contient une tâche qui spécifie le fichier mezzanine à transcoder en un ensemble de MP4 à débit adaptatif à l’aide de **Media Encoder Standard**. Le code envoie la tâche et attend qu'elle soit terminée.

Une fois la tâche terminée, vous pourrez diffuser votre élément multimédia ou télécharger progressivement les fichiers MP4 qui ont été créés après le transcodage.

Ajoutez la méthode suivante à la classe Program.

```csharp
    static public IAsset EncodeToAdaptiveBitrateMP4s(IAsset asset, AssetCreationOptions options)
    {

        // Prepare a job with a single task to transcode the specified asset
        // into a multi-bitrate asset.

        IJob job = _context.Jobs.CreateWithSingleTask(
            "Media Encoder Standard",
            "Adaptive Streaming",
            asset,
            "Adaptive Bitrate MP4",
            options);

        Console.WriteLine("Submitting transcoding job...");


        // Submit the job and wait until it is completed.
        job.Submit();

        job = job.StartExecutionProgressTask(
            j =>
            {
                Console.WriteLine("Job state: {0}", j.State);
                Console.WriteLine("Job progress: {0:0.##}%", j.GetOverallProgress());
            },
            CancellationToken.None).Result;

        Console.WriteLine("Transcoding job finished.");

        IAsset outputAsset = job.OutputMediaAssets[0];

        return outputAsset;
    }
```

## <a name="publish-the-asset-and-get-urls-for-streaming-and-progressive-download"></a>Publier l'élément et obtenir les URL de diffusion et de téléchargement progressif

Pour diffuser en continu ou télécharger un élément multimédia, vous devez tout d'abord le « publier » en créant un localisateur. Les localisateurs assurent l’accès aux fichiers contenus dans l’élément multimédia. Media Services prend en charge deux types de localisateurs : les localisateurs OnDemandOrigin, utilisés pour diffuser du contenu multimédia (par exemple, MPEG DASH, HLS ou Smooth Streaming) et les localisateurs d'URL SAS (signature d'accès partagé), utilisés pour télécharger des fichiers multimédias.

### <a name="some-details-about-url-formats"></a>Informations sur les formats d’URL

Une fois que vous avez créé les localisateurs, vous pouvez générer les URL qui seront utilisées pour diffuser en continu ou télécharger vos fichiers. L’exemple dans ce didacticiel affiche les URL que vous pouvez coller dans les navigateurs appropriés. Cette section donne simplement quelques rapides exemples de l’aspect des différents formats.

#### <a name="a-streaming-url-for-mpeg-dash-has-the-following-format"></a>Les URL de diffusion en continu pour MPEG DASH ont le format suivant :

{nom du point de terminaison de streaming-nom du compte media services}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest **(format=mpd-time-csf)**

#### <a name="a-streaming-url-for-hls-has-the-following-format"></a>Les URL de diffusion en continu pour HLS ont le format suivant :

{nom du point de terminaison de streaming-nom du compte media services}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest **(format=m3u8-aapl)**

#### <a name="a-streaming-url-for-smooth-streaming-has-the-following-format"></a>Les URL de diffusion en continu pour Smooth Streaming ont le format suivant :

{nom du point de terminaison de diffusion en continu-nom du compte media services}.streaming.mediaservices.windows.net/{ID_de_localisateur}/{nom_de_fichier}.ISM/Manifest


#### <a name="a-sas-url-used-to-download-files-has-the-following-format"></a>Les URL SAS permettant de télécharger les fichiers ont le format suivant :

{nom du conteneur d’objets blob}/{nom de la ressource}/{nom du fichier}/{signature SAS}

Les extensions du Kit de développement logiciel (SDK) Media Services pour .NET fournissent des méthodes d'assistance pratiques qui retournent des URL formatées pour la ressource publiée.

Le code suivant utilise les extensions du Kit de développement logiciel (SDK) Media Services pour .NET pour créer des localisateurs, obtenir la diffusion en continu et les URL de téléchargement progressif. Le code montre également comment télécharger les fichiers vers un dossier local.

Ajoutez la méthode suivante à la classe Program.

```csharp
    static public void PublishAssetGetURLs(IAsset asset)
    {
        // Publish the output asset by creating an Origin locator for adaptive streaming,
        // and a SAS locator for progressive download.

        _context.Locators.Create(
            LocatorType.OnDemandOrigin,
            asset,
            AccessPermissions.Read,
            TimeSpan.FromDays(30));

        _context.Locators.Create(
            LocatorType.Sas,
            asset,
            AccessPermissions.Read,
            TimeSpan.FromDays(30));


        IEnumerable<IAssetFile> mp4AssetFiles = asset
                .AssetFiles
                .ToList()
                .Where(af => af.Name.EndsWith(".mp4", StringComparison.OrdinalIgnoreCase));

        // Get the Smooth Streaming, HLS and MPEG-DASH URLs for adaptive streaming,
        // and the Progressive Download URL.
        Uri smoothStreamingUri = asset.GetSmoothStreamingUri();
        Uri hlsUri = asset.GetHlsUri();
        Uri mpegDashUri = asset.GetMpegDashUri();

        // Get the URls for progressive download for each MP4 file that was generated as a result
        // of encoding.
        List<Uri> mp4ProgressiveDownloadUris = mp4AssetFiles.Select(af => af.GetSasUri()).ToList();


        // Display  the streaming URLs.
        Console.WriteLine("Use the following URLs for adaptive streaming: ");
        Console.WriteLine(smoothStreamingUri);
        Console.WriteLine(hlsUri);
        Console.WriteLine(mpegDashUri);
        Console.WriteLine();

        // Display the URLs for progressive download.
        Console.WriteLine("Use the following URLs for progressive download.");
        mp4ProgressiveDownloadUris.ForEach(uri => Console.WriteLine(uri + "\n"));
        Console.WriteLine();

        // Download the output asset to a local folder.
        string outputFolder = "job-output";
        if (!Directory.Exists(outputFolder))
        {
            Directory.CreateDirectory(outputFolder);
        }

        Console.WriteLine();
        Console.WriteLine("Downloading output asset files to a local folder...");
        asset.DownloadToFolder(
            outputFolder,
            (af, p) =>
            {
                Console.WriteLine("Downloading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
            });

        Console.WriteLine("Output asset files available at '{0}'.", Path.GetFullPath(outputFolder));
    }
```

## <a name="test-by-playing-your-content"></a>Tester en lisant votre contenu

Une fois que vous exécutez le programme défini dans la section précédente, les URL similaires à celles qui suivent seront affichées dans la fenêtre de la console.

URL de diffusion adaptative :

Smooth Streaming

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest`

HLS

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=m3u8-aapl)`

MPEG DASH

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=mpd-time-csf)`

URL de téléchargement progressif (audio et vidéo).

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1500kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1000kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_56kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`


Pour diffuser votre vidéo, collez votre URL dans la zone de texte URL du [lecteur Azure Media Services](https://aka.ms/azuremediaplayer).

Pour tester le téléchargement progressif, collez l'URL dans un navigateur (par exemple, Internet Explorer, Chrome ou Safari).

Pour plus d'informations, voir les rubriques suivantes :

- [Lecture de votre contenu à l’aide des lecteurs existants](media-services-playback-content-with-existing-players.md)
- [Incorporation d’une vidéo de diffusion en continu adaptative MPEG-DASH dans une application HTML5 avec DASH.js](media-services-embed-mpeg-dash-in-html5.md)

## <a name="download-sample"></a>Charger l’exemple
L’exemple de code suivant contient le code que vous avez créé dans ce didacticiel : [exemple](https://azure.microsoft.com/documentation/samples/media-services-dotnet-on-demand-encoding-with-media-encoder-standard/).

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]



<!-- Anchors. -->


<!-- URLs. -->
[Web Platform Installer]: https://go.microsoft.com/fwlink/?linkid=255386
[Portal]: https://portal.azure.com/
