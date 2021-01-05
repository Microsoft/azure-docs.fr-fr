---
title: Recommandations sur les performances - Azure Database pour MySQL
description: Cet article décrit la fonctionnalité Recommandations sur les performances dans Azure Database pour MySQL
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 6/3/2020
ms.openlocfilehash: 1445ad3d6be020326c34d60c555c24c435548fd0
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94537643"
---
# <a name="performance-recommendations-in-azure-database-for-mysql"></a>Recommandations sur les performances dans Azure Database pour MySQL

**S’applique à :** Azure Database pour MySQL 5.7, 8.0

La fonctionnalité Recommandations sur les performances analyse vos bases de données pour créer des suggestions personnalisées afin d’améliorer les performances. Pour produire les recommandations, l’analyse examine les différentes caractéristiques de la base de données, y compris le schéma. Activez le [Magasin des requêtes](concepts-query-store.md) sur votre serveur pour utiliser pleinement la fonctionnalité Recommandations sur les performances. Si le schéma de performances est désactivé, l’activation du magasin de données des requêtes activera performance_schema et un sous-ensemble d’instruments de schéma de performances requis pour la fonctionnalité. Après avoir implémenté une recommandation sur les performances, vous devez tester les performances pour évaluer l’impact des changements.

## <a name="permissions"></a>Autorisations

Les autorisations **Propriétaire** ou **Contributeur** sont nécessaires pour exécuter l’analyse avec la fonctionnalité Recommandations sur les performances.

## <a name="performance-recommendations"></a>Recommandations en matière de performances

La fonctionnalité [Recommandations sur les performances](concepts-performance-recommendations.md) analyse les charges de travail sur votre serveur pour identifier les index ayant le potentiel pour améliorer les performances.

Ouvrez **Recommandations sur les performances** à partir de la section **Performances intelligentes** de la barre de menus sur la page du portail Azure pour votre serveur MySQL.

:::image type="content" source="./media/concepts-performance-recommendations/performance-recommendations-page.png" alt-text="Page d’accueil des suggestions sur les performances":::

Sélectionnez **Analyser** et choisissez une base de données, l’analyse commence alors. En fonction de votre charge de travail, l’analyse peut prendre plusieurs minutes. Une fois l’analyse terminée, une notification apparaît dans le portail. L’analyse effectue une étude approfondie de votre base de données. Nous vous recommandons d’effectuer une analyse pendant les périodes creuses.

La fenêtre **Recommandations** affiche une liste de recommandations si des recommandations ont été trouvées, ainsi que l’ID de requête associé qui a généré cette recommandation. Avec l’ID de requête, vous pouvez utiliser la vue [mysql.query_store](concepts-query-store.md#mysqlquery_store) pour en savoir plus sur la requête.

:::image type="content" source="./media/concepts-performance-recommendations/performance-recommendations-result.png" alt-text="Nouvelle page Suggestions sur les performances":::

Les recommandations ne sont pas appliquées automatiquement. Pour appliquer la recommandation, copiez le texte de la requête et exécutez-le à partir du client de votre choix. N’oubliez pas de tester et de surveiller pour évaluer la recommandation.

## <a name="recommendation-types"></a>Types de recommandation

### <a name="index-recommendations"></a>Suggestions d'index

Les recommandations de *création d’index* suggèrent de nouveaux index pour accélérer les requêtes les plus fréquemment exécutées ou prenant du temps dans la charge de travail. Ce type de recommandation nécessite l’activation du [magasin des requêtes](concepts-query-store.md). Le magasin des requêtes collecte des informations de requête et fournit de façon détaillée les statistiques de fréquence et le runtime des requêtes, utilisés par l’analyse pour faire apporter la recommandation.

### <a name="query-recommendations"></a>Suggestions de requête

Les suggestions de requête suggèrent des optimisations et des réécritures pour les requêtes dans la charge de travail. En identifiant les anti-modèles de requêtes MySQL et en les corrigeant syntaxiquement, vous pouvez améliorer les performances des requêtes qui prennent du temps. Ce type de suggestion nécessite l’activation du Magasin des requêtes. Le magasin des requêtes collecte des informations de requête et fournit de façon détaillée les statistiques de fréquence et le runtime des requêtes, utilisés par l’analyse pour faire apporter la recommandation.

## <a name="next-steps"></a>Étapes suivantes
- Découvrez plus d’informations sur la [surveillance et l’optimisation](concepts-monitoring.md) dans Azure Database pour MySQL.