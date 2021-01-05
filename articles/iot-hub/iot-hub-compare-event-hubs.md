---
title: Comparaison entre Azure IoT Hub et Azure Event Hub | Microsoft Docs
description: Comparaison des services Azure IoT Hub et Azure Event Hubs mettant en avant les différences de fonction et des exemples d’utilisation. La comparaison inclut les protocoles pris en charge, la surveillance, la gestion des appareils et les téléchargements de fichiers.
author: kgremban
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 02/20/2019
ms.author: kgremban
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: System Architecture'
ms.openlocfilehash: 56bb179c50862f09f1b789c359db97976017bbb5
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92142766"
---
# <a name="connecting-iot-devices-to-azure-iot-hub-and-event-hubs"></a>Connexion des appareils IoT à Azure : IoT Hub et Event Hubs

Azure propose des services spécialement développés pour divers types de connectivité et de communication pour vous aider à connecter vos données à la puissance du cloud. Azure IoT Hub et Azure Event Hubs sont des services cloud qui peuvent ingérer de grandes quantités de données et des processus ou stocker ces données pour des perspectives. Les deux services sont similaires, car ils prennent tous deux en charge l’ingestion des données avec une latence faible et une haute fiabilité, mais ils sont conçus à des fins différentes. IoT Hub a été développé pour répondre aux besoins uniques de connexion des appareils IoT au cloud Azure, tandis que le service Event Hubs a été conçu pour la diffusion en continu du Big Data. Microsoft recommande d'utiliser Azure IoT Hub pour connecter des appareils IoT à Azure.

Azure IoT Hub est la passerelle cloud qui connecte les appareils IoT afin de collecter des données et de stimuler perspectives et automatisation. En outre, IoT Hub inclut des fonctionnalités qui enrichissent la relation entre vos appareils et vos systèmes principaux. Grâce aux fonctionnalités de communication bidirectionnelle, vous pouvez renvoyer des commandes et des stratégies aux appareils tout en recevant des données de ceux-ci. Par exemple, utilisez la messagerie cloud-à-appareil pour mettre à jour les propriétés ou appeler des actions de gestion des périphériques. La communication cloud-à-appareil vous permet également de fournir une intelligence cloud à vos appareils de périmètre avec Azure IoT Edge. L’identité unique au niveau des appareils du service IoT Hub permet de mieux sécuriser votre solution IoT contre des attaques potentielles. 

[Azure Event Hubs](../event-hubs/event-hubs-about.md) est le service de diffusion en continu du Big Data d’Azure. Il est conçu pour les scénarios de diffusion en continu de données haut débit dans lesquels les clients peuvent envoyer des milliards de requêtes quotidiennes. Le service Event Hubs utilise un modèle de consommateur partitionné pour effectuer un scale-out de votre flux de données. Il est intégré au service Big Data et analytique d’Azure comprenant Databricks, Stream Analytics, ADLS et HDInsight. Avec des fonctionnalités comme Event Hubs Capture et Majoration automatique, ce service est conçu pour prendre en charge vos solutions et applications du Big Data. En outre, IoT Hub utilise le chemin d'accès au flux de données de télémétrie d'Event Hubs. Votre solution IoT bénéficie également de la puissance considérable d'Event Hubs.

Pour résumer, les deux solutions sont conçues pour l'ingestion de données à très grande échelle. Seul le service IoT Hub fournit des fonctionnalités enrichies propres à l'IoT conçues pour vous permettre d'optimiser la valeur commerciale de la connexion de vos appareils IoT au cloud Azure.  Si vous en êtes au tout début de votre migration vers l’IoT, commencer par IoT Hub pour prendre en charge vos scénarios d’ingestion des données vous permettra de disposer d’un accès instantané aux fonctionnalités IoT complètes pour satisfaire à vos besoins commerciaux et techniques.

La table suivante fournit des détails sur le mode de comparaison entre les deux niveaux de IoT Hub et Event Hubs lorsque vous procédez à une évaluation par rapport aux capacités IoT. Pour plus d’informations sur les niveaux de base et standard d’IoT Hub, consultez l’article [Choisir le niveau IoT Hub correspondant à votre solution](iot-hub-scaling.md).

| Fonctionnalité IoT | Niveau standard IoT Hub | Niveau de base IoT Hub | Event Hubs |
| --- | --- | --- | --- |
| Messages d’appareil-à-cloud | ![Vérification][checkmark] | ![Vérification][checkmark] | ![Vérification][checkmark] |
| Protocols: HTTPS, AMQP, AMQP sur WebSockets | ![Vérification][checkmark] | ![Vérification][checkmark] | ![Vérification][checkmark] |
| Protocols: MQTT, MQTT sur WebSockets | ![Vérification][checkmark] | ![Vérification][checkmark] |  |
| Identité par appareil | ![Vérification][checkmark] | ![Vérification][checkmark] |  |
| Fichier téléchargé à partir d’appareils | ![Vérification][checkmark] | ![Vérification][checkmark] |  |
| Service Device Provisioning | ![Vérification][checkmark] | ![Vérification][checkmark] |  |
| Messages de cloud-à-appareil | ![Vérification][checkmark] |  |  |
| Gestion des jumeaux d'appareils et des appareils | ![Vérification][checkmark] |  |  |
| Device Streams (préversion) | ![Vérification][checkmark] |  |  |
| IoT Edge | ![Vérification][checkmark] |  |  |

Même si le seul cas d’usage est l’ingestion de données appareil-à-cloud, nous recommandons fortement l’utilisation d’IoT Hub car il fournit un service conçu pour la connectivité des appareils IoT. 

### <a name="next-steps"></a>Étapes suivantes

Pour explorer davantage les capacités d’IoT Hub, consultez le [Guide du développeur IoT Hub](iot-hub-devguide.md).

<!-- This one reference link is used over and over. --robinsh -->
[checkmark]: ./media/iot-hub-compare-event-hubs/ic195031.png