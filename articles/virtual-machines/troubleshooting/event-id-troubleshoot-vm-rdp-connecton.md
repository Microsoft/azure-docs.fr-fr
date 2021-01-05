---
title: Résoudre les problèmes de connexion RDP des machines virtuelles Azure par ID d’événement | Microsoft Docs
description: Utilisez les ID d’événement pour résoudre les différents problèmes qui empêchent une connexion par protocole RDP (Remote Desktop Protocol) à une machine virtuelle Azure.
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
ms.openlocfilehash: 507cd6cfe9f251dbc304b579d634ff986b001264
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87088595"
---
# <a name="troubleshoot-azure-vm-rdp-connection-issues-by-event-id"></a>Résoudre les problèmes de connexion RDP des machines virtuelles Azure par ID d’événement 

Cet article explique comment utiliser les ID d’événement pour résoudre les problèmes qui empêchent une connexion par protocole RDP (Remote Desktop Protocol) à une machine virtuelle Azure.

## <a name="symptoms"></a>Symptômes

Vous essayez d’utiliser une session RDP (Remote Desktop Protocol) pour vous connecter à une machine virtuelle Azure. Vous entrez vos informations d’identification, puis la connexion échoue, et vous recevez le message d’erreur suivant :

**Cet ordinateur ne peut pas se connecter à l’ordinateur distant. Réessayez de vous connecter. Si le problème persiste, contactez le propriétaire de l’ordinateur distant ou votre administrateur réseau.**

Pour résoudre ce problème, passez en revue les journaux d’événements sur la machine virtuelle, puis reportez-vous aux scénarios suivants.

## <a name="before-you-troubleshoot"></a>Avant de résoudre le problème

### <a name="create-a-backup-snapshot"></a>Créer un instantané de sauvegarde

Pour créer un instantané de sauvegarde, procédez de la manière décrite dans [Effectuer la capture instantanée d’un disque](../windows/snapshot-copy-managed-disk.md).

### <a name="connect-to-the-vm-remotely"></a>Se connecter à la machine virtuelle à distance

Pour vous connecter à la machine virtuelle à distance, appliquez l’une des méthodes décrites dans [Comment utiliser les outils à distance pour résoudre les problèmes de machine virtuelle Azure](remote-tools-troubleshoot-azure-vm-issues.md).

## <a name="scenario-1"></a>Scénario 1

### <a name="event-logs"></a>Journaux d’événements

Dans une instance CMD, exécutez les commandes suivantes pour vérifier si l’événement 1058 ou 1057 a été enregistré dans le journal système au cours des dernières 24 heures :

```cmd
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Microsoft-Windows-TerminalServices-RemoteConnectionManager'] and EventID=1058 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Microsoft-Windows-TerminalServices-RemoteConnectionManager'] and EventID=1057 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
```

**Nom du journal :**      Système <br />
**Source :**        Microsoft-Windows-TerminalServices-RemoteConnectionManager <br />
**Date :**          *heure* <br />
**ID d’événement :**      1058 <br />
**Catégorie de tâche :** None <br />
**Niveau :**         Error <br />
**Mots clés :**      Classique <br />
**Utilisateur :**          N/A <br />
**Ordinateur :**      *ordinateur* <br />
**Description :** Le serveur hôte de session Bureau à distance n’a pas pu remplacer le certificat autosigné ayant expiré et qui était utilisé pour l’authentification du serveur hôte de session Bureau à distance sur les connexions TLS. Le code d’état approprié est Access is denied (L’accès est refusé).

**Nom du journal :**      Système <br />
**Source :**        Microsoft-Windows-TerminalServices-RemoteConnectionManager <br />
**Date :**          *heure* <br />
**ID d’événement :**      1058 <br />
**Catégorie de tâche :** None <br />
**Niveau :**         Error <br />
**Mots clés :**      Classique <br />
**Utilisateur :**          N/A <br />
**Ordinateur :**      *ordinateur* <br />
**Description :** Le serveur hôte de session Bureau à distance n’a pas pu créer un certificat autosigné à utiliser pour l’authentification du serveur hôte de session Bureau à distance sur les connexions TLS. Le code d’état approprié est « Object already exists » (L’objet existe déjà).

