---
title: Présentation des points de terminaison Azure IoT Hub personnalisés | Microsoft Docs
description: 'Guide du développeur : Utilisation de requêtes de routage pour router les messages appareil-à-cloud vers des points de terminaison.'
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/09/2018
ms.openlocfilehash: 8aebb5b6f6a3ac53bc49fd1d2f75de88667865fb
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92147636"
---
# <a name="use-message-routes-and-custom-endpoints-for-device-to-cloud-messages"></a>Utiliser des itinéraires de messages et des points de terminaison personnalisés pour les messages appareil-à-cloud

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Le [routage des messages](iot-hub-devguide-routing-query-syntax.md) IoT Hub permet aux utilisateurs de router les messages appareil-à-cloud vers les points de terminaison accessibles aux services. Le routage fournit également une fonctionnalité de requête pour filtrer les données avant de les router vers les points de terminaison. Chaque requête de routage que vous configurez a les propriétés suivantes :

| Propriété      | Description |
| ------------- | ----------- |
| **Nom**      | Nom unique qui identifie la requête. |
| **Source**    | Origine du flux de données qui fait l’objet du traitement. Par exemple, télémétrie des appareils. |
| **Condition** | L’expression de requête pour la requête de routage qui est exécutée sur les propriétés de l’application de messages, les propriétés système, le corps des messages, les étiquettes de jumeau d’appareil et les propriétés de jumeau d’appareil, pour déterminer s’il s’agit d’une correspondance pour le point de terminaison. Pour plus d’informations sur la construction d’une requête, consultez la [syntaxe des requêtes de routage des messages](iot-hub-devguide-routing-query-syntax.md) |
| **Point de terminaison**  | Nom du point de terminaison auquel IoT Hub envoie les messages qui correspondent à la requête. Nous vous recommandons de choisir un point de terminaison dans la même région que votre hub IoT. |

Un même message peut correspondre à la condition de plusieurs requêtes de routage, auquel cas IoT Hub remet le message au point de terminaison associé à chaque requête ayant une correspondance. Par ailleurs, IoT Hub déduplique automatiquement la remise des messages. Ainsi, si un message correspond à plusieurs requêtes ayant la même destination, il n’est écrit qu’une seule fois dans cette destination.

## <a name="endpoints-and-routing"></a>Points de terminaison et routage

Un IoT Hub a un [point de terminaison intégré](iot-hub-devguide-messages-read-builtin.md) unique. Vous pouvez créer des points de terminaison personnalisés pour y acheminer les messages en liant d’autres services de votre abonnement au hub. IoT Hub prend actuellement en charge les conteneurs de stockage Azure, les points de terminaison personnalisés Event Hubs, les files d’attente Service Bus et les rubriques Service Bus.

Quand vous utilisez des points de terminaison de routage et personnalisés, les messages sont remis seulement au point de terminaison intégré s’ils ne correspondent à aucune requête. Pour remettre des messages au point de terminaison intégré ainsi qu’à un point de terminaison personnalisé, ajoutez un itinéraire qui envoie des messages au point de terminaison intégré des **événements**.

> [!NOTE]
> * IoT Hub prend uniquement en charge l’écriture de données dans des conteneurs de stockage Azure en tant qu’objets blob.
> * Les files d’attente et rubriques Service Bus dont les options **Sessions** ou **Détection des doublons** sont activées ne sont pas prises en charge comme points de terminaison personnalisés.

Pour plus d’informations sur la création de points de terminaison personnalisés dans IoT Hub, consultez la page [Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md).

Pour plus d’informations sur la lecture à partir de points de terminaison personnalisés, consultez :

* Lecture à partir des [conteneurs de Stockage Azure](../storage/blobs/storage-blobs-introduction.md).

* Lecture à partir d’[Event Hubs](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md).

* Lecture à partir de [files d’attente Service Bus](../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md).

* Lecture à partir de [rubriques Service Bus](../service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les points de terminaison IoT Hub, consultez [Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md).

* Pour plus d’informations sur le langage de requête que vous utilisez pour définir les requêtes de routage, consultez [Syntaxe des requêtes de routage des messages](iot-hub-devguide-routing-query-syntax.md).

* Le didacticiel [Traiter les messages appareil-à-cloud IoT Hub en utilisant des itinéraires](tutorial-routing.md) vous montre comment utiliser des requêtes de routage et des points de terminaison personnalisés.