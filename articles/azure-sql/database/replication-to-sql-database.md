---
title: Réplication d’Azure SQL Server sur Azure SQL Database
description: Vous pouvez configurer une base de données dans Azure SQL Database en tant qu’abonné par émission de données dans une topologie de réplication transactionnelle unidirectionnelle ou de capture instantanée.
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: mathoma
ms.date: 04/28/2020
ms.openlocfilehash: 6ff1d485ab4c0662ae8a9d754ce67b1446b76fcc
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92780952"
---
# <a name="replication-to-azure-sql-database"></a>Réplication sur Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Vous pouvez configurer une base de données SQL Azure en tant qu’abonné par émission de données dans une topologie de réplication de capture instantanée ou transactionnelle unidirectionnelle.

> [!NOTE]
> Cet article décrit l’utilisation de la [réplication transactionnelle](/sql/relational-databases/replication/transactional/transactional-replication) dans Azure SQL Database. Elle n’est pas liée à la [géo-réplication active](./active-geo-replication-overview.md), une fonctionnalité Azure SQL Database qui vous permet de créer des réplicas lisibles complets de bases de données individuelles.

## <a name="supported-configurations"></a>Configurations prises en charge
  
- Azure SQL Database peut uniquement être l’abonné par émission de données d’un serveur de publication et d’une base de données du serveur de distribution SQL Server.  
- L’instance SQL Server agissant en tant qu’éditeur ou distributeur peut être une instance de [SQL Server s’exécutant localement](https://www.microsoft.com/sql-server/sql-server-downloads), une [Azure SQL Managed Instance](../managed-instance/instance-create-quickstart.md) ou une instance de [SQL Server s’exécutant sur une machine virtuelle Azure dans le cloud](../virtual-machines/windows/sql-vm-create-portal-quickstart.md). 
- La base de données de distribution et les agents de réplication ne peuvent pas être placés sur une base de données dans Azure SQL Database.  
- Les réplications par [capture instantanée](/sql/relational-databases/replication/snapshot-replication) et les réplications [transactionnelles unidirectionnelles](/sql/relational-databases/replication/transactional/transactional-replication) sont prises en charge. Les réplications transactionnelles pair à pair et les réplications de fusion ne sont pas prises en charge.

### <a name="versions"></a>Versions  

Pour répliquer correctement vers une base de données dans Azure SQL Database, les éditeurs et distributeurs SQL Server doivent utiliser (au moins) l’une des versions suivantes :

La publication dans une base de données SQL Azure à partir d’une base de données SQL Server est prise en charge par les versions suivantes de SQL Server :

- SQL Server 2016 et versions ultérieures
- SQL Server 2014 [RTM CU10 (12.0.4427.24)](https://support.microsoft.com/help/3094220/cumulative-update-10-for-sql-server-2014) ou [SP1 CU3 (12.0.2556.4)](https://support.microsoft.com/help/3094221/cumulative-update-3-for-sql-server-2014-service-pack-1)
- SQL Server 2012 [SP2 CU8 (11.0.5634.1)](https://support.microsoft.com/help/3082561/cumulative-update-8-for-sql-server-2012-sp2) ou [SP3 (11.0.6020.0)](https://www.microsoft.com/download/details.aspx?id=49996)

> [!NOTE]
> Si vous tentez de configurer la réplication avec une version non prise en charge, les erreurs suivantes peuvent se produire : MSSQL_REPL20084 (Le processus n’a pas pu se connecter à l’abonné) et MSSQL_REPL40532 (Impossible d’ouvrir le serveur \<name> demandé par la connexion. La connexion a échoué).  

Pour bénéficier de toutes les fonctionnalités Azure SQL Database, vous devez utiliser les dernières versions de [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) et de [SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt).  

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

  
## <a name="remarks"></a>Notes

- Seuls les abonnements par push à Azure SQL Database sont pris en charge.  
- La réplication peut être configurée à l’aide de [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) ou en exécutant des instructions Transact-SQL sur le serveur de publication. Vous ne pouvez pas configurer la réplication dans le portail Azure.  
- La réplication peut uniquement utiliser des connexions d’authentification SQL Server pour se connecter à Azure SQL Database.
- Les tables répliquées doivent avoir une clé primaire.  
- Vous devez disposer d’un abonnement Azure existant.  
- L’abonné Azure SQL Database peut se trouver dans n’importe quelle région.  
- Une même publication sur SQL Server peut prendre en charge à la fois les abonnés Azure SQL Database et SQL Server (localement et sur une machine virtuelle Azure).  
- La gestion, la supervision et la résolution des problèmes des réplications doivent être effectuées à partir de SQL Server plutôt que d’Azure SQL Database.  
- Seul `@subscriber_type = 0` est pris en charge dans **sp_addsubscription** pour SQL Database.  
- Azure SQL Database ne prend pas en charge les réplications d’égal à égal, bidirectionnelles, immédiates ou pouvant être mises à jour.

## <a name="replication-architecture"></a>Architecture de réplication  

![Diagramme illustrant l’architecture de réplication avec Azure SQL Database, qui contient plusieurs clusters d’abonnés dans différentes régions, et des machines virtuelles Azure locales, qui contiennent un serveur de publication, un exécutable Logread et des exécutables de serveurs de distribution qui se connectent à des clusters distants](./media/replication-to-sql-database/replication-to-sql-database.png)  

## <a name="scenarios"></a>Scénarios  

### <a name="typical-replication-scenario"></a>Scénario de réplication classique  

1. Créez une publication de réplication transactionnelle sur une base de données SQL Server.  
2. Sur SQL Server, servez-vous de **l’Assistant Nouvel abonnement** ou utilisez des instructions Transact-SQL pour envoyer l’abonnement vers Azure SQL Database.  
3. Avec des bases de données uniques et mises en pool dans Azure SQL Database, le jeu de données initial est un instantané créé par l’Agent d’instantané, puis distribué et appliqué par l’Agent de distribution. Avec un serveur de publication SQL Database Managed Instance, vous pouvez également utiliser une sauvegarde de la base de données pour alimenter la base de données des abonnés Azure SQL Database.

### <a name="data-migration-scenario"></a>Scénario de migration de données  

1. Utilisez la réplication transactionnelle pour répliquer les données d’une base de données SQL Server vers Azure SQL Database.  
2. Redirigez les applications clientes ou de couche intermédiaire pour mettre à jour la copie de la base de données.  
3. Arrêtez la mise à jour de la version SQL Server de la table, puis supprimez la publication.  

## <a name="limitations"></a>Limites

Les options suivantes ne sont pas prises en charge pour les abonnements Azure SQL Database :

- Copier les associations de groupes de fichiers  
- Copier les schémas de partition de table  
- Copier les schémas de partition d’index  
- Copier les statistiques définies par l’utilisateur  
- Copier les liaisons par défaut  
- Copier les liaisons de règle  
- Copier les index de recherche en texte intégral  
- Copier le schéma XSD XML  
- Copier les index XML  
- Copier les autorisations  
- Copier les index spatiaux  
- Copier les index filtrés  
- Copier l’attribut de compression de données  
- Copier l’attribut de colonne éparse  
- Convertir filestream en types de données MAX  
- Convertir hierarchyId en types de données MAX  
- Convertir le type spatial en types de données MAX  
- Copier les propriétés étendues  

### <a name="limitations-to-be-determined"></a>Limitations à déterminer

- Copier le classement  
- Exécution de la procédure stockée dans une transaction sérialisée  

## <a name="examples"></a>Exemples

Créez une publication et un abonnement par émission de données. Pour plus d'informations, consultez les pages suivantes :
  
- [Créer une publication](/sql/relational-databases/replication/publish/create-a-publication)
- [Créez un abonnement par émission de données](/sql/relational-databases/replication/create-a-push-subscription/) en utilisant le nom du serveur en tant qu’abonné (par exemple **N'azuresqldbdns.database.windows.net'** ) et le nom de la base de données SQL Azure comme base de données de destination (par exemple **AdventureWorks** ).  

## <a name="see-also"></a>Voir aussi  

- [Réplication transactionnelle](../managed-instance/replication-transactional-overview.md)
- [Créer une publication](/sql/relational-databases/replication/publish/create-a-publication)
- [Créer un abonnement par émission de données](/sql/relational-databases/replication/create-a-push-subscription/)
- [Types de réplication](/sql/relational-databases/replication/types-of-replication)
- [Surveillance (réplication)](/sql/relational-databases/replication/monitor/monitoring-replication)
- [Initialiser un abonnement](/sql/relational-databases/replication/initialize-a-subscription)