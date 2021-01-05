---
title: Recommandations automatisées sur les performances, les coûts et la sécurité pour Azure Cosmos DB
description: Découvrez comment afficher les recommandations personnalisées en matière de performances, de coût, de sécurité et d’autres recommandations pour Azure Cosmos DB en fonction de vos modèles de charge de travail.
author: ThomasWeiss
ms.author: thweiss
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 07/28/2020
ms.reviewer: sngun
ms.openlocfilehash: d9eb55030c7ec52f9b2ac79fbab19944f0a3e190
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93087823"
---
# <a name="automated-recommendations-for-azure-cosmos-db"></a>Recommandations automatisées pour Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Tous les services cloud comme Azure Cosmos DB reçoivent des mises à jour fréquentes s’accompagnant de nouvelles fonctionnalités, de nouvelles capacités et des améliorations. Il est important pour votre application de suivre les dernières mises à jour de sécurité et de performances. Le portail Azure offre des recommandations personnalisées qui vous permettent d’optimiser le niveau de performance de votre application. Le moteur de conseil d’Azure Cosmos DB analyse en permanence l’historique d’utilisation de vos ressources Azure Cosmos DB et fournit des recommandations en fonction de vos modèles de charge de travail. Ces recommandations correspondent à différents domaines comme le partitionnement, l’indexation, le réseau, la sécurité, etc. Ces recommandations personnalisées vous aident à améliorer le niveau de performance de votre application.

## <a name="view-recommendations"></a>Afficher les recommandations

Vous pouvez afficher les recommandations pour Azure Cosmos DB de différentes manières :

- L’un des moyens de consulter les recommandations est d’accéder à l’onglet Notifications. S’il existe de nouvelles recommandations, une barre de message est affichée. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre compte Azure Cosmos. Dans votre compte Azure Cosmos, ouvrez le volet **Notifications** , puis sélectionnez l’onglet **Recommandations**. Vous pouvez sélectionner le message et afficher les recommandations.  

   :::image type="content" source="./media/automated-recommendations/cosmos-db-pane-recommendations.png" alt-text="Afficher les recommandations à partir du volet Azure Cosmos DB":::

- Vous pouvez aussi chercher les recommandations via [Azure Advisor](../advisor/advisor-overview.md). Celles-ci sont classées par catégories dans différents groupes tels que le coût, la sécurité, la fiabilité, le niveau de performance et l’excellence opérationnelle. Vous pouvez sélectionner des abonnements spécifiques et filtrer par type de ressource, en l’occurrence : **Comptes Azure Cosmos DB**.  Quand vous sélectionnez une recommandation spécifique, elle affiche les actions que vous pouvez entreprendre au profit de vos charges de travail.

   :::image type="content" source="./media/automated-recommendations/advisor-pane-recommendations.png" alt-text="Afficher les recommandations à partir du volet Azure Advisor":::

Les recommandations affichées dans le volet Azure Cosmos DB ne sont pas toutes disponibles dans Azure Advisor et vice versa. En effet, cela dépend du type de recommandation auquel elles appartiennent dans le volet Azure Advisor, le volet Azure Cosmos ou les deux.

Pour l’heure, Azure Cosmos DB prend en charge les recommandations dans les domaines suivants. Chacune de ces recommandations s’accompagne d’un lien vers la section correspondante de la documentation ; il est donc facile de passer aux étapes suivantes.

## <a name="sdk-usage-recommendations"></a>Recommandations en matière d’utilisation de kits SDK

Dans cette catégorie, le conseiller détecte l’utilisation d’une ancienne version de kits SDK et vous recommande de procéder à une mise à niveau vers une version plus récente pour tirer parti des derniers correctifs de bogues et des améliorations sur le plan des performances. À l’heure actuelle, voici les recommandations propres aux SDK qui sont disponibles :

|Nom  |Description  |
|---------|---------|
| Ancienne version du connecteur Spark | Détecte l’utilisation d’anciennes versions du connecteur Spark et recommande une mise à niveau. |
| Ancienne version du SDK .NET | Détecte l’utilisation d’anciennes versions du SDK .NET et recommande une mise à niveau. |
| Ancienne version du SDK Java | Détecte l’utilisation d’anciennes versions du connecteur Java et recommande une mise à niveau. |

## <a name="indexing-recommendations"></a>Recommandations d’indexation

