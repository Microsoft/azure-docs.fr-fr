---
title: Activer ou désactiver une règle de pare-feu sur un système d’exploitation invité d’une machine virtuelle Azure | Microsoft Docs
description: Apprenez à utiliser des paramètres de registre ou des outils de contrôle à distance en ligne ou hors connexion pour activer ou désactiver les règles de pare-feu du système d'exploitation invité sur une machine virtuelle Azure distante.
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
ms.openlocfilehash: 17616a223292ec07186b0a3fba264400423977ac
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87058768"
---
# <a name="enable-or-disable-a-firewall-rule-on-an-azure-vm-guest-os"></a>Activer ou désactiver une règle de pare-feu sur un système d’exploitation invité d’une machine virtuelle Azure

Cet article fournit une référence pour résoudre une situation dans laquelle vous pensez que le pare-feu du système d’exploitation invité filtre le trafic partiel sur une machine virtuelle. Cela peut être utile pour les raisons suivantes :

*   Si une modification apportée volontairement au pare-feu a provoqué l’échec des connexions RDP, la fonctionnalité Extension de script personnalisé peut résoudre ce problème.

*   La désactivation de tous les profils de pare-feu est une solution plus efficace que la définition de la règle de pare-feu spécifiques à RDP.

## <a name="solution"></a>Solution

La façon dont vous configurez les règles de pare-feu dépend du niveau d’accès à la machine virtuelle requis. Les exemples suivants utilisent les règles RDP. Toutefois, les mêmes méthodes sont applicables à n’importe quel autre type de trafic en pointant vers la bonne clé de registre.

### <a name="online-troubleshooting"></a>Résolution des problèmes en ligne 

#### <a name="mitigation-1-custom-script-extension"></a>Atténuation 1 : Extension de script personnalisé

1.  Créez votre script en utilisant l’exemple suivant.

    *   Pour activer une règle :
        ```cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=yes
        ```

    *   Pour désactiver une règle :
        ```cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=no
        ```

2.  Chargez ce script dans le portail Azure en utilisant la fonctionnalité [Extension de script personnalisé](../extensions/custom-script-windows.md). 

#### <a name="mitigation-2-remote-powershell"></a>Atténuation 2 : PowerShell à distance

Si la machine virtuelle est en ligne et accessible sur une autre machine virtuelle sur le même réseau virtuel, vous pouvez atténuer les risques suivants à l’aide de l’autre machine virtuelle.

1.  Sur la machine virtuelle de dépannage, ouvrez une fenêtre de console PowerShell.

2.  Exécutez les commandes suivantes, si nécessaire.

    *   Pour activer une règle :
        ```powershell
        Enter-PSSession (New-PSSession -ComputerName "<HOSTNAME>" -Credential (Get-Credential) -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck)) 
        Enable-NetFirewallRule -DisplayName  "RemoteDesktop-UserMode-In-TCP"
        exit
        ```

    *   Pour désactiver une règle :
        ```powershell
        Enter-PSSession (New-PSSession -ComputerName "<HOSTNAME>" -Credential (Get-Credential) -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck)) 
        Disable-NetFirewallRule -DisplayName  "RemoteDesktop-UserMode-In-TCP"
        exit
        ```

#### <a name="mitigation-3-pstools-commands"></a>Atténuation 3 : Commandes PSTools

Si la machine virtuelle est en ligne et accessible sur une autre machine virtuelle sur le même réseau virtuel, vous pouvez atténuer les risques suivants à l’aide de l’autre machine virtuelle.

1.  Sur la machine virtuelle de dépannage, téléchargez [PSTools](/sysinternals/downloads/pstools).

2.  Ouvrez une instance CMD, puis accédez à la machine virtuelle via son adresse IP interne (DIP). 

    * Pour activer une règle :
        ```cmd
        psexec \\<DIP> -u <username> cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=yes
        ```

    *   Pour désactiver une règle :
        ```cmd
        psexec \\<DIP> -u <username> cmd
        netsh advfirewall firewall set rule dir=in name="Remote Desktop - User Mode (TCP-In)" new enable=no
        ```

#### <a name="mitigation-4-remote-registry"></a>Atténuation 4 : Registre distant

