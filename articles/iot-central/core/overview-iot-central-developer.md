---
title: Développement d’appareils pour Azure IoT Central | Microsoft Docs
description: Azure IoT Central est une plateforme d’applications IoT qui simplifie la création de solutions IoT. Cet article fournit une vue d’ensemble du développement d’appareils pour se connecter à votre application IoT Central. Les appareils utilisent la télémétrie pour envoyer des données de streaming et des propriétés pour signaler leur état. IOT Central peut définir l’état d’un appareil à l’aide de propriétés accessibles en écriture et appeler des commandes sur un appareil.
author: dominicbetts
ms.author: dobett
ms.date: 05/05/2020
ms.topic: overview
ms.service: iot-central
services: iot-central
ms.custom:
- mvc
- device-developer
ms.openlocfilehash: e33f48c9496ffa3cca9d8b1aa71d524be9a311bb
ms.sourcegitcommit: b8a175b6391cddd5a2c92575c311cc3e8c820018
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96122253"
---
# <a name="iot-central-device-development-overview"></a>Vue d’ensemble du développement d’appareils IoT Central

*Cet article s’applique aux développeurs d’appareils.*

Une application IoT Central vous permet de surveiller et de gérer des millions d’appareils tout au long de leur cycle de vie. Cette vue d’ensemble s’adresse aux développeurs d’appareils qui implémentent un code qui s’exécutera sur des appareils connectés à IoT Central.

Les appareils interagissent avec une application IoT Central à l’aide des primitives suivantes :

- La _télémétrie_ représente les données qu’un appareil envoie à IoT Central. Par exemple, un flux de valeurs de température provenant d’un capteur intégré.
- Les _propriétés_ représentent des valeurs d’état qu’un appareil transmet à IoT Central. Par exemple, la version actuelle du microprogramme installé sur l’appareil. Vous pouvez également utiliser des propriétés accessibles en écriture qu’IoT Central peut mettre à jour sur l’appareil telles qu’une température cible.
- Les _commandes_ sont appelées à partir d’IoT Central pour contrôler le comportement d’un appareil. Par exemple, votre application IoT Central peut appeler une commande pour redémarrer un appareil.

Un générateur de solutions est responsable de la configuration des tableaux de bord et des vues dans l’interface utilisateur Web IoT Central pour visualiser les données de télémétrie, gérer les propriétés et appeler les commandes.

## <a name="types-of-device"></a>Types d’appareils

Les sections suivantes décrivent les principaux types d’appareils que vous pouvez connecter à une application IoT Central :

### <a name="standalone-device"></a>Appareil autonome

Un appareil autonome se connecte directement à IoT Central. Un appareil autonome envoie généralement des données de télémétrie à partir de ses capteurs intégrés ou connectés à votre application IoT Central. Les appareils autonomes peuvent également transmettre des valeurs de propriété, recevoir des valeurs de propriété accessibles en écriture, et répondre à des commandes.

### <a name="gateway-device"></a>Appareil de passerelle

Un appareil de passerelle gère un ou plusieurs appareils en aval qui se connectent à votre application IoT Central. Vous utilisez IoT Central pour configurer les relations entre les appareils en aval et l’appareil de passerelle. Pour en savoir plus, voir [Définir un nouveau type d’appareil de passerelle IoT dans votre application Azure IoT Central](./tutorial-define-gateway-device-type.md).

### <a name="edge-device"></a>Appareil de périphérie

Un appareil de périphérie se connecte directement à IoT Central, mais agit en tant qu’intermédiaire pour d’autres appareils appelés _appareils de nœud terminal_. Un appareil de périphérie est généralement situé près des appareils de nœud terminal pour lesquels il joue le rôle d’intermédiaire. Les scénarios suivants utilisent des appareils de périphérie :

- Activer les appareils qui ne peuvent pas se connecter directement à IoT Central pour se connecter via l’appareil de périphérie. Par exemple, un appareil de nœud terminal peut utiliser Bluetooth pour se connecter à l’appareil de périphérie, qui se connecte ensuite via Internet à IoT Central.
- Agréger les données de télémétrie avant leur envoi à IoT Central. Cette approche peut aider à réduire les coûts liés à l’envoi de données à IoT Central.
- Contrôler localement des appareils de nœud terminal pour éviter la latence associée à la connexion à IoT Central via Internet.

