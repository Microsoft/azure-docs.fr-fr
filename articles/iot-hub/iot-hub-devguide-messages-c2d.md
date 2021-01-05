---
title: Présentation de la messagerie de cloud-à-appareil Azure IoT Hub | Microsoft Docs
description: Ce guide du développeur explique comment utiliser la messagerie cloud-à-appareil avec IoT Hub. Il inclut des informations sur le cycle de vie des messages et les options de configuration.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/15/2018
ms.custom: mqtt, devx-track-azurecli
ms.openlocfilehash: ba58f7897827cf7ce7f6156df1434733d89d7f42
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844452"
---
# <a name="send-cloud-to-device-messages-from-an-iot-hub"></a>Envoyer des messages cloud-à-appareil à partir d’IoT Hub

Pour envoyer des notifications unidirectionnelles à l’application pour appareil à partir du back-end de votre solution, envoyez des messages cloud-à-appareil depuis IoT Hub vers votre appareil. Pour une description des autres options cloud-à-appareil prises en charge par Azure IoT Hub, voir [Conseils pour les communications cloud-à-appareil](iot-hub-devguide-c2d-guidance.md).

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Vous envoyez les messages cloud-à-appareil via un point de terminaison côté service, appelé */messages/devicebound*. Un appareil reçoit ensuite les messages via un point de terminaison spécifique de l’appareil ( */devices/{IdAppareil}/messages/devicebound*).

Pour cibler chaque message cloud-à-appareil sur un seul appareil, IoT Hub définit la propriété **to** sur */devices/{IdAppareil}/messages/devicebound*.

Chaque file d’attente d’appareil peut contenir jusqu’à 50 messages cloud-à-appareil. Les tentatives d’envoi d’un nombre supérieur de messages au même appareil entraînent une erreur.

## <a name="the-cloud-to-device-message-life-cycle"></a>Cycle de vie des messages cloud-à-appareil

Pour garantir la remise des messages au moins une fois, IoT Hub conserve les messages cloud-à-appareil dans des files d’attente par appareil. Les appareils doivent explicitement confirmer *l’achèvement* afin qu’IoT Hub supprime les messages de la file d’attente. Cette approche garantit la résilience contre les échecs de connectivité et d’appareils.

Le graphique de l’état du cycle de vie s’affiche dans le diagramme suivant :

![Cycle de vie des messages cloud-à-appareil](./media/iot-hub-devguide-messages-c2d/lifecycle.png)

Quand le service IoT Hub envoie un message à un appareil, il définit l’état du message sur *Enqueued* (En file attente). Lorsqu’un appareil veut *recevoir* un message, IoT Hub *verrouille* ce message en définissant l’état sur *Invisible*. Cet état permet aux autres threads sur l’appareil à commencer à recevoir d’autres messages. Lorsqu’un thread d’appareil a fini de traiter un message, il notifie IoT Hub en *terminant* le message. Ensuite, IoT Hub définit l’état sur *Completed* (Terminé).

Un appareil peut également :

* *Rejeter* le message, ce qui amène IoT Hub à lui attribuer l’état *Dead lettered* (Mis en file d’attente de lettres mortes). Les appareils qui se connectent via le protocole MQTT (Message Queuing Telemetry Transport) ne peuvent pas rejeter les messages cloud-à-appareil.

* *Abandonner* le message, ce qui amène IoT Hub à replacer ce dernier dans la file d’attente avec l’état *Enqueued*(En file attente). Les appareils qui se connectent via le protocole MQTT ne peuvent pas abandonner les messages cloud-à-appareil.

Il est possible qu’un thread ne parvienne pas à traiter un message, mais n’en avertisse pas IoT Hub. Dans ce cas, les messages passent automatiquement de l’état *Invisible* à l’état *Enqueued* (En file d’attente) après un délai d’attente de *visibilité* (ou de *verrouillage*). La valeur de ce délai d’attente est d’une minute et ne peut pas être modifiée.

