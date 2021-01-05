---
title: Console série Azure pour les appels SysRq et NMI | Microsoft Docs
description: Utilisation de la console série pour les appels SysRq et NMI dans les machines virtuelles Azure.
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
ms.date: 08/14/2018
ms.author: alsin
ms.openlocfilehash: 545399e1d7941351ce861ac98d995d5e57006ea1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022851"
---
# <a name="use-the-azure-serial-console-for-sysrq-and-nmi-calls"></a>Utiliser la console série Azure pour les appels SysRq et NMI

## <a name="system-request-sysrq"></a>Requête système (SysRq)
Une SysRq est une séquence de clés comprise par le noyau du système d’exploitation Linux et pouvant déclencher un ensemble d’actions prédéfinies. Ces commandes sont souvent utilisées quand l’administration classique ne permet pas de résoudre les problèmes ou de récupérer les machines virtuelles (par exemple, quand celles-ci ne répondent pas). La fonction SysRq de la console série Azure permet de simuler l’appui sur la touche SysRq et la saisie de caractères sur un clavier physique.

Une fois que la séquence SysRq est livrée, la configuration du noyau contrôle le mode de réponse du système. Pour plus d’informations sur l’activation et la désactivation de SysRq, consultez le *SysRq Admin Guide* (Guide administrateur SysRq) [texte](https://aka.ms/kernelorgsysreqdoc) | [markdown](https://aka.ms/linuxsysrq).

La console série Azure peut être utilisée pour envoyer une SysRq à une machine virtuelle Azure via l’icône de clavier dans la barre de commande ci-dessous.

![Capture d’écran de la console série Azure. L’icône de clavier est mise en surbrillance et son menu est visible. Ce menu contient une option Envoyer une commande SysRq.](../media/virtual-machines-serial-console/virtual-machine-serial-console-command-menu.jpg)

En sélectionnant « Envoyer une commande SysRq », une boîte de dialogue s’ouvre et propose des options SysRq communes ou accepte une séquence de commandes SysRq entrées dans la boîte de dialogue.  Une série de SysRq peut ainsi exécuter une opération de haut niveau, par exemple un redémarrage sûr : `REISUB`.

![Capture d’écran de la boîte de dialogue Envoyer une commande SysRq à un invité. L’option permettant d’entrer des commandes est sélectionnée et la zone de commande contient REISUB.](../media/virtual-machines-serial-console/virtual-machine-serial-console-sysreq_UI.png)

La commande SysRq ne peut pas être utilisée sur les machines virtuelles arrêtées ou dont le noyau n’est pas en état réactif. (par exemple un état d’alerte du noyau).

### <a name="enable-sysrq"></a>Activer SysRq
Comme décrit dans le *Guide administrateur SysRq* ci-dessus, il est possible de configurer SysRq de façon à ce que toutes, aucune ou seulement certaines commandes soient disponibles. Vous pouvez activer toutes les commandes SysRq en suivant l’étape ci-dessous, mais cette configuration ne sera pas conservée après un redémarrage :
```
echo "1" >/proc/sys/kernel/sysrq
```
Pour conserver la configuration de SysReq, vous pouvez procéder comme suit pour activer toutes les commandes SysRq.
1. Ajout de cette ligne à */etc/sysctl.conf* <br>
    `kernel.sysrq = 1`
1. Redémarrage ou mise à jour de sysctl en exécutant <br>
    `sysctl -p`

### <a name="command-keys"></a>Touches de commande
Dans le guide administrateur SysRq ci-dessus :

|Commande| Fonction
| ------| ----------- |
|``b``  |   Redémarre immédiatement le système sans synchroniser ni démonter vos disques.
|``c``  |   Effectue un incident système en déréférençant un pointeur NULL. Une copie de sauvegarde sera effectuée si elle est configurée.
|``d``  |   Affiche tous les verrous maintenus.
|``e``  |   Envoie un SIGTERM à tous les processus, sauf init.
|``f``  |   Appelle le débogueur de mémoire insuffisante pour arrêter un processus trop gourmand en mémoire, mais ne vous inquiétez pas si rien ne peut être arrêté.
|``g``  |   Utilisé par kgdb (débogueur de noyau)
|``h``  |   Affiche l’aide (les touches qui ne sont pas mentionnées ici affichent aussi l’aide, mais il est facile de mémoriser ``h`` :-)
|``i``  |    Envoie un SIGKILL à tous les processus, sauf init.
|``j``  |    Forçage de « Just thaw it » (simple libération) - systèmes de fichiers gelés par l’ioctl FIFREEZE.
|``k``  |    La clé d’accès sécurisée (SAK) arrête tous les programmes de la console virtuelle en cours. REMARQUE :  consultez les commentaires importants disponibles ci-dessous, à la section SAK.
|``l``  |    Affiche un historique des piles de tous les processeurs actifs.
|``m``  |    Sauvegarde les informations de la mémoire actuelle sur votre console.
|``n``  |    Utilisé pour que les tâches RT soient agréables
|``o``  |    Ferme votre système (si configuré et pris en charge).
|``p``  |    Sauvegarde les registres et les indicateurs actuels sur votre console.
|``q``  |    Sauvegarde par listes de processeur de tous les minuteurs haute résolution armés (mais PAS les minuteurs timer_list classiques) et informations détaillées de tous les périphériques clockevent.
|``r``  |    Désactive le mode brut du clavier et le définit à XLATE.
|``s``  |    Tente de synchroniser tous les systèmes de fichiers montés.
|``t``  |    Sauvegarde une liste des tâches actuelles et leurs informations sur votre console.
|``u``  |    Tente de remonter tous les systèmes de fichiers montés en lecture seule.
|``v``  |    Force la restauration de la console framebuffer
|``v``  |    Lance la copie de sauvegarde de la mémoire tampon ETM [spécifique à l’ARM]
|``w``  |    Vide les tâches dont l'état est ininterrompu (bloqué).
|``x``  |    Utilisé par l’interface xmon sur les plateformes ppc/powerpc. Affiche les registres PMU globaux sur sparc64. Sauvegarde toutes les entrées TLB sur MIPS.
|``y``  |    Affiche les registres CPU globaux [spécifique à SPARC-64]
|``z``  |    Sauvegarde la mémoire tampon ftrace
|``0``-``9`` | Définit le niveau du journal de la console, en déterminant quels messages du noyau doivent être imprimés sur votre console. (``0``, par exemple, permet de n’afficher que les messages d’urgence comme PANIC ou OOPS sur votre console.)

### <a name="distribution-specific-documentation"></a>Documentation propre à la distribution ###
Pour obtenir la documentation propre à la distribution sur SysRq et les étapes de configuration de Linux permettant de créer un vidage sur incident à la réception d’une commande « Crash » de SysRq, consultez les liens ci-dessous :

#### <a name="ubuntu"></a>Ubuntu ####
 - [Vidage sur incident du noyau](https://help.ubuntu.com/lts/serverguide/kernel-crash-dump.html)

#### <a name="red-hat"></a>Red Hat ####
- [En quoi consiste la fonctionnalité SysRq et comment l’utiliser ?](https://access.redhat.com/articles/231663)
- [Collecter des informations à partir d’un serveur RHEL à l’aide de la fonctionnalité SysRq](https://access.redhat.com/solutions/2023)

#### <a name="suse"></a>SUSE ####
- [Configurer la capture de la sauvegarde du noyau central](https://www.suse.com/support/kb/doc/?id=3374462)

#### <a name="coreos"></a>CoreOS ####
- [Collecte des journaux d’activité d’incident](https://coreos.com/os/docs/latest/collecting-crash-logs.html)

## <a name="non-maskable-interrupt-nmi"></a>Interruption non masquable (NMI)
Une interruption non masquable (NMI) est conçue pour créer un signal que les logiciels sur une machine virtuelle n’ignoreront pas. À l’origine, les NMI ont été utilisées pour surveiller les problèmes matériels sur les systèmes nécessitant des temps de réponse spécifiques.  Aujourd’hui, les programmeurs et les administrateurs système utilisent souvent NMI comme mécanisme de débogage ou de dépannage des systèmes qui ne répondent pas.

La console série peut être utilisée pour envoyer une NMI à une machine virtuelle Azure à l’aide de l’icône de clavier dans la barre de commandes ci-dessous. Une fois que la NMI est remise, la configuration de machine virtuelle contrôle le mode de réponse du système.  Les systèmes d’exploitation Linux peuvent être configurés de façon à provoquer un incident et à créer une image mémoire lorsque le système d’exploitation reçoit une NMI.

![Capture d’écran de la console série. L’icône de clavier est mise en surbrillance et son menu est visible. Ce menu contient une option Envoyer une interruption non masquable.](../media/virtual-machines-serial-console/virtual-machine-serial-console-command-menu.jpg) <br>

### <a name="enable-nmi"></a>Activer une NMI
Pour les systèmes Linux qui prennent en charge sysctl pour configurer les paramètres du noyau, vous pouvez activer une alerte lors de la réception de la NMI en procédant comme suit :
1. Ajout de cette ligne à */etc/sysctl.conf* <br>
    `kernel.panic_on_unrecovered_nmi=1`
1. Redémarrage ou mise à jour de sysctl en exécutant <br>
    `sysctl -p`

Pour plus d'informations sur les configurations du noyau Linux, notamment `unknown_nmi_panic`, `panic_on_io_nmi` et `panic_on_unrecovered_nmi`, consultez ce qui suit : [Documentation de /proc/sys/kernel/*](https://www.kernel.org/doc/Documentation/sysctl/kernel.txt). Pour obtenir de la documentation propre à la distribution sur NMI et les étapes de configuration de Linux pour créer un vidage sur incident à la réception d’une NMI, consultez les liens ci-dessous :

### <a name="ubuntu"></a>Ubuntu
 - [Vidage sur incident du noyau](https://help.ubuntu.com/lts/serverguide/kernel-crash-dump.html)

### <a name="red-hat"></a>Red Hat
 - [En quoi consiste une NMI et à quoi peut-elle servir ?](https://access.redhat.com/solutions/4127)
 - [Comment configurer mon système pour qu’il provoque un incident lorsque le commutateur NMI est actionné ?](https://access.redhat.com/solutions/125103)
 - [Guide administrateur de vidage sur incident](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/pdf/kernel_crash_dump_guide/kernel-crash-dump-guide.pdf)

### <a name="suse"></a>SUSE
- [Configurer la capture de la sauvegarde du noyau central](https://www.suse.com/support/kb/doc/?id=3374462)

### <a name="coreos"></a>CoreOS
- [Collecte des journaux d’activité d’incident](https://coreos.com/os/docs/latest/collecting-crash-logs.html)

## <a name="next-steps"></a>Étapes suivantes
* La page principale de documentation de la console série Linux se trouve [ici](serial-console-linux.md).
* Utiliser la console série pour [démarrer dans GRUB et entrer en mode mono-utilisateur](serial-console-grub-single-user-mode.md)
* La console série est également disponible pour les machines virtuelles [Windows](serial-console-windows.md)
* En savoir plus sur les [diagnostics de démarrage](boot-diagnostics.md)