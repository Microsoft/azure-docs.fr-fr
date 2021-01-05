---
title: Appliquer les recommandations en matière de performances
description: Utilisez le Portail Azure pour rechercher des suggestions vous permettant d’optimiser les performances de votre base de données.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: danimir
ms.author: danil
ms.reviewer: wiassaf, sstein
ms.date: 12/19/2018
ms.openlocfilehash: 748ac448ad8bf5c06e5be8b7a4a8b00a9b7af84b
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500884"
---
# <a name="find-and-apply-performance-recommendations"></a>Rechercher et appliquer les recommandations en matière de performances
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Vous pouvez utiliser le Portail Azure pour rechercher des suggestions vous permettant d’optimiser les performances de votre base de données dans Azure SQL Database ou pour résoudre certains problèmes identifiés dans votre charge de travail. La page **Recommandation en matière de performances** du Portail Azure vous permet de rechercher les meilleures suggestions en fonction de leur impact potentiel.

## <a name="viewing-recommendations"></a>Affichage des recommandations

Pour afficher et appliquer des recommandations en matière de performances, vous devez disposer des autorisations du [contrôle d’accès basé sur le rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md) adéquates dans Azure. Les autorisations **Lecteur** et **Collaborateur de base de données SQL** sont obligatoires pour afficher les recommandations, et les autorisations **Propriétaire** et **Collaborateur de base de données SQL** sont obligatoires pour exécuter toutes les actions, créer ou supprimer des index et annuler la création d’index.

Pour rechercher des suggestions en matière de performances sur le Portail Azure, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Accédez à **Tous les services** > **Bases de données SQL**, puis sélectionnez votre base de données.
3. Accédez à **Recommandation sur les performances** pour afficher les recommandations disponibles pour la base de données choisie.

Les recommandations en matière de performances s’affichent dans un tableau similaire à celui de la figure suivante :

![La capture d’écran montre les recommandations en matière de performances dans une table présentant des descriptions d’action et les recommandations correspondantes.](./media/database-advisor-find-recommendations-portal/recommendations.png)

Les recommandations sont triées en fonction de leur impact potentiel sur les performances dans les catégories suivantes :

| Impact | Description |
|:--- |:--- |
| Élevé |Les recommandations ayant un impact élevé fournissent l’impact le plus important sur les performances. |
| Moyenne |Les recommandations ayant un impact moyen améliorent les performances, mais pas de manière substantielle. |
| Faible |Les recommandations ayant un faible impact fournissent de meilleures performances, mais les améliorations ne sont toutefois pas significatives. |

> [!NOTE]
> Azure SQL Database a besoin surveiller les activités au moins pendant une journée afin d’identifier certaines recommandations. Azure SQL Database peut plus facilement optimiser des modèles de requête cohérents que des pics d’activité aléatoires. Si les recommandations ne sont pas disponibles actuellement, la page **Recommandation sur les performances** fournit un message explicatif.

Vous pouvez également afficher l’état des opérations historiques. Sélectionnez une suggestion ou un état pour afficher plus d’informations.

Voici un exemple de recommandation « Créer un index » dans le Portail Azure.

![Créer un index](./media/database-advisor-find-recommendations-portal/sql-database-performance-recommendation.png)

## <a name="applying-recommendations"></a>Application des recommandations

Grâce à Azure SQL Database, vous pouvez contrôler totalement la façon dont les recommandations sont activées à l’aide de l’une des trois options suivantes :

* Appliquer les recommandations individuelles une à la fois.
* Activer le réglage automatique pour appliquer automatiquement les recommandations.
* Pour implémenter une recommandation manuellement, exécutez le script T-SQL recommandé sur votre base de données.

Sélectionnez une recommandation pour afficher ses détails, puis cliquez sur **Afficher le script** pour savoir exactement comment la recommandation doit être créée.

La base de données reste en ligne tant que la recommandation est appliquée. L’utilisation de la recommandation en matière de performances ou du réglage automatique ne place jamais une base de données hors connexion.

