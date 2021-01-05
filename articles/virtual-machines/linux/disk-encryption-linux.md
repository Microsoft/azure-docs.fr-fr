---
title: Scénarios Azure Disk Encryption sur les machines virtuelles Linux
description: Cet article fournit des instructions pour l’activation de Microsoft Azure Disk Encryption pour les machines virtuelles Linux dans différents scénarios.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 70ebe8ede75935bcc8d8db8729fef165a5616fe7
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96779802"
---
# <a name="azure-disk-encryption-scenarios-on-linux-vms"></a>Scénarios Azure Disk Encryption sur les machines virtuelles Linux


Azure Disk Encryption pour machines virtuelles Linux utilise la fonctionnalité DM-Crypt de Linux pour effectuer un chiffrement complet du disque du système d’exploitation et des disques de données. Il assure en outre le chiffrement des disques temporaires lors de l’utilisation de la fonctionnalité EncryptFormatAll.

Azure Disk Encryption est [intégré à Azure Key Vault](disk-encryption-key-vault.md) pour faciliter le contrôle et la gestion des clés et des secrets de chiffrement des disques. Pour obtenir une vue d’ensemble du service, consultez [Azure Disk Encryption pour les machines virtuelles Linux](disk-encryption-overview.md).

Vous pouvez appliquer le chiffrement de disque uniquement aux machines virtuelles dont [la taille et le système d’exploitation sont pris en charge](disk-encryption-overview.md#supported-vms-and-operating-systems). Vous devez également satisfaire les prérequis suivants :

- [Conditions supplémentaires pour les machines virtuelles](disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Exigences réseau](disk-encryption-overview.md#networking-requirements)
- [Exigences liées au stockage des clés de chiffrement](disk-encryption-overview.md#encryption-key-storage-requirements)

Dans tous les cas, vous devez [capturer un instantané](snapshot-copy-managed-disk.md) et/ou créer une sauvegarde avant le chiffrement des disques. Les sauvegardes vous garantissent une possibilité de récupération en cas de défaillance inattendue lors du chiffrement. Les machines virtuelles avec des disques managés imposent une sauvegarde avant que le chiffrement soit effectué. Une fois la sauvegarde effectuée, vous pouvez utiliser la cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) pour chiffrer des disques managés en spécifiant le paramètre -skipVmBackup. Pour plus d’informations sur la façon de sauvegarder et de restaurer des machines virtuelles chiffrées, consultez l’article [Sauvegarde Azure](../../backup/backup-azure-vms-encryption.md). 

>[!WARNING]
> - Si vous avez déjà utilisé Azure Disk Encryption avec Azure AD pour chiffrer une machine virtuelle, vous devez continuer à utiliser cette option pour chiffrer votre machine virtuelle. Pour plus d’informations, consultez [Azure Disk Encryption avec Azure AD (version précédente)](disk-encryption-overview-aad.md). 
>
> - Lors du chiffrement de volumes de système d’exploitation Linux, la machine virtuelle ne devrait pas être disponible. Nous vous recommandons vivement d'éviter les connexions SSH pendant le chiffrement afin d'éviter tout problème risquant de bloquer les fichiers ouverts qui devront être accessibles pendant le processus de chiffrement. Pour vérifier la progression, utilisez l’applet de commande PowerShell [Get-AzVMDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus) ou la commande CLI [vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). Ce processus peut prendre quelques heures pour un volume de système d’exploitation de 30 Go, et du temps supplémentaire pour le chiffrement des volumes de données. La durée de chiffrement de volume de données est proportionnelle à la taille et à la quantité des volumes de données, sauf si l’option « EncryptFormatAll » est utilisée. 
> - La désactivation du chiffrement sur les machines virtuelles Linux est prise en charge seulement pour les volumes de données. Elle n’est pas prise en charge sur les volumes de données ou de système d’exploitation si le volume du système d’exploitation a été chiffré.  

## <a name="install-tools-and-connect-to-azure"></a>Installer les outils et se connecter à Azure

Vous pouvez activer et gérer Azure Disk Encryption par le biais d’[Azure CLI](/cli/azure) et d’[Azure PowerShell](/powershell/azure/new-azureps-module-az). Pour ce faire, vous devez installer les outils localement et vous connecter à votre abonnement Azure.

### <a name="azure-cli"></a>Azure CLI

[Azure CLI 2.0](/cli/azure) est un outil en ligne de commande pour la gestion des ressources Azure. L’interface CLI est conçue pour interroger les données de manière flexible, pour prendre en charge les opérations de longue durée en tant que processus non bloquants et pour simplifier l’écriture de scripts. Vous pouvez l’installer localement en effectuant les étapes décrites dans [Installer Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).

 

Pour [vous connecter à votre compte Azure avec Azure CLI](/cli/azure/authenticate-azure-cli), utilisez la commande [az login](/cli/azure/reference-index?view=azure-cli-latest#az-login).

```azurecli
az login
```

Si vous voulez sélectionner un locataire sous lequel vous connecter, utilisez :
    
```azurecli
az login --tenant <tenant>
```

Si vous avez plusieurs abonnements et que vous voulez spécifier un de ceux-ci, récupérez la liste de vos abonnements avec [az account list](/cli/azure/account#az-account-list) et spécifiez le compte avec [az account set](/cli/azure/account#az-account-set).
     
```azurecli
az account list
az account set --subscription "<subscription name or ID>"
```

Pour plus d’informations, consultez [Bien démarrer avec Azure CLI 2.0](/cli/azure/get-started-with-azure-cli). 

### <a name="azure-powershell"></a>Azure PowerShell
Le [module az Azure PowerShell](/powershell/azure/new-azureps-module-az) fournit un ensemble de cmdlets qui utilise le modèle [Azure Resource Manager](../../azure-resource-manager/management/overview.md) pour gérer vos ressources Azure. Vous pouvez l’utiliser dans votre navigateur avec [Azure Cloud Shell](../../cloud-shell/overview.md), ou vous pouvez l’installer sur votre ordinateur local à l’aide des instructions fournies dans [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). 

Si vous l’avez déjà installé localement, veillez à utiliser la dernière version du Kit de développement logiciel (SDK) Azure PowerShell pour configurer Azure Disk Encryption. Téléchargez la dernière version [d’Azure PowerShell](https://github.com/Azure/azure-powershell/releases).

Pour [vous connecter à votre compte Azure avec Azure PowerShell](/powershell/azure/authenticate-azureps?view=azps-2.5.0), utilisez la cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0).

```powershell
Connect-AzAccount
```

Si vous avez plusieurs abonnements et que vous souhaitez en spécifier un, utilisez la cmdlet [Get-AzSubscription](/powershell/module/Az.Accounts/Get-AzSubscription) pour les lister, puis la cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext?view=azps-2.5.0) :

```powershell
Set-AzContext -Subscription -Subscription <SubscriptionId>
```

L’exécution de la cmdlet [Get-AzContext](/powershell/module/Az.Accounts/Get-AzContext) permet de vérifier que l’abonnement approprié a été sélectionné.

Pour vérifier que les cmdlets Azure Disk Encryption sont installées, utilisez la cmdlet [Get-command](/powershell/module/microsoft.powershell.core/get-command?view=powershell-6) :
     
```powershell
Get-command *diskencryption*
```
Pour plus d’informations, consultez [Bien démarrer avec Azure PowerShell](/powershell/azure/get-started-azureps). 

## <a name="enable-encryption-on-an-existing-or-running-linux-vm"></a>Activer le chiffrement sur une machine virtuelle Linux existante ou en cours d’exécution
Dans ce scénario, vous pouvez activer le chiffrement en utilisant le modèle Resource Manager, des applets de commande PowerShell ou des commandes CLI. Si vous avez besoin d’informations de schéma pour l’extension de machine virtuelle, consultez l’article [Azure Disk Encryption pour extension Linux](../extensions/azure-disk-enc-linux.md).

>[!IMPORTANT]
 >Il est impératif de réaliser un instantané et/ou une sauvegarde d’une instance de machine virtuelle basée sur un disque managé en dehors d’Azure Disk Encryption et avant de l’activer. Vous pouvez prendre un instantané du disque managé à partir du portail ou par le biais de [Sauvegarde Azure](../../backup/backup-azure-vms-encryption.md). Les sauvegardes vous garantissent une possibilité de récupération en cas de défaillance inattendue pendant le chiffrement. Une fois la sauvegarde effectuée, il est possible d’utiliser la cmdlet Set-AzVMDiskEncryptionExtension pour chiffrer des disques managés en spécifiant le paramètre -skipVmBackup. La commande Set-AzVMDiskEncryptionExtension échoue sur les machines virtuelles basées sur des disques managés tant qu’une sauvegarde n’a pas été effectuée et que ce paramètre n’a pas été spécifié. 
>
>Le chiffrement ou la désactivation du chiffrement peut entraîner le redémarrage de la machine virtuelle. 
>

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-using-azure-cli"></a>Activer le chiffrement sur une machine virtuelle Linux existante ou en cours d’exécution avec Azure CLI 

Vous pouvez activer le chiffrement de disque sur votre disque dur virtuel chiffré en installant et en utilisant l’outil en ligne de commande [Azure CLI](/cli/azure/?view=azure-cli-latest). Vous pouvez l’ouvrir dans le navigateur avec [Azure Cloud Shell](../../cloud-shell/overview.md), ou vous pouvez l’installer sur votre ordinateur local et l’utiliser dans une session PowerShell. Pour activer le chiffrement sur des machines virtuelles Linux existantes ou en cours d’exécution dans Azure, utilisez les commandes CLI suivantes :

Utilisez la commande [az vm encryption enable](/cli/azure/vm/encryption?view=azure-cli-latest#az-vm-encryption-show) pour activer le chiffrement sur une machine virtuelle en cours d’exécution dans Azure.

- **Chiffrer une machine virtuelle en cours d’exécution :**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
     ```

- **Chiffrer une machine virtuelle en cours d’exécution avec la clé KEK :**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

    >[!NOTE]
    > La syntaxe de la valeur du paramètre disk-encryption-keyvault est la chaîne d’identificateur complet : /subscriptions/[GUID_ID_abonnement]/resourceGroups/[nom_groupe_ressources]/providers/Microsoft.KeyVault/vaults/[nom_coffre_clés]</br>
La syntaxe de la valeur du paramètre key-encryption-key est l’URI complet de la clé KEK comme dans : https://[nom_coffre_clés].vault.azure.net/keys/[nom_clé_kek]/[ID_unique_clé_kek] 

- **Vérifier que les disques sont chiffrés :** Pour vérifier l’état du chiffrement d’une machine virtuelle, utilisez la commande [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). 

     ```azurecli-interactive
     az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
     ```

- **Désactiver le chiffrement :** pour désactiver le chiffrement, utilisez la commande [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). La désactivation du chiffrement est autorisée seulement sur les volumes de données pour les machines virtuelles Linux.

     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type "data"
     ```

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-using-powershell"></a>Activer le chiffrement sur une machine virtuelle Linux existante ou en cours d’exécution avec PowerShell
Utilisez l’applet de commande [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) pour activer le chiffrement sur une machine virtuelle en cours d’exécution dans Azure. Prenez un [instantané](snapshot-copy-managed-disk.md) et/ou effectuez une sauvegarde de la machine virtuelle avec [Sauvegarde Azure](../../backup/backup-azure-vms-encryption.md) avant de chiffrer les disques. Le paramètre -skipVmBackup est déjà spécifié dans les scripts PowerShell pour chiffrer une machine virtuelle Linux en cours d'exécution.

-  **Chiffrer une machine virtuelle en cours d’exécution :** Le script ci-dessous initialise vos variables et exécute la cmdlet Set-AzVMDiskEncryptionExtension. Les prérequis que sont le groupe de ressources, la machine virtuelle et le coffre de clés ont été créés. Remplacez MyVirtualMachineResourceGroup, MySecureVM et MySecureVault par vos valeurs. Modifiez le paramètre -VolumeType pour spécifier les disques que vous chiffrez.

     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;
      $sequenceVersion = [Guid]::NewGuid();  

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType '[All|OS|Data]' -SequenceVersion $sequenceVersion -skipVmBackup;
     ```
- **Chiffrer une machine virtuelle en cours d’exécution avec la clé KEK :** Il peut être nécessaire d’ajouter le paramètre -VolumeType si vous chiffrez des disques de données et non le disque du système d’exploitation. 

     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MyExtraSecureVM';
      $KeyVaultName = 'MySecureVault';
      $keyEncryptionKeyName = 'MyKeyEncryptionKey';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;
      $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;
      $sequenceVersion = [Guid]::NewGuid();  

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType '[All|OS|Data]' -SequenceVersion $sequenceVersion -skipVmBackup;
     ```

    >[!NOTE]
    > La syntaxe de la valeur du paramètre disk-encryption-keyvault est la chaîne d’identificateur complet : /subscriptions/[GUID_ID_abonnement]/resourceGroups/[nom_groupe_ressources]/providers/Microsoft.KeyVault/vaults/[nom_coffre_clés]</br> La syntaxe de la valeur du paramètre key-encryption-key est l’URI complet de la clé KEK comme dans : https://[nom_coffre_clés].vault.azure.net/keys/[nom_clé_kek]/[ID_unique_clé_kek] 
    
- **Vérifier que les disques sont chiffrés :** Pour vérifier l’état du chiffrement d’une machine virtuelle, utilisez l’applet de commande [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus). 
    
     ```azurepowershell-interactive 
     Get-AzVmDiskEncryptionStatus -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```
    
- **Désactiver le chiffrement de disque :** Pour désactiver le chiffrement, utilisez la cmdlet [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). La désactivation du chiffrement est autorisée seulement sur les volumes de données pour les machines virtuelles Linux.
     
     ```azurepowershell-interactive 
     Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-with-a-template"></a>Activer le chiffrement sur une machine virtuelle Linux existante ou en cours d’exécution avec un modèle

Vous pouvez activer le chiffrement de disque sur une machine virtuelle Linux existante ou en cours d’exécution dans Azure à l’aide du [modèle Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm-without-aad).

1. Cliquez sur **Déployer sur Azure** sur le modèle de démarrage rapide Azure.

2. Sélectionnez l’abonnement, le groupe de ressources, l’emplacement du groupe de ressources, les paramètres, les conditions légales et le contrat. Cliquez sur **Créer** pour activer le chiffrement sur la machine virtuelle existante ou en cours d’exécution.

Le tableau suivant répertorie des paramètres du modèle Resource Manager pour les machines virtuelles existantes ou en cours d’exécution :

| Paramètre | Description |
| --- | --- |
| vmName | Nom de la machine virtuelle d’exécution de l’opération de chiffrement. |
| keyVaultName | Nom du coffre de clés où la clé de chiffrement doit être chargée. Vous pouvez l’obtenir avec l’applet de commande `(Get-AzKeyVault -ResourceGroupName <MyKeyVaultResourceGroupName>). Vaultname` ou la commande Azure CLI `az keyvault list --resource-group "MyKeyVaultResourceGroupName"`.|
| keyVaultResourceGroup | Nom du groupe de ressources qui contient le coffre de clés. |
|  keyEncryptionKeyURL | URL de la clé de chiffrement principale utilisée pour chiffrer la clé de chiffrement. Ce paramètre est facultatif si vous sélectionnez **nokek** dans la liste déroulante UseExistingKek. Si vous sélectionnez **kek** dans la liste déroulante UseExistingKek, vous devez entrer la valeur _keyEncryptionKeyURL_. |
| volumeType | Type de volume sur lequel l’opération de chiffrement est effectuée. Les valeurs valides sont _Système d’exploitation_, _Données_ et _Tous_. 
| forceUpdateTag | Passez à une valeur unique comme un GUID chaque fois que l’opération doit être exécutée de force. |
| location | Emplacement pour toutes les ressources. |

Pour plus d’informations sur la configuration du modèle de chiffrement de disque de machine virtuelle Linux, consultez [Azure Disk Encryption pour Linux](../extensions/azure-disk-enc-linux.md).

## <a name="use-encryptformatall-feature-for-data-disks-on-linux-vms"></a>Utiliser la fonctionnalité EncryptFormatAll pour les disques de données sur les machines virtuelles Linux

Le paramètre **EncryptFormatAll** réduit le temps de chiffrement des disques de données Linux. Les partitions remplissant certains critères sont formatées, avec leurs systèmes de fichiers actuels, puis remontées à l’emplacement où elles se trouvaient avant l’exécution de la commande. Si vous voulez exclure un disque de données répondant aux critères, vous pouvez le démonter avant d’exécuter la commande.

 Après l’exécution de cette commande, tous les lecteurs montés précédemment seront formatés, et la couche de chiffrement sera démarrée sur le lecteur désormais vide. Quand cette option est sélectionnée, le disque temporaire attaché à la machine virtuelle est également chiffré. Si le disque temporaire est réinitialisé, il est reformaté et rechiffré par la solution Azure Disk Encryption dès que l’occasion s’en présente. Quand le disque de ressources est chiffré, l’[Agent Microsoft Azure Linux](../extensions/agent-linux.md) ne peut pas gérer le disque de ressources ni activer le fichier d’échange, mais ce dernier peut être configuré manuellement.

>[!WARNING]
> EncryptFormatAll ne doit pas être utilisé quand des données indispensables se trouvent sur les volumes de données d’une machine virtuelle. Vous pouvez exclure des disques du chiffrement en les démontant. Vous devez d’abord essayer EncryptFormatAll sur une machine virtuelle de test, comprendre le paramètre de la fonctionnalité et son implication, avant de l’essayer sur la machine virtuelle de production. L’option EncryptFormatAll formate le disque de données et toutes ses données seront perdues. Avant de continuer, vérifiez que les disques que vous souhaitez exclure sont correctement démontés. </br></br>
 >Si ce paramètre est défini durant la mise à jour des paramètres de chiffrement, un redémarrage peut se produire avant le chiffrement proprement dit. Dans ce cas, il peut être utile de supprimer du fichier fstab le disque que vous ne voulez pas formater. De même, vous devez ajouter la partition que vous voulez chiffrer-formater au fichier fstab avant de lancer l’opération de chiffrement. 

### <a name="encryptformatall-criteria"></a>Critères pour EncryptFormatAll
Ce paramètre fait que la commande parcourt toutes les partitions et les chiffre, à condition qu’elles répondent à **tous** les critères ci-dessous :
- Elle n’est pas une partition root/de système d’exploitation/de démarrage
- Elle n’est pas déjà chiffrée
- Elle n’est pas un volume BEK
- Elle n’est pas un volume RAID
- Elle n’est pas un volume LVM
- Elle est montée

Chiffrez les disques composant le volume RAID ou LVM au lieu de chiffrer le volume RAID ou LVM lui-même.

### <a name="use-the-encryptformatall-parameter-with-azure-cli"></a>Utiliser le paramètre EncryptFormatAll avec Azure CLI
Utilisez la commande [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) pour activer le chiffrement sur une machine virtuelle en cours d’exécution dans Azure.

-  **Chiffrer une machine virtuelle en cours d’exécution avec EncryptFormatAll :**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type "data" --encrypt-format-all
     ```

### <a name="use-the-encryptformatall-parameter-with-a-powershell-cmdlet"></a>Utiliser le paramètre EncryptFormatAll avec une applet de commande PowerShell
Utilisez l’applet de commande [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) avec le paramètre EncryptFormatAll. 

**Chiffrer une machine virtuelle en cours d’exécution avec EncryptFormatAll :** à titre d’exemple, le script ci-dessous initialise vos variables et exécute l’applet de commande Set-AzVMDiskEncryptionExtension avec le paramètre EncryptFormatAll. Les prérequis que sont le groupe de ressources, la machine virtuelle et le coffre de clés ont été créés. Remplacez MyVirtualMachineResourceGroup, MySecureVM et MySecureVault par vos valeurs.
  
```azurepowershell
$KVRGname = 'MyKeyVaultResourceGroup';
$VMRGName = 'MyVirtualMachineResourceGroup';
$vmName = 'MySecureVM';
$KeyVaultName = 'MySecureVault';
$KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
$KeyVaultResourceId = $KeyVault.ResourceId;

Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "data" -EncryptFormatAll
```


### <a name="use-the-encryptformatall-parameter-with-logical-volume-manager-lvm"></a>Utiliser le paramètre EncryptFormatAll avec Logical Volume Manager (LVM) 
Nous recommandons une installation LVM-on-crypt. Pour les exemples suivants, remplacez le chemin de périphérique et les points de montage par ce qui correspond à votre cas d’utilisation. Cette installation peut se faire comme suit :

1.  Ajoutez les disques de données qui constitueront la machine virtuelle.

1. Formatez, montez et ajoutez ces disques au fichier fstab.

1. Choisissez un standard de partition, créez une partition qui couvre la totalité du lecteur, puis formatez-la. Nous utilisons ici des liens symboliques générés par Azure. L’utilisation de liens symboliques évite les problèmes liés à la modification des noms des périphériques. Pour plus d’informations, consultez [Résoudre les problèmes relatifs aux noms des périphériques](../troubleshooting/troubleshoot-device-names-problems.md).
    
    ```bash
    parted /dev/disk/azure/scsi1/lun0 mklabel gpt
    parted -a opt /dev/disk/azure/scsi1/lun0 mkpart primary ext4 0% 100%
    
    mkfs -t ext4 /dev/disk/azure/scsi1/lun0-part1
    ```

1. Montez les disques :

    ```bash
    mount /dev/disk/azure/scsi1/lun0-part1 /mnt/mountpoint
    ````
    
    Ajoutez-les au fichier fstab :

    ```bash
    echo "/dev/disk/azure/scsi1/lun0-part1 /mnt/mountpoint ext4 defaults,nofail 0 2" >> /etc/fstab
    ```
    
1. Exécutez la cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension?view=azps-3.8.0) d’Azure PowerShell avec -EncryptFormatAll pour chiffrer ces disques.

    ```azurepowershell-interactive
    $KeyVault = Get-AzKeyVault -VaultName "MySecureVault" -ResourceGroupName "MySecureGroup"
    
    Set-AzVMDiskEncryptionExtension -ResourceGroupName "MySecureGroup" -VMName "MySecureVM" -DiskEncryptionKeyVaultUrl $KeyVault.VaultUri -DiskEncryptionKeyVaultId $KeyVault.ResourceId -EncryptFormatAll -SkipVmBackup -VolumeType Data
    ```

    Si vous souhaitez utiliser une clé de chiffrement à clé (KEK), transmettez l’URI de votre KEK et le ResourceID de votre coffre de clés aux paramètres -KeyEncryptionKeyUrl et -KeyEncryptionKeyVaultId, respectivement :

    ```azurepowershell-interactive
    $KeyVault = Get-AzKeyVault -VaultName "MySecureVault" -ResourceGroupName "MySecureGroup"
    $KEKKeyVault = Get-AzKeyVault -VaultName "MyKEKVault" -ResourceGroupName "MySecureGroup"
    $KEK = Get-AzKeyVaultKey -VaultName "myKEKVault" -KeyName "myKEKName"
    
    Set-AzVMDiskEncryptionExtension -ResourceGroupName "MySecureGroup" -VMName "MySecureVM" -DiskEncryptionKeyVaultUrl $KeyVault.VaultUri -DiskEncryptionKeyVaultId $KeyVault.ResourceId -EncryptFormatAll -SkipVmBackup -VolumeType Data -KeyEncryptionKeyUrl $$KEK.id -KeyEncryptionKeyVaultId $KEKKeyVault.ResourceId
    ```

1. Configurez le gestionnaire de volumes logiques sur ces nouveaux disques. Notez que les lecteurs chiffrés sont déverrouillés une fois que la machine virtuelle a fini de démarrer. Ainsi, le montage du gestionnaire de volumes logiques devra également être retardé.

## <a name="new-vms-created-from-customer-encrypted-vhd-and-encryption-keys"></a>Nouvelles machines virtuelles créées à partir de disques durs virtuels et de clés de chiffrement chiffrés par le client
Dans ce scénario, vous pouvez activer le chiffrement avec des applets de commande PowerShell ou avec des commandes CLI. 

Utilisez les instructions des mêmes scripts Azure Disk Encryption pour préparer des images préchiffrées qui peuvent être utilisées dans Azure. Une fois l’image créée, vous pouvez suivre la procédure décrite dans la section suivante pour créer une machine virtuelle Azure chiffrée.

* [Préparer un disque dur virtuel Linux déjà chiffré](disk-encryption-sample-scripts.md#prepare-a-pre-encrypted-linux-vhd)

>[!IMPORTANT]
 >Il est impératif de réaliser un instantané et/ou une sauvegarde d’une instance de machine virtuelle basée sur un disque managé en dehors d’Azure Disk Encryption et avant de l’activer. Vous pouvez prendre un instantané du disque managé à partir du portail ou utiliser le service [Sauvegarde Azure](../../backup/backup-azure-vms-encryption.md). Les sauvegardes vous garantissent une possibilité de récupération en cas de défaillance inattendue pendant le chiffrement. Une fois la sauvegarde effectuée, il est possible d’utiliser la cmdlet Set-AzVMDiskEncryptionExtension pour chiffrer des disques managés en spécifiant le paramètre -skipVmBackup. La commande Set-AzVMDiskEncryptionExtension échoue sur les machines virtuelles basées sur des disques managés tant qu’une sauvegarde n’a pas été effectuée et que ce paramètre n’a pas été spécifié. 
>
> Le chiffrement ou la désactivation du chiffrement peut entraîner le redémarrage de la machine virtuelle. 



### <a name="use-azure-powershell-to-encrypt-vms-with-pre-encrypted-vhds"></a>Utiliser Azure PowerShell pour chiffrer des machines virtuelles avec des disques durs virtuels préchiffrés 
Vous pouvez activer le chiffrement de disque sur votre disque dur virtuel chiffré avec la cmdlet PowerShell [Set-AzVMOSDisk](/powershell/module/Az.Compute/Set-AzVMOSDisk#examples). L’exemple ci-dessous vous montre certains paramètres courants. 

```azurepowershell
$VirtualMachine = New-AzVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Linux -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzVM -VM $VirtualMachine -ResourceGroupName "MyVirtualMachineResourceGroup"
```

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>Activer le chiffrement sur un disque de données nouvellement ajouté

Vous pouvez ajouter un nouveau disque de données avec [az vm disk attach](add-disk.md) ou [via le portail Azure](attach-disk-portal.md). Avant de pouvoir chiffrer, vous devez d’abord monter le disque de données nouvellement attaché. Vous devez demander le chiffrement du lecteur de données, car le lecteur sera inutilisable pendant que le chiffrement est en cours. 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-cli"></a>Activer le chiffrement sur un disque nouvellement ajouté avec Azure CLI

 Si la machine virtuelle a déjà été chiffrée avec « All », le paramètre --volume-type doit rester « All ». All inclut les disques de système d’exploitation et de données. Si la machine virtuelle a déjà été chiffrée avec le type de volume « OS », le paramètre --volume-type doit être remplacé par « All » afin que le système d’exploitation et le nouveau disque de données soient inclus. Si la machine virtuelle a été chiffrée avec uniquement le type de volume « Data », le paramètre « Data » peut être conservé comme illustré ci-dessous. L’ajout et l’attachement d’un nouveau disque de données à une machine virtuelle ne constituent pas une préparation suffisante pour le chiffrement. Le disque nouvellement attaché doit aussi être formaté et monté correctement au sein de la machine virtuelle avant l’activation du chiffrement. Sur Linux, le disque doit être monté dans/etc/fstab avec un [nom de l’appareil de bloc persistant](../troubleshooting/troubleshoot-device-names-problems.md).  

Contrairement à la syntaxe de PowerShell, l’interface CLI ne nécessite pas que l’utilisateur fournisse une version de séquence unique lors de l’activation du chiffrement. L’interface CLI génère automatiquement et utilise sa propre valeur de version de séquence unique.

-  **Chiffrer des volumes de données d’une machine virtuelle en cours d’exécution :**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type "Data"
     ```

- **Chiffrer des volumes de données d’une machine virtuelle en cours d’exécution à l’aide de KEK :**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "Data"
     ```

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>Activer le chiffrement sur un disque nouvellement ajouté avec Azure PowerShell
 Quand vous utilisez PowerShell pour chiffrer un nouveau disque pour Linux, vous devez spécifier une nouvelle version de la séquence. La version de la séquence doit être unique. Le script ci-dessous génère un GUID pour la version de la séquence. Prenez un [instantané](snapshot-copy-managed-disk.md) et/ou effectuez une sauvegarde de la machine virtuelle avec [Sauvegarde Azure](../../backup/backup-azure-vms-encryption.md) avant de chiffrer les disques. Le paramètre -skipVmBackup est déjà spécifié dans les scripts PowerShell pour chiffrer un disque de données nouvellement ajouté.
 

-  **Chiffrer des volumes de données d’une machine virtuelle en cours d’exécution :** Le script ci-dessous initialise vos variables et exécute la cmdlet Set-AzVMDiskEncryptionExtension. Les prérequis que sont le groupe de ressources, la machine virtuelle et le coffre de clés doivent déjà avoir été créés. Remplacez MyVirtualMachineResourceGroup, MySecureVM et MySecureVault par vos valeurs. Les valeurs acceptables pour le paramètre -VolumeType sont « All », « OS » et « Data ». Si la machine virtuelle a déjà été chiffrée avec le type de volume « OS » ou « All », le paramètre -VolumeType doit être remplacé par « All » afin que le système d’exploitation et le nouveau disque de données soient inclus.

      ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;
      $sequenceVersion = [Guid]::NewGuid();

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'data' –SequenceVersion $sequenceVersion -skipVmBackup;
      ```
- **Chiffrer des volumes de données d’une machine virtuelle en cours d’exécution à l’aide de KEK :** Les valeurs acceptables pour le paramètre -VolumeType sont « All », « OS » et « Data ». Si la machine virtuelle a déjà été chiffrée avec un type de volume « OS » ou « All », le paramètre -VolumeType doit être remplacé par All afin que le système d’exploitation et le nouveau disque de données soient inclus.

     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MyExtraSecureVM';
      $KeyVaultName = 'MySecureVault';
      $keyEncryptionKeyName = 'MyKeyEncryptionKey';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;
      $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;
      $sequenceVersion = [Guid]::NewGuid();

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'data' –SequenceVersion $sequenceVersion -skipVmBackup;
     ```

    >[!NOTE]
    > La syntaxe de la valeur du paramètre disk-encryption-keyvault est la chaîne d’identificateur complet : /subscriptions/[GUID_ID_abonnement]/resourceGroups/[nom_groupe_ressourcesKV]/providers/Microsoft.KeyVault/vaults/[nom_coffre_clés]</br> La syntaxe de la valeur du paramètre key-encryption-key est l’URI complet de la clé KEK comme dans : https://[nom_coffre_clés].vault.azure.net/keys/[nom_clé_kek]/[ID_unique_clé_kek] 


## <a name="disable-encryption-for-linux-vms"></a>Désactiver le chiffrement pour les machines virtuelles Linux
[!INCLUDE [disk-encryption-disable-encryption-cli](../../../includes/disk-encryption-disable-cli.md)]

## <a name="unsupported-scenarios"></a>Scénarios non pris en charge

Azure Disk Encryption ne fonctionne pas pour les scénarios, fonctionnalités et technologies Linux suivants :

- Chiffrement de machines virtuelles de niveau de base ou de machines virtuelles créées par le biais de la méthode de création de machine virtuelle classique
- Désactivation du chiffrement sur un lecteur de système d’exploitation ou de données d’une machine virtuelle Linux quand le lecteur de système d’exploitation est chiffré
- Chiffrement du lecteur de système d’exploitation pour des groupes de machines virtuelles identiques.
- Chiffrement d’images personnalisées sur des machines virtuelles Linux
- Intégration à un système de gestion de clés local
- Azure Files (système de fichiers partagés).
- NFS (système de gestion de fichiers en réseau).
- Volumes dynamiques.
- Disques de système d’exploitation éphémères.
- Chiffrement des systèmes de fichiers partagés/distribués comme (liste non exhaustive) : DFS, GFS, DRDB et CephFS.
- Déplacement d’une machine virtuelle chiffrée vers un autre abonnement.
- Création d’une image ou d’une capture instantanée d’une machine virtuelle chiffrée et utilisation de celle-ci pour déployer des machines virtuelles supplémentaires.
- Vidage sur incident du noyau (kdump)
- Oracle ACFS (ASM Cluster File System).
- Machines virtuelles Gen2 (consultez : [Prise en charge des machines virtuelles de génération 2 sur Azure](../generation-2.md#generation-1-vs-generation-2-capabilities)).
- Les disques NVMe des machines virtuelles de la série Lsv2 (cf. [Série Lsv2](../lsv2-series.md)).
- Une machine virtuelle avec des « points de montage imbriqués », autrement dit, avec plusieurs points de montage dans un même chemin d’accès (par exemple, « /1stmountpoint/data/2stmountpoint »).
- Machine virtuelle avec un lecteur de données monté sur un dossier du système d’exploitation.
- Machines virtuelles de la série M avec des disques Accélérateur d’écriture.
- Application d’ADE à une machine virtuelle qui possède des disques chiffrés avec un [chiffrement côté serveur avec des clés gérées par le client](../disk-encryption.md) (SSE + CMK). L’application de SSE + CMK à un disque de données sur une machine virtuelle chiffrée avec ADE est également un scénario non pris en charge.
- Migration d’une machine virtuelle chiffrée par ADE ou n’a **jamais** été chiffrée avec ADE, pour un [chiffrement côté serveur avec clés gérées par le client](../disk-encryption.md).
- [Tailles de machine virtuelle Azure sans disque temporaire local](../azure-vms-no-temp-disk.md) ; spécifiquement, Dv4, Dsv4, Ev4 et Esv4.
- Chiffrement des machines virtuelles dans les clusters de basculement.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble d’Azure Disk Encryption](disk-encryption-overview.md)
- [Exemples de scripts Azure Disk Encryption](disk-encryption-sample-scripts.md)
- [Résolution des problèmes liés à Azure Disk Encryption](disk-encryption-troubleshooting.md)