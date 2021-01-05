---
title: Azure Event Hubs - Kits de développement logiciel (SDK) client | Microsoft Docs
description: Cet article fournit des informations sur les kits de développement logiciel (SDK) clients pour Azure Event Hubs.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 9f4eec603245f1e4ea6fa4d97b843ca6a770f2ad
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88930874"
---
# <a name="azure-event-hubs---client-sdks"></a>Azure Event Hubs - Kits de développement logiciel (SDK)
Cet article fournit les informations suivantes pour les kits de développement logiciel (SDK) pris en charge par Azure Event Hubs : 

- Emplacement du package que vous pouvez utiliser dans vos applications 
- Emplacement GitHub où vous pouvez trouver le code source, les exemples, le fichier Lisez-moi, le journal des modifications, les problèmes signalés, et signaler de nouveaux problèmes 
- Liens vers les tutoriels de démarrage rapide 

## <a name="client-sdks"></a>Kits de développement logiciel (SDK) client
Le tableau suivant décrit tous les clients de runtime Azure Event Hubs actuellement disponibles. Bien que certaines de ces bibliothèques incluent aussi des fonctionnalités de gestion limitées, il existe des bibliothèques dédiées aux opérations de gestion. Ces bibliothèques sont principalement destinées à **envoyer et recevoir des messages** d’un hub d’événements.

| Langage | Package | Informations de référence | 
| -------- | ------- | --------------- | 
| . NET Standard (**version la plus récente**, prend en charge .NET Core et .NET Framework) | [Azure.Messaging.EventHubs](https://www.nuget.org/packages/Azure.Messaging.EventHubs/) |<ul><li>[Emplacement GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs)</li><li>[Didacticiel](event-hubs-dotnet-standard-getstarted-send.md)</li></ul> |
|       | [Azure.Messaging.EventHubs.Processor](https://www.nuget.org/packages/Azure.Messaging.EventHubs.Processor/) | <ul><li>[Emplacement GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor)</li><li>[Didacticiel](event-hubs-dotnet-standard-getstarted-send.md)</li></ul> |
| . NET Standard (**version héritée**, prend en charge .NET Core et .NET Framework) | [Microsoft.Azure.EventHubs](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) | <ul><li>[Emplacement GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.EventHubs)</li><li>[Didacticiel](event-hubs-dotnet-standard-getstarted-send.md)</li></ul> | 
|       | [Microsoft.Azure.EventHubs.Processor](https://www.nuget.org/packages/Microsoft.Azure.EventHubs.Processor) | <ul><li>[Emplacement GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.EventHubs.Processor)</li><li>[Didacticiel](event-hubs-dotnet-standard-getstarted-send.md)</li></ul> |
| . NET Framework (**ancienne version**) | [WindowsAzure.Messaging](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) |<ul><li>[Didacticiel](event-hubs-dotnet-framework-getstarted-send.md)</li></ul> |
| Java | [azure-messaging-eventhubs](https://search.maven.org/search?q=a:azure-messaging-eventhubs) | <ul><li>[Emplacement GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs)</li><li>[Didacticiel](event-hubs-java-get-started-send.md)</li></ul> |
|      | [azure-eventhubs](https://search.maven.org/search?q=a:azure-eventhubs) **(version héritée)** | <ul><li>[Emplacement GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/microsoft-azure-eventhubs)</li><li>[Didacticiel](event-hubs-java-get-started-send.md)</li></ul> |
| Python |  [azure-eventhub](https://pypi.org/project/azure-eventhub/) | <ul><li>[Emplacement GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/eventhub/azure-eventhub)</li><li>[Didacticiel](event-hubs-python-get-started-send.md)</li></ul> |
|        | [azure-eventhub-checkpointstoreblob-aio](https://pypi.org/project/azure-eventhub-checkpointstoreblob-aio/) | <ul><li>[Emplacement GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/eventhub/azure-eventhub-checkpointstoreblob-aio)</li><li>[Didacticiel](event-hubs-python-get-started-send.md)</li></ul> |
| JavaScript | [azure/event-hubs](https://www.npmjs.com/package/@azure/event-hubs) | <ul><li>[Emplacement GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs)</li><li>[Didacticiel](event-hubs-node-get-started-send.md)</li></ul> |
|            | [azure/eventhubs-checkpointstore-blob](https://www.npmjs.com/package/@azure/eventhubs-checkpointstore-blob) | <ul><li>[Emplacement GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/eventhubs-checkpointstore-blob)</li><li>[Didacticiel](event-hubs-node-get-started-send.md)</li></ul> |
| Go | [azure-event-hubs-go](https://github.com/Azure/azure-event-hubs-go) | <ul><li>[Emplacement GitHub](https://github.com/Azure/azure-event-hubs-go)</li><li>[Didacticiel](event-hubs-go-get-started-send.md)</li></ul> |
| C | [azure-event-hubs-c](https://github.com/Azure/azure-event-hubs-c) | <ul><li>[Emplacement GitHub](https://github.com/Azure/azure-event-hubs-c)</li><li>[Didacticiel](event-hubs-c-getstarted-send.md)</li></ul> |

## <a name="management-sdks"></a>Kits SDK de gestion
Le tableau suivant répertorie toutes les bibliothèques dédiées à la gestion actuellement disponibles. Aucune de ces bibliothèques ne contient d’opérations de runtime. Elles sont uniquement destinées à la **gestion des entités Event Hubs**.

| Langage | Package | Informations de référence | 
| -------- | ------- | --------------- | 
| .NET Standard | [Microsoft.Azure.Management.EventHub](https://www.nuget.org/packages/Microsoft.Azure.Management.EventHub) |<ul><li>[Emplacement GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.Management.EventHub)</li><li>[Didacticiel](event-hubs-dotnet-standard-getstarted-send.md)</li></ul> |


## <a name="next-steps"></a>Étapes suivantes

Vous pouvez en apprendre plus sur Event Hubs en consultant les liens suivants :

* [Vue d’ensemble d’Event Hubs](./event-hubs-about.md)
* [Créer un hub d’événements](event-hubs-create.md)
* [FAQ sur les hubs d’événements](event-hubs-faq.md)
