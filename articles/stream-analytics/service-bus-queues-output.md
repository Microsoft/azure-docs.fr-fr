---
title: Sortie des files d’attente Service Bus à partir d’Azure Stream Analytics
description: Cet article décrit les files d’attente Service Bus en tant que sortie pour Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: 3527bdbbf1ba337d24dc390f97ed090c8dfbe039
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93348387"
---
# <a name="service-bus-queues-output-from-azure-stream-analytics"></a>Sortie des files d’attente Service Bus à partir d’Azure Stream Analytics

Les [files d’attente Service Bus](../service-bus-messaging/service-bus-queues-topics-subscriptions.md) permettent de livrer des messages FIFO à un ou plusieurs destinataires concurrents. En règle générale, les messages sont reçus et traités par les destinataires dans l’ordre dans lequel ils ont été ajoutés à la file d’attente. Chaque message est reçu et traité par un seul destinataire du message.

Dans le [niveau de compatibilité 1.2](stream-analytics-compatibility-level.md), Azure Stream Analytics utilise le protocole de messagerie [AMQP (Advanced Message Queueing Protocol)](../service-bus-messaging/service-bus-amqp-overview.md) pour écrire dans les files d’attente et les rubriques Service Bus. Le protocole AMQP vous permet de développer des applications hybrides interplateforme à l’aide d’un protocole open standard.

## <a name="output-configuration"></a>Configuration de la sortie

Le tableau suivant répertorie les noms de propriétés et leur description pour la création d’une sortie de file d’attente.

