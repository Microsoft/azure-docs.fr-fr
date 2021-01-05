---
title: Le démarrage de la machine virtuelle est bloqué sur « Préparation de Windows. N’éteignez l’ordinateur » dans Azure | Microsoft Docs
description: Présentation des étapes de résolution du problème de démarrage de machine virtuelle bloqué sur « Préparation de Windows. N’éteignez pas l’ordinateur. »
services: virtual-machines-windows
documentationcenter: ''
author: Deland-Han
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: article
ms.date: 09/18/2018
ms.author: delhan
ms.openlocfilehash: d7e56fe36af3d841cfd888dd6c1bf05502837cdd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87079840"
---
# <a name="vm-startup-is-stuck-on-getting-windows-ready-dont-turn-off-your-computer-in-azure"></a>Le démarrage de la machine virtuelle est bloqué sur « Préparation de Windows. N’éteignez pas l’ordinateur. » dans Azure

Cet article décrit les écrans « Préparation » et « Préparation de Windows » que vous pouvez rencontrer quand vous démarrez une machine virtuelle Windows dans Microsoft Azure. Il fournit les étapes pour vous aider à collecter des données pour un ticket de support.

 

## <a name="symptoms"></a>Symptômes

Une machine virtuelle Windows ne démarre pas. Quand vous utilisez les **Diagnostics de démarrage** pour obtenir la capture d’écran de la machine virtuelle, vous pouvez constater que la machine virtuelle affiche le message « Préparation » ou « Préparation de Windows ».

![Exemple de message pour Windows Server 2012 R2](./media/troubleshoot-vm-configure-update-boot/message1.png)

![Exemple de message](./media/troubleshoot-vm-configure-update-boot/message2.png)

## <a name="cause"></a>Cause :

Ce problème se produit généralement lorsque le serveur effectue le redémarrage final après un changement de configuration. Le changement de configuration peut résulter de mises à jour de Windows ou de modifications des rôles ou fonctionnalité du serveur. Pour Windows Update, si l’ampleur des mises à jour est importante, le système d’exploitation a besoin de plus de temps pour reconfigurer les modifications.

## <a name="collect-an-os-memory-dump"></a>Collecter un vidage mémoire du système d’exploitation

Si le problème n’est pas résolu après que vous avez attendu que les modifications soient traitées, vous devez recueillir un fichier d’image mémoire et contacter le support technique. Pour collecter le fichier d’image, effectuez les étapes suivantes :

### <a name="attach-the-os-disk-to-a-recovery-vm"></a>Attachez le disque du système d’exploitation à une machine virtuelle de récupération

1. Prenez un instantané du disque du système d’exploitation de la machine virtuelle affectée en guise de sauvegarde. Pour plus d’informations, consultez [Créer un instantané](../windows/snapshot-copy-managed-disk.md).
2. [Attachez le disque du système d’exploitation à une machine virtuelle de récupération](./troubleshoot-recovery-disks-portal-windows.md).
3. Connectez-vous à la machine virtuelle de récupération à l’aide du Bureau à distance. 
4. Si le disque du système d’exploitation est chiffré, vous devez désactiver le chiffrement avant de passer à l’étape suivante. Pour plus d’informations, consultez [Déchiffrer le disque de système d’exploitation chiffré sur la machine virtuelle qui ne peut pas démarrer](troubleshoot-bitlocker-boot-error.md#solution).

### <a name="locate-dump-file-and-submit-a-support-ticket"></a>Rechercher le fichier d’image et envoyer un ticket de support

1. Sur la machine virtuelle de récupération, accédez au dossier Windows dans le disque du système d’exploitation attaché. Si la lettre de lecteur qui est affectée au disque du système d’exploitation attaché est F, vous devez accéder à F:\Windows.
2. Recherchez le fichier memory.dmp, puis [envoyez un ticket de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) avec le fichier d’image. 

Si le fichier d’image est introuvable, passez à l’étape suivante pour activer le journal de vidage et la console série.

### <a name="enable-dump-log-and-serial-console"></a>Activer le journal de vidage et la console série

Pour activer le journal de vidage et la console série, exécutez le script suivant.

1. Ouvrez une session Invite de commande avec élévation de privilèges (Exécuter en tant qu’administrateur).
2. Exécutez le script suivant :

    Dans ce script, nous partons du principe que la lettre de lecteur qui est affectée au disque du système d’exploitation attaché est F. Remplacez-la par la valeur appropriée dans votre machine virtuelle.

    ```powershell
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

    REM Enable Serial Console
    bcdedit /store F:\boot\bcd /set {bootmgr} displaybootmenu yes
    bcdedit /store F:\boot\bcd /set {bootmgr} timeout 5
    bcdedit /store F:\boot\bcd /set {bootmgr} bootems yes
    bcdedit /store F:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON
    bcdedit /store F:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200

    REM Suggested configuration to enable OS Dump
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

    reg unload HKLM\BROKENSYSTEM
    ```

    1. Vérifiez que l’espace est suffisant sur le disque pour allouer autant de mémoire que de RAM, ce qui dépend de la taille que vous sélectionnez pour cette machine virtuelle.
    2. Si l’espace n’est pas suffisant ou qu’il s’agit d’une machine virtuelle de grande taille (série G, GS ou E), vous pouvez ensuite modifier l’emplacement où ce fichier sera créé et le référencer sur n’importe quel autre disque de données qui est attaché à la machine virtuelle. Pour ce faire, vous devez modifier la clé suivante :
    
        ```console
        reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

        REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f
        REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f

        reg unload HKLM\BROKENSYSTEM
        ```

3. [Détachez le disque du système d’exploitation et rattachez-le à la machine virtuelle affectée](./troubleshoot-recovery-disks-portal-windows.md).
4. Démarrez la machine virtuelle et accédez à la console série.
5. Sélectionnez **Envoyer une interruption non masquable (NMI)** pour déclencher l’image mémoire.
    ![image sur l’emplacement où envoyer une interruption non masquable](./media/troubleshoot-vm-configure-update-boot/run-nmi.png)
6. Reconnectez le disque du système d’exploitation à une machine virtuelle de récupération, puis recueillez le fichier d’image mémoire.

## <a name="contact-microsoft-support"></a>Contact Microsoft support

Après avoir recueilli le fichier de vidage mémoire, contactez le [Support Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour lui demander d’analyser la cause racine.
