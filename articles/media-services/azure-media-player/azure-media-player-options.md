---
title: Options du Lecteur multimédia Azure
description: Le code incorporé du Lecteur multimédia Azure est constitué d'une simple balise vidéo HTML5. Par conséquent, vous pouvez utiliser les attributs des balises standard pour définir de nombreuses options.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: reference
ms.date: 04/20/2020
ms.custom: devx-track-js
ms.openlocfilehash: 67f1f2fe3f6ac1061f1edcd22532bffaf0a1e815
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92366868"
---
# <a name="options"></a>Options #

## <a name="setting-options"></a>Configuration des options ##

Le code incorporé du Lecteur multimédia Azure est constitué d'une simple balise vidéo HTML5. Par conséquent, vous pouvez utiliser les attributs des balises standard pour définir de nombreuses options.

`<video controls autoplay ...>`

Vous pouvez aussi utiliser l'attribut data-setup pour fournir des options au format [JSON](http://json.org/example.html). Cette méthode peut également être utilisée pour définir des options non standard sur la balise vidéo.

`<video data-setup='{ "controls": true, "autoplay": false }'...>`

Enfin, si vous n'utilisez pas l'attribut data-setup pour déclencher la configuration du lecteur, vous pouvez transmettre un objet doté des options du lecteur comme deuxième argument dans la fonction de configuration JavaScript.

`amp("vid1", { "controls": true, "autoplay": false });`

> [!NOTE]
> Les options du constructeur ne sont définies que lors de la première initialisation, avant que la source soit définie.  Si vous souhaitez modifier les options sur le même élément initialisé du Lecteur multimédia Azure, vous devez mettre les options à jour avant de modifier la source. Pour mettre les options à jour en JavaScript, utilisez `myPlayer.options({/*updated options*/});`. Notez que seules les options modifiées seront affectées ; toutes les autres options précédemment définies seront conservées.

## <a name="individual-options"></a>Options individuelles ##

> [!NOTE]
>Les attributs des balises vidéo ne peuvent être définis que sur true ou false (booléens). Il suffit d'inclure l'attribut (signe différent de) pour l'activer, ou de l'exclure pour le désactiver. Par exemple, pour activer les contrôles : INCORRECT `<video controls="true" ...>` CORRECT `<video controls ...>` Le plus difficile pour les utilisateurs est d'essayer de définir ces valeurs sur false en utilisant false comme valeur (par exemple, controls="false"), ce qui produit l'inverse et définit la valeur sur true car l'attribut est toujours inclus.

### <a name="controls"></a>controls ###

L'option controls détermine si le lecteur dispose ou non de contrôles avec lesquels l'utilisateur peut interagir. Sans contrôles, la seule façon de démarrer la lecture vidéo est d'utiliser l'attribut autoplay ou l'API.

`<video controls ...>` ou `{ "controls": true }`

### <a name="autoplay"></a>autoplay ###

