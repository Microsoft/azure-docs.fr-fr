---
title: Scénarios Azure Disk Encryption sur les machines virtuelles Windows
description: Cet article fournit des instructions sur l’activation de Microsoft Azure Disk Encryption pour les machines virtuelles Windows dans différents scénarios.
author: msmbaldwin
ms.service: virtual-machines-windows
ms.subservice: security
ms.topic: how-to
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: df74a15a84ba04addb75b021d3b77b06cfe2ea48
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96781111"
---
# <a name="azure-disk-encryption-scenarios-on-windows-vms"></a>Scénarios Azure Disk Encryption sur les machines virtuelles Windows

Le service Azure Disk Encryption pour machines virtuelles Windows utilise la fonctionnalité BitLocker de Windows pour effectuer un chiffrement complet des disques de système d’exploitation et du disques de données. Il assure en outre le chiffrement du disque temporaire lorsque le paramètre VolumeType est défini sur Tous.

Azure Disk Encryption est [intégré à Azure Key Vault](disk-encryption-key-vault.md) pour faciliter le contrôle et la gestion des clés et des secrets de chiffrement des disques. Pour obtenir une vue d’ensemble du service, consultez [Azure Disk Encryption pour les machines virtuelles Windows](disk-encryption-overview.md).

Vous pouvez appliquer le chiffrement de disque uniquement aux machines virtuelles dont [la taille et le système d’exploitation sont pris en charge](disk-encryption-overview.md#supported-vms-and-operating-systems). Vous devez également satisfaire les prérequis suivants :

- [Exigences réseau](disk-encryption-overview.md#networking-requirements)
- [Exigences de stratégies de groupe](disk-encryption-overview.md#group-policy-requirements)
- [Exigences liées au stockage des clés de chiffrement](disk-encryption-overview.md#encryption-key-storage-requirements)

>[!IMPORTANT]
> - Si vous avez déjà utilisé Azure Disk Encryption avec Azure AD pour chiffrer une machine virtuelle, vous devez continuer à utiliser cette option pour chiffrer votre machine virtuelle. Pour plus d’informations, consultez [Azure Disk Encryption avec Azure AD (version précédente)](disk-encryption-overview-aad.md). 
>
> - Vous devez [prendre un instantané](snapshot-copy-managed-disk.md) et/ou créer une sauvegarde avant le chiffrement des disques. Les sauvegardes vous garantissent une possibilité de récupération en cas de défaillance inattendue lors du chiffrement. Les machines virtuelles avec des disques managés imposent une sauvegarde avant que le chiffrement soit effectué. Une fois la sauvegarde effectuée, vous pouvez utiliser la cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) pour chiffrer des disques managés en spécifiant le paramètre -skipVmBackup. Pour plus d’informations sur la façon de sauvegarder et de restaurer des machines virtuelles chiffrées, consultez l’article [Sauvegarder et restaurer une machine virtuelle Azure chiffrée](../../backup/backup-azure-vms-encryption.md). 
>
> - Le chiffrement ou la désactivation du chiffrement peut entraîner le redémarrage d’une machine virtuelle.

## <a name="install-tools-and-connect-to-azure"></a>Installer les outils et se connecter à Azure

[!INCLUDE [disk-encryption-install-cli-powershell](../../../includes/disk-encryption-install-cli-powershell.md)]

## <a name="enable-encryption-on-an-existing-or-running-windows-vm"></a>Activer le chiffrement sur une machine virtuelle Windows existante ou en cours d’exécution
Dans ce scénario, vous pouvez activer le chiffrement en utilisant le modèle Resource Manager, des applets de commande PowerShell ou des commandes CLI. Si vous avez besoin d’informations de schéma pour l’extension de machine virtuelle, consultez l’article [Azure Disk Encryption pour extension Windows](../extensions/azure-disk-enc-windows.md).

### <a name="enable-encryption-on-existing-or-running-vms-with-azure-powershell"></a>Activer le chiffrement sur des machines virtuelles existantes ou en cours d’exécution avec Azure PowerShell 
Utilisez la cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) pour activer le chiffrement sur une machine virtuelle IaaS en cours d’exécution dans Azure. 

-  **Chiffrer une machine virtuelle en cours d’exécution :** Le script ci-dessous initialise vos variables et exécute la cmdlet Set-AzVMDiskEncryptionExtension. Les prérequis que sont le groupe de ressources, la machine virtuelle et le coffre de clés doivent déjà avoir été créés. Remplacez MyKeyVaultResourceGroup, MyVirtualMachineResourceGroup, MySecureVM et MySecureVault par vos valeurs.

     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId;
    ```
- **Chiffrer une machine virtuelle en cours d’exécution avec la clé KEK :** 

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

     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;

     ```
     
   >[!NOTE]
   > La syntaxe de la valeur du paramètre disk-encryption-keyvault est la chaîne d’identificateur complet : /subscriptions/[GUID_ID_abonnement]/resourceGroups/[nom_groupe_ressources]/providers/Microsoft.KeyVault/vaults/[nom_coffre_clés]</br> La syntaxe de la valeur du paramètre key-encryption-key est l’URI complet de la clé KEK comme dans : https://[nom_coffre_clés].vault.azure.net/keys/[nom_clé_kek]/[ID_unique_clé_kek] 

- **Vérifier que les disques sont chiffrés :** Pour vérifier l’état du chiffrement d’une machine virtuelle IaaS, utilisez la cmdlet [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus). 
     ```azurepowershell-interactive
     Get-AzVmDiskEncryptionStatus -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```
    
- **Désactiver le chiffrement de disque :** Pour désactiver le chiffrement, utilisez la cmdlet [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). La désactivation du chiffrement de disque de données sur la machine virtuelle Windows lorsque les disques du système d’exploitation et de données ont été chiffrés ne fonctionne pas comme prévu. Désactivez plutôt le chiffrement sur tous les disques.

     ```azurepowershell-interactive
     Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```

### <a name="enable-encryption-on-existing-or-running-vms-with-the-azure-cli"></a>Activer le chiffrement sur des machines virtuelles existantes ou en cours d’exécution avec Azure CLI
Utilisez la commande [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) pour activer le chiffrement sur une machine virtuelle IaaS en cours d’exécution dans Azure.

- **Chiffrer une machine virtuelle en cours d’exécution :**

    ```azurecli-interactive
    az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
    ```

- **Chiffrer une machine virtuelle en cours d’exécution avec la clé KEK :**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

     >[!NOTE]
     > La syntaxe de la valeur du paramètre disk-encryption-keyvault est la chaîne d’identificateur complet : /subscriptions/[GUID_ID_abonnement]/resourceGroups/[nom_groupe_ressources]/providers/Microsoft.KeyVault/vaults/[nom_coffre_clés] </br> La syntaxe de la valeur du paramètre key-encryption-key est l’URI complet de la clé KEK comme dans : https://[nom_coffre_clés].vault.azure.net/keys/[nom_clé_kek]/[ID_unique_clé_kek] 

- **Vérifier que les disques sont chiffrés :** Pour vérifier l’état du chiffrement d’une machine virtuelle IaaS, utilisez la commande [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). 

     ```azurecli-interactive
     az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
     ```

- **Désactiver le chiffrement :** pour désactiver le chiffrement, utilisez la commande [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). La désactivation du chiffrement de disque de données sur la machine virtuelle Windows lorsque les disques du système d’exploitation et de données ont été chiffrés ne fonctionne pas comme prévu. Désactivez plutôt le chiffrement sur tous les disques.

     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type [ALL, DATA, OS]
     ```

### <a name="using-the-resource-manager-template"></a>Utilisation du modèle Resource Manager

Vous pouvez activer le chiffrement de disque sur des machines virtuelles Windows IaaS existantes ou en cours d’exécution dans Azure en utilisant le [modèle Resource Manager pour chiffrer une machine virtuelle Windows en cours d’exécution](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm-without-aad).


1. Dans le modèle de démarrage rapide Azure, cliquez sur **Déployer sur Azure**.

2. Sélectionnez l’abonnement, le groupe de ressources, l’emplacement, les paramètres, les conditions juridiques et le contrat. Cliquez sur **Acheter** pour activer le chiffrement sur la machine virtuelle IaaS existante ou en cours d’exécution.

Le tableau suivant répertorie les paramètres du modèle Resource Manager pour les machines virtuelles existantes ou en cours d’exécution :

| Paramètre | Description |
| --- | --- |
| vmName | Nom de la machine virtuelle d’exécution de l’opération de chiffrement. |
| keyVaultName | Nom du coffre de clés dans lequel la clé BitLocker doit être téléchargée. Vous pouvez l’obtenir avec la cmdlet `(Get-AzKeyVault -ResourceGroupName <MyKeyVaultResourceGroupName>). Vaultname` ou la commande Azure CLI `az keyvault list --resource-group "MyKeyVaultResourceGroup"`|
| keyVaultResourceGroup | Nom du groupe de ressources qui contient le coffre de clés|
|  keyEncryptionKeyURL | URL de la clé de chiffrement principale au format https://&lt;nom-coffre-clés&gt;.vault.azure.net/key/&lt;nom-clé&gt;. Si vous ne souhaitez pas utiliser de clé de chiffrement principale, laissez ce champ vide. |
| volumeType | Type de volume sur lequel l’opération de chiffrement est effectuée. Les valeurs valides sont _Système d’exploitation_, _Données_ et _Tous_. 
| forceUpdateTag | Passez à une valeur unique comme un GUID chaque fois que l’opération doit être exécutée de force. |
| resizeOSDisk | Si la partition du système d’exploitation doit être redimensionnée pour occuper tout le disque dur virtuel du système d’exploitation avant de fractionner le volume système. |
| location | Emplacement pour toutes les ressources. |

## <a name="enable-encryption-on-nvme-disks-for-lsv2-vms"></a>Activer le chiffrement sur des disques NVMe pour machines virtuelles Lsv2

Ce scénario décrit l’activation de Azure Disk Encryption sur des disques NVMe pour les machines virtuelles de la série Lsv2.  La série Lsv2 est dotée d’un stockage NVMe local. Les disques NVMe locaux sont temporaire. Les données présentes sur ces disques seront perdues si vous arrêtez/libérez votre machine virtuelle (voir [Série Lsv2](../lsv2-series.md)).

Pour activer le chiffrement sur les disques NVMe :

1. Initialisez les disques NVMe et créez des volumes NTFS.
1. Activez le chiffrement sur la machine virtuelle avec le paramètre VolumeType défini sur All. Cela active le chiffrement pour tous les disques de système de système d’exploitation et de données, y compris les volumes basés sur des disques NVMe. Pour plus d’informations, consultez [Activer le chiffrement sur une machine virtuelle Windows existante ou en cours d’exécution](#enable-encryption-on-an-existing-or-running-windows-vm).

Le chiffrement est conservé sur les disques NVMe dans les scénarios suivants :
- Redémarrage de machine virtuelle
- Réimageage VMSS
- Permutation de système d’exploitation

Les disques NVMe ne seront pas initialisés dans les scénarios suivants :

- Démarrage de machine virtuelle après désallocation/libération
- Réparation de service
- Sauvegarde

Dans ces scénarios, les disques NVMe doivent être initialisés après le démarrage de la machine virtuelle. Pour activer le chiffrement sur les disques NVMe, exécutez la commande pour réactiver Azure Disk Encryption après l’initialisation des disques NVMe.

Outre les scénarios énumérés dans la section [Scénarios non pris en charge](#unsupported-scenarios), le chiffrement des disques NVMe n’est pas pris en charge pour les ressources suivantes :

- Machines virtuelles chiffrées avec Azure Disk Encryption avec AAD (version précédente)
- Disques NVMe avec espaces de stockage
- Azure Site Recovery de références (SKU) avec des disques NVMe (consultez [Prendre en charge la matrice de la récupération d’urgence de machines virtuelles Azure entre les régions Azure : Machines répliquées – Stockage](../../site-recovery/azure-to-azure-support-matrix.md#replicated-machines---storage)).

## <a name="new-iaas-vms-created-from-customer-encrypted-vhd-and-encryption-keys"></a>Nouvelles machines virtuelles IaaS créées à partir d’un disque dur virtuel chiffré par le client et de clés de chiffrement

Dans le cadre de ce scénario, vous pouvez créer une machine virtuelle à partir d’un disque dur virtuel préchiffré et des clés de chiffrement associées à l’aide de cmdlets PowerShell ou de commandes CLI. 

Suivez les instructions de la section [Préparer un disque dur virtuel Windows préchiffré](disk-encryption-sample-scripts.md#prepare-a-pre-encrypted-windows-vhd). Une fois l’image créée, vous pouvez suivre la procédure décrite dans la section suivante pour créer une machine virtuelle Azure chiffrée.


### <a name="encrypt-vms-with-pre-encrypted-vhds-with-azure-powershell"></a>Chiffrer des machines virtuelles avec des disques durs virtuels préchiffrés à l’aide d’Azure PowerShell
Vous pouvez activer le chiffrement de disque sur votre disque dur virtuel chiffré avec la cmdlet PowerShell [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk#examples). L’exemple ci-dessous vous montre certains paramètres courants. 

```azurepowershell
$VirtualMachine = New-AzVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Windows -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myKVresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzVM -VM $VirtualMachine -ResourceGroupName "MyVirtualMachineResourceGroup"
```

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>Activer le chiffrement sur un disque de données nouvellement ajouté
Vous pouvez [ajouter un nouveau disque à une machine virtuelle Windows avec PowerShell](attach-disk-ps.md) ou via le [portail Azure](attach-managed-disk-portal.md). 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>Activer le chiffrement sur un disque nouvellement ajouté avec Azure PowerShell
 Quand vous utilisez PowerShell pour chiffrer un nouveau disque de machines virtuelles Windows, vous devez spécifier une nouvelle version de la séquence. La version de la séquence doit être unique. Le script ci-dessous génère un GUID pour la version de la séquence. Dans certains cas, un disque de données nouvellement ajouté peut être chiffré automatiquement par l’extension Azure Disk Encryption. Le chiffrement automatique se produit généralement quand la machine virtuelle redémarre après la mise en ligne du nouveau disque. Cela est généralement dû au fait que « Tous » était spécifié pour le type de volume quand le chiffrement de disque a été précédemment exécuté sur la machine virtuelle. Si le chiffrement automatique se produit sur un disque de données nouvellement ajouté, nous recommandons de réexécuter l’applet de commande Set-AzVmDiskEncryptionExtension avec la nouvelle version de la séquence. Si votre nouveau disque de données est automatiquement chiffré et que vous ne voulez pas qu’il le soit, déchiffrez d’abord tous les lecteurs, puis chiffrez-les à nouveau avec une nouvelle version de séquence spécifiant le système d’exploitation pour le type de volume. 
  
 

-  **Chiffrer une machine virtuelle en cours d’exécution :** Le script ci-dessous initialise vos variables et exécute la cmdlet Set-AzVMDiskEncryptionExtension. Les prérequis que sont le groupe de ressources, la machine virtuelle et le coffre de clés doivent déjà avoir été créés. Remplacez MyKeyVaultResourceGroup, MyVirtualMachineResourceGroup, MySecureVM et MySecureVault par vos valeurs. Cet exemple utilise « All » pour le paramètre VolumeType, qui inclut les volumes de données et de systèmes d’exploitation. Si vous souhaitez uniquement chiffrer le volume de systèmes d’exploitation, utilisez « OS » pour le paramètre VolumeType. 

     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;
      $sequenceVersion = [Guid]::NewGuid();

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "All" –SequenceVersion $sequenceVersion;
    ```
- **Chiffrer une machine virtuelle en cours d’exécution avec la clé KEK :** Cet exemple utilise « All » pour le paramètre VolumeType, qui inclut les volumes de données et de systèmes d’exploitation. Si vous souhaitez uniquement chiffrer le volume de systèmes d’exploitation, utilisez « OS » pour le paramètre VolumeType.

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

     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "All" –SequenceVersion $sequenceVersion;

     ```

    >[!NOTE]
    > La syntaxe de la valeur du paramètre disk-encryption-keyvault est la chaîne d’identificateur complet : /subscriptions/[GUID_ID_abonnement]/resourceGroups/[nom_groupe_ressources]/providers/Microsoft.KeyVault/vaults/[nom_coffre_clés]</br> La syntaxe de la valeur du paramètre key-encryption-key est l’URI complet de la clé KEK comme dans : https://[nom_coffre_clés].vault.azure.net/keys/[nom_clé_kek]/[ID_unique_clé_kek] 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-cli"></a>Activer le chiffrement sur un disque nouvellement ajouté avec Azure CLI
 La commande Azure CLI vous propose automatiquement une nouvelle version de la séquence quand vous exécutez la commande pour activer le chiffrement. L’exemple utilise « All » pour le paramètre volume-type. Vous devrez peut-être remplacer le paramètre volume-type par « OS » si vous ne chiffrez que le disque du système d’exploitation. Contrairement à la syntaxe de PowerShell, l’interface CLI ne nécessite pas que l’utilisateur fournisse une version de séquence unique lors de l’activation du chiffrement. L’interface CLI génère automatiquement et utilise sa propre valeur de version de séquence unique.   

-  **Chiffrer une machine virtuelle en cours d’exécution :**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type "All"
     ```

- **Chiffrer une machine virtuelle en cours d’exécution avec la clé KEK :**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "All"
     ```


## <a name="disable-encryption"></a>Désactiver le chiffrement
[!INCLUDE [disk-encryption-disable-encryption-powershell](../../../includes/disk-encryption-disable-powershell.md)]

## <a name="unsupported-scenarios"></a>Scénarios non pris en charge

Azure Disk Encryption ne fonctionne pas pour les scénarios, fonctionnalités et technologies suivants :

- Chiffrement de machines virtuelles de niveau de base ou de machines virtuelles créées par le biais de la méthode de création de machine virtuelle classique
- Chiffrement de machines virtuelles configurées avec des systèmes RAID logiciels
- Chiffrement de machines virtuelles configurées avec des espaces de stockage direct (S2D) ou des versions de Windows Server antérieures à 2016 configurées avec des espaces de stockage Windows.
- Intégration à un système de gestion de clés local
- Azure Files (système de fichiers partagés).
- NFS (système de gestion de fichiers en réseau).
- Volumes dynamiques.
- Conteneurs Windows Server qui créent des volumes dynamiques pour chaque conteneur.
- Disques de système d’exploitation éphémères.
- Chiffrement de systèmes de fichiers partagés/distribués comme DFS, GFS, DRDB, etc.
- Déplacement d’une machine virtuelle chiffrée vers un autre abonnement ou une autre région.
- Création d’une image ou d’une capture instantanée d’une machine virtuelle chiffrée et utilisation de celle-ci pour déployer des machines virtuelles supplémentaires.
- Machines virtuelles Gen2 (consultez : [Prise en charge des machines virtuelles de génération 2 sur Azure](../generation-2.md#generation-1-vs-generation-2-capabilities))
- Machines virtuelles de la série M avec des disques Accélérateur d’écriture.
- Application d’ADE à une machine virtuelle qui possède des disques chiffrés avec un [chiffrement côté serveur avec des clés gérées par le client](../disk-encryption.md) (SSE + CMK). L’application de SSE + CMK à un disque de données sur une machine virtuelle chiffrée avec ADE est également un scénario non pris en charge.
- Migration d’une machine virtuelle chiffrée par ADE ou n’a **jamais** été chiffrée avec ADE, pour un [chiffrement côté serveur avec clés gérées par le client](../disk-encryption.md).
- [Tailles de machine virtuelle Azure sans disque temporaire local](../azure-vms-no-temp-disk.md) ; spécifiquement, Dv4, Dsv4, Ev4 et Esv4.
- Chiffrement des machines virtuelles dans les clusters de basculement.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble d’Azure Disk Encryption](disk-encryption-overview.md)
- [Exemples de scripts Azure Disk Encryption](disk-encryption-sample-scripts.md)
- [Résolution des problèmes liés à Azure Disk Encryption](disk-encryption-troubleshooting.md)