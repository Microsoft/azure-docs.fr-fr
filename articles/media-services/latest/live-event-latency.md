---
title: Paramètres de Latence faible d’événement en direct dans Azure Media Services | Microsoft Docs
description: Cette rubrique donne une vue d’ensemble des paramètres de latence faible de LiveEvent et montre comment définir une latence faible.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 114efe668fba47e9d83741b8fa45e1f396725198
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89291515"
---
# <a name="live-event-low-latency-settings"></a>Paramètres de latence faible d’événement en direct

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Cet article explique comment définir une faible latence sur un [événement en direct](/rest/api/media/liveevents). Il traite également des résultats généralement obtenus avec des paramètres de faible latence sur différents lecteurs. Les résultats varient en fonction de la latence réseau et du CDN.

Pour utiliser la nouvelle fonction **LowLatency**, définissez le paramètre **StreamOptionsFlag** sur **LowLatency** dans l’événement **LiveEvent**. Lors de la création de [LiveOutput](/rest/api/media/liveoutputs) pour la lecture HLS, définissez [LiveOutput.Hls.fragmentsPerTsSegment](/rest/api/media/liveoutputs/create#hls) sur 1. Une fois le flux opérationnel, vous pouvez ouvrir la page de démonstration du [Lecteur multimédia Azure](https://ampdemo.azureedge.net/) et configurer les options de lecture afin d’utiliser le profil heuristique à faible latence (« Low Latency Heuristics Profile »).

> [!NOTE]
> Actuellement, le profil heuristique à faible latence dans le Lecteur multimédia Azure est conçu pour lire des flux dans le protocole MPEG-DASH, avec le format CSF ou CMAF (par exemple, `format=mdp-time-csf` ou `format=mdp-time-cmaf`). 

L’exemple .NET suivant montre comment définir **LowLatency** sur l’événement **LiveEvent** :

```csharp
LiveEvent liveEvent = new LiveEvent(
            location: mediaService.Location, 
            description: "Sample LiveEvent for testing",
            vanityUrl: false,
            encoding: new LiveEventEncoding(
                        // Set this to Standard to enable a transcoding LiveEvent, and None to enable a pass-through LiveEvent
                        encodingType:LiveEventEncodingType.None, 
                        presetName:null
                    ),
            input: new LiveEventInput(LiveEventInputProtocol.RTMP,liveEventInputAccess), 
            preview: liveEventPreview,
            streamOptions: new List<StreamOptionsFlag?>()
            {
                // Set this to Default or Low Latency
                // To use low latency optimally, you should tune your encoder settings down to 1 second "Group Of Pictures" (GOP) length instead of 2 seconds.
                StreamOptionsFlag.LowLatency
            }
        );
```                

Pour voir l’exemple complet : [MediaV3LiveApp](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/blob/master/NETCore/Live/MediaV3LiveApp/Program.cs#L126).

## <a name="live-events-latency"></a>Latence d’événements en direct

Les tableaux suivants illustrent les résultats classiques de latence (quand l’indicateur LowLatency est activé) dans Media Services, mesurée entre le moment où le flux de contribution atteint le service et celui où un utilisateur voit la lecture sur le lecteur. Pour utiliser de façon optimale une faible latence, vous devez baisser les paramètres de votre encodeur en les définissant sur une longueur GOP (« groupe d’images ») de 1 seconde. Quand vous utilisez une longueur GOP supérieure, vous réduisez la consommation de bande passante ainsi que la vitesse de transmission sous la même fréquence d’images. Cela est particulièrement utile dans les vidéos avec moins de mouvement.

### <a name="pass-through"></a>Requête directe 

||GOP 2 s à faible latence|GOP 1 s à faible latence|
|---|---|---|
|**DASH dans AMP**|10 s|8 s|
|**HLS sur lecteur iOS natif**|14 s|10 s|

### <a name="live-encoding"></a>Encodage en direct

||GOP 2 s à faible latence|GOP 1 s à faible latence|
|---|---|---|
|**DASH dans AMP**|14 s|10 s|
|**HLS sur lecteur iOS natif**|18 s|13 s|

> [!NOTE]
> La latence de bout en bout peut varier en fonction des conditions du réseau local ou en introduisant une couche de mise en cache d’un réseau de distribution de contenu. Faites des tests spécifiquement dans vos configurations.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble du streaming en direct](live-streaming-overview.md)
- [Didacticiel sur le streaming en direct](stream-live-tutorial-with-api.md)
