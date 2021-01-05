---
title: Déplacer des ressources vers une nouvelle région
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: Découvrez comment déplacer votre base de données ou instance gérée vers une autre région.
services: sql-database
ms.service: sql-db-mi
ms.subservice: data-movement
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 06/25/2019
ms.openlocfilehash: ae6c87c9eabea837ba9c43676d4ca712caa385cb
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94594162"
---
# <a name="move-resources-to-new-region---azure-sql-database--azure-sql-managed-instance"></a>Déplacer des ressources vers une nouvelle région - Azure SQL Database et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Cet article présente un flux de travail générique pour déplacer votre base de données ou instance gérée vers une nouvelle région.

## <a name="overview"></a>Vue d’ensemble

Il existe différents scénarios dans lesquels vous pourriez souhaiter déplacer votre base de données ou instance gérée existantes d’une région à une autre. Par exemple, vous étendez vos activités à une nouvelle région et souhaitez les optimiser pour la nouvelle base de clients. Ou vous avez besoin de déplacer les opérations vers une autre région pour des raisons de conformité. Ou encore, Azure a publié une nouvelle région qui offre une meilleure proximité et améliore l’expérience client.  

Cet article fournit un flux de travail général pour déplacer des ressources vers une autre région. Le flux de travail se déroule comme suit :

1. Vérifiez les prérequis pour le déplacement
1. Préparez le déplacement des ressources dans l’étendue.
1. Analysez le processus de préparation.
1. Testez le processus de déplacement.
1. Initiez le déplacement.
1. Supprimez les ressources de la région source.

> [!NOTE]
> Cet article s’applique aux migrations dans le cloud public Azure ou dans le même cloud souverain.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="move-a-database"></a>Déplacer une base de données

### <a name="verify-prerequisites"></a>Vérifier la configuration requise

