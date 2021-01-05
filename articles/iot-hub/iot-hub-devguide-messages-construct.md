---
title: Présentation du format des messages Azure IoT Hub | Microsoft Docs
description: Le guide du développeur décrit le format et le contenu attendu des messages IoT Hub.
author: ash2017
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 07/22/2019
ms.author: asrastog
ms.custom:
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 6d6b7122963b51619f26b8d02a8be4ad39261afb
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92147688"
---
# <a name="create-and-read-iot-hub-messages"></a>Créer et lire des messages IoT Hub

Pour prendre en charge une interopérabilité transparente entre les différents protocoles, IoT Hub définit un format de message commun pour tous les protocoles d’appareil. Ce format de message est utilisé pour les messages [de routage appareil-à-cloud](iot-hub-devguide-messages-d2c.md) et [cloud-à-appareil](iot-hub-devguide-messages-c2d.md). 

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

IoT Hub implémente les messages appareil-à-cloud à l’aide d’un modèle de messagerie en streaming. Les messages appareil-à-cloud d’IoT Hub s’apparentent plus à des *événements* [Event Hubs](../event-hubs/index.yml) qu’à des *messages* [Service Bus](../service-bus-messaging/index.yml), dans la mesure où de nombreux événements passant à travers le service peuvent être lus par plusieurs lecteurs.

Un message IoT Hub comprend les éléments suivants :

* Un ensemble prédéfini de *propriétés système*, telles que répertoriées ci-dessous.

* Un ensemble de *propriétés de l’application*. Dictionnaire de propriétés de chaîne que l’application peut définir et auquel elle peut accéder sans avoir à désérialiser le corps du message. IoT Hub ne modifie jamais ces propriétés.

* Un corps binaire opaque.

Les valeurs et les noms de propriétés peuvent contenir seulement des caractères alphanumériques ASCII, plus les caractères ``{'!', '#', '$', '%, '&', ''', '*', '+', '-', '.', '^', '_', '`', '|', '~'}`` quand vous envoyez des messages appareil-à-cloud avec le protocole HTTPS ou quand vous envoyez des messages cloud-à-appareil.

La messagerie appareil-à-cloud avec IoT Hub présente les caractéristiques suivantes :

* Les messages appareil-à-cloud sont durables et sont conservés dans le point de terminaison **messages/events** par défaut d’un hub IoT jusqu’à sept jours.

* Les messages appareil-à-cloud ne doivent pas dépasser 256 Ko et peuvent être groupés en lots pour optimiser les envois. Les lots ne peuvent pas dépasser 256 Ko.

* IoT Hub n’autorise pas le partitionnement arbitraire. Les messages appareil-à-cloud sont partitionnés selon leur **deviceId**d’origine.

* Comme expliqué dans [Contrôler l’accès à IoT Hub](iot-hub-devguide-security.md), IoT Hub permet l’authentification et le contrôle d’accès pour chaque appareil.

* Vous pouvez marquer les messages avec des informations qui sont placées dans les propriétés de l’application. Pour plus d’informations, consultez [Enrichissements de messages](iot-hub-message-enrichments-overview.md).

Pour plus d’informations sur l’encodage et le décodage des messages envoyés avec différents protocoles, consultez [SDK Azure IoT](iot-hub-devguide-sdks.md).

## <a name="system-properties-of-d2c-iot-hub-messages"></a>Propriétés système des messages IoT Hub **D2C**

