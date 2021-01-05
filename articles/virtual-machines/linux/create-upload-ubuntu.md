---
title: Création et téléchargement d'un disque dur virtuel Linux Ubuntu dans Azure
description: Apprenez à créer et à charger un disque dur virtuel (VHD) Azure contenant un système d'exploitation Linux Ubuntu.
author: danielsollondon
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 06/06/2020
ms.author: danis
ms.openlocfilehash: d3bdc3d0705bd8edc70f55b8372818e60859eedd
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500561"
---
# <a name="prepare-an-ubuntu-virtual-machine-for-azure"></a>Préparation d'une machine virtuelle Linux Ubuntu pour Azure


Ubuntu publie désormais des disques durs virtuels Azure officiels téléchargeables depuis l’adresse [https://cloud-images.ubuntu.com/](https://cloud-images.ubuntu.com/). Si vous avez besoin de créer votre propre image Ubuntu spécialisée pour Azure, plutôt que d’utiliser la procédure manuelle ci-après, nous vous recommandons de démarrer avec ces disques durs virtuels opérationnels connus et de les personnaliser selon vos besoins. Les dernières versions de l’image se trouvent toujours aux emplacements suivants :

* Ubuntu 16.04/Xenial : [ubuntu-16.04-server-cloudimg-amd64-disk1.vmdk](https://cloud-images.ubuntu.com/releases/xenial/release/ubuntu-16.04-server-cloudimg-amd64-disk1.vmdk)
* Ubuntu 18.04/Bionic : [bionic-server-cloudimg-amd64.vmdk](https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.vmdk)

## <a name="prerequisites"></a>Prérequis
Cet article suppose que vous avez déjà installé un système d'exploitation Linux Ubuntu dans un disque dur virtuel. Il existe de multiples outils dédiés à la création de fichiers .vhd, comme la solution de virtualisation Hyper-V. Pour obtenir des instructions, consultez la page [Installation du rôle Hyper-V et configuration d'une machine virtuelle](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

**Notes d'installation Ubuntu**

* Consultez également les [Notes générales d’installation sous Linux](create-upload-generic.md#general-linux-installation-notes) pour obtenir d’autres conseils sur la préparation de Linux pour Azure.
* Azure ne prend pas en charge le format VHDX, seulement le **VHD fixe**.  Vous pouvez convertir le disque au format VHD à l’aide du gestionnaire Hyper-V ou de la cmdlet `Convert-VHD`.
* Lors de l’installation du système Linux, il est recommandé d’utiliser les partitions standard plutôt que LVM (qui est souvent le choix par défaut pour de nombreuses installations). Ceci permettra d'éviter les conflits de noms avec des machines virtuelles clonées, notamment si un disque de système d'exploitation doit être relié à une autre machine virtuelle pour la dépanner. La technique [LVM](/previous-versions/azure/virtual-machines/linux/configure-lvm?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) ou [RAID](/previous-versions/azure/virtual-machines/linux/configure-raid?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) peut être utilisée sur les disques de données, le cas échéant.
* Ne configurez pas de partition ou de fichier d’échange sur le disque du système d’exploitation. L’agent de provisionnement cloud-init peut être configuré pour créer un fichier ou une partition d’échange sur le disque de ressources temporaire. Les étapes ci-dessous fournissent plus d’informations à ce sujet.
* Tous les VDH sur Azure doivent avoir une taille virtuelle alignée à 1 Mo. Lors de la conversion d’un disque brut vers VDH, vous devez vous assurer que la taille du disque brut est un multiple de 1 Mo avant la conversion. Pour plus d’informations, consultez [Notes d’installation sur Linux](create-upload-generic.md#general-linux-installation-notes).

## <a name="manual-steps"></a>Étapes manuelles
> [!NOTE]
> Avant d’essayer de créer votre propre image Ubuntu personnalisée pour Azure, envisagez d’utiliser les images préconçues et testées disponibles sur [https://cloud-images.ubuntu.com/](https://cloud-images.ubuntu.com/).
> 
> 

1. Dans le panneau central de Hyper-V Manager, sélectionnez la machine virtuelle.

2. Cliquez sur **Connect** pour ouvrir la fenêtre de la machine virtuelle.

3. Remplacez les référentiels actuels dans l’image par le référentiel Azure d’Ubuntu.

    Avant de modifier `/etc/apt/sources.list`, il est recommandé d’effectuer une sauvegarde :

    ```console
    # sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
    ```

    Ubuntu 16.04 et Ubuntu 18.04 :

    ```console
    # sudo sed -i 's/http:\/\/archive\.ubuntu\.com\/ubuntu\//http:\/\/azure\.archive\.ubuntu\.com\/ubuntu\//g' /etc/apt/sources.list
    # sudo sed -i 's/http:\/\/[a-z][a-z]\.archive\.ubuntu\.com\/ubuntu\//http:\/\/azure\.archive\.ubuntu\.com\/ubuntu\//g' /etc/apt/sources.list
    # sudo apt-get update
    ```

4. Les images Ubuntu Azure utilisent à présent le [noyau adapté à Azure](https://ubuntu.com/blog/microsoft-and-canonical-increase-velocity-with-azure-tailored-kernel). Mettez à jour le système d’exploitation pour passer au dernier noyau adapté à Azure et installez les outils Linux Azure (y compris les dépendances Hyper-V) en exécutant les commandes suivantes :

    Ubuntu 16.04 et Ubuntu 18.04 :

    ```console
    # sudo apt update
    # sudo apt install linux-azure linux-image-azure linux-headers-azure linux-tools-common linux-cloud-tools-common linux-tools-azure linux-cloud-tools-azure
    (recommended) # sudo apt full-upgrade

    # sudo reboot
    ```

5. Modifiez la ligne de démarrage du noyau afin que Grub y inclue les paramètres de noyau supplémentaires pour Azure. Pour cela, ouvrez le fichier `/etc/default/grub` dans un éditeur de texte. Recherchez la variable nommée `GRUB_CMDLINE_LINUX_DEFAULT` (ou ajoutez-la le cas échéant) et modifiez-la pour inclure les paramètres suivants :

    ```text
    GRUB_CMDLINE_LINUX_DEFAULT="console=tty1 console=ttyS0,115200n8 earlyprintk=ttyS0,115200 rootdelay=300 quiet splash"
    ```

    Enregistrez ce fichier, puis fermez-le. Exécutez ensuite la commande `sudo update-grub`. Ce permet d'assurer que tous les messages de la console sont envoyés vers le premier port série, ce qui peut simplifier les problèmes de débogage pour l'assistance technique d'Azure.

6. Vérifiez que le serveur SSH est installé et configuré pour démarrer au moment prévu.  C'est généralement le cas par défaut.

7. Installez cloud-init (l’agent de provisionnement) et l’agent Linux Azure (le gestionnaire d’extensions invitées). cloud-init utilise Netplan pour configurer la configuration réseau système lors du provisionnement et à chaque démarrage suivant.

    ```console
    # sudo apt update
    # sudo apt install cloud-init netplan.io walinuxagent && systemctl stop walinuxagent
    ```

   > [!Note]
   >  Le package `walinuxagent` peut entraîner la suppression des packages `NetworkManager` et `NetworkManager-gnome` (s’ils sont installés).

8. Supprimez les configurations par défaut de cloud-init et les artefacts Netplan restants qui pourraient entrer en conflit avec le provisionnement cloud-init sur Azure :

    ```console
    # rm -f /etc/cloud/cloud.cfg.d/50-curtin-networking.cfg /etc/cloud/cloud.cfg.d/curtin-preserve-sources.cfg
    # rm -f /etc/cloud/ds-identify.cfg
    # rm -f /etc/netplan/*.yaml
    ```

9. Configurez cloud-init de façon à provisionner le système à l’aide de la source de données Azure :

    ```console
    # cat > /etc/cloud/cloud.cfg.d/90_dpkg.cfg << EOF
    datasource_list: [ Azure ]
    EOF

    # cat > /etc/cloud/cloud.cfg.d/90-azure.cfg << EOF
    system_info:
       package_mirrors:
         - arches: [i386, amd64]
           failsafe:
             primary: http://archive.ubuntu.com/ubuntu
             security: http://security.ubuntu.com/ubuntu
           search:
             primary:
               - http://azure.archive.ubuntu.com/ubuntu/
             security: []
         - arches: [armhf, armel, default]
           failsafe:
             primary: http://ports.ubuntu.com/ubuntu-ports
             security: http://ports.ubuntu.com/ubuntu-ports
    EOF

    # cat > /etc/cloud/cloud.cfg.d/10-azure-kvp.cfg << EOF
    reporting:
      logging:
        type: log
      telemetry:
        type: hyperv
    EOF
    ```

10. Configurez l’agent Linux Azure de sorte qu’il s’appuie sur cloud-init pour effectuer le provisionnement. Pour plus d’informations sur ces options, consultez le [projet WALinuxAgent](https://github.com/Azure/WALinuxAgent).

    ```console
    sed -i 's/Provisioning.Enabled=y/Provisioning.Enabled=n/g' /etc/waagent.conf
    sed -i 's/Provisioning.UseCloudInit=n/Provisioning.UseCloudInit=y/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.Format=y/ResourceDisk.Format=n/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.EnableSwap=y/ResourceDisk.EnableSwap=n/g' /etc/waagent.conf

    cat >> /etc/waagent.conf << EOF
    # For Azure Linux agent version >= 2.2.45, this is the option to configure,
    # enable, or disable the provisioning behavior of the Linux agent.
    # Accepted values are auto (default), waagent, cloud-init, or disabled.
    # A value of auto means that the agent will rely on cloud-init to handle
    # provisioning if it is installed and enabled, which in this case it will.
    Provisioning.Agent=auto
    EOF
    ```

11. Nettoyez les journaux et artefacts d’exécution de l’agent Linux Azure et cloud-init :

    ```console
    # sudo cloud-init clean --logs --seed
    # sudo rm -rf /var/lib/cloud/
    # sudo systemctl stop walinuxagent.service
    # sudo rm -rf /var/lib/waagent/
    # sudo rm -f /var/log/waagent.log
    ```

12. Exécutez les commandes suivantes pour annuler le déploiement de la machine virtuelle et préparer son déploiement sur Azure :

    > [!NOTE]
    > La commande `sudo waagent -force -deprovision+user` essaie de nettoyer le système et de le préparer pour le reprovisionnement. L’option `+user` supprime le dernier compte d’utilisateur provisionné et les données associées.

    > [!WARNING]
    > Le déprovisionnement à l’aide de la commande ci-dessus ne garantit pas que l’image est exempte de toute information sensible ni qu’elle convient à la redistribution.

    ```console
    # sudo waagent -force -deprovision+user
    # rm -f ~/.bash_history
    # export HISTSIZE=0
    # logout
    ```

13. Cliquez sur **Action -> Arrêter** dans le Gestionnaire Hyper-V.

14. Azure accepte uniquement les VHD de taille fixe. Si le disque du système d’exploitation de la machine virtuelle n’en est pas un, utilisez la cmdlet PowerShell `Convert-VHD` et spécifiez l’option `-VHDType Fixed`. Consultez les documents relatifs à `Convert-VHD` dans [Convert-VHD](/powershell/module/hyper-v/convert-vhd?view=win10-ps).


## <a name="next-steps"></a>Étapes suivantes
Vous êtes maintenant prêt à utiliser votre disque dur virtuel Ubuntu Linux pour créer des machines virtuelles dans Azure. S’il s’agit de votre premier chargement du fichier .vhd sur Azure, consultez [Créer une machine virtuelle Linux à partir d’un disque personnalisé](upload-vhd.md#option-1-upload-a-vhd).