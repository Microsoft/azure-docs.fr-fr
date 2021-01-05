---
title: Sauvegarde et restauration sélectives de disques pour les machines virtuelles Azure
description: Dans cet article, découvrez la sauvegarde et la restauration sélectives de disques à travers la solution de sauvegarde de machine virtuelle Azure.
ms.topic: conceptual
ms.date: 07/17/2020
ms.custom: references_regions , devx-track-azurecli
ms.openlocfilehash: 95104f231e7b4d4d2135ac3c5dde27512d465775
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746988"
---
# <a name="selective-disk-backup-and-restore-for-azure-virtual-machines"></a>Sauvegarde et restauration sélectives de disques pour les machines virtuelles Azure

Sauvegarde Azure prend en charge la sauvegarde de tous les disques (système d’exploitation et données) d’une machine virtuelle à travers la solution de sauvegarde de machine virtuelle. Désormais, en utilisant la fonctionnalité de sauvegarde et restauration sélectives de disques, vous pouvez sauvegarder un sous-ensemble de disques de données d’une machine virtuelle. Cela offre une solution efficace et économique pour vos besoins en sauvegarde et restauration. Chaque point de récupération contient uniquement les disques inclus dans l’opération de sauvegarde. Cela vous permet de disposer d’un sous-ensemble de disques restaurés à partir du point de récupération donné pendant l’opération de restauration. Cela vaut pour les restaurations effectuées à partir de captures instantanées et du coffre.

## <a name="scenarios"></a>Scénarios

Cette solution s’avère particulièrement utile dans les scénarios suivants :

1. Si vous avez des données critiques à sauvegarder sur un seul disque ou sur un sous-ensemble de disques et que vous ne voulez pas sauvegarder les autres disques attachés à une machine virtuelle pour limiter les coûts de stockage de sauvegarde.
2. Si vous disposez d’autres solutions de sauvegarde pour une partie de votre machine virtuelle ou de vos données. Par exemple, vous sauvegardez vos bases de données ou vos données avec une autre solution de sauvegarde de charge de travail et vous souhaitez utiliser la sauvegarde au niveau de la machine virtuelle Azure pour le reste des données ou des disques afin de générer un système efficace et robuste utilisant les meilleures fonctionnalités disponibles.

En utilisant PowerShell ou Azure CLI, vous pouvez configurer la sauvegarde sélective de disques de la machine virtuelle Azure.  En utilisant un script, vous pouvez inclure ou exclure des disques de données à partir de leur numéro d’unité logique (LUN).  Pour l’heure, la possibilité de configurer la sauvegarde sélective de disques via le portail Azure se limite à l’option **Sauvegarder le disque du système d’exploitation uniquement** . Vous pouvez donc configurer la sauvegarde de votre machine virtuelle Azure avec le disque du système d’exploitation et exclure tous les disques de données qui y sont attachés.

>[!NOTE]
> Le disque du système d’exploitation est ajouté par défaut à la sauvegarde de la machine virtuelle et ne peut pas être exclu.

## <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure

Veillez à utiliser Az CLI version 2.0.80 ou supérieure. Vous pouvez obtenir la version de l’interface CLI avec cette commande :

```azurecli
az --version
```

Connectez-vous à l’ID d’abonnement dans lequel se trouvent le coffre Recovery Services et la machine virtuelle :

```azurecli
az account set -s {subscriptionID}
```

>[!NOTE]
>Seul le nom **resourcegroup** (et non l’objet) correspondant au coffre est nécessaire dans chaque commande ci-dessous.

### <a name="configure-backup-with-azure-cli"></a>Configurer la sauvegarde avec Azure CLI

Pendant l’opération de configuration de la protection, vous devez spécifier le paramètre de liste de disques avec un paramètre d’ **inclusion** / **exclusion** , en indiquant les numéros LUN des disques à inclure ou à exclure dans la sauvegarde.

```azurecli
az backup protection enable-for-vm --resource-group {resourcegroup} --vault-name {vaultname} --vm {vmname} --policy-name {policyname} --disk-list-setting include --diskslist {LUN number(s) separated by space}
```

```azurecli
az backup protection enable-for-vm --resource-group {resourcegroup} --vault-name {vaultname} --vm {vmname} --policy-name {policyname} --disk-list-setting exclude --diskslist 0 1
```

