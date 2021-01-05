---
title: Schéma de métadonnées de sortie Azure Media Services | Microsoft Docs
description: Cet article fournit une vue d’ensemble du schéma de métadonnées de sortie d’Azure Media Services v3.
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
ms.openlocfilehash: aa6d4edc4348fa850eeb7e8d91ce0791ee4c7170
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91336409"
---
# <a name="output-metadata"></a>Métadonnées de sortie

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Un travail d’encodage est associé à un élément multimédia d’entrée (ou plusieurs) sur lequel vous souhaitez effectuer des tâches d’encodage. Par exemple, encoder un fichier MP4 en ensembles de fichiers MP4 à vitesse de transmission adaptative H.264, créer une miniature, créer des superpositions. À l’achèvement d’une tâche, une ressource de sortie est générée.  L’élément multimédia de sortie contient de la vidéo, de l’audio, des miniatures et d’autres fichiers. L’élément multimédia de sortie contient également un fichier avec des métadonnées relatives à l’élément multimédia de sortie. Le nom du fichier JSON de métadonnées a le format suivant : `<source_file_name>_manifest.json` (par exemple, `BigBuckBunny_manifest.json`). Vous devez rechercher tout fichier *_metadata.json et interroger la chaîne du chemin d’accès au fichier pour trouver le nom de fichier source (sans troncation).

Media Services n’analyse pas de manière préemptive les éléments multimédias d’entrée pour générer des métadonnées. Les métadonnées d’entrée sont générées uniquement sous forme d’artefact quand un élément multimédia d’entrée est traité au sein d’un travail. Par conséquent, cet artefact est écrit dans l’élément multimédia de sortie. Différents outils sont utilisés pour générer des métadonnées pour les éléments multimédias d’entrée et de sortie. Ainsi, les métadonnées d’entrée présentent un schéma légèrement différent de celui des métadonnées de sortie.

Cet article décrit les éléments et types du schéma JSON sur lesquels les métadonnées de sortie (&lt;source_file_name&gt;_manifest.json) sont basées. <!--For information about the file that contains metadata about the input asset, see [Input metadata](input-metadata-schema.md).  -->

Vous pouvez trouver le code du schéma complet et un exemple de JSON à la fin de cet article.  

## <a name="assetfile"></a>AssetFile

Collection d’entrées AssetFile pour le travail d’encodage.  

