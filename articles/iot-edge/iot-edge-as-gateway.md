---
title: Passerelles pour les appareils en aval - Azure IoT Edge | Microsoft Docs
description: Utilisez Azure IoT Edge pour créer un appareil de passerelle proxy, opaque ou transparent qui envoie des données à partir de plusieurs appareils en aval vers le cloud ou qui traite ces données localement.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 11/10/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 83e8089073f7e7e7634ddf00f7276e12aaf645b0
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94536436"
---
# <a name="how-an-iot-edge-device-can-be-used-as-a-gateway"></a>Guide pratique pour utiliser un appareil IoT Edge en tant que passerelle

Les appareils IoT Edge peuvent fonctionner en tant que passerelles, en fournissant une connexion entre d’autres appareils sur le réseau et sur IoT Hub.

Le module hub IoT Edge agit comme IoT Hub, de sorte qu’il peut gérer les connexions à partir de tous les appareils qui ont une identité auprès d’IoT Hub, y compris d’autres appareils IoT Edge. Ce type de modèle de passerelle est appelé *transparent*, car les messages peuvent passer d’appareils en aval à IoT Hub comme s’il n’existait pas de passerelle entre eux.

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"
À partir de la version 1.2 d’IoT Edge, les passerelles transparentes peuvent gérer les connexions en aval à partir d’autres appareils IoT Edge.
::: moniker-end

Pour les appareils qui ne se connectent pas ou ne peuvent pas se connecter à IoT Hub eux-mêmes, les passerelles IoT Edge peuvent leur fournir cette connexion. Ce type de modèle de passerelle est appelé *traduction*, car l’appareil IoT Edge doit effectuer le traitement des messages entrants d’appareils en aval avant de les transmettre à IoT Hub. Ces scénarios requièrent des modules supplémentaires sur la passerelle IoT Edge pour gérer les étapes de traitement.

Les modèles de passerelle transparente et de traduction ne s’excluent pas mutuellement. Un seul appareil IoT Edge peut fonctionner à la fois en tant que passerelle transparente et passerelle de traduction.

Tous les modèles de passerelles fournissent les avantages suivants :

* **Analytique en périphérie** : utilisez les services IA localement pour traiter les données provenant des appareils en aval, sans envoyer de données de télémétrie haute fidélité au cloud. Recherchez des insights localement, réagissez-y, et envoyez uniquement un sous-ensemble de données à IoT Hub.
* **Isolation d’appareil en aval** : l’appareil de passerelle peut protéger tous les appareils en aval contre une exposition sur Internet. Il peut être placé entre un réseau de technologies opérationnelles (OT) qui n’a pas de connectivité et un réseau de technologies de l’information (IT) qui fournit l’accès au web. De même, les appareils qui n’ont pas la capacité de se connecter à IoT Hub eux-mêmes peuvent se connecter à un appareil de passerelle à la place.
* **Multiplexage des connexions** : tous les appareils qui se connectent à IoT Hub via une passerelle IoT Edge utilisent la même connexion sous-jacente.
* **Lissage du trafic** : l’appareil IoT Edge implémente automatiquement une interruption exponentielle si IoT Hub limite le trafic, tout en assurant une conservation locale des messages. Cet avantage rend votre solution résiliente face aux pics de trafic.
* **Prise en charge hors connexion** : l’appareil de passerelle stocke les messages et les mises à jour du jumeau qui ne peuvent pas être remis à IoT Hub.

## <a name="transparent-gateways"></a>Passerelles transparentes

Dans le modèle de passerelle transparente, les appareils qui, en théorie, peuvent se connecter à IoT Hub peuvent se connecter à un appareil de passerelle à la place. Les appareils situés en aval ont leurs propres identités IoT Hub et se connectent à l’aide des protocoles MQTT ou AMQP. La passerelle se contente de transférer les communications entre les appareils et IoT Hub. Les appareils et les utilisateurs qui interagissent avec eux par le biais d’IoT Hub ne savent pas qu’une passerelle effectue la médiation de leurs communications. Cette ignorance signifie que la passerelle est considérée comme *transparente*.

<!-- 1.0.10 -->
::: moniker range="iotedge-2018-06"

Les appareils IoT Edge ne peuvent pas se trouver en aval d’une passerelle IoT Edge.

![Diagramme : modèle de passerelle transparente](./media/iot-edge-as-gateway/edge-as-gateway-transparent.png)

::: moniker-end

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"

À partir de la version 1.2.0, les appareils IoT Edge peuvent se connecter via des passerelles transparentes.

