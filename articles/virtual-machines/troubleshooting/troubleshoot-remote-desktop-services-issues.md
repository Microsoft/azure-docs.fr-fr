---
title: Les Services Bureau à distance ne démarrent pas sur une machine virtuelle Azure | Microsoft Docs
description: Découvrez comment résoudre les problèmes des Services Bureau à distance en cas de connexion à une machine virtuelle | Microsoft Docs
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/23/2018
ms.author: genli
ms.openlocfilehash: 0b00785fed7708986885e9da9102e8f1b4fd4539
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86508880"
---
# <a name="remote-desktop-services-isnt-starting-on-an-azure-vm"></a>Les Services Bureau à distance ne démarrent pas sur une machine virtuelle Azure

Cet article explique comment résoudre les problèmes de connexion à une machine virtuelle Azure quand les Services Bureau à distance, ou TermService, ne démarrent pas ou échouent au démarrage.


## <a name="symptoms"></a>Symptômes

Lorsque vous essayez de vous connecter à une machine virtuelle, vous rencontrez les situations suivantes :

- La capture d’écran de la VM montre que le système d’exploitation est entièrement chargé et attend des informations d’identification.

    ![Capture d’écran de l’état de la machine virtuelle](./media/troubleshoot-remote-desktop-services-issues/login-page.png)

- Vous consultez à distance les journaux des événements dans la machine virtuelle avec l’observateur d’événements. Il apparaît que les Services Bureau à distance, TermService, ne démarrent pas ou échouent au démarrage. Voici un exemple de journal :

    **Nom du journal** :      Système </br>
    **Source** :        Gestionnaire de contrôle des services </br>
    **Date** :          16/12/2017 11:19:36 AM</br>
    **ID d’événement** :      7022</br>
    **Catégorie de tâche** : None</br>
    **Niveau** :         Error</br>
    **Mots clés** :      Classique</br>
    **Utilisateur** :          N/A</br>
    **Ordinateur** :      vm.contoso.com</br>
    **Description** : Le service Services Bureau à distance s’est bloqué au démarrage. 

    Vous pouvez aussi utiliser la fonctionnalité Console d’accès série pour rechercher ces erreurs en exécutant la requête suivante : 

    ```console
   wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Service Control Manager'] and EventID=7022 and TimeCreated[timediff(@SystemTime) <= 86400000]]]" | more
    ```

## <a name="cause"></a>Cause
 
Ce problème se produit, car les Services Bureau à distance ne s’exécutent pas sur la machine virtuelle. La cause peut varier selon les scénarios suivants : 

- Le service TermService est défini sur **Désactivé**. 
- Le service TermService est en bloqué ou suspendu. 
- Le service TermService ne démarre pas raison d’une configuration incorrecte.

## <a name="solution"></a>Solution

