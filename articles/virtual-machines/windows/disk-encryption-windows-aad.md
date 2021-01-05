---
title: Azure Disk Encryption avec Azure AD pour les machines virtuelles Windows (version précédente)
description: Cet article fournit des instructions sur l’activation de Microsoft Azure Disk Encryption pour les machines virtuelles IaaS Windows.
author: msmbaldwin
ms.service: virtual-machines
ms.subservice: security
ms.topic: how-to
ms.author: mbaldwin
ms.date: 03/15/2019
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 352c8848b98bfb463c03ceea89ebe3f4b6ad6d5b
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92742428"
---
# <a name="azure-disk-encryption-with-azure-ad-for-windows-vms-previous-release"></a>Azure Disk Encryption avec Azure AD pour les machines virtuelles Windows (version précédente)

**La nouvelle version d’Azure Disk Encryption vous évite de fournir un paramètre d’application Azure AD pour activer le chiffrement de disque de machine virtuelle. Avec la nouvelle version, vous n’êtes plus obligé de fournir les informations d’identification Azure AD lors de l’étape d’activation du chiffrement. Avec la nouvelle version, toutes les nouvelles machines virtuelles doivent être chiffrées sans les paramètres d’application Azure AD. Pour consulter les instructions d’activation du chiffrement de disque des machines virtuelles grâce à la nouvelle version, consultez l’article [Azure Disk Encryption pour les machines virtuelles Windows](disk-encryption-windows.md). Les machines virtuelles déjà chiffrées avec les paramètres d’application Azure AD sont toujours prises en charge et doivent continuer à être gérées avec la syntaxe AAD.**


Il existe de nombreux scénarios permettant d’activer le chiffrement de disque et les étapes peuvent varier selon le scénario. Les sections suivantes décrivent ces scénarios de façon plus détaillée pour les machines virtuelles IaaS Windows. Avant de pouvoir utiliser le chiffrement de disque, les [prérequis pour Azure Disk Encryption](disk-encryption-overview-aad.md) doivent être satisfaits. 


>[!IMPORTANT]
> - Vous devez [prendre un instantané](snapshot-copy-managed-disk.md) et/ou créer une sauvegarde avant le chiffrement des disques. Les sauvegardes vous garantissent une possibilité de récupération en cas de défaillance inattendue lors du chiffrement. Les machines virtuelles avec des disques managés imposent une sauvegarde avant que le chiffrement soit effectué. Une fois la sauvegarde effectuée, vous pouvez utiliser la cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) pour chiffrer des disques managés en spécifiant le paramètre -skipVmBackup. Pour plus d’informations sur la façon de sauvegarder et de restaurer des machines virtuelles chiffrées, consultez l’article [Sauvegarder et restaurer une machine virtuelle Azure chiffrée](../../backup/backup-azure-vms-encryption.md). 
>
> - Le chiffrement ou la désactivation du chiffrement peut entraîner le redémarrage d’une machine virtuelle.

## <a name="enable-encryption-on-new-iaas-vms-created-from-the-marketplace"></a>Activer le chiffrement sur de nouvelles machines virtuelles IaaS créées à partir de la Place de marché
Vous pouvez activer le chiffrement de disque sur de nouvelles machines virtuelles IaaS Windows à partir de la Place de marché dans Azure en utilisant un modèle Resource Manager. Le modèle crée une machine virtuelle Windows chiffrée en utilisant l’image de la galerie Windows Server 2012.

1. Sur le [modèle Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-create-new-vm-gallery-image), cliquez sur **Déployer sur Azure** .

2. Sélectionnez l’abonnement, le groupe de ressources, l’emplacement du groupe de ressources, les paramètres, les conditions légales et le contrat. Cliquez sur **Acheter** pour déployer une nouvelle machine virtuelle IaaS où le chiffrement est activé.

3. Après avoir déployé le modèle, vérifiez l’état du chiffrement de la machine virtuelle avec la méthode de votre choix :
     - Vérifiez avec l’interface CLI en utilisant la commande [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). 

         ```azurecli-interactive 
         az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
         ```

     - Vérifiez avec Azure PowerShell en utilisant l’applet de commande [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus). 

         ```azurepowershell-interactive
         Get-AzVmDiskEncryptionStatus -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
         ```

     -  Sélectionnez la machine virtuelle, puis cliquez sur **Disques** sous le titre **Paramètres** pour vérifier l’état du chiffrement dans le portail. Dans le graphique sous **Chiffrement** , vous voyez s’il est activé. 
           ![Portail Azure - Chiffrement de disque activé](../media/disk-encryption/disk-encryption-fig2.png)

