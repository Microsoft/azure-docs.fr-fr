---
title: Le pare-feu du système d’exploitation invité des machines virtuelles Azure bloque le trafic entrant | Microsoft Docs
description: Découvrez comment résoudre le problème de connexion RDP (Remote Desktop Protocol) qui se produit si le pare-feu du système d’exploitation invité bloque le trafic entrant.
services: virtual-machines-windows
documentationcenter: ''
author: Deland-Han
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: azurecli
ms.date: 11/22/2018
ms.author: delhan
ms.openlocfilehash: 1b80fc997a4b3d2b472717b1ec2f379a4e958d8c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80422553"
---
# <a name="azure-vm-guest-os-firewall-is-blocking-inbound-traffic"></a>Le pare-feu du système d’exploitation invité des machines virtuelles Azure bloque le trafic entrant

Cet article explique comment résoudre le problème RDP (Remote Desktop Protocol) qui se produit si le pare-feu du système d’exploitation invité bloque le trafic entrant.

## <a name="symptoms"></a>Symptômes

Vous ne pouvez pas utiliser une connexion RDP pour vous connecter à une machine virtuelle Azure. Dans Diagnostics de démarrage -> Capture d’écran, il est indiqué que le système d’exploitation est entièrement chargé sur l’écran d’accueil (Ctrl + Alt + Suppr).

## <a name="cause"></a>Cause

### <a name="cause-1"></a>Cause 1

La règle RDP n’est pas configurée pour autoriser le trafic RDP.

### <a name="cause-2"></a>Cause 2

Les profils de pare-feu du système invité sont configurés pour bloquer toutes les connexions entrantes, y compris le trafic RDP.

![Paramètre de pare-feu](./media/guest-os-firewall-blocking-inbound-traffic/firewall-advanced-setting.png)

## <a name="solution"></a>Solution

Avant de suivre ces étapes, prenez un instantané du disque système de la machine virtuelle affectée comme sauvegarde. Pour plus d’informations, consultez  [Créer un instantané](../windows/snapshot-copy-managed-disk.md).

Pour résoudre ce problème, utilisez l’une des méthodes dans [Utiliser des outils à distance pour résoudre les problèmes de machine virtuelle Azure](remote-tools-troubleshoot-azure-vm-issues.md) pour se connecter à distance à la machine virtuelle, puis modifiez les règles de pare-feu du système d’exploitation invité pour **Autoriser** le trafic RDP.

### <a name="online-troubleshooting"></a>Résolution des problèmes en ligne

