---
title: Suppression réversible pour les machines virtuelles
description: Découvrez comment la suppression réversible pour les machines virtuelles rend les sauvegardes plus sécurisées.
ms.topic: conceptual
ms.date: 04/30/2020
ms.custom: references_regions
ms.openlocfilehash: a8b70d4c8240d096c19e5a8d7449921557b8896c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89022239"
---
# <a name="soft-delete-for-virtual-machines"></a>Suppression réversible pour les machines virtuelles

La suppression réversible pour les machines virtuelles protège les sauvegardes de vos machines virtuelles des suppressions involontaires. Même après la suppression des sauvegardes, celles-ci sont conservées à l’état de suppression réversible pendant 14 jours supplémentaires.

> [!NOTE]
> La suppression réversible protège uniquement les données de sauvegarde supprimées. Si une machine virtuelle est supprimée sans sauvegarde, la fonctionnalité de suppression réversible ne conserve pas les données. Toutes les ressources doivent être protégées avec Sauvegarde Azure pour garantir une résilience complète.
>

## <a name="supported-regions"></a>Régions prises en charge

La suppression réversible est prise en charge dans les régions USA Centre-Ouest, Asie Est, Canada Centre, Canada Est, France Centre, France Sud, Corée Centre, Corée Sud, Royaume-Uni Sud, Royaume-Uni Ouest, Australie Est, Australie Sud-Est, Europe Nord, USA Ouest, USA Ouest 2, USA Centre, Asie Sud-Est, USA Centre Nord, USA Centre Sud, Japon Est, Japon Ouest, Inde Sud, Inde Centre, Inde Ouest, USA Est 2, Suisse Nord, Suisse Ouest, Norvège Ouest, Norvège Est et toutes les régions Pays.

## <a name="soft-delete-for-vms-using-azure-portal"></a>Suppression réversible pour les machines virtuelles avec le portail Azure

1. Pour supprimer les données de sauvegarde d’une machine virtuelle, vous devez arrêter la sauvegarde. Dans le portail Azure, accédez à votre coffre Recovery Services, cliquez avec le bouton droit sur l’élément de sauvegarde, puis choisissez **Arrêter la sauvegarde**.

   ![Capture d’écran des éléments de sauvegarde du portail Azure](./media/backup-azure-security-feature-cloud/backup-stopped.png)

2. Dans la fenêtre suivante, vous pouvez choisir de supprimer ou de conserver les données de sauvegarde. Si vous choisissez **Supprimer les données de sauvegarde**, puis **Arrêter la sauvegarde**, la sauvegarde de la machine virtuelle n’est pas supprimée définitivement. Au lieu de cela, les données de sauvegarde sont conservées pendant 14 jours dans l’état de suppression réversible. Si vous choisissez **Supprimer les données de sauvegarde**, une alerte de suppression par e-mail est envoyée à l’ID d’e-mail configuré afin d’informer l’utilisateur qu’il reste 14 jours de conservation prolongée pour les données de sauvegarde. De plus, une alerte par e-mail est envoyée le douzième jour afin de signaler qu’il reste deux jours supplémentaires pour récupérer les données supprimées. La suppression est différée jusqu’au quinzième jour, moment auquel la suppression définitive a lieu et où une dernière alerte par e-mail est envoyée pour signaler la suppression définitive des données.

   ![Capture d’écran du portail Azure, écran Arrêter la sauvegarde](./media/backup-azure-security-feature-cloud/delete-backup-data.png)

3. Pendant ces 14 jours, dans le coffre Recovery Services, la machine virtuelle supprimée de manière réversible s’affiche avec une icône rouge de suppression réversible.

   ![Capture d’écran du portail Azure, machine virtuelle dans l’état de suppression réversible](./media/backup-azure-security-feature-cloud/vm-soft-delete.png)

   > [!NOTE]
   > Si des éléments de sauvegarde supprimés de manière réversible sont présents dans le coffre, celui-ci ne peut pas être supprimé à ce moment-là. Essayez de supprimer le coffre une fois que les éléments de sauvegarde ont été supprimés définitivement et qu’il ne contient plus d’éléments en état de suppression réversible.

4. Pour restaurer la machine virtuelle supprimée de manière réversible, vous devez d’abord annuler sa suppression. Pour ce faire, choisissez la machine virtuelle supprimée de manière réversible, puis sélectionnez l’option **Annuler la suppression**.

   ![Capture d’écran du portail Azure, annulation de la suppression de la machine virtuelle](./media/backup-azure-security-feature-cloud/choose-undelete.png)

   Une fenêtre s’affiche pour vous avertir que si vous choisissez l’annulation de la suppression, tous les points de restauration de la machine virtuelle sont restaurés et disponibles pour effectuer une opération de restauration. La machine virtuelle est conservée à l’état « Arrêter la protection avec conservation des données » avec suspension des sauvegardes et conservation des données de sauvegarde à l’infini sans que la stratégie de sauvegarde soit effective.

   ![Capture d’écran du portail Azure, confirmation de l’annulation de la suppression de la machine virtuelle](./media/backup-azure-security-feature-cloud/undelete-vm.png)

   À ce stade, vous pouvez également restaurer la machine virtuelle en sélectionnant **Restaurer la machine virtuelle** à partir du point de restauration choisi.  

   ![Capture d’écran du portail Azure, option Restaurer la machine virtuelle](./media/backup-azure-security-feature-cloud/restore-vm.png)

   > [!NOTE]
   > Le récupérateur de mémoire s’exécute et nettoie les points de récupération expirés une fois que l’utilisateur a effectué l’opération **Reprendre la sauvegarde**.

