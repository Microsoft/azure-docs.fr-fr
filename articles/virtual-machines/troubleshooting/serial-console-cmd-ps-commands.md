---
title: CMD et PowerShell dans les machines virtuelles Azure Windows | Microsoft Docs
description: Comment utiliser les commandes CMD et PowerShell au sein de la console SAC dans les machines virtuelles Azure Windows
services: virtual-machines-windows
documentationcenter: ''
author: alsin
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/14/2018
ms.author: alsin
ms.openlocfilehash: ef533b3566ac557b57f1435a2a9b2dbe26896993
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91306886"
---
# <a name="windows-commands---cmd-and-powershell"></a>Commandes Windows - CMD et PowerShell

Cette section présente des exemples de commandes permettant d’effectuer des tâches courantes dans les situations où vous pouvez avoir besoin d’utiliser la console SAC pour accéder à votre machine virtuelle Windows, notamment pour résoudre les échecs de connexion RDP.

La console SAC est incluse dans toutes les versions de Windows depuis Windows Server 2003, mais elle est désactivée par défaut. La console SAC s’appuie sur le pilote du noyau `sacdrv.sys`, le service `Special Administration Console Helper` (`sacsvr`) et le processus `sacsess.exe`. Pour plus d’informations, consultez la page [Emergency Management Services Tools and Settings (Paramètres et outils des services de gestion d’urgence)](/previous-versions/windows/it-pro/windows-server-2003/cc787940(v%3dws.10)).

La console SAC vous permet de vous connecter à votre système d’exploitation en cours d’exécution par le biais du port série. Lorsque vous lancez CMD à partir de la console SAC, `sacsess.exe` lance `cmd.exe` au sein de votre système d’exploitation en cours d’exécution. Dans le Gestionnaire des tâches, vous pouvez constater que si vous établissez une connexion RDP vers votre machine virtuelle simultanément, vous êtes connecté à la console SAC par le biais de la fonctionnalité de console série. L’instance CMD à laquelle vous accédez par le biais de la console SAC est la même `cmd.exe` que vous utilisez lorsque la connexion est établie par le biais du protocole RDP. Les mêmes commandes et les mêmes outils sont disponibles, y compris la possibilité de lancer PowerShell à partir de cette instance CMD. La console SAC diffère de l’environnement de récupération Windows (WinRE) : la console SAC vous permet de gérer votre système d’exploitation en cours d’exécution, tandis que WinRE démarre dans un système d’exploitation minimal. Alors que les machines virtuelles Azure ne permettent pas d’accéder à WinRE, les machines virtuelles Azure peuvent être gérées par le biais de la console SAC avec la fonctionnalité de console série.

La console SAC étant limitée à une mémoire tampon d’écran de 80x24 sans défilement arrière, ajoutez `| more` aux commandes pour afficher la sortie page par page. Utilisez `<spacebar>` pour afficher la page suivante ou `<enter>` pour afficher la ligne suivante.

`SHIFT+INSERT` est le raccourci de collage pour la fenêtre de console série.

En raison de la taille limitée de la mémoire tampon d’écran de la console SAC, il peut être plus facile de saisir les commandes les plus longues dans un éditeur de texte local, puis de les coller dans la console SAC.

## <a name="view-and-edit-windows-registry-settings-using-cmd"></a>Afficher et modifier les paramètres du Registre Windows avec CMD
### <a name="verify-rdp-is-enabled"></a>Vérifier que le protocole RDP est activé
`reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections`

`reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fDenyTSConnections`

La deuxième clé (sous \Policies) existe uniquement si le paramètre de stratégie de groupe approprié est configuré.

### <a name="enable-rdp"></a>Activer le protocole RDP
`reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0`

`reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fDenyTSConnections /t REG_DWORD /d 0`

La deuxième clé (sous \Policies) est nécessaire uniquement si le paramètre de stratégie de groupe approprié a été configuré. La valeur sera réécrite lors de la prochaine actualisation de la stratégie de groupe si ce paramètre est configuré dans la stratégie de groupe.

## <a name="manage-windows-services-using-cmd"></a>Gérer les services Windows avec CMD

### <a name="view-service-state"></a>Afficher l’état du service
`sc query termservice`
###  <a name="view-service-logon-account"></a>Afficher le compte de connexion au service
`sc qc termservice`
### <a name="set-service-logon-account"></a>Paramétrer le compte de connexion au service
`sc config termservice obj= "NT Authority\NetworkService"`

Un espace est requis après le signe égal.
### <a name="set-service-start-type"></a>Paramétrer le type de démarrage de service
`sc config termservice start= demand`

