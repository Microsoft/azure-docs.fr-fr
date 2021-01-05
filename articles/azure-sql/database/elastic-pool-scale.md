---
title: Mettre à l’échelle des ressources de pool élastique
description: Cette page décrit les ressources de mise à l’échelle pour les pools élastiques dans Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: elastic-pools
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: oslake
ms.author: moslake
ms.reviewer: sstein
ms.date: 09/16/2020
ms.openlocfilehash: 947d842860452425f8b30fbdaf9558c2a94a89a2
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92781207"
---
# <a name="scale-elastic-pool-resources-in-azure-sql-database"></a>Mettre à l’échelle un pool élastique dans Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article décrit comment faire évoluer les ressources de calcul et de stockage disponibles pour les pools élastiques et les bases de données mises en pool dans Azure SQL Database.

## <a name="change-compute-resources-vcores-or-dtus"></a>Modifier les ressources de calcul (vCores ou DTU)

Après la sélection initiale du nombre de vCores ou eDTU, vous pouvez effectuer un scale-up ou un scale-down d’un pool élastique de façon dynamique selon l’expérience réelle à l’aide du [portail Azure](elastic-pool-manage.md#azure-portal), de [PowerShell](/powershell/module/az.sql/Get-AzSqlElasticPool), d’[Azure CLI](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-update) ou de l’[API REST](/rest/api/sql/elasticpools/update).

### <a name="impact-of-changing-service-tier-or-rescaling-compute-size"></a>Impact du changement de niveau de service ou de la remise à l’échelle de la taille de calcul

La modification du niveau de service ou de la taille de calcul d’un pool élastique suit un modèle similaire à celui des bases de données uniques qui implique principalement que le service effectue les étapes suivantes :

1. Créer une instance de calcul pour le pool élastique  

    Une nouvelle instance de calcul est créée pour le pool élastique avec le niveau de service et la taille de calcul demandés. Pour certaines combinaisons de modifications de niveau de service et de taille de calcul, un réplica de chaque base de données doit être créé dans la nouvelle instance de calcul, ce qui implique de copier des données et peut influencer fortement la latence globale. Malgré tout, les bases de données restent en ligne pendant cette étape et les connexions continuent d’être dirigées vers les bases de données dans l’instance de calcul d’origine.

2. Basculement de l’acheminement des connexions vers la nouvelle instance de calcul

    Les connexions existantes aux bases de données dans l’instance de calcul d’origine sont supprimées. Toutes les nouvelles connexions sont établies aux bases de données de la nouvelle instance de calcul. Pour certaines combinaisons de modifications du niveau de service et de la taille de calcul, les fichiers de base de données sont détachés, puis rattachés pendant ce basculement.  Malgré tout, le basculement peut entraîner une brève interruption de service quand les bases de données ne sont pas en disponibilité générale pendant moins de 30 secondes et souvent pendant quelques secondes seulement. Si des transactions durables sont en cours d’exécution quand des connexions sont supprimées, la durée de cette étape peut prendre plus de temps pour permettre la récupération des transactions abandonnées. La [récupération de base de données accélérée](../accelerated-database-recovery.md) permet de réduire l’impact de l’abandon des transactions durables.

> [!IMPORTANT]
> Aucune donnée n’est perdue au cours des étapes du workflow.

### <a name="latency-of-changing-service-tier-or-rescaling-compute-size"></a>Latence de modification du niveau de service ou de remise à l’échelle de la taille de calcul

La latence estimée pour modifier le niveau de service, mettre à l’échelle la taille de calcul d’une base de données unique ou d’un pool élastique, déplacer une base de données dans un pool élastique ou hors de celui-ci, ou déplacer une base de données entre des pools élastiques est paramétrée comme suit :

|Niveau de service|Base de données unique de base,</br>Standard (S0-S1)|Pool élastique de base,</br>Standard (S2-S12), </br>Base de données unique ou pool élastique à usage général|Base de données unique ou pool élastique Premium ou critique pour l’entreprise|Hyperscale
|:---|:---|:---|:---|:---|
|**Base de données unique de base,</br> Standard (S0-S1)**|&bull; &nbsp;Latence constante indépendante de l’espace utilisé</br>&bull; &nbsp;Généralement moins de 5 minutes|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|
|**Pool élastique de base, </br>Standard (S2-S12), </br>Base de données unique ou pool élastique à usage général**|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Pour les bases de données individuelles, latence constante indépendante de l’espace utilisé</br>&bull; &nbsp;En règle générale, moins de 5 minutes pour les bases de données individuelles</br>&bull; &nbsp;Pour les pools élastiques, proportionnelle au nombre de bases de données|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|
|**Base de données unique ou pool élastique Premium ou critique pour l’entreprise**|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|
|**Hyperscale**|N/A|N/A|N/A|&bull; &nbsp;Latence constante indépendante de l’espace utilisé</br>&bull; &nbsp;Généralement moins de 2 minutes|

> [!NOTE]
>
> - En cas de modification du niveau de service ou de remise à l’échelle de la taille de calcul pour un pool élastique, la somme de l’espace utilisé sur toutes les bases de données du pool doit être utilisée pour calculer l’estimation.
> - En cas de déplacement d’une base de données vers/à partir d’un pool élastique, seul l’espace utilisé par la base de données a un impact sur la latence, pas l’espace utilisé par le pool élastique.
> - Pour des pools élastiques standard et à usage général, la latence de déplacement d’une base de données à l’intérieur ou à l’extérieur d’un pool élastique, ou entre des pools élastiques, sera proportionnelle à la taille de la base de données si le pool élastique utilise un stockage de partage de fichiers Premium ([PFS](../../storage/files/storage-files-introduction.md)). Pour déterminer si un pool utilise un stockage PFS, exécutez la requête suivante dans le contexte d’une base de données quelconque du pool. Si la valeur de la colonne AccountType est `PremiumFileStorage` ou `PremiumFileStorage-ZRS`, le pool utilise un stockage PFS.

```sql
SELECT s.file_id,
       s.type_desc,
       s.name,
       FILEPROPERTYEX(s.name, 'AccountType') AS AccountType
FROM sys.database_files AS s
WHERE s.type_desc IN ('ROWS', 'LOG');
```

> [!TIP]
> Pour surveiller des opérations en cours, voir : [Gérer des opérations à l’aide de l’API REST SQL](/rest/api/sql/operations/list), [Gérer des opérations à l’aide de l’interface de ligne de commande](/cli/azure/sql/db/op), [Surveiller des opérations à l’aide de T-SQL](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database) et les deux commandes PowerShell suivantes : [Get-AzSqlDatabaseActivity](/powershell/module/az.sql/get-azsqldatabaseactivity) et [Stop-AzSqlDatabaseActivity](/powershell/module/az.sql/stop-azsqldatabaseactivity).

### <a name="additional-considerations-when-changing-service-tier-or-rescaling-compute-size"></a>Autres considérations liées à la modification du niveau de service ou à la remise à l’échelle de la taille de calcul

- Lors de la réduction des vCores ou eDTU d’un pool élastique, l’espace utilisé par le pool doit être inférieur à la taille maximale autorisée du niveau de service cible et des eDTU du pool.
- Lors de la remise à l’échelle des eDTU pour un pool élastique, un coût de stockage supplémentaire s’applique si (1) la taille de stockage maximale du pool est prise en charge par le pool cible, et si (2) la taille maximale de stockage dépasse la quantité de stockage incluse dans le pool cible. Par exemple, si un pool Standard de 100 eDTU avec une taille maximale de 100 Go est rétrogradé en pool Standard de 50 eDTU, un coût de stockage supplémentaire s’applique, car le pool cible prend en charge une taille maximale de 100 Go et la quantité de stockage incluse est de seulement 50 Go. Par conséquent, la quantité de stockage supplémentaire est de 100 Go - 50 Go = 50 Go. Pour la tarification du stockage supplémentaire, consultez [Tarification des bases de données SQL](https://azure.microsoft.com/pricing/details/sql-database/). Si la quantité réelle d’espace utilisé est inférieure à la quantité de stockage incluse, ce coût supplémentaire peut être évité en réduisant la taille maximale de la base de données à la quantité incluse.

### <a name="billing-during-rescaling"></a>Facturation pendant la remise à l’échelle

Vous êtes facturé pour chaque heure d’existence de la base de données avec le niveau de service le plus élevé, la taille de calcul appliquée pendant cette heure quel que soit l’usage, ou si la base de données a été active pendant moins d’une heure. Par exemple, si vous avez créé une base de données unique et que vous l’avez supprimée cinq minutes après, votre facture mentionne le coût d’une heure de base de données.

## <a name="change-elastic-pool-storage-size"></a>modifier la taille de stockage de pool élastique

> [!IMPORTANT]
> Dans certaines circonstances, vous devrez peut-être réduire une base de données pour récupérer l’espace inutilisé. Pour plus d’informations, consultez [Gérer l’espace des fichiers dans Azure SQL Database](file-space-manage.md).

### <a name="vcore-based-purchasing-model"></a>Modèle d’achat vCore

- Le stockage peut être approvisionné jusqu’à la limite de taille maximale :

  - Pour le stockage des niveaux de service Standard et Usage général, augmentez ou diminuez la taille par incrément de 10 Go.
  - Pour le stockage des niveaux de service Premium et Critique pour l’entreprise, augmentez ou diminuez la taille par incrément de 250 Go.
- Le stockage pour un pool élastique peut être configuré en augmentant ou diminuant sa taille maximale.
- Le prix du stockage pour un pool élastique est égal à la quantité de stockage multiplié par le prix unitaire du stockage pour le niveau de service. Pour plus d’informations sur le prix du stockage supplémentaire, consultez [Tarification des bases de données SQL](https://azure.microsoft.com/pricing/details/sql-database/).

> [!IMPORTANT]
> Dans certaines circonstances, vous devrez peut-être réduire une base de données pour récupérer l’espace inutilisé. Pour plus d’informations, consultez [Gérer l’espace des fichiers dans Azure SQL Database](file-space-manage.md).

### <a name="dtu-based-purchasing-model"></a>Modèle d’achat DTU

- Le prix des eDTU pour un pool élastique inclut une certaine quantité de stockage sans coût supplémentaire. Un espace de stockage en plus du volume inclus peut être approvisionné pour un coût supplémentaire jusqu’à la limite de taille par incréments de 250 Go jusqu’à 1 To, puis par incréments de 256 Go au-delà de 1 To. Pour les quantités de stockage et limites de taille maximale incluses, consultez [Pool élastique : tailles de stockage et tailles de calcul](resource-limits-dtu-elastic-pools.md#elastic-pool-storage-sizes-and-compute-sizes).
- Vous pouvez configurer du stockage supplémentaire pour un pool élastique en augmentant sa taille maximale à l’aide du [portail Azure](elastic-pool-manage.md#azure-portal), de [PowerShell](/powershell/module/az.sql/Get-AzSqlElasticPool), [d’Azure CLI](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-update) ou de [l’API REST](/rest/api/sql/elasticpools/update).
- Le prix de l’espace de stockage supplémentaire pour un pool élastique est égal à la quantité de stockage supplémentaire multiplié par le prix unitaire du stockage supplémentaire pour le niveau de service. Pour plus d’informations sur le prix du stockage supplémentaire, consultez [Tarification des bases de données SQL](https://azure.microsoft.com/pricing/details/sql-database/).

> [!IMPORTANT]
> Dans certaines circonstances, vous devrez peut-être réduire une base de données pour récupérer l’espace inutilisé. Pour plus d’informations, consultez [Gérer l’espace des fichiers dans Azure SQL Database](file-space-manage.md).

## <a name="next-steps"></a>Étapes suivantes

Pour les limites de ressources globales, consultez [SQL Database vCore-based resource limits - elastic pools](resource-limits-vcore-elastic-pools.md) (limites de ressource vCore SQL Database - pools élastiques) et [SQL Database DTU-based resource limits - elastic pools](resource-limits-dtu-elastic-pools.md) (limites de ressource DTU SQL Database - pools élastiques).