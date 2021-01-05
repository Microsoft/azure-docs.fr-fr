---
title: Surveiller Azure Cosmos DB avec Azure Monitor pour Cosmos DB | Microsoft Docs
description: Cet article décrit la fonctionnalité Azure Monitor pour Cosmos DB, qui fournit aux propriétaires de Cosmos DB une compréhension rapide des problèmes de performances et d’utilisation avec leurs comptes Cosmos DB.
author: mrbullwinkle
ms.author: mbullwin
ms.topic: conceptual
ms.date: 05/11/2020
ms.openlocfilehash: 5e40f91945f83af7880f272f1578370f2ee1ec42
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97608491"
---
# <a name="explore-azure-monitor-for-azure-cosmos-db"></a>Explorer Azure Monitor pour Azure Cosmos DB

Azure Monitor pour Azure Cosmos DB fournit une vue de l’ensemble des performances, des échecs, de la capacité et de l’intégrité opérationnelle pour toutes vos ressources Azure Cosmos DB dans une expérience interactive unifiée. Cet article vous aidera à comprendre les avantages de cette nouvelle expérience de surveillance et comment vous pouvez modifier et adapter l’expérience en fonction des besoins uniques de votre organisation.   

## <a name="introduction"></a>Introduction

Avant d’entrer dans les détails de l’expérience avec la fonctionnalité, vous devez comprendre comment cette fonctionnalité présente et affiche les informations. 

Elle offre :

* **Un point de vue à l’échelle** de vos ressources Azure Cosmos DB sur l’ensemble de vos abonnements dans un emplacement unique, avec la possibilité de limiter de manière sélective les abonnements et les ressources que vous souhaitez évaluer.

* **Une analyse dans le détail** d’une ressource Azure CosmosDB particulière afin de diagnostiquer les problèmes ou d’effectuer une analyse détaillée par catégorie : utilisation, défaillances, capacité et opérations. La sélection de l’une de ces options fournit une vue détaillée des métriques Azure Cosmos DB pertinentes.  

* **Personnalisable** : cette expérience s’appuie sur les modèles de classeur d’Azure Monitor, ce qui vous permet de modifier les métriques affichées, de régler ou de définir des seuils qui s’alignent sur vos limites, puis de les enregistrer dans un classeur personnalisé. Les graphiques du classeur peuvent ensuite être épinglés au tableau de bord Azure.  

Cette fonctionnalité ne vous oblige pas à activer ou à configurer quoi que ce soit : ces métriques Azure Cosmos DB sont collectées par défaut.

