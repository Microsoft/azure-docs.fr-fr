---
title: Surveillance des opérations Azure IoT Hub (déconseillé) | Microsoft Docs
description: Découvrez comment utiliser la surveillance des opérations Azure IoT Hub pour surveiller l’état des opérations sur votre hub IoT en temps réel.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/11/2019
ms.author: robinsh
ms.custom: amqp, devx-track-csharp
ms.openlocfilehash: 045d5693c4388c6285bc6983ac2a385ceac9f6d0
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94408122"
---
# <a name="iot-hub-operations-monitoring-deprecated"></a>Surveillance des opérations IoT Hub (déconseillé)

La surveillance des opérations IoT Hub vous permet de surveiller l’état des opérations sur votre hub IoT en temps réel. IoT Hub effectue le suivi des événements entre différentes catégories d’opérations. Vous pouvez opter pour l’envoi des événements d’une ou plusieurs catégories à un point de terminaison de votre IoT Hub en vue de leur traitement. Vous pouvez surveiller les données des erreurs ou configurer un traitement plus complexe basé sur des modèles de données.

>[!NOTE]
>La **surveillance des opérations d’IoT Hub est déconseillée et a été supprimée d’IoT Hub le 10 mars 2019**. Pour surveiller les opérations et l'intégrité d'IoT Hub, consultez [Surveiller IoT Hub](monitor-iot-hub.md). Pour plus d’informations sur la chronologie de dépréciation, consultez l’article [Surveiller vos solutions Azure IoT avec Azure Monitor et Azure Resource Health](https://azure.microsoft.com/blog/monitor-your-azure-iot-solutions-with-azure-monitor-and-azure-resource-health).

IoT Hub surveille six catégories d’événements :

* Opérations d’identité des appareils
* Télémétrie d’appareil
* Messages Cloud vers appareil
* Connexions
* Chargements de fichiers
* Routage de messages

> [!IMPORTANT]
> La surveillance des opérations d’IoT Hub ne garantit pas une remise fiable ou chronologique des événements. Selon l’infrastructure sous-jacente d’IoT Hub, certains événements peuvent être perdus ou remis de manière non chronologique. Utilisez la surveillance des opérations pour générer des alertes basées sur des signaux d’erreur, comme les échecs de tentatives de connexion ou les déconnexions fréquentes d’appareils spécifiques. Vous ne devez pas vous fier aux événements de surveillance des opérations pour créer un magasin cohérent d’état des appareils, par exemple, un magasin assurant le suivi de l’état connecté ou déconnecté d’un appareil. 

## <a name="how-to-enable-operations-monitoring"></a>Comment activer la surveillance des opérations

1. Créez un hub IoT. Pour savoir comment créer un hub IoT, voir le guide [Prise en main](quickstart-send-telemetry-dotnet.md).

2. Ouvrez le panneau de votre hub IoT. De là, cliquez sur **Surveillance des opérations**.

    ![Accéder à la configuration de la surveillance des opérations dans le portail](./media/iot-hub-operations-monitoring/enable-OM-1.png)

3. Sélectionnez les catégories de surveillance qui vous intéressent, puis cliquez sur **Enregistrer**. Les événements sont disponibles en lecture depuis le point de terminaison compatible avec le hub d’événements répertorié dans **Paramètres de surveillance**. Le point de terminaison IoT Hub s’appelle `messages/operationsmonitoringevents`.

    ![Configurer la surveillance des opérations sur votre IoT Hub](./media/iot-hub-operations-monitoring/enable-OM-2.png)

> [!NOTE]
> Si vous sélectionnez la surveillance **détaillée** dans la catégorie **Connexions**, IoT Hub génère des messages de diagnostic supplémentaires. Pour toutes les autres catégories, le paramètre **Détaillée** modifie la quantité d’informations qu’IoT Hub inclut dans chaque message d’erreur.

## <a name="event-categories-and-how-to-use-them"></a>Catégories d’événements et utilisation respective

Chaque catégorie de surveillance d’opérations assure le suivi d’un type spécifique d’interaction avec IoT Hub et a un schéma qui définit la façon dont sont structurés les événements qu’elle comporte.

### <a name="device-identity-operations"></a>Opérations d’identité des appareils

La catégorie d’opérations d’identité des appareils effectue le suivi des erreurs qui se produisent quand vous tentez de créer, mettre à jour ou supprimer une entrée dans le registre d’identité de votre hub IoT. Le suivi de cette catégorie est utile pour les scénarios d’approvisionnement.

```json
{
    "time": "UTC timestamp",
    "operationName": "create",
    "category": "DeviceIdentityOperations",
    "level": "Error",
    "statusCode": 4XX,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "durationMs": 1234,
    "userAgent": "userAgent",
    "sharedAccessPolicy": "accessPolicy"
}
```

### <a name="device-telemetry"></a>Télémétrie d’appareil

La catégorie de télémétrie d’appareil effectue le suivi des erreurs qui se produisent au niveau du hub IoT et qui sont liées au pipeline de télémétrie. Cette catégorie inclut notamment les erreurs concernant l’envoi d’événements de télémétrie (par exemple, une limitation) et la réception des événements de télémétrie (par exemple, un lecteur non autorisé). Cette catégorie ne peut pas intercepter les erreurs provoquées par le code en cours d’exécution sur l’appareil lui-même.

```json
{
    "messageSizeInBytes": 1234,
    "batching": 0,
    "protocol": "Amqp",
    "authType": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "time": "UTC timestamp",
    "operationName": "ingress",
    "category": "DeviceTelemetry",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "EventProcessedUtcTime": "UTC timestamp",
    "PartitionId": 1,
    "EventEnqueuedUtcTime": "UTC timestamp"
}
```

### <a name="cloud-to-device-commands"></a>Commandes cloud-à-appareil

La catégorie de commandes cloud-à-appareil effectue le suivi des erreurs qui se produisent au niveau du hub IoT et qui sont liées au pipeline de messages cloud-à-appareil. Cette catégorie inclut notamment les erreurs concernant l’envoi de messages cloud-à-appareil (telles qu’un expéditeur non autorisé), la réception des messages cloud-à-appareil (telles que le dépassement du nombre de remises) et la réception des commentaires de message cloud-à-appareil (telles que des commentaires arrivés à expiration). Cette catégorie n’intercepte pas d’erreurs dans le cas d’un appareil qui gère mal un message cloud-à-appareil si le message a été correctement remis.

```json
{
    "messageSizeInBytes": 1234,
    "authType": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "deliveryAcknowledgement": 0,
    "protocol": "Amqp",
    "time": " UTC timestamp",
    "operationName": "ingress",
    "category": "C2DCommands",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "EventProcessedUtcTime": "UTC timestamp",
    "PartitionId": 1,
    "EventEnqueuedUtcTime": "UTC timestamp"
}
```

### <a name="connections"></a>Connexions

La catégorie de connexions effectue le suivi des erreurs provoquées par la connexion des appareils à un hub IoT ou leur déconnexion de celui-ci. Le suivi de cette catégorie est utile pour identifier les tentatives de connexion non autorisées et pour repérer les moments auxquels une connexion est perdue pour les appareils qui se trouvent dans des zones bénéficiant d’une connectivité médiocre.

```json
{
    "durationMs": 1234,
    "authType": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "protocol": "Amqp",
    "time": " UTC timestamp",
    "operationName": "deviceConnect",
    "category": "Connections",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID"
}
```

### <a name="file-uploads"></a>Chargements de fichiers

La catégorie de chargement de fichiers effectue le suivi des erreurs qui se produisent au niveau de l’IoT hub et qui sont liées à la fonctionnalité de chargement. Cette catégorie inclut :

* Erreurs qui se produisent avec l’URI SAP, par exemple en cas d’expiration avant qu’un appareil notifie le hub d’un téléchargement terminé.

* Échecs des téléchargements signalés par l’appareil.

* Erreurs qui se produisent lorsqu’un fichier est introuvable dans le stockage lors de la création du message de notification IoT Hub.

Cette catégorie ne peut pas détecter les erreurs qui surviennent directement pendant que l’appareil charge un fichier de stockage.

```json
{
    "authType": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "protocol": "HTTP",
    "time": " UTC timestamp",
    "operationName": "ingress",
    "category": "fileUpload",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "blobUri": "http//bloburi.com",
    "durationMs": 1234
}
```

### <a name="message-routing"></a>Routage de messages

La catégorie de routage des messages assure le suivi des erreurs qui se produisent pendant l’évaluation du routage des messages et de l’intégrité du point de terminaison perçue par IoT Hub. Cette catégorie inclut les événements tels qu’une règle prenant la valeur « Non définie », lorsqu’IoT Hub marque un point de terminaison comme inactif et toute autre erreur reçue à partir d’un point de terminaison. Cette catégorie n’inclut pas les erreurs spécifiques sur les messages eux-mêmes (telles que les erreurs de limitation des appareils) qui sont signalées dans la catégorie « télémétrie des appareils ».

```json
{
    "messageSizeInBytes": 1234,
    "time": "UTC timestamp",
    "operationName": "ingress",
    "category": "routes",
    "level": "Error",
    "deviceId": "device-ID",
    "messageId": "ID of message",
    "routeName": "myroute",
    "endpointName": "myendpoint",
    "details": "ExternalEndpointDisabled"
}
```

## <a name="connect-to-the-monitoring-endpoint"></a>Se connecter au point de terminaison de surveillance

Le point de terminaison de surveillance de votre IoT Hub est un point de terminaison compatible Event Hub. Vous pouvez utiliser n’importe quel mécanisme compatible avec Event Hubs pour lire les messages de surveillance à partir de ce point de terminaison. L’exemple suivant crée un lecteur de base qui ne convient pas dans le cas d’un déploiement à débit élevé. Pour plus d’informations sur la façon de traiter les messages à partir des concentrateurs d’événements, reportez-vous au didacticiel [Prise en main des concentrateurs d’événements](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md) .

Pour vous connecter au point de terminaison de surveillance, vous avez besoin d’une chaîne de connexion et du nom du point de terminaison. Les étapes suivantes vous montrent comment trouver les valeurs nécessaires dans le portail :

1. Dans le portail, accédez à votre panneau de ressources IoT Hub.

2. Sélectionnez **Surveillance des opérations**, notez les valeurs du **Nom compatible Event Hub** et du **Point de terminaison compatible Event Hub** :

    ![Valeurs du point de terminaison compatible Event Hub](./media/iot-hub-operations-monitoring/monitoring-endpoint.png)

3. Sélectionnez **Stratégies d’accès partagé**, puis **service**. Prenez note de la valeur de **Clé primaire** :

    ![Clé primaire de la stratégie d’accès partagé du service](./media/iot-hub-operations-monitoring/service-key.png)

L’exemple de code C# suivant est extrait d’une application console C# **Bureau classique Windows** de Visual Studio. Le projet possède le package NuGet **WindowsAzure.ServiceBus**.

* Remplacez l’espace réservé de chaîne de connexion par une chaîne de connexion qui utilise les valeurs du **Point de terminaison compatible Event Hub** et de la **Clé primaire** du service notées précédemment, comme l’illustre l’exemple suivant :

    ```csharp
    "Endpoint={your Event Hub-compatible endpoint};SharedAccessKeyName=service;SharedAccessKey={your service primary key value}"
    ```

* Remplacez l’espace réservé de nom de point de terminaison de surveillance par la valeur du **Nom compatible Event Hub** notée précédemment.

```csharp
class Program
{
    static string connectionString = "{your monitoring endpoint connection string}";
    static string monitoringEndpointName = "{your monitoring endpoint name}";
    static EventHubClient eventHubClient;

    static void Main(string[] args)
    {
        Console.WriteLine("Monitoring. Press Enter key to exit.\n");

        eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, monitoringEndpointName);
        var d2cPartitions = eventHubClient.GetRuntimeInformation().PartitionIds;
        CancellationTokenSource cts = new CancellationTokenSource();
        var tasks = new List<Task>();

        foreach (string partition in d2cPartitions)
        {
            tasks.Add(ReceiveMessagesFromDeviceAsync(partition, cts.Token));
        }

        Console.ReadLine();
        Console.WriteLine("Exiting...");
        cts.Cancel();
        Task.WaitAll(tasks.ToArray());
    }

    private static async Task ReceiveMessagesFromDeviceAsync(string partition, CancellationToken ct)
    {
        var eventHubReceiver = eventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition, DateTime.UtcNow);
        while (true)
        {
            if (ct.IsCancellationRequested)
            {
                await eventHubReceiver.CloseAsync();
                break;
            }

            EventData eventData = await eventHubReceiver.ReceiveAsync(new TimeSpan(0,0,10));

            if (eventData != null)
            {
                string data = Encoding.UTF8.GetString(eventData.GetBytes());
                Console.WriteLine("Message received. Partition: {0} Data: '{1}'", partition, data);
            }
        }
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

Pour approfondir l’exploration de l’utilisation d’Azure Monitor pour surveiller IoT Hub, consultez :

* [Superviser avec IoT Hub](monitor-iot-hub.md)

* [Migrer de la Surveillance des opérations IoT Hub vers Azure Monitor](iot-hub-migrate-to-diagnostics-settings.md)