Si la machine virtuelle n’est pas dans le même groupe de ressources que le coffre, **ResourceGroup** fait référence au groupe de ressources dans lequel le coffre a été créé. Fournissez l’ID et non le nom de la machine virtuelle comme indiqué ci-dessous.

```azurecli
az backup protection enable-for-vm  --resource-group {ResourceGroup} --vault-name {vaultname} --vm $(az vm show -g VMResourceGroup -n MyVm --query id --output tsv) --policy-name {policyname} --disk-list-setting include --diskslist {LUN number(s) separated by space}
```

### <a name="modify-protection-for-already-backed-up-vms-with-azure-cli"></a>Modifier la protection pour les machines virtuelles déjà sauvegardées avec Azure CLI

```azurecli
az backup protection update-for-vm --resource-group {resourcegroup} --vault-name {vaultname} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM --disk-list-setting exclude --diskslist {LUN number(s) separated by space}
```

### <a name="backup-only-os-disk-during-configure-backup-with-azure-cli"></a>Sauvegarder uniquement le disque du système d’exploitation pendant la configuration de la sauvegarde avec Azure CLI

```azurecli
az backup protection enable-for-vm --resource-group {resourcegroup} --vault-name {vaultname} --vm {vmname} --policy-name {policyname} --exclude-all-data-disks
```

### <a name="backup-only-os-disk-during-modify-protection-with-azure-cli"></a>Sauvegarder uniquement le disque du système d’exploitation pendant la modification de la protection avec Azure CLI

```azurecli
az backup protection update-for-vm --resource-group {resourcegroup} --vault-name {vaultname} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM --exclude-all-data-disks
```

### <a name="restore-disks-with-azure-cli"></a>Restaurer les disques avec Azure CLI

```azurecli
az backup restore restore-disks --resource-group {resourcegroup} --vault-name {vaultname} -c {vmname} -i {vmname} -r {restorepoint} --target-resource-group {targetresourcegroup} --storage-account {storageaccountname} --diskslist {LUN number of the disk(s) to be restored}
```

### <a name="restore-only-os-disk-with-azure-cli"></a>Restaurer uniquement le disque du système d’exploitation avec Azure CLI

```azurecli
az backup restore restore-disks --resource-group {resourcegroup} --vault-name {vaultname} -c {vmname} -i {vmname} -r {restorepoint} } --target-resource-group {targetresourcegroup} --storage-account {storageaccountname} --restore-only-osdisk
```

### <a name="get-protected-item-to-get-disk-exclusion-details-with-azure-cli"></a>Obtenir l’élément protégé pour obtenir les détails de l’exclusion de disques avec Azure CLI

```azurecli
az backup item show -c {vmname} -n {vmname} --vault-name {vaultname} --resource-group {resourcegroup} --backup-management-type AzureIaasVM
```

Un paramètre **diskExclusionProperties** supplémentaire est ajouté à l’élément protégé, comme indiqué ci-dessous :

```azurecli
"extendedProperties": {
      "diskExclusionProperties": {
        "diskLunList": [
          0,
          1
        ],
        "isInclusionList": true
      }
```

### <a name="get-backup-job-with-azure-cli"></a>Obtenir un travail de sauvegarde avec Azure CLI

```azurecli
az backup job show --vault-name {vaultname} --resource-group {resourcegroup} -n {BackupJobID}
```

Cette commande permet d’obtenir les détails des disques sauvegardés et des disques exclus, comme indiqué ci-dessous :

```output
   "Backed-up disk(s)": "diskextest_OsDisk_1_170808a95d214428bad92efeecae626b; diskextest_DataDisk_0; diskextest_DataDisk_1",  "Backup Size": "0 MB",
   "Excluded disk(s)": "diskextest_DataDisk_2",
```

### <a name="list-recovery-points-with-azure-cli"></a>Lister les points de récupération avec Azure CLI

```azurecli
az backup recoverypoint list --vault-name {vaultname} --resource-group {resourcegroup} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM
```

Cela renseigne sur le nombre de disques attachés et sauvegardés dans la machine virtuelle.

```azurecli
      "recoveryPointDiskConfiguration": {
        "excludedDiskList": null,
        "includedDiskList": null,
        "numberOfDisksAttachedToVm": 4,
        "numberOfDisksIncludedInBackup": 3
};
```

