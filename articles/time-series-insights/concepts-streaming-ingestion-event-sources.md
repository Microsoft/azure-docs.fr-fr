---
title: 'Sources d’événements d’ingestion de diffusion en continu : Azure Time Series Insights Gen2 | Microsoft Docs'
description: En savoir plus sur la diffusion en continu de données dans Azure Time Series Insights Gen2.
author: lyrana
ms.author: lyhughes
manager: deepakpalled
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 10/01/2020
ms.openlocfilehash: 4e83cca79a4dc99533ab17cca7e96e1ac802d598
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95020791"
---
# <a name="azure-time-series-insights-gen2-event-sources"></a>Sources d’événements Azure Time Series Insights Gen2

 Votre environnement Azure Time Series Insights Gen2 peut avoir jusqu’à deux sources d’événements de diffusion en continu. Deux types de ressources Azure sont pris en charge en tant qu’entrées :

- [Azure IoT Hub](../iot-hub/about-iot-hub.md)
- [Azure Event Hubs](../event-hubs/event-hubs-about.md)

Les événements doivent être envoyés en tant que JSON encodé en UTF-8.

## <a name="create-or-edit-event-sources"></a>Créer ou modifier des sources d’événements

Vos ressources de source d’événements peuvent être dans le même abonnement Azure que votre environnement Azure Time Series Insights Gen2 ou dans un autre abonnement. Vous pouvez utiliser [le portail Azure](./tutorials-set-up-tsi-environment.md#create-an-azure-time-series-insights-gen2-environment), [l’interface Azure CLI](https://github.com/Azure/azure-cli-extensions/tree/master/src/timeseriesinsights), [les modèles Resource Manager](time-series-insights-manage-resources-using-azure-resource-manager-template.md) et [l’API REST](/rest/api/time-series-insights/management(gen1/gen2)/eventsources) pour créer, modifier ou supprimer des sources d’événements de votre environnement.

Lorsque vous connectez une source d’événements, votre environnement Azure Time Series Insights Gen2 lit tous les événements stockés dans votre hub IoT ou Event Hub, en commençant par l’événement le plus ancien.

> [!IMPORTANT]
>
> - Vous pouvez rencontrer une latence initiale élevée lorsque vous connectez une source d’événement à votre environnement Azure Time Series Insights Gen2.
> - La latence de la source d’événement dépend du nombre d’événements actuellement présents dans votre IoT Hub ou votre Event Hub.
> - Une latence élevée sera impartie après la réception des données de la source d’événements. Si vous rencontrez une latence élevée, soumettez un ticket de support via le portail Azure.

## <a name="streaming-ingestion-best-practices"></a>Meilleures pratiques en matière d’ingestion de diffusion en continu

- Créez toujours un groupe de consommateurs unique pour votre environnement Azure Time Series Insights Gen2 afin de consommer des données de votre source d’événements. La réutilisation des groupes de consommateurs peut entraîner des déconnexions aléatoires et peut entraîner une perte de données.

- Configurez votre environnement Azure Time Series Insights Gen2 et votre hub IoT et/ou Event Hubs dans la même région Azure. Bien qu’il soit possible de configurer une source d’événement dans une région distincte, ce scénario n’est pas pris en charge et nous ne pouvons pas garantir une haute disponibilité.

- N’allez pas au-delà de la [limite du débit](./concepts-streaming-ingress-throughput-limits.md) de votre environnement ou de la limite de partition.

- Configurez une [alerte](./time-series-insights-environment-mitigate-latency.md#monitor-latency-and-throttling-with-alerts) de latence pour être averti si votre environnement rencontre des problèmes de traitement des données.

- Utilisez l’ingestion de streaming uniquement pour des données récentes et en quasi-temps réel. Les données historiques de streaming ne sont pas prises en charge.

- Vous devez comprendre la manière dont les propriétés sont placées dans une séquence d’échappement et comment les données JSON [aplaties et stockées.](./concepts-json-flattening-escaping-rules.md)

- Suivez le principe des privilèges minimum lorsque vous fournissez des chaînes de connexion à la source d’événements. Pour Event Hubs, configurez une stratégie d’accès partagé avec l’*envoi* de revendication uniquement, et pour IoT Hub utilisez l’autorisation *service Connect* uniquement.

### <a name="historical-data-ingestion"></a>Ingestion de données historiques

L’utilisation du pipeline de streaming pour importer des données historiques n’est actuellement pas prise en charge dans Azure Time Series Insights Gen2. Si vous devez importer des données passées dans votre environnement, suivez les instructions ci-dessous :

- Ne diffusez pas parallèlement des données historiques et dynamiques. L’ingestion de données dans le désordre entraînera une dégradation des performances de requête.
- Ingérez les données historiques de façon chronologique pour obtenir des performances optimales.
- Restez dans les limites du débit d’ingestion ci-dessous.
- Désactivez le magasin Warm si les données sont antérieures à la période de rétention du magasin Warm.

## <a name="event-source-timestamp"></a>Horodateur de la source de l’événement

Quand vous configurez une source d’événement, vous êtes invité à fournir une propriété d’ID d’horodatage. La propriété d’horodatage est utilisée pour effectuer le suivi des événements dans le temps. Il s’agit de l’heure qui sera utilisée comme $event.$ts dans les [API de requête](/rest/api/time-series-insights/dataaccessgen2/query/execute) et pour tracer des séries dans Explorateur Azure Time Series Insights. Si aucune propriété n’est fournie au moment de la création, ou si la propriété d’horodatage est absente d’un événement, IoT Hub ou Event Hubs sont utilisés par défaut. Les valeurs de propriété d’horodatage sont stockées au format UTC.

En général, les utilisateurs choisissent de personnaliser la propriété d’horodatage et utilisent l’heure à laquelle le capteur ou la balise a généré la lecture au lieu d’utiliser le hub par défaut mis en file d’attente. Cela est particulièrement nécessaire lorsque les appareils présentent une perte de connectivité intermittente et qu’un lot de messages retardés est transféré à Azure Time Series Insights Gen2.

Si votre horodateur personnalisé se trouve dans un objet JSON imbriqué ou un tableau, vous devrez fournir le nom de propriété correct en suivant nos conventions d’affectation de noms [de mise à plat et d’échappement](concepts-json-flattening-escaping-rules.md). Par exemple, l’horodateur de la source de l’événement pour la charge utile JSON indiquée [ici](concepts-json-flattening-escaping-rules.md#example-a) doit être saisi ainsi : `"values.time"`.

### <a name="time-zone-offsets"></a>Décalage de fuseau horaire

Les horodateurs doivent être envoyés au format ISO 8601 et seront stockés au format UTC. Si un décalage de fuseau horaire est fourni, l’offset est appliqué, puis l’heure est stockée et retournée au format UTC. Si le format du décalage est incorrect, il sera ignoré. Dans les situations où votre solution n’a peut-être pas le contexte du décalage d’origine, vous pouvez envoyer les données de décalage dans une propriété d’événement distincte supplémentaire pour vous assurer qu’elles sont conservées et que votre application peut faire référence à une réponse de requête.

Le décalage de fuseau horaire doit être mis en forme de l’une des manières suivantes :

±HHMMZ</br>
±HH:MM</br>
±HH:MMZ</br>

## <a name="next-steps"></a>Étapes suivantes

- Lisez les [Règles de mise à plat et d’échappement JSON](./concepts-json-flattening-escaping-rules.md) pour comprendre comment les événements seront stockés.

- Comprendre les [limites de débit](./concepts-streaming-ingress-throughput-limits.md) de votre environnement