Un espace est requis après le signe égal. Les valeurs possibles de démarrage sont `boot`, `system`, `auto`, `demand`, `disabled` et `delayed-auto`.
### <a name="set-service-dependencies"></a>Paramétrer les dépendances de services
`sc config termservice depend= RPCSS`

Un espace est requis après le signe égal.
### <a name="start-service"></a>Démarrer le service
`net start termservice`

or

`sc start termservice`
### <a name="stop-service"></a>Arrêter le service
`net stop termservice`

or

`sc stop termservice`
## <a name="manage-networking-features-using-cmd"></a>Gérer les fonctionnalités réseau avec CMD
### <a name="show-nic-properties"></a>Afficher les propriétés de la carte réseau
`netsh interface show interface`
### <a name="show-ip-properties"></a>Afficher les propriétés IP
`netsh interface ip show config`
### <a name="show-ipsec-configuration"></a>Afficher la configuration IPSec
`netsh nap client show configuration`
### <a name="enable-nic"></a>Activer la carte réseau
`netsh interface set interface name="<interface name>" admin=enabled`
### <a name="set-nic-to-use-dhcp"></a>Paramétrer la carte réseau pour utiliser le protocole DHCP
`netsh interface ip set address name="<interface name>" source=dhcp`

Pour plus d’informations sur `netsh`, [cliquez ici](/windows-server/networking/technologies/netsh/netsh-contexts).

Les machines virtuelles Azure doivent toujours être configurées dans le système d’exploitation invité pour que vous puissiez utiliser le protocole DHCP afin d’obtenir une adresse IP. Le paramètre IP statique Azure utilise toujours le protocole DHCP pour fournir l’adresse IP statique à la machine virtuelle.
### <a name="ping"></a>Ping
`ping 8.8.8.8`
### <a name="port-ping"></a>Ping au niveau du port
Installer le client telnet

`dism /online /Enable-Feature /FeatureName:TelnetClient`

Tester la connectivité

`telnet bing.com 80`

Suppression du client telnet

`dism /online /Disable-Feature /FeatureName:TelnetClient`

Lorsque vous êtes limité aux méthodes disponibles dans Windows par défaut, PowerShell peut constituer une meilleure approche pour tester la connectivité des ports. Consultez la section PowerShell ci-dessous pour obtenir des exemples.
### <a name="test-dns-name-resolution"></a>Tester la résolution du nom DNS
`nslookup bing.com`
### <a name="show-windows-firewall-rule"></a>Afficher la règle de Pare-feu Windows
`netsh advfirewall firewall show rule name="Remote Desktop - User Mode (TCP-In)"`
### <a name="disable-windows-firewall"></a>Désactiver le Pare-feu Windows
`netsh advfirewall set allprofiles state off`

Vous pouvez utiliser cette commande lors de la résolution des problèmes pour écarter temporairement le Pare-feu Windows. Il sera activé au prochain redémarrage ou lorsque vous l’activerez à l’aide de la commande ci-dessous. N’arrêtez pas le service de Pare-feu Windows (MPSSVC) ni le service de moteur de filtrage de base (BFE) pour écarter le Pare-feu Windows. L’arrêt des services MPSSVC ou BFE entraîne le blocage de toute la connectivité.
### <a name="enable-windows-firewall"></a>Activer le Pare-feu Windows
`netsh advfirewall set allprofiles state on`
## <a name="manage-users-and-groups-using-cmd"></a>Gérer les utilisateurs et les groupes avec CMD
### <a name="create-local-user-account"></a>Créer un compte d’utilisateur local
`net user /add <username> <password>`
### <a name="add-local-user-to-local-group"></a>Ajouter un utilisateur local au groupe local
`net localgroup Administrators <username> /add`
### <a name="verify-user-account-is-enabled"></a>Vérifier que le compte d’utilisateur est activé
`net user <username> | find /i "active"`

Pour les machines virtuelles Azure créées à partir de l’image généralisée, le compte d’administrateur local sera renommé avec le nom spécifié lors de l’approvisionnement de la machine virtuelle. Il ne sera donc généralement pas `Administrator`.
### <a name="enable-user-account"></a>Activer un compte d’utilisateur
`net user <username> /active:yes`
### <a name="view-user-account-properties"></a>Afficher les propriétés du compte d’utilisateur
`net user <username>`

Exemples de lignes d’intérêt provenant d’un compte administrateur local :

`Account active Yes`

`Account expires Never`

`Password expires Never`

`Workstations allowed All`

`Logon hours allowed All`

`Local Group Memberships *Administrators`

