---
title: Résoudre les erreurs d’authentification quand vous utilisez le protocole RDP pour vous connecter à une machine virtuelle Azure | Microsoft Docs
description: Découvrez comment résoudre les erreurs d’authentification qui se produisent quand vous utilisez Remote Desktop Protocol (RDP) pour vous connecter à une machine virtuelle Azure.
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
ms.date: 11/01/2018
ms.author: delhan
ms.openlocfilehash: cc1ad3104596cc7ad4bb48f88e4c8312bc833371
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87088561"
---
# <a name="troubleshoot-authentication-errors-when-you-use-rdp-to-connect-to-azure-vm"></a>Résoudre les erreurs d’authentification quand vous utilisez le protocole RDP pour vous connecter à une machine virtuelle Azure

Cet article peut vous aider à résoudre les erreurs d’authentification qui se produisent quand vous utilisez une connexion RDP (Remote Desktop Protocol) pour vous connecter à une machine virtuelle.

## <a name="symptoms"></a>Symptômes

Vous capturez une capture d’écran d’une machine virtuelle Azure qui affiche l’écran d’accueil et indique que le système d’exploitation est en cours d’exécution. Toutefois, quand vous essayez de vous connecter à la machine virtuelle à l’aide de Connexion Bureau à distance, vous recevez l’un des messages d’erreur suivants.

### <a name="error-message-1"></a>Message d’erreur 1

**Une erreur d’authentification s’est produite. L’autorité de sécurité locale ne peut pas être contactée.**

### <a name="error-message-2"></a>Message d’erreur 2

**L’ordinateur distant auquel vous essayez de vous connecter nécessite l’authentification au niveau du réseau (NLA), mais votre contrôleur de domaine Windows ne peut pas être contacté pour effectuer l’authentification NLA. Si vous êtes administrateur sur l’ordinateur distant, vous pouvez désactiver l’authentification NLA en utilisant les options sous l’onglet Distant de la boîte de dialogue Propriétés système.**

### <a name="error-message-3-generic-connection-error"></a>Message d’erreur 3 (erreur de connexion générique)

**Cet ordinateur ne peut pas se connecter à l’ordinateur distant. Réessayez de vous connecter. Si le problème persiste, contactez le propriétaire de l’ordinateur distant ou votre administrateur réseau.**

## <a name="cause"></a>Cause

Il existe plusieurs raisons pour lesquelles l’authentification NLA peut bloquer l’accès RDP à une machine virtuelle.

### <a name="cause-1"></a>Cause 1

La machine virtuelle ne peut pas communiquer avec le contrôleur de domaine. Ce problème peut empêcher une session Bureau à distance d’accéder à une machine virtuelle à l’aide des informations d’identification de domaine. Toutefois, vous pourriez toujours vous connecter en utilisant les informations d’identification d’Administrateur Local. Ce problème peut se produire dans les situations suivantes :

1. Le canal de sécurité Active Directory entre cet ordinateur virtuel et le contrôleur de domaine est rompu.

2. La machine virtuelle a une ancienne copie du mot de passe de compte, et le contrôleur de domaine a une copie plus récente.

3. Le contrôleur de domaine auquel cette machine virtuelle se connecte est défectueux.

### <a name="cause-2"></a>Cause 2

Le niveau de chiffrement de la machine virtuelle est supérieur à celui utilisé par l’ordinateur client.

### <a name="cause-3"></a>Cause 3

Les protocoles (serveur) TLS 1.0, 1.1 ou 1.2 sont désactivés sur la machine virtuelle.

### <a name="cause-4"></a>Cause 4

La machine virtuelle a été configurée pour désactiver l’ouverture de session à l’aide des informations d’identification de domaine, et l’autorité de sécurité locale (LSA) est configurée de manière incorrecte.

### <a name="cause-5"></a>Cause 5

La machine virtuelle a été configurée pour accepter uniquement les connexions à algorithmes conformes à la norme FIPS (Federal Information Processing Standard). Cette action s’effectue généralement à l’aide d’une stratégie Active Directory. Il s’agit d’une configuration rare, mais la norme FIPS peut être appliquée pour les connexions Bureau à distance uniquement.

## <a name="before-you-troubleshoot"></a>Avant de résoudre le problème

### <a name="create-a-backup-snapshot"></a>Créer un instantané de sauvegarde

Pour créer un instantané de sauvegarde, procédez de la manière décrite dans [Effectuer la capture instantanée d’un disque](../windows/snapshot-copy-managed-disk.md).

### <a name="connect-to-the-vm-remotely"></a>Se connecter à la machine virtuelle à distance

Pour vous connecter à la machine virtuelle à distance, appliquez l’une des méthodes décrites dans [Comment utiliser les outils à distance pour résoudre les problèmes de machine virtuelle Azure](remote-tools-troubleshoot-azure-vm-issues.md).

### <a name="group-policy-client-service"></a>Service Client de stratégie de groupe

