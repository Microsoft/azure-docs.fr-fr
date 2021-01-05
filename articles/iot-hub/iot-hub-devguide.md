---
title: Guide du développeur pour Azure IoT Hub | Microsoft Docs
description: Le guide du développeur Azure IoT Hub porte sur les points de terminaison, la sécurité, le registre des identités, la gestion des appareils, les méthodes directes, les jumeaux d’appareil, les chargements de fichiers, les tâches, le langage de requête IoT Hub et la messagerie.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/29/2018
ms.custom: mqtt
ms.openlocfilehash: 03e05eb2bc56a07c2bda3d3fb224012abb3c9e4a
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92735938"
---
# <a name="azure-iot-hub-developer-guide"></a>Guide du développeur Azure IoT Hub

Azure IoT Hub est un service entièrement géré qui permet d’autoriser des communications bidirectionnelles fiables et sécurisées entre des millions d’appareils et un serveur principal de solution.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Azure IoT Hub vous offre :

* La sécurité des communications grâce aux informations d’identification de sécurité par appareil et au contrôle d’accès

* Plusieurs options de communication appareil-à-cloud et cloud-à-appareil à très grande échelle.

* Stockage requêtable de métadonnées et d’informations d’état par appareil.

* Une connectivité des appareils aisée avec les bibliothèques d’appareils pour les langages et les plateformes les plus courants.

Ce guide du développeur IoT Hub inclut les articles suivants :

* [L’aide sur la communication appareil-à-cloud](iot-hub-devguide-d2c-guidance.md) vous permet de choisir entre les messages appareil-à-cloud, les propriétés signalées des jumeaux d’appareil et le chargement de fichiers.

* [L’aide sur la communication cloud-à-appareil](iot-hub-devguide-c2d-guidance.md) vous permet de choisir entre les méthodes directes, les propriétés souhaitées des jumeaux d’appareil et les messages cloud-à-appareil.

* L’article [Messagerie d’appareil-à-cloud et de cloud-à-appareil avec IoT Hub](iot-hub-devguide-messaging.md) décrit les fonctionnalités de messages (appareil-à-cloud et cloud-à-appareil) qui sont exposées par IoT Hub.

  * [Envoyer des messages appareil-à-cloud sur IoT Hub](iot-hub-devguide-messages-d2c.md).

  * [Lire des messages appareil-à-cloud à partir du point de terminaison intégré](iot-hub-devguide-messages-read-builtin.md).

  * [Utiliser des points de terminaison et des règles de routage personnalisés pour les messages appareil-à-cloud](iot-hub-devguide-messages-read-custom.md).

  * [Envoyer des messages cloud-à-appareil à partir d’IoT Hub](iot-hub-devguide-messages-c2d.md).

  * [Créer et lire des messages IoT Hub](iot-hub-devguide-messages-construct.md).

* L’article [Charger des fichiers à partir d’un appareil](iot-hub-devguide-file-upload.md) décrit comment vous pouvez charger des fichiers à partir d’un appareil. L’article comprend également des informations sur des sujets tels que les notifications qui peuvent être envoyées par le processus de chargement.

* L’article [Gérer les identités des appareils dans IoT Hub](iot-hub-devguide-identity-registry.md) décrit les informations stockées par le registre des identités de chaque hub IoT. Il explique aussi comment y accéder et le modifier.

* L’article [Contrôler l’accès à IoT Hub](iot-hub-devguide-security.md) décrit le modèle de sécurité utilisé pour autoriser l’accès à la fonctionnalité IoT Hub pour les appareils et les composants du cloud. L’article inclut des informations sur l’utilisation des jetons et des certificats X.509, ainsi que des détails sur les autorisations que vous pouvez accorder.

* L’article [Utiliser des jumeaux d’appareil pour synchroniser l’état et la configuration](iot-hub-devguide-device-twins.md) décrit le concept de *jumeau d’appareil* . L’article décrit également les fonctionnalités exposées par les jumeaux d’appareil, telles que la synchronisation d’un appareil avec son jumeau. L’article inclut des informations sur les données stockées dans un jumeau d’appareil.

* L’article [Appeler une méthode directe sur un appareil](iot-hub-devguide-direct-methods.md) décrit le cycle de vie d’une méthode directe. L’article décrit comment appeler des méthodes sur un appareil à partir de votre application backend, et comment gérer la méthode directe sur votre appareil.

* L’article [Planifier des travaux sur plusieurs appareils](iot-hub-devguide-jobs.md) décrit comment vous pouvez planifier des travaux sur plusieurs appareils. Cet article décrit comment envoyer des travaux qui effectuent des tâches telles que l’exécution d’une méthode directe ou la mise à jour d’un appareil à l’aide d’un jumeau d’appareil. Il décrit également comment interroger l’état d’un travail.

* L’article [Référence - Choisir un protocole de communication](iot-hub-devguide-protocols.md) décrit les protocoles de communication qu’IoT Hub prend en charge pour la communication des appareils et répertorie les ports qui doivent être ouverts.

* L’article [Référence - Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md) décrit les différents points de terminaison que chaque IoT Hub expose pour les opérations d’exécution et de gestion. Cet article explique également comment créer des points de terminaison supplémentaires dans IoT Hub et comment utiliser une passerelle de champ pour activer la connectivité avec les points de terminaison IoT Hub dans des scénarios non standard.

* L’article [Référence - Langage de requête IoT Hub pour les jumeaux d’appareil, les travaux et le routage des messages](iot-hub-devguide-query-language.md) décrit le langage de requête d’IoT Hub permettant de récupérer à partir de votre hub des informations sur vos jumeaux d’appareil et vos travaux.

* L’article [Référence - Quotas et limitation](iot-hub-devguide-quotas-throttling.md) récapitule les quotas définis dans le service IoT Hub et la limitation appliquée lorsque vous dépassez un quota.

* L’article [Référence - Tarification](iot-hub-devguide-pricing.md) fournit des informations générales sur les différentes références SKU et les tarifs d’IoT Hub, ainsi que des informations sur la façon dont les différentes fonctionnalités d’IoT Hub sont mesurées en tant que messages par IoT Hub.

* L’article [Référence - Kits de développement logiciel (SDK) de services et d’appareils](iot-hub-devguide-sdks.md) répertorie les Kits de développement logiciel (SDK) Azure IoT que vous pouvez utiliser lors du développement d’appareils et d’applications de service qui interagissent avec IoT Hub. L’article inclut des liens vers la documentation en ligne sur les API.

* L’article [Référence - Prise en charge de MQTT au niveau d’IoT Hub](iot-hub-mqtt-support.md) fournit des informations détaillées sur la prise en charge du protocole MQTT par IoT Hub. L’article décrit la prise en charge du protocole MQTT intégré dans les Kits de développement logiciel (SDK) Azure IoT et fournit des informations sur l’utilisation directe du protocole MQTT.

* [Glossaire](iot-hub-devguide-glossary.md) contient une liste des termes courants relatifs à IoT Hub.
