---
title: Azure Disk Encryption avec Azure AD App pour les machines virtuelles IaaS Linux (version précédente)
description: Cet article fournit des instructions sur l’activation Microsoft Azure Disk Encryption pour les machines virtuelles IaaS Linux.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 03/15/2019
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: c8228086eb67478d80aa041004e0da3eed71f896
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92741793"
---
# <a name="enable-azure-disk-encryption-with-azure-ad-on-linux-vms-previous-release"></a>Activer Azure Disk Encryption avec Azure AD sur des machines virtuelles Linux (version précédente)

La nouvelle version d’Azure Disk Encryption évite d’avoir à indiquer un paramètre d’application Azure Active Directory (Azure AD) pour activer le chiffrement de disque de machine virtuelle. Avec la nouvelle version, vous n'êtes plus obligé de fournir les informations d'identification Azure AD lors de l'étape d’activation du chiffrement. À l’aide de la nouvelle version, toutes les nouvelles machines virtuelles doivent être chiffrées sans les paramètres d’application Azure AD. Pour obtenir des instructions sur l’activation du chiffrement de disque de machine virtuelle à l’aide de la nouvelle version, consultez [Azure Disk Encryption pour les machines virtuelles Linux](disk-encryption-linux.md). Les machines virtuelles déjà chiffrées avec les paramètres d’application Azure AD sont toujours prises en charge et doivent continuer à être gérées avec la syntaxe AAD.

