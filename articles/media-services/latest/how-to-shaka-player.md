---
title: Guide pratique pour utiliser le lecteur Shaka avec Azure Media Services
description: Cet article explique comment utiliser le lecteur Shaka avec Azure Media Services
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-js
ms.openlocfilehash: e4be3d4508d342c77507cf824fb036e32084b617
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91329735"
---
# <a name="how-to-use-the-shaka-player-with-azure-media-services"></a>Guide pratique pour utiliser le lecteur Shaka avec Azure Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

## <a name="overview"></a>Vue d’ensemble

Shaka Player est une bibliothèque JavaScript open source destinée au contenu multimédia adaptatif. Elle lit les formats multimédias adaptatifs (tels que DASH et HLS) dans un navigateur, sans utiliser de plug-ins ni Flash. À la place, le lecteur Shaka utilise les normes web ouvertes Media Source Extensions et Encrypted Media Extensions.

Nous vous conseillons d’utiliser [Mux.js](https://github.com/videojs/mux.js/) car, sans lui, le lecteur Shaka prend en charge le format HLS CMAF, mais pas HLS TL.

Sa documentation officielle est disponible dans [Documentation de Shaka Player](https://shaka-player-demo.appspot.com/docs/api/tutorial-welcome.html).

## <a name="sample-code"></a>Exemple de code
Un exemple de code pour cet article est accessible sur cette page [Azure-Samples/media-services-3rdparty-player-samples](https://github.com/Azure-Samples/media-services-3rdparty-player-samples).

## <a name="implementing-the-player"></a>Implémentation du lecteur

Suivez ces instructions si vous avez besoin d’implémenter votre propre instance du lecteur :

1. Créez un fichier `index.html` dans lequel vous hébergerez le lecteur. Ajoutez les lignes de code suivantes (vous pouvez remplacer les versions par des versions plus récentes, le cas échéant) :

    ```html
    <html>
      <head>
        <script src="//cdn.jsdelivr.net/npm/shaka-player@3.0.1/dist/shaka-player.compiled.js"></script>
        <script src="//cdn.jsdelivr.net/npm/mux.js@5.6.3/dist/mux.js"></script>
        <script type="module" src="index.js"></script>
      </head>
      <body>
        <video id="video" controls></video>
      </body>
    </html>
    ```

1. Ajoutez un fichier JavaScript avec le code suivant :

    ```javascript
    // myScript.js
    shaka.polyfill.installAll();

    var video = document.getElementById('video');
    var player = new shaka.Player(video);
    window.player = player;

    var manifestUrl = 'https://amsplayeraccount-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/sample-vod.ism/manifest(format=m3u8-aapl)';
    player.load(manifestUrl);
    ```

1. Remplacez `manifestUrl` par l’URL HLS ou DASH du localisateur de streaming de votre élément multimédia, accessible à partir de la page du localisateur de streaming, dans le portail Azure.
    ![URL du localisateur de streaming](media/how-to-shaka-player/streaming-urls.png)

1. Exécutez un serveur (par exemple avec `npm http-server`) et votre lecteur devrait fonctionner...

## <a name="set-up-captions"></a>Configurer les sous-titres

### <a name="set-up-vod-captions"></a>Configurer les sous-titres VOD

Exécutez les lignes de code suivantes et remplacez `captionUrl` par votre répertoire .vtt (le fichier vtt doit se trouver dans le même hôte pour éviter une erreur CORS), `lang` par le code à deux lettres pour la langue, et `type` par `caption` ou `subtitle` :

```javascript
player.configure('streaming.alwaysStreamText', true)
player.load(manifestUrl).then(function(){
        player.addTextTrack(captionUrl, lang, type, 'text/vtt');
        var tracks = player.getTextTracks();
        player.selectTextTrack(tracks[0]);
});
```

### <a name="set-up-live-stream-captions"></a>Configurer les sous-titres de stream en direct

L’activation des sous-titres dans le stream en direct est configurée par l’ajout de la ligne de code suivante :

```javascript
player.setTextTrackVisibility(true)
```

## <a name="set-up-token-authentication"></a>Configurer l’authentification du jeton

Exécutez les lignes de code suivantes et remplacez `token` par votre chaîne de jeton :

```javascript
player.getNetworkingEngine().registerRequestFilter(function (type, request) {
  if (type === shaka.net.NetworkingEngine.RequestType.LICENSE) {
    request.headers['Authorization'] = 'Bearer ' + token;
  }
});
```

## <a name="set-up-aes-128-encryption"></a>Configurer le chiffrement AES-128

Shaka Player ne prend pas actuellement en charge le chiffrement AES-128.

Lien vers un [problème](https://github.com/google/shaka-player/issues/850) GitHub pour suivre l’état de cette fonctionnalité.

## <a name="set-up-drm-protection"></a>Configurer la protection DRM

Shaka Player utilise Encrypted Media Extensions (EME), qui demande l’utilisation d’une URL sécurisée. Ainsi, pour tester un contenu DRM protégé, il est nécessaire d’utiliser le protocole https. Si le site utilise https, le manifeste et chaque segment devront également utiliser le protocole https. Des exigences en matière de contenu mixte en sont la cause.

Voici l’ordre de préférence de Shaka pour la gestion des URL de ses serveurs de licences :

1. ClearKey config, utilisé pour le débogage, doit se substituer à tout le reste. (L’application peut toujours spécifier un serveur de licences ClearKey.)
2. Les serveurs configurés par l’application, s’ils existent, doivent se substituer à tout ce qui émane du manifeste.
3. Les serveurs de licences fournis par le manifeste sont utilisés uniquement si rien d’autre n’est précisé.

Vous pouvez spécifier l’URL du serveur de licences pour Widevine en utilisant le code suivant :

```javascript
player.configure({
  drm: {
    servers: {
      "com.widevine.alpha": "YOUR WIDEVINE LICENSE URL",
      "com.microsoft.playready": "YOUR PLAYREADY LICENSE URL"
    }
  }
});

```

Tout le contenu FairPlay demande la définition d’un certificat de serveur. Il est défini dans la configuration du lecteur :

```javascript
const req = await fetch("YOUR FAIRPLAY CERTIFICATE URL");
const cert = await req.arrayBuffer();
player.configure('drm.advanced.com\\.apple\\.fps\\.1_0.serverCertificate', new Uint8Array(cert));
```

Pour plus d’informations, consultez la [documentation sur la protection DRM de Shaka Player](https://shaka-player-demo.appspot.com/docs/api/tutorial-drm-config.html).

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser le lecteur multimédia Azure](../azure-media-player/azure-media-player-overview.md)
* [Démarrage rapide : Chiffrer du contenu](encrypt-content-quickstart.md)
