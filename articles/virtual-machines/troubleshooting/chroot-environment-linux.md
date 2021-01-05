---
title: Environnement chroot dans une machine virtuelle Linux de secours.
description: Cet article explique comment détecter les problèmes dans l’environnement chroot de la machine virtuelle de secours dans Linux.
mservices: virtual-machines-linux
documentationcenter: ''
author: kailashmsft
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines-linux
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.date: 05/05/2020
ms.author: kaib
ms.openlocfilehash: 0d8a030061ef6aa848344152edaa3267ad916e2a
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93377936"
---
# <a name="chroot-environment-in-a-linux-rescue-vm"></a>Environnement chroot dans une machine virtuelle Linux de secours

Cet article explique comment détecter les problèmes dans l’environnement chroot de la machine virtuelle de secours dans Linux.

## <a name="ubuntu-16x--ubuntu-18x"></a>Ubuntu 16.x et Ubuntu 18.x

1. Arrêtez ou désallouez la machine virtuelle concernée.
1. Créez une image de machine virtuelle de secours avec la même version du système d’exploitation, dans le même groupe de ressources (RSG) et dans le même emplacement à l’aide du disque managé.
1. Utilisez le Portail Azure pour prendre un instantané du disque du système d’exploitation de la machine virtuelle concernée.
1. Créez un disque à partir de la capture instantanée du disque du système d’exploitation et attachez-le à la machine virtuelle de secours.
1. Une fois le disque créé, résolvez les problèmes liés à l’environnement chroot dans la machine virtuelle de secours.

   1. Accédez à votre machine virtuelle en tant qu’utilisateur racine à l’aide de la commande suivante :

      `sudo su -`

   1. Recherchez le disque à l’aide de `dmesg` (la méthode que vous utilisez pour découvrir votre nouveau disque peut varier). L’exemple suivant utilise **dmesg** pour filtrer les disques **SCSI**  :

      `dmesg | grep SCSI`

      Votre résultat doit ressembler à l’exemple qui suit. Dans cet exemple, nous recherchons le disque **sdc**  :

      ```
      [    0.294784] SCSI subsystem initialized
      [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
      [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
      [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
      [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
      ```

   1. Utilisez les commandes suivantes pour accéder à l’environnement chroot :

      ```
      mkdir /rescue
      mount /dev/sdc1 /rescue
      mount /dev/sdc15 /rescue/boot/efi
      cd /rescue

      mount -t proc proc proc
      mount -t sysfs sys sys/
      mount -o bind /dev dev/
      mount -o bind /dev/pts dev/pts/
      mount -o bind /run run/
      chroot /rescue
      ```

   1. Résoudre les problèmes liés à l’environnement chroot.

   1. Utilisez les commandes suivantes pour quitter l’environnement chroot :

      ```
      exit

      umount /rescue/proc/
      umount /rescue/sys/
      umount /rescue/dev/pts
      umount /rescue/dev/
      umount /rescue/run
      cd /
      umount /rescue/boot/efi
      umount /rescue
      ```

      > [!NOTE]
      > Si vous recevez le message d’erreur `unable to unmount /rescue`, ajoutez l’option -l à la commande umount.
      >
      > Exemple : `umount -l /rescue`

1. Détachez le disque de la machine virtuelle de secours et échangez le disque avec la machine virtuelle d’origine.
1. Démarrez la machine virtuelle d’origine et vérifiez sa connectivité.

## <a name="rhelcentosoracle-6x--oracle-8x--rhelcentos-7x-with-raw-partitions"></a>RHEL/Centos/Oracle 6.x et Oracle 8.x et RHEL/Centos 7.x avec partitions brutes

