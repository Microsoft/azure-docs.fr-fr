---
title: Bien comprendre le chargement de fichiers Azure IoT Hub | Microsoft Docs
description: Guide du développeur - utilisez la fonctionnalité de chargement de fichier de IoT Hub pour gérer le chargement de fichiers depuis un appareil vers un conteneur d’objets blob de stockage Azure.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 11/07/2018
ms.custom:
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 3286b464051b8fea88d2797d4f82b20fe432b4b8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90019527"
---
# <a name="upload-files-with-iot-hub"></a>Chargement de fichiers avec IoT Hub

Comme nous l’avons expliqué dans l’article [Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md), un appareil peut lancer le chargement de fichiers en envoyant une notification par le biais d’un point de terminaison orienté appareil ( **/devices/{deviceId}/files**). Lorsqu’un appareil indique à IoT Hub la fin d’un chargement, IoT Hub envoie un message de notification de chargement de fichiers via le point de terminaison côté service ( **/messages/servicebound/filenotifications**).

Au lieu de distribuer les messages via sa propre plate-forme, IoT Hub joue le rôle de répartiteur vers un compte Stockage Azure associé. Un appareil demande à IoT Hub un jeton de stockage spécifique au fichier que l’appareil souhaite télécharger. L’appareil utilise l’URI SAP pour télécharger le fichier vers le stockage. Une fois le téléchargement terminé, l’appareil envoie une notification à IoT Hub pour l’en informer. IoT Hub vérifie que le chargement de fichiers est terminé, puis ajoute un message de notification de chargement de fichiers au point de terminaison côté service dédié à la notification de fichiers.

