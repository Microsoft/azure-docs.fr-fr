---
title: Rubriques système dans Azure Event Grid
description: Décrit les rubriques système d’Azure Event Grid.
ms.topic: conceptual
ms.date: 09/24/2020
ms.openlocfilehash: b3a6e7528da2a11c2f91007425ab8beecaf920c3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91297281"
---
# <a name="system-topics-in-azure-event-grid"></a>Rubriques système dans Azure Event Grid
Une rubrique système dans Event Grid représente un ou plusieurs événements publiés par les services Azure, tels que Stockage Azure et Azure Event Hubs. Par exemple, une rubrique système peut représenter **tous les événements blob** ou uniquement les événements **de création de blob** et **de suppression de blob** publiés pour un **compte de stockage spécifique**. Dans cet exemple, lorsqu’un blob est chargé sur le compte de stockage, le service Stockage Azure publie un événement **de création de blob** dans la rubrique système d’Event Grid, qui transfère ensuite l’événement aux [abonnés](event-handlers.md) de la rubrique qui reçoivent et traitent l’événement. 

> [!NOTE] 
> Seuls les services Azure peuvent publier des événements dans les rubriques système. Par conséquent, vous ne disposez pas d’un point de terminaison ni de clés d’accès que vous pouvez utiliser pour publier des événements comme vous le faites pour des rubriques ou des domaines personnalisés.

## <a name="azure-services-that-support-system-topics"></a>Services Azure qui prennent en charge les rubriques système
Voici la liste actuelle des services Azure qui prennent en charge la création de rubriques système.

- [Azure App Configuration](event-schema-app-configuration.md)
- [Azure App Service](event-schema-app-service.md)
- [Stockage Blob Azure](event-schema-blob-storage.md)
- [Azure Communication Services](event-schema-communication-services.md) 
- [Azure Container Registry](event-schema-container-registry.md)
- [Azure Event Hubs](event-schema-event-hubs.md)
- [Azure IoT Hub](event-schema-iot-hub.md)
- [Azure Key Vault](event-schema-key-vault.md)
- [Azure Machine Learning](event-schema-machine-learning.md)
- [Azure Maps](event-schema-azure-maps.md)
- [Azure Media Services](event-schema-media-services.md)
- [Groupes de ressources Azure](event-schema-resource-groups.md)
- [Azure Service Bus](event-schema-service-bus.md)
- [Azure SignalR](event-schema-azure-signalr.md)
- [Abonnements Azure](event-schema-subscriptions.md)

## <a name="system-topics-as-azure-resources"></a>Rubriques système en tant que ressources Azure
Dans le passé, une rubrique système était implicite et n’était pas exposée pour des raisons de simplicité. Les rubriques système sont désormais visibles en tant que ressources Azure et offrent les capacités suivantes :

- [Afficher les rubriques système dans le portail Azure](create-view-manage-system-topics.md#view-all-system-topics)
- Exporter les modèles Resource Manager des rubriques système et des abonnements aux événements dans le portail Azure
- [Configurer les journaux de diagnostic des rubriques système](enable-diagnostic-logs-topic.md#enable-diagnostic-logs-for-a-system-topic)
- Configurer des alertes en cas d’échec de publication et de livraison 

## <a name="lifecycle-of-system-topics"></a>Cycle de vie des rubriques système
Vous pouvez créer une rubrique système de deux manières : 

- Créez un [abonnement à un événement sur une ressource Azure en tant que ressource d’extension](/rest/api/eventgrid/version2020-06-01/eventsubscriptions/createorupdate), ce qui crée automatiquement une rubrique système dont le nom est au format suivant : `<Azure resource name>-<GUID>`. La rubrique système créée de cette façon est automatiquement supprimée lorsque le dernier abonnement à un événement de la rubrique est supprimé. 
- Créez une rubrique système pour une ressource Azure, puis créez un abonnement à un événement pour cette rubrique système. Lorsque vous utilisez cette méthode, vous pouvez spécifier un nom pour la rubrique système. La rubrique système n’est pas supprimée automatiquement lors de la suppression du dernier abonnement à un événement. Vous devez la supprimer manuellement. 

    Lorsque vous utilisez le portail Azure, vous utilisez toujours cette méthode. Lorsque vous créez un abonnement à un événement à l’aide de la [page **Événements** d’une ressource Azure](blob-event-quickstart-portal.md#subscribe-to-the-blob-storage), la rubrique système est créée en premier, puis l’abonnement pour la rubrique. Vous pouvez créer une rubrique système de manière explicite en utilisant la [page **Rubriques système Event Grid**](create-view-manage-system-topics.md#create-a-system-topic), puis créer un abonnement pour cette rubrique. 

Lorsque vous utilisez [CLI](create-view-manage-system-topics-cli.md), [REST](/rest/api/eventgrid/version2020-06-01/eventsubscriptions/createorupdate) ou un [modèle Resource Manager](create-view-manage-system-topics-arm.md), vous pouvez choisir l’une des méthodes ci-dessus. Nous vous recommandons de créer d’abord une rubrique système, puis de créer un abonnement sur la rubrique, car il s’agit de la méthode la plus récente de création de rubriques système.

La création d’une rubrique système échoue si vous avez configuré des stratégies Azure de telle sorte que le service Event Grid ne puisse pas la créer. Par exemple, vous pouvez avoir une stratégie qui autorise la création de certains types de ressources uniquement (par exemple : Stockage Azure, Azure Event Hubs, etc.) dans l’abonnement. 

## <a name="location-and-resource-group-for-a-system-topic"></a>Emplacement et groupe de ressources pour une rubrique système
Pour les sources d’événements Azure qui se trouvent dans une région/un emplacement spécifique, la rubrique système est créée au même endroit que la source d’événements Azure. Par exemple, si vous créez un abonnement aux événements pour un stockage Blob dans USA Est, la rubrique système est créée dans la région USA Est. Pour les sources d’événements Azure globales, comme les abonnements Azure, les groupes de ressources ou les Azure Maps, Event Grid crée la rubrique système dans l’emplacement **global**. 

En général, la rubrique système est créée dans le même groupe de ressources dans lequel se trouve la source d’événements Azure. Pour les abonnements aux événements créés dans l’étendue de l’abonnement Azure, la rubrique système est créée dans le groupe de ressources **Default-EventGrid** dans la région **USA Ouest 2**. Si le groupe de ressources n’existe pas, Azure Event Grid le crée avant de créer la rubrique système. 

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants : 

- [Créer, afficher et gérer des rubriques système à l’aide du portail Azure](create-view-manage-system-topics.md)
- [Créer, afficher et gérer des rubriques système Event Grid à l’aide d’Azure CLI](create-view-manage-system-topics-cli.md)
- [Créer des rubriques système Event Grid à l’aide de modèles Resource Manager](create-view-manage-system-topics-arm.md)