### <a name="apply-an-individual-recommendation"></a>Appliquer une recommandation individuelle

Vous pouvez consulter et accepter les recommandations une à la fois.

1. Dans la page **Recommandations**, cliquez sur une recommandation.
2. Dans la page **Détails**, cliquez sur le bouton **Appliquer**.

   ![Appliquer une recommandation](./media/database-advisor-find-recommendations-portal/apply.png)

La recommandation sélectionnée est appliquée à la base de données.

### <a name="removing-recommendations-from-the-list"></a>Suppression de recommandations de la liste

Si votre liste de recommandations contient des éléments que vous souhaitez supprimer de la liste, vous pouvez ignorer la recommandation :

1. Sélectionnez une recommandation dans la liste des **Recommandations** pour afficher les informations.
2. Dans la page **Détails**, cliquez sur **Ignorer**.

Si vous le souhaitez, vous pouvez rajouter des éléments rejetés à la liste **Recommandations** :

1. Dans la page **Recommandations**, cliquez sur **Affichage ignoré**.
2. Sélectionnez un élément rejeté dans la liste pour afficher les détails le concernant.
3. Si vous le souhaitez, vous pouvez cliquer sur **Annuler le rejet** pour ajouter l’index à la liste principale de **Recommandations**.

> [!NOTE]
> Notez que, si le [paramétrage automatique](automatic-tuning-overview.md) de SQL Database est activé, et que vous avez manuellement ignoré l’une des recommandations de la liste, celle-ci ne s’appliquera jamais automatiquement. Il est pratique d’ignorer une recommandation pour activer le paramétrage automatique dans les cas où une recommandation en particulier ne doit pas s’appliquer.
> Vous pouvez rétablir l’ancien comportement en sélectionnant l’option Annuler l’action Ignorer pour ajouter les recommandations ignorées à la liste Recommandations.

### <a name="enable-automatic-tuning"></a>Activer le réglage automatique

Vous pouvez configurer votre base de données de manière à implémenter automatiquement des suggestions. Dès qu’une recommandation est disponible, elle est automatiquement appliquée. Comme pour toutes les recommandations gérées par le service, si l’impact sur les performances est négatif, la recommandation est annulée.

1. Dans la page **Recommandations**, cliquez sur **Automatiser**:

   ![Paramètres du conseiller](./media/database-advisor-find-recommendations-portal/settings.png)
2. Sélectionnez les actions à automatiser :

   ![Capture d’écran montrant où sélectionner les actions à automatiser.](./media/database-advisor-find-recommendations-portal/server.png)

> [!NOTE]
> Notez que l’option **DROP_INDEX** n’est actuellement pas compatible avec les applications utilisant la commutation de partition et les conseils d’index.

Une fois que vous avez sélectionné la configuration voulue, cliquez sur Appliquer.

### <a name="manually-apply-recommendations-through-t-sql"></a>Appliquer manuellement des recommandations par le biais de T-SQL

Sélectionnez une recommandation, puis cliquez sur **Afficher le script**. Exécutez ce script sur votre base de données pour appliquer la recommandation manuellement.

*Les index qui sont exécutés manuellement ne sont pas surveillés ni validés en ce qui concerne l’impact du service sur les performances*. Nous vous suggérons donc de surveiller ces index après leur création pour vérifier s’ils améliorent les performances et de les ajuster ou de les supprimer, si nécessaire. Pour plus d’informations sur la création d’index, consultez [CREATE INDEX (Transact-SQL)](/sql/t-sql/statements/create-index-transact-sql). En outre, les recommandations appliquées manuellement restent actives et figurent dans la liste des recommandations pendant 24 à 48 heures avant que le système ne les retire automatiquement. Pour supprimer une recommandation plus tôt, vous pouvez l’ignorer manuellement.

### <a name="canceling-recommendations"></a>Annulation de recommandations

