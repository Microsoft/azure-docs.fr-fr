---
title: Mise à jour de la gestion des alertes et de la supervision classiques dans Azure Monitor
description: Description de la mise hors service des services et des fonctionnalités de supervision classique, présentée précédemment dans le Portail Azure sous Alertes (classique).
author: yanivlavi
services: azure-monitor
ms.topic: conceptual
ms.date: 2/7/2019
ms.author: yalavi
ms.subservice: alerts
ms.openlocfilehash: 720a0e25f3486f32ffed897e54033fd4b68dace4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87848123"
---
# <a name="unified-alerting--monitoring-in-azure-monitor-replaces-classic-alerting--monitoring"></a>La gestion des alertes et la supervision unifiées dans Azure Monitor remplace la gestion des alertes et la supervision classiques

Azure Monitor est désormais devenu un service unifié de supervision de la pile complète qui prend maintenant en charge les fonctionnalités « Métriques unifiées » et « Alertes unifiées » pour l’ensemble des ressources. Pour plus d’informations, consultez notre [billet de blog sur la nouvelle instance Azure Monitor](https://azure.microsoft.com/blog/new-full-stack-monitoring-capabilities-in-azure-monitor/). La nouvelle plateforme de supervision et de gestion des alertes Azure a été conçue pour être plus rapide, plus intelligente et extensible, permettant ainsi de suivre le rythme de l’expansion croissante du cloud computing et conformément à la philosophie du cloud intelligent Microsoft.

Avec la nouvelle plateforme de surveillance et d’alerte Azure en place, les alertes classiques dans Azure Monitor sont mises hors service, bien qu’elles soient toujours utilisées pour les ressources qui ne prennent pas encore en charge les nouvelles alertes. La date de mise hors service de ces alertes a été repoussée. Une nouvelle date sera annoncée bientôt pour la migration des alertes restantes ([cloud Azure Government](../../azure-government/documentation-government-welcome.md) et [Azure China 21Vianet](https://docs.azure.cn/)).

 ![Alerte classique dans le portail Azure](media/monitoring-classic-retirement/monitor-alert-screen2.png) 

Nous vous encourageons à l’utiliser et à recréer vos alertes sur la nouvelle plateforme.

> [!IMPORTANT]
> Les règles d'alerte classiques de journal d'activité ne seront ni déconseillées ni migrées. Toutes les règles d’alerte classiques créées sur le journal d’activité sont accessibles et utilisées en l'état dans le nouveau Azure Monitor - Alertes. Pour plus d'informations, consultez [Créer, afficher et gérer des alertes de journal d'activité avec Azure Monitor](./alerts-activity-log.md). De même, les alertes Service Health sont accessibles et peuvent être utilisées en l'état dans la nouvelle section Service Health. Pour plus d’informations, consultez [Alertes sur les notifications Service Health](../../service-health/alerts-activity-log-service-notifications-portal.md).

## <a name="unified-metrics-and-alerts-in-application-insights"></a>Métriques et alertes unifiées dans Application Insights

La nouvelle plateforme de métrique d’Azure Monitor va maintenant dynamiser la supervision provenant d’Application Insights. Ce déplacement signifie qu’Application Insights se raccrochera aux groupes d’actions, ce qui offrira bien plus d’options que les simples appels d’e-mail et webhook précédents. Les alertes peuvent maintenant déclencher des appels vocaux, Azure Functions, Logic Apps, des SMS et des outils ITSM comme ServiceNow et les runbooks Automation. Avec la supervision et la gestion des alertes en quasi temps réel, la nouvelle plateforme permet aux utilisateurs Application Insights de tirer parti de la même technologie en exploitant la supervision d’autres ressources Azure et en soutenant la supervision de produits Microsoft.

Les nouvelles supervision et gestion des alertes unifiées pour Application Insights engloberont les éléments suivants :

- **Métriques de plateforme Application Insights** : fournit des métriques prédéfinies populaires à partir du produit Application Insights. Pour plus d’informations, consultez cet article relatif à l’utilisation de [métriques de plateforme pour Application Insights sur la nouvelle instance Azure Monitor](../app/pre-aggregated-metrics-log-metrics.md#pre-aggregated-metrics).
- **Test web et de disponibilité d’Application Insights** : vous offre la possibilité d’évaluer la réactivité et la disponibilité de votre serveur ou application web. Pour plus d’informations, consultez cet article relatif à l’utilisation de [tests de disponibilité et d’alertes pour Application Insights sur la nouvelle instance Azure Monitor](../app/monitor-web-app-availability.md).
- **Métriques personnalisées Application Insights** : vous permet de définir et d’émettre vos propres métriques pour la supervision et les alertes. Pour plus d’informations, consultez cet article relatif à l’utilisation d’une [métrique personnalisée pour Application Insights sur la nouvelle instance Azure Monitor](../app/pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation).
- **Anomalies des échecs Application Insights (partie de la Détection intelligente)**  : vous avertit automatiquement en quasi temps réel si une augmentation anormale du taux des requêtes HTTP en échec ou des appels de dépendance est détectée dans votre application web. Pour plus d’informations, consultez cet article sur l’utilisation de la [Détection intelligente des anomalies de type échec](../app/proactive-failure-diagnostics.md).

## <a name="unified-metrics-and-alerts-for-other-azure-resources"></a>Métriques et alertes unifiées pour d'autres ressources Azure

La nouvelle génération de gestion des alertes et la supervision multidimensionnelle pour les ressources Azure sont disponibles depuis mars 2018. La nouvelle plateforme de métrique et la nouvelle gestion des alertes sont maintenant plus rapides avec des fonctionnalités en temps quasi réel. Plus important encore, les alertes de la nouvelle plateforme de métrique fournissent plus de granularité, car cette dernière inclut l’option des dimensions, qui vous permettent de segmenter et de filtrer une combinaison de valeurs, une condition ou une opération spécifique. Comme toutes les alertes dans la nouvelle instance Azure Monitor, les alertes de métrique plus récentes sont plus extensibles avec l’utilisation d’ActionGroups, ce qui permet d’étendre les notifications au-delà des e-mails ou des webhooks en utilisant des SMS, des appels vocaux, Azure Functions, des runbooks Automation, et bien plus encore. Pour plus d'informations, consultez [Créer, afficher et gérer des alertes de métrique à l'aide d'Azure Monitor](./alerts-metric.md).
Les métriques plus récentes pour les ressources Azure sont disponibles sous les formes suivantes :

- **Métriques standard de la plateforme Azure Monitor**, ce qui fournit des métriques préremplies populaires à partir de divers produits et services Azure. Pour plus d’informations, consultez cet article relatif aux [Métriques prises en charge sur Azure Monitor](./alerts-metric-near-real-time.md#metrics-and-dimensions-supported) et aux [Alertes pour les métriques prises en charge sur Azure Monitor](./alerts-metric-overview.md#supported-resource-types-for-metric-alerts).
- **Métriques personnalisées Azure Monitor**, ce qui fournit des métriques provenant de sources contrôlées par l’utilisateur, notamment l’agent de diagnostics Azure. Pour plus d’informations, consultez cet article relatif aux [métriques personnalisées dans Azure Monitor](./metrics-custom-overview.md). En utilisant des métriques personnalisées, vous pouvez également publier des métriques collectées par [l’agent Diagnostics Azure pour Windows](./collect-custom-metrics-guestos-resource-manager-vm.md) et [l’agent InfluxData Telegraf](./collect-custom-metrics-linux-telegraf.md).

## <a name="retirement-of-classic-monitoring-and-alerting-platform"></a>Mise hors service de la plateforme de supervision et de gestion des alertes classiques

Comme indiqué précédemment, l’ancienne plateforme de supervision et de gestion des alertes classiques est mise hors service (dont la fermeture des API, de l’interface du portail Azure et des services qui y sont liés), bien qu’elle soient toujours utilisée pour les ressources qui ne prennent pas encore en charge les nouvelles alertes. Plus précisément, les fonctionnalités suivantes seront dépréciées :

- Les anciennes métriques et alertes (classiques) pour les ressources Azure telles que disponibles par le biais de la [section Alertes (classiques)](./alerts-classic.overview.md) du portail Azure et accessibles en tant que ressource [microsoft.insights/alertrules](/rest/api/monitor/alertrules)
- Les anciennes métriques personnalisées et de plateforme (classiques) pour Application Insights ainsi que la gestion des alertes définies dessus, telles que disponibles par le biais de la [section Alertes (classiques)](./alerts-classic.overview.md) du portail Azure et accessibles en tant que ressource [microsoft.insights/alertrules](/rest/api/monitor/alertrules)
- L’ancienne alerte Anomalies d’échecs (classique) actuellement disponible sous forme de [détection intelligente au sein d’Application Insights](../app/proactive-diagnostics.md) sur le portail Azure, avec des alertes configurées affichées dans la [section Alertes (classiques)](./alerts-classic.overview.md) du portail Azure

Cela implique :

- Le service de supervision et gestion des alertes classiques sera supprimé et ne sera plus disponible pour la création de nouvelles règles d’alerte.
- Les règles d’alerte qui existeront toujours dans Alertes (classiques) continueront de s’exécuter et de déclencher des notifications.
- Les règles d’alerte restant dans la plateforme de supervision et de gestion des alertes classiques pouvant être migrées seront automatiquement déplacées par Microsoft vers leur équivalent dans la nouvelle plateforme Azure Monitor par phases s’étalant sur quelques semaines. Le processus sera transparent et sans interruption de service, et les clients ne subiront aucune perte de couverture de la supervision.
- Les règles d'alerte migrées vers la nouvelle plateforme d'alertes fourniront la même supervision qu'auparavant, mais déclencheront une notification avec de nouvelles charges utiles. Toute adresse e-mail, tout point de terminaison webhook ou tout lien d’application logique associé à une règle d’alerte classique sera reporté lors de la migration, mais avec un risque de comportement incorrect car la charge utile de l’alerte sera différente sur la nouvelle plateforme.
- Certaines [règles d’alerte classiques ne pouvant pas être migrées automatiquement](alerts-understand-migration.md#manually-migrating-classic-alerts-to-newer-alerts) et nécessitant une action manuelle des utilisateurs continueront de s’exécuter.

> [!IMPORTANT]
> Microsoft Azure Monitor a déployé par phases un [outil de migration volontaire](alerts-using-migration-tool.md) de leurs règles d’alerte classiques vers la nouvelle plateforme. Il sera possible de forcer son exécution pour toutes les règles d’alerte classiques qui subsisteront et qui peuvent être migrées. Les clients devront veiller à ce que la charge utile des règles d'alerte classiques qui ont recours à l'automatisation soit adaptée à la gestion de la nouvelle charge utile à partir de [Métriques et alertes unifiées dans Application Insights](#unified-metrics-and-alerts-in-application-insights) ou [Métriques et alertes unifiées pour d'autres ressources Azure](#unified-metrics-and-alerts-for-other-azure-resources), après la migration des règles d'alerte classiques. Pour plus d’informations, consultez la section de [préparation à la migration de règle d’alerte classique](alerts-prepare-migration.md)

Cet article sera continuellement mis à jour avec des liens et des informations détaillées sur la nouvelle fonctionnalité de supervision et de gestion des alertes Azure, ainsi que sur la disponibilité des outils permettant aux utilisateurs d'adopter la nouvelle plateforme Azure Monitor.

## <a name="pricing-for-migrated-alert-rules"></a>Tarifs des règles d’alerte migrées

Nous déployons un outil de migration pour vous aider à migrer vos [alertes classiques](./alerts-classic.overview.md) Azure Monitor vers la nouvelle expérience alertes. Les règles d’alerte migrées et les groupes d’actions migrés correspondants (e-mail, webhook ou LogicApp) restent gratuits. La fonctionnalité dont vous disposiez avec les alertes classiques, y compris la possibilité de modifier le seuil, le type d’agrégation et la granularité de l’agrégation, restera disponible gratuitement avec votre règle d’alerte migrée. Toutefois, si vous modifiez la règle d’alerte migrée pour utiliser l’une des nouvelles fonctionnalités de la plateforme d’alerte, les notifications ou les types d’action, les frais correspondants s’appliquent. Pour plus d’informations sur la tarification des règles d’alerte et des notifications, consultez la [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

Voici des exemples de cas dans lesquels vous devrez payer votre règle d’alerte :

- Toute règle d’alerte (non migrée) créée au-delà des unités gratuites, sur la nouvelle plateforme Azure Monitor
- Toutes les données reçues et conservées au-delà des unités gratuites incluses par Azure Monitor
- Tous les tests web multiples exécutés par Application Insights
- Toutes les métriques personnalisées stockées au-delà des unités gratuites incluses dans Azure Monitor
- Toutes les règles d’alerte migrées qui sont modifiées pour utiliser de nouvelles fonctionnalités d’alertes de métrique comme la fréquence, les ressources/dimensions multiples, les [Seuils dynamiques](alerts-dynamic-thresholds.md), le changement de ressource/signal, et ainsi de suite.
- Tous les groupes d’actions migrés qui sont modifiés afin d’utiliser des notifications plus récentes ou des types d’actions comme les SMS, les appels vocaux et/ou l’intégration ITSM.

## <a name="next-steps"></a>Étapes suivantes

* Découvrir la [nouvelle instance Azure Monitor](../overview.md).
* Découvrir les nouvelles [Alertes Azure](./alerts-overview.md).