### <a name="view-local-groups"></a>Afficher les groupes locaux
`net localgroup`
## <a name="manage-the-windows-event-log-using-cmd"></a>Gérer le journal des événements Windows avec CMD
### <a name="query-event-log-errors"></a>Interroger les erreurs du journal des événements
`wevtutil qe system /c:10 /f:text /q:"Event[System[Level=2]]" | more`

Remplacez `/c:10` par le nombre souhaité d’événements à retourner, ou supprimez-le pour retourner tous les événements correspondant au filtre.
### <a name="query-event-log-by-event-id"></a>Interroger le journal des événements en fonction de l’ID d’événement
`wevtutil qe system /c:1 /f:text /q:"Event[System[EventID=11]]" | more`
### <a name="query-event-log-by-event-id-and-provider"></a>Interroger le journal des événements en fonction de l’ID d’événement et du fournisseur
`wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Microsoft-Windows-Hyper-V-Netvsc'] and EventID=11]]" | more`
### <a name="query-event-log-by-event-id-and-provider-for-the-last-24-hours"></a>Interroger le journal des événements en fonction de l’ID d’événement et du fournisseur sur les 24 dernières heures
`wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Microsoft-Windows-Hyper-V-Netvsc'] and EventID=11 and TimeCreated[timediff(@SystemTime) <= 86400000]]]"`

Utilisez `604800000` pour effectuer la recherche sur 7 jours au lieu de 24 heures.
### <a name="query-event-log-by-event-id-provider-and-eventdata-in-the-last-7-days"></a>Interroger le journal des événements en fonction de l’ID d’événement, du fournisseur et d’EventData sur les 7 derniers jours
`wevtutil qe security /c:1 /f:text /q:"Event[System[Provider[@Name='Microsoft-Windows-Security-Auditing'] and EventID=4624 and TimeCreated[timediff(@SystemTime) <= 604800000]] and EventData[Data[@Name='TargetUserName']='<username>']]" | more`
## <a name="view-or-remove-installed-applications-using-cmd"></a>Afficher ou supprimer les applications installées avec CMD
### <a name="list-installed-applications"></a>Répertorier les applications installées
`wmic product get Name,InstallDate | sort /r | more`

`sort /r` permet d’effectuer un tri décroissant en fonction de la date d’installation pour vous aider à voir ce qui a été récemment installé. Utilisez `<spacebar>` pour passer à la page suivante de sortie ou `<enter>` pour avancer d’une ligne.
### <a name="uninstall-an-application"></a>Désinstaller une application
`wmic path win32_product where name="<name>" call uninstall`

Remplacez `<name>` par le nom retourné dans la commande ci-dessus pour l’application que vous souhaitez supprimer.

## <a name="file-system-management-using-cmd"></a>Gestion du système de fichiers avec CMD
### <a name="get-file-version"></a>Obtenir la version du fichier
`wmic datafile where "drive='C:' and path='\\windows\\system32\\drivers\\' and filename like 'netvsc%'" get version /format:list`

Cet exemple retourne la version du fichier du pilote de carte réseau virtuel, qui se nomme netvsc.sys, netvsc63.sys ou netvsc60.sys selon la version de Windows.
### <a name="scan-for-system-file-corruption"></a>Analyser l’endommagement des fichiers système
`sfc /scannow`

Voir aussi [Repair a Windows Image (Réparer une image Windows)](/windows-hardware/manufacture/desktop/repair-a-windows-image).
### <a name="scan-for-system-file-corruption"></a>Analyser l’endommagement des fichiers système
`dism /online /cleanup-image /scanhealth`

Voir aussi [Repair a Windows Image (Réparer une image Windows)](/windows-hardware/manufacture/desktop/repair-a-windows-image).
### <a name="export-file-permissions-to-text-file"></a>Exporter les autorisations de fichier vers un fichier texte
`icacls %programdata%\Microsoft\Crypto\RSA\MachineKeys /t /c > %temp%\MachineKeys_permissions_before.txt`
### <a name="save-file-permissions-to-acl-file"></a>Enregistrer les autorisations de fichier dans un fichier ACL
`icacls %programdata%\Microsoft\Crypto\RSA\MachineKeys /save %temp%\MachineKeys_permissions_before.aclfile /t`
### <a name="restore-file-permissions-from-acl-file"></a>Restaurer les autorisations de fichier à partir d’un fichier ACL
`icacls %programdata%\Microsoft\Crypto\RSA /save %temp%\MachineKeys_permissions_before.aclfile /t`

