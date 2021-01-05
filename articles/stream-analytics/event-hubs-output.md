---
title: Event Hubs à partir d’Azure Stream Analytics
description: Cet article explique comment sortir des données de Azure Stream Analytics vers Azure Event Hubs.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: e01a4418681e0f78864eacbf70016cfb33fa6f53
ms.sourcegitcommit: 44844a49afe8ed824a6812346f5bad8bc5455030
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97739635"
---
# <a name="event-hubs-output-from-azure-stream-analytics"></a>Event Hubs à partir d’Azure Stream Analytics

Le service [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/) est un ingesteur d’événements de publication/abonnement hautement évolutif. Il peut collecter des millions d’événements par seconde. Un hub d’événements peut être utilisé en tant que sortie lorsque la sortie d’un travail Stream Analytics correspond à l’entrée d’un autre travail de diffusion en continu. Pour obtenir des informations sur la taille de message maximale et sur l’optimisation de la taille de lot, consultez la section [taille de lot de sortie](#output-batch-size).

## <a name="output-configuration"></a>Configuration de la sortie

La table suivante a les paramètres nécessaires pour configurer des flux de données à partir d’Event Hubs en tant que sortie.

| Nom de la propriété | Description |
| --- | --- |
| Alias de sortie | Nom convivial utilisé dans les requêtes pour diriger la sortie de requête vers cet Event Hub. |
| Espace de noms du hub d’événements | Conteneur pour un ensemble d’entités de messagerie. En créant un Event Hub, vous avez également créé un espace de noms Event Hub. |
| Nom du hub d’événements | Nom de votre sortie Event Hub. |
| Nom de la stratégie du hub d’événements | Stratégie d’accès partagé que vous pouvez créer dans l’onglet **Configurer** de l’Event Hub. Chaque stratégie d’accès partagé a un nom, les autorisations que vous définissez ainsi que des clés d’accès. |
| Clé de la stratégie du Event Hub | Clé d’accès partagé utilisée pour authentifier l’accès à l’espace de noms Event Hub. |
| Colonne de clé de partition | facultatif. Colonne qui contient la clé de partition pour la sortie Event Hub. |
| Format de sérialisation de l’événement | Format de sérialisation pour les données de sortie. JSON, CSV et Avro sont pris en charge. |
| Encodage | Pour CSV et JSON, UTF-8 est le seul format d’encodage actuellement pris en charge. |
| Délimiteur | Applicable uniquement pour la sérialisation CSV. Stream Analytics prend en charge un certain nombre de délimiteurs communs pour sérialiser des données dans un format CSV. Valeurs prises en charge : virgule, point-virgule, espace, tabulation et barre verticale. |
| Format | Applicable uniquement pour la sérialisation JSON. L’expression **Séparé par une ligne** indique que la sortie est mise en forme de sorte que tous les objets JSON soient séparés par une nouvelle ligne. Si vous sélectionnez **Séparé par une ligne**, le JSON est lu un objet à la fois. Le contenu entier seul ne serait pas un JSON valide. Le terme **Tableau** indique que la sortie est mise en forme en tant que tableau d’objets JSON.  |
| Colonnes de propriété | facultatif. Colonnes séparées par des virgules qui doivent être jointes en tant que propriétés de l’utilisateur du message sortant au lieu de la charge utile. Vous trouverez plus d’informations sur cette fonctionnalité dans la section [Propriétés de métadonnées personnalisées pour la sortie](#custom-metadata-properties-for-output). |

## <a name="partitioning"></a>Partitionnement

Le partitionnement varie en fonction de l’alignement des partitions. Lorsque la clé de partition de la sortie Event Hub s’aligne parfaitement avec l’étape de requête (précédente) en amont, le nombre d’enregistreurs est égal au nombre de partitions dans la sortie Event Hub. Chaque enregistreur utilise la [classe EventHubSender](/dotnet/api/microsoft.servicebus.messaging.eventhubsender?view=azure-dotnet&preserve-view=true) pour envoyer des événements à la partition concernée. Lorsque la clé de partition de la sortie Event Hub ne s’aligne pas avec l’étape de requête (précédente) en amont, le nombre d’enregistreurs est égal au nombre de partitions de cette précédente étape. Chaque enregistreur utilise la [classe SendBatchAsync](/dotnet/api/microsoft.servicebus.messaging.eventhubclient.sendasync?view=azure-dotnet&preserve-view=true) dans **EventHubClient** pour envoyer des événements à toutes les partitions de sortie. 

## <a name="output-batch-size"></a>Taille de lot de sortie

La taille maximale du message est de 256 Ko ou 1 Mo par message. Pour plus d'informations, consultez les [limites Event Hubs](../event-hubs/event-hubs-quotas.md). Lorsque le partitionnement entrée/sortie ne s’aligne pas, chaque événement est empaqueté individuellement dans `EventData` et envoyé dans un lot dans la limite de la taille de message maximale. Cela se produit également si les [propriétés de métadonnées personnalisées](#custom-metadata-properties-for-output) sont utilisées. Lorsque le partitionnement entrée/sortie est aligné, plusieurs événements sont empaquetés dans une seule instance `EventData`, dans la limite de la taille de message maximale, et envoyés.

## <a name="custom-metadata-properties-for-output"></a>Propriétés de métadonnées personnalisées pour la sortie

Vous pouvez joindre des colonnes de requête en tant que propriétés de l’utilisateur aux messages sortants. Ces colonnes ne font pas partie de la charge utile. Les propriétés sont présentes sous la forme d’un dictionnaire sur le message de sortie. *Clé* est le nom de colonne et *valeur* est la valeur de la colonne dans le dictionnaire de propriétés. Tous les types de données Stream Analytics sont pris en charge à l’exception des enregistrements et des tableaux.

Dans l’exemple suivant, les champs `DeviceId` et `DeviceStatus` sont ajoutés aux métadonnées.

1. Utilisez la requête suivante :

   ```sql
   select *, DeviceId, DeviceStatus from iotHubInput
   ```

1. Configurez `DeviceId,DeviceStatus` comme colonnes de propriété dans la sortie.

   :::image type="content" source="media/event-hubs-output/property-columns.png" alt-text="Colonnes de propriété":::

L’illustration suivante présente les propriétés de message de sortie attendues qui sont inspectées dans un Event Hub à l’aide de [Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer).

:::image type="content" source="media/event-hubs-output/custom-properties.png" alt-text="Propriétés personnalisées d’événement":::

## <a name="next-steps"></a>Étapes suivantes

* [Accès à Event Hubs à partir d’un travail Azure Stream Analytics à l’aide des identités managées (préversion)](event-hubs-managed-identity.md)
* [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md)