Connectez-vous à la [console série et ouvrez une instance PowerShell](serial-console-windows.md#use-cmd-or-powershell-in-serial-console). Si la console série n’est pas activée sur la machine virtuelle, accédez à [Réparer la machine virtuelle en mode hors connexion](troubleshoot-rdp-internal-error.md#repair-the-vm-offline).

#### <a name="mitigation-1"></a>Atténuation 1

1.  Si Azure Agent est installé et fonctionne correctement sur la machine virtuelle, vous pouvez utiliser l’option « Réinitialiser seulement la configuration » sous **Support + dépannage** > **Réinitialiser le mot de passe** dans le menu de la machine virtuelle.

2.  L’exécution de cette option de récupération :

    *   Active un composant RDP s’il est désactivé.

    *   Active tous les profils de pare-feu Windows.

    *   Vérifiez que la règle RDP est activée dans le Pare-feu Windows.

    *   Si les étapes précédentes ne fonctionnent pas, réinitialisez manuellement la règle de pare-feu. Pour ce faire, interrogez toutes les règles qui contiennent le nom « Bureau à distance » en exécutant la commande suivante :

        ```cmd
        netsh advfirewall firewall show rule dir=in name=all | select-string -pattern "(Name.*Remote Desktop)" -context 9,4 | more
        ```

        Si le port RDP a été défini sur un port autre que le port 3389, vous devez trouver une règle personnalisée qui peut avoir été créée et définie sur ce port. Pour rechercher toutes les règles de trafic entrant qui ont un port personnalisé, exécutez la commande suivante :

        ```cmd
        netsh advfirewall firewall show rule dir=in name=all | select-string -pattern "(LocalPort.*<CUSTOM PORT>)" -context 9,4 | more
        ```

3.  Si vous voyez que la règle est désactivée, activez-la. Pour ouvrir un groupe entier, comme le groupe Bureau à distance intégré, exécutez la commande suivante :

    ```cmd
    netsh advfirewall firewall set rule group="Remote Desktop" new enable=yes
    ```

    Sinon, pour ouvrir la règle Bureau à distance (TCP-In) spécifique, exécutez la commande suivante :

    ```cmd
    netsh advfirewall firewall set rule name="<CUSTOM RULE NAME>" new enable=yes
    ```

4.  Pour le dépannage, vous pouvez définir les profils de pare-feu sur DÉSACTIVÉ :

    ```cmd
    netsh advfirewall set allprofiles state off
    ```

    Une fois que vous avez fini correctement le dépannage et la configuration du pare-feu, réactivez le pare-feu.

    > [!Note]
    > Vous n’êtes pas obligé de redémarrer la machine virtuelle pour appliquer ces changements.

5.  Essayez d’établir une connexion RDP pour accéder à la machine virtuelle.

#### <a name="mitigation-2"></a>Atténuation 2

1.  Interrogez les profils de pare-feu pour déterminer si la stratégie de pare-feu de trafic entrant est définie sur *BlockInboundAlways* :

    ```cmd
    netsh advfirewall show allprofiles | more
    ```

    ![Allprofiles](./media/guest-os-firewall-blocking-inbound-traffic/firewall-profiles.png)

    > [!Note]
    > Les instructions suivantes s’appliquent à la stratégie de pare-feu, en fonction de la façon dont elle est configurée :
    >    * *BlockInbound* : l’ensemble du trafic entrant est bloqué, sauf si vous avez une règle effective pour autoriser ce trafic.
    >    * *BlockInboundAlways* : toutes les règles de pare-feu sont ignorées et l’ensemble du trafic est bloqué.

2.  Modifiez l’action *DefaultInboundAction* de sorte que ces profils **autorisent** le trafic. Pour ce faire, exécutez la commande suivante :

    ```cmd
    netsh advfirewall set allprofiles firewallpolicy allowinbound,allowoutbound
    ```

3.  Réinterrogez les profils pour vérifier que votre modification a bien été effectuée. Pour ce faire, exécutez la commande suivante :

    ```cmd
    netsh advfirewall show allprofiles | more
    ```

    > [!Note]
    > Vous n’êtes pas obligé de redémarrer la machine virtuelle pour appliquer les changements.

4.  Réessayez d’accéder à votre machine virtuelle via RDP.

### <a name="offline-mitigations"></a>Atténuations hors connexion

1.  [Attachez le disque système à une machine virtuelle de récupération](troubleshoot-recovery-disks-portal-windows.md).

2.  Établissez une connexion Bureau à distance avec la machine virtuelle de récupération.

3.  Vérifiez que le disque est marqué comme étant **En ligne** dans la console Gestion des disques. Notez la lettre de lecteur affectée au disque système attaché.

#### <a name="mitigation-1"></a>Atténuation 1

Consultez [Activer ou désactiver une règle de pare-feu sur un système d’exploitation invité](enable-disable-firewall-rule-guest-os.md).

#### <a name="mitigation-2"></a>Atténuation 2

1.  [Attachez le disque système à une machine virtuelle de récupération](troubleshoot-recovery-disks-portal-windows.md).

2.  Établissez une connexion Bureau à distance avec la machine virtuelle de récupération.

3.  Une fois le disque système attaché à la machine virtuelle de récupération, vérifiez qu’il est marqué **En ligne** dans la console Gestion des disques. Notez la lettre de lecteur qui est affectée au disque du système d’exploitation attaché.

4.  Ouvrez une instance CMD avec élévation de privilèges et exécutez le script suivant :

    ```cmd
    REM Backup the registry prior doing any change
    robocopy f:\windows\system32\config f:\windows\system32\config.BACK /MT

    REM Mount the hive
    reg load HKLM\BROKENSYSTEM f:\windows\system32\config\SYSTEM

    REM Delete the keys to block all inbound connection scenario
    REG DELETE "HKLM\BROKENSYSTEM\ControlSet001\services\SharedAccess\Parameters\FirewallPolicy\DomainProfile" /v DoNotAllowExceptions
    REG DELETE "HKLM\BROKENSYSTEM\ControlSet001\services\SharedAccess\Parameters\FirewallPolicy\PublicProfile" /v DoNotAllowExceptions
    REG DELETE "HKLM\BROKENSYSTEM\ControlSet001\services\SharedAccess\Parameters\FirewallPolicy\StandardProfile" /v DoNotAllowExceptions
    REG DELETE "HKLM\BROKENSYSTEM\ControlSet002\services\SharedAccess\Parameters\FirewallPolicy\DomainProfile" /v DoNotAllowExceptions
    REG DELETE "HKLM\BROKENSYSTEM\ControlSet002\services\SharedAccess\Parameters\FirewallPolicy\PublicProfile" /v DoNotAllowExceptions
    REG DELETE "HKLM\BROKENSYSTEM\ControlSet002\services\SharedAccess\Parameters\FirewallPolicy\StandardProfile" /v DoNotAllowExceptions

    REM Unmount the hive
    reg unload HKLM\BROKENSYSTEM
    ```

5.  [Détachez le disque système et recréez la machine virtuelle](troubleshoot-recovery-disks-portal-windows.md).

6.  Vérifiez si le problème est résolu.