Le chemin d’accès lors de l’utilisation de `/restore` doit être le dossier parent du dossier que vous avez spécifié lors de l’utilisation de `/save`. Dans cet exemple, `\RSA` est le parent du dossier `\MachineKeys` spécifié dans l’exemple `/save` ci-dessus.
### <a name="take-ntfs-ownership-of-a-folder"></a>S’approprier les fichiers NTFS d’un dossier
`takeown /f %programdata%\Microsoft\Crypto\RSA\MachineKeys /a /r`
### <a name="grant-ntfs-permissions-to-a-folder-recursively"></a>Accorder des autorisations NTFS pour un dossier de manière récursive
`icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "BUILTIN\Administrators:(F)"`
## <a name="manage-devices"></a>Gérer des appareils
### <a name="remove-non-present-pnp-devices"></a>Supprimer les appareils Plug-and-Play non présents
`%windir%\System32\RUNDLL32.exe %windir%\System32\pnpclean.dll,RunDLL_PnpClean /Devices /Maxclean`
## <a name="manage-group-policy"></a>Gérer la stratégie de groupe
### <a name="force-group-policy-update"></a>Forcer la mise à jour de la stratégie de groupe
`gpupdate /force /wait:-1`
## <a name="miscellaneous-tasks-using-cmd"></a>Exécution de tâches diverses avec CMD
### <a name="show-os-version"></a>Afficher la version du système d’exploitation
`ver`

or

`wmic os get caption,version,buildnumber /format:list`

or

`systeminfo  find /i "os name"`

`systeminfo | findstr /i /r "os.*version.*build"`
### <a name="view-os-install-date"></a>Afficher la date d’installation du système d’exploitation
`systeminfo | find /i "original"`

or

`wmic os get installdate`
### <a name="view-last-boot-time"></a>Afficher l’heure du dernier démarrage
`systeminfo | find /i "system boot time"`
### <a name="view-time-zone"></a>Afficher le fuseau horaire
`systeminfo | find /i "time zone"`

or

`wmic timezone get caption,standardname /format:list`
### <a name="restart-windows"></a>Redémarrer Windows
`shutdown /r /t 0`

L’ajout de `/f` force la fermeture des applications en cours d’exécution sans avertir l’utilisateur.
### <a name="detect-safe-mode-boot"></a>Détecter le démarrage en mode sans échec
`bcdedit /enum | find /i "safeboot"`

## <a name="windows-commands---powershell"></a>Commandes Windows - PowerShell

Pour exécuter PowerShell dans la console SAC, après avoir accédé à une invite de commandes, entrez :

`powershell <enter>`

> [!CAUTION]
> Supprimez le module PSReadLine à partir de la session PowerShell avant d’exécuter les autres commandes PowerShell. Il existe un problème connu qui entraîne l’introduction de caractères supplémentaires dans le texte collé à partir du Presse-papiers si le module PSReadLine est en cours d’exécution dans une session PowerShell au sein de la console SAC.

Vérifiez tout d’abord si PSReadLine est chargé. Il est chargé par défaut sur Windows Server 2016, Windows 10 et les versions ultérieures de Windows. Il sera uniquement présent dans les versions antérieures de Windows s’il a été installé manuellement.

Si cette commande renvoie à une invite sans aucune sortie, cela signifie que le module n’a pas été chargé et vous pouvez continuer à utiliser normalement la session PowerShell dans la console SAC.

`get-module psreadline`

Si la commande ci-dessus retourne la version du module PSReadLine, exécutez la commande suivante pour la décharger. Cette commande ne supprime pas et ne désinstalle pas le module. Elle la décharge uniquement de la session PowerShell active.

`remove-module psreadline`

## <a name="view-and-edit-windows-registry-settings-using-powershell"></a>Afficher et modifier les paramètres du Registre Windows avec PowerShell
### <a name="verify-rdp-is-enabled"></a>Vérifier que le protocole RDP est activé
`get-itemproperty -path 'hklm:\system\curRentcontrolset\control\terminal server' -name 'fdenytsconNections'`

`get-itemproperty -path 'hklm:\software\policies\microsoft\windows nt\terminal services' -name 'fdenytsconNections'`

La deuxième clé (sous \Policies) existe uniquement si le paramètre de stratégie de groupe approprié est configuré.
### <a name="enable-rdp"></a>Activer le protocole RDP
`set-itemproperty -path 'hklm:\system\curRentcontrolset\control\terminal server' -name 'fdenytsconNections' 0 -type dword`

`set-itemproperty -path 'hklm:\software\policies\microsoft\windows nt\terminal services' -name 'fdenytsconNections' 0 -type dword`

