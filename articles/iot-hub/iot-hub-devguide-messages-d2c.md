---
title: Comprendre le routage des messages IoT Hub | Microsoft Docs
description: 'Guide du développeur : Comment utiliser le routage de messages pour envoyer des messages appareil-à-cloud. Inclut des informations sur l’envoi de données de télémétrie et d’autres données.'
author: ash2017
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/15/2019
ms.author: asrastog
ms.custom:
- 'Role: Cloud Development'
- devx-track-csharp
ms.openlocfilehash: 64821819530e142eb207c001d3e3ccfe349cf917
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547773"
---
# <a name="use-iot-hub-message-routing-to-send-device-to-cloud-messages-to-different-endpoints"></a>Utiliser le routage des messages IoT Hub pour envoyer des messages appareil-à-cloud à différents points de terminaison

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Le routage des messages vous permet d’envoyer des messages de vos appareils vers des services cloud de façon automatisée, évolutive et fiable. Le routage des messages peut être utilisé pour : 

* **L'envoi des événements et des messages relatifs aux données de télémétrie des appareils** , c'est-à-dire les événements de cycle de vie des appareils et les événements de modification des jumeaux d'appareil aux points de terminaison intégrés et aux de points de terminaison personnalisés. Découvrez plus d’informations sur les [points de terminaison de routage](#routing-endpoints).

* **Le filtrage des données avant leur routage vers différents points de terminaison** en appliquant des requêtes avancées. Le routage de messages vous permet d’interroger les propriétés et le corps d’un message, ainsi que les étiquettes et les propriétés d’un jumeau d’appareil. Découvrez plus d’informations sur l’utilisation de [requêtes dans le routage des messages](iot-hub-devguide-routing-query-syntax.md).

IoT Hub doit pouvoir accéder en écriture à ces points de terminaison de service pour que le routage des messages fonctionne. Si vous configurez vos points de terminaison via le portail Azure, les autorisations nécessaires sont ajoutées pour vous. Veillez à configurer vos services pour prendre en charge le débit prévu. Par exemple, si vous utilisez Event Hubs comme point de terminaison personnalisé, vous devez configurer les **unités de débit** pour ce hub d’événements afin qu’il puisse gérer l’entrée des événements que vous envisagez d’envoyer via le routage des messages IoT Hub. De même, lors de l’utilisation d’une file d’attente Service Bus comme point de terminaison, vous devez configurer la **taille maximale** afin que la file d’attente puisse contenir toutes les données entrantes, jusqu’à ce qu’elles soient sorties par les consommateurs. Lorsque vous configurez votre solution IoT pour la première fois, vous devrez peut-être surveiller vos points de terminaison supplémentaires et apporter les modifications nécessaires en fonction de la charge réelle.

IoT Hub définit un [format commun](iot-hub-devguide-messages-construct.md) pour tous les envois de messages appareil-à-cloud à des fins d'interopérabilité entre les protocoles. Si un message correspond à plusieurs routes qui pointent vers le même point de terminaison, IoT Hub ne remet ce message qu’une seule fois à ce point de terminaison. Par conséquent, vous n’avez pas besoin de configurer une déduplication sur votre file d’attente ou votre rubrique Service Bus. Dans les files d’attente partitionnées, l’affinité de la partition assure le classement des messages. Utilisez ce tutoriel pour découvrir comment [configurer le routage des messages](tutorial-routing.md).

## <a name="routing-endpoints"></a>Points de terminaison de routage

