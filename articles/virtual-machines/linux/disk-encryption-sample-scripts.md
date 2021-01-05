---
title: Exemples de scripts Azure Disk Encryption
description: Cet article constitue l’annexe de Microsoft Azure Disk Encryption pour les machines virtuelles Linux.
author: msmbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: how-to
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18, devx-track-azurepowershell
ms.openlocfilehash: d178ae39d3af6b39047501f0bc47acbc6e792f48
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92911492"
---
# <a name="azure-disk-encryption-sample-scripts-for-linux-vms"></a>Exemples de scripts Azure Disk Encryption pour machines virtuelles Linux

Cet article fournit des exemples de scripts pour la préparation de disques durs virtuels préchiffrés et d’autres tâches.  

> [!NOTE]
> Sauf mention contraire, tous les scripts font référence à la dernière version hors AAD d’ADE.

## <a name="sample-powershell-scripts-for-azure-disk-encryption"></a>Exemples de scripts PowerShell pour Azure Disk Encryption 

- **Répertorier toutes les machines virtuelles chiffrées dans votre abonnement**
  
  Vous pouvez rechercher toutes les machines virtuelles chiffrées avec ADE et la version de l’extension, dans tous les groupes de ressources d’un abonnement, à l’aide de [ce script PowerShell](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/Find_1passAdeVersion_VM.ps1).

  À l’inverse, ces cmdlets affichent toutes les machines virtuelles chiffrées avec ADE (sans la version de l’extension) :

   ```azurepowershell-interactive
   $osVolEncrypted = {(Get-AzVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).OsVolumeEncrypted}
   $dataVolEncrypted= {(Get-AzVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).DataVolumesEncrypted}
   Get-AzVm | Format-Table @{Label="MachineName"; Expression={$_.Name}}, @{Label="OsVolumeEncrypted"; Expression=$osVolEncrypted}, @{Label="DataVolumesEncrypted"; Expression=$dataVolEncrypted}
   ```

- **Répertorier toutes les instances VMSS chiffrées de votre abonnement**
    
    Vous pouvez rechercher toutes les instances VMSS chiffrées avec ADE et la version de l’extension, dans tous les groupes de ressources d’un abonnement, à l’aide de [ce script PowerShell](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/Find_1passAdeVersion_VMSS.ps1).

- **Répertorier tous les secrets de chiffrement de disque utilisées pour chiffrer les machines virtuelles dans un coffre de clés** 

   ```azurepowershell-interactive
   Get-AzKeyVaultSecret -VaultName $KeyVaultName | where {$_.Tags.ContainsKey('DiskEncryptionKeyFileName')} | format-table @{Label="MachineName"; Expression={$_.Tags['MachineName']}}, @{Label="VolumeLetter"; Expression={$_.Tags['VolumeLetter']}}, @{Label="EncryptionKeyURL"; Expression={$_.Id}}
   ```

### <a name="using-the-azure-disk-encryption-prerequisites-powershell-script"></a>Utilisation du script PowerShell des prérequis Azure Disk Encryption
Si vous êtes déjà familiarisé avec les prérequis d’Azure Disk Encryption, vous pouvez utiliser le [script PowerShell des prérequis d’Azure Disk Encryption](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/AzureDiskEncryptionPreRequisiteSetup.ps1 ). Vous trouverez un exemple d’utilisation de ce script PowerShell dans [Démarrage rapide du chiffrement d’une machine virtuelle](disk-encryption-powershell-quickstart.md). Vous pouvez supprimer les commentaires d’une section du script, en commençant à la ligne 211, pour chiffrer tous les disques des machines virtuelles d’un groupe de ressources existant. 

Le tableau suivant présente les paramètres pouvant être utilisés dans le script PowerShell : 


