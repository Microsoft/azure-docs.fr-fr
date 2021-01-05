---
title: Azure Monitor pour Azure Cache pour Redis | Microsoft Docs
description: Cet article décrit la fonctionnalité Azure Monitor pour Azure Cache pour Redis, qui permet aux propriétaires de cache de comprendre rapidement les problèmes de performances et d’utilisation.
ms.topic: conceptual
author: mrbullwinkle
ms.author: mbullwin
ms.date: 09/10/2020
ms.openlocfilehash: e7b8a8e131cf7e0aae051f42043abf0640c927e4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90887015"
---
# <a name="explore-azure-monitor-for-azure-cache-for-redis"></a>Explorer Azure Monitor pour Azure Cache pour Redis

Pour toutes vos ressources Azure Cache pour Redis, Azure Monitor pour Azure Cache pour Redis fournit une vue unifiée et interactive des éléments suivants :

- Performances globales
- Échecs
- Capacité
- Intégrité opérationnelle

Cet article vous aide à comprendre les avantages de cette nouvelle expérience de surveillance. Il montre également comment modifier et adapter l’expérience en fonction des besoins uniques de votre organisation.

## <a name="introduction"></a>Introduction

Avant de commencer l’expérience, vous devez comprendre la façon dont Azure Monitor pour Azure Cache pour Redis présente visuellement ses informations.

Elle offre :

- **Une perspective à grande échelle** de vos ressources Azure Cache pour Redis dans un emplacement unique pour l’ensemble de vos abonnements. Vous avez la possibilité de sélectionner uniquement les abonnements et les ressources que vous souhaitez évaluer.

- **Une analyse détaillée** d’une ressource Azure Cache pour Redis particulière. Vous pouvez diagnostiquer les problèmes et voir une analyse détaillée de l’utilisation, des échecs, de la capacité et des opérations. Sélectionnez l’une de ces catégories pour obtenir une vue détaillée des informations pertinentes.  

- **La personnalisation** de cette expérience, qui est basée sur des modèles de classeur Azure Monitor. L’expérience vous permet de changer les métriques affichées et de modifier ou définir les seuils qui s’alignent sur vos limites. Vous pouvez enregistrer les modifications dans un classeur personnalisé, puis épingler les graphiques du classeur à des tableaux de bord Azure.

Cette fonctionnalité ne vous oblige pas à activer ou à configurer quoi que ce soit. Les informations Azure Cache pour Redis sont collectées par défaut.