1. Arrêtez ou désallouez la machine virtuelle concernée.
1. Créez une image de machine virtuelle de secours avec la même version du système d’exploitation, dans le même groupe de ressources (RSG) et dans le même emplacement à l’aide du disque managé.
1. Utilisez le Portail Azure pour prendre un instantané du disque du système d’exploitation de la machine virtuelle concernée.
1. Créez un disque à partir de la capture instantanée du disque du système d’exploitation et attachez-le à la machine virtuelle de secours.
1. Une fois le disque créé, résolvez les problèmes liés à l’environnement chroot dans la machine virtuelle de secours.

   1. Accédez à votre machine virtuelle en tant qu’utilisateur racine à l’aide de la commande suivante :

      `sudo su -`

   1. Recherchez le disque à l’aide de `dmesg` (la méthode que vous utilisez pour découvrir votre nouveau disque peut varier). L’exemple suivant utilise **dmesg** pour filtrer les disques **SCSI**  :

      `dmesg | grep SCSI`

      Votre résultat doit ressembler à l’exemple qui suit. Dans cet exemple, nous recherchons le disque **sdc**  :

      ```
      [    0.294784] SCSI subsystem initialized
      [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
      [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
      [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
      [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
      ```

   1. Utilisez les commandes suivantes pour accéder à l’environnement chroot :

      ```
      mkdir /rescue
      mount -o nouuid /dev/sdc2 /rescue
      mount -o nouuid /dev/sdc1 /rescue/boot/
      cd /rescue

      mount -t proc proc proc
      mount -t sysfs sys sys/
      mount -o bind /dev dev/
      mount -o bind /dev/pts dev/pts/
      mount -o bind /run run/
      chroot /rescue
      ```

   1. Résoudre les problèmes liés à l’environnement chroot.

   1. Utilisez les commandes suivantes pour quitter l’environnement chroot :

      ```
      exit

      umount /rescue/proc/
      umount /rescue/sys/
      umount /rescue/dev/pts
      umount /rescue/dev/
      umount /rescue/run
      cd /
      umount /rescue/boot/
      umount /rescue
      ```

      > [!NOTE]
      > Si vous recevez le message d’erreur `unable to unmount /rescue`, ajoutez l’option -l à la commande umount.
      >
      > Exemple : `umount -l /rescue`

1. Détachez le disque de la machine virtuelle de secours et échangez le disque avec la machine virtuelle d’origine.
1. Démarrez la machine virtuelle d’origine et vérifiez sa connectivité.

## <a name="rhelcentos-7x-with-lvm"></a>RHEL/CentOS 7.x avec LVM

   > [!NOTE]
   > Si votre machine virtuelle d’origine comprend le gestionnaire de volumes logiques (LVM) sur le disque du système d’exploitation, créez la machine virtuelle de secours à l’aide de l’image avec des partitions brutes sur le disque du système d’exploitation.

