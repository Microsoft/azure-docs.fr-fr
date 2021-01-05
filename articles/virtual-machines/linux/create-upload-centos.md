---
title: Créer et charger un disque dur virtuel Linux basé sur CentOS
description: Apprenez à créer et à télécharger un disque dur virtuel (VHD) Azure contenant le système d'exploitation Linux CentOS.
author: gbowerman
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 11/25/2019
ms.author: guybo
ms.openlocfilehash: aefad880d788502ece0e3ba246a06a7529a3cab9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499694"
---
# <a name="prepare-a-centos-based-virtual-machine-for-azure"></a>Préparation d'une machine virtuelle CentOS pour Azure

Apprenez à créer et à télécharger un disque dur virtuel (VHD) Azure contenant le système d'exploitation Linux CentOS.

* [Préparation d’une machine virtuelle CentOS 6.x pour Azure](#centos-6x)
* [Préparation d’une machine virtuelle CentOS 7.0+ pour Azure](#centos-70)


## <a name="prerequisites"></a>Conditions préalables requises

Cet article suppose que vous avez déjà installé un système d'exploitation CentOS (ou une distribution dérivée similaire) de Linux dans un disque dur virtuel. Il existe de multiples outils dédiés à la création de fichiers .vhd, comme la solution de virtualisation Hyper-V. Pour obtenir des instructions, consultez la page [Installation du rôle Hyper-V et configuration d'une machine virtuelle](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

**Notes d'installation CentOS**

* Consultez également les [Notes générales d’installation sous Linux](create-upload-generic.md#general-linux-installation-notes) pour obtenir d’autres conseils sur la préparation de Linux pour Azure.
* Azure ne prend pas en charge le format VHDX, seulement le **VHD fixe**.  Vous pouvez convertir le disque au format VHD à l'aide de Hyper-V Manager ou de l’applet de commande convert-vhd. Si vous utilisez VirtualBox, vous devez sélectionner **Taille fixe** par opposition à la valeur par défaut allouée dynamiquement lors de la création du disque.
* Lors de l’installation du système Linux, il est *recommandé* d’utiliser les partitions standard plutôt que LVM (qui est souvent le choix par défaut pour de nombreuses installations). Ceci permettra d’éviter les conflits de noms avec des machines virtuelles clonées, notamment si un disque de système d’exploitation doit être relié à une autre machine virtuelle identique à des fins de dépannage. La technique [LVM](/previous-versions/azure/virtual-machines/linux/configure-lvm?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) ou [RAID](/previous-versions/azure/virtual-machines/linux/configure-raid?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) peut être utilisée sur les disques de données.
* La prise en charge du noyau pour le montage de systèmes de fichiers UDF est requise. Au premier démarrage sur Azure, la configuration d’approvisionnement est transmise à la machine virtuelle Linux via des supports au format UDF reliés à l’invité. L’agent Linux Azure doit être en mesure de monter le système de fichiers UDF pour lire sa configuration et approvisionner la machine virtuelle.
* Les versions du noyau Linux antérieures à la version 2.6.37 ne prennent pas en charge NUMA sur Hyper-V avec des machines virtuelles de taille supérieure. Ce problème concerne principalement les distributions antérieures utilisant le noyau Red Hat 2.6.32 en amont et a été corrigé dans la version RHEL 6.6 (kernel-2.6.32-504). Pour les systèmes exécutant des noyaux personnalisés dont la version est antérieure à la version 2.6.37 ou des noyaux basés sur RHEL antérieurs à la version 2.6.32-504, le paramètre de démarrage `numa=off` doit être défini sur la ligne de commande du noyau dans grub.conf. Pour plus d’informations, consultez l’article [KB 436883](https://access.redhat.com/solutions/436883) sur Red Hat.
* Ne configurez pas une partition d'échange sur le disque du système d'exploitation. L'agent Linux est configurable pour créer un fichier d'échange sur le disque de ressources temporaire.  Les étapes ci-dessous fournissent plus d’informations à ce sujet.
* Tous les VDH sur Azure doivent avoir une taille virtuelle alignée à 1 Mo. Lors de la conversion d’un disque brut vers VDH, vous devez vous assurer que la taille du disque brut est un multiple de 1 Mo avant la conversion. Pour plus d’informations, consultez [Notes d’installation sur Linux](create-upload-generic.md#general-linux-installation-notes).

## <a name="centos-6x"></a>CentOS 6.x

1. Dans le Gestionnaire Hyper-V, sélectionnez la machine virtuelle.

2. Cliquez sur **Connecter** pour ouvrir une fenêtre de console de la machine virtuelle.

3. Dans CentOS 6, NetworkManager peut interférer avec l’agent Linux Azure. Exécutez la commande suivante pour désinstaller le package :

    ```bash
    sudo rpm -e --nodeps NetworkManager
    ```

4. Créez ou modifiez le fichier `/etc/sysconfig/network`, puis ajoutez le texte suivant :

    ```console
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

5. Créez ou modifiez le fichier `/etc/sysconfig/network-scripts/ifcfg-eth0`, puis ajoutez le texte suivant :

    ```console
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    ```

6. Modifiez les règles udev pour éviter la génération de règles statiques pour les interfaces Ethernet. Ces règles peuvent causer des problèmes lors du clonage d’une machine virtuelle dans Microsoft Azure ou Hyper-V :

    ```bash
    sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
    ```

7. Assurez-vous que le service réseau commencera aux heures de démarrage en exécutant la commande suivante :

    ```bash
    sudo chkconfig network on
    ```

8. Si vous voulez utiliser les miroirs OpenLogic hébergés dans les centres de données Azure, remplacez le fichier `/etc/yum.repos.d/CentOS-Base.repo` par les référentiels suivants.  Cette action ajoute également le référentiel **[openlogic]** qui comprend des packages supplémentaires, comme l'agent Linux Azure :

   ```console
   [openlogic]
   name=CentOS-$releasever - openlogic packages for $basearch
   baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
   enabled=1
   gpgcheck=0

   [base]
   name=CentOS-$releasever - Base
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

   #released updates
   [updates]
   name=CentOS-$releasever - Updates
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

   #additional packages that may be useful
   [extras]
   name=CentOS-$releasever - Extras
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

   #additional packages that extend functionality of existing packages
   [centosplus]
   name=CentOS-$releasever - Plus
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
   gpgcheck=1
   enabled=0
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

   #contrib - packages by Centos Users
   [contrib]
   name=CentOS-$releasever - Contrib
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/contrib/$basearch/
   gpgcheck=1
   enabled=0
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
   ```

    > [!Note]
    > La suite de ce guide suppose que vous utilisez au moins le référentiel `[openlogic]` qui sera utilisé pour installer l'agent Linux Azure ci-dessous.

9. Ajoutez la ligne suivante au fichier /etc/yum.conf :

    ```console
    http_caching=packages
    ```

10. Exécutez la commande suivante pour effacer les métadonnées yum actuelles et mettre à jour le système avec les dernières mises à jour :

    ```bash
    yum clean all
    ```

    Sauf si vous créez une image pour version antérieure de CentOS, il est recommandé de mettre à jour tous les packages vers la dernière version :

    ```bash
    sudo yum -y update
    ```

    Un redémarrage peut être nécessaire après avoir exécuté cette commande.

11. (Facultatif) Installez les pilotes pour les services d'intégration Linux (LIS).

    > [!IMPORTANT]
    > Cette étape est **requise** pour CentOS 6.3 et versions antérieures, et facultative pour les versions ultérieures.

    ```bash
    sudo rpm -e hypervkvpd  ## (may return error if not installed, that's OK)
    sudo yum install microsoft-hyper-v
    ```

    Vous pouvez également suivre le les instructions d’installation manuelle sur la [page de téléchargement des services d’intégration Linux (LIS)](https://www.microsoft.com/download/details.aspx?id=55106) et installez le package RPM sur votre machine virtuelle.

12. Installez l'agent Linux Azure et les dépendances. Démarrez et activez le service waagent :

    ```bash
    sudo yum install python-pyasn1 WALinuxAgent
    sudo service waagent start
    sudo chkconfig waagent on
    ```


    Le package WALinuxAgent entraîne la suppression des packages NetworkManager et NetworkManager-gnome, s'ils n'avaient pas déjà été supprimés comme indiqué à l'étape 3.

13. Modifiez la ligne de démarrage du noyau dans votre configuration grub pour y inclure les paramètres de noyau supplémentaires pour Azure. Pour cela, ouvrez `/boot/grub/menu.lst` dans un éditeur de texte et vérifiez que le noyau par défaut comprend les paramètres suivants :

    ```console
    console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```

    Ce permet également d'assurer que tous les messages de la console sont envoyés vers le premier port série, ce qui peut simplifier les problèmes de débogage pour la prise en charge d'Azure.

    Outre les précautions ci-dessus, il est recommandé de *supprimer* les paramètres suivants :

    ```console
    rhgb quiet crashkernel=auto
    ```

    Le démarrage graphique et transparent n’est pas utile dans un environnement cloud où nous voulons que tous les journaux d’activité soient envoyés au port série.  L’option `crashkernel` peut éventuellement être conservée. Notez cependant que ce paramètre réduit la quantité de mémoire disponible dans la machine virtuelle de 128 Mo ou plus, ce qui peut être problématique sur les machines virtuelles de petite taille.

    > [!Important]
    > CentOS 6.5 et versions antérieures doivent également définir le paramètre de noyau `numa=off`. Consultez Red Hat [KB 436883](https://access.redhat.com/solutions/436883).

14. Vérifiez que le serveur SSH est installé et configuré pour démarrer au moment prévu.  C'est généralement le cas par défaut.

15. Ne créez pas d'espace swap sur le disque du système d'exploitation.

    L'agent Linux Azure peut configurer automatiquement un espace swap à l'aide du disque local de ressources connecté à la machine virtuelle après déploiement sur Azure. Notez que le disque de ressources local est un disque *temporaire* et qu'il peut être vidé lors de l'annulation de l'approvisionnement de la machine virtuelle. Après avoir installé l’agent Linux Azure (voir l’étape précédente), modifiez en conséquence les paramètres suivants dans le fichier `/etc/waagent.conf` :

    ```console
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048 ## NOTE: set this to whatever you need it to be.
    ```

16. Exécutez les commandes suivantes pour annuler le déploiement de la machine virtuelle et préparer son déploiement sur Azure :

    ```bash
    sudo waagent -force -deprovision
    export HISTSIZE=0
    logout
    ```

17. Cliquez sur **Action -> Arrêter** dans le Gestionnaire Hyper-V. Votre disque dur virtuel Linux est alors prêt pour le téléchargement dans Azure.



## <a name="centos-70"></a>CentOS 7.0+

**Modifications dans CentOS 7 (et les distributions dérivées)**

La préparation d'une machine virtuelle CentOS 7 pour Azure est très similaire à CentOS 6 ; cependant, certaines différences importantes méritent d'être notées :

* Le package NetworkManager n'est plus en conflit avec l'agent Azure Linux. Ce package est installé par défaut ; nous recommandons de ne pas le supprimer.
* GRUB2 est maintenant utilisé comme chargeur de démarrage (bootloader) par défaut ; la modification des paramètres du noyau a donc changé (voir ci-dessous).
* XFS est maintenant le système de fichiers par défaut. Le système de fichiers ext4 est toujours utilisable si vous le souhaitez.

**Configuration**

1. Dans le Gestionnaire Hyper-V, sélectionnez la machine virtuelle.

2. Cliquez sur **Connecter** pour ouvrir une fenêtre de console de la machine virtuelle.

3. Créez ou modifiez le fichier `/etc/sysconfig/network`, puis ajoutez le texte suivant :

    ```console
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

4. Créez ou modifiez le fichier `/etc/sysconfig/network-scripts/ifcfg-eth0`, puis ajoutez le texte suivant :

    ```console
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    ```

5. Modifiez les règles udev pour éviter la génération de règles statiques pour les interfaces Ethernet. Ces règles peuvent causer des problèmes lors du clonage d’une machine virtuelle dans Microsoft Azure ou Hyper-V :

    ```bash
    sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    ```

6. Si vous voulez utiliser les miroirs OpenLogic hébergés dans les centres de données Azure, remplacez le fichier `/etc/yum.repos.d/CentOS-Base.repo` par les référentiels suivants.  Cette action ajoute également le référentiel **[openlogic]** qui comprend les packages de l'agent Linux Azure :

   ```console
   [openlogic]
   name=CentOS-$releasever - openlogic packages for $basearch
   baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
   enabled=1
   gpgcheck=0
    
   [base]
   name=CentOS-$releasever - Base
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    
   #released updates
   [updates]
   name=CentOS-$releasever - Updates
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    
   #additional packages that may be useful
   [extras]
   name=CentOS-$releasever - Extras
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    
   #additional packages that extend functionality of existing packages
   [centosplus]
   name=CentOS-$releasever - Plus
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
   gpgcheck=1
   enabled=0
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
   ```
    
   > [!Note]
   > La suite de ce guide suppose que vous utilisez au moins le référentiel `[openlogic]` qui sera utilisé pour installer l'agent Linux Azure ci-dessous.

7. Exécutez la commande suivante pour effacer les métadonnées yum actuelles et installer les mises à jour le cas échéant :

    ```bash
    sudo yum clean all
    ```

    Sauf si vous créez une image pour version antérieure de CentOS, il est recommandé de mettre à jour tous les packages vers la dernière version :

    ```bash
    sudo yum -y update
    ```

    Un redémarrage peut être nécessaire après avoir exécuté cette commande.

8. Modifiez la ligne de démarrage du noyau dans votre configuration grub pour y inclure les paramètres de noyau supplémentaires pour Azure. Pour cela, ouvrez le fichier `/etc/default/grub` dans un éditeur de texte et modifiez le paramètre `GRUB_CMDLINE_LINUX` , par exemple :

    ```console
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   Ce permet également d'assurer que tous les messages de la console sont envoyés vers le premier port série, ce qui peut simplifier les problèmes de débogage pour la prise en charge d'Azure. Cela désactive également les nouvelles conventions d’affectation de noms CentOS 7 pour les cartes réseau. Outre les précautions ci-dessus, il est recommandé de *supprimer* les paramètres suivants :

    ```console
    rhgb quiet crashkernel=auto
    ```

    Le démarrage graphique et transparent n’est pas utile dans un environnement cloud où nous voulons que tous les journaux d’activité soient envoyés au port série. L’option `crashkernel` peut éventuellement être conservée. Notez cependant que ce paramètre réduit la quantité de mémoire disponible dans la machine virtuelle de 128 Mo ou plus, ce qui peut être problématique sur les machines virtuelles de petite taille.

9. Lorsque vous avez fini de modifier `/etc/default/grub` comme indiqué ci-dessus, exécutez la commande suivante pour régénérer la configuration grub :

    ```bash
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

10. Si vous créez l’image à partir de **VMware, VirtualBox ou KVM :** assurez-vous que les pilotes Hyper-V sont inclus dans l’initramfs :

    Modifiez `/etc/dracut.conf`, ajoutez le contenu :

    ```console
    add_drivers+=" hv_vmbus hv_netvsc hv_storvsc "
    ```

    Générez de nouveau initramfs :

    ```bash
    sudo dracut -f -v
    ```

11. Installez l'agent Linux Azure et les dépendances :

    ```bash
    sudo yum install python-pyasn1 WALinuxAgent
    sudo systemctl enable waagent
    ```

12. Ne créez pas d'espace swap sur le disque du système d'exploitation.

    L'agent Linux Azure peut configurer automatiquement un espace swap à l'aide du disque local de ressources connecté à la machine virtuelle après déploiement sur Azure. Notez que le disque de ressources local est un disque *temporaire* et qu'il peut être vidé lors de l'annulation de l'approvisionnement de la machine virtuelle. Après avoir installé l’agent Linux Azure (voir l’étape précédente), modifiez en conséquence les paramètres suivants dans le fichier `/etc/waagent.conf` :

    ```console
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.
    ```

13. Exécutez les commandes suivantes pour annuler le déploiement de la machine virtuelle et préparer son déploiement sur Azure :

    ```bash
    sudo waagent -force -deprovision
    export HISTSIZE=0
    logout
    ```

14. Cliquez sur **Action -> Arrêter** dans le Gestionnaire Hyper-V. Votre disque dur virtuel Linux est alors prêt pour le téléchargement dans Azure.

## <a name="next-steps"></a>Étapes suivantes

Vous êtes maintenant prêt à utiliser votre disque dur virtuel CentOS Linux pour créer des machines virtuelles dans Azure. S’il s’agit de votre premier chargement du fichier .vhd sur Azure, consultez [Créer une machine virtuelle Linux à partir d’un disque personnalisé](upload-vhd.md#option-1-upload-a-vhd).