---
title: Console série Azure pour le GRUB et mode mono-utilisateur | Microsoft Docs
description: Cet article explique comment utiliser la console série pour GRUB dans les machines virtuelles Azure.
services: virtual-machines-linux
documentationcenter: ''
author: asinn826
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/06/2019
ms.author: alsin
ms.openlocfilehash: 5341cc62a7d02c3072df90becf893dec18427ac2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87439545"
---
# <a name="use-serial-console-to-access-grub-and-single-user-mode"></a>Utiliser la console série pour accéder au GRUB et au mode mono-utilisateur
Le GRand Unified Bootloader (GRUB) est probablement la première chose que vous voyez lorsque vous démarrez une machine virtuelle. Parce qu’il s’affiche avant que le système d’exploitation ne démarre, le GRUB n’est pas accessible par le biais du protocole SSH. Dans GRUB, vous pouvez modifier votre configuration d’amorçage, notamment pour démarrer en mode mono-utilisateur.

Le mode mono-utilisateur est un environnement minimal comportant des fonctionnalités minimales. Il peut être utile pour examiner des problèmes de démarrage, des problèmes de système de fichiers ou des problèmes de réseau. Les services pouvant s’exécuter en arrière-plan sont moins nombreux et, selon le niveau d’exécution, il peut même arriver qu’un système de fichiers ne soit pas monté automatiquement.

Le mode mono-utilisateur est également utile dans les situations où votre machine virtuelle ne peut être configurée que pour accepter uniquement des clés SSH pour la connexion. Dans ce cas, il se peut que vous puissiez utiliser le mode mono-utilisateur pour créer un compte avec authentification par mot de passe.

> [!NOTE]
> Le service de la console série autorise uniquement les utilisateurs dotés d’autorisations *contributeur* ou plus à accéder à la console série d’une machine virtuelle.

Pour passer en mode mono-utilisateur, entrez un GRUB lors de du démarrage de votre machine virtuelle, puis modifier la configuration d’amorçage dans le GRUB. Des instructions détaillées pour entrer dans le GRUB figurent dans la section suivante. En règle générale, vous pouvez utiliser le bouton de redémarrage au sein de la console série de la machine virtuelle pour redémarrer votre machine virtuelle et afficher le GRUB si votre machine virtuelle a été configurée pour afficher le GRUB.

![Le bouton de redémarrage de la console série Linux](./media/virtual-machines-serial-console/virtual-machine-serial-console-restart-button-bar.png)

## <a name="general-grub-access"></a>Accès général au GRUB
Pour accéder au GRUB, redémarrez votre machine virtuelle alors que le volet Console série est ouvert. Certaines distributions nécessitent une entrée au clavier pour afficher le GRUB, et d’autres l’affichent automatiquement pendant quelques secondes afin de permettre à l’utilisateur d’effectuer une entrée au clavier pour annuler le délai d’expiration.

Vous devez vous assurer que le GRUB est activé sur votre machine virtuelle afin de pouvoir accéder au mode mono-utilisateur. Selon votre distribution, un travail de configuration peut être nécessaire pour s’assurer que le GRUB est activé. Pour obtenir des informations spécifiques de la distribution, consultez la section suivante.

### <a name="restart-your-vm-to-access-grub-in-serial-console"></a>Redémarrer votre machine virtuelle pour accéder au GRUB dans la console série
Vous pouvez redémarrer votre machine virtuelle dans la console série en pointant sur le bouton **Redémarrer**, puis en sélectionnant **Redémarrer la machine virtuelle**. Une notification sur le redémarrage s’affiche en bas du volet.

Vous pouvez également redémarrer votre machine virtuelle en exécutant une commande SysRq « b » si [SysRq](./serial-console-nmi-sysrq.md) est activé. Pour voir ce que vous pouvez attendre du GRUB lorsque vous redémarrez, suivez les instructions spécifiques des sections suivantes.

![Redémarrage de la console série Linux](./media/virtual-machines-serial-console/virtual-machine-serial-console-restart-button-ubuntu.gif)

## <a name="general-single-user-mode-access"></a>Accès général au mode mono-utilisateur
Un accès manuel au mode mono-utilisateur peut être nécessaire dans les situations où vous n’avez pas configuré de compte avec authentification par mot de passe. Modifiez la configuration du GRUB pour passer manuellement en mode mono-utilisateur. Une fois cela fait, consultez la section « Utiliser la console série pour accéder au GRUB et au mode mono-utilisateur » pour obtenir des instructions supplémentaires.

