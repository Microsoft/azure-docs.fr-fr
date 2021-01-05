---
title: Préparer un disque dur virtuel Windows à charger sur Azure
description: Découvrez comment préparer un disque dur virtuel ou VHDX Windows pour le charger sur Azure
author: glimoli
manager: dcscontentpm
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: troubleshooting
ms.date: 09/02/2020
ms.author: genli
ms.openlocfilehash: 390cda604b71404735b7c14382d30067e154ef70
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91976180"
---
# <a name="prepare-a-windows-vhd-or-vhdx-to-upload-to-azure"></a>Préparer un disque dur virtuel Windows à charger sur Azure

Avant de charger une machine virtuelle Windows locale sur Azure, vous devez préparer le disque dur virtuel (VHD ou VHDX). Azure prend en charge les machines virtuelles des générations 1 et 2 qui sont au format de fichier VHD et ont un disque de taille fixe. La taille maximale autorisée pour le disque dur virtuel du système d’exploitation sur une machine virtuelle de première génération est de 2 To.

Vous pouvez convertir un fichier VHDX en VHD, convertir un disque à expansion dynamique en disque de taille fixe, mais vous ne pouvez pas modifier la génération d’une machine virtuelle. Pour plus d’informations, consultez les articles [Dois-je créer une machine virtuelle de génération 1 ou 2 dans Hyper-V ?](/windows-server/virtualization/hyper-v/plan/Should-I-create-a-generation-1-or-2-virtual-machine-in-Hyper-V) et [Support pour les machines virtuelles de génération 2 sur Azure](../generation-2.md).

