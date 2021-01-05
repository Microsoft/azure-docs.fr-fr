---
title: Problèmes connus du Lecteur multimédia Azure
description: La version actuelle présente les problèmes connus suivants.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: reference
ms.date: 05/11/2020
ms.openlocfilehash: a31c089971b7e70e70c5906480deb2b17c197b9f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87043613"
---
# <a name="known-issues"></a>Problèmes connus #

La version actuelle présente les problèmes connus suivants :

## <a name="azure-media-player"></a>Azure Media Player ##

- Des encodeurs mal configurés peuvent provoquer des problèmes de lecture
- Les flux dont les horodateurs sont supérieurs à 2^53 peuvent rencontrer des problèmes de lecture.
  - Atténuation des risques : Utiliser des échelles de temps vidéo de 90 kHz et audio de 44,1 kHz
- Aucune lecture automatique sur les appareils mobiles sans intervention de l’utilisateur. Elle est bloquée par la plateforme.
- La recherche de discontinuités proches peut entraîner un échec de lecture.
- Le téléchargement de présentations volumineuses peut entraîner le blocage de l’interface utilisateur.
- Impossible de relire automatiquement la même source après la fin de la présentation.
  - Pour relire une source une fois qu’elle a pris fin, il est nécessaire de définir à nouveau la source.
- Les manifestes vides peuvent entraîner des problèmes avec le lecteur.
  - Ce problème peut se produire lorsqu’un stream en direct démarre et que le manifeste ne dispose pas de suffisamment de blocs.
- La position de lecture peut être en dehors de la barre de recherche de l’interface utilisateur.
- L’ordre des événements n’est pas le même pour toutes les technologies.
- La propriété Mise en mémoire tampon n’est pas la même pour toutes les technologies.
- nativeControlsForTouch doit avoir la valeur false (par défaut) pour éviter l’erreur « L’objet ne prend pas en charge la propriété ou la méthode "setControls" ».
- Les affiches doivent maintenant être des URL absolues
- Des problèmes esthétiques mineurs peuvent survenir dans l’interface utilisateur lors de l’utilisation du mode de contraste élevé de l’appareil.
- Les URL contenant les symboles « % » ou « + » dans la chaîne entièrement décodée peuvent avoir des problèmes lors de la définition de la source.

## <a name="ad-insertion"></a>Insertion de publicité ##

- Des problèmes d’insertion de publicités peuvent survenir (à la demande ou en direct) lorsqu’un bloqueur de publicités est installé sur le navigateur.
- Les appareils mobiles peuvent avoir des difficultés à lire les publicités.
- Les publicités MidRoll MP4 ne sont pas actuellement prises en charge par le Lecteur multimédia Azure.

## <a name="azurehtml5js"></a>AzureHtml5JS ##

- Dans la fenêtre DVR du contenu en direct, lorsque le contenu se termine, la ligne de temps continue à s’allonger jusqu’à la sollicitation de la zone ou jusqu’à la fin de la présentation.
- Les présentations en direct dans Firefox avec MSE activé rencontrent des problèmes.

