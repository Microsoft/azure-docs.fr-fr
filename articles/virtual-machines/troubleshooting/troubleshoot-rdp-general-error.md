---
title: Résoudre une erreur générale RDP sur une machine virtuelle Windows dans Azure | Microsoft Docs
description: Apprendre à résoudre une erreur générale RDP sur une machine virtuelle Windows dans Azure | Microsoft Docs
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: f2a1a5f3eaf79a345b0d33f43d260fe6aa15236b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87439257"
---
# <a name="troubleshoot-an-rdp-general-error-in-azure-vm"></a>Résoudre une erreur générale RDP sur une machine virtuelle Azure

Cet article décrit une erreur générale que vous êtes susceptible de rencontrer au moment d’établir une connexion RDP (Remote Desktop Protocol) à une machine virtuelle Windows dans Azure.

## <a name="symptom"></a>Symptôme

Au moment d’établir une connexion RDP à une machine virtuelle Windows dans Azure, vous pouvez obtenir le message d’erreur général suivant :

**Le Bureau à distance ne peut pas se connecter à l’ordinateur distant pour l’une des raisons suivantes :**

1. **L’accès à distance au serveur n’est pas activé**

2. **L’ordinateur distant est éteint**

3. **L’ordinateur distant n’est pas disponible sur le réseau**

**Vérifiez que l’ordinateur distant est allumé et connecté au réseau et que l’accès à distance est activé.**

## <a name="cause"></a>Cause

Ce problème peut se produire pour les raisons suivantes :

### <a name="cause-1"></a>Cause 1

Le composant RDP est désactivé comme suit :

- Au niveau du composant
- Au niveau de l’écouteur
- Sur le serveur Bureau à distance
- Sur le rôle Hôte de session Bureau à distance

### <a name="cause-2"></a>Cause 2

Les services Bureau à distance (TermService) ne sont pas en cours d’exécution.

### <a name="cause-3"></a>Cause 3

L’écouteur RDP est mal configuré.

## <a name="solution"></a>Solution

Avant de suivre cette procédure, faites en sauvegarde en prenant un instantané du disque du système d’exploitation de la machine virtuelle affectée. Pour résoudre ce problème, utilisez le contrôle série ou réparez la machine virtuelle en mode hors connexion.

### <a name="serial-console"></a>Console série

#### <a name="step-1-open-cmd-instance-in-serial-console"></a>Étape 1 : ouvrir l’instance CMD dans la console série

1. Accédez à la [Console série](serial-console-windows.md) en sélectionnant **Support & Troubleshooting (Support et dépannage)**  > **Console série (préversion)** . Si la fonctionnalité est activée sur la machine virtuelle, vous pouvez connecter la machine.

2. Créez un canal pour une instance CMD. Tapez **CMD** pour démarrer le canal et en obtenir le nom.

3. Basculez vers le canal qui exécute l’instance CMD, en l’occurrence le canal 1.

   ```
   ch -si 1
   ```

#### <a name="step-2-check-the-values-of-rdp-registry-keys"></a>Étape 2 : vérifier les valeurs des clés de Registre RDP :

1. Vérifiez si le protocole RDP est désactivé par des stratégies de groupe.

    ```
    REM Get the group policy setting
    reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fDenyTSConnections
    ```
    Si la stratégie de groupe stipule que RDP est désactivé (la valeur fDenyTSConnections est 0x1), exécutez la commande suivante pour activer le service TermService. Si la clé de Registre est introuvable, aucune stratégie de groupe n’est configurée pour désactiver RDP. Vous pouvez passer à l’étape suivante.

    ```
    REM update the fDenyTSConnections value to enable TermService service
    reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fDenyTSConnections /t REG_DWORD /d 0 /f
    ```
    > [!NOTE]
    > Cette étape active temporairement le service TermService. Ce changement sera réinitialisé au moment de l’actualisation des paramètres de stratégie de groupe. Pour résoudre le problème, vous devez vérifier si le service TermService est désactivé par la stratégie de groupe locale ou la stratégie de groupe de domaine, puis mettre à jour les paramètres de stratégie en conséquence.
    