Le tableau suivant répertorie les paramètres du modèle Resource Manager pour les nouvelles machines virtuelles dans un scénario Place de marche utilisant l’ID de client Azure AD :

| Paramètre | Description | 
| --- | --- |
| adminUsername | Nom de l’utilisateur administrateur de la machine virtuelle. |
| adminPassword | Mot de passe de l’utilisateur administrateur de la machine virtuelle. |
| newStorageAccountName | Nom du compte de stockage pour stocker les disques durs virtuels du système d’exploitation et de données. |
| vmSize | Taille de la machine virtuelle. Actuellement, seules les séries A, D et G standard sont prises en charge. |
| virtualNetworkName | Nom du réseau virtuel auquel la carte d’interface réseau de machine virtuelle appartient. |
| subnetName | Nom du sous-réseau du réseau virtuel auquel la carte d’interface réseau de machine virtuelle appartient. |
| AADClientID | ID de client de l’application Azure AD qui dispose des autorisations pour écrire des clés secrètes dans le coffre de clés. |
| AADClientSecret | Clé secrète de client de l’application Azure AD qui dispose des autorisations pour écrire des clés secrètes dans le coffre de clés. |
| keyVaultURL | URL du coffre de clés dans lequel la clé BitLocker doit être téléchargée. Vous pouvez l’obtenir avec l’applet de commande `(Get-AzKeyVault -VaultName "MyKeyVault" -ResourceGroupName "MyKeyVaultResourceGroupName").VaultURI` ou la commande Azure CLI `az keyvault show --name "MySecureVault" --query properties.vaultUri` |
| keyEncryptionKeyURL | URL de la clé de chiffrement à clé utilisée pour chiffrer la clé BitLocker générée (facultatif). </br> </br>KeyEncryptionKeyURL est un paramètre facultatif. Vous pouvez apporter vos propres clés de chiffrement à clé (KEK) pour renforcer la clé de chiffrement des données (clé secrète de chiffrement) dans votre coffre de clés. |
| keyVaultResourceGroup | Groupe de ressources du coffre de clés. |
| vmName | Nom de la machine virtuelle sur laquelle l’opération de chiffrement doit être effectuée. |


## <a name="enable-encryption-on-existing-or-running-iaas-windows-vms"></a><a name="bkmk_RunningWinVM"></a> Activer le chiffrement sur des machines virtuelles IaaS Windows existantes ou en cours d’exécution
Dans ce scénario, vous pouvez activer le chiffrement avec un modèle, avec des applets de commande PowerShell ou avec des commandes CLI. Les sections suivantes expliquent en détail comment activer Azure Disk Encryption. 


### <a name="enable-encryption-on-existing-or-running-vms-with-azure-powershell"></a><a name="bkmk_RunningWinVMPSH"></a> Activer le chiffrement sur des machines virtuelles existantes ou en cours d’exécution avec Azure PowerShell 
Utilisez la cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) pour activer le chiffrement sur une machine virtuelle IaaS en cours d’exécution dans Azure. Pour plus d’informations sur l’activation du chiffrement avec Azure Disk Encryption à l’aide des applets de commande PowerShell, consultez les billets de blog [Explorer Azure Disk Encryption avec Azure PowerShell - partie 1](/archive/blogs/azuresecurity/explore-azure-disk-encryption-with-azure-powershell) et [Explorer Azure Disk Encryption avec Azure PowerShell - partie 2](/archive/blogs/azuresecurity/explore-azure-disk-encryption-with-azure-powershell-part-2).

