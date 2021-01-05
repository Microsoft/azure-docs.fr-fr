---
title: Mettre à l’échelle les ressources d’une base de données unique
description: Cet article décrit comment mettre à l’échelle les ressources de calcul et de stockage disponibles d’une base de données unique dans Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1, references_regions
ms.devlang: ''
ms.topic: conceptual
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: sstein
ms.date: 09/16/2020
ms.openlocfilehash: da3c70baccc3c86f2ac57d61539456464e3042b6
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96493404"
---
# <a name="scale-single-database-resources-in-azure-sql-database"></a>Mettre à l’échelle des ressources de base de données unique dans Azure SQL Database

Cet article décrit la procédure de mise à l’échelle des ressources de calcul et de stockage disponibles pour Azure SQL Database dans le niveau de calcul configuré. Sinon, le [niveau de calcul serverless](serverless-tier-overview.md) offre une mise à l’échelle automatique du calcul et facture le calcul utilisé par seconde.

Après la sélection initiale du nombre de vCores ou d’unités de transaction de base de données (DTU), vous pouvez monter ou descendre en puissance une base de données unique de façon dynamique en fonction de l’expérience réelle à l’aide du [Portail Azure](single-database-manage.md#the-azure-portal), de [Transact-SQL](/sql/t-sql/statements/alter-database-transact-sql#examples-1), de [PowerShell](/powershell/module/az.sql/set-azsqldatabase), [d’Azure CLI](/cli/azure/sql/db#az-sql-db-update) ou de [l’API REST](/rest/api/sql/databases/update).

La vidéo suivante montre la modification dynamique du niveau de service et de la taille de calcul pour augmenter les DTU disponibles pour une base de données unique.

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-SQL-Database-dynamically-scale-up-or-scale-down/player]

> [!IMPORTANT]
> Dans certaines circonstances, vous devrez peut-être réduire une base de données pour récupérer l’espace inutilisé. Pour plus d’informations, consultez [Gérer l’espace des fichiers dans Azure SQL Database](file-space-manage.md).

## <a name="impact"></a>Impact

La modification du niveau de service ou de la taille de calcul implique principalement que le service suive les étapes ci-après :

1. Création d’une instance de calcul pour la base de données. 

    Une nouvelle instance de calcul est créée pour le niveau de service et la taille de calcul demandés. Pour certaines combinaisons de modifications du niveau de service et de la taille de calcul, un réplica de la base de données doit être créé dans la nouvelle instance de calcul, ce qui implique de copier des données et peut avoir une incidence substantielle sur la latence globale. Malgré tout, la base de données reste en ligne pendant cette étape, et les connexions continuent d’être dirigées vers la base de données dans l’instance de calcul d’origine.

2. Basculement de l’acheminement des connexions vers la nouvelle instance de calcul.

    Les connexions existantes à la base de données dans l’instance de calcul d’origine sont supprimées. Toutes les nouvelles connexions sont établies vers la base de données de la nouvelle instance de calcul. Pour certaines combinaisons de modifications du niveau de service et de la taille de calcul, les fichiers de base de données sont détachés, puis rattachés pendant ce basculement.  Malgré tout, le basculement peut entraîner une brève interruption de service au cours de laquelle la base de données reste inaccessible pendant moins de 30 secondes, et le plus souvent, pendant quelques secondes seulement. Si des transactions longues sont en cours d’exécution quand des connexions sont supprimées, la durée de cette étape peut prendre plus de temps pour permettre la récupération des transactions abandonnées. La [récupération de base de données accélérée](../accelerated-database-recovery.md) permet de réduire l’impact de l’abandon des transactions durables.

> [!IMPORTANT]
> Aucune donnée n’est perdue au cours des étapes du workflow. Assurez-vous que vous avez implémenté une [logique de nouvelle tentative](troubleshoot-common-connectivity-issues.md) dans les applications et les composants qui utilisent Azure SQL Database pendant la modification du niveau de service.

## <a name="latency"></a>Latence

La latence estimée pour modifier le niveau de service, mettre à l’échelle la taille de calcul d’une base de données unique ou d’un pool élastique, déplacer une base de données dans un pool élastique ou hors de celui-ci, ou déplacer une base de données entre des pools élastiques est paramétrée comme suit :

