---
title: Gérer et superviser des bases de données SQL Server sur une machine virtuelle Azure
description: Cet article décrit comment gérer et surveiller des bases de données SQL Server s’exécutant sur une machine virtuelle Azure.
ms.topic: conceptual
ms.date: 09/11/2019
ms.openlocfilehash: b0df22002521c8148cac1200e79aeb0ae5a60546
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021501"
---
# <a name="manage-and-monitor-backed-up-sql-server-databases"></a>Gérer et surveiller des bases de données SQL Server sauvegardées

Cet article décrit les tâches courantes de gestion et de surveillance des bases de données SQL Server s’exécutant sur une machine virtuelle Azure et sauvegardées dans un coffre Recovery Services par le service [Sauvegarde Azure](backup-overview.md). Vous apprendrez à surveiller les travaux et les alertes, à arrêter et à réactiver la protection de la base de données, à exécuter des travaux de sauvegarde, ainsi qu’à désinscrire une machine virtuelle des sauvegardes.

Si vous n’avez pas encore configuré de sauvegardes pour vos bases de données SQL Server, voir [Sauvegarder des bases de données SQL Server sur des machines virtuelles Azure](backup-azure-sql-database.md).

## <a name="monitor-backup-jobs-in-the-portal"></a>Surveiller les travaux de sauvegarde sur le portail

Sauvegarde Azure affiche toutes les opérations planifiées et à la demande sous **Travaux de sauvegarde** dans le portail, à l’exception des sauvegardes de fichiers journaux planifiées qui peuvent s’avérer particulièrement fréquentes. Les travaux que vous voyez sur ce portail incluent les opérations de découverte, d’inscription, de configuration, de sauvegarde et de restauration de base de données.

![Portail Travaux de sauvegarde](./media/backup-azure-sql-database/jobs-list.png)

Pour plus d’informations sur les scénarios de surveillance, voir [Surveillance dans le portail Azure](backup-azure-monitoring-built-in-monitor.md) et [Surveillance à l’aide d’Azure Monitor](backup-azure-monitoring-use-azuremonitor.md).  

## <a name="view-backup-alerts"></a>Afficher les alertes de sauvegarde

Comme les sauvegardes de fichiers journaux interviennent toutes les 15 minutes, la surveillance des tâches de sauvegarde peut s'avérer fastidieuse. Le service Sauvegarde Azure facilite la surveillance en envoyant des alertes par courrier électronique. Les alertes par courrier électronique sont :

- Déclenchées pour tous les échecs de sauvegarde.
- Consolidées au niveau de la base de données par code d’erreur.
- Envoyées uniquement pour le premier échec de sauvegarde d’une base de données.

Pour surveiller les alertes de sauvegarde de base de données :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Sur le tableau de bord du coffre, sélectionnez **Alertes et événements**.

   ![Sélectionner Alertes et événements](./media/backup-azure-sql-database/vault-menu-alerts-events.png)

3. Dans **Alertes et événements**, sélectionnez **Alertes de sauvegarde**.

   ![Sélectionner Alertes de sauvegarde](./media/backup-azure-sql-database/backup-alerts-dashboard.png)

## <a name="stop-protection-for-a-sql-server-database"></a>Arrêter la protection d’une base de données SQL Server

Pour arrêter la sauvegarde d'une base de données SQL Server, deux méthodes sont disponibles :

- Arrêter tous les travaux de sauvegarde à venir et supprimer tous les points de récupération.
- Arrêter tous les travaux de sauvegarde à venir, mais conserver les points de récupération.

Si vous choisissez de conserver les points de récupération, gardez à l’esprit les considérations suivantes :

