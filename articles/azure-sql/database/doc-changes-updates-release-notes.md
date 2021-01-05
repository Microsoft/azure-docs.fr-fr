---
title: Nouveautés
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Découvrez les nouvelles fonctionnalités et améliorations apportées à la documentation et aux services Azure SQL Database et SQL Managed Instance.
services: sql-database
author: stevestein
ms.service: sql-db-mi
ms.subservice: service
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
ms.date: 06/17/2020
ms.author: sstein
ms.openlocfilehash: 36c12fa7dd37ce1ffebde16cf6ca856d9fcdca0a
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93391974"
---
# <a name="whats-new-in-azure-sql-database--sql-managed-instance"></a>Nouveautés d’Azure SQL Database et de SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Cet article liste les fonctionnalités Azure SQL Database et Azure SQL Managed Instance actuellement en préversion publique. Pour connaître les mises à jour et améliorations apportées à SQL Database et SQL Managed Instance, consultez [Mises à jour des services SQL Database et SQL Managed Instance](https://azure.microsoft.com/updates/?product=sql-database). Pour connaître les mises à jour et améliorations apportées aux autres services Azure, consultez [Mises à jour des services](https://azure.microsoft.com/updates).

## <a name="whats-new"></a>Nouveautés

La documentation relative à Azure SQL Database et Azure SQL Managed Instance a été divisée en sections distinctes. Nous avons également mis à jour la manière dont nous faisons référence à une instance managée en remplaçant *instance managée Azure SQL Database* par *instance Azure SQL Managed Instance*.

Nous avons procédé à ce changement parce que certaines caractéristiques et fonctionnalités varient considérablement entre une base de données unique et une instance gérée, et il est devenu de plus en plus difficile d’expliquer les nuances complexes entre Azure SQL Database et Azure SQL Managed Instance dans des articles individuels communs.

Cette clarification entre les différents produits Azure SQL devrait simplifier et rationaliser le processus d’utilisation du moteur de base de données SQL Server dans Azure, qu’il s’agisse d’une base de données managée unique dans Azure SQL Database, d’une instance managée à part entière hébergeant plusieurs bases de données dans Azure SQL Managed Instance ou du produit SQL Server familier hébergé sur une machine virtuelle dans Azure.

Considérez qu’il s’agit d’un travail en cours et que les articles n’ont pas tous été encore mis à jour. Par exemple, la documentation sur les instructions Transact-SQL (T-SQL), les procédures stockées et de nombreuses fonctionnalités partagées entre Azure SQL Database et Azure SQL Managed Instance n’étant pas encore complète, nous vous remercions de votre patience car nous continuons à clarifier le contenu. 

Ce tableau fournit une comparaison rapide concernant la modification de la terminologie : 


|**Nouveau terme**  | **Terme précédent**  |**Explication** |
|---------|---------|---------|
|**Azure SQL Managed Instance** | *Instance managée* Azure SQL Database| Azure SQL Managed Instance représente son propre produit au sein de la famille Azure SQL, et non simplement une option de déploiement dans Azure SQL Database. | 
|**Azure SQL Database**|*Base de données unique* Azure SQL Database| Sauf indication contraire, le nom de produit Azure SQL Database comprend à la fois les bases de données uniques et les bases de données déployées sur un pool élastique. |
|**Azure SQL Database**|*Pool élastique* Azure SQL Database| Sauf indication contraire, le nom de produit Azure SQL Database comprend à la fois les bases de données uniques et les bases de données déployées sur un pool élastique.  |
|**Azure SQL Database** |Azure SQL Database | Bien que le terme reste le même, il s’applique à présent uniquement aux déploiements de base de données unique et de pool élastique, et n’inclut pas l’instance managée. |
| **Azure SQL**| N/A | Ce terme fait référence à la famille des produits de moteur de base de données SQL Server qui sont disponibles dans Azure : Azure SQL Database, Azure SQL Managed Instance et SQL Server sur des machines virtuelles Azure. | 

## <a name="features-in-public-preview"></a>Fonctionnalités en préversion publique

### <a name="azure-sql-database"></a>[Azure SQL Database](#tab/single-database)

