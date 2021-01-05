---
title: Options appareil-à-cloud d’Azure IoT Hub | Microsoft Docs
description: Guide du développeur - Recommandations sur les périodes d’utilisation des messages appareil-à-cloud, des propriétés signalées et du chargement des fichiers pour les communications cloud-à-appareil.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/29/2018
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: dd4f4ad7f84ebc1f5e254843e7afa4aa0f3f224f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87319131"
---
# <a name="device-to-cloud-communications-guidance"></a>Recommandations sur les communications appareil-à-cloud

Lors de l’envoi d’informations de l’application d’appareil au serveur principal de solution, IoT Hub expose trois options :

* [Messages appareil-à-cloud](iot-hub-devguide-messages-d2c.md) pour la télémétrie et les alertes de série chronologique.

* [Propriétés signalées du jumeau d’appareil](iot-hub-devguide-device-twins.md) pour la consignation des informations sur l’état de l’appareil, telles que les fonctionnalités disponibles, les conditions ou l’état des flux de travail de longue durée. Par exemple, les mises à jour de configuration et de logiciels.

* [Chargement des fichiers](iot-hub-devguide-file-upload.md) pour les fichiers multimédias et les lots volumineux de données télémétriques chargés par des appareils connectés par intermittence ou compressés pour économiser de la bande passante.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Voici une comparaison détaillée des différentes options de communication appareil-à-cloud.

| Factor | Messages appareil-à-cloud | Propriétés signalées du jumeau d’appareil | Chargements de fichiers |
| ---- | ------- | ---------- | ---- |
| Scénario | Télémétrie et alertes de série chronologique. Par exemple, les lots de données de capteur de 256 Ko envoyés toutes les 5 minutes. | Capacités et conditions disponibles. Par exemple, le mode de connectivité d’appareil actuel, tel que réseau mobile ou Wi-Fi. Synchronisation des workflows de longue durée, comme les mises à jour logicielles et de la configuration. | Fichiers multimédias. Lots de télémétrie volumineux (généralement compressés). |
| Stockage et récupération | Stockage temporaire par IoT Hub, jusqu’à 7 jours. Lecture uniquement séquentielle. | Stockage par IoT Hub dans le jumeau d’appareil. Récupérables à l’aide du [langage de requête IoT Hub](iot-hub-devguide-query-language.md). | Stockées dans le compte de stockage Azure fourni par l’utilisateur. |
| Taille | Messages d’une taille maximale de 256 Ko. | La taille maximale des propriétés signalées est de 32 Ko. | Taille maximale de fichier prise en charge par le stockage Blob Azure. |
| Fréquence | Élevée. Pour plus d’informations, consultez les [limites d’IoT Hub](iot-hub-devguide-quotas-throttling.md). | Moyenne. Pour plus d’informations, consultez les [limites d’IoT Hub](iot-hub-devguide-quotas-throttling.md). | Faible. Pour plus d’informations, consultez les [limites d’IoT Hub](iot-hub-devguide-quotas-throttling.md). |
| Protocol | Disponible sur tous les protocoles. | Disponible à l’aide du protocole MQTT ou AMQP. | Disponible avec tous les protocoles, mais nécessite HTTPS sur l’appareil. |

Une application peut avoir besoin d’envoyer des informations en tant que série chronologique de télémétrie ou d’alerte, et de les mettre à disposition sur le jumeau d’appareil. Dans ce scénario, vous pouvez choisir l’une des options suivantes :

* L’application pour appareil envoie un message appareil-à-cloud et signale une modification de propriété.
* Le serveur principal de la solution peut stocker les informations dans les étiquettes du jumeau d’appareil quand il reçoit le message.

Dans la mesure où les messages appareil-à-cloud prennent en charge un débit bien plus important que les mises à jour de jumeaux d’appareil, il est parfois souhaitable d’éviter de mettre à jour le jumeau d’appareil lors de chaque message appareil-à-cloud.
