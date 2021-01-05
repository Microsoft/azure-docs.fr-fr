---
title: Automatisation de travaux
description: Utiliser l’automatisation des travaux pour exécuter des scripts T-SQL (Transact-SQL) sur une ou plusieurs bases de données
services: sql-database
ms.service: sql-db-mi
ms.subservice: features
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/10/2020
ms.openlocfilehash: 7ecd7e847a91847db8f57c640a374dc329fce7ea
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92782941"
---
# <a name="automate-management-tasks-using-database-jobs"></a>Automatiser des tâches de gestion avec des travaux de base de données
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Vous pouvez créer et planifier des travaux qui s’exécutent périodiquement sur une ou plusieurs bases de données afin d’exécuter des requêtes T-SQL (Transact-SQL) et d’effectuer des tâches de maintenance.

Vous pouvez définir la base de données ou les groupes de bases de données cibles où le travail sera exécuté, et aussi définir des planifications pour l’exécution d’un travail.
Un travail gère la tâche de connexion à la base de données cible. Vous pouvez également définir, gérer et conserver des scripts Transact-SQL à exécuter sur un groupe de bases de données.

Chaque tâche consigne l’état de l’exécution et retente également automatiquement les opérations en cas d’échec.

## <a name="when-to-use-automated-jobs"></a>Quand utiliser les travaux automatisés

Vous pouvez utiliser l’automatisation des travaux dans plusieurs scénarios :

- Automatiser des tâches de gestion et les planifier pour qu’elles s’exécutent tous les jours ouvrés, en dehors des heures d’ouverture, etc.
  - Déployer des changements de schéma, la gestion des informations d’identification, la collecte des données de performances ou la collecte des données de télémétrie du locataire (client).
  - Mettre à jour des données de référence (des informations communes à toutes les bases de données), charger des données depuis Stockage Blob Azure.
  - Recréer des indexes pour améliorer les performances d’une requête. Configurer des travaux de manière à ce qu’ils s’exécutent sur une collection de bases de données de façon récurrente, par exemple pendant les heures creuses.
  - Collecter les résultats de la requête à partir d'un ensemble de bases de données dans une table centrale sur une base continue. Les requêtes de performances peuvent être exécutées en permanence et configurées pour déclencher des tâches supplémentaires à exécuter.
- Collecter des données de création de rapports
  - Agréger des données provenant d’une collection de bases de données dans une table de destination unique.
  - Exécuter des requêtes de traitement de données avec un temps d’exécution plus long sur un grand ensemble de bases de données, par exemple, la collection de télémétrie de client. Les résultats sont rassemblés dans une table de destination unique pour une analyse ultérieure.
- Déplacements de données
  - Créez des travaux qui répliquent les modifications apportées à vos bases de données sur d’autres bases de données, ou qui collectent les mises à jour effectuées dans des bases de données distantes et appliquent les modifications à la base de données.
  - Créez des travaux qui chargent des données depuis ou vers vos bases de données avec SQL Server Integration Services (SSIS).

## <a name="overview"></a>Vue d’ensemble

Les technologies de planification de travaux suivantes sont disponibles :

- **Travaux SQL Agent** est le composant classique et éprouvé de planification de travaux SQL Server disponible dans Azure SQL Managed Instance. Les travaux SQL Agent ne sont pas disponibles dans Azure SQL Database.
- **Travaux de base de données élastique (préversion)** est le service de planification de travaux qui exécute des travaux personnalisés sur une ou plusieurs bases de données dans Azure SQL Database.

Il convient de noter quelques différences entre SQL Agent (disponible localement et dans SQL Managed Instance) et l’agent de travail élastique de base de données (disponible pour les bases de données uniques dans Azure SQL Database et les bases de données dans Azure Synapse Analytics).

