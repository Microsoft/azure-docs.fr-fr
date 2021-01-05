---
title: Méthodes de l'API Lecteur multimédia Azure
description: L'API Lecteur multimédia Azure vous permet d'interagir avec la vidéo via JavaScript, que le navigateur lise la vidéo via HTML5 Video, Flash, Silverlight ou toute autre technologie de lecture prise en charge.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: reference
ms.date: 04/20/2020
ms.openlocfilehash: d1ad0cbe30ce20e61a31f0534a47fe031f6f257b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81725906"
---
# <a name="api"></a>API #

L'API Lecteur multimédia Azure vous permet d'interagir avec la vidéo via JavaScript, que le navigateur lise la vidéo via HTML5 Video, Flash, Silverlight ou toute autre technologie de lecture prise en charge.

## <a name="referencing-the-player"></a>Référencement du lecteur ##

Pour utiliser les fonctions de l'API, vous devez avoir accès à l'objet player. Heureusement, vous pouvez facilement vous le procurer. Il vous suffit de vous assurer que votre balise vidéo possède un ID. Dans l'exemple de code incorporé, l'ID est `vid1`. En présence de plusieurs vidéos sur une même page, vérifiez que chaque balise vidéo possède un ID unique.

`var myPlayer = amp('vid1');`

> [!NOTE]
> Si le lecteur n'a pas encore été initialisé via l'attribut data-setup ou une autre méthode, cela l'initialisera également.

## <a name="wait-until-the-player-is-ready"></a>Attendre que le lecteur soit prêt ##

Le délai nécessaire au Lecteur multimédia Azure pour configurer la vidéo et l'API varie en fonction de la technologie de lecture utilisée. HTML5 est souvent beaucoup plus rapide à charger que Flash ou Silverlight. C'est la raison pour laquelle la fonction « ready » (prêt) du lecteur doit être utilisée pour déclencher tout code nécessitant l'API du lecteur.

```javacript
    amp("vid_1").ready(function(){
      var myPlayer = this;

      // EXAMPLE: Start playing the video.
      myPlayer.play();
    });
```

OR

```javacript
    var myPlayer = amp("vid_1", myOptions, function(){
        //this is the ready function and will only execute after the player is loaded
    });
```

## <a name="api-methods"></a>Méthodes d’API ##

Maintenant que vous avez accès à un lecteur prêt, vous pouvez contrôler la vidéo, obtenir des valeurs ou répondre à des événements vidéo. Les noms des fonctions de l'API Lecteur multimédia Azure s'inspirent de l'[API multimédia HTML5](http://www.whatwg.org/specs/web-apps/current-work/multipage/the-video-element.html). La principale différence est que les fonctions getter/setter sont utilisées pour les propriétés vidéo.

```javacript
    // setting a property on a bare HTML5 video element
    myVideoElement.currentTime = 120;

    // setting a property with Azure Media Player
    myPlayer.currentTime(120);
```

## <a name="registering-for-events"></a>Inscription aux événements ##
Les événements doivent être enregistrés immédiatement après l'initialisation du lecteur afin de veiller à ce que tous les événements soient correctement signalés à l'application, et cela doit être effectué en dehors de l'événement ready.

```javacript
    var myPlayer = amp("vid_1", myOptions, function(){
        //this is the ready function and will only execute after the player is loaded
    });
    myPlayer.addEventListener(amp.eventName.error, _ampEventHandler);
    //add other event listeners
```

## <a name="next-steps"></a>Étapes suivantes ##

<!---Some context for the following links goes here--->
- [Démarrage rapide du Lecteur multimédia Azure](azure-media-player-quickstart.md)