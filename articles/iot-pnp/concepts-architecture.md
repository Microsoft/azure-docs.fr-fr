---
title: Architecture IoT Plug-and-Play | Microsoft Docs
description: En tant que créateur de solutions, comprenez les principaux éléments architecturaux d’IoT Plug-and-Play.
author: ridomin
ms.author: rmpablos
ms.date: 09/15/2020
ms.topic: conceptual
ms.custom: mvc
ms.service: iot-pnp
services: iot-pnp
manager: philmea
ms.openlocfilehash: 2cccb1fdfe775250f80da6cc2ecdcc4ddaa3d88e
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95484081"
---
# <a name="iot-plug-and-play-architecture"></a>Architecture IoT Plug-and-Play

IoT Plug-and-Play permet aux créateurs de solutions d’intégrer des appareils intelligents dans leurs solutions sans configuration manuelle. Au cœur d’IoT Plug-and-Play se trouve un _modèle_ d’appareil qui décrit les capacités d’un appareil à une application IoT Plug-and-Play. Ce modèle est structuré sous la forme d’un ensemble d’interfaces qui définissent ce qui suit :

- _Propriétés_ qui représentent l’état en lecture seule ou en écriture d’un appareil ou d’une autre entité. Par exemple, un numéro de série d’appareil peut être une propriété en lecture seule et une température cible sur un thermostat peut être une propriété en écriture.
- _Télémétrie_ qui correspond aux données émises par un appareil, qu’il s’agisse d’un flux régulier de relevés de capteurs, d’une erreur occasionnelle ou d’un message d’information.
- _Commandes_ qui décrivent une fonction ou une opération qui peut être effectuée sur un appareil. Par exemple, une commande peut redémarrer une passerelle ou prendre une photo à l’aide d’une caméra distante.

Chaque modèle et interface possèdent un ID unique.

Le diagramme suivant montre les éléments clés d’une solution IoT Plug-and-Play :

:::image type="content" source="media/concepts-architecture/pnp-architecture.png" alt-text="Architecture IoT Plug-and-Play":::

## <a name="model-repository"></a>Référentiel de modèles

Le [référentiel de modèles](./concepts-model-repository.md) est un magasin de définitions de modèles et d’interfaces. Vous définissez des modèles et des interfaces à l’aide du [langage DTDL (Digital Twins Definition Language)](https://github.com/Azure/opendigitaltwins-dtdl).

L’interface utilisateur web vous permet de gérer les modèles et les interfaces.

Le référentiel de modèles dispose de contrôles d’accès en fonction du rôle intégrés qui vous permettent de limiter l’accès aux définitions d’interface.

## <a name="devices"></a>Appareils

Un générateur d’appareils implémente le code à exécuter sur une smart device IoT à l’aide de l’un des [Azure IoT device SDK](./libraries-sdks.md). Les Kits de développement logiciel (SDK) pour appareils aident le générateur d’appareils à :

- Se connecter en toute sécurité à un hub IoT.
- Inscrire l’appareil auprès de votre hub IoT et annoncer l’ID de modèle qui identifie la collection d’interfaces DTDL implémentée par l’appareil.
- Synchroniser les propriétés définies dans les interfaces DTDL entre l’appareil et votre hub IoT.
- Ajouter des gestionnaires de commandes pour les commandes définies dans les interfaces DTDL.
- Envoyer la télémétrie au hub IoT.

## <a name="iot-edge-gateway"></a>Passerelle IoT Edge

Une passerelle IoT Edge agit comme un intermédiaire pour connecter des appareils IoT Plug-and-Play qui ne peuvent pas se connecter directement à un hub IoT. Pour en savoir plus, consultez [Guide pratique pour utiliser un appareil IoT Edge en tant que passerelle](../iot-edge/iot-edge-as-gateway.md).

## <a name="iot-edge-modules"></a>Modules IoT Edge

Un _module IoT Edge_ vous permet de déployer et de gérer la logique métier sur la périphérie. Les modules Azure IoT Edge sont la plus petite unité de calcul gérée par IoT Edge. Ils peuvent contenir des services Azure (par exemple Azure Stream Analytics) ou votre propre code spécifique à la solution.

Le _hub IoT Edge_ est l’un des modules qui composent le runtime Azure IoT Edge. Il joue le rôle de proxy local pour IoT Hub en exposant les mêmes points de terminaison de protocole qu’IoT Hub. Cette cohérence signifie que les clients (qu’il s’agisse d’appareils ou de modules) peuvent se connecter au runtime IoT Edge comme à IoT Hub.

Les Kits de développement logiciel (SDK) aident le générateur de module à :

- Utiliser le hub IoT Edge pour se connecter de façon sécurisée à votre hub IoT.
- Inscrire le module auprès de votre hub IoT et annoncer l’ID de modèle qui identifie la collection d’interfaces DTDL implémentée par l’appareil.
- Synchroniser les propriétés définies dans les interfaces DTDL entre l’appareil et votre hub IoT.
- Ajouter des gestionnaires de commandes pour les commandes définies dans les interfaces DTDL.
- Envoyer la télémétrie au hub IoT.

## <a name="iot-hub"></a>IoT Hub

[IoT Hub](../iot-hub/about-iot-hub.md) est un service hébergé dans le cloud qui fait office de hub de messages central pour la communication bidirectionnelle entre votre solution IoT et les appareils qu’il gère.

Un hub IoT :

- Rend disponible l’ID de modèle implémenté par un appareil pour une solution principale.
- Gère les jumeaux numériques associés à chaque appareil Plug-and-Play connecté au hub.
- Transfère des flux de télémétrie à d’autres services à des fins de traitement ou de stockage.
- Achemine les événements de changement de jumeau numérique vers d’autres services pour activer l’analyse des appareils.

## <a name="backend-solution"></a>Solution principale

Une solution principale surveille et contrôle des appareils connectés en interagissant avec les jumeaux numériques dans le hub IoT. Utilisez l’un des Kits de développement logiciel (SDK) du service pour implémenter votre solution principale. Pour comprendre les capacités d’un appareil connecté, la solution principale :

1. Récupère l’ID de modèle que l’appareil a inscrit auprès du hub IoT.
1. Utilise l’ID de modèle pour récupérer les définitions d’interface à partir de tout référentiel de modèles.
1. Utilise l’analyseur de modèle pour extraire des informations à partir des définitions d’interface.

La solution principale peut utiliser les informations des définitions d’interface pour :

- Lire les valeurs de propriété rapportées par les appareils.
- Mettre à jour les propriétés accessibles en écriture sur un appareil.
- Appeler des commandes implémentées par un appareil.
- Comprendre le format de la télémétrie envoyée par un appareil.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous disposez d’une vue d’ensemble de l’architecture d’une solution IoT Plug-and-Play, les étapes suivantes permettent d’en savoir plus :

- [Référentiel de modèles](./concepts-model-repository.md)
- [Intégration de modèles de jumeau numérique](./concepts-model-discovery.md)
- [Développement d’IoT Plug-and-Play](./concepts-developer-guide-device.md)