| |Travaux élastiques |SQL Agent |
|---------|---------|---------|
|**Portée** | N’importe quel nombre de bases de données dans Azure SQL Database et/ou d’entrepôts de données dans le même cloud Azure que l’agent de travail. Les cibles peuvent se trouver dans des serveurs, régions et/ou abonnements différents. <br><br>Les groupes cibles peuvent être composés de bases de données ou d’entrepôts de données individuels, ou bien de toutes les bases de données d’un serveur, d’un pool ou d’une carte de partitions (énumérées dynamiquement au moment de l’exécution du travail). | N’importe quelle base de données individuelle dans la même instance que SQL Agent. |
|**API et outils pris en charge** | Portail, PowerShell, T-SQL, Azure Resource Manager | T-SQL, SSMS (SQL Server Management Studio) |

## <a name="sql-agent-jobs"></a>Travaux SQL Agent

Les travaux SQL Agent sont constitués de séries spécifiées de scripts T-SQL sur votre base de données. Utilisez des travaux pour définir une tâche d’administration qui peut être exécutée une ou plusieurs fois, et dont la réussite ou l’échec sont surveillés.
Un travail peut s’exécuter sur un serveur local ou sur plusieurs serveurs distants. Travaux SQL Agent est un composant du moteur de base de données interne qui est exécuté au sein du service Managed Instance.
Il existe plusieurs concepts clés dans les travaux SQL Agent :

- **Étapes de travail**  : ensemble d’une ou plusieurs étapes qui doivent être exécutées au sein du travail. Pour chaque étape de travail, vous pouvez définir une stratégie de nouvelle tentative et l’action qui doit se produire si l’étape de travail réussit ou échoue.
- Les **planifications** définissent quand le travail doit être exécuté.
- Les **notifications** vous permettent de définir des règles qui sont utilisées pour notifier les opérateurs via des e-mails une fois le travail terminé.

### <a name="job-steps"></a>Étapes de travail

Les étapes de travail SQL Agent sont des séquences d’actions que SQL Agent doit s’exécuter. Pour chaque étape, vous définissez l’étape suivante à exécuter si elle réussit ou si elle échoue, ainsi que le nombre de nouvelles tentatives en cas d’échec.

SQL Agent vous permet de créer différents types d’étapes de travail, comme une étape de travail Transact-SQL qui exécute un lot Transact-SQL sur la base de données, ou des étapes de commande du système d’exploitation/PowerShell qui peuvent exécuter un script personnalisé du système d’exploitation, des étapes de travail SSIS qui vous permettent de charger des données avec le runtime SSIS, ou des étapes de [réplication](../managed-instance/replication-transactional-overview.md) qui peuvent publier les modifications de votre base de données auprès d’autres bases de données.

La [réplication transactionnelle](../managed-instance/replication-transactional-overview.md) est une fonctionnalité du moteur de base de données qui vous permet de publier les modifications apportées à une ou plusieurs tables d’une base de données, et de les publier/distribuer à un ensemble de bases de données abonnées. La publication des modifications est implémentée avec les types d’étape de travail SQL Agent suivants :

- Lecteur de journaux de transactions
- Instantané
- Serveur de distribution.

Les autres types d’étapes de travail ne sont pas actuellement pris en charge, notamment :

- L’étape de travail de réplication de fusion n’est pas prise en charge.
- L’agent de lecture de la file d’attente n’est pas pris en charge.
- Analysis Services n’est pas pris en charge

### <a name="job-schedules"></a>Calendriers de travaux

Une planification spécifie quand un travail s’exécute. Plusieurs travaux peuvent s’exécuter sur la même planification, et plusieurs planifications peuvent appliquer le même travail.
Une planification peut définir les conditions suivantes pour l’heure d’exécution d’un travail :

- À chaque redémarrage de l’instance (ou au démarrage de SQL Server Agent). Le travail est activé après chaque basculement.
- Une seule fois, à une date et heure spécifiques, ce qui est pratique pour l’exécution différée d’un travail.
- Selon une planification récurrente.

> [!Note]
> SQL Managed Instance ne permet pas actuellement le démarrage d’un travail quand l’instance est « inactive ».

### <a name="job-notifications"></a>Notifications pour les travaux

Les travaux SQL Agent vous permettent de recevoir des notifications quand le travail se termine avec succès ou quand il échoue. Vous pouvez recevoir des notifications par e-mail.

Vous devez d’abord configurer le compte e-mail utilisé pour envoyer les notifications par e-mail et affecter le compte au profil de messagerie nommé `AzureManagedInstance_dbmail_profile`, comme illustré dans l’exemple suivant :

