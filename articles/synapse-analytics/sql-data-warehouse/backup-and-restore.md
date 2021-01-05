---
title: Sauvegarde et restauration - instantanés, géo-redondant
description: Découvrez comment la sauvegarde et la restauration fonctionnent dans un pool SQL dédié Azure Synapse Analytics. Utilisez des sauvegardes pour restaurer votre entrepôt de données à un point de restauration dans la région primaire. Utilisez des sauvegardes géoredondantes pour restaurer dans une autre région géographique.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/13/2020
ms.author: joanpo
ms.reviewer: igorstan
ms.custom: seo-lt-2019"
ms.openlocfilehash: 254e07d5a3266927c9677107772b4ac2dbfd2ce0
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96454402"
---
# <a name="backup-and-restore-in-azure-synapse-dedicated-sql-pool"></a>Sauvegarde et restauration dans un pool SQL dédié Azure Synapse

Découvrez comment utiliser la sauvegarde et la restauration dans un pool SQL dédié Azure Synapse. Utilisez des points de restauration de pool SQL dédié pour récupérer ou copier votre entrepôt de données dans un état précédent dans la région primaire. Utilisez des sauvegardes géoredondantes d’entrepôts de données pour effectuer une restauration dans une autre région géographique.

## <a name="what-is-a-data-warehouse-snapshot"></a>Qu’est-ce qu’une capture instantanée d’entrepôt de données ?

Une *capture instantanée d’entrepôt de données* crée un point de restauration que vous pouvez utiliser pour récupérer ou copier votre entrepôt de données dans un état antérieur.  Comme un pool SQL dédié est un système distribué, une capture instantanée d’entrepôt de données est constituée de nombreux fichiers qui sont stockés dans le Stockage Azure. Les captures instantanées capturent les changements incrémentiels à partir des données stockées dans votre entrepôt de données.

Une *restauration d’entrepôt de données* est un nouvel entrepôt de données créé à partir d’un point de restauration d’un entrepôt de données existant ou supprimé. La restauration de votre entrepôt de données est une partie essentielle de toute stratégie de continuité d’activité ou de récupération d’urgence, dans la mesure où elle recrée vos données après des corruptions et des suppressions accidentelles. Un entrepôt de données est également un mécanisme puissant pour créer des copies de votre entrepôt de données à des fins de test ou de développement. Les taux de restauration de pool SQL dédié peuvent varier en fonction de la taille de la base de données et de l’emplacement des entrepôts de données source et cible.

## <a name="automatic-restore-points"></a>Points de restauration automatiques

La fonctionnalité intégrée de captures instantanées crée des points de restauration. Il n’est pas nécessaire de l’activer. Toutefois, le pool SQL dédié doit être dans un état actif pour permettre la création de points de restauration. S’il est fréquemment suspendu, il est possible que les points de restauration automatiques ne soient pas créés. Veillez donc à créer un point de restauration défini par l’utilisateur avant de suspendre le pool SQL dédié. Les points de restauration automatiques ne peuvent actuellement pas être supprimés par les utilisateurs étant donné que le service utilise ces points de restauration pour conserver les contrats SLA pour la récupération.

Des captures instantanées de votre entrepôt de données sont prises pendant la journée, créant des points de restauration qui sont disponibles pendant sept jours. Cette période de conservation ne peut pas être modifiée. Le pool SQL dédié prend en charge un objectif de point de récupération (RPO) de huit heures. Vous pouvez restaurer votre entrepôt de données dans la région primaire à partir de n’importe quelle capture instantanée prise au cours des sept derniers jours.

Pour voir quand la dernière capture instantanée a démarré, exécutez cette requête sur votre pool SQL dédié en ligne.

```sql
select   top 1 *
from     sys.pdw_loader_backup_runs
order by run_id desc
;
```

## <a name="user-defined-restore-points"></a>Points de restauration définis par l’utilisateur

