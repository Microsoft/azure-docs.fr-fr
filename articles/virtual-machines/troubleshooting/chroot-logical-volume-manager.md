---
title: Récupérer des machines virtuelles Linux à l’aide de chroot où LVM (Logical Volume Manager) est utilisé – Machines virtuelles Azure
description: Récupération de machines virtuelles Linux à l’aide de LVM.
services: virtual-machines-linux
documentationcenter: ''
author: vilibert
manager: spogge
editor: ''
tags: Linux chroot LVM
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 11/24/2019
ms.author: vilibert
ms.openlocfilehash: 390443874ea63a8661ef8baea627015fcf679719
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002695"
---
# <a name="troubleshooting-a-linux-vm-when-there-is-no-access-to-the-azure-serial-console-and-the-disk-layout-is-using-lvm-logical-volume-manager"></a>Résolution des problèmes d’une machine virtuelle Linux quand il n’y a aucun accès à la console série Azure et que la disposition du disque utilise LVM (Logical Volume Manager)

Ce guide de résolution des problèmes présente un avantage dans les scénarios où une machine virtuelle Linux ne démarre pas, que SSH n’est pas possible et que la disposition sous-jacente du système de fichiers est configurée avec LVM (Logical Volume Manager).

## <a name="take-snapshot-of-the-failing-vm"></a>Effectuer un instantané de la machine virtuelle en échec

Effectuer un instantané de la machine virtuelle concernée. 

