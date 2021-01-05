---
title: Guide pratique pour utiliser le lecteur Video.js avec Azure Media Services
description: Cet article explique comment utiliser l’objet vidéo HTML et JavaScript avec Azure Media Services
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
ms.openlocfilehash: 99e17db99842546b7f595d62f82f8bacb0ee6a8a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91329701"
---
# <a name="how-to-use-the-videojs-player-with-azure-media-services"></a>Guide pratique pour utiliser le lecteur Video.js avec Azure Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

## <a name="overview"></a>Vue d’ensemble

Video.js est un lecteur vidéo web conçu pour un monde HTML5. Il lit les formats multimédias adaptatifs (tels que DASH et HLS) dans un navigateur, sans utiliser de plug-ins ni Flash. Au lieu de cela, Video.js utilise les normes web ouvertes extensions MediaSource Extensions et Encrypted Media Extensions. En outre, il prend en charge la lecture vidéo sur les ordinateurs de bureau et les appareils mobiles.

Sa documentation officielle est accessible à l’adresse [https://docs.videojs.com/](https://docs.videojs.com/).

## <a name="sample-code"></a>Exemple de code
Un exemple de code pour cet article est accessible sur cette page  [Azure-Samples/media-services-3rdparty-player-samples](https://github.com/Azure-Samples/media-services-3rdparty-player-samples).

## <a name="implement-the-player"></a>Implémenter le lecteur

1. Créez un fichier `index.html` dans lequel vous hébergerez le lecteur. Ajoutez les lignes de code suivantes (vous pouvez remplacer les versions par des versions plus récentes, le cas échéant) :

    ```html
    <html>
      <head>
        <link href="https://vjs.zencdn.net/7.8.2/video-js.css" rel="stylesheet" />
      </head>
      <body>
        <video id="video" class="video-js vjs-default-skin vjs-16-9" controls data-setup="{}">
        </video>

        <script src="https://vjs.zencdn.net/7.8.2/video.js"></script>
        <script src="https://cdn.jsdelivr.net/npm/videojs-contrib-eme@3.7.0/dist/videojs-contrib-eme.min.js"></script>
        <script type="module" src="index.js"></script>
      </body>
    <html>
    ```

2. Ajoutez un fichier `index.js` avec le code suivant :

    ```javascript
    var videoJS = videojs("video");
    videoJS.src({
      src: "manifestUrl",
      type: "protocolType",
    });
    ```

3. Remplacez `manifestUrl` par l’URL HLS ou DASH du localisateur de diffusion en continu de votre élément multimédia, accessible à partir de la page du localisateur de diffusion en continu dans le portail Azure.
    ![streaming locator URLs](media/how-to-shaka-player/streaming-urls.png)

4. Remplacez `protocolType` par les options suivantes :

    - « application/x-mpegURL » pour les protocoles HLS
    - « application/dash+xml » pour les protocoles DASH

### <a name="set-up-captions"></a>Configurer des sous-titres

Exécutez la méthode `addRemoteTextTrack` et remplacez :

- `subtitleKind` par `"captions"`, `"subtitles"`,`"descriptions"` ou `"metadata"`  
- `caption` par le chemin d’accès du fichier .vtt (le fichier VTT doit être dans le même hôte pour éviter l’erreur CORS)
- `subtitleLang` par le code BCP 47 pour le langage, par exemple, `"eng"` pour l’anglais ou `"es"` pour l’espagnol
- `subtitleLabel` par le nom d’affichage du sous-titre de votre choix

```javascript
videojs.players.video.addRemoteTextTrack({
  kind: subtitleKind,
  src: caption,
  srclang: subtitleLang,
  label: subtitleLabel
});
```

### <a name="set-up-token-authentication"></a>Configurer l’authentification du jeton

Le jeton doit être défini dans le champ d’autorisation de l’en-tête de la demande. Afin d’éviter les problèmes avec CORS, ce jeton doit être défini uniquement dans les demandes dont l’URL contient `'keydeliver'`. Les lignes de code suivantes devraient le permettre :

```javascript
setupTokenForDecrypt (options) {
  if (options.uri.includes('keydeliver')) {
    options.headers = options.headers || {}
    options.headers.Authorization = 'Bearer=' + this.getInputToken()
  }

  return options
}
```

Ensuite, la fonction ci-dessus doit être attachée à l’événement `videojs.Hls.xhr.beforeRequest`.

```javascript
videojs.Hls.xhr.beforeRequest = setupTokenForDecrypt;
```

### <a name="set-up-aes-128-encryption"></a>Configurer le chiffrement AES-128

Video.js prend en charge le chiffrement AES-128 sans aucune configuration supplémentaire. 

> [!NOTE] 
> Il existe actuellement un [problème](https://github.com/videojs/video.js/issues/6717) avec le chiffrement et le contenu CMAF HLS/DASH, qui ne sont pas lisibles.

### <a name="set-up-drm-protection"></a>Configurer la protection DRM

Pour prendre en charge la protection relative à la gestion des droits numériques (DRM), vous devez ajouter l’extension officielle [videojs-contrib-eme](https://github.com/videojs/videojs-contrib-eme). Une version CDN fonctionne également.

1. Dans le fichier `index.js` détaillé ci-dessus, vous devez initialiser l’extension EME en ajoutant `videoJS.eme();` *avant* d’ajouter la source de la vidéo :

   ```javascript
    videoJS.eme();
   ```

2. Vous pouvez maintenant définir les URL des services DRM et celles des licences correspondantes comme suit :

   ```javascript
   videoJS.src({
       keySystems: {
           "com.microsoft.playready": "YOUR PLAYREADY LICENSE URL",
           "com.widevine.alpha": "YOUR WIDEVINE LICENSE URL",
           "com.apple.fps.1_0": {
            certificateUri: "YOUR FAIRPLAY CERTIFICATE URL",
            licenseUri: "YOUR FAIRPLAY LICENSE URL"
           }
         }
       })

   ```

#### <a name="acquiring-the-license-url"></a>Obtention de l’URL de la licence

Pour obtenir l’URL de la licence, vous pouvez :

- consulter la configuration de votre fournisseur DRM ; ou
- si vous utilisez l’exemple, consulter le document `output.json` généré lorsque vous avez exécuté le script PowerShell *setup-vod.ps1* pour les vidéos à la demande ou le script *start-live.ps1* pour les streams en direct. Vous trouverez également les KID dans ce fichier.

#### <a name="using-tokenized-drm"></a>Utilisation de DRM sous forme de jetons

Pour prendre en charge la protection DRM sous forme de jetons, vous devez ajouter la ligne suivante à la propriété `src` du lecteur :

```javascript
videoJS.src({
src: ...,
emeHeaders: {'Authorization': "Bearer=" + "YOUR TOKEN"},
keySystems: {...
```

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser le lecteur multimédia Azure](../azure-media-player/azure-media-player-overview.md)  
- [Démarrage rapide : Chiffrer du contenu](encrypt-content-quickstart.md)
