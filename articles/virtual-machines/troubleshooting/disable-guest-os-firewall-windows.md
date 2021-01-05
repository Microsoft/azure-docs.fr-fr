---
title: Désactiver le pare-feu du système d’exploitation invité dans une machine virtuelle Azure | Microsoft Docs
description: Découvrez une méthode de contournement pour résoudre les situations dans lesquelles un pare-feu sur le système d’exploitation invité filtre le trafic partiel ou complet vers une machine virtuelle.
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
ms.openlocfilehash: c0426c5359e4d82d0316613586b9298596d82605
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87009762"
---
# <a name="disable-the-guest-os-firewall-in-azure-vm"></a>Désactiver le pare-feu du système d’exploitation invité dans une machine virtuelle Azure

Cet article fournit une référence pour les situations dans lesquelles vous pensez que le pare-feu du système d’exploitation invité filtre le trafic partiel ou complet vers une machine virtuelle. Cela peut se produire si des modifications ont été délibérément apportées au pare-feu et ont provoqué l’échec des connexions RDP.

## <a name="solution"></a>Solution

Le processus qui est décrit dans cet article est destiné à être utilisé comme solution de contournement afin que vous puissiez vous concentrer sur la résolution du problème réel, c’est-à-dire comment correctement configurer les règles de pare-feu. L’une des bonnes pratiques de Microsoft est que le composant Pare-feu Windows soit activé. La façon dont vous configurez les règles de pare-feu dépend du niveau d’accès à la machine virtuelle requis.

### <a name="online-solutions"></a>Solutions en ligne 

Si la machine virtuelle est en ligne et accessible sur une autre machine virtuelle sur le même réseau virtuel, vous pouvez atténuer les risques à l’aide de l’autre machine virtuelle.

#### <a name="mitigation-1-custom-script-extension-or-run-command-feature"></a>Atténuation 1 : fonctionnalité Extension de script personnalisée ou Exécuter une commande

Si vous avez un agent Azure fonctionnel, vous pouvez utiliser la fonctionnalité [Extension de script personnalisée](../extensions/custom-script-windows.md) ou [Exécuter des commandes](../windows/run-command.md) (machines virtuelles Resource Manager uniquement) pour exécuter à distance les scripts suivants.

> [!Note]
> * Si le pare-feu est défini localement, exécutez le script suivant :
>   ```
>   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\DomainProfile' -name "EnableFirewall" -Value 0
>   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\PublicProfile' -name "EnableFirewall" -Value 0
>   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\Standardprofile' -name "EnableFirewall" -Value 0 
>   Restart-Service -Name mpssvc
>   ```
> * Si le pare-feu est défini via une stratégie Active Directory, vous pouvez exécuter le script suivant pour un accès temporaire. 
>   ```
>   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\WindowsFirewall\DomainProfile' -name "EnableFirewall" -Value 0
>   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\WindowsFirewall\PublicProfile' -name "EnableFirewall" -Value 0
>   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\WindowsFirewall\StandardProfile' -name "EnableFirewall" -Value 0
>   Restart-Service -Name mpssvc
>   ```
>   Toutefois, dès que la stratégie est appliquée à nouveau, vous serez exclu de la session à distance. Le correctif permanent pour ce problème consiste à modifier la stratégie qui est appliquée sur cet ordinateur.

#### <a name="mitigation-2-remote-powershell"></a>Atténuation 2 : PowerShell à distance

1.  Connectez-vous à une machine virtuelle qui se trouve sur le même réseau virtuel que la machine virtuelle que vous ne pouvez pas atteindre à l’aide de la connexion RDP.

2.  Ouvrez une fenêtre de console PowerShell.

3.  Exécutez les commandes suivantes :

    ```powershell
    Enter-PSSession (New-PSSession -ComputerName "<HOSTNAME>" -Credential (Get-Credential) -SessionOption (New-PSSessionOption -SkipCACheck -SkipCNCheck)) 
    netsh advfirewall set allprofiles state off
    Restart-Service -Name mpssvc 
    exit
    ```

> [!Note]
> Si le pare-feu est défini via un objet stratégie de groupe, cette méthode peut ne pas fonctionner, car cette commande modifie uniquement les entrées de Registre local. Si une stratégie est en place, elle remplace cette modification. 

#### <a name="mitigation-3-pstools-commands"></a>Atténuation 3 : Commandes PSTools

1.  Sur la machine virtuelle de dépannage, téléchargez [PSTools](/sysinternals/downloads/pstools).

2.  Ouvrez une instance CMD, puis accédez à la machine virtuelle via son adresse IP privée.

3.  Exécutez les commandes suivantes :

    ```cmd
    psexec \\<DIP> -u <username> cmd
    netsh advfirewall set allprofiles state off
    psservice restart mpssvc
    ```

#### <a name="mitigation-4-remote-registry"></a>Atténuation 4 : Registre distant 

