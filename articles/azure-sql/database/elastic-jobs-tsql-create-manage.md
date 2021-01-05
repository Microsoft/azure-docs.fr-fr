---
title: Créer et gérer des travaux de base de données élastique (préversion) avec Transact-SQL (T-SQL)
description: Exécuter des scripts sur plusieurs bases de données avec l’agent de travail de base de données élastique à l’aide de Transact-SQL (T-SQL).
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: how-to
ms.author: jaredmoo
author: jaredmoo
ms.reviewer: sstein
ms.date: 02/07/2020
ms.openlocfilehash: 9c9f5972cdd2690b86610ea585bdd82d736ed163
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792138"
---
# <a name="use-transact-sql-t-sql-to-create-and-manage-elastic-database-jobs-preview"></a>Utiliser Transact-SQL (T-SQL) pour créer et gérer des travaux de base de données élastique (préversion)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article fournit de nombreux exemples de scénarios pour commencer à utiliser les travaux élastiques à l’aide de T-SQL.

Les exemples utilisent les [procédures stockées](#job-stored-procedures) et les [affichages](#job-views) disponibles dans la [*base de données du travail*](job-automation-overview.md#job-database).

Transact-SQL (T-SQL) est utilisé pour créer, configurer, exécuter et gérer des travaux. Comme la création de l’agent de travail élastique n'est pas prise en charge dans T-SQL, vous devez d’abord créer un *agent de travail élastique* à l’aide du portail ou de [PowerShell](elastic-jobs-powershell-create.md#create-the-elastic-job-agent).

## <a name="create-a-credential-for-job-execution"></a>Créer des informations d’identification pour l’exécution du travail

Les informations d’identification sont utilisées pour se connecter à vos bases de données cibles pour l’exécution du script. Les informations d’identification ont besoin d’autorisations appropriées pour les bases de données spécifiées par le groupe cible afin de pouvoir exécuter le script. Lors de l’utilisation d’un [serveur SQL logique](logical-servers.md) ou d’un membre du groupe cible de pools, il est fortement recommandé de créer des informations d’identification principales à utiliser pour actualiser les informations d’identification avant l’expansion du serveur ou du pool au moment de l’exécution du travail. Les informations d’identification incluses dans l’étendue de la base de données sont créées dans la base de données de l’agent de travail. Les mêmes informations d’identification doivent être utilisée pour *créer une connexion* et *créer un utilisateur à partir de la connexion afin d’accorder les autorisations de connexion à la base de donnée* sur les bases de données cibles.

```sql
--Connect to the job database specified when creating the job agent

-- Create a db master key if one does not already exist, using your own password.  
CREATE MASTER KEY ENCRYPTION BY PASSWORD='<EnterStrongPasswordHere>';  
  
-- Create a database scoped credential.  
CREATE DATABASE SCOPED CREDENTIAL myjobcred WITH IDENTITY = 'jobcred',
    SECRET = '<EnterStrongPasswordHere>';
GO

-- Create a database scoped credential for the master database of server1.
CREATE DATABASE SCOPED CREDENTIAL mymastercred WITH IDENTITY = 'mastercred',
    SECRET = '<EnterStrongPasswordHere>';
GO
```

## <a name="create-a-target-group-servers"></a>Créer un groupe cible (serveurs)

L’exemple suivant montre comment exécuter un travail sur toutes les bases de données d’un serveur.  
Se connecter à la [*base de données de travail*](job-automation-overview.md#job-database) et exécuter la commande suivante :

```sql
-- Connect to the job database specified when creating the job agent

-- Add a target group containing server(s)
EXEC jobs.sp_add_target_group 'ServerGroup1'

-- Add a server target member
EXEC jobs.sp_add_target_group_member
'ServerGroup1',
@target_type = 'SqlServer',
@refresh_credential_name='mymastercred', --credential required to refresh the databases in a server
@server_name='server1.database.windows.net'

--View the recently created target group and target group members
SELECT * FROM jobs.target_groups WHERE target_group_name='ServerGroup1';
SELECT * FROM jobs.target_group_members WHERE target_group_name='ServerGroup1';
```

## <a name="exclude-an-individual-database"></a>Exclure une base de données individuelle

L’exemple suivant montre comment exécuter un travail sur toutes les bases de données d’un serveur, à l’exception de la base de données nommée *MappingDB* .  
Se connecter à la [*base de données de travail*](job-automation-overview.md#job-database) et exécuter la commande suivante :

```sql
--Connect to the job database specified when creating the job agent

-- Add a target group containing server(s)
EXEC [jobs].sp_add_target_group N'ServerGroup'
GO

-- Add a server target member
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'ServerGroup',
@target_type = N'SqlServer',
@refresh_credential_name=N'mymastercred', --credential required to refresh the databases in a server
@server_name=N'London.database.windows.net'
GO

-- Add a server target member
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'ServerGroup',
@target_type = N'SqlServer',
@refresh_credential_name=N'mymastercred', --credential required to refresh the databases in a server
@server_name='server2.database.windows.net'
GO

--Exclude a database target member from the server target group
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'ServerGroup',
@membership_type = N'Exclude',
@target_type = N'SqlDatabase',
@server_name = N'server1.database.windows.net',
@database_name =N'MappingDB'
GO

--View the recently created target group and target group members
SELECT * FROM [jobs].target_groups WHERE target_group_name = N'ServerGroup';
SELECT * FROM [jobs].target_group_members WHERE target_group_name = N'ServerGroup';
```

## <a name="create-a-target-group-pools"></a>Créer un groupe cible (pools)

L’exemple suivant montre comment cibler toutes les bases de données dans un ou plusieurs pools élastiques.  
Se connecter à la [*base de données de travail*](job-automation-overview.md#job-database) et exécuter la commande suivante :

```sql
--Connect to the job database specified when creating the job agent

-- Add a target group containing pool(s)
EXEC jobs.sp_add_target_group 'PoolGroup'

-- Add an elastic pool(s) target member
EXEC jobs.sp_add_target_group_member
'PoolGroup',
@target_type = 'SqlElasticPool',
@refresh_credential_name='mymastercred', --credential required to refresh the databases in a server
@server_name='server1.database.windows.net',
@elastic_pool_name='ElasticPool-1'

-- View the recently created target group and target group members
SELECT * FROM jobs.target_groups WHERE target_group_name = N'PoolGroup';
SELECT * FROM jobs.target_group_members WHERE target_group_name = N'PoolGroup';
```

## <a name="deploy-new-schema-to-many-databases"></a>Déployer un nouveau schéma sur plusieurs bases de données

L’exemple suivant montre comment déployer le nouveau schéma sur toutes les bases de données.  
Se connecter à la [*base de données de travail*](job-automation-overview.md#job-database) et exécuter la commande suivante :

```sql
--Connect to the job database specified when creating the job agent

--Add job for create table
EXEC jobs.sp_add_job @job_name='CreateTableTest', @description='Create Table Test'

-- Add job step for create table
EXEC jobs.sp_add_jobstep @job_name='CreateTableTest',
@command=N'IF NOT EXISTS (SELECT * FROM sys.tables WHERE object_id = object_id(''Test''))
CREATE TABLE [dbo].[Test]([TestId] [int] NOT NULL);',
@credential_name='myjobcred',
@target_group_name='PoolGroup'
```

## <a name="data-collection-using-built-in-parameters"></a>Collecte de données à l’aide de paramètres intégrés

Dans de nombreux scénarios de collecte de données, il peut être utile d’inclure certaines de ces variables de script pour favoriser le post-traitement des résultats du travail.

- $(job_name)
- $(job_id)
- $(job_version)
- $(step_id)
- $(step_name)
- $(job_execution_id)
- $(job_execution_create_time)
- $(target_group_name)

Par exemple, pour regrouper tous les résultats de l’exécution d’un même travail, utilisez *$(job_execution_id)* comme indiqué dans la commande suivante :

```sql
@command= N' SELECT DB_NAME() DatabaseName, $(job_execution_id) AS job_execution_id, * FROM sys.dm_db_resource_stats WHERE end_time > DATEADD(mi, -20, GETDATE());'
```

## <a name="monitor-database-performance"></a>Surveiller les performances de la base de données

L’exemple suivant crée un nouveau travail pour collecter des données de performances de plusieurs bases de données.

Par défaut, l’agent de travail crée la table de sortie dans laquelle sont stockés les résultats retournés. Le principal de base de données associé aux informations d’identification de sortie doit donc disposer au minimum des autorisations suivantes : `CREATE TABLE` sur la base de données, `ALTER`, `SELECT`, `INSERT` et `DELETE` sur la table de sortie ou son schéma et `SELECT` sur la vue de catalogue [sys.indexes](/sql/relational-databases/system-catalog-views/sys-indexes-transact-sql).

Si vous souhaitez créer manuellement la table à l’avance, elle doit avoir les propriétés suivantes :

1. Colonnes avec le nom et les types de données corrects pour le jeu de résultats.
2. Colonne supplémentaire pour internal_execution_id avec le type de données uniqueidentifier.
3. Un index non cluster nommé `IX_<TableName>_Internal_Execution_ID` dans la colonne internal_execution_id.
4. Toutes les autorisations listées ci-dessus, à l’exception de l’autorisation `CREATE TABLE` sur la base de données.

Se connecter à la [*base de données de travail*](job-automation-overview.md#job-database) et exécuter les commandes suivantes :

```sql
--Connect to the job database specified when creating the job agent

-- Add a job to collect perf results
EXEC jobs.sp_add_job @job_name ='ResultsJob', @description='Collection Performance data from all customers'

-- Add a job step w/ schedule to collect results
EXEC jobs.sp_add_jobstep
@job_name='ResultsJob',
@command= N' SELECT DB_NAME() DatabaseName, $(job_execution_id) AS job_execution_id, * FROM sys.dm_db_resource_stats WHERE end_time > DATEADD(mi, -20, GETDATE());',
@credential_name='myjobcred',
@target_group_name='PoolGroup',
@output_type='SqlDatabase',
@output_credential_name='myjobcred',
@output_server_name='server1.database.windows.net',
@output_database_name='<resultsdb>',
@output_table_name='<resutlstable>'
Create a job to monitor pool performance
--Connect to the job database specified when creating the job agent

-- Add a target group containing master database
EXEC jobs.sp_add_target_group 'MasterGroup'

-- Add a server target member
EXEC jobs.sp_add_target_group_member
@target_group_name='MasterGroup',
@target_type='SqlDatabase',
@server_name='server1.database.windows.net',
@database_name='master'

-- Add a job to collect perf results
EXEC jobs.sp_add_job
@job_name='ResultsPoolsJob',
@description='Demo: Collection Performance data from all pools',
@schedule_interval_type='Minutes',
@schedule_interval_count=15

-- Add a job step w/ schedule to collect results
EXEC jobs.sp_add_jobstep
@job_name='ResultsPoolsJob',
@command=N'declare @now datetime
DECLARE @startTime datetime
DECLARE @endTime datetime
DECLARE @poolLagMinutes datetime
DECLARE @poolStartTime datetime
DECLARE @poolEndTime datetime
SELECT @now = getutcdate ()
SELECT @startTime = dateadd(minute, -15, @now)
SELECT @endTime = @now
SELECT @poolStartTime = dateadd(minute, -30, @startTime)
SELECT @poolEndTime = dateadd(minute, -30, @endTime)

SELECT elastic_pool_name , end_time, elastic_pool_dtu_limit, avg_cpu_percent, avg_data_io_percent, avg_log_write_percent, max_worker_percent, max_session_percent,
        avg_storage_percent, elastic_pool_storage_limit_mb FROM sys.elastic_pool_resource_stats
        WHERE end_time > @poolStartTime and end_time <= @poolEndTime;
'),
@credential_name='myjobcred',
@target_group_name='MasterGroup',
@output_type='SqlDatabase',
@output_credential_name='myjobcred',
@output_server_name='server1.database.windows.net',
@output_database_name='resultsdb',
@output_table_name='resutlstable'
```

## <a name="view-job-definitions"></a>Afficher les définitions des travaux

L’exemple suivant montre comment afficher les définitions du travail actuel.  
Se connecter à la [*base de données de travail*](job-automation-overview.md#job-database) et exécuter la commande suivante :

```sql
--Connect to the job database specified when creating the job agent

-- View all jobs
SELECT * FROM jobs.jobs

-- View the steps of the current version of all jobs
SELECT js.* FROM jobs.jobsteps js
JOIN jobs.jobs j
  ON j.job_id = js.job_id AND j.job_version = js.job_version

-- View the steps of all versions of all jobs
select * from jobs.jobsteps
```

## <a name="begin-unplanned-execution-of-a-job"></a>Commencer l’exécution non planifiée d’un travail

L’exemple suivant montre comment démarrer un travail immédiatement.  
Se connecter à la [*base de données de travail*](job-automation-overview.md#job-database) et exécuter la commande suivante :

```sql
--Connect to the job database specified when creating the job agent

-- Execute the latest version of a job
EXEC jobs.sp_start_job 'CreateTableTest'

-- Execute the latest version of a job and receive the execution id
declare @je uniqueidentifier
exec jobs.sp_start_job 'CreateTableTest', @job_execution_id = @je output
select @je

select * from jobs.job_executions where job_execution_id = @je

-- Execute a specific version of a job (e.g. version 1)
exec jobs.sp_start_job 'CreateTableTest', 1
```

## <a name="schedule-execution-of-a-job"></a>Planifier l’exécution d’un travail

L’exemple ci-dessous montre comment planifier un travail pour une exécution future.  
Se connecter à la [*base de données de travail*](job-automation-overview.md#job-database) et exécuter la commande suivante :

```sql
--Connect to the job database specified when creating the job agent

EXEC jobs.sp_update_job
@job_name='ResultsJob',
@enabled=1,
@schedule_interval_type='Minutes',
@schedule_interval_count=15
```

## <a name="monitor-job-execution-status"></a>Surveiller l’état d’exécution du travail

L’exemple suivant montre comment afficher les détails de l’état d’exécution de tous les travaux.  
Se connecter à la [*base de données de travail*](job-automation-overview.md#job-database) et exécuter la commande suivante :

```sql
--Connect to the job database specified when creating the job agent

--View top-level execution status for the job named 'ResultsPoolJob'
SELECT * FROM jobs.job_executions
WHERE job_name = 'ResultsPoolsJob' and step_id IS NULL
ORDER BY start_time DESC

--View all top-level execution status for all jobs
SELECT * FROM jobs.job_executions WHERE step_id IS NULL
ORDER BY start_time DESC

--View all execution statuses for job named 'ResultsPoolsJob'
SELECT * FROM jobs.job_executions
WHERE job_name = 'ResultsPoolsJob'
ORDER BY start_time DESC

-- View all active executions
SELECT * FROM jobs.job_executions
WHERE is_active = 1
ORDER BY start_time DESC
```

## <a name="cancel-a-job"></a>Annulation d’une tâche

L’exemple suivant montre comment annuler un travail.  
Se connecter à la [*base de données de travail*](job-automation-overview.md#job-database) et exécuter la commande suivante :

```sql
--Connect to the job database specified when creating the job agent

-- View all active executions to determine job execution id
SELECT * FROM jobs.job_executions
WHERE is_active = 1 AND job_name = 'ResultPoolsJob'
ORDER BY start_time DESC
GO

-- Cancel job execution with the specified job execution id
EXEC jobs.sp_stop_job '01234567-89ab-cdef-0123-456789abcdef'
```

## <a name="delete-old-job-history"></a>Supprimer l’historique d’un ancien travail

L’exemple suivant montre comment supprimer l’historique d’un travail avant une date spécifique.  
Se connecter à la [*base de données de travail*](job-automation-overview.md#job-database) et exécuter la commande suivante :

```sql
--Connect to the job database specified when creating the job agent

-- Delete history of a specific job’s executions older than the specified date
EXEC jobs.sp_purge_jobhistory @job_name='ResultPoolsJob', @oldest_date='2016-07-01 00:00:00'

--Note: job history is automatically deleted if it is >45 days old
```

## <a name="delete-a-job-and-all-its-job-history"></a>Supprimer un travail et tous ses historiques

L’exemple suivant montre comment supprimer un travail et tout l’historique qui s’y rapporte.  
Se connecter à la [*base de données de travail*](job-automation-overview.md#job-database) et exécuter la commande suivante :

```sql
--Connect to the job database specified when creating the job agent

EXEC jobs.sp_delete_job @job_name='ResultsPoolsJob'

--Note: job history is automatically deleted if it is >45 days old
```

## <a name="job-stored-procedures"></a>Procédures de travail stockées

Les procédures stockées suivantes se trouvent dans la [base de données des travaux](job-automation-overview.md#job-database).

|Procédure stockée  |Description  |
|---------|---------|
|[sp_add_job](#sp_add_job)     |     Ajoute un nouveau travail.    |
|[sp_update_job](#sp_update_job)    |      Met à jour un travail existant.   |
|[sp_delete_job](#sp_delete_job)     |      Supprime un travail existant.   |
|[sp_add_jobstep](#sp_add_jobstep)    |    Ajoute une étape à un travail.     |
|[sp_update_jobstep](#sp_update_jobstep)     |     Met à jour une étape de travail.    |
|[sp_delete_jobstep](#sp_delete_jobstep)     |     Supprime une étape de travail.    |
|[sp_start_job](#sp_start_job)    |  Démarre l’exécution d’un travail.       |
|[sp_stop_job](#sp_stop_job)     |     Arrête l’exécution d’un travail.   |
|[sp_add_target_group](#sp_add_target_group)    |     Ajoute un groupe cible.    |
|[sp_delete_target_group](#sp_delete_target_group)     |    Supprime un groupe cible.     |
|[sp_add_target_group_member](#sp_add_target_group_member)     |    Ajoute une base de données ou un groupe de bases de données à un groupe cible.     |
|[sp_delete_target_group_member](#sp_delete_target_group_member)     |     Supprime un membre du groupe cible d’un groupe cible.    |
|[sp_purge_jobhistory](#sp_purge_jobhistory)    |    Supprime les enregistrements d'historique d'un travail.     |

### <a name="sp_add_job"></a><a name="sp_add_job"></a>sp_add_job

Ajoute un nouveau travail.
  
#### <a name="syntax"></a>Syntaxe  
  
```syntaxsql
[jobs].sp_add_job [ @job_name = ] 'job_name'  
  [ , [ @description = ] 'description' ]
  [ , [ @enabled = ] enabled ]
  [ , [ @schedule_interval_type = ] schedule_interval_type ]  
  [ , [ @schedule_interval_count = ] schedule_interval_count ]
  [ , [ @schedule_start_time = ] schedule_start_time ]
  [ , [ @schedule_end_time = ] schedule_end_time ]
  [ , [ @job_id = ] job_id OUTPUT ]
```

#### <a name="arguments"></a>Arguments  

[ **\@job_name =** ] 'job_name'  
Nom du travail. Le nom doit être unique et ne peut pas contenir le caractère de pourcentage (%). job_name est nvarchar (128), sans valeur par défaut.

[ **\@description =** ] 'description'  
Description du travail. description est nvarchar(512), avec une valeur par défaut NULL. Si la description est omise, une chaîne vide est utilisée.

[ **\@enabled =** ] enabled  
Spécifie si la planification du travail est activée. Activée est de type bit, avec une valeur par défaut 0 (désactivé). Si 0, le travail n’est pas activé et ne s’exécute pas en fonction de sa planification. Il peut toutefois être exécuté manuellement. Si 1, le travail s’exécutera conformément à sa planification et peut également être exécuté manuellement.

[ **\@schedule_interval_type =** ] schedule_interval_type  
La valeur indique quand le travail doit être exécuté. schedule_interval_type est nvarchar (50), avec une valeur par défaut Une fois et peut prendre l’une des valeurs suivantes :

- « Une fois »,
- « Minutes »,
- « Heures »,
- « Jours »,
- « Semaines »,
- « Mois »

[ **\@schedule_interval_count =** ] schedule_interval_count  
Nombre de périodes schedule_interval_count entre chaque exécution du travail. schedule_interval_count est de type int, avec 1 comme valeur par défaut. La valeur doit être supérieure ou égale à 1.

[ **\@schedule_start_time =** ] schedule_start_time  
Date à laquelle l'exécution du travail peut commencer. schedule_start_time est DATETIME2, avec 0001-01-01 00:00:00.0000000 comme valeur par défaut.

[ **\@schedule_end_time =** ] schedule_end_time  
Date à laquelle l'exécution du travail peut s'arrêter. schedule_end_time is DATETIME2, avec 9999-12-31 11:59:59.0000000 comme valeur par défaut.

[ **\@job_id =** ] job_id OUTPUT  
Numéro d'identification du travail affecté si le travail est correctement créé. job_id est une variable de sortie de type uniqueidentifier.

#### <a name="return-code-values"></a>Codet de retour

0 (réussite) ou 1 (échec)

#### <a name="remarks"></a>Notes

sp_add_job doit être exécuté à partir de la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail.
Une fois que sp_add_job a été exécutée pour ajouter un travail, sp_add_jobstep peut servir à ajouter des étapes qui effectuent les activités pour le travail. Le numéro de version initiale du travail est 0 et passera à 1 lors de l’ajout de la première étape.

#### <a name="permissions"></a>Autorisations

Par défaut, les membres du rôle serveur fixe sysadmin peuvent exécuter cette procédure stockée. L’utilisateur peut uniquement superviser les travaux. Vous pouvez l’autoriser à faire partie du rôle de base de données suivant dans la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail :

- jobs_reader

Pour plus d’informations sur les autorisations de ces rôles, consultez la section Autorisations dans ce document. Seuls des membres de sysadmin peuvent utiliser cette procédure stockée pour modifier les attributs des travaux qui appartiennent à d’autres utilisateurs.

### <a name="sp_update_job"></a><a name="sp_update_job"></a>sp_update_job

Met à jour un travail existant.

#### <a name="syntax"></a>Syntaxe

```syntaxsql
[jobs].sp_update_job [ @job_name = ] 'job_name'  
  [ , [ @new_name = ] 'new_name' ]
  [ , [ @description = ] 'description' ]
  [ , [ @enabled = ] enabled ]
  [ , [ @schedule_interval_type = ] schedule_interval_type ]  
  [ , [ @schedule_interval_count = ] schedule_interval_count ]
  [ , [ @schedule_start_time = ] schedule_start_time ]
  [ , [ @schedule_end_time = ] schedule_end_time ]
```

#### <a name="arguments"></a>Arguments

[ **\@job_name =** ] 'job_name'  
Nom du travail à mettre à jour. job_name est nvarchar (128).

[ **\@new_name =** ] 'new_name'  
Nouveau nom du travail. new_name est nvarchar(128).

[ **\@description =** ] 'description'  
Description du travail. la description est nvarchar(512).

[ **\@enabled =** ] enabled  
Spécifie si la planification du travail est activée (1) ou désactivée (0). Activée est de type bit.

[ **\@schedule_interval_type=** ] schedule_interval_type  
La valeur indique quand le travail doit être exécuté. schedule_interval_type est nvarchar (50) et peut prendre l’une des valeurs suivantes :

- « Une fois »,
- « Minutes »,
- « Heures »,
- « Jours »,
- « Semaines »,
- « Mois »

[ **\@schedule_interval_count=** ] schedule_interval_count  
Nombre de périodes schedule_interval_count entre chaque exécution du travail. schedule_interval_count est de type int, avec 1 comme valeur par défaut. La valeur doit être supérieure ou égale à 1.

[ **\@schedule_start_time=** ] schedule_start_time  
Date à laquelle l'exécution du travail peut commencer. schedule_start_time est DATETIME2, avec 0001-01-01 00:00:00.0000000 comme valeur par défaut.

[ **\@schedule_end_time=** ] schedule_end_time  
Date à laquelle l'exécution du travail peut s'arrêter. schedule_end_time is DATETIME2, avec 9999-12-31 11:59:59.0000000 comme valeur par défaut.

#### <a name="return-code-values"></a>Codet de retour

0 (réussite) ou 1 (échec)

#### <a name="remarks"></a>Notes

Une fois que sp_add_job a été exécutée pour ajouter un travail, sp_add_jobstep peut servir à ajouter des étapes qui effectuent les activités pour le travail. Le numéro de version initiale du travail est 0 et passera à 1 lors de l’ajout de la première étape.

#### <a name="permissions"></a>Autorisations

Par défaut, les membres du rôle serveur fixe sysadmin peuvent exécuter cette procédure stockée. L’utilisateur peut uniquement superviser les travaux. Vous pouvez l’autoriser à faire partie du rôle de base de données suivant dans la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail :

- jobs_reader

Pour plus d’informations sur les autorisations de ces rôles, consultez la section Autorisations dans ce document. Seuls des membres de sysadmin peuvent utiliser cette procédure stockée pour modifier les attributs des travaux qui appartiennent à d’autres utilisateurs.

### <a name="sp_delete_job"></a><a name="sp_delete_job"></a>sp_delete_job

Supprime un travail existant.

#### <a name="syntax"></a>Syntaxe

```syntaxsql
[jobs].sp_delete_job [ @job_name = ] 'job_name'
  [ , [ @force = ] force ]
```

#### <a name="arguments"></a>Arguments

[ **\@job_name =** ] 'job_name'  
Nom du travail à supprimer. job_name est nvarchar (128).

[ **\@force =** ] force  
Spécifie s’il faut supprimer si le travail est en cours d’exécution et supprimer toutes les exécutions en cours (1) ou faire échouer si toutes les exécutions de travaux sont en cours (0). force est de type bit.

#### <a name="return-code-values"></a>Codet de retour

0 (réussite) ou 1 (échec)

#### <a name="remarks"></a>Notes

L’historique du travail est automatiquement supprimé lorsqu’un travail est supprimé.

#### <a name="permissions"></a>Autorisations

Par défaut, les membres du rôle serveur fixe sysadmin peuvent exécuter cette procédure stockée. L’utilisateur peut uniquement superviser les travaux. Vous pouvez l’autoriser à faire partie du rôle de base de données suivant dans la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail :

- jobs_reader

Pour plus d’informations sur les autorisations de ces rôles, consultez la section Autorisations dans ce document. Seuls des membres de sysadmin peuvent utiliser cette procédure stockée pour modifier les attributs des travaux qui appartiennent à d’autres utilisateurs.

### <a name="sp_add_jobstep"></a><a name="sp_add_jobstep"></a>sp_add_jobstep

Ajoute une étape à un travail.

#### <a name="syntax"></a>Syntaxe

```syntaxsql
[jobs].sp_add_jobstep [ @job_name = ] 'job_name'
     [ , [ @step_id = ] step_id ]
     [ , [ @step_name = ] step_name ]
     [ , [ @command_type = ] 'command_type' ]
     [ , [ @command_source = ] 'command_source' ]  
     , [ @command = ] 'command'
     , [ @credential_name = ] 'credential_name'
     , [ @target_group_name = ] 'target_group_name'
     [ , [ @initial_retry_interval_seconds = ] initial_retry_interval_seconds ]
     [ , [ @maximum_retry_interval_seconds = ] maximum_retry_interval_seconds ]
     [ , [ @retry_interval_backoff_multiplier = ] retry_interval_backoff_multiplier ]
     [ , [ @retry_attempts = ] retry_attempts ]
     [ , [ @step_timeout_seconds = ] step_timeout_seconds ]
     [ , [ @output_type = ] 'output_type' ]
     [ , [ @output_credential_name = ] 'output_credential_name' ]
     [ , [ @output_subscription_id = ] 'output_subscription_id' ]
     [ , [ @output_resource_group_name = ] 'output_resource_group_name' ]
     [ , [ @output_server_name = ] 'output_server_name' ]
     [ , [ @output_database_name = ] 'output_database_name' ]
     [ , [ @output_schema_name = ] 'output_schema_name' ]
     [ , [ @output_table_name = ] 'output_table_name' ]
     [ , [ @job_version = ] job_version OUTPUT ]
     [ , [ @max_parallelism = ] max_parallelism ]
```

#### <a name="arguments"></a>Arguments

[ **\@job_name =** ] 'job_name'  
Nom du travail auquel ajouter l'étape. job_name est nvarchar (128).

[ **\@step_id =** ] step_id  
Numéro d'identification de la séquence de l'étape de travail. Les numéros d’identification d’étape commencent à 1 et augmentent sans intervalle. Si une étape existante a déjà cet ID, alors cette étape et toutes les étapes suivantes verront leur ID incrémenté afin que cette nouvelle étape puisse être insérée dans la séquence. Si rien n’est spécifié, +step_id sera automatiquement attribué à la dernière des séquences d’étapes. step_id est de type int.

[ **\@step_name =** ] step_name  
Nom de l'étape. Doit être spécifié, à l’exception de la première étape d’un travail, qui a le nom « JobStep » par défaut (pour plus de commodité). step_name est nvarchar(128).

[ **\@command_type =** ] 'command_type'  
Le type de commande exécuté par cette étape de travail. command_type est nvarchar(50), avec TSql comme valeur par défaut, ce qui signifie que la valeur du paramètre @command_type est un script T-SQL.

S’il est spécifié, la valeur doit être TSql.

[ **\@command_source =** ] 'command_source'  
Type d’emplacement de stockage de la commande. command_source est nvarchar(50), avec Inclus comme valeur par défaut, ce qui signifie que la valeur du paramètre @command_source est le texte littéral de la commande.

S’il est spécifié, le paramètre doit être inclus.

[ **\@command =** ] 'command'  
La commande doit être un script T-SQL valide. Elle est alors exécutée par cette étape de travail. la commande est nvarchar(max), avec NULL comme valeur par défaut.

[ **\@credential_name =** ] 'credential_name'  
Nom des informations d’identification incluses dans l’étendue de la base de données et utilisées pour se connecter à chacune des bases de données cibles au sein du groupe cible lors de l’exécution de cette étape. credential_name est nvarchar(128).

[ **\@target_group_name =** ] 'target-group_name'  
Nom du groupe cible qui contient les bases de données cibles sur lesquelles l’étape de travail sera exécutée. target_group_name est nvarchar(128).

[ **\@initial_retry_interval_seconds =** ] initial_retry_interval_seconds  
Le délai avant la première nouvelle tentative, si l’étape de travail échoue lors de la tentative d’exécution initiale. initial_retry_interval_seconds est de type int, avec 1 comme valeur par défaut.

[ **\@maximum_retry_interval_seconds =** ] maximum_retry_interval_seconds  
Délai maximal entre les nouvelles tentatives. Si le délai entre deux tentatives dépasse cette valeur, il est limité à cette valeur à la place. maximum_retry_interval_seconds est de type int, avec 120 comme valeur par défaut.

[ **\@retry_interval_backoff_multiplier =** ] retry_interval_backoff_multiplier  
Le multiplicateur à appliquer au délai de nouvelle tentative si plusieurs tentatives d’exécution d’une étape de travail échouent. Par exemple, si la première nouvelle tentative avait un délai de 5 secondes et que le multiplicateur d’interruption est 2,0, la deuxième tentative aura un délai de 10 secondes et la troisième un délai de 20 secondes. retry_interval_backoff_multiplier est de type real, avec une valeur par défaut de 2,0.

[ **\@retry_attempts =** ] retry_attempts  
Nombre de nouvelles tentatives d’exécution si la tentative initiale échoue. Par exemple, si la valeur retry_attempts est 10, il y aura 1 tentative initiale et 10 nouvelles tentatives, ce qui donne un total de 11 tentatives. Si la dernière tentative échoue, l’exécution du travail se terminera avec un cycle de vie Échec. retry_attempts est de type int, avec 10 comme valeur par défaut.

[ **\@step_timeout_seconds =** ] step_timeout_seconds  
La durée maximale autorisée pour l’étape à exécuter. Si cette durée est dépassée, l’exécution du travail se terminera par un cycle de vie TimedOut. step_timeout_seconds est de type int, avec 43 200 secondes (12 heures) comme valeur par défaut.

[ **\@output_type =** ] 'output_type'  
Si la valeur n’est pas null, le type de destination où le premier jeu de résultats de la commande est écrit. output_type est nvarchar(50), avec NULL comme valeur par défaut.

S’il est spécifié, la valeur doit être SqlDatabase.

[ **\@output_credential_name =** ] 'output_credential_name'  
Si la valeur n’est pas null, le nom des informations d’identification incluses dans la base de données qui sont utilisées pour se connecter à la base de données de destination de sortie. Doit être spécifié si output_type est égale à SqlDatabase. output_credential_name est nvarchar (128), avec NULL comme valeur par défaut.

[ **\@output_subscription_id =** ] 'output_subscription_id'  
Description nécessaire.

[ **\@output_resource_group_name =** ] 'output_resource_group_name'  
Description nécessaire.

[ **\@output_server_name =** ] 'output_server_name'  
Si la valeur n’est pas null, le nom DNS complet du serveur qui contient la base de données de destination de sortie. Doit être spécifié si output_type est égale à SqlDatabase. output_server_name est nvarchar(256), avec NULL comme valeur par défaut.

[ **\@output_database_name =** ] 'output_database_name'  
Si la valeur n’est pas null, le nom de la base de données qui contient la table de destination de sortie. Doit être spécifié si output_type est égale à SqlDatabase. output_database_name est nvarchar(128), avec NULL comme valeur par défaut.

[ **\@output_schema_name =** ] 'output_schema_name'  
Si la valeur n’est pas null, le nom du schéma SQL qui contient la table de destination de sortie. Si output_type est égal à SqlDatabase, la valeur par défaut est dbo. output_schema_name est nvarchar(128).

[ **\@output_table_name =** ] 'output_table_name'  
Si la valeur n’est pas null, le nom de la table où le premier jeu de résultats de la commande sera écrit. Si la table n’existe pas encore, elle sera créée selon le schéma du jeu de résultats retourné. Doit être spécifié si output_type est égale à SqlDatabase. output_table_name est nvarchar (128), avec NULL comme valeur par défaut.

[ **\@job_version =** ] job_version OUTPUT  
Paramètre de sortie auquel sera assigné le nouveau numéro de version de travail. job_version est de type int.

[ **\@max_parallelism =** ] max_parallelism OUTPUT  
Le niveau maximal de parallélisme par pool élastique. S’il est défini, l’étape de travail sera limitée à l’exécution sur un nombre maximum de ces nombreuses bases de données par pool élastique. Ceci s’applique à chaque pool élastique qui est soit directement inclus dans le groupe cible, ou se trouve à l’intérieur d’un serveur inclus dans le groupe cible. max_parallelism est de type int.

#### <a name="return-code-values"></a>Codet de retour

0 (réussite) ou 1 (échec)

#### <a name="remarks"></a>Notes

Lorsque sp_add_jobstep réussit, le numéro de version actuel du travail est incrémenté. La prochaine fois que le travail sera exécuté, la nouvelle version sera utilisée. Si le travail est en cours d’exécution, cette exécution ne contiendra pas la nouvelle étape.

#### <a name="permissions"></a>Autorisations

Par défaut, les membres du rôle serveur fixe sysadmin peuvent exécuter cette procédure stockée. L’utilisateur peut uniquement superviser les travaux. Vous pouvez l’autoriser à faire partie du rôle de base de données suivant dans la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail :  

- jobs_reader

Pour plus d’informations sur les autorisations de ces rôles, consultez la section Autorisations dans ce document. Seuls des membres de sysadmin peuvent utiliser cette procédure stockée pour modifier les attributs des travaux qui appartiennent à d’autres utilisateurs.

### <a name="sp_update_jobstep"></a><a name="sp_update_jobstep"></a>sp_update_jobstep

Met à jour une étape de travail.

#### <a name="syntax"></a>Syntaxe

```syntaxsql
[jobs].sp_update_jobstep [ @job_name = ] 'job_name'
     [ , [ @step_id = ] step_id ]
     [ , [ @step_name = ] 'step_name' ]
     [ , [ @new_id = ] new_id ]
     [ , [ @new_name = ] 'new_name' ]
     [ , [ @command_type = ] 'command_type' ]
     [ , [ @command_source = ] 'command_source' ]  
     , [ @command = ] 'command'
     , [ @credential_name = ] 'credential_name'
     , [ @target_group_name = ] 'target_group_name'
     [ , [ @initial_retry_interval_seconds = ] initial_retry_interval_seconds ]
     [ , [ @maximum_retry_interval_seconds = ] maximum_retry_interval_seconds ]
     [ , [ @retry_interval_backoff_multiplier = ] retry_interval_backoff_multiplier ]
     [ , [ @retry_attempts = ] retry_attempts ]
     [ , [ @step_timeout_seconds = ] step_timeout_seconds ]
     [ , [ @output_type = ] 'output_type' ]
     [ , [ @output_credential_name = ] 'output_credential_name' ]
     [ , [ @output_server_name = ] 'output_server_name' ]
     [ , [ @output_database_name = ] 'output_database_name' ]
     [ , [ @output_schema_name = ] 'output_schema_name' ]
     [ , [ @output_table_name = ] 'output_table_name' ]
     [ , [ @job_version = ] job_version OUTPUT ]
     [ , [ @max_parallelism = ] max_parallelism ]
```

#### <a name="arguments"></a>Arguments

[ **\@job_name =** ] 'job_name'  
Nom du travail auquel l'étape appartient. job_name est nvarchar (128).

[ **\@step_id =** ] step_id  
Numéro d'identification de l'étape d'un travail à modifier. step_id ou step_name doit être spécifié. step_id est de type int.

[ **\@step_name =** ] 'step_name'  
Nom de l’étape à modifier. step_id ou step_name doit être spécifié. step_name est nvarchar(128).

[ **\@new_id =** ] new_id  
Le nouveau numéro d’identification de séquence pour l’étape de travail. Les numéros d’identification d’étape commencent à 1 et augmentent sans intervalle. Si une étape est réorganisée, les autres étapes seront automatiquement renumérotées.

[ **\@new_name =** ] 'new_name'  
Nouveau nom de l'étape. new_name est nvarchar(128).

[ **\@command_type =** ] 'command_type'  
Le type de commande exécuté par cette étape de travail. command_type est nvarchar(50), avec TSql comme valeur par défaut, ce qui signifie que la valeur du paramètre @command_type est un script T-SQL.

S’il est spécifié, la valeur doit être TSql.

[ **\@command_source =** ] 'command_source'  
Type d’emplacement de stockage de la commande. command_source est nvarchar(50), avec Inclus comme valeur par défaut, ce qui signifie que la valeur du paramètre @command_source est le texte littéral de la commande.

S’il est spécifié, le paramètre doit être inclus.

[ **\@command =** ] 'command'  
Les commandes doivent être un script T-SQL valide. Elles sont alors exécutées par cette étape de travail. la commande est nvarchar(max), avec NULL comme valeur par défaut.

[ **\@credential_name =** ] 'credential_name'  
Nom des informations d’identification incluses dans l’étendue de la base de données et utilisées pour se connecter à chacune des bases de données cibles au sein du groupe cible lors de l’exécution de cette étape. credential_name est nvarchar(128).

[ **\@target_group_name =** ] 'target-group_name'  
Nom du groupe cible qui contient les bases de données cibles sur lesquelles l’étape de travail sera exécutée. target_group_name est nvarchar(128).

[ **\@initial_retry_interval_seconds =** ] initial_retry_interval_seconds  
Le délai avant la première nouvelle tentative, si l’étape de travail échoue lors de la tentative d’exécution initiale. initial_retry_interval_seconds est de type int, avec 1 comme valeur par défaut.

[ **\@maximum_retry_interval_seconds =** ] maximum_retry_interval_seconds  
Délai maximal entre les nouvelles tentatives. Si le délai entre deux tentatives dépasse cette valeur, il est limité à cette valeur à la place. maximum_retry_interval_seconds est de type int, avec 120 comme valeur par défaut.

[ **\@retry_interval_backoff_multiplier =** ] retry_interval_backoff_multiplier  
Le multiplicateur à appliquer au délai de nouvelle tentative si plusieurs tentatives d’exécution d’une étape de travail échouent. Par exemple, si la première nouvelle tentative avait un délai de 5 secondes et que le multiplicateur d’interruption est 2,0, la deuxième tentative aura un délai de 10 secondes et la troisième un délai de 20 secondes. retry_interval_backoff_multiplier est de type real, avec une valeur par défaut de 2,0.

[ **\@retry_attempts =** ] retry_attempts  
Nombre de nouvelles tentatives d’exécution si la tentative initiale échoue. Par exemple, si la valeur retry_attempts est 10, il y aura 1 tentative initiale et 10 nouvelles tentatives, ce qui donne un total de 11 tentatives. Si la dernière tentative échoue, l’exécution du travail se terminera avec un cycle de vie Échec. retry_attempts est de type int, avec 10 comme valeur par défaut.

[ **\@step_timeout_seconds =** ] step_timeout_seconds  
La durée maximale autorisée pour l’étape à exécuter. Si cette durée est dépassée, l’exécution du travail se terminera par un cycle de vie TimedOut. step_timeout_seconds est de type int, avec 43 200 secondes (12 heures) comme valeur par défaut.

[ **\@output_type =** ] 'output_type'  
Si la valeur n’est pas null, le type de destination où le premier jeu de résultats de la commande est écrit. Pour réinitialiser la valeur d’output_type back sur NULL, définir la valeur de ce paramètre sur '' (chaîne vide). output_type est nvarchar(50), avec NULL comme valeur par défaut.

S’il est spécifié, la valeur doit être SqlDatabase.

[ **\@output_credential_name =** ] 'output_credential_name'  
Si la valeur n’est pas null, le nom des informations d’identification incluses dans la base de données qui sont utilisées pour se connecter à la base de données de destination de sortie. Doit être spécifié si output_type est égale à SqlDatabase. Pour réinitialiser la valeur d’output_credential_name sur NULL, définir la valeur de ce paramètre sur '' (chaîne vide). output_credential_name est nvarchar (128), avec NULL comme valeur par défaut.

[ **\@output_server_name =** ] 'output_server_name'  
Si la valeur n’est pas null, le nom DNS complet du serveur qui contient la base de données de destination de sortie. Doit être spécifié si output_type est égale à SqlDatabase. Pour réinitialiser la valeur d’output_server_name sur NULL, définir la valeur de ce paramètre sur '' (chaîne vide). output_server_name est nvarchar(256), avec NULL comme valeur par défaut.

[ **\@output_database_name =** ] 'output_database_name'  
Si la valeur n’est pas null, le nom de la base de données qui contient la table de destination de sortie. Doit être spécifié si output_type est égale à SqlDatabase. Pour réinitialiser la valeur d’output_database_name sur NULL, définir la valeur de ce paramètre sur '' (chaîne vide). output_database_name est nvarchar(128), avec NULL comme valeur par défaut.

[ **\@output_schema_name =** ] 'output_schema_name'  
Si la valeur n’est pas null, le nom du schéma SQL qui contient la table de destination de sortie. Si output_type est égal à SqlDatabase, la valeur par défaut est dbo. Pour réinitialiser la valeur d’output_server_schema sur NULL, définir la valeur de ce paramètre sur '' (chaîne vide). output_schema_name est nvarchar(128).

[ **\@output_table_name =** ] 'output_table_name'  
Si la valeur n’est pas null, le nom de la table où le premier jeu de résultats de la commande sera écrit. Si la table n’existe pas encore, elle sera créée selon le schéma du jeu de résultats retourné. Doit être spécifié si output_type est égale à SqlDatabase. Pour réinitialiser la valeur d’output_server_name sur NULL, définir la valeur de ce paramètre sur '' (chaîne vide). output_table_name est nvarchar (128), avec NULL comme valeur par défaut.

[ **\@job_version =** ] job_version OUTPUT  
Paramètre de sortie auquel sera assigné le nouveau numéro de version de travail. job_version est de type int.

[ **\@max_parallelism =** ] max_parallelism OUTPUT  
Le niveau maximal de parallélisme par pool élastique. S’il est défini, l’étape de travail sera limitée à l’exécution sur un nombre maximum de ces nombreuses bases de données par pool élastique. Ceci s’applique à chaque pool élastique qui est soit directement inclus dans le groupe cible, ou se trouve à l’intérieur d’un serveur inclus dans le groupe cible. Pour réinitialiser la valeur de max_parallelism sur null, définir la valeur de ce paramètre sur -1. max_parallelism est de type int.

#### <a name="return-code-values"></a>Codet de retour

0 (réussite) ou 1 (échec)

#### <a name="remarks"></a>Notes

Les exécutions en cours du travail ne seront pas affectées. Lorsque sp_update_jobstep réussit, le numéro de version du travail est incrémenté. La prochaine fois que le travail sera exécuté, la nouvelle version sera utilisée.

#### <a name="permissions"></a>Autorisations

Par défaut, les membres du rôle serveur fixe sysadmin peuvent exécuter cette procédure stockée. L’utilisateur peut uniquement superviser les travaux. Vous pouvez l’autoriser à faire partie du rôle de base de données suivant dans la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail :

- jobs_reader

Pour plus d’informations sur les autorisations de ces rôles, consultez la section Autorisations dans ce document. Seuls des membres de sysadmin peuvent utiliser cette procédure stockée pour modifier les attributs des travaux qui appartiennent à d’autres utilisateurs.

### <a name="sp_delete_jobstep"></a><a name="sp_delete_jobstep"></a>sp_delete_jobstep

Supprime une étape d'un travail.

#### <a name="syntax"></a>Syntaxe

```syntaxsql
[jobs].sp_delete_jobstep [ @job_name = ] 'job_name'
     [ , [ @step_id = ] step_id ]
     [ , [ @step_name = ] 'step_name' ]
     [ , [ @job_version = ] job_version OUTPUT ]
```

#### <a name="arguments"></a>Arguments

[ **\@job_name =** ] 'job_name'  
Nom du travail duquel il faut supprimer l'étape. job_name est nvarchar (128), sans valeur par défaut.

[ **\@step_id =** ] step_id  
Le numéro d’identification pour l’étape de travail à supprimer. step_id ou step_name doit être spécifié. step_id est de type int.

[ **\@step_name =** ] 'step_name'  
Nom de l’étape à supprimer. step_id ou step_name doit être spécifié. step_name est nvarchar(128).

[ **\@job_version =** ] job_version OUTPUT  
Paramètre de sortie auquel sera assigné le nouveau numéro de version de travail. job_version est de type int.

#### <a name="return-code-values"></a>Codet de retour

0 (réussite) ou 1 (échec)

#### <a name="remarks"></a>Notes

Les exécutions en cours du travail ne seront pas affectées. Lorsque sp_update_jobstep réussit, le numéro de version du travail est incrémenté. La prochaine fois que le travail sera exécuté, la nouvelle version sera utilisée.

Les autres étapes de travail seront automatiquement renumérotées pour combler le vide laissé par l’étape de travail supprimée.

#### <a name="permissions"></a>Autorisations

Par défaut, les membres du rôle serveur fixe sysadmin peuvent exécuter cette procédure stockée. L’utilisateur peut uniquement superviser les travaux. Vous pouvez l’autoriser à faire partie du rôle de base de données suivant dans la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail :

- jobs_reader

Pour plus d’informations sur les autorisations de ces rôles, consultez la section Autorisations dans ce document. Seuls des membres de sysadmin peuvent utiliser cette procédure stockée pour modifier les attributs des travaux qui appartiennent à d’autres utilisateurs.

### <a name="sp_start_job"></a><a name="sp_start_job"></a>sp_start_job

Démarre l’exécution d’un travail.

#### <a name="syntax"></a>Syntaxe

```syntaxsql
[jobs].sp_start_job [ @job_name = ] 'job_name'
     [ , [ @job_execution_id = ] job_execution_id OUTPUT ]
```

#### <a name="arguments"></a>Arguments

[ **\@job_name =** ] 'job_name'  
Nom du travail duquel il faut supprimer l'étape. job_name est nvarchar (128), sans valeur par défaut.

[ **\@job_execution_id =** ] job_execution_id OUTPUT  
Paramètre de sortie auquel sera assigné l’ID de l’exécution du travail. job_version est de type uniqueidentifier.

#### <a name="return-code-values"></a>Codet de retour

0 (réussite) ou 1 (échec)

#### <a name="remarks"></a>Notes

Aucun.

#### <a name="permissions"></a>Autorisations

Par défaut, les membres du rôle serveur fixe sysadmin peuvent exécuter cette procédure stockée. L’utilisateur peut uniquement superviser les travaux. Vous pouvez l’autoriser à faire partie du rôle de base de données suivant dans la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail :

- jobs_reader

Pour plus d’informations sur les autorisations de ces rôles, consultez la section Autorisations dans ce document. Seuls des membres de sysadmin peuvent utiliser cette procédure stockée pour modifier les attributs des travaux qui appartiennent à d’autres utilisateurs.

### <a name="sp_stop_job"></a><a name="sp_stop_job"></a>sp_stop_job

Arrête l’exécution d’un travail.

#### <a name="syntax"></a>Syntaxe

```syntaxsql
[jobs].sp_stop_job [ @job_execution_id = ] ' job_execution_id '
```

#### <a name="arguments"></a>Arguments

[ **\@job_execution_id =** ] job_execution_id  
Numéro d’identification de l’exécution du travail à arrêter. job_execution_id est de type uniqueidentifier, avec NULL comme valeur par défaut.

#### <a name="return-code-values"></a>Codet de retour

0 (réussite) ou 1 (échec)

#### <a name="remarks"></a>Notes

Aucun.

#### <a name="permissions"></a>Autorisations

Par défaut, les membres du rôle serveur fixe sysadmin peuvent exécuter cette procédure stockée. L’utilisateur peut uniquement superviser les travaux. Vous pouvez l’autoriser à faire partie du rôle de base de données suivant dans la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail :

- jobs_reader

Pour plus d’informations sur les autorisations de ces rôles, consultez la section Autorisations dans ce document. Seuls des membres de sysadmin peuvent utiliser cette procédure stockée pour modifier les attributs des travaux qui appartiennent à d’autres utilisateurs.

### <a name="sp_add_target_group"></a><a name="sp_add_target_group"></a>sp_add_target_group

Ajoute un groupe cible.

#### <a name="syntax"></a>Syntaxe

```syntaxsql
[jobs].sp_add_target_group [ @target_group_name = ] 'target_group_name'
     [ , [ @target_group_id = ] target_group_id OUTPUT ]
```

#### <a name="arguments"></a>Arguments

[ **\@target_group_name =** ] 'target_group_name'  
Nom du groupe cible à créer. target_group_name est nvarchar(128), sans valeur par défaut.

[ **\@target_group_id =** ] target_group_id OUTPUT Numéro d’identification du groupe cible affecté au travail s’il a été créé avec succès. target_group_id est une variable de sortie de type uniqueidentifier, avec NULL comme valeur par défaut.

#### <a name="return-code-values"></a>Codet de retour

0 (réussite) ou 1 (échec)

#### <a name="remarks"></a>Notes

Les groupes cibles permettent de cibler facilement un travail dans une collection de bases de données.

#### <a name="permissions"></a>Autorisations

Par défaut, les membres du rôle serveur fixe sysadmin peuvent exécuter cette procédure stockée. L’utilisateur peut uniquement superviser les travaux. Vous pouvez l’autoriser à faire partie du rôle de base de données suivant dans la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail :

- jobs_reader

Pour plus d’informations sur les autorisations de ces rôles, consultez la section Autorisations dans ce document. Seuls des membres de sysadmin peuvent utiliser cette procédure stockée pour modifier les attributs des travaux qui appartiennent à d’autres utilisateurs.

### <a name="sp_delete_target_group"></a><a name="sp_delete_target_group"></a>sp_delete_target_group

Supprime un groupe cible.

#### <a name="syntax"></a>Syntaxe

```syntaxsql
[jobs].sp_delete_target_group [ @target_group_name = ] 'target_group_name'
```

#### <a name="arguments"></a>Arguments

[ **\@target_group_name =** ] 'target_group_name'  
Nom du groupe cible à supprimer. target_group_name est nvarchar(128), sans valeur par défaut.

#### <a name="return-code-values"></a>Codet de retour

0 (réussite) ou 1 (échec)

#### <a name="remarks"></a>Notes

Aucun.

#### <a name="permissions"></a>Autorisations

Par défaut, les membres du rôle serveur fixe sysadmin peuvent exécuter cette procédure stockée. L’utilisateur peut uniquement superviser les travaux. Vous pouvez l’autoriser à faire partie du rôle de base de données suivant dans la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail :

- jobs_reader

Pour plus d’informations sur les autorisations de ces rôles, consultez la section Autorisations dans ce document. Seuls des membres de sysadmin peuvent utiliser cette procédure stockée pour modifier les attributs des travaux qui appartiennent à d’autres utilisateurs.

### <a name="sp_add_target_group_member"></a><a name="sp_add_target_group_member"></a>sp_add_target_group_member

Ajoute une base de données ou un groupe de bases de données à un groupe cible.

#### <a name="syntax"></a>Syntaxe

```syntaxsql
[jobs].sp_add_target_group_member [ @target_group_name = ] 'target_group_name'
         [ @membership_type = ] 'membership_type' ]
        [ , [ @target_type = ] 'target_type' ]
        [ , [ @refresh_credential_name = ] 'refresh_credential_name' ]
        [ , [ @server_name = ] 'server_name' ]
        [ , [ @database_name = ] 'database_name' ]
        [ , [ @elastic_pool_name = ] 'elastic_pool_name' ]
        [ , [ @shard_map_name = ] 'shard_map_name' ]
        [ , [ @target_id = ] 'target_id' OUTPUT ]
```

#### <a name="arguments"></a>Arguments

[ **\@target_group_name =** ] 'target_group_name'  
Nom du groupe cible auquel le membre sera ajouté. target_group_name est nvarchar(128), sans valeur par défaut.

[ **\@membership_type =** ] 'membership_type'  
Spécifie si le membre du groupe cible sera inclus ou exclu. target_group_name est nvarchar(128), avec ‘Include’ comme valeur par défaut. Les valeurs valides pour membership_type sont 'Include' ou 'Exclude'.

[ **\@target_type =** ] 'target_type'  
Type de base de données cible ou de collection de bases de données incluant toutes les bases de données dans un serveur, toutes les bases de données dans un pool élastique, toutes les bases de données dans un mappage de partition, ou une base de données individuelle. target_type est nvarchar(128), sans valeur par défaut. Les valeurs valides pour target_type sont 'SqlServer', 'SqlElasticPool', 'SqlDatabase' ou 'SqlShardMap'.

[ **\@refresh_credential_name =** ] 'refresh_credential_name'  
Le nom des informations d’identification incluses dans l’étendue de la base de données. refresh_credential_name est nvarchar(128), sans valeur par défaut.

[ **\@server_name =** ] 'server_name'  
Nom du serveur qui doit être ajouté au groupe cible spécifié. server_name doit être spécifié lorsque target_type est 'SqlServer'. server_name est nvarchar(128), sans valeur par défaut.

[ **\@database_name =** ] 'database_name'  
Nom de la base de données qui doit être ajoutée au groupe cible spécifié. database_name doit être spécifié lorsque target_type est 'SqlDatabase'. database_name est nvarchar (128), sans valeur par défaut.

[ **\@elastic_pool_name =** ] 'elastic_pool_name'  
Nom du pool élastique qui doit être ajouté au groupe cible spécifié. elastic_pool_name doit être spécifié lorsque target_type est ‘SqlElasticPool’. elastic_pool_name est nvarchar(128), sans valeur par défaut.

[ **\@shard_map_name =** ] 'shard_map_name'  
Nom de l’outil de mappage de partition qui doit être ajouté au groupe cible spécifié. elastic_pool_name doit être spécifié lorsque target_type est 'SqlShardMap'. shard_map_name est nvarchar (128), sans valeur par défaut.

[ **\@target_id =** ] target_group_id OUTPUT  
Numéro d’identification cible affecté au membre du groupe cible s’il a été créé et ajouté au groupe cible. target_id est une variable de sortie de type uniqueidentifier, avec NULL comme valeur par défaut.
Codet de retour 0 (réussite) ou 1 (échec)

#### <a name="remarks"></a>Notes

Un travail s’exécute sur toutes les bases de données uniques au sein d’un serveur ou dans un pool élastique au moment de l’exécution, lorsqu’un serveur ou un pool élastique est inclus dans le groupe cible.

#### <a name="permissions"></a>Autorisations

Par défaut, les membres du rôle serveur fixe sysadmin peuvent exécuter cette procédure stockée. L’utilisateur peut uniquement superviser les travaux. Vous pouvez l’autoriser à faire partie du rôle de base de données suivant dans la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail :

- jobs_reader

Pour plus d’informations sur les autorisations de ces rôles, consultez la section Autorisations dans ce document. Seuls des membres de sysadmin peuvent utiliser cette procédure stockée pour modifier les attributs des travaux qui appartiennent à d’autres utilisateurs.

#### <a name="examples"></a>Exemples

L’exemple suivant ajoute toutes les bases de données des serveurs de Londres et de New York aux groupes de serveurs qui informent les clients. Vous devez vous connecter à la base de données de travaux spécifiée lors de la création de l’agent de travail, dans ce cas ElasticJobs.

```sql
--Connect to the jobs database specified when creating the job agent
USE ElasticJobs ;
GO

-- Add a target group containing server(s)
EXEC jobs.sp_add_target_group @target_group_name =  N'Servers Maintaining Customer Information'
GO

-- Add a server target member
EXEC jobs.sp_add_target_group_member
@target_group_name = N'Servers Maintaining Customer Information',
@target_type = N'SqlServer',
@refresh_credential_name=N'mymastercred', --credential required to refresh the databases in server
@server_name=N'London.database.windows.net' ;
GO

-- Add a server target member
EXEC jobs.sp_add_target_group_member
@target_group_name = N'Servers Maintaining Customer Information',
@target_type = N'SqlServer',
@refresh_credential_name=N'mymastercred', --credential required to refresh the databases in server
@server_name=N'NewYork.database.windows.net' ;
GO

--View the recently added members to the target group
SELECT * FROM [jobs].target_group_members WHERE target_group_name= N'Servers Maintaining Customer Information';
GO
```

### <a name="sp_delete_target_group_member"></a><a name="sp_delete_target_group_member"></a>sp_delete_target_group_member

Supprime un membre du groupe cible d’un groupe cible.

#### <a name="syntax"></a>Syntaxe

```syntaxsql
[jobs].sp_delete_target_group_member [ @target_group_name = ] 'target_group_name'
   [ , [ @target_id = ] 'target_id']
```

#### <a name="arguments"></a>Arguments

[ @target_group_name = ] 'target_group_name'  
Nom du groupe cible à partir duquel supprimer le membre du groupe cible. target_group_name est nvarchar(128), sans valeur par défaut.

[ @target_id = ] target_id  
 Numéro d’identification cible affecté au membre du groupe cible. target_id est de type uniqueidentifier, avec NULL comme valeur par défaut.

#### <a name="return-code-values"></a>Codet de retour

0 (réussite) ou 1 (échec)

#### <a name="remarks"></a>Notes

Les groupes cibles permettent de cibler facilement un travail dans une collection de bases de données.

#### <a name="permissions"></a>Autorisations

Par défaut, les membres du rôle serveur fixe sysadmin peuvent exécuter cette procédure stockée. L’utilisateur peut uniquement superviser les travaux. Vous pouvez l’autoriser à faire partie du rôle de base de données suivant dans la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail :

- jobs_reader

Pour plus d’informations sur les autorisations de ces rôles, consultez la section Autorisations dans ce document. Seuls des membres de sysadmin peuvent utiliser cette procédure stockée pour modifier les attributs des travaux qui appartiennent à d’autres utilisateurs.

#### <a name="examples"></a>Exemples

L’exemple suivant supprime le serveur de Londres du groupe de serveurs qui informent les clients. Vous devez vous connecter à la base de données de travaux spécifiée lors de la création de l’agent de travail, dans ce cas ElasticJobs.

```sql
--Connect to the jobs database specified when creating the job agent
USE ElasticJobs ;
GO

-- Retrieve the target_id for a target_group_members
declare @tid uniqueidentifier
SELECT @tid = target_id FROM [jobs].target_group_members WHERE target_group_name = 'Servers Maintaining Customer Information' and server_name = 'London.database.windows.net'

-- Remove a target group member of type server
EXEC jobs.sp_delete_target_group_member
@target_group_name = N'Servers Maintaining Customer Information',
@target_id = @tid
GO
```

### <a name="sp_purge_jobhistory"></a><a name="sp_purge_jobhistory"></a>sp_purge_jobhistory

Supprime les enregistrements d'historique d'un travail.

#### <a name="syntax"></a>Syntaxe

```syntaxsql
[jobs].sp_purge_jobhistory [ @job_name = ] 'job_name'
      [ , [ @job_id = ] job_id ]
      [ , [ @oldest_date = ] oldest_date []
```

#### <a name="arguments"></a>Arguments

[ **\@job_name =** ] 'job_name'  
Nom du travail dont il faut supprimer les enregistrements d'historique. job_name est nvarchar(128), avec NULL comme valeur par défaut. Job_id ou job_name doit être spécifié, mais ne peut pas l’être.

[ **\@job_id =** ] job_id  
 Numéro d'identification du travail dont les enregistrements doivent être supprimés. job_id est de type uniqueidentifier, avec NULL comme valeur par défaut. Job_id ou job_name doit être spécifié, mais ne peut pas l’être.

[ **\@oldest_date =** ] oldest_date  
 Enregistrement le plus ancien à conserver dans l'historique. oldest_date est DATETIME2, avec NULL comme valeur par défaut. Lorsque l’argument oldest_date est spécifié, sp_purge_jobhistory supprime uniquement les enregistrements qui sont antérieurs à la valeur spécifiée.

#### <a name="return-code-values"></a>Codet de retour

0 (réussite) ou 1 (échec)

#### <a name="remarks"></a>Notes

Les groupes cibles permettent de cibler facilement un travail dans une collection de bases de données.

#### <a name="permissions"></a>Autorisations

Par défaut, les membres du rôle serveur fixe sysadmin peuvent exécuter cette procédure stockée. L’utilisateur peut uniquement superviser les travaux. Vous pouvez l’autoriser à faire partie du rôle de base de données suivant dans la base de données de l’agent de travail spécifiée lors de la création de l’agent de travail :

- jobs_reader

Pour plus d’informations sur les autorisations de ces rôles, consultez la section Autorisations dans ce document. Seuls des membres de sysadmin peuvent utiliser cette procédure stockée pour modifier les attributs des travaux qui appartiennent à d’autres utilisateurs.

#### <a name="examples"></a>Exemples

L’exemple suivant ajoute toutes les bases de données des serveurs de Londres et de New York aux groupes de serveurs qui informent les clients. Vous devez vous connecter à la base de données de travaux spécifiée lors de la création de l’agent de travail, dans ce cas ElasticJobs.

```sql
--Connect to the jobs database specified when creating the job agent

EXEC sp_delete_target_group_member
    @target_group_name = N'Servers Maintaining Customer Information',  
    @server_name = N'London.database.windows.net';  
GO
```

## <a name="job-views"></a>Affichages des travaux

Les vues suivantes sont disponibles dans la [base de données des travaux](job-automation-overview.md#job-database).

|Affichage  |Description  |
|---------|---------|
|[job_executions](#job_executions-view)     |  Afficher l'historique d'exécution des travaux.      |
|[jobs](#jobs-view)     |   Afficher tous les travaux.      |
|[job_versions](#job_versions-view)     |   Affiche toutes les versions du travail.      |
|[jobsteps](#jobsteps-view)     |     Affiche toutes les étapes dans la version actuelle de chaque travail.    |
|[jobstep_versions](#jobstep_versions-view)     |     Affiche toutes les étapes dans toutes les versions de chaque travail.    |
|[target_groups](#target_groups-view)     |      Affiche tous les groupes cibles.   |
|[target_group_members](#target_group_members-view)     |   Affiche tous les membres de tous les groupes cibles.      |

### <a name="job_executions-view"></a><a name="job_executions-view"></a>job_executions view

[jobs].[job_executions]

Afficher l'historique d'exécution des travaux.

|Nom de la colonne | Type de données | Description |
|---------|---------|---------|
|**job_execution_id** | UNIQUEIDENTIFIER | ID unique d’une instance de l’exécution d’un travail.
|**job_name** | nvarchar(128) | Nom du travail.
|**job_id** | UNIQUEIDENTIFIER | ID unique du travail.
|**job_version** | int | Version du travail (mise à jour automatique à chaque modification).
|**step_id** |int | Identificateur unique de cette étape (pour ce travail). NULL indique qu’il s’agit de l’exécution du travail parent.
|**is_active** | bit | Indique si les informations sont actives ou inactives. 1 indique les travaux actifs, et 0 indique qu’ils sont inactifs.
|**lifecycle** | nvarchar(50) | Valeur indiquant l’état du travail :‘Created’, ‘In Progress’, ‘Failed’, ‘Succeeded’, ‘Skipped’, ‘SucceededWithSkipped’|
|**create_time**| datetime2(7) | Date et heure de la création du travail.
|**heure-début** | datetime2(7) | Date et heure de début de l’exécution du travail. NULL si le travail n’a pas encore été exécuté.
|**heure-fin** | datetime2(7) | Date et heure de fin de l’exécution du travail. NULL si le travail n’a pas encore été exécuté, ou si son exécution n’est pas terminée.
|**current_attempts** | int | Nombre de tentatives pour l’étape. Travail parent sera égal à 0, les exécutions de travail enfant seront égales ou supérieures à 1 selon la stratégie d’exécution.
|**current_attempt_start_time** | datetime2(7) | Date et heure de début de l’exécution du travail. NULL indique qu’il s’agit de l’exécution du travail parent.
|**last_message** | nvarchar(max) | Message d'historique d'étape ou de travail.
|**target_type** | nvarchar(128) | Type de base de données cible ou de collection de bases de données incluant toutes les bases de données dans un serveur, toutes les bases de données dans un pool élastique, ou une base de données individuelle. Les valeurs valides pour target_type sont 'SqlServer', 'SqlElasticPool' ou 'SqlDatabase'. NULL indique qu’il s’agit de l’exécution du travail parent.
|**target_id** | UNIQUEIDENTIFIER | ID unique du membre du groupe cible.  NULL indique qu’il s’agit de l’exécution du travail parent.
|**target_group_name** | nvarchar(128) | Nom du groupe cible. NULL indique qu’il s’agit de l’exécution du travail parent.
|**target_server_name** | nvarchar(256)  | Nom du serveur contenu dans le groupe cible. Spécifié uniquement si target_type est 'SqlServer'. NULL indique qu’il s’agit de l’exécution du travail parent.
|**target_database_name** | nvarchar(128) | Nom de la base de données contenue dans le groupe cible. Spécifié uniquement lorsque target_type est ‘SqlDatabase’. NULL indique qu’il s’agit de l’exécution du travail parent.

### <a name="jobs-view"></a>affichage des tâches

[jobs].[jobs]

Afficher tous les travaux.

|Nom de la colonne | Type de données |Description|
|------|------|-------|
|**job_name** | nvarchar(128) | Nom du travail.|
|**job_id**| UNIQUEIDENTIFIER |ID unique du travail.|
|**job_version** |int |Version du travail (mise à jour automatique à chaque modification).|
|**description** |nvarchar(512)| Description du travail. Bit activé : indique si le travail est activé ou non. 1 indique les travaux sont activés, et 0 indique les travaux sont désactivés.|
|**schedule_interval_type**|nvarchar(50) |Valeur indiquant à quel moment le travail doit être exécuté : « Une fois », « Minutes », « Heures », 'Jours', « Semaines », « Mois »
|**schedule_interval_count**|int|Nombre de périodes schedule_interval_type entre chaque exécution du travail.|
|**schedule_start_time**|datetime2(7)|Date et heure de début de la dernière exécution du travail.|
|**schedule_end_time**|datetime2(7)|Date et heure de la dernière exécution du travail terminé.|

### <a name="job_versions-view"></a><a name="job_versions-view"></a>affichage de job_versions

[jobs].[job_versions]

Affiche toutes les versions du travail.

|Nom de la colonne|Type de données|Description|
|------|------|-------|
|**job_name**|nvarchar(128)|Nom du travail.|
|**job_id**|UNIQUEIDENTIFIER|ID unique du travail.|
|**job_version**|int|Version du travail (mise à jour automatique à chaque modification).|

### <a name="jobsteps-view"></a>affichage des étapes de travail

[jobs].[jobsteps]

Affiche toutes les étapes dans la version actuelle de chaque travail.

|Nom de la colonne|Type de données|Description|
|------|------|-------|
|**job_name**|nvarchar(128)|Nom du travail.|
|**job_id**|UNIQUEIDENTIFIER|ID unique du travail.|
|**job_version**|int|Version du travail (mise à jour automatique à chaque modification).|
|**step_id**|int|Identificateur unique de cette étape (pour ce travail).|
|**step_name**|nvarchar(128)|Nom unique (pour ce travail) de l’étape.|
|**command_type**|nvarchar(50)|Type de commande à exécuter dans l’étape de travail. Pour v1, la valeur doit être égale à et la valeur par défaut doit être ‘TSql’.|
|**command_source**|nvarchar(50)|Emplacement de la commande. Pour v1, 'Inline' est la valeur par défaut et la seule acceptée.|
|**command**|nvarchar(max)|Les commandes à exécuter par les travaux élastiques via command_type.|
|**credential_name**|nvarchar(128)|Nom des informations d'identification incluses dans l'étendue de la base de données utilisée pour l’exécution du travail.|
|**target_group_name**|nvarchar(128)|Nom du groupe cible.|
|**target_group_id**|UNIQUEIDENTIFIER|ID unique du groupe cible.|
|**initial_retry_interval_seconds**|int|Le délai avant la première nouvelle tentative. La valeur par défaut est 1.|
|**maximum_retry_interval_seconds**|int|Délai maximal entre les nouvelles tentatives. Si le délai entre deux tentatives dépasse cette valeur, il est limité à cette valeur à la place. La valeur par défaut est 120.|
|**retry_interval_backoff_multiplier**|real|Le multiplicateur à appliquer au délai de nouvelle tentative si plusieurs tentatives d’exécution d’une étape de travail échouent. La valeur par défaut est 2,0.|
|**retry_attempts**|int|Nombre de tentatives à effectuer si l'étape échoue. Valeur par défaut de 10, qui indique l’absence de nouvelles tentatives.|
|**step_timeout_seconds**|int|Nombre de minutes s'écoulant entre chaque tentative de reprise. La valeur par défaut est 0 et indique un intervalle de 0 minute.|
|**output_type**|nvarchar(11)|Emplacement de la commande. Dans la préversion actuelle, 'Inline' est la valeur par défaut et la seule acceptée.|
|**output_credential_name**|nvarchar(128)|Nom des informations d'identification à utiliser pour se connecter au serveur de destination afin de stocker le jeu de résultats.|
|**output_subscription_id**|UNIQUEIDENTIFIER|ID unique de l’abonnement du serveur\de la base de données de destination pour le jeu de résultats de l’exécution de la requête.|
|**output_resource_group_name**|nvarchar(128)|Nom du groupe de ressources où réside le serveur de destination.|
|**output_server_name**|nvarchar(256)|Nom du serveur de destination pour le jeu de résultats.|
|**output_database_name**|nvarchar(128)|Nom de la base de données de destination pour le jeu de résultats.|
|**output_schema_name**|nvarchar(max)|Nom du schéma de destination. La valeur par défaut est dbo si rien d’autre n’est spécifié.|
|**output_table_name**|nvarchar(max)|Nom de la table pour stocker le jeu de résultats de la requête. La table sera créée automatiquement selon le schéma du jeu de résultats si elle n’existe pas encore. Le schéma doit correspondre au schéma du jeu de résultats.|
|**max_parallelism**|int|Le nombre maximal de bases de données par pool élastique sur lesquelles l’étape de travail sera exécutée simultanément. La valeur par défaut est NULL, ce qui signifie qu’il n’y a aucune limite. |

### <a name="jobstep_versions-view"></a><a name="jobstep_versions-view"></a>affichage de jobstep_versions

[jobs].[jobstep_versions]

Affiche toutes les étapes dans toutes les versions de chaque travail. Le schéma est identique à [jobsteps](#jobsteps-view).

### <a name="target_groups-view"></a><a name="target_groups-view"></a>affichage de target_groups

[jobs].[target_groups]

Répertorie tous les groupes cibles.

|Nom de la colonne|Type de données|Description|
|-----|-----|-----|
|**target_group_name**|nvarchar(128)|Le nom du groupe cible, une collection de bases de données.
|**target_group_id**|UNIQUEIDENTIFIER|ID unique du groupe cible.

### <a name="target_group_members-view"></a><a name="target_group_members-view"></a>target_group_members view

[jobs].[target_group_members]

Affiche tous les membres de tous les groupes cibles.

|Nom de la colonne|Type de données|Description|
|-----|-----|-----|
|**target_group_name**|nvarchar(128|Le nom du groupe cible, une collection de bases de données. |
|**target_group_id**|UNIQUEIDENTIFIER|ID unique du groupe cible.|
|**membership_type**|int|Spécifie si le membre du groupe cible est inclus ou exclu dans le groupe cible. Les valeurs valides pour target_group_name sont 'Include' ou 'Exclude'.|
|**target_type**|nvarchar(128)|Type de base de données cible ou de collection de bases de données incluant toutes les bases de données dans un serveur, toutes les bases de données dans un pool élastique, ou une base de données individuelle. Les valeurs valides pour target_type sont 'SqlServer', 'SqlElasticPool', 'SqlDatabase' ou 'SqlShardMap'.|
|**target_id**|UNIQUEIDENTIFIER|ID unique du membre du groupe cible.|
|**refresh_credential_name**|nvarchar(128)|Nom des informations d'identification incluses dans l'étendue de la base de données utilisées pour se connecter au membre du groupe cible.|
|**subscription_id**|UNIQUEIDENTIFIER|ID unique de l’abonnement.|
|**resource_group_name**|nvarchar(128)|Nom du groupe de ressources dans lequel réside le membre du groupe cible.|
|**server_name**|nvarchar(128)|Nom du serveur contenu dans le groupe cible. Spécifié uniquement si target_type est 'SqlServer'. |
|**database_name**|nvarchar(128)|Nom de la base de données contenue dans le groupe cible. Spécifié uniquement lorsque target_type est ‘SqlDatabase’.|
|**elastic_pool_name**|nvarchar(128)|Nom du pool élastique contenu dans le groupe cible. Spécifié uniquement lorsque target_type est ‘SqlElasticPool’.|
|**shard_map_name**|nvarchar(128)|Nom des cartes de partition contenues dans le groupe cible. Spécifié uniquement lorsque target_type est ‘SqlShardMap’.|

## <a name="resources"></a>Ressources

- ![Icône du lien de rubrique](/sql/database-engine/configure-windows/media/topic-link.gif "Icône du lien de rubrique") [Conventions de la syntaxe Transact-SQL](/sql/t-sql/language-elements/transact-sql-syntax-conventions-transact-sql)  

## <a name="next-steps"></a>Étapes suivantes

- [Créer et gérer des tâches élastiques à l’aide de PowerShell](elastic-jobs-powershell-create.md)
- [Autorisation et privilèges](/dotnet/framework/data/adonet/sql/authorization-and-permissions-in-sql-server)