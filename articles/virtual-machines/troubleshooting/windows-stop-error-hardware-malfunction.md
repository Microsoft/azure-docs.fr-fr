---
title: Erreur Windows Stop - Dysfonctionnement matériel
description: Cet article explique comment résoudre les problèmes liés aux machines virtuelles Windows Server 2008 qui se bloquent avec un message d’erreur indiquant qu’il y a une défaillance matérielle.
services: virtual-machines-windows
documentationcenter: ''
author: mibufo
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
ms.date: 11/13/2020
ms.author: v-mibufo
ms.openlocfilehash: 18622d60f3a33658fadfd28c53c93a07b4b438a9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96488644"
---
# <a name="windows-stop-error---hardware-malfunction"></a>Erreur Windows Stop - Dysfonctionnement matériel

Cet article explique comment résoudre les problèmes liés aux machines virtuelles Windows Server 2008 qui se bloquent avec un message d’erreur indiquant qu’il y a une défaillance matérielle.

## <a name="symptoms"></a>Symptômes

Quand vous utilisez les [Diagnostics de démarrage](./boot-diagnostics.md) pour visualiser la capture d’écran de la machine virtuelle, vous constatez que cette capture affiche un écran bleu avec le message suivant :

**\*\*\*Défaillance matérielle**

**Contactez votre fournisseur pour obtenir de l’aide**

**\*\*\*Le système s’est arrêté\*\*\***

#### <a name="blue-screen"></a>Écran bleu

![La capture d’écran montre l’écran bleu indiquant un incident dû à une défaillance matérielle.](media/windows-stop-error-hardware-malfunction/windows-stop-error-hardware-malfunction-1.png)

#### <a name="serial-console"></a>Console série

![La capture d’écran montre le message « Défaillance matérielle » sur la fonctionnalité de la console série si la console série a été activée.](media/windows-stop-error-hardware-malfunction/windows-stop-error-hardware-malfunction-2.png)

## <a name="cause"></a>Cause

Cet écran s’affiche lorsque le système d’exploitation invité n’a pas été configuré correctement et qu’une interruption non masquable (NMI) a été envoyée. Le message d’erreur indique qu’un programme en mode noyau a généré une exception qui n’a pas été interceptée par le gestionnaire. Vous pouvez identifier l’exception qui a été générée en collectant une image mémoire.

## <a name="solution"></a>Solution

### <a name="process-overview"></a>Vue d’ensemble du processus 

1. Configurer la clé de Registre d’interruption non masquable (NMI) 
2. Créer une machine virtuelle de réparation et y accéder 
3. Activer la console série et la collecte de l’image mémoire 
4. Regénérer la machine virtuelle 

### <a name="set-up-the-non-maskable-interrupt-nmi-registry-key"></a>Configurer la clé de Registre d’interruption non masquable (NMI)

1. À l’aide du portail Azure, redémarrez la machine virtuelle afin que le système d’exploitation invité démarre normalement. 
2. Une fois qu’un certain accès à la machine virtuelle a été restauré, ouvrez une invite de commandes avec élévation de privilèges (exécuter en tant qu’administrateur). 
3. Configurez la clé de Registre NMI à l’aide de la commande suivante :

    ```
    REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f
    ```
    [Afficher des informations supplémentaires sur la commande REG ADD](/windows-server/administration/windows-commands/reg-add)
4. *(Facultatif)* Configurer la collecte d’images mémoire :

    ```
    REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f  
    REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f 

    ```
5. *(Facultatif)* Configurer l’accès de la console série :

    ```
    BCDEDIT /ems {current} on, or bcdedit /ems '{current}' on if you are using PowerShell
    BCDEDIT /emssettings EMSPORT:1 EMSBAUDRATE:115200 
    ```
    [Afficher des informations supplémentaires sur la commande BCDEDIT](/windows-server/administration/windows-commands/bcdedit)
6. Redémarrez la machine virtuelle à l’aide de la commande suivante :

    ```
    SHUTDOWN /r /t 0 /f 
    ```
    [Afficher des informations supplémentaires sur la commande SHUTDOWN](/windows-server/administration/windows-commands/shutdown)

> [!IMPORTANT]
> Le problème doit maintenant être résolu !

> [!NOTE]
> Après avoir redémarré, testez votre machine virtuelle pour vous assurer qu’elle fonctionne comme d’habitude. Si les problèmes persistent, vous pouvez passer à la section suivante pour obtenir des instructions supplémentaires.

