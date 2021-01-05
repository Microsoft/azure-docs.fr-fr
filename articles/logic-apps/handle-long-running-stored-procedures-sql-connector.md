---
title: Gérer les procédures stockées de longue durée dans le connecteur SQL
description: Comment gérer les procédures stockées qui expirent lors de l’utilisation du connecteur SQL dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: camerost, logicappspm
ms.topic: conceptual
ms.date: 10/27/2020
ms.openlocfilehash: f5b04c563dc81497f591788dc4890d379c0f898f
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93102660"
---
# <a name="handle-stored-procedure-timeouts-in-the-sql-connector-for-azure-logic-apps"></a>Gérer les délais d’expiration des procédures stockées dans le connecteur SQL pour Azure Logic Apps

Lorsque votre application logique fonctionne avec des jeux de résultats si volumineux que le [connecteur SQL](../connectors/connectors-create-api-sqlazure.md) ne retourne pas tous les résultats en même temps, ou si vous souhaitez plus de contrôle sur la taille et la structure de vos jeux de résultats, vous pouvez créer une [procédure stockée ](/sql/relational-databases/stored-procedures/stored-procedures-database-engine) qui organise les résultats comme vous le souhaitez. Le connecteur SQL fournit de nombreuses fonctionnalités back-end auxquelles vous pouvez accéder à l’aide d’[Azure Logic Apps](../logic-apps/logic-apps-overview.md) afin d’automatiser plus facilement les tâches métier qui utilisent des tables de base de données SQL.

