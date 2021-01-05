---
title: Suppression réversible pour les charges de travail SQL Server dans Azure VM et SAP HANA dans Azure VM
description: Découvrez comment la suppression réversible pour les charges de travail SQL Server dans Azure VM et SAP HANA dans Azure VM permet de rendre les sauvegardes plus sécurisées.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 2a442997d426ff0bf4c74b0b45f7657cc0593b82
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91254293"
---
# <a name="soft-delete-for-sql-server-in-azure-vm-and-sap-hana-in-azure-vm-workloads"></a>Suppression réversible pour les charges de travail SQL Server dans Azure VM et SAP HANA dans Azure VM

Le service Sauvegarde Azure propose désormais la suppression réversible pour les charges de travail SQL Server dans Azure VM et SAP HANA dans Azure VM. Ceci s’ajoute au [scénario de suppression réversible de machine virtuelle Azure](soft-delete-virtual-machines.md) déjà pris en charge.

La [suppression réversible](backup-azure-security-feature-cloud.md) est une fonctionnalité de sécurité qui permet de protéger les données de sauvegarde même après leur suppression. Avec la suppression réversible, même si un intervenant malveillant supprime la sauvegarde d’une base de données (ou même si les données de sauvegarde sont accidentellement supprimées), les données de sauvegarde sont conservées pendant 14 jours supplémentaires. Cela permet de récupérer cet élément de sauvegarde sans perte de données. Cette conservation durant 14 jours supplémentaires des données de sauvegarde à l’état de « suppression réversible » n’entraîne aucun frais pour le client.

## <a name="soft-delete-for-sql-server-in-azure-vm-using-azure-portal"></a>Suppression réversible pour SQL Server dans Azure VM via le portail Azure

>[!NOTE]
>Ces instructions s’appliquent également à SAP HANA dans Azure VM.

1. Pour supprimer les données de sauvegarde d’une machine virtuelle sur un serveur SQL Server, vous devez arrêter la sauvegarde. Dans le portail Azure, accédez à votre coffre Recovery Services, accédez à l’élément de sauvegarde, puis choisissez **Arrêter la sauvegarde**.

   ![Arrêter la sauvegarde](./media/soft-delete-sql-saphana-in-azure-vm/stop-backup.png)

2. Dans la fenêtre suivante, vous pouvez choisir de supprimer ou de conserver les données de sauvegarde. Si vous choisissez **Supprimer les données de sauvegarde**, la sauvegarde de base de données n’est pas supprimée définitivement. Au lieu de cela, les données de sauvegarde sont conservées pendant 14 jours dans l’état de suppression réversible. La suppression est différée jusqu’au 15e jour avec des e-mails d’alerte réguliers les premier, 12e et 15e jours informant l’utilisateur sur l’état de sauvegarde de la base de données.

   ![Supprimer les données de sauvegarde](./media/soft-delete-sql-saphana-in-azure-vm/delete-backup-data.png)

3. Pendant ces 14 jours, dans le coffre Recovery Services, l’élément supprimé de manière réversible s’affiche avec une icône rouge de suppression réversible.

   ![Éléments supprimés de manière réversible](./media/soft-delete-sql-saphana-in-azure-vm/soft-deleted-items.png)

4. Pour restaurer la base de données supprimée de manière réversible, vous devez d’abord annuler sa suppression. Pour ce faire, choisissez la base de données supprimée de manière réversible, puis sélectionnez l’option **Annuler la suppression**.

   ![Annuler la suppression de la base de données](./media/soft-delete-sql-saphana-in-azure-vm/undelete-database.png)

   Une fenêtre s’affiche pour vous avertir que si vous choisissez d’annuler la suppression, tous les points de restauration de la base de données sont restaurés et disponibles pour une opération de restauration. L’élément de sauvegarde est conservé à l’état « Arrêter la protection avec conservation des données ». Les sauvegardes sont suspendues et les données de sauvegarde conservées à l’infini sans qu’une stratégie de sauvegarde ne soit effective.

   ![Avertissement relatif à l’annulation de la suppression](./media/soft-delete-sql-saphana-in-azure-vm/undelete-warning.png)

