---
title: Métriques dans Azure Monitor - Azure Event Hubs | Microsoft Docs
description: Cet article fournit des informations sur l’utilisation d’Azure Monitor pour une surveillance d’Azure Event Hubs.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 7ad570a41fd9dfff01e3a1da6b2d309a7a8464cc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88931146"
---
# <a name="azure-event-hubs-metrics-in-azure-monitor"></a>Métriques Azure Event Hubs dans Azure Monitor

Les métriques Event Hubs vous donnent l’état de vos ressources Event Hubs dans votre abonnement Azure. Avec un ensemble riche de données de métriques, vous pouvez évaluer l’intégrité globale de vos concentrateurs d’événements au niveau de l’espace de noms, mais également au niveau de l’entité. Ces statistiques peuvent être importantes car elles vous aident à surveiller l’état de vos concentrateurs d’événements. Les métriques peuvent également vous aider à résoudre les problèmes constituant des causes premières sans avoir à contacter le support technique Azure.

Azure Monitor fournit des interfaces utilisateur unifiées pour la surveillance entre divers services Azure. Pour plus d’informations, consultez [Analyse dans Microsoft Azure](../azure-monitor/overview.md) et [Récupérer des métriques Azure Monitor avec .NET](https://github.com/Azure-Samples/monitor-dotnet-metrics-api) sur GitHub.

## <a name="access-metrics"></a>Accéder aux mesures

Azure Monitor propose plusieurs méthodes d’accès aux mesures. Vous pouvez accéder aux métriques via le [portail Azure](https://portal.azure.com), ou bien utiliser les API Azure Monitor (REST et .NET) et des solutions d’analyse comme Log Analytics et Event Hubs. Pour plus d’informations, consultez [Supervision des données collectées par Azure Monitor](../azure-monitor/platform/data-platform.md).

Les métriques sont activées par défaut, et vous pouvez accéder aux 30 derniers jours de données. Si vous souhaitez conserver des données sur une période plus longue, vous pouvez archiver les données de mesures dans un compte de stockage Azure. Celui-ci est configuré dans les [paramètres de diagnostic](../azure-monitor/platform/diagnostic-settings.md) dans Azure Monitor.


## <a name="access-metrics-in-the-portal"></a>Accéder aux métriques dans le portail

Vous pouvez surveiller les mesures au fil du temps dans le [portail Azure](https://portal.azure.com). L’exemple suivant montre comment afficher les demandes réussies et les demandes entrantes au niveau du compte :

![Affichage des métriques de réussite][1]

Vous pouvez également accéder aux métriques directement via l’espace de noms. Pour ce faire, sélectionnez votre espace de noms, puis cliquez sur **Métriques**. Pour afficher les métriques filtrées à l’étendue du concentrateur d’événements, sélectionnez le concentrateur d’événements, puis cliquez sur **Métriques**.

Pour les métriques prenant en charge des dimensions, vous devez filtrer avec la valeur de dimension souhaitée, comme dans l’exemple suivant :

![Filtrage avec une valeur de dimension][2]

## <a name="billing"></a>Facturation

L’utilisation de mesures dans Azure Monitor est actuellement gratuite. Toutefois, si vous utilisez des solutions supplémentaires absorbant des données de métriques, vous pouvez être facturé par ces solutions. Par exemple, vous êtes facturé par Stockage Azure si vous archivez des données de mesures pour un compte de stockage Azure. Vous êtes également facturé par Azure si vous diffusez des données de métriques vers des journaux Azure Monitor pour une analyse avancée.

Les métriques suivantes vous donnent une vue d’ensemble de l’intégrité de votre service. 

> [!NOTE]
> Nous déprécions plusieurs métriques lors de leur déplacement sous un autre nom. Vous devrez peut-être mettre à jour vos références. Les métriques marquées avec le mot-clé « dépréciée » ne seront pas gérées à l’avenir.

Toutes les valeurs de métriques sont envoyées à Azure Monitor toutes les minutes. Le fragment de temps définit l’intervalle de temps pour lequel des valeurs de métriques sont présentées. L’intervalle de temps pris en charge pour toutes les métriques de Event Hubs est de 1 minute.

## <a name="request-metrics"></a>Métriques de demande

Compte le nombre de requêtes d’opérations de données et de gestion.

| Nom de métrique | Description |
| ------------------- | ----------------- |
| Demandes entrantes  | Le nombre de requêtes effectuées auprès de Azure Event Hubs sur une période spécifiée. <br/><br/> Unité : nombre <br/> Type d’agrégation : total <br/> Dimension : EntityName |
| Requêtes ayant réussi    | Le nombre de requêtes réussies auprès de Azure Event Hubs sur une période spécifiée. <br/><br/> Unité : nombre <br/> Type d’agrégation : total <br/> Dimension : EntityName |
| Erreurs de serveur  | Le nombre de requêtes non traitées en raison d’une erreur sur Azure Event Hubs sur une période spécifiée. <br/><br/>Unité : nombre <br/> Type d’agrégation : total <br/> Dimension : EntityName |
|Erreurs d’utilisateur |Le nombre de demandes non traitées en raison d’erreurs utilisateur sur une période spécifiée.<br/><br/> Unité : nombre <br/> Type d’agrégation : total <br/> Dimension : EntityName|
|Erreurs de dépassement de quota |Le nombre de requêtes a dépassé le quota disponible. Consultez [cet article](event-hubs-quotas.md) pour plus d’informations sur les quotas Event Hubs.<br/><br/> Unité : nombre <br/> Type d’agrégation : total <br/> Dimension : EntityName|

## <a name="throughput-metrics"></a>Métriques de débit

| Nom de métrique | Description |
| ------------------- | ----------------- |
|Requêtes limitées |Le nombre de demandes qui ont été limitées car l’utilisation d’une unité de débit a été dépassée.<br/><br/> Unité : nombre <br/> Type d’agrégation : total <br/> Dimension : EntityName|

## <a name="message-metrics"></a>Métriques de message

| Nom de métrique | Description |
| ------------------- | ----------------- |
|Messages entrants |Le nombre d’événements ou de messages envoyés à Event Hubs sur une période spécifiée.<br/><br/> Unité : nombre <br/> Type d’agrégation : total <br/> Dimension : EntityName|
|Messages sortants |Le nombre d’événements ou de messages récupérés depuis Event Hubs sur une période spécifiée.<br/><br/> Unité : nombre <br/> Type d’agrégation : total <br/> Dimension : EntityName|
|Octets entrants |Le nombre d’octets envoyés au service Azure Event Hubs sur une période spécifiée.<br/><br/> Unité : octets <br/> Type d’agrégation : total <br/> Dimension : EntityName|
|Octets sortants |Le nombre d’octets récupérés depuis le service Azure Event Hubs sur une période spécifiée.<br/><br/> Unité : octets <br/> Type d’agrégation : total <br/> Dimension : EntityName|

## <a name="connection-metrics"></a>Métriques de connexion

| Nom de métrique | Description |
| ------------------- | ----------------- |
|ActiveConnections |Le nombre de connexions actives sur un espace de noms ainsi que sur une entité.<br/><br/> Unité : nombre <br/> Type d’agrégation : total <br/> Dimension : EntityName|
|Connexions ouvertes |Le nombre de connexions ouvertes.<br/><br/> Unité : nombre <br/> Type d’agrégation : total <br/> Dimension : EntityName|
|Connexions fermées |Le nombre de connexions fermées.<br/><br/> Unité : nombre <br/> Type d’agrégation : total <br/> Dimension : EntityName|

## <a name="event-hubs-capture-metrics"></a>Métriques de capture Event Hubs

Vous pouvez surveiller les métriques de capture Event Hubs lorsque vous activez la fonctionnalité de capture pour vos concentrateurs d’événements. Les métriques suivantes décrivent ce que vous pouvez analyser avec la capture activée.

| Nom de métrique | Description |
| ------------------- | ----------------- |
|Backlog des captures |Le nombre d’octets qui ne sont pas encore être capturés dans la destination choisie.<br/><br/> Unité : octets <br/> Type d’agrégation : total <br/> Dimension : EntityName|
|Messages capturés |Le nombre de messages ou d’événements capturés dans la destination choisie sur une période spécifiée.<br/><br/> Unité : nombre <br/> Type d’agrégation : total <br/> Dimension : EntityName|
|Octets capturés |Le nombre d’octets capturés dans la destination choisie sur une période spécifiée.<br/><br/> Unité : octets <br/> Type d’agrégation : total <br/> Dimension : EntityName|

## <a name="metrics-dimensions"></a>Dimensions de mesures

Azure Event Hubs prend en charge les dimensions suivantes pour les mesures dans Azure Monitor. L’ajout de dimensions à vos métriques est facultatif. Si vous n’ajoutez pas de dimensions, les métriques sont spécifiées au niveau de l’espace de noms. 

| Nom de métrique | Description |
| ------------------- | ----------------- |
|EntityName| Event Hubs prend en charge les entités de concentrateurs d’événements sous l’espace de noms.|

## <a name="azure-monitor-integration-with-siem-tools"></a>Intégration d'Azure Monitor aux outils SIEM
Le routage de vos données de surveillance (journaux d’activité, journaux de diagnostic, etc.) vers un Event Hub avec Azure Monitor vous permet d’intégrer facilement les outils Security Information and Event Management (SIEM). Pour plus d’informations, consultez les articles/billets de blog suivants :

- [Diffuser des données de surveillance Azure vers un hub d’événements pour les utiliser dans un outil externe](../azure-monitor/platform/stream-monitoring-data-event-hubs.md)
- [Présentation d'Azure Log Integration](/previous-versions/azure/security/fundamentals/azure-log-integration-overview)
- [Utiliser Azure Monitor pour opérer une intégration avec des outils de SIEM](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

Si un outil SIEM consomme des données de journal à partir d’un Event Hub et que vous ne voyez aucun message entrant ou voyez des messages entrants mais pas de messages sortants dans le graphe des métriques, procédez comme suit :

- Une **absence de messages entrants** indique que le service Azure Monitor ne déplace pas les journaux d’audit/de diagnostic dans le Event Hub. Si tel est le cas, ouvrez un ticket de support auprès de l’équipe Azure Monitor. 
- La présence de messages entrants mais **pas de messages sortants** indique que l’application SIEM ne lit pas les messages. Contactez le fournisseur SIEM afin de déterminer si la configuration du Event Hub de ces applications est correcte.


## <a name="next-steps"></a>Étapes suivantes

* Consultez la [Vue d’ensemble de surveillance Azure](../azure-monitor/overview.md).
* Exemple [Récupérer les métriques Azure Monitor avec .NET](https://github.com/Azure-Samples/monitor-dotnet-metrics-api) sur GitHub. 

Pour plus d’informations sur les concentrateurs d’événements, accédez aux liens suivants :

- Tutoriel Prise en main d'Event Hubs
    - [.NET Core](event-hubs-dotnet-standard-getstarted-send.md)
    - [Java](event-hubs-java-get-started-send.md)
    - [Python](event-hubs-python-get-started-send.md)
    - [JavaScript](event-hubs-java-get-started-send.md)
* [FAQ sur les hubs d’événements](event-hubs-faq.md)
* [Exemples d’application complets qui utilisent des concentrateurs d’événements](https://github.com/Azure/azure-event-hubs/tree/master/samples)

[1]: ./media/event-hubs-metrics-azure-monitor/event-hubs-monitor1.png
[2]: ./media/event-hubs-metrics-azure-monitor/event-hubs-monitor2.png
