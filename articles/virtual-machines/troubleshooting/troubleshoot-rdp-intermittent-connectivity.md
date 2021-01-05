---
title: Le Bureau à distance se déconnecte fréquemment sur une machine virtuelle Azure| Microsoft Docs
description: Découvrez comment résoudre les problèmes de déconnexions fréquentes du Bureau à distance sur une machine virtuelle Azure.
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/24/2018
ms.author: genli
ms.openlocfilehash: 60be7c234a0166331c35eb6528eae11bfbbf518f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87074314"
---
# <a name="remote-desktop-disconnects-frequently-in-azure-vm"></a>Le Bureau à distance se déconnecte fréquemment sur une machine virtuelle Azure

Cet article explique comment résoudre les problèmes de déconnexions fréquentes sur une machine virtuelle Azure par le biais du protocole RDP (Remote Desktop Protocol).


## <a name="symptom"></a>Symptôme

Vous rencontrez des problèmes intermittents de connectivité RDP au cours de vos sessions. Vous pouvez vous connecter à la machine virtuelle dans un premier temps, mais ensuite la connexion tombe.

## <a name="cause"></a>Cause

Ce problème peut survenir si l’écouteur RDP est mal configuré. Ce problème survient généralement sur une machine virtuelle qui utilise une image personnalisée.

## <a name="solution"></a>Solution

Avant de suivre cette procédure, [prenez un instantané du disque du système d’exploitation](../windows/snapshot-copy-managed-disk.md) de la machine virtuelle affectée en guise de sauvegarde. 