-  **Chiffrer une machine virtuelle en cours d’exécution avec un secret client :** Le script ci-dessous initialise vos variables et exécute la cmdlet Set-AzVMDiskEncryptionExtension. Les prérequis sont que le groupe de ressources, la machine virtuelle, le coffre de clés, l’application AAD et le secret client doivent déjà avoir été créés. Remplacez MyKeyVaultResourceGroup, MyVirtualMachineResourceGroup, MySecureVM, MySecureVault, My-AAD-client-ID et My-AAD-client-secret par vos valeurs.
     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $aadClientID = 'My-AAD-client-ID';
      $aadClientSecret = 'My-AAD-client-secret';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId;
    ```
- **Chiffrer une machine virtuelle en cours d’exécution avec une clé KEK pour wrapper le secret client :** Azure Disk Encryption vous permet de spécifier une clé existante de votre coffre de clés pour inclure dans un wrapper les secrets de chiffrement de disque qui ont été générés lors de l’activation du chiffrement. Quand une clé de chiffrement principale est spécifiée, Azure Disk Encryption utilise cette clé pour wrapper les secrets de chiffrement avant d’écrire dans Key Vault. 

     ```azurepowershell
     $KVRGname = 'MyKeyVaultResourceGroup';
     $VMRGName = 'MyVirtualMachineResourceGroup';
     $vmName = ‘MyExtraSecureVM’;
     $aadClientID = 'My-AAD-client-ID';
     $aadClientSecret = 'My-AAD-client-secret';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;

     ```
     
   >[!NOTE]
   > La syntaxe de la valeur du paramètre disk-encryption-keyvault est la chaîne d’identificateur complet : /subscriptions/[GUID_ID_abonnement]/resourceGroups/[nom_groupe_ressources]/providers/Microsoft.KeyVault/vaults/[nom_coffre_clés]</br> La syntaxe de la valeur du paramètre key-encryption-key est l’URI complet de la clé KEK comme dans : https://[nom_coffre_clés].vault.azure.net/keys/[nom_clé_kek]/[ID_unique_clé_kek] 

- **Vérifier que les disques sont chiffrés :** Pour vérifier l’état du chiffrement d’une machine virtuelle IaaS, utilisez la cmdlet [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus). 
     ```azurepowershell-interactive
     Get-AzVmDiskEncryptionStatus -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```
    
- **Désactiver le chiffrement de disque :** pour désactiver le chiffrement, utilisez l’applet de commande [Disable-AzureRmVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). 
     ```azurepowershell-interactive
     Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```

### <a name="enable-encryption-on-existing-or-running-vms-with--azure-cli"></a><a name="bkmk_RunningWinVMCLI"></a> Activer le chiffrement sur des machines virtuelles existantes ou en cours d’exécution avec Azure CLI
Utilisez la commande [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) pour activer le chiffrement sur une machine virtuelle IaaS en cours d’exécution dans Azure.

- **Chiffrer une machine virtuelle en cours d’exécution avec un secret client :**

    ```azurecli-interactive
    az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
    ```

- **Chiffrer une machine virtuelle en cours d’exécution avec une clé KEK pour wrapper le secret client :**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

     >[!NOTE]
     > La syntaxe de la valeur du paramètre disk-encryption-keyvault est la chaîne d’identificateur complet : /subscriptions/[GUID_ID_abonnement]/resourceGroups/[nom_groupe_ressources]/providers/Microsoft.KeyVault/vaults/[nom_coffre_clés] </br> La syntaxe de la valeur du paramètre key-encryption-key est l’URI complet de la clé KEK comme dans : https://[nom_coffre_clés].vault.azure.net/keys/[nom_clé_kek]/[ID_unique_clé_kek] 

- **Vérifier que les disques sont chiffrés :** Pour vérifier l’état du chiffrement d’une machine virtuelle IaaS, utilisez la commande [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). 

     ```azurecli-interactive
     az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
     ```

- **Désactiver le chiffrement :** pour désactiver le chiffrement, utilisez la commande [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). 
     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type [ALL, DATA, OS]
     ```


### <a name="using-the-resource-manager-template"></a><a name="bkmk_RunningWinVMwRM"> </a>Utilisation du modèle Resource Manager
Vous pouvez activer le chiffrement de disque sur des machines virtuelles Windows IaaS existantes ou en cours d’exécution dans Azure en utilisant le [modèle Resource Manager pour chiffrer une machine virtuelle Windows en cours d’exécution](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm).


1. Dans le modèle de démarrage rapide Azure, cliquez sur **Déployer sur Azure** .

2. Sélectionnez l’abonnement, le groupe de ressources, l’emplacement du groupe de ressources, les paramètres, les conditions légales et le contrat. Cliquez sur **Acheter** pour activer le chiffrement sur la machine virtuelle IaaS existante ou en cours d’exécution.