Procédez comme suit pour utiliser le [Registre à distance](https://support.microsoft.com/help/314837/how-to-manage-remote-access-to-the-registry).

1.  Sur la machine virtuelle de dépannage, démarrez l’éditeur de Registre et accédez à **Fichier** > **Connexion au Registre réseau**.

2.  Ouvrez la branche *MACHINE CIBLE*\SYSTEM et spécifiez les valeurs suivantes :

    ```
    <TARGET MACHINE>\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\DomainProfile\EnableFirewall           -->        0 
    <TARGET MACHINE>\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\PublicProfile\EnableFirewall           -->        0 
    <TARGET MACHINE>\SYSTEM\CurrentControlSet\services\SharedAccess\Parameters\FirewallPolicy\StandardProfile\EnableFirewall         -->        0
    ```

3.  Redémarrez le service. Étant donné que vous ne pouvez pas faire cela en utilisant le Registre à distance, vous devez utiliser Supprimer la console de service.

4.  Ouvrez une instance de **Services.msc**.

5.  Cliquez sur **Services (Local)** .

6.  Sélectionnez **Se connecter à un autre ordinateur**.

7.  Entrez l’**adresse IP privée (DIP)** de la machine virtuelle posant problème.

8.  Redémarrez la stratégie de pare-feu local.

9.  Essayez de vous reconnecter à la machine virtuelle via RDP à partir de votre ordinateur local.

### <a name="offline-solutions"></a>Solutions en mode hors connexion 

Si vous rencontrez une situation dans laquelle vous ne pouvez pas atteindre la machine virtuelle par n’importe quelle méthode, Extension de script personnalisée échoue, et vous devrez travailler en mode HORS CONNEXION directement via le disque système. Pour ce faire, procédez comme suit :

1.  [Attachez le disque système à une machine virtuelle de récupération](troubleshoot-recovery-disks-portal-windows.md).

2.  Établissez une connexion Bureau à distance avec la machine virtuelle de récupération.

3.  Vérifiez que le disque est marqué comme étant En ligne dans la console Gestion des disques. Notez la lettre de lecteur affectée au disque système attaché.

4.  Avant d’effectuer des changements, créez une copie du dossier \windows\system32\config au cas où vous souhaiteriez annuler les changements.

5.  Sur la machine virtuelle de dépannage, démarrez l’éditeur de Registre (regedit.exe). 

6.  Pour cette procédure de résolution des problèmes, nous montons les hives en tant que BROKENSYSTEM et BROKENSOFTWARE.

7.  Mettez en surbrillance la clé HKEY_LOCAL_MACHINE, puis sélectionnez Fichier > Charger Hive à partir du menu.

8.  Recherchez le fichier \windows\system32\config\SYSTEM sur le disque système attaché.

9.  Ouvrez une instance PowerShell avec élévation de privilèges et exécutez les commandes suivantes :

    ```cmd
    # Load the hives - If your attached disk is not F, replace the letter assignment here
    reg load HKLM\BROKENSYSTEM f:\windows\system32\config\SYSTEM
    reg load HKLM\BROKENSOFTWARE f:\windows\system32\config\SOFTWARE 
    # Disable the firewall on the local policy
    $ControlSet = (get-ItemProperty -Path 'HKLM:\BROKENSYSTEM\Select' -name "Current").Current
    $key = 'BROKENSYSTEM\ControlSet00'+$ControlSet+'\services\SharedAccess\Parameters\FirewallPolicy\DomainProfile'
    Set-ItemProperty -Path $key -name 'EnableFirewall' -Value 0 -Type Dword -force
    $key = 'BROKENSYSTEM\ControlSet00'+$ControlSet+'\services\SharedAccess\Parameters\FirewallPolicy\PublicProfile'
    Set-ItemProperty -Path $key -name 'EnableFirewall' -Value 0 -Type Dword -force
    $key = 'BROKENSYSTEM\ControlSet00'+$ControlSet+'\services\SharedAccess\Parameters\FirewallPolicy\StandardProfile'
    Set-ItemProperty -Path $key -name 'EnableFirewall' -Value 0 -Type Dword -force
    # To ensure the firewall is not set through AD policy, check if the following registry entries exist and if they do, then check if the following entries exist:
    $key = 'HKLM:\BROKENSOFTWARE\Policies\Microsoft\WindowsFirewall\DomainProfile'
    Set-ItemProperty -Path $key -name 'EnableFirewall' -Value 0 -Type Dword -force
    $key = 'HKLM:\BROKENSOFTWARE\Policies\Microsoft\WindowsFirewall\PublicProfile'
    Set-ItemProperty -Path $key -name 'EnableFirewall' -Value 0 -Type Dword -force
    $key = 'HKLM:\BROKENSOFTWARE\Policies\Microsoft\WindowsFirewall\StandardProfile'
    Set-ItemProperty -Path $key -name 'EnableFirewall' -Value 0 -Type Dword -force
    # Unload the hives
    reg unload HKLM\BROKENSYSTEM
    reg unload HKLM\BROKENSOFTWARE
    ```

10. [Détachez le disque système et recréez la machine virtuelle](troubleshoot-recovery-disks-portal-windows.md).

11. Vérifiez si le problème est résolu.
