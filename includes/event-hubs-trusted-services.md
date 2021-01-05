---
title: Fichier include
description: Fichier include
services: event-hubs
author: spelluru
ms.service: event-hubs
ms.topic: include
ms.date: 11/09/2020
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: d828a0c3eb2582a833ee8ad07bdf4f18002c9dca
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015573"
---
## <a name="trusted-microsoft-services"></a>Services Microsoft approuvés
Lorsque vous activez le paramètre **Autoriser les services Microsoft approuvés pour contourner ce pare-feu**, les services suivants sont autorisés à accéder à vos ressources Event Hubs.

| Service approuvé | Scénarios d’utilisation pris en charge | 
| --------------- | ------------------------- | 
| Azure Event Grid | Autorise Azure Event Grid à envoyer des événements à Event Hubs dans votre espace de noms Event Hubs. Vous devez également procéder comme suit : <ul><li>Activer une identité attribuée par le système pour un sujet ou un domaine</li><li>Ajouter l’identité au rôle d’expéditeur de données Azure Event Hubs sur l’espace de noms Event Hubs</li><li>Ensuite, configurez l’abonnement aux événements qui utilise un hub d’événements comme point de terminaison pour utiliser l’identité affectée au système.</li></ul> <p>Pour plus d’informations, consultez [Remise d’événement avec une identité managée](../articles/event-grid/managed-service-identity.md)</p>|
| Azure Monitor (paramètres de diagnostic) | Autorise Azure Monitor à envoyer des informations de diagnostic à Event Hubs dans votre espace de noms Event Hubs. |
| Azure Stream Analytics | Permet à un travail Azure Stream Analytics de lire des données ([entrée](../articles/stream-analytics/stream-analytics-add-inputs.md)) ou d’écrire des données dans des hubs d’événements ([sortie](../articles/stream-analytics/event-hubs-output.md)) dans votre espace de noms Event Hubs. |
| Azure IoT Hub | Autorise IoT Hub à envoyer des messages aux hubs d’événements dans votre espace de noms Event Hub. Vous devez également procéder comme suit : <ul><li>Activer une identité attribuée par le système pour le hub IoT.</li><li>Ajouter l’identité au rôle d’expéditeur de données Azure Event Hubs sur l’espace de noms Event Hubs.</li><li>Ensuite, configurer l’IoT Hub qui utilise un Event Hub comme point de terminaison personnalisé pour utiliser l’authentification basée sur l’identité.</li></ul>