- Les ressources audio uniquement ne seront pas lues par la technologie AzureHtml5JS.
  - Si vous souhaitez lire des ressources sans audio, vous pouvez le faire en insérant un fichier audio vierge à l’aide de l’[outil Azure Media Services Explorer](https://aka.ms/amse).
  - Vous trouverez les instructions pour l’insertion d’une piste audio silencieuse [ici](../previous/media-services-advanced-encoding-with-mes.md#silent_audio).

## <a name="flash"></a>Clignote ##

- Le contenu AES n’est pas lu dans Flash version 30.0.0.134 en raison d’un bogue dans la logique de mise en cache d’Adobe. Adobe a résolu le problème et a publié le correctif dans la version 30.0.0.154.
- En cas de défaillance technique et http (comme les expirations du délai réseau 404), le lecteur mettra plus de temps à se rétablir que les autres technologies.
- Cliquez sur la zone de la vidéo avec la technologie FlashSS ne permet pas de lire/mettre en pause le lecteur.
- Si l’utilisateur a installé Flash, mais ne donne pas l’autorisation de le charger sur le site, le lecteur peut patiner à l’infini. Cela est dû au fait que le lecteur pense que le plug-in est installé et disponible et il considère que le plug-in exécute le contenu. Du code JavaScript a été envoyé, mais les paramètres du navigateur ont bloqué l’exécution du plug-in jusqu’à ce que l’utilisateur accepte l’invite pour autoriser le plug-in. Cela peut se produire dans tous les navigateurs.  

## <a name="silverlight"></a>Silverlight ##

- Fonctionnalités manquantes
- En cas de défaillance technique et http (comme les expirations du délai réseau 404), le lecteur mettra plus de temps à se rétablir que les autres technologies.
- La lecture de Safari et Firefox sur Mac avec Silverlight requiert de définir explicitement `"http://` ou `https://` pour la source.
- Si une API manque à cette technologie, elle retourne généralement la valeur Null.
- Si l’utilisateur a installé Flash, mais ne donne pas l’autorisation de le charger sur le site, le lecteur peut patiner à l’infini. Cela est dû au fait que le lecteur pense que le plug-in est installé et disponible et il considère que le plug-in exécute le contenu. Du code JavaScript a été envoyé, mais les paramètres du navigateur ont bloqué l’exécution du plug-in jusqu’à ce que l’utilisateur accepte l’invite pour autoriser le plug-in. Cela peut se produire dans tous les navigateurs.  

## <a name="native-html5"></a>HTML5 natif ##

- La technologie HTML5 envoie uniquement l’événement canplaythrough pour le premier ensemble de sources.
- Cette technologie prend uniquement en charge ce que le navigateur a implémenté.  Certaines fonctionnalités peuvent manquer dans cette technologie.  
- Si une API manque à cette technologie, elle retourne généralement la valeur Null.
- Il existe des problèmes avec les sous-titres codés et les sous-titres de dialogue sur cette technologie. Ils peuvent ou non être disponibles ou consultables sur cette technologie.
- Le fait de disposer d’une bande passante limitée dans un scénario de technologie HLS/HTML5 entraîne la lecture de l’audio sans la vidéo.

### <a name="microsoft"></a>Microsoft ###

- La lecture IE8 ne fonctionne pas actuellement en raison d’une incompatibilité avec ECMAScript 5
- Dans Internet Explorer et certaines versions de Edge, le plein écran ne peut pas être saisi en appuyant sur la touche de tabulation et en le sélectionnant ou en utilisant la touche d’accès rapide F/f.

## <a name="google"></a>Google ##

- Les profils d’encodage multiples dans le même manifeste présentent certains problèmes de lecture dans Chrome et ne sont pas recommandés.
- Chrome ne peut pas lire HE-AAC avec AzureHtml5JS. Suivez les détails sur le [traqueur de bogues](https://bugs.chromium.org/p/chromium/issues/detail?id=534301).
- À partir de Chrome v58, le contenu Widevine doit être chargé/lu via le protocole https://, sinon la lecture échoue.

## <a name="mozilla"></a>Mozilla ##

- Le commutateur de flux audio nécessite que les flux audio aient les mêmes données privées de codec lors de l’utilisation d’AzureHtml5JS. C’est ce que requiert la plateforme Firefox.

## <a name="apple"></a>Pomme ##

- Safari sur Mac active souvent le mode Économiseur d’énergie par défaut avec le paramètre « Arrêter les plug-ins pour économiser de l’énergie », ce qui bloque les plug-ins tels que Flash et Silverlight lorsqu’ils considèrent que leur utilisation n’est pas en faveur de l’utilisateur. Ce bloc ne bloque pas le plug-in en lui-même, mais seulement ses capacités. Compte tenu du techOrder par défaut, cela peut provoquer des problèmes lors de la tentative de lecture.
  - Atténuation 1 : Si le lecteur vidéo est « à l’avant et au centre » (dans une limite de 3000 x 3000 pixels en partant du coin supérieur gauche du document), il doit toujours être lu.
  - Atténuation 2 : Remplacez le techOrder pour Safari par ["azureHtml5JS", "html5"]. Cette atténuation a pour conséquence de ne pas obtenir toutes les fonctionnalités disponibles dans la technologie FlashSS.
- Le contenu PlayReady via Silverlight peut rencontrer des problèmes de relecture dans Safari.
- Le contenu AES et celui des jetons restreints ne sont pas lus par iOS et les appareils Android plus anciens.
  - Pour réaliser ce scénario, un proxy doit être ajouté à votre service.
- L’apparence par défaut du téléphone iOS est visible.
- La lecture sur iPhone se fait toujours en plein écran dans le lecteur natif.
  - Les fonctionnalités, y compris les sous-titres codés, peuvent ne pas persister dans cette lecture non incluse.
- Une fois la présentation en direct terminée, les appareils iOS ne terminent pas correctement la présentation.
- iOS n’autorise pas les capacités DVR.
- Le changement de flux audio iOS peut être effectué uniquement via l’interface utilisateur du lecteur natif iOS et nécessite que les flux audio aient les mêmes données privées de codec.
- Les versions antérieures de Safari peuvent potentiellement rencontrer des problèmes lors de la relecture de streams en direct.

## <a name="older-android"></a>Versions Android antérieures ##

- Le contenu AES et celui des jetons restreints ne sont pas lus par iOS et les appareils Android plus anciens.
  - Pour réaliser ce scénario, un proxy doit être ajouté à votre service.

## <a name="next-steps"></a>Étapes suivantes ##

- [Démarrage rapide du Lecteur multimédia Azure](azure-media-player-quickstart.md)