Cette fonctionnalité vous permet de déclencher manuellement des captures instantanées pour créer des points de restauration de votre entrepôt de données avant et après des modifications importantes. Cette fonctionnalité garantit que les points de restauration sont logiquement cohérents, ce qui renforce la protection des données en cas d’interruptions de la charge de travail ou d’erreurs d’utilisateur pendant le temps de récupération rapide. Les points de restauration définis par l’utilisateur sont disponibles pendant sept jours et sont automatiquement supprimés pour votre compte. Vous ne pouvez pas changer la période de conservation des points de restauration définis par l’utilisateur. **42 points de restauration définis par l’utilisateur** sont garantis à un instant donné. Ils doivent donc être [supprimés](https://go.microsoft.com/fwlink/?linkid=875299) avant qu’un autre point de restauration soit créé. Vous pouvez déclencher des captures instantanées pour créer des points de restauration définis par l’utilisateur par le biais de [PowerShell](/powershell/module/az.sql/new-azsqldatabaserestorepoint?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.jsont#examples) ou du portail Azure.

> [!NOTE]
> Si vous avez besoin de points de restauration de plus de 7 jours, votez pour cette fonctionnalité [ici](https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/35114410-user-defined-retention-periods-for-restore-points). Vous pouvez également créer un point de restauration défini par l’utilisateur et effectuer une restauration à partir du point de restauration nouvellement créé dans un nouvel entrepôt de données. Une fois la restauration effectuée, vous avez le pool SQL dédié en ligne et vous pouvez le suspendre indéfiniment pour réduire les coûts de calcul. La base de données en pause entraîne des frais de stockage aux tarifs du Stockage Premium Azure. Si vous avez besoin d’une copie active de l’entrepôt de données restauré, vous pouvez reprendre, ce qui ne doit prendre que quelques minutes.

### <a name="restore-point-retention"></a>Conservation des points de restauration

La section suivante contient plus d’informations sur les périodes de rétention des points de restauration :

1. Le pool SQL dédié supprime un point de restauration à l’issue de la période de conservation de 7 jours **et** quand il y a au moins 42 points de restauration au total (points de restauration définis par l’utilisateur et points de restauration automatiques).
2. Aucune capture instantanée n’est prise quand un pool SQL dédié est suspendu.
3. L’âge d’un point de restauration est mesuré selon les jours calendaires absolus à partir du moment où le point de restauration est pris, ce qui inclut le moment où le pool SQL est suspendu.
4. À tout moment, un pool SQL dédié peut stocker jusqu’à 42 points de restauration définis par l’utilisateur et 42 points de restauration automatiques, à condition que ces points de restauration n’aient pas atteint la période de conservation de 7 jours
5. Si une capture instantanée est prise, le pool SQL dédié est alors suspendu pendant plus de 7 jours, puis son activité est reprise. Le point de restauration persiste jusqu’à ce qu’il y ait 42 points de restauration au total (points de restauration définis par l’utilisateur et points de restauration automatiques)

### <a name="snapshot-retention-when-a-sql-pool-is-dropped"></a>Conservation des captures instantanées quand un pool SQL est supprimé

Quand vous supprimez un pool SQL dédié, une capture instantanée finale est créée et enregistrée pendant sept jours. Vous pouvez restaurer le pool SQL dédié au point de restauration final créé au moment de la suppression. Si le pool SQL dédié est supprimé dans l’état suspendu, aucune capture instantanée n’est prise. Dans ce scénario, veillez à créer un point de restauration défini par l’utilisateur avant de supprimer le pool SQL dédié.

> [!IMPORTANT]
> Si vous supprimez le serveur/l’espace de travail qui héberge un pool SQL dédié, toutes les bases de données qui appartiennent au serveur/à l’espace de travail sont également supprimées, sans pouvoir être restaurées. Vous ne pouvez pas restaurer un serveur supprimé.

## <a name="geo-backups-and-disaster-recovery"></a>Sauvegardes de géoréplication et récupération d’urgence

Une géosauvegarde est créée une fois par jour dans un [centre de données associé](../../best-practices-availability-paired-regions.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Le RPO pour une géo-restauration est de 24 heures. Vous pouvez restaurer la géosauvegarde sur un serveur dans n’importe quelle autre région où le pool SQL dédié est pris en charge. Une géosauvegarde vous garantit de pouvoir restaurer un entrepôt de données dans le cas où vous ne pouvez pas accéder aux points de restauration de votre région primaire.

> [!NOTE]
> Si vous avez besoin d’un objectif de point de récupération (RPO) plus court pour les géosauvegardes, votez pour cette fonctionnalité [ici](https://feedback.azure.com/forums/307516-sql-data-warehouse). Vous pouvez également créer un point de restauration défini par l’utilisateur et effectuer une restauration à partir du point de restauration nouvellement créé dans un nouvel entrepôt de données se trouvant dans une autre région. Une fois la restauration effectuée, vous retrouvez l’entrepôt de données en ligne que vous pouvez mettre en pause indéfiniment pour réduire les coûts de calcul. La base de données en pause entraîne des frais de stockage aux tarifs du Stockage Premium Azure. Si vous avez besoin d’une copie active de l’entrepôt de données, vous pouvez reprendre, ce qui ne doit prendre que quelques minutes.

## <a name="backup-and-restore-costs"></a>Coûts de sauvegarde et de restauration

Vous remarquerez que la facture Azure comprend un élément de ligne pour le Stockage et un élément de ligne pour le Stockage de reprise d’activité après sinistre. Les frais de stockage correspondent au coût total du stockage de vos données dans la région primaire et aux changements incrémentiels pris par des captures instantanés. Pour en savoir plus sur la facturation des captures instantanées, reportez-vous à [Facturation des captures instantanées](/rest/api/storageservices/Understanding-How-Snapshots-Accrue-Charges?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Les frais géoredondants couvrent le coût du stockage des géosauvegardes.  

Le coût total de votre entrepôt de données principal et de sept jours de changements de captures instantanées est arrondi au To le plus proche. Par exemple, si la taille de votre entrepôt de données est de 1,5 To et que les captures instantanées occupent 100 Go, vous êtes facturé pour 2 To de données aux prix du Stockage Premium Azure.

Si vous utilisez le stockage géoredondant, vous payez des frais de stockage distincts. Le stockage géoredondant est facturé au prix standard du stockage géoredondant avec accès en lecture (RA-GRS).

Pour plus d’informations sur la tarification Azure Synapse, consultez [Tarification Azure Synapse Analytics](https://azure.microsoft.com/pricing/details/sql-data-warehouse/gen2/). La sortie des données ne vous est pas facturée lors de la restauration dans différentes régions.

## <a name="restoring-from-restore-points"></a>Restauration à partir de points de restauration

Chaque capture instantanée crée un point de restauration qui représente son heure de début. Pour restaurer un entrepôt de données, choisissez un point de restauration et émettez une commande de restauration.  

Vous pouvez conserver les entrepôts de données restauré et actuel ou les supprimer tous les deux. Si vous voulez remplacer l’entrepôt de données actuel par l’entrepôt de données restauré, vous pouvez le renommer à l’aide d’[ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) avec l’option MODIFY NAME.

Pour restaurer un entrepôt de données, consultez [Restaurer un pool SQL dédié](sql-data-warehouse-restore-points.md#create-user-defined-restore-points-through-the-azure-portal).

Si vous devez restaurer un entrepôt de données supprimé ou suspendu, vous pouvez [créer un ticket de support](sql-data-warehouse-get-started-create-support-ticket.md).

## <a name="cross-subscription-restore"></a>Restauration avec plusieurs abonnements

Si vous devez effectuer une restauration directe dans plusieurs abonnements, votez pour cette fonctionnalité [ici](https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/36256231-enable-support-for-cross-subscription-restore). Procédez à la restauration sur un autre serveur et [« déplacez »](../../azure-resource-manager/management/move-resource-group-and-subscription.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) le serveur entre les abonnements pour effectuer une restauration inter-abonnements.

## <a name="geo-redundant-restore"></a>Restauration géoredondante

Vous pouvez [restaurer votre pool SQL dédié](sql-data-warehouse-restore-from-geo-backup.md#restore-from-an-azure-geographical-region-through-powershell) dans n’importe quelle région prenant en charge le pool SQL dédié au niveau de performances que vous avez choisi.

> [!NOTE]
> Pour effectuer une restauration géoredondante, vous devez avoir activé cette fonctionnalité.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les points de restauration, consultez [Points de restauration définis par l’utilisateur](sql-data-warehouse-restore-points.md)