L’instantané est ensuite attaché à une machine virtuelle **de secours**. Suivez les instructions décrites [ici](../linux/snapshot-copy-managed-disk.md#use-azure-portal) sur la façon d’effectuer un **instantané**.

## <a name="create-a-rescue-vm"></a>Créer une machine virtuelle de secours
En règle générale, il est recommandé d’avoir une machine virtuelle de secours dont la version du système d’exploitation est identique ou similaire. Utiliser les mêmes **région** et **groupe de ressources** que la machine virtuelle concernée

## <a name="connect-to-the-rescue-vm"></a>Se connecter à la machine virtuelle de secours
Se connecter à l’aide de SSH sur la machine virtuelle **de secours**. Élever les privilèges et devenir super utilisateur à l’aide de

`sudo su -`

## <a name="attach-the-disk"></a>Attacher le disque
Attachez un disque à la machine virtuelle **de secours** créée à partir de l’instantané effectué précédemment.

Portail Azure -> sélectionnez la machine virtuelle **de secours** -> **Disques** 

![Créer le disque](./media/chroot-logical-volume-manager/create-disk-from-snap.png)

Renseignez les champs. Attribuez un nom à votre nouveau disque, sélectionnez le même groupe de ressources que l’instantané, la machine virtuelle concerné et la machine virtuelle de secours.

Le **type de source** est **Instantané**.
L’**Instantané source** est le nom de l’**instantané** précédemment créé.

![créer le disque 2](./media/chroot-logical-volume-manager/create-disk-from-snap-2.png)

Créez un point de montage pour le disque attaché.

`mkdir /rescue`

Exécutez la commande **fdisk -l** pour vérifier que le disque d’instantané a été attaché et répertorier tous les appareils et toutes les partitions disponibles

`fdisk -l`

Dans la plupart des scénarios, le disque d’instantané attaché sera vu comme **/dev/sdc** affichant deux partitions **/dev/sdc1** et **/dev/sdc2**

![Fdisk](./media/chroot-logical-volume-manager/fdisk-output-sdc.png)

La * *\** _ indique une partition de démarrage ; les deux partitions doivent être montées.

Exécutez la commande _ *lsblk** pour voir les LVM de la machine virtuelle concernée

`lsblk`

![Capture d’écran montrant la sortie de la commande lsblk.](./media/chroot-logical-volume-manager/lsblk-output-mounted.png)


Vérifiez si les LVM de la machine virtuelle concernée s’affichent.
Si ce n’est pas le cas, utilisez les commandes ci-dessous pour les activer et réexécutez **lsblk**.
Assurez-vous que les LVM du disque attaché sont visibles avant de continuer.

```
vgscan --mknodes
vgchange -ay
lvscan
mount –a
lsblk
```

Localisez le chemin d’accès pour monter le volume logique qui contient la partition / (racine). Il contient les fichiers config tels que /etc/default/grub

Dans cet exemple, prendre la sortie de la commande **lsblk** précédente **rootvg-rootlv** est le bon volume logique **racine** à monter et peut être utilisé dans la commande suivante.

La sortie de la commande suivante affiche le chemin d’accès à monter pour le volume logique **racine**

`pvdisplay -m | grep -i rootlv`

![Rootlv](./media/chroot-logical-volume-manager/locate-rootlv.png)

Procédez au montage de cet appareil sur le répertoire /rescue

`mount /dev/rootvg/rootlv /rescue`

Montez la partition qui a l’**indicateur de démarrage** défini sur /rescue/boot

`
mount /dev/sdc1 /rescue/boot
`

Vérifiez que les systèmes de fichiers du disque attaché sont maintenant correctement montés à l’aide de la commande **lsblk**

![Exécuter lsblk](./media/chroot-logical-volume-manager/lsblk-output-1.png)

ou la commande **df -Th**

![Df](./media/chroot-logical-volume-manager/df-output.png)

## <a name="gaining-chroot-access"></a>Obtention de l’accès chroot

Obtenez l’accès **chroot**, ce qui vous permettra d’effectuer différents correctifs ; de légères variations existent pour chaque distribution Linux.

```
 cd /rescue
 mount -t proc proc proc
 mount -t sysfs sys sys/
 mount -o bind /dev dev/
 mount -o bind /dev/pts dev/pts/
 chroot /rescue
```

Si une erreur est rencontrée, par exemple :

**chroot : impossible d’exécuter la commande « /bin/bash » : Pas de fichier ou de répertoire correspondant**

tentative de montage du volume logique **usr**

`
mount  /dev/mapper/rootvg-usrlv /rescue/usr
`

> [!TIP]
> Quand vous exécutez des commandes dans un environnement **chroot**, notez qu’elles sont exécutées sur le disque de système d’exploitation attaché, et non sur la machine virtuelle **de secours** locale. 

Les commandes peuvent être utilisées pour installer, supprimer et mettre à jour des logiciels. Détectez un problème sur les machines virtuelles afin de corriger les erreurs.


Exécutez la commande lsblk : /rescue devient /, et /rescue/boot devient /boot. ![Capture d’écran d’une fenêtre de console avec la commande lsblk et son arborescence de sortie](./media/chroot-logical-volume-manager/chrooted.png)

## <a name="perform-fixes"></a>Effectuer des corrections

### <a name="example-1---configure-the-vm-to-boot-from-a-different-kernel"></a>Exemple 1 : configurer la machine virtuelle pour qu’elle démarre à partir d’un autre noyau

Un scénario courant consiste à forcer une machine virtuelle à démarrer à partir d’un noyau précédent, car le noyau actuellement installé peut être corrompu ou une mise à niveau n’a pas été effectuée correctement.


```
cd /boot/grub2

grep -i linux grub.cfg

grub2-editenv list

grub2-set-default "CentOS Linux (3.10.0-1062.1.1.el7.x86_64) 7 (Core)"

grub2-editenv list

grub2-mkconfig -o /boot/grub2/grub.cfg
```

*walkthrough*

La commande **grep** répertorie les noyaux dont **grub.cfg** a connaissance.
![Capture d’écran d’une fenêtre de console affichant le résultat d’une recherche grep de noyaux](./media/chroot-logical-volume-manager/kernels.png)

**grub2-editenv list** affiche le noyau qui sera chargé au prochain démarrage ![Noyau par défaut](./media/chroot-logical-volume-manager/kernel-default.png)

**grub2-set-default** permet de basculer sur un autre noyau ![Ensemble grub2](./media/chroot-logical-volume-manager/grub2-set-default.png)

La liste **grub2-editenv** affiche le noyau qui sera chargé au prochain démarrage ![Nouveau noyau](./media/chroot-logical-volume-manager/kernel-new.png)

**grub2-mkconfig** regénère grub.cfg à l’aide des versions requises ![Mkconfig grub2](./media/chroot-logical-volume-manager/grub2-mkconfig.png)



### <a name="example-2---upgrade-packages"></a>Exemple 2 : packages de mise à niveau

Une mise à niveau du noyau qui a échoué peut rendre la machine virtuelle non démarrable.
Monter tous les volumes logiques pour autoriser la suppression ou la réinstallation des packages

Exécutez la commande **lvs** pour vérifier quels **volumes logiques** sont disponibles pour le montage ; chaque machine virtuelle qui a été migrée ou provient d’un autre fournisseur de cloud varie en fonction de la configuration.

Quittez l’environnement **chroot** et montez le **volume logique** requis

![Capture d’écran d’une fenêtre de console avec une commande lvs, puis le montage d’un volume logique](./media/chroot-logical-volume-manager/advanced.png)

À présent, accédez de nouveau à l’environnement **chroot** en exécutant

`chroot /rescue`

Tous les volumes logiques doivent être visibles en tant que partitions montées

![Capture d’écran montrant les volumes logiques en tant que partitions montées.](./media/chroot-logical-volume-manager/chroot-all-mounts.png)

Interroger le **noyau** installé

![Capture d’écran montrant comment interroger le noyau installé.](./media/chroot-logical-volume-manager/rpm-kernel.png)

Si nécessaire, supprimez ou mettez à niveau le **noyau**
![Avancé](./media/chroot-logical-volume-manager/rpm-remove-kernel.png)


### <a name="example-3---enable-serial-console"></a>Exemple 3 : activer la console série
Si l’accès à la console série Azure n’est pas possible, vérifiez les paramètres de configuration GRUB pour votre machine virtuelle Linux et corrigez-les. Vous trouverez des informations détaillées [dans ce document](./serial-console-grub-proactive-configuration.md)

### <a name="example-4---kernel-loading-with-problematic-lvm-swap-volume"></a>Exemple 4 : chargement du noyau avec un volume d’échange de LVM problématique

Une machine virtuelle peut ne pas démarrer complètement et s’arrêter à l’invite **dracut**.
Pour plus d’informations sur l’échec, accédez à la console série Azure ou accédez au portail Azure -> Diagnostics de démarrage -> Journal série


Une erreur semblable à celle-ci peut être présente :

```
[  188.000765] dracut-initqueue[324]: Warning: /dev/VG/SwapVol does not exist
         Starting Dracut Emergency Shell...
Warning: /dev/VG/SwapVol does not exist
```

Le fichier grub.cfg est configuré dans cet exemple pour charger un volume logique nommé **rd.lvm.lv=VG/SwapVol** que la machine virtuelle ne parvient pas à localiser. Cette ligne indique comment le noyau est chargé en référençant le volume logique SwapVol.

```
[    0.000000] Command line: BOOT_IMAGE=/vmlinuz-3.10.0-1062.4.1.el7.x86_64 root=/dev/mapper/VG-OSVol ro console=tty0 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0 biosdevname=0 crashkernel=256M rd.lvm.lv=VG/OSVol rd.lvm.lv=VG/SwapVol nodmraid rhgb quiet
[    0.000000] e820: BIOS-provided physical RAM map:
```

 Supprimez le volume logique concerné de la configuration /etc/default/grub et regénérez grub2.cfg.


## <a name="exit-chroot-and-swap-the-os-disk"></a>Quitter chroot et échanger le disque du système d’exploitation

Après avoir réparé le problème, procédez au démontage et détachez le disque de la machine virtuelle de secours, ce qui lui permet d’être échangé avec le disque de système d’exploitation de la machine virtuelle concernée.

```
exit
cd /
umount /rescue/proc/
umount /rescue/sys/
umount /rescue/dev/pts
umount /rescue/dev/
umount /rescue/boot
umount /rescue
```

Détachez le disque de la machine virtuelle de secours et échangez le disque.

Sélectionnez la machine virtuelle dans le portail **Disques** et sélectionnez **détacher**
![Détacher le disque](./media/chroot-logical-volume-manager/detach-disk.png) 

Enregistrez les modifications ![Enregistrer détacher](./media/chroot-logical-volume-manager/save-detach.png) 

Le disque devient alors disponible, ce qui lui permet d’être échangé avec le disque du système d’exploitation d’origine de la machine virtuelle concernée.

Dans le Portail Azure, accédez à la machine virtuelle défaillante et sélectionnez **Disques** -> **Échanger le disque du système d’exploitation**
![Échanger le disque](./media/chroot-logical-volume-manager/swap-disk.png) 

Renseignez les champs : **Choisir le disque** est le disque d’instantané qui vient d’être détaché à l’étape précédente. Le nom de machine virtuelle de la machine virtuelle concernée est également requis. Sélectionnez ensuite **OK**

![Nouveau disque de système d’exploitation](./media/chroot-logical-volume-manager/new-osdisk.png) 

Si la machine virtuelle est en cours d’exécution, l’échange de disque l’arrête, puis redémarre la machine virtuelle une fois l’opération d’échange de disque terminée.


## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur

 [Console série Azure]( ./serial-console-linux.md)

[Mode mono-utilisateur](./serial-console-grub-single-user-mode.md)