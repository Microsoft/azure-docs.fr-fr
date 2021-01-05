---
title: Gérer les sauvegardes de partage de fichiers Azure avec PowerShell
description: Découvrez comment PowerShell pour gérer et superviser les partages de fichiers Azure sauvegardés par le service Sauvegarde Azure.
ms.topic: conceptual
ms.date: 1/27/2020
ms.openlocfilehash: e2f07e56fb9a8715b1b53165ab5f4b45b4e20ccb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89000224"
---
# <a name="manage-azure-file-share-backups-with-powershell"></a>Gérer les sauvegardes de partage de fichiers Azure avec PowerShell

Cet article explique comment utiliser Azure PowerShell pour gérer et superviser les partages de fichiers Azure sauvegardés par le service Sauvegarde Azure.

> [!WARNING]
> Vérifiez que la version de PowerShell est mise à niveau vers la version minimale d’« Az.RecoveryServices 2.6.0 » pour les sauvegardes AFS. Pour plus d’informations, reportez-vous à [la section](backup-azure-afs-automation.md#important-notice-backup-item-identification) détaillant les conditions de ce changement.

## <a name="modify-the-protection-policy"></a>Modifier la stratégie de protection

Pour modifier la stratégie utilisée pour sauvegarder le partage de fichiers Azure, utilisez [Enable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection). Spécifiez l’élément de sauvegarde pertinent et la nouvelle stratégie de sauvegarde.

Les exemples suivants modifient la stratégie de protection de **testAzureFS** en remplaçant **dailyafs** par **monthlyafs**.

```powershell
$monthlyafsPol =  Get-AzRecoveryServicesBackupProtectionPolicy -Name "monthlyafs"
$afsContainer = Get-AzRecoveryServicesBackupContainer -FriendlyName "testStorageAcct" -ContainerType AzureStorage
$afsBkpItem = Get-AzRecoveryServicesBackupItem -Container $afsContainer -WorkloadType AzureFiles -Name "testAzureFS"
Enable-AzRecoveryServicesBackupProtection -Item $afsBkpItem -Policy $monthlyafsPol
```

## <a name="track-backup-and-restore-jobs"></a>Suivi des travaux de sauvegarde et de restauration

Les opérations de sauvegarde et de restauration à la demande retournent un travail assorti d’un ID, comme lorsque vous [exécutez une sauvegarde à la demande](backup-azure-afs-automation.md#trigger-an-on-demand-backup). Utilisez la cmdlet [Get-AzRecoveryServicesBackupJobDetails](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob) pour suivre l’avancement du travail et obtenir des détails.

```powershell
$job = Get-AzRecoveryServicesBackupJob -JobId 00000000-6c46-496e-980a-3740ccb2ad75 -VaultId $vaultID

 $job | fl


IsCancellable        : False
IsRetriable          : False
ErrorDetails         : {Microsoft.Azure.Commands.RecoveryServices.Backup.Cmdlets.Models.AzureFileShareJobErrorInfo}
ActivityId           : 00000000-5b71-4d73-9465-8a4a91f13a36
JobId                : 00000000-6c46-496e-980a-3740ccb2ad75
Operation            : Restore
Status               : Failed
WorkloadName         : testAFS
StartTime            : 12/10/2018 9:56:38 AM
EndTime              : 12/10/2018 11:03:03 AM
Duration             : 01:06:24.4660027
BackupManagementType : AzureStorage

$job.ErrorDetails

 ErrorCode ErrorMessage                                          Recommendations
 --------- ------------                                          ---------------
1073871825 Microsoft Azure Backup encountered an internal error. Wait for a few minutes and then try the operation again. If the issue persists, please contact Microsoft support.
```

## <a name="stop-protection-on-a-file-share"></a>Arrêter la protection sur un partage de fichiers

Il existe deux façons de suspendre la protection des partages de fichiers Azure :

* Arrêter tous les travaux de sauvegarde à venir et *supprimer* tous les points de récupération
* Arrêter tous les travaux de sauvegarde à venir mais *conserver* les points de récupération

Le fait de laisser des points de récupération dans le stockage peut avoir un coût, étant donné que les instantanés sous-jacents créés par Sauvegarde Azure sont conservés. Cependant la conservation des points de récupération présente l’avantage de vous permettre de restaurer le partage de fichiers par la suite si nécessaire. Pour plus d’informations sur les coûts de conservation des points de récupération, voir les [détails de la tarification](https://azure.microsoft.com/pricing/details/storage/files/). Si vous décidez de supprimer tous les points de récupération, vous ne pouvez plus restaurer le partage de fichiers.

## <a name="stop-protection-and-retain-recovery-points"></a>Arrêter la protection et conserver les points de récupération

Pour arrêter la protection tout en conservant les données, utilisez la cmdlet [Disable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection).

L’exemple suivant arrête la protection du partage de fichiers *afsfileshare*, mais conserve tous les points de récupération :

```powershell
$vaultID = Get-AzRecoveryServicesVault -ResourceGroupName "afstesting" -Name "afstest" | select -ExpandProperty ID
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureStorage -WorkloadType AzureFiles -Name "afsfileshare" -VaultId $vaultID
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $vaultID
```

```output
WorkloadName     Operation         Status         StartTime                 EndTime                   JobID
------------     ---------         ------         ---------                 -------                   -----
afsfileshare     DisableBackup     Completed      1/26/2020 2:43:59 PM      1/26/2020 2:44:21 PM      98d9f8a1-54f2-4d85-8433-c32eafbd793f
```

L’attribut Job ID figurant dans la sortie correspond à l’ID du travail créé par le service de sauvegarde pour votre opération d’arrêt de la protection (« stop protection »). Pour obtenir l’état du travail, utilisez la cmdlet [Get-AzRecoveryservicesBackupJob](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob).

## <a name="stop-protection-without-retaining-recovery-points"></a>Arrêter la protection sans conserver les points de récupération

Pour arrêter la protection sans conserver les points de récupération, utilisez la cmdlet [Disable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection) en ajoutant le paramètre **-RemoveRecoveryPoints**.

L’exemple suivant arrête la protection du partage de fichiers *afsfileshare* sans conserver les points de récupération :

```powershell
$vaultID = Get-AzRecoveryServicesVault -ResourceGroupName "afstesting" -Name "afstest" | select -ExpandProperty ID
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureStorage -WorkloadType AzureFiles -Name "afsfileshare" -VaultId $vaultID
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $vaultID -RemoveRecoveryPoints
```

```output
WorkloadName     Operation            Status         StartTime                 EndTime                   JobID
------------     ---------            ------         ---------                 -------                   -----
afsfileshare     DeleteBackupData     Completed      1/26/2020 2:50:57 PM      1/26/2020 2:51:39 PM      b1a61c0b-548a-4687-9d15-9db1cc5bcc85
```

## <a name="next-steps"></a>Étapes suivantes

[Apprenez-en davantage](manage-afs-backup.md) sur la gestion des sauvegardes de partages de fichiers Azure dans le portail Azure.