Un hub IoT a un point de terminaison intégré par défaut ( **messages/événements** ) qui est compatible avec Event Hubs. Vous pouvez créer des [points de terminaison personnalisés](iot-hub-devguide-endpoints.md#custom-endpoints) pour y router les messages en liant d’autres services de votre abonnement au hub IoT. 

Chaque message est routé vers tous les points de terminaison dont il correspond aux requêtes de routage. En d’autres termes, un message peut être routé vers plusieurs points de terminaison.

Si votre point de terminaison personnalisé a des configurations de pare-feu, envisagez d’utiliser l’exception interne approuvée de Microsoft pour accorder à votre hub IoT l’accès au point de terminaison spécifique : le [Stockage Azure](./virtual-network-support.md#egress-connectivity-to-storage-account-endpoints-for-routing), [Azure Event Hubs](./virtual-network-support.md#egress-connectivity-to-event-hubs-endpoints-for-routing) et [Azure Service Bus](./virtual-network-support.md#egress-connectivity-to-service-bus-endpoints-for-routing). Cette option est disponible uniquement dans certaines régions pour les hubs IoT avec une [identité de service managé](./virtual-network-support.md).

IoT Hub prend actuellement en charge les points de terminaison suivants :

 - Point de terminaison intégré
 - Stockage Azure
 - Files d’attente et rubriques Service Bus
 - Event Hubs

## <a name="built-in-endpoint-as-a-routing-endpoint"></a>Point de terminaison intégré en tant que point de terminaison de routage

Vous pouvez utiliser [l’intégration et les SDK standard Event Hubs](iot-hub-devguide-messages-read-builtin.md) pour recevoir des messages appareil-à-cloud du point de terminaison intégré ( **messages/événements** ). Une fois qu’une route est créée, les données cessent de circuler vers le point de terminaison intégré, sauf si une route est créée vers ce point de terminaison.

## <a name="azure-storage-as-a-routing-endpoint"></a>Stockage Azure en tant que point de terminaison de routage

Il existe deux services de stockage vers lesquels IoT Hub peut acheminer des messages : les comptes de [stockage d’objets Blob Azure](../storage/blobs/storage-blobs-introduction.md) et [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md) (ADLS Gen2). Les comptes Azure Data Lake Storage sont des comptes de stockage compatibles avec [l’espace de noms hiérarchique](../storage/blobs/data-lake-storage-namespace.md) basés sur le stockage d’objets Blob. Les deux utilisent des objets Blob pour leur stockage.

IoT Hub prend en charge l’écriture de données sur Stockage Azure au format [Apache Avro](https://avro.apache.org/) ainsi qu’au format JSON. La valeur par défaut est AVRO. Quand vous utilisez l’encodage JSON, vous devez définir contentType sur **application/json** et contentEncoding sur **UTF-8** dans les [propriétés système](iot-hub-devguide-routing-query-syntax.md#system-properties) du message. Ces deux valeurs sont insensibles à la casse. Si ce codage n’est pas défini, IoT Hub écrit les messages dans un format codé base-64.

Le format d’encodage ne peut être défini qu’au moment de configurer le point de terminaison du stockage de blobs ; il ne peut pas être modifié pour un point de terminaison existant. Pour changer les formats d’encodage d’un point de terminaison existant, vous devez supprimer et recréer le point de terminaison personnalisé avec le format souhaité. Une stratégie utile peut consister à créer un nouveau point de terminaison personnalisé avec le format d’encodage souhaité et à ajouter un itinéraire parallèle à ce point de terminaison. De cette façon, vous pouvez vérifier vos données avant de supprimer le point de terminaison existant.

Vous pouvez sélectionner le format d’encodage à l’aide de l’API REST de création ou de mise à jour d’IoT Hub, en particulier [RoutingStorageContainerProperties](/rest/api/iothub/iothubresource/createorupdate#routingstoragecontainerproperties), le portail Azure, [Azure CLI](/cli/azure/iot/hub/routing-endpoint) ou [Azure PowerShell](/powershell/module/az.iothub/add-aziothubroutingendpoint). L’image suivante illustre la façon de sélectionner le format d’encodage dans le portail Azure.

![Encodage du point de terminaison de stockage Blob](./media/iot-hub-devguide-messages-d2c/blobencoding.png)

IoT Hub regroupe les messages dans des lots et écrit les données dans un stockage quand le lot atteint une certaine taille ou après un certain laps de temps. IoT Hub utilise par défaut la convention d’affectation de noms de fichiers suivante :

```
{iothub}/{partition}/{YYYY}/{MM}/{DD}/{HH}/{mm}
```

Vous pouvez utiliser n’importe quelle convention de nommage des fichiers, mais vous devez utiliser tous les jetons listés. IoT Hub écrit dans un objet blob vide s’il n’y a aucune donnée à écrire.

Nous vous recommandons de répertorier les objets blob ou les fichiers, puis d’exécuter une itération sur ces derniers, afin de garantir que tous les objets blob ou les fichiers seront lus, sans avoir à faire de suppositions concernant la partition. La plage de partition peut changer pendant un [basculement initié par Microsoft](iot-hub-ha-dr.md#microsoft-initiated-failover) ou pendant un [basculement manuel](iot-hub-ha-dr.md#manual-failover) IoT Hub. Vous pouvez utiliser l’[API Lister les blobs](/rest/api/storageservices/list-blobs) pour énumérer la liste des objets blob ou l’[API Lister ADLS Gen2 API](/rest/api/storageservices/datalakestoragegen2/path/list) pour lister les fichiers. Consultez l’exemple suivant à titre de conseils et d’aide.

```csharp
public void ListBlobsInContainer(string containerName, string iothub)
{
    var storageAccount = CloudStorageAccount.Parse(this.blobConnectionString);
    var cloudBlobContainer = storageAccount.CreateCloudBlobClient().GetContainerReference(containerName);
    if (cloudBlobContainer.Exists())
    {
        var results = cloudBlobContainer.ListBlobs(prefix: $"{iothub}/");
        foreach (IListBlobItem item in results)
        {
            Console.WriteLine(item.Uri);
        }
    }
}
```

Pour créer un compte de stockage compatible Azure Data Lake Gen2, créez un nouveau compte de stockage v2 et sélectionnez *activé* dans le champ *Espace de noms hiérarchique* de l’onglet **Avancé** , comme indiqué dans l’image suivante :

![Sélectionner le stockage Azure Data Lake Gen2](./media/iot-hub-devguide-messages-d2c/selectadls2storage.png)

## <a name="service-bus-queues-and-service-bus-topics-as-a-routing-endpoint"></a>Files d’attente et rubriques Service Bus en tant que point de terminaison de routage

Les options **Sessions** ou **Détection des doublons** ne doivent pas être activées pour les files d’attente et rubriques Service Bus utilisées comme points de terminaison IoT Hub. Si l’une de ces options est activée, le point de terminaison s’affiche comme **Inaccessible** dans le portail Azure.

## <a name="event-hubs-as-a-routing-endpoint"></a>Event Hubs en tant que point de terminaison de routage

En plus du point de terminaison compatible Event Hubs intégré, vous pouvez router des données vers des points de terminaison personnalisés de type Event Hubs. 

## <a name="reading-data-that-has-been-routed"></a>Lecture de données qui ont été routées

Vous pouvez configurer une route en suivant ce [tutoriel](tutorial-routing.md).

Utilisez les tutoriels suivants pour découvrir comment lire un message à partir d’un point de terminaison.

* Lecture à partir du [point de terminaison intégré](quickstart-send-telemetry-node.md)

* Lecture à partir du [Stockage Blob](../storage/blobs/storage-blob-event-quickstart.md)

* Lecture à partir [d’Event Hubs](../event-hubs/event-hubs-dotnet-standard-getstarted-send.md)

* Lecture à partir de [files d’attente Service Bus](../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md)

* Lecture à partir de [rubriques Service Bus](../service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions.md)


## <a name="fallback-route"></a>Itinéraire de secours

La route de secours envoie tous les messages qui ne satisfont pas aux conditions de la requête sur une des routes existantes aux hubs d’événements existants ( **messages/événements** ), compatible avec [Event Hubs](../event-hubs/index.yml). Si le routage des messages est activé, vous pouvez activer la fonctionnalité de route de secours. Une fois qu’une route est créée, les données cessent de circuler vers le point de terminaison intégré, sauf si une route est créée vers ce point de terminaison. S’il n’existe pas de route vers le point de terminaison intégré et qu’une route de secours est activée, seuls les messages qui ne correspondent pas aux conditions de la requête sur les routes sont envoyées au point de terminaison intégré. En outre, si toutes les routes existantes sont supprimées, la route de secours doit être activée pour recevoir toutes les données sur le point de terminaison intégré.

Vous pouvez activer/désactiver la route de secours dans le portail Azure -> Panneau Routage des messages. Vous pouvez également utiliser Azure Resource Manager pour que [FallbackRouteProperties](/rest/api/iothub/iothubresource/createorupdate#fallbackrouteproperties) utilise un point de terminaison personnalisé pour la route de secours.

## <a name="non-telemetry-events"></a>Événements autres que les événements de télémétrie

En plus de la télémétrie des appareils, le routage des messages permet également l’envoi d’événements de modification des jumeaux d’appareil, des événements du cycle de vie des appareils et des événements de modification du jumeau numérique. Par exemple, si une route est créée avec la source de données définie sur **Événements de modification de jumeau d’appareil** , IoT Hub envoie les messages au point de terminaison qui contient la modification du jumeau d’appareil. De même, si une route est créée avec la source de données définie sur **Événements de cycle de vie d’appareil** , IoT Hub envoie un message qui indique si l’appareil a été supprimé ou créé. Enfin, dans le cadre d’ [IOT Plug-and-Play](../iot-pnp/overview-iot-plug-and-play.md), un développeur peut créer des itinéraires avec une source de données définie sur les **événements de modification du jumeau numérique** et IOT Hub envoie des messages chaque fois qu’une [propriété](../iot-pnp/iot-plug-and-play-glossary.md) de jumeau numérique est définie ou modifiée, qu’un [jumeau numérique](../iot-pnp/iot-plug-and-play-glossary.md) est remplacé ou lorsqu’un événement de modification se produit pour le jumeau d'appareil sous-jacent.

[IoT Hub s’intègre également à Azure Event Grid](iot-hub-event-grid.md) pour publier des événements d’appareils et prendre en charge les intégrations en temps réel ainsi que l’automatisation des workflows basés sur ces événements. Pour découvrir ce qui convient le mieux à votre scénario, consultez les [différences principales entre le routage des messages et Event Grid](iot-hub-event-grid-routing-comparison.md).

## <a name="testing-routes"></a>Test des routes

Quand vous créez un nouvelle route ou que modifiez une route existante, vous devez tester la requête de route avec un exemple de message. Vous pouvez tester des routes individuelles ou toutes les routes à la fois. Aucun message n’est routé vers les points de terminaison pendant le test. Vous pouvez utiliser le portail Azure, Azure Resource Manager, Azure PowerShell et Azure CLI pour les tests. Les résultats vous permettent de déterminer si l’exemple de message correspondait ou non à la requête, ou si le test n’a pas pu s’exécuter, l’exemple de message ou la syntaxe de la requête étant incorrects. Pour plus d’informations, consultez [Tester une route](/rest/api/iothub/iothubresource/testroute) et [Tester toutes les routes](/rest/api/iothub/iothubresource/testallroutes).

## <a name="ordering-guarantees-with-at-least-once-delivery"></a>L’ordonnancement garantit au moins une remise

Le routage des messages IoT Hub garantit au moins une remise de messages aux points de terminaison. Cela signifie qu’il peut y avoir des messages en double et qu’une série de messages peuvent être retransmis en respectant l’ordre d’origine des messages. Par exemple, si l’ordre d’origine des messages est [1, 2, 3, 4], vous pouvez recevoir une séquence de messages telle que [1, 2, 1, 2, 3, 1, 2, 3, 4]. La garantie d’ordonnancement fait que, si vous recevez le message [1], il est toujours suivi de [2, 3, 4].

Pour gérer les doublons, nous vous recommandons de marquer un identificateur unique dans les propriétés de l’application du message au point d’origine, qui est généralement un appareil ou un module. Le service consommant les messages peut gérer les messages en double à l’aide de cet identificateur.

## <a name="latency"></a>Latence

Lorsque vous routez des messages de télémétrie appareil-à-cloud à l’aide de points de terminaison intégrés, une légère augmentation de la latence de bout en bout se produit après la création du premier itinéraire.

Dans la plupart des cas, l’augmentation moyenne de la latence est inférieure à 500 ms. Vous pouvez surveiller la latence avec **Routage : latence des messages de messages/d’événements** ou avec la métrique IoT Hub **d2c.endpoints.latency.builtIn.events**. La création ou la suppression d’un itinéraire après celle du premier itinéraire n’affecte pas la latence de bout en bout.

## <a name="monitoring-and-troubleshooting"></a>Surveillance et dépannage

IoT Hub fournit plusieurs métriques liées aux routages et aux point de terminaison pour vous donner une vue d’ensemble de l’intégrité de votre hub et des messages envoyés. Pour obtenir la liste de toutes les métriques IoT Hub décomposées par catégorie fonctionnelle, consultez [Informations de référence sur les données de supervision – Métriques](monitor-iot-hub-reference.md#metrics). Vous pouvez suivre les erreurs qui se produisent lors de l’évaluation d’une requête de routage et de l’intégrité du point de terminaison telles que perçues par IoT Hub grâce à la [catégorie **Itinéraires** dans les journaux de diagnostic d’IoT Hub](monitor-iot-hub-reference.md#routes). Pour en savoir plus sur l’utilisation des métriques et des journaux de ressources avec IoT Hub, consultez [Superviser avec IoT Hub](monitor-iot-hub.md).

Vous pouvez utiliser l’API REST [Obtenir l’intégrité du point de terminaison](/rest/api/iothub/iothubresource/getendpointhealth#iothubresource_getendpointhealth) pour récupérer l’[état d’intégrité](iot-hub-devguide-endpoints.md#custom-endpoints) des points de terminaison.

Pour obtenir plus d’informations et de l’aide sur le résolution des problèmes liés au routage, utilisez le [Guide de résolution des problèmes concernant le routage](troubleshoot-message-routing.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour découvrir comment créer des routes de messages, consultez [Traiter les messages appareil-à-cloud IoT Hub avec des routes](tutorial-routing.md).

* [Guide pratique pour envoyer des messages appareil-à-cloud](quickstart-send-telemetry-node.md)

* Pour plus d’informations sur les SDK que vous pouvez utiliser pour envoyer des messages appareil-à-cloud, consultez [SDK Azure IoT](iot-hub-devguide-sdks.md).