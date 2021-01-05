---
title: Réécriture d’URL du Lecteur multimédia Azure
description: Le Lecteur multimédia Azure doit réécrire une URL donnée à partir d’Azure Media Services pour fournir des flux à SMOOTH, DASH, TLS v3 et HLS v4.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 04/20/2020
ms.custom: devx-track-js
ms.openlocfilehash: b037eec13fda0b5ec16a4f2f53ad2a64fb5f8da1
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91275555"
---
# <a name="url-rewriter"></a>Réécriture d’URL #

Par défaut, le Lecteur multimédia Azure réécrit une URL donnée à partir d’Azure Media Services pour fournir des flux à SMOOTH, DASH, HLS v3 et HLS v4. Par exemple, si la source est indiquée comme suit, le Lecteur multimédia Azure garantit qu’il tente de lire tous les protocoles ci-dessus :

```html
    <video id="vid1" class="azuremediaplayer amp-default-skin">
        <source src="//example/path/to/myVideo.ism/manifest" type="application/vnd.ms-sstr+xml" />
    </video>
```

Toutefois, si vous ne souhaitez pas utiliser la réécriture d’URL, il vous suffit de le préciser en ajoutant la propriété `disableUrlRewriter` au paramètre. Cela signifie que toutes les informations transmises aux sources sont directement transmises au Lecteur, sans modification.  Voici un exemple d’ajout de deux sources au Lecteur, l’une DASH et l’autre SMOOTH Streaming.

```html
    <video id="vid1" class="azuremediaplayer amp-default-skin">
        <source src="//example/path/to/myVideo.ism/manifest(format=mpd-time-csf)" type="application/dash+xml" data-setup='{"disableUrlRewriter": true}'/>
        <source src="//example/path/to/myVideo.ism/manifest" type="application/vnd.ms-sstr+xml" data-setup='{"disableUrlRewriter": true}'/>
    </video>
```

ou

```javascript
    myPlayer.src([
        { src: "//example/path/to/myVideo.ism/manifest(format=mpd-time-csf)", type: "application/dash+xml", disableUrlRewriter: true },
        { src: "//example/path/to/myVideo.ism/manifest", type: "application/vnd.ms-sstr+xml", disableUrlRewriter: true }
    ]);
```

De plus, si vous le souhaitez, vous pouvez indiquer les formats de streaming particuliers vers lesquels vous voudriez que le Lecteur multimédia Azure réécrive à l’aide du paramètre `streamingFormats`. Ces options comprennent `DASH`, `SMOOTH`, `HLSv3`, `HLSv4`, `HLS`. La différence entre HLS et HLSv3 & v4 vient de la prise en charge par le format HLS de la lecture du contenu FairPlay. v3 et v4 ne prennent pas en charge FairPlay. C’est utile si vous ne disposez d’aucune stratégie de distribution pour un protocole particulier disponible.  Voici un exemple dans lequel un protocole DASH n’est pas activé avec votre actif multimédia.

```html
    <video id="vid1" class="azuremediaplayer amp-default-skin">
        <source src="//example/path/to/myVideo.ism/manifest" type="application/vnd.ms-sstr+xml" data-setup='{"streamingFormats": ["SMOOTH", "HLS","HLS-V3", "HLS-V4"] }'/>
    </video>
```

ou

```javascript
    myPlayer.src([
        { src: "//example/path/to/myVideo.ism/manifest", type: "application/vnd.ms-sstr+xml", streamingFormats: ["SMOOTH", "HLS","HLS-V3", "HLS-V4"]},
    ]);
```

Les deux extraits ci-dessus peuvent être utilisés en étant combinés l’un à l’autre pour diverses circonstances, en fonction de votre actif multimédia particulier.

> [!NOTE]
> Les informations de protection Widevine ne persistent que sur le protocole DASH.

## <a name="next-steps"></a>Étapes suivantes ##

- [Démarrage rapide du Lecteur multimédia Azure](azure-media-player-quickstart.md)