---
title: Erreurs dans un écran bleu lors du démarrage d’une machine virtuelle Azure | Microsoft Docs
description: Découvrez comment résoudre le problème d’apparition d’une erreur dans un écran bleu lors du démarrage | Microsoft Docs
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 09/28/2018
ms.author: genli
ms.openlocfilehash: b382efc4d283d64ce0f833bde9104fa2e3bc973a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87088543"
---
# <a name="windows-shows-blue-screen-error-when-booting-an-azure-vm"></a>Windows affiche une erreur dans un écran bleu lors du démarrage d’une machine virtuelle Azure
Cet article décrit les erreurs dans un écran bleu que vous pouvez rencontrer quand vous démarrez une machine virtuelle Windows dans Microsoft Azure. Il fournit les étapes pour vous aider à collecter des données pour un ticket de support. 


## <a name="symptom"></a>Symptôme 

Une machine virtuelle Windows ne démarre pas. Quand vous vérifiez les captures d’écran de démarrage de la fenêtre [Diagnostics de démarrage](./boot-diagnostics.md), vous voyez l’un des messages d’erreur suivants dans un écran bleu :

- Votre PC a rencontré un problème et doit redémarrer. Nous collectons simplement quelques informations sur l’erreur et vous pourrez ensuite redémarrer.
- Votre PC a rencontré un problème et doit redémarrer.

Cette section répertorie les messages d’erreur pouvant apparaître lors de la gestion des machines virtuelles :

## <a name="cause"></a>Cause :

Plusieurs raisons peuvent expliquer l’affichage d’une erreur d’arrêt. Les causes les plus courantes sont :

- Problème au niveau d’un pilote
- Mémoire ou fichier système endommagé
- Une application accède à un secteur interdit de la mémoire

## <a name="collect-memory-dump-file"></a>Collecter le fichier d’image mémoire

Pour résoudre ce problème, vous devez d’abord collecter le fichier d’image du plantage et contacter le support technique muni de ce fichier. Pour collecter le fichier d’image, effectuez les étapes suivantes :

### <a name="attach-the-os-disk-to-a-recovery-vm"></a>Attachez le disque du système d’exploitation à une machine virtuelle de récupération

1. Prenez un instantané du disque du système d’exploitation de la machine virtuelle affectée en guise de sauvegarde. Pour plus d’informations, consultez [Créer un instantané](../windows/snapshot-copy-managed-disk.md).
2. [Attachez le disque du système d’exploitation à une machine virtuelle de récupération](./troubleshoot-recovery-disks-portal-windows.md). 
3. Connectez-vous à la machine virtuelle de récupération à l’aide du Bureau à distance.

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

    ```config-reg
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f

    reg unload HKLM\BROKENSYSTEM
    ```

3. [Détachez le disque du système d’exploitation et rattachez-le à la machine virtuelle affectée](./troubleshoot-recovery-disks-portal-windows.md).
4. Démarrez la machine virtuelle pour reproduire le problème, et un fichier d’image est généré.
5. Attachez le disque du système d’exploitation à une machine virtuelle de récupération, collectez le fichier d’image, puis [envoyez un ticket de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) avec le fichier d’image.