La propriété **Nombre maximal de remises** sur IoT Hub détermine le nombre maximal de fois qu’un message peut passer de l’état *Enqueued* (En file d’attente) à l’état *Invisible*, et inversement. Une fois ce nombre de transitions atteint, IoT Hub attribue au message l’état *Dead lettered* (Mis en file d’attente de lettres mortes). De même, IoT Hub attribue à un message l’état *Dead lettered* (Mis en file d’attente de lettres mortes) à l’issue de son délai d’expiration. Pour en savoir plus, voir [Durée de vie](#message-expiration-time-to-live).

L’article [Envoyer des messages du cloud à votre appareil avec IoT Hub](iot-hub-csharp-csharp-c2d.md) explique comment envoyer des messages cloud-à-appareil depuis le cloud et comment les recevoir sur un appareil.

Généralement, un appareil achève un message cloud-à-appareil lorsque la perte du message n’affecte pas la logique d’application. C’est par exemple le cas quand l’appareil a rendu persistant le contenu du message en local, ou qu’il a exécuté une opération correctement. Le message peut également transporter des informations temporaires, dont la perte éventuelle n’aurait aucun impact sur les fonctionnalités de l’application. Quelquefois, pour les tâches longues, vous pouvez :

* Terminer le message cloud-à-appareil après que l’appareil a rendu persistante la description de la tâche dans le stockage local.

* Notifier le backend de la solution à l’aide d’un ou de plusieurs messages appareil-à-cloud à différents stades de la progression de la tâche.

## <a name="message-expiration-time-to-live"></a>Expiration du message (durée de vie)

Chaque message cloud-à-appareil est doté d’un délai d’expiration. Cette durée est définie par l’un des éléments suivants :

* La propriété **ExpiryTimeUtc** dans le service
* IoT Hub, via l’utilisation de la valeur *Durée de vie* par défaut spécifiée comme propriété IoT Hub

Consultez [Options de configuration Cloud vers appareil](#cloud-to-device-configuration-options).

Un moyen courant de tirer parti de l’expiration du message et d’éviter l’envoi de messages à des appareils déconnectés consiste à définir des valeurs de *durée de vie* courtes. Cette approche produit le même résultat que la gestion de l’état de connexion de l’appareil, mais s’avère plus efficace. Lorsque vous demandez des accusés de réception pour les messages, IoT Hub vous indique quels sont les appareils :

* En mesure de recevoir des messages.
* Qui ne sont pas en ligne ou qui ont échoué.

## <a name="message-feedback"></a>Commentaires de messages

Lorsque vous envoyez un message cloud-à-appareil, le service peut demander la remise d’un commentaire par message concernant l’état final de ce dernier. Pour cela, définissez la propriété d’application **iothub-ack** figurant dans le message cloud-à-appareil qui est envoyé sur l’une des quatre valeurs suivantes :

| Valeur de la propriété Ack | Comportement |
| ------------ | -------- |
| Aucun     | IoT Hub ne génère aucun message de commentaire (comportement par défaut). |
| positif | Si le message cloud-à-appareil prend l’état *Completed* (Terminé), IoT Hub génère un message de commentaire. |
| négatif | Si le message cloud-à-appareil prend l’état *Dead lettered* (Mis en file d’attente de lettres mortes), IoT Hub génère un message de commentaire. |
| complet     | IoT Hub génère un message de commentaires dans les deux cas. |

Si la propriété **Ack** est définie sur *full* et que vous ne recevez pas de message de commentaire, cela signifie que le message de commentaire est arrivé à expiration. Le service ne peut pas savoir ce qui est arrivé au message d’origine. Dans la pratique, un service doit s'assurer qu'il peut traiter les commentaires avant leur expiration. Le délai d’expiration maximal est de deux jours, ce qui vous laisse le temps de faire refonctionner le service en cas de défaillance.

Comme l’explique la section [Points de terminaison](iot-hub-devguide-endpoints.md), IoT Hub fournit des commentaires sous la forme de messages via un point de terminaison accessible au service ( */messages/servicebound/feedback*). La sémantique pour recevoir des commentaires est identique aux messages de cloud-à-appareil. Chaque fois que c’est possible, des commentaires de messages sont mis en lot dans un seul message, au format suivant :

| Propriété     | Description |
| ------------ | ----------- |
| EnqueuedTime | Horodatage indiquant à quel moment le message de commentaires a été reçu par le hub. |
| UserId       | `{iot hub name}` |
| ContentType  | `application/vnd.microsoft.iothub.feedback.json` |

Le corps est un tableau sérialisé JSON des enregistrements, chacun disposant des propriétés suivantes :

| Propriété           | Description |
| ------------------ | ----------- |
| EnqueuedTimeUtc    | Horodatage qui indique quand le résultat du message s’est produit (par exemple, le hub a reçu le message de commentaire, ou le message d’origine est arrivé à expiration) |
| OriginalMessageId  | Valeur *MessageId* du message cloud-à-appareil auquel se rapportent ces informations de commentaire. |
| StatusCode         | Chaîne obligatoire, utilisée dans les messages de commentaire générés par IoT Hub : <br/> *Success* <br/> *Expired* <br/> *DeliveryCountExceeded* <br/> *Rejected* <br/> *Purged* |
| Description        | Valeurs de chaîne pour *StatusCode*. |
| deviceId           | Valeur *DeviceId* de l’appareil cible du message cloud-à-appareil auquel se rapporte ce commentaire. |
| DeviceGenerationId | Valeur *DeviceGenerationId* de l’appareil cible du message cloud-à-appareil auquel se rapporte ce commentaire. |

Le service doit spécifier une valeur *MessageId* pour le message cloud-à-appareil afin de pouvoir mettre en corrélation ses commentaires avec le message d’origine.

Le code suivant montre le corps d’un message de commentaire :

```json
[
  {
    "OriginalMessageId": "0987654321",
    "EnqueuedTimeUtc": "2015-07-28T16:24:48.789Z",
    "StatusCode": 0,
    "Description": "Success",
    "DeviceId": "123",
    "DeviceGenerationId": "abcdefghijklmnopqrstuvwxyz"
  },
  {
    ...
  },
  ...
]
```

**Commentaires en attente pour les appareils supprimés**

Quand un appareil est supprimé, les commentaires en attente sont également supprimés. Les commentaires des appareils sont envoyés par lots. Si un appareil est supprimé dans la fenêtre étroite (souvent inférieure à 1 seconde) entre le moment où l’appareil confirme la réception du message et le moment où le lot de commentaires suivant est préparé, les commentaires ne se produisent pas.

Vous pouvez résoudre ce problème en attendant un certain temps que les commentaires en attente arrivent avant de supprimer votre appareil. Les commentaires des messages associés doivent être présumés perdus une fois qu’un appareil est supprimé.

## <a name="cloud-to-device-configuration-options"></a>Options de configuration cloud-à-appareil

Chaque hub IoT expose les options de configuration suivantes pour la messagerie cloud-à-appareil :

| Propriété                  | Description | Plage et valeur par défaut |
| ------------------------- | ----------- | ----------------- |
| defaultTtlAsIso8601       | Durée de vie par défaut des messages cloud-à-appareil | Intervalle ISO_8601 de 2 jours maximum (minimum 1 minute) ; valeur par défaut : 1 heure |
| maxDeliveryCount          | Nombre de remises maximal pour les files d’attente par appareil cloud-à-appareil | 1 à 100 ; valeur par défaut : 10 |
| feedback.ttlAsIso8601     | Rétention des messages de commentaire liés au service | Intervalle ISO_8601 de 2 jours maximum (minimum 1 minute) ; valeur par défaut : 1 heure |
| feedback.maxDeliveryCount | Nombre de remises maximal pour la file d’attente de commentaires | 1 à 100 ; valeur par défaut : 10 |
| feedback.lockDurationAsIso8601 | Nombre de remises maximal pour la file d’attente de commentaires | Intervalle ISO_8601 de 5 à 300 secondes (minimum 5 secondes) ; valeurs par défaut : 60 secondes. |

Vous pouvez définir les options de configuration de l’une des manières suivantes :

* **Portail Azure**: Sous **Paramètres** dans votre hub IoT, sélectionnez **Points de terminaison intégrés**, puis développez **Messagerie cloud-à-appareil**. (La définition des propriétés **feedback.maxDeliveryCount** et **feedback.lockDurationAsIso8601** n’est actuellement pas prise en charge dans le Portail Azure.)

    ![Définir les options de configuration pour la messagerie cloud-à-appareil dans le portail](./media/iot-hub-devguide-messages-c2d/c2d-configuration-portal.png)

* **Azure CLI** : Utilisez la commande [az iot hub update](/cli/azure/iot/hub?view=azure-cli-latest#az-iot-hub-update) :

    ```azurecli
    az iot hub update --name {your IoT hub name} \
        --set properties.cloudToDevice.defaultTtlAsIso8601=PT1H0M0S

    az iot hub update --name {your IoT hub name} \
        --set properties.cloudToDevice.maxDeliveryCount=10

    az iot hub update --name {your IoT hub name} \
        --set properties.cloudToDevice.feedback.ttlAsIso8601=PT1H0M0S

    az iot hub update --name {your IoT hub name} \
        --set properties.cloudToDevice.feedback.maxDeliveryCount=10

    az iot hub update --name {your IoT hub name} \
        --set properties.cloudToDevice.feedback.lockDurationAsIso8601=PT0H1M0S
    ```

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les Kits de développement logiciel (SDK) que vous pouvez utiliser pour recevoir des messages cloud-à-appareil, voir [Kits Azure IoT SDK](iot-hub-devguide-sdks.md).

Pour essayer de recevoir des messages cloud-à-appareil, consultez le didacticiel [Envoyer des messages du cloud à votre appareil ](iot-hub-csharp-csharp-c2d.md).