2. Vérifiez la configuration de la connexion à distance actuelle.
    ```
    REM Get the local remote connection setting
    reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections
    ```
    Si la commande retourne 0x1, la machine virtuelle n’autorise pas la connexion à distance. Ensuite, autorisez la connexion à distance à l’aide de la commande suivante :
     ```
     reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
     ```
    
1. Vérifiez la configuration actuelle du serveur Bureau à distance.

    ```
    REM Get the local remote connection setting
    reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v TSEnabled
    ```

      Si la commande retourne 0, le serveur Bureau à distance est désactivé. Activez alors le serveur Bureau à distance comme suit :

      ```
      reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v TSEnabled /t REG_DWORD /d 1 /f
      ```

3. Le module Serveur Bureau à distance est défini en mode de drainage si le serveur se trouve dans une batterie de serveurs Bureau à distance (RDS ou Citrix). Vérifiez le mode actuel du module Serveur Bureau à distance.

      ```
      reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v TSServerDrainMode
      ```

      Si la commande retourne 1, le module Serveur Bureau à distance est défini en mode de drainage. Définissez alors le module en mode de travail comme suit :

      ```
      reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v TSServerDrainMode /t REG_DWORD /d 0 /f
      ```

4. Vérifiez si vous pouvez vous connecter au serveur Bureau à distance.

      ```
      reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v TSUserEnabled
      ```

      Si la commande retourne 1, vous ne pouvez pas vous connecter au serveur Bureau à distance. Activez alors la connexion comme suit :

      ```
      reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v TSUserEnabled /t REG_DWORD /d 0 /f
      ```
5. Vérifiez la configuration actuelle de l’écouteur RDP.

      ```
      reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp" /v fEnableWinStation
      ```

      Si la commande retourne 0, l’écouteur RDP est désactivé. Activez alors l’écouteur comme suit :

      ```
      reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp" /v fEnableWinStation /t REG_DWORD /d 1 /f
      ```

6. Vérifiez si vous pouvez vous connecter à l’écouteur RDP.

      ```
      reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp" /v fLogonDisabled
      ```

   Si la commande retourne 1, vous ne pouvez pas vous connecter à l’écouteur RDP. Activez alors la connexion comme suit :

      ```
      reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp" /v fLogonDisabled /t REG_DWORD /d 0 /f
      ```

7. Redémarrez la machine virtuelle.

8. Quittez l’instance CMD en tapant `exit`, puis appuyez deux fois sur **Entrée**.

9. Redémarrez la machine virtuelle en tapant `restart`, puis connectez-vous à la machine virtuelle.

Si le problème persiste, passez à l’étape 2.

#### <a name="step-2-enable-remote-desktop-services"></a>Étape 2 : activer les Services Bureau à distance

Pour plus d’informations, consultez [Les Services Bureau à distance ne démarrent pas sur une machine virtuelle Azure](troubleshoot-remote-desktop-services-issues.md).

#### <a name="step-3-reset-rdp-listener"></a>Étape 3 : réinitialiser l’écouteur RDP

Pour plus d'informations, consultez [Le Bureau à distance se déconnecte régulièrement sur une machine virtuelle Azure](troubleshoot-rdp-intermittent-connectivity.md).

### <a name="offline-repair"></a>Réparation en mode hors connexion

#### <a name="step-1-turn-on-remote-desktop"></a>Étape 1 : activer le Bureau à distance

