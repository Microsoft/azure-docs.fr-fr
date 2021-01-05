---
title: Configurer LVM et RAID sur des appareils chiffrés – Azure Disk Encryption
description: Cet article fournit des instructions sur la configuration de LVM et RAID sur des appareils chiffrés pour des machines virtuelles Linux.
author: jofrance
ms.service: virtual-machines
ms.subservice: security
ms.topic: how-to
ms.author: jofrance
ms.date: 03/17/2020
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 46d2c039806e4e6a72e091458d44e7b21b3dfa70
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94843517"
---
# <a name="configure-lvm-and-raid-on-encrypted-devices"></a>Configurer LVM et RAID sur des appareils chiffrés

Cet article est un processus pas à pas qui explique comment effectuer une gestion des volumes logiques (LVM) et un RAID sur des appareils chiffrés. Le processus s’applique aux environnements suivants :

- Distributions Linux
    - RHEL 7.6+
    - Ubuntu 18.04+
    - SUSE 12+
- Extension Azure Disk Encryption à un seul passage
- Extension Azure Disk Encryption à deux passages


## <a name="scenarios"></a>Scénarios

Les procédures décrites dans cet article prennent en charge les scénarios suivants :  

- Configurer LVM sur des appareils chiffrés (LVM-on-crypt)
- Configurer RAID sur des appareils chiffrés (RAID-on-crypt)

Une fois que les appareils sous-jacents sont chiffrés, vous pouvez créer les structures LVM ou RAID par-dessus cette couche chiffrée. 

Les volumes physiques (VP) sont créés par-dessus la couche chiffrée. Les volumes physiques sont utilisés pour créer le groupe de volumes. Vous créez les volumes et ajoutez les entrées requises sur /etc/fstab. 

![Diagramme des couches des structures LVM](./media/disk-encryption/lvm-raid-on-crypt/000-lvm-raid-crypt-diagram.png)

De la même façon, l’appareil RAID est créé par-dessus la couche chiffrée sur les disques. Un système de fichiers est créé par-dessus l’appareil RAID et ajouté à /etc/fstab comme appareil standard.

## <a name="considerations"></a>Considérations

Nous vous recommandons d’utiliser LVM-on-crypt. RAID est un choix possible lorsque LVM ne peut pas être utilisé en raison de limitations spécifiques de l’application ou de l’environnement.

