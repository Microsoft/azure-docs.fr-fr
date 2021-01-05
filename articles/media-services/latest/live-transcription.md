---
title: Transcription en direct
titleSuffix: Azure Media Services
description: En savoir plus sur la transcription en direct Azure Media Services.
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
ms.openlocfilehash: c3465e294af104c4d9c3b34960f5e95cf41e7cb8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89291413"
---
# <a name="live-transcription-preview"></a>Transcription en direct (préversion)

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Azure Media Services diffuse de la vidéo, de l’audio et du texte dans différents protocoles. Lorsque vous publiez votre stream en direct en MPEG-DASH ou HLS/CMAF, notre service diffuse le texte transcrit en IMSC1.1 compatible TTML avec la vidéo et le son. La diffusion est empaquetée sous forme de fragments MPEG-4 Partie 30 (ISO/CEI 14496-30). Si vous utilisez la diffusion via HLS/TS, le texte est diffusé sous forme de VTT segmenté.

Des frais supplémentaires s’appliquent quand la transcription en direct est activée. Veuillez consulter les informations relatives aux tarifs dans la section Vidéo en direct de la [page des tarifs de Media Services](https://azure.microsoft.com/pricing/details/media-services/).

Cet article explique comment activer la transcription en direct lors du streaming d’un événement en direct grâce à Azure Media Services. Avant de continuer, assurez-vous de connaître l’utilisation des API REST de Media Services v3 (pour plus d’informations, consultez [ce tutoriel](stream-files-tutorial-with-rest.md)). Vous devez également maîtriser le concept de [streaming en direct](live-streaming-overview.md). Nous vous conseillons de suivre le tutoriel [Diffuser en direct avec Media Services](stream-live-tutorial-with-api.md).

## <a name="live-transcription-preview-regions-and-languages"></a>Régions et langues de la transcription en direct (préversion)

La transcription en direct est disponible dans les régions suivantes :

- Asie Sud-Est
- Europe Ouest
- Europe Nord
- USA Est
- USA Centre
- États-Unis - partie centrale méridionale
- USA Ouest 2
- Brésil Sud

Voici la liste des langues qui peuvent être transcrites ; utilisez le code langue dans l’API.

| Langage | Code langue |
| -------- | ------------- |
| Catalan  | ca-ES |
| Danois (Danemark) | da-DK |
| Allemand (Allemagne) | de-DE |
| Anglais (Australie) | en-AU |
| Anglais (Canada) | en-CA |
| Anglais (Royaume-Uni) | en-GB |
| Anglais (Inde) | en-IN |
| Anglais (Nouvelle-Zélande) | en-NZ |
| Anglais (États-Unis) | fr-FR |
| Espagnol (Espagne) | es-ES |
| Espagnol (Mexique) | es-MX |
| Finnois (Finlande) | fi-FI |
| Français (Canada) | fr-CA |
| Français (France) | fr-FR |
| Italien (Italie) | it-IT |
| Néerlandais (Pays-Bas) | nl-NL |
| Portugais (Brésil) | pt-br |
| Portugais (Portugal) | pt-PT |
| Suédois (Suède) | sv-SE |

## <a name="create-the-live-event-with-live-transcription"></a>Créer l’événement en direct avec transcription en direct

Pour créer un événement en direct avec la transcription activée, envoyez l’opération PUT avec la version d’API 2019-05-01-Preview, par exemple :

```
PUT https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/liveEvents/:liveEventName?api-version=2019-05-01-preview&autoStart=true 
```

L’opération a le corps suivant (où un événement en direct pass-through est créé avec RTMP comme protocole de réception). Notez l’ajout d’une propriété de transcriptions.

```
{
  "properties": {
    "description": "Demonstrate how to enable live transcriptions",
    "input": {
      "streamingProtocol": "RTMP",
      "accessControl": {
        "ip": {
          "allow": [
            {
              "name": "Allow All",
              "address": "0.0.0.0",
              "subnetPrefixLength": 0
            }
          ]
        }
      }
    },
    "preview": {
      "accessControl": {
        "ip": {
          "allow": [
            {
              "name": "Allow All",
              "address": "0.0.0.0",
              "subnetPrefixLength": 0
            }
          ]
        }
      }
    },
    "encoding": {
      "encodingType": "None"
    },
    "transcriptions": [
      {
        "language": "en-US"
      }
    ],
    "vanityUrl": false,
    "streamOptions": [
      "Default"
    ]
  },
  "location": "West US 2"
}
```

## <a name="start-or-stop-transcription-after-the-live-event-has-started"></a>Démarrer ou arrêter la transcription une fois l’événement en direct lancé

Vous pouvez démarrer et arrêter la transcription en direct quand l’événement en direct est en cours d’exécution. Pour plus d’informations sur le démarrage et l’arrêt des événements en direct, consultez la section Opérations de longue durée dans [Développer avec les API Media Services v3](media-services-apis-overview.md#long-running-operations).

Pour activer les transcriptions en direct ou mettre à jour la langue d’une transcription, corrigez l’événement en direct pour inclure une propriété « transcriptions ». Pour désactiver les transcriptions en direct, supprimez la propriété « transcriptions » de l’objet d’événement en direct.  

> [!NOTE]
> L’activation ou la désactivation de la transcription **à plusieurs reprises** lors de l’événement en direct n’est pas un scénario pris en charge.

Voici l’exemple d’appel permettant d’activer les transcriptions en direct.

PATCH : ```https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/liveEvents/:liveEventName?api-version=2019-05-01-preview```

```
{
  "properties": {
    "description": "Demonstrate how to enable live transcriptions", 
    "input": {
      "streamingProtocol": "RTMP",
      "accessControl": {
        "ip": {
          "allow": [
            {
              "name": "Allow All",
              "address": "0.0.0.0",
              "subnetPrefixLength": 0
            }
          ]
        }
      }
    },
    "preview": {
      "accessControl": {
        "ip": {
          "allow": [
            {
              "name": "Allow All",
              "address": "0.0.0.0",
              "subnetPrefixLength": 0
            }
          ]
        }
      }
    },
    "encoding": {
      "encodingType": "None"
    },
    "transcriptions": [
      {
        "language": "en-US"
      }
    ],
    "vanityUrl": false,
    "streamOptions": [
      "Default"
    ]
  },
  "location": "West US 2"
}
```

## <a name="transcription-delivery-and-playback"></a>Diffusion et lecture de la transcription

Consultez l’article [Vue d’ensemble de l’empaquetage dynamique](dynamic-packaging-overview.md#to-prepare-your-source-files-for-delivery) sur la façon dont notre service utilise l’empaquetage dynamique pour diffuser de la vidéo, de l’audio et du texte dans différents protocoles. Lorsque vous publiez votre stream en direct en MPEG-DASH ou HLS/CMAF, notre service diffuse le texte transcrit en IMSC1.1 compatible TTML avec la vidéo et le son. Cette diffusion est empaquetée sous forme de fragments MPEG-4 Partie 30 (ISO/CEI 14496-30). Si vous utilisez la diffusion via HLS/TS, le texte est diffusé sous forme en blocs VTT. Vous pouvez utiliser un lecteur Web, tel que le [Lecteur multimédia Azure](use-azure-media-player.md) pour lire le flux.  

> [!NOTE]
> Si vous utilisez le Lecteur multimédia Azure, utilisez la version 2.3.3 ou une version ultérieure.

## <a name="known-issues"></a>Problèmes connus

Pour la préversion, voici les problèmes connus liés à la transcription en direct :

- Les applications doivent utiliser les API de préversion, décrites dans la documentation [Spécification OpenAPI de Media Services v3](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/mediaservices/resource-manager/Microsoft.Media/preview/2019-05-01-preview/streamingservice.json).
- La protection de la gestion des droits numériques (DRM) ne s’applique pas à la piste de texte ; seul le chiffrement de l’enveloppe AES est possible.

## <a name="next-steps"></a>Étapes suivantes

* [Présentation de Media Services](media-services-overview.md)
