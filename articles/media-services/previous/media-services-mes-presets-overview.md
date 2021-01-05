---
title: Présélections de tâches pour MES (Media Encoder Standard) | Microsoft Docs
description: Cette rubrique vous propose une vue d’ensemble d’exemples de présélections définies par service pour MES (Media Encoder Standard).
author: Juliako
manager: femila
editor: johndeu
services: media-services
documentationcenter: ''
ms.assetid: f243ed1c-ac9c-4300-a5f7-f092cf9853b9
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: aa0514834d1619bbbae4501fe6b1af16d7964ff7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89261473"
---
# <a name="sample-presets-for-media-encoder-standard-mes"></a>Exemples de présélections pour MES (Media Encoder Standard)

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

**Media Encoder Standard** définit un ensemble de présélections d’encodage système prédéfinies à utiliser lors de la création des travaux d’encodage. Il est recommandé d’utiliser les paramètres prédéfinis « Diffusion en continu adaptative » si vous souhaitez encoder une vidéo pour une diffusion en continu avec Media Services. Lorsque vous spécifiez cette présélection, Media Encoder Standard [générera automatiquement une échelle des vitesses de transmission](media-services-autogen-bitrate-ladder-with-mes.md). 

### <a name="creating-custom-presets-from-samples"></a>Création de présélections personnalisées à partir d’exemples
Media Services prend entièrement en charge la personnalisation de toutes les valeurs dans les présélections afin de répondre à vos exigences et vos besoins spécifiques de codage. Si vous avez besoin de personnaliser une valeur d’encodage prédéfinie, nous vous conseillons de commencer par l’une des présélections système ci-dessous qui sont fournies dans cette section comme modèle pour votre configuration personnalisée. Pour consulter une explication de la signification des éléments des présélections et les valeurs valides pour chaque élément, consultez la rubrique [Schéma Media Encoder Standard](media-services-mes-schema.md).  
  
> [!NOTE]
>  Lorsque vous utilisez une présélection pour les encodages 4K, vous devez obtenir le type d’unité réservé `S3`. Pour plus d’informations, consultez [Mise à l’échelle de l’encodage](./media-services-scale-media-processing-overview.md).  

#### <a name="video-rotation-default-setting-in-presets"></a>Paramètre par défaut de rotation de la vidéo dans les présélections :
Lorsque vous utilisez Media Encoder Standard, la rotation vidéo est activée par défaut. Si votre vidéo a été enregistrée sur un appareil mobile en mode Portrait, alors ces présélections font pivoter la font pivoter en mode Paysage avant l’encodage.
 