Si la machine virtuelle ne parvient pas à démarrer, les distributions vous ramènent souvent automatiquement au mode mono-utilisateur ou au mode urgence. Toutefois, d’autres distributions nécessitent une configuration supplémentaire, telle que la définition d’un mot de passe racine, pour pouvoir vous ramener automatiquement au mode mono-utilisateur ou au mode urgence.

### <a name="use-single-user-mode-to-reset-or-add-a-password"></a>Utiliser le mode mono-utilisateur pour réinitialiser ou ajouter un mot de passe
Une fois en mode mono-utilisateur, ajoutez un nouvel utilisateur doté de privilèges sudo en procédant comme suit :
1. Exécutez `useradd <username>` pour ajouter un utilisateur.
1. Exécutez `sudo usermod -a -G sudo <username>` pour accorder au nouvel utilisateur des privilèges racine.
1. Utilisez `passwd <username>` pour définir le mot de passe pour le nouvel utilisateur. Vous pouvez ensuite vous connecter en tant que nouvel utilisateur.


## <a name="access-for-red-hat-enterprise-linux-rhel"></a>Accès pour Red Hat Enterprise Linux (RHEL)
RHEL vous amène directement en mode mono-utilisateur s’il ne peut pas démarrer normalement. Toutefois, si vous n’avez pas configuré d’accès racine pour le mode mono-utilisateur, vous n’aurez pas de mot de passe racine et vous ne pourrez pas vous connecter. Il existe une solution de contournement (voir la section « Entrer manuellement en mode mono-utilisateur dans RHEL »), mais nous vous suggérons de configurer l’accès racine dès le départ.

### <a name="grub-access-in-rhel"></a>Accès au GRUB dans RHEL
RHEL est fourni avec le GRUB activé par défaut. Pour entrer dans le GRUB, redémarrez votre machine virtuelle en exécutant `sudo reboot` et appuyez sur n’importe quelle touche. Le volet du GRUB doit s’afficher. S’il ne s’affiche pas, assurez-vous que les lignes suivantes sont présentes dans votre fichier GRUB (`/etc/default/grub`) :

**Pour RHEL 8**

>[!NOTE]
> Red Hat recommande l’utilisation de Grubby pour configurer les paramètres de ligne de commande du noyau dans RHEL 8+ et versions ultérieures. Il n’est actuellement pas possible de mettre à jour le délai d’expiration grub et les paramètres de terminal à l’aide de grubby. Pour modifier l’argument GRUB_CMDLINE_LINUX pour toutes les entrées de démarrage, exécutez `grubby --update-kernel=ALL --args="console=ttyS0,115200 console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"`. Des informations supplémentaires sont disponibles [ici](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_monitoring_and_updating_the_kernel/configuring-kernel-command-line-parameters_managing-monitoring-and-updating-the-kernel).

```
GRUB_TIMEOUT=5
GRUB_TERMINAL="serial console"
GRUB_CMDLINE_LINUX="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"
```

**Pour RHEL 7**

```
GRUB_TIMEOUT=5
GRUB_TERMINAL_OUTPUT="serial console"
GRUB_CMDLINE_LINUX="console=tty1 console=ttyS0,115200n8 earlyprintk=ttyS0,115200 rootdelay=300 net.ifnames=0"
```

