---
title: Installation complète du Lecteur multimédia Azure
description: Découvrez comment installer le Lecteur multimédia Azure.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: how-to
ms.date: 04/20/2020
ms.custom: devx-track-js
ms.openlocfilehash: 15f5918748df80cec01ccf89835a0ef51da64529
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91296244"
---
# <a name="azure-media-player-full-setup"></a>Installation complète du Lecteur multimédia Azure #

Le lecteur multimédia Azure est facile à configurer. Seuls quelques instants sont nécessaires pour lire du contenu multimédia directement depuis votre compte Media Services. Des [exemples](https://github.com/Azure-Samples/azure-media-player-samples) sont également fournis dans le répertoire d'exemples de la version.


## <a name="step-1-include-the-javascript-and-css-files-in-the-head-of-your-page"></a>Étape 1 : Inclure les fichiers JavaScript et CSS dans l'en-tête de votre page ##

Le Lecteur multimédia Azure vous permet d'accéder aux scripts à partir de la version hébergée du CDN. Il est généralement recommandé de placer JavaScript avant la balise body de fin `<body>` plutôt que `<head>`, mais le Lecteur multimédia Azure inclut un « HTML5 Shiv », qui doit se trouver dans l'en-tête des anciennes versions IE pour respecter la balise vidéo en tant qu’élément valide.

> [!NOTE]
> Si vous utilisez déjà un HTML5 Shiv tel que [Modernizr](https://modernizr.com/), vous pouvez inclure le Lecteur multimédia Azure JavaScript partout. Cela étant, assurez-vous que votre version de Modernizr inclut le Shiv pour la vidéo.

### <a name="cdn-version"></a>Version CDN ###

```html
    <link href="//amp.azure.net/libs/amp/latest/skins/amp-default/azuremediaplayer.min.css" rel="stylesheet">
    <script src= "//amp.azure.net/libs/amp/latest/azuremediaplayer.min.js"></script>
```

> [!IMPORTANT]
> Vous ne devez **PAS** utiliser la version `latest` en production, car elle est susceptible de changer à la demande. Remplacez `latest` par une version du Lecteur multimédia Azure. Par exemple, remplacez `latest` par `2.1.1`. Les versions du Lecteur multimédia Azure peuvent être interrogées à partir de [cet emplacement](azure-media-player-changelog.md).

> [!NOTE]
> Depuis la version `1.2.0`, il n’est plus nécessaire d’inclure l’emplacement aux technologies de secours (elle sélectionne automatiquement l’emplacement à partir du chemin d’accès relatif du fichier azuremediaplayer.min.js). Vous pouvez modifier l’emplacement des technologies de secours en ajoutant le script suivant à `<head>` après les scripts ci-dessus.

> [!NOTE]
> Compte tenu de la nature des plug-ins Flash et Silverlight, les fichiers swf et xap doivent être hébergés dans un domaine ne contenant pas d'informations ou de données sensibles, ce dont se charge automatiquement la version hébergée Azure CDN.

```javascript
    <script>
      amp.options.flashSS.swf = "//amp.azure.net/libs/amp/latest/techs/StrobeMediaPlayback.2.0.swf"
      amp.options.silverlightSS.xap = "//amp.azure.net/libs/amp/latest/techs/SmoothStreamingPlayer.xap"
    </script>
```

## <a name="step-2-add-an-html5-video-tag-to-your-page"></a>Étape 2 : Ajouter une balise vidéo HTML5 à votre page ##

Le Lecteur multimédia Azure vous permet d'utiliser une balise vidéo HTML5 pour incorporer une vidéo. Le Lecteur multimédia Azure lit ensuite la balise et permet son bon fonctionnement dans tous les navigateurs, et pas uniquement ceux qui prennent en charge la vidéo HTML5. Au-delà du balisage de base, le Lecteur multimédia Azure requiert quelques éléments supplémentaires.

1. L’attribut `<data-setup>` de `<video>` indique au Lecteur multimédia Azure de configurer automatiquement la vidéo lorsque la page est prête, et de lire (au format JSON) à partir de l’attribut.
1. L'attribut `id` : doit être utilisé et spécifique à chaque vidéo sur la même page.
1. L’attribut `class` contient deux classes :
    - `azuremediaplayer` applique les styles requis pour la fonctionnalité d'interface utilisateur du Lecteur multimédia Azure
    - `amp-default-skin` applique l’apparence par défaut aux contrôles HTML5
1. L'élément `<source>` inclut deux attributs requis.
    - L'attribut `src` peut inclure un fichier * *.ism/manifest* ajouté à partir d'Azure Media Services, le Lecteur multimédia Azure ajoute automatiquement les URL pour DASH, SMOOTH et TLS au lecteur.
    - L'attribut `type` correspond au type MIME requis du flux. Le type MIME associé à *« .ism/manifest »* est *« application/vnd.ms-sstr+xml »*
1. L'attribut *facultatif* `<data-setup>` de `<source>` indique au Lecteur multimédia Azure s’il existe des stratégies de remise uniques pour le flux d'Azure Media Services, y compris, mais sans s’y limiter, le type de chiffrement (AES ou PlayReady, Widevine ou FairPlay) et le jeton.

Incluez/excluez des attributs, des paramètres, des sources et des pistes comme vous le feriez pour la vidéo HTML5.

```html
    <video id="vid1" class="azuremediaplayer amp-default-skin" autoplay controls width="640" height="400" poster="poster.jpg" data-setup='{"techOrder": ["azureHtml5JS", "flashSS", "html5FairPlayHLS","silverlightSS", "html5"], "nativeControlsForTouch": false}'>
        <source src="http://amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest" type="application/vnd.ms-sstr+xml" />
        <p class="amp-no-js">
            To view this video please enable JavaScript, and consider upgrading to a web browser that supports HTML5 video
        </p>
    </video>
```

Par défaut, le gros bouton de lecture se trouve dans l’angle supérieur gauche pour ne pas obstruer les parties intéressantes qui s'affichent. Si vous préférez centrer le gros bouton de lecture, vous pouvez ajouter une classe `amp-big-play-centered` `class` supplémentaire à votre élément `<video>`.

### <a name="alternative-setup-for-dynamically-loaded-html"></a>Autre configuration pour le code HTML chargé dynamiquement ###

Si votre page ou votre application web charge la balise vidéo dynamiquement (ajax, appendChild, etc.), pour qu'elle soit présente lors du chargement de la page, vous souhaiterez configurer manuellement le lecteur plutôt que d'utiliser l'attribut data-setup. Pour ce faire, supprimez d’abord l’attribut data-setup de la balise afin d'éviter toute confusion lors de l’initialisation du lecteur. Puis, exécutez le code JavaScript suivant une fois le Lecteur multimédia Azure JavaScript chargé et une fois que la balise vidéo chargée dans le DOM.

```javascript
    var myPlayer = amp('vid1', { /* Options */
            techOrder: ["azureHtml5JS", "flashSS", "html5FairPlayHLS","silverlightSS", "html5"],
            "nativeControlsForTouch": false,
            autoplay: false,
            controls: true,
            width: "640",
            height: "400",
            poster: ""
        }, function() {
              console.log('Good to go!');
               // add an event listener
              this.addEventListener('ended', function() {
                console.log('Finished!');
            }
          }
    );
    myPlayer.src([{
        src: "http://samplescdn.origin.mediaservices.windows.net/e0e820ec-f6a2-4ea2-afe3-1eed4e06ab2c/AzureMediaServices_Overview.ism/manifest",
        type: "application/vnd.ms-sstr+xml"
    }]);
```

Le premier argument de la fonction `amp` est l’ID de votre balise vidéo. Remplacez-le par le vôtre.

Le deuxième argument est un objet d’options. Il vous permet de définir des options supplémentaires comme vous pouvez le faire avec l’attribut data-setup.

Le troisième argument est un rappel « ready ». Une fois le Lecteur multimédia Azure initialisé, il appelle cette fonction. Dans le rappel « ready », cet objet fait référence à l'instance de lecteur.

Plutôt que d'utiliser un ID d’élément, vous pouvez également transmettre une référence à l’élément lui-même.

```javascript

    amp(document.getElementById('example_video_1'), {/*Options*/}, function() {
        //This is functionally the same as the previous example.
    });
    myPlayer.src([{ src: "//example/path/to/myVideo.ism/manifest", type: "application/vnd.ms-sstr+xml"]);
```

## <a name="next-steps"></a>Étapes suivantes ##

- [Démarrage rapide du Lecteur multimédia Azure](azure-media-player-quickstart.md)