>[!NOTE]
>L’accès à cette fonctionnalité est gratuit. Vous êtes facturé uniquement pour les fonctionnalités essentielles d’Azure Monitor que vous configurez ou activez, comme décrit dans la page [Détails de la tarification d’Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

## <a name="view-utilization-and-performance-metrics-for-azure-cache-for-redis"></a>Afficher les métriques d’utilisation et de performances pour Azure Cache pour Redis

Pour afficher l’utilisation et les performances de vos comptes de stockage pour l’ensemble de vos abonnements, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Recherchez **Monitor** et sélectionnez **Monitor**.

    ![Zone de recherche avec le mot « Monitor » et le résultat de la recherche de services qui affiche « Monitor » avec un symbole de compteur de vitesse](./media/cosmosdb-insights-overview/search-monitor.png)

1. Sélectionnez **Azure Cache pour Redis**. Si cette option n’est pas présente, sélectionnez **Plus** > **Azure Cache pour Redis**.

### <a name="overview"></a>Vue d’ensemble

Dans **Vue d’ensemble**, le tableau affiche les métriques interactives d’Azure Cache pour Redis. Vous pouvez filtrer les résultats en fonction des options que vous sélectionnez dans les listes déroulantes suivantes :

- **Abonnements** : seuls les abonnements qui ont une ressource Azure Cache pour Redis sont répertoriés.  

- **Azure Cache pour Redis** : vous pouvez sélectionner tout, un sous-ensemble ou une ressource Azure Cache pour Redis unique.

- **Intervalle de temps** : par défaut, le tableau affiche les quatre dernières heures d’informations en fonction des sélections correspondantes.

Une vignette de compteur se trouve sous les listes déroulantes. Elle indique le nombre total de ressources Azure Cache pour Redis dans les abonnements sélectionnés. Des cartes thermiques ou des codes de couleur conditionnels pour les colonnes du classeur indiquent les métriques des transactions. La couleur la plus profonde représente la valeur la plus élevée. Les couleurs plus claires représentent des valeurs inférieures.

La sélection d’une flèche de liste déroulante à côté d’une des ressources Azure Cache pour Redis révèle une décomposition des métriques de performances au niveau de la ressource individuelle.

![Capture d’écran de l’expérience de présentation](./media/redis-cache-insights-overview/overview.png)

Lorsque vous sélectionnez le nom de la ressource Azure Cache pour Redis mis en surbrillance en bleu, vous voyez le tableau **Vue d’ensemble** par défaut pour le compte associé. Il affiche les colonnes suivantes :

- **Mémoire utilisée**
- **Pourcentage de mémoire utilisé**
- **Charge du serveur**
- **Chronologie de la charge du serveur**
- **UC**
- **Clients connectés**
- **Absences dans le cache**
- **Erreurs (Max)**

### <a name="operations"></a>Opérations

Lorsque vous sélectionnez **Opérations** en haut de la page, le tableau **Opérations** du modèle de classeur s’ouvre. Il affiche les colonnes suivantes :

- **Nombre total d’opérations**
- **Chronologie du total des opérations**
- **Opérations par seconde**
- **Gets**
- **Sets**

![Capture d’écran de l’expérience des opérations](./media/redis-cache-insights-overview/operations.png)

### <a name="usage"></a>Usage

Lorsque vous sélectionnez **Utilisation** en haut de la page, le tableau **Utilisation** du modèle de classeur s’ouvre. Il affiche les colonnes suivantes :

- **Lecture du cache**
- **Chronologie de lecture du cache**
- **Écriture dans le cache**
- **Correspondances dans le cache**
- **Absences dans le cache**

![Capture d’écran de l’expérience d’utilisation](./media/redis-cache-insights-overview/usage.png)

### <a name="failures"></a>Échecs

Lorsque vous sélectionnez **Échecs** en haut de la page, le tableau **Échecs** du modèle de classeur s’ouvre. Il affiche les colonnes suivantes :

- **Nombre total d’erreurs**
- **Failover/Erreurs**
- **UnresponsiveClient/Erreurs**
- **RDB/Erreurs**
- **AOF/Erreurs**
- **Export/Erreurs**
- **Dataloss/Erreurs**
- **Import/Erreurs**

![Capture d’écran des échecs avec décomposition par type de requête HTTP](./media/redis-cache-insights-overview/failures.png)

### <a name="metric-definitions"></a>Définitions de métriques

Pour obtenir la liste complète des définitions de métriques qui composent ces classeurs, consultez l’[article sur les mesures disponibles et les intervalles de création des rapports](../../azure-cache-for-redis/cache-how-to-monitor.md#available-metrics-and-reporting-intervals).

## <a name="view-from-an-azure-cache-for-redis-resource"></a>Afficher à partir d’une ressource Azure Cache pour Redis

Pour accéder à Azure Monitor pour Azure Cache pour Redis directement à partir d’une ressource :

1. Dans le portail Azure, sélectionnez Azure Cache pour Redis.

2. Dans la liste, choisissez une ressource Azure Cache pour Redis. Dans la section d’analyse, choisissez Insights.

    ![Capture d’écran des options de menu avec le mot « Insights » mis en surbrillance dans un cadre rouge](./media/redis-cache-insights-overview/insights.png)

Vous pouvez également accéder à ces vues en sélectionnant le nom d’une ressource Azure Cache pour Redis à partir du classeur au niveau d’Azure Monitor.

### <a name="resource-level-overview"></a>Vue d’ensemble au niveau des ressources

Le classeur **Vue d’ensemble** d’Azure Cache pour Redis affiche plusieurs métrique de performances qui vous permettent d’accéder aux éléments suivants :

- Graphiques de performances interactifs présentant les détails les plus importants relatifs aux performances d’Azure Cache pour Redis.

- Vignettes de métriques et d’état mettant en évidence les performances des partitions, le nombre total de clients connectés et la latence globale.

![Capture d’écran du tableau de bord Vue d’ensemble affichant des informations relatives aux performances de l’UC, à la mémoire utilisée, aux clients connectés, aux erreurs, aux clés expirées et aux clés supprimées](./media/redis-cache-insights-overview/resource-overview.png)

La sélection de l’un des autres onglets pour **Performances** ou **Opérations** ouvre les classeurs respectifs.

### <a name="resource-level-performance"></a>Performances au niveau des ressources

![Capture d’écran des graphiques de performances des ressources](./media/redis-cache-insights-overview/resource-performance.png)

### <a name="resource-level-operations"></a>Opérations au niveau des ressources

![Capture d’écran des graphiques d’opérations sur les ressources](./media/redis-cache-insights-overview/resource-operations.png)

## <a name="pin-export-and-expand"></a>Épingler, exporter et développer

Pour épingler une section de métrique à un [tableau de bord Azure](../../azure-portal/azure-portal-dashboards.md), sélectionnez le symbole en forme de punaise dans le coin supérieur droit de la section.

![Une section de métrique avec le symbole en forme de punaise mis en surbrillance](./media/cosmosdb-insights-overview/pin.png)

Pour exporter vos données dans un format Excel, sélectionnez la flèche vers le bas située à gauche du symbole en forme de punaise.

![Symbole d’exportation du classeur mis en surbrillance](./media/cosmosdb-insights-overview/export.png)

Pour développer ou réduire toutes les vues d’un classeur, sélectionnez le symbole de développement à gauche du symbole d’exportation.

![Symbole de développement du classeur mis en surbrillance](./media/cosmosdb-insights-overview/expand.png)

## <a name="customize-azure-monitor-for-azure-cache-for-redis"></a>Personnaliser Azure Monitor pour Azure Cache pour Redis

Cette expérience s’appuyant sur des modèles de classeur Azure Monitor, vous pouvez sélectionner **Personnaliser** > **Modifier** > **Enregistrer** pour enregistrer une copie de votre version modifiée dans un classeur personnalisé.

![Barre de commandes avec l’option Personnaliser mise en surbrillance](./media/cosmosdb-insights-overview/customize.png)

Les classeurs sont enregistrés dans un groupe de ressources, soit dans la section **Mes rapports**, soit dans la section **Rapports partagés**. **Mes rapports** n’est disponible que pour vous. **Rapports partagés** est disponible pour tous les utilisateurs ayant accès au groupe de ressources.

Une fois que vous avez enregistré un classeur personnalisé, accédez à la galerie de classeurs pour l’ouvrir.

![Barre de commandes avec l’option Galerie mise en surbrillance](./media/cosmosdb-insights-overview/gallery.png)

## <a name="troubleshooting"></a>Dépannage

Pour obtenir des conseils sur la résolution des problèmes, consultez l’[article de résolution des problèmes](troubleshoot-workbooks.md) pour les insights basés sur des classeurs.

## <a name="next-steps"></a>Étapes suivantes

* Configurez les [alertes de métriques](../platform/alerts-metric.md) et les [notifications d’intégrité du service](../../service-health/alerts-activity-log-service-notifications-portal.md) pour configurer des alertes automatisées qui facilitent la détection des problèmes.

* Découvrez les scénarios que les classeurs prennent en charge, comment créer ou personnaliser des rapports, et bien plus encore en consultant la rubrique [Créer des rapports interactifs avec les classeurs Azure Monitor](../platform/workbooks-overview.md).