**Nom du journal :**      Système <br />
**Source :**        Microsoft-Windows-TerminalServices-RemoteConnectionManager <br />
**Date :**          *heure* <br />
**ID d’événement :**      1057 <br />
**Catégorie de tâche :** None <br />
**Niveau :**         Error <br />
**Mots clés :**      Classique <br />
**Utilisateur :**          N/A <br />
**Ordinateur :**      *ordinateur* <br />
**Description :** Le serveur hôte de session Bureau à distance n’a pas pu créer un certificat autosigné à utiliser pour l’authentification du serveur hôte de session Bureau à distance sur les connexions TLS. Le code d’état approprié est Keyset does not exist (Le jeu de clés n’existe pas).

Vous pouvez également vérifier les événements d’erreur SCHANNEL 36872 et 36870 en exécutant les commandes suivantes :

```cmd
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Schannel'] and EventID=36870 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Schannel'] and EventID=36872 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
```

**Nom du journal :**      Système <br />
**Source :**        Schannel <br />
**Date :**          — <br />
**ID d’événement :**      36870 <br />
**Catégorie de tâche :** None <br />
**Niveau :**         Error <br />
**Mots clés :**       <br />
**Utilisateur :**          SYSTEM <br />
**Ordinateur :**      *ordinateur* <br />
**Description :** Une erreur irrécupérable s’est produite lors de la tentative d’accès à la clé privée des informations d’identification du serveur TLS. Le code d’erreur retourné par le module de chiffrement est 0x8009030D.  <br />
L’état d’erreur interne est 10001.

### <a name="cause"></a>Cause
Ce problème se produit parce que les clés de chiffrement RSA locales dans le dossier MachineKeys sur la machine virtuelle ne sont pas accessibles. Ce problème peut se produire pour l’une des raisons suivantes :

1. Configuration des autorisations incorrecte sur le dossier Machinekeys ou les fichiers RSA.

2. Clé RSA endommagée ou manquante.

### <a name="resolution"></a>Résolution

Pour résoudre ce problème, vous devez configurer les autorisations appropriées sur le certificat RDP en appliquant la procédure suivante.

#### <a name="grant-permission-to-the-machinekeys-folder"></a>Accorder l’autorisation au dossier MachineKeys

1. Créez un script en utilisant le contenu suivant :

   ```powershell
   remove-module psreadline 
   icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\BeforeScript_permissions.txt
   takeown /f "C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys" /a /r
   icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\System:(F)"
   icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "NT AUTHORITY\NETWORK SERVICE:(R)"
   icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "BUILTIN\Administrators:(F)"
   icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c > c:\temp\AfterScript_permissions.txt
   Restart-Service TermService -Force
   ```

2.  Exécutez ce script pour réinitialiser les autorisations du dossier MachineKey et réinitialiser les fichiers RSA selon les valeurs par défaut.

3.  Réessayez d’accéder à la machine virtuelle.

Après avoir exécuté le script, vous pouvez vérifier les fichiers suivants rencontrant des problèmes d’autorisation :

* c:\temp\BeforeScript_permissions.txt
* c:\temp\AfterScript_permissions.txt

#### <a name="renew-rdp-self-signed-certificate"></a>Renouveler le certificat auto-signé RDP

Si le problème persiste, exécutez le script suivant pour vous assurer que le certificat auto-signé RDP est renouvelé :

```powershell
Import-Module PKI
Set-Location Cert:\LocalMachine
$RdpCertThumbprint = 'Cert:\LocalMachine\Remote Desktop\'+((Get-ChildItem -Path 'Cert:\LocalMachine\Remote Desktop\').thumbprint)
Remove-Item -Path $RdpCertThumbprint
Stop-Service -Name "SessionEnv"
Start-Service -Name "SessionEnv"
```

Si vous ne pouvez pas renouveler le certificat, suivez ces étapes pour essayer de supprimer le certificat :

1. Sur une autre machine virtuelle du même réseau virtuel, ouvrez la zone **Exécuter**, tapez **mmc**, puis appuyez sur **OK**. 

