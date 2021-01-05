---
title: Configurer et gérer une référence de contenu
titleSuffix: Azure SQL Managed Instance
description: Guide de référence de contenu qui explique comment configurer et gérer Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: guide
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 04/16/2019
ms.openlocfilehash: b7f2f060f32cf3bf92660a2e8b75a45b7ec76b82
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92779762"
---
# <a name="azure-sql-managed-instance-content-reference"></a>Référence du contenu Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Dans cet article, vous trouverez une référence du contenu des guides, des scripts et des explications qui vous aideront à gérer et à configurer Azure SQL Managed Instance.

## <a name="load-data"></a>Charger les données

- [Migrer vers Azure SQL Managed Instance](migrate-to-instance-from-sql-server.md) : découvrez les processus de migration et les outils recommandés pour la migration vers Azure SQL Managed Instance.
- [Migrer le certificat TDE vers Azure SQL Managed Instance](tde-certificate-migrate.md) : si votre base de données SQL Server est protégée par le chiffrement transparent des données (TDE), vous devez migrer le certificat que SQL Managed Instance peut utiliser pour déchiffrer la sauvegarde que vous souhaitez restaurer dans Azure.
- [Importer une base de données depuis un fichier BACPAC](../database/database-import.md)
- [Exporter une base de données vers un fichier BACPAC](../database/database-export.md)
- [Chargement des données avec BCP](../load-from-csv-with-bcp.md)
- [Téléchargement de données avec Azure Data Factory](../../data-factory/connector-azure-sql-database.md?toc=/azure/sql-database/toc.json)

## <a name="network-configuration"></a>Configuration réseau