Vous utiliserez l’option **EncryptFormatAll**. Pour plus d’informations sur cette option, consultez [Utiliser la fonctionnalité EncryptFormatAll pour les disques de données sur des machines virtuelles Linux](./disk-encryption-linux.md#use-encryptformatall-feature-for-data-disks-on-linux-vms).

Bien que vous puissiez utiliser cette méthode lorsque vous chiffrez également le système d’exploitation, nous chiffrons uniquement les lecteurs de données ici.

Les procédures supposent que vous avez déjà examiné les conditions préalables indiquées dans [Scénarios Azure Disk Encryption sur les machines virtuelles Linux](./disk-encryption-linux.md) et dans [Démarrage rapide : Créer et chiffrer une machine virtuelle Linux avec Azure CLI](./disk-encryption-cli-quickstart.md).

La version Azure Disk Encryption à deux passages est sur le point d’être déconseillée et ne doit plus être utilisée sur les nouveaux chiffrements.

## <a name="general-steps"></a>Étapes générales

Lorsque vous utilisez les configurations « on-crypt », utilisez le processus décrit dans les procédures suivantes.

>[!NOTE] 
>Nous utilisons des variables dans l’ensemble de l’article. Remplacez les valeurs en conséquence.

### <a name="deploy-a-vm"></a>Déployer une machine virtuelle 
Les commandes suivantes sont facultatives, mais nous vous recommandons de les appliquer sur une machine virtuelle nouvellement déployée.

PowerShell :

```powershell
New-AzVm -ResourceGroupName ${RGNAME} `
-Name ${VMNAME} `
-Location ${LOCATION} `
-Size ${VMSIZE} `
-Image ${OSIMAGE} `
-Credential ${creds} `
-Verbose
```
Azure CLI :

```bash
az vm create \
-n ${VMNAME} \
-g ${RGNAME} \
--image ${OSIMAGE} \
--admin-username ${username} \
--admin-password ${password} \
-l ${LOCATION} \
--size ${VMSIZE} \
-o table
```
### <a name="attach-disks-to-the-vm"></a>Attacher des disques à la machine virtuelle
Répétez les commandes suivantes pour un nombre `$N` de nouveaux disques que vous souhaitez attacher à la machine virtuelle.

PowerShell :

```powershell
$storageType = 'Standard_LRS'
$dataDiskName = ${VMNAME} + '_datadisk0'
$diskConfig = New-AzDiskConfig -SkuName $storageType -Location $LOCATION -CreateOption Empty -DiskSizeGB 5
$dataDisk1 = New-AzDisk -DiskName $dataDiskName -Disk $diskConfig -ResourceGroupName ${RGNAME}
$vm = Get-AzVM -Name ${VMNAME} -ResourceGroupName ${RGNAME} 
$vm = Add-AzVMDataDisk -VM $vm -Name $dataDiskName -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 0
Update-AzVM -VM ${VM} -ResourceGroupName ${RGNAME}
```

Azure CLI :

```bash
az vm disk attach \
-g ${RGNAME} \
--vm-name ${VMNAME} \
--name ${VMNAME}datadisk1 \
--size-gb 5 \
--new \
-o table
```

### <a name="verify-that-the-disks-are-attached-to-the-vm"></a>Vérifier que les disques sont attachés à la machine virtuelle
PowerShell :
```powershell
$VM = Get-AzVM -ResourceGroupName ${RGNAME} -Name ${VMNAME}
$VM.StorageProfile.DataDisks | Select-Object Lun,Name,DiskSizeGB
```
![Liste des disques attachés dans PowerShell](./media/disk-encryption/lvm-raid-on-crypt/001-lvm-raid-check-disks-powershell.png)

Azure CLI :

```bash
az vm show -g ${RGNAME} -n ${VMNAME} --query storageProfile.dataDisks -o table
```
![Liste des disques attachés dans Azure CLI](./media/disk-encryption/lvm-raid-on-crypt/002-lvm-raid-check-disks-cli.png)

Portail :

![Liste des disques attachés dans le portail](./media/disk-encryption/lvm-raid-on-crypt/003-lvm-raid-check-disks-portal.png)

Système d’exploitation :

```bash
lsblk 
```
![Liste des disques attachés dans le système d’exploitation](./media/disk-encryption/lvm-raid-on-crypt/004-lvm-raid-check-disks-os.png)

### <a name="configure-the-disks-to-be-encrypted"></a>Configurer les disques à chiffrer
Cette configuration est effectuée au niveau du système d’exploitation. Les disques correspondants sont configurés pour un chiffrement traditionnel par le biais d’Azure Disk Encryption :

- Les systèmes de fichiers sont créés par-dessus les disques.
- Des points de montage temporaires sont créés pour monter les systèmes de fichiers.
- Les systèmes de fichiers sont configurés sur /etc/fstab afin d’être montés au moment du démarrage.

Vérifiez la lettre d’appareil attribuée aux nouveaux disques. Dans cet exemple, nous utilisons quatre disques de données.

```bash
lsblk 
```
![Disques de données attachés au système d’exploitation](./media/disk-encryption/lvm-raid-on-crypt/004-lvm-raid-check-disks-os.png)

### <a name="create-a-file-system-on-top-of-each-disk"></a>Créer un système de fichiers par-dessus chaque disque
Cette commande itère la création d’un système de fichiers ext4 sur chaque disque défini sur la partie « in » du cycle « for ».

```bash
for disk in c d e f; do echo mkfs.ext4 -F /dev/sd${disk}; done |bash
```
![Création d’un système de fichiers ext4](./media/disk-encryption/lvm-raid-on-crypt/005-lvm-raid-create-temp-fs.png)

Recherchez l’identificateur unique universel (UUID) des systèmes de fichiers que vous avez créés récemment, créez un dossier temporaire, ajoutez les entrées correspondantes sur /etc/fstab et montez tous les systèmes de fichiers.

Cette commande effectue également une itération sur chaque disque défini sur la partie « in » du cycle « for » :

```bash
for disk in c d e f; do diskuuid="$(blkid -s UUID -o value /dev/sd${disk})"; \
mkdir /tempdata${disk}; \
echo "UUID=${diskuuid} /tempdata${disk} ext4 defaults,nofail 0 0" >> /etc/fstab; \
mount -a; \
done
``` 

### <a name="verify-that-the-disks-are-mounted-properly"></a>Vérifier que les disques sont correctement montés
```bash
lsblk
```
![Liste des systèmes de fichiers temporaires montés](./media/disk-encryption/lvm-raid-on-crypt/006-lvm-raid-verify-temp-fs.png)

Vérifiez également que les disques sont configurés :

```bash
cat /etc/fstab
```
![Informations de configuration via fstab](./media/disk-encryption/lvm-raid-on-crypt/007-lvm-raid-verify-temp-fstab.png)

### <a name="encrypt-the-data-disks"></a>Chiffrer les disques de données
PowerShell à l’aide d’une clé de chiffrement principale (KEK) :

```powershell
$sequenceVersion = [Guid]::NewGuid() 
Set-AzVMDiskEncryptionExtension -ResourceGroupName $RGNAME `
-VMName ${VMNAME} `
-DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl `
-DiskEncryptionKeyVaultId $KeyVaultResourceId `
-KeyEncryptionKeyUrl $keyEncryptionKeyUrl `
-KeyEncryptionKeyVaultId $KeyVaultResourceId `
-VolumeType 'DATA' `
-EncryptFormatAll `
-SequenceVersion $sequenceVersion `
-skipVmBackup;
```

Azure CLI à l’aide d’une KEK :

```bash
az vm encryption enable \
--resource-group ${RGNAME} \
--name ${VMNAME} \
--disk-encryption-keyvault ${KEYVAULTNAME} \
--key-encryption-key ${KEYNAME} \
--key-encryption-keyvault ${KEYVAULTNAME} \
--volume-type "DATA" \
--encrypt-format-all \
-o table
```
### <a name="verify-the-encryption-status"></a>Vérifier l’état du chiffrement

Passez à l’étape suivante uniquement lorsque tous les disques sont chiffrés.

PowerShell :

```powershell
Get-AzVmDiskEncryptionStatus -ResourceGroupName ${RGNAME} -VMName ${VMNAME}
```
![État du chiffrement dans PowerShell](./media/disk-encryption/lvm-raid-on-crypt/008-lvm-raid-verify-encryption-status-ps.png)

Azure CLI :

```bash
az vm encryption show -n ${VMNAME} -g ${RGNAME} -o table
```
![État du chiffrement dans Azure CLI](./media/disk-encryption/lvm-raid-on-crypt/009-lvm-raid-verify-encryption-status-cli.png)

Portail :

![État du chiffrement dans le portail](./media/disk-encryption/lvm-raid-on-crypt/010-lvm-raid-verify-encryption-status-portal.png)

Niveau du système d’exploitation :

```bash
lsblk
```
![État du chiffrement dans le système d’exploitation](./media/disk-encryption/lvm-raid-on-crypt/011-lvm-raid-verify-encryption-status-os.png)

L’extension ajoutera les systèmes de fichiers à /var/lib/azure_disk_encryption_config/azure_crypt_mount (un ancien chiffrement) ou à /etc/crypttab (nouveaux chiffrements).

>[!NOTE] 
>Ne modifiez aucun de ces fichiers.

Ce fichier s’occupera de l’activation de ces disques pendant le processus de démarrage afin que LVM ou RAID puisse les utiliser ultérieurement. 

Ne vous préoccupez pas des points de montage sur ce fichier. Azure Disk Encryption perdra la capacité de monter les disques comme système de fichiers normal après que nous ayons créé un volume physique ou un appareil RAID sur ces appareils chiffrés. (Cela supprimera le format de système de fichiers que nous avons utilisé pendant le processus de préparation.)

### <a name="remove-the-temporary-folders-and-temporary-fstab-entries"></a>Supprimer les dossiers temporaires et les entrées fstab temporaires
Vous démontez les systèmes de fichiers des disques qui seront utilisés dans le cadre de LVM.

```bash
for disk in c d e f; do unmount /tempdata${disk}; done
```
Et supprimez les entrées /etc/fstab :

```bash
vi /etc/fstab
```
### <a name="verify-that-the-disks-are-not-mounted-and-that-the-entries-on-etcfstab-were-removed"></a>Vérifiez que les disques ne sont pas montés et que les entrées sur /etc/fstab ont été supprimées.

```bash
lsblk
```
![Vérification du démontage des systèmes de fichiers temporaires](./media/disk-encryption/lvm-raid-on-crypt/012-lvm-raid-verify-disks-not-mounted.png)

Vérifiez également que les disques sont configurés :
```bash
cat /etc/fstab
```
![Vérification de la suppression des entrées fstab temporaires](./media/disk-encryption/lvm-raid-on-crypt/013-lvm-raid-verify-fstab-temp-removed.png)

## <a name="steps-for-lvm-on-crypt"></a>Étapes pour LVM-on-crypt
Maintenant que les disques sous-jacents sont chiffrés, vous pouvez créer des structures LVM.

Au lieu d’utiliser le nom de l’appareil, utilisez les chemins d’accès /dev/mapper pour chacun des disques afin de créer un volume physique (sur la couche de chiffrement par-dessus le disque, et non sur le disque lui-même).

### <a name="configure-lvm-on-top-of-the-encrypted-layers"></a>Configurer LVM par-dessus les couches chiffrées
#### <a name="create-the-physical-volumes"></a>Créer les volumes physiques
Un avertissement s’affiche pour vous demander si vous souhaitez effacer la signature du système de fichiers. Continuez en entrant **y** ou utilisez l’écho **y**, comme indiqué ci-dessous :

```bash
echo "y" | pvcreate /dev/mapper/c49ff535-1df9-45ad-9dad-f0846509f052
echo "y" | pvcreate /dev/mapper/6712ad6f-65ce-487b-aa52-462f381611a1
echo "y" | pvcreate /dev/mapper/ea607dfd-c396-48d6-bc54-603cf741bc2a
echo "y" | pvcreate /dev/mapper/4159c60a-a546-455b-985f-92865d51158c
```
![Vérification de la création d’un volume physique](./media/disk-encryption/lvm-raid-on-crypt/014-lvm-raid-pvcreate.png)

>[!NOTE] 
>Les noms /dev/mapper/device doivent ici être remplacés par vos valeurs réelles en fonction de la sortie de **lsblk**.

#### <a name="verify-the-information-for-physical-volumes"></a>Vérifier les informations des volumes physiques
```bash
pvs
```

![Informations des volumes physiques](./media/disk-encryption/lvm-raid-on-crypt/015-lvm-raid-pvs.png)

#### <a name="create-the-volume-group"></a>Créer le groupe de volumes
Créez le groupe de volumes en utilisant les mêmes appareils déjà initialisés :

```bash
vgcreate vgdata /dev/mapper/
```

### <a name="check-the-information-for-the-volume-group"></a>Vérifier les informations du groupe de volumes

```bash
vgdisplay -v vgdata
```
```bash
pvs
```
![Informations du groupe de volumes](./media/disk-encryption/lvm-raid-on-crypt/016-lvm-raid-pvs-on-vg.png)

#### <a name="create-logical-volumes"></a>Créer des volumes logiques

```bash
lvcreate -L 10G -n lvdata1 vgdata
lvcreate -L 7G -n lvdata2 vgdata
``` 

#### <a name="check-the-created-logical-volumes"></a>Vérifier les volumes logiques créés

```bash
lvdisplay
lvdisplay vgdata/lvdata1
lvdisplay vgdata/lvdata2
```
![Informations des volumes logiques](./media/disk-encryption/lvm-raid-on-crypt/017-lvm-raid-lvs.png)

#### <a name="create-file-systems-on-top-of-the-structures-for-logical-volumes"></a>Créer des systèmes de fichiers par-dessus les structures des volumes logiques

```bash
echo "yes" | mkfs.ext4 /dev/vgdata/lvdata1
echo "yes" | mkfs.ext4 /dev/vgdata/lvdata2
```

#### <a name="create-the-mount-points-for-the-new-file-systems"></a>Créer les points de montage des nouveaux systèmes de fichiers

```bash
mkdir /data0
mkdir /data1
```

#### <a name="add-the-new-file-systems-to-etcfstab-and-mount-them"></a>Ajouter les nouveaux systèmes de fichiers à /etc/fstab et les monter

```bash
echo "/dev/mapper/vgdata-lvdata1 /data0 ext4 defaults,nofail 0 0" >>/etc/fstab
echo "/dev/mapper/vgdata-lvdata2 /data1 ext4 defaults,nofail 0 0" >>/etc/fstab
mount -a
```

#### <a name="verify-that-the-new-file-systems-are-mounted"></a>Vérifier le montage des nouveaux systèmes de fichiers

```bash
lsblk -fs
df -h
```
![Capture d’écran montrant une fenêtre de console avec des systèmes de fichiers montés en tant que data0 et data1.](./media/disk-encryption/lvm-raid-on-crypt/018-lvm-raid-lsblk-after-lvm.png)

Sur cette variation de **lsblk**, nous répertorions les appareils qui affichent les dépendances dans l’ordre inverse. Cette option permet d’identifier les appareils regroupés par le volume logique à la place des noms d’appareil /dev/sd[disk] d’origine.

Il est important de s’assurer que l’option **nofail** est ajoutée aux options de point de montage des volumes LVM créés par-dessus un appareil chiffré par Azure Disk Encryption. Elle empêche que le système d’exploitation se bloque pendant le processus de démarrage (ou en mode de maintenance).

Si vous n’utilisez pas l’option **nofail** :

- Le système d’exploitation n’arrive jamais à l’étape où Azure Disk Encryption est lancé et les disques de données sont déverrouillés et montés. 
- Les disques chiffrés seront déverrouillés à la fin du processus de démarrage. Les volumes LVM et les systèmes de fichiers sont automatiquement montés jusqu’à ce qu’Azure Disk Encryption les déverrouille. 

Vous pouvez tester le redémarrage de la machine virtuelle et vérifier que les systèmes de fichiers sont également montés automatiquement après le démarrage. Ce processus peut prendre plusieurs minutes en fonction du nombre et de la taille des systèmes de fichiers.

#### <a name="reboot-the-vm-and-verify-after-reboot"></a>Redémarrer la machine virtuelle et vérifier après le redémarrage

```bash
shutdown -r now
```
```bash
lsblk
df -h
```
## <a name="steps-for-raid-on-crypt"></a>Étapes pour RAID-on-crypt
Maintenant que les disques sous-jacents sont chiffrés, vous pouvez continuer en créant les structures RAID. Le processus est identique à celui pour LVM, mais au lieu d’utiliser le nom de l’appareil, utilisez les chemins d’accès /dev/mapper pour chaque disque.

#### <a name="configure-raid-on-top-of-the-encrypted-layer-of-the-disks"></a>Configurer RAID par-dessus la couche chiffrée des disques
```bash
mdadm --create /dev/md10 \
--level 0 \
--raid-devices=4 \
/dev/mapper/c49ff535-1df9-45ad-9dad-f0846509f052 \
/dev/mapper/6712ad6f-65ce-487b-aa52-462f381611a1 \
/dev/mapper/ea607dfd-c396-48d6-bc54-603cf741bc2a \
/dev/mapper/4159c60a-a546-455b-985f-92865d51158c
```
![Informations du RAID configuré via la commande mdadm](./media/disk-encryption/lvm-raid-on-crypt/019-lvm-raid-md-creation.png)

>[!NOTE] 
>Les noms /dev/mapper/device doivent ici être remplacés par vos valeurs réelles en fonction de la sortie de **lsblk**.

### <a name="checkmonitor-raid-creation"></a>Vérifier/surveiller la création de RAID
```bash
watch -n1 cat /proc/mdstat
mdadm --examine /dev/mapper/[]
mdadm --detail /dev/md10
```
![État de RAID](./media/disk-encryption/lvm-raid-on-crypt/020-lvm-raid-md-details.png)

### <a name="create-a-file-system-on-top-of-the-new-raid-device"></a>Créer un système de fichiers par-dessus le nouvel appareil RAID
```bash
mkfs.ext4 /dev/md10
```

Créez un nouveau point de montage pour le système de fichiers, ajoutez le nouveau système de fichiers à /etc/fstab et montez-le :

```bash
for device in md10; do diskuuid="$(blkid -s UUID -o value /dev/${device})"; \
mkdir /raiddata; \
echo "UUID=${diskuuid} /raiddata ext4 defaults,nofail 0 0" >> /etc/fstab; \
mount -a; \
done
```

Vérifiez que le nouveau système de fichiers est monté :

```bash
lsblk -fs
df -h
```
![Capture d’écran montrant une fenêtre de console avec un système de fichiers monté en tant que raiddata.](./media/disk-encryption/lvm-raid-on-crypt/021-lvm-raid-lsblk-md-details.png)

Il est important de s’assurer que l’option **nofail** est ajoutée aux options de point de montage des volumes RAIS créés par-dessus un appareil chiffré par Azure Disk Encryption. Elle empêche que le système d’exploitation se bloque pendant le processus de démarrage (ou en mode de maintenance).

Si vous n’utilisez pas l’option **nofail** :

- Le système d’exploitation n’arrive jamais à l’étape où Azure Disk Encryption est lancé et les disques de données sont déverrouillés et montés.
- Les disques chiffrés seront déverrouillés à la fin du processus de démarrage. Les volumes RAID et les systèmes de fichiers sont automatiquement montés jusqu’à ce qu’Azure Disk Encryption les déverrouille.

Vous pouvez tester le redémarrage de la machine virtuelle et vérifier que les systèmes de fichiers sont également montés automatiquement après le démarrage. Ce processus peut prendre plusieurs minutes en fonction du nombre et de la taille des systèmes de fichiers.

```bash
shutdown -r now
```

Et quand vous pouvez vous connecter :

```bash
lsblk
df -h
```
## <a name="next-steps"></a>Étapes suivantes

- [Redimensionner des appareils de gestion de volumes logiques chiffrés avec Azure Disk Encryption](how-to-resize-encrypted-lvm.md)
- [Résolution des problèmes liés à Azure Disk Encryption](disk-encryption-troubleshooting.md)