|Paramètre|Description|Obligatoire ?|
|------|------|------|
|$resourceGroupName| Nom du groupe de ressources auquel appartient le coffre de clés.  Un groupe de ressources portant ce nom sera créé s’il n’existe pas encore.| True|
|$keyVaultName|Nom du coffre de clés dans lequel les clés de chiffrement doivent être placées. Un coffre portant ce nom sera créé s’il n’existe pas encore.| True|
|$location|Emplacement du coffre de clés. Assurez-vous que le coffre de clés et les machines virtuelles à chiffrer se trouvent au même emplacement. Obtenez une liste des emplacements avec `Get-AzLocation`.|True|
|$subscriptionId|Identificateur de l’abonnement Azure à utiliser.  Vous pouvez obtenir votre ID d’abonnement avec `Get-AzSubscription`.|True|
|$aadAppName|Nom de l’application Azure AD qui sera utilisée pour écrire des secrets dans le coffre de clés. S’il n’existe aucune application de ce nom, une nouvelle application sera créée en utilisant le nom que vous avez saisi. Si cette application existe déjà, appliquez le paramètre aadClientSecret au script.|False|
|$aadClientSecret|Clé secrète client de l’application Azure AD qui a été créée précédemment.|False|
|$keyEncryptionKeyName|Nom de la clé de chiffrement principale facultative dans KeyVault. Une clé portant ce nom sera créée si elle n’existe pas encore.|False|

### <a name="encrypt-or-decrypt-vms-without-an-azure-ad-app"></a>Chiffrer ou déchiffrer des machines virtuelles sans application Azure AD

- [Activer le chiffrement de disque sur une machine virtuelle Linux existante ou en cours d’exécution](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm-without-aad)  
- [Désactiver le chiffrement sur une machine virtuelle Linux en cours d’exécution](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-linux-vm-without-aad) 
    - La désactivation du chiffrement est autorisée seulement sur les volumes de données pour les machines virtuelles Linux.  

### <a name="encrypt-or-decrypt-vms-with-an-azure-ad-app-previous-release"></a>Chiffrer ou déchiffrer des machines virtuelles avec une application Azure AD (version précédente)
 
- [Activer le chiffrement de disque sur une machine virtuelle Linux existante ou en cours d’exécution](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm)    


-  [Désactiver le chiffrement sur une machine virtuelle Linux en cours d’exécution](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-linux-vm) 
    - La désactivation du chiffrement est autorisée seulement sur les volumes de données pour les machines virtuelles Linux. 


