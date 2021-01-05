---
title: Incorporation d’une vidéo de diffusion en continu adaptative MPEG-DASH dans une application HTML5 avec DASH.js | Microsoft Docs
description: Cette rubrique montre comment incorporer une vidéo de diffusion en continu adaptative MPEG-DASH dans une application HTML5 avec DASH.js.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 5aa0e7b6-f5c3-4cc1-aa33-ed16ea4780c2
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 55c22a58ea76c268c40894c0ea64d43312b1ba27
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89269091"
---
# <a name="embedding-an-mpeg-dash-adaptive-streaming-video-in-an-html5-application-with-dashjs"></a>Incorporation d’une vidéo de diffusion en continu adaptative MPEG-DASH dans une application HTML5 avec DASH.js

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

## <a name="overview"></a>Vue d’ensemble
MPEG-DASH est une norme ISO pour la diffusion en continu adaptative de contenu vidéo, qui offre des avantages significatifs pour les développeurs qui souhaitent proposer un résultat de diffusion vidéo en continu adaptative de haute qualité. Avec MPEG-DASH, le flux vidéo s’ajuste automatiquement à une définition inférieure quand le réseau est encombré. Cela réduit le risque pour un utilisateur de voir une vidéo « interrompue » pendant que le lecteur télécharge les quelques secondes suivantes à lire (c’est-à-dire qu’il réduit la probabilité d’une mise en mémoire tampon). À mesure que l’encombrement du réseau diminue, le lecteur vidéo renvoie à son tour un flux de qualité supérieure. Cette capacité d'adaptation de la bande passante requise entraîne également un temps de départ plus rapide pour la vidéo. Cela signifie que les premières secondes peuvent être lues dans un segment de moindre qualité rapide à télécharger, puis que la qualité s'améliore une fois le contenu suffisant mis en mémoire tampon.

Dash.js est un lecteur de vidéo MPEG-DASH open source écrit en JavaScript. Son objectif est de fournir un lecteur robuste, inter-plateformes qui peut être réutilisé librement dans les applications qui requièrent une lecture vidéo. Il assure la lecture MPEG-DASH dans n’importe quel navigateur prenant en charge W3C Media Source Extensions (MSE) aujourd’hui, à savoir Chrome, Microsoft Edge et IE11 (d’autres navigateurs ont indiqué leur intention de prendre en charge MSE). Pour plus d'informations sur DASH.js, consultez le référentiel dash.js GitHub.

## <a name="creating-a-browser-based-streaming-video-player"></a>Création d'un lecteur vidéo de diffusion en continu basé sur le navigateur
Pour créer une page web simple qui affiche un lecteur vidéo avec les contrôles courants comme Lecture, Pause, Retour rapide, etc., vous devez effectuer les tâches suivantes :

1. Créer une page HTML
2. Ajouter la balise vidéo
3. Ajouter le lecteur dash.js
4. Initialiser le lecteur
5. Ajouter un style CSS
6. Afficher les résultats dans un navigateur qui implémente MSE

L'initialisation du lecteur peut être effectuée en seulement quelques lignes de code JavaScript. À l’aide de dash.js, il est vraiment très simple d’incorporer une vidéo MPEG-DASH dans vos applications basées sur le navigateur.

## <a name="creating-the-html-page"></a>Création de la page HTML
La première étape consiste à créer une page HTML standard qui contient l’élément **video**, à enregistrer ce fichier sous basicPlayer.html, comme l’illustre l’exemple suivant :

```html
    <!DOCTYPE html>
    <html>
      <head><title>Adaptive Streaming in HTML5</title></head>
      <body>
        <h1>Adaptive Streaming with HTML5</h1>
        <video id="videoplayer" controls></video>
      </body>
    </html>
```

## <a name="adding-the-dashjs-player"></a>Ajout du lecteur DASH.js
Pour ajouter l’implémentation de référence dash.js à l’application, vous devez extraire le fichier dash.all.js de la dernière version du projet dash.js. Celui-ci doit être enregistré dans le dossier JavaScript de votre application. Ce fichier est un fichier de convenance qui rassemble tout le code dash.js requis dans un seul fichier. En examinant le contenu du dépôt dash.js, vous trouverez entre autres les fichiers individuels et le code de test, mais si vous voulez seulement utiliser dash.js, alors c’est du fichier dash.all.js dont vous avez besoin.

Pour ajouter le lecteur dash.js à vos applications, ajoutez une balise de script à la section d'en-tête de basicPlayer.html :

```html
    <!-- DASH-AVC/265 reference implementation -->
    < script src="js/dash.all.js"></script>
```

Ensuite, créez une fonction pour initialiser le lecteur pendant le chargement de la page. Ajoutez le script suivant après la ligne dans laquelle vous chargez dash.all.js :

```html
    <script>
    // setup the video element and attach it to the Dash player
    function setupVideo() {
      var url = "http://wams.edgesuite.net/media/MPTExpressionData02/BigBuckBunny_1080p24_IYUV_2ch.ism/manifest(format=mpd-time-csf)";
      var context = new Dash.di.DashContext();
      var player = new MediaPlayer(context);
                      player.startup();
                      player.attachView(document.querySelector("#videoplayer"));
                      player.attachSource(url);
    }
    </script>
```

Cette fonction crée d'abord un DashContext. Celui-ci permet de configurer l'application pour un environnement d'exécution spécifique. D'un point de vue technique, il définit les classes que l'infrastructure d'injection de dépendance doit utiliser pour construire l'application. Dans la plupart des cas, vous utilisez Dash.di.DashContext.

Ensuite, instanciez la classe principale de l'infrastructure dash.js, MediaPlayer. Cette classe contient les principales méthodes requises, telles que la lecture et la mise en pause, gère la relation avec l’élément vidéo et gère également l’interprétation du fichier MPD (Media Presentation Description) qui décrit la vidéo à lire.

La fonction startup() de la classe MediaPlayer est appelée pour s'assurer que le lecteur est prêt à lire la vidéo. Entre autres choses, cette fonction garantit que toutes les classes nécessaires (comme définies par le contexte) ont été chargées. Une fois que le lecteur est prêt, vous pouvez y associer l'élément vidéo à l'aide de la fonction attachView(). La fonction startup permet à MediaPlayer d’injecter le flux vidéo dans l’élément et également de contrôler la lecture si besoin.

Passez l’URL du fichier MPD à MediaPlayer, afin qu’il ait connaissance de la vidéo qu’il est censé lire. La fonction setupVideo() qui vient d’être créée devra être exécutée une fois la page entièrement chargée. Pour cela, utilisez l'événement onload de l'élément body. Remplacez votre élément `<body>` par :

```html
    <body onload="setupVideo()">
```

Enfin, définissez la taille de l'élément vidéo à l'aide de CSS. Dans un environnement de diffusion en continu adaptative, cela s'avère particulièrement important car la taille de la vidéo lue peut changer au gré de l'adaptation de la lecture aux conditions changeantes du réseau. Cette démonstration simple force l'élément vidéo à constituer 80 % de la fenêtre de navigateur disponible en ajoutant le fichier CSS suivant à la section head de la page :

```html
    <style>
    video {
      width: 80%;
      height: 80%;
    }
    </style>
```

## <a name="playing-a-video"></a>Lecture d'une vidéo
Pour lire une vidéo, pointez votre navigateur sur le fichier basicPlayback.html et cliquez sur Lire sur le lecteur vidéo affiché.

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>Voir aussi

[Référentiel dash.js GitHub](https://github.com/Dash-Industry-Forum/dash.js) 