La deuxième clé (sous \Policies) est nécessaire uniquement si le paramètre de stratégie de groupe approprié a été configuré. La valeur sera réécrite lors de la prochaine actualisation de la stratégie de groupe si ce paramètre est configuré dans la stratégie de groupe.
## <a name="manage-windows-services-using-powershell"></a>Gérer les services Windows avec PowerShell
### <a name="view-service-details"></a>Afficher les détails sur le service
`get-wmiobject win32_service -filter "name='termservice'" |  format-list Name,DisplayName,State,StartMode,StartName,PathName,ServiceType,Status,ExitCode,ServiceSpecificExitCode,ProcessId`

Il est possible d’utiliser `Get-Service`, mais cela n’inclut pas le compte d’ouverture de session du service. `Get-WmiObject win32-service` l’inclut.
### <a name="set-service-logon-account"></a>Paramétrer le compte de connexion au service
`(get-wmiobject win32_service -filter "name='termservice'").Change($null,$null,$null,$null,$null,$false,'NT Authority\NetworkService')`

Lorsque vous utilisez un compte de service autre que `NT AUTHORITY\LocalService`, `NT AUTHORITY\NetworkService` ou `LocalSystem`, spécifiez le mot de passe du compte en tant que dernier argument (huitième) après le nom du compte.
### <a name="set-service-startup-type"></a>Paramétrer le type de démarrage du service
`set-service termservice -startuptype Manual`

`Set-service` accepte `Automatic`, `Manual` ou `Disabled` comme type de démarrage.
### <a name="set-service-dependencies"></a>Paramétrer les dépendances de services
`Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\TermService' -Name DependOnService -Value @('RPCSS','TermDD')`
### <a name="start-service"></a>Démarrer le service
`start-service termservice`
### <a name="stop-service"></a>Arrêter le service
`stop-service termservice`
## <a name="manage-networking-features-using-powershell"></a>Gérer les fonctionnalités réseau avec PowerShell
### <a name="show-nic-properties"></a>Afficher les propriétés de la carte réseau
`get-netadapter | where {$_.ifdesc.startswith('Microsoft Hyper-V Network Adapter')} |  format-list status,name,ifdesc,macadDresS,driverversion,MediaConNectState,MediaDuplexState`

or

`get-wmiobject win32_networkadapter -filter "servicename='netvsc'" |  format-list netenabled,name,macaddress`

`Get-NetAdapter` est disponible dans les versions 2012 et supérieures. Pour 2008R2, utilisez `Get-WmiObject`.
### <a name="show-ip-properties"></a>Afficher les propriétés IP
`get-wmiobject Win32_NetworkAdapterConfiguration -filter "ServiceName='netvsc'" |  format-list DNSHostName,IPAddress,DHCPEnabled,IPSubnet,DefaultIPGateway,MACAddress,DHCPServer,DNSServerSearchOrder`
### <a name="enable-nic"></a>Activer la carte réseau
`get-netadapter | where {$_.ifdesc.startswith('Microsoft Hyper-V Network Adapter')} | enable-netadapter`

or

`(get-wmiobject win32_networkadapter -filter "servicename='netvsc'").enable()`

`Get-NetAdapter` est disponible dans les versions 2012 et supérieures. Pour 2008R2, utilisez `Get-WmiObject`.
### <a name="set-nic-to-use-dhcp"></a>Paramétrer la carte réseau pour utiliser le protocole DHCP
`get-netadapter | where {$_.ifdesc.startswith('Microsoft Hyper-V Network Adapter')} | Set-NetIPInterface -DHCP Enabled`

`(get-wmiobject Win32_NetworkAdapterConfiguration -filter "ServiceName='netvsc'").EnableDHCP()`

`Get-NetAdapter` est disponible dans les versions 2012 et supérieures. Pour 2008R2, utilisez `Get-WmiObject`. Les machines virtuelles Azure doivent toujours être configurées dans le système d’exploitation invité pour que vous puissiez utiliser le protocole DHCP afin d’obtenir une adresse IP. Le paramètre IP statique Azure utilise toujours le protocole DHCP pour fournir l’adresse IP à la machine virtuelle.
### <a name="ping"></a>Ping
`test-netconnection`

> [!NOTE]
> La cmdlet Write-Progress peut ne pas fonctionner avec cette commande. Pour résoudre le problème, vous pouvez exécuter `$ProgressPreference = "SilentlyContinue"` dans PowerShell afin de désactiver la barre de progression.

or

`get-wmiobject Win32_PingStatus -Filter 'Address="8.8.8.8"' | format-table -autosize IPV4Address,ReplySize,ResponseTime`