| Nom de la propriété | Description |
| --- | --- |
| Alias de sortie |Nom convivial utilisé dans les requêtes pour diriger la sortie de requête vers cette file d’attente Service Bus. |
| Espace de noms Service Bus |Conteneur pour un ensemble d’entités de messagerie. |
| Nom de la file d'attente |Nom de la file d’attente Service Bus. |
| Nom de la stratégie de file d’attente |Lorsque vous créez une file d’attente, vous pouvez également créer des stratégies d’accès partagé dans l’onglet **Configurer** de la file d’attente. Chaque stratégie d’accès partagé a un nom, les autorisations que vous définissez ainsi que des clés d’accès. |
| Clé de la stratégie de file d’attente |Clé d’accès partagé qui permet d’authentifier l’accès à l’espace de noms Service Bus. |
| Format de sérialisation de l’événement |Format de sérialisation pour les données de sortie. JSON, CSV et Avro sont pris en charge. |
| Encodage |Pour CSV et JSON, UTF-8 est le seul format d’encodage actuellement pris en charge. |
| Délimiteur |Applicable uniquement pour la sérialisation CSV. Stream Analytics prend en charge un certain nombre de délimiteurs communs pour sérialiser des données dans un format CSV. Valeurs prises en charge : virgule, point-virgule, espace, tabulation et barre verticale. |
| Format |Applicable uniquement pour le type JSON. L’expression **Séparé par une ligne** indique que la sortie est mise en forme de sorte que tous les objets JSON soient séparés par une nouvelle ligne. Si vous sélectionnez **Séparé par une ligne** , le JSON est lu un objet à la fois. Le contenu entier seul ne serait pas un JSON valide. Le terme **Tableau** indique que la sortie est mise en forme en tant que tableau d’objets JSON. |
| Colonnes de propriété | facultatif. Colonnes séparées par des virgules qui doivent être jointes en tant que propriétés de l’utilisateur du message sortant au lieu de la charge utile. Vous trouverez plus d’informations sur cette fonctionnalité dans la section [Propriétés de métadonnées personnalisées pour la sortie](#custom-metadata-properties-for-output). |
| Colonnes de propriétés système | facultatif. Paires clé/valeur de propriétés système et noms de colonnes correspondants qui doivent être attachés au message sortant au lieu de la charge utile.  |

Le nombre de partitions est [basé sur la référence Service Bus et sa taille](../service-bus-messaging/service-bus-partitioning.md). La clé de partition est une valeur entière unique pour chaque partition.

## <a name="partitioning"></a>Partitionnement

Le partitionnement est choisi automatiquement. Le nombre de partitions est basé sur la [référence Service Bus et sa taille](../service-bus-messaging/service-bus-partitioning.md). La clé de partition est une valeur entière unique pour chaque partition. Le nombre de writers de sortie est le même que le nombre de partitions dans la file d’attente de sortie.

## <a name="output-batch-size"></a>Taille de lot de sortie

La taille maximale du message est de 256 Ko par message pour le niveau Standard et 1 Mo pour le niveau Premium. Pour plus d’informations, consultez [Limites de Service Bus](../service-bus-messaging/service-bus-quotas.md). Pour optimiser, utilisez un seul événement par message.

## <a name="custom-metadata-properties-for-output"></a>Propriétés de métadonnées personnalisées pour la sortie

Vous pouvez joindre des colonnes de requête en tant que propriétés de l’utilisateur aux messages sortants. Ces colonnes ne font pas partie de la charge utile. Les propriétés sont présentes sous la forme d’un dictionnaire sur le message de sortie. *Clé* est le nom de colonne et *valeur* est la valeur de la colonne dans le dictionnaire de propriétés. Tous les types de données Stream Analytics sont pris en charge à l’exception des enregistrements et des tableaux.

Dans l’exemple suivant, les champs `DeviceId` et `DeviceStatus` sont ajoutés aux métadonnées.

1. Utilisez la requête suivante :

   ```sql
   select *, DeviceId, DeviceStatus from iotHubInput
   ```

1. Configurez `DeviceId,DeviceStatus` comme colonnes de propriété dans la sortie.

   :::image type="content" source="media/service-bus-queues-output/property-columns.png" alt-text="Colonnes de propriété":::

L’illustration suivante présente les propriétés de message de sortie attendues qui sont inspectées dans un Event Hub à l’aide de [Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer).

:::image type="content" source="media/service-bus-queues-output/custom-properties.png" alt-text="Propriétés personnalisées d’événement":::

## <a name="system-properties"></a>Propriétés système

Vous pouvez attacher des colonnes de requête en tant que [propriétés système](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage?view=azure-dotnet&preserve-view=true#properties) à vos messages sortants de file d’attente ou de rubrique Service Bus.

Ces colonnes ne sont pas placées dans la charge utile ; au lieu de cela, la [propriété système](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage?view=azure-dotnet&preserve-view=true#properties) BrokeredMessage correspondante est renseignée avec les valeurs de la colonne de requête.
Les propriétés système suivantes sont prises en charge : `MessageId, ContentType, Label, PartitionKey, ReplyTo, SessionId, CorrelationId, To, ForcePersistence, TimeToLive, ScheduledEnqueueTimeUtc`.

Les valeurs de chaîne de ces colonnes sont analysées en tant que type de valeur de propriété système correspondante et les échecs d’analyse sont traités comme des erreurs de données.
Ce champ est fourni sous la forme d’un format d’objet JSON. Les détails relatifs à ce format sont les suivants :

* Entouré par des accolades {}.
* Écrit dans des paires clé-valeur.
* Les clés et les valeurs doivent être des chaînes.
* La clé est le nom de la propriété système et la valeur est le nom de la colonne de requête.
* Les clés et les valeurs sont séparées par un signe deux-points.
* Chaque paire clé-valeur est séparée par une virgule.

Voici comment utiliser cette propriété :

* Requête : `select *, column1, column2 INTO queueOutput FROM iotHubInput`
* Colonnes de propriétés système : `{ "MessageId": "column1", "PartitionKey": "column2"}`

Cela définit le `MessageId` sur les messages de la file d’attente Service bus avec les valeurs de `column1`, et PartitionKey est défini avec les valeurs de `column2`.

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md)
* [Démarrage rapide : Créer un travail Azure Stream Analytics avec l’interface de ligne de commande Azure](quick-create-azure-cli.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics à l’aide d’un modèle Resource Manager](quick-create-azure-resource-manager.md)
* [Démarrage rapide : Créer un travail Stream Analytics à l’aide d’Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics à l’aide de Visual Studio](stream-analytics-quick-create-vs.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics dans Visual Studio Code](quick-create-visual-studio-code.md)