## <a name="available-presets"></a>Présélections disponibles : 

 [H264 - Vitesse de transmission multiple -1 080 pixels - Audio 5.1](media-services-mes-preset-H264-Multiple-Bitrate-1080p-Audio-5.1.md) produit un ensemble de 8 fichiers MP4 alignés sur le groupe d’images, de 6 000 kbit/s à 400 kbit/s, et de l’audio AAC 5.1.  
  
 [H264 - Vitesse de transmission multiple -1 080 pixels](media-services-mes-preset-H264-Multiple-Bitrate-1080p.md) produit un ensemble de 8 fichiers MP4 alignés sur le groupe d’images, de 6 000 kbit/s à 400 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission multiple -16 x 9 pour iOS](media-services-mes-preset-H264-Multiple-Bitrate-16x9-for-iOS.md) produit un ensemble de 8 fichiers MP4 alignés sur le groupe d’images, de 8 500 kbit/s à 200 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission multiple -16 x 9 SD - Audio 5.1](media-services-mes-preset-H264-Multiple-Bitrate-16x9-SD-Audio-5.1.md) produit un ensemble de 5 fichiers MP4 alignés sur le groupe d’images, de 1 900 kbit/s à 400 kbit/s, et de l’audio AAC 5.1.  
  
 [H264 - Vitesse de transmission multiple -16 x 9 SD](media-services-mes-preset-H264-Multiple-Bitrate-16x9-SD.md) produit un ensemble de 5 fichiers MP4 alignés sur le groupe d’images, de 1 900 kbit/s à 400 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission multiple -4 K - Audio 5.1](media-services-mes-preset-H264-Multiple-Bitrate-4K-Audio-5.1.md) produit un ensemble de 12 fichiers MP4 alignés sur le groupe d’images, de 20 000 kbit/s à 1 000 kbit/s, et de l’audio AAC 5.1.  
  
 [H264 - Vitesse de transmission multiple -4K](media-services-mes-preset-H264-Multiple-Bitrate-4K.md) produit un ensemble de 12 fichiers MP4 alignés sur le groupe d’images, de 20 000 kbit/s à 1 000 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission multiple -4 x 3 pour iOS](media-services-mes-preset-H264-Multiple-Bitrate-4x3-for-iOS.md) produit un ensemble de 8 fichiers MP4 alignés sur le groupe d’images, de 8 500 kbit/s à 200 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission multiple -4 x 3 SD - Audio 5.1](media-services-mes-preset-H264-Multiple-Bitrate-4x3-SD-Audio-5.1.md) produit un ensemble de 5 fichiers MP4 alignés sur le groupe d’images, de 1 600 kbit/s à 400 kbit/s, et de l’audio AAC 5.1.  
  
 [H264 - Vitesse de transmission multiple -4 x 3 pour iOS](media-services-mes-preset-H264-Multiple-Bitrate-4x3-SD.md) produit un ensemble de 5 fichiers MP4 alignés sur le groupe d’images, de 1 600 kbit/s à 400 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission multiple -720 pixels - Audio 5.1](media-services-mes-preset-H264-Multiple-Bitrate-720p-Audio-5.1.md) produit un ensemble de 6 fichiers MP4 alignés sur le groupe d’images, de 3 400 kbit/s à 400 kbit/s, et de l’audio AAC 5.1.  
  
 [H264 - Vitesse de transmission multiple -720 pixels](media-services-mes-preset-H264-Multiple-Bitrate-720p.md) produit un ensemble de 6 fichiers MP4 alignés sur le groupe d’images, de 3400 kbit/s à 400 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission unique - 1 080 pixels - Audio 5.1](media-services-mes-preset-H264-Single-Bitrate-1080p-Audio-5.1.md) produit un seul fichier MP4 avec une vitesse de transmission de 6 750 kbit/s, et de l’audio AAC 5.1.  
  
 [H264 - Vitesse de transmission unique - 1 080 pixels](media-services-mes-preset-H264-Single-Bitrate-1080p.md) produit un seul fichier MP4 avec une vitesse de transmission de 6 750 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission unique - 4K - Audio 5.1](media-services-mes-preset-H264-Single-Bitrate-4K-Audio-5.1.md) produit un seul fichier MP4 avec une vitesse de transmission de 18 000 kbit/s, et de l’audio AAC 5.1.  
  
 [H264 - Vitesse de transmission unique - 4K](media-services-mes-preset-H264-Single-Bitrate-4K.md) produit un seul fichier MP4 avec une vitesse de transmission de 18 000 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission unique - 4 x 3 SD - Audio 5.1](media-services-mes-preset-H264-Single-Bitrate-4x3-SD-Audio-5.1.md) produit un seul fichier MP4 avec une vitesse de transmission de 1 800 kbit/s, et de l’audio AAC 5.1.  
  
 [H264 - Vitesse de transmission unique - 4 x 3 SD](media-services-mes-preset-H264-Single-Bitrate-4x3-SD.md) produit un seul fichier MP4 avec une vitesse de transmission de 1 800 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission unique - 16 x 9 SD - Audio 5.1](media-services-mes-preset-H264-Single-Bitrate-16x9-SD-Audio-5.1.md) produit un seul fichier MP4 avec une vitesse de transmission de 2 200 kbit/s, et de l’audio AAC 5.1.  
  
 [H264 - Vitesse de transmission unique - 16 x 9 SD](media-services-mes-preset-H264-Single-Bitrate-16x9-SD.md) produit un seul fichier MP4 avec une vitesse de transmission de 2 200 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission unique - 720p - Audio 5.1](media-services-mes-preset-H264-Single-Bitrate-720p-Audio-5.1.md) produit un seul fichier MP4 avec une vitesse de transmission de 4 500 kbit/s, et de l’audio AAC 5.1.  
  
 [H264 - Vitesse de transmission unique - 720p pour Android](media-services-mes-preset-H264-Single-Bitrate-720p-for-Android.md) produit un seul fichier MP4 avec une vitesse de transmission de 2 000 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission unique - 720p](media-services-mes-preset-H264-Single-Bitrate-720p.md) produit un seul fichier MP4 avec une vitesse de transmission de 4 500 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission unique haute qualité SD pour Android](media-services-mes-preset-H264-Single-Bitrate-High-Quality-SD-for-Android.md) produit un seul fichier MP4 avec une vitesse de transmission de 500 kbit/s, et de l’audio stéréo AAC.  
  
 [H264 - Vitesse de transmission unique qualité faible SD pour Android](media-services-mes-preset-H264-Single-Bitrate-Low-Quality-SD-for-Android.md) produit un seul fichier MP4 avec une vitesse de transmission de 56 kbit/s, et de l’audio stéréo AAC.  
  
 Pour plus d’informations sur les encodeurs Azure Media Services, consultez la section [Vue d’ensemble et comparaison d’encodeurs multimédia à la demande Azure](./media-services-encode-asset.md).