`Test-Netconnection` sans aucun paramètre va tenter d’exécuter une commande ping sur `internetbeacon.msedge.net`. Cette commande est disponible sur les versions 2012 et supérieures. Pour 2008R2, utilisez `Get-WmiObject` comme dans le deuxième exemple.
### <a name="port-ping"></a>Ping au niveau du port
`test-netconnection -ComputerName bing.com -Port 80`

or

`(new-object Net.Sockets.TcpClient).BeginConnect('bing.com','80',$null,$null).AsyncWaitHandle.WaitOne(300)`

`Test-NetConnection` est disponible dans les versions 2012 et supérieures. Pour 2008R2, utilisez `Net.Sockets.TcpClient`
### <a name="test-dns-name-resolution"></a>Tester la résolution du nom DNS
`resolve-dnsname bing.com`

or

`[System.Net.Dns]::GetHostAddresses('bing.com')`

`Resolve-DnsName` est disponible dans les versions 2012 et supérieures. Pour 2008R2, utilisez `System.Net.DNS`.
### <a name="show-windows-firewall-rule-by-name"></a>Afficher la règle de Pare-feu Windows en fonction du nom
`get-netfirewallrule -name RemoteDesktop-UserMode-In-TCP`
### <a name="show-windows-firewall-rule-by-port"></a>Afficher la règle de Pare-feu Windows en fonction du port
`get-netfirewallportfilter | where {$_.localport -eq 3389} | foreach {Get-NetFirewallRule -Name $_.InstanceId} | format-list Name,Enabled,Profile,Direction,Action`

or

`(new-object -ComObject hnetcfg.fwpolicy2).rules | where {$_.localports -eq 3389 -and $_.direction -eq 1} | format-table Name,Enabled`

`Get-NetFirewallPortFilter` est disponible dans les versions 2012 et supérieures. Pour 2008R2, utilisez l’objet COM `hnetcfg.fwpolicy2`.
### <a name="disable-windows-firewall"></a>Désactiver le Pare-feu Windows
`Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False`

`Set-NetFirewallProfile` est disponible dans les versions 2012 et supérieures. Pour 2008R2, utilisez `netsh advfirewall` comme indiqué dans la section CMD ci-dessus.
## <a name="manage-users-and-groups-using-powershell"></a>Gérer les utilisateurs et les groupes avec PowerShell
### <a name="create-local-user-account"></a>Créer un compte d’utilisateur local
`new-localuser <name>`
### <a name="verify-user-account-is-enabled"></a>Vérifier que le compte d’utilisateur est activé
`(get-localuser | where {$_.SID -like "S-1-5-21-*-500"}).Enabled`

or

`(get-wmiobject Win32_UserAccount -Namespace "root\cimv2" -Filter "SID like 'S-1-5-%-500'").Disabled`

`Get-LocalUser` est disponible dans les versions 2012 et supérieures. Pour 2008R2, utilisez `Get-WmiObject`. Cet exemple montre le compte administrateur local intégré, qui présente toujours le SID `S-1-5-21-*-500`. Pour les machines virtuelles Azure créées à partir de l’image généralisée, le compte d’administrateur local sera renommé avec le nom spécifié lors de l’approvisionnement de la machine virtuelle. Il ne sera donc généralement pas `Administrator`.
### <a name="add-local-user-to-local-group"></a>Ajouter un utilisateur local au groupe local
`add-localgroupmember -group Administrators -member <username>`
### <a name="enable-local-user-account"></a>Activer un compte d’utilisateur local
`get-localuser | where {$_.SID -like "S-1-5-21-*-500"} | enable-localuser`

Cet exemple active le compte administrateur local intégré, qui présente toujours le SID `S-1-5-21-*-500`. Pour les machines virtuelles Azure créées à partir de l’image généralisée, le compte d’administrateur local sera renommé avec le nom spécifié lors de l’approvisionnement de la machine virtuelle. Il ne sera donc généralement pas `Administrator`.
### <a name="view-user-account-properties"></a>Afficher les propriétés du compte d’utilisateur
`get-localuser | where {$_.SID -like "S-1-5-21-*-500"} | format-list *`

or

`get-wmiobject Win32_UserAccount -Namespace "root\cimv2" -Filter "SID like 'S-1-5-%-500'" |  format-list Name,Disabled,Status,Lockout,Description,SID`

