---
title: Utiliser l’interface CLI pour créer des filtres avec Azure Media Services | Microsoft Docs
description: Cet article montre comment utiliser l’interface CLI pour créer des filtres avec Azure Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: f86b3fbebfaa2eb6fc35d5373eab2bb296ff5043
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92740603"
---
# <a name="creating-filters-with-cli"></a>Créer des filtres avec l’interface CLI

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Quand vous transmettez votre contenu à un client (événements de streaming en direct ou vidéo à la demande), le fichier manifeste de l’actif multimédia par défaut ne permet peut-être pas au client d’interagir avec le contenu comme il le voudrait. Avec Azure Media Services, vous pouvez définir des filtres de compte et d’élément multimédia à appliquer à votre contenu. 

Pour obtenir une description détaillée de cette fonctionnalité et des scénarios dans lesquels elle est utilisée, consultez [Manifestes dynamiques](filters-dynamic-manifest-overview.md) et [Filtres](filters-concept.md).

Cette rubrique explique comment définir un filtre pour un actif multimédia Vidéo à la demande et comment utiliser l’interface CLI dans Media Services v3 pour créer des [filtres de compte](/cli/azure/ams/account-filter?view=azure-cli-latest) et des [filtres d’actif multimédia](/cli/azure/ams/asset-filter?view=azure-cli-latest). 

> [!NOTE]
> Veillez à consulter [presentationTimeRange](filters-concept.md#presentationtimerange).

## <a name="prerequisites"></a>Prérequis 

- [Créer un compte Media Services](./create-account-howto.md). Veillez à mémoriser le nom du groupe de ressources et le nom du compte Media Services. 

[!INCLUDE [media-services-cli-instructions](../../../includes/media-services-cli-instructions.md)]

## <a name="define-a-filter"></a>Définir un filtre 

L’exemple suivant définit les conditions de sélection de piste qui sont ajoutées au manifeste final. Ce filtre inclut toutes les pistes audio EC-3 et toutes les pistes vidéo dont la vitesse de transmission est comprise entre 0 et 1 000 000.

> [!TIP]
> Si vous prévoyez de définir **Filtres** dans REST, vous remarquerez que vous devez inclure l’objet JSON wrapper « Propriétés ».  

```json
[
    {
        "trackSelections": [
            {
                "property": "Type",
                "value": "Audio",
                "operation": "Equal"
            },
            {
                "property": "FourCC",
                "value": "EC-3",
                "operation": "NotEqual"
            }
        ]
    },
    {
        "trackSelections": [
            {
                "property": "Type",
                "value": "Video",
                "operation": "Equal"
            },
            {
                "property": "Bitrate",
                "value": "0-1000000",
                "operation": "Equal"
            }
        ]
    }
]
```

## <a name="create-account-filters"></a>Créer des filtres de compte

La commande [az ams account-filter](/cli/azure/ams/account-filter?view=azure-cli-latest) ci-dessous crée un filtre de compte qui filtre les sélections de pistes [définies précédemment](#define-a-filter). 

La commande vous permet de transmettre paramètre `--tracks` facultatif contenant un JSON qui représente les sélections de pistes.  Utilisez @{file} pour charger le JSON à partir d’un fichier. Si vous utilisez l’interface de ligne de commande Azure en local, spécifiez le chemin d’accès de la totalité du fichier :

```azurecli
az ams account-filter create -a amsAccount -g resourceGroup -n filterName --tracks @tracks.json
```

Voir aussi [Exemples de filtres JSON](/rest/api/media/accountfilters/createorupdate#create-an-account-filter).

## <a name="create-asset-filters"></a>Créer des filtres d’élément multimédia

La commande [az ams asset-filter](/cli/azure/ams/asset-filter?view=azure-cli-latest) ci-dessous crée un filtre d’actif multimédia qui filtre les sélections de pistes [définies précédemment](#define-a-filter). 

```azurecli
az ams asset-filter create -a amsAccount -g resourceGroup -n filterName --asset-name assetName --tracks @tracks.json
```

Voir aussi [Exemples de filtres JSON](/rest/api/media/assetfilters/createorupdate#create-an-asset-filter).

## <a name="associate-filters-with-streaming-locator"></a>Associer des filtres à un localisateur de streaming

Vous pouvez spécifier une liste de filtres de comptes ou de ressources qui s’appliquent à votre localisateur de streaming. Le [packager dynamique (point de terminaison de streaming)](dynamic-packaging-overview.md) applique cette liste de filtres avec ceux spécifiés par votre client dans l’URL. Cette combinaison génère un [manifeste dynamique](filters-dynamic-manifest-overview.md) qui est basé sur les filtres spécifiés dans l’URL ainsi que sur ceux que vous spécifiez dans le localisateur de streaming. Nous vous recommandons d’utiliser cette fonctionnalité si vous voulez appliquer des filtres, mais que vous ne voulez pas exposer les noms de filtre dans l’URL.

Le code CLI suivant montre comment créer un localisateur de streaming et spécifier `filters`. Il s’agit d’une propriété optionnelle qui prend une liste de noms de filtre de ressources et/ou de noms de filtre de comptes séparés par un espace.

```azurecli
az ams streaming-locator create -a amsAccount -g resourceGroup -n streamingLocatorName \
                                --asset-name assetName \                               
                                --streaming-policy-name policyName \
                                --filters filterName1 filterName2
                                
```

## <a name="stream-using-filters"></a>Effectuer un streaming à l’aide de filtres

Lorsque vous définissez des filtres, vos clients peuvent les utiliser dans l'URL de diffusion en continu. Il serait possible d’appliquer des filtres à des protocoles de streaming à débit adaptatif : HLS (HTTP Live Streaming) Apple, MPEG-DASH et Smooth Streaming.

Le tableau suivant présente des exemples d’URL utilisant des filtres :

|Protocol|Exemple|
|---|---|
|HLS|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=m3u8-aapl,filter=myAccountFilter)`|
|MPEG DASH|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=mpd-time-csf,filter=myAssetFilter)`|
|Smooth Streaming|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(filter=myAssetFilter)`|

## <a name="next-step"></a>Étape suivante

[Diffuser des vidéos en streaming](stream-files-tutorial-with-api.md) 

## <a name="see-also"></a>Voir aussi

[Azure CLI](/cli/azure/ams?view=azure-cli-latest)
