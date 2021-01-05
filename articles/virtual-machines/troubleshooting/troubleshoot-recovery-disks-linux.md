---
title: Utiliser une machine virtuelle de dépannage Linux avec Azure CLI | Microsoft Docs
description: Découvrez comment résoudre les problèmes de machines virtuelles Linux en connectant le disque du système d’exploitation à une machine virtuelle de récupération à l’aide de l’interface de ligne de commande Azure.
services: virtual-machines-linux
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 02/16/2017
ms.author: genli
ms.openlocfilehash: bc796b4d9708c18f4d0612e6f3ebde9b2485d2f7
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92093138"
---
# <a name="troubleshoot-a-linux-vm-by-attaching-the-os-disk-to-a-recovery-vm-with-the-azure-cli"></a>Résoudre les problèmes d’une machine virtuelle Linux en attachant le disque de système d’exploitation à une machine virtuelle de récupération avec Azure CLI
Si votre machine virtuelle Linux rencontre une erreur de démarrage ou de disque, il vous faudra éventuellement appliquer la procédure de dépannage directement sur le disque dur virtuel. Comme exemple courant, citons une entrée non valide dans `/etc/fstab` qui empêche le bon démarrage de la machine virtuelle. Cet article vous explique comment utiliser l’interface de ligne de commande Azure pour connecter votre disque dur virtuel à une autre machine virtuelle Linux pour corriger les éventuelles erreurs, puis pour régénérer votre machine virtuelle d’origine. 

## <a name="recovery-process-overview"></a>Vue d’ensemble du processus de récupération
Le processus de résolution de problème se présente comme suit :

1. Arrêter la machine virtuelle affectée.
1. Prenez une capture instantanée du disque du système d’exploitation de la machine virtuelle.
1. Créer un disque à partir de la capture instantanée du disque de système d’exploitation.
1. Joignez et montez le disque dur virtuel sur une autre machine virtuelle Linux à des fins de résolution des problèmes.
1. Connectez-vous à la machine virtuelle de dépannage. Modifiez les fichiers ou exécutez des outils afin de corriger les problèmes du nouveau disque du système d’exploitation.
1. Démontez et détachez le nouveau disque de système d’exploitation de la machine virtuelle de résolution des problèmes.
1. Changer le disque de système d’exploitation pour la machine virtuelle affectée.

Pour effectuer ces étapes de dépannage, vous devez installer la dernière version [d’Azure CLI](/cli/azure/install-az-cli2) et vous connecter à un compte Azure avec [az login](/cli/azure/reference-index).