Un appareil de périphérie peut également envoyer ses propres données de télémétrie, transmettre ses propriétés, et répondre à des commandes et des mises à jour de propriétés accessibles en écriture.

IoT Central ne voit que l’appareil de périphérie et pas les appareils de nœud terminal qui y sont connectés.

Pour en savoir plus, voir [Ajouter un appareil Azure IoT Edge à votre application Azure IoT Central](./tutorial-add-edge-as-leaf-device.md).

## <a name="connect-a-device"></a>Connexion d'un appareil

Azure IoT Central utilise le [service Azure IoT Device Provisioning (DPS)](../../iot-dps/about-iot-dps.md) pour gérer toutes les inscriptions et connexions d’appareils.

L’utilisation de DPS permet :

- à IoT Central de prendre en charge l’intégration et la connexion des appareils à l’échelle ;
- aux clients de générer des informations d’identification d’appareils et de les configurer hors ligne sans avoir à les inscrire via l’interface utilisateur d’IoT Central ;
- aux clients d’utiliser leurs propres ID d’appareils pour inscrire des appareils dans IoT Central. L’utilisation de vos propres ID d’appareils simplifie l’intégration dans des systèmes back-office existants.
- C’est une manière cohérente de connecter des appareils à IoT Central.

Pour plus d’informations, consultez [Se connecter à Azure IoT Central](./concepts-get-connected.md).

### <a name="security"></a>Sécurité