Les recommandations ayant l’état **En attente**, **En cours de validation** ou **Réussite** peuvent être annulées. Les recommandations avec l'état **En cours d'exécution** ne peuvent pas être annulées.

1. Sélectionnez une recommandation dans la zone **Historique des réglages** pour ouvrir la page **Détails des recommandations**.
2. Cliquez sur **Annuler** pour abandonner le processus d'application de la recommandation.

## <a name="monitoring-operations"></a>Surveillance des opérations

L’application d’une recommandation ne se produit pas toujours instantanément. Le portail fournit des informations concernant l’état de la recommandation. Voici les états possibles des index :

| Statut | Description |
|:--- |:--- |
| Pending |La commande Appliquer la recommandation a été reçue et son exécution est planifiée. |
| En cours d'exécution |La recommandation est en cours d’application. |
| Validation |La recommandation a été correctement appliquée et le service évalue les avantages. |
| Succès |La recommandation a été correctement appliquée et les avantages ont été évalués. |
| Error |Une erreur s’est produite au cours du processus d'application de la recommandation. Il peut s’agir d’un problème temporaire ou éventuellement d’un changement de schéma dans la table auquel cas le script n’est plus valide. |
| Annulation |La recommandation a été appliquée, mais a été jugée non performante et est automatiquement annulée. |
| Annulée |La recommandation a été annulée. |

Cliquez sur une recommandation in-process dans la liste pour afficher plus d’informations :

![Capture d’écran montrant la liste des recommandations in-process.](./media/database-advisor-find-recommendations-portal/operations.png)

### <a name="reverting-a-recommendation"></a>Annulation d'une recommandation

Si vous avez utilisé les recommandations en matière de performances pour appliquer la recommandation (ce qui signifie que vous n’avez pas exécuté manuellement le script T-SQL), celle-ci annule automatiquement la modification s’il apparaît que son impact sur les performances est négatif. Si vous souhaitez simplement annuler une recommandation, vous pouvez procéder comme suit :

1. Sélectionnez une recommandation appliquée avec succès dans la zone **Historique de paramétrage** .
2. Dans la page **Détails de la recommandation**, cliquez sur **Annuler**.

![Index recommandés](./media/database-advisor-find-recommendations-portal/details.png)

## <a name="monitoring-performance-impact-of-index-recommendations"></a>Analyse de l’impact des recommandations d’index sur les performances

Une fois les recommandations correctement implémentées (actuellement, seulement les opérations d’index et les recommandations de paramétrage des requêtes), vous pouvez cliquer sur **Informations sur la requête** dans la page Détails de la recommandation pour ouvrir [Query Performance Insight](query-performance-insight-use.md) et voir l’impact de vos principales requêtes sur les performances.

![Surveiller l’impact sur les performances](./media/database-advisor-find-recommendations-portal/query-insights.png)

## <a name="summary"></a>Résumé

Azure SQL Database fournit des suggestions pour améliorer les performances des bases de données. Les scripts T-SQL vous aident à optimiser votre base de données, avec à la clé une amélioration des performances des requêtes.

## <a name="next-steps"></a>Étapes suivantes

Surveillez vos recommandations et continuez à les appliquer pour affiner les performances. Les charges de travail d’une base de données sont dynamiques et évoluent en permanence. Azure SQL Database continue à surveiller et à fournir des recommandations pouvant potentiellement améliorer les performances de votre base de données.

* Consultez [Réglage automatique](automatic-tuning-overview.md) pour en savoir plus sur le réglage automatique dans Azure SQL Database.
* Consultez [Recommandations en matière de performances](database-advisor-implement-performance-recommendations.md) pour obtenir une vue d’ensemble des recommandations relatives aux performances Azure SQL Database.
* Pour connaître l’impact de vos principales requêtes sur les performances, consultez [Query Performance Insights](query-performance-insight-use.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Magasin de requêtes](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store)
* [CREATE INDEX](/sql/t-sql/statements/create-index-transact-sql)
* [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md)