<!-- TODO add a downstream IoT Edge device to graphic -->

::: moniker-end

### <a name="parent-and-child-relationships"></a>Relations parent-enfant

Vous pouvez déclarer des relations de passerelle transparente dans IoT Hub en définissant la passerelle IoT Edge comme *parent* d’un appareil en aval *enfant* qui s’y connecte.

La relation parent-enfant est établie en trois points dans la configuration de la passerelle :

#### <a name="cloud-identities"></a>Identités cloud

Tous les appareils d’un scénario de passerelle transparente ont besoin d’identités cloud pour pouvoir s’authentifier auprès d’IoT Hub. Lorsque vous créez ou mettez à jour une identité d’appareil, vous pouvez définir les appareils parent ou enfant de l’appareil. Cette configuration autorise l’appareil de passerelle parent à gérer l’authentification pour ses appareils enfants.

Les appareils enfants ne peuvent avoir qu’un seul parent. Chaque parent peut avoir jusqu’à 100 enfants.

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"
Les appareils IoT Edge peuvent être à la fois parents et enfants dans des relations de passerelle transparente. Il est possible de créer une hiérarchie de plusieurs appareils IoT Edge qui se rapportent les uns aux autres. Le nœud supérieur d’une hiérarchie de passerelle peut avoir jusqu’à cinq générations d’enfants. Par exemple, un appareil IoT Edge peut avoir cinq couches d’appareils IoT Edge qui lui sont liés en tant qu’enfants au-dessous de lui. Mais l’appareil IoT Edge de la cinquième génération ne peut pas avoir d’enfants, IoT Edge ou autre.
::: moniker-end

#### <a name="gateway-discovery"></a>Détection de passerelle

Un appareil enfant doit être capable de trouver son appareil parent sur le réseau local. Configurez les appareils de passerelle avec un **nom d’hôte**, soit un nom de domaine complet (FQDN), soit une adresse IP, que ses appareils enfants utiliseront pour le localiser.

Sur les appareils IoT en aval, utilisez le paramètre **gatewayHostname** dans la chaîne de connexion pour pointer vers l’appareil parent.

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"
Sur les appareils IoT Edge en aval, utilisez le paramètre **parent_hostname** dans le fichier config.yaml pour pointer vers l’appareil parent.
::: moniker-end

#### <a name="secure-connection"></a>Connexion sécurisée

Les appareils parents et enfants doivent également authentifier leurs connexions entre eux. Chaque appareil a besoin de la copie d’un certificat partagé d’autorité de certification racine que les appareils enfants utilisent pour vérifier qu’ils se connectent à la passerelle appropriée.

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"
Lorsque plusieurs passerelles IoT Edge se connectent les unes aux autres dans une hiérarchie de passerelle, tous les appareils de la hiérarchie doivent utiliser une seule chaîne de certificats.
::: moniker-end

### <a name="device-capabilities-behind-transparent-gateways"></a>Capacités de l’appareil derrière les passerelles transparentes

Toutes les primitives IoT Hub qui fonctionnent avec le pipeline de messagerie d’IoT Edge prennent également en charge les scénarios de passerelle transparente. Chaque passerelle IoT Edge dispose de capacités de stockage et de transfert pour les messages qui passent par elle.

Utilisez le tableau suivant pour voir comment différentes capacités IoT Hub sont prises en charge par les appareils, par rapport aux appareils qui se trouvent derrière les passerelles.

<!-- 1.0.10 -->
::: moniker range="iotedge-2018-06"

| Fonctionnalité | Appareil IoT | IoT derrière une passerelle |
| ---------- | ---------- | -------------------- |
| [Messages appareil-à-cloud (D2C)](../iot-hub/iot-hub-devguide-messages-d2c.md) |  ![Oui : IoT D2C](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : IoT D2C enfant](./media/iot-edge-as-gateway/check-yes.png) |
| [Messages cloud-à-appareil (C2D)](../iot-hub/iot-hub-devguide-messages-c2d.md) | ![Oui : IoT C2D](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : IoT C2D enfant](./media/iot-edge-as-gateway/check-yes.png) |
| [Méthodes directes](../iot-hub/iot-hub-devguide-direct-methods.md) | ![Oui : méthode directe IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : méthode directe IoT enfant](./media/iot-edge-as-gateway/check-yes.png) |
| [Jumeaux d’appareil](../iot-hub/iot-hub-devguide-device-twins.md) et [jumeaux de module](../iot-hub/iot-hub-devguide-module-twins.md) | ![Oui : jumeaux IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : jumeaux IoT enfant](./media/iot-edge-as-gateway/check-yes.png) |
| [Chargement de fichiers](../iot-hub/iot-hub-devguide-file-upload.md) | ![Oui : chargement de fichier IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Non : chargement de fichier enfant IoT](./media/iot-edge-as-gateway/crossout-no.png) |

