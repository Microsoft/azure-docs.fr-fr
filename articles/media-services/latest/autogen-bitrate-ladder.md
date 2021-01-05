---
title: Encoder des vidéos avec l’encodeur standard dans Azure Media Services | Microsoft Docs
description: Cette rubrique montre comment utiliser l’encodeur standard dans Media Services pour encoder une vidéo d’entrée avec une échelle de débit générée automatiquement en fonction de la résolution d’entrée et du débit.
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
ms.custom: seodec18
ms.openlocfilehash: 05accd69f1868b8b0e0f6dbd4fb5c21ee843ec5e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89297715"
---
#  <a name="encode-with-an-auto-generated-bitrate-ladder"></a>Encoder avec une échelle de débit générée automatiquement

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

## <a name="overview"></a>Vue d’ensemble

Cet article explique comment utiliser l’encodeur standard dans Media Services pour encoder une vidéo d’entrée dans une échelle de débit générée automatiquement (paires débit-résolution) en fonction de la résolution d’entrée et du débit. Ce paramètre d’encodeur intégré, ou présélection, ne dépassera jamais la résolution d’entrée et le débit. Par exemple, si l’entrée est 720p à 3 Mbits/s, la sortie reste à 720p maximum et démarre à des vitesses inférieures à 3 Mbits/s.

### <a name="encoding-for-streaming"></a>Encodage pour la diffusion en continu

Quand vous utilisez le préréglage **AdaptiveStreaming** dans une **transformation**, vous obtenez une sortie qui convient pour une diffusion via des protocoles de streaming comme HLS et DASH. Quand vous utilisez ce préréglage, l’encodeur détermine automatiquement le nombre de couches vidéo à générer, ainsi que le débit et la résolution. Le contenu en sortie contient des fichiers MP4 où l’audio encodé en AAC et la vidéo encodée en H.264 ne sont pas entrelacés.

Pour voir un exemple d’utilisation de cette présélection, consultez [Diffuser un fichier](stream-files-dotnet-quickstart.md).

## <a name="output"></a>Output

Cette section présente trois exemples de couches vidéo de sortie produites par l’encodeur Media Services suite à un encodage avec la présélection **AdaptiveStreaming**. Dans tous les cas, la sortie contient un fichier MP4 audio uniquement avec des données audio stéréo encodées à 128 Kbits/s.

### <a name="example-1"></a>Exemple 1
Une source avec une hauteur de « 1080 » et une fréquence d’images de « 29.970 » crée 6 couches vidéo :

|Couche|Hauteur|Largeur|Débit binaire (Kbits/s)|
|---|---|---|---|
|1|1080|1920|6780|
|2|720|1 280|3520|
|3|540|960|2210|
|4|360|640|1150|
|5|270|480|720|
|6|180|320|380|

### <a name="example-2"></a>Exemple 2
Une source avec une hauteur de « 720 » et une fréquence d’images de « 23.970 » crée 5 couches vidéo :

|Couche|Hauteur|Largeur|Débit binaire (Kbits/s)|
|---|---|---|---|
|1|720|1 280|2940|
|2|540|960|1850|
|3|360|640|960|
|4|270|480|600|
|5|180|320|320|

### <a name="example-3"></a>Exemple 3
Une source avec une hauteur de « 360 » et une fréquence d’images de « 29.970 » crée 3 couches vidéo :

|Couche|Hauteur|Largeur|Débit binaire (Kbits/s)|
|---|---|---|---|
|1|360|640|700|
|2|270|480|440|
|3|180|320|230|

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Diffuser un fichier](stream-files-dotnet-quickstart.md)
