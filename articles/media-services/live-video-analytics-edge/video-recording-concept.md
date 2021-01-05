---
title: Enregistrement de vidéo - Azure
description: Dans le contexte d’un système de gestion vidéo pour les caméras de surveillance, l’enregistrement vidéo fait référence au processus de capture des vidéos à partir des caméras et à l’enregistrement pour l’affichage via des applications mobiles et de navigateur. L’enregistrement vidéo peut être catégorisé comme enregistrement vidéo continu et enregistrement vidéo basé sur les événements.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: c03472f577e67d5b2705a2ebe3f53e85e0849819
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84260192"
---
# <a name="video-recording"></a>Enregistrement de vidéo

Dans le contexte d’un système de gestion vidéo pour les caméras de surveillance, l’enregistrement vidéo fait référence au processus de capture des vidéos à partir des caméras et à l’enregistrement pour l’affichage via des applications mobiles et de navigateur. L’enregistrement vidéo peut être catégorisé comme enregistrement vidéo continu et enregistrement vidéo basé sur les événements. 

## <a name="continuous-video-recording"></a>Enregistrement de vidéo continu  

L’enregistrement vidéo continu (CVR) fait référence au processus d’enregistrement vidéo en continu à partir d’une source vidéo. L’enregistrement vidéo continu garantit que toutes les vidéos sont disponibles (spécifié par la [stratégie d’enregistrement](#recording-policy)) pour analyser et/ou auditer ultérieurement.

## <a name="event-based-video-recording"></a>Enregistrement de vidéo basé sur les événements  

Un enregistrement vidéo basé sur les événements est un enregistrement vidéo qui a été déclenché par un événement. L’événement en question peut provenir du traitement du signal vidéo lui-même ou d’une source indépendante (par exemple, d’un capteur de porte). L’enregistrement vidéo basé sur les événements (EVR) peut entraîner des économies de stockage, mais nécessite des composants supplémentaires qui génèrent les événements et déclenchent l’enregistrement vidéo (conformément à une stratégie prédéfinie). En d’autres termes, l’enregistrement vidéo basé sur les événements nécessite de prendre des décisions supplémentaires concernant les événements qui doivent déclencher l’enregistrement vidéo et la stratégie associée à l’enregistrement vidéo (également appelée stratégie d’enregistrement). Voici un exemple de stratégie : enregistrer la vidéo pendant 2 minutes à partir de 30 secondes avant l’heure de l’événement. Les événements en question peuvent provenir du traitement vidéo sur la caméra. Par exemple, plusieurs caméras prennent en charge la génération d’un événement de signal de détection de mouvement quand un mouvement est détecté dans une zone d’intérêt préconfigurée dans la fenêtre d’affichage de la caméra. Les événements peuvent également être générés en traitant la vidéo sur un autre appareil qui capture la vidéo à partir de la caméra et l’analyse à l’aide de techniques de traitement d’image simples ou de modèles Machine Learning sophistiqués. 

## <a name="choosing-recording-modes"></a>Choix des modes d’enregistrement  

Le choix d’utiliser les solutions CVR ou EVR dépend des objectifs métier. Live Video Analytics sur IoT Edge fournit des fonctionnalités de plateforme pour CVR et EVR. Vous pouvez en savoir plus à ce sujet dans les articles des scénarios [CVR](continuous-video-recording-concept.md) et [EVR](event-based-video-recording-concept.md).

## <a name="recording-policy"></a>Stratégie d’enregistrement  

La stratégie d’enregistrement fait référence aux stratégies qui dictent l’heure de début/d’arrêt de l’enregistrement (dans le cas de l’EVR), la durée d’enregistrement et la suppression de l’enregistrement. Les stratégies d’enregistrement vous permettent d’équilibrer les coûts de stockage avec les besoins métier. Les stratégies d’enregistrement diffèrent entre CVR et EVR. Pour CVR, la stratégie d’enregistrement définit le nombre de jours pendant lesquels la vidéo doit être stockée (par exemple, les 7 derniers jours). Pour EVR, la stratégie d’enregistrement définit le moment où l’enregistrement doit commencer et se terminer, ainsi que la durée de la vidéo. Vous pouvez en savoir plus à ce sujet dans les articles des scénarios [CVR](continuous-video-recording-concept.md) et [EVR](event-based-video-recording-concept.md).

## <a name="next-steps"></a>Étapes suivantes

* [Détecter les mouvements et enregistrer des clips vidéo sur Azure Media Services](detect-motion-record-video-clips-media-services-quickstart.md)
* [Détecter les mouvements et enregistrer des clips vidéos sur des appareils périphériques](detect-motion-record-video-clips-edge-devices-quickstart.md)

