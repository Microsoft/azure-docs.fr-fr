---
title: Métriques Azure Relay dans Azure Monitor | Microsoft Docs
description: Cet article fournit des informations sur l’utilisation d’Azure Monitor pour superviser l’état d’Azure Relay.
services: service-bus-relay
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 5fcc0f820983da388d808cadf8ce64a555e8ef0a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000179"
---
# <a name="azure-relay-metrics-in-azure-monitor"></a>Métriques Azure Relay dans Azure Monitor 
Les métriques Azure Relay indiquent l’état des ressources d’un abonnement Azure. Avec un ensemble complet de données de métriques, vous pouvez évaluer l’intégrité globale des ressources Relay, non seulement au niveau de l’espace de noms, mais également au niveau de l’entité. Ces statistiques peuvent être importantes, car elles vous aident à surveiller l’état d’Azure Relay. Les métriques peuvent également vous aider à résoudre les problèmes constituant des causes premières sans avoir à contacter le support technique Azure.

Azure Monitor fournit des interfaces utilisateur unifiées pour la surveillance entre divers services Azure. Pour plus d’informations, consultez [Analyse dans Microsoft Azure](../azure-monitor/overview.md) et [Récupérer des métriques Azure Monitor avec .NET](https://github.com/Azure-Samples/monitor-dotnet-metrics-api) sur GitHub.

> [!IMPORTANT]
> Cet article s’applique uniquement à la fonctionnalité Connexions hybrides d’Azure Relay, et non à WCF Relay. 

## <a name="access-metrics"></a>Accéder aux mesures

Azure Monitor propose plusieurs méthodes d’accès aux mesures. Vous pouvez accéder à ces métriques depuis le [portail Azure](https://portal.azure.com), ou utiliser les API d’Azure Monitor (REST et .Net) et les solutions d’analyse comme Operation Management Suite et Event Hub. Pour plus d’informations, consultez [Supervision des données collectées par Azure Monitor](../azure-monitor/platform/data-platform.md).

Les métriques sont activées par défaut, et vous pouvez accéder aux 30 derniers jours de données. Si vous souhaitez conserver des données sur une période plus longue, vous pouvez archiver les données de mesures dans un compte de stockage Azure. Celui-ci est configuré dans les [paramètres de diagnostic](../azure-monitor/platform/diagnostic-settings.md) dans Azure Monitor.

## <a name="access-metrics-in-the-portal"></a>Accéder aux métriques dans le portail

Vous pouvez surveiller les mesures au fil du temps dans le [portail Azure](https://portal.azure.com). L’exemple suivant montre comment afficher les demandes réussies et les demandes entrantes au niveau du compte :

![Une page intitulée « Surveiller – Métriques (préversion) » affiche un graphique linéaire représentant l’utilisation de la mémoire au cours des 30 derniers jours.][1]

Vous pouvez également accéder aux métriques directement via l’espace de noms. Pour ce faire, sélectionnez votre espace de noms, puis cliquez sur **Métriques **. 

Pour les mesures prenant en charge des dimensions, vous devez filtrer avec la valeur de dimension souhaitée.

## <a name="billing"></a>Facturation

L’utilisation de mesures dans Azure Monitor est actuellement gratuite pendant la préversion. Toutefois, si vous utilisez des solutions supplémentaires absorbant des données de métriques, vous pouvez être facturé par ces solutions. Par exemple, vous êtes facturé par Stockage Azure si vous archivez des données de mesures pour un compte de stockage Azure. Vous êtes également facturé par la solution Journaux Azure Monitor si vous envoyez en streaming des données de métriques vers Journaux Azure Monitor pour une analyse avancée.

Les métriques suivantes vous donnent une vue d’ensemble de l’intégrité de votre service. 

> [!NOTE]
> Nous déprécions plusieurs métriques lors de leur déplacement sous un autre nom. Vous devrez peut-être mettre à jour vos références. Les métriques marquées avec le mot-clé « dépréciée » ne seront pas gérées à l’avenir.

Toutes les valeurs de métriques sont envoyées à Azure Monitor toutes les minutes. Le fragment de temps définit l’intervalle de temps pour lequel des valeurs de métriques sont présentées. L’intervalle de temps pris en charge pour toutes les métriques d’Azure Relay est de 1 minute.

## <a name="connection-metrics"></a>Métriques de connexion

| Nom de métrique | Description |
| ------------------- | ----------------- |
| ListenerConnections-Success  | Nombre de connexions réussies de l’écouteur vers Azure Relay au cours d’une période spécifiée. <br/><br/> Unité : Count <br/> Type d’agrégation : Total <br/> Dimension : EntityName|
|ListenerConnections-ClientError |Nombre d’erreurs du client sur les connexions de l’écouteur au cours d’une période spécifiée.<br/><br/> Unité : Count <br/> Type d’agrégation : Total <br/> Dimension : EntityName|
|ListenerConnections-ServerError |Nombre d’erreurs du serveur sur les connexions de l’écouteur au cours d’une période spécifiée.<br/><br/> Unité : Count <br/> Type d’agrégation : Total <br/> Dimension : EntityName|
|SenderConnections-Success |Nombre de connexions réussies de l’expéditeur au cours d’une période spécifiée.<br/><br/> Unité : Count <br/> Type d’agrégation : Total <br/> Dimension : EntityName|
|SenderConnections-ClientError |Nombre d’erreurs du client sur les connexions de l’expéditeur au cours d’une période spécifiée.<br/><br/> Unité : Count <br/> Type d’agrégation : Total <br/> Dimension : EntityName|
|SenderConnections-ServerError |Nombre d’erreurs du serveur sur les connexions de l’expéditeur au cours d’une période spécifiée.<br/><br/> Unité : Count <br/> Type d’agrégation : Total <br/> Dimension : EntityName|
|ListenerConnections-TotalRequests |Nombre total de connexions de l’écouteur au cours d’une période spécifiée.<br/><br/> Unité : Count <br/> Type d’agrégation : Total <br/> Dimension : EntityName|
|SenderConnections-TotalRequests |Demandes de connexion effectuées par les expéditeurs sur une période spécifiée.<br/><br/> Unité : Count <br/> Type d’agrégation : Total <br/> Dimension : EntityName|
|ActiveConnections |Nombre de connexions actives. Cette valeur est une valeur à un point dans le temps.<br/><br/> Unité : Count <br/> Type d’agrégation : Total <br/> Dimension : EntityName|
|ActiveListeners |Nombre d’écouteurs actifs. Cette valeur est une valeur à un point dans le temps.<br/><br/> Unité : Count <br/> Type d’agrégation : Total <br/> Dimension : EntityName|
|ListenerDisconnects |Nombre d’écouteurs déconnectés au cours d’une période spécifiée.<br/><br/> Unité : Octets <br/> Type d’agrégation : Total <br/> Dimension : EntityName|
|SenderDisconnects |Nombre d’expéditeurs déconnectés au cours d’une période spécifiée.<br/><br/> Unité : Octets <br/> Type d’agrégation : Total <br/> Dimension : EntityName|

## <a name="memory-usage-metrics"></a>Mesures d’utilisation de la mémoire

| Nom de métrique | Description |
| ------------------- | ----------------- |
|BytesTransferred |Nombre d’octets transférés au cours d’une période spécifiée.<br/><br/> Unité : Octets <br/> Type d’agrégation : Total <br/> Dimension : EntityName|

## <a name="metrics-dimensions"></a>Dimensions de mesures

Azure Relay prend en charge les dimensions suivantes pour les mesures dans Azure Monitor. L’ajout de dimensions à vos métriques est facultatif. Si vous n’ajoutez pas de dimensions, les métriques sont spécifiées au niveau de l’espace de noms. 

|Nom de la dimension|Description|
| ------------------- | ----------------- |
|EntityName| Azure Relay prend en charge les entités de messagerie sous l’espace de noms.|

## <a name="next-steps"></a>Étapes suivantes

Consultez la [Vue d’ensemble de surveillance Azure](../azure-monitor/overview.md).

[1]: ./media/relay-metrics-azure-monitor/relay-monitor1.png