::: moniker-end

<!-- 1.2.0 -->
::: moniker range=">=iotedge-2020-11"

| Fonctionnalité | Appareil IoT | IoT derrière une passerelle | Appareil IoT Edge | IoT Edge derrière une passerelle |
| ---------- | ---------- | --------------------------- | --------------- | -------------------------------- |
| [Messages appareil-à-cloud (D2C)](../iot-hub/iot-hub-devguide-messages-d2c.md) |  ![Oui : IoT D2C](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : IoT D2C enfant](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : IoT Edge D2C](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : IoT Edge D2C enfant](./media/iot-edge-as-gateway/check-yes.png) |
| [Messages cloud-à-appareil (C2D)](../iot-hub/iot-hub-devguide-messages-c2d.md) | ![Oui : IoT C2D](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : IoT C2D enfant](./media/iot-edge-as-gateway/check-yes.png) | ![Non : IoT Edge C2D](./media/iot-edge-as-gateway/crossout-no.png) | ![Non : IoT Edge C2D enfant](./media/iot-edge-as-gateway/crossout-no.png) |
| [Méthodes directes](../iot-hub/iot-hub-devguide-direct-methods.md) | ![Oui : méthode directe IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : méthode directe IoT enfant](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : méthode directe IoT Edge](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : méthode directe IoT Edge enfant](./media/iot-edge-as-gateway/check-yes.png) |
| [Jumeaux d’appareil](../iot-hub/iot-hub-devguide-device-twins.md) et [jumeaux de module](../iot-hub/iot-hub-devguide-module-twins.md) | ![Oui : jumeaux IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : jumeaux IoT enfant](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : jumeaux IoT Edge](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : jumeaux IoT Edge enfant](./media/iot-edge-as-gateway/check-yes.png) |
| [Chargement de fichiers](../iot-hub/iot-hub-devguide-file-upload.md) | ![Oui : chargement de fichier IoT](./media/iot-edge-as-gateway/check-yes.png) | ![Non : chargement de fichier enfant IoT](./media/iot-edge-as-gateway/crossout-no.png) | ![Non : chargement de fichier IoT Edge](./media/iot-edge-as-gateway/crossout-no.png) | ![Non : chargement de fichier enfant IoT Edge](./media/iot-edge-as-gateway/crossout-no.png) |
| Tirages (pull) d’images conteneur |   |   | ![Oui : tirage de conteneur IoT Edge](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : tirage de conteneur IoT Edge enfant](./media/iot-edge-as-gateway/check-yes.png) |
| Chargement de blobs |   |   | ![Oui : chargement de blobs IoT Edge](./media/iot-edge-as-gateway/check-yes.png) | ![Oui : chargement de blobs IoT Edge enfant](./media/iot-edge-as-gateway/check-yes.png) |

**Les images de conteneur** peuvent être téléchargées, stockées et transmises d’appareils parents vers des appareils enfants.

**Les blobs**, y compris les offres groupées d’assistance et les journaux, peuvent être chargés d’appareils enfants sur des appareils parents.

::: moniker-end

## <a name="translation-gateways"></a>Passerelles de traduction

Si les appareils en aval ne peuvent pas se connecter à IoT Hub, la passerelle IoT Edge doit alors jouer le rôle de traducteur. Ce modèle est souvent requis pour les appareils qui ne prennent pas en charge les protocoles MQTT, AMQP ou HTTP. Comme ces appareils ne peuvent pas se connecter à IoT Hub, ils ne peuvent pas non plus se connecter au module hub IoT Edge sans traitement préalable.

Des modules tiers ou personnalisés qui sont souvent spécifiques au matériel ou au protocole de l’appareil en aval doivent être déployés sur la passerelle IoT Edge. Ces modules de traduction prennent les messages entrants et les transforment en un format qui peut être compris par IoT Hub.

Il existe deux modèles pour les passerelles de traduction : *traduction de protocole* et *traduction d’identité*.

![Diagramme: modèles de passerelle de traduction](./media/iot-edge-as-gateway/edge-as-gateway-translation.png)

### <a name="protocol-translation"></a>Traduction de protocole

