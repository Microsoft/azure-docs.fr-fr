---
title: Query Performance Insight
description: La supervision des performances de requêtes identifie les requêtes les plus longues et consommant le plus de processeur pour les bases de données uniques et mises en pool dans Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: danimir
ms.author: danil
ms.reviewer: wiassaf, sstein
ms.date: 03/10/2020
ms.openlocfilehash: a016781e5b05c8f43c043bf1f3368a58064e43ad
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96501258"
---
# <a name="query-performance-insight-for-azure-sql-database"></a>Query Performance Insight pour Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Query Performance Insight fournit une analyse intelligente des requêtes pour les bases de données uniques et mises en pool. Il aide à identifier les requêtes les plus longues et les plus consommatrices de ressources dans votre charge de travail. Cela vous permet de trouver les requêtes à optimiser pour améliorer les performances globales de charge de travail et pour utiliser efficacement la ressource que vous payez. Query Performance Insight vous permet de résoudre plus rapidement les problèmes de performances des bases de données en fournissant :

* Un insight approfondi de votre consommation des ressources des bases de données (DTU)
* Des informations sur les requêtes de base de données principales par processeur, la durée et le nombre d’exécutions (candidats de réglage possibles pour l’amélioration des performances)
* La possibilité d’explorer une requête, de voir son texte et l’historique d’utilisation des ressources
* Les annotations qui montrent des recommandations de performances à partir des [conseillers en bases de données](database-advisor-implement-performance-recommendations.md)

![Query Performance Insight](./media/query-performance-insight-use/opening-title.png)

## <a name="prerequisites"></a>Prérequis

Query Performance Insight nécessite que le [magasin de requêtes](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store) soit actif sur votre base de données. Il est automatiquement activé pour toutes les bases de données dans Azure SQL Database par défaut. Si le magasin de requêtes n’est pas exécuté, le portail Azure vous invite à l’activer.

