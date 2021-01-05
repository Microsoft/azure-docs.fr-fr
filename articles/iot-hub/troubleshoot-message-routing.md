---
title: Résoudre les problèmes de routage des messages Azure IoT
description: Comment résoudre les problèmes liés au routage de messages Azure IoT
author: ash2017
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/06/2020
ms.author: asrastog
ms.openlocfilehash: 29127a9dff42c0f733e3721d1ea5fea7350e774e
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547353"
---
# <a name="troubleshooting-message-routing"></a>Résoudre les problèmes de routage des messages

Cet article apporte des conseils de surveillance et de dépannage pour les problèmes courants et la résolution des problèmes liés au [routage des messages](iot-hub-devguide-messages-d2c.md) IoT Hub.

## <a name="monitoring-message-routing"></a>Surveiller le routage des messages

Nous vous recommandons de surveiller [les mesures IoT Hub relatives au routage des messages et aux points de terminaison](monitor-iot-hub-reference.md#routing-metrics) pour obtenir une vue d’ensemble des messages envoyés. Vous pouvez également créer un paramètre de diagnostic pour envoyer des opérations pour des [**itinéraires** dans des journaux de ressources IoT Hub](monitor-iot-hub-reference.md#routes) aux journaux Azure Monitor, à Event Hubs ou au stockage Azure en vue d’un traitement personnalisé. Pour en savoir plus sur l’utilisation des métriques, des journaux de ressources et des paramètres de diagnostic, consultez [Surveiller IoT Hub](monitor-iot-hub.md). Pour un didacticiel, consultez [Configurer et utiliser les métriques et journaux de ressources avec un hub IoT](tutorial-use-metrics-and-diags.md).

Nous vous recommandons également d’activer [l’itinéraire de secours](iot-hub-devguide-messages-d2c.md#fallback-route) si vous souhaitez conserver les messages qui ne correspondent pas à la requête sur l’une des routes. Celles-ci peuvent être conservées dans le [point de terminaison intégré](iot-hub-devguide-messages-read-builtin.md) pour le nombre de jours de conservation des données configuré.

## <a name="top-issues"></a>Problèmes principaux

Voici les problèmes les plus courants rencontrés avec le routage des messages. Pour commencer la résolution des problèmes, cliquez sur le problème afin d’obtenir des instructions détaillées.

* [Les messages de mes appareils ne sont pas routés comme prévu](#messages-from-my-devices-are-not-being-routed-as-expected)
* [J’ai soudainement cessé de recevoir des messages au point de terminaison Event Hubs intégré](#i-suddenly-stopped-getting-messages-at-the-built-in-endpoint)

### <a name="messages-from-my-devices-are-not-being-routed-as-expected"></a>Les messages de mes appareils ne sont pas routés comme prévu

Pour résoudre ce problème, analysez les éléments suivants.

#### <a name="the-routing-metrics-for-this-endpoint"></a>Les métriques de routage pour ce point de terminaison

Toutes les [métriques IoT Hub](monitor-iot-hub-reference.md#routing-metrics) liées au routage sont précédées de *Routage* . Vous pouvez combiner les informations de plusieurs métriques pour identifier la cause racine des problèmes. Par exemple, utilisez la métrique **Routage : tentatives de livraison** pour identifier le nombre de messages qui ont été livrés à un point de terminaison ou supprimés quand ils ne correspondaient pas aux requêtes sur aucun des itinéraires et que l’itinéraire de secours était désactivé. Vérifiez la métrique de **latence du routage** pour déterminer si la latence de la livraison des messages est stable ou en constante augmentation. Une latence croissante peut indiquer un problème avec un point de terminaison spécifique et nous vous recommandons de vérifier [l’intégrité du point de terminaison](#the-health-of-the-endpoint). Ces métriques de routage ont également des [dimensions](monitor-iot-hub-reference.md#metric-dimensions) qui fournissent des détails sur la métrique comme le type de point de terminaison, le nom du point de terminaison spécifique et la raison pour laquelle le message n’a pas été livré.

#### <a name="the-resource-logs-for-any-operational-issues"></a>Les journaux de ressource pour les problèmes opérationnels

Observez les [**Journaux de ressources des** routes](monitor-iot-hub-reference.md#routes) pour obtenir plus d’informations sur les [opérations](#operation-names) de routage et de point de terminaison ou identifier les erreurs et le [code d’erreur](#common-error-codes) correspondant pour mieux comprendre le problème. Par exemple, le nom de l’opération **RouteEvaluationError** dans le journal indique que l’itinéraire n’a pas pu être évalué en raison d’un problème avec le format du message. Utilisez les conseils fournis pour les [noms d’opération](#operation-names) spécifiques pour résoudre le problème. Lorsqu’un événement est consigné comme une erreur, le journal fournit également plus d’informations sur la raison de l’échec de l’évaluation. Par exemple, si le nom de l’opération est **EndpointUnhealthy** , un [code d’erreur](#common-error-codes) de 403004 indique que l’espace du point de terminaison est insuffisant.

#### <a name="the-health-of-the-endpoint"></a>L’intégrité du point de terminaison

Vous pouvez utiliser l’API REST [Obtenir l’intégrité du point de terminaison](/rest/api/iothub/iothubresource/getendpointhealth#iothubresource_getendpointhealth) pour récupérer l’[état d’intégrité](iot-hub-devguide-endpoints.md#custom-endpoints) des points de terminaison. L’API *Obtenir l’intégrité du point de terminaison* fournit également des informations sur la dernière fois qu’un message a bien été envoyé au point de terminaison, la [dernière erreur connue](#last-known-errors-for-iot-hub-routing-endpoints), l’heure de la dernière erreur connue et la dernière fois qu’une tentative d’envoi a été effectuée pour ce point de terminaison. Utilisez la résolution possible proposée pour la [dernière erreur connue](#last-known-errors-for-iot-hub-routing-endpoints) spécifique.

### <a name="i-suddenly-stopped-getting-messages-at-the-built-in-endpoint"></a>J’ai soudainement cessé de recevoir des messages au point de terminaison intégré

Pour résoudre ce problème, analysez les éléments suivants.

#### <a name="was-a-new-route-created"></a>Une nouvelle route a-t-elle été créée ?

Une fois qu’une route est créée, les données cessent de circuler vers le point de terminaison intégré, sauf si une route est créée vers ce point de terminaison. Pour garantir que la circulation des messages vers le point de terminaison intégré malgré l’ajout d’une nouvelle route, configurez une route vers le point de terminaison *d’événements* . 

#### <a name="was-the-fallback-route-disabled"></a>L’itinéraire de secours a-t-elle été désactivée ?

L’itinéraire de secours envoie tous les messages qui ne satisfont pas aux conditions de la requête sur une des routes existantes aux [hubs d’événements existants](iot-hub-devguide-messages-read-builtin.md) (messages/événements), compatible avec [Event Hubs](../event-hubs/index.yml). Si le routage des messages est activé, vous pouvez activer la fonctionnalité de route de secours. S’il n’existe pas de route vers le point de terminaison intégré et qu’une route de secours est activée, seuls les messages qui ne correspondent pas aux conditions de la requête sur les routes sont envoyées au point de terminaison intégré. En outre, si toutes les routes existantes sont supprimées, la route de secours doit être activée pour recevoir toutes les données sur le point de terminaison intégré.

Vous pouvez activer/désactiver la route de secours dans le portail Azure -> Panneau Routage des messages. Vous pouvez également utiliser Azure Resource Manager pour que [FallbackRouteProperties](/rest/api/iothub/iothubresource/createorupdate#fallbackrouteproperties) utilise un point de terminaison personnalisé pour la route de secours.

## <a name="last-known-errors-for-iot-hub-routing-endpoints"></a>Les dernières erreurs connues pour les points de terminaison de routage IoT Hub

<a id="last-known-errors"></a>
[!INCLUDE [iot-hub-include-last-known-errors](../../includes/iot-hub-include-last-known-errors.md)]

## <a name="routes-resource-logs"></a>Journaux de ressources des routes

Voici les noms des opérations et les codes d’erreur consignés dans les [journaux des ressources de ressources](monitor-iot-hub-reference.md#routes).

<a id="diagnostics-operation-names"></a>
### <a name="operation-names"></a>Noms des opérations

[!INCLUDE [iot-hub-diagnostics-operation-names](../../includes/iot-hub-diagnostics-operation-names.md)]

<a id="diagnostics-error-codes"></a>
### <a name="common-error-codes"></a>Common error codes (Codes d’erreur courants)

[!INCLUDE [iot-hub-diagnostics-error-codes](../../includes/iot-hub-diagnostics-error-codes.md)]

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’aide supplémentaire, vous avez la possibilité de contacter les experts Azure sur [les forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support** .