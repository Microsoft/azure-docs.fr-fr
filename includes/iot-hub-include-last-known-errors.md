---
title: Fichier include
description: Fichier include
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/22/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 54f4835a904b897370cf9f075ae3c005b1114992
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95555398"
---
[Obtenir l’intégrité du point de terminaison](/rest/api/iothub/iothubresource/getendpointhealth#iothubresource_getendpointhealth) dans l’API REST donne l’état d’intégrité des points de terminaison, ainsi que la dernière erreur connue, afin d’identifier la raison pour laquelle un point de terminaison n’est pas sain. Le tableau ci-dessous répertorie les erreurs les plus courantes.

|Dernière erreur connue|Description/lorsqu’elle se produit|Atténuation possible|
|-----|-----|-----|
|Temporaire|Une erreur temporaire s’est produite et l’IoT Hub retentera l’opération.|Observez les [journaux de ressources des routes](../articles/iot-hub/monitor-iot-hub-reference.md#routes).|
|InternalError|Une erreur s’est produite lors de la transmission d’un message à un point de terminaison.|Il s’agit d’une exception interne, mais observez également les [journaux de ressources](../articles/iot-hub/monitor-iot-hub-reference.md#routes) des itinéraires.|
|Non autorisé|L’IoT Hub n’est pas autorisé à envoyer des messages au point de terminaison spécifié.|Vérifiez que la chaîne de connexion est à jour pour le point de terminaison. Si une modification a été apportée, envisagez une mise à jour sur votre IoT Hub. Si le point de terminaison utilise l’identité managée, vérifiez que l’IoT Hub principal dispose des autorisations requises sur la cible.|
|Throttled|L’IoT Hub est limité lors de l’écriture de messages dans le point de terminaison.|Passez en revue les seuils de limitation pour le point de terminaison affecté. Modifiez les configurations pour que le point de terminaison soit mis à l’échelle si nécessaire.|
|Délai d'expiration|Délai d’expiration de l’opération.|Retentez l’opération.|
|Introuvable|La ressource cible n’existe pas.|Assurez-vous que la ressource cible existe.|
|Conteneur introuvable|Le conteneur de stockage n’existe pas.|Assurez-vous que le conteneur de stockage existe.|
|Conteneur désactivé|Le conteneur de stockage est désactivé.|Assurez-vous que le conteneur de stockage est activé.|
|MaxMessageSizeExceeded|Le routage des messages a une limite de taille de message de 256 Ko. La taille du message en cours de routage a dépassé cette limite.|Vérifiez si la taille des messages peut être réduite en utilisant moins de propriétés de l’application ou moins d’enrichissements de messages.|
|PartitioningAndDuplicateDetectionNotSupported|La détection des doublons n’est peut-être pas activée pour service bus.|Désactivez la détection des doublons de Service Bus ou utilisez une entité sans détection des doublons.|
|SessionfulEntityNotSupported|Les sessions ne sont peut-être pas activées pour service bus.|Désactivez la session de Service Bus ou utilisez une entité sans sessions.|
|NoMatchingSubscriptionsForMessage|Il n’existe aucun abonnement pour écrire des messages dans la rubrique service bus.|Créez un abonnement pour que les messages de l’IoT Hub y soient acheminés.|
|EndpointExternallyDisabled|Le point de terminaison n’est pas dans un état actif, l’IoT Hub peut donc lui envoyer des messages.|Activez le point de terminaison pour rétablir l’état actif.|
|DeviceMaximumQueueDepthExceeded|La taille maximale de service bus a été atteinte.|Envisagez de supprimer des messages de la cible Event Hubs pour permettre l’ingestion de nouveaux messages dans les Event Hubs.|