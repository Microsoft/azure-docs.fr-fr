---
title: Réplication transactionnelle
titleSuffix: Azure SQL Managed Instance
description: Découvrez comment utiliser la réplication transactionnelle SQL Server avec Azure SQL Managed Instance (préversion).
services: sql-database
ms.service: sql-managed-instance
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: MashaMSFT
ms.author: mathoma
ms.reviewer: sstein
ms.date: 04/20/2020
ms.openlocfilehash: 76bb4ffb4ebeb01baf8236d6be84c900b23ffbc0
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790812"
---
# <a name="transactional-replication-with-azure-sql-managed-instance-preview"></a>Réplication transactionnelle avec Azure SQL Managed Instance (préversion)
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

La réplication transactionnelle est une fonctionnalité d’Azure SQL Managed Instance et SQL Server qui vous permet de répliquer les données d’une table dans Azure SQL Managed Instance ou une instance de SQL Server dans des tables placées dans des bases de données distantes. Cette fonctionnalité vous permet de synchroniser plusieurs tables dans différentes bases de données. 

La réplication transactionnelle est actuellement disponible en préversion publique pour SQL Managed Instance. 

## <a name="overview"></a>Vue d’ensemble

Vous pouvez utiliser la réplication transactionnelle pour transmettre les modifications apportées à une instance managée Azure SQL à :

- Une base de données SQL Server, locale ou sur une machine virtuelle Azure.
- Une base de données dans Azure SQL Database
- Une base de données d’instance dans Azure SQL Managed Instance

  > [!NOTE]
  > Pour bénéficier de toutes les fonctionnalités d’Azure SQL Managed Instance, vous devez utiliser les dernières versions de [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) et de [SQL Server Data Tools (SSDT)](/sql/ssdt/download-sql-server-data-tools-ssdt).

### <a name="components"></a>Components

Les principaux composants de la réplication transactionnelle sont la **base de données du serveur de publication** , la **base de données du serveur de distribution** et l’ **abonné** , comme indiqué dans l’image suivante :  

![réplication avec SQL Database](./media/replication-transactional-overview/replication-to-sql-database.png)

| Role | Azure SQL Database | Azure SQL Managed Instance |
| :----| :------------- | :--------------- |
| **Publisher** | Non | Oui |
| **Serveur de distribution** | Non | Oui|
| **Abonné de type pull** | Non | Oui|
| **Abonné de type push**| Oui | Oui|
| &nbsp; | &nbsp; | &nbsp; |

La **base de données du serveur de publication** publie les changements apportés à certaines tables (articles) en envoyant les mises à jour à la base de données du serveur de distribution. La base de données du serveur de publication peut être une instance managée Azure SQL ou une instance de SQL Server.

La **base de données du serveur de distribution** collecte les changements apportés aux articles à partir d’une base de données du serveur de publication et les distribue aux Abonnés. La base de données du serveur de distribution peut être une instance managée Azure SQL ou une instance de SQL Server (n’importe quelle version, tant qu’elle est supérieure ou égale à la version de la base de données du serveur de publication).

L’ **abonné** reçoit les modifications apportées sur la base de données du serveur de publication. Une instance de SQL Server et Azure SQL Managed Instance peuvent être des abonnés d’envoi (push) et d’extraction (pull), bien qu’un abonnement par extraction ne soit pas pris en charge quand la base de données du serveur de distribution est une instance managée Azure SQL et que l’abonné n’en est pas une. Une base de données dans Azure SQL Database peut uniquement être un abonné par envoi.

Azure SQL Managed Instance peut prendre en charge le fait d’être un abonné des versions suivantes de SQL Server :