> [!TIP]
> Il est recommandé de configurer la clé de Registre d’interruption non masquable (NMI) dans la section ci-dessus. Toutefois, si votre machine virtuelle n’a pas démarré normalement par la suite, les modifications prévues au registre du système d’exploitation invité n’ont peut-être pas été effectuées. Si c’est le cas, vous pouvez suivre les instructions ci-dessous pour ajouter manuellement les paramètres du registre.

### <a name="create-and-access-a-repair-vm"></a>Créer une machine virtuelle de réparation et y accéder

1. Effectuez les étapes 1 à 3 des [commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une machine virtuelle de réparation.
2. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="enable-serial-console-and-memory-dump-collection"></a>Activer la console série et la collecte de l’image mémoire

Avant de régénérer la machine virtuelle, il est recommandé d’activer la collecte des images mémoire et la console série. Pour ce faire, exécutez le script suivant : 

1. Ouvrez une session Invite de commande avec élévation de privilèges (exécuter en tant qu’administrateur). 
2. Répertoriez les données du Store BCD et déterminez l’identificateur du chargeur de démarrage, que vous utiliserez à l’étape suivante. 
    1. Pour une machine virtuelle de 1ère génération, entrez la commande suivante et notez l’identificateur indiqué : 
 
        ```
        bcdedit /store <BOOT PARTITON>:\boot\bcd /enum
        ```
        Dans la commande, remplacez `<BOOT PARTITON>` par la lettre de la partition du disque joint qui contient le dossier de démarrage. 

        ![La capture d’écran montre le résultat du listing du magasin BCD dans une machine virtuelle de 1re génération, qui répertorie le numéro d’identificateur sous le chargeur d’amorçage Windows.](media/windows-stop-error-hardware-malfunction/windows-stop-error-hardware-malfunction-3.png)
    2. Pour une machine virtuelle de 2e génération, entrez la commande suivante et notez l’identificateur indiqué :
    
        ```
        BCDEDIT /store <LETTER OF THE EFI SYSTEM PARTITION>:EFI\Microsoft\boot\bcd /enum 
        ```
        * Dans la commande, remplacez `<LETTER OF THE EFI SYSTEM PARTITION>` par la lettre de la partition système EFI.
        * Il peut être utile de lancer la console de gestion des disques pour identifier la partition système appropriée nommée *Partition système EFI*.
        * L’identificateur peut être un GUID unique ou la valeur par défaut *bootmgr*.
3. Exécutez les commandes suivantes pour activer la Console série :

    ```
    BCDEDIT /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON  
    BCDEDIT /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200 

    ```
    * Dans la commande, remplacez `<VOLUME LETTER WHERE THE BCD FOLDER IS>` par la lettre du dossier BCD.
    * Dans la commande, remplacez `<BOOT LOADER IDENTIFIER>` par l’identificateur que vous avez trouvé à l’étape précédente.
4. Vérifiez que l’espace disponible sur le disque du système d’exploitation est supérieur à la taille de la mémoire (RAM) sur la machine virtuelle. 
    1. S’il n’y a pas suffisamment d’espace sur le disque du système d’exploitation, vous devez modifier l’emplacement où le fichier d’image mémoire sera créé. Au lieu de créer le fichier sur le disque du système d’exploitation, vous pouvez y référer à partir de n’importe quel autre disque de données joint à la machine virtuelle qui dispose de suffisamment d’espace libre. Pour changer l’emplacement, remplacez **%SystemRoot%** par la lettre de lecteur (par exemple, **F:** ) du disque de données dans les commandes répertoriées ci-dessous. 
    2. Entrez les commandes ci-dessous (configuration de vidage suggérée) :

    **Charger la ruche du Registre à partir du disque de système d’exploitation endommagé :**

    ```
    REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM
    ```

    **Activer sur ControlSet001 :**

    ```
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f 
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f 
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f 
    ```

    **Activer sur ControlSet002 :**

    ```
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f 
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f 
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f 
    ```

    **Décharger le disque de système d’exploitation corrompu :**

    ```
    REG UNLOAD HKLM\BROKENSYSTEM
    ```
### <a name="rebuild-the-virtual-machine"></a>Régénérer la machine virtuelle

* Utilisez [l’étape 5 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour régénérer la machine virtuelle.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Résoudre les erreurs de démarrage de machine virtuelle Azure](./boot-error-troubleshoot.md)