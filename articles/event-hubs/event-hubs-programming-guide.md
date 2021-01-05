---
title: Guide de programmation .NET - Azure Event Hubs (hérité) | Microsoft Docs
description: Cet article explique comment rédiger du code pour Azure Event Hubs à l’aide du SDK Azure .NET.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 17bec931f79a6dbb3d98270ab0ff6e2d1d4c6541
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89013909"
---
# <a name="net-programming-guide-for-azure-event-hubs-legacy-microsoftazureeventhubs-package"></a>Guide de programmation .NET pour Azure Event Hubs (package Microsoft.Azure.EventHubs hérité)
Cet article décrit quelques scénarios courants de l’écriture de code à l’aide du service Azure Event Hubs. Il suppose une connaissance préalable des concentrateurs d’événements. Pour une vue d’ensemble conceptuelle des concentrateurs d’événements, consultez [Vue d'ensemble des concentrateurs d’événements](./event-hubs-about.md).

> [!WARNING]
> Ce guide concerne l'ancien package **Microsoft.Azure.EventHubs**. Nous vous recommandons de [migrer](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md) votre code pour utiliser le dernier package [Azure.Messaging.EventHubs](event-hubs-dotnet-standard-getstarted-send.md).  


## <a name="event-publishers"></a>Éditeurs d'événements

Vous envoyez des événements vers un concentrateur d’événements soit en utilisant HTTP POST, soit via une connexion AMQP 1.0. Le choix entre les deux méthodes à utiliser et à quel moment dépend du scénario spécifique qui est adressé. Les connexions AMQP 1.0 sont limitées en tant que connexions réparties dans Service Bus et sont plus appropriées dans les scénarios avec des volumes de messages plus importants fréquents et des conditions de latence plus faible, car elles fournissent un canal de messagerie permanent.

L’utilisation des API gérées avec .NET, les constructions principales pour publier des données sur les concentrateurs d’événements sont les classes [EventHubClient][] et [EventData][]. [EventHubClient][] fournit le canal de communication AMQP par le biais duquel les événements sont envoyés au concentrateur d’événements. La classe [EventData][] représente un événement et sert à publier des messages sur un concentrateur d’événements. Cette classe inclut le corps, certaines métadonnées (Properties) et les informations d'en-tête (SystemProperties) sur l'événement. D’autres propriétés sont ajoutées à l’objet [EventData][] lorsqu’il traverse un concentrateur d’événements.

## <a name="get-started"></a>Bien démarrer
Les classes .NET qui prennent en charge Event Hubs sont fournies dans le package NuGet [Microsoft.Azure.EventHubs](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/). Vous pouvez installer ce dernier à l’aide de l’Explorateur de solutions Visual Studio ou de la [Console du gestionnaire de package](https://docs.nuget.org/docs/start-here/using-the-package-manager-console) dans Visual Studio. Pour cela, entrez la commande suivante dans la fenêtre de la [console du gestionnaire du package](https://docs.nuget.org/docs/start-here/using-the-package-manager-console) :

```shell
Install-Package Microsoft.Azure.EventHubs
```

## <a name="create-an-event-hub"></a>Créer un hub d’événements

Pour créer des Event Hubs, vous pouvez utiliser le Portail Azure, Azure PowerShell ou Azure CLI. Pour plus d’informations, consultez l’article [Créer un espace de noms Event Hubs et un concentrateur d’événements avec le portail Azure](event-hubs-create.md).

## <a name="create-an-event-hubs-client"></a>Création d’un client de concentrateurs d’événements

La classe principale d’interaction avec Event Hubs est [Microsoft.Azure.EventHubs.EventHubClient][EventHubClient]. Vous pouvez instancier cette classe à l’aide de la méthode [CreateFromConnectionString](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createfromconnectionstring), comme indiqué dans l’exemple suivant :

```csharp
private const string EventHubConnectionString = "Event Hubs namespace connection string";
private const string EventHubName = "event hub name";

var connectionStringBuilder = new EventHubsConnectionStringBuilder(EventHubConnectionString)
{
    EntityPath = EventHubName

};
eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());
```

## <a name="send-events-to-an-event-hub"></a>Envoyer des événements à un concentrateur d'événements

Vous envoyez des événements à un Event Hub en créant une instance [EventHubClient][] et en l’envoyant de manière asynchrone par le biais de la méthode [SendAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync). Cette méthode prend un seul paramètre d’instance [EventData][] et l’envoie de façon asynchrone à un hub d’événements.

## <a name="event-serialization"></a>Sérialisation d'événement

La classe [EventData][] comporte [deux constructeurs surchargés](/dotnet/api/microsoft.azure.eventhubs.eventdata.-ctor) qui utilisent un grand nombre de paramètres, des octets ou un tableau d’octets, qui représentent la charge utile des données d’événement. Lorsque vous utilisez JSON avec [EventData][], vous pouvez utiliser **Encoding.UTF8.GetBytes()** pour récupérer le tableau d'octets d'une chaîne encodée JSON. Par exemple :

```csharp
for (var i = 0; i < numMessagesToSend; i++)
{
    var message = $"Message {i}";
    Console.WriteLine($"Sending message: {message}");
    await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
}
```

## <a name="partition-key"></a>Clé de partition

> [!NOTE]
> Si vous n’êtes pas familiarisé avec les partitions, consultez [cet article](event-hubs-features.md#partitions). 

Lors de l’envoi des données d’événement, vous pouvez spécifier une valeur hachée afin de produire une affectation de partition. Vous spécifiez la partition à l’aide de la propriété [PartitionSender.PartitionID](/dotnet/api/microsoft.azure.eventhubs.partitionsender.partitionid). Toutefois, la décision d’utiliser des partitions implique d’effectuer un choix entre disponibilité et cohérence. 

### <a name="availability-considerations"></a>Considérations relatives à la disponibilité

L’utilisation d’une clé de partition est facultative, et vous devez réfléchir attentivement au besoin d’en utiliser une ou non. Si vous ne spécifiez aucune clé de partition lors de la publication d’un événement, une affectation de type tourniquet (round robin) est utilisée. Dans de nombreux cas, l’utilisation d’une clé de partition est un bon choix si l’ordre des événements est important. Lorsque vous utilisez une clé de partition, les partitions nécessitent une disponibilité sur un nœud unique, et des interruptions peuvent se produire dans le temps, par exemple, lors du redémarrage et de la correction des nœuds de calcul. Par conséquent, si vous définissez un ID de partition et que cette partition devient indisponible pour une raison quelconque, une tentative d’accès aux données de la partition échouera. Si la haute disponibilité représente le critère le plus important, ne spécifiez pas de clé de partition ; dans ce cas, les événements sont envoyés aux partitions à l’aide du modèle de tourniquet (round-robin) décrit précédemment. Dans ce scénario, vous effectuez un choix explicite entre la disponibilité (aucun ID de partition) et la cohérence (épinglage d’événements à un ID de partition).

Une autre considération peut être la gestion des retards dans le traitement des événements. Dans certains cas, il est préférable de supprimer les données et d’effectuer une nouvelle tentative plutôt que d’essayer de poursuivre le traitement, ce qui peut entraîner davantage de retards de traitement en aval. Par exemple, avec un téléscripteur pour le marché boursier, il est préférable d’attendre les données à jour complètes, mais dans une conversation en direct ou un scénario VOIP, vous préférerez plutôt obtenir les données rapidement, même incomplètes.

Étant donné ces considérations relatives à la disponibilité, dans ces scénarios vous pouvez choisir l’une des stratégies de gestion des erreurs suivantes :

- Arrêter (arrêter la lecture à partir de concentrateurs d’événements jusqu’à la résolution des problèmes)
- Supprimer (les messages ne sont pas importants, les supprimer)
- Réessayer (effectuer une nouvelle tentative pour les messages selon vos besoins)

Pour obtenir plus d’informations et consulter une discussion concernant les compromis entre la disponibilité et la cohérence, consultez [Disponibilité et cohérence dans Event Hubs](event-hubs-availability-and-consistency.md). 

## <a name="batch-event-send-operations"></a>Opérations d'envoi d’événements par lot

L’envoi d’événements par lots permet d’augmenter le débit. Vous pouvez utiliser l’API [CreateBatch](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch) afin de créer un lot auquel des objets de données pourront être ajoutés par la suite pour un appel [SendAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync).

Un lot ne doit pas dépasser la limite de 1 Mo d’un événement. En outre, chaque message du lot utilise la même identité d’éditeur. Il incombe à l'expéditeur de s’assurer que le lot ne dépasse pas la taille d'événement maximale. Le cas échéant, une erreur **Send** cliente est générée. Vous pouvez utiliser la méthode d’assistance [EventHubClient.CreateBatch](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch) pour vous assurer que le lot ne dépasse pas 1 Mo. Vous obtenez un [EventDataBatch](/dotnet/api/microsoft.azure.eventhubs.eventdatabatch) vide de l’API [CreateBatch](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch), puis vous utilisez [TryAdd](/dotnet/api/microsoft.azure.eventhubs.eventdatabatch.tryadd) pour ajouter des événements pour construire le lot. 

## <a name="send-asynchronously-and-send-at-scale"></a>Envoi de manière asynchrone et envoi à l'échelle

Vous envoyez des événements à un Event Hub de manière asynchrone. L’envoi en mode asynchrone augmente la vitesse à laquelle un client peut envoyer des événements. La méthode [SendAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync) renvoie un objet [Task](/dotnet/api/system.threading.tasks.task?view=netcore-3.1). Vous pouvez utiliser la classe [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) sur le client pour contrôler les options de nouvelle tentative du client.

## <a name="event-consumers"></a>Consommateurs d'événements
La classe [EventProcessorHost][] traite les données à partir des concentrateurs d’événements. Vous devez utiliser cette implémentation lors de la création de lecteurs d'événement sur la plateforme .NET. [EventProcessorHost][] fournit un environnement d'exécution sécurisé, multiprocessus, thread-safe pour des implémentations d’événements qui fournissent également une gestion de contrôle et de location de partition.

Pour utiliser la classe [EventProcessorHost][], vous pouvez implémenter [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor). Cette interface contient quatre méthodes :

* [OpenAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.openasync)
* [CloseAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.closeasync)
* [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync)
* [ProcessErrorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processerrorasync)

Pour commencer le traitement des événements, vous devez instancier [EventProcessorHost][]en fournissant les paramètres appropriés pour votre concentrateur d'événements. Par exemple :

> [!NOTE]
> EventProcessorHost et ses classes connexes sont mentionnées dans le package **Microsoft.Azure.EventHubs.Processor**. Ajoutez le package à votre projet Visual Studio en suivant les instructions fournies dans [cet article](event-hubs-dotnet-framework-getstarted-send.md#add-the-event-hubs-nuget-package) ou en émettant la commande suivante dans la fenêtre [Console du Gestionnaire de Package](https://docs.nuget.org/docs/start-here/using-the-package-manager-console) :`Install-Package Microsoft.Azure.EventHubs.Processor`.

```csharp
var eventProcessorHost = new EventProcessorHost(
        EventHubName,
        PartitionReceiver.DefaultConsumerGroupName,
        EventHubConnectionString,
        StorageConnectionString,
        StorageContainerName);
```

Ensuite, appelez [RegisterEventProcessorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.registereventprocessorasync) pour inscrire votre implémentation [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor) auprès du runtime :

```csharp
await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();
```

À ce stade, l’hôte tente d’acquérir un bail sur chaque partition dans le hub d’événements à l’aide d’un algorithme « gourmand ». Ces baux sont valables pour une période donnée et doivent ensuite être renouvelés. Étant donné que de nouveaux nœuds (ici, des instances de rôle) sont en ligne, ils placent des réservations de bail et, en même temps, la charge est déplacée entre les nœuds tandis que chaque nœud tente d’acquérir plus de baux.

![Hôte du processeur d’événements](./media/event-hubs-programming-guide/IC759863.png)

Au fil du temps, un équilibre est établi. Cette fonctionnalité dynamique permet d’appliquer aux consommateurs une mise à l'échelle basée sur le processeur pour une augmentation et une diminution d’échelle. Event Hubs n’ayant pas de concept direct du nombre de messages, l’utilisation moyenne du processeur est souvent la meilleure solution pour mesurer la mise à l’échelle du serveur principal ou du consommateur. Si les éditeurs commencent à publier plus d'événements que ce que les consommateurs peuvent traiter, l'augmentation du processeur sur les consommateurs peut être utilisée pour effectuer une mise à l’échelle automatique sur le nombre d’instances de travail.

La classe [EventProcessorHost][] implémente également un mécanisme de point de contrôle basé sur le stockage Azure. Ce mécanisme stocke le décalage sur une base par partition, afin que chaque consommateur puisse déterminer quel était le dernier point de contrôle du consommateur précédent. Étant donné que la transition des partitions entre les nœuds se fait via les baux, il s’agit d’un mécanisme de synchronisation qui facilité le déplacement de la charge.

## <a name="publisher-revocation"></a>Révocation de l’éditeur

Outre les fonctionnalités d’exécution avancées de l’hôte du processeur d’événements, le service Hubs d’événements permet la [révocation de l’éditeur](/rest/api/eventhub/revoke-publisher) pour empêcher certains éditeurs d’envoyer des événements à un concentrateur d’événements. Ces fonctionnalités sont utiles si le jeton d’un éditeur a été compromis ou une mise à jour de logiciel les fait se comporter de façon inappropriée. Dans ces situations, l’identité de l'éditeur, qui fait partie de leur jeton SAP, peut être bloquée à partir d'événements de publication.

> [!NOTE]
> Actuellement, seule l'API REST prend en charge cette fonctionnalité ([révocation de l'éditeur](/rest/api/eventhub/revoke-publisher)).

Pour plus d’informations sur la révocation de l’éditeur et l’envoi vers des concentrateurs d’événements en tant qu’éditeur, consultez l’exemple [Publication sécurisée à grande échelle des concentrateurs d’événements de Service Bus](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-99ce67ab).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les scénarios des concentrateurs d’événements, consultez ces liens :

* [Vue d’ensemble de l'API Event Hubs](./event-hubs-samples.md)
* [Qu’est-ce qu’Event Hubs](./event-hubs-about.md)
* [Disponibilité et cohérence dans Event Hubs](event-hubs-availability-and-consistency.md)
* [Informations de référence des API hôtes du processeur d’événements](/dotnet/api/microsoft.servicebus.messaging.eventprocessorhost)

[NamespaceManager]: /dotnet/api/microsoft.servicebus.namespacemanager
[EventHubClient]: /dotnet/api/microsoft.azure.eventhubs.eventhubclient
[EventData]: /dotnet/api/microsoft.azure.eventhubs.eventdata
[CreateEventHubIfNotExists]: /dotnet/api/microsoft.servicebus.namespacemanager.createeventhubifnotexists
[PartitionKey]: /dotnet/api/microsoft.servicebus.messaging.eventdata#Microsoft_ServiceBus_Messaging_EventData_PartitionKey
[EventProcessorHost]: /dotnet/api/microsoft.azure.eventhubs.processor
