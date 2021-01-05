---
title: Présentation du réglage automatique
description: Azure SQL Database et Azure SQL Managed Instance analysent la requête SQL et s'adaptent automatiquement aux charges de travail utilisateur.
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: danimir
ms.author: danil
ms.reviewer: wiassaf, sstein
ms.date: 03/30/2020
ms.openlocfilehash: 4204254754307f8310d5ccfda19400de57381075
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500867"
---
# <a name="automatic-tuning-in-azure-sql-database-and-azure-sql-managed-instance"></a>Réglage manuel dans Azure SQL Database et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Le réglage automatique Azure SQL Database et Azure SQL Managed Instance fournit une optimisation des performances et une stabilisation des charges de travail grâce à un réglage continu des performances basé sur l'intelligence artificielle et l'apprentissage automatique.

Le réglage automatique est un service de performances intelligent entièrement géré qui utilise l’intelligence intégrée pour surveiller en continu les requêtes exécutées sur une base de données, dont il améliore automatiquement les performances. Pour cela, il adapte de manière dynamique la base de données à l’évolution des charges de travail et applique des recommandations de réglage. Le réglage automatique apprend horizontalement de toutes les bases de données sur Azure grâce à l’intelligence artificielle et améliore ses actions de réglage de manière dynamique. Plus une base de données s’exécute avec le réglage automatique activé, meilleures sont ses performances.

Le réglage automatique Azure SQL Database et Azure SQL Managed Instance est peut-être l'une des fonctionnalités les plus importantes à activer pour fournir des charges de travail de base de données stables avec des performances optimales.

## <a name="what-can-automatic-tuning-do-for-you"></a>En quoi peut vous aider le réglage automatique ?

- Réglage automatisé des performances des bases de données
- Vérification automatisée des gains de performances
- Restauration automatisée et correction automatique
- Historique des réglages
- Scripts Transact-SQL (T-SQL) d'action de réglage pour les déploiements manuels
- Surveillance proactive des performances des charges de travail
- Capacité de scale-out sur des centaines de milliers de bases de données
- Impact positif sur les ressources de DevOps et le coût total de possession

## <a name="safe-reliable-and-proven"></a>Sécurité, fiabilité et efficacité avérée

Les opérations de réglage appliquées aux bases de données dans Azure SQL Database n’affectent en rien les performances, même celles de vos charges de travail les plus lourdes. Le système a été conçu avec soin afin de ne pas interférer avec les charges de travail utilisateur. Les recommandations de réglage automatique sont appliquées uniquement pendant les périodes d’utilisation normale. Le système peut également désactiver temporairement les opérations de réglage automatique afin de protéger les performances des charges de travail. Dans ce cas, le message « Désactivé par le système » s’affiche dans le portail Azure. Le réglage automatique concerne les charges de travail avec la priorité de ressources la plus élevée.

Les mécanismes de réglage automatique sont matures et ont été éprouvés sur des millions de bases de données exécutées sur Azure. Les opérations de réglage automatique appliquées sont vérifiées automatiquement afin de s’assurer qu’il y a une amélioration positive des performances des charges de travail. Les recommandations de performances ayant un impact négatif sont détectées de manière dynamique et inversées rapidement. Grâce à l'historique des réglages enregistré, il existe une trace claire des améliorations de réglage apportées à chaque base de données dans Azure SQL Database et Azure SQL Managed Instance.

![Fonctionnement du réglage automatique](./media/automatic-tuning-overview/how-does-automatic-tuning-work.png)

Le réglage automatique Azure SQL Database partage sa logique de base avec la fonctionnalité de réglage automatique de SQL Server dans le moteur de base de données. Pour obtenir des informations techniques supplémentaires sur le mécanisme d’intelligence intégré, consultez [Réglage automatique dans SQL Server](/sql/relational-databases/automatic-tuning/automatic-tuning).

Pour obtenir une vue d’ensemble du fonctionnement du réglage automatique et des scénarios d’utilisation classiques, regardez la vidéo incorporée :

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Improve-Azure-SQL-Database-Performance-with-automatic-tuning/player]

## <a name="enable-automatic-tuning"></a>Activer le réglage automatique