`Get-LocalUser` est disponible dans les versions 2012 et supérieures. Pour 2008R2, utilisez `Get-WmiObject`. Cet exemple montre le compte administrateur local intégré, qui présente toujours le SID `S-1-5-21-*-500`.
### <a name="view-local-groups"></a>Afficher les groupes locaux
`(get-localgroup).name | sort` `(get-wmiobject win32_group).Name | sort`

`Get-LocalUser` est disponible dans les versions 2012 et supérieures. Pour 2008R2, utilisez `Get-WmiObject`.
## <a name="manage-the-windows-event-log-using-powershell"></a>Gérer le journal des événements Windows avec PowerShell
### <a name="query-event-log-errors"></a>Interroger les erreurs du journal des événements
`get-winevent -logname system -maxevents 1 -filterxpath "*[System[Level=2]]" | more`

Remplacez `/c:10` par le nombre souhaité d’événements à retourner, ou supprimez-le pour retourner tous les événements correspondant au filtre.
### <a name="query-event-log-by-event-id"></a>Interroger le journal des événements en fonction de l’ID d’événement
`get-winevent -logname system -maxevents 1 -filterxpath "*[System[EventID=11]]" | more`
### <a name="query-event-log-by-event-id-and-provider"></a>Interroger le journal des événements en fonction de l’ID d’événement et du fournisseur
`get-winevent -logname system -maxevents 1 -filterxpath "*[System[Provider[@Name='Microsoft-Windows-Hyper-V-Netvsc'] and EventID=11]]" | more`
### <a name="query-event-log-by-event-id-and-provider-for-the-last-24-hours"></a>Interroger le journal des événements en fonction de l’ID d’événement et du fournisseur sur les 24 dernières heures
`get-winevent -logname system -maxevents 1 -filterxpath "*[System[Provider[@Name='Microsoft-Windows-Hyper-V-Netvsc'] and EventID=11 and TimeCreated[timediff(@SystemTime) <= 86400000]]]"`

Utilisez `604800000` pour effectuer la recherche sur 7 jours au lieu de 24 heures. |
### <a name="query-event-log-by-event-id-provider-and-eventdata-in-the-last-7-days"></a>Interroger le journal des événements en fonction de l’ID d’événement, du fournisseur et d’EventData sur les 7 derniers jours
`get-winevent -logname system -maxevents 1 -filterxpath "*[System[Provider[@Name='Microsoft-Windows-Security-Auditing'] and EventID=4624 and TimeCreated[timediff(@SystemTime) <= 604800000]] and EventData[Data[@Name='TargetUserName']='<username>']]" | more`
## <a name="view-or-remove-installed-applications-using-powershell"></a>Afficher ou supprimer les applications installées avec PowerShell
### <a name="list-installed-software"></a>Répertorier les logiciels installés
`get-wmiobject win32_product | select installdate,name | sort installdate -descending | more`
### <a name="uninstall-software"></a>Désinstaller un logiciel
`(get-wmiobject win32_product -filter "Name='<name>'").Uninstall()`
## <a name="file-system-management-using-powershell"></a>Gestion du système de fichiers avec PowerShell
### <a name="get-file-version"></a>Obtenir la version du fichier
`(get-childitem $env:windir\system32\drivers\netvsc*.sys).VersionInfo.FileVersion`

Cet exemple retourne la version du fichier du pilote de carte réseau virtuel, qui se nomme netvsc.sys, netvsc63.sys ou netvsc60.sys selon la version de Windows.
### <a name="download-and-extract-file"></a>Télécharger et extraire le fichier
`$path='c:\bin';md $path;cd $path;(new-object net.webclient).downloadfile( ('htTp:/'+'/download.sysinternals.com/files/SysinternalsSuite.zip'),"$path\SysinternalsSuite.zip");(new-object -com shelL.apPlication).namespace($path).CopyHere( (new-object -com shelL.apPlication).namespace("$path\SysinternalsSuite.zip").Items(),16)`

Cet exemple crée un dossier `c:\bin`, puis télécharge et extrait la suite d’outils Sysinternals dans `c:\bin`.
## <a name="miscellaneous-tasks-using-powershell"></a>Exécution de tâches diverses avec PowerShell
### <a name="show-os-version"></a>Afficher la version du système d’exploitation
`get-wmiobject win32_operatingsystem | format-list caption,version,buildnumber`
### <a name="view-os-install-date"></a>Afficher la date d’installation du système d’exploitation
`(get-wmiobject win32_operatingsystem).converttodatetime((get-wmiobject win32_operatingsystem).installdate)`
### <a name="view-last-boot-time"></a>Afficher l’heure du dernier démarrage
`(get-wmiobject win32_operatingsystem).lastbootuptime`
### <a name="view-windows-uptime"></a>Afficher la durée de fonctionnement dans Windows
`"{0:dd}:{0:hh}:{0:mm}:{0:ss}.{0:ff}" -f ((get-date)-(get-wmiobject win32_operatingsystem).converttodatetime((get-wmiobject win32_operatingsystem).lastbootuptime))`

