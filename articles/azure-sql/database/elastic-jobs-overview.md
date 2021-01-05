---
title: Tâches de base de données élastique (préversion)
description: Configurer des tâches de base de données élastique (préversion) pour exécuter des scripts T-SQL (Transact-SQL) sur une ou plusieurs bases de données d'Azure SQL Database
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: srinia
ms.author: srinia
ms.reviewer: sstein
ms.date: 12/18/2018
ms.openlocfilehash: f9a026ed47d662b80ef01e505bfbcf8f32d20b04
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792172"
---
# <a name="create-configure-and-manage-elastic-jobs-preview"></a>Créer, configurer et gérer des travaux élastiques (préversion)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Dans cet article, vous allez apprendre à créer, configurer et gérer des travaux élastiques.

Si vous n’avez pas encore utilisé de travaux élastiques, [découvrez-en plus sur les concepts d’automatisation des travaux dans Azure SQL Database](job-automation-overview.md).

## <a name="create-and-configure-the-agent"></a>Créer et configurer l’agent

1. Créez ou identifiez une base de données de niveau S0 ou supérieur. Celle-ci sera utilisée comme *base de données des travaux* durant la création de l’agent de travail élastique.
2. Créez un agent de travail élastique dans le [portail](https://portal.azure.com/#create/Microsoft.SQLElasticJobAgent) ou avec [PowerShell](elastic-jobs-powershell-create.md#create-the-elastic-job-agent).

   ![Création d’un agent de travail élastique](./media/elastic-jobs-overview/create-elastic-job-agent.png)

## <a name="create-run-and-manage-jobs"></a>Créer, exécuter et gérer des travaux

1. Créez des informations d’identification pour l’exécution de travaux dans la *Base de données des travaux* à l’aide de [PowerShell](elastic-jobs-powershell-create.md) ou de [T-SQL](elastic-jobs-tsql-create-manage.md#create-a-credential-for-job-execution).
2. Définissez le groupe cible (bases de données sur lesquelles exécuter le travail) à l’aide de [PowerShell](elastic-jobs-powershell-create.md) ou de [T-SQL](elastic-jobs-tsql-create-manage.md#create-a-target-group-servers).
3. Créez des informations d’identification d’agent de travail dans chaque base de données sur laquelle le travail sera exécuté [(ajoutez l’utilisateur, ou le rôle, à chaque base de données dans le groupe)](logins-create-manage.md). Pour obtenir un exemple, consultez le [tutoriel PowerShell](elastic-jobs-powershell-create.md).
4. Créez un travail à l’aide de [PowerShell](elastic-jobs-powershell-create.md) ou de [T-SQL](elastic-jobs-tsql-create-manage.md#deploy-new-schema-to-many-databases).
5. Ajoutez des étapes de travail à l’aide de [PowerShell](elastic-jobs-powershell-create.md) ou de [T-SQL](elastic-jobs-tsql-create-manage.md#deploy-new-schema-to-many-databases).
6. Exécutez un travail à l’aide de [PowerShell](elastic-jobs-powershell-create.md#run-the-job) ou de [T-SQL](elastic-jobs-tsql-create-manage.md#begin-unplanned-execution-of-a-job).
7. Supervisez l’état d’exécution du travail à l’aide du portail, de [PowerShell](elastic-jobs-powershell-create.md#monitor-status-of-job-executions) ou de [T-SQL](elastic-jobs-tsql-create-manage.md#monitor-job-execution-status).

   ![Portail](./media/elastic-jobs-overview/elastic-job-executions-overview.png)

## <a name="credentials-for-running-jobs"></a>Informations d’identification pour l’exécution de travaux

Les travaux utilisent des [informations d’identification incluses dans l’étendue de la base de données](/sql/t-sql/statements/create-database-scoped-credential-transact-sql) pour se connecter aux bases de données spécifiées par le groupe cible lors de l’exécution. Si un groupe cible contient des serveurs ou des pools, ces informations d’identification incluses dans l’étendue de la base de données sont utilisées pour se connecter à la base de données master afin d’énumérer les bases de données disponibles.

La configuration d’informations d’identification appropriées pour exécuter un travail pouvant parfois porter à confusion, gardez à l’esprit les points suivants :

- Les informations d’identification incluses dans l’étendue de la base de données doivent être créées dans la *base de données de travaux* .
- **Toutes les bases de données cibles doivent avoir une connexion disposant d’ [autorisations suffisantes](/sql/relational-databases/security/permissions-database-engine) pour que le travail puisse s’exécuter** (`jobuser` dans le diagramme ci-dessous).
- Les informations d’identification peuvent être réutilisées dans différents travaux, et les mots de passe associés sont chiffrés et sécurisés pour que les utilisateurs ayant accès en lecture seule aux objets de travail ne puissent pas y accéder.

L’image suivante est conçue pour vous aider à comprendre et à configurer les informations d’identification de travail appropriées. **N’oubliez pas de créer l’utilisateur dans chaque base de données (toutes les *bases de données utilisateur cibles* ) sur laquelle le travail doit s’exécuter** .

![Informations d’identification des travaux élastiques](./media/elastic-jobs-overview/job-credentials.png)

## <a name="security-best-practices"></a>Bonnes pratiques de sécurité

Voici quelques bonnes pratiques liées à l’utilisation de travaux élastiques :

- Limitez l'utilisation des API aux personnes de confiance.
- Les informations d’identification doivent avoir les privilèges minimaux nécessaires pour effectuer l’étape de travail. Pour plus d'informations, consultez [Autorisations et permissions](/dotnet/framework/data/adonet/sql/authorization-and-permissions-in-sql-server).
- Quand vous utilisez le membre d’un groupe cible dans un serveur et/ou un pool, il est fortement recommandé de créer des informations d’identification distinctes avec des droits sur la base de données master pour afficher/lister les bases de données afin de développer les listes de bases de données du ou des serveurs/pools avant l’exécution du travail.

## <a name="agent-performance-capacity-and-limitations"></a>Performances, capacité et limitations de l’agent

Les travaux élastiques utilisent des ressources de calcul minimales dans l’attente de la finalisation des travaux de longue durée.

En fonction de la taille du groupe cible de bases de données et du délai d’exécution souhaité d’un travail (nombre de workers simultanés), l’agent nécessite différents niveaux de calcul et de performance de la part de la *base de données de travaux* (plus le nombre de cibles et de travaux est élevé, plus les calculs nécessaires sont importants).

La préversion est actuellement limitée à 100 travaux simultanés.

### <a name="prevent-jobs-from-reducing-target-database-performance"></a>Empêcher les travaux de réduire le niveau de performance de la base de données cible

Pour éviter la surcharge des ressources durant l’exécution de travaux sur des bases de données dans un pool élastique SQL, vous pouvez configurer les travaux de manière à limiter le nombre de bases de données sur lesquelles un travail peut s’exécuter simultanément.

Définissez le nombre de bases de données simultanées sur lesquelles un travail s’exécute en définissant le paramètre `@max_parallelism` de la procédure stockée `sp_add_jobstep`dans T-SQL, ou `Add-AzSqlElasticJobStep -MaxParallelism` dans PowerShell.

## <a name="best-practices-for-creating-jobs"></a>Bonnes pratiques relatives à la création de travaux

### <a name="idempotent-scripts"></a>Scripts idempotents
Les scripts T-SQL d’un travail doivent être [idempotents](https://en.wikipedia.org/wiki/Idempotence). **Idempotent** signifie que si le script réussit et qu’il est réexécuté, le résultat est le même. Un script peut échouer en raison de problèmes réseau provisoires. Dans ce cas, le travail va automatiquement refaire des tentatives d’exécution du script un nombre prédéfini de fois. Un script idempotent a le même résultat, même s’il a été correctement exécuté deux fois (ou plus).

Une tactique simple consiste à tester l’existence d’un objet avant de créer ce dernier.


```sql
IF NOT EXISTS (some_object)
    -- Create the object
    -- If it exists, drop the object before recreating it.
```

De même, un script doit être capable d’exécuter avec succès par une vérification logique et s’adapter aux conditions qu’il rencontre.



## <a name="next-steps"></a>Étapes suivantes

- [Créer et gérer des tâches élastiques à l’aide de PowerShell](elastic-jobs-powershell-create.md)
- [Créer et gérer des tâches élastiques à l’aide de T-SQL (Transact-SQL)](elastic-jobs-tsql-create-manage.md)