| Nom | Description |
| --- | --- |
| **Sources** |Collection de fichiers multimédias d’entrée/source qui a été traitée afin de produire cet AssetFile.<br />Exemple : `"Sources": [{"Name": "Ignite-short_1280x720_AACAudio_3551.mp4"}]`|
| **VideoTracks**|Chaque élément AssetFile physique peut contenir zéro ou plusieurs pistes vidéo entrelacées dans un format de conteneur approprié. <br />Consultez [VideoTracks](#videotracks). |
| **AudioTracks**|Chaque élément AssetFile physique peut contenir zéro ou plusieurs pistes audio entrelacées dans un format de conteneur approprié. Il s’agit de la collection de toutes les pistes audio.<br /> Pour plus d’informations, consultez [AudioTracks](#audiotracks). |
| **Nom**<br />Obligatoire |Le nom du fichier multimédia. <br /><br />Exemple : `"Name": "Ignite-short_1280x720_AACAudio_3551.mp4"`|
| **Taille**<br />Obligatoire |Taille du fichier de ressource en octets. <br /><br />Exemple : `"Size": 32414631`|
| **Durée**<br />Obligatoire |Durée de lecture du contenu. Pour plus d’informations, consultez le format [ISO8601](https://www.iso.org/iso-8601-date-and-time-format.html). <br /><br />Exemple : `"Duration": "PT1M10.315S"`|

## <a name="videotracks"></a>VideoTracks 

Chaque élément AssetFile physique peut contenir zéro ou plusieurs pistes vidéo entrelacées dans un format de conteneur approprié. L’élément **VideoTracks** représente une collection de toutes les pistes vidéo.  

| Nom | Description |
| --- | --- |
| **Id**<br /> Obligatoire |Index de base zéro de cette piste vidéo. **Remarque :**  Cet **ID** ne correspond pas nécessairement au TrackID tel qu’utilisé dans un fichier MP4. <br /><br />Exemple : `"Id": 1`|
| **FourCC**<br />Obligatoire | Code FourCC du codec vidéo signalé par ffmpeg.  <br /><br />Exemple : `"FourCC": "avc1"`|
| **Profil** |Profil H264 (uniquement applicable au codec H264).  <br /><br />Exemple : `"Profile": "High"` |
| **Niveau** |Niveau H264 (uniquement applicable au codec H264).  <br /><br />Exemple : `"Level": "3.2"`|
| **Width**<br />Obligatoire |Largeur vidéo encodée en pixels.  <br /><br />Exemple : `"Width": "1280"`|
| **Height**<br />Obligatoire |Hauteur vidéo encodée en pixels.  <br /><br />Exemple : `"Height": "720"`|
| **DisplayAspectRatioNumerator**<br />Obligatoire|Numérateur des proportions d’affichage vidéo.  <br /><br />Exemple : `"DisplayAspectRatioNumerator": 16.0`|
| **DisplayAspectRatioDenominator**<br />Obligatoire |Dénominateur des proportions d’affichage vidéo.  <br /><br />Exemple : `"DisplayAspectRatioDenominator": 9.0`|
| **Framerate**<br />Obligatoire |Fréquence d’images vidéo mesurée au format .3f.  <br /><br />Exemple : `"Framerate": 29.970`|
| **Bitrate**<br />Obligatoire |Vitesse de transmission vidéo moyenne en bits par seconde, telle que calculée à partir de l’AssetFile. Compte uniquement la charge utile de flux élémentaire et n’inclut pas la surcharge de packaging.  <br /><br />Exemple : `"Bitrate": 3551567`|
| **TargetBitrate**<br />Obligatoire |Vitesse de transmission cible moyenne pour cette piste vidéo, comme demandé via les paramètres prédéfinis d’encodage, en bits par seconde. <br /><br />Exemple : `"TargetBitrate": 3520000` |

## <a name="audiotracks"></a>AudioTracks 

Chaque élément AssetFile physique peut contenir zéro ou plusieurs pistes audio entrelacées dans un format de conteneur approprié. L’élément **AudioTracks** représente une collection de toutes les pistes audio.  

| Nom  | Description |
| --- | --- |
| **Id**<br />Obligatoire  |Index de base zéro de cette piste audio. **Remarque :**  Il ne s’agit pas nécessairement du TrackID tel qu’utilisé dans un fichier MP4.  <br /><br />Exemple : `"Id": 2`|
| **Codec**  |Chaîne du codec de piste audio.  <br /><br />Exemple : `"Codec": "aac"`|
| **Langage**|Exemple : `"Language": "eng"`|
| **Canaux**<br />Obligatoire|Nombre de canaux audio.  <br /><br />Exemple : `"Channels": 2`|
| **SamplingRate**<br />Obligatoire |Taux d’échantillonnage audio en échantillons/sec ou Hz.  <br /><br />Exemple : `"SamplingRate": 48000`|
| **Bitrate**<br />Obligatoire |Débit binaire audio moyen en bits par seconde, tel que calculé à partir du fichier de ressource. Compte uniquement la charge utile de flux élémentaire et n’inclut pas la surcharge de packaging.  <br /><br />Exemple : `"Bitrate": 128041`|

## <a name="json-schema-example"></a>Exemple de schéma JSON

```json
{
  "AssetFile": [
    {
      "Sources": [
        {
          "Name": "Ignite-short_1280x720_AACAudio_3551.mp4"
        }
      ],
      "VideoTracks": [
        {
          "Id": 1,
          "FourCC": "avc1",
          "Profile": "High",
          "Level": "3.2",
          "Width": "1280",
          "Height": "720",
          "DisplayAspectRatioNumerator": 16.0,
          "DisplayAspectRatioDenominator": 9.0,
          "Framerate": 29.970,
          "Bitrate": 3551567,
          "TargetBitrate": 3520000
        }
      ],
      "AudioTracks": [
        {
          "Id": 2,
          "Codec": "aac",
          "Language": "eng",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 128041
        }
      ],
      "Name": "Ignite-short_1280x720_AACAudio_3551.mp4",
      "Size": 32414631,
      "Duration": "PT1M10.315S"
    },
    {
      "Sources": [
        {
          "Name": "Ignite-short_960x540_AACAudio_2216.mp4"
        }
      ],
      "VideoTracks": [
        {
          "Id": 1,
          "FourCC": "avc1",
          "Profile": "High",
          "Level": "3.1",
          "Width": "960",
          "Height": "540",
          "DisplayAspectRatioNumerator": 16.0,
          "DisplayAspectRatioDenominator": 9.0,
          "Framerate": 29.970,
          "Bitrate": 2216326,
          "TargetBitrate": 2210000
        }
      ],
      "AudioTracks": [
        {
          "Id": 2,
          "Codec": "aac",
          "Language": "eng",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 128041
        }
      ],
      "Name": "Ignite-short_960x540_AACAudio_2216.mp4",
      "Size": 20680897,
      "Duration": "PT1M10.315S"
    },
    {
      "Sources": [
        {
          "Name": "Ignite-short_640x360_AACAudio_1150.mp4"
        }
      ],
      "VideoTracks": [
        {
          "Id": 1,
          "FourCC": "avc1",
          "Profile": "High",
          "Level": "3.0",
          "Width": "640",
          "Height": "360",
          "DisplayAspectRatioNumerator": 16.0,
          "DisplayAspectRatioDenominator": 9.0,
          "Framerate": 29.970,
          "Bitrate": 1150440,
          "TargetBitrate": 1150000
        }
      ],
      "AudioTracks": [
        {
          "Id": 2,
          "Codec": "aac",
          "Language": "eng",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 128041
        }
      ],
      "Name": "Ignite-short_640x360_AACAudio_1150.mp4",
      "Size": 11313920,
      "Duration": "PT1M10.315S"
    },
    {
      "Sources": [
        {
          "Name": "Ignite-short_480x270_AACAudio_722.mp4"
        }
      ],
      "VideoTracks": [
        {
          "Id": 1,
          "FourCC": "avc1",
          "Profile": "High",
          "Level": "2.1",
          "Width": "480",
          "Height": "270",
          "DisplayAspectRatioNumerator": 16.0,
          "DisplayAspectRatioDenominator": 9.0,
          "Framerate": 29.970,
          "Bitrate": 722682,
          "TargetBitrate": 720000
        }
      ],
      "AudioTracks": [
        {
          "Id": 2,
          "Codec": "aac",
          "Language": "eng",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 128041
        }
      ],
      "Name": "Ignite-short_480x270_AACAudio_722.mp4",
      "Size": 7554708,
      "Duration": "PT1M10.315S"
    },
    {
      "Sources": [
        {
          "Name": "Ignite-short_320x180_AACAudio_380.mp4"
        }
      ],
      "VideoTracks": [
        {
          "Id": 1,
          "FourCC": "avc1",
          "Profile": "High",
          "Level": "1.3",
          "Width": "320",
          "Height": "180",
          "DisplayAspectRatioNumerator": 16.0,
          "DisplayAspectRatioDenominator": 9.0,
          "Framerate": 29.970,
          "Bitrate": 380655,
          "TargetBitrate": 380000
        }
      ],
      "AudioTracks": [
        {
          "Id": 2,
          "Codec": "aac",
          "Language": "eng",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 128041
        }`
      ],
      "Name": "Ignite-short_320x180_AACAudio_380.mp4",
      "Size": 4548932,
      "Duration": "PT1M10.315S"
    }
  ]
}
```

## <a name="next-steps"></a>Étapes suivantes

[Créer une entrée de travail à partir d’une URL HTTPS](job-input-from-http-how-to.md)