Dans cette catégorie, le conseiller détecte le mode d’indexation, la stratégie d’indexation, les chemins indexés et recommande de changer si la configuration actuelle a un impact sur le niveau de performance des requêtes. À l’heure actuelle, voici les recommandations d’indexation disponibles :

|Nom  |Description  |
|---------|---------|
| Indexation différée | Détecte l’utilisation du mode d’indexation différée et recommande d’utiliser plutôt le mode d’indexation cohérente. Dans la mesure où la fonction du mode d’indexation différée d’Azure Cosmos DB est limitée et qu’elle peut avoir un impact sur l’actualisation des résultats de requêtes dans certaines situations, le mode d’indexation cohérente est recommandé. |
| Indexation composite| Détecte les comptes où les requêtes pourraient bénéficier d’index composites et recommande leur utilisation. Les index composites peuvent améliorer considérablement le niveau de performance et la consommation de débit de certaines requêtes.|
| Stratégie d’indexation par défaut avec de nombreux chemins indexés | Détecte les conteneurs s’exécutant dans l’indexation par défaut avec de nombreux chemins indexés et recommande une personnalisation de la stratégie d’indexation.|
| Requêtes ORDER BY présentant des frais de RU/s élevés| Détecte les conteneurs émetteurs de requêtes ORDER BY présentant des frais de RU/s élevés et recommande d’explorer les index composites.|
| Comptes MongoDB 3.6 sans index présentant une consommation de RU/s élevée| Détecte l’API d’Azure Cosmos DB pour MongoDB avec des conteneurs de version 3.6 émettant des requêtes avec des frais de RU/s élevés et recommande d’ajouter des index.|

## <a name="cost-optimization-recommendations"></a>Recommandations d’optimisation des coûts

Dans cette catégorie, le conseiller détecte l’utilisation de RU/s et détermine que vous pouvez optimiser le prix en apportant des modifications à vos ressources ou en mettant à profit un autre modèle tarifaire. À l’heure actuelle, voici les recommandations d’optimisation des coûts disponibles :

|Nom  |Description  |
|---------|---------|
| Capacité réservée | Détecte votre utilisation de RU/s et recommande des instances réservées aux utilisateurs qui peuvent en bénéficier. |
| Conteneurs inactifs | Détecte les conteneurs qui n’ont pas été utilisés depuis plus de 30 jours et recommande de réduire le débit pour ces conteneurs ou de les supprimer.|
| Nouveaux abonnements avec un débit élevé | Détecte les nouveaux abonnements dont les comptes présentent une consommation de RU/s par jour exceptionnellement élevée par jour et leur adresse une notification. Cette notification vise précisément à attirer l’attention des nouveaux clients sur le mode de fonctionnement d’Azure Cosmos DB, qui repose sur un modèle basé sur le débit provisionné et non sur un modèle basé sur la consommation. |

## <a name="migration-recommendations"></a>Recommandations en matière de migration

Dans cette catégorie, le conseiller détecte que vous utilisez les fonctionnalités héritées et vous recommande de migrer de façon à tirer parti de la scalabilité considérable s d’Azure Cosmos DB et d’autres avantages. À l’heure actuelle, voici les recommandations de migration disponibles :

|Nom  |Description  |
|---------|---------|
| Conteneurs non partitionnés | Détecte les conteneurs de taille fixe qui approchent de leur limite de stockage maximale et recommande de les migrer vers des conteneurs partitionnés.|

## <a name="query-usage-recommendations"></a>Recommandations en matière d’utilisation de requêtes

Dans cette catégorie, le conseiller détecte l’exécution de requêtes et identifie que les performances des requêtes peuvent être ajustées moyennant certaines modifications. À l’heure actuelle, voici les recommandations d’utilisation de requêtes qui sont disponibles :

|Nom  |Description  |
|---------|---------|
| Requêtes présentant une taille de page fixe | Détecte les requêtes émises avec une taille de page fixe et recommande l’utilisation de la valeur -1 (aucune limite sur la taille de page) au lieu de définir une valeur spécifique. Cette option réduit le nombre d’allers-retours réseau nécessaires pour récupérer tous les résultats. |

## <a name="next-steps"></a>Étapes suivantes

* [Réglage des performances des requêtes dans Azure Cosmos DB](sql-api-query-metrics.md)
* [Résoudre les problèmes de requêtes](troubleshoot-query-performance.md) quand Azure Cosmos DB est utilisé