Par exemple, lorsque vous procédez à l’extraction ou à l’insertion de plusieurs lignes, votre application logique peut effectuer une itération dans ces lignes en utilisant une [boucle **Until**](../logic-apps/logic-apps-control-flow-loops.md#until-loop) dans ces [limites](../logic-apps/logic-apps-limits-and-config.md). Toutefois, lorsque votre application logique doit manipuler des milliers ou millions de lignes, vous pouvez réduire les coûts liés aux appels à la base de données. Pour plus d’informations, consultez [Gérer des données en bloc à l’aide du connecteur SQL](../connectors/connectors-create-api-sqlazure.md#handle-bulk-data).

<a name="timeout-limit"></a>

## <a name="timeout-limit-on-stored-procedure-execution"></a>Limite de délai d’expiration pour l’exécution d’une procédure stockée

Le connecteur SQL a une limite de délai d’expiration de procédure stockée qui est [inférieure à 2 minutes](/connectors/sql/#known-issues-and-limitations). Certaines procédures stockées peuvent prendre plus de temps que cette limite, provoquant une erreur `504 Timeout`. Parfois, ces processus de longue durée sont codées en tant que procédures stockées explicitement à cette fin. En raison de cette limite de délai d’expiration, l’appel de ces procédures à partir d’Azure Logic Apps peut occasionner des problèmes. Bien que le connecteur SQL ne prenne en charge nativement aucun mode asynchrone, vous pouvez contourner ce problème et simuler un tel mode à l’aide d’un déclencheur d’achèvement SQL, d’une requête pass-through SQL native, d’une table des états et de travaux côté serveur. Pour cette tâche, vous pouvez utiliser l’[Agent de travail élastique Azure](../azure-sql/database/elastic-jobs-overview.md) pour [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md). Pour [SQL Server en local](/sql/sql-server/sql-server-technical-documentation) et [Azure SQL Managed Instance](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md), vous pouvez utiliser le service [SQL Server Agent](/sql/ssms/agent/sql-server-agent).

Supposons, par exemple, que vous avez la procédure stockée de longue durée suivante, qui prend plus de temps que la limite de délai d’expiration. Si vous exécutez cette procédure stockée à partir d’une application logique à l’aide du connecteur SQL, vous obtenez une erreur `HTTP 504 Gateway Timeout`.

```sql
CREATE PROCEDURE [dbo].[WaitForIt]
   @delay char(8) = '00:03:00'
AS
BEGIN
   SET NOCOUNT ON;
   WAITFOR DELAY @delay
END
```

Au lieu d’appeler directement la procédure stockée, vous pouvez l’exécuter de manière asynchrone en arrière-plan à l’aide d’un *agent de travail*. Vous pouvez stocker les entrées et sorties dans une table d’états avec laquelle vous pouvez interagir via votre application logique. Si vous n’avez pas besoin des entrées et sorties, ou si vous écrivez déjà les résultats dans une table de la procédure stockée, vous pouvez simplifier cette approche.

> [!IMPORTANT]
> Assurez-vous que votre procédure stockée et tous les travaux sont *idempotents* , ce qui signifie qu’ils peuvent s’exécuter plusieurs fois sans affecter les résultats. Si le traitement asynchrone échoue ou expire, l’agent de travail peut réessayer d’effectuer l’étape, et donc votre procédure stockée, à plusieurs reprises. Pour éviter la duplication des sorties, avant de créer des objets, consultez ces [meilleures pratiques et approches](../azure-sql/database/elastic-jobs-overview.md#idempotent-scripts).

La section suivante décrit comment utiliser l’Agent de travail élastique Azure pour Azure SQL Database. Pour SQL Server et Azure SQL Managed Instance, vous pouvez utiliser le service SQL Server Agent. Certains détails de gestion diffèrent, mais les étapes fondamentales restent les mêmes que la configuration d’un agent de travail pour Azure SQL Database.

<a name="azure-sql-database"></a>

## <a name="job-agent-for-azure-sql-database"></a>Agent de travail pour Azure SQL Database

Pour créer un travail pouvant exécuter la procédure stockée pour [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md), utilisez l’[Agent de travail élastique Azure](../azure-sql/database/elastic-jobs-overview.md). Créez votre agent de travail dans le portail Azure. Cette approche ajoute plusieurs procédures stockées à la base de données que l’agent utilise, également appelée *base de données de l’agent*. Vous pouvez ensuite créer un travail exécutant votre procédure stockée dans la base de données cible, et capturant la sortie une fois que vous avez terminé.

Avant de pouvoir créer le travail, vous devez configurer des autorisations, des groupes et des cibles en procédant de la manière décrite dans la [documentation complète de l’Agent de travail élastique Azure](../azure-sql/database/elastic-jobs-overview.md). Vous devez également créer une table associée dans la base de données cible, comme décrit dans les sections suivantes.

<a name="create-state-table"></a>

### <a name="create-state-table-for-registering-parameters-and-storing-inputs"></a>Créer une table d’états pour l’inscription de paramètres et le stockage d’entrées

Les travaux de SQL Agent n’acceptent pas de paramètres d’entrée. Au lieu de cela, dans la base de données cible, créez une table d’états dans laquelle vous inscrivez les paramètres et stockez les entrées à utiliser pour appeler vos procédures stockées. Toutes les étapes du travail de l’agent s’exécutent sur la base de données cible, mais les procédures stockées du travail s’exécutent sur la base de données de l’agent. 

Pour créer la table d’états, utilisez le schéma suivant :

```sql
CREATE TABLE [dbo].[LongRunningState](
   [jobid] [uniqueidentifier] NOT NULL,
   [rowversion] [timestamp] NULL,
   [parameters] [nvarchar](max) NULL,
   [start] [datetimeoffset](7) NULL,
   [complete] [datetimeoffset](7) NULL,
   [code] [int] NULL,
   [result] [nvarchar](max) NULL,
   CONSTRAINT [PK_LongRunningState] PRIMARY KEY CLUSTERED
      (   [jobid] ASC
      )WITH (STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF) ON [PRIMARY]
      ) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
```

Voici à quoi ressemble la table dans [SQL Server Management Studio (SMSS)](/sql/ssms/download-sql-server-management-studio-ssms) :

![Capture d’écran montrant la table d’états créée qui stocke les entrées pour la procédure stockée.](media/handle-long-running-stored-procedures-sql-connector/state-table-input-parameters.png)

Pour garantir de bonnes performances et s’assurer que le travail de l’agent peut trouver l’enregistrement associé, la table utilise l’ID d’exécution du travail (`jobid`) comme clé primaire. Si vous le souhaitez, vous pouvez également ajouter des colonnes pour les paramètres d’entrée. Le schéma décrit précédemment peut généralement gérer plusieurs paramètres, mais il est limité à la taille calculée par `NVARCHAR(MAX)`.

<a name="create-top-level-job"></a>

### <a name="create-a-top-level-job-to-run-the-stored-procedure"></a>Créer un travail de niveau supérieur pour exécuter la procédure stockée

Pour exécuter la procédure stockée de longue durée, créez cet agent de travail de niveau supérieur dans la base de données de l’agent :

```sql
EXEC jobs.sp_add_job 
   @job_name='LongRunningJob',
   @description='Execute Long-Running Stored Proc',
   @enabled = 1
```

À présent, ajoutez au travail des étapes qui paramètrent, exécutent et accomplissent la procédure stockée. Par défaut, une étape de travail expire au bout de 12 heures. Si votre procédure stockée a besoin de plus de temps, ou si vous souhaitez que la procédure expire plus tôt, vous pouvez modifier le paramètre `step_timeout_seconds` en spécifiant une autre valeur exprimée en secondes. Par défaut, une étape comporte 10 tentatives intégrées avec un délai d’attente d’interruption entre chaque nouvelle tentative, que vous pouvez utiliser à votre avantage.

Voici les étapes à ajouter :

1. Attendre que les paramètres s’affichent dans la table `LongRunningState`.

   Cette première étape attend que les paramètres soient ajoutés dans `LongRunningState` table, ce qui se produit peu après le démarrage du travail. Si l’ID d’exécution du travail (`jobid`) n’est pas ajouté à la table `LongRunningState`, l’étape échoue simplement et le délai d’attente par défaut de nouvelle tentative ou d’interruption est le suivant :

   ```sql
   EXEC jobs.sp_add_jobstep
      @job_name='LongRunningJob',
      @step_name= 'Parameterize WaitForIt',
      @step_timeout_seconds = 30,
      @command= N'
         IF NOT EXISTS(SELECT [jobid] FROM [dbo].[LongRunningState]
            WHERE jobid = $(job_execution_id))
            THROW 50400, ''Failed to locate call parameters (Step1)'', 1',
      @credential_name='JobRun',
      @target_group_name='DatabaseGroupLongRunning'
   ```

1. Interroger les paramètres de la table d’états et les transmettre à la procédure stockée. Cette étape exécute également la procédure en arrière-plan. 

   Si votre procédure stockée n’a pas besoin de paramètres, appelez directement la procédure stockée. Sinon, pour passer le paramètre `@timespan`, utilisez la valeur `@callparams`, que vous pouvez également étendre pour passer des paramètres supplémentaires.

   ```sql
   EXEC jobs.sp_add_jobstep
      @job_name='LongRunningJob',
      @step_name='Execute WaitForIt',
      @command=N'
         DECLARE @timespan char(8)
         DECLARE @callparams NVARCHAR(MAX)
         SELECT @callparams = [parameters] FROM [dbo].[LongRunningState]
            WHERE jobid = $(job_execution_id))
         SET @timespan = @callparams
         EXECUTE [dbo].[WaitForIt] @delay = @timespan', 
      @credential_name='JobRun',
      @target_group_name='DatabaseGroupLongRunning'
   ```

1. Terminer le travail et enregistrer les résultats.

   ```sql
   EXEC jobs.sp_add_jobstep
      @job_name='LongRunningJob',
      @step_name='Complete WaitForIt',
      @command=N'
         UPDATE [dbo].[LongRunningState]
            SET [complete] = GETUTCDATE(),
               [code] = 200,
               [result] = ''Success''
            WHERE jobid = $(job_execution_id)',
      @credential_name='JobRun',
      @target_group_name='DatabaseGroupLongRunning'
   ```

<a name="start-job-pass-parameters"></a>

### <a name="start-the-job-and-pass-the-parameters"></a>Démarrer le travail et passer les paramètres

Pour démarrer le travail, utilisez une requête native pass-through avec l’ [action **Exécuter une requête SQL**](/connectors/sql/#execute-a-sql-query-(v2)), et envoyez immédiatement les paramètres du travail à la table d’états. Pour fournir une entrée à l’attribut `jobid` de la table cible, Logic Apps ajoute une boucle **For Each** qui itère dans la sortie de la table résultant de l’action précédente. Pour chaque ID d’exécution de travail, exécutez une action **Insérer une ligne** qui utilise la sortie de données dynamiques, `ResultSets JobExecutionId`, pour ajouter les paramètres du travail à décompresser et à passer à la procédure stockée cible.

![Capture d’écran montrant les actions à utiliser pour démarrer le travail et passer des paramètres à la procédure stockée.](media/handle-long-running-stored-procedures-sql-connector/start-job-actions.png)

Une fois terminé, le travail met à jour la table `LongRunningState` afin que vous puissiez facilement déclencher sur le résultat à l’aide du [déclencheur **Quand un élément est modifié**](/connectors/sql/#when-an-item-is-modified-(v2)). Si vous n’avez pas besoin de la sortie ou si vous disposez déjà d’un déclencheur qui surveille une table de sortie, vous pouvez ignorer cette partie.

![Capture d’écran montrant le déclencheur SQL quand un élément est modifié.](media/handle-long-running-stored-procedures-sql-connector/trigger-on-results-after-job-completes.png)

<a name="sql-on-premises-or-managed-instance"></a>

## <a name="job-agent-for-sql-server-or-azure-sql-managed-instance"></a>Agent de travail pour SQL Server ou Azure SQL Managed Instance

Pour le même scénario, vous pouvez utiliser le service [SQL Server Agent](/sql/ssms/agent/sql-server-agent) pour [SQL Server local](/sql/sql-server/sql-server-technical-documentation) et [Azure SQL Managed Instance](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md). Bien que certains détails de gestion diffèrent, les étapes fondamentales restent les mêmes que pour la configuration d’un agent de travail pour Azure SQL Database.

## <a name="next-steps"></a>Étapes suivantes

[Établir une connexion à SQL Server, Azure SQL Database ou Azure SQL Managed Instance](../connectors/connectors-create-api-sqlazure.md)

