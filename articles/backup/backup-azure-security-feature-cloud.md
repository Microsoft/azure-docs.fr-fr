---
title: Suppression réversible pour la Sauvegarde Azure
description: Découvrez comment utiliser les fonctionnalités de sécurité dans Sauvegarde Azure pour renforcer la sécurité des sauvegardes.
ms.topic: conceptual
ms.date: 04/30/2020
ms.openlocfilehash: da473b1d886ec2fe95a7baae76b09aff38fb3cd7
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95254047"
---
# <a name="soft-delete-for-azure-backup"></a>Suppression réversible pour la Sauvegarde Azure

Les préoccupations en matière de risques de sécurité, comme les logiciels malveillants, le ransomware et les intrusions, sont de plus en plus nombreuses. Ces problèmes de sécurité peuvent coûter cher, à la fois en termes d’argent et de données. Pour vous protéger contre ces attaques, Sauvegarde Azure fournit désormais des fonctionnalités de sécurité visant à protéger les données de sauvegarde après la suppression.

Une fonctionnalité de ce type est la suppression réversible. Avec la suppression réversible, même si un intervenant malveillant supprime une sauvegarde (ou même si les données de sauvegarde sont accidentellement supprimées), les données de sauvegarde sont conservées pendant 14 jours supplémentaires, ce qui permet la récupération de cet élément de sauvegarde sans perte de données. La conservation des données de sauvegarde pendant 14 jours supplémentaires à l’état « suppression réversible » n’engendre aucun frais pour vous.

La protection Suppression réversible est disponible pour les services suivants :

- [Suppression réversible pour les machines virtuelles Azure](soft-delete-virtual-machines.md)
- [Suppression réversible pour SQL Server dans les machines virtuelles Azure, et suppression réversible pour SAP HANA dans les charges de travail des machines virtuelles Azure](soft-delete-sql-saphana-in-azure-vm.md)

Cet organigramme montre les différentes étapes et états d’un élément de sauvegarde lorsque la suppression réversible est activée :

![Cycle de vie d’un élément de sauvegarde supprimé de manière réversible](./media/backup-azure-security-feature-cloud/lifecycle.png)

## <a name="enabling-and-disabling-soft-delete"></a>Activation et désactivation de la suppression réversible

La suppression réversible est activée par défaut sur les coffres nouvellement créés pour protéger les données de sauvegarde des suppressions accidentelles ou malveillantes.  La désactivation de cette fonctionnalité est déconseillée. La seule circonstance où vous devez envisager la désactivation de la suppression réversible est lorsque vous vous préparez à déplacer vos éléments protégés vers un nouveau coffre et que vous ne pouvez pas attendre les 14 jours requis avant d’effectuer la suppression et la reprotection (dans un environnement de test, par exemple). Seul le propriétaire du coffre peut désactiver cette fonctionnalité. Si vous désactivez cette fonctionnalité, toutes les suppressions d’éléments protégés à venir entraîneront une suppression immédiate, sans possibilité de restauration. Les données de sauvegarde existant à l’état de suppression réversible avant la désactivation de cette fonctionnalité conservent cet état pendant 14 jours. Si vous souhaitez immédiatement les supprimer définitivement, vous devez en annuler la suppression et les supprimer à nouveau pour les supprimer définitivement.

Il est important de se rappeler qu’une fois la fonctionnalité de suppression réversible désactivée, elle le reste pour tous les types de charge de travail. Par exemple, il n’est plus possible de désactiver la suppression réversible uniquement pour les bases de données SQL Server ou SAP HANA tout en la gardant activée pour les machines virtuelles présentes dans le même coffre. Vous pouvez créer des coffres distincts pour un contrôle plus précis.

### <a name="disabling-soft-delete-using-azure-portal"></a>Désactivation de la suppression réversible avec le portail Azure

Pour désactiver la suppression réversible, procédez comme suit :

1. Dans le Portail Azure, accédez à votre coffre, puis accédez à **Paramètres** -> **Propriétés**.
2. Dans le volet Propriétés, sélectionnez **Paramètres de sécurité** -> **Mettre à jour**.  
3. Dans le volet Paramètres de sécurité, sous **Suppression réversible**, sélectionnez **Désactiver**.

![Désactiver la suppression réversible](./media/backup-azure-security-feature-cloud/disable-soft-delete.png)

### <a name="disabling-soft-delete-using-azure-powershell"></a>Désactivation de la suppression réversible à l’aide d’Azure PowerShell

> [!IMPORTANT]
> La version minimale d’Az.RecoveryServices qui permet d’utiliser la suppression réversible à l’aide d’Azure PowerShell est la version 2.2.0. Utilisez ```Install-Module -Name Az.RecoveryServices -Force``` pour récupérer la version la plus récente.

Pour effectuer la désactivation, utilisez l’applet de commande PowerShell [Set-AzRecoveryServicesVaultBackupProperty](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupproperty).

