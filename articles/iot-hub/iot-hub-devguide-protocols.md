---
title: Ports et protocoles de communication IoT Hub | Microsoft Docs
description: 'Guide du développeur : décrit les protocoles de communication pris en charge pour les communications appareil-à-cloud et cloud-à-appareil et les numéros de port qui doivent être ouverts.'
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/29/2018
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 1792535fab79ed20bdf77f96b4fc39f13b0c7bbb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90015996"
---
# <a name="reference---choose-a-communication-protocol"></a>Référence - Choisir un protocole de communication

IoT Hub permet aux appareils d’utiliser les protocoles suivants pour les communications côté appareil :

* [MQTT](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.pdf)
* MQTT sur WebSockets
* [AMQP](https://docs.oasis-open.org/amqp/core/v1.0/os/amqp-core-complete-v1.0-os.pdf)
* AMQP sur WebSockets
* HTTPS

Pour plus d’informations sur la façon dont ces protocoles prennent en charge les fonctionnalités spécifiques d’IoT Hub, consultez [Recommandations sur les communications appareil-à-cloud](iot-hub-devguide-d2c-guidance.md) et [Conseils pour les communications cloud-à-appareil](iot-hub-devguide-c2d-guidance.md).

Le tableau suivant fournit des recommandations de haut niveau pour votre choix de protocole :

| Protocol | Quand choisir ce protocole |
| --- | --- |
| MQTT <br> MQTT sur WebSocket |Utilisez sur tous les appareils ne nécessitant pas de connecter plusieurs appareils (chacun avec ses propres informations d’identification par appareil) à l’aide de la même connexion TLS. |
| AMQP <br> AMQP sur WebSocket |Utiliser sur les passerelles de champ et de cloud pour tirer parti du multiplexage de connexion sur les appareils. |
| HTTPS |Utiliser pour les appareils qui ne peuvent pas prendre en charge les autres protocoles. |

Prenez en compte les points suivants lorsque vous choisissez votre protocole pour les communications côté appareil :

* **Modèle Cloud vers appareil**. HTTPS ne dispose pas d’un moyen efficace de mettre en œuvre la transmission des messages par le serveur. Ainsi, quand vous utilisez HTTPS, les appareils interrogent IoT Hub pour rechercher les messages cloud-à-appareil. Cette approche est inefficace pour l’appareil et pour IoT Hub. Conformément aux recommandations actuelles concernant HTTP,S chaque appareil doit interroger la présence de messages toutes les 25 minutes ou plus. AMQP et MQTT prennent en charge les notifications Push sur le serveur lors de la réception de messages cloud-à-appareil. Ils permettent d’obtenir des notifications Push immédiates pour les messages IoT Hub-à-appareil. Si la latence de remise pose problème, MQTT ou AMQP sont les meilleurs protocoles à utiliser. Pour les appareils rarement connectés, HTTPS fonctionne aussi bien.

* **Passerelles de champ**. MQTT et HTTPS prennent uniquement en charge une identité d’appareil unique (ID d’appareil et informations d’identification) par connexion TLS. Pour cette raison, ces protocoles ne sont pas pris en charge pour les [scénarios de passerelle de champ](iot-hub-devguide-endpoints.md#field-gateways) qui nécessitent le multiplexage de messages à l’aide de plusieurs identités d’appareil sur une seule connexion ou un pool de connexions en amont d’IoT Hub. Ces passerelles peuvent utiliser un protocole qui prend en charge plusieurs identités d’appareil par connexion, comme AMQP, pour leur trafic en amont.

* **Appareils faibles en ressources**. Les bibliothèques MQTT et HTTPS sont moins encombrantes que les bibliothèques AMQP. Par conséquent, si l’appareil dispose de ressources limitées (par exemple, moins de 1 Mo de mémoire RAM), ces protocoles sont peut-être les seuls protocoles d’implémentation disponibles.

* **Traversée réseau**. Le protocole AMQP standard utilise le port 5671 tandis que MQTT écoute sur le port 8883. L’utilisation de ces ports peut entraîner des problèmes dans les réseaux fermés aux protocoles autres que HTTPS. Utilisez MQTT sur WebSockets, AMQP sur WebSockets ou HTTPS dans ce scénario.

* **Taille de charge utile**. MQTT et AMQP sont des protocoles binaires qui génèrent des charges utiles plus compactes que HTTPS.

> [!WARNING]
> Quand vous utilisez HTTPS, chaque appareil doit interroger les messages cloud-à-appareil pas plus d’une fois toutes les 25 minutes. Pendant le développement, chaque appareil peut les interroger plus fréquemment, le cas échéant.

[!INCLUDE [iot-hub-include-x509-ca-signed-support-note](../../includes/iot-hub-include-x509-ca-signed-support-note.md)]

## <a name="port-numbers"></a>Numéros de ports

Les appareils peuvent communiquer avec IoT Hub dans Azure à l’aide de divers protocoles. En règle générale, le choix du protocole dépend des exigences spécifiques de la solution. Le tableau suivant répertorie les ports de sortie qui doivent être ouverts pour qu’un appareil puisse utiliser un protocole spécifique :

| Protocol | Port |
| --- | --- |
| MQTT |8883 |
| MQTT sur WebSockets |443 |
| AMQP |5671 |
| AMQP sur WebSockets |443 |
| HTTPS |443 |

Lorsque vous avez créé un hub IoT dans une région Azure, ce hub conserve la même adresse IP pendant sa durée de vie. Toutefois, si Microsoft le fait passer à une autre unité d’échelle pour maintenir la qualité du service, il reçoit une nouvelle adresse IP.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la façon dont IoT Hub implémente le protocole MQTT, consultez [Communication avec votre IoT Hub à l’aide du protocole MQTT](iot-hub-mqtt-support.md).