| Propriété | Description  |L’utilisateur est-il définissable ?|Mot clé pour </br>requête de routage|
| --- | --- | --- | --- |
| message-id |Identificateur correspondant au message défini par l’utilisateur utilisé pour les modèles demande-réponse. Format: Une chaîne qui respecte la casse (jusqu’à 128 caractères) de caractères alphanumériques 7 bits ASCII + `{'-', ':', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '$', '''}`.  | Oui | messageId |
| iothub-enqueuedtime |Date et heure de réception du message [Appareil-à-cloud](iot-hub-devguide-d2c-guidance.md) par IoT Hub. | Non | enqueuedTime |
| user-id |Un ID utilisé pour spécifier l’origine des messages. Lorsque des messages sont générés par IoT Hub, la propriété est définie sur `{iot hub name}`. | Oui | userId |
| iothub-connection-device-id |Un ID défini par IoT Hub sur les messages appareil vers cloud. Elle contient la propriété **deviceId** de l’appareil qui a envoyé le message. | Non | connectionDeviceId |
| iothub-connection-module-id |Un ID défini par IoT Hub sur les messages appareil vers cloud. Il contient la propriété **moduleId** de l’appareil qui a envoyé le message. | Non | connectionModuleId |
| iothub-connection-auth-generation-id |Un ID défini par IoT Hub sur les messages appareil vers cloud. Il contient la propriété **connectionDeviceGenerationId** (conformément aux [Propriétés d’identité des appareils](iot-hub-devguide-identity-registry.md#device-identity-properties)) de l’appareil qui a envoyé le message. | Non |connectionDeviceGenerationId |
| iothub-connection-auth-method |Une méthode d’authentification définie par IoT Hub sur les messages appareil-à-cloud. Cette propriété contient des informations sur la méthode d’authentification utilisée pour authentifier l’appareil qui a envoyé le message.| Non | connectionAuthMethod |
| dt-dataschema | Cette valeur est définie par IoT Hub sur les messages appareil-à-cloud. Elle contient l’ID du modèle d’appareil défini dans la connexion de l’appareil. | Non | N/A |
| dt-subject | Nom du composant qui envoie les messages appareil-à-cloud. | Oui | N/A |

## <a name="system-properties-of-c2d-iot-hub-messages"></a>Propriétés système des messages IoT Hub **C2D**

| Propriété | Description  |L’utilisateur est-il définissable ?|
| --- | --- | --- |
| message-id |Identificateur correspondant au message défini par l’utilisateur utilisé pour les modèles demande-réponse. Format: Une chaîne qui respecte la casse (jusqu’à 128 caractères) de caractères alphanumériques 7 bits ASCII + `{'-', ':', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '$', '''}`.  |Oui|
| sequence-number |Un numéro (unique par file d’attente d’appareil) affecté par IoT Hub à chaque message cloud-à-appareil. |Non|
| to |Une destination spécifiée dans les messages [cloud vers appareil](iot-hub-devguide-c2d-guidance.md) . |Non|
| absolute-expiry-time |Date et heure d’expiration du message. |Non|   |
| correlation-id |Une propriété de chaîne d’un message de réponse qui contient généralement l'ID du message de la demande dans les modèles demande-réponse. |Oui|
| user-id |Un ID utilisé pour spécifier l’origine des messages. Lorsque des messages sont générés par IoT Hub, la propriété est définie sur `{iot hub name}`. |Oui|
| iothub-ack |Un générateur de messages de commentaires. Cette propriété est utilisée dans les messages cloud-à-appareil pour demander à IoT Hub de générer des messages de commentaires à la suite de la consommation du message par l’appareil. Valeurs possibles : **none** (par défaut) : aucun message de commentaires n’est généré ; **positive** : recevoir un message de commentaires si le message est achevé ; **negative** : recevoir un message de commentaires si le message a expiré (ou si le nombre maximal de remises a été atteint) sans être achevé par l’appareil, ou **full** : propriétés à la fois positive et négative. |Oui|

### <a name="system-property-names"></a>Noms des propriétés système

Les noms de propriété système varient en fonction du point de terminaison vers lequel les messages sont acheminés. Pour plus d’informations sur ces noms, consultez le tableau ci-dessous.

|Nom de propriété système|Event Hubs|Stockage Azure|Service Bus|Event Grid|
|--------------------|----------|-------------|-----------|----------|
|ID de message|message-id|messageId|MessageId|message-id|
|ID d’utilisateur|user-id|userId|UserId|user-id|
|ID de l’appareil de connexion|iothub-connection-device-id| connectionDeviceId|iothub-connection-device-id|iothub-connection-device-id|
|ID du module de connexion|iothub-connection-module-id|connectionModuleId|iothub-connection-module-id|iothub-connection-module-id|
|ID de génération d’authentification de la connexion|iothub-connection-auth-generation-id|connectionDeviceGenerationId| iothub-connection-auth-generation-id|iothub-connection-auth-generation-id|
|Méthode d’authentification de la connexion|iothub-connection-auth-method|connectionAuthMethod|iothub-connection-auth-method|iothub-connection-auth-method|
|contentType|content-type|contentType|ContentType|iothub-content-type|
|contentEncoding|content-encoding|contentEncoding|ContentEncoding|iothub-content-encoding|
|iothub-enqueuedtime|iothub-enqueuedtime|enqueuedTime| N/A |iothub-enqueuedtime|
|CorrelationId|correlation-id|correlationId|CorrelationId|correlation-id|
|dt-dataschema|dt-dataschema|dt-dataschema|dt-dataschema|dt-dataschema|
|dt-subject|dt-subject|dt-subject|dt-subject|dt-subject|

## <a name="message-size"></a>Taille des messages

IoT Hub mesure la taille des messages d’une façon indépendante du protocole, en considérant uniquement la charge utile réelle. La taille en octets est calculée comme la somme des valeurs suivantes :

* La taille du corps en octets.
* La taille en octets de toutes les valeurs des propriétés système du message.
* La taille en octets de tous les noms et valeurs des propriétés des utilisateurs.

Les noms et les valeurs des propriétés sont limités aux caractères ASCII ; ainsi, la longueur des chaînes est égale à la taille en octets.

## <a name="anti-spoofing-properties"></a>Propriétés de détection d’usurpation d’identité

Pour éviter l’usurpation d’appareil dans les messages appareil-à-cloud, IoT Hub marque tous les messages avec les propriétés suivantes :

* **iothub-connection-device-id**
* **iothub-connection-auth-generation-id**
* **iothub-connection-auth-method**

Les deux premières propriétés contiennent le **deviceId** et le **generationId** de l’appareil d’origine, conformément aux [Propriétés d’identité des appareils](iot-hub-devguide-identity-registry.md#device-identity-properties).

La propriété **iothub-connection-auth-method** contient un objet sérialisé JSON avec les propriétés suivantes :

```json
{
  "scope": "{ hub | device }",
  "type": "{ symkey | sas | x509 }",
  "issuer": "iothub"
}
```

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les limites de taille des messages dans IoT Hub, consultez [Quotas et limitations IoT Hub](iot-hub-devguide-quotas-throttling.md).

* Pour découvrir comment créer et lire des messages IoT Hub dans différents langages de programmation, consultez les [Démarrages rapides](quickstart-send-telemetry-node.md).