Retourne la durée de fonctionnement sous la forme `<days>:<hours>:<minutes>:<seconds>:<milliseconds>`, par exemple `49:16:48:00.00`.
### <a name="restart-windows"></a>Redémarrer Windows
`restart-computer`

L’ajout de `-force` force la fermeture des applications en cours d’exécution sans avertir l’utilisateur.
## <a name="instance-metadata"></a>Métadonnées d’instance

Vous pouvez interroger les métadonnées d’instance Azure à partir de votre machine virtuelle Azure pour afficher des détails tels qu’osType, Location, vmSize, vmId, name, resourceGroupName, subscriptionId, privateIpAddress et publicIpAddress.

L’interrogation des métadonnées d’instance nécessite une connectivité de réseau invité saine, car elle effectue un appel REST par le biais de l’hôte Azure vers le service de métadonnées d’instance. Par conséquent, si vous êtes en mesure d’interroger les métadonnées d’instance, cela vous indique que l’invité peut communiquer sur le réseau vers un service Azure.

Pour plus d’informations, consultez [Service de métadonnées d’instance Azure](../windows/instance-metadata-service.md).

### <a name="instance-metadata"></a>Métadonnées d’instance
`$im = invoke-restmethod -headers @{"metadata"="true"} -uri http://169.254.169.254/metadata/instance?api-version=2017-08-01 -method get`

`$im | convertto-json`
### <a name="os-type-instance-metadata"></a>Type de système d’exploitation (métadonnées d’instance)
`$im.Compute.osType`
### <a name="location-instance-metadata"></a>Emplacement (métadonnées d’instance)
`$im.Compute.Location`
### <a name="size-instance-metadata"></a>Taille (métadonnées d’instance)
`$im.Compute.vmSize`
### <a name="vm-id-instance-metadata"></a>ID de la machine virtuelle (métadonnées d’instance)
`$im.Compute.vmId`
### <a name="vm-name-instance-metadata"></a>Nom de la machine virtuelle (métadonnées d’instance)
`$im.Compute.name`
### <a name="resource-group-name-instance-metadata"></a>Nom du groupe de ressources (métadonnées d’instance)
`$im.Compute.resourceGroupName`
### <a name="subscription-id-instance-metadata"></a>ID d’abonnement (métadonnées d’instance)
`$im.Compute.subscriptionId`
### <a name="tags-instance-metadata"></a>Balises (métadonnées d’instance)
`$im.Compute.tags`
### <a name="placement-group-id-instance-metadata"></a>ID du groupe de placement (métadonnées d’instance)
`$im.Compute.placementGroupId`
### <a name="platform-fault-domain-instance-metadata"></a>Domaine d’erreur de plateforme (métadonnées d’instance)
`$im.Compute.platformFaultDomain`
### <a name="platform-update-domain-instance-metadata"></a>Domaine de mise à jour de plateforme (métadonnées d’instance)
`$im.Compute.platformUpdateDomain`
### <a name="ipv4-private-ip-address-instance-metadata"></a>Adresse IP privée IPv4 (métadonnées d’instance)
`$im.network.interface.ipv4.ipAddress.privateIpAddress`
### <a name="ipv4-public-ip-address-instance-metadata"></a>Adresse IP publique IPv4 (métadonnées d’instance)
`$im.network.interface.ipv4.ipAddress.publicIpAddress`
### <a name="ipv4-subnet-address--prefix-instance-metadata"></a>Adresse de sous-réseau/préfixe IPv4 (métadonnées d’instance)
`$im.network.interface.ipv4.subnet.address`

`$im.network.interface.ipv4.subnet.prefix`
### <a name="ipv6-ip-address-instance-metadata"></a>Adresse IP IPv6 (métadonnées d’instance)
`$im.network.interface.ipv6.ipAddress`
### <a name="mac-address-instance-metadata"></a>Adresse MAC (métadonnées d’instance)
`$im.network.interface.macAddress`

## <a name="next-steps"></a>Étapes suivantes
* La page principale de documentation de la console série Windows se trouve [ici](serial-console-windows.md).
* La console série est également disponible pour les machines virtuelles [Linux](serial-console-linux.md).
* En savoir plus sur les [diagnostics de démarrage](boot-diagnostics.md).
