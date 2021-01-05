---
title: Configurer et gérer une référence de contenu
description: Recherchez une référence de contenu qui vous apprend à configurer et à gérer Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: guide
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 01/14/2020
ms.openlocfilehash: 19cec1deeef344ea5897061db0fec88066b90b05
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92791764"
---
# <a name="configure-and-manage-content-reference---azure-sql-database"></a>Configurer et gérer une référence de contenu – Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Dans cet article, vous trouverez une référence de contenu des guides, des scripts et des explications pour vous aider à gérer et à configurer votre instance Azure SQL Database. 

## <a name="load-data"></a>Charger les données

- [Migrer vers SQL Database](migrate-to-database-from-sql-server.md)
- Découvrez comment [gérer Microsoft Azure SQL Database après la migration](manage-data-after-migrating-to-database.md).
- [Copier une base de données](database-copy.md)
- [Importer une base de données depuis un fichier BACPAC](database-import.md)
- [Exporter une base de données vers un fichier BACPAC](database-export.md)
- [Chargement des données avec BCP](../load-from-csv-with-bcp.md)
- [Chargement des données avec ADF](../../data-factory/connector-azure-sql-database.md?toc=/azure/sql-database/toc.json)

## <a name="configure-features"></a>Configurer des fonctionnalités

- [Configurer l’authentification Azure Active Directory (Azure AD)](authentication-aad-configure.md)
- [Configurer un accès conditionnel](conditional-access-configure.md)
- [Authentification multifacteur Azure AD](authentication-mfa-ssms-overview.md)
- [Configurer l’authentification multifacteur](authentication-mfa-ssms-configure.md)
- [Configurer une stratégie de rétention temporelle](temporal-tables-retention-policy.md)
- [Configurer TDE avec BYOK](transparent-data-encryption-byok-configure.md)
- [Effectuer une rotation des clés TDE BYOK](transparent-data-encryption-byok-key-rotation.md)
- [Supprimer le protecteur TDE](transparent-data-encryption-byok-remove-tde-protector.md)
- [Configurer OLTP In-Memory](../in-memory-oltp-configure.md)
- [Configurer Azure Automation](automation-manage.md)
- [Configurez la réplication transactionnelle](replication-to-sql-database.md) pour répliquer votre date entre les bases de données.
- [Configurez la détection des menaces](threat-detection-configure.md) pour permettre à Azure SQL Database d’identifier les activités suspectes, telles que l’injection de code SQL ou l’accès à partir d’emplacements suspects.
- [Configurez le masquage dynamique des données](dynamic-data-masking-configure-portal.md) pour protéger vos données sensibles.
- [Configurez la conservation des sauvegardes](long-term-backup-retention-configure.md) pour une base de données afin de conserver vos sauvegardes sur le Stockage Blob Azure. 
- [Configurez la géoréplication](active-geo-replication-overview.md) pour conserver un réplica de votre base de données dans une autre région.
- [Configurez la sécurité des géo-réplicas](active-geo-replication-security-configure.md).

## <a name="monitor-and-tune-your-database"></a>Superviser et régler votre base de données

