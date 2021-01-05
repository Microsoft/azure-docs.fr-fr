---
title: 'Tutoriel : Sauvegarder plusieurs machines virtuelles Azure avec PowerShell'
description: Ce tutoriel décrit en détail la sauvegarde de plusieurs machines virtuelles Azure dans un coffre Recovery Services à l’aide d’Azure PowerShell.
ms.topic: tutorial
ms.date: 03/05/2019
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 53a5df430a3ea57201ecb19bac68ef9f073beb14
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90980966"
---
# <a name="back-up-azure-vms-with-powershell"></a>Sauvegarder des machines virtuelles Azure avec PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Ce tutoriel explique comment déployer un coffre Recovery Services de la [Sauvegarde Azure](backup-overview.md) pour sauvegarder plusieurs machines virtuelles Azure à l’aide de PowerShell.  

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Créer un coffre Recovery Services et définir le contexte du coffre.
> * Créer une stratégie de sauvegarde
> * Appliquer la stratégie de sauvegarde pour protéger plusieurs machines virtuelles
> * Déclencher un travail de sauvegarde à la demande pour les machines virtuelles protégées Avant de sauvegarder (ou de protéger) une machine virtuelle, vous devez remplir les [prérequis](backup-azure-arm-vms-prepare.md) pour préparer votre environnement à la protection de vos machines virtuelles.

> [!IMPORTANT]
> Ce didacticiel suppose que vous avez déjà créé un groupe de ressources et une machine virtuelle Azure.

## <a name="sign-in-and-register"></a>Se connecter et s’inscrire

1. Connectez-vous à votre abonnement Azure avec la commande `Connect-AzAccount` et suivez les instructions à l’écran.

    ```powershell
    Connect-AzAccount
    ```

2. Si vous utilisez le service Sauvegarde Azure pour la première fois, vous devez inscrire le fournisseur Azure Recovery Services dans votre abonnement avec [Register-AzResourceProvider](/powershell/module/az.Resources/Register-azResourceProvider). Si vous avez déjà effectué l’inscription, ignorez cette étape.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

Un [coffre Recovery Services](backup-azure-recovery-services-vault-overview.md) est un conteneur logique qui stocke des données de sauvegarde pour des ressources protégées, telles que des machines virtuelles Azure. Quand un travail de sauvegarde s’exécute, il crée un point de récupération à l’intérieur du coffre Recovery Services. Vous pouvez ensuite utiliser un de ces points de récupération pour restaurer des données à un moment donné dans le temps.

* Dans ce tutoriel, vous créez le coffre dans le même groupe de ressources et au même emplacement que la machine virtuelle que vous voulez sauvegarder.
* La Sauvegarde Azure gère automatiquement le stockage des données sauvegardées. Par défaut, le coffre utilise [le stockage géoredondant (GRS)](../storage/common/storage-redundancy.md#geo-redundant-storage). La géoredondance garantit que les données sauvegardées sont répliquées dans une région Azure secondaire, à des centaines de kilomètres de la région primaire.

Créez le coffre comme suit :

1. Utilisez [New-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault) pour créer le coffre. Spécifiez le nom et l’emplacement de groupe de ressources de la machine virtuelle que vous voulez sauvegarder.

    ```powershell
    New-AzRecoveryServicesVault -Name myRSvault -ResourceGroupName "myResourceGroup" -Location "EastUS"
    ```

2. De nombreuses applets de commande Azure Backup nécessitent l’objet coffre Recovery Services en tant qu’entrée. Pour cette raison, il est pratique de stocker l’objet coffre Backup Recovery Services dans une variable.

    ```powershell
    $vault1 = Get-AzRecoveryServicesVault –Name myRSVault
    ```

3. Définissez le contexte du coffre avec [Set-AzRecoveryServicesVaultContext](/powershell/module/az.RecoveryServices/Set-azRecoveryServicesVaultContext).

   * Le contexte du coffre spécifie le type de données protégées dans le coffre.
   * Une fois le contexte défini, il s’applique à toutes les applets de commande suivantes.

     ```powershell
     Get-AzRecoveryServicesVault -Name "myRSVault" | Set-AzRecoveryServicesVaultContext
     ```

## <a name="back-up-azure-vms"></a>Sauvegarder des machines virtuelles Azure

Les sauvegardes s’exécutent conformément à la planification spécifiée dans la stratégie de sauvegarde. Quand vous créez un coffre Recovery Services, il est fourni avec des stratégies de protection et de conservation par défaut.

* La stratégie de protection par défaut déclenche un travail de sauvegarde une fois par jour à une heure spécifiée.
* La stratégie de conservation par défaut conserve le point de récupération quotidien pendant 30 jours.

Pour activer et sauvegarder la machine virtuelle Azure dans ce tutoriel, nous effectuons les opérations suivantes :

1. Spécifiez un conteneur dans le coffre qui contient vos données de sauvegarde avec [Get-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/get-Azrecoveryservicesbackupcontainer).
2. Chaque machine virtuelle pour la sauvegarde est un élément. Pour démarrer un travail de sauvegarde, vous obtenez des informations sur la machine virtuelle avec [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupItem).
3. Exécutez une sauvegarde à la demande avec [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/backup-Azrecoveryservicesbackupitem).
    * Le premier travail de sauvegarde initiale crée un point de récupération complet.
    * Après cette sauvegarde initiale, chaque travail de sauvegarde crée des points de récupération incrémentielle.
    * Les points de récupération incrémentielle constituent un mode de stockage rapide et efficace, car ils transfèrent uniquement les modifications apportées depuis la dernière sauvegarde.

Activez et exécutez la sauvegarde comme suit :

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM -Status Registered -FriendlyName "V2VM"
$item = Get-AzRecoveryServicesBackupItem -Container $namedContainer -WorkloadType AzureVM
$job = Backup-AzRecoveryServicesBackupItem -Item $item
```

## <a name="troubleshooting"></a>Dépannage

Si vous rencontrez des problèmes pendant la sauvegarde de votre machine virtuelle, consultez cet [article sur la résolution des problèmes](backup-azure-vms-troubleshoot.md).

### <a name="deleting-a-recovery-services-vault"></a>Supprimer un coffre Recovery Services

Si vous devez supprimer un coffre, commencez par supprimer les points de récupération dans le coffre, puis annulez l’inscription du coffre, comme suit :

```powershell
$Cont = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM -Status Registered
$PI = Get-AzRecoveryServicesBackupItem -Container $Cont[0] -WorkloadType AzureVm
Disable-AzRecoveryServicesBackupProtection -RemoveRecoveryPoints $PI[0]
Unregister-AzRecoveryServicesBackupContainer -Container $namedContainer
Remove-AzRecoveryServicesVault -Vault $vault1
```

## <a name="next-steps"></a>Étapes suivantes

* [Consultez](backup-azure-vms-automation.md) une procédure pas à pas détaillée de la sauvegarde et de la restauration de machines virtuelles Azure avec PowerShell.
* [Gérer et superviser des machines virtuelles Azure](backup-azure-manage-vms.md)
* [Restauration de machines virtuelles Azure](backup-azure-arm-restore-vms.md)