Vous pouvez utiliser les commandes de réparation de machine virtuelle pour automatiser les étapes 1, 2, 3, 4, 6 et 7. Pour obtenir plus de documentation et d’instructions, consultez [Réparer une machine virtuelle Linux à l’aide des commandes de réparation de machine virtuelle Azure](repair-linux-vm-using-azure-virtual-machine-repair-commands.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

> [!Important]
> Les scripts dans cet article s’appliquent uniquement aux machines virtuelles qui utilisent [Disque managé](../managed-disks-overview.md). 

Dans les exemples suivants, remplacez les noms de paramètres par vos propres valeurs, telles que `myResourceGroup` et `myVM`.

## <a name="determine-boot-issues"></a>Identifier les problèmes de démarrage
Examinez la sortie en série afin d’identifier la raison pour laquelle votre machine virtuelle ne démarre pas correctement. Comme exemple courant, citons une entrée non valide dans `/etc/fstab`, ou le disque dur virtuel en cours de suppression ou de déplacement.

Accédez aux journaux d’activité du démarrage avec [az vm boot-diagnostics get-boot-log](/cli/azure/vm/boot-diagnostics). L’exemple suivant récupère la sortie en série de la machine virtuelle nommée `myVM` dans le groupe de ressources nommé `myResourceGroup` :

```azurecli
az vm boot-diagnostics get-boot-log --resource-group myResourceGroup --name myVM
```

Examinez la sortie en série afin d’identifier la raison pour laquelle le démarrage de la machine virtuelle est mis en échec. Si la sortie en série ne fournit aucune indication, il vous faudra éventuellement passer en revue les fichiers journaux de `/var/log` une fois que le disque dur virtuel est connecté à une machine virtuelle de dépannage.

## <a name="stop-the-vm"></a>Arrêtez la machine virtuelle.

L’exemple suivant arrête la machine virtuelle nommée `myVM` dans le groupe de ressource nommé `myResourceGroup` :

```azurecli
az vm stop --resource-group MyResourceGroup --name MyVm
```
## <a name="take-a-snapshot-from-the-os-disk-of-the-affected-vm"></a>Prenez une capture instantanée du disque du système d’exploitation de la machine virtuelle concernée.

Une capture instantanée est une copie complète en lecture seule d’un disque dur virtuel. Elle ne peut pas être attachée à une machine virtuelle. Dans l’étape suivante, vous allez créer un disque à partir de cet instantané. L’exemple suivant crée une capture instantanée avec le nom `mySnapshot` à partir du disque de système d’exploitation de la machine virtuelle nommée « myVM ». 

```azurecli
#Get the OS disk Id 
$osdiskid=(az vm show -g myResourceGroup -n myVM --query "storageProfile.osDisk.managedDisk.id" -o tsv)

#creates a snapshot of the disk
az snapshot create --resource-group myResourceGroupDisk --source "$osdiskid" --name mySnapshot
```
## <a name="create-a-disk-from-the-snapshot"></a>Créer un disque à partir de la capture instantanée

Ce script crée un disque managé portant le nom `myOSDisk` à partir de l’instantané nommé `mySnapshot`.  

```azurecli
#Provide the name of your resource group
$resourceGroup="myResourceGroup"

#Provide the name of the snapshot that will be used to create Managed Disks
$snapshot="mySnapshot"

#Provide the name of the Managed Disk
$osDisk="myNewOSDisk"

#Provide the size of the disks in GB. It should be greater than the VHD file size.
$diskSize=128

#Provide the storage type for Managed Disk. Premium_LRS or Standard_LRS.
$storageType="Premium_LRS"

#Provide the OS type
$osType="linux"

#Get the snapshot Id 
$snapshotId=(az snapshot show --name $snapshot --resource-group $resourceGroup --query id -o tsv)

# Create a new Managed Disks using the snapshot Id.

az disk create --resource-group $resourceGroup --name $osDisk --sku $storageType --size-gb $diskSize --source $snapshotId

```

Si le groupe de ressources et l’instantané source ne se trouvent pas dans la même région, vous recevrez ce message d’erreur « la ressource est introuvable » lorsque vous exécutez `az disk create`. Dans ce cas, vous devez indiquer `--location <region>` pour créer le disque dans la même région que l’instantané source.

Vous disposez maintenant d’une copie du disque de système d’exploitation d’origine. Vous pouvez monter ce disque sur une autre machine virtuelle Windows à des fins de résolution des problèmes.

## <a name="attach-the-new-virtual-hard-disk-to-another-vm"></a>Joignez le nouveau disque dur virtuel existant sur une autre machine virtuelle
Pour les prochaines étapes, vous utilisez une autre machine virtuelle à des fins de résolution des problèmes. Vous joignez le disque à cette machine virtuelle de résolution des problèmes et modifiez le contenu du disque. Ce processus vous permet de corriger les éventuelles erreurs de configuration ou d'examiner d'autres fichiers journaux du système ou des applications.

Ce script joint le disque `myNewOSDisk` à la machine virtuelle `MyTroubleshootVM` :

```azurecli
# Get ID of the OS disk that you just created.
$myNewOSDiskid=(az disk show -g $resourceGroup -n $osDisk --query id -o tsv)

# Attach the disk to the troubleshooting VM
az vm disk attach --disk $myNewOSDiskid --resource-group $resourceGroup --size-gb $diskSize --sku $storageType --vm-name MyTroubleshootVM
```
## <a name="mount-the-attached-data-disk"></a>Monter le disque de données associé

> [!NOTE]
> Les exemples suivants décrivent les étapes requises sur une machine virtuelle Ubuntu. Si vous utilisez une distribution Linux différente, comme Red Hat Enterprise Linux ou SUSE, les emplacements de fichiers journaux et les commandes `mount` peuvent varier. Pour connaître les modifications appropriées dans les commandes, reportez-vous à la documentation de votre distribution spécifique.

1. Exécutez une commande SSH sur votre machine virtuelle de dépannage à l’aide des informations d’identification appropriées. Si ce disque est le premier disque de données associé à votre machine virtuelle de dépannage, il est probablement connecté à `/dev/sdc`. Pour afficher les disques associés, utilisez `dmesg` :

    ```bash
    dmesg | grep SCSI
    ```

    Le résultat ressemble à l’exemple suivant :

    ```bash
    [    0.294784] SCSI subsystem initialized
    [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
    [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
    [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
    [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
    ```

    Dans l’exemple précédent, le disque du système d’exploitation se trouve au niveau de `/dev/sda`, et le disque temporaire fourni pour chaque machine virtuelle se trouve au niveau de `/dev/sdb`. Si vous possédiez plusieurs disques de données, ils doivent se trouver au niveau de `/dev/sdd`, `/dev/sde`, etc.

2. Créez un répertoire pour monter votre disque dur virtuel existant. L’exemple suivant crée un répertoire nommé `troubleshootingdisk` :

    ```bash
    sudo mkdir /mnt/troubleshootingdisk
    ```

3. Si votre disque dur virtuel existant présente plusieurs partitions, montez la partition requise. L’exemple suivant monte la première partition primaire au niveau de `/dev/sdc1` :

    ```bash
    sudo mount /dev/sdc1 /mnt/troubleshootingdisk
    ```

    > [!NOTE]
    > Nous vous recommandons de monter les disques de données sur les machines virtuelles à l’aide de l’identifiant unique universel (UUID) du disque dur virtuel. Pour ce scénario court de résolution des problèmes, il n’est pas nécessaire de monter le disque dur virtuel à l’aide de l’UUID. Toutefois, dans des conditions normales, la modification de `/etc/fstab` pour monter les disques durs virtuels avec le nom d’appareil en lieu et place de l’UUID peut entraîner la mise en échec du démarrage de la machine virtuelle.


## <a name="fix-issues-on-the-new-os-disk"></a>Résoudre les problèmes sur le nouveau disque de système d’exploitation
Avec le disque dur virtuel existant monté, vous pouvez désormais exécuter les procédures de maintenance et de dépannage requises. Une fois que vous avez résolu les problèmes, passez aux étapes suivantes.


## <a name="unmount-and-detach-the-new-os-disk"></a>Démontez et détachez le nouveau disque de système d’exploitation
Une fois les erreurs résolues, vous démontez et dissociez le disque dur virtuel existant de votre machine virtuelle de dépannage. Vous ne pouvez pas utiliser votre disque dur virtuel avec une autre machine virtuelle avant la libération du bail associant le disque dur virtuel à la machine virtuelle de dépannage.

1. À partir de la session SSH vers votre machine virtuelle de dépannage, démontez le disque dur virtuel existant. Commencez par modifier votre point de montage dans le répertoire parent :

    ```bash
    cd /
    ```

    Montez alors le disque dur virtuel existant. L’exemple suivant démonte l’appareil au niveau de `/dev/sdc1` :

    ```bash
    sudo umount /dev/sdc1
    ```

2. Dissociez le disque dur virtuel de la machine virtuelle. Quittez la session SSH vers votre machine virtuelle de résolution des problèmes :

    ```azurecli
    az vm disk detach -g MyResourceGroup --vm-name MyTroubleShootVm --name myNewOSDisk
    ```

## <a name="change-the-os-disk-for-the-affected-vm"></a>Changer le disque de système d’exploitation pour la machine virtuelle affectée

Vous pouvez utiliser Azure CLI pour échanger les disques de système d’exploitation. Il est inutile de supprimer et de recréer la machine virtuelle.

Cet exemple arrête la machine virtuelle nommée `myVM` et affecte le disque nommé `myNewOSDisk` en tant que nouveau disque de système d’exploitation.

```azurecli
# Stop the affected VM
az vm stop -n myVM -g myResourceGroup

# Get ID of the OS disk that is repaired.
$myNewOSDiskid=(az vm show -g $resourceGroup -n $osDisk --query id -o tsv)

# Change the OS disk of the affected VM to "myNewOSDisk"
az vm update -g myResourceGroup -n myVM --os-disk $myNewOSDiskid

# Start the VM
az vm start -n myVM -g myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes
Si vous rencontrez des problèmes pour vous connecter à votre machine virtuelle, consultez la rubrique [Dépannage d’une connexion SSH à une machine virtuelle Linux Azure défaillante, qui génère une erreur ou qui est refusée](troubleshoot-ssh-connection.md). Pour résoudre les problèmes liés à l’accès aux applications exécutées sur votre machine virtuelle, consultez la section [Résoudre les problèmes de connectivité des applications sur une machine virtuelle Linux Azure](troubleshoot-app-connection.md).

