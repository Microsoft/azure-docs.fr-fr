---
title: Migrer de DTU vers vCore
description: Migrez une base de données dans Azure SQL Database du modèle DTU vers le modèle vCore. La migration vers vCore s’apparente à la mise à niveau ou au passage à une version antérieure entre les niveaux Standard et Premium.
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.topic: conceptual
ms.custom: sqldbrb=1
author: stevestein
ms.author: sstein
ms.reviewer: sashan, moslake
ms.date: 05/28/2020
ms.openlocfilehash: aa236ecaaa9c38c68e66d1813280cd98b85b9463
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790387"
---
# <a name="migrate-azure-sql-database-from-the-dtu-based-model-to-the-vcore-based-model"></a>Migrer Azure SQL Database à partir du modèle DTU vers le modèle vCore
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article explique comment migrer votre base de données dans Azure SQL Database du [modèle d’achat DTU](service-tiers-dtu.md) vers le [modèle d’achat vCore](service-tiers-vcore.md). 

## <a name="migrate-a-database"></a>Migrer une base de données

La migration d’une base de données du modèle d’achat DTU vers le modèle d’achat vCore est semblable à la mise à l’échelle entre les objectifs de service des niveaux de service De base, Standard et Premium, avec une [durée](single-database-scale.md#latency) similaire et un [temps d’arrêt minimal](scale-resources.md) au terme du processus de migration. Une base de données migrée vers le modèle d’achat vCore peut, à tout moment, être à nouveau migrée vers le modèle d’achat DTU et ce, de la même manière, à l’exception des bases de données migrées vers le niveau de service [Hyperscale](service-tier-hyperscale.md). 

## <a name="choose-the-vcore-service-tier-and-service-objective"></a>Choisir le niveau de service et l’objectif de service vCore

Dans la plupart des scénarios de migration DTU vers vCore, les bases de données et les pools élastiques des niveaux de service De base et Standard sont mappés vers le niveau de service [Usage général](service-tier-general-purpose.md). Les bases de données et les pools élastiques du niveau de service Premium sont mappés vers le niveau de service [Critique pour l’entreprise](service-tier-business-critical.md). Selon le scénario et la configuration requise de l’application, le niveau de service [Hyperscale](service-tier-hyperscale.md) peut souvent être utilisé comme cible de migration pour les bases de données uniques dans tous les niveaux de service DTU.

Pour choisir l’objectif de service, ou la taille de calcul, pour la base de données migrée dans le modèle vCore, vous pouvez utiliser une règle simple mais approximative : chaque tranche de 100 DTU des niveaux De base ou Standard nécessite *au moins* 1 vCore, et chaque tranche de 125 DTU du niveau Premium nécessite *au moins* 1 vCore. 

> [!TIP]
> Cette règle est approximative, car elle ne prend pas en compte la génération de matériel utilisé pour la base de données ou le pool élastique DTU. 

Dans le modèle DTU, toute [génération de matériel](purchasing-models.md#hardware-generations-in-the-dtu-based-purchasing-model) disponible peut être utilisée pour votre base de données ou pool élastique. En outre, vous disposez uniquement d’un contrôle indirect sur le nombre de vCores (UC logiques), en choisissant des valeurs DTU ou eDTU supérieures ou inférieures. 

Avec le modèle vCore, les clients doivent faire un choix explicite en termes de génération de matériel et de nombre de vCores (UC logiques). Le modèle DTU n’offre pas ces choix, mais la génération de matériel et le nombre d’UC logiques utilisées pour chaque base de données et pool élastique sont exposés via des vues de gestion dynamique. Il est ainsi possible de déterminer de manière plus précise l’objectif de service vCore correspondant. 

L’approche suivante utilise ces informations pour déterminer un objectif de service vCore avec une allocation de ressources semblable, afin d’obtenir un niveau de performance similaire après migration vers le modèle vCore.

### <a name="dtu-to-vcore-mapping"></a>Mappage DTU vers vCore

Une requête T-SQL ci-dessous, lorsqu’elle est exécutée dans le contexte d’une base de données DTU à migrer, renvoie un nombre de vCores correspondant (éventuellement fractionnaire) dans chaque génération de matériel du modèle vCore. En arrondissant ce nombre au nombre de vCores disponibles le plus proche pour les [bases de données](resource-limits-vcore-single-databases.md) et [pools élastiques](resource-limits-vcore-elastic-pools.md) dans chaque génération de matériel du modèle vCore, les clients peuvent choisir l’objectif de service vCore qui correspond le mieux à la base de données ou au pool élastique DTU. 

Les exemples de scénarios de migration utilisant cette approche sont décrits dans la section [Exemples](#dtu-to-vcore-migration-examples).

Exécutez cette requête dans le contexte de la base de données à migrer plutôt que dans la base de données `master`. Lors de la migration d’un pool élastique, exécutez la requête dans le contexte d’une base de données du pool.

```SQL
WITH dtu_vcore_map AS
(
SELECT TOP (1) rg.slo_name,
               CASE WHEN rg.slo_name LIKE '%SQLG4%' THEN 'Gen4'
                    WHEN rg.slo_name LIKE '%SQLGZ%' THEN 'Gen4'
                    WHEN rg.slo_name LIKE '%SQLG5%' THEN 'Gen5'
                    WHEN rg.slo_name LIKE '%SQLG6%' THEN 'Gen5'
               END AS dtu_hardware_gen,
               s.scheduler_count * CAST(rg.instance_cap_cpu/100. AS decimal(3,2)) AS dtu_logical_cpus,
               CAST((jo.process_memory_limit_mb / s.scheduler_count) / 1024. AS decimal(4,2)) AS dtu_memory_per_core_gb
FROM sys.dm_user_db_resource_governance AS rg
CROSS JOIN (SELECT COUNT(1) AS scheduler_count FROM sys.dm_os_schedulers WHERE status = 'VISIBLE ONLINE') AS s
CROSS JOIN sys.dm_os_job_object AS jo
WHERE dtu_limit > 0
      AND
      DB_NAME() <> 'master'
)
SELECT dtu_logical_cpus,
       dtu_hardware_gen,
       dtu_memory_per_core_gb,
       CASE WHEN dtu_hardware_gen = 'Gen4' THEN dtu_logical_cpus
            WHEN dtu_hardware_gen = 'Gen5' THEN dtu_logical_cpus * 0.7
       END AS Gen4_vcores,
       7 AS Gen4_memory_per_core_gb,
       CASE WHEN dtu_hardware_gen = 'Gen4' THEN dtu_logical_cpus * 1.7
            WHEN dtu_hardware_gen = 'Gen5' THEN dtu_logical_cpus
       END AS Gen5_vcores,
       5.05 AS Gen5_memory_per_core_gb,
       CASE WHEN dtu_hardware_gen = 'Gen4' THEN dtu_logical_cpus
            WHEN dtu_hardware_gen = 'Gen5' THEN dtu_logical_cpus * 0.8
       END AS Fsv2_vcores,
       1.89 AS Fsv2_memory_per_core_gb,
       CASE WHEN dtu_hardware_gen = 'Gen4' THEN dtu_logical_cpus * 1.4
            WHEN dtu_hardware_gen = 'Gen5' THEN dtu_logical_cpus * 0.9
       END AS M_vcores,
       29.4 AS M_memory_per_core_gb
FROM dtu_vcore_map;
```

### <a name="additional-factors"></a>Autres facteurs

En plus du nombre de vCores (UC logiques) et de la génération de matériel, plusieurs autres facteurs peuvent influencer le choix de l’objectif de service vCore :

- La requête T-SQL de mappage correspond aux objectifs de service DTU et vCore en termes de capacité d’UC. Dès lors, les résultats sont plus précis pour les charges de travail liées au processeur.
- Pour la même génération de matériel et le même nombre de vCores, les limites IOPS et de débit du journal des transactions des bases de données vCore sont souvent plus élevées que celles des bases de données DTU. Pour les charges de travail dépendantes des E/S, il est possible de réduire le nombre de vCores du modèle vCore afin obtenir le même niveau de performance. Les limites de ressources des bases de données DTU et vCore en valeurs absolues sont exposées dans la vue [sys.dm_user_db_resource_governance](/sql/relational-databases/system-dynamic-management-views/sys-dm-user-db-resource-governor-azure-sql-database). La comparaison de ces valeurs entre la base de données DTU à migrer et une base de données vCore utilisant un objectif de service proche vous permet de sélectionner plus précisément l’objectif de service vCore.
- La requête de mappage renvoie également la quantité de mémoire par cœur pour la base de données ou le pool élastique DTU à migrer, et pour chaque génération de matériel du modèle vCore. Il convient d’obtenir une mémoire totale similaire ou supérieure après migration vers vCore pour les charges de travail nécessitant un cache de données important à des fins de performances suffisantes, ou pour les charges de travail nécessitant des allocations de mémoire importantes à des fins de traitement des requêtes. Pour ces charges de travail, en fonction des performances réelles, il peut s’avérer nécessaire d’augmenter le nombre de vCores afin d’obtenir suffisamment de mémoire totale.
- L’[utilisation des ressources d’historique](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database) de la base de données DTU doit être prise en compte lors du choix de l’objectif de service vCore. Les bases de données DTU présentant systématiquement des ressources processeur sous-exploitées peuvent nécessiter moins de vCores que le nombre renvoyé par la requête de mappage. À l’inverse, les bases de données DTU présentant une utilisation du processeur systématiquement élevée traduisant des performances de charge de travail inadaptées peuvent nécessiter plus de vCores que le nombre renvoyé par la requête.
- Si vous migrez des bases de données avec des modèles d’utilisation intermittents ou imprévisibles, pensez à recourir au niveau de calcul [Serverless](serverless-tier-overview.md).  Notez que le nombre maximal de workers (requêtes) simultanés sur un serveur est de 75 % la limite du calcul configurée pour le même nombre de vcores maximal configuré.  En outre, la mémoire maximale disponible sur un serveur est de 3 Go multiplié par le nombre maximal de vcores configurés. Par exemple, la mémoire maximale est de 120 Go lorsque 40 vcores Max sont configurés.   
- Dans le modèle vCore, la taille maximale de base de données prise en charge peut varier en fonction de la génération de matériel. Pour les bases de données volumineuses, vérifiez les tailles maximales prises en charge dans le modèle vCore pour les [bases de données uniques](resource-limits-vcore-single-databases.md) et les [pools élastiques](resource-limits-vcore-elastic-pools.md).
- Pour les pools élastiques, les modèles [DTU](resource-limits-dtu-elastic-pools.md) et [vCore](resource-limits-vcore-elastic-pools.md) présentent des différences en termes de nombre maximal de bases de données par pool prises en charge. Cela doit être pris en compte lors de la migration de pools élastiques avec de nombreuses bases de données.
- Certaines générations de matériel peuvent ne pas être disponibles dans toutes les régions. Vérifiez la disponibilité sous [Générations de matériel](service-tiers-vcore.md#hardware-generations).

> [!IMPORTANT]
> Les instructions relatives au dimensionnement de DTU vers vCore ci-dessus sont fournies pour vous aider à faire une estimation initiale de l’objectif de service de la base de données cible.
>
> La configuration optimale de la base de données cible dépend de la charge de travail. Ainsi, pour atteindre le rapport prix/performances optimal après migration, il peut être nécessaire de tirer parti de la flexibilité du modèle vCore pour ajuster le nombre de vCores, la [génération de matériel](service-tiers-vcore.md#hardware-generations) et les niveaux de [service](service-tiers-vcore.md#service-tiers) et de [calcul](service-tiers-vcore.md#compute-tiers), ainsi que d’ajuster d’autres paramètres de configuration de base de données, comme le [degré maximal de parallélisme](/sql/relational-databases/query-processing-architecture-guide#parallel-query-processing).
> 

### <a name="dtu-to-vcore-migration-examples"></a>Exemples de migration DTU vers vCore

> [!NOTE]
> Les valeurs indiquées dans les exemples ci-dessous sont présentées à des fins d’illustration uniquement. Les valeurs réelles renvoyées dans les scénarios décrits peuvent être différentes.
> 

**Migration d’une base de données S9 standard**

La requête de mappage renvoie le résultat suivant (certaines colonnes ne sont pas affichées par souci de concision) :

|dtu_logical_cpus|dtu_hardware_gen|dtu_memory_per_core_gb|Gen4_vcores|Gen4_memory_per_core_gb|Gen5_vcores|Gen5_memory_per_core_gb|
|----------------|----------------|----------------------|-----------|-----------------------|-----------|-----------------------|
|24.00|Gen5|5.40|16.800|7|24.000|5.05|

Nous constatons que la base de données DTU dispose de 24 UC logiques (vCores), avec 5,4 Go de mémoire par vCore, et utilise le matériel Gen5. La correspondance directe à cela est une base de données Usage générale à 24 vCores sur le matériel Gen5, c’est-à-dire l’objectif de service vCore **GP_Gen5_24** .

**Migration d’une base de données S0 Standard**

La requête de mappage renvoie le résultat suivant (certaines colonnes ne sont pas affichées par souci de concision) :

|dtu_logical_cpus|dtu_hardware_gen|dtu_memory_per_core_gb|Gen4_vcores|Gen4_memory_per_core_gb|Gen5_vcores|Gen5_memory_per_core_gb|
|----------------|----------------|----------------------|-----------|-----------------------|-----------|-----------------------|
|0,25|Gen4|0.42|0.250|7|0.425|5.05|

Nous constatons que la base de données DTU dispose de l’équivalent de 0.25 UC logiques (vCores), avec 0,42 Go de mémoire par vCore, et utilise le matériel Gen4. Les plus petits objectifs de service vCore dans les générations de matériel Gen4 et Gen5, **GP_Gen4_1** et **GP_Gen5_2** , fournissent davantage de ressources de calcul que la base de données S0 standard, de sorte qu’une correspondance directe n’est pas possible. Le matériel Gen4 étant [mis hors service](https://azure.microsoft.com/updates/gen-4-hardware-on-azure-sql-database-approaching-end-of-life-in-2020/), l’option **GP_Gen5_2** est préférable. En outre, si la charge de travail est bien adaptée au niveau de calcul [Serverless](serverless-tier-overview.md), **GP_S_Gen5_1** est une correspondance plus proche.

**Migration d’une base de données P15 Premium**

La requête de mappage renvoie le résultat suivant (certaines colonnes ne sont pas affichées par souci de concision) :

|dtu_logical_cpus|dtu_hardware_gen|dtu_memory_per_core_gb|Gen4_vcores|Gen4_memory_per_core_gb|Gen5_vcores|Gen5_memory_per_core_gb|
|----------------|----------------|----------------------|-----------|-----------------------|-----------|-----------------------|
|42.00|Gen5|4.86|29.400|7|42.000|5.05|

Nous constatons que la base de données DTU dispose de 42 UC logiques (vCores), avec 4,86 Go de mémoire par vCore, et utilise le matériel Gen5. S’il n’existe pas d’objectif de service vCore avec 42 cœurs, l’objectif de service **BC_Gen5_40** est très proche en termes de capacité de mémoire et de processeur, et constitue une bonne correspondance.

**Migration d’un pool élastique 200 eDTU De base**

La requête de mappage renvoie le résultat suivant (certaines colonnes ne sont pas affichées par souci de concision) :

|dtu_logical_cpus|dtu_hardware_gen|dtu_memory_per_core_gb|Gen4_vcores|Gen4_memory_per_core_gb|Gen5_vcores|Gen5_memory_per_core_gb|
|----------------|----------------|----------------------|-----------|-----------------------|-----------|-----------------------|
|4,00|Gen5|5.40|2.800|7|4.000|5.05|

Nous constatons que le pool élastique DTU dispose de 4 UC logiques (vCores), avec 5,4 Go de mémoire par vCore, et utilise le matériel Gen5. La correspondance directe dans le modèle vCore est un pool élastique **GP_Gen5_4** . Cependant, cet objectif de service prend en charge un maximum de 200 bases de données par pool, tandis que le pool élastique 200 eDTU De base prend en charge jusqu’à 500 bases de données. Si le pool élastique à migrer contient plus de 200 bases de données, l’objectif de service vCore correspondant doit être **GP_Gen5_6** , qui prend en charge jusqu’à 500 bases de données.

## <a name="migrate-geo-replicated-databases"></a>Migration des bases de données géorépliquées

La migration du modèle DTU vers le modèle vCore est similaire à la mise à niveau (ou à la rétrogradation) des relations de géoréplication entre les bases de données dans les niveaux de service standard et premium. Pendant la migration, vous n’êtes pas obligé d’arrêter la géoréplication, mais vous devez suivre ces règles de séquencement :

- Lors d’une mise à niveau, vous devez mettre à niveau la base de données secondaire, avant de mettre à niveau la base de données primaire.
- Lors d’une rétrogradation, inversez l’ordre : rétrogradez d’abord la base de données primaire, puis la base de données secondaire.

Lorsque vous utilisez la géoréplication entre deux pools élastiques, nous vous recommandons de désigner un pool comme le pool principal et l’autre comme le pool secondaire. Dans ce cas, lorsque vous effectuez une migration de pools élastiques, vous devez utiliser les mêmes recommandations de séquencement. Toutefois, si vous avez des pools élastiques qui contiennent des bases de données primaires et secondaire, traitez le pool le plus utilisé comme pool principal et suivez les règles de séquencement en conséquence.  

Le tableau suivant fournit des conseils pour certains scénarios de migration :

|Niveau de service actuel|Niveau de service cible|Type de migration|Actions utilisateur|
|---|---|---|---|
|Standard|Usage général|Latéral|Peut effectuer la migration dans n’importe quel ordre, mais doit garantir un redimensionnement vCore, comme décrit ci-dessus|
|Premium|Critique pour l'entreprise|Latéral|Peut effectuer la migration dans n’importe quel ordre, mais doit garantir un redimensionnement vCore, comme décrit ci-dessus|
|Standard|Critique pour l'entreprise|Mettre à niveau|Doit d’abord effectuer la migration de la base de données secondaire|
|Critique pour l'entreprise|Standard|Rétrogradation|Doit d’abord effectuer la migration de la base de données primaire|
|Premium|Usage général|Rétrogradation|Doit d’abord effectuer la migration de la base de données primaire|
|Usage général|Premium|Mettre à niveau|Doit d’abord effectuer la migration de la base de données secondaire|
|Critique pour l'entreprise|Usage général|Rétrogradation|Doit d’abord effectuer la migration de la base de données primaire|
|Usage général|Critique pour l'entreprise|Mettre à niveau|Doit d’abord effectuer la migration de la base de données secondaire|
||||

## <a name="migrate-failover-groups"></a>Migrer des groupes de basculement

La migration des groupes de basculement comprenant plusieurs bases de données nécessite que la base de données primaire et la base de données secondaire soient migrées séparément. Pendant ce processus, les mêmes recommandations et règles de séquencement s’appliquent. Une fois les bases de données converties au modèle d’achat vCore, le groupe de basculement reste actif, avec les mêmes paramètres de stratégie.

### <a name="create-a-geo-replication-secondary-database"></a>Créer une base de données secondaire de géoréplication

Vous pouvez créer une base de données secondaire géoréplication (réplica géosecondaire) uniquement dans le même niveau de service que celui utilisé pour la base de données primaire. Pour les bases de données avec un taux élevé de génération de journaux, nous vous recommandons de créer le réplica géosecondaire avec la même taille de calcul que le réplica principal.

Si vous créez une base de données secondaire de géoréplication dans le pool élastique pour une base de données primaire unique, le paramètre `maxVCore` du pool doit correspondre à la taille de calcul de la base de données primaire. Si vous créez une base de données secondaire de géoréplication pour une base de données primaire située dans un autre pool élastique, nous vous recommandons d’attribuer la même valeur au paramètre `maxVCore` des deux pools.

## <a name="use-database-copy-to-migrate-from-dtu-to-vcore"></a>Utiliser la copie de base de données pour migrer de DTU vers vCore

Vous pouvez copier n’importe quelle base de données avec une taille de calcul DTU vers une base de données avec une taille de calcul vCore, sans aucune restriction ni séquencement spécial, tant que la taille de calcul cible prend en charge la taille maximale de la base de données source. La copie de base de données crée un instantané des données dès que commence l’opération de copie, et elle ne synchrone pas les données entre la source et la cible.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les tailles de calcul et les tailles de stockage disponibles pour les bases de données uniques, consultez [Limites des ressources vCore SQL Database pour les bases de données uniques](resource-limits-vcore-single-databases.md).
- Pour plus d’informations sur les tailles de calcul et les tailles de stockage disponibles pour les pools élastiques, consultez [Limites des ressources vCore SQL Database pour les pools élastiques](resource-limits-vcore-elastic-pools.md).