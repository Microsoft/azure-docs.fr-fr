---
title: Publier du contenu Azure Media Services à l’aide de .NET | Microsoft Docs
description: Apprenez à créer un localisateur utilisé pour générer une URL de diffusion en continu. Les exemples de code sont écrits en C# et utilisent le Kit de développement logiciel (SDK) Media Services pour .NET.
author: juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: c53b1f83-4cb1-4b09-840f-9c145b7d6f8d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: a0343c607a342c3438d68627b97c71cdf05199b2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89267783"
---
# <a name="publish-media-services-content-using-net"></a>Publier du contenu Media Services à l’aide de .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [REST](media-services-rest-deliver-streaming-content.md)
> * [.NET](media-services-deliver-streaming-content.md)
> * [Portail](media-services-portal-publish.md)
> 
> 

## <a name="overview"></a>Vue d’ensemble
Vous pouvez diffuser un MP4 à débit adaptatif défini par la création d’un localisateur de diffusion en continu à la demande et la création d’une URL de diffusion en continu. La rubrique [Encodage d’un élément multimédia](media-services-encode-asset.md) indique comment encoder un ensemble de fichiers MP4 à débit adaptatif. 

> [!NOTE]
> Si votre contenu est chiffré, configurez la stratégie de livraison d’éléments multimédias (comme décrit dans [cette](media-services-dotnet-configure-asset-delivery-policy.md) rubrique) avant de créer un localisateur. 
> 
> 

Vous pouvez également utiliser un localisateur de diffusion en continu à la demande pour créer des URL qui pointent vers les fichiers MP4 pouvant être téléchargés progressivement.  

Cette rubrique montre comment créer un localisateur de streaming à la demande pour publier votre élément multimédia et créer des URL de diffusion en continu Smooth, MPEG DASH et HLS. Elle explique également la création d’URL de téléchargement progressif. 

## <a name="create-an-ondemand-streaming-locator"></a>Création d’un localisateur de diffusion en continu à la demande
Pour créer le localisateur de diffusion en continu à la demande et obtenir les URL, vous devez effectuer les opérations suivantes :

1. Si le contenu est chiffré, définissez une stratégie d'accès.
2. Création d’un localisateur de diffusion en continu à la demande.
3. Si vous envisagez de diffuser en continu, obtenez le fichier manifeste de diffusion en continu (.ism) dans la ressource. 
   
   Si vous souhaitez télécharger progressivement, obtenez les noms des fichiers MP4 dans la ressource.  
4. Création d’URL vers le fichier manifeste ou les fichiers MP4. 