S’il s’agit d’une machine virtuelle jointe à un domaine, arrêtez tout d’abord le service Client de stratégie de groupe pour empêcher toute stratégie Active Directory de remplacer les modifications. Pour ce faire, exécutez la commande suivante :

```cmd
REM Disable the member server to retrieve the latest GPO from the domain upon start
REG add "HKLM\SYSTEM\CurrentControlSet\Services\gpsvc" /v Start /t REG_DWORD /d 4 /f
```

Une fois le problème résolu, restaurez la capacité de cette machine virtuelle à contacter le domaine pour récupérer l’objet de stratégie de groupe le plus récent à partir du domaine. Pour cela, exécutez les commandes suivantes :

```cmd
sc config gpsvc start= auto
sc start gpsvc

gpupdate /force
```

Si la modification est annulée, cela signifie que le problème est dû à une stratégie Active Directory. 

### <a name="workaround"></a>Solution de contournement

Pour contourner ce problème, exécutez les commandes suivantes dans la fenêtre de commande pour désactiver l’authentification NLA :

```cmd
REM Disable the Network Level Authentication
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v SecurityLayer /t REG_DWORD /d 0
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v fAllowSecProtocolNegotiation /t REG_DWORD /d 0
```

Ensuite, redémarrez la machine virtuelle.

Pour réactiver l’authentification NLA, exécutez la commande suivante puis redémarrez la machine virtuelle :

```cmd
REG add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v disabledomaincreds /t REG_DWORD /d 0 /f

REG add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v SecurityLayer /t REG_DWORD /d 1 /f
REG add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 1 /f
REG add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v fAllowSecProtocolNegotiation /t REG_DWORD /d 1 /f
```

## <a name="troubleshooting"></a>Dépannage

### <a name="for-domain-joined-vms"></a>Pour les machines virtuelles jointes à un domaine

Pour résoudre ce problème, vérifiez tout d’abord si la machine virtuelle peut se connecter à un contrôleur de domaine, et si le contrôleur de domaine est à l’état « sain » et peut gérer les requêtes en provenance de la machine virtuelle.

>[!Note] 
>Pour tester l’intégrité du contrôleur de domaine, vous pouvez utiliser une autre machine virtuelle sur le même réseau virtuel et sous-réseau qui partagent le même serveur d’ouverture de session.

Connectez-vous à la machine virtuelle qui rencontre le problème à l’aide de Serial console, de l’environnement de commande distant ou de PowerShell à distance, en respectant les étapes décrites dans la section « Se connecter à la machine virtuelle à distance ».

Pour identifier le contrôleur de domaine auquel la machine virtuelle se connecte, exécutez la commande suivante dans la console : 

```cmd
set | find /i "LOGONSERVER"
```

Ensuite, vérifiez l’intégrité du canal sécurisé entre la machine virtuelle et le contrôleur de domaine. Pour ce faire, exécutez la commande suivante dans une instance de PowerShell avec élévation de privilèges. Cette commande retourne un indicateur booléen qui indique si le canal sécurisé est actif :

```powershell
Test-ComputerSecureChannel -verbose
```

Si le canal est rompu, exécutez la commande suivante afin de le réparer :

```powershell
Test-ComputerSecureChannel -repair
```

Vérifiez que le mot de passe du compte d’ordinateur dans Active Directory est mis à jour sur la machine virtuelle et sur le contrôleur de domaine :

```powershell
Reset-ComputerMachinePassword -Server "<COMPUTERNAME>" -Credential <DOMAIN CREDENTIAL WITH DOMAIN ADMIN LEVEL>
```

Si la communication entre le contrôleur de domaine et la machine virtuelle est bonne, mais que le contrôleur de domaine n’est pas suffisamment sain pour ouvrir une session RDP, vous pouvez essayer de redémarrer le contrôleur de domaine.

Si les commandes précédentes n’ont pas résolu le problème de communication avec le domaine, vous pouvez rejoindre cette machine virtuelle au domaine. Pour ce faire, procédez comme suit :

1. Créez un script nommé Unjoin.ps1 en utilisant le contenu suivant, puis déployez-le en tant qu’Extension de script personnalisé sur le portail Azure :

    ```cmd
    cmd /c "netdom remove <<MachineName>> /domain:<<DomainName>> /userD:<<DomainAdminhere>> /passwordD:<<PasswordHere>> /reboot:10 /Force"
    ```
    
    Ce script extrait la machine virtuelle du domaine de manière forcée et la redémarre 10 secondes plus tard. Ensuite, vous devez nettoyer l’objet ordinateur du côté du domaine.