> [!NOTE]
> Si le message « Le magasin de requêtes n’est pas correctement configuré sur cette base de données » s’affiche dans le portail, consultez [Optimisation de la configuration du magasin de requêtes](#optimize-the-query-store-configuration).

## <a name="permissions"></a>Autorisations

Vous avez besoin des autorisations du [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md) suivantes pour utiliser Query Performance Insight :

* Les autorisations **Lecteur**, **Propriétaire**, **Contributeur**, **Contributeur de base de données SQL** ou **Contributeur SQL Server** sont obligatoire pour voir les requêtes et graphiques consommant le plus de ressources.
* Les autorisations **Owner**, **Contributor**, **SQL DB Contributor** ou **SQL Server Contributor** sont requises pour afficher le texte de requête.

## <a name="use-query-performance-insight"></a>Utiliser Query Performance Insight

Query Performance Insight est simple d’utilisation :

1. Ouvrez le [portail Azure](https://portal.azure.com/) et recherchez la base de données que vous souhaitez examiner.
2. Dans le menu de gauche, ouvrez **Performances intelligentes** > **Query Performance Insight**.
  
   ![Query Performance Insight dans le menu](./media/query-performance-insight-use/tile.png)

3. Dans le premier onglet, passez en revue la liste des requêtes consommant le plus de ressources.
4. Sélectionnez une requête pour en afficher les détails.
5. Ouvrez **Performances intelligentes** > **Recommandations sur les performances** et vérifiez si des recommandations de performances sont disponibles. Pour plus d’informations sur les recommandations de performances intégrées, consultez [Azure SQL Database Advisor](database-advisor-implement-performance-recommendations.md).
6. Utilisez les curseurs ou les icônes de zoom pour changer l’intervalle observé.

   ![Tableau de bord des performances](./media/query-performance-insight-use/performance.png)

> [!NOTE]
> Pour qu’Azure SQL Database affiche les informations dans Query Performance Insight, le Magasin des requêtes doit capturer quelques heures de données. Si la base de données n’a pas d’activité ou que le magasin de requêtes est resté inactif pendant une certaine période, les graphiques sont vides quand Query Performance Insight affiche cet intervalle de temps. Vous pouvez activer le magasin de requêtes à tout moment s’il n’est pas en cours d’exécution. Pour plus d’informations, consultez [Bonnes pratiques avec le magasin des requêtes](/sql/relational-databases/performance/best-practice-with-the-query-store).
>

Pour des recommandations sur les performances de base de données, sélectionnez [Recommandations](database-advisor-implement-performance-recommendations.md) dans le panneau de navigation de Query Performance Insight.

![Onglet Recommandations](./media/query-performance-insight-use/ia.png)

## <a name="review-top-cpu-consuming-queries"></a>Examiner les requêtes principales consommatrices de processeur

Par défaut, Query Performance Insight affiche les cinq premières requêtes consommatrices de processeur quand vous l’ouvrez pour la première fois.

1. Sélectionnez ou désélectionnez des requêtes individuelles pour les inclure ou les exclure du graphique à l’aide des cases à cocher.

   La première ligne montre le pourcentage DTU général de la base de données. Les barres montrent le pourcentage de processeur que les requêtes ont consommé pendant l’intervalle sélectionné. Par exemple, si **Semaine dernière** est sélectionné, chaque barre représente une journée.

   ![Premières requêtes](./media/query-performance-insight-use/top-queries.png)

   > [!IMPORTANT]
   > La ligne DTU indiquée est agrégée à une valeur maximale de consommation dans des périodes d’une heure. Elle sert de moyen de comparaison au niveau global uniquement avec les statistiques d’exécution de requête. Dans certains cas, l’utilisation de DTU peut sembler trop élevée par rapport aux requêtes exécutées, mais ce n’est peut-être pas le cas.
   >
   > Par exemple, si une requête atteint la limite DTU de 100 % pendant quelques minutes seulement, la ligne DTU dans Query Performance Insight affiche l’heure entière de consommation à 100 % (la conséquence de la valeur agrégée maximale).
   >
   > Pour une comparaison plus fine (jusqu’à une minute), créez un graphique d’utilisation de DTU personnalisé :
   >
   > 1. Dans le portail Azure, sélectionnez **Azure SQL Database** > **Supervision**.
   > 2. Sélectionnez **Métriques**.
   > 3. Sélectionnez **+Ajouter un graphique**.
   > 4. Sélectionnez le pourcentage de DTU sur le graphique.
   > 5. Par ailleurs, sélectionnez **Dernières 24 heures** dans le menu en haut à gauche et remplacez la valeur par 1 minute.
   >
   > Utilisez le graphique DTU personnalisé avec un niveau de détails plus fin pour la comparaison avec le graphique d’exécution de requête.

   La grille du bas montre les informations agrégées pour les requêtes visibles :

   * ID de requête, qui est un identificateur unique de la requête dans la base de données.
   * Processeur par requête pendant l’intervalle observable (dépend de la fonction d’agrégation).
   * Durée par requête, qui dépend aussi de la fonction d’agrégation.
   * Nombre total d’exécutions pour une requête particulière.

2. Si vos données deviennent obsolètes, sélectionnez le bouton **Actualiser**.

3. Utilisez les curseurs et les boutons de zoom pour changer l’intervalle d’observation et examiner les pics de consommation :

   ![Curseurs et boutons de zoom pour changer l’intervalle](./media/query-performance-insight-use/zoom.png)

4. Vous pouvez aussi sélectionner l’onglet **Personnalisé** pour personnaliser la vue des éléments suivants :

   * Mesures (processeur, durée, nombre d’exécutions).
   * Intervalle de temps (dernières 24 heures, semaine dernière ou mois dernier).
   * le nombre de requêtes ;
   * la fonction d’agrégation.
  
   ![Onglet Personnalisé](./media/query-performance-insight-use/custom-tab.png)
  
5. Sélectionnez le bouton **Atteindre >** pour voir la vue personnalisée.

   > [!IMPORTANT]
   > Query Performance Insight affiche seulement les 5 à 20 premières requêtes consommatrices, selon votre sélection. Votre base de données peut exécuter plus de requêtes que les premières affichées, mais elles ne figurent pas sur le graphique.
   >
   > Vous pouvez avoir un type de charge de travail de base de données dans lequel beaucoup de requêtes plus petites (bien plus que celles affichées) s’exécutent souvent et utilisent la plupart des DTU. Ces requêtes n’apparaissent pas sur le graphique de performances.
   >
   > Par exemple, une requête peut avoir consommé une quantité substantielle de DTU pendant un certain temps, alors que sa consommation totale sur la période observée est inférieure à celle des autres requêtes les plus consommatrices. Dans ce cas, l’utilisation des ressources de cette requête n’apparaît pas sur le graphique.
   >
   > Si vous avez besoin de comprendre les exécutions des principales requêtes au-delà des limites de Query Performance Insight, utilisez [Azure SQL Analytics](../../azure-monitor/insights/azure-sql.md) pour une supervision avancée des performances de base de données et la résolution des problèmes.
   >

## <a name="view-individual-query-details"></a>Voir les détails d’une requête individuelle

Pour afficher les détails d’une requête :

1. Sélectionnez une requête dans la liste des principales requêtes.

    ![Liste des principales requêtes](./media/query-performance-insight-use/details.png)

   Une vue détaillée s’ouvre. Elle montre le nombre d’exécutions, la durée et la consommation du processeur dans le temps.

2. Sélectionnez les fonctionnalités de graphique pour plus d’informations.

   * Le graphique du haut montre une ligne avec le pourcentage général de DTU de base de données. Les barres représentent le pourcentage de processeur consommé par la requête sélectionnée.
   * Le deuxième graphique montre la durée totale de la requête sélectionnée.
   * Le graphique du bas montre le nombre total d’exécutions pour la requête sélectionnée.

   ![Détails de la requête](./media/query-performance-insight-use/query-details.png)

3. Vous pouvez également utiliser les curseurs ou les boutons de zoom, ou sélectionner **Paramètres** pour personnaliser l’affichage des données de requête ou pour choisir un autre intervalle de temps.

   > [!IMPORTANT]
   > Query Performance Insight ne capture aucune requête DDL. Dans certains cas, il peut ne pas capturer toutes les requêtes ad hoc.
   >

## <a name="review-top-queries-per-duration"></a>Révision des requêtes principales par durée

Deux métriques dans Query Performance Insight peuvent vous aider à trouver les goulots d’étranglement potentiels : le nombre d’exécutions et la durée.

Les requêtes de longue durée ont le plus grand risque de verrouiller des ressources le plus longtemps, de bloquer d’autres utilisateurs et de limiter l’évolutivité. Mais elles sont également facilement optimisables.

Pour identifier les requêtes longues :

1. Ouvrez l’onglet **Personnalisé** dans Query Performance Insight pour la base de données sélectionnée.
2. Remplacez la métrique par **durée**.
3. Sélectionnez le nombre de requêtes et l’intervalle d’observation.
4. Sélectionnez la fonction d’agrégation :

   * **Somme** calcule la durée d’exécution totale de toutes les requêtes pendant tout l’intervalle d’observation.
   * **Max** recherche les requêtes avec un temps d’exécution maximum pendant tout l’intervalle d’observation.
   * **Moy** calcule la durée d’exécution moyenne de toutes les exécutions de requête et montre les premières requêtes avec ces moyennes.

   ![Durée de la requête](./media/query-performance-insight-use/top-duration.png)

5. Sélectionnez le bouton **Atteindre >** pour voir la vue personnalisée.

   > [!IMPORTANT]
   > Le réglage de la vue de requête ne met pas à jour la ligne DTU. La ligne DTU montre toujours la valeur maximale de consommation pour l’intervalle.
   >
   > Pour comprendre la consommation de DTU de base de données plus en détail (jusqu’à une minute), créez un graphique personnalisé dans le portail Azure :
   >
   > 1. Sélectionner **Azure SQL Database** > **Supervision**.
   > 2. Sélectionnez **Métriques**.
   > 3. Sélectionnez **+Ajouter un graphique**.
   > 4. Sélectionnez le pourcentage de DTU sur le graphique.
   > 5. Par ailleurs, sélectionnez **Dernières 24 heures** dans le menu en haut à gauche et remplacez la valeur par 1 minute.
   >
   > Nous vous recommandons d’utiliser le graphique DTU personnalisé pour la comparaison avec le graphique de performances de requête.
   >

## <a name="review-top-queries-per-execution-count"></a>Révision des requêtes principales par nombre d’exécutions

Une application utilisateur qui utilise la base de données peut devenir lente, même si le nombre élevé d’exécutions n’affecte peut-être pas la base de données et que l’utilisation des ressources est faible.

Dans certains cas, un nombre d’exécutions élevé peut augmenter le nombre d’allers-retours réseau. Les allers-retours réseau affectent les performances. Ils peuvent entraîner une latence du réseau et une latence du serveur en aval.

Par exemple, de nombreux sites web pilotés par les données accèdent à la base de données à chaque requête de l’utilisateur. Même si le regroupement de connexions a un effet positif, l’augmentation du trafic réseau et de la charge de traitement sur le serveur peuvent ralentir les performances. En règle générale, maintenez le nombre d’allers-retours au minimum.

Pour identifier les requêtes fréquemment exécutées (« bavardes ») :

1. Ouvrez l’onglet **Personnalisé** dans Query Performance Insight pour la base de données sélectionnée.
2. Remplacez la métrique par **nombre d’exécutions**.
3. Sélectionnez le nombre de requêtes et l’intervalle d’observation.
4. Sélectionnez le bouton **Atteindre >** pour voir la vue personnalisée.

   ![Nombre d’exécutions de requête](./media/query-performance-insight-use/top-execution.png)

## <a name="understand-performance-tuning-annotations"></a>Présentation des annotations de réglage des performances

Quand vous explorez votre charge de travail dans Query Performance Insight, vous pouvez remarquer des icônes avec une ligne verticale en haut du graphique.

Ces icônes sont des annotations. Elles montrent les recommandations de performances d’[Azure SQL Database Advisor](database-advisor-implement-performance-recommendations.md). En plaçant le curseur sur une annotation, vous pouvez obtenir des informations récapitulatives sur les recommandations de performances.

   ![Annotation de requête](./media/query-performance-insight-use/annotation.png)

Pour plus d’informations sur la recommandation du conseiller ou pour l’appliquer, sélectionnez l’icône permettant d’ouvrir les détails de l’action recommandée. S’il s’agit d’une recommandation active, vous pouvez l’appliquer directement dans le portail.

   ![Détails d’annotation de requête](./media/query-performance-insight-use/annotation-details.png)

Dans certains cas, en raison du niveau de zoom, les annotations proches les unes des autres peuvent être réduites en une seule annotation. Query Performance Insight représente cela sous la forme d’une icône d’annotation de groupe. Sélectionnez l’icône d’annotation de groupe pour ouvrir un nouveau panneau qui liste les annotations.

La corrélation de requêtes et des actions de réglage des performances peut vous aider à mieux comprendre votre charge de travail.

## <a name="optimize-the-query-store-configuration"></a>Optimiser la configuration du magasin de requêtes

Pendant l’utilisation de Query Performance Insight, vous pouvez voir les messages suivants du magasin de requêtes :

* « Le magasin de requête n’est pas correctement configuré dans cette base de données. Cliquez ici pour en savoir plus. »
* « Le magasin de requête n’est pas correctement configuré dans cette base de données. Cliquez ici pour modifier les paramètres. »

Ces messages s’affichent généralement quand le magasin de requêtes ne peut plus collecter de nouvelles données.

Le premier cas se produit quand le magasin de requêtes est en lecture seule et que les paramètres sont définis de façon optimale. Vous pouvez résoudre ce problème en augmentant la taille du magasin de requêtes ou en le nettoyant. (Si vous nettoyez le magasin de requêtes, toutes les données de télémétrie collectées précédemment sont perdues.)

   ![Détails du magasin de requêtes](./media/query-performance-insight-use/qds-off.png)

Le deuxième cas se produit quand le magasin de requêtes est désactivé ou que les paramètres ne sont pas définis de façon optimale. Vous pouvez changer la stratégie de conservation et de capture, et aussi activer le magasin de requêtes en exécutant les commandes suivantes fournies dans [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) ou le portail Azure.

### <a name="recommended-retention-and-capture-policy"></a>Stratégie de rétention et de capture recommandée

Il existe deux types de stratégies de rétention :

* **En fonction de la taille** : Si cette stratégie est définie sur **AUTO**, les données sont automatiquement nettoyées quand la taille maximale est atteinte.
* **En fonction de l’heure** : Par défaut, cette stratégie est définie sur 30 jours. Si le magasin de requêtes manque d’espace, il supprime les informations de requête de plus de 30 jours.

Vous pouvez définir la stratégie de capture sur :

* **Tout** : Le magasin des requêtes capture toutes les requêtes.
* **Auto** : Le magasin des requêtes ignore les requêtes peu fréquentes et les requêtes avec une durée de compilation et d’exécution insignifiante. Les seuils du nombre d’exécutions, de la durée de compilation et de la durée d’exécution sont déterminés en interne. Il s'agit de l'option par défaut.
* **Aucun** : Le magasin de requêtes arrête la capture de nouvelles requêtes, mais les statistiques d’exécution pour les requêtes déjà capturées sont toujours collectées.

Nous vous recommandons de définir toutes les stratégies sur **AUTO** et la stratégie de nettoyage sur 30 jours en exécutant les commandes suivantes dans [SSMS](/sql/ssms/download-sql-server-management-studio-ssms) ou le portail Azure. (Remplacez `YourDB` par le nom de la base de données.)

```sql
    ALTER DATABASE [YourDB]
    SET QUERY_STORE (SIZE_BASED_CLEANUP_MODE = AUTO);

    ALTER DATABASE [YourDB]
    SET QUERY_STORE (CLEANUP_POLICY = (STALE_QUERY_THRESHOLD_DAYS = 30));

    ALTER DATABASE [YourDB]
    SET QUERY_STORE (QUERY_CAPTURE_MODE = AUTO);
```

Augmentez la taille du magasin de requêtes en vous connectant à une base de données dans [SSMS](/sql/ssms/download-sql-server-management-studio-ssms) ou le portail Azure et en exécutant la requête suivante. (Remplacez `YourDB` par le nom de la base de données.)

```SQL
    ALTER DATABASE [YourDB]
    SET QUERY_STORE (MAX_STORAGE_SIZE_MB = 1024);
```

L’application de ces paramètres permet au magasin de requêtes de collecter les données de télémétrie des nouvelles requêtes. Si vous avez besoin que le magasin de requêtes soit immédiatement opérationnel, vous pouvez choisir de le nettoyer en exécutant la requête suivante dans SSMS ou le portail Azure. (Remplacez `YourDB` par le nom de la base de données.)

> [!NOTE]
> L’exécution de la requête suivante supprime toute les données de télémétrie supervisées collectées précédemment dans le magasin de requêtes.

```SQL
    ALTER DATABASE [YourDB] SET QUERY_STORE CLEAR;
```

## <a name="next-steps"></a>Étapes suivantes

Utilisez [Azure SQL Analytics](../../azure-monitor/insights/azure-sql.md) pour une supervision avancée des performances d’une flotte importante de bases de données uniques et mises en pool, de pools élastiques, d’instances managées et de bases de données d’instance.