---
title: Guide de démarrage rapide - Sauvegarder une machine virtuelle avec PowerShell
description: Dans ce démarrage rapide, découvrez comment sauvegarder vos machines virtuelles Azure avec le module Azure PowerShell.
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 04/16/2019
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 5e83c599ceed76927f2a313f78c83638d708f1bb
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "90985128"
---
# <a name="back-up-a-virtual-machine-in-azure-with-powershell"></a>Sauvegarder une machine virtuelle dans Azure avec PowerShell

Le module [AZ Azure PowerShell](/powershell/azure/new-azureps-module-az) est utilisé pour créer et gérer des ressources Azure à partir de la ligne de commande ou dans des scripts.

La [sauvegarde Azure](backup-overview.md) sauvegarde les applications et machines locales, ainsi que les machines virtuelles Azure. Cet article montre comment sauvegarder une machine virtuelle Azure avec le module AZ. Vous pouvez également sauvegarder une machine virtuelle en utilisant l’interface [Azure CLI](quick-backup-vm-cli.md), ou le [portail Azure](quick-backup-vm-portal.md).

Ce démarrage rapide permet la sauvegarde sur une machine virtuelle Azure existante. Si vous devez créer une machine virtuelle, vous pouvez [créer une machine virtuelle avec Azure PowerShell](../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fpowershell%2fmodule%2ftoc.json).

Ce guide de démarrage rapide nécessite le module AZ Azure PowerShell version 1.0.0 ou ultérieure. Exécutez `Get-Module -ListAvailable Az` pour trouver la version. Si vous devez installer ou mettre à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="sign-in-and-register"></a>Se connecter et s’inscrire

1. Connectez-vous à votre abonnement Azure avec la commande `Connect-AzAccount` et suivez les instructions à l’écran.

    ```powershell
    Connect-AzAccount
    ```

2. Si vous utilisez la sauvegarde Azure pour la première fois, vous devez inscrire le fournisseur Azure Recovery Services dans votre abonnement avec [Register-AzResourceProvider](/powershell/module/az.Resources/Register-azResourceProvider), de la façon suivante :

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

Un [coffre Recovery Services](backup-azure-recovery-services-vault-overview.md) est un conteneur logique qui stocke des données de sauvegarde pour des ressources protégées, telles que des machines virtuelles Azure. Quand un travail de sauvegarde s’exécute, il crée un point de récupération à l’intérieur du coffre Recovery Services. Vous pouvez ensuite utiliser un de ces points de récupération pour restaurer des données à un moment donné dans le temps.

Lorsque vous créez le coffre :