Avant de charger un fichier sur IoT Hub à partir d’un appareil, vous devez configurer votre hub en [l’associant à un compte de stockage Azure](iot-hub-devguide-file-upload.md#associate-an-azure-storage-account-with-iot-hub).

Votre appareil peut ensuite [initialiser un chargement](iot-hub-devguide-file-upload.md#initialize-a-file-upload) et [notifier IoT Hub](iot-hub-devguide-file-upload.md#notify-iot-hub-of-a-completed-file-upload) quand le chargement est terminé. Éventuellement, lorsqu’un appareil notifie IoT Hub que le chargement est terminé, le service peut générer un [message de notification](iot-hub-devguide-file-upload.md#file-upload-notifications).

[!INCLUDE [iot-hub-include-x509-ca-signed-file-upload-support-note](../../includes/iot-hub-include-x509-ca-signed-file-upload-support-note.md)]

### <a name="when-to-use"></a>Quand l’utiliser

Utilisez le chargement des fichiers pour envoyer des fichiers multimédias et de gros traitements télémétriques par lots chargés par des appareils connectés par intermittence ou compressés pour économiser de la bande passante.

Reportez-vous à [l’aide sur la communication appareil-à-cloud](iot-hub-devguide-d2c-guidance.md) si vous hésitez entre l’utilisation des propriétés signalées, l’utilisation des messages appareil-à-cloud ou l’utilisation du chargement de fichiers.

## <a name="associate-an-azure-storage-account-with-iot-hub"></a>Association d’un compte Stockage Azure à IoT Hub

Pour utiliser la fonctionnalité de téléchargement de fichier, vous devez d’abord lier un compte Stockage Azure à IoT Hub. Vous pouvez effectuer cette tâche via le portail Azure, ou par programmation avec les [API REST du fournisseur de ressources IoT Hub](/rest/api/iothub/iothubresource). Une fois que vous avez associé un compte Stockage Azure à IoT Hub, le service retourne un URI SAP vers un appareil lorsque ce dernier démarre une demande de téléchargement de fichier.

Les guides pratiques [Télécharger des fichiers de votre appareil vers le cloud avec IoT Hub](iot-hub-csharp-csharp-file-upload.md) fournissent la description complète du processus de chargement de fichier. Ces guides pratiques vous montrent comment utiliser le portail Azure pour associer un compte de stockage à un hub IoT.

> [!NOTE]
> Les kits [Azure IoT SDK](iot-hub-devguide-sdks.md) gèrent automatiquement la récupération de l’URI SAP, le chargement du fichier et l’envoi d’une notification à IoT Hub de la fin du chargement.

## <a name="initialize-a-file-upload"></a>Initialiser un téléchargement de fichier
IoT Hub a un point de terminaison spécifique aux appareils pour demander une URI SAS pour le stockage afin de télécharger un fichier. Pour démarrer le processus de chargement de fichiers, l’appareil envoie une requête POST à `{iot hub}.azure-devices.net/devices/{deviceId}/files` avec le corps JSON suivant :

```json
{
    "blobName": "{name of the file for which a SAS URI will be generated}"
}
```

IoT Hub renvoie les données suivantes. L’appareil l’utilise pour télécharger le fichier :

```json
{
    "correlationId": "somecorrelationid",
    "hostName": "yourstorageaccount.blob.core.windows.net",
    "containerName": "testcontainer",
    "blobName": "test-device1/image.jpg",
    "sasToken": "1234asdfSAStoken"
}
```

### <a name="deprecated-initialize-a-file-upload-with-a-get"></a>Déconseillé : initialiser un téléchargement de fichier avec une commande GET

> [!NOTE]
> Cette section décrit les fonctionnalités déconseillées pour la réception d’une URI SAS d’IoT Hub. Utilisez la méthode POST décrite précédemment.

IoT Hub utilise deux points de terminaison REST pour prendre en charge le téléchargement de fichier, le premier afin d’obtenir l’URI SAP pour le stockage et le second pour informer IoT hub de la fin du téléchargement. L’appareil démarre le processus de téléchargement de fichier en envoyant une commande GET à IoT Hub à `{iot hub}.azure-devices.net/devices/{deviceId}/files/{filename}`. L’IoT Hub renvoie :

* Un URI SAS spécifique au fichier à charger.

* Un ID de corrélation à utiliser une fois le chargement terminé.

## <a name="notify-iot-hub-of-a-completed-file-upload"></a>Notifier IoT Hub de la fin du téléchargement d’un fichier

L’appareil charge le fichier dans le stockage à l’aide des kits SDK Stockage Azure. Une fois le chargement terminé, l’appareil envoie une requête POST à `{iot hub}.azure-devices.net/devices/{deviceId}/files/notifications` avec le corps JSON suivant :

```json
{
    "correlationId": "{correlation ID received from the initial request}",
    "isSuccess": bool,
    "statusCode": XXX,
    "statusDescription": "Description of status"
}
```

La valeur de `isSuccess` est une valeur booléenne qui indique si le fichier a bien été téléchargé. Le code d’état de `statusCode` est l’état pour le téléchargement du fichier vers le stockage et `statusDescription` correspond à `statusCode`.

## <a name="reference-topics"></a>Rubriques de référence :

Les rubriques de référence suivantes vous fournissent des informations supplémentaires sur le téléchargement de fichiers depuis un appareil.

## <a name="file-upload-notifications"></a>Notifications de téléchargement de fichier

Éventuellement, lorsqu’un appareil notifie IoT Hub que le chargement est terminé, le service peut générer un message de notification. Ce message contient l’emplacement de stockage et le nom du fichier.

Comme l’explique la section [Points de terminaison](iot-hub-devguide-endpoints.md), IoT Hub émet des notifications de chargement de fichiers sous forme de messages par le biais d’un point de terminaison orienté service ( **/messages/servicebound/fileuploadnotifications**). La sémantique de réception des notifications de chargement de fichiers est identique à celle des messages cloud-à-appareil et présente le même [cycle de vie des messages](iot-hub-devguide-messages-c2d.md#the-cloud-to-device-message-life-cycle). Chaque message récupéré à partir du point de terminaison de notification de téléchargement de fichier est un enregistrement JSON qui possède les propriétés suivantes :

| Propriété | Description |
| --- | --- |
| EnqueuedTimeUtc |Horodatage indiquant la date et l’heure de création de la notification. |
| deviceId |**DeviceId** de l’appareil qui a téléchargé le fichier. |
| BlobUri |URI du fichier téléchargé. |
| BlobName |Nom du fichier téléchargé. |
| LastUpdatedTime |Horodatage indiquant la date et l’heure de dernière mise à jour du fichier. |
| BlobSizeInBytes |Taille du fichier téléchargé. |

**Exemple**. Voici un exemple illustrant le corps de message de notification de téléchargement de fichier.

```json
{
    "deviceId":"mydevice",
    "blobUri":"https://{storage account}.blob.core.windows.net/{container name}/mydevice/myfile.jpg",
    "blobName":"mydevice/myfile.jpg",
    "lastUpdatedTime":"2016-06-01T21:22:41+00:00",
    "blobSizeInBytes":1234,
    "enqueuedTimeUtc":"2016-06-01T21:22:43.7996883Z"
}
```

## <a name="file-upload-notification-configuration-options"></a>Options de configuration de notification de téléchargement de fichier

Chaque IoT Hub dispose des options de configuration suivantes pour les notifications de chargement de fichier :

| Propriété | Description | Plage et valeur par défaut |
| --- | --- | --- |
| **enableFileUploadNotifications** |Indique si les notifications de téléchargement de fichier sont écrites dans le point de terminaison de notification de fichier. |Valeur booléenne. Valeur par défaut : True. |
| **fileNotifications.ttlAsIso8601** |Durée de vie par défaut des notifications de téléchargement de fichier. |Intervalle ISO_8601 jusqu’à 48h (minimum 1 minute). Valeur par défaut : 1 heure. |
| **fileNotifications.lockDuration** |Durée de verrouillage de la file d’attente des notifications de téléchargement de fichiers. |5 à 300 secondes (5 secondes au minimum). Valeur par défaut : 60 secondes. |
| **fileNotifications.maxDeliveryCount** |Nombre maximal de diffusions pour la file d’attente de notification de téléchargement de fichier. |1 à 100. Valeur par défaut : 100. |

Vous pouvez définir ces propriétés sur votre hub IoT en utilisant le portail Azure, Azure CLI ou PowerShell. Pour savoir comment procéder, consultez les rubriques sous [Configurer le chargement de fichiers](iot-hub-configure-file-upload.md).

## <a name="additional-reference-material"></a>Matériel de référence supplémentaire

Les autres rubriques de référence dans le Guide du développeur IoT Hub comprennent :

* La rubrique [Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md), qui décrit les différents points de terminaison de chaque hub IoT pour les opérations d’exécution et de gestion.

* La rubrique [Quotas et limitation IoT Hub](iot-hub-devguide-quotas-throttling.md) décrit les quotas et le comportement de limitation qui s’appliquent au service IoT Hub.

* La rubrique [SDK des services et appareils Azure IoT](iot-hub-devguide-sdks.md), qui répertorie les SDK en différents langages que vous pouvez utiliser pour le développement d’applications d’appareil et de service qui interagissent avec IoT Hub.

* La rubrique [Langage de requête IoT Hub](iot-hub-devguide-query-language.md) décrit le langage de requête permettant de récupérer à partir d’IoT Hub des informations sur les jumeaux d’appareil et les travaux.

* La rubrique [Prise en charge de MQTT au niveau d’IoT Hub](iot-hub-mqtt-support.md), qui fournit des informations supplémentaires sur la prise en charge du protocole MQTT par IoT Hub.

## <a name="next-steps"></a>Étapes suivantes

À présent que vous savez comment télécharger des fichiers depuis des appareils avec IoT Hub, vous serez peut-être intéressé par les rubriques suivantes du Guide du développeur IoT :

* [Gérer les identités des appareils dans IoT Hub](iot-hub-devguide-identity-registry.md)

* [Contrôler l’accès à IoT Hub](iot-hub-devguide-security.md)

* [Utiliser des jumeaux d’appareil pour synchroniser les données d’état et de configuration](iot-hub-devguide-device-twins.md)

* [Appeler une méthode directe sur un appareil](iot-hub-devguide-direct-methods.md)

* [Planifier des travaux sur plusieurs appareils](iot-hub-devguide-jobs.md)

Pour mettre en pratique certains des concepts décrits dans cet article, consultez le didacticiel IoT Hub suivant :

* [Guide pratique pour charger des fichiers à partir d’appareils vers le cloud avec IoT Hub](iot-hub-csharp-csharp-file-upload.md)