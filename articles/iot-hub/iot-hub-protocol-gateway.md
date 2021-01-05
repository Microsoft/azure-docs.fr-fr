---
title: Passerelle de protocole IoT Azure | Microsoft Docs
description: Comment utiliser une passerelle de protocole Azure IoT pour étendre les capacité IoT Hub et la prise en charge du protocole pour autoriser les appareils à se connecter à votre hub ç l’aide des protocoles non pris en charge par IoT Hub en mode natif.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 07/11/2017
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 315d76715973b79a971f3ca15bcc5186d20135a2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87325030"
---
# <a name="support-additional-protocols-for-iot-hub"></a>Prise en charge de protocoles supplémentaires pour IoT Hub

Azure IoT Hub en mode natif prend en charge la communication par le biais des protocoles AMQP, MQTT et HTTPS. Dans certains cas, il est possible que des appareils ou des passerelles ne soient pas en mesure d’utiliser l’un des protocoles standard et nécessitent une adaptation de protocole. Dans ce cas, vous pouvez utiliser une passerelle personnalisée. Une passerelle personnalisée active l’adaptation de protocole pour les points de terminaison IoT Hub en effectuant le pontage du trafic vers et à partir d’IoT Hub. Vous pouvez utiliser la [passerelle de protocole Azure IoT](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md) comme passerelle personnalisée pour permettre l’adaptation de protocole pour IoT Hub.

## <a name="azure-iot-protocol-gateway"></a>passerelle de protocole Azure IoT

La passerelle de protocole Azure IoT est une infrastructure pour l’adaptation du protocole, conçue pour la communication bidirectionnelle à grande échelle entre les appareils, avec IoT Hub. La passerelle de protocole est un composant direct qui accepte les connexions des appareils via un protocole spécifique. Elle réduit le trafic vers IoT Hub via AMQP 1.0.

Vous pouvez déployer la passerelle de protocole dans Azure de manière très évolutive grâce à Azure Service Fabric, aux rôles de travail d’Azure Cloud Services ou aux machines virtuelles Windows. En outre, la passerelle de protocole peut être déployée dans les environnements locaux comme des passerelles de champ.

La passerelle de protocole Azure IoT inclut un adaptateur de protocole MQTT grâce auquel vous pouvez personnaliser le comportement du protocole MQTT, si nécessaire. Étant donné qu’IoT Hub prend naturellement en charge le protocole MQTT v3.1.1, vous ne devez envisager l’utilisation de l’adaptateur de protocole MQTT que si des personnalisations de protocole ou de fonctionnalités supplémentaires sont requises.

L’adaptateur MQTT illustre également le modèle de programmation pour la création d’adaptateurs de protocole pour d’autres protocoles. En outre, le modèle de programmation de passerelle de protocole Azure IoT vous permet de connecter des composants personnalisés pour les traitements spécialisés, comme l’authentification personnalisée, les transformations de messages, la compression / décompression ou le chiffrement / déchiffrement du trafic entre les appareils et IoT Hub.

Pour plus de flexibilité, la passerelle de protocole Azure IoT et l’implémentation de MQTT sont fournies dans un projet logiciel open source. Vous pouvez utiliser le projet open source pour ajouter la prise en charge de différents protocoles et versions de protocole, ou personnaliser l’implémentation en fonction de votre scénario. 

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la passerelle de protocole Azure IoT, son utilisation et son déploiement dans le cadre de votre solution IoT, consultez :

* [Référentiel sur la passerelle de protocole Azure IoT sur GitHub](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md)

* [Guide du développeur de passerelle de protocole Azure IoT](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/docs/DeveloperGuide.md)

Pour en savoir plus sur la planification de votre déploiement IoT Hub, consultez :

* [Comparer à Event Hubs](iot-hub-compare-event-hubs.md)

* [Mise à l’échelle, haute disponibilité et récupération d’urgence](iot-hub-scaling.md)

* [Guide du développeur d’IoT Hub](iot-hub-devguide.md)