1. Arrêtez ou désallouez la machine virtuelle concernée.
1. Créez une image de machine virtuelle de secours avec la même version du système d’exploitation, dans le même groupe de ressources (RSG) et dans le même emplacement à l’aide du disque managé.
1. Utilisez le Portail Azure pour prendre un instantané du disque du système d’exploitation de la machine virtuelle concernée.
1. Créez un disque à partir de la capture instantanée du disque du système d’exploitation et attachez-le à la machine virtuelle de secours.
1. Une fois le disque créé, résolvez les problèmes liés à l’environnement chroot dans la machine virtuelle de secours.

   1. Accédez à votre machine virtuelle en tant qu’utilisateur racine à l’aide de la commande suivante :

      `sudo su -`

   1. Recherchez le disque à l’aide de `dmesg` (la méthode que vous utilisez pour découvrir votre nouveau disque peut varier). L’exemple suivant utilise **dmesg** pour filtrer les disques **SCSI**  :

      `dmesg | grep SCSI`

      Votre résultat doit ressembler à l’exemple qui suit. Dans cet exemple, nous recherchons le disque **sdc**  :

      ```
      [    0.294784] SCSI subsystem initialized
      [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
      [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
      [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
      [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
      ```

   1. Utilisez la commande suivante pour activer le groupe de volumes logiques :

      ```
      vgscan --mknodes
      vgchange -ay
      lvscan
      ```

   1. Utilisez la commande `lsblk` pour récupérer les noms LVM :

      ```
      [user@myvm ~]$ lsblk
      NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
      sda                 8:0    0   64G  0 disk
      ├─sda1              8:1    0  500M  0 part /boot
      ├─sda2              8:2    0   63G  0 part /
      sdb                 8:16   0    4G  0 disk
      └─sdb1              8:17   0    4G  0 part /mnt/resource
      sdc                 8:0    0   64G  0 disk
      ├─sdc1              8:1    0  500M  0 part
      ├─sdc2              8:2    0   63G  0 part
      ├─sdc3              8:3    0    2M  0 part
      ├─sdc4              8:4    0   63G  0 part
        ├─rootvg-tmplv  253:0    0    2G  0 lvm  
        ├─rootvg-usrlv  253:1    0   10G  0 lvm  
        ├─rootvg-optlv  253:2    0    2G  0 lvm  
        ├─rootvg-homelv 253:3    0    1G  0 lvm  
        ├─rootvg-varlv  253:4    0    8G  0 lvm  
        └─rootvg-rootlv 253:5    0    2G  0 lvm
      ```

   1. Utilisez les commandes suivantes pour accéder à l’environnement chroot :

      ```
      mkdir /rescue
      mount /dev/mapper/rootvg-rootlv /rescue
      mount /dev/mapper/rootvg-varlv /rescue/var
      mount /dev/mapper/rootvg-homelv /rescue/home
      mount /dev/mapper/rootvg-usrlv /rescue/usr
      mount /dev/mapper/rootvg-tmplv /rescue/tmp
      mount /dev/mapper/rootvg-optlv /rescue/opt
      mount /dev/sdc2 /rescue/boot/
      mount /dev/sdc1 /rescue/boot/efi
      cd /rescue

      mount -t proc proc proc
      mount -t sysfs sys sys/
      mount -o bind /dev dev/
      mount -o bind /dev/pts dev/pts/
      mount -o bind /run run/
      chroot /rescue
      ```

   1. Résoudre les problèmes liés à l’environnement chroot.

   1. Utilisez les commandes suivantes pour quitter l’environnement chroot :

      ```
      exit

      umount /rescue/proc/
      umount /rescue/sys/
      umount /rescue/dev/pts
      umount /rescue/dev/
      umount /rescue/run
      cd /
      umount /rescue/boot/efi
      umount /rescue/boot
      umount /rescue/home
      umount /rescue/var
      umount /rescue/usr
      umount /rescue/tmp
      umount /rescue/opt
      umount /rescue
      ```

      > [!NOTE]
      > Si vous recevez le message d’erreur `unable to unmount /rescue`, ajoutez l’option -l à la commande umount.
      >
      > Exemple : `umount -l /rescue`

1. Détachez le disque de la machine virtuelle de secours et échangez le disque avec la machine virtuelle d’origine.
1. Démarrez la machine virtuelle d’origine et vérifiez sa connectivité.

## <a name="rhel-8x-with-lvm"></a>RHEL 8.x avec LVM

   > [!NOTE]
   > Si votre machine virtuelle d’origine comprend le gestionnaire de volumes logiques (LVM) sur le disque du système d’exploitation, créez la machine virtuelle de secours à l’aide de l’image avec des partitions brutes sur le disque du système d’exploitation.