Si autoplay est défini sur true, la lecture de la vidéo commence dès que la page est chargée (sans aucune interaction de la part de l'utilisateur).

> [!NOTE]
> Cette option n'est pas prise en charge par les appareils mobiles tels que Windows Phone, Apple iOS et Android. Les appareils mobiles bloquent la fonctionnalité de lecture automatique pour éviter une utilisation excessive des forfaits de données mensuels (souvent coûteux) des consommateurs. Dans ce cas, l'utilisateur doit toucher/cliquer pour lancer la vidéo.

`<video autoplay ...>`ou `{ "autoplay": true }`

### <a name="poster"></a>poster ###
L'attribut poster définit l'image qui s'affiche avant la lecture de la vidéo. Il s'agit souvent d'une image de la vidéo ou d'un écran de titre personnalisé. Dès que l'utilisateur clique sur Lire, l'image disparaît.

`<video poster="myPoster.jpg" ...>` ou `{ "poster": "myPoster.jpg" }`

### <a name="width"></a>width ###

L'attribut width définit la largeur d'affichage de la vidéo.

`<video width="640" ...>` ou `{ "width": 640 }`

### <a name="height"></a>height ###

L'attribut Height définit la hauteur d'affichage de la vidéo.

`<video height="480" ...>` ou `{ "height": 480 }`

### <a name="plugins"></a>plugins ###

Le JSON plugins détermine les plug-ins chargés avec cette instance du Lecteur multimédia Azure et vous permet de configurer toutes les options correspondantes.

   `<video... data-setup='{plugins: { "contentTitle": {"name": "Azure Medi Services Overview"}}}'...>`

Pour plus d'informations sur le développement et l'utilisation des plug-ins, consultez [Créer des plug-ins](azure-media-player-writing-plugins.md).

### <a name="other-options"></a>autres options ###

D'autres options peuvent être définies sur la balise `<video>` à l'aide du paramètre `data-setup` qui accepte un JSON.
`<video ... data-setup='{"nativeControlsForTouch": false}'>`

#### <a name="nativecontrolsfortouch"></a>nativeControlsForTouch ####

Cette option est explicitement définie sur false. En la définissant sur false, l'apparence du Lecteur multimédia Azure peut être identique d'une plateforme à une autre.  De plus, le toucher restera toujours activé.

### <a name="fluid"></a>fluid ###

En définissant cette option sur true, l'élément vidéo prendra toute la largeur du conteneur parent et la hauteur sera ajustée pour s'adapter à une vidéo aux proportions 16:9 standard.

`<video ... data-setup='{"fluid": true}'>`

L'option `fluid` prévaut sur les paramètres explicites `width` et `height`. Cette option n'est disponible qu'à partir de la version `2.0.0` du Lecteur multimédia Azure.

### <a name="playbackspeed"></a>playbackSpeed ###

L'option `playbackSpeed` contrôle la vitesse de lecture (contrôle playbackSpeed) et l'ensemble des paramètres de vitesse de lecture accessibles à l'utilisateur. `playbackSpeed` accepte un objet. Afin d'activer le contrôle de vitesse de lecture sur la barre de contrôle, la propriété `enabled` de l'objet doit être définie sur true. Exemple d'activation de la vitesse de lecture dans la balise :

`<video ... data-setup='{"playbackSpeed": {"enabled": true}}'>`


Les autres propriétés du paramètre `playbackSpeed` sont fournies par l'objet [PlaybackSpeedOptions](/javascript/api/azuremediaplayer/playbackspeedoptions).

Exemple de configuration des options de vitesse de lecture en JavaScript :

```javascript
    var myPlayer = amp('vid1', {
        fluid: true,
        playbackSpeed: {
            enabled: true,
            initialSpeed: 1.0,
            speedLevels: [
                { name: "x4.0", value: 4.0 },
                { name: "x3.0", value: 3.0 },
                { name: "x2.0", value: 2.0 },
                { name: "x1.75", value: 1.75 },
                { name: "x1.5", value: 1.5 },
                { name: "x1.25", value: 1.25 },
                { name: "normal", value: 1.0 },
                { name: "x0.75", value: 0.75 },
                { name: "x0.5", value: 0.5 },
            ]
        }
    });
```

Cette option n'est disponible qu'à partir de la version 2.0.0 du Lecteur multimédia Azure.

### <a name="staledatatimelimitinsec"></a>staleDataTimeLimitInSec ###

L'option `staleDataTimeLimitInSec` est une optimisation qui vous permet de configurer le nombre de secondes de données périmées que vous souhaitez conserver dans les tampons mediaSource. Elle est désactivée par défaut.

### <a name="cea708captionssettings"></a>cea708CaptionsSettings ###

En définissant enabled sur true, vous pouvez afficher les légendes CEA en direct dans vos flux et archives en direct. L'attribut label n'est pas obligatoire. S'il n'est pas inclus, le lecteur reviendra à une étiquette par défaut.

```javascript
     cea708CaptionsSettings: {
                enabled: true,
                srclang: 'en',
                label: 'Live CC'
            }
```

Cette option est uniquement disponible à partir de la version 2.1.1 du Lecteur multimédia Azure.

## <a name="next-steps"></a>Étapes suivantes ##

- [Démarrage rapide du Lecteur multimédia Azure](azure-media-player-quickstart.md)
