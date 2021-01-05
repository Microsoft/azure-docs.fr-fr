---
title: Création de filtres avec le Kit de développement logiciel (SDK) .NET Azure Media Services v3
description: Cette rubrique décrit comment créer des filtres pour que votre client puisse les utiliser pour diffuser des sections spécifiques d'un flux. Le SDK .NET Media Services v3 crée des manifestes dynamiques pour obtenir cette diffusion sélective.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 9f54a505d63660fb7c97b3226df4c232e48125e1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90527358"
---
# <a name="create-filters-with-media-services-net-sdk"></a>Créer des filtres à l’aide du kit SDK .NET Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Quand vous transmettez votre contenu à un client (événements de streaming en direct ou vidéo à la demande), le fichier manifeste de l’élément multimédia par défaut ne permet pas toujours au client d’interagir avec le contenu comme il le voudrait. Avec Azure Media Services, vous pouvez définir des filtres de compte et d’élément multimédia à appliquer à votre contenu. 

Pour obtenir une description détaillée de cette fonctionnalité et des scénarios dans lesquels elle est utilisée, consultez [Manifestes dynamiques](filters-dynamic-manifest-overview.md) et [Filtres](filters-concept.md).

Cette rubrique explique comment utiliser le SDK .NET Media Services pour définir un filtre pour un actif multimédia Vidéo à la demande, et créer des [filtres de compte](/dotnet/api/microsoft.azure.management.media.models.accountfilter?view=azure-dotnet) et des [filtres d’actif multimédia](/dotnet/api/microsoft.azure.management.media.models.assetfilter?view=azure-dotnet). 

> [!NOTE]
> Veillez à consulter [presentationTimeRange](filters-concept.md#presentationtimerange).

## <a name="prerequisites"></a>Prérequis 

- Lire [Filtres et manifestes dynamiques](filters-dynamic-manifest-overview.md).
- [Créer un compte Media Services](./create-account-howto.md). Veillez à mémoriser le nom du groupe de ressources et le nom du compte Media Services. 
- Obtenir les informations nécessaires pour [accéder aux API](./access-api-howto.md)
- Lire la rubrique [Charger, encoder et transmettre en continu des vidéos à l’aide d’Azure Media Services](stream-files-tutorial-with-api.md) pour voir comment [démarrer avec le SDK .NET](stream-files-tutorial-with-api.md#start-using-media-services-apis-with-net-sdk)

## <a name="define-a-filter"></a>Définir un filtre  

Dans .NET, vous configurez les sélections de pistes avec les classes [FilterTrackSelection](/dotnet/api/microsoft.azure.management.media.models.filtertrackselection?view=azure-dotnet) et [FilterTrackPropertyCondition](/dotnet/api/microsoft.azure.management.media.models.filtertrackpropertycondition?view=azure-dotnet). 

Le code suivant définit un filtre qui inclut toutes les pistes audio contenant EC-3 et toutes les pistes vidéo ayant une vitesse de transmission entre 0 et 1 000 000.

```csharp
var audioConditions = new List<FilterTrackPropertyCondition>()
{
    new FilterTrackPropertyCondition(FilterTrackPropertyType.Type, "Audio", FilterTrackPropertyCompareOperation.Equal),
    new FilterTrackPropertyCondition(FilterTrackPropertyType.FourCC, "EC-3", FilterTrackPropertyCompareOperation.Equal)
};

var videoConditions = new List<FilterTrackPropertyCondition>()
{
    new FilterTrackPropertyCondition(FilterTrackPropertyType.Type, "Video", FilterTrackPropertyCompareOperation.Equal),
    new FilterTrackPropertyCondition(FilterTrackPropertyType.Bitrate, "0-1000000", FilterTrackPropertyCompareOperation.Equal)
};

List<FilterTrackSelection> includedTracks = new List<FilterTrackSelection>()
{
    new FilterTrackSelection(audioConditions),
    new FilterTrackSelection(videoConditions)
};
```

## <a name="create-account-filters"></a>Créer des filtres de compte

Le code suivant montre comment utiliser .NET pour créer un filtre de compte qui inclut toutes les sélections de pistes [définies ci-dessus](#define-a-filter). 

```csharp
AccountFilter accountFilterParams = new AccountFilter(tracks: includedTracks);
client.AccountFilters.CreateOrUpdate(config.ResourceGroup, config.AccountName, "accountFilterName1", accountFilter);
```

## <a name="create-asset-filters"></a>Créer des filtres d’élément multimédia

Le code suivant montre comment utiliser .NET pour créer un filtre d’actif multimédia qui inclut toutes les sélections de pistes [définies ci-dessus](#define-a-filter). 

```csharp
AssetFilter assetFilterParams = new AssetFilter(tracks: includedTracks);
client.AssetFilters.CreateOrUpdate(config.ResourceGroup, config.AccountName, encodedOutputAsset.Name, "assetFilterName1", assetFilterParams);
```

## <a name="associate-filters-with-streaming-locator"></a>Associer des filtres à un localisateur de streaming

Vous pouvez spécifier une liste de filtres de comptes ou de ressources qui s’appliquent à votre localisateur de streaming. Le [packager dynamique (point de terminaison de streaming)](dynamic-packaging-overview.md) applique cette liste de filtres avec ceux spécifiés par votre client dans l’URL. Cette combinaison génère un [manifeste dynamique](filters-dynamic-manifest-overview.md) qui est basé sur les filtres spécifiés dans l’URL ainsi que sur ceux que vous spécifiez dans le localisateur de streaming. Nous vous recommandons d’utiliser cette fonctionnalité si vous voulez appliquer des filtres, mais que vous ne voulez pas exposer les noms de filtre dans l’URL.

Le code C# suivant montre comment créer un localisateur de streaming et spécifier `StreamingLocator.Filters`. Il s’agit d’une propriété facultative qui prend un `IList<string>` de noms de filtres.

```csharp
IList<string> filters = new List<string>();
filters.Add("filterName");

StreamingLocator locator = await client.StreamingLocators.CreateAsync(
    resourceGroup,
    accountName,
    locatorName,
    new StreamingLocator
    {
        AssetName = assetName,
        StreamingPolicyName = PredefinedStreamingPolicy.ClearStreamingOnly,
        Filters = filters
    });
```
      
## <a name="stream-using-filters"></a>Effectuer un streaming à l’aide de filtres

Lorsque vous définissez des filtres, vos clients peuvent les utiliser dans l'URL de diffusion en continu. Il serait possible d’appliquer des filtres à des protocoles de streaming à débit adaptatif : HLS (HTTP Live Streaming) Apple, MPEG-DASH et Smooth Streaming.

Le tableau suivant présente des exemples d’URL utilisant des filtres :

|Protocol|Exemple|
|---|---|
|HLS|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=m3u8-aapl,filter=myAccountFilter)`|
|MPEG DASH|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=mpd-time-csf,filter=myAssetFilter)`|
|Smooth Streaming|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(filter=myAssetFilter)`|

## <a name="next-steps"></a>Étapes suivantes

[Diffuser des vidéos en streaming](stream-files-tutorial-with-api.md) 
