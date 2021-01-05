---
title: Présentation des points de terminaison Azure IoT Hub | Microsoft Docs
description: Guide du développeur – informations de référence sur les points de terminaison côté appareil et côté service IoT Hub.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 06/10/2019
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: System Architecture'
ms.openlocfilehash: a58e141c6232db08b125b265e3d4ad74c784ba24
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92152189"
---
# <a name="reference---iot-hub-endpoints"></a>Référence - Points de terminaison IoT Hub

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="iot-hub-names"></a>Noms IoT Hub

Vous pouvez rechercher le nom d’hôte de l’IoT Hub qui héberge vos points de terminaison dans le portail sur la page **Vue d’ensemble** de votre hub. Par défaut, le nom DNS d’un IoT Hub ressemble à : `{your iot hub name}.azure-devices.net`.

## <a name="list-of-built-in-iot-hub-endpoints"></a>Liste de points de terminaison IoT Hub intégrés

Azure IoT Hub est un service multilocataire qui propose ses fonctionnalités à différents acteurs. Le schéma qui suit illustre les différents points de terminaison proposés par IoT Hub.

![Points de terminaison IoT Hub](./media/iot-hub-devguide-endpoints/endpoints.png)

La liste ci-dessous décrit les points de terminaison :

* **Fournisseur de ressources**. Le fournisseur de ressources IoT Hub expose une interface [Azure Resource Manager](../azure-resource-manager/management/overview.md). Cette interface permet aux propriétaires d’abonnement Azure de créer et de supprimer des IoT Hubs et de mettre à jour les propriétés IoT Hub. Les propriétés des hubs IoT régissent les [stratégies de sécurité au niveau du hub](iot-hub-devguide-security.md#access-control-and-permissions), par opposition au contrôle d’accès au niveau de l’appareil, et les options fonctionnelles pour les messages cloud-à-appareil et appareil-à-cloud. Le fournisseur de ressources IoT Hub vous permet également [d’exporter les identités des appareils](iot-hub-devguide-identity-registry.md#import-and-export-device-identities).

* **Gestion d’identité de l’appareil**. Chaque IoT Hub expose un ensemble de points de terminaison HTTPS REST afin de gérer les identités des appareils (par exemple pour les opérations de création, de récupération, de mise à jour et de suppression). Les [identités des appareils](iot-hub-devguide-identity-registry.md) sont utilisées pour l’authentification et le contrôle d’accès des appareils.

* **Gestion des représentations d’appareils**. Chaque hub IoT expose un ensemble de points de terminaison REST HTTPS orientés service pour interroger et mettre à jour les [jumeaux d’appareil](iot-hub-devguide-device-twins.md) (mise à jour des étiquettes et des propriétés). 

* **Gestion des travaux**. Chaque hub IoT Hub expose un ensemble de points de terminaison REST HTTPS orientés service pour interroger et gérer les [travaux](iot-hub-devguide-jobs.md).

* **Points de terminaison des appareils**. Pour chaque appareil figurant dans le registre des identités, IoT Hub expose un ensemble de points de terminaison. Sauf indication contraire, ces points de terminaison sont exposés en utilisant les protocoles [MQTT v3.1.1](https://mqtt.org/), HTTPS 1.1 et [AMQP 1.0](https://www.amqp.org/). Les protocoles AMQP et MQTT sont également disponibles sur [WebSockets](https://tools.ietf.org/html/rfc6455) sur le port 443.

  * *Envoyer des messages appareil-à-cloud*. Un appareil utilise ce point de terminaison pour [envoyer des messages appareil-à-cloud](iot-hub-devguide-messages-d2c.md).

  * *Recevoir des messages cloud-à-appareil*. Un appareil utilise ce point de terminaison pour recevoir les [messages cloud-à-appareil](iot-hub-devguide-messages-c2d.md) qui lui sont adressés.

  * *Initier des téléchargements de fichiers*. Un appareil utilise ce point de terminaison pour recevoir un URI SAP du Stockage Azure provenant d’IoT Hub pour [charger un fichier](iot-hub-devguide-file-upload.md).

  * *Récupérer et mettre à jour les propriétés d’une représentation d’appareil*. Un appareil utilise ce point de terminaison pour accéder aux propriétés de son [jumeau d’appareil](iot-hub-devguide-device-twins.md). HTTPS n’est pas pris en charge.

  * *Recevoir des requêtes de méthodes directes*. Un appareil utilise ce point de terminaison pour écouter les requêtes des [méthodes directes](iot-hub-devguide-direct-methods.md). HTTPS n’est pas pris en charge.

  [!INCLUDE [iot-hub-include-x509-ca-signed-support-note](../../includes/iot-hub-include-x509-ca-signed-support-note.md)]

* **Points de terminaison de service**. Chaque IoT Hub expose un ensemble de points de terminaison pour que votre système principal de solution puisse communiquer avec vos appareils. À une exception près, ces points de terminaison sont uniquement exposés au moyen des protocoles [AMQP](https://www.amqp.org/) et AMQP sur WebSockets. Le point de terminaison d’appel de méthode directe est exposé via le protocole HTTPS.
  
  * *Recevoir des messages Appareil vers cloud*. Ce point de terminaison est compatible avec [Azure Event Hubs](https://azure.microsoft.com/documentation/services/event-hubs/). Il peut être utilisé par un backend pour lire les [messages appareil-à-cloud](iot-hub-devguide-messages-d2c.md) envoyés par vos appareils. Vous pouvez créer des points de terminaison sur votre IoT Hub en plus de ce point de terminaison prédéfini.
  
  * *Envoyer des messages Cloud vers appareil et recevoir des accusés de remise*. Ces points de terminaison autorisent le backend de votre solution à envoyer des [messages cloud-à-appareil](iot-hub-devguide-messages-c2d.md) fiables et à recevoir les accusés de réception ou d’expiration correspondants.
  
  * *Recevoir les notifications de fichier*. Ce point de terminaison de messagerie vous permet de recevoir des notifications lorsqu’un fichier est correctement téléchargé sur votre appareil. 
  
  * *Invocation de méthode directe*. Ce point de terminaison permet à un service backend d’appeler une [méthode directe](iot-hub-devguide-direct-methods.md) sur un appareil.
  
  * *Recevoir les événements de surveillance des opérations*. Ce point de terminaison vous permet de recevoir les événements de surveillance des opérations si votre IoT hub a été configuré pour les émettre. Pour plus d’informations, consultez [Surveillance des opérations IoT Hub](iot-hub-operations-monitoring.md).

L’article [Kits Azure IoT SDK](iot-hub-devguide-sdks.md) décrit les différentes méthodes permettant d’accéder à ces points de terminaison.

Tous les points de terminaison IoT Hub utilisent le protocole [TLS](https://tools.ietf.org/html/rfc5246), et aucun point de terminaison n’est jamais exposé sur des canaux non chiffrés/non sécurisés.

## <a name="custom-endpoints"></a>Points de terminaison personnalisés

Vous pouvez lier des services Azure existants dans votre abonnement à votre hub IoT pour qu’ils jouent le rôle de points de terminaison pour le routage des messages. Ces points de terminaison de service jouent le rôle de points de terminaison de service et sont utilisés pour les routages de message. Les appareils ne peuvent pas écrire directement dans des points de terminaison supplémentaires. Découvrez-en plus sur le [routage de messages](../iot-hub/iot-hub-devguide-messages-d2c.md).

IoT Hub prend actuellement en charge les services Azure suivants en tant que points de terminaison supplémentaires :

* Conteneurs de stockage Azure
* Event Hubs
* Files d'attente Service Bus
* Rubriques de Service Bus

Pour connaître les limites du nombre de points de terminaison que vous pouvez ajouter, consultez [Quotas et limitation](iot-hub-devguide-quotas-throttling.md).

## <a name="endpoint-health"></a>État d’intégrité des points de terminaison

[!INCLUDE [iot-hub-endpoint-health](../../includes/iot-hub-include-endpoint-health.md)]

## <a name="field-gateways"></a>Passerelles de champ

Dans une solution IoT, une *passerelle de champ* se situe entre vos appareils et vos points de terminaison IoT Hub. Elle est généralement située près de vos appareils. Vos appareils communiquent directement avec la passerelle de champ à l’aide d’un protocole pris en charge. La passerelle de champ se connecte à un point de terminaison IoT Hub à l’aide d’un protocole pris en charge par ce dernier. Une passerelle de champ peut être un matériel dédié ou un ordinateur à faible puissance exécutant un logiciel de passerelle personnalisé.

Vous pouvez utiliser [Azure IoT Edge](../iot-edge/index.yml) pour implémenter une passerelle de champ. IoT Edge offre des fonctionnalités, comme la possibilité de multiplexer les communications à partir de plusieurs appareils sur la même connexion IoT Hub.

## <a name="next-steps"></a>Étapes suivantes

Les autres rubriques de référence de ce Guide du développeur IoT Hub comprennent :

* [Langage de requête IoT Hub pour les jumeaux d’appareil, les travaux et le routage des messages](iot-hub-devguide-query-language.md)
* [Quotas et limitation](iot-hub-devguide-quotas-throttling.md)
* [Prise en charge de MQTT dans IoT Hub](iot-hub-mqtt-support.md)
* [Comprendre l’adresse IP de votre hub IoT](iot-hub-understand-ip-address.md)