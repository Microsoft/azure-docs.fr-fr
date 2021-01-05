---
title: Schéma de métadonnées d’entrée Azure Media Services | Microsoft Docs
description: Cet article fournit une vue d’ensemble du schéma de métadonnées d’entrée d’Azure Media Services.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 57a91622bef401d946a383e3be39f2e566fa50b4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89267834"
---
# <a name="input-metadata"></a>Métadonnées d'entrée

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Un travail d’encodage est associé à un élément multimédia d’entrée (ou plusieurs) sur lequel vous souhaitez effectuer des tâches d’encodage.  À l’achèvement d’une tâche, une ressource de sortie est générée.  L’élément multimédia de sortie contient la vidéo, l’audio, les miniatures, le manifeste, et ainsi de suite. Il contient également un fichier avec des métadonnées relatives à l’élément multimédia d’entrée. Le nom du fichier XML de métadonnées a le format suivant : &lt;asset_id&gt;_metadata.xml (par exemple, 41114ad3-eb5e-4c57-8d92-5354e2b7d4a4_metadata.xml), où &lt;asset_id&gt; est la valeur AssetId de l’élément multimédia d’entrée.  

Media Services n’analyse pas préemptivement les éléments multimédia d’entrée pour générer des métadonnées. Les métadonnées d’entrée sont générées uniquement sous forme d’artefact lorsqu’un élément multimédia d’entrée est traité au sein d’un travail. Par conséquent, cet artefact est écrit dans l’élément multimédia de sortie. Différents outils sont utilisés pour générer les métadonnées pour les éléments multimédias d’entrée et de sortie. Ainsi, les métadonnées d’entrée présentent un schéma légèrement différent de celui des métadonnées de sortie.

Si vous souhaitez examiner le fichier de métadonnées, vous pouvez créer un localisateur **SAS** et télécharger le fichier sur votre ordinateur local. Vous trouverez un exemple illustrant comment créer un localisateur SAS et télécharger un fichier dans la rubrique [Utilisation des extensions du SDK Media Services .NET](media-services-dotnet-get-started.md).  

Cet article décrit les éléments et types du schéma XML sur lesquels les métadonnées d’entrée (&lt;asset_id&gt;_metadata.xml) sont basées.  Pour plus d’informations sur le fichier qui contient des métadonnées sur la ressource de sortie, consultez [Métadonnées de sortie](media-services-output-metadata-schema.md).  

