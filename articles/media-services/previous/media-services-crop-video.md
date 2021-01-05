---
title: Guide pratique pour rogner des vidéos avec Media Encoder Standard - Azure | Microsoft Docs
description: Le rognage consiste à sélectionner une fenêtre rectangulaire dans l’image vidéo et à encoder uniquement les pixels dans cette fenêtre. Cet article explique comment rogner des vidéos avec Media Encoder Standard.
services: media-services
documentationcenter: ''
author: anilmur
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/18/2019
ms.author: anilmur
ms.reviewer: juliako
ms.openlocfilehash: 4264d1c0d83f14da02b26107d336521250fcf13b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89257855"
---
# <a name="crop-videos-with-media-encoder-standard"></a>Rogner des vidéos avec l’encodeur multimédia standard

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

Vous pouvez utiliser Media Encoder Standard (MES) pour rogner votre vidéo d’entrée. Le rognage consiste à sélectionner une fenêtre rectangulaire dans l’image vidéo et à encoder uniquement les pixels dans cette fenêtre. Le schéma suivant permet d’illustrer le processus.

![Rogner une vidéo](./media/media-services-crop-video/media-services-crop-video01.png)

Vous disposez en tant qu’entrée d’une vidéo présentant une résolution de 1920 x 1080 pixels (proportions 16:9), avec des barres noires (pillarbox) à gauche et à droite, de manière à ce que seule une fenêtre de 4:3 ou 1440 x 1080 pixels puisse contenir une vidéo active. Vous pouvez utiliser MES pour rogner ou modifier les barres noires, et coder la région 1440 x 1080.

Le rognage dans MES étant une étape de prétraitement, les paramètres de rognage de la présélection d’encodage s’appliquent à la vidéo d’entrée d’origine. L’encodage s’effectue à un stade ultérieur, et les paramètres de largeur/hauteur s’appliquent à la vidéo *prétraitée* et non à la vidéo d’origine. Lorsque vous concevez votre présélection, vous devez effectuer les opérations suivantes : (a) sélectionnez les paramètres de rognage en fonction de la vidéo d’entrée d’origine et (b) sélectionnez vos paramètres d’encodage en fonction de la vidéo rognée. Si vos paramètres d’encodage ne correspondent pas à la vidéo rognée, le résultat ne répondra pas à vos attentes.

La rubrique [suivante](media-services-custom-mes-presets-with-dotnet.md#encoding_with_dotnet) montre comment créer une tâche d’encodage avec MES et comment spécifier une présélection personnalisée pour la tâche d’encodage. 

## <a name="creating-a-custom-preset"></a>Création d’une présélection personnalisée
Dans l’exemple ci-dessous :

1. L’entrée d’origine est 1920 x 1080.
2. Elle doit être rognée sur une sortie de 1440 x 1080, centrée dans le cadre d’entrée.
3. Cela implique un décalage X de (1920 – 1440)/2 = 240 et un décalage Y de zéro.
4. La largeur et la hauteur du rectangle de rognage sont de 1440 et 1080, respectivement.
5. Dans la phase de codage, la tâche consiste à produire trois couches avec des résolutions respectives de 1440 x 1080, 960 x 720 et 480 x 360.

### <a name="json-preset"></a>Présélection JSON

```json
{
  "Version": 1.0,
  "Sources": [
    {
      "Streams": [],
      "Filters": {
        "Crop": {
          "X": 240,
          "Y": 0,
          "Width": 1440,
          "Height": 1080
        }
      },
      "Pad": true
    }
  ],
  "Codecs": [
    {
      "KeyFrameInterval": "00:00:02",
      "H264Layers": [
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 3400,
          "MaxBitrate": 3400,
          "BufferWindow": "00:00:05",
          "Width": 1440,
          "Height": 1080,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 2250,
          "MaxBitrate": 2250,
          "BufferWindow": "00:00:05",
          "Width": 960,
          "Height": 720,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 1250,
          "MaxBitrate": 1250,
          "BufferWindow": "00:00:05",
          "Width": 480,
          "Height": 360,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        }
      ],
      "Type": "H264Video"
    },
    {
      "Profile": "AACLC",
      "Channels": 2,
      "SamplingRate": 48000,
      "Bitrate": 128,
      "Type": "AACAudio"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

## <a name="restrictions-on-cropping"></a>Restrictions sur le rognage
La fonctionnalité de rognage est destinée à être manuelle. Vous devez charger votre vidéo d’entrée dans un outil d’édition approprié qui vous permet de sélectionner des images d’intérêt, de positionner le curseur pour déterminer les décalages du rectangle de rognage, de déterminer que la présélection d’encodage est réglée pour une vidéo en particulier, etc. Cette fonctionnalité n’est pas destinée à activer des éléments tels que la détection automatique et la suppression des bordures noires letterbox/pillarbox de votre vidéo d’entrée.

Les contraintes suivantes s’appliquent à la fonctionnalité de rognage. Si elles ne sont pas remplies, la tâche de codage peut échouer ou produire un résultat inattendu.

1. Les coordonnées et la taille du rectangle de rognage doivent tenir dans la vidéo d’entrée.
2. Comme mentionné ci-dessus, la largeur et la hauteur dans les paramètres d’encodage doivent correspondre à la vidéo rognée.
3. Le rognage s’applique aux vidéos capturées en mode paysage (c’est-à-dire qu’il ne s’applique pas aux vidéos enregistrées avec un smartphone maintenu verticalement ou en mode portrait).
4. Fonctionne mieux avec une vidéo progressive capturée avec des pixels carrés.

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-step"></a>Étape suivante
Consultez les parcours d’apprentissage Azure Media Services pour en savoir plus sur les fonctionnalités exceptionnelles offertes par AMS.  

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]