Pour résoudre ce problème, utilisez le contrôle série ou [réparez la machine virtuelle en mode hors connexion](#repair-the-vm-offline) en attachant le disque du système d’exploitation de la machine virtuelle à une machine virtuelle de récupération.

### <a name="serial-control"></a>Contrôle série

1. Connectez-vous à la [console série et ouvrez une instance CMD](./serial-console-windows.md). Ensuite, exécutez les commandes suivantes pour réinitialiser les configurations RDP. Si la console série n’est pas activée sur votre machine virtuelle, accédez à l’étape suivante.
2. Réduisez la couche de sécurité RDP à 0. Avec ce paramètre, les communications entre serveur et client utilisent le chiffrement RDP natif.

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'SecurityLayer' /t REG_DWORD /d 0 /f`

3. Réduisez le niveau de chiffrement au paramètre minimal pour autoriser les clients RDP hérités à se connecter.

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'MinEncryptionLevel' /t REG_DWORD /d 1 /f`

4. Définissez RDP pour charger la configuration utilisateur de l’ordinateur client.

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'fQueryUserConfigFromLocalMachine' /t REG_DWORD /d 1 /f`

5. Activez le contrôle RDP Keep-Alive :

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'KeepAliveTimeout' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'KeepAliveEnable' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'KeepAliveInterval' /t REG_DWORD /d 1 /f`

6. Définissez le contrôle de reconnexion du RDP :

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritReconnectSame' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'fReconnectSame' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'fDisableAutoReconnect' /t REG_DWORD /d 0 /f`

7. Définissez le contrôle d’heure de session du RDP :

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxSessionTime' /t REG_DWORD /d 1 /f`

8. Définissez le contrôle d’heure de déconnexion du RDP : 

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxDisconnectionTime' /t REG_DWORD /d 1 /f` `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxDisconnectionTime' /t REG_DWORD /d 0 /f`

9. Définissez le contrôle d’heure de connexion RDP :

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxConnectionTime' /t REG_DWORD /d 0 /f`

10. Définissez le contrôle de durée d’inactivité de la session du RDP :

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxIdleTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxIdleTime' /t REG_DWORD /d 0 /f`

11. Définissez le contrôle « Limiter le nombre maximal de connexions simultanées » :

    `REG ADD "HKLM\SYSTEM\CurrentControlSet\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxInstanceCount' /t REG_DWORD /d 4294967295 /f`

12. Redémarrez la machine virtuelle et essayez de vous y connecter à nouveau avec le protocole RDP (Remote Desktop Protocol).

### <a name="repair-the-vm-offline"></a>Réparer la machine virtuelle en mode hors connexion

1. [Attachez le disque du système d’exploitation à une machine virtuelle de récupération](./troubleshoot-recovery-disks-portal-windows.md).
2. Une fois le disque de système d’exploitation attaché à une machine virtuelle de récupération, vérifiez qu’il est marqué **En ligne** dans la console Disk Management. Notez la lettre de lecteur qui est affectée au disque du système d’exploitation attaché.
3. Sur le disque du système d’exploitation que vous avez attaché, accédez au dossier **\windows\system32\config**. Copiez tous les fichiers dans ce dossier de sauvegarde pour le cas où une restauration serait requise.
4. Démarrez l’Éditeur du Registre (regedit.exe).
5. Sélectionnez la clé **HKEY_LOCAL_MACHINE**. Dans le menu, sélectionnez **Fichier** > **Charger la ruche** :
6. Sur le disque du système d’exploitation que vous avez attaché, accédez au dossier **\windows\system32\config\SYSTEM**. Pour le nom de la ruche, entrez **BROKENSYSTEM**. La nouvelle ruche du Registre est affichée sous la clé **HKEY_LOCAL_MACHINE**. Chargez ensuite la ruche logicielle **\windows\system32\config\SOFTWARE** sous la clé **HKEY_LOCAL_MACHINE**. Pour le nom du logiciel de la ruche, entrez **BROKENSOFTWARE**. 
7. Ouvrez une fenêtre d’invite de commandes avec élévation de privilèges (**Exécuter en tant qu’administrateur**) et exécutez les commandes dans les étapes restantes pour réinitialiser les configurations RDP. 
8. Réduisez la couche de sécurité RDP à 0 afin que les communications entre le serveur et le client utilisent le chiffrement du RDP natif :

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'SecurityLayer' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'SecurityLayer' /t REG_DWORD /d 0 /f`

9. Réduisez le niveau de chiffrement au paramètre minimal pour autoriser les clients RDP hérités à se connecter :

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'MinEncryptionLevel' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'MinEncryptionLevel' /t REG_DWORD /d 1 /f`

10. Définissez RDP pour charger la configuration utilisateur de la machine cliente.

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fQueryUserConfigFromLocalMachine' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fQueryUserConfigFromLocalMachine' /t REG_DWORD /d 1 /f`

11. Activez le contrôle RDP Keep-Alive :

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'KeepAliveTimeout' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'KeepAliveTimeout' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'KeepAliveEnable' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'KeepAliveInterval' /t REG_DWORD /d 1 /f`

12. Définissez le contrôle de reconnexion du RDP :

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritReconnectSame' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fReconnectSame' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritReconnectSame' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fReconnectSame' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v 'fDisableAutoReconnect' /t REG_DWORD /d 0 /f`

13. Définissez le contrôle d’heure de session du RDP :

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxSessionTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxSessionTime' /t REG_DWORD /d 1 /f`

14. Définissez le contrôle d’heure de déconnexion du RDP :

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxDisconnectionTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxDisconnectionTime' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxDisconnectionTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxDisconnectionTime' /t REG_DWORD /d 0 /f`

15. Définissez le contrôle d’heure de connexion RDP :

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxConnectionTime' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxConnectionTime' /t REG_DWORD /d 0 /f`

16. Définissez le contrôle de durée d’inactivité de la session du RDP :

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxIdleTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v ' MaxIdleTime' /t REG_DWORD /d 0 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'fInheritMaxIdleTime' /t REG_DWORD /d 1 /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v ' MaxIdleTime' /t REG_DWORD /d 0 /f`

17. Définissez le contrôle « Limiter le nombre maximal de connexions simultanées » :

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxInstanceCount' /t REG_DWORD /d ffffffff /f`

    `REG ADD "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v 'MaxInstanceCount' /t REG_DWORD /d ffffffff /f`

18. Redémarrez la machine virtuelle et essayez de vous y connecter à nouveau avec le protocole RDP (Remote Desktop Protocol).

## <a name="need-help"></a>Vous avez besoin d’aide ? 
Contactez le support technique. Si vous avez besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir une prise en charge rapide de votre problème.