| Fonctionnalité | Détails |
| ---| --- |
| Récupération de base de données accélérée à l’aide de bases de données uniques et de pools élastiques | Pour plus d’informations, consultez [Récupération de base de données accélérée](../accelerated-database-recovery.md).|
| Découverte et classification des données  |Pour plus d’informations, consultez [Découverte et classification des données pour Azure SQL Database et Azure Synapse Analytics](data-discovery-and-classification-overview.md).|
| Tâches de base de données élastique (préversion) | Pour plus d’informations, consultez [Créer, configurer et gérer des travaux élastiques](elastic-jobs-overview.md). |
| Requêtes élastiques | Pour plus d’informations, consultez [Vue d’ensemble de la requête élastique](elastic-query-overview.md). |
| Transactions élastiques | [Transactions distribuées entre bases de données cloud](elastic-transactions-overview.md). |
| Éditeur de requête dans le portail Azure |Pour plus d’informations, consultez [Utiliser l’éditeur de requête SQL du portail Azure pour se connecter aux données et les interroger](connect-query-portal.md).|
| Services R / machine learning avec des bases de données uniques et des pools élastiques |Pour plus d’informations, consultez [Machine Learning Services dans Azure SQL Database](/sql/advanced-analytics/what-s-new-in-sql-server-machine-learning-services?view=sql-server-2017#machine-learning-services-in-azure-sql-database).|
|SQL Analytics|Pour plus d’informations, consultez [Azure SQL Analytics](../../azure-monitor/insights/azure-sql.md).|
| &nbsp; |

### <a name="azure-sql-managed-instance"></a>[Azure SQL Managed Instance](#tab/managed-instance)

| Fonctionnalité | Détails |
| ---| --- |
| <a href="/azure/azure-sql/database/elastic-transactions-overview">Transactions distribuées</a> | Transactions distribuées sur les instances managées. |
| <a href="/azure/sql-database/sql-database-instance-pools">Pools d’instances</a> | Moyen pratique et économique de migrer des instances SQL plus petites vers le cloud. |
| <a href="/en-gb/sql/t-sql/statements/create-login-transact-sql">Principaux de serveur (connexions) Azure AD au niveau de l’instance</a> | Créez des connexions de niveau instance avec l’instruction <a href="/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current">CREATE LOGIN FROM EXTERNAL PROVIDER</a>. |
| [Réplication transactionnelle](../managed-instance/replication-transactional-overview.md) | Répliquez les modifications de vos tableaux dans d’autres bases de données dans SQL Managed Instance, SQL Database ou SQL Server. Ou mettez à jour vos tableaux lorsque certaines lignes sont modifiées dans d’autres instances de SQL Managed Instance ou SQL Server. Pour plus d’informations, consultez [Configurer la réplication dans Azure SQL Managed Instance](../managed-instance/replication-between-two-instances-configure-tutorial.md). |
| Détection de menaces |Pour plus d’informations, consultez [Configurer la détection des menaces dans Azure SQL Managed Instance](../managed-instance/threat-detection-configure.md).|
| Rétention des sauvegardes à long terme | Pour plus d’informations, consultez [Configurer la conservation des sauvegardes à long terme dans Azure SQL Managed Instance](../managed-instance/long-term-backup-retention-configure.md), actuellement en préversion publique limitée. | 

---

## <a name="new-features"></a>Nouvelles fonctionnalités

### <a name="sql-managed-instance-h2-2019-updates"></a>Mises à jour de SQL Managed Instance (deuxième partie de l’exercice 2019)

- La [Configuration de sous-réseau assistée par le service](https://azure.microsoft.com/updates/service-aided-subnet-configuration-for-managed-instance-in-azure-sql-database-available/) est un moyen sécurisé et pratique de gérer la configuration de sous-réseau ; vous contrôlez le trafic de données tandis que SQL Managed Instance assure le flux ininterrompu du trafic de gestion.
- [Transparent Data Encryption (TDE) avec Bring Your Own Key (BYOK)](https://azure.microsoft.com/updates/general-avilability-transparent-data-encryption-with-customer-managed-keys-for-azure-sql-database-managed-instance/) active le scénario Bring Your Own Key (BYOK) pour la protection des données au repos et permet aux organisations de séparer les tâches de gestion relatives aux clés et aux données.
- Des [groupes de basculement automatique](https://azure.microsoft.com/updates/azure-sql-database-auto-failover-groups-feature-now-available-in-all-regions/) vous permettent de répliquer toutes les bases de données de l’instance principale vers une instance secondaire dans une autre région.
- Les [indicateurs de trace généraux](https://azure.microsoft.com/updates/global-trace-flags-are-now-available-in-azure-sql-database-managed-instance/) vous permettent de configurer le comportement de SQL Managed Instance.

### <a name="sql-managed-instance-h1-2019-updates"></a>Mises à jour de SQL Managed Instance (première partie de l’exercice 2019)

Les fonctionnalités suivantes sont activées dans le modèle de déploiement SQL Managed Instance de la première partie de l’exercice 2019 :
  - Prise en charge des abonnements avec le <a href="/azure/azure-sql/managed-instance/resource-limits">crédit mensuel Azure pour les abonnés Visual Studio</a> et des [limites régionales](../managed-instance/resource-limits.md#regional-resource-limitations) étendues.
  - Prise en charge de <a href="/sharepoint/administration/deploy-azure-sql-managed-instance-with-sharepoint-servers-2016-2019"> SharePoint 2016 et SharePoint 2019 </a> ainsi que de <a href="/business-applications-release-notes/october18/dynamics365-business-central/support-for-azure-sql-database-managed-instance">Dynamics 365 Business Central </a>
  - Créez des instances gérées avec le <a href="/azure/azure-sql/managed-instance/scripts/create-powershell-azure-resource-manager-template">classement au niveau de l’instance</a> et le <a href="https://azure.microsoft.com/updates/managed-instance-time-zone-ga/">fuseau horaire</a>.
  - Les instances managées sont maintenant protégées par un [pare-feu intégré](../managed-instance/management-endpoint-verify-built-in-firewall.md).
  - Configurez des instances SQL Managed Instance pour utiliser des [points de terminaison publics](../managed-instance/public-endpoint-configure.md), [remplacer par proxy](connectivity-architecture.md#connection-policy) une connexion afin d’obtenir de meilleures performances réseau, utiliser <a href="https://aka.ms/four-cores-sql-mi-update"> 4 vCores sur la génération de matériel Gen5</a> ou <a href="/azure/azure-sql/database/automated-backups-overview">configurer la conservation des sauvegardes jusqu’à 35 jours</a> pour une restauration dans le temps. La [rétention des sauvegardes à long terme](long-term-retention-overview.md#sql-managed-instance-support) (jusqu’à 10 ans) est actuellement en version préliminaire publique limitée.  
  - De nouvelles fonctionnalités vous permettent de <a href="https://medium.com/@jocapc/geo-restore-your-databases-on-azure-sql-instances-1451480e90fa">géo-restaurer votre base de données dans un autre centre de données à l’aide de PowerShell</a>, de [renommer la base de données](https://azure.microsoft.com/updates/azure-sql-database-managed-instance-database-rename-is-supported/) et de [supprimer un cluster virtuel](../managed-instance/virtual-cluster-delete.md).
  - Le nouveau [rôle Contributeur d’instance intégré](../../role-based-access-control/built-in-roles.md#sql-managed-instance-contributor) permet la conformité de la séparation des tâches aux principes de sécurité et le respect des normes d’entreprise.
  - SQL Managed Instance est en disponibilité générale dans les régions Azure Government suivantes (US Gov Texas, US Gov Arizona) ainsi que dans les régions Chine Nord 2 et Chine Est 2. Elle est également disponible dans les régions publiques suivantes : Australie Centre, Australie Centre 2, Brésil Sud, France Sud, Émirats arabes unis Centre, Émirats arabes unis Nord, Afrique du Sud Nord, Afrique du Sud Ouest.

## <a name="known-issues"></a>Problèmes connus

|Problème  |Date de la détection  |Statut  |Date de la résolution  |
|---------|---------|---------|---------|
|[Les transactions distribuées peuvent être exécutées après la suppression de Managed Instance du groupe d'approbation de serveurs](#distributed-transactions-can-be-executed-after-removing-managed-instance-from-server-trust-group)|Octobre 2020|Solution de contournement||
|[Les transactions distribuées ne peuvent pas être exécutées après l'opération de mise à l'échelle de Managed Instance](#distributed-transactions-cannot-be-executed-after-managed-instance-scaling-operation)|Octobre 2020|Solution de contournement||
|L’instruction [BULK INSERT](/sql/t-sql/statements/bulk-insert-transact-sql)/[OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql?view=sql-server-ver15) dans Azure SQL, et l’instruction `BACKUP`/`RESTORE` dans Managed Instance ne peuvent pas utiliser Azure AD Manage Identity pour s’authentifier auprès du service Stockage Azure|Septembre 2020|Solution de contournement||
|[Le principal du service ne peut pas accéder à Azure AD et à AKV](#service-principal-cannot-access-azure-ad-and-akv)|Août 2020|Solution de contournement||
|[La restauration d’une sauvegarde manuelle sans CHECKSUM peut échouer](#restoring-manual-backup-without-checksum-might-fail)|Mai 2020|Résolu|Juin 2020|
|[L’agent ne répond plus lors de la modification, la désactivation ou l’activation de travaux existants](#agent-becomes-unresponsive-upon-modifying-disabling-or-enabling-existing-jobs)|Mai 2020|Résolu|Juin 2020|
|[Autorisations sur le groupe de ressources non appliquées à SQL Managed Instance](#permissions-on-resource-group-not-applied-to-sql-managed-instance)|Février 2020|Résolu|Nov. 2020|
|[Limitation du basculement manuel via le portail pour les groupes de basculement](#limitation-of-manual-failover-via-portal-for-failover-groups)|janvier 2020|Solution de contournement||
|[Les rôles SQL Agent requièrent des autorisations d’exécution explicites pour les connexions non-sysadmin](#in-memory-oltp-memory-limits-are-not-applied)|Décembre 2019|Solution de contournement||
|[Les travaux SQL Agent peuvent être interrompus par le redémarrage du processus de l’agent](#sql-agent-jobs-can-be-interrupted-by-agent-process-restart)|Décembre 2019|Résolu|Mars 2020|
|[Les utilisateurs et connexions Azure AD ne sont pas pris en charge dans SSDT](#azure-ad-logins-and-users-are-not-supported-in-ssdt)|Nov 2019|Aucune solution de contournement||
|[Les limites de mémoire OLTP en mémoire ne sont pas appliquées](#in-memory-oltp-memory-limits-are-not-applied)|Octobre 2019|Solution de contournement||
|[Erreur retournée lors de la tentative de suppression d’un fichier qui n’est pas vide](#wrong-error-returned-while-trying-to-remove-a-file-that-is-not-empty)|Octobre 2019|Solution de contournement||
|[Les opérations de changement de niveau de service et de création d’instance sont bloquées par la restauration de base de données en cours](#change-service-tier-and-create-instance-operations-are-blocked-by-ongoing-database-restore)|Septembre 2019|Solution de contournement||
|[Il peut être nécessaire de reconfigurer Resource Governor sur le niveau de service Critique pour l’entreprise après le basculement](#resource-governor-on-business-critical-service-tier-might-need-to-be-reconfigured-after-failover)|Septembre 2019|Solution de contournement||
|[Les boîtes de dialogue Service Broker utilisées entre plusieurs bases de données doivent être réinitialisées après la mise à niveau du niveau de service](#cross-database-service-broker-dialogs-must-be-reinitialized-after-service-tier-upgrade)|août 2019|Solution de contournement||
|[L’emprunt d’identité des types de connexion Azure AD n’est pas pris en charge](#impersonation-of-azure-ad-login-types-is-not-supported)|Juillet 2019|Aucune solution de contournement||
|[Le paramètre @query n’est pas pris en charge dans sp_send_db_mail](#-parameter-not-supported-in-sp_send_db_mail)|Avril 2019|Aucune solution de contournement||
|[La réplication transactionnelle doit être reconfigurée après un basculement géographique](#transactional-replication-must-be-reconfigured-after-geo-failover)|mars 2019|Aucune solution de contournement||
|[Une base de données temporaire est utilisée d’une opération de restauration](#temporary-database-is-used-during-restore-operation)||Solution de contournement||
|[La structure et le contenu de TEMPDB sont recréés](#tempdb-structure-and-content-is-re-created)||Aucune solution de contournement||
|[Dépassement de l’espace de stockage avec des fichiers de base de données de petite taille](#exceeding-storage-space-with-small-database-files)||Solution de contournement||
|[Affichage des valeurs GUID à la place des noms de base de données](#guid-values-shown-instead-of-database-names)||Solution de contournement||
|[Les journaux des erreurs ne sont pas conservés](#error-logs-arent-persisted)||Aucune solution de contournement||
|[L’utilisation de la même étendue de transaction pour deux bases de données appartenant à une même instance n’est pas prise en charge](#transaction-scope-on-two-databases-within-the-same-instance-isnt-supported)||Solution de contournement|Mars 2020|
|[Les modules CLR et les serveurs liés parfois ne parviennent pas à référencer une adresse IP locale](#clr-modules-and-linked-servers-sometimes-cant-reference-a-local-ip-address)||Solution de contournement||
|La cohérence de la base de données est vérifiée à l’aide de DBCC CHECKDB après restauration de la base de données à partir de Stockage Blob Azure.||Résolu|Nov 2019|
|La restauration intégrée de base de données à un point dans le temps du niveau Critique pour l’entreprise vers le niveau Usage général échoue si la base de données source contient des objets OLTP en mémoire.||Résolu|Octobre 2019|
|Fonctionnalité Database Mail avec des serveurs de messagerie externes (non Azure) utilisant une connexion sécurisée||Résolu|Octobre 2019|
|Bases de données autonomes non prises en charge dans SQL Managed Instance||Résolu|août 2019|

### <a name="distributed-transactions-can-be-executed-after-removing-managed-instance-from-server-trust-group"></a>Les transactions distribuées peuvent être exécutées après la suppression de Managed Instance du groupe d'approbation de serveurs

Les [groupes d'approbation de serveurs](../managed-instance/server-trust-group-overview.md) sont utilisés pour établir une relation de confiance entre les instances gérées, condition préalable à l'exécution de [transactions distribuées](./elastic-transactions-overview.md). Après avoir supprimé Managed Instance du groupe d'approbation de serveurs ou supprimé le groupe, vous pourrez peut-être encore exécuter des transactions distribuées. Il existe une solution de contournement que vous pouvez appliquer pour vous assurer que les transactions distribuées sont désactivées : le [basculement manuel initié par l'utilisateur](../managed-instance/user-initiated-failover.md) sur Managed Instance.

### <a name="distributed-transactions-cannot-be-executed-after-managed-instance-scaling-operation"></a>Les transactions distribuées ne peuvent pas être exécutées après l'opération de mise à l'échelle de Managed Instance

Les opérations de mise à l'échelle de Managed Instance qui incluent la modification du niveau de service ou du nombre de vCores réinitialisent les paramètres du groupe d'approbation de serveurs sur le back-end et désactivent les [transactions distribuées](./elastic-transactions-overview.md) en cours d'exécution. Pour contourner ce problème, supprimez le [groupe d'approbation de serveurs](../managed-instance/server-trust-group-overview.md) et créez-en un nouveau sur le portail Azure.

### <a name="bulk-insert-and-backuprestore-statements-cannot-use-managed-identity-to-access-azure-storage"></a>Les instructions BULK INSERT et BACKUP/RESTORE ne peuvent pas utiliser Managed Identity pour accéder au service Stockage Azure

Les instructions BULK INSERT, BACKUP, et RESTORE, et la fonction OPENROWSET, ne peuvent pas utiliser `DATABASE SCOPED CREDENTIAL` avec une identité managée pour l’authentification auprès du service Stockage Azure. Pour contourner ce problème, basculez vers l'authentification par SIGNATURE D'ACCÈS PARTAGÉ. L'exemple suivant ne fonctionne pas sur SQL Azure (Database et Managed Instance) :

```sql
CREATE DATABASE SCOPED CREDENTIAL msi_cred WITH IDENTITY = 'Managed Identity';
GO
CREATE EXTERNAL DATA SOURCE MyAzureBlobStorage
  WITH ( TYPE = BLOB_STORAGE, LOCATION = 'https://****************.blob.core.windows.net/curriculum', CREDENTIAL= msi_cred );
GO
BULK INSERT Sales.Invoices FROM 'inv-2017-12-08.csv' WITH (DATA_SOURCE = 'MyAzureBlobStorage');
```

**Solution de contournement**  : Utilisez la [signature d'accès partagé pour vous authentifier auprès du service de stockage](/sql/t-sql/statements/bulk-insert-transact-sql?view=sql-server-ver15#f-importing-data-from-a-file-in-azure-blob-storage).

### <a name="service-principal-cannot-access-azure-ad-and-akv"></a>Le principal du service ne peut pas accéder à Azure AD et à AKV

Dans certains cas, il existe peut exister un problème avec le principal de service utilisé pour accéder aux services Azure AD et Azure Key Vault (AKV). Par conséquent, ce problème a un impact sur l’utilisation de l’authentification Azure AD et le chiffrement transparent de base de données (TDE) avec SQL Managed Instance. Cela peut être vécu comme un problème de connectivité intermittente ou l’impossibilité d’exécuter des instructions telles que CREATE LOGIN/USER FROM EXTERNAL PROVIDER ou EXECUTE AS LOGIN/USER. La configuration de TDE avec une clé gérée par le client sur un nouveau service Azure SQL Managed Instance peut également ne pas fonctionner dans certaines circonstances.

**Solution de contournement**  : Pour éviter que ce problème ne se produise sur votre service SQL Managed Instance avant d'exécuter des commandes de mise à jour, ou si vous avez déjà rencontré ce problème après l'exécution de commandes de mise à jour, accédez au portail Azure, puis au [panneau d'administration Active Directory](./authentication-aad-configure.md?tabs=azure-powershell#azure-portal) du service SQL Managed Instance. Vérifiez si vous pouvez voir le message d’erreur « Managed Instance a besoin d’un principal du service pour accéder à Azure Active Directory. Cliquez ici pour créer un principal du service ». Si vous rencontrez ce message d’erreur, cliquez dessus, puis suivez les instructions pas à pas fournies jusqu’à ce que cette erreur soit résolue.

### <a name="restoring-manual-backup-without-checksum-might-fail"></a>La restauration d’une sauvegarde manuelle sans CHECKSUM peut échouer

Dans certains cas, la restauration d’une copie de sauvegarde manuelle de bases de données effectuée sur une instance gérée sans CHECKSUM peut échouer. Vous devez alors réessayer de restaurer la copie de sauvegarde jusqu’à ce que l’opération réussisse.

**Solution de contournement**  : Effectuez des copies de sauvegarde manuelles des bases de données sur une instance gérée avec activation de la CHECKSUM.

### <a name="agent-becomes-unresponsive-upon-modifying-disabling-or-enabling-existing-jobs"></a>L’agent ne répond plus lors de la modification, la désactivation ou l’activation de travaux existants

Dans certaines circonstances, la modification d’un travail existant, sa désactivation ou son activation peuvent entraîner le blocage de l’agent. Le problème est automatiquement atténué lors de la détection, ce qui entraîne le redémarrage du processus de l’agent.

### <a name="permissions-on-resource-group-not-applied-to-sql-managed-instance"></a>Autorisations sur le groupe de ressources non appliquées à SQL Managed Instance

Si le rôle Azure Contributeur SQL Managed Instance est appliqué à un groupe de ressources, il n’est pas appliqué à SQL Managed Instance et n’a aucun effet.

**Solution de contournement**  : Configurez le rôle Contributeur SQL Managed Instance pour les utilisateurs au niveau de l’abonnement.

### <a name="limitation-of-manual-failover-via-portal-for-failover-groups"></a>Limitation du basculement manuel via le portail pour les groupes de basculement

Si le groupe de basculement s’étend sur plusieurs instances dans différents abonnements ou groupes de ressources Azure, le basculement manuel ne peut pas être initié à partir de l’instance principale dans le groupe de basculement.

**Solution de contournement**  : Lancez le basculement via le portail à partir de l’instance géographique secondaire.

### <a name="sql-agent-roles-need-explicit-execute-permissions-for-non-sysadmin-logins"></a>Les rôles d’agent SQL requièrent des autorisations d’exécution explicites pour les connexions non-sysadmin

Si des connexions non-sysadmin sont ajoutées à un [rôle de base de données fixe de SQL Agent](/sql/ssms/agent/sql-server-agent-fixed-database-roles), il y a un problème du fait que des autorisations EXECUTE explicites doivent être accordées aux procédures stockées principales pour que ces connexions fonctionnent. Si ce problème survient, le message d’erreur « L’autorisation EXECUTE a été refusée sur l’objet <object_name> (Microsoft SQL Server, erreur : 229) » s’affiche.

**Solution de contournement**  : Une fois que vous avez ajouté des connexions à un rôle de base de données fixe SQL Agent (SQLAgentUserRole, SQLAgentReaderRole, ou SQLAgentOperatorRole) pour chaque connexion ajoutée à ces rôles, exécutez le script T-SQL ci-dessous pour autoriser explicitement l’accès EXÉCUTER aux procédures stockées listées.

```tsql
USE [master]
GO
CREATE USER [login_name] FOR LOGIN [login_name]
GO
GRANT EXECUTE ON master.dbo.xp_sqlagent_enum_jobs TO [login_name]
GRANT EXECUTE ON master.dbo.xp_sqlagent_is_starting TO [login_name]
GRANT EXECUTE ON master.dbo.xp_sqlagent_notify TO [login_name]
```

### <a name="sql-agent-jobs-can-be-interrupted-by-agent-process-restart"></a>Les travaux de l’agent SQL peuvent être interrompus par le redémarrage du processus de l’agent

**(Résolu en mars 2020)** SQL Agent crée une session chaque fois qu’une tâche démarre, ce qui augmente progressivement la consommation de mémoire. Pour éviter d’atteindre la limite de mémoire interne qui bloquerait l’exécution de tâches planifiées, le processus de l’agent sera redémarré une fois que la consommation de mémoire aura atteint le seuil. Cela peut entraîner l’interruption de l’exécution des tâches en cours au moment du redémarrage.

### <a name="in-memory-oltp-memory-limits-are-not-applied"></a>Les limites de mémoire OLTP en mémoire ne sont pas appliquées

Le niveau de service critique pour l'entreprise n’appliquera pas correctement les [limites max de mémoire pour les objets à mémoire optimisée ](../managed-instance/resource-limits.md#in-memory-oltp-available-space) dans certains cas. SQL Managed Instance peut permettre à la charge de travail d’utiliser davantage de mémoire pour les opérations OLTP en mémoire, ce qui peut affecter la disponibilité et la stabilité de l’instance. Les requêtes OLTP en mémoire qui atteignent les limites peuvent ne pas échouer immédiatement. Ce problème sera corrigé bientôt. Les requêtes qui utilisent plus de mémoire OLTP en mémoire échoueront plus tôt si elles atteignent les [limites](../managed-instance/resource-limits.md#in-memory-oltp-available-space).

**Solution de contournement**  : [Surveillez l’utilisation du stockage OLTP en mémoire](../in-memory-oltp-monitor-space.md) à l’aide de [SQL Server Management Studio](/sql/relational-databases/in-memory-oltp/monitor-and-troubleshoot-memory-usage#bkmk_Monitoring) pour vous assurer que la charge de travail n’utilise pas plus que la mémoire disponible. Augmentez les limites de mémoire qui dépendent du nombre de vCores ou optimisez votre charge de travail pour utiliser moins de mémoire.
 
### <a name="wrong-error-returned-while-trying-to-remove-a-file-that-is-not-empty"></a>Erreur retournée lors de la tentative de suppression d’un fichier qui n’est pas vide

SQL Server et SQL Managed Instance [ne permettent pas à l’utilisateur de déposer un fichier qui n’est pas vide](/sql/relational-databases/databases/delete-data-or-log-files-from-a-database#Prerequisites). Si vous tentez de supprimer un fichier de données non vide à l’aide d’une instruction; `ALTER DATABASE REMOVE FILE`l’erreur `Msg 5042 – The file '<file_name>' cannot be removed because it is not empty` n’est pas retournée immédiatement. SQL Managed Instance continue d’essayer de déposer le fichier et l’opération échoue après 30 minutes avec `Internal server error`.

**Solution de contournement**  : Supprimez les contenus du fichier à l’aide de la commande `DBCC SHRINKFILE (N'<file_name>', EMPTYFILE)`. S’il s’agit du seul fichier dans le groupe de fichiers, vous devez supprimer les données de la table ou de la partition associées à ce groupe de fichiers avant de réduire le fichier, et éventuellement charger ces données dans un autre tableau/partition.

### <a name="change-service-tier-and-create-instance-operations-are-blocked-by-ongoing-database-restore"></a>Les opérations de changement de niveau de service et de création d’instance sont bloquées par la restauration de base de données en cours

Les instructions `RESTORE` en cours, le processus de migration Data Migration Service et la limite de restauration dans le temps intégrée bloquent la mise à jour du niveau de service ou le redimensionnement de l’instance existante et la création de nouvelles instances jusqu’à la fin du processus de restauration. 

Le processus de restauration bloquera ces opérations sur les instances managées et les pools d’instances sur le sous-réseau où le processus de restauration est en cours d’exécution. Les instances dans les pools d’instances ne sont pas affectées. Les opérations de création ou de modification du niveau de service n’échouent pas ou expirent. Elles se poursuivront une fois le processus de restauration terminé ou annulé.

**Solution de contournement**  : Attendez la fin du processus de restauration ou annulez-le si l’opération de création ou de mise à jour du niveau de service a une priorité plus élevée.

### <a name="resource-governor-on-business-critical-service-tier-might-need-to-be-reconfigured-after-failover"></a>Il peut être nécessaire de reconfigurer Resource Governor sur le niveau de service Critique pour l’entreprise après le basculement

Il se peut que la fonctionnalité [Resource Governor](/sql/relational-databases/resource-governor/resource-governor) qui permet de limiter les ressources affectées aux charges de travail utilisateur classe erronément une charge de travail utilisateur après un basculement ou une modification du niveau de service apportée par un utilisateur (par exemple, la modification du nombre maximal de vCores ou de la taille maximale de stockage d’instance).

**Solution de contournement**  : Exécutez `ALTER RESOURCE GOVERNOR RECONFIGURE` régulièrement ou dans le cadre du travail du SQL Agent qui exécute la tâche SQL lorsque l’instance démarre si vous utilisez [Resource Governor](/sql/relational-databases/resource-governor/resource-governor).

### <a name="cross-database-service-broker-dialogs-must-be-reinitialized-after-service-tier-upgrade"></a>Les boîtes de dialogue Service Broker utilisées entre plusieurs bases de données doivent être réinitialisées après la mise à niveau du niveau de service

Les boîtes de dialogue Service Broker utilisées entre plusieurs bases de données cessent de transmettre les messages aux services dans d’autres bases de données après un changement du niveau de service. Les messages ne sont *pas perdus* et peuvent être retrouvés dans la file d’attente de l’expéditeur. Toute modification du nombre de vCores ou de la taille de stockage des instances dans SQL Managed Instance entraîne le changement de la valeur `service_broke_guid` affichée dans [sys.databases](/sql/relational-databases/system-catalog-views/sys-databases-transact-sql) pour toutes les bases de données. Tout élément `DIALOG` créé à l’aide de l’instruction [COMMENCER LE DIALOGUE](/sql/t-sql/statements/begin-dialog-conversation-transact-sql) qui référence des répartiteurs Service Brokers dans une autre base de données cesse de remettre les messages au service cible.

**Solution de contournement**  : Arrêtez toutes les activités qui utilisent des conversations de boîtes de dialogue Service Broker entre plusieurs bases de données avant de mettre à jour le niveau de service et réinitialisez-les ensuite. S’il reste des messages non transmis après le changement de niveau de service, consultez les messages en question dans la file d’attente source et renvoyez-les à la file d’attente cible.

### <a name="impersonation-of-azure-ad-login-types-is-not-supported"></a>L’emprunt d’identité des types de connexion Azure AD n’est pas pris en charge

L’emprunt d’identité à l’aide de `EXECUTE AS USER` ou `EXECUTE AS LOGIN` des principaux Azure Active Directory (Azure AD) suivants n’est pas pris en charge :
-   Ajoutez des utilisateurs Azure AD. L’erreur suivante est renvoyée dans ce cas : `15517`.
- Connexions et utilisateurs Azure AD basés sur des applications ou des principaux de service Azure AD. Les erreurs suivantes sont retournées dans ces cas `15517` et `15406`.

### <a name="query-parameter-not-supported-in-sp_send_db_mail"></a>Le paramètre @query n’est pas pris en charge dans sp_send_db_mail

Le paramètre `@query` dans la procédure [sp_send_db_mail](/sql/relational-databases/system-stored-procedures/sp-send-dbmail-transact-sql) ne fonctionne pas.

### <a name="transactional-replication-must-be-reconfigured-after-geo-failover"></a>La réplication transactionnelle doit être reconfigurée après un basculement géographique

Si la réplication transactionnelle est activée sur une base de données dans un groupe de basculement automatique, l’administrateur de SQL Managed Instance doit nettoyer toutes les publications de l’ancien principal et les reconfigurer sur le nouveau principal après un basculement vers une autre région. Pour plus d'informations, voir [Réplication](../managed-instance/transact-sql-tsql-differences-sql-server.md#replication).

### <a name="azure-ad-logins-and-users-are-not-supported-in-ssdt"></a>Les utilisateurs et connexions Azure AD ne sont pas pris en charge dans SSDT

SQL Server Data Tools ne prend pas complètement en charge les utilisateurs et les connexions Azure AD.

### <a name="temporary-database-is-used-during-restore-operation"></a>Une base de données temporaire est utilisée d’une opération de restauration

Lors de la restauration d’une base de données sur SQL Managed Instance, le service de restauration commence par créer une base de données vide du nom souhaité pour allouer le nom sur l’instance. Après un certain temps, cette base de données est supprimée et la restauration de la base de données réelle commence. 

Une valeur GUID aléatoire est allouée temporairement, plutôt qu’un nom, à la base de données en état de *Restauration*. Le nom temporaire est remplacé par le nom spécifié dans l’instruction `RESTORE` une fois le processus de restauration terminé. 

Au cours de la phase initiale, un utilisateur peut accéder à la base de données vide et même créer des tableaux ou charger des données dans celui-ci. Cette base de données temporaire est supprimée lorsque le service de restauration démarre la deuxième phase.

**Solution de contournement**  : N’accédez pas à la base de données que vous restaurez tant que la restauration n’est pas terminée.

### <a name="tempdb-structure-and-content-is-re-created"></a>La structure et le contenu de TEMPDB sont recréés

La base de données `tempdb` est toujours fractionnée en 12 fichiers de données, et cette structure de fichiers ne peut pas être modifiée. La taille maximale par fichier n’est pas modifiable, et il n’est pas possible d’ajouter de nouveaux fichiers dans `tempdb`. `Tempdb` est toujours recréé en tant que base de données vide lorsque l’instance démarre ou bascule, et les modifications apportées dans `tempdb` ne sont pas conservées.

### <a name="exceeding-storage-space-with-small-database-files"></a>Dépassement de l’espace de stockage avec des fichiers de base de données de petite taille

Les instructions `CREATE DATABASE`, `ALTER DATABASE ADD FILE` et `RESTORE DATABASE` risquent d’échouer, car l’instance peut atteindre la limite de Stockage Azure.

Chaque instance SQL Managed Instance à usage général a jusqu’à 35 To de stockage réservé pour l’espace disque Azure Premium. Chaque fichier de base de données est placé sur un disque physique distinct. Les tailles de disque peuvent être de 128 Go, 256 Go, 512 Go, 1 To ou 4 To. L’espace non utilisé sur le disque n’est pas facturé, mais la somme des tailles des disques Premium Azure ne peut pas dépasser 35 To. Dans certains cas, une instance managée qui n’a pas besoin de 8 To au total peut dépasser la limite Azure de 35 To en taille de stockage à cause d’une fragmentation interne.

Par exemple, une instance SQL Managed Instance à usage général peut avoir un fichier d’une taille de 1,2 To placé sur un disque de 4 To. Elle peut également posséder 248 fichiers, d’une taille de 1 Go chacun, qui sont placés sur des disques distincts de 128 Go. Dans cet exemple :

- La taille totale du stockage de disque alloué est de 1 x 4 To + 248 x 128 Go = 35 To.
- L’espace total réservé pour les bases de données sur l’instance est de 1 x 1,2 To + 248 x 1 Go = 1,4 To.

Cet exemple montre que, dans certaines circonstances, du fait d’une répartition spécifique des fichiers, une instance SQL Managed Instance peut atteindre la limite des 35 To réservée pour un disque Azure Premium attaché, sans que vous vous y attendiez.

Dans cet exemple, les bases de données existantes continuent de fonctionner et peuvent croître sans aucun problème du moment que de nouveaux fichiers ne sont pas ajoutés. La création ou la restauration de bases de données est impossible, car il n’y a pas suffisamment d’espace pour les nouveaux lecteurs de disque, même si la taille totale de toutes les bases de données n’atteint pas la limite de taille d’instance. L’erreur qui est retournée dans ce cas n’est pas claire.

Vous pouvez [identifier le nombre de fichiers restants](https://medium.com/azure-sqldb-managed-instance/how-many-files-you-can-create-in-general-purpose-azure-sql-managed-instance-e1c7c32886c1) à l’aide de vues système. Si vous atteignez cette limite, essayez de [vider et de supprimer certains fichiers plus petits au moyen de l’instruction DBCC SHRINKFILE](/sql/t-sql/database-console-commands/dbcc-shrinkfile-transact-sql#d-emptying-a-file), ou basculez vers le [niveau Critique pour l’entreprise, qui ne connaît pas cette limite](../managed-instance/resource-limits.md#service-tier-characteristics).

### <a name="guid-values-shown-instead-of-database-names"></a>Affichage des valeurs GUID à la place des noms de base de données

Plusieurs vues système, compteurs de performances, messages d’erreur, événements XEvent et entrées du journal des erreurs affichent des identificateurs de base de données GUID au lieu d’afficher les noms des bases de données. Ne prenez pas en compte ces identificateurs GUID, car ils vont être remplacés à l’avenir par les noms des bases de données.

**Solution de contournement**  : Utilisez l’affichage sys.databases pour résoudre le nom de la base de données à partir du nom de la base de données physique, spécifié sous forme d’identificateurs de base de données GUID :

```tsql
SELECT name as ActualDatabaseName, physical_database_name as GUIDDatabaseIdentifier 
FROM sys.databases
WHERE database_id > 4
```

### <a name="error-logs-arent-persisted"></a>Les journaux des erreurs ne sont pas conservés

Les journaux des erreurs qui sont disponibles dans SQL Managed Instance ne sont pas persistants, et leur taille n’est pas comprise dans la limite de stockage maximale. Les journaux des erreurs peuvent être automatiquement effacés si un basculement se produit. Il peut y avoir des lacunes dans l’historique du journal des erreurs, car l’instance SQL Managed Instance a été déplacée plusieurs fois sur plusieurs machines virtuelles.

### <a name="transaction-scope-on-two-databases-within-the-same-instance-isnt-supported"></a>L’utilisation de la même étendue de transaction pour deux bases de données appartenant à une même instance n’est pas prise en charge

**(Résolu en mars 2020)** Dans .NET, la classe `TransactionScope` ne fonctionne pas si deux requêtes sont envoyées à deux bases de données appartenant à la même instance et à la même étendue de transaction :

```csharp
using (var scope = new TransactionScope())
{
    using (var conn1 = new SqlConnection("Server=quickstartbmi.neu15011648751ff.database.windows.net;Database=b;User ID=myuser;Password=mypassword;Encrypt=true"))
    {
        conn1.Open();
        SqlCommand cmd1 = conn1.CreateCommand();
        cmd1.CommandText = string.Format("insert into T1 values(1)");
        cmd1.ExecuteNonQuery();
    }

    using (var conn2 = new SqlConnection("Server=quickstartbmi.neu15011648751ff.database.windows.net;Database=b;User ID=myuser;Password=mypassword;Encrypt=true"))
    {
        conn2.Open();
        var cmd2 = conn2.CreateCommand();
        cmd2.CommandText = string.Format("insert into b.dbo.T2 values(2)");        cmd2.ExecuteNonQuery();
    }

    scope.Complete();
}

```

**Solution de contournement (non nécessaire depuis mars 2020)** : servez-vous de [SqlConnection.ChangeDatabase(String)](/dotnet/api/system.data.sqlclient.sqlconnection.changedatabase) pour utiliser une autre base de données dans un contexte de connexion au lieu d’utiliser deux connexions.

### <a name="clr-modules-and-linked-servers-sometimes-cant-reference-a-local-ip-address"></a>Les modules CLR et les serveurs liés parfois ne parviennent pas à référencer une adresse IP locale

Il arrive que des modules CLR placés dans une instance SQL Managed Instance et que des serveurs liés ou des requêtes distribuées référençant une instance active ne parviennent pas à résoudre l’adresse IP d’une instance locale. Cette erreur est un problème temporaire.

**Solution de contournement**  : utilisez des connexions contextuelles dans un module CLR, si possible.

## <a name="updates"></a>Mises à jour

Pour obtenir la liste des mises à jour et améliorations de SQL Database, consultez [Mises à jour du service SQL Database](https://azure.microsoft.com/updates/?product=sql-database).

Pour connaître les mises à jour et améliorations apportées à tous les services Azure, consultez [Mises à jour des services](https://azure.microsoft.com/updates).

## <a name="contribute-to-content"></a>Contribuer au contenu

Pour contribuer à la documentation Azure SQL, consultez le [guide du contributeur Docs](/contribute/).