2. Dans le menu **Fichier**, sélectionnez **Ajouter/Supprimer un composant logiciel enfichable**.

3. Dans la liste **Composants logiciels enfichables disponibles**, sélectionnez **Certificats**, puis sélectionnez **Ajouter**.

4. Sélectionnez **Compte d’ordinateur**, puis sélectionnez **Suivant**.

5. Sélectionnez **Un autre ordinateur**, puis ajoutez l’adresse IP de la machine virtuelle qui présente des problèmes.
   >[!Note]
   >Essayez d’utiliser le réseau interne pour éviter de recourir à une adresse IP virtuelle.

6. Sélectionnez **Terminer**, puis **OK**.

   ![Sélectionner l’ordinateur](./media/event-id-troubleshoot-vm-rdp-connecton/select-computer.png)

7. Développez les certificats, accédez au dossier Bureau à distance\Certificats, cliquez avec le bouton droit sur le certificat, puis sélectionnez **Supprimer**.

8. Redémarrez le service Configuration Bureau à distance :

   ```cmd
   net stop SessionEnv
   net start SessionEnv
   ```

   >[!Note]
   >À ce stade, si vous actualisez le magasin à partir de la console MMC, le certificat s’affiche à nouveau. 

Réessayez d’accéder à la machine virtuelle en utilisant le protocole RDP.

#### <a name="update-tlsssl-certificate"></a>Mettre à jour le certificat TLS/SSL

Si vous configurez la machine virtuelle pour qu’elle utilise un certificat TLS/SSL, exécutez la commande suivante pour obtenir l’empreinte. Vérifiez ensuite si elle est identique à l’empreinte du certificat :

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v SSLCertificateSHA1Hash
```

Si ce n’est pas le cas, modifiez l’empreinte :

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v SSLCertificateSHA1Hash /t REG_BINARY /d <CERTIFICATE THUMBPRINT>
```

Vous pouvez également essayer de supprimer la clé, de sorte que le protocole RDP utilise le certificat auto-signé pour RDP :

```cmd
reg delete "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v SSLCertificateSHA1Hash
```

## <a name="scenario-2"></a>Scénario 2

### <a name="event-log"></a>Journal des événements

Dans une instance CMD, exécutez les commandes suivantes pour vérifier si l’événement d’erreur SCHANNEL 36871 a été enregistré dans le journal système au cours des dernières 24 heures :

```cmd
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Schannel'] and EventID=36871 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
```

**Nom du journal :**      Système <br />
**Source :**        Schannel <br />
**Date :**          — <br />
**ID d’événement :**      36871 <br />
**Catégorie de tâche :** None <br />
**Niveau :**         Error <br />
**Mots clés :**       <br />
**Utilisateur :**          SYSTEM <br />
**Ordinateur :**      *ordinateur* <br />
**Description :** une erreur irrécupérable s’est produite lors de la création des informations d’identification du serveur TLS. L’état d’erreur interne est 10013.
 
### <a name="cause"></a>Cause

Ce problème est dû aux stratégies de sécurité. Lorsque des versions antérieures de TLS (par exemple, 1.0) sont désactivées, l’accès RDP échoue.

### <a name="resolution"></a>Résolution

Le protocole RDP utilise TLS 1.0 en tant que protocole par défaut. Toutefois, le protocole peut être modifié et le protocole TLS 1.1 utilisé (nouvelle norme).

