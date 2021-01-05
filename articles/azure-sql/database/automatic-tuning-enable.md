---
title: Activer le réglage automatique
description: Vous pouvez facilement activer le réglage automatique sur votre base de données à l’aide du portail Azure.
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: danimir
ms.author: danil
ms.reviewer: wiassaf, sstein
ms.date: 12/03/2019
ms.openlocfilehash: 35e2a73b0cfae104cee417e7d4a159e7fd169a17
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500901"
---
# <a name="enable-automatic-tuning-in-the-azure-portal-to-monitor-queries-and-improve-workload-performance"></a>Activer le réglage automatique dans le portail Azure pour superviser les requêtes et améliorer les performances de la charge de travail
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]


Azure SQL Database gère automatiquement les services de données qui supervisent vos requêtes en permanence et identifie l’action que vous pouvez effectuer pour améliorer les performances de votre charge de travail. Vous pouvez consulter les recommandations et les appliquer manuellement ou laisser Azure SQL Database appliquer automatiquement des actions correctives : il s’agit du **mode de réglage automatique**.

Pour activer le réglage automatique au niveau du serveur ou de la base de données, vous pouvez l’une des méthodes suivantes :

- [Portail Azure](automatic-tuning-enable.md#azure-portal)
- Appels de l’[API REST](automatic-tuning-enable.md#rest-api)
- Commandes [T-SQL](/sql/t-sql/statements/alter-database-transact-sql-set-options?view=azuresqldb-current&preserve-view=true)

> [!NOTE]
> Pour Azure SQL Managed Instance, l’option prise en charge FORCE_LAST_GOOD_PLAN peut être configurée par le biais de [T-SQL](https://azure.microsoft.com/blog/automatic-tuning-introduces-automatic-plan-correction-and-t-sql-management) uniquement. La configuration basée sur le portail Azure et les options de réglage automatique des index décrites dans cet article ne s’appliquent pas à Azure SQL Managed Instance.

> [!NOTE]
> La configuration des options de réglage automatique par le biais du modèle ARM (Azure Resource Manager) n’est pas prise en charge pour l’instant.

## <a name="enable-automatic-tuning-on-server"></a>Activer le réglage automatique sur le serveur

Au niveau du serveur, vous pouvez choisir d’hériter de la configuration du réglage automatique à partir de « Valeurs Azure par défaut » ou de ne pas hériter de la configuration. Les valeurs Azure par défaut sont FORCE_LAST_GOOD_PLAN activé, CREATE_INDEX désactivé et DROP_INDEX désactivé.

> [!IMPORTANT]
> Depuis mars 2020, les valeurs par défaut Azure pour le réglage automatique sont les suivantes :
>
> - FORCE_LAST_GOOD_PLAN activé, CREATE_INDEX désactivé et DROP_INDEX désactivé.
> - Les serveurs existants sans préférences de réglage automatique configurées sont automatiquement configurés pour hériter (INHERIT) des paramètres Azure par défaut. Cela s’applique à tous les clients qui ont actuellement des paramètres de serveur pour un réglage automatique dans un état non défini.
> - Les nouveaux serveurs créés sont automatiquement configurés pour hériter (INHERIT) des paramètres Azure par défaut (contrairement aux versions précédentes où la configuration du réglage automatique était dans un état non défini lors de la création d’un serveur).

### <a name="azure-portal"></a>Portail Azure

Pour activer le réglage automatique sur un [serveur](logical-servers.md) dans Azure SQL Database, accédez au serveur dans le portail Azure, puis sélectionnez **Paramétrage automatique** dans le menu.

![Capture d’écran montrant le Paramétrage automatique dans le Portail Azure, où vous pouvez appliquer des options pour un serveur.](./media/automatic-tuning-enable/server.png)

> [!NOTE]
> Notez que l’option **DROP_INDEX** n’est pas actuellement compatible avec les applications utilisant la commutation de partition et les indicateurs d’index, et ne doit pas être activée dans ces cas. La suppression des index inutilisés n’est pas prise en charge pour les niveaux de service Premium et Critique pour l’entreprise.

Sélectionnez les options de réglage automatique que vous souhaitez activer et sélectionnez **Appliquer**.

Les options de réglage automatique sur un serveur sont appliquées à toutes les bases de données de ce serveur. Par défaut, toutes les bases de données héritent de la configuration de leur serveur parent, mais celle-ci peut être remplacée et spécifiée individuellement pour chaque base de données.

### <a name="rest-api"></a>API REST

Pour en savoir plus sur l’utilisation d’une l’API REST pour activer le réglage automatique sur un **serveur**, consultez les [méthodes HTTP UPDATE et GET de réglage automatique d’un serveur](/rest/api/sql/serverautomatictuning).

## <a name="enable-automatic-tuning-on-an-individual-database"></a>Activer le réglage automatique sur une base de données individuelle

Azure SQL Database vous permet de spécifier individuellement la configuration du réglage automatique pour chaque base de données. Au niveau de la base de données, vous pouvez choisir d’hériter de la configuration du réglage automatique du serveur parent, des valeurs Azure par défaut ou de ne pas hériter de la configuration. Les valeurs Azure par défaut sont FORCE_LAST_GOOD_PLAN activé, CREATE_INDEX désactivé et DROP_INDEX désactivé.

> [!TIP]
> Il est généralement recommandé de gérer la configuration du réglage automatique au **niveau du serveur**, afin que les mêmes paramètres de configuration soient appliqués automatiquement à chaque base de données. Configurez le paramétrage automatique sur une base de données individuelle seulement si vous avez besoin que celle-ci ait des paramètres différents de ceux dont les autres ont hérité du même serveur.

### <a name="azure-portal"></a>Portail Azure

Pour activer le réglage automatique sur une **base de données unique**, accédez à la base de données dans le portail Azure et sélectionnez **Paramétrage automatique**.

Des paramètres de réglage automatique individuels peuvent être configurés séparément pour chaque base de données. Vous pouvez configurer manuellement une option de réglage automatique individuelle ou spécifier qu’une option hérite de ses paramètres du serveur.

![Capture d’écran montrant le Paramétrage automatique dans le Portail Azure, où vous pouvez appliquer des options pour une base de données unique.](./media/automatic-tuning-enable/database.png)

Notez que l’option DROP_INDEX n’est pas compatible avec les applications utilisant la commutation de partition et les indicateurs d’index, et ne doit pas être activée dans ces cas.

Une fois que vous avez sélectionné la configuration voulue, cliquez sur **Appliquer**.

### <a name="rest-api"></a>API REST

Pour en savoir plus sur l’utilisation de l’API REST afin d’activer le réglage automatique sur une base de données unique, consultez les [méthodes HTTP UPDATE et GET de réglage automatique dans Azure SQL Database](/rest/api/sql/databaseautomatictuning).

### <a name="t-sql"></a>T-SQL

Pour activer le réglage automatique sur une base de données par le biais de T-SQL, connectez-vous à la base de données et exécutez la requête suivante :

```SQL
ALTER DATABASE current SET AUTOMATIC_TUNING = AUTO | INHERIT | CUSTOM
```

Si vous affectez la valeur AUTO au réglage automatique, les Valeurs Azure par défaut sont appliquées. Si vous affectez la valeur INHERIT, la configuration du réglage automatique est héritée du serveur parent. Si vous choisissez CUSTOM, vous devez configurer manuellement le réglage automatique.

Pour configurer des options de réglage automatique individuelles par le biais de T-SQL, connectez-vous à la base de données et exécutez une requête telle que celle-ci :

```SQL
ALTER DATABASE current SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON, CREATE_INDEX = ON, DROP_INDEX = OFF)
```

Si vous affectez la valeur ON à l’option de réglage, les paramètres hérités par la base de données sont substitués et l’option de réglage est activée. Si vous affectez la valeur OFF, les paramètres hérités par la base de données sont également substitués et l’option de réglage est désactivée. L’option de réglage automatique pour laquelle DEFAULT est spécifié hérite de la configuration du réglage automatique à partir des paramètres au niveau du serveur.  

> [!IMPORTANT]
> Dans le cas de la [géoréplication active](auto-failover-group-overview.md), le réglage automatique doit être configuré sur la base de données primaire uniquement. Les actions de réglage appliquées automatiquement, comme la création ou la suppression d’index, sont automatiquement répliquées sur la base de données secondaire en lecture seule. Toute tentative d’activation du paramétrage automatique au moyen de T-SQL sur la base de données secondaire en lecture seule se solde par un échec, car une configuration de paramétrage différente sur la base de données secondaire en lecture seule n’est pas prise en charge.
>

Pour en savoir plus sur les options T-SQL permettant de configurer le réglage automatique, consultez [Options ALTER DATABASE SET (Transact-SQL)](/sql/t-sql/statements/alter-database-transact-sql-set-options?view=azuresqldb-current&preserve-view=true).

## <a name="disabled-by-the-system"></a>Désactivée par le système

Le réglage automatique surveille toutes les actions effectuées sur la base de données et, dans certains cas, il peut déterminer que le réglage automatique ne peut pas fonctionner correctement sur la base de données. Dans ce cas, l’option de réglage est désactivée par le système. Dans la plupart des cas, cela est dû au fait que le Magasin des requêtes n’est pas activé ou est en lecture seule sur une base de données spécifique.

## <a name="permissions"></a>Autorisations

Le réglage automatique étant une fonctionnalité Azure, pour vous en servir, vous devez utiliser les rôles intégrés d’Azure. L’authentification SQL seule ne suffit pas pour utiliser la fonctionnalité du portail Azure.

Pour utiliser le réglage automatique, l’autorisation minimale requise à accorder à l’utilisateur est le rôle [Contributeur SQL Database](../../role-based-access-control/built-in-roles.md#sql-db-contributor) intégré d’Azure. Vous pouvez également utiliser des rôles avec des privilèges plus élevés, comme Contributeur SQL Server, Contributeur SQL Managed Instance, Contributeur et Propriétaire.

## <a name="configure-automatic-tuning-e-mail-notifications"></a>Configurer les notifications par e-mail sur l’ajustement automatique

Consultez le guide des [notifications par e-mail du réglage automatique](automatic-tuning-email-notifications-configure.md).

## <a name="next-steps"></a>Étapes suivantes

- Lisez l’[article Réglage automatique](automatic-tuning-overview.md) pour en savoir plus sur le réglage automatique et sur la manière dont il peut vous aider à améliorer vos performances.
- Consultez [Recommandations en matière de performances](database-advisor-implement-performance-recommendations.md) pour obtenir une vue d’ensemble des recommandations relatives aux performances Azure SQL Database.
- Pour connaître l’impact de vos principales requêtes sur les performances, consultez [Query Performance Insights](query-performance-insight-use.md).
