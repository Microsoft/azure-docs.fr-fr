---
title: Restaurer Azure Files avec PowerShell
description: Dans cet article, vous découvrirez comment restaurer Azure Files à l’aide du service de Sauvegarde Azure et PowerShell.
ms.topic: conceptual
ms.date: 1/27/2020
ms.openlocfilehash: 60c9848e12de80bcafe4553a9e8f3e27e8876d41
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021382"
---
# <a name="restore-azure-files-with-powershell"></a>Restaurer Azure Files avec PowerShell

Cet article explique comment restaurer l'intégralité d'un partage de fichiers ou des fichiers spécifiques à partir d'un point de restauration créé par le service [Sauvegarde Azure](backup-overview.md) à l'aide d'Azure PowerShell.

Vous pouvez restaurer un partage de fichiers entier ou des fichiers spécifiques sur le partage. Vous pouvez restaurer à l’emplacement d’origine ou à un autre emplacement.

> [!WARNING]
> Vérifiez que la version de PowerShell est mise à niveau vers la version minimale d'« Az.RecoveryServices 2.6.0 » pour les sauvegardes AFS. Pour plus d'informations, reportez-vous à [la section](backup-azure-afs-automation.md#important-notice-backup-item-identification) détaillant les conditions de ce changement.

>[!NOTE]
>Sauvegarde Azure prend désormais en charge la restauration de plusieurs fichiers ou dossiers à l'emplacement d'origine ou à un autre emplacement à l'aide de PowerShell. Pour en savoir plus, reportez-vous à [cette section](#restore-multiple-files-or-folders-to-original-or-alternate-location) du document.

## <a name="fetch-recovery-points"></a>Récupérer des points de récupération

Utilisez [Get-AzRecoveryServicesBackupRecoveryPoint](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint) pour lister tous les points de récupération correspondant à l’élément sauvegardé.

Dans le script suivant :

* La variable **$rp** est un tableau de points de récupération des 7 derniers jours pour l’élément de sauvegarde sélectionné.
* Le tableau est trié dans l’ordre chronologique inverse, le point de récupération le plus récent détenant l’index **0**.
* Utilisez l'indexation de tableau PowerShell standard pour sélectionner le point de récupération.
* Dans l’exemple, **$rp[0]** sélectionne le dernier point de récupération.

```powershell
$vault = Get-AzRecoveryServicesVault -ResourceGroupName "azurefiles" -Name "azurefilesvault"
$Container = Get-AzRecoveryServicesBackupContainer -ContainerType AzureStorage -Status Registered -FriendlyName "afsaccount" -VaultId $vault.ID
$BackupItem = Get-AzRecoveryServicesBackupItem -Container $Container -WorkloadType AzureFiles -VaultId $vault.ID -FriendlyName "azurefiles"
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $BackupItem -VaultId $vault.ID -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime()
$rp[0] | fl
```

Le résultat ressemble à ce qui suit.

```powershell
FileShareSnapshotUri : https://testStorageAcct.file.core.windows.net/testAzureFS?sharesnapshot=2018-11-20T00:31:04.00000
                       00Z
RecoveryPointType    : FileSystemConsistent
RecoveryPointTime    : 11/20/2018 12:31:05 AM
RecoveryPointId      : 86593702401459
ItemName             : testAzureFS
Id                   : /Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Micros                      oft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;teststorageRG;testStorageAcct/protectedItems/AzureFileShare;testAzureFS/recoveryPoints/86593702401462
WorkloadType         : AzureFiles
ContainerName        : storage;teststorageRG;testStorageAcct
ContainerType        : AzureStorage
BackupManagementType : AzureStorage
```

Une fois le point de récupération pertinent sélectionné, vous restaurez le partage de fichiers ou le fichier à l’emplacement d’origine ou à un autre emplacement.

## <a name="restore-an-azure-file-share-to-an-alternate-location"></a>Restaurer un partage de fichiers Azure à un autre emplacement

Utilisez [AzRecoveryServicesBackupItem de restauration](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) pour restaurer jusqu’au point de récupération sélectionné. Spécifiez ces paramètres pour identifier l’autre emplacement :

* **TargetStorageAccountName** : Compte de stockage sur lequel le contenu sauvegardé est restauré. Le compte de stockage cible doit se trouver au même emplacement que le coffre.
* **TargetFileShareName** : Partages de fichiers du compte de stockage cible sur lesquels le contenu sauvegardé est restauré.
* **TargetFolder** : Dossier situé sous le partage de fichiers sur lequel les données sont restaurées. Si le contenu sauvegardé doit être restauré dans le dossier racine, indiquez les valeurs du dossier cible sous forme de chaîne vide.
* **ResolveConflict** : Instruction en cas de conflit avec les données restaurées. Accepte **Remplacer** ou **Ignorer**.

Exécutez la cmdlet avec les paramètres comme suit :

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -TargetStorageAccountName "TargetStorageAcct" -TargetFileShareName "DestAFS" -TargetFolder "testAzureFS_restored" -ResolveConflict Overwrite
```

La commande retourne un travail avec un ID à suivre, comme indiqué dans l’exemple suivant.

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
testAzureFS        Restore              InProgress           12/10/2018 9:56:38 AM                               9fd34525-6c46-496e-980a-3740ccb2ad75
```

## <a name="restore-an-azure-file-to-an-alternate-location"></a>Restaurer un fichier Azure à un autre emplacement

Utilisez [AzRecoveryServicesBackupItem de restauration](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) pour restaurer jusqu’au point de récupération sélectionné. Spécifiez ces paramètres pour identifier l’autre emplacement et pour identifier de façon unique le fichier que vous souhaitez restaurer.

* **TargetStorageAccountName** : Compte de stockage sur lequel le contenu sauvegardé est restauré. Le compte de stockage cible doit se trouver au même emplacement que le coffre.
* **TargetFileShareName** : Partages de fichiers du compte de stockage cible sur lesquels le contenu sauvegardé est restauré.
* **TargetFolder** : Dossier situé sous le partage de fichiers sur lequel les données sont restaurées. Si le contenu sauvegardé doit être restauré dans le dossier racine, indiquez les valeurs du dossier cible sous forme de chaîne vide.
* **SourceFilePath** : Chemin d'accès absolu du fichier, à restaurer dans le partage de fichiers, sous forme de chaîne. Ce chemin d’accès est le même que celui utilisé dans la cmdlet PowerShell **Get-AzStorageFile**.
* **SourceFileType** : Indique si un répertoire ou un fichier est sélectionné. Accepte **Répertoire** ou **Fichier**.
* **ResolveConflict** : Instruction en cas de conflit avec les données restaurées. Accepte **Remplacer** ou **Ignorer**.

Les paramètres supplémentaires (SourceFilePath and SourceFileType) concernent uniquement le fichier individuel que vous souhaitez restaurer.

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -TargetStorageAccountName "TargetStorageAcct" -TargetFileShareName "DestAFS" -TargetFolder "testAzureFS_restored" -SourceFileType File -SourceFilePath "TestDir/TestDoc.docx" -ResolveConflict Overwrite
```

Cette commande retourne un travail avec un ID à suivre, comme indiqué dans la section précédente.

## <a name="restore-azure-file-shares-and-files-to-the-original-location"></a>Restaurer des partages de fichiers et fichiers Azure à l’emplacement d’origine

En cas de restauration à l’emplacement d’origine, vous ne devez pas spécifier tous les paramètres liés à la destination/cible. Seul **ResolveConflict** doit être fourni.

### <a name="overwrite-an-azure-file-share"></a>Remplacer un partage de fichiers Azure

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -ResolveConflict Overwrite
```

### <a name="overwrite-an-azure-file"></a>Remplacer un fichier Azure

```powershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -SourceFileType File -SourceFilePath "TestDir/TestDoc.docx" -ResolveConflict Overwrite
```

## <a name="restore-multiple-files-or-folders-to-original-or-alternate-location"></a>Restaurer plusieurs fichiers ou dossiers à l'emplacement d'origine ou à un autre emplacement

Utilisez la commande [Restore-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) en transmettant le chemin de tous les fichiers ou dossiers que vous souhaitez restaurer en tant que valeur du paramètre **MultipleSourceFilePath**.

### <a name="restore-multiple-files"></a>Restaurer plusieurs fichiers

Dans le script suivant, nous tentons de restaurer les fichiers *FileSharePage.png* et *MyTestFile.txt*.

```powershell
$vault = Get-AzRecoveryServicesVault -ResourceGroupName "azurefiles" -Name "azurefilesvault"

$Container = Get-AzRecoveryServicesBackupContainer -ContainerType AzureStorage -Status Registered -FriendlyName "afsaccount" -VaultId $vault.ID

$BackupItem = Get-AzRecoveryServicesBackupItem -Container $Container -WorkloadType AzureFiles -VaultId $vault.ID -FriendlyName "azurefiles"

$RP = Get-AzRecoveryServicesBackupRecoveryPoint -Item $BackupItem -VaultId $vault.ID

$files = ("FileSharePage.png", "MyTestFile.txt")

Restore-AzRecoveryServicesBackupItem -RecoveryPoint $RP[0] -MultipleSourceFilePath $files -SourceFileType File -ResolveConflict Overwrite -VaultId $vault.ID -VaultLocation $vault.Location
```

### <a name="restore-multiple-directories"></a>Restaurer plusieurs répertoires

Dans le script suivant, nous tentons de restaurer les répertoires *zrs1_restore* et *Restore*.

```powershell
$vault = Get-AzRecoveryServicesVault -ResourceGroupName "azurefiles" -Name "azurefilesvault"

$Container = Get-AzRecoveryServicesBackupContainer -ContainerType AzureStorage -Status Registered -FriendlyName "afsaccount" -VaultId $vault.ID

$BackupItem = Get-AzRecoveryServicesBackupItem -Container $Container -WorkloadType AzureFiles -VaultId $vault.ID -FriendlyName "azurefiles"

$RP = Get-AzRecoveryServicesBackupRecoveryPoint -Item $BackupItem -VaultId $vault.ID

$files = ("Restore","zrs1_restore")

Restore-AzRecoveryServicesBackupItem -RecoveryPoint $RP[0] -MultipleSourceFilePath $files -SourceFileType Directory -ResolveConflict Overwrite -VaultId $vault.ID -VaultLocation $vault.Location
```

Vous devez obtenir un résultat semblable à ce qui suit :

```output
WorkloadName         Operation         Status          StartTime                EndTime       JobID
------------         ---------         ------          ---------                -------       -----
azurefiles           Restore           InProgress      4/5/2020 8:01:24 AM                    cd36abc3-0242-44b1-9964-0a9102b74d57
```

Si vous souhaitez restaurer plusieurs fichiers ou dossiers à un autre emplacement, utilisez les scripts ci-dessus en spécifiant les valeurs des paramètres liés à l'emplacement cible, comme expliqué précédemment dans [Restaurer un fichier Azure à un autre emplacement](#restore-an-azure-file-to-an-alternate-location).

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur](restore-afs.md) la restauration d’Azure Files dans le portail Microsoft Azure.