- Vous pouvez [activer le réglage automatique d'Azure SQL Database à partir du portail Azure](automatic-tuning-enable.md) ou en utilisant l'instruction T-SQL [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql-set-options?view=azuresqldb-current).
- Vous pouvez activer le réglage automatique d'Azure SQL Managed Instance en utilisant l'instruction T-SQL [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql-set-options?view=azuresqldb-mi-current).

## <a name="automatic-tuning-options"></a>Options de réglage automatique

Les options de réglage automatique disponibles dans Azure SQL Database et Azure SQL Managed Instance sont les suivantes :

| Option de réglage automatique | Prise en charge d’une base de données unique et d’une base de données mise en pool | Prise en charge de la base de données d’instance |
| :----------------------------- | ----- | ----- |
| **CREATE INDEX** : identifie les index qui peuvent améliorer les performances de votre charge de travail, crée des index et vérifie automatiquement que les performances des requêtes sont améliorées. | Oui | Non |
| **DROP INDEX** : identifie quotidiennement les index redondants et en double, excepté pour les index uniques, ainsi que les index qui n’ont pas été utilisés depuis longtemps (>90 jours). Notez que l’option n’est pas compatible avec les applications utilisant la commutation de partition et les indicateurs d’index. La suppression des index inutilisés n’est pas prise en charge pour les niveaux de service Premium et Critique pour l’entreprise. | Oui | Non |
| **FORCE LAST GOOD PLAN** (correction de plan automatique) : identifie les requêtes Azure SQL utilisant un plan d'exécution qui est plus lent que le plan correct précédent, ainsi que les requêtes utilisant le dernier plan correct connu au lieu du plan de régression. | Oui | Oui |

### <a name="automatic-tuning-for-sql-database"></a>Réglage automatique pour SQL Database

Le réglage automatique Azure SQL Database utilise les recommandations de Database Advisor **CREATE INDEX**, **DROP INDEX** et **FORCE LAST GOOD PLAN** pour optimiser les performances de votre base de données. Pour plus d’informations, consultez [Recommandations des conseillers de base de données dans le portail Azure](database-advisor-find-recommendations-portal.md), dans [PowerShell](/powershell/module/az.sql/get-azsqldatabaserecommendedaction) et dans l’[API REST](/rest/api/sql/serverautomatictuning).

Vous pouvez appliquer manuellement les recommandations de réglage à l’aide du portail Azure ou laisser le réglage automatique les appliquer pour vous en toute autonomie. L’avantage de laisser le système appliquer de manière autonome les recommandations de réglage pour vous est qu’il valide automatiquement qu’il existe un accroissement des performances de la charge de travail. En revanche, si une régression est détectée, il annule automatiquement les recommandations de réglage. Notez qu’en cas de requêtes affectées par des recommandations de réglage qui ne sont pas exécutées fréquemment, la phase de validation peut, par nature, prendre jusqu’à 72 heures.

Si vous appliquez les recommandations de réglage via T-SQL, les mécanismes de validation automatique des performances et d’annulation ne sont pas disponibles. Les recommandations appliquées de cette manière restent actives et s’affichent dans la liste des recommandations de paramétrage pour 24-48 heures. avant que le système ne les retire automatiquement. Si vous souhaitez supprimer une recommandation avant cela, vous pouvez l’abandonner dans le portail Azure.

Vous pouvez activer ou désactiver les options de réglage automatique par base de données, ou vous pouvez les configurer au niveau du serveur et les appliquer sur chaque base de données qui hérite des paramètres du serveur. Les serveurs peuvent hériter des valeurs Azure par défaut pour les paramètres de réglage automatique. Actuellement, les valeurs Azure par défaut sont FORCE_LAST_GOOD_PLAN activé, CREATE_INDEX activé et DROP_INDEX désactivé.

> [!IMPORTANT]
> Depuis mars 2020, les modifications apportées aux paramètres par défaut Azure pour le réglage automatique prennent effet comme suit :
>
> - Les nouveaux paramètres par défaut Azure sont FORCE_LAST_GOOD_PLAN = enabled, CREATE_INDEX = disabled, and DROP_INDEX = disabled.
> - Les serveurs existants sans préférences de réglage automatique configurées sont automatiquement configurés pour HÉRITER les nouveaux paramètres par défaut Azure. Cela s’applique à tous les clients qui ont actuellement des paramètres de serveur pour un réglage automatique dans un état non défini.
> - Les nouveaux serveurs créés sont automatiquement configurés pour HÉRITER les nouveaux paramètres par défaut Azure (contrairement à auparavant, où la configuration de réglage automatique était dans un état non défini lors de la création d’un serveur).

Une méthode recommandée consiste à configurer les options de réglage automatique sur un serveur et à hériter des paramètres des bases de données appartenant au serveur parent. Cette méthode simplifie la gestion des options de réglage automatique pour un grand nombre de bases de données.

Pour en savoir plus sur la création de notifications par e-mail pour les recommandations de réglage automatique, consultez [Notifications par e-mail pour le réglage automatique](automatic-tuning-email-notifications-configure.md).

### <a name="automatic-tuning-for-azure-sql-managed-instance"></a>Réglage automatique pour Azure SQL Managed Instance

Le réglage automatique SQL Managed Instance prend uniquement en charge **FORCE LAST GOOD PLAN**. Pour plus d’informations sur la configuration des options de réglage automatique par le biais de T-SQL, consultez [Le réglage automatique introduit la correction de plan automatique](https://azure.microsoft.com/blog/automatic-tuning-introduces-automatic-plan-correction-and-t-sql-management/) et [Correction de plan automatique](/sql/relational-databases/automatic-tuning/automatic-tuning?view=sql-server-ver15#automatic-plan-correction).

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur l'intelligence intégrée utilisée dans le réglage automatique, consultez [L'intelligence artificielle règle Azure SQL Database](https://azure.microsoft.com/blog/artificial-intelligence-tunes-azure-sql-databases/).
- Pour découvrir les mécanismes sous-jacents du réglage automatique, consultez [Indexation automatique de millions de bases de données dans Microsoft Azure SQL Database](https://www.microsoft.com/research/uploads/prod/2019/02/autoindexing_azuredb.pdf).