Pour résoudre ce problème, consultez [Résoudre les erreurs d’authentification lorsque vous utilisez le protocole RDP pour vous connecter à une machine virtuelle Azure](troubleshoot-authentication-error-rdp-vm.md#tls-version).

## <a name="scenario-3"></a>Scénario 3

Si vous avez installé le rôle de **service Broker pour les connexions Bureau à distance** sur la machine virtuelle, vérifiez si l’événement 2056 ou 1296 s’est produit au cours des dernières 24 heures. Dans une instance CMD, exécutez les commandes suivantes : 

```cmd
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name=' Microsoft-Windows-TerminalServices-SessionBroker '] and EventID=2056 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name=' Microsoft-Windows-TerminalServices-SessionBroker-Client '] and EventID=1296 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
```

**Nom du journal :**      Microsoft-Windows-TerminalServices-SessionBroker/Operational <br />
**Source :**        Microsoft-Windows-TerminalServices-SessionBroker <br />
**Date :**          *heure* <br />
**ID d’événement :**      2056 <br />
**Catégorie de tâche :** (109) <br />
**Niveau :**         Error <br />
**Mots clés :**       <br />
**Utilisateur :**          SERVICE RÉSEAU <br />
**Ordinateur :**      *nom de domaine complet de l’ordinateur* <br />
**Description :** la description de l’ID d’événement 2056 provenant de la source Microsoft-Windows-TerminalServices-SessionBroker est introuvable. Le composant qui déclenche cet événement n’est pas installé sur votre ordinateur local, ou l’installation est endommagée. Vous pouvez installer ou réparer le composant sur l’ordinateur local. <br />
Si l’événement provient d’un autre ordinateur, les informations d’affichage ont dû être enregistrées avec l’événement. <br />
Les informations suivantes ont été incluses avec l’événement : <br />
NULL <br />
NULL <br />
L’ouverture de session sur la base de données a échoué.

**Nom du journal :**      Microsoft-Windows-TerminalServices-SessionBroker-Client/Operational <br />
**Source :**        Microsoft-Windows-TerminalServices-SessionBroker-Client <br />
**Date :**          *heure* <br />
**ID d’événement :**      1296 <br />
**Catégorie de tâche :** (104) <br />
**Niveau :**         Error <br />
**Mots clés :**       <br />
**Utilisateur :**          SERVICE RÉSEAU <br />
**Ordinateur :**      *nom de domaine complet de l’ordinateur* <br />
**Description :** la description de l’ID d’événement 1296 provenant de la source Microsoft-Windows-TerminalServices-SessionBroker-Client est introuvable. Le composant qui déclenche cet événement n’est pas installé sur votre ordinateur local, ou l’installation est endommagée. Vous pouvez installer ou réparer le composant sur l’ordinateur local.
Si l’événement provient d’un autre ordinateur, les informations d’affichage ont dû être enregistrées avec l’événement.
Les informations suivantes ont été incluses avec l’événement :  <br />
*text* <br />
*text* <br />
Le service Broker pour les connexions Bureau à distance n’est pas prêt pour la communication RPC.

### <a name="cause"></a>Cause

Ce problème se produit parce que le nom d’hôte du serveur Broker pour les connexions Bureau à distance a été modifié et que cette modification n’est pas prise en charge. 

Le nom d’hôte comporte des entrées et des dépendances vis-à-vis de la base de données interne Windows, requise par la batterie de services Bureau à distance pour pouvoir fonctionner. La modification du nom d’hôte une fois la batterie créée provoque de nombreuses erreurs et peut entraîner l’interruption de fonctionnement du serveur Broker.

### <a name="resolution"></a>Résolution 

Pour résoudre ce problème, le rôle de service Broker pour les connexions Bureau à distance et la base de données interne Windows doivent être réinstallés.

## <a name="next-steps"></a>Étapes suivantes

[Schannel Events](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn786445(v=ws.11)) (Événements Schannel)

[Schannel SSP Technical Overview](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn786429(v=ws.11)) (Présentation technique du fournisseur SSP Schannel)

[RDP Fails with Event ID 1058 & Event 36870 with Remote Desktop Session Host Certificate & SSL Communication](https://techcommunity.microsoft.com/t5/ask-the-performance-team/bg-p/AskPerf) (Le protocole RDP échoue avec les ID d’événement 1058 et 36870 avec le certificat Hôte de session Bureau à distance et la communication SSL)

[Schannel 36872 or Schannel 36870 on a Domain Controller](/archive/blogs/instan/schannel-36872-or-schannel-36870-on-a-domain-controller) (Schannel 36872 ou Schannel 36870 sur un contrôleur de domaine)

[Event ID 1058 — Remote Desktop Services Authentication and Encryption](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ee890862(v=ws.10)) (ID d’événement 1058 : authentification et chiffrement des Services Bureau à distance)