La connexion entre un appareil et votre application IoT Central est sécurisée à l’aide de [signatures d’accès partagé](./concepts-get-connected.md#sas-group-enrollment) ou de [certificats X.509](./concepts-get-connected.md#x509-group-enrollment) standard.

### <a name="communication-protocols"></a>Protocoles de communication

Les protocoles de communication qu’un appareil peut utiliser pour se connecter à IoT Central incluent MQTT, AMQP et HTTPS. En interne, IoT Central utilise un hub IoT pour activer la connectivité des appareils. Pour plus d’informations sur les protocoles de communication pris en charge par IoT Hub pour la connectivité des appareils, voir [Choisir un protocole de communication](../../iot-hub/iot-hub-devguide-protocols.md).

## <a name="implement-the-device"></a>Implémenter l’appareil

Un modèle d’appareil IoT Central comprend un _modèle de capacité_ qui spécifie les comportements qu’un appareil de ce type doit implémenter. Les comportements incluent une télémétrie, des propriétés et des commandes.

> [!TIP]
> Vous pouvez exporter le modèle de capacité à partir d’IoT Central en tant que fichier JSON [DTDL (Digital Twins Definition Language) v2](https://github.com/Azure/opendigitaltwins-dtdl).

Chaque modèle de capacité a un _identificateur de modèle de capacité de jumeau d’appareil unique_ (DTMI), tel que `dtmi:com:example:Thermostat;1`. Lorsqu’un appareil se connecte à IoT Central, il envoie le DTMI du modèle de capacité qu’il implémente. IoT Central peut ensuite associer le modèle d’appareil approprié à l’appareil.

[IoT Plug-and-Play](../../iot-pnp/overview-iot-plug-and-play.md) définit un ensemble de conventions qu’un appareil doit suivre lorsqu’il implémente un modèle DTDL.

Les kits [Azure IoT device SDK](#languages-and-sdks) incluent la prise en charge des conventions IoT Plug-and-Play.

### <a name="device-model"></a>Modèle de l'appareil

Un modèle de capacité d’appareil est défini à l’aide du langage [DTDL](https://github.com/Azure/opendigitaltwins-dtdl). Ce langage vous permet de définir :

- La télémétrie envoyée par l’appareil. La définition comprend le nom et le type de données de la télémétrie. Par exemple, un appareil peut envoyer une télémétrie de température en tant que double.
- Les propriétés signalées par l’appareil à IoT Central. Une définition de propriété comprend son nom et son type de données. Par exemple, un appareil peut signaler l’état d’une vanne en tant que valeur booléenne.
- Les propriétés que l’appareil peut recevoir d’IoT Central. Si vous le souhaitez, vous pouvez marquer une propriété comme étant accessible en écriture. Par exemple, IoT Central peut envoyer une température cible en tant que double à un appareil.
- Les commandes auxquelles un appareil répond. La définition comprend le nom de la commande ainsi que les noms et les types de données des éventuels paramètres. Par exemple, un appareil peut répondre à une commande de redémarrage qui spécifie le nombre de secondes à attendre avant le redémarrage.

Un modèle DTDL peut être un modèle _sans composant_ ou _à plusieurs composants_ :

- Modèle sans composant : un modèle simple n’utilise pas de composants incorporés ou en cascade. Toutes les données de télémétrie, propriétés et commandes sont définies en tant que _composant par défaut_ unique. Pour obtenir un exemple, consultez le modèle [Thermostat](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json).
- Modèle à plusieurs composants : modèle plus complexe qui inclut deux composants ou plus. Ces composants incluent un composant unique par défaut et un ou plusieurs composants imbriqués supplémentaires. Pour obtenir un exemple, consultez le modèle [Contrôleur de température](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json).

Pour plus d’informations, consultez [Composants IoT Plug-and-Play dans les modèles](../../iot-pnp/concepts-components.md).

### <a name="conventions"></a>Conventions

Un appareil doit respecter les conventions IoT Plug-and-Play lorsqu’il échange des données avec IoT Central. Ces conventions sont les suivantes :

- Envoyer le DTMI lorsqu’il se connecte à IoT Central
- Envoyer des charges utiles JSON et des métadonnées correctement mises en forme à IoT Central
- Répondre correctement aux commandes et aux propriétés accessibles en écriture d’IoT Central
- Respecter les conventions de nommage pour les commandes de composant

> [!NOTE]
> Actuellement, IoT Central ne prend pas entièrement en charge les types de données DTDL **Array** et **Geospatial**.

Pour en savoir plus sur le format des messages JSON qu’un appareil échange avec IoT Central, consultez [Charges utiles de télémétrie, de propriétés et de commandes](concepts-telemetry-properties-commands.md).

Pour en savoir plus sur les conventions IoT Plug-and-Play, consultez [Conventions IoT Plug-and-Play](../../iot-pnp/concepts-convention.md).

### <a name="device-sdks"></a>Kits de développement logiciel (SDK) d’appareil

Utilisez l’un des [kits Azure IoT device SDK](#languages-and-sdks) pour implémenter le comportement de votre appareil. Le code doit :

- Inscrire l’appareil auprès de DPS et utiliser les informations de DPS pour se connecter au hub IoT interne de votre application IoT Central.
- Annoncer le DTMI du modèle de capacité implémenté par l’appareil.
- Envoyer la télémétrie au format spécifié par le modèle de capacité d’appareil. IoT Central utilise le modèle de capacité spécifié dans le modèle d’appareil afin de déterminer comment utiliser les données de télémétrie pour les visualisations et l’analyse.
- Synchroniser les valeurs de propriété entre l’appareil et IoT Central. Le modèle de capacité spécifie les noms des propriétés et les types de données afin qu’IoT Central puisse afficher les informations.
- Implémenter des gestionnaires de commandes pour les commandes spécifiées dans le modèle de capacité. Le modèle de capacité spécifie les noms des commandes et les paramètres que l’appareil doit utiliser.

Pour plus d’informations sur le rôle des modèles d’appareil, consultez [Présentation des modèles d’appareils](./concepts-device-templates.md).

Pour obtenir un exemple de code, consultez [Créer et connecter une application cliente](./tutorial-connect-device.md).

### <a name="languages-and-sdks"></a>Langues et kits de développement logiciel (SDK)

Pour plus d’informations sur les langages et kits de développement logiciel (SDK) pris en charge, voir [Comprendre et utiliser les kits de développement logiciel (SDK) Azure IoT Hub](../../iot-hub/iot-hub-devguide-sdks.md#azure-iot-hub-device-sdks).

## <a name="next-steps"></a>Étapes suivantes

Si vous êtes développeur d’appareils et que vous souhaitez vous immerger dans du code, l’étape suivante suggérée consiste à [Créer et connecter une application cliente à votre application Azure IoT Central](./tutorial-connect-device.md).

Si vous souhaitez en savoir plus sur l’utilisation d’IoT Central, nous vous suggérons de tester les démarrages rapides, en commençant par [Créer une application Azure IoT Central](./quick-deploy-iot-central.md).
