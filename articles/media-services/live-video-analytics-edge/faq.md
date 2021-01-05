---
title: FAQ Live Video Analytics sur IoT Edge – Azure
description: Cette rubrique fournit des réponses pour les FAQ concernant Live Video Analytics sur IoT Edge.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 705c74b1ba6d3a28911185b82a558099e87dc18d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87011513"
---
# <a name="frequently-asked-questions-faqs"></a>Forum Aux Questions (FAQ)

Cette rubrique fournit des réponses pour les FAQ concernant Live Video Analytics sur IoT Edge.

## <a name="general"></a>Général

Quelles variables système peuvent être utilisées dans une définition de topologie de graphique ?

|Variable   |Description|
|---|---|
|[System.DateTime](/dotnet/framework/data/adonet/sql/linq/system-datetime-methods)|Représente un instant, généralement exprimé sous la forme d’une date ou d’une heure.|
|System.GraphTopologyName   |Représente une topologie de graphique multimédia, et contient le blueprint d’un graphique.|
|System.GraphInstanceName|  Représente une instance de graphique multimédia ; contient des valeurs de paramètre et référence la topologie.|

## <a name="configuration-and-deployment"></a>Configuration et déploiement

Puis-je déployer le module multimédia Edge sur un appareil Windows 10 ?
    * Oui. Consultez l’article [Conteneurs Linux sur Windows 10](/virtualization/windowscontainers/deploy-containers/linux-containers).

## <a name="capture-from-ip-camera-and-rtsp-settings"></a>Capture des paramètres de caméra IP et RTSP

* Dois-je utiliser un Kit de développement logiciel (SDK) spécial sur mon appareil pour envoyer un flux vidéo ?
    * Non. Live Video Analytics sur IoT Edge prend en charge la capture multimédia à l’aide du protocole RTSP de diffusion vidéo en continu (qui est géré par la plupart des caméras IP).
* Puis-je envoyer des données multimédias à Live Video Analytics sur IoT Edge à l’aide de RTMP ou Smooth (comme un événement en direct Media Services) ?
    * Non. Live Video Analytics prend uniquement en charge RTSP pour la capture de vidéos à partir de caméras IP.
    * Toute caméra qui prend en charge le streaming RTSP sur TCP/HTTP doit être compatible. 
* Puis-je réinitialiser ou mettre à jour l’URL source RTSP sur une instance de graphique ?
    * Oui, quand l’instance de graphique est à l’état inactif.  
* Existe-t-il un simulateur RTSP pouvant être utilisé pendant les tests et le développement ?
    * Oui. Il existe un module de périphérie de [simulateur RTSP](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555), qui peut être utilisé dans les didacticiels et les guides de démarrage rapide afin de prendre en charge le processus d’apprentissage. Ce module est fourni en guise de service Meilleur effort et peut ne pas toujours être disponible. Il est vivement recommandé de ne pas l’utiliser pendant plus de quelques heures. Nous vous recommandons d’effectuer des tests avec votre source RTSP réelle avant de planifier un déploiement de production.
* Prenez-vous en charge la découverte ONVIF des caméras IP en périphérie ?
    * Non, la découverte ONVIF des appareils en périphérie n’est pas gérée.

## <a name="streaming-and-playback"></a>Diffusion en continu et lecture

* Les ressources enregistrées dans AMS à partir de la périphérie sont-elles lues à l’aide de technologies de diffusion en continu de Media Services, comme HLS ou DASH ?
    * Oui. Les ressources enregistrées peuvent être diffusées en continu comme n’importe quelle autre ressource dans Azure Media Services. Pour pouvoir diffuser le contenu en continu, vous devez créer un point de terminaison de streaming, qui doit être en cours d’exécution. Le processus de création du localisateur de streaming standard vous donne accès à un manifeste HLS ou DASH pour la diffusion en continu vers n’importe quelle infrastructure de lecteur la prenant en charge. Pour plus d’informations sur la création de manifestes de publication HLS ou DASH, voir [L’empaquetage dynamique](../latest/dynamic-packaging-overview.md).