Si la machine virtuelle est en ligne et accessible sur une autre machine virtuelle sur le même réseau virtuel, vous pouvez utiliser [Registre à distance](https://support.microsoft.com/help/314837/how-to-manage-remote-access-to-the-registry) sur l’autre machine virtuelle.

1.  Sur la machine virtuelle de dépannage, démarrez l’éditeur de Registre (regedit.exe), puis sélectionnez **Fichier** > **Connexion au Registre réseau**.

2.  Ouvrez la branche *MACHINE CIBLE*\SYSTEM, puis spécifiez les valeurs suivantes :

    * Pour activer une règle, ouvrez la valeur de Registre suivante :
    
        *TARGET MACHINE*\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP
    
        Ensuite, remplacez **Active=FALSE** par **Active=TRUE** dans la chaîne :

        `v2.22|Action=Allow|Active=TRUE|Dir=In|Protocol=6|Profile=Domain|Profile=Private|Profile=Public|LPort=3389|App=%SystemRoot%\system32\svchost.exe|Svc=termservice|Name=\@FirewallAPI.dll,-28775|Desc=\@FirewallAPI.dll,-28756|EmbedCtxt=\@FirewallAPI.dll,-28752|`
    
    * Pour désactiver une règle, ouvrez la valeur de Registre suivante :
    
        *TARGET MACHINE*\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP

        Puis remplacez **Active=TRUE** par **Active=FALSE**:
        
        `v2.22|Action=Allow|Active=FALSE|Dir=In|Protocol=6|Profile=Domain|Profile=Private|Profile=Public|LPort=3389|App=%SystemRoot%\system32\svchost.exe|Svc=termservice|Name=\@FirewallAPI.dll,-28775|Desc=\@FirewallAPI.dll,-28756|EmbedCtxt=\@FirewallAPI.dll,-28752|`

3.  Redémarrez la machine virtuelle pour appliquer les changements.

### <a name="offline-troubleshooting"></a>Résolution des problèmes hors connexion 

Si vous ne pouvez pas accéder à la machine virtuelle par n’importe quelle méthode, Extension de script personnalisée échoue, et vous devrez travailler en mode HORS CONNEXION directement via le disque système.

Avant de suivre ces étapes, prenez un instantané du disque système de la machine virtuelle affectée comme sauvegarde. Pour plus d’informations, consultez [Créer un instantané](../windows/snapshot-copy-managed-disk.md).

1.  [Attachez le disque système à une machine virtuelle de récupération](troubleshoot-recovery-disks-portal-windows.md).

2.  Établissez une connexion Bureau à distance avec la machine virtuelle de récupération.

3.  Vérifiez que le disque est marqué comme étant **En ligne** dans la console Gestion des disques. Notez la lettre de lecteur affectée au disque système attaché.

4.  Avant d’effectuer des changements, créez une copie du dossier \windows\system32\config au cas où vous souhaiteriez annuler les changements.

5.  Sur la machine virtuelle de dépannage, démarrez l’éditeur de Registre (regedit.exe).

6.  Mettez en surbrillance la clé **HKEY_LOCAL_MACHINE**, puis sélectionnez **Fichier** > **Charger Hive** dans le menu.

    ![Regedit](./media/enable-or-disable-firewall-rule-guest-os/load-registry-hive.png)

7.  Recherchez et ouvrez le fichier \windows\system32\config\SYSTEM. 

    > [!Note]
    > Vous êtes invité à saisir un nom. Entrez **BROKENSYSTEM**, puis développez **HKEY_LOCAL_MACHINE**. Vous voyez maintenant une clé supplémentaire nommée**BROKENSYSTEM**. Pour cette procédure de dépannage, nous montons ces ruches défectueuses en tant que **BROKENSYSTEM**.

8.  Sur la branche BROKENSYSTEM, apportez les modifications suivantes :

    1.  Vérifiez à partir de quelle clé de Registre **ControlSet** la machine virtuelle démarre. Son numéro de clé s’affiche dans HKLM\BROKENSYSTEM\Select\Current.

    2.  Pour activer une règle, ouvrez la valeur de Registre suivante :
    
        HKLM\BROKENSYSTEM\ControlSet00X\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP
        
        Puis remplacez **Active=FALSE** par **Active=True**.
        
        `v2.22|Action=Allow|Active=TRUE|Dir=In|Protocol=6|Profile=Domain|Profile=Private|Profile=Public|LPort=3389|App=%SystemRoot%\system32\svchost.exe|Svc=termservice|Name=\@FirewallAPI.dll,-28775|Desc=\@FirewallAPI.dll,-28756|EmbedCtxt=\@FirewallAPI.dll,-28752|`

    3.  Pour désactiver une règle, ouvrez la clé de Registre suivante :

        HKLM\BROKENSYSTEM\ControlSet00X\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules\RemoteDesktop-UserMode-In-TCP

        Puis remplacez **Active=True** par **Active=FALSE**.
        
        `v2.22|Action=Allow|Active=FALSE|Dir=In|Protocol=6|Profile=Domain|Profile=Private|Profile=Public|LPort=3389|App=%SystemRoot%\system32\svchost.exe|Svc=termservice|Name=\@FirewallAPI.dll,-28775|Desc=\@FirewallAPI.dll,-28756|EmbedCtxt=\@FirewallAPI.dll,-28752|`

9.  Mettez la clé **BROKENSYSTEM** en surbrillance, puis sélectionnez **Fichier** > **Charger la ruche** dans le menu.

10. [Détachez le disque système et recréez la machine virtuelle](troubleshoot-recovery-disks-portal-windows.md).

11. Vérifiez si le problème est résolu.