- Tous les points de récupération resteront intacts et toutes les opérations de nettoyage s’arrêteront à l’arrêt de la protection avec données conservées.
- Vous serez facturé pour l’instance protégée et le stockage utilisé. Pour plus d’informations, voir [Tarification Sauvegarde Azure](https://azure.microsoft.com/pricing/details/backup/).
- Si vous supprimez une source de données sans arrêter les sauvegardes, les nouvelles sauvegardes échouent. Les anciens points de récupération expirent conformément à la stratégie, mais le point de récupération le plus récent est conservé jusqu’à ce que vous arrêtiez les sauvegardes et supprimiez les données.

Pour arrêter la protection de la base de données :

1. Dans le tableau de bord du coffre, sélectionnez **Éléments de la sauvegarde**.

2. Sous **Type de gestion des sauvegardes**, sélectionnez **SQL dans une machine virtuelle Azure**.

    ![Sélectionner SQL in Azure VM](./media/backup-azure-sql-database/sql-restore-backup-items.png)

3. Sélectionnez la base de données dont vous souhaitez arrêter la protection.

    ![Sélectionner la protection de la base de données a arrêter](./media/backup-azure-sql-database/sql-restore-sql-in-vm.png)

4. Dans le menu de la base de données, sélectionnez **Arrêter la sauvegarde**.

    ![Sélectionner Arrêter la sauvegarde](./media/backup-azure-sql-database/stop-db-button.png)

5. Dans le menu **Arrêter la sauvegarde**, indiquez si vous souhaitez conserver ou supprimer les données. Vous pouvez également indiquer une raison ou formuler un commentaire.

    ![Conserver ou supprimer des données dans le menu Arrêter la sauvegarde](./media/backup-azure-sql-database/stop-backup-button.png)

6. Sélectionnez **Arrêter la sauvegarde**.

> [!NOTE]
>
>Pour plus d’informations sur l’option de suppression de données, consultez le FAQ ci-dessous :
>
>- [Si je supprime une base de données d’une instance protégée automatiquement, qu’advient-il des sauvegardes ?](faq-backup-sql-server.md#if-i-delete-a-database-from-an-autoprotected-instance-what-will-happen-to-the-backups)
>- [Si j’arrête une opération de sauvegarde d’une base de données protégée automatiquement, comment se comportera-t-elle ?](faq-backup-sql-server.md#if-i-change-the-name-of-the-database-after-it-has-been-protected-what-will-be-the-behavior)
>
>

## <a name="resume-protection-for-a-sql-database"></a>Rétablir la protection d’une base de données SQL

Lorsque vous arrêtez la protection de la base de données SQL, si vous sélectionnez l’option **Conserver les données de sauvegarde**, vous pourrez réactiver la protection ultérieurement. Si vous ne conservez pas les données de sauvegarde, vous ne pourrez pas réactiver la protection.

Pour réactiver la protection d’une base de données SQL :

1. Ouvrez l’élément de la sauvegarde, puis sélectionnez **Reprendre la sauvegarde**.

    ![Sélectionner Reprendre la sauvegarde pour relancer la protection de la base de données](./media/backup-azure-sql-database/resume-backup-button.png)

2. Dans le menu **Stratégie de sauvegarde**, sélectionnez une stratégie, puis sélectionnez **Enregistrer**.

## <a name="run-an-on-demand-backup"></a>Exécuter une sauvegarde à la demande

Vous pouvez exécuter différents types de sauvegardes à la demande :

- Sauvegarde complète
- Sauvegarde de copie uniquement
- Sauvegarde différentielle
- Sauvegarde du journal

Si vous devez spécifier la durée de conservation de l’option Copier uniquement la sauvegarde complète, la durée de conservation pour la sauvegarde complète à la demande est automatiquement définie à 45 jours à partir de l’heure actuelle.

Pour plus d’informations, voir [Types de sauvegardes SQL Server](backup-architecture.md#sql-server-backup-types).

## <a name="modify-policy"></a>Modifier la stratégie

Dans la stratégie, modifiez la fréquence de sauvegarde ou la plage de rétention.

> [!NOTE]
> Toute modification de la période de rétention sera appliquée rétroactivement à tous les anciens points de récupération ainsi qu’aux nouveaux.

Dans le tableau de bord du coffre, accédez à **Gérer** > **Stratégies de sauvegarde** et choisissez la stratégie que vous souhaitez modifier.

  ![Gérer la stratégie de sauvegarde](./media/backup-azure-sql-database/modify-backup-policy.png)

  ![Modifier la stratégie de sauvegarde](./media/backup-azure-sql-database/modify-backup-policy-impact.png)

La modification de la stratégie aura un impact sur tous les éléments de sauvegarde associés et déclenchera les tâches de **configuration de la protection** correspondantes.

### <a name="inconsistent-policy"></a>Stratégie incohérente

Parfois, une opération de modification de la stratégie peut aboutir à une version de stratégie **incohérente** pour certains éléments de sauvegarde. Cela se produit lorsque la tâche de **configuration de la protection** correspondante échoue pour l’élément de sauvegarde après le déclenchement d’une opération de modification de la stratégie. Elle se présente comme suit dans l’affichage des éléments de sauvegarde :

  ![Stratégie incohérente](./media/backup-azure-sql-database/inconsistent-policy.png)

Vous pouvez corriger la version de la stratégie pour tous les éléments concernés en un seul clic :

  ![Corriger la stratégie incohérente](./media/backup-azure-sql-database/fix-inconsistent-policy.png)

## <a name="unregister-a-sql-server-instance"></a>Supprimer l'inscription d'une instance SQL Server

Désinscrivez une instance SQL Server après avoir désactivé la protection, mais avant de supprimer le coffre :

1. Sur le tableau de bord du coffre, sous **Gestion**, sélectionnez **Infrastructure de sauvegarde**.  

   ![Choisir Infrastructure de sauvegarde](./media/backup-azure-sql-database/backup-infrastructure-button.png)

2. Sous **Management Servers** (Serveurs d’administration), sélectionnez **Protected Servers** (serveurs protégés).

   ![Sélectionner Serveurs protégés](./media/backup-azure-sql-database/protected-servers.png)

3. Dans **Serveurs protégés**, sélectionnez le serveur dont vous souhaitez annuler l'inscription. Si vous souhaitez supprimer le coffre, vous devez annuler l’inscription de tous les serveurs.

4. Cliquez sur le serveur protégé, puis sélectionnez **Désinscrire**.

   ![Sélectionner Supprimer](./media/backup-azure-sql-database/delete-protected-server.jpg)

## <a name="re-register-extension-on-the-sql-server-vm"></a>Réinscrire une extension sur la machine virtuelle SQL Server

Parfois, l’extension de charge de travail sur la machine virtuelle peut être affectée pour une raison ou une autre. Dans ce cas, toutes les opérations déclenchées sur la machine virtuelle commencent à échouer. Vous devrez alors réinscrire l’extension sur la machine virtuelle. L’opération de **réinscription** a pour effet de réinstaller l’extension de sauvegarde de charge de travail sur la machine virtuelle pour que les opérations se poursuivent. Cette option est disponible sous **Infrastructure de sauvegarde** dans le coffre Recovery Services.

![Serveurs protégés sous Infrastructure de sauvegarde](./media/backup-azure-sql-database/protected-servers-backup-infrastructure.png)

Utilisez cette option avec précaution. Une fois déclenchée sur une machine virtuelle avec une extension déjà saine, cette opération provoque le redémarrage de l’extension. Cela peut provoquer l’échec de tous les travaux en cours. Vérifiez la présence d’un ou plusieurs de ces [symptômes](backup-sql-server-azure-troubleshoot.md#re-registration-failures) avant de déclencher l’opération de réinscription.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, voir [Résoudre les problèmes de sauvegarde des bases de données SQL Server](backup-sql-server-azure-troubleshoot.md).
