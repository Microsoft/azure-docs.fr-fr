---
title: Schéma Media Encoder Standard | Microsoft Docs
description: Cet article décrit certains des éléments et types du schéma XML sur lequel les préréglages Media Encoder Standard sont basés.
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
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 35f79702b7aad51b764ce2edb37c3c76d1fe98e8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89261510"
---
# <a name="media-encoder-standard-schema"></a>Schéma Media Encoder Standard

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Cet article décrit certains des éléments et types du schéma XML sur lequel [les préréglages Media Encoder Standard](media-services-mes-presets-overview.md) sont basés. L’article fournit des explications sur les éléments et leurs valeurs valides.  

## <a name="preset-root-element"></a><a name="Preset"></a> Présélection (élément racine)
Définit une valeur prédéfinie d’encodage.  

### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **Encodage** |[Encodage](media-services-mes-schema.md#Encoding) |Élément racine, indique que les sources d’entrée doivent être encodées. |
| **Sorties** |[Sorties](media-services-mes-schema.md#Output) |Collection de fichiers de sortie souhaités. |
| **StretchMode**<br/>minOccurs="0"<br/>default="AutoSize|xs:string|Contrôlez la taille de trame vidéo de sortie, le remplissage, le pixel ou les proportions de l’affichage. **StretchMode** peut être l’une des valeurs suivantes : **None**, **AutoSize** (par défaut) ou **AutoFit**.<br/><br/>**Aucun** : Suivez strictement la résolution de sortie (par exemple, la **largeur** et la **hauteur** dans la présélection) sans prendre en considération la valeur de proportion de pixels ou d’affichage de la vidéo d’entrée. Recommandé dans les scénarios tels que le [rognage](media-services-crop-video.md), où la vidéo de sortie a des proportions différentes par rapport à l’entrée. <br/><br/>**AutoSize** : La résolution de la sortie tient à l’intérieur de la fenêtre (largeur*hauteur) spécifiée par la présélection. Toutefois, l’encodeur génère une vidéo de sortie qui a une valeur de proportion de pixels carrée (1:1). Par conséquent, la largeur ou hauteur de sortie peut être substituée de manière à conserver les proportions d’affichage de l’entrée, sans remplissage. Par exemple, si l’entrée est 1920x1080 et la présélection d’encodage demande 1280x1280, la valeur de hauteur dans la présélection est remplacée, et la sortie sera à 1280x720, qui conserve les proportions d’entrée en 16:9. <br/><br/>**AutoFit** : Si nécessaire, remplissez la vidéo de sortie (avec un cadre ou le format « pillar box ») pour respecter la résolution de la sortie souhaitée, tout en garantissant que la région active de la vidéo dans la sortie a les mêmes proportions que l’entrée. Par exemple, supposons que l’entrée est 1920x1080 et que la présélection d’encodage demande 1280x1280. La sortie vidéo sera à 1280x1280, mais contiendra un rectangle interne de 1280x720 de « vidéo active » avec les proportions en 16:9, et des zones de cadre de 280 pixels de hauteur en haut et bas. Autre exemple  si l’entrée est 1440x1080 et que la présélection d’encodage demande 1280x720, la sortie sera à 1280x720, avec un rectangle interne de 960x720 aux proportions 4:3 et des régions au format « pillar box » de 160 pixels de large à gauche et à droite. 

### <a name="attributes"></a>Attributs

| Nom | Type | Description |
| --- | --- | --- |
| **Version**<br/><br/> Obligatoire |**xs: decimal** |La version de présélection. Les restrictions suivantes s’appliquent : xs:fractionDigits value="1"  et xs:minInclusive value="1" Par exemple, **version="1.0"** . |

## <a name="encoding"></a><a name="Encoding"></a> Encodage
Contient une séquence des éléments suivants :  

### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **H264Video** |[H264Video](media-services-mes-schema.md#H264Video) |Paramètres d’encodage vidéo H.264. |
| **AACAudio** |[AACAudio](media-services-mes-schema.md#AACAudio) |Paramètres d’encodage audio AAC. |
| **BmpImage** |[BmpImage](media-services-mes-schema.md#BmpImage) |Paramètres de l’image Bmp. |
| **PngImage** |[PngImage](media-services-mes-schema.md#PngImage) |Paramètres de l’image Png. |
| **JpgImage** |[JpgImage](media-services-mes-schema.md#JpgImage) |Paramètres de l’image Jpg. |

## <a name="h264video"></a><a name="H264Video"></a> H264Video
### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **TwoPass**<br/><br/> minOccurs="0" |**xs:boolean** |Actuellement, seul l’encodage en une étape est pris en charge. |
| **KeyFrameInterval**<br/><br/> minOccurs="0"<br/><br/> **default="00:00:02"** |**xs:time** |Détermine l’espacement fixe entre les images IDR en secondes. Également appelé durée GOP. Consultez **SceneChangeDetection** pour savoir si l’encodeur peut s’écarter de cette valeur. |
| **SceneChangeDetection**<br/><br/> minOccurs="0"<br/><br/> default=”false” |**xs: boolean** |Si la valeur est définie sur true, l’encodeur essaie de détecter le changement de scène dans la vidéo et insère une trame IDR. |
| **Complexité**<br/><br/> minOccurs="0"<br/><br/> default="Balanced" |**xs:string** |Contrôle le compromis entre vitesse d’encodage et qualité de la vidéo. Peut être l’une des valeurs suivantes : **Vitesse**, **Équilibrée** ou **qualité**<br/><br/> Valeur par défaut : **Équilibrée** |
| **SyncMode**<br/><br/> minOccurs="0" | |Cette fonctionnalité sera présentée dans une version à venir. |
| **H264Layers**<br/><br/> minOccurs="0" |[H264Layers](media-services-mes-schema.md#H264Layers) |Collection de couches vidéo de sortie. |

### <a name="attributes"></a>Attributs

| Nom | Type | Description |
| --- | --- | --- |
| **Condition** |**xs:string** | Lorsque l’entrée ne comporte aucune vidéo, vous pouvez vouloir forcer l’encodeur à insérer une piste vidéo monochrome. Pour ce faire, utilisez Condition="InsertBlackIfNoVideoBottomLayerOnly" (pour insérer une vidéo uniquement avec le débit le plus bas) ou Condition="InsertBlackIfNoVideo" (pour insérer une vidéo à tous les débits binaires de sortie). Pour plus d’informations, consultez [cet](media-services-advanced-encoding-with-mes.md#no_video) article.|

## <a name="h264layers"></a><a name="H264Layers"></a> H264Layers

Par défaut, si vous envoyez à l’encodeur une entrée contenant uniquement de l’audio (sans contenu vidéo), le composant de sortie regroupe des fichiers qui contiennent uniquement des données audio. Certains lecteurs ne sont peut-être pas capables de gérer ces flux de sortie. Dans ce cas, vous pouvez utiliser l’attribut **InsertBlackIfNoVideo** H264Video pour forcer l’encodeur à ajouter une piste vidéo à la sortie. Pour plus d’informations, consultez [cet](media-services-advanced-encoding-with-mes.md#no_video) article.
              
### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **H264Layer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[H264Layer](media-services-mes-schema.md#H264Layer) |Collection de couches H264. |

## <a name="h264layer"></a><a name="H264Layer"></a> H264Layer
> [!NOTE]
> Les limites vidéo sont basées sur les valeurs décrites dans le tableau [Niveaux H264](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC#Levels).  
> 
> 

### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **Profil**<br/><br/> minOccurs="0"<br/><br/> default=”Auto” |**xs: string** |Peut être l’une des valeurs **xs:string** suivantes : **Auto**, **Ligne de base**, **Principal**, **Élevé**. |
| **Niveau**<br/><br/> minOccurs="0"<br/><br/> default=”Auto” |**xs: string** | |
| **Bitrate**<br/><br/> minOccurs="0" |**xs:int** |Le débit utilisé pour cette couche vidéo, spécifiée en kbit/s. |
| **MaxBitrate**<br/><br/> minOccurs="0" |**xs: int** |Le débit maximal utilisé pour cette couche vidéo, spécifiée en kbit/s. |
| **BufferWindow**<br/><br/> minOccurs="0"<br/><br/> default="00:00:05" |**xs: time** |Longueur de la mémoire tampon vidéo. |
| **Width**<br/><br/> minOccurs="0" |**xs: int** |La largeur de l’image vidéo de sortie, en pixels.<br/><br/> Actuellement, vous devez spécifier la largeur et la hauteur. La largeur et la hauteur doivent être des nombres pairs. |
| **Height**<br/><br/> minOccurs="0" |**xs:int** |La hauteur de l’image vidéo de sortie, en pixels.<br/><br/> Actuellement, vous devez spécifier la largeur et la hauteur. La largeur et la hauteur doivent être des nombres pairs.|
| **BFrames**<br/><br/> minOccurs="0" |**xs: int** |Nombre de trames B entre les trames de référence. |
| **ReferenceFrames**<br/><br/> minOccurs="0"<br/><br/> default=”3” |**xs:int** |Nombre de trames de référence dans un GOP. |
| **EntropyMode**<br/><br/> minOccurs="0"<br/><br/> default=”Cabac” |**xs: string** |Peut être l’une des valeurs suivantes : **Cabac** et **Cavlc**. |
| **FrameRate**<br/><br/> minOccurs="0" |nombre rationnel |Détermine la fréquence d’images de la vidéo de sortie. Utilisez « 0/1 » par défaut pour permettre à l’encodeur d’utiliser la même fréquence d’images que l’entrée vidéo. Les valeurs autorisées doivent idéalement être des fréquences d’images vidéo courantes. Cependant, toute valeur rationnelle est autorisée. Par exemple, 1/1 correspondrait à 1 i/s et serait valide.<br/><br/> - 12/1  (12 i/s)<br/><br/> - 15/1 (15 i/s)<br/><br/> - 24/1 (24 i/s)<br/><br/> - 24000/1001 (23,976 i/s)<br/><br/> - 25/1 (25 i/s)<br/><br/>  - 30/1 (30 i/s)<br/><br/> - 30000/1001 (29,97 i/s) <br/> <br/>**REMARQUE** Si vous créez une présélection personnalisée pour l’encodage multidébit, toutes les couches de la présélection **doivent** utiliser la même valeur de taux de trames (FrameRate).|
| **AdaptiveBFrame**<br/><br/> minOccurs="0" |**xs: boolean** |Copie depuis l’encodeur multimédia Azure |
| **Tranches**<br/><br/> minOccurs="0"<br/><br/> default="0" |**xs:int** |Détermine le nombre de tranches dans lequel une trame est divisée. Nous vous recommandons d'utiliser la valeur par défaut. |

## <a name="aacaudio"></a><a name="AACAudio"></a> AACAudio
 Contient une séquence des éléments et groupes suivants.  

 Pour plus d’informations sur AAC, consultez [AAC](https://en.wikipedia.org/wiki/Advanced_Audio_Coding).  

### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **Profil**<br/><br/> minOccurs="0 "<br/><br/> default="AACLC" |**xs: string** |Peut être l’une des valeurs suivantes : **AACLC**, **HEAACV1** ou **HEAACV2**. |

### <a name="attributes"></a>Attributs

| Nom | Type | Description |
| --- | --- | --- |
| **Condition** |**xs: string** |Pour forcer l’encodeur à produire un élément multimédia contenant une piste audio en mode silencieux lorsque l’entrée ne comporte pas de son, spécifiez la valeur « InsertSilenceIfNoAudio ».<br/><br/> Par défaut, si vous envoyez à l’encodeur une entrée contenant uniquement de la vidéo (sans contenu audio), l’élément multimédia de sortie regroupe les fichiers qui contiennent uniquement des données vidéo. Certains lecteurs ne sont peut-être pas capables de gérer ces flux de sortie. Dans ce cas, vous pouvez utiliser ce paramètre pour forcer l’encodeur à ajouter à la sortie une piste audio en mode silencieux. |

### <a name="groups"></a>Groupes

| Informations de référence | Description |
| --- | --- |
| [AudioGroup](media-services-mes-schema.md#AudioGroup)<br/><br/> minOccurs="0" |Consultez la description de [AudioGroup](media-services-mes-schema.md#AudioGroup) pour connaître le nombre approprié de canaux, le taux d’échantillonnage et le taux de bits qui peuvent être configurés pour chaque profil. |

## <a name="audiogroup"></a><a name="AudioGroup"></a> AudioGroup
Pour plus d’informations sur les valeurs qui sont valides pour chaque profil, consultez le tableau « Détails des codecs audio ».  

### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **Canaux**<br/><br/> minOccurs="0" |**xs: int** |Le nombre de canaux audio encodés. Les valeurs suivantes sont valides : 1, 2, 5, 6, 8.<br/><br/> Valeur par défaut : 2. |
| **SamplingRate**<br/><br/> minOccurs="0" |**xs: int** |Le taux d’échantillonnage audio, spécifié en Hz. |
| **Bitrate**<br/><br/> minOccurs="0" |**xs: int** |Le débit utilisé pour l’encodage de l’audio, spécifié en kbit/s. |

### <a name="audio-codec-details"></a>Détails du codec audio

Codec audio|Détails  
-----------------|---  
**AACLC** |1 :<br/><br/> - 11025: 8 &lt;= bitrate &lt; 16<br/><br/> - 12000: 8 &lt;= bitrate &lt; 16<br/><br/> - 16000: 8 &lt;= bitrate &lt;32<br/><br/>- 22050: 24 &lt;= bitrate &lt; 32<br/><br/> - 24000: 24 &lt;= bitrate &lt; 32<br/><br/> - 32000: 32 &lt;= bitrate &lt;= 192<br/><br/> - 44100: 56 &lt;= bitrate &lt;= 288<br/><br/> - 48000: 56 &lt;= bitrate &lt;= 288<br/><br/> - 88200 : 128 &lt;= bitrate &lt;= 288<br/><br/> - 96000 : 128 &lt;= bitrate &lt;= 288<br/><br/> 2 :<br/><br/> - 11025: 16 &lt;= bitrate &lt; 24<br/><br/> - 12000: 16 &lt;= bitrate &lt; 24<br/><br/> - 16000: 16 &lt;= bitrate &lt; 40<br/><br/> - 22050: 32 &lt;= bitrate &lt; 40<br/><br/> - 24000 : 32 &lt;= bitrate &lt; 40<br/><br/> - 32000:  40 &lt;= bitrate &lt;= 384<br/><br/> - 44100: 96 &lt;= bitrate &lt;= 576<br/><br/> - 48000 : 96 &lt;= bitrate &lt;= 576<br/><br/> - 88200: 256 &lt;= bitrate &lt;= 576<br/><br/> - 96000: 256 &lt;= bitrate &lt;= 576<br/><br/> 5/6:<br/><br/> - 32000: 160 &lt;= bitrate &lt;= 896<br/><br/> - 44100: 240 &lt;= bitrate &lt;= 1024<br/><br/> - 48000: 240 &lt;= bitrate &lt;= 1024<br/><br/> - 88200: 640 &lt;= bitrate &lt;= 1024<br/><br/> - 96000: 640 &lt;= bitrate &lt;= 1024<br/><br/> 8:<br/><br/> - 32000 : 224 &lt;= bitrate &lt;= 1024<br/><br/> - 44100 : 384 &lt;= bitrate &lt;= 1024<br/><br/> - 48000: 384 &lt;= bitrate &lt;= 1024<br/><br/> - 88200: 896 &lt;= bitrate &lt;= 1024<br/><br/> - 96000: 896 &lt;= bitrate &lt;= 1024  
**HEAACV1** |1 :<br/><br/> - 22050: bitrate = 8<br/><br/> - 24000: 8 &lt;= bitrate &lt;= 10<br/><br/> - 32000: 12 &lt;= bitrate &lt;= 64<br/><br/> - 44100: 20 &lt;= bitrate &lt;= 64<br/><br/> - 48000: 20 &lt;= bitrate &lt;= 64<br/><br/> - 88200: bitrate = 64<br/><br/> 2 :<br/><br/> - 32000: 16 &lt;= bitrate &lt;= 128<br/><br/> - 44100: 16 &lt;= bitrate &lt;= 128<br/><br/> - 48000: 16 &lt;= bitrate &lt;= 128<br/><br/> - 88200 : 96 &lt;= bitrate &lt;= 128<br/><br/> - 96000: 96 &lt;= bitrate &lt;= 128<br/><br/> 5/6:<br/><br/> - 32000 : 64 &lt;= bitrate &lt;= 320<br/><br/> - 44100: 64 &lt;= bitrate &lt;= 320<br/><br/> - 48000: 64 &lt;= bitrate &lt;= 320<br/><br/> - 88200 : 256 &lt;= bitrate &lt;= 320<br/><br/> - 96000: 256 &lt;= bitrate &lt;= 320<br/><br/> 8:<br/><br/> - 32000: 96 &lt;= bitrate &lt;= 448<br/><br/> - 44100: 96 &lt;= bitrate &lt;= 448<br/><br/> - 48000: 96 &lt;= bitrate &lt;= 448<br/><br/> - 88200: 384 &lt;= bitrate &lt;= 448<br/><br/> - 96000: 384 &lt;= bitrate &lt;= 448  
**HEAACV2** |2 :<br/><br/> - 22050: 8 &lt;= bitrate &lt;= 10<br/><br/> - 24000: 8 &lt;= bitrate &lt;= 10<br/><br/> - 32000: 12 &lt;= bitrate &lt;= 64<br/><br/> - 44100: 20 &lt;= bitrate &lt;= 64<br/><br/> - 48000: 20 &lt;= bitrate &lt;= 64<br/><br/> - 88200: 64 &lt;= bitrate &lt;= 64  
  
## <a name="clip"></a><a name="Clip"></a> Séquence
### <a name="attributes"></a>Attributs

| Nom | Type | Description |
| --- | --- | --- |
| **StartTime** |**xs:duration** |Spécifie l’heure de début d’une présentation. La valeur de StartTime doit correspondre aux horodatages absolus de la vidéo d'entrée. Par exemple, si la première image de la vidéo d'entrée a un horodatage de 12:00:10.000, la valeur de StartTime doit être égale ou supérieure à 12:00:10.000. |
| **Durée** |**xs:duration** |Spécifie la durée d’une présentation (par exemple, apparition d’une superposition sur la vidéo). |

## <a name="output"></a><a name="Output"></a> Sortie
### <a name="attributes"></a>Attributs

| Nom | Type | Description |
| --- | --- | --- |
| **FileName** |**xs:string** |Le nom du fichier de sortie.<br/><br/> Vous pouvez utiliser les macros décrites dans le tableau suivant pour générer les noms de fichier de sortie. Par exemple :<br/><br/> **"Outputs": [      {       "FileName": "{Basename} *{Resolution}* {Bitrate}.mp4",       "Format": {         "Type": "MP4Format"       }     }   ]** |

### <a name="macros"></a>Macros

| Macro | Description |
| --- | --- |
| **{Basename}** |Si vous effectuez l’encodage de VoD, {Basename} est composé des 32 premiers caractères de la propriété AssetFile.Name du fichier principal de la ressource d’entrée.<br/><br/> Si la ressource d’entrée est une archive en direct, {Basename} est dérivé des attributs trackName dans le manifeste du serveur. Si vous envoyez un travail de sous-clip avec TopBitrate, comme dans : « <VideoStream\>TopBitrate</VideoStream\> » et que le fichier de sortie contient de la vidéo, {Basename} correspond alors aux 32 premiers caractères du trackName de la couche vidéo avec le débit le plus élevé.<br/><br/> Par contre, si vous envoyez un travail de sous-clip avec tous les débits d’entrée, comme dans : « <VideoStream\>*</VideoStream\> » et que le fichier de sortie contient de la vidéo, {Basename} correspond alors aux 32 premiers caractères du trackName de la couche vidéo associée. |
| **{Codec}** |Mappage avec « H264 » pour la vidéo et « AAC » pour l’audio. |
| **{Bitrate}** |Le débit vidéo cible si le fichier de sortie contient de la vidéo et de l’audio, ou le débit audio cible si le fichier de sortie contient uniquement des données audio. La valeur utilisée est le débit en kbit/s. |
| **{Channel}** |Nombre de canaux audio si le fichier contient des données audio. |
| **{Width}** |Largeur de la vidéo en pixels dans le fichier de sortie, si le fichier contient de la vidéo. |
| **{Height}** |Hauteur de la vidéo en pixels dans le fichier de sortie, si le fichier contient de la vidéo. |
| **{Extension}** |Hérite de la propriété « Type » du fichier de sortie. Le nom de fichier de sortie a l’une des extensions suivantes : « mp4 », « ts », « jpg », « png » ou « bmp ». |
| **{Index}** |Obligatoire pour la miniature. Ne doit être présent qu’une seule fois. |

## <a name="video-complex-type-inherits-from-codec"></a><a name="Video"></a> Vidéo (le type complexe hérite de Codec)
### <a name="attributes"></a>Attributs

| Nom | Type | Description |
| --- | --- | --- |
| **Start** |**xs:string** | |
| **Étape** |**xs:string** | |
| **Plage** |**xs:string** | |
| **PreserveResolutionAfterRotation** |**xs:boolean** |Pour une explication détaillée, voir la section suivante : [PreserveResolutionAfterRotation](media-services-mes-schema.md#PreserveResolutionAfterRotation) |

### <a name="preserveresolutionafterrotation"></a><a name="PreserveResolutionAfterRotation"></a> PreserveResolutionAfterRotation
Nous vous recommandons d’utiliser l’indicateur **PreserveResolutionAfterRotation** en combinaison avec les valeurs de résolution exprimées en pourcentage (Width="100%" , Height="100%").  

Par défaut, les paramètres de résolution d’encodage (largeur, hauteur) dans les paramètres prédéfinis de Media Encoder Standard (MES) sont destinés aux vidéos avec une rotation de zéro degré. Par exemple, si votre vidéo d’entrée a une résolution de 1280 x 720 avec une rotation de zéro degré, les paramètres prédéfinis garantissent que la sortie a la même résolution.  

![MESRoation1](./media/media-services-shemas/media-services-mes-roation1.png) 

Si la vidéo d’entrée a été capturée avec une rotation différente de zéro (par exemple, un smartphone ou une tablette à la verticale), MES applique par défaut les paramètres de résolution d’encodage (largeur, hauteur) à l’entrée vidéo, puis il compense la rotation. Consultez par exemple l’image qui suit. La présélection utilise les valeurs Width = “100%”, Height = “100%”, que MES interprète comme une exigence que la sortie soit de 1280 pixels de large et de 720 pixels de haut. Après rotation de la vidéo, MES réduit ensuite l’image pour tenir dans cette fenêtre, ce qui cause l’affichage de bandes sur la gauche et la droite.  

![MESRoation2](./media/media-services-shemas/media-services-mes-roation2.png) 

Vous pouvez aussi utiliser l’indicateur **PreserveResolutionAfterRotation** et lui affecter la valeur « true » (la valeur par défaut est « false »). Ainsi, si votre présélection a Width = “100%”, Height = “100%” et que PreserveResolutionAfterRotation a la valeur « true », une vidéo d’entrée qui fait 1280 pixels de large et 720 pixels de hauteur avec une rotation de 90 degrés produit une sortie avec une rotation de zéro degré, mais faisant 720 pixels de large et 1280 pixels de hauteur. Consultez l’illustration suivante :  

![MESRoation3](./media/media-services-shemas/media-services-mes-roation3.png) 

## <a name="formatgroup-group"></a><a name="FormatGroup"></a> FormatGroup (groupe)
### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **BmpFormat** |**BmpFormat** | |
| **PngFormat** |**PngFormat** | |
| **JpgFormat** |**JpgFormat** | |

## <a name="bmplayer"></a><a name="BmpLayer"></a> BmpLayer
### <a name="element"></a>Élément

| Nom | Type | Description |
| --- | --- | --- |
| **Width**<br/><br/> minOccurs="0" |**xs:int** | |
| **Height**<br/><br/> minOccurs="0" |**xs:int** | |

### <a name="attributes"></a>Attributs

| Nom | Type | Description |
| --- | --- | --- |
| **Condition** |**xs:string** | |

## <a name="pnglayer"></a><a name="PngLayer"></a> PngLayer
### <a name="element"></a>Élément

| Nom | Type | Description |
| --- | --- | --- |
| **Width**<br/><br/> minOccurs="0" |**xs:int** | |
| **Height**<br/><br/> minOccurs="0" |**xs:int** | |

### <a name="attributes"></a>Attributs

| Nom | Type | Description |
| --- | --- | --- |
| **Condition** |**xs:string** | |

## <a name="jpglayer"></a><a name="JpgLayer"></a> JpgLayer
### <a name="element"></a>Élément

| Nom | Type | Description |
| --- | --- | --- |
| **Width**<br/><br/> minOccurs="0" |**xs:int** | |
| **Height**<br/><br/> minOccurs="0" |**xs:int** | |
| **Qualité**<br/><br/> minOccurs="0" |**xs:int** |Valeurs valides : 1(worst)-100(best) |

### <a name="attributes"></a>Attributs

| Nom | Type | Description |
| --- | --- | --- |
| **Condition** |**xs:string** | |

## <a name="pnglayers"></a><a name="PngLayers"></a> PngLayers
### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **PngLayer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[PngLayer](media-services-mes-schema.md#PngLayer) | |

## <a name="bmplayers"></a><a name="BmpLayers"></a> BmpLayers
### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **BmpLayer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[BmpLayer](media-services-mes-schema.md#BmpLayer) | |

## <a name="jpglayers"></a><a name="JpgLayers"></a> JpgLayers
### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **JpgLayer**<br/><br/> minOccurs="0" maxOccurs="unbounded" |[JpgLayer](media-services-mes-schema.md#JpgLayer) | |

## <a name="bmpimage-complex-type-inherits-from-video"></a><a name="BmpImage"></a> BmpImage (le type complexe hérite de Video)
### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **PngLayers**<br/><br/> minOccurs="0" |[PngLayers](media-services-mes-schema.md#PngLayers) |Couches PNG |

## <a name="jpgimage-complex-type-inherits-from-video"></a><a name="JpgImage"></a> JpgImage (le type complexe hérite de Video)
### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **PngLayers**<br/><br/> minOccurs="0" |[PngLayers](media-services-mes-schema.md#PngLayers) |Couches PNG |

## <a name="pngimage-complex-type-inherits-from-video"></a><a name="PngImage"></a> PngImage (le type complexe hérite de Video)
### <a name="elements"></a>Éléments

| Nom | Type | Description |
| --- | --- | --- |
| **PngLayers**<br/><br/> minOccurs="0" |[PngLayers](media-services-mes-schema.md#PngLayers) |Couches PNG |

## <a name="examples"></a>Exemples
Pour voir des exemples de présélections XML qui sont générées conformément à ce schéma, consultez [Présélections de tâches pour MES (Media Encoder Standard)](media-services-mes-presets-overview.md).

## <a name="next-steps"></a>Étapes suivantes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