2.  Une fois le nettoyage terminé, rejoignez cette machine virtuelle au domaine. Pour ce faire, créez un script nommé JoinDomain.ps1 en utilisant le contenu suivant, puis déployez-le en tant qu’Extension de script personnalisé sur le portail Azure : 

    ```cmd
    cmd /c "netdom join <<MachineName>> /domain:<<DomainName>> /userD:<<DomainAdminhere>> /passwordD:<<PasswordHere>> /reboot:10"
    ```

    >[!Note] 
    >Ce script joint la machine virtuelle au domaine en utilisant les informations d’identification spécifiées.

Si le canal Active Directory est sain, que le mot de passe d’ordinateur est mis à jour et que le contrôleur de domaine fonctionne comme prévu, essayez les étapes suivantes.

Si le problème persiste, vérifiez si les informations d’identification de domaine sont désactivées. Pour ce faire, ouvrez une fenêtre d’invite de commandes avec élévation de privilèges, puis exécutez la commande suivante pour déterminer si la machine virtuelle est configurée afin de désactiver les comptes de domaine pour la connexion à la machine virtuelle :

```cmd
REG query "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v disabledomaincreds
```

Si la clé a la valeur **1**, cela signifie que le serveur a été configuré pour interdire les informations d’identification de domaine. Affectez la valeur **0** à cette clé.

### <a name="for-standalone-vms"></a>Pour les machines virtuelles autonomes

#### <a name="check-minencryptionlevel"></a>Vérifier MinEncryptionLevel

Dans une instance CMD, exécutez la commande suivante pour interroger la valeur de Registre **MinEncryptionLevel** :

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MinEncryptionLevel
```

En fonction de la valeur de Registre, effectuez les étapes suivantes :

* 4 (FIPS) : accédez à [Vérifier les connexions à algorithmes conformes à la norme FIPS](#fips-compliant).

* 3 (chiffrement 128 bits) : affectez la valeur **2** au niveau de gravité en exécutant la commande suivante :

    ```cmd
    reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MinEncryptionLevel /t REG_DWORD /d 2 /f
    ```

* 2 (chiffrement le plus élevé possible, comme stipulé par le client) : vous pouvez essayer d’affecter la valeur minimale **1** au chiffrement en exécutant la commande suivante :

    ```cmd
    reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MinEncryptionLevel /t REG_DWORD /d 1 /f
    ```
    
Redémarrez la machine virtuelle pour que les modifications apportées au Registre prennent effet.

#### <a name="tls-version"></a>Version de TLS

En fonction du système, RDP utilise le protocole (serveur) TLS 1.0, 1.1 ou 1.2. Pour interroger la façon dont ces protocoles sont configurés sur la machine virtuelle, ouvrez une instance CMD, puis exécutez les commandes suivantes :

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled
reg query "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled
reg query "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled
```

Si les valeurs retournées ne sont pas toutes **1**, cela signifie que le protocole est désactivé. Pour activer ces protocoles, exécutez les commandes suivantes :

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" /v Enabled /t REG_DWORD /d 1 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server" /v Enabled /t REG_DWORD /d 1 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server" /v Enabled /t REG_DWORD /d 1 /f
```

Pour les autres versions de protocole, vous pouvez exécuter les commandes suivantes :

<pre lang="bat">
reg query "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS <i>x.x</i>\Server" /v Enabled
reg query "HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS <i>x.x</i>\Server" /v Enabled
</pre>

> [!Note]
> Obtenez la version x.x de SSH/TLS à partir des journaux d’activité du système d’exploitation invité sur les erreurs SCHANNEL.

#### <a name="check-fips-compliant-algorithms-connections"></a><a name="fips-compliant"></a> Vérifier les connexions à algorithmes conformes à la norme FIPS

Vous pouvez forcer Bureau à distance à utiliser uniquement des connexions à algorithmes compatibles avec la norme FIPS. Cette configuration peut être effectuée à l’aide d’une clé de Registre. Pour ce faire, ouvrez une fenêtre d’invite de commandes avec élévation de privilèges, puis interrogez les clés suivantes :

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Lsa\FIPSAlgorithmPolicy" /v Enabled
```

Si la commande retourne **1**, affectez la valeur **0** à la valeur de Registre.

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Lsa\FIPSAlgorithmPolicy" /v Enabled /t REG_DWORD /d 0
```

Vérifiez quel est le MinEncryptionLevel actuel sur la machine virtuelle :

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MinEncryptionLevel
```

Si la commande retourne **4**, affectez la valeur **2** à la valeur de Registre.

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MinEncryptionLevel /t REG_DWORD /d 2
```

Redémarrez la machine virtuelle pour que les modifications apportées au Registre prennent effet.

## <a name="next-steps"></a>Étapes suivantes

[Méthode SetEncryptionLevel de la classe Win32_TSGeneralSetting](/windows/desktop/termserv/win32-tsgeneralsetting-setencryptionlevel)

[Configurer l’authentification du serveur et les niveaux de chiffrement](/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/cc770833(v=ws.11))

[Classe Win32_TSGeneralSetting](/windows/desktop/termserv/win32-tsgeneralsetting)