1. Arrêtez ou désallouez la machine virtuelle concernée.
1. Créez une image de machine virtuelle de secours avec la même version du système d’exploitation, dans le même groupe de ressources (RSG) et dans le même emplacement à l’aide du disque managé.
1. Utilisez le Portail Azure pour prendre un instantané du disque du système d’exploitation de la machine virtuelle concernée.
1. Créez un disque à partir de la capture instantanée du disque du système d’exploitation et attachez-le à la machine virtuelle de secours.
1. Une fois le disque créé, résolvez les problèmes liés à l’environnement chroot dans la machine virtuelle de secours.

   1. Accédez à votre machine virtuelle en tant qu’utilisateur racine à l’aide de la commande suivante :

      `sudo su -`

   1. Recherchez le disque à l’aide de `dmesg` (la méthode que vous utilisez pour découvrir votre nouveau disque peut varier). L’exemple suivant utilise **dmesg** pour filtrer les disques **SCSI**  :

      `dmesg | grep SCSI`

      Votre résultat doit ressembler à l’exemple qui suit. Dans cet exemple, nous recherchons le disque **sdc**  :

      ```
      [    0.294784] SCSI subsystem initialized
      [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
      [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
      [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
      [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
      ```

   1. Utilisez la commande suivante pour activer le groupe de volumes logiques :

      ```
      vgscan --mknodes
      vgchange -ay
      lvscan
      ```

   1. Utilisez la commande `lsblk` pour récupérer les noms LVM :

      ```
      [user@myvm ~]$ lsblk
      NAME              MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
      sda                 8:0    0   64G  0 disk
      ├─sda1              8:1    0  500M  0 part /boot
      ├─sda2              8:2    0   63G  0 part /
      sdb                 8:16   0    4G  0 disk
      └─sdb1              8:17   0    4G  0 part /mnt/resource
      sdc                 8:0    0   64G  0 disk
      ├─sdc1              8:1    0  500M  0 part
      ├─sdc2              8:2    0   63G  0 part
      │ ├─rootvg-tmplv  253:0    0    2G  0 lvm  
      │ ├─rootvg-usrlv  253:1    0   10G  0 lvm  
      │ ├─rootvg-homelv 253:2    0    1G  0 lvm  
      │ ├─rootvg-varlv  253:3    0    8G  0 lvm  
      │ └─rootvg-rootlv 253:4    0    2G  0 lvm  
      ├─sdc14             8:14   0    4M  0 part
      └─sdc15             8:15   0  495M  0 part
      ```

   1. Utilisez les commandes suivantes pour accéder à l’environnement chroot :

      ```
      mkdir /rescue
      mount /dev/mapper/rootvg-rootlv /rescue
      mount /dev/mapper/rootvg-varlv /rescue/var
      mount /dev/mapper/rootvg-homelv /rescue/home
      mount /dev/mapper/rootvg-usrlv /rescue/usr
      mount /dev/mapper/rootvg-tmplv /rescue/tmp
      mount /dev/sdc1 /rescue/boot/
      mount /dev/sdc15 /rescue/boot/efi
      cd /rescue

      mount -t proc proc proc
      mount -t sysfs sys sys/
      mount -o bind /dev dev/
      mount -o bind /dev/pts dev/pts/
      mount -o bind /run run/
      chroot /rescue
      ```

   1. Résoudre les problèmes liés à l’environnement chroot.

   1. Utilisez les commandes suivantes pour quitter l’environnement chroot :

      ```
      exit

      umount /rescue/proc/
      umount /rescue/sys/
      umount /rescue/dev/pts
      umount /rescue/dev/
      umount /rescue/run
      cd /
      umount /rescue/boot/efi
      umount /rescue/boot
      umount /rescue/home
      umount /rescue/var
      umount /rescue/usr
      umount /rescue/tmp
      umount /rescue
      ```

      > [!NOTE]
      > Si vous recevez le message d’erreur `unable to unmount /rescue`, ajoutez l’option -l à la commande umount.
      >
      > Exemple : `umount -l /rescue`

1. Détachez le disque de la machine virtuelle de secours et échangez le disque avec la machine virtuelle d’origine.
1. Démarrez la machine virtuelle d’origine et vérifiez sa connectivité.

## <a name="oracle-7x"></a>Oracle 7.x

1. Arrêtez ou désallouez la machine virtuelle concernée.
1. Créez une image de machine virtuelle de secours avec la même version du système d’exploitation, dans le même groupe de ressources (RSG) et dans le même emplacement à l’aide du disque managé.
1. Utilisez le Portail Azure pour prendre un instantané du disque du système d’exploitation de la machine virtuelle concernée.
1. Créez un disque à partir de la capture instantanée du disque du système d’exploitation et attachez-le à la machine virtuelle de secours.
1. Une fois le disque créé, résolvez les problèmes liés à l’environnement chroot dans la machine virtuelle de secours.

   1. Accédez à votre machine virtuelle en tant qu’utilisateur racine à l’aide de la commande suivante :

      `sudo su -`

   1. Recherchez le disque à l’aide de `dmesg` (la méthode que vous utilisez pour découvrir votre nouveau disque peut varier). L’exemple suivant utilise **dmesg** pour filtrer les disques **SCSI**  :

      `dmesg | grep SCSI`

      Votre résultat doit ressembler à l’exemple qui suit. Dans cet exemple, nous recherchons le disque **sdc**  :

      ```
      [    0.294784] SCSI subsystem initialized
      [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
      [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
      [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
      [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
      ```

   1. Utilisez les commandes suivantes pour accéder à l’environnement chroot :

      ```
      mkdir /rescue
      mount -o nouuid /dev/sdc2 /rescue
      mount -o nouuid /dev/sdc1 /rescue/boot/
      mount /dev/sdc15 /rescue/boot/efi
      cd /rescue

      mount -t proc proc proc
      mount -t sysfs sys sys/
      mount -o bind /dev dev/
      mount -o bind /dev/pts dev/pts/
      mount -o bind /run run/
      chroot /rescue
      ```

   1. Résoudre les problèmes liés à l’environnement chroot.

   1. Utilisez les commandes suivantes pour quitter l’environnement chroot :

      ```
      exit

      umount /rescue/proc/
      umount /rescue/sys/
      umount /rescue/dev/pts
      umount /rescue/dev/
      umount /rescue/run
      cd /
      umount /rescue/boot/efi
      umount /rescue/boot
      umount /rescue
      ```

      > [!NOTE]
      > Si vous recevez le message d’erreur `unable to unmount /rescue`, ajoutez l’option -l à la commande umount.
      >
      > Exemple : `umount -l /rescue`