Dans le modèle de passerelle de traduction de protocole, seule la passerelle IoT Edge a une identité avec IoT Hub. Le module de traduction reçoit des messages des appareils en aval et les traduit en un protocole pris en charge, puis l’appareil IoT Edge envoie les messages au nom des appareils en aval. Toutes les informations semblent provenir d’un seul appareil, la passerelle. Les appareils situés en aval doivent intégrer des informations d'identification supplémentaires dans leurs messages si les applications cloud veulent analyser les données pour chaque appareil. De plus, les primitives IoT Hub comme les jumeaux et les méthodes directes sont uniquement prises en charge pour l’appareil de passerelle, et non pour les appareils situés en aval. Les passerelles de ce modèle sont considérées comme *opaques* contrairement aux passerelles transparentes, car elles masquent les identités des appareils en aval.

La traduction de protocole prend en charge les appareils dont les ressources sont limitées. De nombreux appareils existants produisent des données qui peuvent alimenter les insights métier : toutefois, ils n’ont pas été conçus pour la connectivité au cloud. Les passerelles opaques permettent le déverrouillage et l’utilisation de ces données dans une solution IoT.

### <a name="identity-translation"></a>Traduction d’identité

Le modèle de passerelle de traduction d’identité s’appuie sur la traduction de protocole, mais la passerelle IoT Edge fournit également une identité d’appareil IoT Hub au nom des appareils en aval. Le module de traduction est chargé de comprendre le protocole utilisé par les appareils en aval, de leur fournir une identité et de traduire leurs messages en primitives IoT Hub. Les appareils en aval s’affichent dans IoT Hub comme des appareils de premier niveau avec des représentations et des méthodes. Un utilisateur peut interagir avec les appareils dans IoT Hub et n’a pas connaissance de l’appareil de passerelle intermédiaire.

La traduction d’identité fournit les avantages de la traduction de protocole et permet, en plus, la gestion complète des appareils en aval à partir du cloud. Tous les appareils associés à votre solution IoT s’affichent dans IoT Hub, quel que soit le protocole qu’ils utilisent.

### <a name="device-capabilities-behind-translation-gateways"></a>Capacités de l’appareil derrière les passerelles de traduction

Le tableau suivant explique la façon dont les fonctionnalités IoT Hub sont étendues aux appareils en aval dans les deux modèles de passerelle de traduction.

| Fonctionnalité | Traduction de protocole | Traduction d’identité |
| ---------- | -------------------- | -------------------- |
| Identités stockées dans le registre des identités IoT Hub | Uniquement l’identité de l’appareil de passerelle | Identités de tous les appareils connectés |
| Jumeau d’appareil | Seule la passerelle a un appareil et des jumeaux de module | Chaque appareil connecté possède son propre jumeau d’appareil |
| Méthodes directes et messages du cloud vers l’appareil | Le cloud ne peut porter que sur l’appareil de passerelle | Le cloud peut porter individuellement sur chaque appareil connecté |
| [Limitations et quotas d’IoT Hub](../iot-hub/iot-hub-devguide-quotas-throttling.md) | Appliquer à l’appareil de passerelle | Appliquer à chaque appareil |

Lorsque vous utilisez un modèle de traduction de protocole, tous les appareils qui se connectent via cette passerelle partagent la même file d’attente cloud-à-appareil, qui peut contenir au maximum 50 messages. Utilisez ce modèle uniquement lorsque peu d’appareils se connectent via chaque passerelle locale et que le trafic cloud-à-appareil est faible.

Le runtime IoT Edge n’inclut pas de protocole ni de capacités de traduction d’identité. Ces modèles nécessitent des modules personnalisés ou tiers qui sont souvent spécifiques au matériel et au protocole utilisés. La [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1&subcategories=iot-edge-modules) contient plusieurs modules de traduction de protocole. Pour un exemple qui utilise le modèle de traduction d’identité, consultez [Kit de démarrage LoRaWAN d’Azure IoT Edge](https://github.com/Azure/iotedge-lorawan-starterkit).

## <a name="next-steps"></a>Étapes suivantes

Découvrez les trois étapes de configuration d’une passerelle transparente :

* [Configurer un appareil IoT Edge en tant que passerelle transparente](how-to-create-transparent-gateway.md)
* [Authentifier un appareil en aval auprès d’Azure IoT Hub](how-to-authenticate-downstream-device.md)
* [Connecter un appareil en aval à une passerelle Azure IoT Edge](how-to-connect-downstream-device.md)