```sql
-- Create a Database Mail account
EXECUTE msdb.dbo.sysmail_add_account_sp
    @account_name = 'SQL Agent Account',
    @description = 'Mail account for Azure SQL Managed Instance SQL Agent system.',
    @email_address = '$(loginEmail)',
    @display_name = 'SQL Agent Account',
    @mailserver_name = '$(mailserver)' ,
    @username = '$(loginEmail)' ,
    @password = '$(password)'

-- Create a Database Mail profile
EXECUTE msdb.dbo.sysmail_add_profile_sp
    @profile_name = 'AzureManagedInstance_dbmail_profile',
    @description = 'E-mail profile used for messages sent by Managed Instance SQL Agent.' ;

-- Add the account to the profile
EXECUTE msdb.dbo.sysmail_add_profileaccount_sp
    @profile_name = 'AzureManagedInstance_dbmail_profile',
    @account_name = 'SQL Agent Account',
    @sequence_number = 1;
```

Vous devez également activer Database Mail sur Managed Instance :

```sql
GO
EXEC sp_configure 'show advanced options', 1;
GO
RECONFIGURE;
GO
EXEC sp_configure 'Database Mail XPs', 1;
GO
RECONFIGURE
```

Vous pouvez notifier l’opérateur que quelque chose s’est produit avec vos travaux SQL Agent. Un opérateur définit les informations de contact d’une personne responsable de la maintenance d’une ou plusieurs instances dans SQL Managed Instance. Parfois, les responsabilités d’opérateur sont affectées à une seule personne.
Dans les systèmes avec plusieurs instances dans SQL Managed Instance ou SQL Server, les responsabilités d’opérateur peuvent être partagées entre de nombreuses personnes. Un opérateur ne contient pas d’informations de sécurité et ne définit pas de principal de sécurité.

Vous pouvez créer des opérateurs avec SSMS ou le script Transact-SQL illustré dans l’exemple suivant :

```sql
EXEC msdb.dbo.sp_add_operator
    @name=N'Mihajlo Pupun',
    @enabled=1,
    @email_address=N'mihajlo.pupin@contoso.com'
```

Vous pouvez modifier n’importe travail et affecter l’opérateur qui est notifié par e-mail si le travail se termine, échoue ou réussit avec SSMS ou avec le script Transact-SQL suivant :

```sql
EXEC msdb.dbo.sp_update_job @job_name=N'Load data using SSIS',
    @notify_level_email=3, -- Options are: 1 on succeed, 2 on failure, 3 on complete
    @notify_email_operator_name=N'Mihajlo Pupun'
```

### <a name="sql-agent-job-limitations"></a>Limitations du travail SQL Agent

Certaines des fonctionnalités de SQL Agent disponibles dans SQL Server ne sont pas prises en charge dans Managed Instance :

- Les paramètres de l’Agent SQL sont en lecture seule. La procédure `sp_set_agent_properties` n’est pas prise en charge dans Managed Instance.
- L’activation/désactivation de SQL Agent n’est actuellement pas prise en charge dans Managed Instance. L’Agent SQL est toujours en cours d’exécution.
- Les notifications sont partiellement prises en charge
  - Les récepteurs de radiomessagerie ne sont pas pris en charge.
  - NetSend n’est pas pris en charge.
  - Les alertes ne sont pas prises en charge.
- Les proxies ne sont pas pris en charge.
- EventLog n’est pas pris en charge.

Pour plus d’informations sur SQL Server Agent, consultez [SQL Server Agent](/sql/ssms/agent/sql-server-agent).

## <a name="elastic-database-jobs-preview"></a>Tâches de base de données élastique (préversion)

Les **travaux de base de données élastiques** permettent d’exécuter un ou plusieurs scripts T-SQL en parallèle sur un grand nombre de bases de données, selon une planification ou à la demande.