Le tableau suivant répertorie les paramètres du modèle Resource Manager pour les machines virtuelles existantes ou en cours d’exécution qui utilisent un ID de client Azure AD :

| Paramètre | Description |
| --- | --- |
| AADClientID | ID de client de l’application Azure AD qui dispose des autorisations pour écrire des clés secrètes dans le coffre de clés. |
| AADClientSecret | Clé secrète de client de l’application Azure AD qui dispose des autorisations pour écrire des clés secrètes dans le coffre de clés. |
| keyVaultName | Nom du coffre de clés dans lequel la clé BitLocker doit être téléchargée. Vous pouvez l’obtenir avec la cmdlet `(Get-AzKeyVault -ResourceGroupName <MyKeyVaultResourceGroupName>). Vaultname` ou la commande Azure CLI `az keyvault list --resource-group "MySecureGroup"`|
|  keyEncryptionKeyURL | URL de la clé de chiffrement à clé utilisée pour chiffrer la clé BitLocker générée. Ce paramètre est facultatif si vous sélectionnez **nokek** dans la liste déroulante UseExistingKek. Si vous sélectionnez **kek** dans la liste déroulante UseExistingKek, vous devez entrer la valeur _keyEncryptionKeyURL_ . |
| volumeType | Type de volume sur lequel l’opération de chiffrement est effectuée. Les valeurs valides sont _Système d’exploitation_ , _Données_ et _Tous_ . |
| sequenceVersion | Version de séquence de l’opération BitLocker. Incrémentez ce numéro de version à chaque fois qu’une opération de chiffrement de disque est exécutée sur la même machine virtuelle. |
| vmName | Nom de la machine virtuelle sur laquelle l’opération de chiffrement doit être effectuée. |


## <a name="new-iaas-vms-created-from-customer-encrypted-vhd-and-encryption-keys"></a><a name="bkmk_VHDpre"> </a> Nouvelles machines virtuelles IaaS créées à partir d’un disque dur virtuel chiffré par le client et de clés de chiffrement
Dans ce scénario, vous pouvez activer le chiffrement à l’aide du modèle Resource Manager, des applets de commande PowerShell ou des commandes CLI. Les sections ci-dessous décrivent de façon plus détaillée le modèle Resource Manager et les commandes CLI. 

Utilisez les instructions de l’annexe pour la préparer d’images préchiffrées qui peuvent être utilisées dans Azure. Une fois l’image créée, vous pouvez suivre la procédure décrite dans la section suivante pour créer une machine virtuelle Azure chiffrée.