Pour plus d’informations sur la stratégie de prise en charge des machines virtuelles Azure, consultez la page [Prise en charge des logiciels serveur Microsoft pour les machines virtuelles Microsoft Azure](https://support.microsoft.com/help/2721672/).

> [!NOTE]
> Les instructions de cet article s’appliquent à :
>
> - la version 64 bits de Windows Server 2008 R2 et aux systèmes d’exploitation Windows Server ultérieurs. Pour plus d’informations sur l’exécution d’un système d'exploitation 32 bits dans Azure, consultez l’article [Prise en charge pour les systèmes d’exploitation 32 bits sur des machines virtuelles Azure](https://support.microsoft.com/help/4021388/).
> - Si un outil de récupération d’urgence comme Azure Site Recovery ou Azure Migrate est utilisé pour migrer la charge de travail, ce processus reste obligatoire sur le système d’exploitation invité pour préparer l’image avant la migration.

## <a name="system-file-checker"></a>Vérificateur des fichiers système

### <a name="run-windows-system-file-checker-utility-before-generalization-of-os-image"></a>Exécuter l’utilitaire Vérificateur des fichiers système Windows avant la généralisation de l’image du système d’exploitation

Le Vérificateur des fichiers système permet de vérifier et de remplacer des fichiers système Windows.

> [!IMPORTANT]
> Pour exécuter les exemples fournis dans cet article, utilisez une session PowerShell avec élévation de privilèges.

Exécutez la commande du Vérificateur des fichiers système :

```powershell
sfc.exe /scannow
```

```Output
Beginning system scan.  This process will take some time.

Beginning verification phase of system scan.
Verification 100% complete.

Windows Resource Protection did not find any integrity violations.
```

Une fois l’analyse du Vérificateur des fichiers système terminée, installez les mises à jour Windows, puis redémarrez l’ordinateur.

## <a name="set-windows-configurations-for-azure"></a>Définir les configurations Windows pour Azure

> [!NOTE]
> La plateforme Azure monte un fichier ISO sur le DVD-ROM lorsqu’une machine virtuelle Windows est créée à partir d’une image généralisée. C’est la raison pour laquelle le DVD-ROM doit être activé dans le système d’exploitation de l’image généralisée. S’il est désactivé, la machine virtuelle Windows est bloquée en mode OOBE (out-of-Box Experience).

1. Supprimez tout itinéraire persistant statique de la table de routage :

   - Pour afficher la table de routage, exécutez `route.exe print`.
   - Vérifiez la section **Itinéraires persistants**. S’il existe un itinéraire persistant, utilisez la commande `route.exe delete` pour le supprimer.

1. Supprimez le proxy WinHTTP :

   ```powershell
   netsh.exe winhttp reset proxy
   ```

    Si la machine virtuelle doit fonctionner avec un proxy spécifique, vous devez ajouter une exception de proxy à l’adresse IP Azure ([168.63.129.16](../../virtual-network/what-is-ip-address-168-63-129-16.md)) pour que la machine virtuelle puisse se connecter à Azure :

    ```
    $proxyAddress='<your proxy server>'
    $proxyBypassList='<your list of bypasses>;168.63.129.16'
    netsh.exe winhttp set proxy $proxyAddress $proxyBypassList
    ```

1. Ouvrez DiskPart :

   ```powershell
   diskpart.exe
   ```

   Définissez la stratégie SAN des disques sur [`Onlineall`](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/gg252636(v=ws.11)) :

   ```DiskPart
   DISKPART> san policy=onlineall
   DISKPART> exit
   ```

1. Définissez l’heure de temps universel coordonné (UTC) pour Windows. Définissez également le type de démarrage du service de temps Windows **w32time** sur **Automatique** :

   ```powershell
   Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Control\TimeZoneInformation -Name RealTimeIsUniversal -Value 1 -Type DWord -Force
   Set-Service -Name w32time -StartupType Automatic
   ```

1. Définissez le profil d’alimentation sur de hautes performances :

   ```powershell
   powercfg.exe /setactive SCHEME_MIN
   ```

1. Vérifiez que les variables d’environnement **TEMP** et **TMP** sont définies avec leurs valeurs par défaut :

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment' -Name TEMP -Value "%SystemRoot%\TEMP" -Type ExpandString -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment' -Name TMP -Value "%SystemRoot%\TEMP" -Type ExpandString -Force
   ```

## <a name="check-the-windows-services"></a>Vérifier les services Windows

Vérifiez que chacun des services Windows suivants est défini sur la valeur par défaut Windows. Ces services sont le minimum à configurer pour garantir la connectivité de la machine virtuelle. Pour définir les paramètres de démarrage, exécutez les commandes suivantes :

```powershell
Get-Service -Name BFE, Dhcp, Dnscache, IKEEXT, iphlpsvc, nsi, mpssvc, RemoteRegistry |
  Where-Object StartType -ne Automatic |
    Set-Service -StartupType Automatic

Get-Service -Name Netlogon, Netman, TermService |
  Where-Object StartType -ne Manual |
    Set-Service -StartupType Manual
```

## <a name="update-remote-desktop-registry-settings"></a>Mettre à jour les paramètres de Registre du Bureau à distance

Assurez-vous que les paramètres suivants sont configurés correctement pour un accès à distance :

> [!NOTE]
> Si vous recevez un message d’erreur lors de l’exécution de `Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services -Name <string> -Value <object>`, vous pouvez l’ignorer en toute sécurité. Il indique que le domaine ne définit pas cette configuration via un objet de stratégie de groupe.

1. Le protocole RDP (Remote Desktop Protocol) est activé :

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server' -Name fDenyTSConnections -Value 0 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name fDenyTSConnections -Value 0 -Type DWord -Force
   ```

1. Le port RDP est correctement configuré à l’aide du port 3389 par défaut :

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name PortNumber -Value 3389 -Type DWord -Force
   ```

   Lorsque vous déployez une machine virtuelle, les règles par défaut sont créés pour le port 3389. Si vous souhaitez modifier le numéro de port, faites-le une fois la machine virtuelle déployée dans Azure.

1. L’écouteur surveille chaque interface réseau :

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name LanAdapter -Value 0 -Type DWord -Force
   ```

1. Configurez le mode d’authentification au niveau du réseau (NLA) pour les connexions RDP :

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name UserAuthentication -Value 1 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name SecurityLayer -Value 1 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name fAllowSecProtocolNegotiation -Value 1 -Type DWord -Force
   ```

1. Définissez la valeur KeepAlive :

   ```PowerShell
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name KeepAliveEnable -Value 1  -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name KeepAliveInterval -Value 1  -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name KeepAliveTimeout -Value 1 -Type DWord -Force
   ```

1. Définissez les options de reconnexion :

   ```PowerShell
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services' -Name fDisableAutoReconnect -Value 0 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name fInheritReconnectSame -Value 1 -Type DWord -Force
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name fReconnectSame -Value 0 -Type DWord -Force
   ```

1. Limitez le nombre de connexions simultanées :

   ```powershell
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\Winstations\RDP-Tcp' -Name MaxInstanceCount -Value 4294967295 -Type DWord -Force
   ```

1. Supprimez les certificats auto-signés liés à l’écouteur RDP :

   ```powershell
   if ((Get-Item -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp').Property -contains 'SSLCertificateSHA1Hash')
   {
       Remove-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name SSLCertificateSHA1Hash -Force
   }
   ```

   Ce code garantit que vous pouvez vous connecter lors du déploiement de la machine virtuelle. Vous pouvez également examiner ces paramètres après le déploiement de la machine virtuelle dans Azure.

1. Si la machine virtuelle fait partie d’un domaine, vérifiez les stratégies suivantes pour vous assurer que les paramètres précédents ne sont pas rétablis.

    |                 Objectif                  |                                                                            Stratégie                                                                            |                           Valeur                            |
    | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
    | Activer le protocole RDP                        | Configuration de l’ordinateur\Stratégies\Paramètres Windows\Modèles d’administration\Composants\Services Bureau à distance\Hôte de session Bureau à distance\Connexions         | Autoriser les utilisateurs à se connecter à distance avec le Bureau à distance    |
    | Stratégie de groupe d’authentification au niveau du réseau                      | Paramètres\Modèles d’administration\Composants\Services Bureau à distance\Hôte de session Bureau à distance\Sécurité                                                    | Exigez une authentification de l’utilisateur pour l’accès à distance à l’aide de l’authentification NLA |
    | Paramètres de connexion toujours active                   | Configuration de l’ordinateur\Stratégies\Paramètres Windows\Modèles d’administration\Composants Windows\Services Bureau à distance\Hôte de session Bureau à distance\Connexions | Configurer l’intervalle de connexion KeepAlive                   |
    | Paramètres de reconnexion                    | Configuration de l’ordinateur\Stratégies\Paramètres Windows\Modèles d’administration\Composants Windows\Services Bureau à distance\Hôte de session Bureau à distance\Connexions | Se reconnecter automatiquement                                    |
    | Nombre limité de paramètres de connexion | Configuration de l’ordinateur\Stratégies\Paramètres Windows\Modèles d’administration\Composants Windows\Services Bureau à distance\Hôte de session Bureau à distance\Connexions | Limiter le nombre de connexions                                |

## <a name="configure-windows-firewall-rules"></a>Configurer les règles du Pare-feu Windows

1. Activez le pare-feu Windows sur les trois profils (domaine, standard et public) :

   ```powershell
   Set-NetFirewallProfile -Profile Domain, Public, Private -Enabled True
   ```

1. Exécutez l’exemple suivant pour autoriser WinRM sur les trois profils de pare-feu (domaine, privé et public) et activer le service à distance PowerShell :

   ```powershell
   Enable-PSRemoting -Force
   Set-NetFirewallRule -DisplayName 'Windows Remote Management (HTTP-In)' -Enabled True
   ```

1. Activez les règles de pare-feu suivantes pour autoriser le trafic RDP :

   ```powershell
   Set-NetFirewallRule -DisplayGroup 'Remote Desktop' -Enabled True
   ```

1. Activez la règle de partage de fichiers et d’imprimantes afin que la machine virtuelle puisse répondre à des demandes ping à l’intérieur du réseau virtuel :

   ```powershell
   Set-NetFirewallRule -DisplayName 'File and Printer Sharing (Echo Request - ICMPv4-In)' -Enabled True
   ```

1. Créez une règle pour le réseau de la plateforme Azure :

   ```powershell
   New-NetFirewallRule -DisplayName AzurePlatform -Direction Inbound -RemoteAddress 168.63.129.16 -Profile Any -Action Allow -EdgeTraversalPolicy Allow
   New-NetFirewallRule -DisplayName AzurePlatform -Direction Outbound -RemoteAddress 168.63.129.16 -Profile Any -Action Allow
   ```

1. Si la machine virtuelle fait partie d’un domaine, vérifiez les stratégies Azure AD suivantes pour vous assurer que les paramètres précédents ne sont pas rétablis.

    |                 Objectif                 |                                                                         Stratégie                                                                          |                  Valeur                  |
    | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
    | Activer les profils de pare-feu Windows | Configuration de l’ordinateur\Stratégies\Paramètres Windows\Modèles d’administration\Réseau\Connexion réseau\Pare-feu Windows\Profil de domaine\Pare-feu Windows   | Protéger toutes les connexions réseau         |
    | Activer le protocole RDP                           | Configuration de l’ordinateur\Stratégies\Paramètres Windows\Modèles d’administration\Réseau\Connexion réseau\Pare-feu Windows\Profil de domaine\Pare-feu Windows   | Autoriser les exceptions du Bureau à distance entrantes |
    |                                      | Configuration de l’ordinateur\Stratégies\Paramètres Windows\Modèles d’administration\Réseau\Connexion réseau\Pare-feu Windows\Profil standard\Pare-feu Windows | Autoriser les exceptions du Bureau à distance entrantes |
    | Activer le protocole ICMP-V4                       | Configuration de l’ordinateur\Stratégies\Paramètres Windows\Modèles d’administration\Réseau\Connexion réseau\Pare-feu Windows\Profil de domaine\Pare-feu Windows   | Autoriser les exceptions ICMP                   |
    |                                      | Configuration de l’ordinateur\Stratégies\Paramètres Windows\Modèles d’administration\Réseau\Connexion réseau\Pare-feu Windows\Profil standard\Pare-feu Windows | Autoriser les exceptions ICMP                   |

## <a name="verify-the-vm"></a>Vérification de la machine virtuelle

Assurez-vous que la machine virtuelle est saine, sécurisé et accessible au protocole RDP (Remote Desktop Protocol) :

1. Pour vous assurer de l’intégrité et de la cohérence du disque, vérifiez le disque au redémarrage suivant de la machine virtuelle :

   ```powershell
   chkdsk.exe /f
   ```

   Vérifiez que le rapport indique que le disque est propre et sain.

1. Définissez les paramètres Données de configuration de démarrage (BCD).

   ```powershell
   bcdedit.exe /set "{bootmgr}" integrityservices enable
   bcdedit.exe /set "{default}" device partition=C:
   bcdedit.exe /set "{default}" integrityservices enable
   bcdedit.exe /set "{default}" recoveryenabled Off
   bcdedit.exe /set "{default}" osdevice partition=C:
   bcdedit.exe /set "{default}" bootstatuspolicy IgnoreAllFailures

   #Enable Serial Console Feature
   bcdedit.exe /set "{bootmgr}" displaybootmenu yes
   bcdedit.exe /set "{bootmgr}" timeout 5
   bcdedit.exe /set "{bootmgr}" bootems yes
   bcdedit.exe /ems "{current}" ON
   bcdedit.exe /emssettings EMSPORT:1 EMSBAUDRATE:115200
   ```

1. Le journal de vidage peut être utile pour résoudre les problèmes de blocage de Windows. Activez la collecte des journaux de vidage :

   ```powershell
   # Set up the guest OS to collect a kernel dump on an OS crash event
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl' -Name CrashDumpEnabled -Type DWord -Force -Value 2
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl' -Name DumpFile -Type ExpandString -Force -Value "%SystemRoot%\MEMORY.DMP"
   Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl' -Name NMICrashDump -Type DWord -Force -Value 1

   # Set up the guest OS to collect user mode dumps on a service crash event
   $key = 'HKLM:\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps'
   if ((Test-Path -Path $key) -eq $false) {(New-Item -Path 'HKLM:\SOFTWARE\Microsoft\Windows\Windows Error Reporting' -Name LocalDumps)}
   New-ItemProperty -Path $key -Name DumpFolder -Type ExpandString -Force -Value 'C:\CrashDumps'
   New-ItemProperty -Path $key -Name CrashCount -Type DWord -Force -Value 10
   New-ItemProperty -Path $key -Name DumpType -Type DWord -Force -Value 2
   Set-Service -Name WerSvc -StartupType Manual
   ```

1. Vérifiez que le référentiel Windows Management Instrumentation (WMI) est cohérent :

   ```powershell
   winmgmt.exe /verifyrepository
   ```

   Si le référentiel est endommagé, consultez [WMI: Corruption du référentiel ou pas](https://techcommunity.microsoft.com/t5/ask-the-performance-team/wmi-repository-corruption-or-not/ba-p/375484).

1. Assurez-vous qu’aucune autre application n’utilise le port 3389. Ce port est utilisé pour le service RDP dans Azure. Pour voir quels ports sont utilisés sur la machine virtuelle, exécutez `netstat.exe -anob` :

   ```powershell
   netstat.exe -anob
   ```

1. Pour charger un disque dur virtuel Windows qui est un contrôleur de domaine :

   - Suivez [ces étapes supplémentaires](https://support.microsoft.com/kb/2904015) pour préparer le disque.

   - Assurez-vous que vous connaissez le mot de passe du mode de restauration des services d’annuaire au cas où vous devriez démarrer la machine virtuelle dans ce mode. Pour plus d’informations, consultez [Définir un mot de passe DSRM](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc754363(v=ws.11)).

1. Assurez-vous que vous connaissez le compte administrateur intégré et le mot de passe. Vous souhaiterez peut-être réinitialiser le mot de passe administrateur local actuel et vérifier que vous pouvez utiliser ce compte pour vous connecter à Windows par le biais de la connexion RDP. Cette autorisation d’accès est contrôlée par l’objet de stratégie de groupe « Autoriser l’ouverture de session par les services Bureau à distance ». Affichez cet objet dans l’Éditeur de stratégie de groupe locale :

   - `Computer Configuration\Windows Settings\Security Settings\Local Policies\User Rights Assignment`

1. Vérifiez les stratégies Azure AD suivantes pour vous assurer qu’elles ne bloquent pas l’accès RDP :

   - `Computer Configuration\Windows Settings\Security Settings\Local Policies\User Rights
      Assignment\Deny access to this computer from the network`

   - `Computer Configuration\Windows Settings\Security Settings\Local Policies\User Rights
      Assignment\Deny log on through Remote Desktop Services`

1. Vérifiez la stratégie Azure AD suivante pour vous assurer qu’elle ne supprime aucun compte d’accès requis :

   - `Computer Configuration\Windows Settings\Security Settings\Local Policies\User Rights Assignment\Access this computer from the network`

   La stratégie doit répertorier les groupes suivants :

   - Administrateurs

   - Opérateurs de sauvegarde

   - Tout le monde

   - Utilisateurs

1. Redémarrez la machine virtuelle pour vous assurer que Windows est toujours sain et qu’il est accessible par le biais de la connexion RDP. À ce stade, envisagez de créer une machine virtuelle sur votre serveur Hyper-V local pour vous assurer que la machine virtuelle démarre complètement. Faites un test pour vous assurer que vous pouvez contacter la machine virtuelle via RDP.

1. Supprimez tous les filtres Interface TDI supplémentaires. Par exemple, supprimez les logiciels qui analysent les paquets TCP ou les pare-feu supplémentaires.

1. Désinstallez tous les autres logiciels ou pilotes tiers liés aux composants physiques ou toute autre technologie de virtualisation.

### <a name="install-windows-updates"></a>Installer les mises à jour Windows

Dans l’idéal, vous devez garder l’ordinateur à jour *au niveau du correctif*. Si cela n’est pas possible, assurez-vous que les mises à jour suivantes sont installées. Pour obtenir les dernières mises à jour, consultez les pages de l’historique des mises à jour de Windows : [Windows 10 et Windows Server 2019](https://support.microsoft.com/help/4000825), [Windows 8.1 et Windows Server 2012 R2](https://support.microsoft.com/help/4009470) et [Windows 7 SP1 et Windows Server 2008 R2 SP1](https://support.microsoft.com/help/4009469).

<br />

|        Composant        |     Binary     | Windows 7 SP1, Windows Server 2008 R2 SP1 |       Windows 8, Windows Server 2012        | Windows 8.1, Windows Server 2012 R2 | Windows 10 v1607, Windows Server 2016 v1607 |      Windows 10 v1703      | Windows 10 v1709, Windows Server 2016 v1709 | Windows 10 v1803, Windows Server 2016 v1803 |
| ----------------------- | -------------- | ----------------------------------------- | ------------------------------------------- | ----------------------------------- | ------------------------------------------- | -------------------------- | ------------------------------------------- | ------------------------------------------- |
| Stockage                 | disk.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.17638 / 6.2.9200.21757 - KB3137061 | 6.3.9600.18203 - KB3137061          | -                                           | -                          | -                                           | -                                           |
|                         | storport.sys   | 6.1.7601.23403 - KB3125574                | 6.2.9200.17188 / 6.2.9200.21306 - KB3018489 | 6.3.9600.18573 - KB4022726          | 10.0.14393.1358 - KB4022715                 | 10.0.15063.332             | -                                           | -                                           |
|                         | ntfs.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.17623 / 6.2.9200.21743 - KB3121255 | 6.3.9600.18654 - KB4022726          | 10.0.14393.1198 - KB4022715                 | 10.0.15063.447             | -                                           | -                                           |
|                         | Iologmsg.dll   | 6.1.7601.23403 - KB3125574                | 6.2.9200.16384 - KB2995387                  | -                                   | -                                           | -                          | -                                           | -                                           |
|                         | Classpnp.sys   | 6.1.7601.23403 - KB3125574                | 6.2.9200.17061 / 6.2.9200.21180 - KB2995387 | 6.3.9600.18334 - KB3172614          | 10.0.14393.953 - KB4022715                  | -                          | -                                           | -                                           |
|                         | Volsnap.sys    | 6.1.7601.23403 - KB3125574                | 6.2.9200.17047 / 6.2.9200.21165 - KB2975331 | 6.3.9600.18265 - KB3145384          | -                                           | 10.0.15063.0               | -                                           | -                                           |
|                         | partmgr.sys    | 6.1.7601.23403 - KB3125574                | 6.2.9200.16681 - KB2877114                  | 6.3.9600.17401 - KB3000850          | 10.0.14393.953 - KB4022715                  | 10.0.15063.0               | -                                           | -                                           |
|                         | volmgr.sys     |                                           |                                             |                                     |                                             | 10.0.15063.0               | -                                           | -                                           |
|                         | Volmgrx.sys    | 6.1.7601.23403 - KB3125574                | -                                           | -                                   | -                                           | 10.0.15063.0               | -                                           | -                                           |
|                         | Msiscsi.sys    | 6.1.7601.23403 - KB3125574                | 6.2.9200.21006 - KB2955163                  | 6.3.9600.18624 - KB4022726          | 10.0.14393.1066 - KB4022715                 | 10.0.15063.447             | -                                           | -                                           |
|                         | Msdsm.sys      | 6.1.7601.23403 - KB3125574                | 6.2.9200.21474 - KB3046101                  | 6.3.9600.18592 - KB4022726          | -                                           | -                          | -                                           | -                                           |
|                         | Mpio.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.21190 - KB3046101                  | 6.3.9600.18616 - KB4022726          | 10.0.14393.1198 - KB4022715                 | -                          | -                                           | -                                           |
|                         | vmstorfl.sys   | 6.3.9600.18907 – KB4072650                | 6.3.9600.18080 – KB3063109                  | 6.3.9600.18907 – KB4072650          | 10.0.14393.2007 – KB4345418                 | 10.0.15063.850 – KB4345419 | 10.0.16299.371 – KB4345420                  | -                                           |
|                         | Fveapi.dll     | 6.1.7601.23311 - KB3125574                | 6.2.9200.20930 - KB2930244                  | 6.3.9600.18294 - KB3172614          | 10.0.14393.576 - KB4022715                  | -                          | -                                           | -                                           |
|                         | Fveapibase.dll | 6.1.7601.23403 - KB3125574                | 6.2.9200.20930 - KB2930244                  | 6.3.9600.17415 - KB3172614          | 10.0.14393.206 - KB4022715                  | -                          | -                                           | -                                           |
| Réseau                 | netvsc.sys     | -                                         | -                                           | -                                   | 10.0.14393.1198 - KB4022715                 | 10.0.15063.250 - KB4020001 | -                                           | -                                           |
|                         | mrxsmb10.sys   | 6.1.7601.23816 - KB4022722                | 6.2.9200.22108 - KB4022724                  | 6.3.9600.18603 - KB4022726          | 10.0.14393.479 - KB4022715                  | 10.0.15063.483             | -                                           | -                                           |
|                         | mrxsmb20.sys   | 6.1.7601.23816 - KB4022722                | 6.2.9200.21548 - KB4022724                  | 6.3.9600.18586 - KB4022726          | 10.0.14393.953 - KB4022715                  | 10.0.15063.483             | -                                           | -                                           |
|                         | mrxsmb.sys     | 6.1.7601.23816 - KB4022722                | 6.2.9200.22074 - KB4022724                  | 6.3.9600.18586 - KB4022726          | 10.0.14393.953 - KB4022715                  | 10.0.15063.0               | -                                           | -                                           |
|                         | tcpip.sys      | 6.1.7601.23761 - KB4022722                | 6.2.9200.22070 - KB4022724                  | 6.3.9600.18478 - KB4022726          | 10.0.14393.1358 - KB4022715                 | 10.0.15063.447             | -                                           | -                                           |
|                         | http.sys       | 6.1.7601.23403 - KB3125574                | 6.2.9200.17285 - KB3042553                  | 6.3.9600.18574 - KB4022726          | 10.0.14393.251 - KB4022715                  | 10.0.15063.483             | -                                           | -                                           |
|                         | vmswitch.sys   | 6.1.7601.23727 - KB4022719                | 6.2.9200.22117 - KB4022724                  | 6.3.9600.18654 - KB4022726          | 10.0.14393.1358 - KB4022715                 | 10.0.15063.138             | -                                           | -                                           |
| Core                    | ntoskrnl.exe   | 6.1.7601.23807 - KB4022719                | 6.2.9200.22170 - KB4022718                  | 6.3.9600.18696 - KB4022726          | 10.0.14393.1358 - KB4022715                 | 10.0.15063.483             | -                                           | -                                           |
| Services Bureau à distance | rdpcorets.dll  | 6.2.9200.21506 - KB4022719                | 6.2.9200.22104 - KB4022724                  | 6.3.9600.18619 - KB4022726          | 10.0.14393.1198 - KB4022715                 | 10.0.15063.0               | -                                           | -                                           |
|                         | termsrv.dll    | 6.1.7601.23403 - KB3125574                | 6.2.9200.17048 - KB2973501                  | 6.3.9600.17415 - KB3000850          | 10.0.14393.0 - KB4022715                    | 10.0.15063.0               | -                                           | -                                           |
|                         | termdd.sys     | 6.1.7601.23403 - KB3125574                | -                                           | -                                   | -                                           | -                          | -                                           | -                                           |
|                         | win32k.sys     | 6.1.7601.23807 - KB4022719                | 6.2.9200.22168 - KB4022718                  | 6.3.9600.18698 - KB4022726          | 10.0.14393.594 - KB4022715                  | -                          | -                                           | -                                           |
|                         | rdpdd.dll      | 6.1.7601.23403 - KB3125574                | -                                           | -                                   | -                                           | -                          | -                                           | -                                           |
|                         | rdpwd.sys      | 6.1.7601.23403 - KB3125574                | -                                           | -                                   | -                                           | -                          | -                                           | -                                           |
| Sécurité                | MS17-010       | KB4012212                                 | KB4012213                                   | KB4012213                           | KB4012606                                   | KB4012606                  | -                                           | -                                           |
|                         |                |                                           | KB4012216                                   |                                     | KB4013198                                   | KB4013198                  | -                                           | -                                           |
|                         |                | KB4012215                                 | KB4012214                                   | KB4012216                           | KB4013429                                   | KB4013429                  | -                                           | -                                           |
|                         |                |                                           | KB4012217                                   |                                     | KB4013429                                   | KB4013429                  | -                                           | -                                           |
|                         | CVE-2018-0886  | KB4103718                                 | KB4103730                                   | KB4103725                           | KB4103723                                   | KB4103731                  | KB4103727                                   | KB4103721                                   |
|                         |                | KB4103712                                 | KB4103726                                   | KB4103715                           |                                             |                            |                                             |                                             |

> [!NOTE]
> Pour éviter un redémarrage accidentel durant le provisionnement d’une machine virtuelle, nous vous recommandons de vous assurer que toutes les installations de Windows Update sont terminées et qu’aucune mise à jour n’est en attente. Pour ce faire, vous pouvez, par exemple, installer toutes les mises à jour Windows disponibles et effectuer un redémarrage avant d’exécuter la commande `sysprep.exe`.

## <a name="determine-when-to-use-sysprep"></a>Déterminer quand utiliser Sysprep

L’outil de préparation du système (`sysprep.exe`) est un processus que vous pouvez exécuter pour réinitialiser une installation Windows.
Sysprep fournit une expérience « prête à l’emploi » en supprimant toutes les données personnelles et en réinitialisant plusieurs composants.

En général, vous exécutez `sysprep.exe` pour créer un modèle à partir duquel déployer plusieurs autres machines virtuelles ayant une configuration spécifique. Le modèle est appelé *image généralisée*.

Si vous souhaitez ne créer qu’une seule machine virtuelle à partir d’un seul disque, vous n’avez pas besoin d’utiliser Sysprep. Au lieu de cela, vous pouvez créer la machine virtuelle à partir d’une *image spécialisée*. Pour obtenir des informations sur la création d’une machine virtuelle à partir d’un disque spécialisé, consultez les pages :

- [Créer une machine virtuelle à partir d’un disque spécialisé](create-vm-specialized.md)
- [Créer une machine virtuelle à partir d’un disque dur virtuel spécialisé](./create-vm-specialized-portal.md)

Pour créer une image généralisée, vous devez exécuter Sysprep. Pour plus d’informations, consultez [Utilisation de Sysprep : Une introduction](/previous-versions/windows/it-pro/windows-xp/bb457073(v=technet.10)).

Tous les rôles ou toutes les applications installés sur un ordinateur Windows ne prennent pas forcément en charge des images généralisées. Avant d’exécuter cette procédure, vérifiez que Sysprep prend bien en charge le rôle de l’ordinateur. Pour plus d’informations, voir [Prise en charge de Sysprep pour les rôles serveur](/windows-hardware/manufacture/desktop/sysprep-support-for-server-roles).

En particulier, un déchiffrement complet des lecteurs est nécessaire avant l’exécution de Sysprep. Si vous avez activé le chiffrement sur votre machine virtuelle, désactivez-le avant d’exécuter Sysprep.


### <a name="generalize-a-vhd"></a>Généraliser un disque dur virtuel

>[!NOTE]
> Après avoir exécuté `sysprep.exe` dans les étapes suivantes, arrêtez la machine virtuelle. Ne la réactivez pas avant d’avoir créé une image à partir de celle-ci dans Azure.

1. Connectez-vous à la machine virtuelle Windows.
1. Démarrez une session PowerShell en tant qu’administrateur.
1. Supprimez le répertoire Panther (C:\Windows\Panther).
1. Modifier le répertoire en `%windir%\system32\sysprep`. Exécutez ensuite `sysprep.exe`.
1. Dans la boîte de dialogue **Outil de préparation du système**, sélectionnez **Entrer en mode OOBE (Out-of-Box Experience)** , puis vérifiez que la case à cocher **Généraliser** est activée.

    ![Outil de préparation système](media/prepare-for-upload-vhd-image/syspre.png)
1. Dans **Options d’arrêt**, sélectionnez **Arrêter**.
1. Sélectionnez **OK**.
1. Lorsque l’exécution de Sysprep est terminée, arrêtez la machine virtuelle. N’utilisez pas l’option **Redémarrer** pour arrêter la machine virtuelle.

Le disque dur virtuel est maintenant prêt à être chargé. Pour plus d’informations sur la création d’une machine virtuelle à partir d’un disque généralisé, consultez la page [Charger un disque dur virtuel généralisé sur Azure pour créer une machine virtuelle](/previous-versions/azure/virtual-machines/windows/sa-upload-generalized).

>[!NOTE]
> Un fichier *unattend.xml* personnalisé n’est pas pris en charge. Bien que nous prenions en charge la propriété **additionalUnattendContent**, celle-ci n’offre qu’une prise en charge limitée de l’ajout d’options [microsoft-windows-shell-setup](/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup) dans le fichier *unattend.xml* que l’agent d’approvisionnement Azure utilise. Vous pouvez, par exemple, utiliser [additionalUnattendContent](/dotnet/api/microsoft.azure.management.compute.models.additionalunattendcontent?view=azure-dotnet&preserve-view=true) pour ajouter FirstLogonCommands et LogonCommands. Pour plus d'informations, consultez l’[exemple additionalUnattendContent FirstLogonCommands](https://github.com/Azure/azure-quickstart-templates/issues/1407).

## <a name="convert-the-virtual-disk-to-a-fixed-size-vhd"></a>Convertir le disque virtuel en disque dur virtuel de taille fixe

Utilisez l’une des méthodes décrites dans cette section pour convertir et redimensionner votre disque virtuel au format requis pour Azure :

1. Sauvegardez la machine virtuelle avant d’exécuter la conversion de disque virtuel ou de redimensionner le processus.

1. Vérifiez que le disque dur virtuel Windows fonctionne correctement sur le serveur local. Résolvez toute erreur dans la machine virtuelle elle-même avant de tenter une conversion ou un chargement sur Azure.

1. Convertissez le disque virtuel en type fixe.

1. Redimensionnez le disque virtuel pour répondre aux exigences d’Azure :

   1. Les disques dans Azure doivent avoir une taille virtuelle alignée sur 1 Mio. Si votre disque dur virtuel est une fraction de 1 Mio, vous devrez redimensionner le disque en un multiple de 1 Mio. Les disques qui sont des fractions d’un Mio entraînent des erreurs lors de la création d’images à partir du disque dur virtuel chargé. Pour vérifier la taille, vous pouvez utiliser la cmdlet PowerShell [Get-VHD](/powershell/module/hyper-v/get-vhd) pour afficher « Size », qui doit être un multiple de 1 Mio dans Azure, et « FileSize », qui est égal à « Size » plus 512 octets pour le pied de page VHD.
   
   1. La taille maximale autorisée pour le disque dur virtuel du système d’exploitation avec une machine virtuelle de première génération est de 2,048 Gio (2 Tio), 
   1. La taille maximale pour un disque de données est de 32,767 Gio (32 Tio).

> [!NOTE]
> - Si vous préparez un disque de système d’exploitation Windows après l’avoir converti en disque fixe et redimensionné, si nécessaire, créez une machine virtuelle qui utilise le disque. Démarrez et connectez-vous à la machine virtuelle et poursuivez avec les sections de cet article pour finir de la préparer pour le téléchargement.  
> - Si vous préparez un disque de données, vous pouvez vous arrêter avec cette section et poursuivre le téléchargement de votre disque.

### <a name="use-hyper-v-manager-to-convert-the-disk"></a>Utilisez le gestionnaire Hyper-V pour convertir le disque

1. Ouvrez le Gestionnaire Hyper-V et sélectionnez votre ordinateur local sur la gauche. Dans le menu au-dessus de la liste d’ordinateurs, sélectionnez **Action** > **Modifier le disque**.
1. À la page **Rechercher un disque dur virtuel**, sélectionnez le disque virtuel.
1. À la page **Choisir une action**, sélectionnez **Convertir** > **suivant**.
1. Pour convertir un VHDX, sélectionnez **VHD** > **Suivant**.
1. Pour convertir un disque à taille dynamique, sélectionnez **Taille fixe** > **Suivant**.
1. Recherchez et sélectionnez un chemin d’accès pour enregistrer le nouveau fichier de disque dur virtuel.
1. Sélectionnez **Terminer**.

### <a name="use-powershell-to-convert-the-disk"></a>Utilisez PowerShell pour convertir le disque

Vous pouvez convertir un disque virtuel à l’aide de la cmdlet [Convert-VHD](/powershell/module/hyper-v/convert-vhd) dans PowerShell. Si vous avez besoin d’informations sur l’installation de cette cmdlet, consultez [Installer le rôle Hyper-V](/windows-server/virtualization/hyper-v/get-started/install-the-hyper-v-role-on-windows-server).

L’exemple de commande suivant convertit le disque VHDX en disque VHD. Il convertit également le disque à taille dynamique en disque de taille fixe.

```powershell
Convert-VHD -Path C:\test\MyVM.vhdx -DestinationPath C:\test\MyNewVM.vhd -VHDType Fixed
```

Dans cette commande, remplacez la valeur **Path** par le chemin d’accès du disque dur virtuel que vous souhaitez convertir. Remplacez la valeur **DestinationPath** par les nouveaux chemin d’accès et nom du disque converti.

### <a name="use-hyper-v-manager-to-resize-the-disk"></a>Utiliser le gestionnaire Hyper-V pour redimensionner le disque

1. Ouvrez le Gestionnaire Hyper-V et sélectionnez votre ordinateur local sur la gauche. Dans le menu au-dessus de la liste d’ordinateurs, sélectionnez **Action** > **Modifier le disque**.
1. À la page **Rechercher un disque dur virtuel**, sélectionnez le disque virtuel.
1. À la page **Choisir une action**, sélectionnez **Développer** > **Suivant**.
1. Sur la page **Rechercher un disque dur virtuel**, entrez la nouvelle taille en Gio > **Suivant**.
1. Sélectionnez **Terminer**.

### <a name="use-powershell-to-resize-the-disk"></a>Utiliser PowerShell pour redimensionner le disque

Vous pouvez redimensionner un disque virtuel à l’aide de la cmdlet [Resize-VHD](/powershell/module/hyper-v/resize-vhd) dans PowerShell. Si vous avez besoin d’informations sur l’installation de cette cmdlet, consultez [Installer le rôle Hyper-V](/windows-server/virtualization/hyper-v/get-started/install-the-hyper-v-role-on-windows-server).

L’exemple suivant redimensionne le disque de 100,5 Mio à 101 Mio pour respecter les exigences d’alignement Azure.

```powershell
Resize-VHD -Path C:\test\MyNewVM.vhd -SizeBytes 105906176
```

Dans cette commande, remplacez la valeur **Path** par le chemin d’accès du disque dur virtuel que vous souhaitez redimensionner. Remplacez la valeur **SizeBytes** par la nouvelle taille en octets du disque.

### <a name="convert-from-vmware-vmdk-disk-format"></a>Convertir à partir du format de disque VMDK VMware

Si vous disposez d’une image de machine virtuelle Windows au [format de fichier VMDK](https://en.wikipedia.org/wiki/VMDK), vous pouvez utiliser [Azure Migrate](../../migrate/server-migrate-overview.md) pour convertir le fichier VMDK et le charger sur Azure.

## <a name="complete-the-recommended-configurations"></a>Remplir les configurations recommandées

Les paramètres suivants n’affectent pas le chargement du disque dur virtuel. Toutefois, nous vous recommandons vivement de les configurer.

- Installez [l’agent Machine virtuelles Azure](https://go.microsoft.com/fwlink/?LinkID=394789). Vous pouvez ensuite activer les extensions de machine virtuelle. Les extensions de machine virtuelle mettent en œuvre la plupart des fonctionnalités stratégiques que vous pourriez vouloir utiliser avec vos machines virtuelles. Vous aurez, par exemple, besoin des extensions pour réinitialiser les mots de passe ou configurer le protocole RDP (Remote Desktop Protocol). Pour plus d’informations, consultez l’article [Vue d’ensemble de l’agent de machine virtuelle Azure](../extensions/agent-windows.md).
- Après avoir créé la machine virtuelle dans Azure, nous vous recommandons de placer le fichier d’échange sur le *volume de disque temporaire* pour améliorer les performances. Vous pouvez définir le positionnement du fichier comme suit :

  ```powershell
  Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management' -Name PagingFiles -Value 'D:\pagefile.sys' -Type MultiString -Force
  ```

  Si un disque de données est attaché à la machine virtuelle, la lettre de lecteur du volume de disque temporaire est généralement *D*. Cette désignation peut être différente, en fonction de vos paramètres et du nombre de disques disponibles.

  - Nous vous recommandons de désactiver les bloqueurs de scripts fournis par des logiciels antivirus. Ils pourraient interférer avec les scripts de l’agent de provisionnement Windows et en bloquer l’exécution lorsque vous déployez une nouvelle machine virtuelle à partir de votre image.

## <a name="next-steps"></a>Étapes suivantes

- [Télécharger une image de machine virtuelle Windows dans Azure pour des déploiements Resource Manager](upload-generalized-managed.md)
- [Résoudre des problèmes liés à l’activation de machines virtuelles Windows Azure](../troubleshooting/troubleshoot-activation-problems.md)