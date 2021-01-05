---
title: Restaurer une base de données à partir d’un fichier de sauvegarde
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Découvrez la limite de restauration dans le temps, qui vous permet de restaurer une base de données dans Azure SQL Database ou une instance dans Azure SQL Managed Instance jusqu’à 35 jours en arrière.
services: sql-database
ms.service: sql-db-mi
ms.subservice: service
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, sstein, danil
ms.date: 11/13/2020
ms.openlocfilehash: 138cd03989008aa4fbd7d6ca3e06aff94fa76e33
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94989443"
---
# <a name="recover-using-automated-database-backups---azure-sql-database--sql-managed-instance"></a>Récupération à l’aide de sauvegardes de bases de données automatisées - Azure SQL Database et SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Les options suivantes sont disponibles pour la récupération des bases de données à l’aide des [sauvegardes de bases de données automatisées](automated-backups-overview.md) : Vous pouvez :

- Créer une nouvelle base de données sur le même serveur, récupérée à un moment donné durant la période de rétention.
- Créer une base de données sur le même serveur, récupérée à un moment donné de la suppression d’une base de données.
- Créer une nouvelle base de données sur un serveur dans la même région, récupérée au point des sauvegardes les plus récentes.
- Créer une nouvelle base de données sur tout serveur dans n’importe quelle autre région, récupérée au point des sauvegardes répliquées les plus récentes.

Si vous avez configuré la [conservation à long terme des sauvegardes](long-term-retention-overview.md), vous pouvez également créer une nouvelle base de données à partir de toute sauvegarde de conservation à long terme, sur n’importe quel serveur.

> [!IMPORTANT]
> Vous ne pouvez pas remplacer une base de données existante lors de la restauration.