- [Réglage manuel](performance-guidance.md)
- [Utiliser des DMV pour surveiller les performances](monitoring-with-dmvs.md)
- [Utiliser le Magasin des requêtes pour superviser les performances](/sql/relational-databases/performance/best-practice-with-the-query-store#Insight)
- [Activez le réglage automatique](automatic-tuning-enable.md) pour permettre à Azure SQL Database d’optimiser les performances de votre charge de travail.
- [Activez les notifications par e-mail pour le réglage automatique](automatic-tuning-email-notifications-configure.md) afin d’obtenir des informations sur les recommandations de réglage.
- [Appliquez les recommandations relatives aux performances](database-advisor-find-recommendations-portal.md) et optimisez votre base de données.
- [Créez des alertes](alerts-insights-configure-portal.md) pour obtenir des notifications à partir d’Azure SQL Database.
- [Résolvez les problèmes de connectivité](troubleshoot-common-errors-issues.md) si vous remarquez des problèmes de connectivité entre les applications et la base de données. Vous pouvez aussi utiliser [Resource Health pour les problèmes de connectivité](resource-health-to-troubleshoot-connectivity.md).
- [Résoudre les problèmes de performances avec Intelligence Insights](intelligent-insights-troubleshoot-performance.md)
- [Gérez l’espace fichier](file-space-manage.md) pour surveiller l’utilisation du stockage dans votre base de données.
- [Utiliser le journal de diagnostic Intelligent Insights](intelligent-insights-use-diagnostics-log.md)
- [Superviser l’espace OLTP en mémoire](../in-memory-oltp-monitor-space.md)

### <a name="extended-events"></a>Événements étendus

- [Événements étendus](xevent-db-diff-from-svr.md)
- [Stocker des événements étendus dans un fichier d’événements](xevent-code-event-file.md)
- [Capturer des événements étendus dans la mémoire tampon](xevent-code-ring-buffer.md)

## <a name="query-distributed-data"></a>Interroger des données distribuées

- [Interrogez des données partitionnées verticalement](elastic-query-getting-started-vertical.md) entre plusieurs bases de données.
- [Créez des rapports sur des couches de données Scale-out](elastic-query-horizontal-partitioning.md).
- [Interrogez des tables avec des schémas différents](elastic-query-vertical-partitioning.md).

### <a name="data-sync"></a>Synchronisation des données

- [Synchronisation des données SQL](sql-data-sync-data-sql-server-sql-database.md)
- [Agent Data Sync](sql-data-sync-agent-overview.md)
- [Répliquer les modifications de schéma](sql-data-sync-update-sync-schema.md)
- [Surveiller avec OMS](./monitor-tune-overview.md)
- [Bonnes pratiques pour Data Sync](sql-data-sync-best-practices.md)
- [Dépannage de Data Sync](sql-data-sync-troubleshoot.md)

## <a name="elastic-database-jobs"></a>Tâches de base de données élastique

- [Créez et gérez](elastic-jobs-powershell-create.md) des tâches de bases de données élastiques à l'aide de PowerShell.
- [Créez et gérez](elastic-jobs-tsql-create-manage.md) des tâches de bases de données élastiques à l'aide de Transact-SQL.
- [Migrez à partir d’anciens travaux élastiques](elastic-jobs-migrate.md).

## <a name="database-sharding"></a>Partitionnement de base de données

- [Mettez à niveau la bibliothèque cliente de base de données élastique](elastic-scale-upgrade-client-library.md).
- [Créez une application partitionnée](elastic-scale-get-started.md).
- [Interrogez des données partitionnées horizontalement](elastic-query-getting-started.md).
- Exécutez des [requêtes sur plusieurs partitions](elastic-scale-multishard-querying.md).
- [Déplacez des données partitionnées](elastic-scale-configure-deploy-split-and-merge.md).
- [Configurez la sécurité](elastic-scale-split-merge-security-configuration.md) dans des partitions de base de données.
- [Ajoutez une partition](elastic-scale-add-a-shard.md) à l’ensemble actuel des partitions de base de données.
- [Résolvez les problèmes de carte de partitions](elastic-database-recovery-manager.md).
- [Migrez la base de données partitionnée](elastic-convert-to-use-elastic-tools.md).
- [Créez des compteurs](elastic-database-perf-counters.md).
- [Utilisez le framework d’entité](elastic-scale-use-entity-framework-applications-visual-studio.md) pour interroger des données partitionnées.
- [Utilisez le framework Dapper](elastic-scale-working-with-dapper.md) pour interroger des données partitionnées.

## <a name="develop-applications"></a>Développer des applications

- [Connectivité](connect-query-content-reference-guide.md#libraries)
- [Utiliser Spark Connector](spark-connector.md)
- [Authentifier l’application](application-authentication-get-client-id-keys.md)
- [Utiliser le traitement par lot pour améliorer les performances](../performance-improve-use-batching.md)
- [Conseils sur la connectivité](troubleshoot-common-connectivity-issues.md)
- [Alias DNS](dns-alias-overview.md)
- [Configurer le PowerShell d’alias DNS](dns-alias-powershell-create.md)
- [Ports - ADO.NET](adonet-v12-develop-direct-route-ports.md)
- [C et C ++](develop-cplusplus-simple.md)
- [Excel](connect-excel.md)

## <a name="design-applications"></a>Concevoir des applications

- [Conception de systèmes de récupération d’urgence](designing-cloud-solutions-for-disaster-recovery.md)
- [Conception de pools élastiques](disaster-recovery-strategies-for-applications-with-elastic-pool.md)
- [Conception de mises à niveau d’applications](manage-application-rolling-upgrade.md)

### <a name="design-multi-tenant-software-as-a-service-saas-applications"></a>Concevoir des applications SaaS mutualisées

- [Modèles de conception SaaS](saas-tenancy-app-design-patterns.md)
- [Video Indexer SaaS](saas-tenancy-video-index-wingtip-brk3120-20171011.md)
- [Sécurité des applications SaaS](saas-tenancy-elastic-tools-multi-tenant-row-level-security.md)

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [Guides pratiques pour Azure SQL Managed Instance](../managed-instance/how-to-content-reference-guide.md)