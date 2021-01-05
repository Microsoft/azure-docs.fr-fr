---
title: Transactions distribuées entre bases de données cloud (préversion)
description: Vue d’ensemble des transactions de bases de données élastiques avec Azure SQL Database et Azure SQL Managed Instance.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/12/2019
ms.openlocfilehash: 5504b9bc87f78682ff584006255d4e75e5e69fa7
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92793345"
---
# <a name="distributed-transactions-across-cloud-databases-preview"></a>Transactions distribuées entre bases de données cloud (préversion)
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Les transactions de bases de données élastiques pour Azure SQL Database et Azure SQL Managed Instance vous permettent d’exécuter des transactions qui s’étendent sur plusieurs bases de données. Ces transactions sont disponibles pour les applications .NET utilisant ADO.NET et s’intègrent à une expérience de programmation familière basée sur les classes [System.Transaction](/dotnet/api/system.transactions). Pour obtenir la bibliothèque, consultez [.NET Framework 4.6.1 (programme d’installation web)](https://www.microsoft.com/download/details.aspx?id=49981).
En outre, pour Managed Instance, les transactions distribuées sont disponibles en [Transact-SQL](/sql/t-sql/language-elements/begin-distributed-transaction-transact-sql).

Localement, un tel scénario nécessite généralement l’exécution de Microsoft Distributed Transaction Coordinator (MSDTC). Étant donné que MSDTC n’est pas disponible pour une application PaaS (Platform-as-a-Service) dans Azure, la fonctionnalité permettant de coordonner les transactions distribuées a maintenant été intégrée directement dans SQL Database ou Managed Instance. Les applications peuvent se connecter à n’importe quelle base de données pour lancer des transactions distribuées, après quoi l’une des base de données ou l’un des serveurs coordonnera en toute transparence les transactions distribuées, comme illustré dans la figure suivante.

Dans ce document, les termes « transactions distribuées » et « transactions de bases de données élastique » sont considérés comme des synonymes et sont utilisés de manière interchangeable.

  ![Transactions distribuées avec Azure SQL Database utilisant les transactions de bases de données élastiques ][1]

## <a name="common-scenarios"></a>Scénarios courants

Les transactions de bases de données élastiques permettent aux applications d’apporter des modifications atomiques aux données stockées dans plusieurs bases de données différentes. La version d’évaluation se concentre sur les expériences de développement côté client en C# et .NET. Une expérience côté serveur (code écrit dans des procédures stockées ou des scripts côté serveur) en [Transact-SQL](/sql/t-sql/language-elements/begin-distributed-transaction-transact-sql) est disponible uniquement pour Managed Instance.
> [!IMPORTANT]
> Dans la préversion, l’exécution de transactions de bases de données élastiques entre Azure SQL Database et Azure SQL Managed Instance n’est pas prise en charge pour le moment. Les transactions de bases de données élastiques ne peuvent s’étendre qu’à un ensemble de bases de données SQL ou à un ensemble d’instances gérées.

Les transactions de bases de données élastiques ciblent les scénarios suivants :

* Applications de bases de données multiples dans Azure : Avec ce scénario, les données sont partitionnées verticalement sur plusieurs bases de données dans SQL Database ou Managed Instance, de telle sorte que différents types de données résident dans différentes bases de données. Certaines opérations nécessitent des modifications des données, qui sont conservées dans au moins deux bases de données. L’application utilise des transactions de bases de données élastiques pour coordonner les modifications entre les bases de données et en garantir l’atomicité.
* Applications de bases de données partitionnées dans Azure : Avec ce scénario, la couche Données utilise la [bibliothèque de client de bases de données élastiques](elastic-database-client-library.md) ou l’autopartitionnement pour partitionner horizontalement les données entre plusieurs bases de données dans SQL Database ou Managed Instance. L’un des scénarios d’utilisation les plus importants concerne la nécessité d’effectuer des modifications atomiques pour une application mutualisée et partitionnée lorsque les modifications s’étendent à plusieurs locataires. Imaginez par exemple un transfert d’un locataire à un autre, tous deux résidant sur des bases de données différentes. Deuxième cas de figure : un partitionnement affiné pour tenir compte des besoins de capacité pour un locataire important qui, à son tour, implique généralement que certaines opérations atomiques doivent s’étendre à plusieurs bases de données utilisées pour le même locataire. Citons un troisième cas, celui des mises à jour atomiques aux données de référence répliquées sur différentes bases de données. La version préliminaire permet maintenant de coordonner les modifications atomiques, les transactions et les opérations le long de ces lignes entre plusieurs bases de données.
  Les transactions de bases de données élastiques utilisent une validation en deux phases pour garantir l’atomicité des transactions entre les différentes bases de données. Il est adapté aux transactions qui impliquent moins de 100 bases de données à la fois dans une même transaction. Ces limites ne sont pas appliquées, mais il faut s’attendre, pour les transactions de bases de données élastiques, à des performances et des taux de réussite inférieurs au-delà de ces limites.

## <a name="installation-and-migration"></a>Installation et migration

Les capacités des transactions de bases de données élastiques sont fournies par le biais des mises à jour des bibliothèques .NET System.Data.dll et System.Transactions.dll. Les DLL garantissent que la validation en deux phases sera utilisée lorsque cela est nécessaire pour garantir l’atomicité. Pour commencer à développer des applications à l’aide de transactions de bases de données élastiques, installez le [.NET Framework 4.6.1](https://www.microsoft.com/download/details.aspx?id=49981) ou une version ultérieure. Lorsque vous exécutez une version antérieure du .NET Framework, les transactions ne peuvent pas être promues vers une transaction distribuée, ce qui génère une exception.

Après l’installation, vous pouvez utiliser les API de transaction distribuée dans System.Transactions avec des connexions à SQL Database et Managed Instance. Si vous avez déjà des applications MSDTC qui utilisent ces API, reconstruisez vos applications existantes pour .NET 4.6 après l’installation du .NET Framework 4.6.1. Si vos projets ciblent .NET 4.6, ils utiliseront automatiquement les DLL mises à jour à partir de la nouvelle version du .NET Framework. Dès lors, les appels d’API de transaction distribuée exécutés parallèlement aux connexions à SQL Database ou Managed Instance aboutiront.

N’oubliez pas que les transactions de bases de données élastiques ne nécessitent pas d’installer MSDTC. Elles sont en fait gérées directement par et depuis le service. Cela simplifie grandement les scénarios cloud, dans la mesure où il n’est pas nécessaire de déployer MSDTC pour utiliser des transactions distribuées avec SQL Database ou Managed Instance. La section 4 explique en détail comment déployer des transactions de bases de données élastiques et le framework .NET requis avec vos applications cloud dans Azure.

## <a name="net-installation-for-azure-cloud-services"></a>Installation de .NET pour Azure Cloud Services

Azure fournit plusieurs offres pour héberger des applications .NET. Une comparaison des différentes offres est disponible dans la section [Comparaison entre Azure App Service, Cloud Services et Virtual Machines](/azure/architecture/guide/technology-choices/compute-decision-tree). Si le SE invité de l'offre est antérieur à la version .NET 4.6.1 requise pour les transactions élastiques, vous devez mettre à niveau le SE invité vers la version 4.6.1.

Pour Azure App Services, les mises à niveau du système d’exploitation invité ne sont actuellement pas prises en charge. Pour Azure Virtual Machines, connectez-vous à la machine virtuelle et exécutez le programme d'installation du .NET Framework le plus récent. Pour Azure Cloud Services, vous devez inclure l'installation d'une version plus récente de .NET dans les tâches de démarrage de votre déploiement. Les concepts et les étapes sont documentés dans [Installer .NET sur un rôle de service cloud](../../cloud-services/cloud-services-dotnet-install-dotnet.md).  

Notez que le programme d'installation pour .NET 4.6.1 peut nécessiter plus de stockage temporaire pendant le processus de démarrage des services cloud Azure que la version .NET 4.6. Pour garantir la réussite de l’installation, vous devez augmenter le stockage temporaire de votre service cloud Azure dans votre fichier ServiceDefinition.csdef dans la section LocalResources et les paramètres d'environnement de votre tâche de démarrage, comme illustré dans l'exemple suivant :

```xml
    <LocalResources>
    ...
        <LocalStorage name="TEMP" sizeInMB="5000" cleanOnRoleRecycle="false" />
        <LocalStorage name="TMP" sizeInMB="5000" cleanOnRoleRecycle="false" />
    </LocalResources>
    <Startup>
        <Task commandLine="install.cmd" executionContext="elevated" taskType="simple">
            <Environment>
        ...
                <Variable name="TEMP">
                    <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='TEMP']/@path" />
                </Variable>
                <Variable name="TMP">
                    <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='TMP']/@path" />
                </Variable>
            </Environment>
        </Task>
    </Startup>
```

## <a name="net-development-experience"></a>Expérience de développement dans .NET

### <a name="multi-database-applications"></a>Applications de bases de données multiples

L’exemple de code suivant utilise l’expérience de programmation familière avec System.Transactions de .NET. La classe TransactionScope établit une transaction ambiante dans .NET. (Une « transaction ambiante » est une transaction qui réside dans le thread actuel). Toutes les connexions ouvertes dans TransactionScope sont impliquées dans la transaction. Si des bases de données différentes sont impliquées, la transaction est automatiquement élevée à une transaction distribuée. Le résultat de la transaction est contrôlé en définissant l’étendue de l’exécution pour indiquer une validation.

```csharp
    using (var scope = new TransactionScope())
    {
        using (var conn1 = new SqlConnection(connStrDb1))
        {
            conn1.Open();
            SqlCommand cmd1 = conn1.CreateCommand();
            cmd1.CommandText = string.Format("insert into T1 values(1)");
            cmd1.ExecuteNonQuery();
        }

        using (var conn2 = new SqlConnection(connStrDb2))
        {
            conn2.Open();
            var cmd2 = conn2.CreateCommand();
            cmd2.CommandText = string.Format("insert into T2 values(2)");
            cmd2.ExecuteNonQuery();
        }

        scope.Complete();
    }
```

### <a name="sharded-database-applications"></a>Applications de bases de données partitionnées

Les transactions de bases de données élastiques pour SQL Database et Managed Instance prennent également en charge la coordination des transactions distribuées dans lesquelles vous utilisez la méthode OpenConnectionForKey de la bibliothèque de client de bases de données élastiques pour ouvrir des connexions pour une couche Données ayant fait l’objet d’un scale-out. Envisagez le cas où vous avez besoin de garantir une cohérence transactionnelle pour des modifications intervenant sur plusieurs valeurs de clé de partitionnement différentes. Les connexions aux partitions hébergeant les différentes valeurs de clé de partitionnement sont réparties à l’aide d’OpenConnectionForKey. En règle générale, les connexions peuvent être établies sur différentes partitions de sorte que les garanties transactionnelles nécessitent une transaction distribuée.
L'exemple de code suivant illustre cette approche. Il suppose d’utiliser une variable appelée shardmap pour représenter un mappage de la partition à partir de la bibliothèque cliente de la base de données élastique :

```csharp
    using (var scope = new TransactionScope())
    {
        using (var conn1 = shardmap.OpenConnectionForKey(tenantId1, credentialsStr))
        {
            SqlCommand cmd1 = conn1.CreateCommand();
            cmd1.CommandText = string.Format("insert into T1 values(1)");
            cmd1.ExecuteNonQuery();
        }

        using (var conn2 = shardmap.OpenConnectionForKey(tenantId2, credentialsStr))
        {
            var cmd2 = conn2.CreateCommand();
            cmd2.CommandText = string.Format("insert into T1 values(2)");
            cmd2.ExecuteNonQuery();
        }

        scope.Complete();
    }
```

## <a name="transact-sql-development-experience"></a>Expérience de développement en Transact-SQL

Les transactions distribuées côté serveur en Transact-SQL sont uniquement disponibles pour Azure SQL Managed Instance. Une transaction distribuée peut être exécutée uniquement entre des instances gérées qui appartiennent au même [groupe d’approbations de serveurs](../managed-instance/server-trust-group-overview.md). Dans ce scénario, les instances gérées doivent utiliser un [serveur lié](/sql/relational-databases/linked-servers/create-linked-servers-sql-server-database-engine#TsqlProcedure) pour se référencer les unes les autres.

L’exemple de code Transact-SQL suivant utilise [BEGIN DISTRIBUTED TRANSACTION](/sql/t-sql/language-elements/begin-distributed-transaction-transact-sql) pour démarrer une transaction distribuée.

```Transact-SQL

    -- Configure the Linked Server
    -- Add one Azure SQL Managed Instance as Linked Server
    EXEC sp_addlinkedserver
        @server='RemoteServer', -- Linked server name
        @srvproduct='',
        @provider='sqlncli', -- SQL Server Native Client
        @datasrc='managed-instance-server.46e7afd5bc81.database.windows.net' -- Managed Instance endpoint

    -- Add credentials and options to this Linked Server
    EXEC sp_addlinkedsrvlogin
        @rmtsrvname = 'RemoteServer', -- Linked server name
        @useself = 'false',
        @rmtuser = '<login_name>',         -- login
        @rmtpassword = '<secure_password>' -- password

    USE AdventureWorks2012;
    GO
    SET XACT_ABORT ON;
    GO
    BEGIN DISTRIBUTED TRANSACTION;
    -- Delete candidate from local instance.
    DELETE AdventureWorks2012.HumanResources.JobCandidate
        WHERE JobCandidateID = 13;
    -- Delete candidate from remote instance.
    DELETE RemoteServer.AdventureWorks2012.HumanResources.JobCandidate
        WHERE JobCandidateID = 13;
    COMMIT TRANSACTION;
    GO
```

## <a name="combining-net-and-transact-sql-development-experience"></a>Combinaison des expériences de développement .NET et Transact-SQL

Les applications .NET qui utilisent les classes System.Transaction peuvent combiner la classe TransactionScope avec l’instruction Transact-SQL BEGIN DISTRIBUTED TRANSACTION. Dans TransactionScope, la transaction interne qui exécute BEGIN DISTRIBUTED TRANSACTION est explicitement promue en transaction distribuée. En outre, lorsque la seconde SqlConnecton est ouverte dans TransactionScope, elle est implicitement promue en transaction distribuée. Une fois la transaction distribuée démarrée, toutes les demandes de transactions ultérieures, qu’elles proviennent de .NET ou de Transact-SQL, se joignent à la transaction distribuée parent. En conséquence, toutes les étendues de transactions imbriquées initiées par l’instruction BEGIN se retrouveront dans la même transaction et les instructions COMMIT/ROLLBACK auront l’effet suivant sur le résultat global :
 * L’instruction COMMIT n’aura aucun effet sur l’étendue de transaction initiée par l’instruction BEGIN, c’est-à-dire qu’aucun résultat ne sera validé avant l’appel de la méthode Complete() sur l’objet TransactionScope. Si l’objet TransactionScope est détruit avant d’être terminé, toutes les modifications effectuées dans l’étendue sont restaurées à l’état antérieur.
 * L’instruction ROLLBACK entraînera la restauration de l’intégralité de TransactionScope à son état antérieur. Toute tentative d’inscription de nouvelles transactions dans TransactionScope échouera par la suite, ainsi que toute tentative d’appeler Complete() sur l’objet TransactionScope.

Voici un exemple où la transaction est explicitement promue en transaction distribuée avec Transact-SQL.

```csharp
    using (TransactionScope s = new TransactionScope())
    {
        using (SqlConnection conn = new SqlConnection(DB0_ConnectionString)
        {
            conn.Open();
        
            // Transaction is here promoted to distributed by BEGIN statement
            //
            Helper.ExecuteNonQueryOnOpenConnection(conn, "BEGIN DISTRIBUTED TRAN");
            // ...
        }
     
        using (SqlConnection conn2 = new SqlConnection(DB1_ConnectionString)
        {
            conn2.Open();
            // ...
        }
        
        s.Complete();
    }
```

L’exemple suivant montre une transaction qui est promue implicitement en transaction distribuée une fois que la seconde SqlConnecton a été démarrée dans TransactionScope.

```csharp
    using (TransactionScope s = new TransactionScope())
    {
        using (SqlConnection conn = new SqlConnection(DB0_ConnectionString)
        {
            conn.Open();
            // ...
        }
        
        using (SqlConnection conn = new SqlConnection(DB1_ConnectionString)
        {
            // Because this is second SqlConnection within TransactionScope transaction is here implicitly promoted distributed.
            //
            conn.Open(); 
            Helper.ExecuteNonQueryOnOpenConnection(conn, "BEGIN DISTRIBUTED TRAN");
            Helper.ExecuteNonQueryOnOpenConnection(conn, lsQuery);
            // ...
        }
        
        s.Complete();
    }
```

## <a name="transactions-across-multiple-servers-for-azure-sql-database"></a>Transactions sur plusieurs serveurs pour Azure SQL Database

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par Azure SQL Database, mais tous les développements futurs sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm.

Les transactions de bases de données élastiques sont prises en charge sur plusieurs serveurs dans Azure SQL Database. Quand les transactions franchissent les limites du serveur, les serveurs participants doivent d’abord entrer dans une relation de communication mutuelle. Une fois la relation de communication établie, n’importe quelle base de données de n’importe lequel des deux serveurs peut participer à des transactions élastiques avec des bases de données de l'autre serveur. Quand les transactions couvrent plus de deux serveurs, une relation de communication doit être mise en place pour toutes les paires de serveurs.

Pour gérer les relations de communication entre serveurs pour les transactions de bases de données élastiques, utilisez les applets de commande PowerShell suivants :

* **New-AzSqlServerCommunicationLink** : Utilisez cette applet de commande pour créer une relation de communication entre deux serveurs dans Azure SQL Database. La relation est symétrique, ce qui signifie que chacun des deux serveurs peut lancer des transactions avec l’autre serveur.
* **Get-AzSqlServerCommunicationLink** : Utilisez ce cmdlet pour extraire les relations de communication existantes et leurs propriétés.
* **Remove-AzSqlServerCommunicationLink** : Utilisez ce cmdlet pour supprimer une relation de communication existante.

## <a name="transactions-across-multiple-servers-for-azure-sql-managed-instance"></a>Transactions sur plusieurs serveurs pour Azure SQL Managed Instance

Les transactions distribuées sont prises en charge sur plusieurs serveurs dans Azure SQL Managed Instance. Quand les transactions franchissent les limites de Managed Instance, les instances participantes doivent d’abord entrer dans une relation réciproque de communication et de sécurité. Pour ce faire, créez un [groupe d’approbations de serveurs](../managed-instance/server-trust-group-overview.md), ce qui peut être effectué sur Portail Azure. Si les instances gérées ne se trouvent pas sur le même réseau virtuel, l’[appairage de réseaux virtuels](../../virtual-network/virtual-network-peering-overview.md) doit être configuré et les règles de trafic entrant et sortant du groupe de sécurité réseau doivent autoriser les ports 5024 et 11000 à 12000 sur tous les réseaux virtuels concernés.

  ![Groupes d’approbations de serveurs sur Portail Azure][3]

Le diagramme suivant illustre le groupe d’approbations de serveurs avec des instances gérées qui peuvent exécuter des transactions distribuées avec .NET ou Transact-SQL.

  ![Transactions distribuées avec Azure SQL Managed Instance utilisant des transactions élastiques][2]

## <a name="monitoring-transaction-status"></a>Surveillance de l’état de transaction

Utilisez les vues de gestion dynamique (DMV) pour surveiller l’état et la progression de vos transactions de bases de données élastiques en cours. Toutes les DMV relatives aux transactions s’appliquent aux transactions distribuées dans SQL Database et Managed Instance. La liste des vues de gestion dynamique est disponible ici : [Vues de gestion dynamique liées à la transaction et fonctions (Transact-SQL)](/sql/relational-databases/system-dynamic-management-views/transaction-related-dynamic-management-views-and-functions-transact-sql).

Les vues de gestion dynamique ci-dessous sont particulièrement utiles :

* **sys.dm\_tran\_active\_transactions** : Répertorie les transactions actives et leur état. La colonne UOW (Unit Of Work) peut identifier les différentes transactions enfants qui appartiennent à la même transaction distribuée. Toutes les transactions associées à la même transaction distribuée ont la même valeur UOW. Pour plus d’informations, consultez la [documentation DMV](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-active-transactions-transact-sql).
* **sys.dm\_tran\_database\_transactions** : Fournit des informations supplémentaires sur les transactions, telles que le placement de la transaction dans le journal. Pour plus d’informations, consultez la [documentation DMV](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-database-transactions-transact-sql).
* **sys.dm\_tran\_locks** : Fournit des informations sur les verrous maintenus par les transactions en cours. Pour plus d’informations, consultez la [documentation DMV](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-locks-transact-sql).

## <a name="limitations"></a>Limites

Les limites suivantes s’appliquent actuellement aux transactions de bases de données élastiques dans SQL Database :

* Seules les transactions entre les bases de données dans SQL Database sont prises en charge. Les autres fournisseurs de ressources [X/Open XA](https://en.wikipedia.org/wiki/X/Open_XA) et les bases de données hors de SQL Database ne peuvent pas participer aux transactions de bases de données élastiques. Cela signifie que les transactions de bases de données élastiques ne peuvent pas s’étendre sur une base de données Azure SQL et SQL Server locale. Pour les transactions distribuées en local, continuez à utiliser MSDTC.
* Seules les transactions coordonnées par le client à partir d’une application .NET sont prises en charge. La prise en charge côté serveur de T-SQL, comme BEGIN DISTRIBUTED TRANSACTION, est planifiée, mais pas encore disponible.
* Les transactions entre les services WCF ne sont pas prises en charge. Par exemple, vous disposez d’une méthode de service WCF qui exécute une transaction. L’inclusion de l’appel dans une étendue de transaction échouera en levant l’exception [System.ServiceModel.ProtocolException](/dotnet/api/system.servicemodel.protocolexception).

Les limites suivantes s’appliquent actuellement aux transactions distribuées dans Managed Instance :

* Seules les transactions entre les bases de données dans Managed Instance sont prises en charge. Les autres fournisseurs de ressources [X/Open XA](https://en.wikipedia.org/wiki/X/Open_XA) et les bases de données en dehors d’Azure SQL Managed Instance ne peuvent pas participer aux transactions distribuées. Cela signifie que les transactions distribuées ne peuvent pas s’étendre sur des instances locales de SQL Server et Azure SQL Managed Instance. Pour les transactions distribuées en local, continuez à utiliser MSDTC.
* Les transactions entre les services WCF ne sont pas prises en charge. Par exemple, vous disposez d’une méthode de service WCF qui exécute une transaction. L’inclusion de l’appel dans une étendue de transaction échouera en levant l’exception [System.ServiceModel.ProtocolException](/dotnet/api/system.servicemodel.protocolexception).
* Azure SQL Managed Instance doit faire partie d’un [groupe d’approbations de serveurs](../managed-instance/server-trust-group-overview.md) pour participer à une transaction distribuée.
* Les limitations des [groupes d’approbations de serveurs](../managed-instance/server-trust-group-overview.md) concernent les transactions distribuées.
* Les instances gérées qui participent à des transactions distribuées doivent disposer d’une connectivité sur des points de terminaison privés (à l’aide d’une adresse IP privée du réseau virtuel sur lequel elles sont déployées) et doivent être référencées réciproquement à l’aide de noms de domaine complets privés. Les applications clientes peuvent utiliser des transactions distribuées sur des points de terminaison privés. En outre, dans les cas où Transact-SQL utilise des serveurs liés référençant des points de terminaison privés, les applications clientes peuvent également utiliser des transactions distribuées sur des points de terminaison publics. Cette limitation est expliquée dans le diagramme suivant.
  ![Private endpoint connectivity limitation][4]
## <a name="next-steps"></a>Étapes suivantes

* Pour toute question, contactez-nous par le biais de la [page de questions Microsoft Q&A pour SQL Database](/answers/topics/azure-sql-database.html).
* Pour les demandes de fonctionnalités, ajoutez-les au [forum de commentaires SQL Database](https://feedback.azure.com/forums/217321-sql-database/) ou au [forum Managed Instance](https://feedback.azure.com/forums/915676-sql-managed-instance).



<!--Image references-->
[1]: ./media/elastic-transactions-overview/distributed-transactions.png
[2]: ./media/elastic-transactions-overview/sql-mi-distributed-transactions.png
[3]: ./media/elastic-transactions-overview/server-trust-groups-azure-portal.png
[4]: ./media/elastic-transactions-overview/managed-instance-distributed-transactions-private-endpoint-limitations.png