>[!NOTE]
>Un nombre limite de 1 000 000 a été défini pour les différentes stratégies AMS (par exemple, pour la stratégie de localisateur ou pour ContentKeyAuthorizationPolicy). Appliquez le même ID de stratégie si vous utilisez toujours le même nombre de jours, les mêmes autorisations d’accès. Par exemple, pour les localisateurs destinés à demeurer en place pendant une longue période (stratégies autres que de type chargement). Pour plus d’informations, consultez [cette rubrique](media-services-dotnet-manage-entities.md#limit-access-policies) .

### <a name="use-media-services-net-sdk"></a>Utilisation du Kit de développement logiciel (SDK) .NET de Media Services
Création d’URL de diffusion 

```csharp
    private static void BuildStreamingURLs(IAsset asset)
    {

        // Create a 30-day readonly access policy. 
          // You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.
        IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
            TimeSpan.FromDays(30),
            AccessPermissions.Read);

        // Create a locator to the streaming content on an origin. 
        ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
            policy,
            DateTime.UtcNow.AddMinutes(-5));

        // Display some useful values based on the locator.
        Console.WriteLine("Streaming asset base path on origin: ");
        Console.WriteLine(originLocator.Path);
        Console.WriteLine();

        // Get a reference to the streaming manifest file from the  
        // collection of files in the asset. 
        var manifestFile = asset.AssetFiles.ToList().Where(f => f.Name.ToLower().
                                    EndsWith(".ism")).
                                    FirstOrDefault();

        // Create a full URL to the manifest file. Use this for playback
        // in streaming media clients. 
        string urlForClientStreaming = originLocator.Path + manifestFile.Name + "/manifest";
        Console.WriteLine("URL to manifest for client streaming using Smooth Streaming protocol: ");
        Console.WriteLine(urlForClientStreaming);
        Console.WriteLine("URL to manifest for client streaming using HLS protocol: ");
        Console.WriteLine(urlForClientStreaming + "(format=m3u8-aapl)");
        Console.WriteLine("URL to manifest for client streaming using MPEG DASH protocol: ");
        Console.WriteLine(urlForClientStreaming + "(format=mpd-time-csf)"); 
        Console.WriteLine();
    }
```

Sorties :

- URL du manifeste pour la diffusion en continu du client à l’aide du protocole Smooth Streaming : \
  `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest`
- URL du manifeste pour la diffusion en continu du client à l’aide du protocole HLS : \
  `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=m3u8-aapl)`
- URL du manifeste pour la diffusion en continu du client à l’aide du protocole MPEG DASH : \
  `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=mpd-time-csf)`

> [!NOTE]
> Vous pouvez aussi diffuser votre contenu via une connexion TLS. Pour ce faire, vérifiez que votre URL de streaming commence par HTTPS. Actuellement, AMS ne prend pas en charge le protocole TLS avec les domaines personnalisés.
> 
> 

Génération d’URL de téléchargement progressif 

```csharp
    private static void BuildProgressiveDownloadURLs(IAsset asset)
    {
        // Create a 30-day readonly access policy. 
        IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
            TimeSpan.FromDays(30),
            AccessPermissions.Read);

        // Create an OnDemandOrigin locator to the asset. 
        ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
            policy,
            DateTime.UtcNow.AddMinutes(-5));

        // Display some useful values based on the locator.
        Console.WriteLine("Streaming asset base path on origin: ");
        Console.WriteLine(originLocator.Path);
        Console.WriteLine();

        // Get MP4 files.
        IEnumerable<IAssetFile> mp4AssetFiles = asset
            .AssetFiles
            .ToList()
            .Where(af => af.Name.EndsWith(".mp4", StringComparison.OrdinalIgnoreCase));

        // Create a full URL to the MP4 files. Use this to progressively download files.
        foreach (var pd in mp4AssetFiles)
            Console.WriteLine(originLocator.Path + pd.Name);
    }
```
Sorties :

- `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4`
- `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4`
- `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4`
- `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4`

    . . . 

### <a name="use-media-services-net-sdk-extensions"></a>Utilisation des extensions du Kit de développement logiciel (SDK) Media Services pour .NET
Le code suivant appelle les méthodes des extensions du Kit de développement logiciel (SDK) .NET, qui créent un localisateur et génèrent les URL Smooth Streaming, HLS et MPEG-DASH pour la diffusion en continu adaptative.
```csharp
    // Create a loctor.
    _context.Locators.Create(
        LocatorType.OnDemandOrigin,
        inputAsset,
        AccessPermissions.Read,
        TimeSpan.FromDays(30));

    // Get the streaming URLs.
    Uri smoothStreamingUri = inputAsset.GetSmoothStreamingUri();
    Uri hlsUri = inputAsset.GetHlsUri();
    Uri mpegDashUri = inputAsset.GetMpegDashUri();

    Console.WriteLine(smoothStreamingUri);
    Console.WriteLine(hlsUri);
    Console.WriteLine(mpegDashUri);
```

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Étapes suivantes
* [Télécharger les éléments multimédias](media-services-deliver-asset-download.md)
* [Configurer une stratégie de distribution d’éléments multimédias](media-services-dotnet-configure-asset-delivery-policy.md)