>[!NOTE]
>Il n’y a pas de frais facturés pour accéder à cette fonctionnalité et vous ne paierez que pour les fonctionnalités essentielles d’Azure Monitor que vous configurez ou activez, comme décrit dans la page [Détails de la tarification d’Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

## <a name="view-utilization-and-performance-metrics-for-azure-cosmos-db"></a>Afficher les métriques d’utilisation et de performances pour Azure Cosmos DB

Pour afficher l’utilisation et les performances de vos comptes de stockage pour l’ensemble de vos abonnements, procédez comme suit.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Recherchez **Monitor** et sélectionnez **Monitor**.

    ![Zone de recherche avec le mot « Monitor » et une liste déroulante indiquant les services « Monitor » avec une image de style compteur de vitesse](./media/cosmosdb-insights-overview/search-monitor.png)

3. Sélectionnez **Cosmos DB**.

    ![Capture d’écran du classeur de vue d’ensemble de Cosmos DB](./media/cosmosdb-insights-overview/cosmos-db.png)

### <a name="overview"></a>Vue d’ensemble

Dans **Vue d’ensemble**, le tableau affiche les métriques interactives d’Azure Cosmos DB. Vous pouvez filtrer les résultats en fonction des options que vous sélectionnez dans les listes déroulantes suivantes :

* **Abonnements** : seuls les abonnements qui ont une ressource Azure Cosmos DB sont répertoriés.  

* **Cosmos DB** : vous pouvez sélectionner tout, un sous-ensemble ou une ressource Azure Cosmos DB unique.

* **Intervalle de temps** : par défaut, affiche les quatre dernières heures d’informations en fonction des sélections correspondantes effectuées.

La vignette de compteur sous les listes déroulantes cumule le nombre total de ressources Azure Cosmos DB se trouvant dans les abonnements sélectionnés. Il existe des codes de couleur conditionnels ou des cartes thermiques pour les colonnes du classeur qui signalent des métriques de transaction. La couleur la plus profonde a la valeur la plus élevée et la couleur la plus claire désigne les valeurs les plus basses. 

La sélection d’une flèche déroulante en regard d’une des ressources Azure Cosmos DB révèle une répartition des métriques de performances au niveau du conteneur de base de données individuel :

![Liste déroulante développée révélant des conteneurs de base de données individuels et la répartition des performances associée](./media/cosmosdb-insights-overview/container-view.png)

Si vous sélectionnez le nom de ressource Azure Cosmos DB mis en surbrillance en bleu, vous obtiendrez la **vue d’ensemble** par défaut pour le compte Azure Cosmos DB associé. 

### <a name="failures"></a>Échecs

Sélectionnez **Défaillances** en haut de la page et la portion **Défaillances** du modèle de classeur s’ouvrira. Il affiche le nombre total de requêtes avec la distribution des réponses qui composent ces requêtes :

![Capture d’écran des défaillances avec répartition par type de requête HTTP](./media/cosmosdb-insights-overview/failures.png)

| Code |  Description       | 
|-----------|:--------------------|
| `200 OK`  | Une des opérations REST suivantes a réussi : </br>- GET sur une ressource. </br> - PUT sur une ressource. </br> - POST sur une ressource. </br> -POST sur une ressource de procédure stockée pour exécuter la procédure stockée.|
| `201 Created` | Une opération POST pour créer une ressource a réussi. |
| `404 Not Found` | L’opération tente d’agir sur une ressource qui n’existe plus. Par exemple, la ressource peut déjà avoir été supprimée. |

Pour obtenir la liste complète des codes d’état, consultez l’article [Codes d'état HTTP d’Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb).

### <a name="capacity"></a>Capacité

Sélectionnez **Capacité** en haut de la page et la portion **Capacité** du modèle de classeur s’ouvrira. Il vous montre le nombre de documents dont vous disposez, la croissance de votre document au fil du temps, l’utilisation des données et la quantité totale de stockage disponible qu’il vous reste.  Cela peut vous aider à identifier les problèmes potentiels de stockage et d’utilisation des données.

![Classeur de capacité](./media/cosmosdb-insights-overview/capacity.png) 

Comme avec le classeur de vue d’ensemble, la sélection de la liste déroulante en regard d’une ressource Azure Cosmos DB dans la colonne **Abonnement** affiche une répartition avec les conteneurs individuels qui composent la base de données.

### <a name="operations"></a>Opérations 

Sélectionnez **Opérations** en haut de la page et la portion **Opérations** du modèle de classeur s’ouvrira. Elle vous donne la possibilité de voir vos requêtes décomposées selon le type des requêtes effectuées. 

Par conséquent, dans l’exemple ci-dessous, vous constatez que `eastus-billingint` reçoit principalement des demandes de lecture, mais un petit nombre de requêtes upsert et de création. De son côté, `westeurope-billingint` est en lecture seule du point de vue des requêtes, au moins au cours des quatre dernières heures auxquelles le classeur est actuellement étendu par le biais de son paramètre d’intervalle de temps.

![Classeur d’opérations](./media/cosmosdb-insights-overview/operation.png) 

## <a name="pin-export-and-expand"></a>Épingler, exporter et développer

Vous pouvez épingler une des sections métriques à un [tableau de bord Azure](../../azure-portal/azure-portal-dashboards.md) en sélectionnant l’icône de punaise en haut à droite de la section.

![Exemple d’épinglage de section de métrique au tableau de bord](./media/cosmosdb-insights-overview/pin.png)

Pour exporter vos données au format Excel, sélectionnez la flèche vers le bas située à gauche de l’icône en forme de punaise.

![Icône d’exportation de classeur](./media/cosmosdb-insights-overview/export.png)

Pour développer ou réduire toutes les vues déroulantes dans le classeur, sélectionnez l’icône de développement à gauche de l’icône d’exportation :

![Icône de développement de classeur](./media/cosmosdb-insights-overview/expand.png)

## <a name="customize-azure-monitor-for-azure-cosmos-db"></a>Personnaliser Azure Monitor pour Azure Cosmos DB

Dans la mesure où cette expérience s’appuie sur des modèles de classeur Azure Monitor, vous avez la possibilité de **Personnaliser** > **Modifier** et **Enregistrer** une copie de votre version modifiée dans un classeur personnalisé. 

![Personnaliser la barre](./media/cosmosdb-insights-overview/customize.png)

Les classeurs sont enregistrés au sein d’un groupe de ressources, soit dans la section **Mes rapports** qui est privée ou dans la section **Rapports partagés** accessible à tous les utilisateurs ayant accès au groupe de ressources. Une fois que vous avez enregistré le classeur personnalisé, vous devez accéder à la galerie de classeurs pour le lancer.

![Lancez la galerie de classeurs à partir de la barre de commandes](./media/cosmosdb-insights-overview/gallery.png)

## <a name="troubleshooting"></a>Dépannage

Pour obtenir des conseils sur la résolution des problèmes, consultez l’[article de résolution des problèmes](troubleshoot-workbooks.md) pour les insights basés sur des classeurs.

## <a name="next-steps"></a>Étapes suivantes

* Configurez les [alertes de métriques](../platform/alerts-metric.md) et les [notifications d’intégrité du service](../../service-health/alerts-activity-log-service-notifications-portal.md) pour configurer l’alerte automatisée afin de faciliter la détection des problèmes.

* Découvrez les scénarios que les classeurs sont conçus pour prendre en charge, comment créer et personnaliser des rapports existants, et bien plus encore en consultant la rubrique [Créer des rapports interactifs avec les classeurs Azure Monitor](../platform/workbooks-overview.md).
