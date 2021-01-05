---
title: Métriques et journaux de diagnostic Media Services avec Azure Monitor
titleSuffix: Azure Media Services
description: Découvrez comment surveiller les métriques et journaux de diagnostic Azure Media Services via Azure Monitor.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 11/02/2020
ms.author: inhenkel
ms.openlocfilehash: ed436336b9b1f9f5815938d13315e821694e5f1c
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94685169"
---
# <a name="monitor-media-services-metrics-and-diagnostic-logs-with-azure-monitor"></a>Surveiller les métriques et journaux de diagnostic Media Services via Azure Monitor

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

[Azure Monitor](../../azure-monitor/overview.md) vous permet de surveiller les métriques et les journaux de diagnostic qui vous aident à comprendre le fonctionnement de vos applications. Toutes les données collectées par Azure Monitor font partie d’un des deux types fondamentaux : métriques et journaux. Vous pouvez surveiller les journaux de diagnostic Media Services et créer des alertes et notifications pour les métriques et les journaux collectés. Vous pouvez visualiser et analyser les métriques à l’aide [Metrics Explorer](../../azure-monitor/platform/metrics-getting-started.md). Vous pouvez envoyer les journaux d’activité au [Stockage Azure](https://azure.microsoft.com/services/storage/), les transmettre à [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/), les exporter vers [Log Analytics](https://azure.microsoft.com/services/log-analytics/) ou utiliser des services tiers.

Pour plus un aperçu détaillé, voir [Métriques Azure Monitor](../../azure-monitor/platform/data-platform.md) et [Journaux de diagnostic Azure Monitor](../../azure-monitor/platform/platform-logs-overview.md).

Cette rubrique traite des [métriques Media Services](#media-services-metrics) et des [Journaux de diagnostic Media Services](#media-services-diagnostic-logs) pris en charge.

## <a name="media-services-metrics"></a>Métriques Media Services

Les métriques sont collectées à intervalles réguliers, que la valeur change ou non. Elles sont utiles pour créer des alertes, dans la mesure où elles peuvent être échantillonnées fréquemment, et où une alerte peut être déclenchée rapidement avec un circuit logique assez simple. Pour plus d’informations sur la création d’alertes de métrique, voir [Créer, afficher et gérer des alertes de métrique à l’aide d’Azure Monitor](../../azure-monitor/platform/alerts-metric.md).

Media Services prend en charge les métriques de surveillance pour les ressources suivantes :

* Compte
* Point de terminaison de streaming

### <a name="account"></a>Compte

Vous pouvez surveiller les métriques de compte suivantes.

|Nom de métrique|Nom complet|Description|
|---|---|---|
|AssetCount|Nombre de ressources|Ressources dans votre compte.|
|AssetQuota|Quota de ressources|Quota de ressources dans votre compte.|
|AssetQuotaUsedPercentage|Pourcentage du quota de ressources utilisé|Pourcentage du quota de ressources déjà utilisé.|
|ContentKeyPolicyCount|Nombre de stratégies de clé de contenu|Stratégies de clé de contenu dans votre compte.|
|ContentKeyPolicyQuota|Quota de stratégies de clé de contenu|Quota de stratégies de clé de contenu dans votre compte.|
|ContentKeyPolicyQuotaUsedPercentage|Pourcentage du quota de stratégies de clé de contenu utilisé|Pourcentage du quota de stratégies de clé de contenu déjà utilisé.|
|StreamingPolicyCount|Nombre de stratégies de diffusion en continu|Stratégies de diffusion en continu dans votre compte.|
|StreamingPolicyQuota|Quota de stratégies de diffusion en continu|Quota de stratégies de diffusion en continu dans votre compte.|
|StreamingPolicyQuotaUsedPercentage|Pourcentage du quota de stratégies de diffusion en continu utilisé|Pourcentage du quota de stratégies de diffusion en continu déjà utilisé.|

Vous devez également examiner les [quotas et limites de compte](limits-quotas-constraints.md).

### <a name="streaming-endpoint"></a>Point de terminaison de streaming

Les métriques de [points de terminaison de streaming](/rest/api/media/streamingendpoints) Media Services suivantes sont pris en charge :

|Nom de métrique|Nom complet|Description|
|---|---|---|
|Demandes|Demandes|Fournit le nombre total de requêtes HTTP traitées par le point de terminaison de streaming.|
|Sortie|Sortie|Nombre total d’octets de sortie par minute par point de terminaison de streaming.|
|SuccessE2ELatency|Latence de réussite de bout en bout|Durée à partir de laquelle le point de terminaison de streaming a reçu la requête lorsque le dernier octet de la réponse a été envoyé.|
|Utilisation de l’UC| | Utilisation de l’UC pour les points de terminaison de streaming Premium. Ces données ne sont pas disponibles pour les points de terminaison de streaming Standard. |
|Bande passante en sortie | | Sortie de la bande passante en bits par seconde.|

### <a name="metrics-are-useful"></a>Utilité des métriques

Voici quelques exemples de la façon dont la surveillance de métriques Media Services peut vous aider à comprendre le fonctionnement de vos applications. Voici quelques questions qui peuvent être traitées avec les métriques Media Services :

* Comment surveiller mon point de terminaison de streaming standard pour savoir quand j’ai dépassé les limites ?
* Comment savoir si j’ai suffisamment d’unités d’échelle de point de terminaison de streaming Premium ?
* Comment définir une alerte pour savoir quand mettre à l’échelle mes points de terminaison de streaming ?
* Comment définir une alerte pour savoir quand la sortie maximale configurée sur le compte a été atteinte ?
* Comment voir la répartition des demandes qui échouent et ce qui est à l’origine de l’échec ?
* Comment voir combien de demandes TLS ou DASH sont extraites du packager ?
* Comment définir une alerte pour savoir quand la valeur seuil du nombre de requêtes ayant échoué a été atteinte ?

L’accès concurrentiel devient une préoccupation pour le nombre de points de terminaison de streaming utilisés dans un seul compte dans le temps. Vous devez garder à l’esprit la relation entre le nombre de flux simultanés et les paramètres de publication complexes tels que l’empaquetage dynamique sur plusieurs protocoles, plusieurs chiffrements DRM, etc. Chaque stream en direct publié supplémentaire vient s’ajouter à l’UC et à la bande passante en sortie sur le point de terminaison de streaming. En tenant compte de cela, vous devez utiliser Azure Monitor pour observer attentivement l’utilisation du point de terminaison de streaming (capacité d’UC et de sortie) afin de vous assurer que vous le mettez à l’échelle de manière appropriée (ou fractionnez le trafic entre plusieurs points de terminaison de streaming si vous obtenez une concurrence très élevée).

### <a name="example"></a>Exemple

Consultez [Surveiller les métriques Media Services](media-services-metrics-howto.md).

## <a name="media-services-diagnostic-logs"></a>Journaux de diagnostic Media Services

Les journaux de diagnostic fournissent des informations complètes et fréquentes sur le fonctionnement d’une ressource Azure. Pour plus d’informations, consultez [Comment collecter et utiliser des données de journal à partir de vos ressources Azure](../../azure-monitor/platform/platform-logs-overview.md).

Media Services prend en charge les journaux de diagnostic suivants :

* Remise de clés

### <a name="key-delivery"></a>Remise de clés

|Nom|Description|
|---|---|
|Demande de service de remise de clé|Journaux contenant les informations de demande de service de remise de clé. Pour plus d’informations, consultez la section relative aux [schémas](media-services-diagnostic-logs-schema.md).|

### <a name="why-would-i-want-to-use-diagnostics-logs"></a>Pourquoi utiliser les journaux de diagnostic ?

Voici quelques éléments que vous pouvez examiner avec les journaux de diagnostic de remise de clé :

* Voir le nombre de licences fournies par type DRM.
* Voir le nombre de licences fournies par stratégie.
* Voir les erreurs par type de DRM ou de stratégie.
* Voir le nombre de requêtes de licence non autorisées émises par les clients.

### <a name="example"></a>Exemple

Consultez [Comment surveiller les journaux de diagnostic Media Services](media-services-diagnostic-logs-howto.md).

## <a name="next-steps"></a>Étapes suivantes

* [Comment collecter et consommer les données des journaux de vos ressources Azure](../../azure-monitor/platform/platform-logs-overview.md)
* [Créer, afficher et gérer des alertes de métrique à l'aide d'Azure Monitor](../../azure-monitor/platform/alerts-metric.md)
* [Surveiller les métriques Media Services](media-services-metrics-howto.md)
* [Surveillez les journaux de diagnostic de Media Services](media-services-diagnostic-logs-howto.md)