|Niveau de service|Base de données unique de base,</br>Standard (S0-S1)|Pool élastique de base,</br>Standard (S2-S12), </br>Base de données unique ou pool élastique à usage général|Base de données unique ou pool élastique Premium ou critique pour l’entreprise|Hyperscale
|:---|:---|:---|:---|:---|
|**Base de données unique de base,</br> Standard (S0-S1)**|&bull; &nbsp;Latence constante indépendante de l’espace utilisé</br>&bull; &nbsp;Généralement moins de 5 minutes|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|
|**Pool élastique de base, </br>Standard (S2-S12), </br>Base de données unique ou pool élastique à usage général**|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Pour les bases de données uniques, latence constante indépendante de l’espace utilisé</br>&bull; &nbsp;En règle générale, moins de 5 minutes pour les bases de données uniques</br>&bull; &nbsp;Pour les pools élastiques, proportionnelle au nombre de bases de données|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|
|**Base de données unique ou pool élastique Premium ou critique pour l’entreprise**|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|&bull; &nbsp;Latence proportionnelle à l’espace de base de données utilisé en raison de la copie des données</br>&bull; &nbsp;En règle générale, moins de 1 minute par Go d’espace utilisé|
|**Hyperscale**|N/A|N/A|N/A|&bull; &nbsp;Latence constante indépendante de l’espace utilisé</br>&bull; &nbsp;Généralement moins de 2 minutes|

> [!NOTE]
> En outre, pour des bases de données standard (S2-S12) et à usage général, la latence du déplacement d’une base de données à l’intérieur ou à l’extérieur d’un pool élastique, ou entre des pools élastiques, sera proportionnelle à la taille de la base de données si celle-ci utilise un stockage de partage de fichiers Premium ([PFS](../../storage/files/storage-files-introduction.md)).
>
> Pour déterminer si une base de données utilise un stockage PFS, exécutez la requête suivante dans le contexte de la base de données. Si la valeur de la colonne AccountType est `PremiumFileStorage` ou `PremiumFileStorage-ZRS`, la base de données utilise un stockage PFS.
 
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

## <a name="cancelling-changes"></a>Annulation des modifications

Vous pouvez annuler une modification du niveau de service ou une opération de remise à l’échelle du calcul.

### <a name="the-azure-portal"></a>Le portail Azure

Dans le panneau de vue d’ensemble de la base de données, accédez à **Notifications**, puis cliquez sur la vignette indiquant qu’une opération est en cours :

![Opération en cours](./media/single-database-scale/ongoing-operations.png)

Ensuite, cliquez sur le bouton intitulé **Annuler cette opération**.

![Annulation de l’opération en cours](./media/single-database-scale/cancel-ongoing-operation.png)

### <a name="powershell"></a>PowerShell

À partir d’une invite de commande PowerShell, définissez les valeurs `$resourceGroupName`, `$serverName` et `$databaseName`, puis exécutez la commande suivante :

```azurecli
$operationName = (az sql db op list --resource-group $resourceGroupName --server $serverName --database $databaseName --query "[?state=='InProgress'].name" --out tsv)
if (-not [string]::IsNullOrEmpty($operationName)) {
    (az sql db op cancel --resource-group $resourceGroupName --server $serverName --database $databaseName --name $operationName)
        "Operation " + $operationName + " has been canceled"
}
else {
    "No service tier change or compute rescaling operation found"
}
```

## <a name="additional-considerations"></a>Considérations supplémentaires

