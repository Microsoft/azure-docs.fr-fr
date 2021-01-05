---
title: Forum Aux Questions (FAQ) Azure Media Services
description: Cet article donne des réponses aux questions les plus fréquemment posées sur Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: ff15206e2373ce6481cb65cbbeeb3c88cd2b7154
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89269418"
---
# <a name="media-services-v2-frequently-asked-questions"></a>Forum Aux Questions (FAQ) Media Services v2

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Cet article répond aux questions fréquemment posées par communauté des utilisateurs d’Azure Media Services (AMS).

## <a name="general-ams-faqs"></a>Forum Aux Questions - Généralités AMS

Q : Comment diffuser en continu vers des appareils Apple iOS ?

R : ajoutez le chemin « (format=m3u8-aapl) » à la partie « /Manifest » de l’URL pour indiquer au serveur de diffusion en continu d’origine qu’il doit retourner le contenu HLS pour la consommation sur les appareils natifs Apple iOS (pour plus d’informations, consultez l’article relatif à la [distribution de contenu](media-services-deliver-content-overview.md)).

Q : Comment mettre à l’échelle l’indexation ?

A : Les unités réservées sont les mêmes pour les tâches d’encodage et d’indexation. Suivez les instructions de la page [Comment mettre à l’échelle des unités réservées mise à l’échelle des unités réservées d’encodage](media-services-scale-media-processing-overview.md). **Remarque** : le fonctionnement de l’indexeur n’est pas affecté par le type d’unité réservée.

Q : J’ai chargé, encodé et publié une vidéo. Pourquoi la vidéo n’est-elle pas lue lorsque j’essaie de la diffuser en continu ?

A : Une des raisons les plus courantes est que le point de terminaison de streaming à partir duquel vous essayez de lire n’est pas dans l’état **En cours d’exécution**.  

Q : La composition d’un flux dynamique est-elle possible ?

A : La composition de flux dynamiques n’est pas disponible pour le moment dans Azure Media Services. Vous devez donc effectuer la composition au préalable sur votre ordinateur.

Q : Puis-je utiliser le CDN Azure avec la vidéo en flux continu ?

A : Media Services prend en charge l’intégration au CDN Azure (pour plus d’informations, consultez la rubrique [Gestion des points de terminaison de diffusion en continu dans un compte Media Services](media-services-portal-manage-streaming-endpoints.md)).  Vous pouvez utiliser la vidéo en flux continu avec le CDN. Azure Media Services fournit des sorties aux formats Smooth Streaming, HLS et MPEG-DASH. Tous ces formats utilisent le protocole HTTP pour transférer les données et bénéficient des avantages de la mise en cache HTTP. Lors d’une diffusion vidéo en flux continu, les données vidéo/audio sont divisées en fragments individuels qui sont mis en cache dans le CDN. Les seules données devant être actualisées sont les données de manifeste. CDN actualise régulièrement les données de manifeste.

Q : Le stockage des images est-il pris en charge par Azure Media Services ?

A : Si vous cherchez uniquement à stocker des images JPEG ou PNG, nous vous recommandons de les conserver dans le Stockage Blob Azure. Il n’y a aucun avantage à les placer dans votre compte Media Services, à moins que vous souhaitiez qu’elles restent associées à vos ressources vidéo ou audio Ou, si vous avez besoin d'utiliser les images sous forme de superpositions dans l'encodeur vidéo, Media Encoder Standard prend en charge la superposition d’images sur les vidéos, ce qui explique pourquoi JPEG et PNG figurent parmi les formats d’entrée pris en charge. Pour plus d’informations, consultez la page [Création de superpositions](media-services-advanced-encoding-with-mes.md#overlay).

Q : Comment puis-je copier des éléments multimédias d’un compte Media Services vers un autre ?

A : Pour copier des éléments multimédias d’un compte Media Services vers un autre à l’aide de .NET, utilisez la méthode d’extension [IAsset.Copy](https://github.com/Azure/azure-sdk-for-media-services-extensions/blob/dev/MediaServices.Client.Extensions/IAssetExtensions.cs#L354) disponible dans le référentiel [Extensions du Kit de développement logiciel (SDK) Azure Media Services](https://github.com/Azure/azure-sdk-for-media-services-extensions/). Pour plus d’informations, consultez [cette publication de forum](https://social.msdn.microsoft.com/Forums/azure/28912d5d-6733-41c1-b27d-5d5dff2695ca/migrate-media-services-across-subscription?forum=MediaServices) .

Q : Quels sont les caractères pris en charge pour les noms des fichiers en utilisant AMS ?

A : Media Services utilise la valeur de la propriété IAssetFile.Name lors de la génération d’URL pour le contenu de streaming (par exemple, http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters). Pour cette raison, l’encodage par pourcentage n’est pas autorisé. La valeur de la propriété **Name** ne peut pas comporter les [caractères réservés à l’encodage en pourcentage suivants](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters) : !* '();:@&=+$,/?%#[]". En outre, il ne peut exister qu’un ’.’ pour l’extension de nom de fichier.

Q : Comment se connecter avec REST ?

A : Pour savoir comment vous connecter à l’API AMS, consultez [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

Q : Comment faire pivoter une vidéo au cours du processus d’encodage ?

A : [Media Encoder Standard](media-services-dotnet-encode-with-media-encoder-standard.md) prend en charge les angles de rotation 90, 180 et 270. Le comportement par défaut est « Auto », ce qui signifie qu’il tente de détecter les métadonnées de rotation dans le fichier MP4/MOV entrant et de les compenser. Incluez l’élément **Sources** suivant dans l’une des présélections json définies [ici](media-services-mes-presets-overview.md):

```json
"Version": 1.0,
"Sources": [
{
  "Streams": [],
  "Filters": {
    "Rotation": "90"
  }
}
],
"Codecs": [

...
```


## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
