---
title: Configuration GRUB proactive de la console série Azure | Microsoft Docs
description: Configurez GRUB sur différentes distributions pour permettre l’accès en mode mono-utilisateur et en mode de récupération dans les machines virtuelles Azure.
services: virtual-machines-linux
documentationcenter: ''
author: mimckitt
manager: vashan
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/10/2019
ms.author: mimckitt
ms.openlocfilehash: aba47500400004c1d6a7044a266bad6f20d5d9c9
ms.sourcegitcommit: d95cab0514dd0956c13b9d64d98fdae2bc3569a0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91360546"
---
# <a name="proactively-ensuring-you-have-access-to-grub-and-sysrq-could-save-you-lots-of-down-time"></a>Vous assurer de manière proactive que vous avez accès à GRUB et sysrq peut vous faire gagner beaucoup de temps

Le fait d’avoir accès à la Serial console et à GRUB permet d’améliorer les délais de récupération de votre machine virtuelle Linux IaaS dans la plupart des cas. Si les options de récupération offertes par GRUB étaient absentes, il faudrait plus de temps pour récupérer votre machine virtuelle. 


Les raisons d’effectuer une récupération de machine virtuelle sont nombreuses et peuvent être attribuées à des scénarios tels que :

   - Systèmes de fichiers/noyau/MBR (enregistrement de démarrage principal) endommagés
   - Échec des mises à niveau du noyau
   - Paramètres incorrects du noyau GRUB
   - Configurations fstab incorrectes
   - Configurations de pare-feu
   - Mot de passe perdu
   - Fichiers de configuration sshd altérés
   - Configurations de mise en réseau

 De nombreux autres scénarios, comme détaillé [ici](./serial-console-linux.md#common-scenarios-for-accessing-the-serial-console)

Vérifiez que vous pouvez accéder à GRUB et à la Serial console sur vos machines virtuelles déployées dans Azure. 

Si vous débutez avec la Serial console, reportez-vous à [ce lien](./serial-console-linux.md).

> [!TIP]
> Veillez à effectuer des sauvegardes de fichiers avant d’apporter des modifications

Regardez cette vidéo ci-dessous pour voir comment récupérer rapidement votre machine virtuelle Linux une fois que vous avez accès à GRUB

[Vidéo sur la récupération de la machine virtuelle Linux](https://youtu.be/KevOc3d_SG4)

Il existe plusieurs méthodes pour faciliter la récupération des machines virtuelles Linux. Dans un environnement cloud, ce processus a été difficile.
Des progrès sont continuellement effectués en matière d’outils et de caractéristiques pour garantir la récupération rapide des services.

Avec la Serial console Azure, vous pouvez interagir avec votre machine virtuelle Linux comme si vous étiez sur la console du système.

Vous pouvez manipuler de nombreux fichiers de configuration, notamment le mode de démarrage du noyau. 

Les administrateurs système Linux/Unix plus expérimentés apprécieront le **mode mono-utilisateur** et le **mode d’urgence** qui sont accessibles via la Serial console Azure, ce qui rend redondants l’échange de disque et la suppression de machines virtuelles pour de nombreux scénarios de récupération.

La méthode de récupération dépend du problème rencontré, par exemple un mot de passe perdu ou égaré peut être réinitialisé par le biais des options du Portail Azure, > **Réinitialiser le mot de passe**. La caractéristique **Réinitialiser le mot de passe** est connue sous le nom d’extension et communique avec l’agent invité Linux.

D’autres extensions, telles que le script personnalisé, sont disponibles. Toutefois, ces options requièrent que le **waagent** Linux soit opérationnel et dans un état sain, ce qui n’est pas toujours le cas.

![état de l’agent](./media/virtual-machines-serial-console/agent-status.png)


La garantie de pouvoir accéder à la Serial console Azure et à GRUB signifie qu’une modification de mot de passe ou une configuration incorrecte peut être corrigée en quelques minutes au lieu de quelques heures. Vous pouvez même forcer le démarrage de la machine virtuelle à partir d’un autre noyau si vous avez plusieurs noyaux sur le disque dans le cas où votre noyau principal serait endommagé.

![multi-noyau](./media/virtual-machines-serial-console/more-kernel.png)

## <a name="suggested-order-of-recovery-methods"></a>Ordre suggéré des méthodes de récupération :

- Console série Azure

- Échange de disque : peut être automatisé à l’aide de l’un des éléments suivants :

   - [Scripts de récupération PowerShell](https://github.com/Azure/azure-support-scripts/tree/master/VMRecovery/ResourceManager)
   - [Scripts de récupération bash](https://github.com/sribs/azure-support-scripts)

- Méthode héritée

## <a name="disk-swap-video"></a>Vidéo sur l’échange de disque :

Si vous n’avez pas accès à GRUB, regardez [cette vidéo](https://youtu.be/m5t0GZ5oGAc) et découvrez comment vous pouvez facilement automatiser la procédure d’échange de disque pour récupérer votre machine virtuelle.

## <a name="challenges"></a>Défis :

Toutes les machines virtuelles Azure Linux ne sont pas configurées par défaut pour l’accès à GRUB et elles ne sont pas non plus toutes configurées pour être interrompues avec les commandes sysrq. Certains anciennes distributions telles que SLES 11 ne sont pas configurées pour afficher l’invite de connexion dans la Serial console Azure

Dans cet article, nous allons examiner différentes distributions Linux et documenter des configurations sur la manière de rendre GRUB disponible.




## <a name="how-to-configure-linux-vm-to-accept-sysrq-keys"></a>Comment configurer une machine virtuelle Linux pour accepter des clés SysRq
La clé sysrq est activée sur certaines distributions Linux plus récentes par défaut, bien que sur d’autres, elle puisse être configurée pour accepter des valeurs uniquement pour certaines fonctions SysRq.
Sur les distributions plus anciennes, il est possible qu’elle soit complètement désactivée.

La caractéristique SysRq est utile pour redémarrer une machine virtuelle bloquée ou qui ne répond pas directement à partir de la console série Azure, ce qui permet également d’accéder au menu GRUB. Vous pouvez également redémarrer une machine virtuelle à partir d’une autre fenêtre du portail ou d’une session ssh pour supprimer la connexion à la console actuelle, ce qui a pour conséquence l’expiration des délais d’attente utilisés pour afficher le menu GRUB.
La machine virtuelle doit être configurée pour accepter la valeur 1 pour le paramètre kernel, ce qui active toutes les fonctions de sysrq ou 128, qui permet le redémarrage ou mise hors tension


[Vidéo sur l’activation de sysrq](https://youtu.be/0doqFRrHz_Mc)


Pour configurer la machine virtuelle afin qu’elle accepte un redémarrage via les commandes SysRq sur le Portail Azure, vous devez définir la valeur 1 pour le paramètre de noyau kernel.sysrq

Pour que cette configuration soit conservée après un redémarrage, ajoutez une entrée au fichier **sysctl.conf**

`echo kernel.sysrq = 1 >> /etc/sysctl.conf`

Pour configurer le paramètre kernel de manière dynamique

`sysctl -w kernel.sysrq=1`

Si vous n’avez pas d’accès **root** ou que sudo est rompu, il ne sera pas possible de configurer sysrq à partir d’une invite d’interpréteur de commandes.

Vous pouvez activer sysrq dans ce scénario à l’aide du Portail Azure. Cette méthode peut être bénéfique si le fichier **sudoers.d/waagent** est endommagé ou a été supprimé.

L’utilisation de la caractéristique du Portail Azure Opérations -> Run Command -> RunShellScript requiert que le processus soit sain. Vous pouvez ensuite injecter cette commande pour activer sysrq

`sysctl -w kernel.sysrq=1 ; echo kernel.sysrq = 1 >> /etc/sysctl.conf`

Comme illustré ici : ![activer sysrq2](./media/virtual-machines-serial-console/enabling-sysrq-2.png)

Une fois l’opération terminée, vous pouvez essayer d’accéder à **sysrq** et vous devriez constater qu’un redémarrage est possible.

![activer sysrq3](./media/virtual-machines-serial-console/enabling-sysrq-3.png)

Sélectionnez **Redémarrer** et la commande **Envoyer SysRq**

![activer sysrq4](./media/virtual-machines-serial-console/enabling-sysrq-4.png)

Le système doit enregistrer un message de réinitialisation tel que celui-ci

![activer sysrq5](./media/virtual-machines-serial-console/enabling-sysrq-5.png)


## <a name="ubuntu-grub-configuration"></a>Configuration d’Ubuntu GRUB

Par défaut, vous devez être en mesure d’accéder à GRUB en maintenant la touche **Échap** enfoncée pendant le démarrage de la machine virtuelle. Si le menu GRUB ne s’affiche pas, vous pouvez forcer et conserver le menu GRUB à l’écran dans la Serial console Azure en utilisant l’une des options suivantes.

**Option 1** : force l’affichage de GRUB à l’écran 

Mettez à jour le fichier/etc/default/grub.d/50-cloudimg-settings.cfg pour conserver le menu GRUB à l’écran pour le TIMEOUT spécifié.
Vous n’êtes pas obligé d’appuyer sur la touche **Échap**, car GRUB s’affiche immédiatement

```
GRUB_TIMEOUT=0

change to

GRUB_TIMEOUT=5
```

**Option 2** : permet d’appuyer sur **Échap** avant le démarrage

Un comportement similaire peut être rencontré en apportant des modifications au fichier /etc/default/grub et en observant un délai d’expiration de 3 secondes avant d’appuyer sur la touche **Échap**


Convertissez ces deux lignes en commentaires :

```
#GRUB_HIDDEN_TIMEOUT=0
#GRUB_HIDDEN_TIMEOUT_QUIET=true
```
et ajoutez cette ligne :

```
GRUB_TIMEOUT_STYLE=countdown
```


## <a name="ubuntu-1204"></a>Ubuntu 12\.04

Ubuntu 12.04 autorise l’accès à la console série, mais n’offre pas la possibilité d’interagir. Aucune invite **login:** ne s’affiche


Pour afficher une invite **login:** dans 12.04 :
1. Créez un fichier appelé /etc/init/ttyS0.conf contenant le texte suivant :

    ```
    # ttyS0 - getty
    #
    # This service maintains a getty on ttyS0 from the point the system is
    # started until it is shut down again.
    start on stopped rc RUNLEVEL=[12345]
    stop on runlevel [!12345]
    
    respawn
    exec /sbin/getty -L 115200 ttyS0 vt102
    ```    

2. Demandez à upstart de démarrer le getty     
    ```
    sudo start ttyS0
    ```
 
Vous trouverez les paramètres requis pour configurer la console série pour les versions Ubuntu [ici](https://help.ubuntu.com/community/SerialConsoleHowto)

## <a name="ubuntu-recovery-mode"></a>Mode de récupération Ubuntu

Des options de récupération et de nettoyage supplémentaires sont disponibles pour Ubuntu via GRUB. Toutefois, ces paramètres sont accessibles uniquement si vous configurez les paramètres du noyau en conséquence.
Si vous ne configurez pas ce paramètre de démarrage du noyau, le menu de récupération sera envoyé à Azure Diagnostics, et non à la Serial console Azure.
Vous pouvez obtenir l’accès au menu de récupération Ubuntu en procédant comme suit :

Interrompre le processus BOOT et accéder au menu GRUB

Sélectionnez Options avancées pour Ubuntu, puis appuyez sur Entrée

![La capture d’écran représente la Serial console dans laquelle les options avancées pour Ubuntu sont sélectionnées.](./media/virtual-machines-serial-console/ubunturec1.png)

Sélectionnez la ligne qui affiche *(mode de récupération)* . N’appuyez pas sur Entrée mais bien sur « e »

![La capture d’écran représente la Serial console dans laquelle la version du mode récupération est sélectionnée.](./media/virtual-machines-serial-console/ubunturec2.png)

Localisez la ligne qui chargera le noyau et remplacez le dernier paramètre **nomodeset** par la destination en tant que **console=ttyS0**

```
linux /boot/vmlinuz-4.15.0-1023-azure root=UUID=21b294f1-25bd-4265-9c4e-d6e4aeb57e97 ro recovery nomodeset

change to

linux /boot/vmlinuz-4.15.0-1023-azure root=UUID=21b294f1-25bd-4265-9c4e-d6e4aeb57e97 ro recovery console=ttyS0
```

![Capture d’écran représentant la Serial console avec la valeur modifiée.](./media/virtual-machines-serial-console/ubunturec3.png)

Appuyez sur **Ctrl-x** pour démarrer et charger le noyau.
Si tout se passe bien, vous verrez ces options supplémentaires, qui peuvent vous aider à effectuer d’autres options de récupération

![Capture d’écran représentant la Serial console dans le menu Récupération, qui offre des options de récupération supplémentaires.](./media/virtual-machines-serial-console/ubunturec4.png)


## <a name="red-hat-grub-configuration"></a>Configuration de Red Hat GRUB

## <a name="red-hat-74-grub-configuration"></a>Configuration de Red Hat 7\.4\+ GRUB
La configuration /etc/default/grub par défaut sur ces versions est correctement réalisée

```
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL="serial console"
GRUB_SERIAL_COMMAND="serial"
GRUB_CMDLINE_LINUX="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"
GRUB_DISABLE_RECOVERY="true"
```

Activer la clé SysRq

```
sysctl -w kernel.sysrq=1;echo kernel.sysrq = 1 >> /etc/sysctl.conf;sysctl -a | grep -i sysrq
```

## <a name="red-hat-72-and-73-grub-configuration"></a>Configuration de Red Hat 7\.2 et 7\.3 GRUB
Le fichier à modifier est /etc/default/grub – une configuration par défaut ressemble à l’exemple suivant :

```
GRUB_TIMEOUT=1
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"
GRUB_DISABLE_RECOVERY="true"
```

Modifiez les lignes suivantes dans /etc/default/grub

```
GRUB_TIMEOUT=1 

to

GRUB_TIMEOUT=5
```


```
GRUB_TERMINAL_OUTPUT="console"

to

GRUB_TERMINAL="serial console"
```

Ajoutez également la ligne suivante :

```
GRUB_SERIAL_COMMAND=”serial –speed=115200 –unit=0 –word=8 –parity=no –stop=1″
```

/etc/default/grub doit maintenant ressembler à cet exemple :

```
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL="serial console"
GRUB_CMDLINE_LINUX="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"
GRUB_DISABLE_RECOVERY="true"
```
 
Terminez et mettez à jour la configuration grub avec

`grub2-mkconfig -o /boot/grub2/grub.cfg`

Définissez le paramètre de noyau SysRq :

`sysctl -w kernel.sysrq = 1;echo kernel.sysrq = 1 >> /etc/sysctl.conf;sysctl -a | grep -i sysrq`

Vous pouvez également configurer GRUB et SysRq à l’aide d’une seule ligne dans l’interpréteur de commandes ou via Run Command. Sauvegardez vos fichiers avant d’exécuter cette commande :


`cp /etc/default/grub /etc/default/grub.bak; sed -i 's/GRUB_TIMEOUT=1/GRUB_TIMEOUT=5/g' /etc/default/grub; sed -i 's/GRUB_TERMINAL_OUTPUT="console"/GRUB_TERMINAL="serial console"/g' /etc/default/grub; echo "GRUB_SERIAL_COMMAND=\"serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1\"" >> /etc/default/grub;grub2-mkconfig -o /boot/grub2/grub.cfg;sysctl -w kernel.sysrq=1;echo kernel.sysrq = 1 /etc/sysctl.conf;sysctl -a | grep -i sysrq`


## <a name="red-hat-6x-grub-configuration"></a>Configuration de Red Hat 6\.x GRUB
Le fichier à modifier est /boot/grub/grub.conf. La valeur `timeout` détermine la durée pendant laquelle GRUB est affiché.

```
#boot=/dev/vda
default=0
timeout=15
splashimage=(hd0,0)/grub/splash.xpm.gz
#hiddenmenu
serial --unit=0 --speed=9600
terminal serial
terminal --timeout=5 serial console
```


La dernière ligne *terminal –-timeout = 5 serial console* augmente davantage le délai d’expiration de **GRUB** en ajoutant une invite de 5 secondes pour afficher **Appuyez sur une touche pour continuer.**

![Capture d’écran représentant une console avec la sortie.](./media/virtual-machines-serial-console/rh6-1.png)

Le menu GRUB doit s’afficher à l’écran pour le délai d’expiration configuré timeout=15 sans avoir besoin d’appuyer sur Échap. Veillez à cliquer dans la console du navigateur pour activer le menu et sélectionner le noyau requis

![Capture d’écran représentant une console avec deux options Linux.](./media/virtual-machines-serial-console/rh6-2.png)

## <a name="suse"></a>SuSE

## <a name="sles-12-sp1"></a>SLES 12 SP1
Utilisez le chargeur de YaST conformément aux [documents](./serial-console-grub-single-user-mode.md#grub-access-in-suse-sles) officiels

Ou ajoutez ou modifiez les paramètres suivants dans /etc/default/grub :

```
GRUB_TERMINAL=serial
GRUB_TIMEOUT=5
GRUB_SERIAL_COMMAND="serial --unit=0 --speed=9600 --parity=no"

```
Vérifiez que ttyS0 est utilisé dans GRUB_CMDLINE_LINUX ou GRUB_CMDLINE_LINUX_DEFAULT

```
GRUB_CMDLINE_LINUX_DEFAULT="console=ttyS0,9600n"
```

Recréez le fichier grub. cfg

`grub2-mkconfig -o /boot/grub2/grub.cfg`


## <a name="sles-11-sp4"></a>SLES 11 SP4 
La Serial console apparaît et affiche les messages de démarrage, mais n’affiche pas d’invite **login:**

Ouvrez une session SSH sur la machine virtuelle et mettez à jour le fichier **/etc/inittab** en annulant les commentaires de cette ligne :

```
#S0:12345:respawn:/sbin/agetty -L 9600 ttyS0 vt102
```

Exécutez ensuite la commande 

`telinit q`

Pour activer GRUB, les modifications suivantes doivent être apportées à /boot/grub/menu.lst 

```
timeout 5
serial --unit=0 --speed=9600 --parity=no
terminal --timeout=5 serial console

root (hd0,0)
kernel /boot/vmlinuz-3.0.101-108.74-default root=/dev/disk/by-uuid/ab6b62bb--
1a8c-45eb-96b1-1fbc535b9265 disk=/dev/sda  USE_BY_UUID_DEVICE_NAMES=1 earlyprinttk=ttyS0 console=ttyS0 rootdelay=300  showopts vga=0x314
```

 Cette configuration permet d’afficher le message **Appuyez sur n’importe quelle touche pour continuer** sur la console pendant 5 secondes 

Elle affiche ensuite le menu GRUB pendant 5 secondes supplémentaires. En appuyant sur la flèche orientée vers le bas, vous interrompez le compteur. Pour sélectionner un noyau que vous souhaitez démarrer, ajoutez le mot clé **single** pour le mode mono-utilisateur qui requiert la définition du mot de passe racine.

L’ajout de la commande **init=/bin/bash** charge le noyau, mais garantit que le programme init est remplacé par un interpréteur de commandes bash.

Vous pouvez accéder à un interpréteur de commandes sans avoir à entrer un mot de passe. Vous pouvez ensuite mettre à jour le mot de passe pour les comptes Linux ou apporter d’autres modifications à la configuration.


## <a name="force-the-kernel-to-a-bash-prompt"></a>Forcer le noyau à afficher une invite bash
L’accès à GRUB vous permet d’interrompre le processus d’initialisation. Cette interaction est utile pour de nombreuses procédures de récupération.
Si vous n’avez pas de mot de passe racine et que le mode mono-utilisateur requiert que vous ayez un mot de passe racine, vous pouvez démarrer le noyau en remplaçant le programme init par une invite bash. Cette interruption peut être obtenue en ajoutant init=/bin/bash à la ligne de démarrage du noyau

![Capture d’écran représentant une console avec la ligne de démarrage mise à jour.](./media/virtual-machines-serial-console/bash1.png)

Remontez votre système de fichiers / (racine) RW à l’aide de la commande

`mount -o remount,rw /`

![Capture d’écran représentant une console avec une action de remontage.](./media/virtual-machines-serial-console/bash2.png)


Vous pouvez maintenant effectuer une modification du mot de passe racine ou de nombreuses autres modifications de configuration Linux

![Capture d’écran représentant une console dans laquelle vous pouvez modifier le mot de passe racine et d’autres configurations.](./media/virtual-machines-serial-console/bash3.png)

Redémarrez la machine virtuelle avec 

`/sbin/reboot -f`




## <a name="single-user-mode"></a>Mode mono-utilisateur

Vous pouvez également avoir besoin d’accéder à la machine virtuelle en mode mon-utilisateur ou en mode d’urgence. Sélectionnez le noyau que vous souhaitez démarrer ou interrompre à l’aide des touches de direction.
Accédez au mode souhaité en ajoutant le mot clé **single** ou **1** à la ligne de démarrage du noyau. Sur les systèmes RHEL, vous pouvez également ajouter **rd.break**.

Pour plus d’informations sur l’accès au mode mono-utilisateur, consultez [ce document](./serial-console-grub-single-user-mode.md#general-single-user-mode-access) 


![single_user_ubuntu](./media/virtual-machines-serial-console/single-user-ubuntu.png)


## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur la [Serial console Azure]( ./serial-console-linux.md)