- [Déterminer la taille du sous-réseau](vnet-subnet-determine-size.md) : le sous-réseau ne pouvant pas être redimensionné après le déploiement de SQL Managed Instance, vous devez calculer la plage d’adresses IP requise pour la quantité et les types d’instances managées que vous envisagez de déployer sur le sous-réseau. 
- [Créer un réseau virtuel et un sous-réseau](virtual-network-subnet-create-arm-template.md) : configurez le réseau virtuel et le sous-réseau en fonction de la [configuration réseau requise](connectivity-architecture-overview.md#network-requirements). 
- [Configurer un réseau virtuel et un sous-réseau existants](vnet-existing-add-subnet.md) : vérifiez la configuration réseau requise et configurez votre réseau virtuel et votre sous-réseau existants pour déployer SQL Managed Instance. 
- [Configurer un système DNS personnalisé](custom-dns-configure.md) : configurez un système DNS personnalisé pour accorder l’accès aux ressources externes à des domaines personnalisés à partir de SQL Managed Instance par le biais d’un serveur lié de profils de messagerie de base de données. 
- [Synchroniser la configuration du réseau](azure-app-sync-network-configuration.md) : actualisez le plan de configuration réseau si vous ne parvenez pas à établir une connexion après avoir [intégré votre application à un réseau virtuel Azure](../../app-service/web-sites-integrate-with-vnet.md).
- [Déterminer l’adresse IP du point de terminaison de gestion](management-endpoint-find-ip-address.md) : identifiez le point de terminaison public utilisé par SQL Managed Instance à des fins de gestion. 
- [Vérifier la protection de pare-feu intégrée](management-endpoint-verify-built-in-firewall.md) : vérifiez que SQL Managed Instance autorise le trafic uniquement sur les ports nécessaires, et d’autres règles de pare-feu intégrées. 
- [Connecter des applications](connect-application-instance.md) : découvrez les différents modèles de connexion des applications à SQL Managed Instance.

## <a name="feature-configuration"></a>Configuration des fonctionnalités

- [Configurer l’authentification Azure AD](../database/authentication-aad-configure.md)
- [Configurer un accès conditionnel](../database/conditional-access-configure.md)
- [Authentification multifacteur Azure AD](../database/authentication-mfa-ssms-overview.md)
- [Configuration de l’authentification multifacteur](../database/authentication-mfa-ssms-configure.md)
- [Configurer une stratégie de conservation temporelle](../database/temporal-tables-retention-policy.md)
- [Configurer TDE avec BYOK](../database/transparent-data-encryption-byok-configure.md)
- [Effectuer une rotation des clés TDE BYOK](../database/transparent-data-encryption-byok-key-rotation.md)
- [Supprimer un protecteur TDE](../database/transparent-data-encryption-byok-remove-tde-protector.md)
- [Configurer OLTP In-Memory](../in-memory-oltp-configure.md)
- [Configurer Azure Automation](../database/automation-manage.md)
- La [réplication transactionnelle](replication-between-two-instances-configure-tutorial.md) vous permet de répliquer vos données entre des instances managées, ou entre un serveur SQL local et SQL Managed Instance et vice-versa.
- [Configurer la détection des menaces](threat-detection-configure.md) : la [détection des menaces](../database/threat-detection-overview.md) est une fonctionnalité intégrée d’Azure SQL Managed Instance qui détecte les attaques potentielles comme l’injection SQL ou l’accès à partir d’emplacements suspects. 
- La [création d’alertes](alerts-create.md) vous permet de configurer des alertes sur des métriques supervisées, telles que l’utilisation du processeur, la consommation de l’espace de stockage, les IOPS, etc. pour SQL Managed Instance. 

## <a name="monitoring-and-tuning"></a>Analyse et réglage

- [Réglage manuel](../database/performance-guidance.md)
- [Utiliser des DMV pour surveiller les performances](../database/monitoring-with-dmvs.md)
- [Utiliser le Magasin des requêtes pour superviser les performances](/sql/relational-databases/performance/best-practice-with-the-query-store#Insight)
- [Résoudre les problèmes de performances avec Intelligence Insights](../database/intelligent-insights-troubleshoot-performance.md)
- [Utiliser le journal de diagnostic Intelligent Insights](../database/intelligent-insights-use-diagnostics-log.md)
- [Superviser l’espace OLTP en mémoire](../in-memory-oltp-monitor-space.md)

### <a name="extended-events"></a>Événements étendus

- [Événements étendus](../database/xevent-db-diff-from-svr.md)
- [Stocker des événements étendus dans un fichier d’événements](../database/xevent-code-event-file.md)
- [Capturer des événements étendus dans la mémoire tampon](../database/xevent-code-ring-buffer.md)

### <a name="alerting"></a>Génération d’alertes

- [Créer des alertes sur une instance managée](alerts-create.md)

## <a name="operations"></a>Opérations

- [Basculement manuel initié par l’utilisateur sur SQL Managed Instance](user-initiated-failover.md)

## <a name="develop-applications"></a>Développer des applications

- [Connectivité](../database/connect-query-content-reference-guide.md#libraries)
- [Utiliser Spark Connector](../../cosmos-db/spark-connector.md)
- [Authentifier une application](../database/application-authentication-get-client-id-keys.md)
- [Utiliser le traitement par lot pour améliorer les performances](../performance-improve-use-batching.md)
- [Conseils sur la connectivité](../database/troubleshoot-common-connectivity-issues.md)
- [Alias DNS](../database/dns-alias-overview.md)
- [Configurer un alias DNS à l’aide de PowerShell](../database/dns-alias-powershell-create.md)
- [Ports - ADO.NET](../database/adonet-v12-develop-direct-route-ports.md)
- [C et C ++](../database/develop-cplusplus-simple.md)
- [Excel](../database/connect-excel.md)

## <a name="design-applications"></a>Concevoir des applications

- [Conception de systèmes de récupération d’urgence](../database/designing-cloud-solutions-for-disaster-recovery.md)
- [Conception de pools élastiques](../database/disaster-recovery-strategies-for-applications-with-elastic-pool.md)
- [Conception de mises à niveau d’applications](../database/manage-application-rolling-upgrade.md)

### <a name="design-multi-tenant-saas-applications"></a>Conception d’applications SaaS mutualisées

- [Modèles de conception SaaS](../database/saas-tenancy-app-design-patterns.md)
- [Video Indexer SaaS](../database/saas-tenancy-video-index-wingtip-brk3120-20171011.md)
- [Sécurité des applications SaaS](../database/saas-tenancy-elastic-tools-multi-tenant-row-level-security.md)

## <a name="next-steps"></a>Étapes suivantes

Commencez par [déployer SQL Managed Instance](instance-create-quickstart.md).