* [Préparer un disque dur virtuel Windows déjà chiffré](disk-encryption-sample-scripts.md#prepare-a-pre-encrypted-windows-vhd)


### <a name="encrypt-vms-with-pre-encrypted-vhds-with-azure-powershell"></a><a name="bkmk_VHDprePSH"> </a> Chiffrer des machines virtuelles avec des disques durs virtuels préchiffrés à l’aide d’Azure PowerShell
Vous pouvez activer le chiffrement de disque sur votre disque dur virtuel chiffré avec la cmdlet PowerShell [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk#examples). L’exemple ci-dessous vous montre certains paramètres courants. 

```powershell
$VirtualMachine = New-AzVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Windows -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myKVresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzVM -VM $VirtualMachine -ResourceGroupName "MyVirtualMachineResourceGroup"
```

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>Activer le chiffrement sur un disque de données nouvellement ajouté
Vous pouvez [ajouter un nouveau disque à une machine virtuelle Windows avec PowerShell](attach-disk-ps.md) ou via le [portail Azure](attach-managed-disk-portal.md). 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>Activer le chiffrement sur un disque nouvellement ajouté avec Azure PowerShell
 Quand vous utilisez PowerShell pour chiffrer un nouveau disque de machines virtuelles Windows, vous devez spécifier une nouvelle version de la séquence. La version de la séquence doit être unique. Le script ci-dessous génère un GUID pour la version de la séquence. Dans certains cas, un disque de données nouvellement ajouté peut être chiffré automatiquement par l’extension Azure Disk Encryption. Le chiffrement automatique se produit généralement quand la machine virtuelle redémarre après la mise en ligne du nouveau disque. Cela est généralement dû au fait que « Tous » était spécifié pour le type de volume quand le chiffrement de disque a été précédemment exécuté sur la machine virtuelle. Si le chiffrement automatique se produit sur un disque de données nouvellement ajouté, nous recommandons de réexécuter l’applet de commande Set-AzVmDiskEncryptionExtension avec la nouvelle version de la séquence. Si votre nouveau disque de données est automatiquement chiffré et que vous ne voulez pas qu’il le soit, déchiffrez d’abord tous les lecteurs, puis chiffrez-les à nouveau avec une nouvelle version de séquence spécifiant le système d’exploitation pour le type de volume. 
 

-  **Chiffrer une machine virtuelle en cours d’exécution avec un secret client :** Le script ci-dessous initialise vos variables et exécute la cmdlet Set-AzVMDiskEncryptionExtension. Les prérequis sont que le groupe de ressources, la machine virtuelle, le coffre de clés, l’application AAD et le secret client doivent déjà avoir été créés. Remplacez MyKeyVaultResourceGroup, MyVirtualMachineResourceGroup, MySecureVM, MySecureVault, My-AAD-client-ID et My-AAD-client-secret par vos valeurs. Cet exemple utilise « All » pour le paramètre VolumeType, qui inclut les volumes de données et de systèmes d’exploitation. Si vous souhaitez uniquement chiffrer le volume de systèmes d’exploitation, utilisez « OS » pour le paramètre VolumeType. 

     ```azurepowershell
      $sequenceVersion = [Guid]::NewGuid();
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $aadClientID = 'My-AAD-client-ID';
      $aadClientSecret = 'My-AAD-client-secret';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'all' –SequenceVersion $sequenceVersion;
    ```
- **Chiffrer une machine virtuelle en cours d’exécution avec une clé KEK pour wrapper le secret client :** Azure Disk Encryption vous permet de spécifier une clé existante de votre coffre de clés pour inclure dans un wrapper les secrets de chiffrement de disque qui ont été générés lors de l’activation du chiffrement. Quand une clé de chiffrement principale est spécifiée, Azure Disk Encryption utilise cette clé pour wrapper les secrets de chiffrement avant d’écrire dans Key Vault. Cet exemple utilise « All » pour le paramètre VolumeType, qui inclut les volumes de données et de systèmes d’exploitation. Si vous souhaitez uniquement chiffrer le volume de systèmes d’exploitation, utilisez « OS » pour le paramètre VolumeType. 

     ```azurepowershell
     $sequenceVersion = [Guid]::NewGuid();
     $KVRGname = 'MyKeyVaultResourceGroup';
     $VMRGName = 'MyVirtualMachineResourceGroup';
     $vmName = 'MyExtraSecureVM';
     $aadClientID = 'My-AAD-client-ID';
     $aadClientSecret = 'My-AAD-client-secret';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'all' –SequenceVersion $sequenceVersion;

     ```

    >[!NOTE]
    > La syntaxe de la valeur du paramètre disk-encryption-keyvault est la chaîne d’identificateur complet : /subscriptions/[GUID_ID_abonnement]/resourceGroups/[nom_groupe_ressources]/providers/Microsoft.KeyVault/vaults/[nom_coffre_clés]</br> La syntaxe de la valeur du paramètre key-encryption-key est l’URI complet de la clé KEK comme dans : https://[nom_coffre_clés].vault.azure.net/keys/[nom_clé_kek]/[ID_unique_clé_kek] 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-cli"></a>Activer le chiffrement sur un disque nouvellement ajouté avec Azure CLI
  La commande Azure CLI vous propose automatiquement une nouvelle version de la séquence quand vous exécutez la commande pour activer le chiffrement. Les valeurs acceptables pour le paramètre volume-type sont « All », « OS » et « Data ». Vous devrez peut-être remplacer le paramètre volume-type du système d’exploitation par « OS » ou « Data » si vous ne chiffrez qu’un seul type de disque pour la machine virtuelle. Les exemples utilisent « All » pour le paramètre volume-type. 

-  **Chiffrer une machine virtuelle en cours d’exécution avec un secret client :**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type "All"
     ```

- **Chiffrer une machine virtuelle en cours d’exécution avec une clé KEK pour wrapper le secret client :**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "all"
     ```


## <a name="enable-encryption-using-azure-ad-client-certificate-based-authentication"></a>Activez le chiffrement avec une authentification basée sur les certificats clients Azure AD.
Vous pouvez utiliser l’authentification par certificat client avec ou sans clé KEK. Avant d’utiliser les scripts PowerShell, vous devez déjà disposer du certificat chargé dans le coffre de clés et déployé sur la machine virtuelle. De même, si vous utilisez une clé KEK, elle doit déjà exister. Pour plus d’informations, consultez la section [Authentification basée sur les certificats pour Azure AD](disk-encryption-key-vault-aad.md#certificate-based-authentication-optional) de l’article sur les prérequis.


### <a name="enable-encryption-using-certificate-based-authentication-with-azure-powershell"></a>Activer le chiffrement en utilisant l’authentification basée sur les certificats avec Azure PowerShell

```powershell
## Fill in 'MyVirtualMachineResourceGroup', 'MyKeyVaultResourceGroup', 'My-AAD-client-ID', 'MySecureVault, and ‘MySecureVM’.

$VMRGName = 'MyVirtualMachineResourceGroup'
$KVRGname = 'MyKeyVaultResourceGroup';
$AADClientID ='My-AAD-client-ID';
$KeyVaultName = 'MySecureVault';
$VMName = 'MySecureVM';
$KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
$KeyVaultResourceId = $KeyVault.ResourceId;

# Fill in the certificate path and the password so the thumbprint can be set as a variable. 

$certPath = '$CertPath = "C:\certificates\mycert.pfx';
$CertPassword ='Password'
$Cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($CertPath, $CertPassword)
$aadClientCertThumbprint = $cert.Thumbprint;

# Enable disk encryption using the client certificate thumbprint

Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $VMName -AadClientID $AADClientID -AadClientCertThumbprint $AADClientCertThumbprint -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId
```
  
### <a name="enable-encryption-using-certificate-based-authentication-and-a-kek-with-azure-powershell"></a>Activer le chiffrement en utilisant l’authentification basée sur les certificats et une clé KEK avec Azure PowerShell

```powershell
# Fill in 'MyVirtualMachineResourceGroup', 'MyKeyVaultResourceGroup', 'My-AAD-client-ID', 'MySecureVault,, 'MySecureVM’, and "KEKName.

$VMRGName = 'MyVirtualMachineResourceGroup';
$KVRGname = 'MyKeyVaultResourceGroup';
$AADClientID ='My-AAD-client-ID';
$KeyVaultName = 'MySecureVault';
$VMName = 'MySecureVM';
$keyEncryptionKeyName ='KEKName';
$KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
$KeyVaultResourceId = $KeyVault.ResourceId;
$keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

## Fill in the certificate path and the password so the thumbprint can be read and set as a variable.

$certPath = '$CertPath = "C:\certificates\mycert.pfx';
$CertPassword ='Password'
$Cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($CertPath, $CertPassword)
$aadClientCertThumbprint = $cert.Thumbprint;

# Enable disk encryption using the client certificate thumbprint and a KEK

Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $VMName -AadClientID $AADClientID -AadClientCertThumbprint $AADClientCertThumbprint -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId
```

## <a name="disable-encryption"></a>Désactiver le chiffrement
Vous pouvez désactiver le chiffrement avec Azure PowerShell, Azure CLI ou un modèle Resource Manager. 

- **Désactiver le chiffrement de disque avec Azure PowerShell :** pour désactiver le chiffrement, utilisez l’applet de commande [Disable-AzureRmVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). 
     ```azurepowershell-interactive
     Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```

- **Désactiver le chiffrement avec Azure CLI :** pour désactiver le chiffrement, utilisez la commande [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). 
     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type [ALL, DATA, OS]
     ```
- **Désactiver le chiffrement avec un modèle Resource Manager :** 

    1. Cliquez sur **Déployer sur Azure** dans le modèle [Désactiver le chiffrement de disque sur une machine virtuelle Windows en cours d’exécution](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm).
    2. Sélectionnez l’abonnement, le groupe de ressources, l’emplacement, les conditions juridiques et le contrat.
    3.  Cliquez sur **Acheter** pour désactiver le chiffrement de disque sur une machine virtuelle Windows en cours d’exécution. 

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Vue d’ensemble d’Azure Disk Encryption](disk-encryption-overview.md)