Quand vous utilisez le niveau de service Standard ou Premium, la restauration de votre base de données peut entraîner un coût de stockage supplémentaire. Le coût supplémentaire s’applique si la taille maximale de la base de données restaurée est supérieure à la quantité de stockage incluse dans le niveau de service et de performance de la base de données cible. Pour les détails de la tarification du stockage supplémentaire, consultez la page [Tarification des bases de données SQL](https://azure.microsoft.com/pricing/details/sql-database/). Si la quantité réelle d’espace utilisé est inférieure à la quantité de stockage incluse, ce coût supplémentaire peut être évité en fixant la taille maximale de la base de données sur la quantité incluse.

## <a name="recovery-time"></a>Temps de récupération

Le temps de récupération pour restaurer une base de données à l’aide des sauvegardes de bases de données automatisées est affecté par plusieurs facteurs :

- La taille de la base de données
- La taille de calcul de la base de données
- Le nombre de journaux de transactions impliqués
- La quantité d’activité devant être relue pour effectuer une récupération au point de restauration
- La bande passante du réseau, si la restauration s’effectue dans une autre région
- le nombre de demandes de restauration simultanées en cours de traitement dans la région cible.

Pour une base de données volumineuse ou très active, la restauration peut prendre plusieurs heures. En cas de panne prolongée dans une région, il est possible qu’un grand nombre de requêtes de géo-restauration soient lancées pour la récupération d’urgence. S’il y a un grand nombre de requêtes, le temps de récupération des bases de données individuelles peut s’en trouver augmenté. La plupart des restaurations de bases de données s’effectuent en moins de 12 heures.

Pour un seul abonnement, le nombre de requêtes de restauration simultanées est limité. Ces limitations s’appliquent à toutes les combinaisons de limites de restauration dans le temps, de géorestaurations et de restaurations issues d’une sauvegarde de conservation à long terme.

| **Option de déploiement** | **Nombre maximum de requêtes simultanées traitées** | **Nombre maximum de requêtes simultanées soumises** |
| :--- | --: | --: |
|**Base de données unique (par abonnement)**|10|60|
|**Pool élastique (par pool)**|4|200|


Il n’existe aucune méthode intégrée permettant de restaurer l’intégralité du serveur. Pour obtenir un exemple d’exécution de cette tâche, consultez [Azure SQL Database : Full Server Recovery](https://gallery.technet.microsoft.com/Azure-SQL-Database-Full-82941666).

> [!IMPORTANT]
> Pour effectuer une récupération à l’aide de sauvegardes automatisées, vous devez être membre du rôle Contributeur SQL Server ou du rôle Contributeur SQL Managed Instance (selon la destination de la récupération) dans l’abonnement, ou être le propriétaire de l’abonnement. Pour plus d’informations, consultez [Azure RBAC : pour les ressources Azure](../../role-based-access-control/built-in-roles.md). Vous pouvez effectuer une récupération en utilisant le portail Azure, PowerShell ou l’API REST. Vous ne pouvez pas utiliser Transact-SQL.

## <a name="point-in-time-restore"></a>Restauration dans le temps

Pour restaurer une base de données autonome, mise en pool ou d’instance à un moment antérieur, vous pouvez utiliser le portail Azure, [PowerShell](/powershell/module/az.sql/restore-azsqldatabase) ou l’[API REST](/rest/api/sql/databases/createorupdate#creates-a-database-from-pointintimerestore.). La demande peut spécifier n’importe quel niveau de service ou taille de calcul pour la base de données restaurée. Assurez-vous d’avoir suffisamment de ressources sur le serveur vers lequel vous restaurez la base de données. 

Une fois terminée, la restauration crée une base de données sur le même serveur que la base de données d’origine. La base de données restaurée est facturée aux tarifs habituels, en fonction du niveau de service et de la taille de calcul. Aucun frais ne vous sera facturé jusqu’à ce que la restauration de la base de données soit terminée.

En règle générale, vous restaurez une base de données à un point antérieur à des fins de récupération. Il est possible d’utiliser la base de données restaurée pour remplacer la base de données d’origine ou comme source de données afin de mettre à jour la base de données d’origine.

- **Remplacement de la base de données**

  Si la base de données restaurée est destinée à remplacer la base de données d’origine, vous devez spécifier la taille de calcul et le niveau de service de la base de données d’origine. Vous pouvez alors renommer la base de données d’origine et donner le nom d’origine à la base de données restaurée avec la commande [ALTER DATABASE](/sql/t-sql/statements/alter-database-azure-sql-database) dans T-SQL.

- **Récupération des données**

  Si vous prévoyez récupérer des données de la base de données restaurée suite à une erreur due à l'utilisateur ou à l'application, écrivez et exécutez un script de récupération de données qui extrait des données de la base de données restaurée et les applique à la base de données d'origine. Bien que l’opération de restauration puisse prendre un certain temps, la base de données en cours de restauration sera visible dans la liste de bases de données pendant tout le processus de restauration. Si vous supprimez la base de données pendant la restauration, l’opération de restauration est annulée et la base de données concernée ne vous est pas facturée.
  
### <a name="point-in-time-restore-by-using-azure-portal"></a>Limite de restauration dans le temps à l’aide du portail Azure

Vous pouvez récupérer une base de données d’instance ou unique à un moment donné à partir du panneau de vue d’ensemble de la base de données que vous souhaitez restaurer dans le Portail Azure.

#### <a name="sql-database"></a>SQL Database

Pour récupérer une base de données à un point dans le temps à l'aide du portail Azure, ouvrez la page de présentation de la base de données et sélectionnez **Restaurer** sur la barre d'outils. Choisissez une source de sauvegarde et sélectionnez le point de sauvegarde dans le temps à partir duquel une nouvelle base de données sera créée.

  ![Capture d’écran des options de restauration de base de données pour SQL Database.](./media/recovery-using-backups/pitr-backup-sql-database-annotated.png)

#### <a name="sql-managed-instance"></a>Instance managée SQL

Pour récupérer une base de données d’instance managée à un point dans le temps à l’aide du portail Azure, ouvrez la page de vue d’ensemble de la base de données, puis sélectionnez **Restaurer** dans la barre d’outils. Choisissez le point de sauvegarde dans le temps à partir duquel une nouvelle base de données sera créée.

  ![Capture d’écran des options de restauration de base de données pour une instance gérée SQL.](./media/recovery-using-backups/pitr-backup-managed-instance-annotated.png)

> [!TIP]
> Pour restaurer par programmation une base de données à partir d’une sauvegarde, consultez [Exécution par programmation d’une récupération à l’aide des sauvegardes automatisées](recovery-using-backups.md).

## <a name="deleted-database-restore"></a>Restauration d’une base de données supprimée

Vous pouvez restaurer une base de données supprimée à l’heure de sa suppression ou à un point antérieur dans le temps, sur le même serveur ou la même instance gérée. Vous pouvez effectuer cette opération par le biais du portail Azure, de [PowerShell](/powershell/module/az.sql/restore-azsqldatabase) ou de [REST (createMode=Restore)](/rest/api/sql/databases/createorupdate). Vous restaurez une base de données supprimée en créant une nouvelle base de données à partir de la sauvegarde.

> [!IMPORTANT]
> Si vous supprimez un serveur ou une instance managée, toutes ses bases de données sont aussi supprimées et leur récupération est impossible. Vous ne pouvez pas restaurer un serveur ou une instance managée supprimé(e).

### <a name="deleted-database-restore-by-using-the-azure-portal"></a>Restauration d’une base de données supprimée à l’aide du portail Azure

Vous pouvez restaurer des bases de données supprimées à partir du Portail Azure à partir de la ressource de serveur et de l’instance gérée.

> [!TIP]
> Plusieurs minutes peuvent être nécessaires pour que les bases de données récemment supprimées apparaissent sur la page **Bases de données supprimées** dans Portail Azure, ou lors de l’affichage des bases de données supprimées [par programmation](#programmatic-recovery-using-automated-backups).

#### <a name="sql-database"></a>SQL Database

Pour récupérer une base de données qui a été supprimée telle qu'elle était au moment de sa suppression à partir du portail Azure, ouvrez la page de présentation du serveur et sélectionnez **Bases de données supprimées**. Sélectionnez une base de données supprimée que vous souhaitez restaurer, puis tapez le nom de la nouvelle base de données qui sera créée avec les données restaurées à partir de la sauvegarde.

  ![Capture d’écran de la restauration d’une base de données supprimée](./media/recovery-using-backups/restore-deleted-sql-database-annotated.png)

#### <a name="sql-managed-instance"></a>Instance managée SQL

Pour récupérer une base de données managée à partir du portail Azure, ouvrez la page de vue d’ensemble de l’instance managée, puis sélectionnez **Bases de données supprimées**. Sélectionnez une base de données supprimée que vous souhaitez restaurer, puis tapez le nom de la nouvelle base de données qui sera créée avec les données restaurées à partir de la sauvegarde.

  ![Capture d’écran de la restauration d’une base de données Azure SQL Managed Instance supprimée](./media/recovery-using-backups/restore-deleted-sql-managed-instance-annotated.png)

### <a name="deleted-database-restore-by-using-powershell"></a>Restauration d’une base de données supprimée à l’aide de PowerShell

Utilisez les exemples de scripts suivants pour restaurer une base de données supprimée pour SQL Database ou SQL Managed Instance à l’aide de PowerShell.

#### <a name="sql-database"></a>SQL Database

Pour obtenir un exemple de script PowerShell illustrant comment restaurer une base de données supprimée dans Azure SQL Database, consultez [Restaurer une base de données à l’aide de PowerShell](scripts/restore-database-powershell.md).

#### <a name="sql-managed-instance"></a>Instance managée SQL

Pour obtenir un exemple de script PowerShell montrant comment restaurer une base de données d’instance supprimée, consultez [Restaurer une base de données supprimée à l'aide de PowerShell](../managed-instance/point-in-time-restore.md#restore-a-deleted-database)

> [!TIP]
> Pour restaurer par programmation une base de données supprimée, consultez [Exécution par programmation d’une récupération à l’aide des sauvegardes automatisées](recovery-using-backups.md).

## <a name="geo-restore"></a>La géorestauration

> [!IMPORTANT]
> La géorestauration est disponible uniquement pour les bases de données SQL ou les instances gérées configurées avec un [stockage de sauvegarde](automated-backups-overview.md#backup-storage-redundancy) géoredondant.

Vous pouvez restaurer une base de données sur n’importe quel serveur SQL Database ou une base de données d’instance sur n’importe quelle instance managée dans toute région Azure à partir des sauvegardes géorépliquées les plus récentes. La géorestauration utilise une sauvegarde géorépliquée comme source. Vous pouvez demander une géorestauration même si la base de données ou le centre de données est inaccessible en raison d’une panne.

La géorestauration constitue l’option de récupération par défaut lorsque la base de données n’est pas disponible en raison d’un incident dans la région d’hébergement. Vous pouvez restaurer la base de données sur un serveur dans n’importe quelle autre région. Il peut y avoir un délai entre le moment où la sauvegarde est effectuée et celui où elle est géo-répliquée dans un objet blob Azure dans une autre région. C’est pourquoi la base de données restaurée peut avoir jusqu’à une heure de retard par rapport à la base de données d’origine. L’illustration ci-dessous montre la restauration d’une base de données à partir de la dernière sauvegarde disponible dans une autre région.

![Graphique de géorestauration](./media/recovery-using-backups/geo-restore-2.png)

### <a name="geo-restore-by-using-the-azure-portal"></a>Géorestauration à l’aide du portail Azure

À partir du portail Azure, créez une base de données d’instance unique ou managée, puis sélectionnez une sauvegarde de géorestauration disponible. La nouvelle base de données contient les données de sauvegarde géorestaurées.

#### <a name="sql-database"></a>SQL Database

Pour géorestaurer une base de données unique à partir du portail Azure dans la région et le serveur de votre choix, effectuez les étapes suivantes :

1. À partir du **Tableau de bord**, sélectionnez **Ajouter** > **Créer une base de données SQL**. Sous l’onglet **De base**, entrez les informations nécessaires.
2. Sélectionnez **Paramètres supplémentaires**.
3. Pour **Utiliser des données existantes**, sélectionnez **Sauvegarde**.
4. Pour **Sauvegarde**, sélectionnez une sauvegarde dans la liste des sauvegardes de géorestauration disponibles.

    ![Capture d’écran des options de création de base de données SQL](./media/recovery-using-backups/geo-restore-azure-sql-database-list-annotated.png)

Terminez le processus de création d’une nouvelle base de données depuis la sauvegarde. Quand vous créez une base de données dans Azure SQL Database, elle contient la sauvegarde de géorestauration restaurée.

#### <a name="sql-managed-instance"></a>Instance managée SQL

Pour géorestaurer une base de données d’instance managée du portail Azure vers une instance managée existante dans une région de votre choix, sélectionnez une instance managée sur laquelle vous souhaitez restaurer une base de données. Procédez comme suit :

1. Sélectionnez **Nouvelle base de données**.
2. Tapez le nom de base de données souhaité.
3. Sous **Utiliser des données existantes**, sélectionnez **Sauvegarde**.
4. Sélectionnez une sauvegarde dans la liste des sauvegardes de géorestauration disponibles.

    ![Capture d’écran des options de nouvelle base de données](./media/recovery-using-backups/geo-restore-sql-managed-instance-list-annotated.png)

Terminez le processus de création de base de données. Quand vous créez la base de données d’instance, elle contient la sauvegarde de géorestauration restaurée.

### <a name="geo-restore-by-using-powershell"></a>Géorestaurer à l’aide de PowerShell

#### <a name="sql-database"></a>SQL Database

Pour obtenir un script PowerShell qui illustre comment effectuer une géorestauration pour une base de données unique, consultez [Utiliser PowerShell pour restaurer une base de données unique à un point antérieur dans le passé](scripts/restore-database-powershell.md).

#### <a name="sql-managed-instance"></a>Instance managée SQL

Pour obtenir un script PowerShell qui illustre comment effectuer une géorestauration pour une base de données d’instance managée, consultez [Utiliser PowerShell pour restaurer une base de données Managed Instance dans une autre région de zone géographique](../managed-instance/scripts/restore-geo-backup.md).

### <a name="geo-restore-considerations"></a>Considérations en matière de géorestauration

Vous ne pouvez pas effectuer une limite de restauration dans le temps sur une base de données géosecondaire. Vous ne pouvez le faire que sur une base de données primaire. Pour plus d’informations sur l’utilisation de la géorestauration pour la récupération suite à une panne, voir [Récupération après une panne](../../key-vault/general/disaster-recovery-guidance.md).

> [!IMPORTANT]
> La géorestauration est la solution de récupération d’urgence la plus basique disponible dans SQL Database et SQL Managed Instance. Elle s’appuie sur des sauvegardes géorépliquées créées automatiquement, dont l’objectif de point de récupération (RPO) est jusqu’à 1 heure et la durée estimée de récupération jusqu’à 12 heures. Elle ne garantit pas que la région cible aura la capacité de restaurer les bases de données après une panne régionale, car il se produira probablement un pic de demande. Si votre application utilise des bases de données relativement petites et qu’elle n’est pas vitale pour l’entreprise, la géorestauration est une solution de reprise d’activité adaptée. 
>
> Pour les applications vitales pour l’entreprise qui utilisent de grosses bases de données et doivent assurer la continuité de l’activité, utilisez des [Groupes de basculement automatique](auto-failover-group-overview.md). Ils offrent un RPO et un objectif de délai de récupération beaucoup plus faibles, et la capacité est toujours garantie. 
>
> Pour plus d’informations sur les choix de continuité d’activité, consultez [Vue d’ensemble de la continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md).

## <a name="programmatic-recovery-using-automated-backups"></a>Récupération par programmation à l’aide des sauvegardes automatisées

Vous pouvez également utiliser Azure PowerShell ou l’API REST pour la récupération. Les tableaux ci-dessous décrivent l’ensemble des commandes disponibles.

### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par SQL Database et SQL Managed Instance, mais tous les développements à venir sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont très similaires à ceux dans les modules Azure Resource Manager.

#### <a name="sql-database"></a>SQL Database

Pour restaurer une base de données autonome ou mise en pool, voir [Restore-AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase).

  | Applet de commande | Description |
  | --- | --- |
  | [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) |Obtient une ou plusieurs bases de données. |
  | [Get-AzSqlDeletedDatabaseBackup](/powershell/module/az.sql/get-azsqldeleteddatabasebackup) | Obtient une base de données supprimée que vous pouvez restaurer. |
  | [Get-AzSqlDatabaseGeoBackup](/powershell/module/az.sql/get-azsqldatabasegeobackup) |Obtient une sauvegarde géo-redondante d’une base de données. |
  | [Restore-AzSqlDatabase](/powershell/module/az.sql/restore-azsqldatabase) |Restaure une base de données. |

  > [!TIP]
  > Pour obtenir un exemple de script PowerShell qui illustre comment restaurer une base de données à un point antérieur dans le temps, consultez [Restaurer une base de données à l’aide de PowerShell](scripts/restore-database-powershell.md).

#### <a name="sql-managed-instance"></a>Instance managée SQL

Pour restaurer une base de données d’instance managée, consultez [Restore-AzSqlInstanceDatabase](/powershell/module/az.sql/restore-azsqlinstancedatabase).

  | Applet de commande | Description |
  | --- | --- |
  | [Get-AzSqlInstance](/powershell/module/az.sql/get-azsqlinstance) |Récupère une ou plusieurs instances gérées. |
  | [Get-AzSqlInstanceDatabase](/powershell/module/az.sql/get-azsqlinstancedatabase) | Récupère une base de données d’instance. |
  | [Restore-AzSqlInstanceDatabase](/powershell/module/az.sql/restore-azsqlinstancedatabase) |Restaure une base de données d’instance. |

### <a name="rest-api"></a>API REST

Pour restaurer une base de données à l'aide de l'API REST :

| API | Description |
| --- | --- |
| [REST (createMode=Recovery)](/rest/api/sql/databases) |Restaure une base de données. |
| [Créer ou mettre à jour l’état de la base de données](/rest/api/sql/operations) |Retourne l’état durant une opération de restauration. |

### <a name="azure-cli"></a>Azure CLI

#### <a name="sql-database"></a>SQL Database

Pour restaurer une base de données à l'aide d'Azure CLI, consultez [az sql db restore](/cli/azure/sql/db#az-sql-db-restore).

#### <a name="sql-managed-instance"></a>Instance managée SQL

Pour restaurer une base de données d’instance managée à l’aide d’Azure CLI, consultez [az sql midb restore](/cli/azure/sql/midb#az-sql-midb-restore).

## <a name="summary"></a>Résumé

Les sauvegardes automatiques protègent vos bases de données des erreurs utilisateur et des erreurs d’application, de la suppression accidentelle d’une base de données et des interruptions prolongées. Cette fonctionnalité intégrée est disponible pour tous les niveaux de service et tailles de calcul.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble de la continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md)
- [Sauvegardes automatisées d’une base de données SQL](automated-backups-overview.md)
- [Rétention à long terme](long-term-retention-overview.md)
- Pour découvrir des options de récupération plus rapides, consultez [Géoréplication active](active-geo-replication-overview.md) ou [Groupes de basculement automatique](auto-failover-group-overview.md).