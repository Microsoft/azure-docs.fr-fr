---
title: Technologie de lecture dans le Lecteur multimédia Azure
description: Apprenez-en davantage sur la technologie de lecture utilisée pour lire du contenu vidéo ou audio.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 04/20/2020
ms.custom: devx-track-js
ms.openlocfilehash: b9772f89f78f21cfb0f2f9baa7c6b9915fbacf37
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91275691"
---
# <a name="playback-technology-tech"></a>Technologie de lecture (« tech ») #

La technologie de lecture fait référence à la technologie particulière de plug-in ou de navigateur qui est utilisée pour lire du contenu vidéo ou audio.

- **azureHtml5JS** : utilise les normes MSE et EME conjointement avec l’élément vidéo pour la lecture sans plug-in du contenu DASH, avec la prise en charge du contenu chiffré d’enveloppe AES 128 bits ou du contenu chiffré commun DRM (via PlayReady et Widevine lorsque le navigateur en assure la prise en charge) depuis Azure Media Services.
- **flashSS** : utilise la technologie Flash Player pour lire du contenu Smooth avec la prise en charge du déchiffrement d’enveloppe AES 128 bits depuis Azure Media Services ; nécessite une version Flash de 11.4 ou supérieure.
- **html5FairPlayHLS** : utilise Safari propre à la technologie de lecture sur un navigateur, via HLS avec l’élément vidéo. Cette technologie demande de lire le contenu protégé FairPlay à partir d’Azure Media Services ; elle est ajoutée à l’ordre des technologies (techOrder) depuis le 19/10/16
- **silverlightSS** : utilise la technologie Silverlight pour lire du contenu Smooth avec la prise en charge du contenu protégé PlayReady depuis Azure Media Services.
- **html5** : s’utilise dans la technologie de lecture sur un navigateur avec l’élément vidéo.  Sur un appareil Apple iOS ou Android, cette technologie permet de lire des flux HLS avec une prise en charge de base du chiffrement d’enveloppe AES 128 bits ou du contenu DRM (via FairPlay quand le navigateur le prend en charge).

## <a name="tech-order"></a>Ordre des technologies ##

Pour vous assurer que votre actif multimédia est lisible par un large choix d’appareils, l’ordre des technologies suivant est recommandé et représente la valeur par défaut : `techOrder: ["azureHtml5JS", "flashSS", "html5FairPlayHLS","silverlightSS", "html5"]` ; il peut être défini directement sur l’élément `<video>`, ou programmatiquement dans les options :

`<video data-setup='{"techOrder": ["azureHtml5JS", "flashSS", "html5FairPlayHLS, "silverlightSS", "html5"]}`

ou

```javascript
    amp("vid1", {
          techOrder: ["azureHtml5JS", "flashSS", "html5FairPlayHLS, "silverlightSS", "html5"]
    });
```

## <a name="compatibility-matrix"></a>Matrice de compatibilité ##

Étant donné l’ordre des technologies recommandé avec le contenu de streaming depuis Azure Media Services, la matrice de compatibilité suivante en matière de lecture est attendue :

| Browser        | Système d’exploitation                                                       | Tech. attendue (Clear)  | Tech. attendue (AES)  | Tech. attendue (DRM)          |
|----------------|----------------------------------------------------------|------------------------|----------------------|------------------------------|
| EdgeIE 11      | Windows 10, Windows 8.1, Windows Phone 101               | azureHtml5JS           | azureHtml5JS         | azureHtml5JS (PlayReady)     |
| IE 11IE 9-101  | Windows 7, Windows Vista<sup>1</sup>                     | flashSS                | flashSS              | silverlightSS (PlayReady)    |
| IE 11          | Windows Phone 8.1                                        | azureHtml5JS           | azureHtml5JS         | non pris en charge                |
| Edge           | Xbox One<sup>1</sup> (mise à jour nov. 2015)                   | azureHtml5JS           | azureHtml5JS         | non pris en charge                |
| Chrome 37+     | Windows 10, Windows 8.1, macOS X Yosemite<sup>1</sup>   | azureHtml5JS           | azureHtml5JS         | azureHtml5JS (Widevine)      |
| Firefox 47+    | Windows 10, Windows 8.1, macOS X Yosemite+<sup>1</sup>  | azureHtml5JS           | azureHtml5JS         | azureHtml5JS (Widevine)      |
| Firefox 42-46  | Windows 10, Windows 8.1, macOS X Yosemite+<sup>1</sup>  | azureHtml5JS           | azureHtml5JS         | silverlightSS (PlayReady)    |
| Firefox 35-41  | Windows 10, Windows 8.1                                  | flashSS                | flashSS              | silverlightSS (PlayReady)    |
| Safari         | iOS 6+                                                   | html5                  | html5 (aucun jeton)3    | non pris en charge                |
| Safari 8+      | OS X Yosemite+                                           | azureHtml5JS           | azureHtml5JS         | html5FairPlayHLS (FairPlay)  |
| Safari 6       | OS X Mountain Lion<sup>1</sup>                           | flashSS                | flashSS              | silverlightSS (PlayReady)    |
| Chrome 37+     | Android 4.4.4+<sup>2</sup>                               | azureHtml5JS           | azureHtml5JS         | azureHtml5JS (Widevine)      |
| Chrome 37+     | Android 4.02                                             | html5                  | html5 (aucun jeton)<sup>3</sup>    | non pris en charge                |
| Firefox 42+    | Android 5.0+<sup>2</sup>                                 | azureHtml5JS           | azureHtml5JS         | non pris en charge                |
| IE 8           | Windows                                                  | non pris en charge          | non pris en charge        | non pris en charge                |

<sup>1</sup> Configuration non prise en charge ou testée ; listé comme référence à achever.

<sup>2</sup> La lecture sur les appareils Android nécessite une combinaison de fonctionnalités de l’appareil, de prise en charge de graphismes, de rendu de codecs, de prise en charge du système d’exploitation et bien plus encore. Android étant une plateforme open source qui autorise les fabricants de téléphone à modifier le système d’exploitation Android Vanilla fourni par Google, certaines fragmentations dans l’espace Android peuvent en résulter, et il est possible que certains appareils ne soient pas pris en charge compte tenu de l’absence de fonctionnalités. Par ailleurs, certains appareils Android n’offrent pas de prise en charge pour tous les codecs.  

<sup>3</sup> Dans les cas où il n’existe aucune prise en charge de jeton, un proxy peut être utilisé pour ajouter cette fonctionnalité. Consultez ce [blog](https://azure.microsoft.com/blog/2015/03/06/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/) pour en savoir plus sur cette solution.

> [!NOTE]
> Si la technologie attendue qui est choisie nécessite l’installation d’un plug-in, tel que Flash, alors qu’il n’est pas installé sur la machine de l’utilisateur, AMP poursuit la vérification des capacités de la technologie suivante, conjointement avec les types de source et les informations de protection, dans la liste des technologies. Par exemple, si vous tentez d’afficher un flux à la demande non protégé dans Safari 8 sur OS X Yosemite, et que ni Flash ni Silverlight ne sont installés, AMP sélectionne la technologie Html5 pour la lecture.<br/><br/>De nouvelles technologies de navigateur apparaissent tous les jours et peuvent par conséquent avoir une incidence sur cette matrice.

## <a name="next-steps"></a>Étapes suivantes ##

- [Démarrage rapide du Lecteur multimédia Azure](azure-media-player-quickstart.md)