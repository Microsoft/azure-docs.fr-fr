---
title: Superviser les opérations et l’activité
titleSuffix: Azure Cognitive Search
description: Activer la journalisation, obtenir des métriques d’activité des requêtes, l’utilisation des ressources et d’autres données système à partir d’un service Recherche cognitive Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: d47f6c20246e3210b58dbc9c802a11c866ae305e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88935005"
---
# <a name="monitor-operations-and-activity-of-azure-cognitive-search"></a>Superviser les opérations et l’activité de Recherche cognitive Azure

Cet article consiste en une vue d’ensemble des concepts et des outils de monitoring pour la Recherche cognitive Azure. Pour une surveillance complète, vous pouvez utiliser conjointement des fonctionnalités intégrées et des services complémentaires comme Azure Monitor.

Cela vous permettra d’effectuer le suivi des éléments suivants :

* Service : intégrité/disponibilité et modifications apportées à la configuration du service.
* Stockage : à la fois utilisé et disponible, avec le volume de chaque type de contenu par rapport au quota autorisé pour le niveau de service.
* Activité de requête : volume, latence et requêtes limitées ou annulées. [Azure Monitor](#add-azure-monitor) est nécessaire pour les demandes de requêtes journalisées.
* Activité d’indexation : [journalisation des diagnostics](#add-azure-monitor) requise avec Azure Monitor.

Comme un service de recherche ne prend pas en charge l’authentification par utilisateur, aucune information d’identité ne sera trouvée dans les journaux.

## <a name="built-in-monitoring"></a>Monitoring intégré

Le monitoring intégré fait référence aux activités journalisées par un service de recherche. À l’exception des diagnostics, aucune configuration n’est requise pour ce niveau de surveillance.

La Recherche cognitive Azure conserve les données internes suivant un calendrier de 30 jours continus pour le reporting sur l’intégrité du service et les métriques de requête, que vous trouverez sur le portail ou par le biais de ces [API REST](#monitoring-apis).

La capture d’écran suivante vous aide à trouver les informations de monitoring sur le portail. Les données sont disponibles dès que vous commencez à utiliser le service. Les pages du portail sont actualisées à des intervalles de quelques minutes.

* L’onglet **Monitoring**, sur la page Vue d’ensemble principale, présente le volume de requêtes et la latence, et indique si le service est sous pression.
* Le **Journal d’activité**, dans le volet de navigation gauche, est connecté à Azure Resource Manager. Le journal d’activité signale les actions effectuées par Resource Manager : la disponibilité et l’état du service, les modifications de la capacité (réplicas et partitions) et les activités liées à la clé API.
* Les paramètres **Monitoring**, plus bas, proposent des alertes configurables, des métriques et des journaux de diagnostic. Créez-les lorsque vous en avez besoin. Une fois les données collectées et stockées, vous pouvez interroger ou visualiser les informations pour obtenir des insights.

![Intégration d’Azure Monitor dans un service de recherche](./media/search-monitor-usage/azure-monitor-search.png
 "Intégration d’Azure Monitor dans un service de recherche")

> [!NOTE]
> Dans la mesure où les pages du portail sont actualisées à quelques minutes d’intervalle, les volumes rapportés sont approximatifs et destinés à donner une idée générale de l’efficacité avec laquelle le système traite les demandes. Les métriques réelles, telles que les requêtes par seconde, peuvent être supérieures ou inférieures au nombre indiqué dans la page. Si la précision fait partie des exigences, envisagez d’utiliser des API.

<a name="monitoring-apis"> </a>

### <a name="apis-useful-for-monitoring"></a>API utiles pour le monitoring

Vous pouvez utiliser les API suivantes pour récupérer les mêmes informations que dans les onglets Monitoring et Utilisation du portail.

* [GET Service Statistics](/rest/api/searchservice/get-service-statistics)
* [GET Index Statistics](/rest/api/searchservice/get-index-statistics)
* [GET Document Counts](/rest/api/searchservice/count-documents)
* [GET Indexer Status](/rest/api/searchservice/get-indexer-status)

### <a name="activity-logs-and-service-health"></a>Journaux d’activité et intégrité du service

La page [**Journal d’activité**](../azure-monitor/platform/activity-log.md#view-the-activity-log) du portail collecte des informations auprès d’Azure Resource Manager et signale les changements de l’intégrité du service. Vous pouvez superviser les conditions critiques, d’erreur et d’avertissement relatives à l’intégrité du service dans le journal d’activité.

Parmi les entrées courantes figurent les références à des clés API, à savoir des notifications d’information génériques comme *Obtenir une clé d’administration* et *Obtenir des clés de requête*. Ces activités indiquent les demandes effectuées à l’aide de la clé d’administration (créer ou supprimer des objets) ou d’une clé de requête, mais n’affichent pas la demande proprement dite. Pour plus d’informations de ce fragment, vous devez configurer la journalisation des diagnostics.

Vous pouvez accéder au **journal d’activité** à partir du volet de navigation de gauche, à partir de l’option Notifications dans la barre de commandes située en haut de la fenêtre ou à partir de la page **Diagnostiquer et résoudre les problèmes**.

### <a name="monitor-storage-in-the-usage-tab"></a>Monitoring du stockage dans l’onglet Utilisation

Pour la supervision visuelle dans le portail, l’onglet **Utilisation** affiche la disponibilité des ressources par rapport aux [limites](search-limits-quotas-capacity.md) actuelles imposées par le niveau de service. Si vous finalisez des décisions concernant le [niveau à utiliser pour les charges de travail de production](search-sku-tier.md) ou la nécessité d’[ajuster le nombre de partitions et de réplicas actifs](search-capacity-planning.md), ces métriques peuvent vous aider dans vos décisions en vous montrant rapidement les ressources consommées et l’efficacité de la configuration actuelle pour gérer la charge existante.

L’illustration suivante concerne le service gratuit, qui est limité à 3 objets de chaque type et à 50 Mo de stockage. Un service De base ou Standard a des limites plus élevées. De plus, si vous augmentez le nombre de partitions, le stockage maximal augmente proportionnellement.

![État d’utilisation par rapport aux limites du niveau](./media/search-monitor-usage/usage-tab.png
 "État d’utilisation par rapport aux limites du niveau")

> [!NOTE]
> Les alertes liées au stockage ne sont pas disponibles pour l’instant ; la consommation de stockage n’est pas agrégée ni consignée dans la table **AzureMetrics** dans Azure Monitor. Pour obtenir des alertes de stockage, vous devez [créer une solution personnalisée](../azure-monitor/insights/solutions.md) qui émet des notifications liées aux ressources et dont le code contrôle la taille de stockage et gère la réponse.

<a name="add-azure-monitor"></a>

## <a name="add-on-monitoring-with-azure-monitor"></a>Module complémentaire de surveillance avec Azure Monitor

De nombreux services, notamment la Recherche cognitive Azure, proposent une intégration avec [Azure Monitor](../azure-monitor/index.yml) pour offrir des alertes et des métriques supplémentaires, et permettre la journalisation des données de diagnostic. 

[Activez la journalisation des diagnostics](search-monitor-logs.md) pour un service de recherche si vous souhaitez contrôler la collecte et le stockage des données. Les événements journalisés capturés par Azure Monitor sont stockés dans la table **AzureDiagnostics** et se composent de données opérationnelles liées aux requêtes et à l’indexation.

Azure Monitor propose plusieurs options de stockage ; votre choix détermine la façon dont vous pouvez consommer les données :

* Choisissez le Stockage Blob Azure si vous souhaitez [visualiser les données de journal](search-monitor-logs-powerbi.md) dans un rapport Power BI.
* Choisissez Log Analytics si vous souhaitez explorer les données au moyen de requêtes Kusto.

Azure Monitor possède sa propre structure de facturation ; les journaux de diagnostic mentionnés dans cette section présentent un coût. Pour plus d’informations, consultez [Utilisation et coûts estimés dans Azure Monitor](../azure-monitor/platform/usage-estimated-costs.md).

## <a name="monitor-user-access"></a>Superviser l’accès des utilisateurs

Étant donné que les index de recherche sont un composant d’une plus grande application cliente, il n’existe aucune méthodologie intégrée pour contrôler ni superviser l’accès par utilisateur à un index. Les demandes sont supposées provenir d’une application cliente, que ce soit pour des demandes d’administration ou de requête. Les opérations de lecture-écriture d’administration incluent la création, la mise à jour et la suppression d’objets dans l’ensemble du service. Les opérations en lecture seule sont des requêtes sur la collection de documents, délimitées à un seul index. 

Par conséquent, ce que vous verrez dans les journaux d’activité sont des références à des appels avec des clés d’administration ou des clés de requête. La clé appropriée est incluse dans les demandes provenant du code client. Le service n’est pas équipé pour gérer les jetons d’identité ni l’emprunt d’identité.

Quand il existe des exigences commerciales pour l’autorisation par utilisateur, la recommandation est l’intégration à Azure Active Directory. Vous pouvez utiliser $filter et des identités utilisateur pour [filtrer les résultats de recherche](search-security-trimming-for-azure-search-with-aad.md) des documents qu’un utilisateur ne doit pas voir. 

Il n’existe aucun moyen de consigner ces informations séparément de la chaîne de requête qui inclut le paramètre $filter. Consultez [Superviser les requêtes](search-monitor-queries.md) pour plus d’informations sur la création de rapports sur les chaînes de requête.

## <a name="next-steps"></a>Étapes suivantes

Une bonne maîtrise d’Azure Monitor est essentielle pour la surveillance de tout service Azure, dont des ressources comme Recherche cognitive Azure. Si vous n’êtes pas familiarisé avec Azure Monitor, prenez le temps de consulter les articles relatifs aux ressources. En plus des tutoriels, l’article suivant est un bon point de départ.

> [!div class="nextstepaction"]
> [Supervision de ressources Azure avec Azure Monitor](../azure-monitor/insights/monitor-azure-resource.md)