- [Créer un disque managé chiffré à partir d’un disque dur virtuel/objet blob de stockage préchiffré](https://github.com/Azure/azure-quickstart-templates/tree/master/201-create-encrypted-managed-disk)
    - Crée un disque managé chiffré fourni par un disque dur virtuel préchiffré et ses paramètres de chiffrement correspondants

## <a name="encrypting-an-os-drive-on-a-running-linux-vm"></a>Chiffrement d’un lecteur du système d’exploitation sur une machine virtuelle Linux en cours d’exécution

### <a name="prerequisites-for-os-disk-encryption"></a>Configuration requise pour le chiffrement du lecteur du système d’exploitation

* La machine virtuelle doit utiliser une distribution compatible avec le chiffrement de disque du système d’exploitation, comme indiqué dans [Systèmes d’exploitation pris en charge avec Azure Disk Encryption](disk-encryption-overview.md#supported-vms). 
* La machine virtuelle doit être créée à partir de l’image Marketplace dans Azure Resource Manager.
* Machine virtuelle Azure au moins 4 Go de RAM (la taille recommandée est de 7 Go).
* (Pour RHEL et CentOS) Désactivez SELinux. Pour désactiver SELinux, consultez la rubrique « 4.4.2. Désactivation de SELinux » dans le [Guide d’utilisation et d’administration SELinux](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SELinux_Users_and_Administrators_Guide/sect-Security-Enhanced_Linux-Working_with_SELinux-Changing_SELinux_Modes.html#sect-Security-Enhanced_Linux-Enabling_and_Disabling_SELinux-Disabling_SELinux) sur la machine virtuelle.
* Après avoir désactivé SELinux, redémarrez au moins une fois la machine virtuelle.

### <a name="steps"></a>Étapes
1. Créez une machine virtuelle en utilisant l’une des distributions spécifiées précédemment.

   Pour CentOS 7.2, le chiffrement du lecteur du système d’exploitation est pris en charge via une image spécifique. Pour utiliser cette image, spécifiez « 7.2n » en tant que référence lorsque vous créez la machine virtuelle :

   ```powershell
    Set-AzVMSourceImage -VM $VirtualMachine -PublisherName "OpenLogic" -Offer "CentOS" -Skus "7.2n" -Version "latest"
   ```
2. Configurez la machine virtuelle selon vos besoins. Si vous souhaitez chiffrer tous les lecteurs (de données et du système d’exploitation), les lecteurs de données doivent être spécifiés et montables à partir de /etc/fstab.

   > [!NOTE]
   > Utilisez UUID =... pour définir les lecteurs de données dans /etc/fstab au lieu de spécifier le nom de l’appareil de traitement par blocs (par exemple, /dev/sdb1). L’ordre des lecteurs sur la machine virtuelle est modifié au cours du chiffrement. Si votre machine virtuelle s’appuie sur un ordre spécifique d’appareils de traitement par blocs, leur montage échouera après le chiffrement.

3. Déconnectez-vous des sessions SSH.

4. Pour chiffrer le système d’exploitation, spécifiez **All**  ou **OS** pour VolumeType quand vous activez le chiffrement.

   > [!NOTE]
   > Tous les processus d’espace utilisateur qui ne s’exécutent pas en tant que services `systemd` doivent être arrêtés avec un `SIGKILL`. Redémarrez la machine virtuelle. Lorsque vous activez le chiffrement du lecteur du système d’exploitation sur une machine virtuelle en cours d’exécution, prévoyez un temps d’arrêt de la machine virtuelle.

5. Surveillez régulièrement la progression du chiffrement à l’aide des instructions fournies dans la [section suivante](#monitoring-os-encryption-progress).

6. Lorsque Get-AzVmDiskEncryptionStatus indique « VMRestartPending », redémarrez votre machine virtuelle en vous y connectant ou bien en utilisant le portail, PowerShell ou CLI.
    ```powershell
    C:\> Get-AzVmDiskEncryptionStatus  -ResourceGroupName $ResourceGroupName -VMName $VMName
    -ExtensionName $ExtensionName

    OsVolumeEncrypted          : VMRestartPending
    DataVolumesEncrypted       : NotMounted
    OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
    ProgressMessage            : OS disk successfully encrypted, reboot the VM
    ```
   Avant le redémarrage, nous vous recommandons d’enregistrer les [diagnostics de démarrage](https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/) de la machine virtuelle.

## <a name="monitoring-os-encryption-progress"></a>Surveillance de la progression du chiffrement du système d’exploitation
Il existe trois façons de surveiller la progression du chiffrement du système d’exploitation :

* Utilisez l’applet de commande `Get-AzVmDiskEncryptionStatus` et examinez le champ ProgressMessage :
    ```powershell
    OsVolumeEncrypted          : EncryptionInProgress
    DataVolumesEncrypted       : NotMounted
    OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
    ProgressMessage            : OS disk encryption started
    ```
  Dès que la machine virtuelle passe à l’état « Le chiffrement du lecteur du système d’exploitation a démarré », l’opération prend entre 40 et 50 minutes sur une machine virtuelle avec un stockage Premium.

  En raison de [l’erreur #388](https://github.com/Azure/WALinuxAgent/issues/388) dans WALinuxAgent, `OsVolumeEncrypted` et `DataVolumesEncrypted` apparaissent comme `Unknown` dans certaines distributions. Ce problème est résolu automatiquement dans WALinuxAgent version 2.1.5 et ultérieure. Si `Unknown` s’affiche dans la sortie, vous pouvez vérifier l’état du chiffrement du disque en utilisant l’Explorateur de ressources Azure.

  Accédez à [l’Explorateur de ressources Azure](https://resources.azure.com/), puis développez cette hiérarchie dans le panneau de sélection de gauche :

  ```
  |-- subscriptions
     |-- [Your subscription]
          |-- resourceGroups
               |-- [Your resource group]
                    |-- providers
                         |-- Microsoft.Compute
                              |-- virtualMachines
                                   |-- [Your virtual machine]
                                        |-- InstanceView
  ```                

  Faites défiler InstanceView pour afficher l’état de chiffrement de vos lecteurs.

  ![Vue d’instance de machine virtuelle](./media/disk-encryption/vm-instanceview.png)

* Recherchez les [diagnostics de démarrage](https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/). Les messages de l’extension ADE doivent être précédés du préfixe `[AzureDiskEncryption]`.

* Connectez-vous à la machine virtuelle via SSH et obtenez le journal des extensions à partir de :

    /var/log/azure/Microsoft.Azure.Security.AzureDiskEncryptionForLinux

  Nous vous déconseillons de vous connecter à la machine virtuelle lorsque le chiffrement du système d’exploitation est en cours. Copiez les journaux d’activité uniquement lorsque les deux autres méthodes ont échoué.

## <a name="prepare-a-pre-encrypted-linux-vhd"></a>Préparer un disque dur virtuel Linux déjà chiffré
La préparation des disques durs virtuels préchiffrés peut varier en fonction de la distribution. Des exemples sur la préparation d’Ubuntu 16, openSUSE 13.2 et CentOS 7 sont disponibles. 

### <a name="ubuntu-16"></a>Ubuntu 16
Configurez le chiffrement lors de l’installation de la distribution en procédant comme suit :

1. Sélectionnez **Configure encrypted volumes** (Configurer les volumes chiffrés) lors du partitionnement des disques.

   ![Configuration Ubuntu 16.04 - Configurer les volumes chiffrés](./media/disk-encryption/ubuntu-1604-preencrypted-fig1.png)

2. Créez un lecteur de démarrage séparé qui ne doit pas être chiffré. Chiffrez votre lecteur racine.

   ![Configuration Ubuntu 16.04 - Sélectionner les appareils à chiffrer](./media/disk-encryption/ubuntu-1604-preencrypted-fig2.png)

3. Indiquez une phrase secrète. Il s’agit de la phrase secrète que vous avez chargée dans le coffre de clés.

   ![Configuration Ubuntu 16.04 - Fournir la phrase secrète](./media/disk-encryption/ubuntu-1604-preencrypted-fig3.png)

4. Terminez le partitionnement.

   ![Configuration Ubuntu 16.04 - Terminer le partitionnement](./media/disk-encryption/ubuntu-1604-preencrypted-fig4.png)

5. Lorsque vous démarrez la machine virtuelle et devez fournir une phrase secrète, utilisez la phrase secrète que vous avez fournie à l’étape 3.

   ![Configuration Ubuntu 16.04 - Fournir la phrase secrète au démarrage](./media/disk-encryption/ubuntu-1604-preencrypted-fig5.png)

6. Préparez la machine virtuelle au chargement dans Azure en suivant [ces instructions](./create-upload-ubuntu.md?toc=/azure/virtual-machines/linux/toc.json). N’exécutez pas encore la dernière étape (annulation de l’approvisionnement de la machine virtuelle).

Configurez le chiffrement pour l’utiliser dans Azure en effectuant les étapes suivantes :

1. Créez un fichier sous /usr/local/sbin/azure_crypt_key.sh, avec le contenu du script ci-dessous. Prêtez une attention particulière à KeyFileName, car il s’agit du nom de fichier de phrase secrète utilisé par Azure.

    ```bash
    #!/bin/sh
    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint
    modprobe vfat >/dev/null 2>&1
    modprobe ntfs >/dev/null 2>&1
    sleep 2
    OPENED=0
    cd /sys/block
    for DEV in sd*; do

        echo "> Trying device: $DEV ..." >&2
        mount -t vfat -r /dev/${DEV}1 $MountPoint >/dev/null||
        mount -t ntfs -r /dev/${DEV}1 $MountPoint >/dev/null
        if [ -f $MountPoint/$KeyFileName ]; then
                cat $MountPoint/$KeyFileName
                umount $MountPoint 2>/dev/null
                OPENED=1
                break
        fi
        umount $MountPoint 2>/dev/null
    done

      if [ $OPENED -eq 0 ]; then
        echo "FAILED to find suitable passphrase file ..." >&2
        echo -n "Try to enter your password: " >&2
        read -s -r A </dev/console
        echo -n "$A"
     else
        echo "Success loading keyfile!" >&2
    fi
   ```

2. Modifiez la configuration du chiffrement dans */etc/crypttab*. Il doit se présenter comme suit :
   ```
    xxx_crypt uuid=xxxxxxxxxxxxxxxxxxxxx none luks,discard,keyscript=/usr/local/sbin/azure_crypt_key.sh
    ```

4. Ajoutez des autorisations exécutables au script :
   ```
    chmod +x /usr/local/sbin/azure_crypt_key.sh
   ```
5. Éditez */etc/initramfs-tools/modules* en ajoutant des lignes :
   ```
    vfat
    ntfs
    nls_cp437
    nls_utf8
    nls_iso8859-1
   ```
6. Exécutez `update-initramfs -u -k all` pour mettre à jour l’initramfs afin de mettre en vigueur le `keyscript`.

7. Vous pouvez maintenant annuler l’approvisionnement de la machine virtuelle.

   ![Configuration d’Ubuntu 16.04 - update-initramfs](./media/disk-encryption/ubuntu-1604-preencrypted-fig6.png)

8. Passez à l’étape suivante et chargez votre disque dur virtuel dans Azure.

### <a name="opensuse-132"></a>openSUSE 13.2
Pour configurer le chiffrement lors de l’installation de la distribution, effectuez les étapes suivantes :
1. Lorsque vous partitionnez les disques, sélectionnez **Chiffrer le groupe de volumes** , puis entrez un mot de passe. Il s’agit du mot de passe que vous allez charger dans votre coffre de clés.

   ![Configuration d’openSUSE 13.2 - Chiffrer un groupe de volumes](./media/disk-encryption/opensuse-encrypt-fig1.png)

2. Démarrez la machine virtuelle à l’aide de votre mot de passe.

   ![Configuration openSUSE 13.2 - Fournir la phrase secrète au démarrage](./media/disk-encryption/opensuse-encrypt-fig2.png)

3. Préparez la machine virtuelle pour le chargement dans Azure en suivant les instructions de la rubrique [Préparation d’une machine virtuelle SLES ou openSUSE pour Azure](./suse-create-upload-vhd.md?toc=/azure/virtual-machines/linux/toc.json#prepare-opensuse-131). N’exécutez pas encore la dernière étape (annulation de l’approvisionnement de la machine virtuelle).

Pour configurer le chiffrement afin de l’utiliser dans Azure, effectuez les étapes suivantes :
1. Modifiez le fichier /etc/dracut.conf et ajoutez la ligne suivante :
    ```
    add_drivers+=" vfat ntfs nls_cp437 nls_iso8859-1"
    ```
2. Mettez ces lignes en commentaire à la fin du fichier /usr/lib/dracut/modules.d/90crypt/module-setup.sh :
   ```bash
    #        inst_multiple -o \
    #        $systemdutildir/system-generators/systemd-cryptsetup-generator \
    #        $systemdutildir/systemd-cryptsetup \
    #        $systemdsystemunitdir/systemd-ask-password-console.path \
    #        $systemdsystemunitdir/systemd-ask-password-console.service \
    #        $systemdsystemunitdir/cryptsetup.target \
    #        $systemdsystemunitdir/sysinit.target.wants/cryptsetup.target \
    #        systemd-ask-password systemd-tty-ask-password-agent
    #        inst_script "$moddir"/crypt-run-generator.sh /sbin/crypt-run-generator
   ```

3. Ajoutez la ligne suivante au début du fichier /usr/lib/dracut/modules.d/90crypt/parse-crypt.sh :
   ```bash
    DRACUT_SYSTEMD=0
   ```
   Puis, remplacez toutes les occurrences de :
   ```bash
    if [ -z "$DRACUT_SYSTEMD" ]; then
   ```
   to:
   ```bash
    if [ 1 ]; then
   ```
4. Modifiez /usr/lib/dracut/modules.d/90crypt/cryptroot-ask.sh et ajoutez-le à « # Open LUKS device » :

    ```bash
    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint >&2
    modprobe vfat >/dev/null >&2
    modprobe ntfs >/dev/null >&2
    for SFS in /dev/sd*; do
    echo "> Trying device:$SFS..." >&2
    mount ${SFS}1 $MountPoint -t vfat -r >&2 ||
    mount ${SFS}1 $MountPoint -t ntfs -r >&2
    if [ -f $MountPoint/$KeyFileName ]; then
        echo "> keyfile got..." >&2
        cp $MountPoint/$KeyFileName /tmp-keyfile >&2
        luksfile=/tmp-keyfile
        umount $MountPoint >&2
        break
    fi
    done
    ```
5. Exécutez `/usr/sbin/dracut -f -v` pour mettre à jour le fichier initrd.

6. Vous pouvez maintenant déprovisionner la machine virtuelle et charger votre disque dur virtuel dans Azure.

### <a name="centos-7-and-rhel-7"></a>CentOS 7 et RHEL 7

Pour configurer le chiffrement lors de l’installation de la distribution, effectuez les étapes suivantes :
1. Sélectionnez **Chiffrer mes données** lors du partitionnement des disques.

   ![Configuration CentOS 7 - Destination de l’installation](./media/disk-encryption/centos-encrypt-fig1.png)

2. Vérifiez que **Chiffrer** est sélectionné pour la partition racine.

   ![Configuration CentOS 7 - Sélectionner le chiffrement pour la partition racine](./media/disk-encryption/centos-encrypt-fig2.png)

3. Indiquez une phrase secrète. Il s’agit de la phrase secrète que vous allez charger dans votre coffre de clés.

   ![Configuration CentOS 7 - Fournir la phrase secrète](./media/disk-encryption/centos-encrypt-fig3.png)

4. Lorsque vous démarrez la machine virtuelle et devez fournir une phrase secrète, utilisez la phrase secrète que vous avez fournie à l’étape 3.

   ![Configuration CentOS 7 - Entrer la phrase secrète lors du démarrage](./media/disk-encryption/centos-encrypt-fig4.png)

5. Préparez la machine virtuelle pour le chargement dans Azure en suivant les instructions « CentOS 7.0+ » dans la rubrique [Préparation d’une machine virtuelle CentOS pour Azure](./create-upload-centos.md?toc=/azure/virtual-machines/linux/toc.json#centos-70). N’exécutez pas encore la dernière étape (annulation de l’approvisionnement de la machine virtuelle).

6. Vous pouvez maintenant déprovisionner la machine virtuelle et charger votre disque dur virtuel dans Azure.

Pour configurer le chiffrement afin de l’utiliser dans Azure, effectuez les étapes suivantes :

1. Modifiez le fichier /etc/dracut.conf et ajoutez la ligne suivante :
    ```
    add_drivers+=" vfat ntfs nls_cp437 nls_iso8859-1"
    ```

2. Mettez ces lignes en commentaire à la fin du fichier /usr/lib/dracut/modules.d/90crypt/module-setup.sh :
   ```bash
    #        inst_multiple -o \
    #        $systemdutildir/system-generators/systemd-cryptsetup-generator \
    #        $systemdutildir/systemd-cryptsetup \
    #        $systemdsystemunitdir/systemd-ask-password-console.path \
    #        $systemdsystemunitdir/systemd-ask-password-console.service \
    #        $systemdsystemunitdir/cryptsetup.target \
    #        $systemdsystemunitdir/sysinit.target.wants/cryptsetup.target \
    #        systemd-ask-password systemd-tty-ask-password-agent
    #        inst_script "$moddir"/crypt-run-generator.sh /sbin/crypt-run-generator
   ```

3. Ajoutez la ligne suivante au début du fichier /usr/lib/dracut/modules.d/90crypt/parse-crypt.sh :
   ```bash
    DRACUT_SYSTEMD=0
   ```
   Puis, remplacez toutes les occurrences de :
   ```bash
    if [ -z "$DRACUT_SYSTEMD" ]; then
   ```
   to
   ```bash
    if [ 1 ]; then
   ```
4. Modifiez /usr/lib/dracut/modules.d/90crypt/cryptroot-ask.sh et ajoutez ce qui suit après « # Open LUKS device » :
    ```bash
    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint >&2
    modprobe vfat >/dev/null >&2
    modprobe ntfs >/dev/null >&2
    for SFS in /dev/sd*; do
    echo "> Trying device:$SFS..." >&2
    mount ${SFS}1 $MountPoint -t vfat -r >&2 ||
    mount ${SFS}1 $MountPoint -t ntfs -r >&2
    if [ -f $MountPoint/$KeyFileName ]; then
        echo "> keyfile got..." >&2
        cp $MountPoint/$KeyFileName /tmp-keyfile >&2
        luksfile=/tmp-keyfile
        umount $MountPoint >&2
        break
    fi
    done
    ```    
5. Exécutez « /usr/sbin/dracut -f -v » pour mettre à jour le fichier initrd.

    ![Configuration CentOS 7 - Exécuter /usr/sbin/dracut -f -v](./media/disk-encryption/centos-encrypt-fig5.png)

## <a name="upload-encrypted-vhd-to-an-azure-storage-account"></a>Télécharger des disques durs virtuels cryptés dans un compte de stockage Azure
Une fois le chiffrement DM-Crypt activé, le disque dur virtuel chiffré local doit être chargé sur votre compte de stockage.
```powershell
    Add-AzVhd [-Destination] <Uri> [-LocalFilePath] <FileInfo> [[-NumberOfUploaderThreads] <Int32> ] [[-BaseImageUriToPatch] <Uri> ] [[-OverWrite]] [ <CommonParameters>]
```
## <a name="upload-the-secret-for-the-pre-encrypted-vm-to-your-key-vault"></a>Charger le secret pour la machine virtuelle préchiffrée dans votre coffre de clés
Lors du chiffrement avec une application Azure AD (version précédente), la clé secrète de chiffrement de disque obtenue précédemment doit être chargée en tant que clé secrète dans votre coffre de clés. Le chiffrement de disque et les autorisations du coffre de clés doivent être activés pour votre client Azure AD.

```powershell 
 $AadClientId = "My-AAD-Client-Id"
 $AadClientSecret = "My-AAD-Client-Secret"

 $key vault = New-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $ResourceGroupName -Location $Location

 Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $ResourceGroupName -ServicePrincipalName $AadClientId -PermissionsToKeys all -PermissionsToSecrets all
 Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $ResourceGroupName -EnabledForDiskEncryption
``` 

### <a name="disk-encryption-secret-not-encrypted-with-a-kek"></a>La clé secrète de chiffrement de disque non chiffré avec une clé de chiffrement à clé KEK
Utilisez [Set-AzKeyVaultSecret](/powershell/module/az.keyvault/set-azkeyvaultsecret) pour configurer la clé secrète dans votre coffre de clés. La phrase secrète est encodée sous forme de chaîne en base64, puis chargée dans le coffre de clés. Assurez-vous également que les balises suivantes sont définies lors de la création de la clé secrète dans le coffre de clés.

```powershell

 # This is the passphrase that was provided for encryption during the distribution installation
 $passphrase = "contoso-password"

 $tags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
 $secretName = [guid]::NewGuid().ToString()
 $secretValue = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($passphrase))
 $secureSecretValue = ConvertTo-SecureString $secretValue -AsPlainText -Force

 $secret = Set-AzKeyVaultSecret -VaultName $KeyVaultName -Name $secretName -SecretValue $secureSecretValue -tags $tags
 $secretUrl = $secret.Id
```


Utilisez `$secretUrl` à l’étape suivante pour [attacher le lecteur du système d’exploitation sans utiliser de clé de chiffrement à clé](#without-using-a-kek).

### <a name="disk-encryption-secret-encrypted-with-a-kek"></a>Disque chiffré avec une clé secrète de chiffrement de disque à clé KEK
Avant de télécharger la clé secrète dans le coffre de clés, vous pouvez éventuellement la chiffrer à l’aide d’une clé de chiffrement à clé. Utilisez [l’API](/rest/api/keyvault/wrapkey) de retour à la ligne pour chiffrer d’abord la clé secrète à l’aide de la clé de chiffrement à clé. La sortie de cette opération de retour à la ligne est une chaîne d’URL encodée en base64 que vous pouvez ensuite charger comme clé secrète à l’aide de l’applet de commande [`Set-AzKeyVaultSecret`](/powershell/module/az.keyvault/set-azkeyvaultsecret).

```powershell
    # This is the passphrase that was provided for encryption during the distribution installation
    $passphrase = "contoso-password"

    Add-AzKeyVaultKey -VaultName $KeyVaultName -Name "keyencryptionkey" -Destination Software
    $KeyEncryptionKey = Get-AzKeyVaultKey -VaultName $KeyVault.OriginalVault.Name -Name "keyencryptionkey"

    $apiversion = "2015-06-01"

    ##############################
    # Get Auth URI
    ##############################

    $uri = $KeyVault.VaultUri + "/keys"
    $headers = @{}

    $response = try { Invoke-RestMethod -Method GET -Uri $uri -Headers $headers } catch { $_.Exception.Response }

    $authHeader = $response.Headers["www-authenticate"]
    $authUri = [regex]::match($authHeader, 'authorization="(.*?)"').Groups[1].Value

    Write-Host "Got Auth URI successfully"

    ##############################
    # Get Auth Token
    ##############################

    $uri = $authUri + "/oauth2/token"
    $body = "grant_type=client_credentials"
    $body += "&client_id=" + $AadClientId
    $body += "&client_secret=" + [Uri]::EscapeDataString($AadClientSecret)
    $body += "&resource=" + [Uri]::EscapeDataString("https://vault.azure.net")
    $headers = @{}

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $access_token = $response.access_token

    Write-Host "Got Auth Token successfully"

    ##############################
    # Get KEK info
    ##############################

    $uri = $KeyEncryptionKey.Id + "?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token}

    $response = Invoke-RestMethod -Method GET -Uri $uri -Headers $headers

    $keyid = $response.key.kid

    Write-Host "Got KEK info successfully"

    ##############################
    # Encrypt passphrase using KEK
    ##############################

    $passphraseB64 = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($Passphrase))
    $uri = $keyid + "/encrypt?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"alg" = "RSA-OAEP"; "value" = $passphraseB64}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $wrappedSecret = $response.value

    Write-Host "Encrypted passphrase successfully"

    ##############################
    # Store secret
    ##############################

    $secretName = [guid]::NewGuid().ToString()
    $uri = $KeyVault.VaultUri + "/secrets/" + $secretName + "?api-version=" + $apiversion
    $secretAttributes = @{"enabled" = $true}
    $secretTags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"value" = $wrappedSecret; "attributes" = $secretAttributes; "tags" = $secretTags}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method PUT -Uri $uri -Headers $headers -Body $body

    Write-Host "Stored secret successfully"

    $secretUrl = $response.id
```

Utilisez `$KeyEncryptionKey` et `$secretUrl` à l’étape suivante pour [attacher le lecteur du système d’exploitation à l’aide de la clé de chiffrement à clé](#using-a-kek).

##  <a name="specify-a-secret-url-when-you-attach-an-os-disk"></a>Spécifier une URL secrète lorsque vous attachez un lecteur de système d’exploitation

###  <a name="without-using-a-kek"></a>Sans utiliser de clé de chiffrement à clé KEK
Lorsque vous attachez le lecteur du système d’exploitation, exécutez la commande `$secretUrl`. L’URL a été générée dans la section « La clé secrète de chiffrement de disque non chiffrée avec une clé de chiffrement à clé ».
```powershell
    Set-AzVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $VhdUri `
            -VhdUri $OSDiskUri `
            -Linux `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl
```
### <a name="using-a-kek"></a>À l’aide d’une clé de chiffrement à clé KEK
Lorsque vous attachez le lecteur du système d’exploitation, exécutez les commandes `$KeyEncryptionKey` et `$secretUrl`. L’URL a été générée dans la section « Secret de chiffrement de disque chiffré avec une clé KEK ».
```powershell
    Set-AzVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $CopiedTemplateBlobUri `
            -VhdUri $OSDiskUri `
            -Linux `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl `
            -KeyEncryptionKeyVaultId $KeyVault.ResourceId `
            -KeyEncryptionKeyURL $KeyEncryptionKey.Id
```
