---
title: Disponibilité et cohérence - Azure Event Hubs | Microsoft Docs
description: Découvrez comment obtenir la quantité maximale de disponibilité et de cohérence avec Azure Event Hubs à l’aide de partitions.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 81bacd5507396352bb814310979498234ee35347
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96902899"
---
# <a name="availability-and-consistency-in-event-hubs"></a>Disponibilité et cohérence dans Event Hubs

## <a name="overview"></a>Vue d’ensemble
Azure Event Hubs utilise un [modèle de partitionnement](event-hubs-scalability.md#partitions) pour améliorer la disponibilité et la parallélisation dans un concentrateur d’événements unique. Par exemple, si un concentrateur d’événements a quatre partitions et que l’une de ces partitions est déplacée d’un serveur à l’autre dans une opération d’équilibrage de charge, vous pouvez quand même envoyer et recevoir à partir des trois autres partitions. En outre, avoir davantage de partitions vous permet d’avoir plus lecteurs pour traiter vos données simultanément, ce qui améliore le débit global. Comprendre les implications en matière de partitionnement et de classement dans un système distribué est un aspect essentiel de la conception de la solution.

Pour expliquer le compromis entre classement et disponibilité, reportez-vous au [théorème CAP](https://en.wikipedia.org/wiki/CAP_theorem), également connu sous le nom de théorème de Brewer. Ce théorème discute le choix entre la cohérence, la disponibilité et la tolérance de la partition. Il stipule que pour les systèmes partitionnés par réseau, il existe toujours un compromis entre cohérence et disponibilité.

Le théorème de Brewer définit la cohérence et la disponibilité de la façon suivante :
* Tolérance de la partition : la capacité d’un système de traitement de données à continuer le traitement des données même en cas de défaillance d’une partition.
* Disponibilité : un nœud sans échec renvoie une réponse raisonnable dans un délai raisonnable (sans erreurs ou expirations de délai).
* Cohérence : une lecture renvoie toujours la dernière écriture pour un client donné.

## <a name="partition-tolerance"></a>Tolérance de la partition
Event Hubs repose sur un modèle de données partitionné. Vous pouvez configurer le nombre de partitions dans votre concentrateur d’événements pendant l’installation, mais vous ne pouvez pas modifier cette valeur par la suite. Étant donné que vous devez utiliser des partitions avec Event Hubs, vous devez prendre une décision concernant la disponibilité et la cohérence de votre application.

## <a name="availability"></a>Disponibilité
La méthode la plus simple pour se familiariser avec Event Hubs est d’utiliser le comportement par défaut. 

#### <a name="azuremessagingeventhubs-500-or-later"></a>[Azure.Messaging.EventHubs (version 5.0.0 ou ultérieure)](#tab/latest)
Si vous créez un objet **[EventHubProducerClient](/dotnet/api/azure.messaging.eventhubs.producer.eventhubproducerclient?view=azure-dotnet)** et que vous utilisez la méthode **[SendAsync](/dotnet/api/azure.messaging.eventhubs.producer.eventhubproducerclient.sendasync?view=azure-dotnet)** , les événements sont automatiquement distribués entre les partitions de votre Event Hub. Ce comportement offre le meilleur temps d’activité.

#### <a name="microsoftazureeventhubs-410-or-earlier"></a>[Microsoft.Azure.EventHubs (version 4.1.0 ou ultérieure)](#tab/old)
Si vous créez un objet **[EventHubClient](/dotnet/api/microsoft.azure.eventhubs.eventhubclient)** et que vous utilisez la méthode **[Send](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync?view=azure-dotnet#Microsoft_Azure_EventHubs_EventHubClient_SendAsync_Microsoft_Azure_EventHubs_EventData_)** , les événements sont automatiquement distribués entre les partitions de votre Event Hub. Ce comportement offre le meilleur temps d’activité.

---

Pour les cas d’utilisation qui exigent un temps d’activité maximum, ce modèle est conseillé.

## <a name="consistency"></a>Cohérence
Dans certains scénarios, l’ordre des événements peut être important. Par exemple, vous souhaiterez peut-être que votre système principal traite une commande de mise à jour avant une commande de suppression. Dans ce cas, vous pouvez définir la clé de partition sur un événement, ou utiliser un objet `PartitionSender` (si vous utilisez l’ancienne bibliothèque Microsoft.Azure.Messaging) pour envoyer uniquement les événements à une certaine partition. Cela garantit que ces événements sont lus dans l’ordre lorsqu’ils sont lus à partir de la partition. 

Si vous utilisez la bibliothèque **Azure.Messaging.EventHubs** la plus récente, consultez [Migration du code de PartitionSender vers EventHubProducerClient pour la publication d’événements sur une partition](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md#migrating-code-from-partitionsender-to-eventhubproducerclient-for-publishing-events-to-a-partition).

#### <a name="azuremessagingeventhubs-500-or-later"></a>[Azure.Messaging.EventHubs (version 5.0.0 ou ultérieure)](#tab/latest)

```csharp
var connectionString = "<< CONNECTION STRING FOR THE EVENT HUBS NAMESPACE >>";
var eventHubName = "<< NAME OF THE EVENT HUB >>";

await using (var producerClient = new EventHubProducerClient(connectionString, eventHubName))
{
    var batchOptions = new CreateBatchOptions() { PartitionId = "my-partition-id" };
    using EventDataBatch eventBatch = await producerClient.CreateBatchAsync(batchOptions);
    eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes("First")));
    eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes("Second")));
    
    await producerClient.SendAsync(eventBatch);
}
```

#### <a name="microsoftazureeventhubs-410-or-earlier"></a>[Microsoft.Azure.EventHubs (version 4.1.0 ou ultérieure)](#tab/old)

```csharp
var connectionString = "<< CONNECTION STRING FOR THE EVENT HUBS NAMESPACE >>";
var eventHubName = "<< NAME OF THE EVENT HUB >>";

var connectionStringBuilder = new EventHubsConnectionStringBuilder(connectionString){ EntityPath = eventHubName }; 
var eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());
PartitionSender partitionSender = eventHubClient.CreatePartitionSender("my-partition-id");
try
{
    EventDataBatch eventBatch = partitionSender.CreateBatch();
    eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes("First")));
    eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes("Second")));

    await partitionSender.SendAsync(eventBatch);
}
finally
{
    await partitionSender.CloseAsync();
    await eventHubClient.CloseAsync();
}
```

---

Avec cette configuration, gardez à l’esprit que si la partition particulière à laquelle vous envoyez les événements n’est pas disponible, vous recevrez une réponse d’erreur. À titre de comparaison, si vous n’avez pas d’affinité avec une partition unique, le service Event Hubs envoie votre événement à la prochaine partition disponible.

Pour garantir le classement tout en optimisant aussi le temps d’activité, vous pouvez regrouper les événements dans le cadre de votre application de traitement des événements. Pour cela, le moyen le plus simple consiste à marquer votre événement avec une propriété de numéro de séquence personnalisée. Le code suivant montre un exemple :

#### <a name="azuremessagingeventhubs-500-or-later"></a>[Azure.Messaging.EventHubs (version 5.0.0 ou ultérieure)](#tab/latest)

```csharp
// create a producer client that you can use to send events to an event hub
await using (var producerClient = new EventHubProducerClient(connectionString, eventHubName))
{
    // get the latest sequence number from your application
    var sequenceNumber = GetNextSequenceNumber();

    // create a batch of events 
    using EventDataBatch eventBatch = await producerClient.CreateBatchAsync();

    // create a new EventData object by encoding a string as a byte array
    var data = new EventData(Encoding.UTF8.GetBytes("This is my message..."));

    // set a custom sequence number property
    data.Properties.Add("SequenceNumber", sequenceNumber);

    // add events to the batch. An event is a represented by a collection of bytes and metadata. 
    eventBatch.TryAdd(data);

    // use the producer client to send the batch of events to the event hub
    await producerClient.SendAsync(eventBatch);
}
```

#### <a name="microsoftazureeventhubs-410-or-earlier"></a>[Microsoft.Azure.EventHubs (version 4.1.0 ou ultérieure)](#tab/old)
```csharp
// Create an Event Hubs client
var client = new EventHubClient(connectionString, eventHubName);

//Create a producer to produce events
EventHubProducer producer = client.CreateProducer();

// Get the latest sequence number from your application 
var sequenceNumber = GetNextSequenceNumber();

// Create a new EventData object by encoding a string as a byte array
var data = new EventData(Encoding.UTF8.GetBytes("This is my message..."));

// Set a custom sequence number property
data.Properties.Add("SequenceNumber", sequenceNumber);

// Send single message async
await producer.SendAsync(data);
```
---

L’exemple envoie votre événement à l’une des partitions disponibles dans votre concentrateur d’événements et définit le numéro de séquence correspondant à partir de votre application. Cette solution nécessite que l’état soit conservé par votre application de traitement, mais offre à vos expéditeurs un point de terminaison plus susceptible d’être disponible.

## <a name="next-steps"></a>Étapes suivantes
Vous pouvez en apprendre plus sur Event Hubs en consultant les liens suivants :

* [Présentation du service Event Hubs](./event-hubs-about.md)
* [Créer un concentrateur d’événements](event-hubs-create.md)
