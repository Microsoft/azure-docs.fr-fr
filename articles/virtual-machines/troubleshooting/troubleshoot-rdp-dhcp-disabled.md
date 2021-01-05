---
title: Impossible de se connecter à distance aux machines virtuelles Azure car DHCP est désactivé | Microsoft Docs
description: Découvrez comment résoudre un problème RDP causé par la désactivation d’un service client DHCP dans Microsoft Azure.| Microsoft Docs
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 11/13/2018
ms.author: genli
ms.openlocfilehash: 945f8896a844e7a73107df44d03abc7290f4e3fc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86999134"
---
#  <a name="cannot-rdp-to-azure-virtual-machines-because-the-dhcp-client-service-is-disabled"></a>Impossible d’établir une connexion RDP à des machines virtuelles Azure car le service client DHCP est désactivé

Cet article décrit un problème qui vous empêche de vous connecter à distance à des machines virtuelles (VM) Windows Azure une fois que le service client DHCP est désactivé sur la machine virtuelle.


## <a name="symptoms"></a>Symptômes
Vous ne pouvez pas établir une connexion RDP à une machine virtuelle dans Azure, car le service client DHCP est désactivé sur la machine virtuelle. Lorsque vous vérifiez la capture d’écran dans les [diagnostics de démarrage](../troubleshooting/boot-diagnostics.md) du portail Azure, vous voyez la machine virtuelle démarrer normalement et attendre les identifiants dans l’écran de connexion. Vous consultez à distance les journaux des événements dans la machine virtuelle avec l’observateur d’événements. Vous voyez que le service client DHCP n’est pas démarré ou ne parvient pas à démarrer. Voici un exemple de journal :

**Nom du journal** : Système </br>
**Source** : Gestionnaire de contrôle des services </br>
**Date** : 12/16/2015 11:19:36 AM </br>
**ID d’événement** : 7022 </br>
**Catégorie de tâche** : None </br>
**Niveau** : Error </br>
**Mots clés** : Classique</br>
**Utilisateur** : N/A </br>
**Ordinateur** : myvm.cosotos.com</br>
**Description** : Le service client DHCP se bloque au démarrage.</br>

Pour les machines virtuelles Resource Manager, vous pouvez utiliser la fonctionnalité Console d’accès série pour rechercher les journaux des événements 7022 à l’aide de la commande suivante :

```console
wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Service Control Manager'] and EventID=7022 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
```

Pour les machines virtuelles classiques, vous devez travailler en mode HORS CONNEXION et collecter les journaux d’activité manuellement.

## <a name="cause"></a>Cause

Le service client DHCP ne s’exécute pas sur la machine virtuelle.

> [!NOTE]
> Cet article s’applique uniquement au service client DHCP et pas au serveur DHCP.

## <a name="solution"></a>Solution

Avant de suivre cette procédure, faites en sauvegarde en prenant un instantané du disque du système d’exploitation de la machine virtuelle affectée. Pour plus d’informations, consultez [Créer un instantané](../windows/snapshot-copy-managed-disk.md).

Pour résoudre ce problème, utilisez le contrôle série pour activer DHCP ou [réinitialisez l’interface réseau](reset-network-interface.md) de la machine virtuelle.

### <a name="use-serial-control"></a>Utiliser le contrôle série

1. Connectez-vous à la [console série et ouvrez une instance CMD](serial-console-windows.md#use-cmd-or-powershell-in-serial-console).
). Si la console série n’est pas activée sur votre machine virtuelle, consultez [Réinitialiser l’interface réseau](reset-network-interface.md).
2. Vérifiez si le protocole DHCP est désactivé sur l’interface réseau :

    ```console
    sc query DHCP
    ```

3. Si DHCP est arrêté, essayez de démarrer le service.

    ```console
    sc start DHCP
    ```

4. Interrogez à nouveau le service pour vous assurer qu’il a réussi à démarrer.

    ```console
    sc query DHCP
    ```

    Essayez de vous connecter à la machine virtuelle et regardez si le problème est résolu.
