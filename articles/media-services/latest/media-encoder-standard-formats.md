---
title: Codecs et formats de Media Encoder Standard - Azure
description: Cet article contient la liste des formats de fichier d’importation et d’exportation les plus courants que vous pouvez utiliser avec StandardEncoderPreset.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.reviewer: anilmur
ms.openlocfilehash: f6e9c0a3d9100bc447314f5874107363c6bf5d8a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89289441"
---
# <a name="standard-encoder-formats-and-codecs"></a>Codecs et formats de Media Encoder Standard

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Cet article contient la liste des formats de fichier d’importation et d’exportation les plus courants que vous pouvez utiliser avec [StandardEncoderPreset](/rest/api/media/transforms/createorupdate#standardencoderpreset). Pour plus d’informations sur la façon de créer des préréglages personnalisés à l’aide de **StandardEncoderPreset**, consultez [Créer une transformation avec un préréglage personnalisé](customize-encoder-presets-how-to.md).

## <a name="input-containerfile-formats"></a>Formats de conteneurs/fichiers d’entrée

| Formats de fichier (extensions de fichier) | Prise en charge |
| --- | --- |
| FLV (avec les codecs H.264 et AAC) (.flv) |Oui |
| MXF    (.mxf) |Oui |
| GXF    (.gxf) |Oui |
| MPEG2-PS, MPEG2-TS, 3GP (.ts, .ps, .3gp, .3gpp, .mpg) |Oui |
| Windows Media Video (WMV)/ASF (.wmv, .asf) |Oui |
| AVI (8 bits/10 bits non compressé) (.avi) |Oui |
| MP4 (.mp4, .m4a, .m4v)/ISMV (.isma, .ismv) |Oui |
| [Microsoft Digital Video Recording (DVR-MS)](/previous-versions/windows/desktop/mstv/about-the-dvr-ms-file-format) (.dvr-ms) |Oui |
| Matroska/WebM (.mkv) |Oui |
| WAVE/WAV (.wav) |Oui |
| QuickTime (.mov) |Oui |

### <a name="audio-formats-in-input-containers"></a>Formats audio dans des conteneurs d’entrée

Media Encoder Standard prend en charge la transmission des formats audio suivants dans des conteneurs d’entrée :

* Fichiers MXF, GXF et QuickTime contenant des pistes audio avec des échantillons Interleaved Stereo ou 5.1

or

* Fichiers MXF, GXF et QuickTime où l’audio est transmis sous forme de pistes PCM distinctes, mais où le mappage de canaux (vers la stéréo ou 5.1) peut être déduit des métadonnées du fichier

## <a name="input-video-codecs"></a>Codecs vidéo d’entrée
| Codecs vidéo d’entrée | Prise en charge |
| --- | --- |
| AVC 8 bits/10 bits, jusqu'à 4:2:2, y compris AVCIntra |8 bits 4:2:0 et 4:2:2 |
| Avid DNxHD (dans MXF) |Oui |
| DVCPro/DVCProHD (dans MXF) |Oui |
| Vidéo numérique (dans les fichiers AVI) |Oui |
| JPEG 2000 |Oui |
| MPEG-2 (jusqu’au profil 422 et haut niveau ; y compris les variantes telles que XDCAM, XDCAM HD, XDCAM IMX, CableLabs® et D10) |Jusqu’à un profil de 422 |
| MPEG-1 |Oui |
| VC-1/WMV9 |Oui |
| Canopus HQ/HQX |Non |
| MPEG-4 partie 2 |Oui |
| [Theora](https://en.wikipedia.org/wiki/Theora) |Oui |
| YUV420 non compressé ou mezzanine |Oui |
| Apple ProRes 422 |Oui |
| Apple ProRes 422 LT |Oui |
| Apple ProRes 422 HQ |Oui |
| Apple ProRes Proxy |Oui |
| Apple ProRes 4444 |Oui |
| Apple ProRes 4444 XQ |Oui |
| HEVC/H.265| Profil Main|

## <a name="input-audio-codecs"></a>Codecs audio d’entrée
| Codecs audio d’entrée | Prise en charge |
| --- | --- |
| AAC (AAC-LC, AAC-HE et AAC-HEv2 ; jusqu’à 5.1) |Oui |
| MPEG Layer 2 |Oui |
| MP3 (MPEG-1 Audio Layer 3) |Oui |
| Windows Media Audio |Oui |
| WAV/PCM |Oui |
| [FLAC](https://en.wikipedia.org/wiki/FLAC)</a> |Oui |
| [Opus](https://go.microsoft.com/fwlink/?LinkId=822667) |Oui |
| [Vorbis](https://en.wikipedia.org/wiki/Vorbis)</a> |Oui |
| AMR (adaptive multi-rate) |Oui |
| AES (SMPTE 331M et 302M, AES3-2003) |Non |
| Dolby® E |Non |
| Dolby® Digital (AC3) |Non |
| Dolby® Digital Plus (E-AC3) |Non |

## <a name="output-formats-and-codecs"></a>Codecs et formats de sortie
Le tableau suivant répertorie les codecs et les formats de fichiers pris en charge pour l'exportation.

| Format de fichier | Codec vidéo | Codec audio |
| --- | --- | --- |
| MP4 <br/><br/>(y compris les conteneurs MP4 à vitesses de transmission multiples) |H.264 (profils High, Main et Baseline) |AAC-LC, HE-AAC v1, HE-AAC v2 |
| MPEG2-TS |H.264 (profils High, Main et Baseline) |AAC-LC, HE-AAC v1, HE-AAC v2 |

## <a name="next-steps"></a>Étapes suivantes

[Créer une transformation avec un préréglage personnalisé](customize-encoder-presets-how-to.md)