### <a name="get-recovery-point-with-azure-cli"></a>Obtenir le point de récupération avec Azure CLI

```azurecli
az backup recoverypoint show --vault-name {vaultname} --resource-group {resourcegroup} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM -n {recoverypointID}
```

Chaque point de récupération contient des informations sur les disques inclus et exclus :

```azurecli
  "recoveryPointDiskConfiguration": {
      "excludedDiskList": [
        {
          "lun": 2,
          "name": "diskextest_DataDisk_2"
        }
      ],
      "includedDiskList": [
        {
          "lun": -1,
          "name": "diskextest_OsDisk_1_170808a95d214428bad92efeecae626b"
        },
        {
          "lun": 0,
          "name": "diskextest_DataDisk_0"
        },
        {
          "lun": 1,
          "name": "diskextest_DataDisk_1"
        }
      ],
      "numberOfDisksAttachedToVm": 4,
      "numberOfDisksIncludedInBackup": 3
```

### <a name="remove-disk-exclusion-settings-and-get-protected-item-with-azure-cli"></a>Supprimer les paramètres d’exclusion de disques et obtenir l’élément protégé avec Azure CLI

```azurecli
az backup protection update-for-vm --vault-name {vaultname} --resource-group {resourcegroup} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM --disk-list-setting resetexclusionsettings

az backup item show -c {vmname} -n {vmname} --vault-name {vaultname} --resource-group {resourcegroup} --backup-management-type AzureIaasVM
```

Quand vous exécutez ces commandes, vous voyez `"diskExclusionProperties": null`.

## <a name="using-powershell"></a>Utilisation de PowerShell

Veillez à utiliser Azure PowerShell version 3.7.0 ou supérieure.

### <a name="enable-backup-with-powershell"></a>Activer la sauvegarde avec PowerShell

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"  -InclusionDisksList[Strings] -VaultId $targetVault.ID
```

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"  -ExclusionDisksList[Strings] -VaultId $targetVault.ID
```

### <a name="backup-only-os-disk-during-configure-backup-with-powershell"></a>Sauvegarder uniquement le disque du système d’exploitation pendant la configuration de la sauvegarde avec PowerShell

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"  -ExcludeAllDataDisks -VaultId $targetVault.ID
```

### <a name="get-backup-item-object-to-be-passed-in-modify-protection-with-powershell"></a>Obtenir l’objet d’élément de sauvegarde à transmettre pendant la modification de la protection avec PowerShell

```azurepowershell
$item= Get-AzRecoveryServicesBackupItem -BackupManagementType "AzureVM" -WorkloadType "AzureVM" -VaultId $Vault.ID -FriendlyName "V2VM"
```

Vous devez transmettre l’objet **$item** obtenu ci-dessus au paramètre **–Item** dans les applets de commande suivantes.

### <a name="modify-protection-for-already-backed-up-vms-with-powershell"></a>Modifier la protection pour les machines virtuelles déjà sauvegardées avec PowerShell

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Item $item -InclusionDisksList[Strings] -VaultId $targetVault.ID
```

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Item $item -ExclusionDisksList[Strings] -VaultId $targetVault.ID
```

### <a name="backup-only-os-disk-during-modify-protection-with-powershell"></a>Sauvegarder uniquement le disque du système d’exploitation pendant la modification de la protection avec PowerShell

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Item $item  -ExcludeAllDataDisks -VaultId $targetVault.ID
```