5. Si le service ne démarre pas, utilisez la solution appropriée suivante en fonction du message d’erreur que vous avez reçu :

    | Error  |  Solution |
    |---|---|
    | 5- ACCÈS REFUSÉ  | Consultez [Le service client DHCP est arrêté en raison d’une erreur d’accès refusé](#dhcp-client-service-is-stopped-because-of-an-access-denied-error).  |
    |1053 - ERROR_SERVICE_REQUEST_TIMEOUT   | Consultez [Le service client DHCP plante ou se bloque](#dhcp-client-service-crashes-or-hangs).  |
    | 1058 - ERROR_SERVICE_DISABLED  | Consultez [Le service client DHCP est désactivé](#dhcp-client-service-is-disabled).  |
    | 1059 - ERROR_CIRCULAR_DEPENDENCY  |[Contactez le support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre rapidement votre problème.   |
    | 1067 - ERROR_PROCESS_ABORTED |Consultez [Le service client DHCP plante ou se bloque](#dhcp-client-service-crashes-or-hangs).   |
    |1068 - ERROR_SERVICE_DEPENDENCY_FAIL   | [Contactez le support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre rapidement votre problème.  |
    |1069 - ERROR_SERVICE_LOGON_FAILED   |  Consultez [Le service client DHCP échoue en raison d’un échec d’ouverture de session](#dhcp-client-service-fails-because-of-logon-failure) |
    | 1070 - ERROR_SERVICE_START_HANG  | Consultez [Le service client DHCP plante ou se bloque](#dhcp-client-service-crashes-or-hangs).  |
    | 1077 - ERROR_SERVICE_NEVER_STARTED  | Consultez [Le service client DHCP est désactivé](#dhcp-client-service-is-disabled).  |
    |1079 - ERROR_DIFERENCE_SERVICE_ACCOUNT   | [Contactez le support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre rapidement votre problème.  |
    |1053 | [Contactez le support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre rapidement votre problème.  |


#### <a name="dhcp-client-service-is-stopped-because-of-an-access-denied-error"></a>Le service client DHCP est arrêté en raison d’une erreur d’accès refusé

1. Connectez-vous à la [console série](serial-console-windows.md) et ouvrez une instance PowerShell.
2. Téléchargez l’outil Process Monitor en exécutant le script suivant :

   ```powershell
   remove-module psreadline
   $source = "https://download.sysinternals.com/files/ProcessMonitor.zip"
   $destination = "c:\temp\ProcessMonitor.zip"
   $wc = New-Object System.Net.WebClient
   $wc.DownloadFile($source,$destination)
   ```
3. Maintenant, lancez une trace **procmon** :

   ```
   procmon /Quiet /Minimized /BackingFile c:\temp\ProcMonTrace.PML
   ```
4. Reproduisez le problème en démarrant le service qui génère le message **Accès refusé** :

   ```
   sc start DHCP
   ```

   Quand il échoue, mettez fin à la trace de Process Monitor :

   ```
   procmon /Terminate
   ```
5. Récupérez le fichier **c:\temp\ProcMonTrace.PML** :

    1. [Attachez un disque de données à la machine virtuelle](../windows/attach-managed-disk-portal.md
).
    2. Utilisez la Console série pour copier le fichier sur le nouveau disque. Par exemple : `copy C:\temp\ProcMonTrace.PML F:\`. Dans cette commande, F est la lettre de lecteur du disque de données attaché. Remplacez la lettre appropriée par la valeur correcte.
    3. Détachez le lecteur de données et attachez-le à une machine virtuelle fonctionnelle sur laquelle Process Monitor ubstakke est installé.

6. Ouvrez **ProcMonTrace.PML** en utilisant Process Monitor sur la machine virtuelle. Ensuite, filtrez sur  **Résultat est ACCÈS REFUSÉ**, comme dans la capture d’écran suivante :

    ![Filtrer par résultat dans Process Monitor](./media/troubleshoot-remote-desktop-services-issues/process-monitor-access-denined.png)

7. Corrigez les clés de Registre, les dossiers ou les fichiers qui se trouvent dans la sortie. En règle générale, ce problème est lié au fait que le compte de connexion utilisé dans le service ne dispose pas de l’autorisation d’ACL nécessaire pour accéder à ces objets. Pour déterminer l’autorisation d’ACL adaptée au compte de connexion, vérifiez sur une machine virtuelle saine.

#### <a name="dhcp-client-service-is-disabled"></a>Le service client DHCP est désactivé

1. Rétablissez la valeur de démarrage par défaut du service :

   ```
   sc config DHCP start= auto
   ```

2. Démarrez le service :

   ```
   sc start DHCP
   ```

3. Interrogez de nouveau l’état du service pour vérifier que celui-ci est en cours d’exécution :

   ```
   sc query DHCP
   ```

4. Essayez de vous connecter à la machine virtuelle à l’aide de Bureau à distance.

#### <a name="dhcp-client-service-fails-because-of-logon-failure"></a>Le service client DHCP échoue en raison d’un échec d’ouverture de session

1. Étant donné que ce problème se produit si le compte de démarrage de ce service a été modifié, rétablissez le compte à son état par défaut :

    ```console
    sc config DHCP obj= 'NT Authority\Localservice'
    ```

2. Démarrez le service :

    ```console
    sc start DHCP
    ```

3. Essayez de vous connecter à la machine virtuelle à l’aide de Bureau à distance.

#### <a name="dhcp-client-service-crashes-or-hangs"></a>Le service client DHCP plante ou se bloque

1. Si l’état du service est bloqué sur **Démarrage** ou sur **Arrêt**, essayez d’arrêter le service :

    ```console
    sc stop DHCP
    ```

2. Isolez le service sur son propre conteneur « svchost » :

    ```console
    sc config DHCP type= own
    ```

3. Démarrez le service :

    ```console
    sc start DHCP
    ```

4. Si le service ne démarre toujours pas, [contactez le support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

### <a name="repair-the-vm-offline"></a>Réparer la machine virtuelle en mode hors connexion

#### <a name="attach-the-os-disk-to-a-recovery-vm"></a>Attachez le disque du système d’exploitation à une machine virtuelle de récupération

1. [Attachez le disque du système d’exploitation à une machine virtuelle de récupération](./troubleshoot-recovery-disks-portal-windows.md).
2. Établissez une connexion Bureau à distance avec la machine virtuelle de récupération. Vérifiez que le disque attaché est marqué comme étant **En ligne** dans la console Gestion des disques. Notez la lettre de lecteur affectée au disque de système d’exploitation attaché.
3.  Ouvrez une instance d’invite de commande avec élévation de privilèges (**Exécuter en tant qu’administrateur**). Ensuite, exécutez le script suivant. Ce script suppose que la lettre de lecteur affectée au disque de système d’exploitation attaché est **F**. Remplacez la lettre appropriée par la valeur de votre machine virtuelle.

    ```
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM

    REM Set default values back on the broken service
    reg add "HKLM\BROKENSYSTEM\ControlSet001\services\DHCP" /v start /t REG_DWORD /d 2 /f
    reg add "HKLM\BROKENSYSTEM\ControlSet001\services\DHCP" /v ObjectName /t REG_SZ /d "NT Authority\LocalService" /f
    reg add "HKLM\BROKENSYSTEM\ControlSet001\services\DHCP" /v type /t REG_DWORD /d 16 /f
    reg add "HKLM\BROKENSYSTEM\ControlSet002\services\DHCP" /v start /t REG_DWORD /d 2 /f
    reg add "HKLM\BROKENSYSTEM\ControlSet002\services\DHCP" /v ObjectName /t REG_SZ /d "NT Authority\LocalService" /f
    reg add "HKLM\BROKENSYSTEM\ControlSet002\services\DHCP" /v type /t REG_DWORD /d 16 /f

    reg unload HKLM\BROKENSYSTEM
    ```

4. [Détachez le disque de système d’exploitation et recréez la machine virtuelle](./troubleshoot-recovery-disks-portal-windows.md). Ensuite, vérifiez que le problème est résolu.

## <a name="next-steps"></a>Étapes suivantes

Si vous avez encore besoin d’aide, [contactez le support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre votre problème.
