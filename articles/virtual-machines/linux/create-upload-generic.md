---
title: Créer et charger un disque dur virtuel (VHD) Linux
description: Apprenez à créer et à télécharger un disque dur virtuel (VHD) Azure contenant un système d'exploitation Linux.
author: gbowerman
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 10/08/2018
ms.author: guybo
ms.openlocfilehash: ef4175d24cfd02bb5cb6470b6334fea190b5bec2
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500595"
---
# <a name="information-for-non-endorsed-distributions"></a>Informations concernant les distributions non approuvées

Le contrat SLA de la plateforme Azure s’applique aux machines virtuelles exécutant le système d’exploitation Linux uniquement lorsqu’une des [distributions approuvées](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) est utilisée. Pour ces distributions approuvées, des images Linux préconfigurées sont fournies dans la Place de marché Microsoft Azure.

* [Linux sur Azure : Distributions approuvées](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Prise en charge d’images Linux dans Microsoft Azure](https://support.microsoft.com/kb/2941892)

Toutes les distributions exécutées dans Azure présentent un certain nombre de conditions préalables. Cet article ne peut pas être exhaustif, car chaque distribution est différente. Même si vous respectez l’ensemble des critères ci-dessous, vous devrez peut-être adapter votre système Linux pour qu’il fonctionne correctement.

Nous vous recommandons de commencer par l’une des [distributions Linux approuvées sur Azure](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). L’article suivant vous montre comment préparer les diverses distributions Linux approuvées prises en charge sur Azure :

- [Distributions CentOS](create-upload-centos.md)
- [Debian Linux](debian-create-upload-vhd.md)
- [Flatcar Container Linux](flatcar-create-upload-vhd.md)
- [Oracle Linux](oracle-create-upload-vhd.md)
- [Red Hat Enterprise Linux](redhat-create-upload-vhd.md)
- [SLES et openSUSE](suse-create-upload-vhd.md)
- [Ubuntu](create-upload-ubuntu.md)

Cet article fournit des conseils généraux pour exécuter votre distribution Linux sur Azure.

## <a name="general-linux-installation-notes"></a>Notes générales d'installation de Linux
* Le format de disque dur virtuel pour Hyper-V (VHDX) n’est pas pris en charge dans Azure ; seul le format *VHD fixe* l’est.  Vous pouvez convertir le disque au format VHD à l’aide de Hyper-V Manager ou de la cmdlet [Convert-VHD](/powershell/module/hyper-v/convert-vhd). Si vous utilisez VirtualBox, il est conseillé de sélectionner **Taille fixe** plutôt que la valeur par défaut (allouée dynamiquement) lorsque vous créez le disque.
* Azure prend en charge les machines virtuelles Gen1 (BIOS Boot) & Gen2 (amorçage UEFI).
* La taille maximale autorisée pour le disque dur virtuel s’élève à 1 023 Go.
* Lorsque vous installez le système Linux, nous vous recommandons d’utiliser les partitions standard plutôt que le Gestionnaire de volumes logiques (LVM), qui constitue le choix par défaut pour de nombreuses installations. L’utilisation des partitions standard permet d’éviter les conflits de noms avec des machines virtuelles clonées, notamment si un disque de système d’exploitation est attaché à une autre machine virtuelle identique à des fins de dépannage. La technique [LVM](/previous-versions/azure/virtual-machines/linux/configure-lvm?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) ou [RAID](/previous-versions/azure/virtual-machines/linux/configure-raid?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) peut être utilisée sur les disques de données.
* La prise en charge du noyau pour le montage du système de fichiers UDF est nécessaire. Au premier démarrage sur Azure, la configuration d’approvisionnement est transmise à la machine virtuelle Linux à l’aide de supports au format UDF qui sont attachés à l’invité. L’agent Linux Azure doit monter le système de fichiers UDF pour lire sa configuration et approvisionner la machine virtuelle.
* Les versions du noyau Linux antérieures à la version 2.6.37 ne prennent pas en charge NUMA sur Hyper-V avec des machines virtuelles de taille supérieure. Ce problème concerne principalement les distributions antérieures utilisant le noyau Red Hat 2.6.32 en amont ; il a été corrigé dans Red Hat Enterprise Linux (RHEL) 6.6 (kernel-2.6.32-504). Pour les systèmes exécutant des noyaux personnalisés dont la version est antérieure à la version 2.6.37 ou des noyaux basés sur RHEL antérieurs à la version 2.6.32-504, le paramètre de démarrage `numa=off` doit être défini sur la ligne de commande du noyau dans grub.conf. Pour plus d’informations, consultez l’article [KB 436883](https://access.redhat.com/solutions/436883) sur Red Hat.
* Ne configurez pas une partition d’échange sur le disque du système d’exploitation. L’agent Linux est configurable pour créer un fichier d’échange sur le disque de ressources temporaire, comme décrit dans les étapes suivantes.
* Tous les VHD sur Azure doivent avoir une taille virtuelle alignée sur 1 Mo. Avant de convertir un disque brut en disque VHD, vous devez vous assurer que la taille du disque brut est un multiple de 1 Mo, comme décrit dans les étapes suivantes.

### <a name="installing-kernel-modules-without-hyper-v"></a>Installation de modules de noyau sans Hyper-V
Azure s’exécute sur l’hyperviseur Hyper-V. Linux nécessite donc l’exécution de certains modules de noyau dans Azure. Si vous disposez d’une machine virtuelle qui a été créée en dehors d’Hyper-V, les programmes d’installation de Linux risquent de ne pas inclure les pilotes pour Hyper-V dans le ramdisk initial (initrd ou initramfs) sauf si la machine virtuelle détecte qu’elle s’exécute dans un environnement Hyper-V. Quand vous utilisez un système de virtualisation différent (VirtualBox, KVM, etc.) pour préparer votre image Linux, vous devrez peut-être regénérer l’initrd afin que les modules du noyau hv_vmbus et hv_storvsc soient au moins disponibles sur le ramdisk initial.  Ce problème connu concerne les systèmes basés sur la distribution Red Hat en amont, et peut-être d’autres systèmes.

Le mécanisme de reconstruction d'image initrd ou initramfs varie en fonction de la distribution. Consultez la documentation ou le support de votre distribution pour trouver la procédure appropriée.  L’exemple suivant permet de régénérer le ramdisk initrd à l’aide de l’utilitaire `mkinitrd` :

1. Sauvegardez l’image initrd existante :

    ```
    cd /boot
    sudo cp initrd-`uname -r`.img  initrd-`uname -r`.img.bak
    ```

2. Régénérez le ramdisk initrd avec les modules noyau hv_vmbus et hv_storvsc :

    ```
    sudo mkinitrd --preload=hv_storvsc --preload=hv_vmbus -v -f initrd-`uname -r`.img `uname -r`
    ```

### <a name="resizing-vhds"></a>Redimensionnement des disques durs virtuels
Les images de disque dur virtuel sur Azure doivent avoir une taille virtuelle alignée à 1 Mo.  En règle générale, les disques durs virtuels créés à l’aide d’Hyper-V sont alignés correctement.  Si le disque dur virtuel n’est pas correctement aligné, un message d’erreur semblable au suivant peut s’afficher quand vous essayez de créer une image à partir de votre disque dur virtuel.

* Le disque dur virtuel http:\//\<mystorageaccount>.blob.core.windows.net/vhds/MyLinuxVM.vhd a une taille virtuelle non prise en charge de 21475270656 octets. La taille doit être un nombre entier (en Mo).

Le cas échéant, redimensionnez la machine virtuelle à l’aide de la console Gestionnaire Hyper-V ou de la cmdlet PowerShell [Resize-VHD](/powershell/module/hyper-v/resize-vhd?view=win10-ps).  Si vous n’utilisez pas un environnement Windows, nous vous recommandons d’utiliser `qemu-img` pour convertir (si nécessaire) le disque dur virtuel et le redimensionner.

> [!NOTE]
> Il existe un [bogue connu dans la version 2.2.1 de qemu-img](https://bugs.launchpad.net/qemu/+bug/1490611), qui entraîne un formatage incorrect de disque dur virtuel. Ce problème a été résolu dans QEMU 2.6. Nous vous recommandons d’utiliser `qemu-img` 2.2.0 ou inférieur, ou les versions 2.6 ou supérieures.
> 

1. Redimensionner le disque dur virtuel directement à l’aide d’outils comme `qemu-img` ou `vbox-manage` peut rendre le disque dur virtuel non démarrable.  Nous vous recommandons de convertir d’abord le disque dur virtuel en image de disque RAW.  Si l’image de machine virtuelle a été créée comme image de disque RAW (valeur par défaut de certains hyperviseurs comme KVM), vous pouvez ignorer cette étape.
 
    ```
    qemu-img convert -f vpc -O raw MyLinuxVM.vhd MyLinuxVM.raw
    ```

1. Calculez la taille nécessaire de l’image de disque afin que la taille virtuelle soit alignée sur 1 Mo.  Le script suivant de l’interpréteur de commandes Bash utilise `qemu-img info` pour déterminer la taille virtuelle de l’image de disque, puis calcule la taille au 1 Mo supérieur.

    ```bash
    rawdisk="MyLinuxVM.raw"
    vhddisk="MyLinuxVM.vhd"

    MB=$((1024*1024))
    size=$(qemu-img info -f raw --output json "$rawdisk" | \
    gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

    rounded_size=$(((($size+$MB-1)/$MB)*$MB))
    
    echo "Rounded Size = $rounded_size"
    ```

3. Redimensionnez le disque brut à l’aide de `$rounded_size` comme défini ci-dessus.

    ```bash
    qemu-img resize MyLinuxVM.raw $rounded_size
    ```

4. À présent, convertissez de nouveau le disque RAW en disque dur virtuel à taille fixe.

    ```bash
    qemu-img convert -f raw -o subformat=fixed -O vpc MyLinuxVM.raw MyLinuxVM.vhd
    ```

   Ou, avec la version qemu 2.6 +, insérez l’option `force_size` :

    ```bash
    qemu-img convert -f raw -o subformat=fixed,force_size -O vpc MyLinuxVM.raw MyLinuxVM.vhd
    ```

## <a name="linux-kernel-requirements"></a>Conditions requises pour le noyau Linux

Les pilotes LIS (Linux Integration Services) pour Hyper-V et Azure sont directement liés au noyau Linux en amont. Ces pilotes sont déjà disponibles dans de nombreuses distributions qui incluent une version récente du noyau Linux (3.x par exemple). Sinon, ces distributions fournissent des versions rétroportées de ces pilotes avec leurs noyaux.  Ces pilotes sont mis à jour en permanence dans le noyau en amont avec de nouveaux correctifs et de nouvelles fonctionnalités. Aussi, dans la mesure du possible, nous vous recommandons d’exécuter une [distribution approuvée](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) comportant ces correctifs et modifications.

Si vous utilisez une variante des versions Red Hat Enterprise Linux 6.0 à 6.3, vous devez installer les [pilotes LIS les plus récents pour Hyper-V](https://go.microsoft.com/fwlink/p/?LinkID=254263&clcid=0x409). Depuis RHEL 6.4+ (et les distributions dérivées), les pilotes LIS sont déjà inclus dans le noyau ; aucun package d’installation supplémentaire n’est donc nécessaire.

Si un noyau personnalisé est requis, nous vous recommandons d’utiliser une version plus récente du noyau (par exemple 3.8+). Pour les distributions ou les fournisseurs qui maintiennent leur propre noyau, vous devez rétroporter régulièrement les pilotes LIS du noyau en amont vers votre noyau personnalisé.  Même si vous utilisez déjà une version relativement récente du noyau, nous vous recommandons vivement de conserver une trace des correctifs en amont des pilotes LIS et de les rétroporter en fonction des besoins. Les emplacements des fichiers sources des pilotes LIS sont spécifiés dans le fichier [MAINTAINERS](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/MAINTAINERS) dans l’arborescence source du noyau Linux :
```
    F:    arch/x86/include/asm/mshyperv.h
    F:    arch/x86/include/uapi/asm/hyperv.h
    F:    arch/x86/kernel/cpu/mshyperv.c
    F:    drivers/hid/hid-hyperv.c
    F:    drivers/hv/
    F:    drivers/input/serio/hyperv-keyboard.c
    F:    drivers/net/hyperv/
    F:    drivers/scsi/storvsc_drv.c
    F:    drivers/video/fbdev/hyperv_fb.c
    F:    include/linux/hyperv.h
    F:    tools/hv/
```
Les correctifs suivants doivent être inclus dans le noyau. Cette liste ne peut pas être complète pour toutes les distributions.

* [ata_piix : reporter des disques dans les pilotes Hyper-V par défaut](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/drivers/ata/ata_piix.c?id=cd006086fa5d91414d8ff9ff2b78fbb593878e3c)
* [storvsc : prendre en compte les paquets en transit dans le chemin RESET](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/drivers/scsi/storvsc_drv.c?id=5c1b10ab7f93d24f29b5630286e323d1c5802d5c)
* [storvsc : éviter l’utilisation de WRITE_SAME](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=3e8f4f4065901c8dfc51407e1984495e1748c090)
* [storvsc : désactiver WRITE_SAME pour RAID et les pilotes adaptateurs de l’hôte virtuel](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=54b2b50c20a61b51199bedb6e5d2f8ec2568fb43)
* [storvsc : correctif de déréférencement de pointeur NULL](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=b12bb60d6c350b348a4e1460cd68f97ccae9822e)
* [storvsc : des défaillances de la mémoire tampon de l’anneau peuvent entraîner un gel des E/S](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=e86fb5e8ab95f10ec5f2e9430119d5d35020c951)
* [scsi_sysfs : se protéger contre une double exécution de __scsi_remove_device](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/scsi_sysfs.c?id=be821fd8e62765de43cc4f0e2db363d0e30a7e9b)

## <a name="the-azure-linux-agent"></a>agent Linux Azure
L’[agent Linux Azure](../extensions/agent-linux.md) `waagent` provisionne une machine virtuelle Linux dans Azure. La dernière version, les problèmes des fichiers ou l’envoi de demandes de tirage (pull requests) sont disponibles dans le [référentiel de l’agent Linux sur GitHub](https://github.com/Azure/WALinuxAgent).

* L'agent Linux est publié avec la licence Apache 2.0. De nombreuses distributions fournissent déjà des packages RPM ou .deb pour l’agent, et ces packages peuvent être installés et mis à jour facilement.
* L'agent Linux Azure nécessite Python v2.6+.
* L'agent nécessite également le module python-pyasn1. La plupart des distributions fournissent ce module sous la forme d’un package distinct à installer.
* Dans certains cas, l’agent Linux Azure n’est pas compatible avec NetworkManager. De nombreux packages RPM/deb fournis par les distributions configurent NetworkManager en conflit avec le package waagent. Dans ce cas, NetworkManager est désinstallé lorsque vous installez le package de l’agent Linux.
* La version de l’agent Linux Azure doit être supérieure ou égale à la [version minimale prise en charge](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

## <a name="general-linux-system-requirements"></a>Configuration générale requise Linux

1. Modifiez la ligne de démarrage du noyau dans GRUB ou GRUB2 afin d’inclure les paramètres suivants pour que tous les messages de la console soient envoyés au premier port série. Ces messages peuvent aider l’équipe du support Azure à procéder au débogage.
    ```  
    console=ttyS0,115200n8 earlyprintk=ttyS0,115200 rootdelay=300
    ```
    Nous vous recommandons également de *supprimer* les paramètres suivants le cas échéant.
    ```  
    rhgb quiet crashkernel=auto
    ```
    Le démarrage graphique et en mode silencieux n’est pas utile dans un environnement cloud où nous voulons tous les journaux d’activité envoyés au port série. L’option `crashkernel` peut être conservée si nécessaire. Notez cependant que ce paramètre réduit la quantité de mémoire disponible dans la machine virtuelle d’au moins 128 Mo, ce qui peut être problématique pour les machines virtuelles plus petites.

1. Installez l'agent Linux Azure.
  
    L'agent Linux Azure est requis pour approvisionner une image Linux sur Azure.  De nombreuses distributions fournissent cet agent sous forme de package RPM ou .deb (ce package est généralement nommé WALinuxAgent ou walinuxagent).  Il est également possible d'installer manuellement cet agent en suivant les instructions du [Guide de l'agent Linux](../extensions/agent-linux.md).

1. Vérifiez que le serveur SSH est installé et configuré pour démarrer au moment prévu.  Cette configuration est généralement définie par défaut.

1. Ne créez pas d’espace d’échange sur le disque du système d’exploitation.
  
    L'agent Linux Azure peut configurer automatiquement un espace swap à l'aide du disque local de ressources connecté à la machine virtuelle après déploiement sur Azure. Le disque de ressources local est un disque *temporaire*, et il peut être vidé quand la machine virtuelle est déprovisionnée. Après avoir installé l’agent Linux Azure (étape 2 ci-dessus), modifiez les paramètres suivants dans le fichier /etc/waagent.conf si nécessaire.
    ```  
        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: Set this to your desired size.
    ```
1. Exécutez les commandes suivantes pour déprovisionner la machine virtuelle.
  
     ```
     sudo waagent -force -deprovision
     export HISTSIZE=0
     logout
     ```  
   > [!NOTE]
   > Sur Virtualbox, vous pouvez voir l’erreur suivante après l’exécution de `waagent -force -deprovision` qui indique `[Errno 5] Input/output error`. Ce message d’erreur n’est pas critique et peut être ignoré.

* Arrêtez la machine virtuelle et chargez le disque dur virtuel sur Azure.