1. Créez un serveur cible pour chaque serveur source.
1. Configurez le pare-feu avec les exceptions appropriées à l’aide de [PowerShell](scripts/create-and-configure-database-powershell.md).  
1. Configurez les serveurs avec les connexions appropriées. Si vous n’êtes pas administrateur de l’abonnement ou administrateur SQL Server, collaborez avec l’administrateur pour affecter les autorisations dont vous avez besoin. Pour plus d’informations, consultez [Guide pratique pour gérer la sécurité d’Azure SQL Database après la récupération d'urgence](active-geo-replication-security-configure.md).
1. Si vos bases de données sont chiffrées avec la clé de chiffrement transparente et que vous utilisez votre propre clé de chiffrement dans Azure Key Vault, vérifiez que le matériel de chiffrement correct est provisionné dans les régions cibles. Pour plus d’informations, consultez [Azure SQL Transparent Data Encryption avec des clés managées dans Azure Key Vault](transparent-data-encryption-byok-overview.md).
1. Si l’audit au niveau de la base de données est activé, désactivez-le et activez plutôt l’audit au niveau du serveur. Après le basculement, l’audit au niveau de la base de données nécessitera le trafic inter-régions, qui n’est pas souhaitable ou possible après le déplacement.
1. Pour les audits au niveau du serveur, vérifiez que :
   - Le conteneur de stockage, Log Analytics ou le hub d’événements avec les journaux d’audit existants est déplacé vers la région cible.
   - L’audit est configuré sur le serveur cible. Pour plus d'informations, consultez [Prise en main de l’audit SQL Database](../../azure-sql/database/auditing-overview.md).
1. Si votre instance a une stratégie de conservation à long terme (LTR), les sauvegardes LTR existantes resteront associées au serveur actuel. Le serveur cible étant différent, vous pourrez accéder aux sauvegardes LTR plus anciennes dans la région source à l’aide du serveur source, même si le serveur est supprimé.

      > [!NOTE]
      > Cela ne sera pas suffisant pour les déplacements entre le cloud souverain et une région publique. Pour une telle migration, vous devrez déplacer les sauvegardes LTR vers le serveur cible, ce qui n’est pas pris en charge actuellement.

### <a name="prepare-resources"></a>Préparer les ressources

1. Créez un [groupe de basculement](failover-group-add-single-database-tutorial.md#2---create-the-failover-group) entre le serveur de la source et le serveur de la cible.  
1. Ajoutez les bases de données que vous souhaitez déplacer au groupe de basculement.
  
    La réplication de toutes les bases de données ajoutées sera lancée automatiquement. Pour plus d’informations, consultez [Meilleures pratiques relatives à l’utilisation des groupes de basculement avec des bases de données uniques et des pools élastiques](auto-failover-group-overview.md#best-practices-for-sql-database).

### <a name="monitor-the-preparation-process"></a>Superviser le processus de préparation

Vous pouvez appeler régulièrement [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) pour superviser la réplication de vos bases de données de la source vers la cible. L’objet de sortie de `Get-AzSqlDatabaseFailoverGroup` comprend une propriété pour **ReplicationState** :

- **ReplicationState = 2** (CATCH_UP) indique que la base de données est synchronisée et peut être basculée en toute sécurité.
- **ReplicationState = 0** (SEEDING) indique que la base de données n’est pas encore amorcée et qu’une tentative de basculement échouera.

### <a name="test-synchronization"></a>Tester la synchronisation

Une fois que **ReplicationState** a la valeur `2`, connectez-vous à chaque base de données ou sous-ensemble de bases de données à l’aide du point de terminaison secondaire `<fog-name>.secondary.database.windows.net` et exécutez une requête sur les bases de données pour vérifier la connectivité, la configuration de sécurité et la réplication des données.

### <a name="initiate-the-move"></a>Lancer le déplacement

1. Connectez-vous au serveur cible à l’aide du point de terminaison secondaire `<fog-name>.secondary.database.windows.net`.
1. Utilisez [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup) pour faire de l’instance managée secondaire la base de données primaire avec synchronisation complète. Cette opération réussira ou sera restaurée.
1. Vérifiez que la commande s’est correctement exécutée en exécutant `nslook up <fog-name>.secondary.database.windows.net` pour vérifier que l’entrée CNAME DNS pointe vers l’adresse IP de la région cible. Si la commande Switch échoue, le CNAME ne sera pas mis à jour.

### <a name="remove-the-source-databases"></a>Supprimer les bases de données sources

Une fois le déplacement terminé, supprimez les ressources dans la région source pour éviter les frais inutiles.

1. Supprimez le groupe de basculement à l’aide de [Remove-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/remove-azsqldatabasefailovergroup).
1. Supprimez chaque base de données source à l’aide de [Remove-AzSqlDatabase](/powershell/module/az.sql/remove-azsqldatabase) pour chacune des bases de données sur le serveur source. Les liens de géoréplication seront automatiquement rompus.
1. Supprimez le serveur source à l’aide de [Remove-AzSqlServer](/powershell/module/az.sql/remove-azsqlserver).
1. Supprimez le coffre de clés, les conteneurs de stockage d’audit, l’event hub, l’instance Azure Active Directory (Azure AD) et les autres ressources dépendantes pour qu’elles ne vous soient plus facturées.

## <a name="move-elastic-pools"></a>Déplacer les pools élastiques

### <a name="verify-prerequisites"></a>Vérifier la configuration requise

1. Créez un serveur cible pour chaque serveur source.
1. Configurez le pare-feu avec les exceptions appropriées à l’aide de [PowerShell](scripts/create-and-configure-database-powershell.md).
1. Configurez les serveurs avec les connexions appropriées. Si vous n’êtes pas un administrateur de l’abonnement ou un administrateur du serveur, collaborez avec l’administrateur pour attribuer les autorisations dont vous avez besoin. Pour plus d’informations, consultez [Guide pratique pour gérer la sécurité d’Azure SQL Database après la récupération d'urgence](active-geo-replication-security-configure.md).
1. Si vos bases de données sont chiffrées avec la clé de chiffrement transparente et que vous utilisez votre propre clé de chiffrement dans Azure Key Vault, vérifiez que le matériel de chiffrement correct est provisionné dans les régions cibles.
1. Créez un pool élastique cible pour chaque pool élastique source, en veillant à ce que le pool soit créé dans le même niveau de service, avec le même nom et la même taille.
1. Si un audit de niveau base de données est activé, désactivez-le et activez plutôt l’audit au niveau du serveur. Après le basculement, l’audit au niveau de la base de données nécessitera le trafic inter-régions, qui n’est pas souhaitable ou possible après le déplacement.
1. Pour les audits au niveau du serveur, vérifiez que :
    - Le conteneur de stockage, Log Analytics ou le hub d’événements avec les journaux d’audit existants est déplacé vers la région cible.
    - La configuration de l’audit est configurée sur le serveur cible. Pour plus d’informations, consultez [Audit SQL Database](../../azure-sql/database/auditing-overview.md).
1. Si votre instance a une stratégie de conservation à long terme (LTR), les sauvegardes LTR existantes resteront associées au serveur actuel. Le serveur cible étant différent, vous pourrez accéder aux sauvegardes LTR plus anciennes dans la région source à l’aide du serveur source, même si le serveur est supprimé.

      > [!NOTE]
      > Cela ne sera pas suffisant pour les déplacements entre le cloud souverain et une région publique. Pour une telle migration, vous devrez déplacer les sauvegardes LTR vers le serveur cible, ce qui n’est pas pris en charge actuellement.

### <a name="prepare-to-move"></a>Préparer le déplacement

1. Créez un [groupe de basculement](failover-group-add-elastic-pool-tutorial.md#3---create-the-failover-group) entre chaque pool élastique sur le serveur source et son pool élastique équivalent sur le serveur cible.
1. Ajoutez toutes les bases de données du pool au groupe de basculement.

    La réplication des bases de données ajoutées sera lancée automatiquement. Pour plus d’informations, consultez [Meilleures pratiques relatives à l’utilisation des groupes de basculement avec des bases de données uniques et des pools élastiques](auto-failover-group-overview.md#best-practices-for-sql-database).

      > [!NOTE]
      > Bien qu’il soit possible de créer un groupe de basculement qui comprend plusieurs pools élastiques, nous vous recommandons vivement de créer un groupe de basculement distinct pour chaque pool. Si vous devez déplacer un grand nombre de bases de données dans plusieurs pools élastiques, vous pouvez exécuter les étapes de préparation en parallèle, puis lancer l’étape de déplacement en parallèle. Ce processus est mieux adapté et prend moins de temps que si vous avez plusieurs pools élastiques dans le même groupe de basculement.

### <a name="monitor-the-preparation-process"></a>Superviser le processus de préparation

Vous pouvez appeler régulièrement [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) pour superviser la réplication de vos bases de données de la source vers la cible. L’objet de sortie de `Get-AzSqlDatabaseFailoverGroup` comprend une propriété pour **ReplicationState** :

- **ReplicationState = 2** (CATCH_UP) indique que la base de données est synchronisée et peut être basculée en toute sécurité.
- **ReplicationState = 0** (SEEDING) indique que la base de données n’est pas encore amorcée et qu’une tentative de basculement échouera.

### <a name="test-synchronization"></a>Tester la synchronisation

Une fois que **ReplicationState** a la valeur `2`, connectez-vous à chaque base de données ou sous-ensemble de bases de données à l’aide du point de terminaison secondaire `<fog-name>.secondary.database.windows.net` et exécutez une requête sur les bases de données pour vérifier la connectivité, la configuration de sécurité et la réplication des données.

### <a name="initiate-the-move"></a>Lancer le déplacement

1. Connectez-vous au serveur cible à l’aide du point de terminaison secondaire `<fog-name>.secondary.database.windows.net`.
1. Utilisez [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup) pour faire de l’instance managée secondaire la base de données primaire avec synchronisation complète. Cette opération réussira ou sera annulée.
1. Vérifiez que la commande s’est correctement exécutée en exécutant `nslook up <fog-name>.secondary.database.windows.net` pour vérifier que l’entrée CNAME DNS pointe vers l’adresse IP de la région cible. Si la commande Switch échoue, le CNAME ne sera pas mis à jour.

### <a name="remove-the-source-elastic-pools"></a>Supprimer les pools élastiques sources

Une fois le déplacement terminé, supprimez les ressources dans la région source pour éviter les frais inutiles.

1. Supprimez le groupe de basculement à l’aide de [Remove-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/remove-azsqldatabasefailovergroup).
1. Supprimez chaque pool élastique source sur le serveur source à l’aide de [Remove-AzSqlElasticPool](/powershell/module/az.sql/remove-azsqlelasticpool).
1. Supprimez le serveur source à l’aide de [Remove-AzSqlServer](/powershell/module/az.sql/remove-azsqlserver).
1. Supprimez le coffre de clés, les conteneurs de stockage d’audit, l’event hub, l’instance Azure AD et les autres ressources dépendantes pour qu’elles ne vous soient plus facturées.

## <a name="move-a-managed-instance"></a>Déplacer une instance gérée

### <a name="verify-prerequisites"></a>Vérifier la configuration requise

1. Pour chaque instance gérée source, créez une instance gérée cible de SQL Managed Instance de la même taille dans la région cible.  
1. Configurez le réseau pour une instance managée. Pour plus d’informations, consultez [Configuration réseau](../managed-instance/how-to-content-reference-guide.md#network-configuration).
1. Configurez la base de données MASTER cible avec les connexions appropriées. Si vous n’êtes pas l’administrateur de l’abonnement ou de SQL Managed Instance, collaborez avec l’administrateur pour attribuer les autorisations dont vous avez besoin.
1. Si vos bases de données sont chiffrées avec un chiffrement de données transparent et que vous utilisez votre propre clé de chiffrement dans Azure Key Vault, vérifiez que Azure Key Vault existe avec des clés de chiffrement identiques dans les régions source et cible. Pour plus d’informations, consultez [le chiffrement des données transparent avec des clés managées dans Azure Key Vault](transparent-data-encryption-byok-overview.md).
1. Si l’audit est activé pour l’instance gérée, vérifiez que :
    - Le conteneur de stockage ou le hub d’événements avec les journaux existants est déplacé vers la région cible.
    - L’audit est configuré sur l’instance cible. Pour plus d’informations, consultez [Audit avec une instance SQL Managed Instance](../managed-instance/auditing-configure.md).
1. Si votre instance a une stratégie de conservation à long terme (LTR), les sauvegardes LTR existantes resteront associées à l’instance actuelle. L’instance cible étant différente, vous pourrez accéder aux sauvegardes LTR plus anciennes dans la région source à l’aide de l’instance source, même si l’instance est supprimée.

  > [!NOTE]
  > Cela ne sera pas suffisant pour les déplacements entre le cloud souverain et une région publique. Pour une telle migration, vous devrez déplacer les sauvegardes LTR vers l’instance cible, ce qui n’est pas pris en charge actuellement.

### <a name="prepare-resources"></a>Préparer les ressources

Créez un groupe de basculement entre chaque instance gérée source et l’instance gérée cible correspondante de SQL Managed Instance.

La réplication de toutes les bases de données sur chaque instance sera lancée automatiquement. Pour plus d’informations, consultez [Groupes de basculement automatique](auto-failover-group-overview.md).

### <a name="monitor-the-preparation-process"></a>Superviser le processus de préparation

Vous pouvez appeler régulièrement [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) pour superviser la réplication de vos bases de données de la source vers la cible. L’objet de sortie de `Get-AzSqlDatabaseFailoverGroup` comprend une propriété pour **ReplicationState** :

- **ReplicationState = 2** (CATCH_UP) indique que la base de données est synchronisée et peut être basculée en toute sécurité.
- **ReplicationState = 0** (SEEDING) indique que la base de données n’est pas encore amorcée et qu’une tentative de basculement échouera.

### <a name="test-synchronization"></a>Tester la synchronisation

Une fois que **ReplicationState** a la valeur `2`, connectez-vous à chaque base de données ou sous-ensemble de bases de données à l’aide du point de terminaison secondaire `<fog-name>.secondary.database.windows.net` et exécutez une requête sur les bases de données pour vérifier la connectivité, la configuration de sécurité et la réplication des données.

### <a name="initiate-the-move"></a>Lancer le déplacement

1. Connectez-vous à l’instance gérée cible à l’aide du point de terminaison secondaire `<fog-name>.secondary.database.windows.net`.
1. Utilisez [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup) pour faire de l’instance managée secondaire la base de données primaire avec synchronisation complète. Cette opération réussira ou sera restaurée.
1. Vérifiez que la commande s’est correctement exécutée en exécutant `nslook up <fog-name>.secondary.database.windows.net` pour vérifier que l’entrée CNAME DNS pointe vers l’adresse IP de la région cible. Si la commande Switch échoue, le CNAME ne sera pas mis à jour.

### <a name="remove-the-source-managed-instances"></a>Supprimer les instances managées sources

Une fois le déplacement terminé, supprimez les ressources dans la région source pour éviter les frais inutiles.

1. Supprimez le groupe de basculement à l’aide de [Remove-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/remove-azsqldatabasefailovergroup). Cela supprimera la configuration du groupe de basculement et rompra les liens de géoréplication entre les deux instances.
1. Supprimez l’instance managée source à l’aide de [Remove-AzSqlInstance](/powershell/module/az.sql/remove-azsqlinstance).
1. Supprimez toutes les ressources supplémentaires dans le groupe de ressources, telles que le cluster virtuel, le réseau virtuel et le groupe de sécurité.

## <a name="next-steps"></a>Étapes suivantes

[Gérez](manage-data-after-migrating-to-database.md) votre base de données après avoir migré.