**Exécutez des travaux sur n’importe quelle combinaison de bases de données** : une ou plusieurs bases de données individuelles, toutes les bases de données d’un serveur, toutes les bases de données dans un pool élastique ou une carte de partitions, avec en plus la possibilité d’inclure ou d’exclure des bases de données spécifiques. **Les travaux peuvent s’exécuter sur plusieurs serveurs, plusieurs pools et même sur des bases de données dans différents abonnements.** Les serveurs et les pools sont énumérés dynamiquement au moment de l’exécution, ce qui permet aux travaux de s’exécuter sur toutes les bases de données présentes dans le groupe cible au moment de l’exécution.

L’image suivante montre un agent de travail qui exécute des travaux sur les différents types de groupes cibles :

![Modèle conceptuel de l’agent de travail élastique](./media/job-automation-overview/conceptual-diagram.png)

### <a name="elastic-job-components"></a>Composants des travaux élastiques

|Composant | Description (détails supplémentaires sous le tableau) |
|---------|---------|
|[**Agent de travail élastique**](#elastic-job-agent) | Ressource Azure que vous créez pour exécuter et gérer des travaux. |
|[**Base de données de travaux**](#job-database) | Base de données dans Azure SQL Database qui est utilisée par l’agent de travail pour stocker les données associées aux travaux, les définitions des travaux, etc. |
|[**Groupe cible**](#target-group) | Ensemble de serveurs, de pools, de bases de données et de cartes de partitions sur lesquels un travail est exécuté. |
|[**Travail**](#job) | Un travail est une unité de travail composée d’une ou plusieurs [étapes de travail](#job-step). Celles-ci spécifient le script T-SQL à exécuter, ainsi que d’autres détails nécessaires pour exécuter le script. |

#### <a name="elastic-job-agent"></a>Agent de travail élastique

Un agent de travail élastique est la ressource Azure qui permet de créer, d’exécuter et de gérer des travaux. L’agent de travail élastique est une ressource Azure que vous créez dans le portail ([PowerShell](elastic-jobs-powershell-create.md) et REST sont également pris en charge).

La création d’un **agent de travail élastique** nécessite l’existence d’une base de données dans Azure SQL Database. L’agent configure cette base de données existante comme [*base de données de travaux*](#job-database).

L’agent de travail élastique est gratuit. La base de données de travaux est facturée au même tarif que les autres bases de données dans Azure SQL Database.

#### <a name="job-database"></a>Base de données de travaux

La *base de données de travaux* permet de définir des travaux et de suivre l’état et l’historique des exécutions des travaux. La *base de données de travaux* permet également de stocker les métadonnées de l’agent, les journaux d’activité, les résultats et les définitions des travaux. Par ailleurs, elle contient de nombreuses procédures stockées utiles, ainsi que d’autres objets de base de données pour créer, exécuter et gérer des travaux avec T-SQL.

Pour la préversion actuelle, une base de données existante dans Azure SQL Database (S0 au minimum) est nécessaire pour créer un agent de travail élastique.

La *base de données de travaux* ne doit pas nécessairement être nouvelle, mais elle doit être nettoyée, vide et configurée avec l’objectif de service S0 ou supérieur. L’objectif de service recommandé pour la *base de données de travaux* est au moins S1, mais le meilleur choix varie en fonction des besoins en termes de performances de vos travaux : nombre d’étapes de travail, nombre de cibles de travail et fréquence d’exécution des travaux. Par exemple, une base de données S0 peut convenir à un agent de travail qui exécute un petit nombre de travaux par heure ciblant moins de dix bases de données, mais elle risque de ne pas être assez rapide pour l’exécution d’un travail toutes les minutes. Dans ce cas, un niveau de service plus élevé peut être préférable.

Si les opérations sur la base de données de travaux sont plus lentes que prévu, [supervisez](monitor-tune-overview.md#azure-sql-database-and-azure-sql-managed-instance-resource-monitoring) les performances de la base de données et l’utilisation des ressources dans la base de données de travaux pendant les périodes de lenteur en utilisant le portail Azure ou la vue de gestion dynamique [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database). Si l’utilisation d’une ressource, telle que le processeur, les E/S de données ou l’écriture de journal, approche 100 % et correspond aux périodes de lenteur, envisagez de mettre la base de données à l’échelle de manière incrémentielle pour atteindre des objectifs de service plus élevés (dans le [modèle DTU](service-tiers-dtu.md) ou dans le [modèle vCore](service-tiers-vcore.md)) jusqu’à ce que les performances de la base de données de travaux soient satisfaisantes.

##### <a name="job-database-permissions"></a>Autorisations associées à la base de données de travaux

Durant la création de l’agent de travail, un schéma, des tables et un rôle appelé *jobs_reader* sont créés dans la *base de données de travaux* . Le rôle, créé avec l’autorisation suivante, est conçu pour permettre aux administrateurs de contrôler plus précisément les accès aux travaux à des fins de monitoring :

|Nom de rôle |Autorisations de schéma « jobs » |Autorisations de schéma « jobs_internal » |
|---------|---------|---------|
|**jobs_reader** | SELECT | None |

> [!IMPORTANT]
> Tenez compte des implications en matière de sécurité avant d’accorder l’accès à la *base de données de travaux* en tant qu’administrateur de base de données. Un utilisateur malveillant disposant d’autorisations appropriées peut créer ou modifier un travail utilisant des informations d’identification stockées pour se connecter à une base de données, ce qui peut ensuite lui permettre de déterminer le mot de passe associé aux informations d’identification.

#### <a name="target-group"></a>Groupe cible

Un *groupe cible* définit l’ensemble de bases de données sur lequel une étape de travail s’exécute. Un groupe cible peut contenir les éléments suivants, dont le nombre et la combinaison peuvent varier :

- **Serveur SQL logique**  : si un serveur est spécifié, toutes les bases de données présentes sur le serveur au moment de l’exécution du travail font partie du groupe. Les informations d’identification de la base de données master doivent être fournies pour que le groupe puisse être énuméré et mis à jour avant l’exécution du travail.
- **Pool élastique** : si un pool élastique est spécifié, toutes les bases de données qui se trouvent dans ce pool au moment de l’exécution du travail font partie du groupe. Au même titre qu’un serveur, les informations d’identification de la base de données master doivent être fournies pour que le groupe puisse être mis à jour avant l’exécution du travail.
- **Base de données unique** : spécifiez une ou plusieurs bases de données individuelles à inclure dans le groupe.
- **Carte de partitions** : bases de données d’une carte de partitions.

> [!TIP]
> Au moment de l’exécution du travail, *l’énumération dynamique* réévalue l’ensemble des bases de données dans les groupes de cibles qui incluent des serveurs ou des pools. L’énumération dynamique garantit que **les travaux s’exécutent sur toutes les bases de données qui existent dans le serveur ou le pool au moment de l’exécution du travail** . La réévaluation de la liste des bases de données au moment de l’exécution est particulièrement utile dans les scénarios où l’appartenance au pool ou au serveur change fréquemment.

Il est possible de désigner des pools et des bases de données uniques comme faisant partie ou non du groupe. Vous pouvez ainsi créer un groupe cible avec n’importe quelle combinaison de bases de données. Par exemple, vous pouvez ajouter un serveur à un groupe cible, mais exclure certaines bases de données d’un pool élastique (ou exclure un pool entier).

Un groupe cible peut inclure des bases de données dans plusieurs abonnements et dans plusieurs régions. Notez que les exécutions entre régions ont une latence plus élevée que celles dans une même région.

Les exemples suivants montrent comment différentes définitions de groupes cibles sont énumérées dynamiquement au moment de l’exécution du travail pour déterminer les bases de données qui seront exécutées par le travail :

![Exemples de groupe cible](./media/job-automation-overview/targetgroup-examples1.png)

L’ **exemple 1** montre un groupe cible se composant d’une liste de bases de données individuelles. Lorsqu’une étape de travail est exécutée à l’aide de ce groupe cible, l’action de l’étape de travail sera exécutée dans chacune de ces bases de données.<br>
L’ **exemple 2** montre un groupe cible qui contient un serveur comme cible. Lorsqu’une étape de travail est exécutée à l’aide de ce groupe cible, le serveur est énuméré dynamiquement pour déterminer la liste des bases de données qui se trouvent actuellement dans le serveur. L’action de l’étape de travail sera exécutée dans chacun de ces bases de données.<br>
L’ **exemple 3** montre un groupe cible similaire à l’ *exemple 2* , mais une base de données individuelle est explicitement exclue. L’action de l’étape de travail ne sera *pas* exécutée dans la base de données exclue.<br>
L’ **exemple 4** montre un groupe cible qui contient un pool élastique comme cible. Semblable à *exemple 2* , le pool est énuméré dynamiquement à l’exécution du travail pour déterminer la liste des bases de données dans le pool.
<br><br>

![Exemples de groupes cibles supplémentaires](./media/job-automation-overview/targetgroup-examples2.png)

L’ **exemple 5** et l’ **exemple 6** montrent des scénarios avancés où les serveurs, les pools élastiques et les bases de données peuvent être combinés avec des règles d’inclusion et d’exclusion.<br>
L’ **exemple 7** montre que les partitions d’une carte de partitions peuvent également être évaluées au moment de l’exécution du travail.

> [!NOTE]
> La base de données de travaux peut être la cible d’un travail. Dans ce scénario, la base de données de travaux est traitée comme toute autre base de données cible. L’utilisateur du travail doit être créé et disposer d’autorisations suffisantes dans la base de données de travaux, et les informations d’identification délimitées à la base de données de l’utilisateur du travail doivent également exister dans la base de données de travaux, comme pour toute autre base de données cible.
>

#### <a name="job"></a>Travail

Un *travail* est un élément unitaire exécuté selon une planification ou de manière ponctuelle. Un travail est constitué d’une ou plusieurs *étapes de travail* .

##### <a name="job-step"></a>Étape de travail

Chaque étape de travail spécifie un script T-SQL à exécuter, un ou plusieurs groupes cibles sur lesquels exécuter le script T-SQL, ainsi que les informations d’identification dont l’agent de travail a besoin pour se connecter à la base de données cible. Chaque étape de travail est associée à un délai d’expiration et à des stratégies de nouvelle tentative personnalisables, et peut éventuellement spécifier des paramètres de sortie.

#### <a name="job-output"></a>Sortie du travail

Les résultats des étapes d’un travail sur chaque base de données sont enregistrés en détail, et la sortie du script peut être capturée dans une table spécifiée. Vous pouvez aussi indiquer une base de données pour enregistrer toutes les données retournées par un travail.

#### <a name="job-history"></a>Historique des travaux

L’historique d’exécution des travaux est stocké dans la *base de données de travaux* . Un travail de nettoyage système vide les éléments de l’historique d’exécution qui datent de plus de 45 jours. Pour supprimer ceux de moins de 45 jours, appelez la procédure stockée **sp_purge_history** dans la *base de données des travaux* .

### <a name="agent-performance-capacity-and-limitations"></a>Performances, capacité et limitations de l’agent

Les travaux élastiques utilisent des ressources de calcul minimales dans l’attente de la finalisation des travaux de longue durée.

En fonction de la taille du groupe cible de bases de données et du délai d’exécution souhaité d’un travail (nombre de workers simultanés), l’agent nécessite différents niveaux de calcul et de performance de la part de la *base de données de travaux* (plus le nombre de cibles et de travaux est élevé, plus les calculs nécessaires sont importants).

La préversion est actuellement limitée à 100 travaux simultanés.

#### <a name="prevent-jobs-from-reducing-target-database-performance"></a>Empêcher les travaux de réduire le niveau de performance de la base de données cible

Pour éviter la surcharge des ressources durant l’exécution de travaux sur des bases de données dans un pool élastique SQL, vous pouvez configurer les travaux de manière à limiter le nombre de bases de données sur lesquelles un travail peut s’exécuter simultanément.

## <a name="next-steps"></a>Étapes suivantes

- [Présentation de SQL Server Agent](/sql/ssms/agent/sql-server-agent)
- [Guide pour créer et gérer des travaux élastiques](elastic-jobs-overview.md)
- [Créer et gérer des tâches élastiques à l’aide de PowerShell](elastic-jobs-powershell-create.md)
- [Créer et gérer des tâches élastiques à l’aide de T-SQL (Transact-SQL)](elastic-jobs-tsql-create-manage.md)