5. Une fois que le processus d’annulation de la suppression est terminé, l’état est redéfini sur « Arrêter la sauvegarde avec conservation des données », puis vous pouvez choisir **Reprendre la sauvegarde**. L’opération **Reprendre la sauvegarde** réactive l’élément de sauvegarde, associé à une stratégie de sauvegarde sélectionnée par l’utilisateur qui définit les planifications de sauvegarde et de conservation.

   ![Capture d’écran du portail Azure, option Reprendre la sauvegarde](./media/backup-azure-security-feature-cloud/resume-backup.png)

## <a name="soft-delete-for-vms-using-azure-powershell"></a>Suppression réversible pour les machines virtuelles à l’aide d’Azure PowerShell

> [!IMPORTANT]
> La version minimale d’Az.RecoveryServices qui permet d’utiliser la suppression réversible à l’aide d’Azure PowerShell est la version 2.2.0. Utilisez ```Install-Module -Name Az.RecoveryServices -Force``` pour récupérer la version la plus récente.

Comme indiqué ci-dessus pour le Portail Azure, la séquence d’étapes est la même quand vous utilisez Azure PowerShell.

### <a name="delete-the-backup-item-using-azure-powershell"></a>Supprimer l’élément de sauvegarde à l’aide d’Azure PowerShell

Supprimez l’élément de sauvegarde en utilisant l’applet de commande PowerShell [Disable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection).

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $myBkpItem -RemoveRecoveryPoints -VaultId $myVaultID -Force

WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
AppVM1           DeleteBackupData     Completed            12/5/2019 12:44:15 PM     12/5/2019 12:44:50 PM     0488c3c2-accc-4a91-a1e0-fba09a67d2fb
```

La valeur « DeleteState » de l’élément de sauvegarde passe de « NotDeleted » à « ToBeDeleted ». Les données de sauvegarde sont conservées pendant 14 jours. Si vous souhaitez rétablir l’opération de suppression, vous devez effectuer une opération d’annulation de suppression.

### <a name="undoing-the-deletion-operation-using-azure-powershell"></a>Annulation de l’opération de suppression à l’aide d’Azure PowerShell

Tout d’abord, récupérez l’élément de sauvegarde approprié qui est en état de suppression réversible (c’est-à-dire, sur le point d’être supprimé).

```powershell

Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -VaultId $myVaultID | Where-Object {$_.DeleteState -eq "ToBeDeleted"}

Name                                     ContainerType        ContainerUniqueName                      WorkloadType         ProtectionStatus     HealthStatus         DeleteState
----                                     -------------        -------------------                      ------------         ----------------     ------------         -----------
VM;iaasvmcontainerv2;selfhostrg;AppVM1    AzureVM             iaasvmcontainerv2;selfhostrg;AppVM1       AzureVM              Healthy              Passed               ToBeDeleted

$myBkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -VaultId $myVaultID -Name AppVM1
```

Ensuite, effectuez l’opération d’annulation de suppression en utilisant l’applet de commande PowerShell [Undo-AzRecoveryServicesBackupItemDeletion](/powershell/module/az.recoveryservices/undo-azrecoveryservicesbackupitemdeletion).

```powershell
Undo-AzRecoveryServicesBackupItemDeletion -Item $myBKpItem -VaultId $myVaultID -Force

WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
AppVM1           Undelete             Completed            12/5/2019 12:47:28 PM     12/5/2019 12:47:40 PM     65311982-3755-46b5-8e53-c82ea4f0d2a2
```

La valeur « DeleteState » de l’élément de sauvegarde est rétablie sur « NotDeleted ». Mais la protection est toujours arrêtée. [Reprenez la sauvegarde](./backup-azure-vms-automation.md#change-policy-for-backup-items) pour réactiver la protection.

## <a name="soft-delete-for-vms-using-rest-api"></a>Suppression réversible pour les machines virtuelles avec l'API REST

- Supprimez les sauvegardes à l’aide de l’API REST comme indiqué [ici](backup-azure-arm-userestapi-backupazurevms.md#stop-protection-and-delete-data).
- Si vous souhaitez annuler ces opérations de suppression, reportez-vous aux étapes mentionnées [ici](backup-azure-arm-userestapi-backupazurevms.md#undo-the-deletion).

## <a name="how-to-disable-soft-delete"></a>Guide pratique pour désactiver la suppression réversible

La désactivation de cette fonctionnalité est déconseillée. La seule circonstance où vous devez envisager la désactivation de la suppression réversible est lorsque vous vous préparez à déplacer vos éléments protégés vers un nouveau coffre et que vous ne pouvez pas attendre les 14 jours requis avant d’effectuer la suppression et la reprotection (dans un environnement de test, par exemple). Pour obtenir des instructions sur la désactivation de la suppression réversible, consultez [Activation et désactivation de la suppression réversible](backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete).

## <a name="next-steps"></a>Étapes suivantes

- Lire les [questions fréquentes (FAQ)](backup-azure-security-feature-cloud.md#frequently-asked-questions) sur la suppression réversible
- En savoir plus sur toutes les [fonctionnalités de sécurité de la Sauvegarde Azure](security-overview.md)