- Si vous effectuez la mise à niveau vers un niveau de service ou une taille de calcul supérieurs, la taille maximale de la base de données n’augmente pas, à moins que vous n’en fassiez la demande (maxsize).
- Pour pouvoir passer à une version antérieure, l’espace utilisé par la base de données doit être inférieur à la taille maximale autorisée pour le service cible et la taille de calcul.
- Lorsque vous rétrogradez du niveau **Premium** vers le niveau **Standard**, un coût de stockage supplémentaire s’applique si (1) la taille maximale de la base de données est prise en charge dans la taille de calcul cible et (2) la taille maximale dépasse la quantité de stockage incluse dans la taille de calcul cible. Par exemple, si une base de données P1 avec une taille maximale de 500 Go est rétrogradée en S3, un coût de stockage supplémentaire s’applique, car S3 prend en charge une taille maximale de 1 To et la quantité de stockage incluse est de seulement 250 Go. Par conséquent, la quantité de stockage supplémentaire est de 500 Go - 250 Go = 250 Go. Pour la tarification du stockage supplémentaire, consultez [Tarification d'Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/). Si la quantité réelle d’espace utilisé est inférieure à la quantité de stockage incluse, ce coût supplémentaire peut être évité en réduisant la taille maximale de la base de données à la quantité incluse.
- Lors de la mise à niveau d’une base de données pour laquelle la [géoréplication](active-geo-replication-configure-portal.md) est activée, vous devez commencer par mettre à niveau les bases de données secondaires associées vers le niveau de service et la taille de calcul souhaités avant de procéder à la mise à niveau de la base de données primaire (conseil général pour de meilleures performances). Lors de la mise à niveau vers une autre édition, il est impératif que la base de données secondaire soit mise à niveau en premier.
- Lors de la mise à niveau descendante d’une base de données pour laquelle la [géoréplication](active-geo-replication-configure-portal.md) est activée, vous devez commencer par mettre à niveau les bases de données primaires associées vers le niveau de service et la taille de calcul inférieurs souhaités avant de procéder à la mise à niveau descendante de la base de données secondaire (conseil général pour de meilleures performances). Lors de la rétrogradation vers une autre édition, il est impératif que la base de données primaire soit rétrogradée en premier.
- Les offres de service de restauration sont différentes selon les niveaux de service. Si vous repassez au niveau de service **De base**, la rétention des fichiers de sauvegarde sera de plus courte durée. Consultez l’article relatif aux [sauvegardes Azure SQL Database](automated-backups-overview.md).
- Les nouvelles propriétés de la base de données ne sont appliquées qu’une fois les modifications terminées.

## <a name="billing"></a>Facturation

Vous êtes facturé pour chaque heure d’existence de la base de données avec le niveau de service le plus élevé, la taille de calcul appliquée pendant cette heure, quel que soit l’usage, ou si la base de données a été active pendant moins d’une heure. Par exemple, si vous avez créé une base de données unique et que vous l’avez supprimée cinq minutes après, votre facture mentionne le coût d’une heure de base de données.

## <a name="change-storage-size"></a>Modifier la taille de stockage

### <a name="vcore-based-purchasing-model"></a>Modèle d’achat vCore

- Le stockage peut être configuré jusqu’à la limite de taille maximale du stockage de données par incréments de 1 Go. Le stockage de données minimum configurable est de 1 Go. Consultez les pages de la documentation consacrées aux limites de ressources des [bases de données uniques](resource-limits-vcore-single-databases.md) et des [pools élastiques](resource-limits-vcore-elastic-pools.md) pour connaître les limites de taille maximale du stockage de données de chaque objectif de service.
- Vous pouvez configurer le stockage de données d'une base de données unique en augmentant ou en diminuant sa taille maximale à l'aide du [portail Azure](https://portal.azure.com), de [Transact-SQL](/sql/t-sql/statements/alter-database-transact-sql#examples-1), de [PowerShell](/powershell/module/az.sql/set-azsqldatabase), d'[Azure CLI](/cli/azure/sql/db#az-sql-db-update) ou de l'[API REST](/rest/api/sql/databases/update). Si la valeur de taille maximale est spécifiée en octets, elle doit être un multiple de 1 Go (1073741824 octets).
- Le volume de données qui peut être stocké dans les fichiers de données d'une base de données est limité par la taille maximale configurée pour le stockage de données. En plus de ce stockage, Azure SQL Database alloue automatiquement 30 % de stockage supplémentaire pour le journal des transactions.
- Azure SQL Database alloue automatiquement 32 Go par vCore pour la base de données `tempdb`. Quel que soit le niveau de service, `tempdb` se trouve toujours sur le stockage SSD local.
- Le prix du stockage d'une base de données unique ou d'un pool élastique est égal à la somme des volumes de stockage de données et de stockage des journaux de transactions multipliée par le prix unitaire du stockage du niveau de service. Le coût de `tempdb` est inclus dans le prix. Pour plus d'informations sur le prix du stockage, consultez [Tarification d'Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/).

> [!IMPORTANT]
> Dans certaines circonstances, vous devrez peut-être réduire une base de données pour récupérer l’espace inutilisé. Pour plus d’informations, consultez [Gérer l’espace des fichiers dans Azure SQL Database](file-space-manage.md).

### <a name="dtu-based-purchasing-model"></a>Modèle d’achat DTU

- Le prix des DTU pour une base de données unique inclut une certaine quantité de stockage sans coût supplémentaire. Un espace de stockage en plus du volume inclus peut être approvisionné pour un coût supplémentaire jusqu’à la limite de taille par incréments de 250 Go jusqu’à 1 To, puis par incréments de 256 Go au-delà de 1 To. Pour les quantités de stockage et limites de taille maximale incluses, voir l’article [Base de données unique : tailles de stockage et tailles de calcul](resource-limits-dtu-single-databases.md#single-database-storage-sizes-and-compute-sizes).
- Vous pouvez approvisionner un espace de stockage supplémentaire pour une base de données unique en augmentant la taille maximale à l’aide du Portail Azure, de [Transact-SQL](/sql/t-sql/statements/alter-database-transact-sql#examples-1), de [PowerShell](/powershell/module/az.sql/set-azsqldatabase), [d’Azure CLI](/cli/azure/sql/db#az-sql-db-update) ou de [l’API REST](/rest/api/sql/databases/update).
- Le prix de l’espace de stockage supplémentaire pour une base de données unique est égal à la quantité de stockage supplémentaire multiplié par le prix unitaire du stockage supplémentaire pour le niveau de service. Pour plus d’informations sur le prix du stockage supplémentaire, consultez [Tarification d'Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/).

> [!IMPORTANT]
> Dans certaines circonstances, vous devrez peut-être réduire une base de données pour récupérer l’espace inutilisé. Pour plus d’informations, consultez [Gérer l’espace des fichiers dans Azure SQL Database](file-space-manage.md).

### <a name="geo-replicated-database"></a>Base de données géorépliquée

Pour changer la taille d’une base de données secondaire répliquée, changez la taille de la base de données primaire. Cette modification est ensuite répliquée et implémentée également sur la base de données secondaire.

## <a name="p11-and-p15-constraints-when-max-size-greater-than-1-tb"></a>Contraintes P11 et P15 lorsque la taille maximale est supérieure à 1 To

Un espace de stockage supérieur à 1 To au niveau Premium est actuellement disponible dans les toutes régions sauf les suivantes : Chine Est, Chine Nord, Allemagne Centre et Allemagne Nord-Est. Dans ces régions, l’espace de stockage maximal au niveau Premium est limité à 1 To. Les considérations et limitations suivantes s’appliquent aux bases de données P11 et P15 avec une taille maximale supérieure à 1 To :

- Si une base de données P11 ou P15 présentait une taille maximale supérieure à 1 To, vous pouvez uniquement la restaurer ou la copier sous la forme d’une base de données P11 ou P15.  Par la suite, vous pourrez remettre à l’échelle la base de données avec une autre taille de calcul à condition que l’espace alloué lors de cette opération ne dépasse pas les limites de taille maximale de la nouvelle taille de calcul.
- Pour les scénarios de géoréplication active :
  - Configuration d’une relation de géoréplication : Si la base de données primaire est P11 ou P15, les bases de données secondaires doivent également être P11 ou P15. Les bases de données dont la taille de calcul est inférieure sont rejetées en tant que réplicas secondaires, car elles ne peuvent pas prendre en charge plus de 1 To.
  - Mise à niveau de la base de données primaire dans une relation de géoréplication : Le fait de modifier la taille maximale de la base de données primaire pour plus de 1 To déclenche la même modification sur la base de données secondaire. Les deux mises à niveau doivent aboutir pour que la modification sur la base de données principale prenne effet. Des limitations de région pour les options de plus de 1 To s’appliquent. Si la base de données secondaire se situe dans une région qui ne prend pas en charge plus de 1 To, la base de données primaire n’est pas mise à niveau.
- L’utilisation du service Import/Export pour le chargement des bases de données P11/P15 avec plus de 1 To n’est pas prise en charge. Utilisez SqlPackage.exe pour [importer](database-import.md) et [exporter](database-export.md) les données.

## <a name="next-steps"></a>Étapes suivantes

Pour les limites de ressources globales, consultez [Limites de ressources vCore Azure SQL Database - bases de données uniques](resource-limits-vcore-single-databases.md) et [Limites de ressources DTU Azure SQL Database - bases de données uniques](resource-limits-dtu-single-databases.md).