- Pour l’emplacement et le groupe de ressources, spécifiez l’emplacement et le groupe de ressources de la machine virtuelle que vous souhaitez sauvegarder.
- Si vous avez utilisé cet [exemple de script](../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fpowershell%2fmodule%2ftoc.json) pour créer la machine virtuelle, le groupe de ressources est **myResourceGroup**, la machine virtuelle est **myVM** et les ressources se trouvent dans la région **WestEurope**.
- La Sauvegarde Azure gère automatiquement le stockage des données sauvegardées. Par défaut, le coffre utilise [le stockage géoredondant (GRS)](../storage/common/storage-redundancy.md#geo-redundant-storage). La géoredondance garantit que les données sauvegardées sont répliquées dans une région Azure secondaire, à des centaines de kilomètres de la région primaire.

À présent, créez un coffre :

1. Utilisez [New-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault) pour créer le coffre :

    ```powershell
    New-AzRecoveryServicesVault `
        -ResourceGroupName "myResourceGroup" `
        -Name "myRecoveryServicesVault" `
    -Location "WestEurope"
    ```

2. Définissez le contexte du coffre avec [Set-AzRecoveryServicesVaultContext](/powershell/module/az.RecoveryServices/Set-azRecoveryServicesVaultContext), de la façon suivante :

    ```powershell
    Get-AzRecoveryServicesVault `
        -Name "myRecoveryServicesVault" | Set-AzRecoveryServicesVaultContext
    ```

3. Modifiez la configuration de la redondance de stockage (LRS/GRS) du coffre avec [Set-AzRecoveryServicesBackupProperty](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupproperty) comme suit :

    ```powershell
    Get-AzRecoveryServicesVault `
        -Name "myRecoveryServicesVault" | Set-AzRecoveryServicesBackupProperty -BackupStorageRedundancy LocallyRedundant/GeoRedundant
    ```

    > [!NOTE]
    > La redondance de stockage peut être modifiée uniquement si aucun élément de sauvegarde n’est protégé dans le coffre.

## <a name="enable-backup-for-an-azure-vm"></a>Activer la sauvegarde pour une machine virtuelle Azure

Vous activez la sauvegarde d’une machine virtuelle Azure et spécifiez une stratégie de sauvegarde.

- La stratégie définit les moments auxquels les sauvegardes s’exécutent, et la durée de conservation des points de récupération créés par les sauvegardes.
- La stratégie de protection par défaut effectue une sauvegarde quotidienne pour la machine virtuelle, et conserve les points de récupération créés pendant 30 jours. Vous pouvez utiliser cette stratégie par défaut pour rapidement protéger votre machine virtuelle.

Configurez la sauvegarde de la façon suivante :

1. Tout d’abord, définissez la stratégie par défaut avec [Get-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy) :

    ```powershell
    $policy = Get-AzRecoveryServicesBackupProtectionPolicy     -Name "DefaultPolicy"
    ```

2. Activez la sauvegarde de machine virtuelle avec [Enable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection). Spécifiez la stratégie, le groupe de ressources et le nom de la machine virtuelle.

    ```powershell
    Enable-AzRecoveryServicesBackupProtection `
        -ResourceGroupName "myResourceGroup" `
        -Name "myVM" `
        -Policy $policy
    ```

## <a name="start-a-backup-job"></a>Démarrer un travail de sauvegarde

Les sauvegardes s’exécutent conformément à la planification spécifiée dans la stratégie de sauvegarde. Vous pouvez également exécuter une sauvegarde à la demande :

- Le premier travail de sauvegarde initiale crée un point de récupération complet.
- Après cette sauvegarde initiale, chaque travail de sauvegarde crée des points de récupération incrémentielle.
- Les points de récupération incrémentielle constituent un mode de stockage rapide et efficace, car ils transfèrent uniquement les modifications apportées depuis la dernière sauvegarde.

Pour exécuter une sauvegarde à la demande, vous utilisez [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem).

- Vous spécifiez un conteneur dans le coffre qui contient vos données de sauvegarde avec [Get-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer).
- Chaque machine virtuelle à sauvegarder est traitée comme un élément. Pour démarrer un travail de sauvegarde, vous obtenez des informations sur la machine virtuelle avec [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem).

Exécutez un travail de sauvegarde à la demande comme suit :

1. Spécifiez le conteneur, obtenez des informations de machine virtuelle et exécutez la sauvegarde.

    ```powershell
    $backupcontainer = Get-AzRecoveryServicesBackupContainer `
        -ContainerType "AzureVM" `
        -FriendlyName "myVM"

    $item = Get-AzRecoveryServicesBackupItem `
        -Container $backupcontainer `
        -WorkloadType "AzureVM"

    Backup-AzRecoveryServicesBackupItem -Item $item
    ```

2. À partir du moment où ce premier travail de sauvegarde crée un point de récupération complet, il est possible que votre attente dure un certain temps, sans toutefois excéder 20 minutes. Supervisez le travail tel qu’il est décrit dans la procédure suivante.

## <a name="monitor-the-backup-job"></a>Surveiller le travail de sauvegarde

1. Exécutez [Get-AzRecoveryservicesBackupJob](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob) pour superviser l’état du travail.

    ```powershell
    Get-AzRecoveryservicesBackupJob
    ```

    Le résultat est similaire à l’exemple suivant, et montre la tâche **en cours** d’exécution :

    ```output
    WorkloadName   Operation         Status       StartTime              EndTime                JobID
    ------------   ---------         ------       ---------              -------                -----
    myvm           Backup            InProgress   9/18/2017 9:38:02 PM                          9f9e8f14
    myvm           ConfigureBackup   Completed    9/18/2017 9:33:18 PM   9/18/2017 9:33:51 PM   fe79c739
    ```

2. Lorsque l’état du travail est **Terminé**, la machine virtuelle est protégée, et dotée d’un point de récupération complet stocké.

## <a name="clean-up-the-deployment"></a>Nettoyer le déploiement

Si vous n’avez plus besoin de sauvegarder la machine virtuelle, vous pouvez la nettoyer.

- Si vous voulez essayer de restaurer la machine virtuelle, ignorez cette opération de nettoyage.
- Si vous avez utilisé une machine virtuelle existante, vous pouvez ignorer l’applet de commande finale [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour laisser en place le groupe de ressources et la machine virtuelle.

Désactivez la protection, supprimez les points de restauration et le coffre. Supprimez ensuite le groupe de ressources et les ressources de machine virtuelle associées comme suit :

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $item -RemoveRecoveryPoints
$vault = Get-AzRecoveryServicesVault -Name "myRecoveryServicesVault"
Remove-AzRecoveryServicesVault -Vault $vault
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un coffre Recovery Services, activé la protection sur une machine virtuelle et créé le point de récupération initial.

- [Découvrez comment](tutorial-backup-vm-at-scale.md) sauvegarder des machines virtuelles dans le portail Azure.
- [Découvrez comment](tutorial-restore-disk.md) rapidement restaurer une machine virtuelle
