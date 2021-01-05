---
title: Options cloud vers appareil Azure IoT Hub | Microsoft Docs
description: 'Guide du développeur : conseils pour l’utilisation de méthodes directes, les propriétés souhaitées des jumeaux d’appareil ou les messages cloud-à-appareil pour les communications cloud à appareil.'
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
ms.openlocfilehash: 10206aced4f38f4d157f46703aac2d28ec863274
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87319148"
---
# <a name="cloud-to-device-communications-guidance"></a>Conseils pour les communications cloud-à-appareil

IoT Hub propose trois options pour les applications pour appareil afin d’exposer les fonctionnalités sur un appareil principal :

* [Méthodes directes](iot-hub-devguide-direct-methods.md), pour les communications qui nécessitent une confirmation immédiate du résultat. Les méthodes directes sont souvent utilisées pour un contrôle interactif d’appareil, tel que la mise en marche d’un ventilateur.

* [Propriétés souhaitées du jumeau](iot-hub-devguide-device-twins.md), pour les commandes de longue durée pour placer l’appareil dans un état souhaité. Par exemple, définissez l’intervalle d’envoi de la télémétrie sur 30 minutes.

* [Messages cloud-à-appareil](iot-hub-devguide-messages-c2d.md), pour les notifications unidirectionnelles vers l’application de l’appareil.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Voici une comparaison détaillée des différentes options de communication cloud-à-appareil.

| Catégories | Méthodes directes | Propriétés souhaitées des représentations | Messages Cloud vers appareil |
| ---------- | -------------- | ------------------------- | ------------------------ |
| Scénario | Commandes qui nécessitent une confirmation immédiate, par exemple, activer un ventilateur. | Commandes de longue durée pour placer l’appareil dans l’état désiré. Par exemple, définissez l’intervalle d’envoi de la télémétrie sur 30 minutes. | Notifications unidirectionnelles à l’application pour appareil. |
| Flux de données | Bidirectionnel. L’application pour appareil peut répondre immédiatement à la méthode. Le dispositif principal reçoit les résultats en fonction du contexte de la demande. | Unidirectionnel. L’application pour appareil reçoit une notification avec la modification de propriété. | Unidirectionnel. L’application pour appareil reçoit le message
| Durabilité | Les appareils déconnectés ne sont pas contactés. Le serveur principal de la solution est averti que l’appareil n’est pas connecté. | Les valeurs de propriété sont conservées dans la représentation d’appareil. L’appareil les lira lors de la reconnexion suivante. Les valeurs de propriété sont récupérables avec le [langage de requête IoT Hub](iot-hub-devguide-query-language.md). | Les messages peuvent être conservés par IoT Hub jusqu’à 48 heures. |
| Cibles | Un seul appareil utilisant **deviceId** ou plusieurs appareils utilisant [jobs](iot-hub-devguide-jobs.md). | Un seul appareil utilisant **deviceId** ou plusieurs appareils utilisant [jobs](iot-hub-devguide-jobs.md). | Appareil unique par **deviceId**. |
| Taille | La taille maximale de charge utile de la méthode directe est de 128 Ko. | La taille maximale des propriétés souhaitées est de 32 Ko. | Messages jusqu’à 64 Ko. |
| Fréquence | Élevée. Pour plus d’informations, consultez les [limites d’IoT Hub](iot-hub-devguide-quotas-throttling.md). | Moyenne. Pour plus d’informations, consultez les [limites d’IoT Hub](iot-hub-devguide-quotas-throttling.md). | Faible. Pour plus d’informations, consultez les [limites d’IoT Hub](iot-hub-devguide-quotas-throttling.md). |
| Protocol | Disponible à l’aide du protocole MQTT ou AMQP. | Disponible à l’aide du protocole MQTT ou AMQP. | Disponible sur tous les protocoles. L’appareil doit interroger lors de l’utilisation de HTTPS. |

Découvrez comment utiliser les méthodes directes, les propriétés souhaitées et les messages cloud-à-appareil grâce aux didacticiels suivants :

* [Utiliser des méthodes directes](quickstart-control-device-node.md)
* [Utiliser des propriétés souhaitées pour configurer des appareils](tutorial-device-twins.md) 
* [Envoyer des messages cloud-à-appareil](iot-hub-node-node-c2d.md)