Il existe de nombreux scénarios permettant d’activer le chiffrement de disque, et les étapes peuvent varier selon le scénario. Les sections suivantes décrivent ces scénarios de façon plus détaillée pour les machines virtuelles IaaS (infrastructure as a service) Linux. Vous pouvez appliquer le chiffrement de disque uniquement aux machines virtuelles dont [la taille et le système d’exploitation sont pris en charge](disk-encryption-overview.md#supported-vms-and-operating-systems). Vous devez également satisfaire les prérequis suivants :

- [Conditions supplémentaires pour les machines virtuelles](disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Mise en réseau et stratégie de groupe](disk-encryption-overview-aad.md#networking-and-group-policy)
- [Exigences liées au stockage des clés de chiffrement](disk-encryption-overview-aad.md#encryption-key-storage-requirements)

Prenez un [instantané](snapshot-copy-managed-disk.md), effectuez une sauvegarde ou faites les deux avant de chiffrer les disques. Les sauvegardes vous garantissent une possibilité de récupération en cas de défaillance inattendue lors du chiffrement. Les machines virtuelles avec des disques managés imposent une sauvegarde avant que le chiffrement soit effectué. Une fois la sauvegarde effectuée, vous pouvez utiliser l’applet de commande Set-AzVMDiskEncryptionExtension pour chiffrer des disques managés en spécifiant le paramètre -skipVmBackup. Pour plus d’informations sur la sauvegarde et la restauration de machines virtuelles chiffrées, consultez [Sauvegarde Azure](../../backup/backup-azure-vms-encryption.md). 

>[!WARNING]
 > - Si vous avez déjà utilisé [Azure Disk Encryption avec l’application Azure AD](disk-encryption-overview-aad.md) pour chiffrer cette machine virtuelle, vous devez continuer à utiliser cette option pour chiffrer votre machine virtuelle. Vous ne pouvez pas utiliser [Azure Disk Encryption](disk-encryption-overview.md) sur cette machine virtuelle chiffrée, car un tel scénario n’est pas pris en charge. Autrement dit, l’utilisation d’une application autre qu’Azure AD pour cette machine virtuelle chiffrée n’est pas encore prise en charge.
 > - Pour garantir que les secrets de chiffrement ne franchissent pas les limites régionales, Azure Disk Encryption exige que le coffre de clés se trouve dans la même région que les machines virtuelles. Créez et utilisez un coffre de clés situé dans la même région que la machine virtuelle à chiffrer.
 > - Lorsque vous chiffrez des volumes de système d’exploitation Linux, le processus peut prendre quelques heures. Il est normal que le chiffrement des volumes de système d’exploitation Linux prenne plus de temps que pour les volumes de données.
> - Lorsque vous chiffrez des volumes de système d’exploitation Linux, la machine virtuelle doit être considérée comme non disponible. Nous vous recommandons vivement d’éviter les connexions SSH pendant le chiffrement afin d’éviter tout problème risquant de bloquer les fichiers ouverts qui doivent être accessibles pendant le processus de chiffrement. Pour vérifier la progression, utilisez la commande [Get-AzVMDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus) ou [vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). Vous pouvez vous attendre à ce que ce processus prenne quelques heures pour un volume de système d’exploitation de 30 Go, ainsi qu’un temps supplémentaire pour le chiffrement des volumes de données. La durée du chiffrement des volumes de données est proportionnelle à la taille et à la quantité des volumes de données, sauf si l’option **encrypt format all** est utilisée. 
 > - La désactivation du chiffrement sur les machines virtuelles Linux est prise en charge seulement pour les volumes de données. Elle n’est pas prise en charge sur les volumes de données ou de système d’exploitation si le volume du système d’exploitation a été chiffré. 

 

## <a name="enable-encryption-on-an-existing-or-running-iaas-linux-vm"></a><a name="bkmk_RunningLinux"> </a> Activer le chiffrement sur une machine virtuelle IaaS Linux existante ou en cours d'exécution

Dans ce scénario, vous pouvez activer le chiffrement en utilisant le modèle Azure Resource Manager, des applets de commande PowerShell ou des commandes Azure CLI. 

>[!IMPORTANT]
 >Il est impératif de réaliser un instantané ou de sauvegarder une instance de machine virtuelle basée sur un disque managé en dehors d’Azure Disk Encryption et avant de l’activer. Vous pouvez prendre un instantané du disque managé à partir du Portail Azure, ou vous pouvez utiliser [Sauvegarde Azure](../../backup/backup-azure-vms-encryption.md). Les sauvegardes vous garantissent une possibilité de récupération en cas de défaillance inattendue pendant le chiffrement. Une fois la sauvegarde effectuée, utilisez l’applet de commande Set-AzVMDiskEncryptionExtension pour chiffrer des disques managés en spécifiant le paramètre -skipVmBackup. La commande Set-AzVMDiskEncryptionExtension échoue sur les machines virtuelles basées sur des disques managés tant qu’une sauvegarde n’a pas été effectuée et que ce paramètre n’a pas été spécifié. 
>
>Le chiffrement ou la désactivation du chiffrement peut entraîner le redémarrage de la machine virtuelle. 
>

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-by-using-the-azure-cli"></a><a name="bkmk_RunningLinuxCLI"> </a>Activer le chiffrement sur une machine virtuelle Linux existante ou en cours d'exécution à l'aide de l'interface de ligne de commande Azure 
Vous pouvez activer le chiffrement de disque sur votre disque dur virtuel chiffré en installant et en utilisant l’outil en ligne de commande [Azure CLI 2.0](/cli/azure). Vous pouvez l’ouvrir dans le navigateur avec [Azure Cloud Shell](../../cloud-shell/overview.md), ou vous pouvez l’installer sur votre ordinateur local et l’utiliser dans une session PowerShell. Pour activer le chiffrement sur des machines virtuelles IaaS Linux existantes ou en cours d’exécution dans Azure, utilisez les commandes CLI suivantes :

Utilisez la commande [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) pour activer le chiffrement sur une machine virtuelle IaaS en cours d’exécution dans Azure.

-  **Chiffrer une machine virtuelle en cours d’exécution à l’aide d’une clé secrète client :**
    
     ```azurecli-interactive
         az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
     ```

- **Chiffrer une machine virtuelle en cours d’exécution à l’aide d’une clé KEK pour inclure la clé secrète client dans un wrapper :**
    
     ```azurecli-interactive
         az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

   >[!NOTE]
   > La syntaxe pour la valeur du paramètre disk-encryption-keyvault est la chaîne complète de l’identificateur : /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name].</br> </br> La syntaxe pour la valeur du paramètre key-encryption-key est l’URI complet de la clé KEK comme dans : https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id].

- **Vérifier que les disques sont chiffrés :** Pour vérifier l’état du chiffrement d’une machine virtuelle IaaS, utilisez la commande [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). 

     ```azurecli-interactive
         az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
     ```

- **Désactiver le chiffrement :** pour désactiver le chiffrement, utilisez la commande [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). La désactivation du chiffrement est autorisée seulement sur les volumes de données pour les machines virtuelles Linux.
    
     ```azurecli-interactive
         az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type DATA
     ```

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-by-using-powershell"></a><a name="bkmk_RunningLinuxPSH"> </a> Activer le chiffrement sur une machine virtuelle Linux existante ou en cours d'exécution à l'aide de PowerShell
Utilisez la cmdlet [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) pour activer le chiffrement sur une machine virtuelle IaaS en cours d’exécution dans Azure. Prenez un [instantané](snapshot-copy-managed-disk.md) ou effectuez une sauvegarde de la machine virtuelle avec [Sauvegarde Azure](../../backup/backup-azure-vms-encryption.md) avant de chiffrer les disques. Le paramètre -skipVmBackup est déjà spécifié dans les scripts PowerShell pour chiffrer une machine virtuelle Linux en cours d'exécution.

- **Chiffrer une machine virtuelle en cours d’exécution à l’aide d’une clé secrète client :** Le script suivant initialise vos variables et exécute la cmdlet Set-AzVMDiskEncryptionExtension. Les prérequis sont que le groupe de ressources, la machine virtuelle, le coffre de clés, l’application Azure AD et la clé secrète client doivent déjà avoir été créés. Remplacez MyVirtualMachineResourceGroup, MyKeyVaultResourceGroup, MySecureVM, MySecureVault, My-AAD-client-ID et My-AAD-client-secret par vos valeurs. Modifiez le paramètre -VolumeType pour spécifier les disques que vous chiffrez.

     ```azurepowershell
         $VMRGName = 'MyVirtualMachineResourceGroup';
         $KVRGname = 'MyKeyVaultResourceGroup';
         $vmName = 'MySecureVM';
         $aadClientID = 'My-AAD-client-ID';
         $aadClientSecret = 'My-AAD-client-secret';
         $KeyVaultName = 'MySecureVault';
         $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
         $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
         $KeyVaultResourceId = $KeyVault.ResourceId;
         $sequenceVersion = [Guid]::NewGuid();
    
         Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType '[All|OS|Data]' -SequenceVersion $sequenceVersion -skipVmBackup;
     ```
- **Chiffrer une machine virtuelle en cours d’exécution à l’aide d’une clé KEK pour inclure la clé secrète client dans un wrapper :** Azure Disk Encryption vous permet de spécifier une clé existante de votre coffre de clés pour inclure dans un wrapper les secrets de chiffrement de disque qui ont été générés lors de l’activation du chiffrement. Quand une clé de chiffrement principale est spécifiée, Azure Disk Encryption utilise cette clé pour inclure dans un wrapper les secrets de chiffrement avant d’écrire dans le coffre de clés. Modifiez le paramètre -VolumeType pour spécifier les disques que vous chiffrez. 

     ```azurepowershell
         $KVRGname = 'MyKeyVaultResourceGroup';
         $VMRGName = 'MyVirtualMachineResourceGroup';
         $aadClientID = 'My-AAD-client-ID';
         $aadClientSecret = 'My-AAD-client-secret';
         $KeyVaultName = 'MySecureVault';
         $keyEncryptionKeyName = 'MyKeyEncryptionKey';
         $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
         $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
         $KeyVaultResourceId = $KeyVault.ResourceId;
         $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;
         $sequenceVersion = [Guid]::NewGuid();
    
         Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType '[All|OS|Data]' -SequenceVersion $sequenceVersion -skipVmBackup;
     ```

  >[!NOTE]
  > La syntaxe pour la valeur du paramètre disk-encryption-keyvault est la chaîne complète de l’identificateur : /subscriptions/[GUID_ID_abonnement]/resourceGroups/[nom_groupe_ressourcesKV]/providers/Microsoft.KeyVault/vaults/[nom_coffre_clés].</br> </br>
  La syntaxe pour la valeur du paramètre key-encryption-key est l’URI complet de la clé KEK comme dans : https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id]. 
    
- **Vérifier que les disques sont chiffrés :** Pour vérifier l’état du chiffrement d’une machine virtuelle IaaS, utilisez la cmdlet [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus). 
    
     ```azurepowershell-interactive 
         Get-AzVmDiskEncryptionStatus -ResourceGroupName MyVirtualMachineResourceGroup -VMName MySecureVM
     ```
    
- **Désactiver le chiffrement de disque :** pour désactiver le chiffrement, utilisez l’applet de commande [Disable-AzureRmVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). La désactivation du chiffrement est autorisée seulement sur les volumes de données pour les machines virtuelles Linux.
     
     ```azurepowershell-interactive 
         Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```


### <a name="enable-encryption-on-an-existing-or-running-iaas-linux-vm-with-a-template"></a><a name="bkmk_RunningLinux"> </a> Activer le chiffrement sur une machine virtuelle IaaS Linux existante ou en cours d'exécution à l'aide d'un modèle

Vous pouvez activer le chiffrement de disque sur une machine virtuelle IaaS Linux existante ou en cours d’exécution dans Azure à l’aide du [modèle Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm).

1. Sélectionnez **Déployer sur Azure** sur le modèle de démarrage rapide Azure.

2. Sélectionnez l’abonnement, le groupe de ressources, l’emplacement du groupe de ressources, les paramètres, les conditions légales et le contrat. Sélectionnez **Créer** pour activer le chiffrement sur la machine virtuelle IaaS existante ou en cours d’exécution.

Le tableau suivant répertorie les paramètres du modèle Resource Manager pour les machines virtuelles existantes ou en cours d’exécution qui utilisent un ID de client Azure AD :

| Paramètre | Description |
| --- | --- |
| AADClientID | ID de client de l’application Azure AD qui dispose des autorisations pour écrire des clés secrètes dans le coffre de clés. |
| AADClientSecret | Clé secrète de client de l’application Azure AD qui dispose des autorisations pour écrire des clés secrètes dans le coffre de clés. |
| keyVaultName | Nom du coffre de clés où la clé doit être chargée. Vous pouvez l’obtenir avec l’applet de commande `az keyvault show --name "MySecureVault" --query KVresourceGroup`. |
|  keyEncryptionKeyURL | URL de la clé de chiffrement principale utilisée pour chiffrer la clé générée. Ce paramètre est facultatif si vous sélectionnez **nokek** dans la liste déroulante **UseExistingKek** . Si vous sélectionnez **kek** dans la liste déroulante **UseExistingKek** , vous devez entrer la valeur _keyEncryptionKeyURL_ . |
| volumeType | Type de volume sur lequel l’opération de chiffrement est effectuée. Les valeurs valides prises en charge sont _OS_ ou _All_ . (Voir les distributions Linux et leurs versions prises en charge pour les disques de système d’exploitation et de données plus haut dans la section des prérequis.) |
| sequenceVersion | Version de séquence de l’opération BitLocker. Incrémentez ce numéro de version à chaque fois qu’une opération de chiffrement de disque est exécutée sur la même machine virtuelle. |
| vmName | Nom de la machine virtuelle sur laquelle l’opération de chiffrement doit être effectuée. |
| phrase secrète | Saisissez une phrase secrète forte comme clé de chiffrement de données. |



## <a name="use-the-encryptformatall-feature-for-data-disks-on-linux-iaas-vms"></a><a name="bkmk_EFA"> </a>Utiliser la fonctionnalité EncryptFormatAll pour les disques de données sur les machines virtuelles IaaS Linux
Le paramètre EncryptFormatAll réduit le temps de chiffrement des disques de données Linux. Les partitions répondant à certains critères sont formatées (avec leur système de fichiers actuel). Ensuite, elles sont remontées là où elles se trouvaient avant l’exécution de la commande. Si vous voulez exclure un disque de données répondant aux critères, vous pouvez le démonter avant d’exécuter la commande.

 Après l’exécution de cette commande, tous les lecteurs qui ont été montés précédemment sont formatés. La couche de chiffrement démarre ensuite sur le lecteur qui est maintenant vide. Quand cette option est activée, le disque temporaire attaché à la machine virtuelle est également chiffré. Si le disque éphémère est réinitialisé, il est reformaté et rechiffré pour la machine virtuelle par la solution Azure Disk Encryption dès que l’occasion s’en présente.

>[!WARNING]
> EncryptFormatAll ne doit pas être utilisé quand des données indispensables se trouvent sur les volumes de données d’une machine virtuelle. Vous pouvez exclure des disques du chiffrement en les démontant. Essayez d’abord le paramètre EncryptFormatAll sur une machine virtuelle de test afin de comprendre le paramètre de la fonctionnalité et son implication avant de l’essayer sur la machine virtuelle de production. L’option EncryptFormatAll formate le disque de données, de sorte que toutes les données seront perdues. Avant de continuer, vérifiez que tous les disques que vous souhaitez exclure sont correctement démontés. </br></br>
 >Si vous définissez ce paramètre lors de la mise à jour des paramètres de chiffrement, cela peut entraîner un redémarrage avant le chiffrement proprement dit. Dans ce cas, vous devez aussi supprimer du fichier fstab le disque que vous ne voulez pas formater. De même, vous devez ajouter la partition que vous voulez chiffrer-formater au fichier fstab avant de lancer l’opération de chiffrement. 

### <a name="encryptformatall-criteria"></a><a name="bkmk_EFACriteria"> </a> Critères pour EncryptFormatAll
Ce paramètre parcourt toutes les partitions et les chiffre, à condition qu’elles répondent à *tous* les critères suivants : 
- Elle n’est pas une partition root/de système d’exploitation/de démarrage
- Elle n’est pas déjà chiffrée
- Elle n’est pas un volume BEK
- Elle n’est pas un volume RAID
- Elle n’est pas un volume LVM
- Elle est montée

Chiffrez les disques composant le volume RAID ou LVM au lieu de chiffrer le volume RAID ou LVM lui-même.

### <a name="use-the-encryptformatall-parameter-with-a-template"></a><a name="bkmk_EFATemplate"> </a> Utiliser le paramètre EncryptFormatAll avec un modèle
Pour utiliser l’option EncryptFormatAll, utilisez un modèle Azure Resource Manager préexistant qui chiffre une machine virtuelle Linux et modifiez le champ **EncryptionOperation** pour la ressource AzureDiskEncryption.

1. Par exemple, utilisez le [modèle Resource Manager pour chiffrer une machine virtuelle IaaS Linux en cours d’exécution](https://github.com/vermashi/azure-quickstart-templates/tree/encrypt-format-running-linux-vm/201-encrypt-running-linux-vm). 
2. Sélectionnez **Déployer sur Azure** sur le modèle de démarrage rapide Azure.
3. Changez le champ **EncryptionOperation** en remplaçant **EnableEncryption** par **EnableEncryptionFormatAl** .
4. Sélectionnez l’abonnement, le groupe de ressources, l’emplacement du groupe de ressources, les autres paramètres, les conditions légales et le contrat. Sélectionnez **Créer** pour activer le chiffrement sur la machine virtuelle IaaS existante ou en cours d’exécution.


### <a name="use-the-encryptformatall-parameter-with-a-powershell-cmdlet"></a><a name="bkmk_EFAPSH"> </a> Utiliser le paramètre EncryptFormatAll avec une cmdlet PowerShell
Utilisez l’applet de commande [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) avec le paramètre EncryptFormatAll.

**Chiffrer une machine virtuelle en cours d’exécution en utilisant une clé secrète client et EncryptFormatAll :** À titre d’exemple, le script suivant initialise vos variables et exécute l’applet de commande Set-AzVMDiskEncryptionExtension avec le paramètre EncryptFormatAll. Les prérequis sont que le groupe de ressources, la machine virtuelle, le coffre de clés, l’application Azure AD et la clé secrète client doivent déjà avoir été créés. Remplacez MyKeyVaultResourceGroup, MyVirtualMachineResourceGroup, MySecureVM, MySecureVault, My-AAD-client-ID et My-AAD-client-secret par vos valeurs.
  
   ```azurepowershell
     $KVRGname = 'MyKeyVaultResourceGroup';
     $VMRGName = 'MyVirtualMachineResourceGroup'; 
     $aadClientID = 'My-AAD-client-ID';
     $aadClientSecret = 'My-AAD-client-secret';
     $KeyVaultName = 'MySecureVault';
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
      
     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -EncryptFormatAll
   ```


### <a name="use-the-encryptformatall-parameter-with-logical-volume-manager-lvm"></a><a name="bkmk_EFALVM"> </a> Utiliser le paramètre EncryptFormatAll avec Logical Volume Manager (LVM) 
Nous recommandons une installation LVM-on-crypt. Pour tous les exemples suivants, remplacez le chemin d’accès de l’appareil et les points de montage par ce qui correspond à votre cas d’usage. Cette installation peut se faire comme suit :

- Ajoutez les disques de données qui constitueront la machine virtuelle.
- Formatez, montez et ajoutez ces disques au fichier fstab.

    1. Formatez le disque nouvellement ajouté. Nous utilisons ici des liens symboliques générés par Azure. L’utilisation de liens symboliques évite les problèmes liés à la modification des noms des périphériques. Pour plus d’informations, consultez [Résoudre les problèmes relatifs aux noms des appareils](../troubleshooting/troubleshoot-device-names-problems.md).
    
        ```console
        mkfs -t ext4 /dev/disk/azure/scsi1/lun0
        ```

    2. Montez les disques.

        ```console
        mount /dev/disk/azure/scsi1/lun0 /mnt/mountpoint
        ```

    3. Ajoutez-les à fstab.

        ```console
        echo "/dev/disk/azure/scsi1/lun0 /mnt/mountpoint ext4 defaults,nofail 1 2" >> /etc/fstab
        ```

    4. Exécutez l’applet de commande PowerShell Set-AzVMDiskEncryptionExtension avec -EncryptFormatAll pour chiffrer ces disques.

       ```azurepowershell-interactive
        Set-AzVMDiskEncryptionExtension -ResourceGroupName "MySecureGroup" -VMName "MySecureVM" -DiskEncryptionKeyVaultUrl "https://mykeyvault.vault.azure.net/" -EncryptFormatAll
        ```

    5. Configurez le gestionnaire de volumes logiques sur ces nouveaux disques. Notez que les lecteurs chiffrés sont déverrouillés une fois que la machine virtuelle a fini de démarrer. Ainsi, le montage du gestionnaire de volumes logiques devra également être retardé.




## <a name="new-iaas-vms-created-from-customer-encrypted-vhd-and-encryption-keys"></a><a name="bkmk_VHDpre"> </a> Nouvelles machines virtuelles IaaS créées à partir d'un disque dur virtuel chiffré par le client et de clés de chiffrement
Dans ce scénario, vous pouvez activer le chiffrement à l’aide du modèle Resource Manager, des applets de commande PowerShell ou des commandes CLI. Les sections ci-dessous décrivent de façon plus détaillée le modèle Resource Manager et les commandes CLI. 

Utilisez les instructions de l’annexe pour la préparer d’images préchiffrées qui peuvent être utilisées dans Azure. Une fois l’image créée, vous pouvez suivre la procédure décrite dans la section suivante pour créer une machine virtuelle Azure chiffrée.

* [Préparer un disque dur virtuel Linux déjà chiffré](disk-encryption-sample-scripts.md)

>[!IMPORTANT]
 >Il est impératif de réaliser un instantané ou de sauvegarder une instance de machine virtuelle basée sur un disque managé en dehors d’Azure Disk Encryption et avant de l’activer. Vous pouvez prendre un instantané du disque managé à partir du portail, ou vous pouvez utiliser [Sauvegarde Azure](../../backup/backup-azure-vms-encryption.md). Les sauvegardes vous garantissent une possibilité de récupération en cas de défaillance inattendue pendant le chiffrement. Une fois la sauvegarde effectuée, utilisez l’applet de commande Set-AzVMDiskEncryptionExtension pour chiffrer des disques managés en spécifiant le paramètre -skipVmBackup. La commande Set-AzVMDiskEncryptionExtension échoue sur les machines virtuelles basées sur des disques managés tant qu’une sauvegarde n’a pas été effectuée et que ce paramètre n’a pas été spécifié. 
>
>Le chiffrement ou la désactivation du chiffrement peut entraîner le redémarrage de la machine virtuelle.



### <a name="use-azure-powershell-to-encrypt-iaas-vms-with-pre-encrypted-vhds"></a><a name="bkmk_VHDprePSH"> </a> Utiliser Azure PowerShell pour chiffrer des machines virtuelles IaaS avec des disques durs virtuels préchiffrés 
Vous pouvez activer le chiffrement de disque sur votre disque dur virtuel chiffré avec la cmdlet PowerShell [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk#examples). L’exemple suivant vous montre certains paramètres communs. 

```powershell
$VirtualMachine = New-AzVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Windows -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzVM -VM $VirtualMachine -ResourceGroupName "MyVirtualMachineResourceGroup"
```

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>Activer le chiffrement sur un disque de données nouvellement ajouté
Vous pouvez ajouter un nouveau disque de données en utilisant [az vm disk attach](add-disk.md) ou [via le Portail Azure](attach-disk-portal.md). Avant de pouvoir chiffrer, vous devez d’abord monter le disque de données nouvellement attaché. Vous devez demander le chiffrement du lecteur de données, car le lecteur sera inutilisable pendant que le chiffrement est en cours. 

### <a name="enable-encryption-on-a-newly-added-disk-with-the-azure-cli"></a>Activer le chiffrement sur un disque nouvellement ajouté à l’aide d’Azure CLI
 Si la machine virtuelle a déjà été chiffrée avec « All », le paramètre --volume-type doit rester « All ». All inclut les disques de système d’exploitation et de données. Si la machine virtuelle a déjà été chiffrée avec le type de volume « OS », le paramètre --volume-type doit être changé en « All » afin que le système d’exploitation et le nouveau disque de données soient inclus. Si la machine virtuelle a été chiffrée avec uniquement le type de volume « Data », le paramètre « Data » peut être conservé comme illustré ici. L’ajout et l’attachement d’un nouveau disque de données à une machine virtuelle ne constituent pas une préparation suffisante pour le chiffrement. Le disque nouvellement attaché doit aussi être formaté et monté correctement au sein de la machine virtuelle avant que vous n’activiez le chiffrement. Sur Linux, le disque doit être monté dans /etc/fstab avec un [nom de périphérique de bloc persistant](../troubleshooting/troubleshoot-device-names-problems.md). 

Contrairement à la syntaxe de PowerShell, l’interface CLI ne nécessite pas que vous fournissiez une version de séquence unique lors de l’activation du chiffrement. L’interface CLI génère automatiquement et utilise sa propre valeur de version de séquence unique.

-  **Chiffrer une machine virtuelle en cours d’exécution à l’aide d’une clé secrète client :** 
    
     ```azurecli-interactive
         az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type "Data"
     ```

- **Chiffrer une machine virtuelle en cours d’exécution à l’aide d’une clé KEK pour inclure la clé secrète client dans un wrapper :**

     ```azurecli-interactive
         az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "Data"
     ```

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>Activer le chiffrement sur un disque nouvellement ajouté avec Azure PowerShell
 Quand vous utilisez PowerShell pour chiffrer un nouveau disque pour Linux, vous devez spécifier une nouvelle version de la séquence. La version de la séquence doit être unique. Le script suivant génère un GUID pour la version de la séquence. 
 

-  **Chiffrer une machine virtuelle en cours d’exécution à l’aide d’une clé secrète client :** Le script suivant initialise vos variables et exécute la cmdlet Set-AzVMDiskEncryptionExtension. Les prérequis sont que le groupe de ressources, la machine virtuelle, le coffre de clés, l’application Azure AD et la clé secrète client doivent déjà avoir été créés. Remplacez MyVirtualMachineResourceGroup, MyKeyVaultResourceGroup, MySecureVM, MySecureVault, My-AAD-client-ID et My-AAD-client-secret par vos valeurs. Le paramètre -VolumeType est défini pour les disques de données et non pas pour le disque du système d’exploitation. Si la machine virtuelle a déjà été chiffrée avec le type de volume « OS » ou « All », le paramètre -VolumeType doit être changé en « All » afin que le système d’exploitation et le nouveau disque de données soient inclus.

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
         $sequenceVersion = [Guid]::NewGuid();
    
         Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'data' –SequenceVersion $sequenceVersion;
     ```
- **Chiffrer une machine virtuelle en cours d’exécution à l’aide d’une clé KEK pour inclure la clé secrète client dans un wrapper :** Azure Disk Encryption vous permet de spécifier une clé existante de votre coffre de clés pour inclure dans un wrapper les secrets de chiffrement de disque qui ont été générés lors de l’activation du chiffrement. Quand une clé de chiffrement principale est spécifiée, Azure Disk Encryption utilise cette clé pour inclure dans un wrapper les secrets de chiffrement avant d’écrire dans le coffre de clés. Le paramètre -VolumeType est défini pour les disques de données et non pas pour le disque du système d’exploitation. Si la machine virtuelle a déjà été chiffrée avec le type de volume « OS » ou « All », le paramètre -VolumeType doit être changé en « All » afin que le système d’exploitation et le nouveau disque de données soient inclus.

     ```azurepowershell
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
         $sequenceVersion = [Guid]::NewGuid();
    
         Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'data' –SequenceVersion $sequenceVersion;
     ```


>[!NOTE]
> La syntaxe pour la valeur du paramètre disk-encryption-keyvault est la chaîne complète de l’identificateur : /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]. </br> </br>
La syntaxe pour la valeur du paramètre key-encryption-key est l’URI complet de la clé KEK comme dans : https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id].

## <a name="disable-encryption-for-linux-vms"></a>Désactiver le chiffrement pour les machines virtuelles Linux
Vous pouvez désactiver le chiffrement en utilisant Azure PowerShell, Azure CLI ou un modèle Resource Manager. 

>[!IMPORTANT]
>La désactivation du chiffrement avec Azure Disk Encryption sur les machines virtuelles Linux est prise en charge seulement pour les volumes de données. Elle n’est pas prise en charge sur les volumes de données ou de système d’exploitation si le volume du système d’exploitation a été chiffré. 

- **Désactiver le chiffrement de disque avec Azure PowerShell :** pour désactiver le chiffrement, utilisez l’applet de commande [Disable-AzureRmVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). 
     ```azurepowershell-interactive
         Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM' [--volume-type {ALL, DATA, OS}]
     ```

- **Désactiver le chiffrement avec Azure CLI :** pour désactiver le chiffrement, utilisez la commande [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). 
     ```azurecli-interactive
         az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type [ALL, DATA, OS]
     ```
- **Désactiver le chiffrement avec un modèle Resource Manager :** pour désactiver le chiffrement, utilisez le modèle [Désactiver le chiffrement sur une machine virtuelle Linux en cours d’exécution](https://aka.ms/decrypt-linuxvm).
     1. Sélectionnez **Déployer sur Azure** .
     2. Sélectionnez l’abonnement, le groupe de ressources, l’emplacement, les conditions juridiques et le contrat.
     3. Sélectionnez **Acheter** pour désactiver le chiffrement de disque sur une machine virtuelle Windows en cours d’exécution. 


## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble d’Azure Disk Encryption pour Linux](disk-encryption-overview-aad.md)
- [Création et configuration d’un coffre de clés pour Azure Disk Encryption avec Azure AD (version précédente)](disk-encryption-key-vault-aad.md)
