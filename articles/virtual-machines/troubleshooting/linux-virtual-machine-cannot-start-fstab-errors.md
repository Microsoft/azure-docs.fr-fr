---
title: Résoudre les problèmes de démarrage de machine virtuelle Linux dus à des erreurs fstab | Microsoft Docs
description: Explique pourquoi une machine virtuelle Linux ne peut pas démarrer et comment résoudre ce problème.
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
ms.openlocfilehash: fd49993e6825c47bbae8f034715c03191e06ab2d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89441661"
---
# <a name="troubleshoot-linux-vm-starting-issues-due-to-fstab-errors"></a>Résoudre les problèmes de démarrage de machine virtuelle Linux dus à des erreurs fstab

Vous ne pouvez pas vous connecter à une machine virtuelle Linux Azure avec une connexion Secure Shell (SSH). Quand vous exécutez la fonctionnalité [Diagnostics de démarrage](./boot-diagnostics.md) sur le [portail Azure](https://portal.azure.com/), des entrées de journal semblables aux exemples suivants s’affichent :

## <a name="examples"></a>Exemples

Voici quelques exemples d’erreurs possibles.

### <a name="example-1-a-disk-is-mounted-by-the-scsi-id-instead-of-the-universally-unique-identifier-uuid"></a>Exemple 1 : Un disque est monté avec l’ID SCSI au lieu de l’identificateur unique universel (UUID).

```
[K[[1;31m TIME [0m] Timed out waiting for device dev-incorrect.device.
[[1;33mDEPEND[0m] Dependency failed for /data.
[[1;33mDEPEND[0m] Dependency failed for Local File Systems.
…
Welcome to emergency mode! After logging in, type "journalctl -xb" to viewsystem logs, "systemctl reboot" to reboot, "systemctl default" to try again to boot into default mode.
Give root password for maintenance
(or type Control-D to continue)
```

### <a name="example-2-an-unattached-device-is-missing-on-centos"></a>Exemple 2 : Un appareil non attaché est manquant sur CentOS.

```
Checking file systems…
fsck from util-linux 2.19.1
Checking all file systems.
/dev/sdc1: nonexistent device ("nofail" fstab option may be used to skip this device)
/dev/sdd1: nonexistent device ("nofail" fstab option may be used to skip this device)
/dev/sde1: nonexistent device ("nofail" fstab option may be used to skip this device)

[/sbin/fsck.ext3 (1) — /CODE] sck.ext3 -a /dev/sdc1
fsck.ext3: No such file or directory while trying to open /dev/sdc1

/dev/sdc1:
The superblock could not be read or does not describe a correct ext2
filesystem. If the device is valid and it really contains an ext2
filesystem (and not swap or ufs or something else), then the superblock
is corrupt, and you might try running e2fsck with an alternate superblock:

e2fsck -b 8193 <device>

[/sbin/fsck.xfs (1) — /GLUSTERDISK] fsck.xfs -a /dev/sdd1
/sbin/fsck.xfs: /dev/sdd1 does not exist
[/sbin/fsck.ext3 (1) — /DATATEMP] fsck.ext3 -a /dev/sde1 fsck.ext3: No such file or directory while trying to open /dev/sde1
```

### <a name="example-3-a-vm-cannot-start-because-of-an-fstab-misconfiguration-or-because-the-disk-is-no-longer-attached"></a>Exemple 3 : Une machine virtuelle ne peut pas démarrer en raison d’une configuration fstab incorrecte ou parce que le disque n’est plus attaché.

```
The disk drive for /var/lib/mysql is not ready yet or not present.
Continue to wait, or Press S to skip mounting or M for manual recovery
```

### <a name="example-4-a-serial-log-entry-shows-an-incorrect-uuid"></a>Exemple 4 : Une entrée de journal série indique un UUID incorrect.

```
Checking filesystems
Checking all file systems.
[/sbin/fsck.ext4 (1) — /] fsck.ext4 -a /dev/sda1
/dev/sda1: clean, 70442/1905008 files, 800094/7608064 blocks
[/sbin/fsck.ext4 (1) — /datadrive] fsck.ext4 -a UUID="<UUID>"
fsck.ext4: Unable to resolve UUID="<UUID>"
[FAILED

*** An error occurred during the file system check.
*** Dropping you to a shell; the system will reboot
*** when you leave the shell.
*** Warning — SELinux is active
*** Disabling security enforcement for system recovery.
*** Run 'setenforce 1' to reenable.
type=1404 audit(1428047455.949:4): enforcing=0 old_enforcing=1 auid=<AUID> ses=4294967295
Give root password for maintenance
(or type Control-D to continue)
```

Ce problème peut se produire si la syntaxe de table de systèmes de fichiers (fstab, file systems table) est incorrecte ou si un disque de données requis mappé à une entrée dans le fichier « /etc/fstab » n’est pas attaché à la machine virtuelle.

## <a name="resolution"></a>Résolution

Pour résoudre ce problème, démarrez la machine virtuelle en mode d’urgence à l’aide de la console série pour les machines virtuelles Azure. Puis, utilisez cet outil pour réparer le système de fichiers. Si la console série n’est pas activée sur votre machine virtuelle, accédez à la section [Réparer la machine virtuelle en mode hors connexion](#repair-the-vm-offline).

## <a name="use-the-serial-console"></a>Utiliser la console série

### <a name="using-single-user-mode"></a>Utilisation du mode mono-utilisateur

1. Connectez-vous à la [console série](./serial-console-linux.md).
2. Utiliser la console série pour passer en mode mono-utilisateur [mode mono-utilisateur](serial-console-grub-single-user-mode.md)
3. Une fois que la machine virtuelle a démarré en mode mono-utilisateur. Ouvrez le fichier fstab dans l’éditeur de texte de votre choix. 

   ```
   # nano /etc/fstab
   ```

4. Passez en revue les systèmes de fichiers listés. Chaque ligne du fichier fstab indique un système de fichiers monté au démarrage de la machine virtuelle. Pour plus d’informations sur la syntaxe du fichier fstab, exécutez la commande fstab man. Pour résoudre un problème de démarrage, passez en revue chaque ligne pour vous assurer qu’elle est correcte en termes de structure et de contenu.

   > [!Note]
   > * Les champs de chaque ligne sont séparés par des tabulations ou des espaces. Les lignes vides sont ignorées. Les lignes ayant comme premier caractère le signe dièse (#) sont des commentaires. Les lignes en commentaire peuvent être conservées dans le fichier fstab, mais elles ne sont pas traitées. Quand vous avez un doute au sujet d’une ligne fstab, nous vous recommandons de la mettre en commentaire plutôt que de la supprimer.
   > * Les seules partitions nécessaires à la récupération et au démarrage de la machine virtuelle doivent être les partitions de système de fichiers. La machine virtuelle peut rencontrer des erreurs d’application liées aux autres partitions commentées. Toutefois, la machine virtuelle devrait démarrer sans ces partitions. Vous pourrez, par la suite, supprimer les commentaires des lignes concernées.
   > * Nous vous recommandons de monter les disques de données sur les machines virtuelles Azure à l’aide de l’UUID de la partition de système de fichiers. Par exemple, exécutez la commande suivante : ``/dev/sdc1: LABEL="cloudimg-rootfs" UUID="<UUID>" TYPE="ext4" PARTUUID="<PartUUID>"``
   > * Pour déterminer l’UUID du système de fichiers, exécutez la commande blkid. Pour plus d’informations sur la syntaxe, exécutez la commande blkid man.
   > * L’option nofail permet de s’assurer que la machine virtuelle démarre même si le système de fichiers est endommagé ou n’existe pas au démarrage. Nous vous recommandons d’utiliser l’option nofail dans le fichier fstab. Le démarrage de la machine virtuelle pourra ainsi se poursuive si des partitions qui ne sont pas nécessaires au démarrage présentent des erreurs.

5. Modifiez ou mettez en commentaire les lignes incorrectes ou superflues dans le fichier fstab pour permettre à la machine virtuelle de démarrer correctement.

6. Enregistrez les modifications apportées au fichier fstab.

7. Redémarrez la machine virtuelle à l’aide de la commande ci-dessous.
   
   ```
   # reboot -f
   ```
> [!Note]
   > Vous pouvez également utiliser la commande « Ctrl + x » qui redémarre également la machine virtuelle.


8. Si la mise en commentaire ou la correction des entrées a réussi, le système doit afficher une invite Bash dans le portail. Vérifiez si vous pouvez vous connecter à la machine virtuelle.

### <a name="using-root-password"></a>Utilisation du mot de passe racine

1. Connectez-vous à la [console série](./serial-console-linux.md).
2. Connectez-vous au système à l’aide d’un mot de passe en tant qu’utilisateur local.

   > [!Note]
   > Vous ne pouvez pas utiliser de clé SSH pour vous connecter au système sur la console série.

3. Recherchez l’erreur indiquant que le disque n’a pas été monté. Dans l’exemple suivant, le système essayait d’attacher un disque qui n’était plus présent :

   ```
   [DEPEND] Dependency failed for /datadisk1.
   [DEPEND] Dependency failed for Local File Systems.
   [DEPEND] Dependency failed for Relabel all filesystems, if necessary.
   [DEPEND] Dependency failed for Migrate local... structure to the new structure.
   Welcome to emergency mode! After logging in, type "journalctl -xb" to view
   system logs, "systemctl reboot" to reboot, "systemctl default" or ^D to try again to boot into default mode.
   Give root password for maintenance
   (or type Control-D to continue):
   ```

4. Connectez-vous à la machine virtuelle à l’aide du mot de passe racine (machines virtuelles Red Hat).

5. Ouvrez le fichier fstab dans l’éditeur de texte de votre choix. Une fois le disque monté, exécutez la commande suivante pour Nano :

   ```
   $ nano /mnt/troubleshootingdisk/etc/fstab
   ```

6. Passez en revue les systèmes de fichiers listés. Chaque ligne du fichier fstab indique un système de fichiers monté au démarrage de la machine virtuelle. Pour plus d’informations sur la syntaxe du fichier fstab, exécutez la commande fstab man. Pour résoudre un problème de démarrage, passez en revue chaque ligne pour vous assurer qu’elle est correcte en termes de structure et de contenu.

   > [!Note]
   > * Les champs de chaque ligne sont séparés par des tabulations ou des espaces. Les lignes vides sont ignorées. Les lignes ayant comme premier caractère le signe dièse (#) sont des commentaires. Les lignes en commentaire peuvent être conservées dans le fichier fstab, mais elles ne sont pas traitées. Quand vous avez un doute au sujet d’une ligne fstab, nous vous recommandons de la mettre en commentaire plutôt que de la supprimer.
   > * Les seules partitions nécessaires à la récupération et au démarrage de la machine virtuelle doivent être les partitions de système de fichiers. La machine virtuelle peut rencontrer des erreurs d’application liées aux autres partitions commentées. Toutefois, la machine virtuelle devrait démarrer sans ces partitions. Vous pourrez, par la suite, supprimer les commentaires des lignes concernées.
   > * Nous vous recommandons de monter les disques de données sur les machines virtuelles Azure à l’aide de l’UUID de la partition de système de fichiers. Par exemple, exécutez la commande suivante : ``/dev/sdc1: LABEL="cloudimg-rootfs" UUID="<UUID>" TYPE="ext4" PARTUUID="<PartUUID>"``
   > * Pour déterminer l’UUID du système de fichiers, exécutez la commande blkid. Pour plus d’informations sur la syntaxe, exécutez la commande blkid man.
   > * L’option nofail permet de s’assurer que la machine virtuelle démarre même si le système de fichiers est endommagé ou n’existe pas au démarrage. Nous vous recommandons d’utiliser l’option nofail dans le fichier fstab. Le démarrage de la machine virtuelle pourra ainsi se poursuive si des partitions qui ne sont pas nécessaires au démarrage présentent des erreurs.

7. Modifiez ou mettez en commentaire les lignes incorrectes ou superflues dans le fichier fstab pour permettre à la machine virtuelle de démarrer correctement.

8. Enregistrez les modifications apportées au fichier fstab.

9. Redémarrez la machine virtuelle.

10. Si la mise en commentaire ou la correction des entrées a réussi, le système doit afficher une invite Bash dans le portail. Vérifiez si vous pouvez vous connecter à la machine virtuelle.

11. Quand vous testez une modification du fichier fstab, vérifiez vos points de montage en exécutant la commande mount –a. Si aucune erreur ne se produit, vos points de montage doivent être corrects.

## <a name="repair-the-vm-offline"></a>Réparer la machine virtuelle en mode hors connexion

1. Attachez le disque système de la machine virtuelle en tant que disque de données à une machine virtuelle de récupération (toute machine virtuelle Linux opérationnelle). Pour ce faire, vous pouvez utiliser les [commandes CLI](./troubleshoot-recovery-disks-linux.md) ou automatiser la configuration de la machine virtuelle de récupération à l’aide des [commandes de réparation de machine virtuelle](repair-linux-vm-using-azure-virtual-machine-repair-commands.md).

2. Une fois le disque système monté en tant que disque de données sur la machine virtuelle de récupération, sauvegardez le fichier fstab avant modification, puis effectuez les étapes suivantes pour corriger le fichier fstab.

3. Recherchez l’erreur indiquant que le disque n’a pas été monté. Dans l’exemple suivant, le système essayait d’attacher un disque qui n’était plus présent :

   ```output
   [DEPEND] Dependency failed for /datadisk1.
   [DEPEND] Dependency failed for Local File Systems.
   [DEPEND] Dependency failed for Relabel all filesystems, if necessary.
   [DEPEND] Dependency failed for Migrate local... structure to the new structure.
   Welcome to emergency mode! After logging in, type "journalctl -xb" to view system logs, "systemctl reboot" to reboot, "systemctl default" or ^D to try again to boot into default mode.
   Give root password for maintenance (or type Control-D to continue):
   ```

4. Connectez-vous à la machine virtuelle à l’aide du mot de passe racine (machines virtuelles Red Hat).

5. Ouvrez le fichier fstab dans l’éditeur de texte de votre choix. Une fois le disque monté, exécutez la commande suivante pour Nano. Veillez à travailler sur le fichier fstab qui se trouve sur le disque monté et non sur celui qui se trouve sur la machine virtuelle de secours.

   ```
   $ nano /mnt/troubleshootingdisk/etc/fstab
   ```

6. Passez en revue les systèmes de fichiers listés. Chaque ligne du fichier fstab indique un système de fichiers monté au démarrage de la machine virtuelle. Pour plus d’informations sur la syntaxe du fichier fstab, exécutez la commande fstab man. Pour résoudre un problème de démarrage, passez en revue chaque ligne pour vous assurer qu’elle est correcte en termes de structure et de contenu.

   > [!Note]
   > * Les champs de chaque ligne sont séparés par des tabulations ou des espaces. Les lignes vides sont ignorées. Les lignes ayant comme premier caractère le signe dièse (#) sont des commentaires. Les lignes en commentaire peuvent être conservées dans le fichier fstab, mais elles ne sont pas traitées. Quand vous avez un doute au sujet d’une ligne fstab, nous vous recommandons de la mettre en commentaire plutôt que de la supprimer.
   > * Les seules partitions nécessaires à la récupération et au démarrage de la machine virtuelle doivent être les partitions de système de fichiers. La machine virtuelle peut rencontrer des erreurs d’application liées aux autres partitions commentées. Toutefois, la machine virtuelle devrait démarrer sans ces partitions. Vous pourrez, par la suite, supprimer les commentaires des lignes concernées.
   > * Nous vous recommandons de monter les disques de données sur les machines virtuelles Azure à l’aide de l’UUID de la partition de système de fichiers. Par exemple, exécutez la commande suivante : ``/dev/sdc1: LABEL="cloudimg-rootfs" UUID="<UUID>" TYPE="ext4" PARTUUID="<PartUUID>"``
   > * Pour déterminer l’UUID du système de fichiers, exécutez la commande blkid. Pour plus d’informations sur la syntaxe, exécutez la commande blkid man. Notez que le disque que vous souhaitez récupérer est maintenant monté sur une nouvelle machine virtuelle. Même si les UUID doivent être cohérents, les ID de partition d’appareil (par exemple, « /dev/sda1 ») sont différents sur cette machine virtuelle. Les partitions de système de fichiers de la machine virtuelle d’origine défaillante situées sur un disque dur virtuel non-système ne sont pas disponibles sur la machine virtuelle de récupération [avec les commandes CLI](./troubleshoot-recovery-disks-linux.md).
   > * L’option nofail permet de s’assurer que la machine virtuelle démarre même si le système de fichiers est endommagé ou n’existe pas au démarrage. Nous vous recommandons d’utiliser l’option nofail dans le fichier fstab. Le démarrage de la machine virtuelle pourra ainsi se poursuive si des partitions qui ne sont pas nécessaires au démarrage présentent des erreurs.

7. Modifiez ou mettez en commentaire les lignes incorrectes ou superflues dans le fichier fstab pour permettre à la machine virtuelle de démarrer correctement.

8. Enregistrez les modifications apportées au fichier fstab.

9. Redémarrez la machine virtuelle ou regénérez la machine virtuelle d’origine.

10. Si la mise en commentaire ou la correction des entrées a réussi, le système doit afficher une invite Bash dans le portail. Vérifiez si vous pouvez vous connecter à la machine virtuelle.

11. Quand vous testez une modification du fichier fstab, vérifiez vos points de montage en exécutant la commande mount –a. Si aucune erreur ne se produit, vos points de montage doivent être corrects.

12. Démontez et détachez le disque dur virtuel d’origine, puis créez une machine virtuelle à partir du disque système d’origine. Pour ce faire, vous pouvez utiliser les [commandes CLI](troubleshoot-recovery-disks-linux.md) ou les [commandes de réparation de machine virtuelle](repair-linux-vm-using-azure-virtual-machine-repair-commands.md) si vous les avez utilisées pour créer la machine virtuelle de récupération.

13. Une fois que vous avez regénéré la machine virtuelle et que vous pouvez vous y connecter avec SSH, effectuez les actions suivantes :
    * Passez en revue les lignes fstab qui ont été modifiées ou mises en commentaire pendant la récupération.
    * Assurez-vous que vous utilisez correctement l’UUID et l’option nofail.
    * Testez les modifications apportées au fichier fstab avant de redémarrer la machine virtuelle. Pour ce faire, utilisez la commande suivante : ``$ sudo mount -a``
    * Créez une copie supplémentaire du fichier fstab corrigé pour l’utiliser dans d’autres scénarios de récupération.

## <a name="next-steps"></a>Étapes suivantes

* [Résoudre les problèmes d’une machine virtuelle Linux en attachant le disque de système d’exploitation à une machine virtuelle de récupération avec Azure CLI 2.0](./troubleshoot-recovery-disks-linux.md)
* [Résoudre les problèmes d’une machine virtuelle Linux en connectant le disque du système d’exploitation à une machine virtuelle de récupération à l’aide du portail Azure](./troubleshoot-recovery-disks-portal-linux.md)
