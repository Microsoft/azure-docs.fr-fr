---
title: Comprendre la prise en charge du protocole MQTT avec Azure IoT Device Provisioning Service | Microsoft Docs
description: 'Guide du développeur : prise en charge des appareils se connectant à un point de terminaison Azure IoT Device Provisioning Service (DPS) côté appareil en utilisant le protocole MQTT.'
author: rajeevmv
ms.service: iot-dps
services: iot-dps
ms.topic: conceptual
ms.date: 10/16/2019
ms.author: ravokkar
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 0a7ec2f4f8fdf631a6bc5096296275291ec41751
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967123"
---
# <a name="communicate-with-your-dps-using-the-mqtt-protocol"></a>Communiquer avec votre DPS à l’aide du protocole MQTT

Grâce à Device Provisioning Service (DPS), les appareils peuvent communiquer avec le point de terminaison d’appareil DPS à l’aide de :

* [MQTT v3.1.1](https://mqtt.org/) sur le port 8883
* [MQTT v3.1.1](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718127) sur WebSocket sur le port 443.

DPS n’est pas un répartiteur MQTT complet et ne prend pas en charge tous les comportements spécifiés dans la norme MQTT v3.1.1. Cet article décrit comment les appareils peuvent utiliser des comportements MQTT pris en charge pour communiquer avec DPS.

Toutes les communications des appareils avec DPS doivent être sécurisées à l’aide de TLS/SSL. Par conséquent, DPS ne prend pas en charge les connexions non sécurisées sur le port 1883.

 > [!NOTE] 
 > Actuellement, DPS ne prend pas en charge les appareils utilisant le [mécanisme d'attestation](./concepts-service.md#attestation-mechanism) TPM via le protocole MQTT.

## <a name="connecting-to-dps"></a>Connexion à DPS

Un appareil peut utiliser le protocole MQTT pour se connecter à un DPS en utilisant une des options suivantes.

* Bibliothèques des [SDK d’approvisionnement Azure IoT](../iot-hub/iot-hub-devguide-sdks.md#microsoft-azure-provisioning-sdks).
* Protocole MQTT directement.

## <a name="using-the-mqtt-protocol-directly-as-a-device"></a>Utilisation directe du protocole MQTT (en tant qu’appareil)

Si un appareil ne peut pas utiliser les Kits device SDK, il peut toujours se connecter aux points de terminaison d’appareil publics à l’aide du protocole MQTT sur le port 8883. Dans le paquet **CONNECT**, l’appareil doit utiliser les valeurs suivantes :

* Dans le champ **ClientId**, utilisez **registrationId**.

* Dans le champ **Username**, utilisez `{idScope}/registrations/{registration_id}/api-version=2019-03-31`, où `{idScope}` est la valeur [idScope](./concepts-service.md#id-scope) du DPS.

* Dans le champ **Password**, utilisez un jeton SAP. Le format du jeton SAP est identique pour les protocoles HTTPS et AMQP :

  `SharedAccessSignature sr={URL-encoded-resourceURI}&sig={signature-string}&se={expiry}&skn=registration` resourceURI doit respecter le format `{idScope}/registrations/{registration_id}`. Le nom de la stratégie doit être `registration`.

  > [!NOTE]
  > Si vous utilisez l’authentification par certificat X.509, les mots de passe de jeton SAS ne sont pas obligatoires.

  Pour plus d'informations sur la génération de jetons SAS, voir la section consacrée aux jetons de sécurité dans [Contrôler l’accès à DPS](how-to-control-access.md#security-tokens).

Voici une liste de comportements spécifiques à l’implémentation de DPS :

 * DPS ne prend pas en charge la fonctionnalité de l'indicateur **CleanSession** dont la valeur est **0**.

 * Quand une application d’appareil s’abonne à une rubrique avec **QoS 2**, DPS accorde le niveau QoS 1 maximal dans le paquet **SUBACK**. Après cela, DPS remet les messages à l’appareil à l’aide de QoS 1.

## <a name="tlsssl-configuration"></a>Configuration TLS/SSL

Pour utiliser le protocole MQTT directement, votre client *doit* se connecter via le protocole TLS 1.2. Sans cette étape, des erreurs de connexion se produiront.


## <a name="registering-a-device"></a>Inscription d’un appareil

Pour inscrire un appareil via DPS, l’appareil doit s’abonner en utilisant un `$dps/registrations/res/#` en tant que **Filtre de rubrique**. Le caractère générique à plusieurs niveaux `#` dans le Filtre de rubrique est utilisé uniquement pour autoriser l’appareil à recevoir des propriétés supplémentaires dans le nom de la rubrique. DPS n’autorise pas l’utilisation des caractères génériques `#` ou `?` pour filtrer les sous-rubriques. DPS n’étant pas un broker de messagerie pub-sub à usage général, il prend uniquement en charge les noms de rubriques et les filtres de rubriques documentés.

L'appareil doit publier un message de registre sur DPS en utilisant `$dps/registrations/PUT/iotdps-register/?$rid={request_id}` comme **Topic Name**. La charge utile doit contenir l'objet [Device Registration](/rest/api/iot-dps/runtimeregistration/registerdevice#deviceregistration) au format JSON.
Dans un scénario réussi, l’appareil recevra une réponse sur le nom de rubrique `$dps/registrations/res/202/?$rid={request_id}&retry-after=x`, où x est la valeur retry-after en secondes. La charge utile de la réponse contiendra l'objet [RegistrationOperationStatus](/rest/api/iot-dps/runtimeregistration/registerdevice#registrationoperationstatus) au format JSON.

## <a name="polling-for-registration-operation-status"></a>Interrogation de l'état de l'opération d'inscription

L'appareil doit interroger le service périodiquement pour recevoir le résultat de l'opération d'inscription de l'appareil. En supposant que l’appareil est déjà inscrit à la rubrique `$dps/registrations/res/#` comme indiqué ci-dessus, il peut publier un message get operationstatus dans la rubrique `$dps/registrations/GET/iotdps-get-operationstatus/?$rid={request_id}&operationId={operationId}`. L'ID d'opération dans ce message doit être la valeur reçue dans le message de réponse RegistrationOperationStatus à l'étape précédente. En cas de succès, le service répondra dans la rubrique `$dps/registrations/res/200/?$rid={request_id}`. La charge utile de la réponse contiendra l'objet RegistrationOperationStatus. L'appareil doit continuer à interroger le service si le code de réponse est 202 après un délai égal à la période retry-after. L'opération d'inscription de l'appareil est réussie si le service renvoie un code d'état 200.

## <a name="connecting-over-websocket"></a>Connexion via WebSocket
Lors d'une connexion via WebSocket, spécifiez le sous-protocole comme `mqtt`. Suivez [RFC 6455](https://tools.ietf.org/html/rfc6455).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le protocole MQTT, consultez la [documentation de MQTT](https://mqtt.org/).

Pour explorer davantage les capacités de DPS, consultez :

* [À propos d’IoT DPS](about-iot-dps.md)