1. [Attachez le disque du système d’exploitation à une machine virtuelle de récupération](./troubleshoot-recovery-disks-portal-windows.md).
2. Établissez une connexion Bureau à distance avec la machine virtuelle de récupération.
3. Vérifiez que le disque est marqué comme étant **En ligne** dans la console Gestion des disques. Notez la lettre de lecteur qui est affectée au disque du système d’exploitation attaché.
4. Établissez une connexion Bureau à distance avec la machine virtuelle de récupération.
5. Ouvrez une session Invite de commande avec élévation de privilèges (**Exécuter en tant qu’administrateur**). Exécutez les scripts suivants. Dans ce script, nous partons du principe que la lettre de lecteur affectée au disque de système d’exploitation attaché est F. Remplacez-la par la valeur correspondant à votre machine virtuelle.

      ```
      reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv 
      reg load HKLM\BROKENSOFTWARE F:\windows\system32\config\SOFTWARE.hiv 
 
      REM Ensure that Terminal Server is enabled 

      reg add "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server" /v TSEnabled /t REG_DWORD /d 1 /f 
      reg add "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server" /v TSEnabled /t REG_DWORD /d 1 /f 

      REM Ensure Terminal Service is not set to Drain mode 
      reg add "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server" /v TSServerDrainMode /t REG_DWORD /d 0 /f 
      reg add "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server" /v TSServerDrainMode /t REG_DWORD /d 0 /f 

      REM Ensure Terminal Service has logon enabled 
      reg add "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server" /v TSUserEnabled /t REG_DWORD /d 0 /f 
      reg add "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server" /v TSUserEnabled /t REG_DWORD /d 0 /f 

      REM Ensure the RDP Listener is not disabled 
      reg add "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v fEnableWinStation /t REG_DWORD /d 1 /f 
      reg add "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v fEnableWinStation /t REG_DWORD /d 1 /f 

      REM Ensure the RDP Listener accepts logons 
      reg add "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server\Winstations\RDP-Tcp" /v fLogonDisabled /t REG_DWORD /d 0 /f 
      reg add "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server\Winstations\RDP-Tcp" /v fLogonDisabled /t REG_DWORD /d 0 /f 

      REM RDP component is enabled 
      reg add "HKLM\BROKENSYSTEM\ControlSet001\control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f 
      reg add "HKLM\BROKENSYSTEM\ControlSet002\control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f 
      reg add "HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fDenyTSConnections /t REG_DWORD /d 0 /f 

      reg unload HKLM\BROKENSYSTEM 
      reg unload HKLM\BROKENSOFTWARE 
      ```

6. Si la machine virtuelle est jointe à un domaine, vérifiez dans la clé de Registre suivante si une stratégie de groupe est appelée à désactiver RDP. 

      ```
      HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows NT\Terminal Services\fDenyTSConnectionS
      ```

      Si cette clé a la valeur 1, cela signifie que RDP est désactivé par la stratégie. Pour activer le Bureau à distance via la stratégie GPO, modifiez la stratégie suivante à partir du contrôleur de domaine :

   
      **Configuration ordinateur\Stratégies\Modèles d'administration :**

      Définitions de stratégies\Composants Windows\Remote Desktop Services\Remote Desktop Session Host\Connections\Autoriser les utilisateurs à se connecter à distance avec les services Bureau à distance
  
1. Détachez le disque de la machine virtuelle de secours.
1. [Créez une machine virtuelle à partir du disque](../windows/create-vm-specialized.md).

Si le problème persiste, passez à l’étape 2.

#### <a name="step-2-enable-remote-desktop-services"></a>Étape 2 : activer les Services Bureau à distance

Pour plus d’informations, consultez [Les Services Bureau à distance ne démarrent pas sur une machine virtuelle Azure](troubleshoot-remote-desktop-services-issues.md).

#### <a name="step-3-reset-rdp-listener"></a>Étape 3 : réinitialiser l’écouteur RDP

Pour plus d'informations, consultez [Le Bureau à distance se déconnecte régulièrement sur une machine virtuelle Azure](troubleshoot-rdp-intermittent-connectivity.md).

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contacter le support technique

Si vous avez besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir une prise en charge rapide de votre problème.