5. À ce stade, vous pouvez également restaurer les données en sélectionnant **Restaurer** pour l’élément de sauvegarde supprimé de manière réversible.

   ![Restaurer une machine virtuelle](./media/soft-delete-sql-saphana-in-azure-vm/restore-vm.png)

6. Une fois terminé le processus d’annulation de la suppression, l’état est redéfini sur « Arrêter la sauvegarde avec conservation des données », puis vous pouvez choisir **Reprendre la sauvegarde**. L’opération **Reprendre la sauvegarde** réactive l’élément de sauvegarde, associé à une stratégie de sauvegarde sélectionnée par l’utilisateur qui définit les planifications de sauvegarde et de conservation.

   ![Reprendre la sauvegarde](./media/soft-delete-sql-saphana-in-azure-vm/resume-backup.png)

## <a name="soft-delete-for-sql-server-in-vm-using-azure-powershell"></a>Suppression réversible pour SQL Server dans une machine virtuelle à l’aide d’Azure PowerShell

>[!NOTE]
>La version minimale d’Az.RecoveryServices qui permet d’utiliser la suppression réversible à l’aide d’Azure PowerShell est la version 2.2.0. Utilisez `Install-Module -Name Az.RecoveryServices -Force` pour récupérer la version la plus récente.

La séquence d’étapes pour l’utilisation d’Azure PowerShell est la même que celle que vous utilisez dans le portail Azure, comme indiqué ci-dessus.

### <a name="delete-the-backup-item-using-azure-powershell"></a>Supprimer l’élément de sauvegarde à l’aide d’Azure PowerShell

Supprimez l’élément de sauvegarde en utilisant l’applet de commande PowerShell [Disable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection).

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $myBkpItem -RemoveRecoveryPoints -VaultId $myVaultID -Force
```

La valeur **DeleteState** de l’élément de sauvegarde passe de **NotDeleted** à **ToBeDeleted**. Les données de sauvegarde sont conservées pendant 14 jours. Si vous souhaitez rétablir l’opération de suppression, vous devez effectuer une opération d’annulation de suppression.

### <a name="undoing-the-deletion-operation-using-azure-powershell"></a>Annulation de l’opération de suppression à l’aide d’Azure PowerShell

Tout d’abord, récupérez l’élément de sauvegarde approprié qui est en état de suppression réversible (c’est-à-dire, sur le point d’être supprimé).

```powershell
Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType SQLDataBase -VaultId $myVaultID | Where-Object {$_.DeleteState -eq "ToBeDeleted"}

$myBkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType SQLDataBase -VaultId $myVaultID -Name AppVM1
```

Ensuite, effectuez l’opération d’annulation de suppression en utilisant l’applet de commande PowerShell [Undo-AzRecoveryServicesBackupItemDeletion](/powershell/module/az.recoveryservices/undo-azrecoveryservicesbackupitemdeletion).

```powershell
Undo-AzRecoveryServicesBackupItemDeletion -Item $myBKpItem -VaultId $myVaultID -Force
```

La valeur **DeleteState** de l’élément de sauvegarde est restaurée à **NotDeleted**. Mais la protection est toujours arrêtée. Reprenez l’exécution de la sauvegarde pour réactiver la protection.

## <a name="how-to-disable-soft-delete"></a>Guide pratique pour désactiver la suppression réversible

La désactivation de cette fonctionnalité est déconseillée. La seule circonstance où vous devez envisager la désactivation de la suppression réversible est lorsque vous vous préparez à déplacer vos éléments protégés vers un nouveau coffre et que vous ne pouvez pas attendre les 14 jours requis avant d’effectuer la suppression et la reprotection (dans un environnement de test, par exemple). Pour obtenir des instructions sur la désactivation de la suppression réversible, consultez [Activation et désactivation de la suppression réversible](backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete).

## <a name="next-steps"></a>Étapes suivantes

- Lire les [questions fréquentes (FAQ)](backup-azure-security-feature-cloud.md#frequently-asked-questions) sur la suppression réversible
- En savoir plus sur toutes les [fonctionnalités de sécurité de la Sauvegarde Azure](security-overview.md)