> [!NOTE]
> Red Hat fournit également la documentation de démarrage en mode de récupération, mode d’urgence et mode de débogage, ainsi que la documentation pour réinitialiser le mot de passe racine. Pour obtenir des instructions, consultez [Modification du menu du terminal lors du démarrage](https://aka.ms/rhel7grubterminal).

### <a name="set-up-root-access-for-single-user-mode-in-rhel"></a>Configurer l’accès racine pour le mode mono-utilisateur dans RHEL
L’utilisateur racine est désactivé par défaut. Dans RHEL, le mode mono-utilisateur nécessite l’activation d’un utilisateur racine. Si vous avez besoin d’activer le mode d’utilisateur unique, appliquez les instructions suivantes :

1. Connectez-vous au système Red Hat via SSH.
1. Basculez vers la racine.
1. Activez le mot de passe pour l’utilisateur racine en procédant comme suit :
    * Exécutez `passwd root` (définissez un mot de passe racine fort).
1. Assurez-vous que l’utilisateur racine ne peut se connecter qu’avec ttyS0 en procédant comme suit : a. Exécutez `edit /etc/ssh/sshd_config` et vérifiez que PermitRootLogIn est réglé sur `no`.
    b. Exécutez `edit /etc/securetty file` pour autoriser la connexion via ttyS0 uniquement.

Désormais, si le système démarre en mode d’utilisateur unique, vous pouvez vous connecter uniquement avec le mot de passe racine.

Sinon, pour RHEL 7.4+ ou 6.9+, vous pouvez activer le mode d’utilisateur unique dans les invites GRUB en suivant les instructions de [Démarrage en mode mono-utilisateur](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/installation_guide/s1-rescuemode-booting-single).

### <a name="manually-enter-single-user-mode-in-rhel"></a>Entrer manuellement en mode mono-utilisateur dans RHEL
Si vous avez configuré le GRUB et l’accès racine à l’aide des instructions précédentes, vous pouvez entrer en mode mono-utilisateur en procédant comme suit :

1. Pour entrer dans le GRUB, appuyez sur ÉCHAP lorsque vous redémarrez la machine virtuelle.
1. Dans le GRUB, appuyez sur E pour modifier le système d’exploitation avec lequel vous souhaitez démarrer. Le système d’exploitation est généralement répertorié sur la première ligne.
1. Recherchez la ligne du noyau. Dans Azure, elle commence par *linux16*.
1. Appuyez sur Ctrl+E pour accéder à la fin de la ligne.
1. À la fin de la ligne, ajoutez *systemd.unit=rescue.target*.

    Cette action vous fait redémarrer en mode mono-utilisateur. Si vous souhaitez utiliser le mode urgence, ajoutez*systemd.unit=emergency.target* à la fin de la ligne (au lieu de *systemd.unit=rescue.target*).

1. Appuyez sur Ctrl+X pour quitter et redémarrer votre ordinateur avec les paramètres appliqués.

   Vous êtes invité à entrer le mot de passe de l’administrateur avant de pouvoir passer en mode mono-utilisateur. Ce mot de passe est celui que vous avez créé avec les instructions précédentes.

    ![Image animée montrant une interface de ligne de commande. L’utilisateur sélectionne un serveur, repère la fin de la ligne du noyau, puis entre le texte spécifié.](../media/virtual-machines-serial-console/virtual-machine-linux-serial-console-rhel-enter-emergency-shell.gif)

### <a name="enter-single-user-mode-without-root-account-enabled-in-rhel"></a>Entrez en mode mono-utilisateur unique sans compte racine activé dans RHEL
Si vous n’avez pas activé l’utilisateur racine en suivant les instructions précédentes, vous pouvez toujours réinitialiser votre mot de passe racine en procédant comme suit :

> [!NOTE]
> Si vous utilisez SELinux, lorsque vous réinitialisez le mot de passe racine, assurez-vous de suivre les étapes supplémentaires qui sont décrites dans la [documentation de Red Hat](https://aka.ms/rhel7grubterminal).

1. Pour entrer dans le GRUB, appuyez sur ÉCHAP lorsque vous redémarrez la machine virtuelle.

1. Dans le GRUB, appuyez sur E pour modifier le système d’exploitation avec lequel vous souhaitez démarrer. Le système d’exploitation est généralement répertorié sur la première ligne.
1. Recherchez la ligne du noyau. Dans Azure, elle commence par *linux16*.
1. À la fin de la ligne, ajoutez *rd.break* à la fin de la ligne. Laissez un espace entre la ligne du noyau et *rd.break*.

    Cette action interrompt le processus de démarrage avant que le contrôle ne passe de `initramfs` à `systemd`, comme décrit dans la [documentation Red Hat](https://aka.ms/rhel7rootpassword).
1. Appuyez sur Ctrl+X pour quitter et redémarrer votre ordinateur avec les paramètres appliqués.

   Après redémarrage, vous êtes amené en mode d’urgence avec un système de fichiers en lecture seule.

1. Entrez `mount -o remount,rw /sysroot` dans l’interpréteur de commandes pour remonter le système de fichiers racine avec des autorisations de lecture/écriture.
1. Après avoir démarré en mode mono-utilisateur, saisissez `chroot /sysroot` pour basculer vers la prison `sysroot`.
1. Vous êtes maintenant à la racine. Vous pouvez réinitialiser votre mot de passe racine en entrant `passwd`, puis suivre les instructions précédentes pour entrer en mode mono-utilisateur.
1. Une fois que vous avez terminé, entrez `reboot -f` pour redémarrer.

![Image animée montrant une interface de ligne de commande. L’utilisateur sélectionne un serveur, repère la fin de la ligne du noyau et entre les commandes spécifiées.](../media/virtual-machines-serial-console/virtual-machine-linux-serial-console-rhel-emergency-mount-no-root.gif)

> [!NOTE]
> Les instructions précédentes vous amènent dans l’interpréteur de commandes d’urgence, ce qui vous permet d’effectuer des tâches telles que la modification de `fstab`. Toutefois, nous vous suggérons généralement de réinitialiser votre mot de passe racine et de l’utiliser pour passer en mode mono-utilisateur.

## <a name="access-for-centos"></a>Accès CentOS
Un peu comme Red Hat Enterprise Linux, le mode mono-utilisateur dans CentOS nécessite l’activation du GRUB et de l’utilisateur racine.

### <a name="grub-access-in-centos"></a>Accès au GRUB dans CentOS
CentOS est fourni avec le GRUB activé par défaut. Pour entrer dans le GRUB, redémarrez votre machine virtuelle en entrant `sudo reboot` et appuyez sur n’importe quelle touche. Cette action affiche le volet du GRUB.

### <a name="single-user-mode-in-centos"></a>Mode mono-utilisateur dans CentOS
Suivez les instructions pour RHEL ci-dessus afin d’activer le mode mono-utilisateur dans CentOS.

## <a name="access-for-ubuntu"></a>Accès Ubuntu
Les images Ubuntu ne nécessitent pas de mot de passe racine. Si le système démarre en mode mono-utilisateur, vous pouvez l’utiliser sans informations d’identification supplémentaires.

### <a name="grub-access-in-ubuntu"></a>Accès au GRUB dans Ubuntu
Pour accéder au GRUB, maintenez la touche Échap enfoncée pendant le démarrage de la machine virtuelle.

Par défaut, les images Ubuntu peuvent ne pas afficher automatiquement le volet du GRUB. Vous pouvez modifier le paramètre en procédant comme suit :
1. Dans un éditeur de texte, ouvrez le fichier */etc/default/grub.d/50-cloudimg-settings.cfg*.

1. Remplacez la valeur `GRUB_TIMEOUT` par une valeur autre que zéro.
1. Dans un éditeur de texte, ouvrez */etc/default/grub*.
1. Commentez la ligne `GRUB_HIDDEN_TIMEOUT=1`.
1. Vérifiez qu’il y a une ligne `GRUB_TIMEOUT_STYLE=menu`.
1. Exécutez `sudo update-grub`.

### <a name="single-user-mode-in-ubuntu"></a>Mode mono-utilisateur dans Ubuntu
Ubuntu vous amène directement en mode mono-utilisateur s’il ne peut pas démarrer normalement. Pour entrer manuellement en mode mono-utilisateur, procédez comme suit :

1. Dans GRUB, appuyez sur E pour modifier votre entrée de démarrage (entrée Ubuntu).
1. Recherchez la ligne qui commence par *linux*, puis recherchez *ro*.
1. Ajoutez *single* après *ro*, tout en vous assurant de la présence d’un espace avant et après *single*.
1. Appuyez sur Ctrl+X pour redémarrer avec ces paramètres, puis entrez en mode mono-utilisateur.

### <a name="use-grub-to-invoke-bash-in-ubuntu"></a>Utiliser GRUB pour appeler Bash dans Ubuntu
Après avoir tenté d’appliquer les instructions précédentes, il peut arriver (par exemple, en cas d’oubli d’un mot de passe racine) que vous ne puissiez pas accéder au mode mono-utilisateur dans votre machine virtuelle Ubuntu. Vous pouvez également indiquer le noyau pour exécuter `/bin/bash` en tant qu’initialisation, plutôt que l’initialisation système. Cela vous donne un shell bash et permet la maintenance du système. Suivez ces instructions :

1. Dans GRUB, appuyez sur E pour modifier votre entrée de démarrage (entrée Ubuntu).

1. Recherchez la ligne qui commence par *linux*, puis recherchez *ro*.
1. Remplacez *ro* par *rw init=/bin/bash*.

    Cette action monte votre système de fichiers en lecture-écriture et utilise `/bin/bash` comme processus d’initialisation.
1. Appuyez sur Ctrl+X pour redémarrer avec ces paramètres.

## <a name="access-for-coreos"></a>Accès CoreOS
Le mode mono-utilisateur dans CoreOS nécessite l’activation du GRUB.

### <a name="grub-access-in-coreos"></a>Accès au GRUB dans CoreOS
Pour accéder au GRUB, appuyez sur n’importe quelle touche lors du démarrage de votre machine virtuelle.

### <a name="single-user-mode-in-coreos"></a>Mode mono-utilisateur dans CoreOS
CoreOS vous amène directement en mode mono-utilisateur s’il ne peut pas démarrer normalement. Pour entrer manuellement en mode mono-utilisateur, procédez comme suit :

1. Dans GRUB, appuyez sur E pour modifier votre entrée de démarrage.

1. Recherchez la ligne qui commence par *linux$* . Il doit y avoir deux instances de la ligne, chacune encapsulée dans une clause *if...else* différente.
1. Ajoutez *coreos.autologin=ttyS0* à la fin de chaque ligne *linux$* .
1. Appuyez sur Ctrl+X pour redémarrer avec ces paramètres, puis entrez en mode mono-utilisateur.

## <a name="access-for-suse-sles"></a>Accès SUSE SLES
Les images plus récentes de SLES 12 SP3+ autorisent l’accès par le biais de la console en série dans les situations où le système démarre en mode d’urgence.

### <a name="grub-access-in-suse-sles"></a>Accès au GRUB dans SUSE SLES
L’accès au GRUB dans SLES requiert une configuration de chargeur de démarrage par le biais de YaST. Pour créer la configuration, procédez comme suit :

1. Utilisez SSH pour vous connecter à votre machine virtuelle SLES, puis exécutez `sudo yast bootloader`. Appuyez sur Tab, puis Entrée, et utilisez alors les touches de direction pour naviguer dans le menu.

1. Accédez à **Paramètres du noyau**, puis activez la case à cocher **Utiliser la console série**.
1. Ajoutez `serial --unit=0 --speed=9600 --parity=no` aux arguments de la **Console**.
1. Appuyez sur F10 pour enregistrer vos paramètres et quitter l’écran.
1. Pour entrer dans le GRUB, redémarrez votre machine virtuelle et appuyez sur n’importe quelle touche pendant la séquence de démarrage pour que le volet du GRUB reste affiché.

    Le délai d’expiration par défaut pour le GRUB est de **1 s**. Vous pouvez modifier ce paramètre en modifiant la variable `GRUB_TIMEOUT` dans le fichier */etc/default/grub*.

![Initialisation de la configuration du chargeur de démarrage](../media/virtual-machines-serial-console/virtual-machine-linux-serial-console-sles-yast-grub-config.gif)

### <a name="single-user-mode-in-suse-sles"></a>Mode mono-utilisateur dans SUSE SLES
Vous êtes automatiquement amené dans l’interpréteur de commandes d’urgence si SLES ne peut pas démarrer normalement. Pour entrer manuellement dans l’interpréteur de commandes d’urgence, procédez comme suit :

1. Dans GRUB, appuyez sur E pour modifier votre entrée de démarrage (entrée SLES).

1. Recherchez la ligne du noyau qui commence par *linux*.
1. Ajoutez *systemd.unit=emergency.target* à la fin de la ligne du noyau.
1. Appuyez sur Ctrl+X pour redémarrer avec ces paramètres, puis entrez dans l’interpréteur de commandes d’urgence.

   > [!NOTE]
   > Cette action vous amène dans l’interpréteur de commandes d’urgence avec un système de fichiers en lecture seule. Pour modifier des fichiers, remontez le système de fichiers avec des autorisations en lecture-écriture. Pour ce faire, entrez `mount -o remount,rw /` dans l’interpréteur de commandes.

## <a name="access-for-oracle-linux"></a>Accès Oracle Linux
Un peu comme Red Hat Enterprise Linux, le mode mono-utilisateur dans Oracle Linux nécessite l’activation du GRUB et de l’utilisateur racine.

### <a name="grub-access-in-oracle-linux"></a>Accès au GRUB dans Oracle Linux
Oracle Linux est fourni avec le GRUB activé par défaut. Pour entrer dans le GRUB, redémarrez votre machine virtuelle en exécutant `sudo reboot` et appuyez sur Échap. Cette action affiche le volet du GRUB. Si le volet du GRUB ne s’affiche pas, vérifiez que la valeur de la ligne `GRUB_TERMINAL` contient *serial console* (c’est-à-dire `GRUB_TERMINAL="serial console"`). Reconstruisez GRUB avec `grub2-mkconfig -o /boot/grub/grub.cfg`.

### <a name="single-user-mode-in-oracle-linux"></a>Mode mono-utilisateur dans Oracle Linux
Suivez les instructions pour RHEL ci-dessus afin d’activer le mode mono-utilisateur dans Oracle Linux.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur la console série, consultez :
* [Documentation de la console série Linux](serial-console-linux.md)
* [Utilisation de la console série pour activer GRUB dans diverses distributions](http://linuxonazure.azurewebsites.net/why-proactively-ensuring-you-have-access-to-grub-and-sysrq-in-your-linux-vm-could-save-you-lots-of-down-time/)
* [Utiliser la console série pour les appels SysRq et NMI](serial-console-nmi-sysrq.md)
* [Machines virtuelles de console série pour Windows](serial-console-windows.md)
* [Diagnostics de démarrage](boot-diagnostics.md)
