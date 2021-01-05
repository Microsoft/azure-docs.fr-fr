---
title: Sauvegarder et récupérer des machines virtuelles Azure avec PowerShell
description: Décrit comment sauvegarder et restaurer des machines virtuelles Azure à l’aide de Sauvegarde Azure avec PowerShell
ms.topic: conceptual
ms.date: 09/11/2019
ms.openlocfilehash: ded2bc8a71bf564e31f40ca9f0d6c8049188768b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95978367"
---
# <a name="back-up-and-restore-azure-vms-with-powershell"></a>Sauvegarder et restaurer des machines virtuelles Azure avec PowerShell

Cet article explique comment sauvegarder et restaurer une machine virtuelle Azure dans un coffre Recovery Services [Sauvegarde Azure](backup-overview.md) à l’aide de cmdlets PowerShell.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
>
> * Créer un coffre Recovery Services et définir le contexte du coffre.
> * Créer une stratégie de sauvegarde
> * Appliquer la stratégie de sauvegarde pour protéger plusieurs machines virtuelles
> * Déclencher un travail de sauvegarde à la demande pour les machines virtuelles protégées Avant de sauvegarder (ou de protéger) une machine virtuelle, vous devez remplir les [prérequis](backup-azure-arm-vms-prepare.md) pour préparer votre environnement à la protection de vos machines virtuelles.

## <a name="before-you-start"></a>Avant de commencer