```powershell
Set-AzRecoveryServicesVaultProperty -VaultId $myVaultID -SoftDeleteFeatureState Disable


StorageModelType       :
StorageType            :
StorageTypeState       :
EnhancedSecurityState  : Enabled
SoftDeleteFeatureState : Disabled
```

### <a name="disabling-soft-delete-using-rest-api"></a>Désactivation de la suppression réversible avec l'API REST

Pour désactiver la fonctionnalité de suppression réversible à l’aide de l’API REST, reportez-vous aux étapes mentionnées [ici](use-restapi-update-vault-properties.md#update-soft-delete-state-using-rest-api).

## <a name="permanently-deleting-soft-deleted-backup-items"></a>Suppression définitive d’éléments de sauvegarde supprimés de manière réversible

Les données de sauvegarde dans l’état de suppression réversible avant la désactivation de cette fonctionnalité conservent cet état. Si vous souhaitez immédiatement les supprimer définitivement, vous devez en annuler la suppression et les supprimer à nouveau pour les supprimer définitivement.

### <a name="using-azure-portal"></a>En passant par le portail Azure

Procédez comme suit :

1. Suivez les étapes pour [désactiver la suppression réversible](#enabling-and-disabling-soft-delete).

2. Dans le portail Azure, accédez à votre coffre, accédez à **Éléments de sauvegarde**, puis choisissez l’élément supprimé de manière réversible.

   ![Choisir l’élément supprimé de manière réversible](./media/backup-azure-security-feature-cloud/vm-soft-delete.png)

3. Sélectionnez l’option **Annuler la suppression**.

   ![Choisir Annuler la suppression](./media/backup-azure-security-feature-cloud/choose-undelete.png)

4. Une fenêtre s’affiche. Sélectionnez **Annuler la suppression**.

   ![Sélectionner Annuler la suppression](./media/backup-azure-security-feature-cloud/undelete-vm.png)

5. Choisissez **Supprimer les données de sauvegarde** pour supprimer définitivement les données de sauvegarde.

   ![Choisir Supprimer les données de sauvegarde](./media/backup-azure-manage-vms/delete-backup-button.png)

6. Tapez le nom de l’élément de sauvegarde pour confirmer la suppression des points de récupération.

   ![Entrez le nom de l’élément de sauvegarde](./media/backup-azure-manage-vms/delete-backup-data.png)

7. Pour supprimer les données de sauvegarde relatives à l’élément, sélectionnez **Supprimer**. Un message de notification vous informe que les données de sauvegarde ont été supprimées.

### <a name="using-azure-powershell"></a>Utilisation de Microsoft Azure PowerShell

Si des éléments ont été supprimés avant que la suppression réversible ne soit désactivée, ils sont dans un état de suppression réversible. Pour les supprimer immédiatement, l’opération de suppression doit être inversée, puis réexécutée.

Identifiez les éléments qui sont dans un état de suppression réversible.

```powershell

Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -VaultId $myVaultID | Where-Object {$_.DeleteState -eq "ToBeDeleted"}

Name                                     ContainerType        ContainerUniqueName                      WorkloadType         ProtectionStatus     HealthStatus         DeleteState
----                                     -------------        -------------------                      ------------         ----------------     ------------         -----------
VM;iaasvmcontainerv2;selfhostrg;AppVM1    AzureVM             iaasvmcontainerv2;selfhostrg;AppVM1       AzureVM              Healthy              Passed               ToBeDeleted

$myBkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -VaultId $myVaultID -Name AppVM1
```

Ensuite, annulez l’opération de suppression qui a été effectuée quand la suppression réversible était activée.

```powershell
Undo-AzRecoveryServicesBackupItemDeletion -Item $myBKpItem -VaultId $myVaultID -Force

WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
AppVM1           Undelete             Completed            12/5/2019 12:47:28 PM     12/5/2019 12:47:40 PM     65311982-3755-46b5-8e53-c82ea4f0d2a2
```

La suppression réversible étant désormais désactivée, l’opération de suppression entraîne la suppression immédiate des données de sauvegarde.

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $myBkpItem -RemoveRecoveryPoints -VaultId $myVaultID -Force

WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
AppVM1           DeleteBackupData     Completed            12/5/2019 12:44:15 PM     12/5/2019 12:44:50 PM     0488c3c2-accc-4a91-a1e0-fba09a67d2fb
```

### <a name="using-rest-api"></a>Utilisation de l'API REST

Si des éléments ont été supprimés avant que la suppression réversible ne soit désactivée, ils sont dans un état de suppression réversible. Pour les supprimer immédiatement, l’opération de suppression doit être inversée, puis réexécutée.

1. Tout d’abord, annulez les opérations de suppression en suivant les étapes mentionnées [ici](backup-azure-arm-userestapi-backupazurevms.md#undo-the-deletion).
2. Ensuite, désactivez la fonctionnalité de suppression réversible à l’aide de l’API REST en suivant les étapes mentionnées [ici](use-restapi-update-vault-properties.md#update-soft-delete-state-using-rest-api).
3. Enfin, supprimez les sauvegardes à l’aide de l’API REST comme indiqué [ici](backup-azure-arm-userestapi-backupazurevms.md#stop-protection-and-delete-data).

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="do-i-need-to-enable-the-soft-delete-feature-on-every-vault"></a>Dois-je activer la fonctionnalité de suppression réversible sur chaque coffre ?

Non, elle est intégrée et activée par défaut pour tous les coffres Recovery Services.

### <a name="can-i-configure-the-number-of-days-for-which-my-data-will-be-retained-in-soft-deleted-state-after-the-delete-operation-is-complete"></a>Puis-je configurer le nombre de jours pendant lesquels mes données sont conservées dans un état de suppression réversible, une fois l’opération de suppression effectuée ?

Non, la durée de conservation supplémentaire après l’opération de suppression est fixée à 14 jours.

### <a name="do-i-need-to-pay-the-cost-for-this-additional-14-day-retention"></a>Dois-je payer le coût de cette conservation supplémentaire de 14 jours ?

Non, cette conservation supplémentaire de 14 jours est gratuite dans le cadre de la fonctionnalité de suppression réversible.

### <a name="can-i-perform-a-restore-operation-when-my-data-is-in-soft-delete-state"></a>Puis-je effectuer une opération de restauration quand mes données sont dans l’état de suppression réversible ?

Non, vous devez annuler la suppression de la ressource supprimée de manière réversible pour effectuer la restauration. L’opération d’annulation de suppression replace la ressource dans l’état **Arrêter la protection avec conservation des données**, qui vous permet d’effectuer une restauration à n’importe quel point dans le temps. Le récupérateur de mémoire reste en pause dans cet état.

### <a name="will-my-snapshots-follow-the-same-lifecycle-as-my-recovery-points-in-the-vault"></a>Mes instantanés suivent-ils le même cycle de vie que mes points de récupération dans le coffre ?

Oui.

### <a name="how-can-i-trigger-the-scheduled-backups-again-for-a-soft-deleted-resource"></a>Comment puis-je redéclencher des sauvegardes planifiées pour une ressource supprimée de manière réversible ?

Une annulation de la suppression suivie d’une opération de reprise reprotège la ressource. L’opération de reprise associe une stratégie de sauvegarde permettant de déclencher les sauvegardes planifiées à la période de conservation sélectionnée. En outre, le récupérateur de mémoire s’exécute dès que l’opération de reprise est terminée. Si vous souhaitez effectuer une restauration à partir d’un point de récupération qui a dépassé sa date d’expiration, il est recommandé de le faire avant de déclencher l’opération de reprise.

### <a name="can-i-delete-my-vault-if-there-are-soft-deleted-items-in-the-vault"></a>Puis-je supprimer mon coffre s’il contient des éléments supprimés de manière réversible ?

Le coffre Recovery Services ne peut pas être supprimé s’il contient des éléments de sauvegarde à l’état de suppression réversible. Les éléments supprimés de manière réversible sont définitivement supprimés au bout de 14 jours après l’opération de suppression. Si vous ne pouvez pas attendre 14 jours, [désactivez la suppression réversible](#enabling-and-disabling-soft-delete), annulez la suppression des éléments supprimés de manière réversible et supprimez-les à nouveau pour qu’ils soient définitivement supprimés. Après vous être assuré qu’il n’y a pas d’éléments protégés ou d’éléments supprimés de manière réversible, le coffre peut être supprimé.  

### <a name="can-i-delete-the-data-earlier-than-the-14-days-soft-delete-period-after-deletion"></a>Puis-je supprimer les données antérieures à la période de suppression réversible de 14 jours après la suppression ?

Non. Vous ne pouvez pas forcer la suppression des éléments supprimés de manière réversible. Ils sont automatiquement supprimés après 14 jours. Cette fonctionnalité de sécurité est activée pour protéger les données sauvegardées contre des suppressions accidentelles ou malintentionnées.  Vous devez attendre 14 jours avant d’effectuer toute autre action sur l’élément.  Les éléments supprimés de manière réversible ne sont pas facturés.  Si vous devez reprotéger les éléments marqués pour une suppression réversible dans un délai de 14 jours dans un nouveau coffre, contactez le Support Microsoft.

### <a name="can-soft-delete-operations-be-performed-in-powershell-or-cli"></a>Est-il possible d’effectuer des opérations de suppression réversible dans PowerShell ou CLI ?

Les opérations de suppression réversible peuvent être effectuées à l’aide de PowerShell. Actuellement, l’interface CLI n’est pas prise en charge.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des fonctionnalités de sécurité de Sauvegarde Azure](security-overview.md)