### <a name="reset-disk-exclusion-setting-with-powershell"></a>Réinitialiser le paramètre d’exclusion de disques avec PowerShell

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Item $item -ResetExclusionSettings -VaultId $targetVault.ID
```

### <a name="restore-selective-disks-with-powershell"></a>Restaurer sélectivement des disques avec PowerShell

```azurepowershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -TargetResourceGroupName "DestRGforManagedDisks" -VaultId $targetVault.ID -RestoreDiskList [Strings]
```

### <a name="restore-only-os-disk-with-powershell"></a>Restaurer uniquement le disque du système d’exploitation avec PowerShell

```azurepowershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -TargetResourceGroupName "DestRGforManagedDisks" -VaultId $targetVault.ID -RestoreOnlyOSDisk
```

## <a name="using-the-azure-portal"></a>Utilisation du portail Azure

[!INCLUDE [backup-center.md](../../includes/backup-center.md)]

À partir du portail Azure, vous pouvez afficher les disques inclus et les disques exclus dans le volet d’informations de la sauvegarde de machine virtuelle et dans le volet d’informations du travail de sauvegarde.  Pendant la restauration, quand vous sélectionnez le point de récupération à partir duquel effectuer la restauration, vous pouvez voir les disques sauvegardés dans ce point de récupération.

Ici, vous pouvez afficher les disques inclus et les disques exclus d’une machine virtuelle sur le portail dans le volet d’informations de la sauvegarde de machine virtuelle :

![Afficher les disques inclus et les disques exclus dans le volet d’informations de la sauvegarde](./media/selective-disk-backup-restore/backup-details.png)

Ici, vous pouvez afficher les disques inclus et les disques exclus au niveau d’une sauvegarde dans le volet d’informations du travail :

![Afficher les disques inclus et les disques exclus dans le volet d’informations du travail](./media/selective-disk-backup-restore/job-details.png)

Ici, vous pouvez afficher les disques sauvegardés pendant la restauration, quand vous sélectionnez le point de récupération à partir duquel effectuer la restauration :

![Afficher les disques sauvegardés pendant la restauration](./media/selective-disk-backup-restore/during-restore.png)

La configuration de l’expérience de sauvegarde sélective de disques pour une machine virtuelle via le portail Azure se limite à l’option **Sauvegarder le disque du système d’exploitation uniquement** . Pour utiliser la sauvegarde sélective de disques sur une machine virtuelle déjà sauvegardée ou pour inclure ou exclure de manière avancée certains disques de données d’une machine virtuelle, utilisez PowerShell ou Azure CLI.

>[!NOTE]
>Si les données occupent plusieurs disques, veillez à inclure tous les disques dépendants dans la sauvegarde. Si vous ne sauvegardez pas tous les disques dépendants dans un volume, pendant la restauration, le volume constitué de disques non sauvegardés ne sera pas créé.

### <a name="backup-os-disk-only-in-the-azure-portal"></a>Sauvegarder uniquement le disque du système d’exploitation sur le portail Azure

Quand vous activez la sauvegarde à partir du portail Azure, vous pouvez choisir l’option **Sauvegarder le disque du système d’exploitation uniquement** . Vous pouvez donc configurer la sauvegarde de votre machine virtuelle Azure avec le disque du système d’exploitation et exclure tous les disques de données qui y sont attachés.

![Configurer la sauvegarde pour le disque du système d’exploitation uniquement](./media/selective-disk-backup-restore/configure-backup-operating-system-disk.png)

## <a name="using-azure-rest-api"></a>Utilisation de l’API REST Azure

Vous pouvez configurer la sauvegarde de machines virtuelles Azure avec quelques disques sélectionnés ou modifier la protection d’une machine virtuelle existante pour inclure ou exclure plusieurs disques comme indiqué [ici](backup-azure-arm-userestapi-backupazurevms.md#excluding-disks-in-azure-vm-backup).

## <a name="selective-disk-restore"></a>Restauration sélective de disques

La restauration sélective de disques est une fonctionnalité ajoutée dont vous pouvez bénéficier en activant la fonctionnalité de sauvegarde sélective de disques. Cette fonctionnalité vous permet de restaurer de façon sélective des disques à partir de tous les disques sauvegardés dans un point de récupération. Elle est plus efficace et permet de gagner du temps dans les scénarios où vous savez quels disques doivent être restaurés.

- Le disque du système d’exploitation est inclus par défaut dans la sauvegarde et la restauration de machine virtuelle et ne peut pas être exclu.
- La restauration sélective de disques est prise en charge uniquement pour les points de récupération créés après l’activation de la fonctionnalité d’exclusion de disques.
- Les sauvegardes pour lesquelles le paramètre d’exclusion de disque est défini sur **ON** prennent en charge uniquement l’option **Restauration de disque** . Les options de restauration **Restauration de machine virtuelle** ou **Remplacer l’existant** ne sont pas prises en charge dans ce cas.

![Les options de restauration de machine virtuelle et de remplacement de l’existant ne sont pas disponibles pendant l’opération de restauration](./media/selective-disk-backup-restore/options-not-available.png)

## <a name="limitations"></a>Limites

La fonctionnalité de sauvegarde sélective de disques n’est pas prise en charge pour les machines virtuelles classiques et les machines virtuelles chiffrées. Par conséquent, les machines virtuelles Azure chiffrées avec Azure Disk Encryption (ADE) en utilisant BitLocker pour le chiffrement de la machine virtuelle Windows et la fonctionnalité dm-crypt pour les machines virtuelles Linux ne sont pas prises en charge.

Les options de restauration **Créer une machine virtuelle** et **Remplacer l’existant** ne sont pas prises en charge par la machine virtuelle pour laquelle la fonctionnalité de sauvegarde sélective de disques est activée.

Actuellement, la sauvegarde de machine virtuelle Azure ne prend pas en charge les machines virtuelles auxquelles sont attachés des disques Ultra ou des disques partagés. La sauvegarde sélective de disques ne peut pas être utilisée dans de tels cas, qui excluent le disque et sauvegardent la machine virtuelle.

## <a name="billing"></a>Facturation

La sauvegarde de machine virtuelle Azure suit le modèle tarifaire existant, qui est expliqué en détail [ici](https://azure.microsoft.com/pricing/details/backup/).

Le **coût de l’instance protégée** est calculé pour le disque du système d’exploitation uniquement si vous optez pour une sauvegarde avec l’option **Disque du système d’exploitation uniquement** .  Si vous configurez la sauvegarde et sélectionnez au moins un disque de données, le coût de l’instance protégée est calculé pour tous les disques attachés à la machine virtuelle. Le **coût du stockage de sauvegarde** est calculé en fonction des disques inclus, ce qui vous permet de réaliser des économies sur le coût du stockage. Le **coût de capture instantanée** est toujours calculé pour tous les disques de la machine virtuelle (disques inclus et exclus).

Si vous avez choisi la fonctionnalité de restauration interrégion, la [tarification de la restauration interrégion](https://azure.microsoft.com/pricing/details/backup/) s’applique sur le coût du stockage de sauvegarde après l’exclusion du disque.

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="how-is-protected-instance-pi-cost-calculated-for-only-os-disk-backup-in-windows-and-linux"></a>Comment le coût de l’instance protégée est-il calculé pour la seule sauvegarde du disque du système d’exploitation dans Windows et Linux ?

Le coût de l’instance protégée est calculé en fonction de la taille réelle (utilisée) de la machine virtuelle.

- Pour Windows : Le calcul de l’espace utilisé est basé sur le lecteur qui stocke le système d’exploitation (généralement C:).
- Pour Linux : Le calcul de l’espace utilisé est basé sur l’appareil sur lequel le système de fichiers racine (/) est monté.

### <a name="i-have-configured-only-os-disk-backup-why-is-the-snapshot-happening-for-all-the-disks"></a>J’ai configuré la sauvegarde du disque du système d’exploitation uniquement, pourquoi l’instantané se produit-il pour tous les disques ?

Les fonctionnalités de sauvegarde sélective de disques vous permettent de réaliser des économies sur le coût de stockage du coffre de sauvegarde en renforçant les disques inclus qui font partie de la sauvegarde. Toutefois, l’instantané est pris pour tous les disques attachés à la machine virtuelle. Le coût de capture instantanée est donc toujours calculé pour tous les disques de la machine virtuelle (disques inclus et exclus). Pour plus d’informations, consultez [Facturation](#billing).

### <a name="i-cant-configure-backup-for-the-azure-virtual-machine-by-excluding-ultra-disk-or-shared-disks-attached-to-the-vm"></a>Je ne parviens pas à configurer la sauvegarde pour la machine virtuelle Azure en excluant le disque Ultra ou les disques partagés attachés à la machine virtuelle

La fonctionnalité de sauvegarde sélective de disques est une capacité fournie en plus de la solution de sauvegarde de machine virtuelle Azure. Actuellement, la sauvegarde de machine virtuelle Azure ne prend pas en charge les machines virtuelles auxquelles sont attachés un disque Ultra ou un disque partagé.

## <a name="next-steps"></a>Étapes suivantes

- [Tableau de prise en charge pour la sauvegarde de machine virtuelle Azure](backup-support-matrix-iaas.md)
- [Questions fréquentes (FAQ) – Sauvegarde des machines virtuelles Azure](backup-azure-vm-backup-faq.md)
