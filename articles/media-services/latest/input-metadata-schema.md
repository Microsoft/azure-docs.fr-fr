---
title: Schéma de métadonnées d’entrée Azure Media Services v3
description: Cet article fournit une vue d’ensemble du schéma de métadonnées d’entrée d’Azure Media Services v3.
author: IngridAtMicrosoft
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 1ca526b7ecbe20a54ec115521cdfbc93c713e0da
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93360052"
---
# <a name="input-metadata"></a>Métadonnées d’entrée

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Un travail d’encodage est associé à un élément multimédia d’entrée (ou plusieurs) sur lequel vous souhaitez effectuer des tâches d’encodage.  À l’achèvement d’une tâche, une ressource de sortie est générée. L’élément multimédia de sortie contient vidéo, audio, miniatures, manifeste et autres fichiers. 

Il contient également un fichier avec des métadonnées relatives à l’élément multimédia d’entrée. Le nom du fichier JSON de métadonnées a un ID aléatoire, ne l’utilisez pas pour identifier la ressource d’entrée à laquelle appartient la ressource de sortie. Pour identifier la ressource d’entrée à laquelle elle appartient, utilisez le champ `Uri` (pour plus d’informations, consultez [Autres éléments enfants](#other-child-elements)).  

Media Services n’analyse pas de manière préemptive les éléments multimédias d’entrée pour générer des métadonnées. Les métadonnées d’entrée sont générées uniquement sous forme d’artefact quand un élément multimédia d’entrée est traité au sein d’un travail. Par conséquent, cet artefact est écrit dans l’élément multimédia de sortie. Différents outils sont utilisés pour générer des métadonnées pour les éléments multimédias d’entrée et de sortie. Ainsi, les métadonnées d’entrée présentent un schéma légèrement différent de celui des métadonnées de sortie.

Cet article décrit les éléments et types du schéma JSON sur lesquels les métadonnées d’entrée (&lt;asset_id&gt;_metadata.json) sont basées. Pour plus d’informations sur le fichier qui contient des métadonnées sur la ressource de sortie, consultez [Métadonnées de sortie](output-metadata-schema.md).  

Vous trouverez l’exemple de JSON à la fin de cet article.  

## <a name="assetfile"></a>AssetFile  

Contient une collection d’éléments AssetFile pour le travail d’encodage.  

> [!NOTE]
> Les quatre éléments enfants suivants doivent apparaître dans une séquence.  
> 
> 

| Nom  | Description |
| --- | --- | 
| **VideoTracks**|Chaque élément AssetFile physique peut contenir zéro, une ou plusieurs pistes vidéo entrelacées dans un format de conteneur approprié. Pour plus d’informations, consultez [VideoTracks](#videotracks). |
| **AudioTracks**|Chaque élément AssetFile physique peut contenir zéro ou plusieurs pistes audio entrelacées dans un format de conteneur approprié. Pour plus d’informations, consultez [AudioTracks](#audiotracks) |
| **Métadonnées**  |Les métadonnées du fichier de ressource représentées sous la forme de chaînes clé/valeur. <br />Par exemple : `<Metadata key="language" value="eng" />` |

### <a name="other-child-elements"></a>Autres éléments enfants

| Nom | Description |
| --- | --- |
| **Nom**<br />Obligatoire |Nom du fichier de ressource. <br /><br />Exemple : `"Name": "Ignite-short.mp4"` |
| **Uri**<br />Obligatoire |URL où se trouve l’élément multimédia d’entrée. Pour identifier la ressource d’entrée à laquelle appartient la ressource de sortie, utilisez le champ `Uri` à la place de l’ID.|
| **Taille**<br />Obligatoire |Taille du fichier de ressource en octets.  <br /><br />Exemple : `"Size": 75739259`|
| **Durée**<br />Obligatoire |Durée de lecture du contenu. <br /><br />Exemple : `"Duration": "PT1M10.304S"`. |
| **NumberOfStreams**<br />Obligatoire |Nombre de flux dans le fichier de ressource.  <br /><br />Exemple : `"NumberOfStreams": 2`|
| **FormatNames**<br />Obligatoire |Noms de format.  <br /><br />Exemple : `"FormatNames": "mov,mp4,m4a,3gp,3g2,mj2"`|
| **FormatVerboseName**<br /> Obligatoire |Noms détaillés des formats. <br /><br />Exemple : `"FormatVerboseName": "QuickTime / MOV"` |
| **StartTime** |Heure de début du contenu.  <br /><br />Exemple : `"StartTime": "PT0S"` |
| **OverallBitRate** |Vitesse de transmission moyenne du fichier de ressource en bits par seconde.  <br /><br />Exemple : `"OverallBitRate": 8618539`|

## <a name="videotracks"></a>VideoTracks

| Nom | Description |
| --- | --- |
| **FourCC**<br />Obligatoire |Code FourCC du codec vidéo signalé par ffmpeg.<br /><br />Exemple : `"FourCC": "avc1"` |
| **Profil** |Profil de la piste vidéo. <br /><br />Exemple : `"Profile": "Main"`|
| **Niveau** |Niveau de la piste vidéo. <br /><br />Exemple : `"Level": "3.2"`|
| **PixelFormat** |Format de pixel de la piste vidéo. <br /><br />Exemple : `"PixelFormat": "yuv420p"`|
| **Width**<br />Obligatoire |Largeur vidéo encodée en pixels. <br /><br />Exemple : `"Width": "1280"`|
| **Height**<br />Obligatoire |Hauteur vidéo encodée en pixels.<br /><br />Exemple : `"Height": "720"` |
| **DisplayAspectRatioNumerator**<br />Obligatoire |Numérateur des proportions d’affichage vidéo.<br /><br />Exemple : `"DisplayAspectRatioNumerator": 16.0` |
| **DisplayAspectRatioDenominator**<br />Obligatoire |Dénominateur des proportions d’affichage vidéo. <br /><br />Exemple : `"DisplayAspectRatioDenominator": 9.0`|
| **SampleAspectRatioNumerator** |Numérateur des proportions d’échantillon vidéo. <br /><br />Exemple : `"SampleAspectRatioNumerator": 1.0`|
| **SampleAspectRatioDenominator**|Exemple : `"SampleAspectRatioDenominator": 1.0`|
| **FrameRate**<br />Obligatoire |Fréquence d’images vidéo mesurée au format .3f. <br /><br />Exemple : `"FrameRate": 29.970`|
| **Bitrate** |Vitesse de transmission vidéo moyenne en bits par seconde, telle que calculée à partir du fichier de ressource. Seule la charge utile de flux élémentaire est comptée et la surcharge de packaging n’est pas incluse. <br /><br />Exemple : `"Bitrate": 8421583`|
| **HasBFrames** |Numéro de piste vidéo des trames B. <br /><br />Exemple : `"HasBFrames": 2`|
| **Métadonnées** |Des chaînes clé/valeur génériques qui peuvent être utilisées pour contenir différents types d’informations. <br />Consultez l’exemple complet à la fin de l’article. |
| **Id**<br />Obligatoire |Index de base zéro de cette piste audio ou vidéo.<br /><br /> Cet **ID** ne correspond pas nécessairement au TrackID tel qu’utilisé dans un fichier MP4. <br /><br />Exemple : `"Id": 2`|
| **Codec** |Chaîne de codec de la piste vidéo. <br /><br />Exemple : `"Codec": "h264"`|
| **CodecLongName** |Nom long du codec de piste audio ou vidéo. <br /><br />Exemple : `"CodecLongName": "H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10"`|
| **Codec** |Chaîne de codec de la piste vidéo. <br /><br />Exemple : `"Codec": "h264"`|
| **TimeBase**<br />Obligatoire |Période.<br /><br />Exemple : `"TimeBase": "1/30000"`|
| **NumberOfFrames** |Nombre de trames (présent pour les pistes vidéo). <br /><br />Exemple : `"NumberOfFrames": 2107`|
| **StartTime** |Suivre l’heure de début.<br /><br />Exemple : `"StartTime": "PT0.033S"` |
| **Durée** |Durée de la piste. <br /><br />Exemple : `"Duration": "PT1M10.304S"`|

## <a name="audiotracks"></a>AudioTracks

| Nom  | Description |
| --- | --- | 
| **SampleFormat** |Exemple de format. <br /><br />Exemple : `"SampleFormat": "fltp"`|
| **ChannelLayout** |Disposition de canal. <br /><br />Exemple : `"ChannelLayout": "stereo"`|
| **Canaux**<br />Obligatoire |Nombre (0 ou plus) de canaux audio. <br /><br />Exemple : `"Channels": 2`|
| **SamplingRate**<br />Obligatoire |Taux d’échantillonnage audio en échantillons/sec ou Hz. <br /><br />Exemple : `"SamplingRate": 48000`|
| **Bitrate** |Débit binaire audio moyen en bits par seconde, tel que calculé à partir du fichier de ressource. Seule la charge utile de flux élémentaire est comptée et la surcharge de packaging n’est pas incluse dans ce nombre. <br /><br />Exemple : `"Bitrate": 192080`|
| **Métadonnées** |Des chaînes clé/valeur génériques qui peuvent être utilisées pour contenir différents types d’informations.  <br />Consultez l’exemple complet à la fin de l’article. |
| **Id**<br />Obligatoire |Index de base zéro de cette piste audio ou vidéo.<br /><br /> Il ne s’agit pas nécessairement du trackid tel qu’utilisé dans un fichier MP4. <br /><br />Exemple : `"Id": 1`|
| **Codec** |Chaîne de codec de la piste vidéo. <br /><br />Exemple : `"Codec": "aac"`|
| **CodecLongName** |Nom long du codec de piste audio ou vidéo. <br /><br />Exemple : `"CodecLongName": "AAC (Advanced Audio Coding)"`|
| **TimeBase**<br />Obligatoire |Période.<br /><br />Exemple : `"TimeBase": "1/48000"` |
| **NumberOfFrames** |Nombre de trames (présent pour les pistes vidéo). <br /><br />Exemple : `"NumberOfFrames": 3294`|
| **StartTime** |Suivre l’heure de début. Pour plus d'informations, consultez [ISO8601](https://www.iso.org/iso-8601-date-and-time-format.html). <br /><br />Exemple : `"StartTime": "PT0S"` |
| **Durée** |Durée de la piste. <br /><br />Exemple : `"Duration": "PT1M10.272S"` |

## <a name="metadata"></a>Métadonnées

| Nom | Description |
| --- | --- |
| **key**<br />Obligatoire |La clé dans la paire clé/valeur. |
| **value**<br /> Obligatoire |La valeur dans la paire clé/valeur. |

## <a name="schema-example"></a>Exemple de schéma

```json
{
  "AssetFile": [
    {
      "VideoTracks": [
        {
          "FourCC": "avc1",
          "Profile": "Main",
          "Level": "3.2",
          "PixelFormat": "yuv420p",
          "Width": "1280",
          "Height": "720",
          "DisplayAspectRatioNumerator": 16.0,
          "DisplayAspectRatioDenominator": 9.0,
          "SampleAspectRatioNumerator": 1.0,
          "SampleAspectRatioNumeratorSpecified": true,
          "SampleAspectRatioDenominator": 1.0,
          "SampleAspectRatioDenominatorSpecified": true,
          "FrameRate": 29.970,
          "Bitrate": 8421583,
          "BitrateSpecified": true,
          "HasBFrames": 2,
          "HasBFramesSpecified": true,
          "Disposition": {
            "Default": 1
          },
          "Metadata": [
            {
              "key": "creation_time",
              "value": "2018-02-21T21:42:08.000000Z"
            },
            {
              "key": "language",
              "value": "eng"
            },
            {
              "key": "handler_name",
              "value": "Video Media Handler"
            },
            {
              "key": "encoder",
              "value": "AVC Coding"
            }
          ],
          "Id": 2,
          "Codec": "h264",
          "CodecLongName": "H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10",
          "TimeBase": "1/30000",
          "NumberOfFrames": 2107,
          "NumberOfFramesSpecified": true,
          "StartTime": "PT0.033S",
          "Duration": "PT1M10.304S"
        }
      ],
      "AudioTracks": [
        {
          "SampleFormat": "fltp",
          "ChannelLayout": "stereo",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 192080,
          "BitrateSpecified": true,
          "BitsPerSampleSpecified": true,
          "Disposition": {
            "Default": 1
          },
          "Metadata": [
            {
              "key": "creation_time",
              "value": "2018-02-21T21:42:08.000000Z"
            },
            {
              "key": "language",
              "value": "eng"
            },
            {
              "key": "handler_name",
              "value": "Sound Media Handler"
            }
          ],
          "Id": 1,
          "Codec": "aac",
          "CodecLongName": "AAC (Advanced Audio Coding)",
          "TimeBase": "1/48000",
          "NumberOfFrames": 3294,
          "NumberOfFramesSpecified": true,
          "StartTime": "PT0S",
          "Duration": "PT1M10.272S"
        }
      ],
      "Metadata": [
        {
          "key": "major_brand",
          "value": "mp42"
        },
        {
          "key": "minor_version",
          "value": "19529854"
        },
        {
          "key": "compatible_brands",
          "value": "mp42isom"
        },
        {
          "key": "creation_time",
          "value": "2018-02-21T21:42:08.000000Z"
        }
      ],
      "Name": "Ignite-short.mp4",
      "Uri": "https://amsstorageacct.blob.core.windows.net/asset-00000000-0000-0000-000000000000/ignite.mp4",
      "Size": 75739259,
      "Duration": "PT1M10.304S",
      "NumberOfStreams": 2,
      "FormatNames": "mov,mp4,m4a,3gp,3g2,mj2",
      "FormatVerboseName": "QuickTime / MOV",
      "StartTime": "PT0S",
      "OverallBitRate": 8618539,
      "OverallBitRateSpecified": true
    }
  ]
}
```

## <a name="next-steps"></a>Étapes suivantes

[Métadonnées de sortie](output-metadata-schema.md)