* Puis-je utiliser la protection de contenu standard et les fonctionnalités DRM de Media Services sur une ressource archivée ?
    * Oui. Toutes les fonctionnalités de protection de contenu, de chiffrement dynamique et DRM standard peuvent être utilisées sur les ressources enregistrées à partir d’un graphique multimédia.
* Quels lecteurs puis-je utiliser pour afficher le contenu des ressources enregistrées ?
   * Tous les lecteurs standard qui gèrent les instances Apple HTTP Live Streaming (HLS) conformes, version 3 ou 4, sont pris en charge. En outre, tout lecteur respectant la norme de lecture MPEG-DASH est également pris en charge.
    Les lecteurs recommandés pour les tests sont les suivants :

    * [Azure Media Player](../latest/use-azure-media-player.md)
    * [HLS.js](https://hls-js.netlify.app/demo/)
    * [Video.js](https://videojs.com/)
    * [Dash.js](https://github.com/Dash-Industry-Forum/dash.js/wiki)
    * [Shaka-Player](https://github.com/google/shaka-player)
    * [ExoPlayer](https://github.com/google/ExoPlayer)
    * [Apple HTTP Live Streaming natif](https://developer.apple.com/streaming/)
    * Lecteur vidéo HTML5 intégré dans Edge, Chrome ou Safari
    * Lecteurs commerciaux qui prennent en charge la lecture HLS ou DASH
* Quelles sont les limites de diffusion en continu d’une ressource de graphique multimédia ?
    * La diffusion en continu d’une ressource en direct ou enregistrée à partir d’un graphique multimédia utilise l’infrastructure à grande échelle et le point de terminaison de streaming que Media Services prend en charge dans le cadre de la diffusion en continu à la demande et en direct pour les clients OTT, de l’audiovisuel, des médias et du divertissement. Cela signifie que vous pouvez rapidement et facilement activer Azure CDN, Verizon ou Akamai pour distribuer votre contenu à un public de quelques spectateurs ou de plusieurs millions de personnes, selon votre scénario.

    Le contenu peut être proposé via Apple HTTP Live Streaming (HLS) ou MPEG-DASH.

## <a name="monitoring-and-metrics"></a>Analyse et métriques

* Puis-je analyser les graphiques multimédias sur la périphérie à l’aide de Microsoft Azure Event Grid ?
    * Non. Actuellement, le logiciel Event Grid n’est pas pris en charge.
* Puis-je utiliser Azure Monitor pour afficher l’intégrité, les métriques et les performances de mes graphiques multimédias dans le Cloud ou à la périphérie ?
    * Non.
* Existe-t-il des outils qui facilitent l’analyse du module IoT Edge de Media Services ?
    * Visual Studio Code prend en charge l’extension Azure IoT Tools, qui vous permet d’analyser facilement les points de terminaison du module LVAEdge. Vous pouvez utiliser cet outil pour commencer rapidement à analyser les « événements » du point de terminaison intégré IoT Hub et pour voir les messages d’inférence qui sont acheminés du périphérique vers le cloud. 

    En outre, vous pouvez utiliser cette extension pour modifier le jumeau du module LVAEdge, afin de redéfinir les paramètres du graphique multimédia.

Pour plus d’informations, consultez l’article [Surveillance et enregistrement](monitoring-logging.md).

## <a name="billing-and-availability"></a>Facturation et disponibilité

* Comment l’utilisation de Live Video Analytics sur IoT Edge est-elle facturée ?
    * Pour plus d’informations, consultez la [page des tarifs](https://azure.microsoft.com/pricing/details/media-services/).

## <a name="next-steps"></a>Étapes suivantes

[Démarrage rapide : Bien démarrer – Live Video Analytics sur IoT Edge](get-started-detect-motion-emit-events-quickstart.md)