Vous trouverez le [Code du schéma](media-services-input-metadata-schema.md#code) et un [exemple de XML](media-services-input-metadata-schema.md#xml) à la fin de cet article.  
 

## <a name="assetfiles-element-root-element"></a><a name="AssetFiles"></a> Élément AssetFiles (élément racine)
Contient une collection [d’éléments AssetFile](media-services-input-metadata-schema.md#AssetFile) pour le travail d’encodage.  

Consultez l’exemple XML à la fin de cet article : [Exemple XML](media-services-input-metadata-schema.md#xml).  

| Nom | Description |
| --- | --- |
| **AssetFile**<br /><br /> minOccurs="1" maxOccurs="unbounded" |Un élément enfant unique. Pour plus d'informations, consultez la page [Élément AssetFile](media-services-input-metadata-schema.md#AssetFile). |

## <a name="assetfile-element"></a><a name="AssetFile"></a> Élément AssetFile
 Contient des attributs et des éléments qui décrivent une ressource.  

 Consultez l’exemple XML à la fin de cet article : [Exemple XML](media-services-input-metadata-schema.md#xml).  

### <a name="attributes"></a>Attributs
| Nom | Type | Description |
| --- | --- | --- |
| **Nom**<br /><br /> Obligatoire |**xs:string** |Nom du fichier de ressource. |
| **Taille**<br /><br /> Obligatoire |**xs:long** |Taille du fichier de ressource en octets. |
| **Durée**<br /><br /> Obligatoire |**xs:duration** |Durée de lecture du contenu. Exemple : Duration="PT25M37.757S". |
| **NumberOfStreams**<br /><br /> Obligatoire |**xs:int** |Nombre de flux dans le fichier de ressource. |
| **FormatNames**<br /><br /> Obligatoire |**xs: string** |Noms de format. |
| **FormatVerboseNames**<br /><br /> Obligatoire |**xs: string** |Noms détaillés des formats. |
| **StartTime** |**xs:duration** |Heure de début du contenu. Exemple : StartTime="PT2.669S". |
| **OverallBitRate** |**xs: int** |Vitesse de transmission moyenne du fichier de ressource en kbit/s. |

> [!NOTE]
> Les quatre éléments enfants suivants doivent apparaître dans une séquence.  
> 
> 

### <a name="child-elements"></a>Éléments enfants
| Nom | Type | Description |
| --- | --- | --- |
| **Programmes**<br /><br /> minOccurs="0" | |Collection de tous les [éléments Programs](media-services-input-metadata-schema.md#Programs) lorsque le fichier de ressource est au format MPEG-TS. |
| **VideoTracks**<br /><br /> minOccurs="0" | |Chaque élément AssetFile physique peut contenir zéro, une ou plusieurs pistes vidéo entrelacées dans un format de conteneur approprié. Cet élément contient une collection de tous les [éléments VideoTracks](media-services-input-metadata-schema.md#VideoTracks) qui font partie du fichier de ressource. |
| **AudioTracks**<br /><br /> minOccurs="0" | |Chaque élément AssetFile physique peut contenir zéro ou plusieurs pistes audio entrelacées dans un format de conteneur approprié. Cet élément contient une collection de tous les [éléments AudioTracks](media-services-input-metadata-schema.md#AudioTracks) qui font partie du fichier de ressource. |
| **Métadonnées**<br /><br /> minOccurs="0" maxOccurs="unbounded" |[MetadataType](media-services-input-metadata-schema.md#MetadataType) |Les métadonnées du fichier de ressource représentées sous la forme de chaînes clé/valeur. Par exemple :<br /><br /> **&lt;Metadata key="language" value="eng" /&gt;** |

## <a name="tracktype"></a><a name="TrackType"></a> TrackType
Consultez l’exemple XML à la fin de cet article : [Exemple XML](media-services-input-metadata-schema.md#xml).  

### <a name="attributes"></a>Attributs
| Nom | Type | Description |
| --- | --- | --- |
| **Id**<br /><br /> Obligatoire |**xs:int** |Index de base zéro de cette piste audio ou vidéo.<br /><br /> Il ne s’agit pas nécessairement du trackid tel qu’utilisé dans un fichier MP4. |
| **Codec** |**xs:string** |Chaîne de codec de la piste vidéo. |
| **CodecLongName** |**xs: string** |Nom long du codec de piste audio ou vidéo. |
| **TimeBase**<br /><br /> Obligatoire |**xs:string** |Période. Exemple : TimeBase="1/48000" |
| **NumberOfFrames** |**xs:int** |Nombre de trames (présent pour les pistes vidéo). |
| **StartTime** |**xs: duration** |Suivre l’heure de début. Exemple : StartTime="PT2.669S" |
| **Durée** |**xs:duration** |Durée de la piste. Exemple : Duration="PTSampleFormat M37.757S". |

> [!NOTE]
> Les deux éléments enfants suivants doivent apparaître dans une séquence.  
> 
> 

### <a name="child-elements"></a>Éléments enfants
| Nom | Type | Description |
| --- | --- | --- |
| **Disposition**<br /><br /> minOccurs="0" maxOccurs="1" |[StreamDispositionType](media-services-input-metadata-schema.md#StreamDispositionType) |Contient des informations de présentation (par exemple, si une piste audio particulière est destinée aux utilisateurs malvoyants). |
| **Métadonnées**<br /><br /> minOccurs="0" maxOccurs="unbounded" |[MetadataType](media-services-input-metadata-schema.md#MetadataType) |Des chaînes clé/valeur génériques qui peuvent être utilisées pour contenir différents types d’informations. Par exemple, key=”language”, et value=”eng”. |

## <a name="audiotracktype-inherits-from-tracktype"></a><a name="AudioTrackType"></a> AudioTrackType (hérite de TrackType)
 **AudioTrackType** est un type complexe global qui hérite de [TrackType](media-services-input-metadata-schema.md#TrackType).  

 Le type représente une piste audio spécifique dans le fichier de ressource.  

 Consultez l’exemple XML à la fin de cet article : [Exemple XML](media-services-input-metadata-schema.md#xml).  

### <a name="attributes"></a>Attributs
| Nom | Type | Description |
| --- | --- | --- |
| **SampleFormat** |**xs:string** |Exemple de format. |
| **ChannelLayout** |**xs: string** |Disposition de canal. |
| **Canaux**<br /><br /> Obligatoire |**xs:int** |Nombre (0 ou plus) de canaux audio. |
| **SamplingRate**<br /><br /> Obligatoire |**xs:int** |Taux d’échantillonnage audio en échantillons/sec ou Hz. |
| **Bitrate** |**xs:int** |Débit binaire audio moyen en bits par seconde, tel que calculé à partir du fichier de ressource. Seule la charge utile de flux élémentaire est comptée et la surcharge de packaging n’est pas incluse dans ce nombre. |
| **BitsPerSample** |**xs:int** |Bits par échantillon pour le type de format wFormatTag. |

## <a name="videotracktype-inherits-from-tracktype"></a><a name="VideoTrackType"></a> VideoTrackType (hérite de TrackType)
**VideoTrackType** est un type complexe global qui hérite de [TrackType](media-services-input-metadata-schema.md#TrackType).  

Le type représente une piste vidéo spécifique dans le fichier de ressource.  

Consultez l’exemple XML à la fin de cet article : [Exemple XML](media-services-input-metadata-schema.md#xml).  

### <a name="attributes"></a>Attributs
| Nom | Type | Description |
| --- | --- | --- |
| **FourCC**<br /><br /> Obligatoire |**xs:string** |Code FourCC du codec vidéo. |
| **Profil** |**xs: string** |Profil de la piste vidéo. |
| **Niveau** |**xs: string** |Niveau de la piste vidéo. |
| **PixelFormat** |**xs: string** |Format de pixel de la piste vidéo. |
| **Width**<br /><br /> Obligatoire |**xs:int** |Largeur vidéo encodée en pixels. |
| **Height**<br /><br /> Obligatoire |**xs:int** |Hauteur vidéo encodée en pixels. |
| **DisplayAspectRatioNumerator**<br /><br /> Obligatoire |**xs: double** |Numérateur des proportions d’affichage vidéo. |
| **DisplayAspectRatioDenominator**<br /><br /> Obligatoire |**xs:double** |Dénominateur des proportions d’affichage vidéo. |
| **DisplayAspectRatioDenominator**<br /><br /> Obligatoire |**xs: double** |Numérateur des proportions d’échantillon vidéo. |
| **SampleAspectRatioNumerator** |**xs: double** |Numérateur des proportions d’échantillon vidéo. |
| **SampleAspectRatioNumerator** |**xs:double** |Dénominateur des proportions d’échantillon vidéo. |
| **FrameRate**<br /><br /> Obligatoire |**xs:decimal** |Fréquence d’images vidéo mesurée au format .3f. |
| **Bitrate** |**xs:int** |Débit binaire vidéo moyen en kilobits par seconde, tel que calculé à partir du fichier de ressource. Seule la charge utile de flux élémentaire est comptée et la surcharge de packaging n’est pas incluse. |
| **MaxGOPBitrate** |**xs: int** |Vitesse de transmission moyenne GOP maximum pour cette piste vidéo, en kilobits par seconde. |
| **HasBFrames** |**xs:int** |Numéro de piste vidéo des trames B. |

## <a name="metadatatype"></a><a name="MetadataType"></a> MetadataType
**MetadataType** est un type complexe global qui décrit les métadonnées d’un fichier de ressource en tant que chaînes clé/valeur. Par exemple, key=”language”, et value=”eng”.  

Consultez l’exemple XML à la fin de cet article : [Exemple XML](media-services-input-metadata-schema.md#xml).  

### <a name="attributes"></a>Attributs
| Nom | Type | Description |
| --- | --- | --- |
| **key**<br /><br /> Obligatoire |**xs:string** |La clé dans la paire clé/valeur. |
| **value**<br /><br /> Obligatoire |**xs:string** |La valeur dans la paire clé/valeur. |

## <a name="programtype"></a><a name="ProgramType"></a> ProgramType
**ProgramType** est un type complexe global qui décrit un programme.  

### <a name="attributes"></a>Attributs
| Nom | Type | Description |
| --- | --- | --- |
| **ProgramId**<br /><br /> Obligatoire |**xs:int** |ID de programme |
| **NumberOfPrograms**<br /><br /> Obligatoire |**xs:int** |Nombre de programmes. |
| **PmtPid**<br /><br /> Obligatoire |**xs:int** |Les tables de correspondance de programme (PMT) contiennent des informations sur les programmes.  Pour plus d'informations, consultez [PMt](https://en.wikipedia.org/wiki/MPEG_transport_stream#PMT). |
| **PcrPid**<br /><br /> Obligatoire |**xs: int** |Utilisé par le décodeur. Pour plus d'informations, consultez [PCR](https://en.wikipedia.org/wiki/MPEG_transport_stream#PCR) |
| **StartPTS** |**xs: long** |Horodatage de début de présentation. |
| **EndPTS** |**xs: long** |Horodatage de fin de présentation. |

## <a name="streamdispositiontype"></a><a name="StreamDispositionType"></a> StreamDispositionType
**StreamDispositionType** est un type complexe global qui décrit le flux de données.  

Consultez l’exemple XML à la fin de cet article : [Exemple XML](media-services-input-metadata-schema.md#xml).  

### <a name="attributes"></a>Attributs
| Nom | Type | Description |
| --- | --- | --- |
| **Par défaut**<br /><br /> Obligatoire |**xs: int** |Définissez cet attribut sur 1 pour indiquer qu’il s’agit de la présentation par défaut. |
| **Dub**<br /><br /> Obligatoire |**xs:int** |Définissez cet attribut sur 1 pour indiquer qu’il s’agit de la présentation doublée. |
| **Ressource d’origine**<br /><br /> Obligatoire |**xs: int** |Définissez cet attribut sur 1 pour indiquer qu’il s’agit de la présentation d’origine. |
| **Commentaire**<br /><br /> Obligatoire |**xs:int** |Définissez cet attribut sur 1 pour indiquer que cette piste contient des commentaires. |
| **Lyrics**<br /><br /> Obligatoire |**xs:int** |Définissez cet attribut sur 1 pour indiquer que cette piste contient des paroles. |
| **Karaoke**<br /><br /> Obligatoire |**xs:int** |Définissez cet attribut sur 1 pour indiquer qu’il s’agit d’une piste de karaoké (musique de fond, sans voix). |
| **Forced**<br /><br /> Obligatoire |**xs:int** |Définissez cet attribut sur 1 pour indiquer qu’il s’agit la présentation forcée. |
| **HearingImpaired**<br /><br /> Obligatoire |**xs:int** |Définissez cet attribut sur 1 pour indiquer que cette piste est destinée aux personnes malentendantes. |
| **VisualImpaired**<br /><br /> Obligatoire |**xs:int** |Définissez cet attribut sur 1 pour indiquer que cette piste est destinée aux personnes malvoyantes. |
| **CleanEffects**<br /><br /> Obligatoire |**xs: int** |Définissez cet attribut sur 1 pour indiquer que cette piste comporte des effets propres. |
| **AttachedPic**<br /><br /> Obligatoire |**xs: int** |Définissez cet attribut sur 1 pour indiquer que cette piste contient des images. |

## <a name="programs-element"></a><a name="Programs"></a> Programmes (élément)
Élément wrapper contenant plusieurs éléments **Program**.  

### <a name="child-elements"></a>Éléments enfants
| Nom | Type | Description |
| --- | --- | --- |
| **Programme**<br /><br /> minOccurs="0" maxOccurs="unbounded" |[ProgramType](media-services-input-metadata-schema.md#ProgramType) |Pour les fichiers de ressource qui sont dans un format MPEG-TS, contient des informations sur les programmes dans le fichier de ressource. |

## <a name="videotracks-element"></a><a name="VideoTracks"></a> Élément VideoTracks
 Élément wrapper contenant plusieurs éléments **VideoTrack**.  

 Consultez l’exemple XML à la fin de cet article : [Exemple XML](media-services-input-metadata-schema.md#xml).  

### <a name="child-elements"></a>Éléments enfants
| Nom | Type | Description |
| --- | --- | --- |
| **VideoTrack**<br /><br /> minOccurs="0" maxOccurs="unbounded" |[VideoTrackType (hérite de TrackType)](media-services-input-metadata-schema.md#VideoTrackType) |Contient des informations sur les pistes vidéo dans le fichier de ressource. |

## <a name="audiotracks-element"></a><a name="AudioTracks"></a> Élément AudioTracks
 Élément wrapper contenant plusieurs éléments **AudioTrack**.  

 Consultez l’exemple XML à la fin de cet article : [Exemple XML](media-services-input-metadata-schema.md#xml).  

### <a name="elements"></a>elements
| Nom | Type | Description |
| --- | --- | --- |
| **AudioTrack**<br /><br /> minOccurs="0" maxOccurs="unbounded" |[AudioTrackType (hérite de TrackType)](media-services-input-metadata-schema.md#AudioTrackType) |Contient des informations sur les pistes audio dans le fichier de ressource. |

## <a name="schema-code"></a><a name="code"></a> Code du schéma
```xml
<?xml version="1.0" encoding="utf-8"?>  
<xs:schema xmlns:xs="https://www.w3.org/2001/XMLSchema" xmlns:msdata="urn:schemas-microsoft-com:xml-msdata" version="1.0"  
            xmlns="http://schemas.microsoft.com/windowsazure/mediaservices/2014/07/mediaencoder/inputmetadata"  
            targetNamespace="http://schemas.microsoft.com/windowsazure/mediaservices/2014/07/mediaencoder/inputmetadata"  
            elementFormDefault="qualified">  

  <xs:complexType name="MetadataType">  
    <xs:attribute name="key"   type="xs:string" use="required"/>  
    <xs:attribute name="value" type="xs:string" use="required"/>  
  </xs:complexType>  

  <xs:complexType name="ProgramType">  
    <xs:attribute name="ProgramId" type="xs:int" use="required">  
      <xs:annotation>  
        <xs:documentation>Program Id</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="NumberOfPrograms" type="xs:int" use="required">  
      <xs:annotation>  
        <xs:documentation>Number of programs</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="PmtPid" type="xs:int" use="required">  
      <xs:annotation>  
        <xs:documentation>pmt pid</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="PcrPid" type="xs:int" use="required">  
      <xs:annotation>  
        <xs:documentation>pcr pid</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="StartPTS" type="xs:long">  
      <xs:annotation>  
        <xs:documentation>start pts</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="EndPTS" type="xs:long">  
      <xs:annotation>  
        <xs:documentation>end pts</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
  </xs:complexType>  

  <xs:complexType name="StreamDispositionType">  
    <xs:attribute name="Default"          type="xs:int" use="required" />  
    <xs:attribute name="Dub"              type="xs:int" use="required" />  
    <xs:attribute name="Original"         type="xs:int" use="required" />  
    <xs:attribute name="Comment"          type="xs:int" use="required" />  
    <xs:attribute name="Lyrics"           type="xs:int" use="required" />  
    <xs:attribute name="Karaoke"          type="xs:int" use="required" />  
    <xs:attribute name="Forced"           type="xs:int" use="required" />  
    <xs:attribute name="HearingImpaired"  type="xs:int" use="required" />  
    <xs:attribute name="VisualImpaired"   type="xs:int" use="required" />  
    <xs:attribute name="CleanEffects"     type="xs:int" use="required" />  
    <xs:attribute name="AttachedPic"      type="xs:int" use="required" />  
  </xs:complexType>  

  <xs:complexType name="TrackType" abstract="true">  
    <xs:sequence>  
      <xs:element name="Disposition" type="StreamDispositionType" minOccurs="0" maxOccurs="1"/>  
      <xs:element name="Metadata" type="MetadataType" minOccurs="0" maxOccurs="unbounded"/>  
    </xs:sequence>  
    <xs:attribute name="Id" use="required">  
      <xs:annotation>  
        <xs:documentation>zero-based index of this video track. Note: this is not necessarily the TrackID as used in an MP4 file</xs:documentation>  
      </xs:annotation>  
      <xs:simpleType>  
        <xs:restriction base="xs:int">  
          <xs:minInclusive value="0"/>  
        </xs:restriction>  
      </xs:simpleType>  
    </xs:attribute>  
    <xs:attribute name="Codec" type="xs:string">  
      <xs:annotation>  
        <xs:documentation>video track codec string</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="CodecLongName" type="xs:string">  
      <xs:annotation>  
        <xs:documentation>video track codec long name</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="TimeBase"  type="xs:string" use="required">  
      <xs:annotation>  
        <xs:documentation>Time base. Example: TimeBase="1/48000"</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="NumberOfFrames">  
      <xs:annotation>  
        <xs:documentation>number of frames</xs:documentation>  
      </xs:annotation>  
      <xs:simpleType>  
        <xs:restriction base="xs:int">  
          <xs:minInclusive value="0"/>  
        </xs:restriction>  
      </xs:simpleType>  
    </xs:attribute>  
    <xs:attribute name="StartTime" type="xs:duration">  
      <xs:annotation>  
        <xs:documentation>Track start time. Example: StartTime="PT2.669S"</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
    <xs:attribute name="Duration" type="xs:duration">  
      <xs:annotation>  
        <xs:documentation>Track duration. Example: Duration="PT25M37.757S"</xs:documentation>  
      </xs:annotation>  
    </xs:attribute>  
  </xs:complexType>  

  <xs:complexType name="VideoTrackType">  
    <xs:annotation>  
      <xs:documentation>A specific video track in the parent AssetFile</xs:documentation>  
    </xs:annotation>  
    <xs:complexContent>  
      <xs:extension base="TrackType">  
        <xs:attribute name="FourCC" type="xs:string" use="required">  
          <xs:annotation>  
            <xs:documentation>video codec FourCC code</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
        <xs:attribute name="Profile" type="xs:string">  
          <xs:annotation>  
            <xs:documentation>profile</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
        <xs:attribute name="Level" type="xs:string">  
          <xs:annotation>  
            <xs:documentation>level</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
        <xs:attribute name="PixelFormat" type="xs:string">  
          <xs:annotation>  
            <xs:documentation>Video track's pixel format</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
        <xs:attribute name="Width" use="required">  
          <xs:annotation>  
            <xs:documentation>encoded video width in pixels</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="Height" use="required">  
          <xs:annotation>  
            <xs:documentation>encoded video height in pixels</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="DisplayAspectRatioNumerator" use="required">  
          <xs:annotation>  
            <xs:documentation>video display aspect ratio numerator</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:double">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="DisplayAspectRatioDenominator" use="required">  
          <xs:annotation>  
            <xs:documentation>video display aspect ratio denominator</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:double">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="SampleAspectRatioNumerator">  
          <xs:annotation>  
            <xs:documentation>video sample aspect ratio numerator</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:double">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="SampleAspectRatioDenominator">  
          <xs:annotation>  
            <xs:documentation>video sample aspect ratio denominator</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:double">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="FrameRate" use="required">  
          <xs:annotation>  
            <xs:documentation>measured video frame rate in .3f format</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:decimal">  
              <xs:minInclusive value="0"/>  
              <xs:fractionDigits value="3"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="Bitrate">  
          <xs:annotation>  
            <xs:documentation>average video bit rate in kilobits per second, as calculated from the AssetFile. Counts only the elementary stream payload, and does not include the packaging overhead</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="MaxGOPBitrate">  
          <xs:annotation>  
            <xs:documentation>Max GOP average bitrate for this video track, in kilobits per second</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="HasBFrames" type="xs:int">  
          <xs:annotation>  
            <xs:documentation>video track number of B frames</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
      </xs:extension>  
    </xs:complexContent>  
  </xs:complexType>  

  <xs:complexType name="AudioTrackType">  
    <xs:annotation>  
      <xs:documentation>a specific audio track in the parent AssetFile</xs:documentation>  
    </xs:annotation>  
    <xs:complexContent>  
      <xs:extension base="TrackType">  
        <xs:attribute name="SampleFormat"  type="xs:string">  
          <xs:annotation>  
            <xs:documentation>sample format</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
        <xs:attribute name="ChannelLayout"  type="xs:string">  
          <xs:annotation>  
            <xs:documentation>channel layout</xs:documentation>  
          </xs:annotation>  
        </xs:attribute>  
        <xs:attribute name="Channels" use="required">  
          <xs:annotation>  
            <xs:documentation>number of audio channels</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="SamplingRate" use="required">  
          <xs:annotation>  
            <xs:documentation>audio sampling rate in samples/sec or Hz</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="Bitrate">  
          <xs:annotation>  
            <xs:documentation>average audio bit rate in bits per second, as calculated from the AssetFile. Counts only the elementary stream payload, and does not include the packaging overhead</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
        <xs:attribute name="BitsPerSample">  
          <xs:annotation>  
            <xs:documentation>Bits per sample for the wFormatTag format type</xs:documentation>  
          </xs:annotation>  
          <xs:simpleType>  
            <xs:restriction base="xs:int">  
              <xs:minInclusive value="0"/>  
            </xs:restriction>  
          </xs:simpleType>  
        </xs:attribute>  
      </xs:extension>  
    </xs:complexContent>  
  </xs:complexType>  

  <xs:element name="AssetFiles">  
    <xs:annotation>  
      <xs:documentation>Collection of AssetFile entries for the encoding job</xs:documentation>  
    </xs:annotation>  
    <xs:complexType>  
      <xs:sequence>  
        <xs:element name="AssetFile" minOccurs="1" maxOccurs="unbounded">  
          <xs:annotation>  
            <xs:documentation>asset file</xs:documentation>  
          </xs:annotation>  
          <xs:complexType>  
            <xs:sequence>  
              <xs:element name="Programs" minOccurs="0">  
                <xs:annotation>  
                  <xs:documentation>This is the collection of all programs when file is MPEG-TS</xs:documentation>  
                </xs:annotation>  
                <xs:complexType>  
                  <xs:sequence>  
                    <xs:element name="Program" type="ProgramType" minOccurs="0" maxOccurs="unbounded" />  
                  </xs:sequence>  
                </xs:complexType>  
              </xs:element>  
              <xs:element name="VideoTracks" minOccurs="0">  
                <xs:annotation>  
                  <xs:documentation>Each physical AssetFile can contain in it zero or more video tracks interleaved into an appropriate container format. This is the collection of all those video tracks</xs:documentation>  
                </xs:annotation>  
                <xs:complexType>  
                  <xs:sequence>  
                    <xs:element name="VideoTrack" type="VideoTrackType" minOccurs="0" maxOccurs="unbounded" />  
                  </xs:sequence>  
                </xs:complexType>  
              </xs:element>  
              <xs:element name="AudioTracks" minOccurs="0">  
                <xs:annotation>  
                  <xs:documentation>each physical AssetFile can contain in it zero or more audio tracks interleaved into an appropriate container format. This is the collection of all those audio tracks</xs:documentation>  
                </xs:annotation>  
                <xs:complexType>  
                  <xs:sequence>  
                    <xs:element name="AudioTrack" type="AudioTrackType" minOccurs="0" maxOccurs="unbounded" />  
                  </xs:sequence>  
                </xs:complexType>  
              </xs:element>  
              <xs:element name="Metadata" type="MetadataType" minOccurs="0" maxOccurs="unbounded" />  
            </xs:sequence>  
            <xs:attribute name="Name" type="xs:string" use="required">  
              <xs:annotation>  
                <xs:documentation>the media asset file name</xs:documentation>  
              </xs:annotation>  
            </xs:attribute>  
            <xs:attribute name="Size" use="required">  
              <xs:annotation>  
                <xs:documentation>size of file in bytes</xs:documentation>  
              </xs:annotation>  
              <xs:simpleType>  
                <xs:restriction base="xs:long">  
                  <xs:minInclusive value="0"/>  
                </xs:restriction>  
              </xs:simpleType>  
            </xs:attribute>  
            <xs:attribute name="Duration" type="xs:duration" use="required">  
              <xs:annotation>  
                <xs:documentation>content play back duration. Example: Duration="PT25M37.757S"</xs:documentation>  
              </xs:annotation>  
            </xs:attribute>  
            <xs:attribute name="NumberOfStreams" type="xs:int" use="required">  
              <xs:annotation>  
                <xs:documentation>number of streams in asset file</xs:documentation>  
              </xs:annotation>  
            </xs:attribute>  
            <xs:attribute name="FormatNames" type="xs:string" use="required">  
              <xs:annotation>  
                <xs:documentation>format names</xs:documentation>  
              </xs:annotation>  
            </xs:attribute>  
            <xs:attribute name="FormatVerboseName" type="xs:string" use="required">  
              <xs:annotation>  
                <xs:documentation>format verbose names</xs:documentation>  
              </xs:annotation>  
            </xs:attribute>  
            <xs:attribute name="StartTime" type="xs:duration">  
              <xs:annotation>  
                <xs:documentation>content start time. Example: StartTime="PT2.669S"</xs:documentation>  
              </xs:annotation>  
            </xs:attribute>  
            <xs:attribute name="OverallBitRate">  
              <xs:annotation>  
                <xs:documentation>average bitrate of the asset file in kbps</xs:documentation>  
              </xs:annotation>  
              <xs:simpleType>  
                <xs:restriction base="xs:int">  
                  <xs:minInclusive value="0"/>  
                </xs:restriction>  
              </xs:simpleType>  
            </xs:attribute>  
          </xs:complexType>  
        </xs:element>  
      </xs:sequence>  
    </xs:complexType>  
  </xs:element>  
</xs:schema>  
```


## <a name="xml-example"></a><a name="xml"></a> Exemple XML
Voici un exemple de fichier de métadonnées d’entrée.  

```xml
<?xml version="1.0" encoding="utf-8"?>  
<AssetFiles xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/windowsazure/mediaservices/2014/07/mediaencoder/inputmetadata">  
  <AssetFile Name="bear.mp4" Size="1973733" Duration="PT12.678S" NumberOfStreams="2" FormatNames="mov,mp4,m4a,3gp,3g2,mj2" FormatVerboseName="QuickTime / MOV" StartTime="PT0S" OverallBitRate="1245">  
    <VideoTracks>  
      <VideoTrack Id="1" Codec="h264" CodecLongName="H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10" TimeBase="1/29970" NumberOfFrames="375" StartTime="PT0.034S" Duration="PT12.645S" FourCC="avc1" Profile="High" Level="4.1" PixelFormat="yuv420p" Width="512" Height="384" DisplayAspectRatioNumerator="4" DisplayAspectRatioDenominator="3" SampleAspectRatioNumerator="1" SampleAspectRatioDenominator="1" FrameRate="29.656" Bitrate="1043" HasBFrames="1">  
        <Disposition Default="1" Dub="0" Original="0" Comment="0" Lyrics="0" Karaoke="0" Forced="0" HearingImpaired="0" VisualImpaired="0" CleanEffects="0" AttachedPic="0" />  
        <Metadata key="creation_time" value="2010-03-10 16:11:56" />  
        <Metadata key="language" value="eng" />  
        <Metadata key="handler_name" value="Mainconcept MP4 Video Media Handler" />  
      </VideoTrack>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="aac" CodecLongName="AAC (Advanced Audio Coding)" TimeBase="1/44100" NumberOfFrames="546" StartTime="PT0S" Duration="PT12.678S" SampleFormat="fltp" ChannelLayout="stereo" Channels="2" SamplingRate="44100" Bitrate="156" BitsPerSample="0">  
        <Disposition Default="1" Dub="0" Original="0" Comment="0" Lyrics="0" Karaoke="0" Forced="0" HearingImpaired="0" VisualImpaired="0" CleanEffects="0" AttachedPic="0" />  
        <Metadata key="creation_time" value="2010-03-10 16:11:56" />  
        <Metadata key="language" value="eng" />  
        <Metadata key="handler_name" value="Mainconcept MP4 Sound Media Handler" />  
      </AudioTrack>  
    </AudioTracks>  
    <Metadata key="major_brand" value="mp42" />  
    <Metadata key="minor_version" value="0" />  
    <Metadata key="compatible_brands" value="mp42mp41" />  
    <Metadata key="creation_time" value="2010-03-10 16:11:53" />  
    <Metadata key="comment" value="Courtesy of National Geographic.  Used by Permission." />  
  </AssetFile>  
</AssetFiles>  
```

## <a name="next-steps"></a>Étapes suivantes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

