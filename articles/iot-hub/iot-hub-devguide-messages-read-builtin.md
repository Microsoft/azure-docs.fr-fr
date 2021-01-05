---
title: Comprendre le point de terminaison intégré Azure IoT Hub | Microsoft Docs
description: 'Guide du développeur : décrit comment utiliser le point de terminaison intégré et compatible avec Event Hub pour lire des messages appareil-à-cloud.'
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 06/01/2020
ms.custom:
- amqp
- 'Role: Cloud Development'
ms.openlocfilehash: 4bb33721625f4fc752745ce2b43051c90b3aaa74
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92147684"
---
# <a name="read-device-to-cloud-messages-from-the-built-in-endpoint"></a>Lire des messages appareil-à-cloud à partir du point de terminaison intégré

Par défaut, les messages sont acheminés vers le point de terminaison côté service intégré (**messages/events**) compatible avec [Event Hubs](https://azure.microsoft.com/documentation/services/event-hubs/). Ce point de terminaison est actuellement uniquement exposé à l’aide du protocole [AMQP](https://www.amqp.org/) sur le port 5671. Un IoT Hub expose les propriétés suivantes pour vous permettre de contrôler le point de terminaison de messages compatible avec Event Hub **messages/events** prédéfini.

| Propriété            | Description |
| ------------------- | ----------- |
| **Nombre de partitions** | Configurez cette propriété lors de la création pour définir le nombre de [partitions](../event-hubs/event-hubs-features.md#partitions) pour la réception d’événements appareil-à-cloud. |
| **Durée de rétention**  | Cette propriété spécifie la durée en jours de conservation des messages par IoT Hub. La durée par défaut est de un jour, mais elle peut être augmentée à sept jours. |

IoT Hub permet la conservation des données dans les Event Hubs intégrés pendant au maximum 7 jours. Vous pouvez définir la durée de conservation lors de la création de votre IoT Hub. La durée de conservation des données dans IoT Hub dépend de votre niveau IoT hub et du type d’unité. En termes de taille, les Event Hubs intégrés peuvent conserver les messages de la taille maximale jusqu’à au moins 24 heures de quota. Par exemple, pour 1 unité S1, IoT Hub fournit un stockage suffisant pour conserver au moins 400 000 messages d’une taille de 4 Ko chacun. Si vos appareils envoient des messages plus petits, ceux-ci peuvent être conservés plus longtemps (jusqu’à 7 jours) en fonction de la quantité de stockage utilisée. Nous garantissons la conservation des données au minimum pendant la durée spécifiée. Les messages expirent et ne sont pas accessibles une fois la durée de conservation écoulée. 

IoT Hub vous permet aussi de gérer des groupes de consommateurs sur le point de terminaison prédéfini de réception appareil vers cloud. Vous pouvez avoir jusqu’à 20 groupes de consommateurs pour chaque IoT Hub.

Si vous utilisez un [routage des messages](iot-hub-devguide-messages-d2c.md) et que la [route de repli](iot-hub-devguide-messages-d2c.md#fallback-route) est activée, tous les messages qui ne correspondent pas à une requête sur un itinéraire sont acheminés vers le point de terminaison intégré. Si vous désactivez cette route de repli, les messages qui ne correspondent à aucune requête sont abandonnés.

La durée de rétention peut être modifiée par programme, avec les [API REST de fournisseur de ressources IoT Hub](/rest/api/iothub/iothubresource) ou avec le [Portail Azure](https://portal.azure.com).

IoT Hub expose le point de terminaison prédéfini **messages/events** pour permettre à vos services principaux de lire les messages appareil vers cloud que reçoit votre hub. Ce point de terminaison est compatible avec Event Hub, ce qui vous permet d’utiliser tout mécanisme pris en charge par le service Event Hubs pour la lecture des messages.

## <a name="read-from-the-built-in-endpoint"></a>Lire à partir du point de terminaison intégré

Certaines intégrations de produits et kits de développement logiciel Event Hubs prennent en compte IoT Hub et vous permettent d’utiliser votre chaîne de connexion au service IoT hub pour se connecter au point de terminaison intégré.

Si vous utilisez des kits de développement logiciel Event Hubs ou des intégrations de produits qui ne prennent pas en compte IoT Hub, vous avez besoin d’un point de terminaison et d’un nom compatibles avec Event Hub. Vous pouvez récupérer ces valeurs à partir du portail comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre IoT Hub.

2. Cliquez sur **Points de terminaison intégrés**.

3. La section **Événements** contient les valeurs suivantes : **Partitions**, **Nom compatible avec les hubs d’événements**, **Point de terminaison compatible Event Hub**, **Durée de rétention** et **Groupes de consommateurs**.

    ![Paramètres Appareil vers cloud](./media/iot-hub-devguide-messages-read-builtin/eventhubcompatible.png)

Dans le portail, le champ Point de terminaison compatible Event Hub contient une chaîne de connexion Event Hubs complète ressemblant à ceci : **Endpoint=sb://abcd1234namespace.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=keykeykeykeykeykey=;EntityPath=iothub-ehub-abcd-1234-123456**. Si le Kit de développement logiciel (SDK) que vous utilisez requiert d’autres valeurs, ce sont les suivantes :

| Nom | Valeur |
| ---- | ----- |
| Point de terminaison | sb://abcd1234namespace.servicebus.windows.net/ |
| HostName | abcd1234namespace.servicebus.windows.net |
| Espace de noms | abcd1234namespace |

Vous pouvez ensuite choisir une stratégie d’accès partagé dans la liste déroulante, comme illustré dans la capture d’écran ci-dessus. Elle montre uniquement les stratégies qui disposent des autorisations **ServiceConnect** pour se connecter à l’Event Hub spécifié.

Les kits de développement logiciel que vous pouvez utiliser pour vous connecter au point de terminaison compatible Event Hub intégré exposé par IoT Hub sont les suivants :

| Langage | Kit SDK | Exemple |
| -------- | --- | ------ |
| .NET | https://www.nuget.org/packages/Azure.Messaging.EventHubs | [Démarrage rapide](quickstart-send-telemetry-dotnet.md) |
| Java | https://mvnrepository.com/artifact/com.azure/azure-messaging-eventhubs | [Démarrage rapide](quickstart-send-telemetry-java.md) |
| Node.js | https://www.npmjs.com/package/@azure/event-hubs | [Démarrage rapide](quickstart-send-telemetry-node.md) |
| Python | https://pypi.org/project/azure-eventhub/ | [Démarrage rapide](quickstart-send-telemetry-python.md) |

Les intégrations de produits que vous pouvez utiliser avec le point de terminaison compatible Event Hub exposé par IoT Hub sont les suivantes :

* [Azure Functions](../azure-functions/index.yml). Voir [Traitement de données à partir d’IoT Hub avec Azure Functions](https://azure.microsoft.com/resources/samples/functions-js-iot-hub-processing/).
* [Azure Stream Analytics](../stream-analytics/index.yml). Voir [Diffuser en continu des données en tant qu’entrées dans Stream Analytics](../stream-analytics/stream-analytics-define-inputs.md#stream-data-from-iot-hub).
* [Time Series Insights](../time-series-insights/index.yml). Voir [Ajouter une source d’événement IoT Hub à votre environnement Time Series Insights](../time-series-insights/how-to-ingest-data-iot-hub.md).
* [Apache Storm spout](../hdinsight/storm/apache-storm-develop-csharp-event-hub-topology.md). Vous pouvez afficher la [source spout](https://github.com/apache/storm/tree/master/external/storm-eventhubs) sur GitHub.
* [Intégration Apache Spark](../hdinsight/spark/apache-spark-ipython-notebook-machine-learning.md).
* [Azure Databricks](/azure/azure-databricks/).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les points de terminaison IoT Hub, consultez [Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md).

* Les [guides de démarrage rapide](quickstart-send-telemetry-node.md) vous montrent comment envoyer des messages appareil-à-cloud à partir d’appareils simulés et lire les messages à partir du point de terminaison intégré. 

Pour plus de détails, consultez le didacticiel [Traiter les messages appareil-à-cloud IoT Hub en utilisant les itinéraires](tutorial-routing.md).

* Si vous voulez acheminer vos messages appareil-à-cloud vers des points de terminaison personnalisés, consultez [Utiliser des itinéraires de messages et des points de terminaison personnalisés pour les messages appareil-à-cloud](iot-hub-devguide-messages-read-custom.md).