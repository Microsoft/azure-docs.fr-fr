---
title: Comprendre le registre des identités d’Azure IoT Hub | Microsoft Docs
description: Guide du développeur - Description du registre des identités IoT Hub et de la manière de l’utiliser pour gérer vos appareils. Contient des informations sur l’importation et l’exportation d’identités d’appareils en bloc.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/29/2018
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 2d9b0d97fa1823314f5109a1c7fc79054806c148
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93146924"
---
# <a name="understand-the-identity-registry-in-your-iot-hub"></a>Comprendre le registre des identités dans votre IoT Hub

Chaque hub IoT a un registre des identités contenant des informations sur les appareils et modules autorisés à se connecter au hub IoT. Pour qu’un appareil ou module puisse se connecter à un hub IoT, une entrée correspondant à cet appareil ou module doit figurer dans le registre des identités du hub IoT. Un appareil ou module doit également s’authentifier auprès du hub IoT à l’aide des informations d’identification stockées dans le registre des identités.

L’ID d’appareil ou de module stocké dans le registre des identités respecte la casse.

À un niveau supérieur, le registre des identités est une collection compatible REST de ressources d’identité d’appareil ou de module. Lorsque vous ajoutez une entrée au registre des identités, IoT Hub crée un jeu de ressources par appareil, comme une file d’attente contenant des messages cloud vers appareil en transit.

Utilisez le registre des identités lorsque vous avez besoin d’effectuer les actions suivantes :

* Provisionner des appareils ou modules qui se connectent à votre hub IoT.
* Contrôler l’accès par appareil/module aux points de terminaison côté appareil ou module de votre hub.

> [!NOTE]
> * Le registre des identités ne contient pas de métadonnées spécifiques de l’application.
> * L’identité de module et le jumeau de module sont disponibles en préversion publique. Les fonctionnalités ci-après seront prises en charge sur l’identité de module quand il sera en disponibilité générale.
>

## <a name="identity-registry-operations"></a>Opérations du registre d’identité

Le registre des identités IoT Hub expose les opérations suivantes :

* Création d’une identité d’appareil ou de module
* Mise à jour de l’identité d’appareil ou de module
* Récupération d’une identité d’appareil ou de module par ID
* Suppression de l’identité d’appareil ou de module
* Création de listes contenant jusqu’à 1 000 identités
* Exportation des identités d’appareil vers le stockage Blob Azure
* Importation des identités d’appareil depuis le stockage Blob Azure

