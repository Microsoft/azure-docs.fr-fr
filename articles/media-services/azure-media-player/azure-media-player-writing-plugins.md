---
title: Écriture de plug-ins pour le Lecteur multimédia Azure
description: Découvrez comment écrire un plug-in avec le Lecteur multimédia Azure à l'aide de JavaScript
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: how-to
ms.date: 04/20/2020
ms.custom: devx-track-js
ms.openlocfilehash: c1a64bff8b81735d9c4c9a14d2c1e12bd0bfe57e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91296159"
---
# <a name="writing-plugins-for-azure-media-player"></a>Écriture de plug-ins pour le Lecteur multimédia Azure #

Un plug-in est écrit en JavaScript pour étendre ou améliorer le lecteur. Vous pouvez écrire des plug-ins qui modifient l’apparence du Lecteur multimédia Azure, sa fonctionnalité, voire son interface avec d’autres services. Pour ce faire, suivez ces deux étapes simples :

## <a name="step-1"></a>Étape 1 ##

Écrivez votre code JavaScript dans une fonction comme suit :

```javascript

    (function () {
        amp.plugin('yourPluginName', function (options) {
        var myPlayer = this;
           myPlayer.addEventListener(amp.eventName.ready, function () {
        console.log("player is ready!");
            });
        });
    }).call(this);
```

Vous pouvez écrire votre code directement dans votre page HTML au sein de balises `<script>` ou dans un fichier JavaScript externe. Si vous optez pour cette dernière solution, veillez à inclure le fichier JavaScript dans l'élément `<head>` de votre page HTML *après* le script AMP.

Exemple :

```javascript
    <!--*****START OF Azure Media Player Scripts*****-->
    <script src="//amp.azure.net/libs/amp/latest/azuremediaplayer.min.js"></script>
    <link href="//amp.azure.net/libs/amp/latest/skins/amp-default/azuremediaplayer.min.css" rel="stylesheet">
    <!--*****END OF Azure Media Player Scripts*****-->
    <!--Add Plugins-->
    <script src="yourPluginName.js"></script>
```

## <a name="step-2"></a>Étape 2 ##
Initialisez le plug-in avec JavaScript de l’une des deux manières suivantes :

Méthode 1 :

```javascript
    var myOptions = {
        autoplay: true,
        controls: true,
        width: "640",
        height: "400",
        poster: "",
        plugins: {
            yourPluginName: {
                [your plugin options]: [example options]
           }
        }
    };     
    var myPlayer = amp([videotag id], myOptions);
```

Méthode 2 :

```javascript
    var video = amp([videotag id]);
    video.yourPluginName({[your plugins option]: [example option]});
```

Les options de plug-in ne sont pas requises, car elles permettent simplement aux développeurs qui utilisent votre plug-in de configurer son comportement sans devoir modifier le code source.

Pour plus d'exemples de création de plug-ins, consultez notre [galerie](azure-media-player-plugin-gallery.md).

>[!NOTE]
> Le code du plug-in modifie dynamiquement les éléments du DOM pendant la durée de vie de l’expérience de l'utilisateur du lecteur, il n’apporte aucune modification permanente au code source du lecteur. Dans un tel contexte, bien comprendre les outils développement de votre navigateur s’avère particulièrement utile. Par exemple, si vous souhaitez modifier l’apparence d’un élément du lecteur, vous pouvez trouver son élément HTML selon son nom de classe, puis ajouter ou modifier des attributs à partir de là. Voici une importante ressource relative à la [modification des attributs HTML](http://www.w3schools.com/js/js_htmldom_html.asp).

### <a name="integrated-plugins"></a>Plug-ins intégrés ###

 Il existe actuellement deux plug-ins intégrés à AMP : le [time-tip](http://sr-test.azurewebsites.net/Tests/Plugin%20Gallery/plugins/timetip/example.html) et les [touches d'accès rapide](http://sr-test.azurewebsites.net/Tests/Plugin%20Gallery/plugins/hotkeys/example.html). À l'origine, ces plug-ins ont été développés en tant que plug-ins modulaires pour le lecteur, mais ils sont désormais inclus dans son code source.

### <a name="plugin-gallery"></a>Galerie de plug-ins ###

La [galerie de plug-ins](https://aka.ms/ampplugins) contient plusieurs plug-ins auxquels la communauté a contribué en termes de fonctionnalités (marqueurs de temps, zoom, analyse, par exemple). La page fournit des accès aux plug-ins et des instructions sur la façon de les configurer, ainsi qu’une démonstration montrant les plug-ins en action. Si vous pensez que votre plug-in mérite d'être inclus dans notre galerie, n’hésitez pas à nous l'envoyer pour nous permettre de le vérifier.

## <a name="next-steps"></a>Étapes suivantes ##

- [Démarrage rapide du Lecteur multimédia Azure](azure-media-player-quickstart.md)