* [En savoir plus](backup-azure-recovery-services-vault-overview.md) sur les coffres Recovery Services.
* [Examinez](backup-architecture.md#architecture-built-in-azure-vm-backup) l’architecture de sauvegarde de la machine virtuelle Azure, [apprenez-en davantage sur](backup-azure-vms-introduction.md) le processus de sauvegarde et [examinez](backup-support-matrix-iaas.md) la prise en charge, les limites et les prérequis.
* Passez en revue la hiérarchie des objets PowerShell pour Recovery Services.

## <a name="recovery-services-object-hierarchy"></a>Hiérarchie des objets dans Recovery Services

La hiérarchie des objets est résumée dans le schéma suivant.

![Hiérarchie des objets dans Recovery Services](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

Passez en revue la [référence sur la cmdlet](/powershell/module/az.recoveryservices/) **Az.RecoveryServices** dans la bibliothèque Azure.

## <a name="set-up-and-register"></a>Configurer et inscrire

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour commencer :

1. [Télécharger la dernière version de PowerShell](/powershell/azure/install-az-ps)

2. Rechercher les applets de commande PowerShell Azure Backup disponibles en tapant la commande suivante :

    ```powershell
    Get-Command *azrecoveryservices*
    ```

    Les alias et cmdlets pour Sauvegarde Azure, Azure Site Recovery et le coffre Recovery Services s’affichent. L’image suivante est un exemple de ce que vous allez voir. Il ne s’agit pas de la liste complète des applets de commande.

    ![Liste des services Recovery Services](./media/backup-azure-vms-automation/list-of-recoveryservices-ps.png)

3. Connectez-vous à votre compte Azure à l'aide de **Connect-AzAccount**. Cette applet de commande permet d’afficher une page web qui vous demande les informations d’identification de votre compte :

    * Vous pouvez également inclure les informations d'identification de votre compte en tant que paramètre dans la cmdlet **Connect-AzAccount** à l'aide du paramètre **-Credential**.
    * Si vous êtes partenaire CSP travaillant pour le compte d’un locataire, spécifiez le client en tant que locataire, à l’aide de son ID locataire ou de son nom de domaine principal. Par exemple : **Connect-AzAccount -Tenant "fabrikam.com"**

4. Associez l’abonnement que vous souhaitez utiliser avec le compte, car un compte peut compter plusieurs abonnements :

    ```powershell
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

5. Si vous utilisez le service de sauvegarde Azure pour la première fois, vous devez utiliser l’applet de commande **[Register-AzResourceProvider](/powershell/module/az.resources/register-azresourceprovider)** pour inscrire le fournisseur Azure Recovery Service avec votre abonnement.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

6. Vous pouvez vérifier que les fournisseurs ont été correctement inscrits à l’aide des commandes suivantes :

    ```powershell
    Get-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

    Dans la sortie de commande, **RegistrationState** doit prendre la valeur **Inscrit**. Si ce n'est pas ce cas, réexécutez la cmdlet **[Register-AzResourceProvider](/powershell/module/az.resources/register-azresourceprovider)** .

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

Les étapes suivantes vous montrent comment créer un coffre Recovery Services. Un coffre Recovery Services diffère d’un coffre de sauvegarde.

1. Le coffre Recovery Services étant une ressource Resource Manager, vous devez le placer dans un groupe de ressources. Vous pouvez utiliser un groupe de ressources existant ou en créer un avec la cmdlet **[New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup)** . Quand vous créez un groupe de ressources, spécifiez son nom et son emplacement.  

    ```powershell
    New-AzResourceGroup -Name "test-rg" -Location "West US"
    ```

2. Utilisez la cmdlet [New-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault) pour créer le coffre Recovery Services. Spécifiez pour le coffre le même emplacement que pour le groupe de ressources.

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "test-rg" -Location "West US"
    ```

3. Spécifiez le type de redondance de stockage à utiliser. Vous pouvez utiliser le [stockage localement redondant (LRS)](../storage/common/storage-redundancy.md#locally-redundant-storage), le [stockage géoredondant (GRS)](../storage/common/storage-redundancy.md#geo-redundant-storage) ou le [stockage redondant interzone (ZRS)](../storage/common/storage-redundancy.md#zone-redundant-storage). L’exemple suivant montre que l’option **-BackupStorageRedundancy** pour *testvault* est définie sur **GeoRedundant**.

    ```powershell
    $vault1 = Get-AzRecoveryServicesVault -Name "testvault"
    Set-AzRecoveryServicesBackupProperty  -Vault $vault1 -BackupStorageRedundancy GeoRedundant
    ```

   > [!TIP]
   > De nombreuses applets de commande Azure Backup nécessitent l’objet coffre Recovery Services en tant qu’entrée. Pour cette raison, il est pratique de stocker l’objet coffre Backup Recovery Services dans une variable.
   >
   >

## <a name="view-the-vaults-in-a-subscription"></a>Afficher les coffres dans un abonnement

Pour afficher tous les coffres de l'abonnement, utilisez [Get-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/get-azrecoveryservicesvault) :

```powershell
Get-AzRecoveryServicesVault
```

La sortie ressemble à celle de l’exemple suivant. Notez que le paramètre ResourceGroupName et l’emplacement associés sont fournis.

```output
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

## <a name="back-up-azure-vms"></a>Sauvegarder des machines virtuelles Azure

Utilisez un coffre Recovery Services pour protéger vos machines virtuelles. Avant d’appliquer la protection, définissez le contexte du coffre (le type de données qu’il protège) et vérifiez la stratégie de protection. La stratégie de protection consiste à planifier l’exécution du travail de sauvegarde et la durée de conservation de chaque instantané de sauvegarde.

### <a name="set-vault-context"></a>Définir le contexte du coffre

Avant d’activer la protection sur une machine virtuelle, utilisez [Set-AzRecoveryServicesVaultContext](/powershell/module/az.recoveryservices/set-azrecoveryservicesvaultcontext) pour définir le contexte du coffre. Une fois le contexte du coffre défini, il s’applique à toutes les applets de commande suivantes. L’exemple suivant définit le contexte du coffre pour le coffre *testvault*.

```powershell
Get-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "Contoso-docs-rg" | Set-AzRecoveryServicesVaultContext
```

### <a name="fetch-the-vault-id"></a>Récupérer l’ID de coffre

Conformément aux instructions d’Azure PowerShell, nous prévoyons de déprécier la définition du contexte de coffre. À la place, vous pouvez stocker ou récupérer l’ID du coffre et le transmettre aux commandes appropriées. Ainsi, si vous n’avez pas défini le contexte du coffre ou si vous souhaitez spécifier la commande à exécuter pour un coffre donné, transmettez l’ID de coffre sous la forme « -vaultID » à l’ensemble des commandes appropriées, comme suit :

```powershell
$targetVault = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault"
$targetVault.ID
```

ou

```powershell
$targetVaultID = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault" | select -ExpandProperty ID
```

### <a name="modifying-storage-replication-settings"></a>Modification des paramètres de réplication du stockage

Utiliser la commande [Set-AzRecoveryServicesBackupProperty](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupproperty) pour définir la configuration de la réplication du stockage du coffre sur LRS/GRS

```powershell
Set-AzRecoveryServicesBackupProperty -Vault $targetVault -BackupStorageRedundancy GeoRedundant/LocallyRedundant
```

> [!NOTE]
> La redondance de stockage peut être modifiée uniquement si aucun élément de sauvegarde n’est protégé dans le coffre.

### <a name="create-a-protection-policy"></a>Création d’une stratégie de protection

Quand vous créez un coffre Recovery Services, il est fourni avec des stratégies de protection et de conservation par défaut. La stratégie de protection par défaut déclenche une tâche de sauvegarde chaque jour à une heure spécifiée. La stratégie de conservation par défaut conserve le point de récupération quotidien pendant 30 jours. Vous pouvez utiliser la stratégie par défaut pour protéger rapidement votre machine virtuelle et modifier la stratégie ultérieurement avec différents détails.

Utilisez **[Get-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy)** pour afficher les stratégies de protection disponibles dans le coffre. Vous pouvez utiliser cette applet de commande pour obtenir une stratégie spécifique ou pour afficher les stratégies associées à un type de charge de travail. L’exemple suivant obtient les stratégies pour le type de charge de travail AzureVM.

```powershell
Get-AzRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureVM" -VaultId $targetVault.ID
```

Le résultat ressemble à l’exemple suivant :

```output
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
DefaultPolicy        AzureVM            AzureVM              4/14/2016 5:00:00 PM
```

> [!NOTE]
> Le fuseau horaire du champ BackupTime dans PowerShell est UTC. Toutefois, lorsque l’heure de sauvegarde s’affiche dans le portail Azure, elle est alignée sur votre fuseau horaire.
>
>

Une stratégie de protection de la sauvegarde est associée à au moins une stratégie de rétention. Une stratégie de rétention définit la durée de conservation d’un point de restauration avant sa suppression.

* Utilisez [Get-AzRecoveryServicesBackupRetentionPolicyObject](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupretentionpolicyobject) pour afficher la stratégie de rétention par défaut.
* De même, vous pouvez utiliser [Get-AzRecoveryServicesBackupSchedulePolicyObject](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupschedulepolicyobject) pour obtenir la stratégie de planification par défaut.
* La cmdlet [New-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy) crée un objet PowerShell contenant des informations sur la stratégie de sauvegarde.
* Les objets des stratégies de planification et de rétention sont utilisés comme entrées pour la cmdlet New-AzRecoveryServicesBackupProtectionPolicy.

Par défaut, une heure de début est définie dans l’objet de la stratégie de planification. Utilisez l’exemple suivant pour modifier l’heure de début sur l’heure de début souhaitée. L’heure de début souhaitée doit également être au format UTC. L’exemple suivant suppose que l’heure de début souhaitée est 01:00 UTC pour les sauvegardes quotidiennes.

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM"
$UtcTime = Get-Date -Date "2019-03-20 01:00:00Z"
$UtcTime = $UtcTime.ToUniversalTime()
$schpol.ScheduleRunTimes[0] = $UtcTime
```

> [!IMPORTANT]
> Vous devez fournir l’heure de début en multiples de 30 minutes uniquement. Dans l’exemple ci-dessus, il peut s’agir uniquement de « 01:00:00 » ou de « 02:30:00 ». L’heure de début ne peut pas être « 01:15:00 »

L’exemple suivant stocke la stratégie de planification et la stratégie de conservation dans des variables. L’exemple utilise ces variables pour définir les paramètres lors de la création d’une stratégie de protection, *NewPolicy*.

```powershell
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
New-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -WorkloadType "AzureVM" -RetentionPolicy $retPol -SchedulePolicy $schPol -VaultId $targetVault.ID
```

Le résultat ressemble à l’exemple suivant :

```output
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
NewPolicy           AzureVM            AzureVM              4/24/2016 1:30:00 AM
```

### <a name="enable-protection"></a>Activer la protection

Une fois que vous avez défini la stratégie de protection, vous devez encore activer la stratégie pour un élément. Utilisez [Enable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection) pour activer la protection. L’activation de la protection nécessite deux objets : l’élément et la stratégie. Une fois que la stratégie a été associée au coffre, le flux de travail de la sauvegarde est déclenché à l’heure planifiée dans la planification de la stratégie.

> [!IMPORTANT]
> Si vous utilisez PowerShell pour activer la sauvegarde de plusieurs machines virtuelles en même temps, assurez-vous qu’il n’y a pas plus de 100 machines virtuelles associées à une même stratégie. Il s’agit d’[une meilleure pratique recommandée](./backup-azure-vm-backup-faq.md#is-there-a-limit-on-number-of-vms-that-can-beassociated-with-the-same-backup-policy). Si, pour l’heure, le client PowerShell ne se bloque pas explicitement dans le cas où plus de 100 machines virtuelles sont présentes, cela sera vérifié à l’avenir.

Les exemples suivants activent la protection pour l’élément V2VM. Ils utilisent la stratégie NewPolicy. Les exemples diffèrent selon que la machine virtuelle est chiffrée et le type de chiffrement.

Pour activer la protection sur des **machines virtuelles Resource Manager non chiffrées** :

```powershell
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1" -VaultId $targetVault.ID
```

Pour activer la protection sur des machines virtuelles chiffrées (à l’aide de clés BEK et KEK), vous devez accorder au service Sauvegarde Azure l’autorisation de lire les clés et les secrets depuis le coffre de clés.

```powershell
Set-AzKeyVaultAccessPolicy -VaultName "KeyVaultName" -ResourceGroupName "RGNameOfKeyVault" -PermissionsToKeys backup,get,list -PermissionsToSecrets get,list -ServicePrincipalName 262044b1-e2ce-469f-a196-69ab7ada62d3
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1" -VaultId $targetVault.ID
```

Pour activer la protection sur des **machines virtuelles chiffrées (à l’aide de clés BEK uniquement)** , vous devez accorder au service Sauvegarde Azure l’autorisation de lire les secrets depuis le coffre de clés.

```powershell
Set-AzKeyVaultAccessPolicy -VaultName "KeyVaultName" -ResourceGroupName "RGNameOfKeyVault" -PermissionsToSecrets backup,get,list -ServicePrincipalName 262044b1-e2ce-469f-a196-69ab7ada62d3
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1" -VaultId $targetVault.ID
```

> [!NOTE]
> Si vous êtes sur le cloud Azure Government, utilisez la valeur `ff281ffe-705c-4f53-9f37-a40e6f2c68f3` pour le paramètre **ServicePrincipalName** dans l’applet de commande [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy).
>

## <a name="monitoring-a-backup-job"></a>Surveillance d’une tâche de sauvegarde

Vous pouvez surveiller les opérations à exécution longue, comme les tâches de sauvegarde, sans utiliser le portail Azure. Pour obtenir l’état d’une tâche en cours d’exécution, utilisez la cmdlet [Get-AzRecoveryservicesBackupJob](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob). Cette applet de commande obtient les tâches de sauvegarde pour un coffre spécifique, et ce coffre est spécifié dans le contexte du coffre. L’exemple suivant obtient l’état d’une tâche en cours d’exécution sous forme de tableau et stocke l’état dans la variable $joblist.

```powershell
$joblist = Get-AzRecoveryservicesBackupJob –Status "InProgress" -VaultId $targetVault.ID
$joblist[0]
```

Le résultat ressemble à l’exemple suivant :

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   ----------
V2VM             Backup               InProgress            4/23/2016                5:00:30 PM                cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

Au lieu d’interroger ces travaux pour connaître leur état d’avancement, ce qui implique du code supplémentaire inutile, utilisez la cmdlet [Wait-AzRecoveryServicesBackupJob](/powershell/module/az.recoveryservices/wait-azrecoveryservicesbackupjob). Cette applet de commande suspend l’exécution jusqu’à la fin de la tâche ou jusqu’à ce que la valeur spécifiée pour l’expiration du délai soit atteinte.

```powershell
Wait-AzRecoveryServicesBackupJob -Job $joblist[0] -Timeout 43200 -VaultId $targetVault.ID
```

## <a name="manage-azure-vm-backups"></a>Gérer les sauvegardes de machines virtuelles Azure

### <a name="modify-a-protection-policy"></a>Modifier une stratégie de protection

Pour modifier la stratégie de protection, utilisez [Set-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy) afin de modifier les objets SchedulePolicy ou RetentionPolicy.

#### <a name="modifying-scheduled-time"></a>Modifier l'heure planifiée

Lorsque vous créez une stratégie de protection, une heure de début lui est attribuée par défaut. Les exemples suivants montrent comment modifier l’heure de début d’une stratégie de protection.

````powershell
$SchPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM"
$UtcTime = Get-Date -Date "2019-03-20 01:00:00Z" (This is the time that you want to start the backup)
$UtcTime = $UtcTime.ToUniversalTime()
$SchPol.ScheduleRunTimes[0] = $UtcTime
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $pol  -SchedulePolicy $SchPol -VaultId $targetVault.ID
````

#### <a name="modifying-retention"></a>Modification de la rétention

L’exemple suivant change la conservation des points de récupération en 365 jours.

```powershell
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
$retPol.DailySchedule.DurationCountInDays = 365
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -VaultId $targetVault.ID
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $pol  -RetentionPolicy $RetPol -VaultId $targetVault.ID
```

#### <a name="configuring-instant-restore-snapshot-retention"></a>Configuration de la rétention des instantanés de la restauration instantanée

> [!NOTE]
> À partir de la version 1.6.0 d’Azure PowerShell, il est possible d’utiliser PowerShell pour mettre à jour la période de conservation des instantanés de la restauration d’instantané dans la stratégie

````powershell
$bkpPol = Get-AzRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureVM" -VaultId $targetVault.ID
$bkpPol.SnapshotRetentionInDays=7
Set-AzRecoveryServicesBackupProtectionPolicy -policy $bkpPol -VaultId $targetVault.ID
````

La valeur par défaut est 2. Vous pouvez définir une valeur comprise entre 1 et 5. Pour les stratégies de sauvegarde hebdomadaire, la période est définie sur 5 et ne peut pas être modifiée.

#### <a name="creating-azure-backup-resource-group-during-snapshot-retention"></a>Créer un groupe de ressources Sauvegarde Azure lors de la conservation des instantanés

> [!NOTE]
> À partir de la version 3.7.0 d’Azure PowerShell, il est possible de créer et modifier le groupe de ressources destiné au stockage des instantanés.

Pour en savoir plus sur les règles de création du groupe de ressources et d’autres informations pertinentes, consultez la documentation [Groupe de ressources Sauvegarde Azure pour les machines virtuelles](./backup-during-vm-creation.md#azure-backup-resource-group-for-virtual-machines).

```powershell
$bkpPol = Get-AzureRmRecoveryServicesBackupProtectionPolicy -name "DefaultPolicyForVMs"
$bkpPol.AzureBackupRGName="Contosto_"
$bkpPol.AzureBackupRGNameSuffix="ForVMs"
Set-AzureRmRecoveryServicesBackupProtectionPolicy -policy $bkpPol
```

### <a name="trigger-a-backup"></a>Déclencher une sauvegarde

Utilisez [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/backup-azrecoveryservicesbackupitem) pour déclencher une tâche de sauvegarde. S’il s’agit de la sauvegarde initiale, cette sauvegarde est complète. Les sauvegardes suivantes prennent une copie incrémentielle. L’exemple suivant porte sur une sauvegarde de machine virtuelle conservée pendant 60 jours.

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM" -VaultId $targetVault.ID
$item = Get-AzRecoveryServicesBackupItem -Container $namedContainer -WorkloadType "AzureVM" -VaultId $targetVault.ID
$endDate = (Get-Date).AddDays(60).ToUniversalTime()
$job = Backup-AzRecoveryServicesBackupItem -Item $item -VaultId $targetVault.ID -ExpiryDateTimeUTC $endDate
```

Le résultat ressemble à l’exemple suivant :

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   ----------
V2VM              Backup              InProgress          4/23/2016                  5:00:30 PM                cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

> [!NOTE]
> Le fuseau horaire des champs StartTime et EndTime affichés dans PowerShell est UTC. Toutefois, lorsque l’heure s’affiche dans le portail Azure, elle est alignée sur votre fuseau horaire.
>
>

### <a name="change-policy-for-backup-items"></a>Modifier la stratégie pour les éléments de sauvegarde

Vous pouvez modifier la stratégie existante ou changer la stratégie de l’élément sauvegardé de Policy1 en Policy2. Pour changer de stratégie pour un élément sauvegardé, récupérez la stratégie et l’élément de sauvegarde appropriés, puis utilisez la commande [Enable-AzRecoveryServices](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection) avec l’élément de sauvegarde comme paramètre.

````powershell
$TargetPol1 = Get-AzRecoveryServicesBackupProtectionPolicy -Name <PolicyName> -VaultId $targetVault.ID
$anotherBkpItem = Get-AzRecoveryServicesBackupItem -WorkloadType AzureVM -BackupManagementType AzureVM -Name "<BackupItemName>" -VaultId $targetVault.ID
Enable-AzRecoveryServicesBackupProtection -Item $anotherBkpItem -Policy $TargetPol1 -VaultId $targetVault.ID
````

La commande attend la fin de la configuration de la sauvegarde et renvoie le résultat suivant.

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
TestVM           ConfigureBackup      Completed            3/18/2019 8:00:21 PM      3/18/2019 8:02:16 PM      654e8aa2-4096-402b-b5a9-e5e71a496c4e
```

### <a name="stop-protection"></a>Arrêter la protection

#### <a name="retain-data"></a>Conserver les données

Si vous souhaitez arrêter la protection, vous pouvez utiliser l’applet de commande PowerShell [Disable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection). Les sauvegardes planifiées seront arrêtées, mais les données sauvegardées jusqu’à présent seront conservées indéfiniment.

````powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -Name "<backup item name>" -VaultId $targetVault.ID
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID
````

#### <a name="delete-backup-data"></a>Supprimer les données de sauvegarde

Pour supprimer complètement les données de sauvegarde stockées dans le coffre, ajoutez l’indicateur/le commutateur '-RemoveRecoveryPoints' à la [commande de protection 'disable'](#retain-data).

````powershell
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID -RemoveRecoveryPoints
````

## <a name="restore-an-azure-vm"></a>Restauration d’une machine virtuelle Azure

Il existe une différence importante entre la restauration d’une machine virtuelle à l’aide du portail Azure et la restauration d’une machine virtuelle à l’aide de PowerShell. Avec PowerShell, l’opération de restauration est terminée dès que sont créés les disques et les informations de configuration à partir d’un point de restauration. L’opération de restauration ne crée pas la machine virtuelle. Pour créer une machine virtuelle à partir d’un disque, consultez la section [Créer une machine virtuelle à partir de disques restaurés](backup-azure-vms-automation.md#create-a-vm-from-restored-disks). Si vous ne voulez pas restaurer l’intégralité de la machine virtuelle, mais que vous voulez restaurer ou récupérer quelques fichiers seulement depuis une sauvegarde de machine virtuelle Azure, reportez-vous à la [section de récupération des fichiers](backup-azure-vms-automation.md#restore-files-from-an-azure-vm-backup).

> [!Tip]
> L’opération de restauration ne crée pas la machine virtuelle.
>
>

L’illustration suivante montre la hiérarchie d’objets à partir de RecoveryServicesVault jusqu’à BackupRecoveryPoint.

![Hiérarchie d’objets Recovery Services montrant BackupContainer](./media/backup-azure-vms-arm-automation/backuprecoverypoint-only.png)

Pour restaurer les données de sauvegarde, identifiez l’élément sauvegardé et le point de récupération contenant les données ponctuelles. Utilisez la cmdlet [Restore-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) pour restaurer les données dans votre compte, à partir du coffre.

Les étapes de base pour restaurer une machine virtuelle Azure sont :

* Sélectionnez la machine virtuelle.
* Choisissez un point de récupération.
* Restaurez les disques.
* Créez la machine virtuelle à partir des disques stockés.

### <a name="select-the-vm-when-restoring-files"></a>Sélectionner la machine virtuelle (lors de la restauration de fichiers)

Pour obtenir l’objet PowerShell permettant d’identifier l’élément à restaurer, vous devez commencer au niveau du conteneur dans le coffre et descendre dans la hiérarchie d’objets. Pour sélectionner le conteneur qui représente la machine virtuelle, utilisez l’applet de commande [Get-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer) et dirigez-la vers l’applet de commande [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem).

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer  -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM" -VaultId $targetVault.ID
$backupitem = Get-AzRecoveryServicesBackupItem -Container $namedContainer  -WorkloadType "AzureVM" -VaultId $targetVault.ID
```

### <a name="choose-a-recovery-point-when-restoring-files"></a>Choisir un point de récupération (lors de la restauration de fichiers)

Utilisez la cmdlet [Get-AzRecoveryServicesBackupRecoveryPoint](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint) afin de répertorier tous les points de récupération correspondant à l'élément de sauvegarde. Ensuite, choisissez le point de récupération à restaurer. Si vous ne savez pas quel point de récupération utiliser, nous vous conseillons de choisir le point RecoveryPointType = AppConsistent le plus récent dans la liste.

Dans le script suivant, la variable **$rp** est un tableau de points de récupération des 7 derniers jours pour l’élément de sauvegarde sélectionné. Le tableau est trié dans l’ordre chronologique inverse, le point de récupération le plus récent détenant l’index 0. Utilisez l'indexation de tableau PowerShell standard pour sélectionner le point de récupération. Dans l’exemple, $rp[0] sélectionne le dernier point de récupération.

```powershell
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $backupitem -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime() -VaultId $targetVault.ID
$rp[0]
```

Le résultat ressemble à l’exemple suivant :

```output
RecoveryPointAdditionalInfo :
SourceVMStorageType         : NormalStorage
Name                        : 15260861925810
ItemName                    : VM;iaasvmcontainer;RGName1;V2VM
RecoveryPointId             : /subscriptions/XX/resourceGroups/ RGName1/providers/Microsoft.RecoveryServices/vaults/testvault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainer;RGName1;V2VM/protectedItems/VM;iaasvmcontainer; RGName1;V2VM/recoveryPoints/15260861925810
RecoveryPointType           : AppConsistent
RecoveryPointTime           : 4/23/2016 5:02:04 PM
WorkloadType                : AzureVM
ContainerName               : IaasVMContainer;iaasvmcontainer; RGName1;V2VM
ContainerType               : AzureVM
BackupManagementType        : AzureVM
```

### <a name="restore-the-disks"></a>Restaurer les disques

Utilisez l’applet de commande [Restore-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) pour restaurer les données et la configuration d’un élément de sauvegarde à un point de récupération. Lorsque vous avez identifié un point de récupération, utilisez-le comme valeur du paramètre **-RecoveryPoint**. Dans l’exemple ci-dessus, **$rp[0]** était le point de récupération à utiliser. Dans l’exemple de code suivant, **$rp [0]** est le point de récupération à utiliser pour restaurer le disque.

Pour restaurer les disques et les informations de configuration :

```powershell
$restorejob = Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -VaultId $targetVault.ID
$restorejob
```

#### <a name="restore-managed-disks"></a>Restaurer des disques managés

> [!NOTE]
> Si la machine virtuelle sauvegardée contient des disques managés que vous souhaitez restaurer, vous pouvez désormais le faire dans Azure PowerShell RM module v 6.7.0. et versions ultérieures.
>
>

Précisez un paramètre supplémentaire **TargetResourceGroupName** correspondant au groupe de ressources dans lequel les disques managés seront restaurés.

> [!IMPORTANT]
> Il est vivement recommandé d’utiliser le paramètre **TargetResourceGroupName** pour la restauration des disques managés, car il améliore considérablement les performances. Si ce paramètre n’est pas fourni, vous ne pouvez pas tirer parti de la fonctionnalité de restauration instantanée et l’opération de restauration sera plus lente en comparaison. Si l’objectif est de restaurer des disques managés en tant que disques non managés, ne spécifiez pas ce paramètre et indiquez clairement l’intention en spécifiant le paramètre `-RestoreAsUnmanagedDisks`. Le paramètre `-RestoreAsUnmanagedDisks` est disponible dans Azure PowerShell 3.7.0 et versions ultérieures. Dans les versions ultérieures, il sera obligatoire de spécifier l’un de ces paramètres pour une expérience de restauration correcte.
>
>

```powershell
$restorejob = Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -TargetResourceGroupName "DestRGforManagedDisks" -VaultId $targetVault.ID
```

Le fichier **VMConfig.JSON** sera restauré dans le compte de stockage, et les disques managés seront restaurés dans le groupe de ressources cible spécifié.

Le résultat ressemble à l’exemple suivant :

```output
WorkloadName     Operation          Status               StartTime                 EndTime            JobID
------------     ---------          ------               ---------                 -------          ----------
V2VM              Restore           InProgress           4/23/2016 5:00:30 PM                        cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
```

Utilisez la cmdlet [Wait-AzRecoveryServicesBackupJob](/powershell/module/az.recoveryservices/wait-azrecoveryservicesbackupjob) pour attendre que la tâche de restauration soit terminée.

```powershell
Wait-AzRecoveryServicesBackupJob -Job $restorejob -Timeout 43200
```

Une fois le travail de restauration terminé, utilisez la cmdlet [Get-AzRecoveryServicesBackupJobDetails](/powershell/module/az.recoveryservices/wait-azrecoveryservicesbackupjob) pour obtenir les détails du travail de restauration. La propriété JobDetails contient les informations nécessaires pour régénérer la machine virtuelle.

```powershell
$restorejob = Get-AzRecoveryServicesBackupJob -Job $restorejob -VaultId $targetVault.ID
$details = Get-AzRecoveryServicesBackupJobDetails -Job $restorejob -VaultId $targetVault.ID
```

Une fois que vous avez restauré les disques, accédez à la section suivante pour créer la machine virtuelle.

## <a name="replace-disks-in-azure-vm"></a>Remplacer des disques sur une machine virtuelle Azure

Pour remplacer les disques et les informations de configuration, procédez comme suit :

* Étape 1 : [Restaurez les disques](backup-azure-vms-automation.md#restore-the-disks)
* Étape 2 : [Détachez un disque de données avec PowerShell](../virtual-machines/windows/detach-disk.md#detach-a-data-disk-using-powershell)
* Étape 3 : [Attachez un disque de données à une machine virtuelle Windows avec PowerShell](../virtual-machines/windows/attach-disk-ps.md)

## <a name="create-a-vm-from-restored-disks"></a>Créer une machine virtuelle à partir de disques restaurés

Après avoir restauré les disques, utilisez les étapes ci-après pour créer et configurer la machine virtuelle à partir du disque.

> [!NOTE]
>
> 1. Le module AzureAz 3.0.0 (ou version ultérieure) est requis. <br>
> 2. Pour créer des machines virtuelles chiffrées à partir de disques restaurés, votre rôle Azure doit avoir l’autorisation d’effectuer l’action **Microsoft.KeyVault/vaults/deploy/action**. Si votre rôle ne dispose pas de cette autorisation, créez un rôle personnalisé avec cette action. Pour plus d’informations, consultez [Rôles Azure personnalisés](../role-based-access-control/custom-roles.md). <br>
> 3. Après avoir restauré les disques, vous pouvez obtenir un modèle de déploiement que vous pouvez utiliser directement pour créer une machine virtuelle. Vous n’avez pas besoin de différentes applets de commande PowerShell pour créer des machines virtuelles managées/non managées qui sont chiffrées/non chiffrées.<br>
> <br>

### <a name="create-a-vm-using-the-deployment-template"></a>Créer une machine virtuelle à l’aide du modèle de déploiement

Les détails du travail obtenu donnent l’URI du modèle qui peut être interrogé et déployé.

```powershell
   $properties = $details.properties
   $storageAccountName = $properties["Target Storage Account Name"]
   $containerName = $properties["Config Blob Container Name"]
   $templateBlobURI = $properties["Template Blob Uri"]
```

Le modèle n’est pas directement accessible, car il se trouve sous le compte de stockage d’un client et le conteneur donné. Nous avons besoin de l’URL complète (avec un jeton SAS temporaire) pour accéder à ce modèle.

1. Tout d’abord, extrayez le nom du modèle à partir de templateBlobURI. Le format est indiqué ci-dessous. Vous pouvez utiliser l’opération de fractionnement dans PowerShell pour extraire le nom du modèle final de cette URL.

    ```http
    https://<storageAccountName.blob.core.windows.net>/<containerName>/<templateName>
    ```

2. L’URL complète peut ensuite être générée comme expliqué [ici](../azure-resource-manager/templates/secure-template-with-sas-token.md?tabs=azure-powershell#provide-sas-token-during-deployment).

    ```powershell
    Set-AzCurrentStorageAccount -Name $storageAccountName -ResourceGroupName <StorageAccount RG name>
    $templateBlobFullURI = New-AzStorageBlobSASToken -Container $containerName -Blob <templateName> -Permission r -FullUri
    ```

3. Déployez le modèle pour créer une machine virtuelle, comme expliqué [ici](../azure-resource-manager/templates/deploy-powershell.md).

    ```powershell
    New-AzResourceGroupDeployment -Name ExampleDeployment ResourceGroupName ExampleResourceGroup -TemplateUri $templateBlobFullURI -storageAccountType Standard_GRS
    ```

### <a name="create-a-vm-using-the-config-file"></a>Créer une machine virtuelle à l’aide du fichier de configuration

La section suivante liste les étapes nécessaires pour créer une machine virtuelle à l’aide du fichier « VMConfig ».

> [!NOTE]
> Il est vivement recommandé d’utiliser le modèle de déploiement décrit ci-dessus pour créer une machine virtuelle. Cette section (points 1 à 6) sera bientôt dépréciée.

1. Interrogez les propriétés des disques restaurés pour obtenir les détails du travail.

   ```powershell
   $properties = $details.properties
   $storageAccountName = $properties["Target Storage Account Name"]
   $containerName = $properties["Config Blob Container Name"]
   $configBlobName = $properties["Config Blob Name"]
   ```

2. Définissez le contexte de stockage Azure et restaurez le fichier de configuration JSON.

   ```powershell
   Set-AzCurrentStorageAccount -Name $storageaccountname -ResourceGroupName "testvault"
   $destination_path = "C:\vmconfig.json"
   Get-AzStorageBlobContent -Container $containerName -Blob $configBlobName -Destination $destination_path
   $obj = ((Get-Content -Path $destination_path -Raw -Encoding Unicode)).TrimEnd([char]0x00) | ConvertFrom-Json
   ```

3. Utilisez le fichier de configuration JSON pour créer la configuration de la machine virtuelle.

   ```powershell
   $vm = New-AzVMConfig -VMSize $obj.'properties.hardwareProfile'.vmSize -VMName "testrestore"
   ```

4. Attachez le disque du système d’exploitation et les disques de données. Cette étape fournit des exemples pour différentes configurations de machines virtuelles managées et chiffrées. Utilisez l’exemple qui correspond le mieux à votre configuration de machine virtuelle.

    * **Machines virtuelles non managées et non chiffrées** : utilisez l’exemple suivant pour une machine virtuelle non managée et non chiffrée.

    ```powershell
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.StorageProfile'.osDisk.vhd.Uri -CreateOption "Attach"
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.StorageProfile'.OsDisk.OsType
        foreach($dd in $obj.'properties.StorageProfile'.DataDisks)
        {
            $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Machines virtuelles non managées et chiffrées (à l’aide de clés BEK uniquement)** avec Azure AD : pour les machines virtuelles non managées et chiffrées (à l’aide de clés BEK uniquement) avec Azure AD, vous devez restaurer le secret dans le coffre de clés avant d’attacher les disques. Pour plus d’informations, consultez [Restaurer une machine virtuelle chiffrée à partir d’un point de récupération Sauvegarde Azure](backup-azure-restore-key-secret.md). L’exemple suivant montre comment attacher des disques de système d’exploitation et de données pour les machines virtuelles chiffrées. Au moment de définir le disque du système d’exploitation, vérifiez que le type de système d’exploitation approprié est mentionné.

    ```powershell
        $dekUrl = "https://ContosoKeyVault.vault.azure.net:443/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
        $dekUrl = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.storageProfile'.osDisk.vhd.uri -DiskEncryptionKeyUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -CreateOption "Attach" -Windows/Linux
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.storageProfile'.osDisk.osType
        foreach($dd in $obj.'properties.storageProfile'.dataDisks)
        {
        $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Machines virtuelles non managées et chiffrées (à l’aide de clés BEK et KEK)** avec Azure AD : pour les machines virtuelles non managées et chiffrées (à l’aide de clés BEK et KEK) avec Azure AD, vous devez restaurer la clé et le secret dans le coffre de clés avant d’attacher les disques. Pour plus d’informations, consultez [Restaurer une machine virtuelle chiffrée à partir d’un point de récupération Sauvegarde Azure](backup-azure-restore-key-secret.md). L’exemple suivant montre comment attacher des disques de système d’exploitation et de données pour les machines virtuelles chiffrées.

    ```powershell
        $dekUrl = "https://ContosoKeyVault.vault.azure.net:443/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
        $kekUrl = "https://ContosoKeyVault.vault.azure.net:443/keys/ContosoKey007/x9xxx00000x0000x9b9949999xx0x006"
        $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.storageProfile'.osDisk.vhd.uri -DiskEncryptionKeyUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -CreateOption "Attach" -Windows
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.storageProfile'.osDisk.osType
        foreach($dd in $obj.'properties.storageProfile'.dataDisks)
        {
        $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Machines virtuelles non managées et chiffrées (à l’aide de clés BEK uniquement) sans Azure AD** : pour les machines virtuelles non managées et chiffrées (à l’aide de clés BEK uniquement) sans Azure AD, si **le secret et le coffre de clés sources ne sont pas disponibles**, restaurez les secrets dans le coffre de clés en appliquant la procédure décrite dans [Restaurer une machine virtuelle non chiffrée à partir d’un point de récupération Sauvegarde Azure](backup-azure-restore-key-secret.md). Ensuite, exécutez les scripts suivants pour définir les détails du chiffrement sur l’objet blob de système d’exploitation restauré (cette étape n’est pas nécessaire pour un objet blob de données). $dekurl peut être extrait du coffre de clés restauré.

    Le script suivant doit être exécuté uniquement quand le secret/keyVault source n’est pas disponible.

    ```powershell
        $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
        $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
        $encSetting = "{""encryptionEnabled"":true,""encryptionSettings"":[{""diskEncryptionKey"":{""sourceVault"":{""id"":""$keyVaultId""},""secretUrl"":""$dekUrl""}}]}"
        $osBlobName = $obj.'properties.StorageProfile'.osDisk.name + ".vhd"
        $osBlob = Get-AzStorageBlob -Container $containerName -Blob $osBlobName
        $osBlob.ICloudBlob.Metadata["DiskEncryptionSettings"] = $encSetting
        $osBlob.ICloudBlob.SetMetadata()
    ```

    Une fois que les **secrets sont disponibles** et que les détails de chiffrement ont été définis sur l’objet blob de système d’exploitation, attachez les disques en utilisant le script ci-dessous.

    Si les secrets/keyVault sources sont déjà disponibles, vous n’êtes pas obligé d’exécuter le script ci-dessus.

    ```powershell
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.StorageProfile'.osDisk.vhd.Uri -CreateOption "Attach"
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.StorageProfile'.OsDisk.OsType
        foreach($dd in $obj.'properties.StorageProfile'.DataDisks)
        {
        $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Machines virtuelles non managées et chiffrées (à l’aide de clés BEK et KEK) sans Azure AD** : pour les machines virtuelles non managées et chiffrées (à l’aide de clés BEK et KEK) sans Azure AD, si le **secret/clé/coffre de clés sources ne sont pas disponibles**, restaurez la clé et les secrets dans le coffre de clés en appliquant la procédure décrite dans [Restaurer une machine virtuelle non chiffrée à partir d’un point de récupération Sauvegarde Azure](backup-azure-restore-key-secret.md). Ensuite, exécutez les scripts suivants pour définir les détails du chiffrement sur l’objet blob de système d’exploitation restauré (cette étape n’est pas nécessaire pour un objet blob de données). $dekurl et kekurl peuvent être extraits du coffre de clés restauré.

    Le script ci-dessous doit être exécuté uniquement quand le secret/keyVault ou la clé source ne sont pas disponibles.

    ```powershell
        $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
        $kekUrl = "https://ContosoKeyVault.vault.azure.net/keys/ContosoKey007/x9xxx00000x0000x9b9949999xx0x006"
        $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
        $encSetting = "{""encryptionEnabled"":true,""encryptionSettings"":[{""diskEncryptionKey"":{""sourceVault"":{""id"":""$keyVaultId""},""secretUrl"":""$dekUrl""},""keyEncryptionKey"":{""sourceVault"":{""id"":""$keyVaultId""},""keyUrl"":""$kekUrl""}}]}"
        $osBlobName = $obj.'properties.StorageProfile'.osDisk.name + ".vhd"
        $osBlob = Get-AzStorageBlob -Container $containerName -Blob $osBlobName
        $osBlob.ICloudBlob.Metadata["DiskEncryptionSettings"] = $encSetting
        $osBlob.ICloudBlob.SetMetadata()
    ```

    Une fois que **les secrets et la clé sont disponibles** et que les détails de chiffrement ont été définis sur l’objet blob de système d’exploitation, attachez les disques en utilisant le script ci-dessous.

    Si les secrets/coffre de clés/clé sources sont disponibles, vous n’êtes pas obligé d’exécuter le script ci-dessus.

    ```powershell
        Set-AzVMOSDisk -VM $vm -Name "osdisk" -VhdUri $obj.'properties.StorageProfile'.osDisk.vhd.Uri -CreateOption "Attach"
        $vm.StorageProfile.OsDisk.OsType = $obj.'properties.StorageProfile'.OsDisk.OsType
        foreach($dd in $obj.'properties.StorageProfile'.DataDisks)
        {
        $vm = Add-AzVMDataDisk -VM $vm -Name "datadisk1" -VhdUri $dd.vhd.Uri -DiskSizeInGB 127 -Lun $dd.Lun -CreateOption "Attach"
        }
    ```

    * **Machines virtuelles managées et non chiffrées** : utilisez l’exemple suivant pour une machine virtuelle managée mais pas chiffrée. Pour plus d’informations, consultez [Attacher un disque de données à une machine virtuelle Windows à l’aide de PowerShell](../virtual-machines/windows/attach-disk-ps.md).

    * **Machines virtuelles managées et chiffrées (à l’aide de clés BEK uniquement)** avec Azure AD : pour les machines virtuelles managées et chiffrées (à l’aide de clés BEK uniquement) avec Azure AD, attachez les disques managés restaurés. Pour plus d’informations, consultez [Attacher un disque de données à une machine virtuelle Windows à l’aide de PowerShell](../virtual-machines/windows/attach-disk-ps.md).

    * **Machines virtuelles managées et chiffrées (à l’aide de clés BEK et KEK)** avec Azure AD : pour les machines virtuelles managées et chiffrées (à l’aide de clés BEK et KEK) avec Azure AD, attachez les disques managés restaurés. Pour plus d’informations, consultez [Attacher un disque de données à une machine virtuelle Windows à l’aide de PowerShell](../virtual-machines/windows/attach-disk-ps.md).

    * **Machines virtuelles managées et chiffrées (à l’aide de clés BEK uniquement) sans Azure AD** : pour les machines virtuelles managées et chiffrées (à l’aide de clés BEK uniquement) sans Azure AD, si **le secret et le coffre de clés sources ne sont pas disponibles**, restaurez les secrets dans le coffre de clés en appliquant la procédure décrite dans [Restaurer une machine virtuelle non chiffrée à partir d’un point de récupération Sauvegarde Azure](backup-azure-restore-key-secret.md). Ensuite, exécutez les scripts suivants pour définir les détails du chiffrement sur le disque de système d’exploitation restauré (cette étape n’est pas nécessaire pour un disque de données). $dekurl peut être extrait du coffre de clés restauré.

    Le script ci-dessous doit être exécuté uniquement quand le secret/coffre de clés source n’est pas disponible.  

    ```powershell
    $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
    $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
    $diskupdateconfig = New-AzDiskUpdateConfig -EncryptionSettingsEnabled $true
    $encryptionSettingsElement = New-Object Microsoft.Azure.Management.Compute.Models.EncryptionSettingsElement
    $encryptionSettingsElement.DiskEncryptionKey = New-Object Microsoft.Azure.Management.Compute.Models.KeyVaultAndSecretReference
    $encryptionSettingsElement.DiskEncryptionKey.SourceVault = New-Object Microsoft.Azure.Management.Compute.Models.SourceVault
    $encryptionSettingsElement.DiskEncryptionKey.SourceVault.Id = $keyVaultId
    $encryptionSettingsElement.DiskEncryptionKey.SecretUrl = $dekUrl
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettings = New-Object System.Collections.Generic.List[Microsoft.Azure.Management.Compute.Models.EncryptionSettingsElement]
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettings.Add($encryptionSettingsElement)
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettingsVersion = "1.1"
    Update-AzDisk -ResourceGroupName "testvault" -DiskName $obj.'properties.StorageProfile'.osDisk.name -DiskUpdate $diskupdateconfig
    ```

    Une fois que les secrets sont disponibles et que les détails de chiffrement ont été définis sur le disque du système d’exploitation, pour attacher les disques managés restaurés, consultez [Attacher un disque de données à une machine virtuelle Windows dans Azure à l’aide de PowerShell](../virtual-machines/windows/attach-disk-ps.md).

    * **Machines virtuelles managées et chiffrées (à l’aide de clés BEK et KEK) sans Azure AD** : pour les machines virtuelles managées et chiffrées (à l’aide de clés BEK et KEK) sans Azure AD, si le **secret/la clé/le coffre de clés sources ne sont pas disponibles**, restaurez la clé et les secrets dans le coffre de clés en appliquant la procédure décrite dans [Restaurer une machine virtuelle non chiffrée à partir d’un point de récupération Sauvegarde Azure](backup-azure-restore-key-secret.md). Ensuite, exécutez les scripts suivants pour définir les détails du chiffrement sur le disque de système d’exploitation restauré (cette étape n’est pas nécessaire pour les disques de données). $dekurl et kekurl peuvent être extraits du coffre de clés restauré.

    Le script suivant doit être exécuté uniquement quand le secret/coffre de clés ou la clé source ne sont pas disponibles.

    ```powershell
    $dekUrl = "https://ContosoKeyVault.vault.azure.net/secrets/ContosoSecret007/xx000000xx0849999f3xx30000003163"
    $kekUrl = "https://ContosoKeyVault.vault.azure.net/keys/ContosoKey007/x9xxx00000x0000x9b9949999xx0x006"
    $keyVaultId = "/subscriptions/abcdedf007-4xyz-1a2b-0000-12a2b345675c/resourceGroups/ContosoRG108/providers/Microsoft.KeyVault/vaults/ContosoKeyVault"
    $diskupdateconfig = New-AzDiskUpdateConfig -EncryptionSettingsEnabled $true
    $encryptionSettingsElement = New-Object Microsoft.Azure.Management.Compute.Models.EncryptionSettingsElement
    $encryptionSettingsElement.DiskEncryptionKey = New-Object Microsoft.Azure.Management.Compute.Models.KeyVaultAndSecretReference
    $encryptionSettingsElement.DiskEncryptionKey.SourceVault = New-Object Microsoft.Azure.Management.Compute.Models.SourceVault
    $encryptionSettingsElement.DiskEncryptionKey.SourceVault.Id = $keyVaultId
    $encryptionSettingsElement.DiskEncryptionKey.SecretUrl = $dekUrl
    $encryptionSettingsElement.KeyEncryptionKey = New-Object Microsoft.Azure.Management.Compute.Models.KeyVaultAndKeyReference
    $encryptionSettingsElement.KeyEncryptionKey.SourceVault = New-Object Microsoft.Azure.Management.Compute.Models.SourceVault
    $encryptionSettingsElement.KeyEncryptionKey.SourceVault.Id = $keyVaultId
    $encryptionSettingsElement.KeyEncryptionKey.KeyUrl = $kekUrl
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettings = New-Object System.Collections.Generic.List[Microsoft.Azure.Management.Compute.Models.EncryptionSettingsElement]
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettings.Add($encryptionSettingsElement)
    $diskupdateconfig.EncryptionSettingsCollection.EncryptionSettingsVersion = "1.1"
    Update-AzDisk -ResourceGroupName "testvault" -DiskName $obj.'properties.StorageProfile'.osDisk.name -DiskUpdate $diskupdateconfig
    ```

    Une fois que les secrets et la clé sont disponibles et que les détails de chiffrement ont été définis sur le disque du système d’exploitation, pour attacher les disques managés restaurés, consultez [Attacher un disque de données à une machine virtuelle Windows dans Azure à l’aide de PowerShell](../virtual-machines/windows/attach-disk-ps.md).

5. Définissez les paramètres réseau.

    ```powershell
    $nicName="p1234"
    $pip = New-AzPublicIpAddress -Name $nicName -ResourceGroupName "test" -Location "WestUS" -AllocationMethod Dynamic
    $virtualNetwork = New-AzVirtualNetwork -ResourceGroupName "test" -Location "WestUS" -Name "testvNET" -AddressPrefix 10.0.0.0/16
    $virtualNetwork | Set-AzVirtualNetwork
    $vnet = Get-AzVirtualNetwork -Name "testvNET" -ResourceGroupName "test"
    $subnetindex=0
    $nic = New-AzNetworkInterface -Name $nicName -ResourceGroupName "test" -Location "WestUS" -SubnetId $vnet.Subnets[$subnetindex].Id -PublicIpAddressId $pip.Id
    $vm=Add-AzVMNetworkInterface -VM $vm -Id $nic.Id
    ```

6. Créez la machine virtuelle.

    ```powershell  
    New-AzVM -ResourceGroupName "test" -Location "WestUS" -VM $vm
    ```

7. Envoyez (Push) l’extension ADE.
   Si les extensions ADE ne sont pas envoyées, les disques de données sont marqués comme non chiffrés. Les étapes ci-dessous doivent donc être impérativement exécutées :

   * **Pour les machines virtuelles avec Azure AD** : utilisez la commande suivante pour activer manuellement le chiffrement des disques de données.  

     **Clés BEK uniquement**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm.Name -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -VolumeType Data
      ```

     **Clés BEK et KEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm.Name -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId  -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -VolumeType Data
      ```

   * **Pour les machines virtuelles sans Azure AD** : utilisez la commande suivante pour activer manuellement le chiffrement des disques de données.

     Si, pendant l’exécution de la commande, vous êtes invité à fournir l’AADClientID, vous devez mettre à jour votre instance Azure PowerShell.

     **Clés BEK uniquement**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm.Name -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -SkipVmBackup -VolumeType "All"
      ```

      **Clés BEK et KEK**

      ```powershell  
      Set-AzVMDiskEncryptionExtension -ResourceGroupName $RG -VMName $vm.Name -DiskEncryptionKeyVaultUrl $dekUrl -DiskEncryptionKeyVaultId $keyVaultId -KeyEncryptionKeyUrl $kekUrl -KeyEncryptionKeyVaultId $keyVaultId -SkipVmBackup -VolumeType "All"
      ```

> [!NOTE]
> Veillez à supprimer manuellement les fichiers JASON créés dans le cadre du processus de restauration de disque des machines virtuelles chiffrées.

## <a name="restore-files-from-an-azure-vm-backup"></a>Restaurer des fichiers à partir d’une sauvegarde de machine virtuelle Azure

En plus de la restauration des disques, vous pouvez également restaurer des fichiers individuels à partir d’une sauvegarde de machine virtuelle Azure. La fonctionnalité de restauration des fichiers permet d’accéder à tous les fichiers dans un point de récupération. Gérez les fichiers via l’Explorateur de fichiers comme vous le feriez pour des fichiers normaux.

Les étapes élémentaires pour restaurer un fichier à partir d’une sauvegarde de machine virtuelle Azure sont les suivantes :

* Sélection de la machine virtuelle
* Choisir un point de récupération
* Monter les disques du point de récupération
* Copier les fichiers nécessaires
* Démonter le disque

### <a name="select-the-vm-when-restoring-the-vm"></a>Sélectionner la machine virtuelle (lors de la restauration de la machine virtuelle)

Pour obtenir l’objet PowerShell permettant d’identifier l’élément à restaurer, vous devez commencer au niveau du conteneur dans le coffre et descendre dans la hiérarchie d’objets. Pour sélectionner le conteneur qui représente la machine virtuelle, utilisez l’applet de commande [Get-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer) et dirigez-la vers l’applet de commande [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem).

```powershell
$namedContainer = Get-AzRecoveryServicesBackupContainer  -ContainerType "AzureVM" -Status "Registered" -FriendlyName "V2VM" -VaultId $targetVault.ID
$backupitem = Get-AzRecoveryServicesBackupItem -Container $namedContainer  -WorkloadType "AzureVM" -VaultId $targetVault.ID
```

### <a name="choose-a-recovery-point-when-restoring-the-vm"></a>Choisir un point de récupération (lors de la restauration de la machine virtuelle)

Utilisez la cmdlet [Get-AzRecoveryServicesBackupRecoveryPoint](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint) afin de répertorier tous les points de récupération correspondant à l'élément de sauvegarde. Ensuite, choisissez le point de récupération à restaurer. Si vous ne savez pas quel point de récupération utiliser, nous vous conseillons de choisir le point RecoveryPointType = AppConsistent le plus récent dans la liste.

Dans le script suivant, la variable **$rp** est un tableau de points de récupération des 7 derniers jours pour l’élément de sauvegarde sélectionné. Le tableau est trié dans l’ordre chronologique inverse, le point de récupération le plus récent détenant l’index 0. Utilisez l'indexation de tableau PowerShell standard pour sélectionner le point de récupération. Dans l’exemple, $rp[0] sélectionne le dernier point de récupération.

```powershell
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $backupitem -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime() -VaultId $targetVault.ID
$rp[0]
```

Le résultat ressemble à l’exemple suivant :

```output
RecoveryPointAdditionalInfo :
SourceVMStorageType         : NormalStorage
Name                        : 15260861925810
ItemName                    : VM;iaasvmcontainer;RGName1;V2VM
RecoveryPointId             : /subscriptions/XX/resourceGroups/ RGName1/providers/Microsoft.RecoveryServices/vaults/testvault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainer;RGName1;V2VM/protectedItems/VM;iaasvmcontainer; RGName1;V2VM/recoveryPoints/15260861925810
RecoveryPointType           : AppConsistent
RecoveryPointTime           : 4/23/2016 5:02:04 PM
WorkloadType                : AzureVM
ContainerName               : IaasVMContainer;iaasvmcontainer; RGName1;V2VM
ContainerType               : AzureVM
BackupManagementType        : AzureVM
```

### <a name="mount-the-disks-of-recovery-point"></a>Monter les disques du point de récupération

Utilisez la cmdlet [Get-AzRecoveryServicesBackupRPMountScript](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprpmountscript) pour le montage de tous les disques du point de récupération par le script.

> [!NOTE]
> Les disques sont montés en tant que disques iSCSI attachés à la machine sur laquelle le script est exécuté. Le montage a lieu immédiatement et vous évitez les frais.
>
>

```powershell
Get-AzRecoveryServicesBackupRPMountScript -RecoveryPoint $rp[0] -VaultId $targetVault.ID
```

Le résultat ressemble à l’exemple suivant :

```output
OsType  Password        Filename
------  --------        --------
Windows e3632984e51f496 V2VM_wus2_8287309959960546283_451516692429_cbd6061f7fc543c489f1974d33659fed07a6e0c2e08740.exe
```

Exécutez le script sur la machine où vous souhaitez récupérer les fichiers. Pour exécuter le script, vous devez entrer le mot de passe fourni. Lorsque les disques sont attachés, utilisez l’Explorateur de fichiers Windows pour parcourir les nouveaux volumes et fichiers. Pour plus d’informations, consultez l’article [Récupérer des fichiers à partir d’une sauvegarde de machine virtuelle Azure](backup-azure-restore-files-from-vm.md) relatif à la sauvegarde.

### <a name="unmount-the-disks"></a>Démonter les disques

Lorsque les fichiers nécessaires à l’opération sont copiés, utilisez [Disable-AzRecoveryServicesBackupRPMountScript](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackuprpmountscript) pour démonter les disques. Veillez à démonter les disques afin que l’accès aux fichiers du point de récupération soit supprimé.

```powershell
Disable-AzRecoveryServicesBackupRPMountScript -RecoveryPoint $rp[0] -VaultId $targetVault.ID
```

## <a name="next-steps"></a>Étapes suivantes

Si vous préférez utiliser PowerShell pour gérer vos ressources Azure, consultez l’article PowerShell [Déployer et gérer une sauvegarde pour Windows Server](backup-client-automation.md). Si vous gérez des sauvegardes DPM, consultez l’article [Déployer et gérer la sauvegarde pour DPM](backup-dpm-automation.md).