1. Détachez le disque de la machine virtuelle de secours et échangez le disque avec la machine virtuelle d’origine.
1. Démarrez la machine virtuelle d’origine et vérifiez sa connectivité.

## <a name="suse-sles-12-sp4-suse-sles-12-sp4-for-sap---suse-sles-15-sp1-suse-sles-15-sp1-for-sap"></a>SUSE-SLES 12 SP4, SUSE-SLES 12 SP4 For SAP et ## SUSE-SLES 15 SP1, SUSE-SLES 15 SP1 For SAP

1. Arrêtez ou désallouez la machine virtuelle concernée.
1. Créez une image de machine virtuelle de secours avec la même version du système d’exploitation, dans le même groupe de ressources (RSG) et dans le même emplacement à l’aide du disque managé.
1. Utilisez le Portail Azure pour prendre un instantané du disque du système d’exploitation de la machine virtuelle concernée.
1. Créez un disque à partir de la capture instantanée du disque du système d’exploitation et attachez-le à la machine virtuelle de secours.
1. Une fois le disque créé, résolvez les problèmes liés à l’environnement chroot dans la machine virtuelle de secours.

   1. Accédez à votre machine virtuelle en tant qu’utilisateur racine à l’aide de la commande suivante :

      `sudo su -`

   1. Recherchez le disque à l’aide de `dmesg` (la méthode que vous utilisez pour découvrir votre nouveau disque peut varier). L’exemple suivant utilise **dmesg** pour filtrer les disques **SCSI**  :

      `dmesg | grep SCSI`

      Votre résultat doit ressembler à l’exemple qui suit. Dans cet exemple, nous recherchons le disque **sdc**  :

      ```
      [    0.294784] SCSI subsystem initialized
      [    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
      [    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
      [    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
      [ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
      ```

   1. Utilisez les commandes suivantes pour accéder à l’environnement chroot :

      ```
      mkdir /rescue
      mount -o nouuid /dev/sdc4 /rescue
      mount -o nouuid /dev/sdc3 /rescue/boot/
      mount /dev/sdc2 /rescue/boot/efi
      cd /rescue

      mount -t proc proc proc
      mount -t sysfs sys sys/
      mount -o bind /dev dev/
      mount -o bind /dev/pts dev/pts/
      mount -o bind /run run/
      chroot /rescue
      ```

   1. Résoudre les problèmes liés à l’environnement chroot.

   1. Utilisez les commandes suivantes pour quitter l’environnement chroot :

      ```
      exit

      umount /rescue/proc/
      umount /rescue/sys/
      umount /rescue/dev/pts
      umount /rescue/dev/
      umount /rescue/run
      cd /
      umount /rescue/boot/efi
      umount /rescue/boot
      umount /rescue
      ```

      > [!NOTE]
      > Si vous recevez le message d’erreur `unable to unmount /rescue`, ajoutez l’option -l à la commande umount.
      >
      > Exemple : `umount -l /rescue`

1. Détachez le disque de la machine virtuelle de secours et échangez le disque avec la machine virtuelle d’origine.
1. Démarrez la machine virtuelle d’origine et vérifiez sa connectivité.

## <a name="next-steps"></a>Étapes suivantes

- [Résoudre les problèmes de connexion SSH](troubleshoot-ssh-connection.md)