Pour résoudre ce problème, utilisez la Console série. Vous pouvez également [réparer la machine virtuelle en mode hors connexion](#repair-the-vm-offline) en attachant le disque de système d’exploitation de la machine virtuelle à une machine virtuelle de récupération.

### <a name="use-serial-console"></a>Utiliser la console série

1. Accédez à la [console série](serial-console-windows.md) en sélectionnant **Support et dépannage** > **Console série**. Si la fonctionnalité est activée sur la machine virtuelle, vous pouvez connecter la machine.

2. Créez un canal pour une instance CMD. Entrez **CMD** pour démarrer le canal et en obtenir le nom.

3. Basculez vers le canal qui exécute l’instance CMD. Dans ce cas, il devrait s’agir du canal 1 :

   ```
   ch -si 1
   ```

4. Sélectionnez de nouveau **Entrée** et entrez un nom d’utilisateur et un mot de passe valides ainsi qu’un ID local ou de domaine pour la machine virtuelle.

5. Interrogez l’état du service TermService :

   ```
   sc query TermService
   ```

6. Si l’état du service indique **Arrêté**, essayez de le démarrer :

    ```
    sc start TermService
     ``` 

7. Interrogez à nouveau le service pour vérifier qu’il a réussi à démarrer :

   ```
   sc query TermService
   ```
8. Si le service échoue au démarrage, suivez la solution qui correspond à l’erreur que vous avez reçue :

    |  Error |  Suggestion |
    |---|---|
    |5- ACCÈS REFUSÉ |Voir [Le service TermService est arrêté en raison d’une erreur d’accès refusé](#termservice-service-is-stopped-because-of-an-access-denied-problem). |
    |1053 - ERROR_SERVICE_REQUEST_TIMEOUT  |Voir [Le service TermService est désactivé](#termservice-service-is-disabled).  |  
    |1058 - ERROR_SERVICE_DISABLED  |Voir [Le service TermService se bloque](#termservice-service-crashes-or-hangs).  |
    |1059 - ERROR_CIRCULAR_DEPENDENCY |[Contactez le support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre rapidement votre problème.|
    |1067 - ERROR_PROCESS_ABORTED  |Voir [Le service TermService se bloque](#termservice-service-crashes-or-hangs).  |
    |1068 - ERROR_SERVICE_DEPENDENCY_FAIL|[Contactez le support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre rapidement votre problème.|
    |1069 - ERROR_SERVICE_LOGON_FAILED  |Voir [Le service TermService échoue en raison d’un échec d’ouverture de session](#termservice-service-fails-because-of-logon-failure). |
    |1070 - ERROR_SERVICE_START_HANG   | Voir [Le service TermService se bloque](#termservice-service-crashes-or-hangs). |
    |1077 - ERROR_SERVICE_NEVER_STARTED   | Voir [Le service TermService est désactivé](#termservice-service-is-disabled).  |
    |1079 - ERROR_DIFERENCE_SERVICE_ACCOUNT   |[Contactez le support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre rapidement votre problème. |
    |1753   |[Contactez le support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre rapidement votre problème.   |
    
#### <a name="termservice-service-is-stopped-because-of-an-access-denied-problem"></a>Le service TermService est arrêté en raison d’un problème d’accès refusé

1. Connectez-vous à la [console série](serial-console-windows.md) et ouvrez une instance PowerShell.
2. Téléchargez l’outil Process Monitor en exécutant le script suivant :

   ```
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

4. Reproduisez le problème en démarrant le service qui indique **Accès refusé** : 

   ```
   sc start TermService 
   ```

   Quand il échoue, mettez fin à la trace de Process Monitor :

   ```   
   procmon /Terminate 
   ```

5. Récupérez le fichier **c:\temp\ProcMonTrace.PML** :

    1. [Attachez un disque de données à la machine virtuelle](../windows/attach-managed-disk-portal.md
).
    2. Utilisez la Console série pour copier le fichier sur le nouveau disque. Par exemple : `copy C:\temp\ProcMonTrace.PML F:\`. Dans cette commande, F est la lettre de lecteur du disque de données attaché.
    3. Détachez le lecteur de données et attachez-le à une machine virtuelle fonctionnelle sur laquelle Process Monitor ubstakke est installé.

6. Ouvrez **ProcMonTrace.PML** en utilisant Process Monitor sur la machine virtuelle. Ensuite, filtrez sur **Résultat est ACCÈS REFUSÉ** comme dans la capture d’écran suivante :

    ![Filtrer par résultat dans Process Monitor](./media/troubleshoot-remote-desktop-services-issues/process-monitor-access-denined.png)

 
6. Corrigez les clés de Registre, les dossiers ou les fichiers qui se trouvent dans la sortie. En règle générale, ce problème est lié au fait que le compte de connexion utilisé dans le service ne dispose pas de l’autorisation d’ACL nécessaire pour accéder à ces objets. Pour connaître l’autorisation d’ACL adaptée au compte de connexion, vérifiez sur une machine virtuelle saine. 

#### <a name="termservice-service-is-disabled"></a>Le service TermService est désactivé

1. Rétablissez la valeur de démarrage par défaut du service :

   ```
   sc config TermService start= demand 
   ```

2. Démarrez le service :

   ```
   sc start TermService
   ```

3. Interrogez de nouveau l’état du service pour vérifier qu’il est en cours d’exécution :

   ```
   sc query TermService 
   ```

4. Essayez de vous connecter à la machine virtuelle à l’aide du Bureau à distance.

#### <a name="termservice-service-fails-because-of-logon-failure"></a>Le service TermService échoue en raison d’un échec d’ouverture de session

1. Ce problème se produit lorsque le compte de démarrage de ce service a été modifié. Rétablissez sa configuration par défaut : 

    ```console
    sc config TermService obj= 'NT Authority\NetworkService'
    ```

2. Démarrez le service :

    ```console
    sc start TermService
    ```

3. Essayez de vous connecter à la machine virtuelle à l’aide du Bureau à distance.

#### <a name="termservice-service-crashes-or-hangs"></a>Le service TermService se bloque
1. Si l’état du service est bloqué sur **Démarrage** ou sur **Arrêt**, essayez de l’arrêter : 

    ```console
    sc stop TermService
    ```

2. Isolez le service sur son propre conteneur « svchost » :

    ```console
    sc config TermService type= own
    ```

3. Démarrez le service :

    ```console
    sc start TermService
    ```

4. Si le service échoue toujours au démarrage, [contactez le support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

### <a name="repair-the-vm-offline"></a>Réparer la machine virtuelle en mode hors connexion

#### <a name="attach-the-os-disk-to-a-recovery-vm"></a>Attachez le disque du système d’exploitation à une machine virtuelle de récupération

1. [Attachez le disque du système d’exploitation à une machine virtuelle de récupération](./troubleshoot-recovery-disks-portal-windows.md).
2. Établissez une connexion Bureau à distance avec la machine virtuelle de récupération. Vérifiez que le disque attaché est marqué comme étant **En ligne** dans la console Gestion des disques. Notez la lettre de lecteur affectée au disque de système d’exploitation attaché.
3. Ouvrez une instance d’invite de commande avec élévation de privilèges (**Exécuter en tant qu’administrateur**). Ensuite, exécutez le script suivant. Supposons que la lettre de lecteur affectée au disque de système d’exploitation attaché est **F**. Remplacez-la par la valeur correspondant à votre machine virtuelle. 

   ```
   reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv
        
   REM Set default values back on the broken service 
   reg add "HKLM\BROKENSYSTEM\ControlSet001\services\TermService" /v start /t REG_DWORD /d 3 /f
   reg add "HKLM\BROKENSYSTEM\ControlSet001\services\TermService" /v ObjectName /t REG_SZ /d "NT Authority\NetworkService“ /f
   reg add "HKLM\BROKENSYSTEM\ControlSet001\services\TermService" /v type /t REG_DWORD /d 16 /f
   reg add "HKLM\BROKENSYSTEM\ControlSet002\services\TermService" /v start /t REG_DWORD /d 3 /f
   reg add "HKLM\BROKENSYSTEM\ControlSet002\services\TermService" /v ObjectName /t REG_SZ /d "NT Authority\NetworkService" /f
   reg add "HKLM\BROKENSYSTEM\ControlSet002\services\TermService" /v type /t REG_DWORD /d 16 /f
   ```

4. [Détachez le disque de système d’exploitation et recréez la machine virtuelle](./troubleshoot-recovery-disks-portal-windows.md). Ensuite, vérifiez que le problème est résolu.

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contacter le support technique

Si vous avez toujours besoin d’aide, [contactez le support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre votre problème.
