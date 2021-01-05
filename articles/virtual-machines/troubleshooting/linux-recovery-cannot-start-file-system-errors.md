---
title: Résoudre les problèmes de démarrage de machine virtuelle Linux dus à des erreurs de système de fichiers | Microsoft Docs
description: Découvrez comment résoudre les problèmes de démarrage de machine virtuelle Linux dus à des erreurs de système de fichiers.
services: virtual-machines-linux
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines-linux
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.date: 10/09/2019
ms.author: v-six
ms.openlocfilehash: 8f12b88a0ddbc6ae31f40ab31b0126e4fd66b1a5
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91325944"
---
# <a name="troubleshoot-linux-vm-starting-issues-due-to-file-system-errors"></a>Résoudre les problèmes de démarrage de machine virtuelle Linux dus à des erreurs de système de fichiers

Vous ne pouvez pas vous connecter à une machine virtuelle Linux Azure avec Secure Shell (SSH). Quand vous exécutez la fonctionnalité Diagnostics de démarrage sur le [portail Azure](https://portal.azure.com/), des entrées de journal semblables aux exemples suivants s’affichent.

## <a name="examples"></a>Exemples

Voici quelques exemples d’erreurs possibles.

### <a name="example-1"></a>Exemple 1 

```
Checking all file systems.
[/sbin/fsck.ext4 (1) — /] fsck.ext4 -a /dev/sda1
/dev/sda1 contains a file system with errors, check forced.
/dev/sda1: Inodes that were part of a corrupted orphan linked list found.
/dev/sda1: UNEXPECTED INCONSISTENCY; RUN fsck MANUALLY
```

### <a name="example-2"></a>Exemple 2

```
EXT4-fs (sda1): INFO: recovery required on readonly filesystem
EXT4-fs (sda1): write access will be enabled during recovery
EXT4-fs warning (device sda1): ext4_clear_journal_err:4531: Filesystem error recorded from previous mount: IO failure
EXT4-fs warning (device sda1): ext4_clear_journal_err:4532: Making fs in need of filesystem check.
```

### <a name="example-3"></a>Exemple 3

```
[  14.252404] EXT4-fs (sda1): Couldn’t remount RDWR because of unprocessed orphan inode list.  Please unmount/remount instead
An error occurred while mounting /.
```

### <a name="example-4"></a>Exemple 4 

Cette erreur est provoquée par une commande fsck clean. Dans ce cas, des disques de données supplémentaires sont attachés à la machine virtuelle (/dev/sdc1 et /dev/sde1).

```
Checking all file systems. 
[/sbin/fsck.ext4 (1) — /] fsck.ext4 -a /dev/sda1
/dev/sda1: clean, 65405/1905008 files, 732749/7608064 blocks
[/sbin/fsck.ext4 (1) — /tmp] fsck.ext4 -a /dev/sdc1
[/sbin/fsck.ext4 (2) — /backup] fsck.ext4 -a /dev/sde1
/dev/sdc1: clean, 12/1048576 files, 109842/4192957 blocks
/dev/sde1 : clean, 51/67043328 files, 4259482/268173037 blocks
```

Ce problème peut se produire si le système de fichiers n’a pas été arrêté correctement ou en présence de problèmes de stockage. Les problèmes incluent les erreurs matérielles ou logicielles, les problèmes liés aux pilotes ou aux programmes, les erreurs d’écriture, etc. Il est toujours important de disposer d’une sauvegarde des données critiques. Les outils décrits dans cet article peuvent vous aider à récupérer des systèmes de fichiers. Cependant, des pertes de données sont toujours possibles.

Plusieurs contrôleurs de système de fichiers sont disponibles dans Linux. Ceux utilisés le plus fréquemment pour les distributions dans Azure sont les suivants : [FSCK](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/storage_administration_guide/fsck-fs-specific), [E2FSCK](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/fsck-fs-specific) et [Xfs_repair](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/xfsrepair).

## <a name="resolution"></a>Résolution

Pour résoudre ce problème, démarrez la machine virtuelle en mode d’urgence à l’aide de la [console série](./serial-console-linux.md) et utilisez cet outil pour réparer le système de fichiers. Si la console série n’est pas activée sur la machine virtuelle ou si elle ne fonctionne pas, consultez la section [Réparer la machine virtuelle en mode hors connexion](#repair-the-vm-offline) de cet article.

## <a name="use-the-serial-console"></a>Utiliser la console série

1. Connectez-vous à la console série.

   > [!Note]
   > Pour plus d’informations sur l’utilisation de la console série pour Linux, consultez :
   > * [Utiliser la console série pour accéder au GRUB et au mode mono-utilisateur](serial-console-grub-single-user-mode.md)
   > * [Utiliser la console série pour les appels SysRq et NMI](./serial-console-nmi-sysrq.md)

2. Sélectionnez l’icône du bouton d’alimentation, puis sélectionnez Redémarrer la machine virtuelle. (Si la console série n’est pas activée ou n’est pas connectée correctement, le bouton ne s’affiche pas.)

   ![IMAGE](./media/linux-recovery-cannot-ssh-to-linux-vm-due-to-file-system-errors-fsck/restart-vm.png)

3. Démarrez la machine virtuelle en mode d’urgence.

4. Entrez le mot de passe de votre compte racine pour vous connecter en mode d’urgence.

5. Utilisez xfs_repair avec l’option -n pour détecter les erreurs dans le système de fichiers. Dans l’exemple suivant, nous supposons que la partition système est /dev/sda1. Remplacez-la par la valeur correspondant à votre machine virtuelle :

   ```
   xfs_repair -n /dev/sda1
   ```

6. Exécutez la commande suivante pour réparer le système de fichiers :

   ```
   xfs_repair /dev/sda1
   ```

7. Si vous recevez le message d’erreur suivant « ERREUR : Le système de fichiers présente des modifications de métadonnées importantes dans un journal qui doit être relu », créez un répertoire temporaire et montez le système de fichiers :

   ```
   mkdir /temp
   mount /dev/sda1 /temp
   ```

8. En cas d’échec du montage du disque, exécutez la commande xfs_repair avec l’option -L (forcer la mise à zéro du journal) :

   ```
   xfs_repair /dev/sda1 -L
   ```

9. Ensuite, essayez de monter le système de fichiers. Si le disque est correctement monté, vous voyez la sortie suivante :
 
   ```
   XFS (sda1): Mounting V1 Filesystem
   XFS (sda1): Ending clean mount
   ```

10. Redémarrez la machine virtuelle et vérifiez que le problème est résolu.

    ```
    Reboot -f
    ```

## <a name="repair-the-vm-offline"></a>Réparer la machine virtuelle en mode hors connexion

1. Attachez le disque système de la machine virtuelle en tant que disque de données à une machine virtuelle de récupération (toute machine virtuelle Linux opérationnelle). Pour ce faire, vous pouvez utiliser les [commandes CLI](./troubleshoot-recovery-disks-linux.md) ou automatiser la configuration de la machine virtuelle de récupération à l’aide des [commandes de réparation de machine virtuelle](repair-linux-vm-using-azure-virtual-machine-repair-commands.md).

2. Identifiez le nom de lecteur du disque système que vous avez attaché. Dans le cas présent, nous supposons que le nom du disque système que vous avez attaché est /dev/sdc1. Remplacez-le par la valeur correspondant à votre machine virtuelle.

3. Utilisez xfs_repair avec l’option -n pour détecter les erreurs dans le système de fichiers.

   ```
   xfs_repair -n /dev/sdc1
   ```

4. Exécutez la commande suivante pour réparer le système de fichiers :

   ```
   xfs_repair /dev/sdc1
   ```

5. Si vous recevez le message d’erreur suivant « ERREUR : Le système de fichiers présente des modifications de métadonnées importantes dans un journal qui doit être relu », créez un répertoire temporaire et montez le système de fichiers :

   ```
   mkdir /temp

   mount /dev/sdc1 /temp
   ```

   En cas d’échec du montage du disque, exécutez la commande xfs_repair avec l’option -L (forcer la mise à zéro du journal) :

   ```
   xfs_repair /dev/sdc1 -L
   ```

6. Ensuite, essayez de monter le système de fichiers. Si le disque est correctement monté, vous voyez la sortie suivante :

   ```
   XFS (sdc1): Mounting V1 Filesystem
   
   XFS (sdc1): Ending clean mount
   ```

7. Démontez et détachez le disque dur virtuel d’origine, puis créez une machine virtuelle à partir du disque système d’origine. Pour ce faire, vous pouvez utiliser les [commandes CLI](troubleshoot-recovery-disks-linux.md) ou les [commandes de réparation de machine virtuelle](repair-linux-vm-using-azure-virtual-machine-repair-commands.md) si vous les avez utilisées pour créer la machine virtuelle de récupération.

8. Vérifiez si le problème est résolu.

## <a name="next-steps"></a>Étapes suivantes

* [Résoudre les problèmes d’une machine virtuelle Linux en attachant le disque de système d’exploitation à une machine virtuelle de récupération avec Azure CLI 2.0](./troubleshoot-recovery-disks-linux.md)
* [Utiliser le portail pour attacher un disque de données à une machine virtuelle Linux](../linux/attach-disk-portal.md)