Toutes ces opérations peuvent utiliser un accès concurrentiel optimiste, comme spécifié dans la [RFC7232](https://tools.ietf.org/html/rfc7232).

> [!IMPORTANT]
> La seule façon de récupérer toutes les identités dans un registre d’identités IoT Hub consiste à utiliser la fonctionnalité [Exporter](iot-hub-devguide-identity-registry.md#import-and-export-device-identities).

Un registre des identités IoT Hub :

* ne contient pas de métadonnées de l’application ;
* est accessible en tant que dictionnaire à l’aide de la clé **deviceId** ou **moduleId**.
* ne prend pas en charge les requêtes expressives.

Une solution IoT possède généralement une zone de stockage distincte spécifique à la solution qui contient les métadonnées propres à l’application. Dans une solution de développement intelligente, par exemple, la zone de stockage spécifique à la solution doit enregistrer l’espace dans lequel un capteur de température sera déployé.

> [!IMPORTANT]
> Utilisez le registre des identités uniquement pour les opérations de gestion et d’approvisionnement. Les opérations à haut débit ne doivent pas dépendre de l’exécution d’opérations dans le registre des identités au moment de leur exécution. Par exemple, la vérification de l’état de la connexion d’un appareil avant l’envoi d’une commande n’est pas un modèle pris en charge. Veillez à vérifier les [taux de limitation](iot-hub-devguide-quotas-throttling.md) du registre d’identités et le modèle de [pulsation de l’appareil](iot-hub-devguide-identity-registry.md#device-heartbeat).

## <a name="disable-devices"></a>Désactivation d’appareils

Vous pouvez désactiver les appareils en mettant à jour la propriété **status** d’une identité dans le registre des identités. Généralement, cette propriété est utilisée dans deux scénarios :

* Au cours d’un processus d’orchestration d’approvisionnement. Pour plus d’informations, consultez [Provisionnement des appareils](iot-hub-devguide-identity-registry.md#device-provisioning).

* Si, pour une raison quelconque, vous pensez qu’un appareil est compromis ou non autorisé.

Cette fonctionnalité n’est pas disponible pour les modules.

## <a name="import-and-export-device-identities"></a>Importer et exporter les identités des appareils

Utilisez des opérations asynchrones sur le [point de terminaison du fournisseur de ressources IoT Hub](iot-hub-devguide-endpoints.md) pour exporter des identités d’appareils en bloc à partir du registre des identités du hub IoT. Les exportations sont des tâches à long terme qui utilisent un conteneur d’objets blob fourni par le client pour enregistrer les données relatives à l’identité des appareils lues dans le registre des identités.

Utilisez des opérations asynchrones sur le [point de terminaison du fournisseur de ressources IoT Hub](iot-hub-devguide-endpoints.md) pour importer des identités d’appareils en bloc dans le registre des identités du hub IoT. Les importations sont des tâches à long terme qui utilisent des données dans un conteneur d’objets blob, fourni par le client, pour écrire les données relatives à l’identité des appareils dans le registre des identités.

Pour plus d’informations sur l’importation et l’exportation d’API, consultez [API REST du fournisseur de ressources IoT Hub](/rest/api/iothub/iothubresource). Pour en savoir plus sur l’exécution des travaux d’importation et d’exportation, consultez [Gestion en bloc des identités d’appareils IoT Hub](iot-hub-bulk-identity-mgmt.md).

Les identités des appareils peuvent également être exportées et importées à partir d’un hub IoT via l’API de service par le biais de l’[API REST](/rest/api/iothub/service/jobs/createimportexportjob) ou de l’un des [SDK de service](./iot-hub-devguide-sdks.md#azure-iot-hub-service-sdks) IoT Hub.

## <a name="device-provisioning"></a>Approvisionnement des appareils

Les données d’appareil qu’une solution IoT donnée stocke dépendent des exigences spécifiques de cette solution. Mais une solution doit au minimum stocker les clés d’authentification et les identités des appareils. Azure IoT Hub inclut un registre d’identité qui peut stocker des valeurs pour chaque appareil, comme les ID, les clés d’authentification et les codes d’état. Une solution peut utiliser d’autres services Azure, tels que le stockage de tables, le stockage Blob ou Cosmos DB pour stocker des données d’appareil supplémentaires.

*Approvisionnement des appareils* est le processus d'ajout des données d'appareil initial aux magasins dans votre solution. Pour permettre à un nouvel appareil de se connecter à votre hub, vous devez ajouter un ID et des clés d’appareil au registre des identités d’IoT Hub. Dans le cadre du processus d’approvisionnement, vous devrez peut-être initialiser les données spécifiques à l’appareil dans d’autres magasins de la solution. Vous pouvez également utiliser le service de d’approvisionnement des appareils IoT Hub pour activer l’approvisionnement sans contact et juste-à-temps vers un ou plusieurs hubs IoT sans qu’une intervention humaine soit nécessaire. Pour plus d’informations, consultez la [documentation relative au service d’approvisionnement](https://azure.microsoft.com/documentation/services/iot-dps).

## <a name="device-heartbeat"></a>Pulsation des appareils

Le registre des identités IoT Hub contient un champ appelé **connectionState**. Vous ne devez utiliser le champ **connectionState** que pendant le développement et le débogage. Les solutions IoT ne doivent pas interroger le champ au moment de l’exécution. Par exemple, n’interrogez pas le champ **connectionState** pour vérifier si un appareil est connecté avant d’envoyer un message cloud vers appareil ou un SMS. Nous vous recommandons de vous abonner à [l’événement **Appareil déconnecté**](iot-hub-event-grid.md#event-types) sur Event Grid pour recevoir des alertes et superviser l’état de la connexion de l’appareil. Suivez ce [tutoriel](iot-hub-how-to-order-connection-state-events.md) pour savoir comment intégrer les événements Appareil connecté et Appareil déconnecté d’IoT Hub à votre solution IoT.

Si votre solution IoT a besoin de savoir si un appareil est connecté, vous pouvez implémenter le *modèle de pulsation*.
Dans le modèle par pulsations, l’appareil envoie des messages appareil-à-cloud au moins une fois par durée fixe (par exemple, au moins une fois par heure). Ainsi, même si un appareil n’a pas de données à envoyer, il envoie toujours un message appareil-à-cloud vide (généralement avec une propriété qui l’identifie comme pulsation). Côté service, la solution gère un mappage avec la dernière pulsation reçue pour chaque appareil. Si la solution ne reçoit pas de message de pulsation dans le temps imparti de la part de l’appareil, elle suppose qu’il y a un problème avec l’appareil.

Une implémentation plus complexe peut inclure les informations d’[Azure Monitor](../azure-monitor/index.yml) et d’[Azure Resource Health](../service-health/resource-health-overview.md) pour identifier les appareils qui ne parviennent pas à se connecter ou à communiquer. Pour plus d’informations, consultez [Analyser IoT Hub](monitor-iot-hub.md) et [Vérifier l’intégrité des ressources IoT Hub](iot-hub-azure-service-health-integration.md#check-health-of-an-iot-hub-with-azure-resource-health). Quand vous implémentez le modèle par pulsations, veillez à vérifier les [quotas et limitations IoT Hub](iot-hub-devguide-quotas-throttling.md).

> [!NOTE]
> Si une solution IoT utilise uniquement l’état de la connexion de l’appareil pour déterminer si elle doit envoyer des messages cloud vers appareil, et que ces messages ne sont pas diffusés à de larges groupes d’appareils, envisagez d’utiliser un modèle de *délai d’expiration court* plus simple. Ce modèle permet d’obtenir le même résultat qu’en maintenant l’état de la connexion de l’appareil avec sa pulsation, tout en étant plus efficace. Si vous demandez des accusés de réception des messages, IoT Hub peut vous informer sur les appareils en mesure de recevoir des messages et ceux qui ne le sont pas.

## <a name="device-and-module-lifecycle-notifications"></a>Notifications de cycle de vie des appareils et des modules

IoT Hub peut avertir votre solution IoT quand une identité est créée ou supprimée, en envoyant des notifications de cycle de vie. Pour ce faire, votre solution IoT doit créer un itinéraire et définir la source de données *DeviceLifecycleEvents* ou *ModuleLifecycleEvents*. Par défaut, aucune notification du cycle de vie n’est envoyée. Autrement dit, aucun itinéraire n’existe préalablement. Le message de notification inclut le corps et les propriétés.

Propriétés : les propriétés système du message ont pour préfixe le symbole `$`.

Message de notification pour l’appareil :

| Nom | Valeur |
| --- | --- |
|$content-type | application/json |
|$iothub-enqueuedtime |  Heure d’envoi de la notification |
|$iothub-message-source | deviceLifecycleEvents |
|$content-encoding | utf-8 |
|opType | **createDeviceIdentity** ou **deleteDeviceIdentity** |
|hubName | Nom de l’IoT Hub |
|deviceId | ID de l’appareil |
|operationTimestamp | Horodatage ISO8601 de l’opération |
|iothub-message-schema | deviceLifecycleNotification |

Corps : cette section est au format JSON et représente le double de l’identité d’appareil créé. Par exemple,

```json
{
    "deviceId":"11576-ailn-test-0-67333793211",
    "etag":"AAAAAAAAAAE=",
    "properties": {
        "desired": {
            "$metadata": {
                "$lastUpdated": "2016-02-30T16:24:48.789Z"
            },
            "$version": 1
        },
        "reported": {
            "$metadata": {
                "$lastUpdated": "2016-02-30T16:24:48.789Z"
            },
            "$version": 1
        }
    }
}
```
Message de notification pour le module :

| Nom | Valeur |
| --- | --- |
$content-type | application/json |
$iothub-enqueuedtime |  Heure d’envoi de la notification |
$iothub-message-source | moduleLifecycleEvents |
$content-encoding | utf-8 |
opType | **createModuleIdentity** ou **deleteModuleIdentity** |
hubName | Nom de l’IoT Hub |
moduleId | ID du module |
operationTimestamp | Horodatage ISO8601 de l’opération |
iothub-message-schema | moduleLifecycleNotification |

Corps : cette section est au format JSON et représente le jumeau de l’identité de module créé. Par exemple,

```json
{
    "deviceId":"11576-ailn-test-0-67333793211",
    "moduleId":"tempSensor",
    "etag":"AAAAAAAAAAE=",
    "properties": {
        "desired": {
            "$metadata": {
                "$lastUpdated": "2016-02-30T16:24:48.789Z"
            },
            "$version": 1
        },
        "reported": {
            "$metadata": {
                "$lastUpdated": "2016-02-30T16:24:48.789Z"
            },
            "$version": 1
        }
    }
}
```

## <a name="device-identity-properties"></a>Propriétés d’identité des appareils

Les identités des appareils sont représentées sous forme de documents JSON avec les propriétés suivantes :

| Propriété | Options | Description |
| --- | --- | --- |
| deviceId |obligatoire, en lecture seule sur les mises à jour |Une chaîne qui respecte la casse (jusqu’à 128 caractères) de caractères alphanumériques 7 bits ASCII plus certains caractères spéciaux :`- . + % _ # * ? ! ( ) , : = @ $ '`. |
| generationId |obligatoire, en lecture seule |Une chaîne qui respecte la casse, générée par IoT Hub, d’une longueur maximale de 128 caractères. Cette valeur permet de distinguer les appareils dotés du même **deviceId** lorsqu’ils ont été supprimés et recréés. |
| etag |obligatoire, en lecture seule |Une chaîne représentant un ETag faible pour l’identité de l’appareil, conformément à la [RFC7232](https://tools.ietf.org/html/rfc7232). |
| auth |facultatif |Un objet composite contenant des informations d’authentification et des éléments de sécurité. |
| auth.symkey |facultatif |Un objet composite contenant une clé primaire et une clé secondaire, stockées au format base64. |
| status |obligatoire |Un indicateur d’accès. Peut être **Activé** ou **Désactivé**. Si la propriété est définie sur **Activé** , l’appareil est autorisé à se connecter. Si la propriété est définie sur **Désactivé** , cet appareil ne peut pas accéder à un point de terminaison de l’appareil. |
| statusReason |facultatif |Une chaîne de 128 caractères qui stocke le motif de l’état de l’identité de l’appareil. Tous les caractères UTF-8 sont autorisés. |
| statusUpdateTime |en lecture seule |Un indicateur temporel, indiquant la date et l’heure de la dernière mise à jour de l’état. |
| connectionState |en lecture seule |Un champ indiquant l’état de la connexion : **Connecté** ou **Déconnecté**. Ce champ représente la vue IoT Hub de l’état de connexion de l’appareil. **Important** : ce champ doit être utilisé uniquement à des fins de développement et de débogage. L’état de la connexion est mis à jour uniquement pour les appareils utilisant les protocoles AMQP ou MQTT. Cet état est basé sur les pings au niveau du protocole (tests ping MQTT ou AMQP) et peut avoir un délai maximum de 5 minutes seulement. Pour ces raisons, de faux positifs peuvent survenir. Par exemple : un appareil peut être signalé comme étant connecté, alors qu’il est déconnecté. |
| connectionStateUpdatedTime |en lecture seule |Un indicateur temporel, indiquant la date et la dernière heure de mise à jour de l’état de la connexion. |
| lastActivityTime |en lecture seule |Un indicateur temporel, indiquant la date et la dernière heure de connexion de l’appareil, de réception d’un message ou d’envoi d’un message. Cette propriété est finalement cohérente, mais peut être retardée pendant 5 à 10 minutes. Pour cette raison, elle ne doit pas être utilisée dans les scénarios de production. |

> [!NOTE]
> L’état de la connexion peut uniquement représenter la vue IoT Hub de l’état de la connexion. Les mises à jour à cet état peuvent être différées en fonction des conditions et des configurations du réseau.

> [!NOTE]
> Les SDK d’appareil ne prennent pas en charge l’utilisation des caractères `+` et `#` dans le **deviceId**.

## <a name="module-identity-properties"></a>Propriétés d’identité des modules

Les identités des modules sont représentées sous forme de documents JSON avec les propriétés suivantes :

| Propriété | Options | Description |
| --- | --- | --- |
| deviceId |obligatoire, en lecture seule sur les mises à jour |Une chaîne qui respecte la casse (jusqu’à 128 caractères) de caractères alphanumériques 7 bits ASCII plus certains caractères spéciaux :`- . + % _ # * ? ! ( ) , : = @ $ '`. |
| moduleId |obligatoire, en lecture seule sur les mises à jour |Une chaîne qui respecte la casse (jusqu’à 128 caractères) de caractères alphanumériques 7 bits ASCII plus certains caractères spéciaux :`- . + % _ # * ? ! ( ) , : = @ $ '`. |
| generationId |obligatoire, en lecture seule |Une chaîne qui respecte la casse, générée par IoT Hub, d’une longueur maximale de 128 caractères. Cette valeur permet de distinguer les appareils dotés du même **deviceId** lorsqu’ils ont été supprimés et recréés. |
| etag |obligatoire, en lecture seule |Une chaîne représentant un ETag faible pour l’identité de l’appareil, conformément à la [RFC7232](https://tools.ietf.org/html/rfc7232). |
| auth |facultatif |Un objet composite contenant des informations d’authentification et des éléments de sécurité. |
| auth.symkey |facultatif |Un objet composite contenant une clé primaire et une clé secondaire, stockées au format base64. |
| status |obligatoire |Un indicateur d’accès. Peut être **Activé** ou **Désactivé**. Si la propriété est définie sur **Activé** , l’appareil est autorisé à se connecter. Si la propriété est définie sur **Désactivé** , cet appareil ne peut pas accéder à un point de terminaison de l’appareil. |
| statusReason |facultatif |Une chaîne de 128 caractères qui stocke le motif de l’état de l’identité de l’appareil. Tous les caractères UTF-8 sont autorisés. |
| statusUpdateTime |en lecture seule |Un indicateur temporel, indiquant la date et l’heure de la dernière mise à jour de l’état. |
| connectionState |en lecture seule |Un champ indiquant l’état de la connexion : **Connecté** ou **Déconnecté**. Ce champ représente la vue IoT Hub de l’état de connexion de l’appareil. **Important** : ce champ doit être utilisé uniquement à des fins de développement et de débogage. L’état de la connexion est mis à jour uniquement pour les appareils utilisant les protocoles AMQP ou MQTT. Cet état est basé sur les pings au niveau du protocole (tests ping MQTT ou AMQP) et peut avoir un délai maximum de 5 minutes seulement. Pour ces raisons, de faux positifs peuvent survenir. Par exemple : un appareil peut être signalé comme étant connecté, alors qu’il est déconnecté. |
| connectionStateUpdatedTime |en lecture seule |Un indicateur temporel, indiquant la date et la dernière heure de mise à jour de l’état de la connexion. |
| lastActivityTime |en lecture seule |Un indicateur temporel, indiquant la date et la dernière heure de connexion de l’appareil, de réception d’un message ou d’envoi d’un message. |

> [!NOTE]
> Les SDK d’appareil ne prennent pas en charge l’utilisation des caractères `+` et `#` dans les **deviceId** et **moduleId**.

## <a name="additional-reference-material"></a>Matériel de référence supplémentaire

Les autres rubriques de référence dans le Guide du développeur IoT Hub comprennent :

* La rubrique [Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md), qui décrit les différents points de terminaison que chaque hub IoT expose pour les opérations d’exécution et de gestion.

* La rubrique [Quotas et limitation IoT Hub](iot-hub-devguide-quotas-throttling.md) décrit les quotas et le comportement de limitation qui s’appliquent au service IoT Hub.

* La rubrique [SDK des services et appareils Azure IoT](iot-hub-devguide-sdks.md), qui répertorie les SDK en différents langages que vous pouvez utiliser pour le développement d’applications d’appareil et de service qui interagissent avec IoT Hub.

* La rubrique [Langage de requête IoT Hub](iot-hub-devguide-query-language.md) décrit le langage de requête permettant de récupérer à partir d’IoT Hub des informations sur les jumeaux d’appareil et les travaux.

* La rubrique [Prise en charge de MQTT au niveau d’IoT Hub](iot-hub-mqtt-support.md), qui fournit des informations supplémentaires sur la prise en charge du protocole MQTT par IoT Hub.

## <a name="next-steps"></a>Étapes suivantes

À présent que vous savez comment utiliser le registre des identités IoT Hub, vous serez peut-être intéressé par les rubriques suivantes du Guide du développeur Iot Hub :

* [Contrôler l’accès à IoT Hub](iot-hub-devguide-security.md)

* [Utiliser des jumeaux d’appareil pour synchroniser les données d’état et de configuration](iot-hub-devguide-device-twins.md)

* [Appeler une méthode directe sur un appareil](iot-hub-devguide-direct-methods.md)

* [Planifier des travaux sur plusieurs appareils](iot-hub-devguide-jobs.md)

Pour mettre en pratique certains des concepts décrits dans cet article, consultez le didacticiel IoT Hub suivant :

* [Mise en route de Azure IoT Hub](quickstart-send-telemetry-dotnet.md)

Pour savoir comment utiliser le service d’approvisionnement des appareils IoT Hub afin d’activer l’approvisionnement sans contact et juste-à-temps, consultez : 

* [Service Azure IoT Hub Device Provisioning](https://azure.microsoft.com/documentation/services/iot-dps)