- SQL Server 2016 et versions ultérieures
- SQL Server 2014 [RTM CU10 (12.0.4427.24)](https://support.microsoft.com/help/3094220/cumulative-update-10-for-sql-server-2014) ou [SP1 CU3 (12.0.2556.4)](https://support.microsoft.com/help/3094221/cumulative-update-3-for-sql-server-2014-service-pack-1)
- SQL Server 2012 [SP2 CU8 (11.0.5634.1)](https://support.microsoft.com/help/3082561/cumulative-update-8-for-sql-server-2012-sp2) ou [SP3 (11.0.6020.0)](https://www.microsoft.com/download/details.aspx?id=49996)

   > [!NOTE]
   >
   > - Pour les autres versions de SQL Server qui ne prennent pas en charge la publication sur des objets dans Azure, il est possible d’utiliser la méthode de [republication des données](/sql/relational-databases/replication/republish-data) pour déplacer des données vers des versions plus récentes de SQL Server.
   > - Si vous configurez la réplication avec une version antérieure, les erreurs suivantes peuvent se produire : MSSQL_REPL20084 (le processus n’a pas pu se connecter à l’Abonné) et MSSQ_REPL40532 (impossible d’ouvrir le serveur \<name> demandé par la connexion. La connexion a échoué).

### <a name="types-of-replication"></a>Types de réplication

Il existe différents [types de réplications](/sql/relational-databases/replication/types-of-replication) :

| Réplication | Azure SQL Database | Azure SQL Managed Instance |
| :----| :------------- | :--------------- |
| [**Transactionnelle standard**](/sql/relational-databases/replication/transactional/transactional-replication) | Oui (uniquement en tant qu’Abonné) | Oui |
| [**Capture instantanée**](/sql/relational-databases/replication/snapshot-replication) | Oui (uniquement en tant qu’Abonné) | Oui|
| [**Réplication de fusion**](/sql/relational-databases/replication/merge/merge-replication) | Non | Non|
| [**Pair à pair**](/sql/relational-databases/replication/transactional/peer-to-peer-transactional-replication) | Non | Non|
| [**Bidirectionnelle**](/sql/relational-databases/replication/transactional/bidirectional-transactional-replication) | Non | Oui|
| [**Abonnements pouvant être mis à jour**](/sql/relational-databases/replication/transactional/updatable-subscriptions-for-transactional-replication) | Non | Non|
| &nbsp; | &nbsp; | &nbsp; |

### <a name="supportability-matrix"></a>Matrice de prise en charge

  La matrice de prise en charge de la réplication transactionnelle pour Azure SQL Managed Instance est identique à celle de SQL Server.
  
| **Publisher**   | **Serveur de distribution** | **Abonné** |
| :------------   | :-------------- | :------------- |
| SQL Server 2019 | SQL Server 2019 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/>  |
| SQL Server 2017 | SQL Server 2019 <br/>SQL Server 2017 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 |
| SQL Server 2016 | SQL Server 2019 <br/>SQL Server 2017 <br/> SQL Server 2016 | SQL Server 2019 <br/> SQL Server 2017 <br/>SQL Server 2016 <br/> SQL Server 2014 <br/> SQL Server 2012 |
| SQL Server 2014 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/>| SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/> SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 |
| SQL Server 2012 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/>SQL Server 2012 <br/> | SQL Server 2016 <br/> SQL Server 2014 <br/> SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 |
| SQL Server 2008 R2 <br/> SQL Server 2008 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/>SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 |  SQL Server 2014 <br/> SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 <br/>  |
| &nbsp; | &nbsp; | &nbsp; |

## <a name="when-to-use"></a>Quand l’utiliser

La réplication transactionnelle est utile dans les scénarios suivants :

- Publier les changements apportés dans une ou plusieurs tables d’une base de données et les distribuer à une ou plusieurs bases de données dans une instance de SQL Server ou Azure SQL Database abonnées aux changements.
- Maintenir plusieurs bases de données distribuées dans un état synchronisé.
- Migrer des bases de données d’une instance de SQL Server ou d’une instance managée Azure SQL vers une autre base de données en publiant les modifications en continu.

### <a name="compare-data-sync-with-transactional-replication"></a>Comparaison entre synchronisation des données et réplication transactionnelle

| Category | Synchronisation des données | Réplication transactionnelle |
|---|---|---|
| Avantages | - Support actif/actif<br/>- Synchronisation bidirectionnelle entre la base de données Azure SQL et locale | - Latence réduite<br/>- Cohérence transactionnelle<br/>- Réutilisation de la topologie existante après la migration |
| Inconvénients | - Latence de 5 minutes ou plus<br/>- Pas de cohérence transactionnelle<br/>- Impact plus important sur les performances | - Impossible de publier à partir d’Azure SQL Database <br/>- Coût de maintenance élevé |

## <a name="common-configurations"></a>Configurations courantes

En règle générale, le serveur de publication et le serveur de distribution doivent se trouver dans le cloud ou en local. Les configurations suivantes sont prises en charge :

### <a name="publisher-with-local-distributor-on-sql-managed-instance"></a>Base de données du serveur de publication avec une base de données du serveur de distribution locale sur SQL Managed Instance

![Instance unique faisant office de serveur de publication et de serveur de distribution](./media/replication-transactional-overview/01-single-instance-asdbmi-pubdist.png)

La base de données du serveur de publication et la base de données du serveur de distribution sont configurées dans une même instance managée SQL, et les changements sont distribués à une autre instance managée de SQL, base de données SQL ou instance de SQL Server.

### <a name="publisher-with-remote-distributor-on-sql-managed-instance"></a>Base de données du serveur de publication avec base de données du serveur de distribution distante sur SQL Managed Instance

Dans cette configuration, une instance managée publie les changements dans une base de données du serveur de distribution placée sur une autre instance managée SQL qui peut servir plusieurs instances managées SQL sources et distribuer les changements à une ou plusieurs cibles sur Azure SQL Database, Azure SQL Managed Instance ou SQL Server.

![Instances séparées pour le serveur de publication et le serveur de distribution](./media/replication-transactional-overview/02-separate-instances-asdbmi-pubdist.png)

Le serveur de publication et le serveur de distribution sont configurés sur deux instances managées. Il existe des contraintes avec cette configuration :

- Les deux instances managées sont sur le même réseau virtuel.
- Les deux instances managées se trouvent au même emplacement.

### <a name="on-premises-publisherdistributor-with-remote-subscriber"></a>Base de données du serveur de publication/distribution locale avec abonné distant

![Azure SQL Database en tant qu’abonné](./media/replication-transactional-overview/03-azure-sql-db-subscriber.png)

Dans cette configuration, une base de données dans Azure SQL Database ou Azure SQL Managed Instance est un abonné. Cette configuration prend en charge la migration de données locales vers Azure. Si un abonné est une base de données dans Azure SQL Database, il doit être en mode push.  

## <a name="requirements"></a>Spécifications

- Utiliser l’authentification SQL pour la connectivité entre les participants de la réplication
- Utiliser un partage de compte Stockage Azure pour le répertoire de travail utilisé par la réplication
- Ouvrir le port TCP 445 sortant dans les règles de sécurité de sous-réseau pour accéder au partage de fichiers Azure
- Ouvrir le port TCP 1433 sortant quand la base de données du serveur de publication/distribution est une instance managée SQL et que l’abonné n’en est pas une. Il vous faudra peut-être aussi modifier la règle de sécurité de trafic sortant du groupe de sécurité réseau de l’instance managée SQL pour `allow_linkedserver_outbound` pour l’ **Étiquette Service de destination** du port 1433 (en remplaçant `virtualnetwork` par `internet`)
- Placer les bases de données du serveur de publication et de distribution dans le cloud, ou toutes les deux localement
- Configurer le peering VPN entre les réseaux virtuels des participants de réplication si les réseaux virtuels sont différents

> [!NOTE]
> Vous pouvez rencontrer l’erreur 53 lors de la connexion à un fichier Stockage Azure si le port 445 du groupe de sécurité réseau sortant est bloqué quand la base de données du serveur de distribution est une base de données Azure SQL Managed Instance et que l’abonné est local. Pour résoudre ce problème, [mettez à jour le NSG vNet](../../storage/files/storage-troubleshoot-windows-file-connection-problems.md).

## <a name="with-failover-groups"></a>Avec les groupes de basculement

[La géoréplication active](../database/active-geo-replication-overview.md) n’est pas prise en charge avec une instance managée SQL utilisant la réplication transactionnelle. Au lieu de la géoréplication active, utilisez des [groupes de basculement automatique](../database/auto-failover-group-overview.md), mais notez que la publication doit être [supprimée manuellement](transact-sql-tsql-differences-sql-server.md#replication) de l’instance managée principale et recréée sur l’instance managée SQL secondaire après le basculement.

Si la géoréplication est activée sur une instance managée SQL assumant le rôle de **base de données du serveur de publication** ou de **base de données du serveur de distribution** dans un [groupe de basculement](../database/auto-failover-group-overview.md), l’administrateur de l’instance managée SQL doit nettoyer toutes les publications de l’ancienne instance principale et les reconfigurer sur la nouvelle instance principale après un basculement. Les activités suivantes sont nécessaires dans ce scénario :

1. Arrêtez tous les travaux de réplication en cours d’exécution sur la base de données, le cas échéant.
1. Supprimez les métadonnées d’abonnement du serveur de publication en exécutant le script suivant sur la base de données du serveur de publication :

   ```sql
   EXEC sp_dropsubscription @publication='<name of publication>', @article='all',@subscriber='<name of subscriber>'
   ```

1. Supprimez les métadonnées d’abonnement de l’abonné. Exécutez le script suivant sur la base de données d’abonnement sur l’instance managée SQL de l’abonné :

   ```sql
   EXEC sp_subscription_cleanup
      @publisher = N'<full DNS of publisher, e.g. example.ac2d23028af5.database.windows.net>',
      @publisher_db = N'<publisher database>',
      @publication = N'<name of publication>';
   ```

1. Supprimez définitivement tous les objets de réplication du serveur de publication en exécutant le script suivant dans la base de données publiée :

   ```sql
   EXEC sp_removedbreplication
   ```

1. Supprimez définitivement l’ancienne base de données du serveur de distribution de l’instance managée SQL principale d’origine (en cas de basculement vers une ancienne instance principale qui utilisait une base de données du serveur de distribution). Exécutez le script suivant sur la base de données MASTER sur l’ancienne instance managée SQL de la base de données du serveur de distribution :

   ```sql
   EXEC sp_dropdistributor 1,1
   ```

Si la géoréplication est activée sur une instance de l’ **abonné** dans un groupe de basculement, la publication doit être configurée pour se connecter au point de terminaison de l’écouteur de groupe de basculement pour l’instance managée de l’abonné. En cas de basculement, l’action suivante de l’administrateur de l’instance managée dépend du type de basculement qui s’est produit :

- Pour un basculement sans perte de données, la réplication continue de fonctionner après le basculement.
- Pour un basculement avec perte de données, la réplication fonctionne également. Elle répliquera à nouveau les modifications perdues.
- Pour un basculement avec perte de données, mais où la perte de données est en dehors de la période de conservation de la base de données de distribution, l’administrateur de l’instance managée SQL doit réinitialiser la base de données d’abonnement.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la configuration de la réplication transactionnelle, consultez les tutoriels suivants :

- [Configurer la réplication entre une base de données du serveur de publication SQL Managed Instance et un abonné](../managed-instance/replication-between-two-instances-configure-tutorial.md)
- [Configurer la réplication entre une base de données du serveur de publication SQL Managed Instance, une base de données du serveur de distribution SQL Managed Instance et un abonné SQL Server](../managed-instance/replication-two-instances-and-sql-server-configure-tutorial.md)
- [Créer une publication](/sql/relational-databases/replication/publish/create-a-publication).
- [Créer un abonnement par émission de données](/sql/relational-databases/replication/create-a-push-subscription) en utilisant le nom du serveur en tant qu’abonné (par exemple `N'azuresqldbdns.database.windows.net`) et le nom de la base de données Azure SQL Database comme base de données de destination (par exemple **Adventureworks** )

## <a name="see-also"></a>Voir aussi  

- [Réplication avec une instance managée SQL et un groupe de basculement](transact-sql-tsql-differences-sql-server.md#replication)
- [Réplication sur SQL Database](../database/replication-to-sql-database.md)
- [Réplication sur une instance managée](../managed-instance/replication-between-two-instances-configure-tutorial.md)
- [Créer une publication](/sql/relational-databases/replication/publish/create-a-publication)
- [Créer un abonnement par émission de données](/sql/relational-databases/replication/create-a-push-subscription/)
- [Types de réplication](/sql/relational-databases/replication/types-of-replication)
- [Surveillance (réplication)](/sql/relational-databases/replication/monitor/monitoring-replication)
- [Initialiser un abonnement](/sql/relational-databases/replication/initialize-a-subscription)