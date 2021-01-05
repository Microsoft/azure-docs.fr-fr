---
title: Effectuer un encodage avancé en personnalisant les présélections MES | Microsoft Docs
description: Cette rubrique explique comment effectuer un encodage avancé en personnalisant les présélections de tâches Media Encoder Standard.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 2a4ade25-e600-4bce-a66e-e29cf4a38369
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 4288e2e955f8205f3b6551c83a5c883eecf02501
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021144"
---
# <a name="perform-advanced-encoding-by-customizing-mes-presets"></a>Effectuer un encodage avancé en personnalisant les présélections MES

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Vue d’ensemble

Cette rubrique montre comment personnaliser des présélections Media Encoder Standard. La rubrique [Encodage avec Media Encoder Standard à l’aide de présélections personnalisées](media-services-custom-mes-presets-with-dotnet.md) explique comment utiliser .NET pour créer une tâche de codage et une tâche qui exécute cette tâche. Une fois que vous avez personnalisé une présélection, fournissez les présélections personnalisées pour la tâche d’encodage. 

Si vous utilisez une présélection XML, veillez à conserver l’ordre des éléments, comme indiqué dans les exemples XML ci-dessous (par exemple, KeyFrameInterval doit précéder SceneChangeDetection).

> [!NOTE] 
> La plupart des fonctionnalités Media Services v2 avancées de Media Encoder Standard ne sont actuellement pas disponibles dans v3. Pour plus d’informations, consultez les [différences de fonctionnalités](../latest/media-services-v2-vs-v3.md#feature-gaps-with-respect-to-v2-apis).

## <a name="support-for-relative-sizes"></a>Prise en charge des tailles relatives

Lors de la génération de miniatures, il est inutile de toujours spécifier la largeur et la hauteur de la sortie en pixels. Vous pouvez les spécifier en pourcentages, dans la plage [1 %,..., 100 %].

### <a name="json-preset"></a>Présélection JSON

```json
"Width": "100%",
"Height": "100%"
```

### <a name="xml-preset"></a>Présélection XML

```xml
<Width>100%</Width>
<Height>100%</Height>
```

## <a name="generate-thumbnails"></a>Génération de miniatures

Cette section montre comment personnaliser une présélection qui génère des miniatures. La présélection définie ci-dessous contient des informations sur la façon dont vous souhaitez encoder votre fichier, ainsi que les informations nécessaires à la génération des miniatures. Vous pouvez utiliser l’une des présélections MES documentées dans [cette](media-services-mes-presets-overview.md) section et ajouter le code qui génère des miniatures.  

> [!NOTE]
> Le paramètre **SceneChangeDetection** figurant dans la présélection qui suit ne peut présenter la valeur true que si votre encodage porte sur une vidéo à vitesse de transmission unique. Si votre encodage s’applique à une vidéo à plusieurs vitesses de transmission et que vous définissez **SceneChangeDetection** sur True, l’encodeur renvoie une erreur.  
>
>

Pour plus d’informations sur le schéma, consultez [cette](media-services-mes-schema.md) rubrique.

Assurez-vous d’examiner la section [Considérations](#considerations) .

### <a name="json-preset"></a>Présélection JSON

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
          "Width": 640,
          "Height": 360
        }
      ],
      "Start": "{Best}",
      "Type": "JpgImage"
    },
    {
      "PngLayers": [
        {
          "Type": "PngLayer",
          "Width": 640,
          "Height": 360,
        }
      ],
      "Start": "00:00:01",
      "Step": "00:00:10",
      "Range": "00:00:58",
      "Type": "PngImage"
    },
    {
      "BmpLayers": [
        {
          "Type": "BmpLayer",
          "Width": 640,
          "Height": 360
        }
      ],
      "Start": "10%",
      "Step": "10%",
      "Range": "90%",
      "Type": "BmpImage"
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
      "FileName": "{Basename}_{Index}{Extension}",
      "Format": {
        "Type": "PngFormat"
      }
    },
    {
      "FileName": "{Basename}_{Index}{Extension}",
      "Format": {
        "Type": "BmpFormat"
      }
    },
    {
      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

### <a name="xml-preset"></a>Présélection XML

```xml
<?xml version="1.0" encoding="utf-16"?>
<Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">
  <Encoding>
    <H264Video>
      <KeyFrameInterval>00:00:02</KeyFrameInterval>
      <SceneChangeDetection>true</SceneChangeDetection>
      <H264Layers>
        <H264Layer>
          <Bitrate>4500</Bitrate>
          <Width>1280</Width>
          <Height>720</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>4500</MaxBitrate>
        </H264Layer>
      </H264Layers>
    </H264Video>
    <AACAudio>
      <Profile>AACLC</Profile>
      <Channels>2</Channels>
      <SamplingRate>48000</SamplingRate>
      <Bitrate>128</Bitrate>
    </AACAudio>
    <JpgImage Start="{Best}">
      <JpgLayers>
        <JpgLayer>
          <Width>640</Width>
          <Height>360</Height>
          <Quality>90</Quality>
        </JpgLayer>
      </JpgLayers>
    </JpgImage>
    <BmpImage Start="10%" Step="10%" Range="90%">
      <BmpLayers>
        <BmpLayer>
          <Width>640</Width>
          <Height>360</Height>
        </BmpLayer>
      </BmpLayers>
    </BmpImage>
    <PngImage Start="00:00:01" Step="00:00:10" Range="00:00:58">
      <PngLayers>
        <PngLayer>
          <Width>640</Width>
          <Height>360</Height>
        </PngLayer>
      </PngLayers>
    </PngImage>
  </Encoding>
  <Outputs>
    <Output FileName="{Basename}_{Width}x{Height}_{VideoBitrate}.mp4">
      <MP4Format />
    </Output>
    <Output FileName="{Basename}_{Index}{Extension}">
      <JpgFormat />
    </Output>
    <Output FileName="{Basename}_{Index}{Extension}">
      <BmpFormat />
    </Output>
    <Output FileName="{Basename}_{Index}{Extension}">
      <PngFormat />
    </Output>
  </Outputs>
</Preset>
```

### <a name="considerations"></a>Considérations

Les considérations suivantes s'appliquent :

* L’utilisation d’horodatages explicites pour Début/Étape/Plage suppose que la source d’entrée a une longueur minimale de 1 minute.
* Les éléments Jpg/Png/BmpImage possèdent les attributs de chaîne Start, Step et Range, qui peuvent être interprétés comme suit :

  * Entiers non négatifs : nombre d'images, par exemple "Start": "120",
  * Présence du suffixe % : durée par rapport à la source, par exemple "Start": "15%", OU
  * Horodatage, s’il est exprimé au format HH:MM:SS, par exemple "Start" : "00:01:00"

    Vous pouvez combiner et apparier les notations à votre guise.

    En outre, Start prend également en charge une macro spéciale, {Best}, qui tente de déterminer la première image de contenu « intéressante ». REMARQUE : (Step et Range sont ignorés quand Start est défini sur {Best})
  * Configuration par défaut : Start:{Best}
* Le format de sortie doit être explicitement fourni pour chaque format d'image : Jpg/Png/BmpFormat. Quand il est présent, MES fait correspondre JpgVideo à JpgFormat et ainsi de suite. OutputFormat introduit une nouvelle macro spécifique au codec d’image, {Index}, qui doit être présente (une fois seulement) pour les formats de sortie d’image.

## <a name="trim-a-video-clipping"></a><a id="trim_video"></a>Rognage d’une vidéo (extrait)
Cette section explique comment modifier les présélections de l’encodeur pour découper ou rogner la vidéo d’entrée, dans laquelle l’entrée est ce que l’on appelle un fichier mezzanine ou un fichier à la demande. L’encodeur peut également servir à découper ou rogner un élément multimédia capturé ou archivé à partir d’un streaming en direct. Pour obtenir des détails à ce sujet, consultez [ce blog](https://azure.microsoft.com/blog/sub-clipping-and-live-archive-extraction-with-media-encoder-standard/).

Pour découper vos vidéos, vous pouvez utiliser l’une des présélections MES documentées dans [cette](media-services-mes-presets-overview.md) section et modifier l’élément **Sources** (comme indiqué ci-dessous). La valeur de StartTime doit correspondre aux horodatages absolus de la vidéo d'entrée. Par exemple, si la première image de la vidéo d'entrée a un horodatage de 12:00:10.000, la valeur de StartTime doit être égale ou supérieure à 12:00:10.000. Dans l'exemple ci-dessous, nous supposons que la vidéo d'entrée a un horodatage de début égal à zéro. **Sources** doit être placé au début de la présélection.

### <a name="json-preset"></a><a id="json"></a>Présélection JSON

```json
{
  "Version": 1.0,
  "Sources": [
    {
      "StartTime": "00:00:04",
      "Duration": "00:00:16"
    }
  ],
  "Codecs": [
    {
      "KeyFrameInterval": "00:00:02",
      "StretchMode": "AutoSize",
      "H264Layers": [
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 3400,
          "MaxBitrate": 3400,
          "BufferWindow": "00:00:05",
          "Width": 1280,
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
          "Bitrate": 2250,
          "MaxBitrate": 2250,
          "BufferWindow": "00:00:05",
          "Width": 960,
          "Height": 540,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 1500,
          "MaxBitrate": 1500,
          "BufferWindow": "00:00:05",
          "Width": 960,
          "Height": 540,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 1000,
          "MaxBitrate": 1000,
          "BufferWindow": "00:00:05",
          "Width": 640,
          "Height": 360,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 650,
          "MaxBitrate": 650,
          "BufferWindow": "00:00:05",
          "Width": 640,
          "Height": 360,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 400,
          "MaxBitrate": 400,
          "BufferWindow": "00:00:05",
          "Width": 320,
          "Height": 180,
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

### <a name="xml-preset"></a>Présélection XML
Pour découper vos vidéos, vous pouvez utiliser l’une des présélections MES documentées [ici](media-services-mes-presets-overview.md) et modifier l’élément **Sources** (comme indiqué ci-dessous).

```xml
<?xml version="1.0" encoding="utf-16"?>
<Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">
  <Sources>
    <Source StartTime="PT4S" Duration="PT14S"/>
  </Sources>
  <Encoding>
    <H264Video>
      <KeyFrameInterval>00:00:02</KeyFrameInterval>
      <H264Layers>
        <H264Layer>
          <Bitrate>3400</Bitrate>
          <Width>1280</Width>
          <Height>720</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>3400</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>2250</Bitrate>
          <Width>960</Width>
          <Height>540</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>2250</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>1500</Bitrate>
          <Width>960</Width>
          <Height>540</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>1500</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>1000</Bitrate>
          <Width>640</Width>
          <Height>360</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>1000</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>650</Bitrate>
          <Width>640</Width>
          <Height>360</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>650</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>400</Bitrate>
          <Width>320</Width>
          <Height>180</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>400</MaxBitrate>
        </H264Layer>
      </H264Layers>
    </H264Video>
    <AACAudio>
      <Profile>AACLC</Profile>
      <Channels>2</Channels>
      <SamplingRate>48000</SamplingRate>
      <Bitrate>128</Bitrate>
    </AACAudio>
  </Encoding>
  <Outputs>
    <Output FileName="{Basename}_{Width}x{Height}_{VideoBitrate}.mp4">
      <MP4Format />
    </Output>
  </Outputs>
</Preset>
```

## <a name="create-an-overlay"></a><a id="overlay"></a>Création d’une superposition

Media Encoder Standard vous permet de superposer une image sur une vidéo existante. Les formats suivants sont actuellement pris en charge : png, jpg, gif et bmp. La présélection définie ci-dessous illustre un exemple de superposition vidéo de base.

Après avoir défini un fichier de présélection, vous devez également indiquer à Media Services quel fichier de la ressource représente l’image de superposition et quel fichier représente la vidéo source sur laquelle vous souhaitez superposer l’image. Le fichier vidéo doit être le fichier **principal** .

Si vous utilisez .NET, ajoutez les deux fonctions suivantes à l’exemple .NET défini dans [cette](media-services-custom-mes-presets-with-dotnet.md#encoding_with_dotnet) rubrique. La fonction **UploadMediaFilesFromFolder** charge les fichiers d'un dossier (par exemple, BigBuckBunny.mp4 et Image001.png) et définit le fichier mp4 comme fichier principal dans la ressource. La fonction **EncodeWithOverlay** utilise le fichier de présélection personnalisé qu’elle a reçu (par exemple, la présélection suivante) pour créer la tâche d’encodage.

```csharp
static public IAsset UploadMediaFilesFromFolder(string folderPath)
{
    IAsset asset = _context.Assets.CreateFromFolder(folderPath, AssetCreationOptions.None);

    foreach (var af in asset.AssetFiles)
    {
        // The following code assumes 
        // you have an input folder with one MP4 and one overlay image file.
        if (af.Name.Contains(".mp4"))
            af.IsPrimary = true;
        else
            af.IsPrimary = false;

        af.Update();
    }

    return asset;
}

static public IAsset EncodeWithOverlay(IAsset assetSource, string customPresetFileName)
{
    // Declare a new job.
    IJob job = _context.Jobs.Create("Media Encoder Standard Job");
    // Get a media processor reference, and pass to it the name of the 
    // processor to use for the specific task.
    IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

    // Load the XML (or JSON) from the local file.
    string configuration = File.ReadAllText(customPresetFileName);

    // Create a task
    ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
        processor,
        configuration,
        TaskOptions.None);

    // Specify the input assets to be encoded.
    // This asset contains a source file and an overlay file.
    task.InputAssets.Add(assetSource);

    // Add an output asset to contain the results of the job. 
    task.OutputAssets.AddNew("Output asset",
        AssetCreationOptions.None);

    job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
    job.Submit();
    job.GetExecutionProgressTask(CancellationToken.None).Wait();

    return job.OutputMediaAssets[0];
}
```

> [!NOTE]
> Limitations actuelles :
>
> Le paramètre d’opacité de superposition n’est pas pris en charge.
>
> Votre fichier vidéo source et le fichier d'image de superposition doivent être dans le même élément multimédia, et le fichier vidéo doit être défini en tant que fichier principal dans cet élément multimédia.
>
>

### <a name="json-preset"></a>Présélection JSON

```json
{
  "Version": 1.0,
  "Sources": [
    {
      "Streams": [],
      "Filters": {
        "VideoOverlay": {
          "Position": {
            "X": 100,
            "Y": 100,
            "Width": 100,
            "Height": 50
          },
          "AudioGainLevel": 0.0,
          "MediaParams": [
            {
              "OverlayLoopCount": 1
            },
            {
              "IsOverlay": true,
              "OverlayLoopCount": 1
            }
          ],
          "Source": "Image001.png",
          "Clip": {
            "Duration": "00:00:05"
          },
          "FadeInDuration": {
            "Duration": "00:00:01"
          },
          "FadeOutDuration": {
            "StartTime": "00:00:03",
            "Duration": "00:00:04"
          }
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
          "Bitrate": 1045,
          "MaxBitrate": 1045,
          "BufferWindow": "00:00:05",
          "ReferenceFrames": 3,
          "EntropyMode": "Cavlc",
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "Width": "640",
          "Height": "360",
          "FrameRate": "0/1"
        }
      ],
      "Type": "H264Video"
    },
    {
      "Type": "CopyAudio"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}{Extension}",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

### <a name="xml-preset"></a>Présélection XML

```xml
<?xml version="1.0" encoding="utf-16"?>
<Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">
  <Sources>
    <Source>
      <Streams />
      <Filters>
        <VideoOverlay>
          <Source>Image001.png</Source>
          <Clip Duration="PT5S" />
          <FadeInDuration Duration="PT1S" />
          <FadeOutDuration StartTime="PT3S" Duration="PT4S" />
          <Position X="100" Y="100" Width="100" Height="50" />
          <Opacity>0</Opacity>
          <AudioGainLevel>0</AudioGainLevel>
          <MediaParams>
            <MediaParam>
              <IsOverlay>false</IsOverlay>
              <OverlayLoopCount>1</OverlayLoopCount>
            </MediaParam>
            <MediaParam>
              <IsOverlay>true</IsOverlay>
              <OverlayLoopCount>1</OverlayLoopCount>
            </MediaParam>
          </MediaParams>
        </VideoOverlay>
      </Filters>
      <Pad>true</Pad>
    </Source>
  </Sources>
  <Encoding>
    <H264Video>
      <KeyFrameInterval>00:00:02</KeyFrameInterval>
      <H264Layers>
        <H264Layer>
          <Bitrate>1045</Bitrate>
          <Width>640</Width>
          <Height>360</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>0</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cavlc</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>1045</MaxBitrate>
        </H264Layer>
      </H264Layers>
    </H264Video>
    <CopyAudio />
  </Encoding>
  <Outputs>
    <Output FileName="{Basename}{Extension}">
      <MP4Format />
    </Output>
  </Outputs>
</Preset>
```

## <a name="insert-a-silent-audio-track-when-input-has-no-audio"></a><a id="silent_audio"></a>Insertion d’une piste audio en mode silencieux lorsque l’entrée ne produit pas de son
Par défaut, si vous envoyez à l’encodeur une entrée contenant uniquement de la vidéo (sans contenu audio), l’élément multimédia de sortie regroupe les fichiers qui contiennent uniquement des données vidéo. Certains lecteurs ne sont peut-être pas capables de gérer ces flux de sortie. Dans ce cas, vous pouvez utiliser ce paramètre pour forcer l’encodeur à ajouter à la sortie une piste audio en mode silencieux.

Pour forcer l’encodeur à produire un élément multimédia contenant une piste audio en mode silencieux lorsque l’entrée ne comporte pas de son, spécifiez la valeur « InsertSilenceIfNoAudio ».

Vous pouvez utiliser l’une des présélections MES documentées dans [cette](media-services-mes-presets-overview.md) section et apporter la modification suivante :

### <a name="json-preset"></a>Présélection JSON

```json
{
  "Channels": 2,
  "SamplingRate": 44100,
  "Bitrate": 96,
  "Type": "AACAudio",
  "Condition": "InsertSilenceIfNoAudio"
}
```

### <a name="xml-preset"></a>Présélection XML

```xml
<AACAudio Condition="InsertSilenceIfNoAudio">
  <Channels>2</Channels>
  <SamplingRate>44100</SamplingRate>
  <Bitrate>96</Bitrate>
</AACAudio>
```

## <a name="disable-auto-de-interlacing"></a><a id="deinterlacing"></a>Désactiver le désentrelacement automatique
Si les clients souhaitent que le contenu d’entrelacement soit automatiquement désentrelacé, aucune action n’est nécessaire. Quand le désentrelacement automatique est activé (par défaut), MES détecte automatiquement les images entrelacées et désentrelace uniquement les images marquées comme entrelacées.

Vous pouvez désactiver le désentrelacement automatique. Cette option n’est pas recommandée.

### <a name="json-preset"></a>Présélection JSON

```json
"Sources": [
  {
    "Filters": {
      "Deinterlace": {
        "Mode": "Off"
      }
    },
  }
]
```

### <a name="xml-preset"></a>Présélection XML

```xml
<Sources>
  <Source>
    <Filters>
      <Deinterlace>
        <Mode>Off</Mode>
      </Deinterlace>
    </Filters>
  </Source>
</Sources>
```

## <a name="audio-only-presets"></a><a id="audio_only"></a>Présélections audio uniquement
Cette section présente deux présélections MES audio uniquement : Audio AAC et Bonne qualité audio AAC.

### <a name="aac-audio"></a>Audio AAC

```json
{
  "Version": 1.0,
  "Codecs": [
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
      "FileName": "{Basename}_AAC_{AudioBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

### <a name="aac-good-quality-audio"></a>Bonne qualité audio AAC

```json
{
  "Version": 1.0,
  "Codecs": [
    {
      "Profile": "AACLC",
      "Channels": 2,
      "SamplingRate": 48000,
      "Bitrate": 192,
      "Type": "AACAudio"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}_AAC_{AudioBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

## <a name="concatenate-two-or-more-video-files"></a><a id="concatenate"></a>Concaténation de deux fichiers vidéo ou plus

L'exemple suivant décrit comment générer une présélection pour concaténer deux fichiers vidéo ou plus. Le scénario le plus courant consiste à ajouter une amorce de début ou de fin à la vidéo principale. Dans le cadre de l’utilisation prévue, les fichiers vidéo en cours de modification conjointe partagent des propriétés (résolution vidéo, fréquence d’images, nombre de pistes audio, etc.). Vous devez prendre soin de ne pas mélanger des vidéos de différentes fréquences d’images ou comportant un nombre différent de pistes audio.

>[!NOTE]
>La conception actuelle de la fonctionnalité de concaténation suppose que les clips vidéo d’entrée soient cohérents en termes de résolution, de fréquence d’images etc. 

### <a name="requirements-and-considerations"></a>Conditions requises et éléments à prendre en compte

* Les vidéos d'entrée ne doivent avoir qu'une seule piste audio.
* Les vidéos d'entrée doivent avoir la même fréquence d'images.
* Vous devez télécharger vos vidéos dans des éléments multimédias séparés et définir les vidéos comme fichier primaire de chaque élément multimédia.
* Vous devez connaître la durée de vos vidéos.
* Les exemples prédéfinis ci-dessous supposent que toutes les vidéos d'entrée commencent avec un timestamp égal à zéro. Vous devez modifier les valeurs StartTime si les vidéos ont un timestamp de début différent, comme c’est généralement le cas avec les archives en direct.
* La présélection JSON fait des références explicites aux valeurs AssetID des éléments multimédias d’entrée.
* L'exemple de code suppose que la présélection JSON a été enregistrée dans un fichier local, par exemple « C:\supportFiles\preset.json ». Il suppose également que les deux éléments multimédias ont été créés en téléchargeant deux fichiers vidéo et que vous connaissez les valeurs AssetID résultantes.
* L'extrait de code et la présélection JSON montrent un exemple de concaténation de deux fichiers vidéo. Vous pouvez l'étendre à plus de deux vidéos en :

  1. Appelant task.InputAssets.Add() à plusieurs reprises pour ajouter d’autres vidéos, dans l'ordre.
  2. Apportant les modifications correspondantes à l'élément « Sources » dans le JSON, en ajoutant d’autres entrées, dans le même ordre.

### <a name="net-code"></a>Code .NET

```csharp
IAsset asset1 = _context.Assets.Where(asset => asset.Id == "nb:cid:UUID:606db602-efd7-4436-97b4-c0b867ba195b").FirstOrDefault();
IAsset asset2 = _context.Assets.Where(asset => asset.Id == "nb:cid:UUID:a7e2b90f-0565-4a94-87fe-0a9fa07b9c7e").FirstOrDefault();

// Declare a new job.
IJob job = _context.Jobs.Create("Media Encoder Standard Job for Concatenating Videos");
// Get a media processor reference, and pass to it the name of the
// processor to use for the specific task.
IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

// Load the XML (or JSON) from the local file.
string configuration = File.ReadAllText(@"c:\supportFiles\preset.json");

// Create a task
ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
    processor,
    configuration,
    TaskOptions.None);

// Specify the input videos to be concatenated (in order).
task.InputAssets.Add(asset1);
task.InputAssets.Add(asset2);
// Add an output asset to contain the results of the job.
// This output is specified as AssetCreationOptions.None, which
// means the output asset is not encrypted.
task.OutputAssets.AddNew("Output asset",
    AssetCreationOptions.None);

job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
job.Submit();
job.GetExecutionProgressTask(CancellationToken.None).Wait();
```

### <a name="json-preset"></a>Présélection JSON

Mettez à jour votre présélection personnalisée avec les ID des éléments multimédias à concaténer et le segment de temps approprié pour chaque vidéo.

```json
{
  "Version": 1.0,
  "Sources": [
    {
      "AssetID": "606db602-efd7-4436-97b4-c0b867ba195b",
      "StartTime": "00:00:01",
      "Duration": "00:00:15"
    },
    {
      "AssetID": "a7e2b90f-0565-4a94-87fe-0a9fa07b9c7e",
      "StartTime": "00:00:02",
      "Duration": "00:00:05"
    }
  ],
  "Codecs": [
    {
      "KeyFrameInterval": "00:00:02",
      "SceneChangeDetection": true,
      "H264Layers": [
        {
          "Level": "auto",
          "Bitrate": 1800,
          "MaxBitrate": 1800,
          "BufferWindow": "00:00:05",
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "Width": "640",
          "Height": "360",
          "FrameRate": "0/1"
        }
      ],
      "Type": "H264Video"
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
      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

## <a name="crop-videos-with-media-encoder-standard"></a><a id="crop"></a>Rogner des vidéos avec l’encodeur multimédia standard
Consultez la rubrique [Rogner des vidéos avec l’encodeur multimédia standard](media-services-crop-video.md) .

## <a name="insert-a-video-track-when-input-has-no-video"></a><a id="no_video"></a>Insertion d’une piste vidéo lorsque l’entrée ne comporte aucune vidéo

Par défaut, si vous envoyez à l’encodeur une entrée contenant uniquement de l’audio (sans contenu vidéo), l’élément multimédia de sortie regroupe les fichiers qui contiennent uniquement des données audio. Certains lecteurs, y compris Azure Media Player (consultez [ceci](https://feedback.azure.com/forums/169396-azure-media-services/suggestions/8082468-audio-only-scenarios)) peuvent ne pas être en mesure de gérer ces flux. Dans ce cas, vous pouvez utiliser ce paramètre pour forcer l’encodeur à ajouter à la sortie une piste vidéo monochrome.

> [!NOTE]
> Le fait de forcer l’encodeur à insérer une piste vidéo de sortie augmente la taille de l’élément multimédia de sortie, et ainsi les coûts associés à la tâche d’encodage. Vous devez exécuter des tests pour vérifier que cette augmentation résultante n’a qu’une légère incidence sur vos frais mensuels.
>

### <a name="inserting-video-at-only-the-lowest-bitrate"></a>Insertion de vidéo uniquement avec le débit le plus bas

Supposons que vous utilisez un encodage à plusieurs vitesses de transmission prédéfinies, par exemple [« H264 multidébit 720p »](media-services-mes-preset-h264-multiple-bitrate-720p.md) pour encoder à des fins de diffusion en continu votre catalogue d’entrée tout entier, qui contient un mélange de fichiers vidéo et audio uniquement. Dans ce scénario, lorsque l’entrée ne comporte aucune vidéo, vous pouvez vouloir forcer l’encodeur à insérer une piste vidéo monochrome au plus faible débit uniquement, et non à toutes les vitesses de transmission de sortie. Pour ce faire, vous devez utiliser l’indicateur **InsertBlackIfNoVideoBottomLayerOnly**.

Vous pouvez utiliser l’une des présélections MES documentées dans [cette](media-services-mes-presets-overview.md) section et apporter la modification suivante :

#### <a name="json-preset"></a>Présélection JSON

```json
{
  "KeyFrameInterval": "00:00:02",
  "StretchMode": "AutoSize",
  "Condition": "InsertBlackIfNoVideoBottomLayerOnly",
  "H264Layers": [
    …
  ]
}
```

#### <a name="xml-preset"></a>Présélection XML

Avec le XML, utilisez Condition="InsertBlackIfNoVideoBottomLayerOnly" en tant qu’attribut pour l’élément **H264Video** et Condition="InsertSilenceIfNoAudio" en tant qu’attribut pour **AACAudio**.

```xml
. . .
<Encoding>
  <H264Video Condition="InsertBlackIfNoVideoBottomLayerOnly">
    <KeyFrameInterval>00:00:02</KeyFrameInterval>
    <SceneChangeDetection>true</SceneChangeDetection>
    <StretchMode>AutoSize</StretchMode>
    <H264Layers>
      <H264Layer>
        . . .
      </H264Layer>
    </H264Layers>
    <Chapters />
  </H264Video>
  <AACAudio Condition="InsertSilenceIfNoAudio">
    <Profile>AACLC</Profile>
    <Channels>2</Channels>
    <SamplingRate>48000</SamplingRate>
    <Bitrate>128</Bitrate>
  </AACAudio>
</Encoding>
. . .
```

### <a name="inserting-video-at-all-output-bitrates"></a>Insertion de vidéo à tous les débits binaires de sortie
Supposons que vous utilisez un encodage à plusieurs vitesses de transmission prédéfinies, par exemple [« H264 multidébit 720p »](media-services-mes-preset-H264-Multiple-Bitrate-720p.md) pour encoder à des fins de diffusion en continu votre catalogue d’entrée tout entier, qui contient un mélange de fichiers vidéo et audio uniquement. Dans ce scénario, lorsque l’entrée ne comporte aucune vidéo, vous pouvez vouloir forcer l’encodeur à insérer une piste vidéo monochrome à toutes les vitesses de transmission de sortie. Cela garantit que vos éléments multimédias de sortie sont tous homogènes en ce qui concerne le nombre de pistes vidéo et de pistes audio. Pour ce faire, vous devez spécifier l’indicateur « InsertBlackIfNoVideo ».

Vous pouvez utiliser l’une des présélections MES documentées dans [cette](media-services-mes-presets-overview.md) section et apporter la modification suivante :

#### <a name="json-preset"></a>Présélection JSON

```json
{
  "KeyFrameInterval": "00:00:02",
  "StretchMode": "AutoSize",
  "Condition": "InsertBlackIfNoVideo",
  "H264Layers": [
    …
  ]
}
```

#### <a name="xml-preset"></a>Présélection XML

Avec le XML, utilisez Condition="InsertBlackIfNoVideo" en tant qu’attribut pour l’élément **H264Video** et Condition="InsertSilenceIfNoAudio" en tant qu’attribut pour **AACAudio**.

```xml
. . .
<Encoding>
  <H264Video Condition="InsertBlackIfNoVideo">
    <KeyFrameInterval>00:00:02</KeyFrameInterval>
    <SceneChangeDetection>true</SceneChangeDetection>
    <StretchMode>AutoSize</StretchMode>
    <H264Layers>
      <H264Layer>
        . . .
      </H264Layer>
    </H264Layers>
    <Chapters />
  </H264Video>
  <AACAudio Condition="InsertSilenceIfNoAudio">
    <Profile>AACLC</Profile>
    <Channels>2</Channels>
    <SamplingRate>48000</SamplingRate>
    <Bitrate>128</Bitrate>
  </AACAudio>
</Encoding>
. . .  
```

## <a name="rotate-a-video"></a><a id="rotate_video"></a>Faire pivoter une vidéo
[Media Encoder Standard](media-services-dotnet-encode-with-media-encoder-standard.md) prend en charge les angles de rotation 0, 90,180 et 270. Le comportement par défaut est « Auto », ce qui signifie qu’il tente de détecter les métadonnées de rotation dans le fichier vidéo entrant et de les compenser. Incluez l’élément **Sources** suivant dans l’une des présélections définies dans [cette](media-services-mes-presets-overview.md) section :

### <a name="json-preset"></a>Présélection JSON

```json
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

### <a name="xml-preset"></a>Présélection XML

```xml
<Sources>
  <Source>
    <Streams />
    <Filters>
      <Rotation>90</Rotation>
    </Filters>
  </Source>
</Sources>
```

Consultez également [cette](media-services-mes-schema.md#PreserveResolutionAfterRotation) rubrique pour plus d’informations sur la manière dont l’encodeur interprète les paramètres de largeur et de hauteur dans la présélection, lorsque la compensation de la rotation est déclenchée.

Vous pouvez utiliser la valeur « 0 » pour indiquer à l’encodeur d’ignorer les métadonnées de rotation, le cas échéant, dans la vidéo d’entrée.

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>Voir aussi
[Vue d’ensemble de l’encodage de Media Services](media-services-encode-asset.md)
