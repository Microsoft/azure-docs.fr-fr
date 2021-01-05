---
title: Comment générer des miniatures à l’aide de l’encodeur standard Azure Media Services avec REST
description: Cet article vous explique comment utiliser REST pour coder une ressource tout en générant des miniatures à l’aide de Media Encoder Standard.
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
ms.openlocfilehash: ec2782297f2659341c9fa7e87ce15d3dbceb022c
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92019560"
---
# <a name="how-to-generate-thumbnails-using-encoder-standard-with-rest"></a>Comment générer des miniatures à l’aide de l’encodeur standard avec REST

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Vous pouvez utiliser Media Encoder Standard pour générer une ou plusieurs miniatures à partir de votre vidéo d’entrée au format de fichier d’image [JPEG](https://en.wikipedia.org/wiki/JPEG), [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics) ou [BMP](https://en.wikipedia.org/wiki/BMP_file_format).

## <a name="recommended-reading-and-practice"></a>Lecture et pratiques recommandées

Nous vous recommandons de vous familiariser avec les transformations personnalisées en lisant [Comment encoder avec une transformation - REST](custom-preset-rest-howto.md).

## <a name="thumbnail-parameters"></a>Paramètres des miniatures

Vous devez définir les paramètres suivants :

- **début** : position dans la vidéo d’entrée à partir de laquelle commencer la génération des miniatures. La valeur peut être au format ISO 8601 (par exemple, PT05S pour commencer à 5 secondes) ou un nombre de cadres (par exemple, 10 pour commencer au dixième cadre) ou une valeur relative à la durée du flux (par exemple, 10 % pour commencer à 10 % de la durée du flux). Prend également en charge une macro {Best}, qui indique à l’encodeur de sélectionner la meilleure miniature parmi les premières secondes de la vidéo pour ne produire qu’une seule miniature, quels que soient les autres paramètres pour Étape et Plage. La valeur par défaut est macro {Best}.
- **étape** : intervalles selon lesquelles les miniatures sont générées. La valeur peut être au format ISO 8601 (par exemple, PT05S pour commencer toutes les 5 secondes) ou un nombre de cadres (par exemple, 30 pour une image tous les 30 cadres) ou une valeur relative à la durée du flux (par exemple, 10 % pour une image à chaque 10 % de la durée du flux). La valeur de l’étape affecte la première miniature générée, qui peut ne pas être exactement celle qui est spécifiée lors de l’heure de début de la présélection de la transformation. Cela est dû à l’encodeur, qui tente de sélectionner la meilleure miniature entre l’heure de début et la position de l’étape à partir de l’heure de début comme première sortie. Étant donné que la valeur par défaut est 10 %, si le flux a une durée longue, la première miniature générée peut être éloignée de celle spécifiée au moment de l’heure de début. Essayez de sélectionner une valeur raisonnable pour l’étape si la première miniature est censée être proche de l’heure de début ou définissez la valeur de la plage sur 1 si une seule miniature est nécessaire au moment de l’heure de début.
- **plage** : position relative à l’heure de début de la présélection de transformation dans la vidéo d’entrée à laquelle arrêter la génération de miniatures. La valeur peut être au format ISO 8601 (par exemple, PT5M30S pour s’arrêter à 5 minutes et 30 secondes à partir de l’heure de début) ou un nombre de cadres (par exemple, 300 pour s’arrêter au niveau de la 300e trame à partir du cadre à l’heure de début. Si cette valeur est 1, cela signifie qu’il ne doit produire qu’une seule miniature à l’heure de début, ou une valeur relative à la durée du flux (par exemple, 50 % pour s’arrêter à la moitié de la durée du flux à partir de l’heure de début). La valeur par défaut est 100 %, ce qui signifie qu’elle s’arrête à la fin du flux.
- **couches** : collection de couches d’images de sortie à produire par l’encodeur.

## <a name="example-of-a-single-png-file-preset"></a>Exemple de présélection d’un « fichier PNG unique »

La présélection JSON suivante peut servir à produire un fichier de sortie PNG unique à partir des premières secondes de la vidéo d’entrée, où l’encodeur effectue la meilleure tentative possible pour trouver une image « intéressante ». Notez que les dimensions de l’image de sortie ont été définies sur 100 %, ce qui signifie qu’elles correspondent à celles de la vidéo d’entrée. Notez également que le paramètre « Format » de la section « Outputs » doit correspondre à l’utilisation de « PngLayers » dans la section « Codecs ».  

```json
{
    "properties": {
        "description": "Basic Transform using a custom encoding preset for thumbnails",
        "outputs": [
            {
                "onError": "StopProcessingJob",
                "relativePriority": "Normal",
                "preset": {
                    "@odata.type": "#Microsoft.Media.StandardEncoderPreset",
                    "codecs": [
                        {
                            "@odata.type": "#Microsoft.Media.PngImage",
                            "stretchMode": "AutoSize",
                            "start": "{Best}",
                            "step": "25%",
                            "range": "80%",
                            "layers": [
                                {
                                    "width": "50%",
                                    "height": "50%"
                                }
                            ]
                        }
                    ],
                    "formats": [
                        {
                            "@odata.type": "#Microsoft.Media.Mp4Format",
                            "filenamePattern": "Video-{Basename}-{Label}-{Bitrate}{Extension}",
                            "outputFiles": []
                        },
                        {
                            "@odata.type": "#Microsoft.Media.PngFormat",
                            "filenamePattern": "Thumbnail-{Basename}-{Index}{Extension}"
                        }
                    ]
                }
            }
        ]
    }
}

```

## <a name="example-of-a-series-of-jpeg-images-preset"></a>Exemple de présélection d’une « série d’images JPEG »

La présélection JSON suivante peut servir à produire un ensemble de 10 images aux timestamps de 5 %, 15 %, ..., 95 % de la chronologie d’entrée, où la taille de l’image est définie pour représenter un quart de la vidéo d’entrée.

### <a name="json-preset"></a>Présélection JSON

```json
{
  "Version": 1.0,
  "Codecs": [
    {
      "JpgLayers": [
        {
          "Quality": 90,
          "Type": "JpgLayer",
          "Width": "25%",
          "Height": "25%"
        }
      ],
      "Start": "5%",
      "Step": "10%",
      "Range": "96%",
      "Type": "JpgImage"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}_{Index}{Extension}",
      "Format": {
        "Type": "JpgFormat"
      }
    }
  ]
}
```

## <a name="example-of-a-one-image-at-a-specific-timestamp-preset"></a>Exemple de présélection d’une « image à un horodatage spécifique »

La présélection JSON suivante peut servir à produire une image JPEG à la marque des 30 secondes de la vidéo d’entrée. Pour cette présélection, la vidéo d’entrée doit durer plus de 30 secondes (sinon le travail échoue).

### <a name="json-preset"></a>Présélection JSON

```json
{
  "Version": 1.0,
  "Codecs": [
    {
      "JpgLayers": [
        {
          "Quality": 90,
          "Type": "JpgLayer",
          "Width": "25%",
          "Height": "25%"
        }
      ],
      "Start": "00:00:30",
      "Step": "1",
      "Range": "1",
      "Type": "JpgImage"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}_{Index}{Extension}",
      "Format": {
        "Type": "JpgFormat"
      }
    }
  ]
}
```

## <a name="example-of-a-thumbnails-at-different-resolutions-preset"></a>Exemple de présélection de « miniatures avec différentes résolutions »

La présélection suivante peut servir à générer des miniatures avec différentes résolutions en une seule tâche. Dans cet exemple, à des positions de 5 %, 15 %, ..., 95 % de la chronologie d’entrée, l’encodeur génère deux images : l’une à 100 % de la résolution vidéo d’entrée et l’autre à 50 %.

Notez l’utilisation de la macro {Resolution} dans le nom de fichier. Elle donne à l’encodeur l’instruction d’utiliser la largeur et la hauteur que vous avez spécifiées dans la section sur l’encodage de la présélection lors de la génération du nom de fichier des images de sortie. Cela vous permet également de faire facilement la distinction entre les différentes images.

### <a name="json-preset"></a>Présélection JSON

```json
{
  "Version": 1.0,
  "Codecs": [
    {
      "JpgLayers": [
{
  "Quality": 90,
  "Type": "JpgLayer",
  "Width": "100%",
  "Height": "100%"
},
{
  "Quality": 90,
  "Type": "JpgLayer",
  "Width": "50%",
  "Height": "50%"
}

      ],
      "Start": "5%",
      "Step": "10%",
      "Range": "96%",
      "Type": "JpgImage"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}_{Resolution}_{Index}{Extension}",
      "Format": {
"Type": "JpgFormat"
      }
    }
  ]
}
```

## <a name="example-of-generating-a-thumbnail-while-encoding"></a>Exemple de génération d’une miniature durant l’encodage

Bien que tous les exemples précédents illustrent la façon de soumettre une tâche d’encodage qui génère uniquement des images, vous pouvez également combiner l’encodage audio/vidéo avec la génération de miniatures. La présélection JSON suivante indique à l’encodeur standard de générer une miniature au cours de l’encodage.

### <a name="json-preset"></a>Présélection JSON

Pour plus d’informations sur le schéma, consultez [cet](../previous/media-services-mes-schema.md) article.

```json
{
  "Version": 1.0,
  "Codecs": [
    {
      "KeyFrameInterval": "00:00:02",
      "SceneChangeDetection": "true",
      "H264Layers": [
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 4500,
          "MaxBitrate": 4500,
          "BufferWindow": "00:00:05",
          "Width": 1280,
          "Height": 720,
          "ReferenceFrames": 3,
          "EntropyMode": "Cabac",
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"

        }
      ],
      "Type": "H264Video"
    },
    {
      "JpgLayers": [
        {
          "Quality": 90,
          "Type": "JpgLayer",
          "Width": "100%",
          "Height": "100%"
        }
      ],
      "Start": "{Best}",
      "Type": "JpgImage"
    },
    {
      "Channels": 2,
      "SamplingRate": 48000,
      "Bitrate": 128,
      "Type": "AACAudio"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}_{Index}{Extension}",
      "Format": {
        "Type": "JpgFormat"
      }
    },
    {
      "FileName": "{Basename}_{Resolution}_{VideoBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

## <a name="next-steps"></a>Étapes suivantes
[Générer des miniatures avec .NET](media-services-generate-thumbnails-dotnet.md)