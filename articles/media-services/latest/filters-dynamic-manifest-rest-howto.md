---
title: Création de filtres avec l’API REST Media Services Azure v3
description: Cette rubrique décrit comment créer des filtres pour que votre client puisse les utiliser pour diffuser des sections spécifiques d'un flux. Media Services v3 crée des manifestes dynamiques pour obtenir cette diffusion sélective.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: c1d7bf933b487c40d571f1912341b5ef771e4e67
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90527324"
---
# <a name="creating-filters-with-media-services-rest-api"></a>Créer des filtres avec l’API REST Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Quand vous transmettez votre contenu à un client (événements de streaming en direct ou vidéo à la demande), le fichier manifeste de l’élément multimédia par défaut ne permet pas toujours au client d’interagir avec le contenu comme il le voudrait. Avec Azure Media Services, vous pouvez définir des filtres de compte et d’élément multimédia à appliquer à votre contenu. 

Pour obtenir une description détaillée de cette fonctionnalité et des scénarios dans lesquels elle est utilisée, consultez [Manifestes dynamiques](filters-dynamic-manifest-overview.md) et [Filtres](filters-concept.md).

Cette rubrique explique comment définir un filtre pour un élément multimédia Vidéo à la demande et utiliser les API REST pour créer des [Filtres de compte](/rest/api/media/accountfilters) et des [Filtres d’élément multimédia](/rest/api/media/assetfilters). 

> [!NOTE]
> Veillez à consulter [presentationTimeRange](filters-concept.md#presentationtimerange).

## <a name="prerequisites"></a>Prérequis 

Pour suivre les étapes décrites dans cette rubrique, vous devez :

- Lire [Filtres et manifestes dynamiques](filters-dynamic-manifest-overview.md).
- [Configurer Postman pour les appels d’API REST Azure Media Services](media-rest-apis-with-postman.md).

    Suivez la dernière étape de la rubrique [Obtenir un jeton Azure AD](media-rest-apis-with-postman.md#get-azure-ad-token). 

## <a name="define-a-filter"></a>Définir un filtre  

L’exemple de **Corps de la demande** suivant définit les conditions de sélection de piste qui sont ajoutées au manifeste. Ce filtre inclut toutes les pistes audio EC-3 et toutes les pistes vidéo dont la vitesse de transmission est comprise entre 0 et 1 000 000.

```json
{
    "properties": {
        "tracks": [
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
                        "operation": "Equal"
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
     }
}
```

## <a name="create-account-filters"></a>Créer des filtres de compte

Dans la collection Postman que vous avez téléchargée, sélectionnez **Filtres de compte**->**Créer ou mettre à jour un filtre de compte**.

La méthode de requête HTTP **PUT** se présente ainsi :

```
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Media/mediaServices/{accountName}/accountFilters/{filterName}?api-version=2018-07-01
```

Sélectionnez l’onglet **Corps** et collez le code JSON que vous avez [défini tout à l’heure](#define-a-filter).

Sélectionnez **Envoyer**. 

Le filtre a été créé.

Pour plus d’informations, voir [Créer ou mettre à jour](/rest/api/media/accountfilters/createorupdate). Voir aussi [Exemples de filtres JSON](/rest/api/media/accountfilters/createorupdate#create-an-account-filter).

## <a name="create-asset-filters"></a>Créer des filtres d’élément multimédia  

Dans la collection Postman « Media Services v3 » que vous avez téléchargée, sélectionnez **Éléments**->**Créer ou mettre à jour un filtre d’élément multimédia**.

La méthode de requête HTTP **PUT** se présente ainsi :

```
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Media/mediaServices/{accountName}/assets/{assetName}/assetFilters/{filterName}?api-version=2018-07-01
```

Sélectionnez l’onglet **Corps** et collez le code JSON que vous avez [défini tout à l’heure](#define-a-filter).

Sélectionnez **Envoyer**. 

Le filtre d’élément multimédia a été créé.

Pour savoir comment créer ou mettre à jour des filtres d’élément multimédia, voir [Créer ou mettre à jour](/rest/api/media/assetfilters/createorupdate). Voir aussi [Exemples de filtres JSON](/rest/api/media/assetfilters/createorupdate#create-an-asset-filter). 

## <a name="associate-filters-with-streaming-locator"></a>Associer des filtres à un localisateur de streaming

Vous pouvez spécifier une liste de filtres de comptes ou de ressources qui s’appliquent à votre localisateur de streaming. Le [packager dynamique (point de terminaison de streaming)](dynamic-packaging-overview.md) applique cette liste de filtres avec ceux spécifiés par votre client dans l’URL. Cette combinaison génère un [manifeste dynamique](filters-dynamic-manifest-overview.md) qui est basé sur les filtres spécifiés dans l’URL ainsi que sur ceux que vous spécifiez dans le localisateur de streaming. Nous vous recommandons d’utiliser cette fonctionnalité si vous voulez appliquer des filtres, mais que vous ne voulez pas exposer les noms de filtre dans l’URL.

Pour créer des filtres et les associer à un localisateur de streaming à l’aide de REST, utilisez l’API [Créer un localisateur de streaming](/rest/api/media/streaminglocators/create) et spécifiez `properties.filters` dans le [corps de la demande](/rest/api/media/streaminglocators/create#request-body).
                                
## <a name="stream-using-filters"></a>Effectuer un streaming à l’aide de filtres

Lorsque vous définissez des filtres, vos clients peuvent les utiliser dans l'URL de diffusion en continu. Il serait possible d’appliquer des filtres à des protocoles de streaming à débit adaptatif : HLS (HTTP Live Streaming) Apple, MPEG-DASH et Smooth Streaming.

Le tableau suivant présente des exemples d’URL utilisant des filtres :

|Protocol|Exemple|
|---|---|
|HLS|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=m3u8-aapl,filter=myAccountFilter)`|
|MPEG DASH|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=mpd-time-csf,filter=myAssetFilter)`|
|Smooth Streaming|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(filter=myAssetFilter)`|

## <a name="next-steps"></a>Étapes suivantes

[Diffuser des vidéos en streaming](stream-files-tutorial-with-rest.md) 
