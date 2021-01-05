---
title: Recommandations sur les performances - Azure Database pour PostgreSQL - Serveur unique
description: Cet article décrit la fonctionnalité Recommandations sur les performances dans Azure Database pour PostgreSQL - Serveur unique.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: conceptual
ms.date: 08/21/2019
ms.openlocfilehash: d547844671e6485e71be7dd8c355de08f3dec5e6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91710564"
---
# <a name="performance-recommendations-in-azure-database-for-postgresql---single-server"></a>Recommandations sur les performances dans Azure Database pour PostgreSQL - Serveur unique

**S’applique à :** Azure Database pour PostgreSQL - Serveur unique versions 9.6, 10, 11

La fonctionnalité Recommandations sur les performances analyse vos bases de données pour créer des suggestions personnalisées pour améliorer les performances. Pour produire les recommandations, l’analyse examine les différentes caractéristiques de la base de données, y compris le schéma. Activez le [Magasin des requêtes](concepts-query-store.md) sur votre serveur pour utiliser pleinement la fonctionnalité Recommandations sur les performances. Après avoir implémenté une recommandation sur les performances, vous devez tester les performances pour évaluer l’impact des changements. 

## <a name="permissions"></a>Autorisations
Les autorisations **Propriétaire** ou **Contributeur** sont nécessaires pour exécuter l’analyse avec la fonctionnalité Recommandations sur les performances.

## <a name="performance-recommendations"></a>Recommandations en matière de performances
La fonctionnalité [Recommandations sur les performances](concepts-performance-recommendations.md) analyse les charges de travail sur votre serveur pour identifier les index ayant le potentiel pour améliorer les performances.

Ouvrez **Recommandations sur les performances** à partir de la section **Performances intelligentes** de la barre de menus sur la page du portail Azure pour votre serveur PostgreSQL.

:::image type="content" source="./media/concepts-performance-recommendations/performance-recommendations-page.png" alt-text="Page d’accueil des suggestions sur les performances":::

Sélectionnez **Analyser** et choisissez une base de données, l’analyse commence alors. En fonction de votre charge de travail, l’analyse peut prendre plusieurs minutes. Une fois l’analyse terminée, une notification apparaît dans le portail. L’analyse effectue une étude approfondie de votre base de données. Nous vous recommandons d’effectuer une analyse pendant les périodes creuses. 

La fenêtre **Recommandations** affiche une liste de recommandations si des recommandations ont été trouvées.

:::image type="content" source="./media/concepts-performance-recommendations/performance-recommendations-result.png" alt-text="Page d’accueil des suggestions sur les performances":::

Les recommandations ne sont pas appliquées automatiquement. Pour appliquer la recommandation, copiez le texte de la requête et exécutez-le à partir du client de votre choix. N’oubliez pas de tester et de surveiller pour évaluer la recommandation. 

## <a name="recommendation-types"></a>Types de recommandation

Deux types de recommandations sont actuellement prises en charge : *Création d’index* et *Suppression d’index*.

### <a name="create-index-recommendations"></a>Recommandations de création d’index
Les recommandations de *création d’index* suggèrent de nouveaux index pour accélérer les requêtes les plus fréquemment exécutées ou prenant du temps dans la charge de travail. Ce type de recommandation nécessite l’activation du [magasin des requêtes](concepts-query-store.md). Le magasin des requêtes collecte des informations de requête et fournit de façon détaillée les statistiques de fréquence et le runtime des requêtes, utilisés par l’analyse pour faire apporter la recommandation.

### <a name="drop-index-recommendations"></a>Recommandations de suppression d’index
Outre la détection des index manquants, Azure Database pour PostgreSQL procède à l’analyse continue du niveau de performance des index existants. Si un index est rarement utilisé ou redondant, l’analyseur recommande sa suppression.

## <a name="considerations"></a>Considérations
* Il n’existe pas de recommandations concernant les performances pour les [réplicas en lecture](concepts-read-replicas.md).
## <a name="next-steps"></a>Étapes suivantes
- Découvrez plus d’informations sur la [surveillance et l’optimisation](concepts-monitoring.md) dans Azure Database pour PostgreSQL.

