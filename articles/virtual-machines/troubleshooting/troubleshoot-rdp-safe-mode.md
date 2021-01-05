---
title: Impossible de se connecter aux machines virtuelles Azure à distance car celles-ci démarrent en mode sans échec | Microsoft Docs
description: Découvrez comment résoudre un problème dans lequel il est impossible d’établir une connexion RDP à une machine virtuelle, car celle-ci démarre en mode sans échec. | Microsoft Docs
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
ms.openlocfilehash: d424cccc0a50198f3ca8c6c040afb87f44282d47
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86508897"
---
#  <a name="cannot-rdp-to-a-vm-because-the-vm-boots-into-safe-mode"></a>Impossible d’établir une connexion RDP à une machine virtuelle car celle-ci démarre en mode sans échec

Cet article explique comment résoudre un problème qui vous empêche de vous connecter à des machines virtuelles Windows Azure car celles-ci sont configurées pour démarrer en mode sans échec.


## <a name="symptoms"></a>Symptômes

Vous ne pouvez pas établir une connexion RDP ou d’autres connexions (par exemple, HTTP) à une machine virtuelle dans Azure, car celle-ci est configurée pour démarrer en mode sans échec. Lorsque vous consultez la capture d’écran dans les [Diagnostics de démarrage](../troubleshooting/boot-diagnostics.md) du portail Azure, vous voyez probablement que la machine virtuelle démarre normalement mais que l’interface réseau n’est pas disponible :

![Image de l’interface réseau en mode sans échec](./media/troubleshoot-rdp-safe-mode/network-safe-mode.png)

## <a name="cause"></a>Cause :

Le service RDP n’est pas disponible en mode sans échec. Seuls les principaux services et programmes système sont chargés quand la machine virtuelle démarre en mode sans échec. Cela s’applique aux deux versions du mode sans échec qui sont « Démarrage sécurisé minimale » et « Démarrage sécurisé avec une connectivité ».


## <a name="solution"></a>Solution

Avant de suivre cette procédure, faites en sauvegarde en prenant un instantané du disque du système d’exploitation de la machine virtuelle affectée. Pour plus d’informations, consultez [Créer un instantané](../windows/snapshot-copy-managed-disk.md).

Pour résoudre ce problème, utilisez le contrôle série pour configurer la machine virtuelle avec un démarrage en mode normal ou [réparez la machine virtuelle hors connexion](#repair-the-vm-offline) à l’aide d’une machine virtuelle de récupération.

### <a name="use-serial-control"></a>Utiliser le contrôle série

1. Connectez-vous à la [console série et ouvrez une instance CMD](./serial-console-windows.md#use-cmd-or-powershell-in-serial-console
   ). Si la console série n’est pas activée sur votre machine virtuelle, consultez [Réparer la machine virtuelle en mode hors connexion](#repair-the-vm-offline).
2. Consultez les données de configuration de démarrage :

    ```console
    bcdedit /enum
    ```

    Si la machine virtuelle est configurée pour démarrer en mode sans échec, vous voyez un indicateur supplémentaire sous la section **Chargeur de démarrage Windows** appelé **safeboot**. Si vous ne voyez pas l’indicateur **safeboot**, la machine virtuelle n’est pas en mode sans échec. Cet article ne s’applique pas à votre scénario.

    L’indicateur **safeboot** peut apparaître avec les valeurs suivantes :
   - Minimales
   - Réseau

     Dans ces deux modes, RDP ne démarrera pas. Par conséquent, le correctif reste inchangé.

     ![Image de l’indicateur Mode sans échec](./media/troubleshoot-rdp-safe-mode/safe-mode-tag.png)

3. Supprimez l’indicateur **safemode** pour que la machine virtuelle démarre en mode normal :

    ```console
    bcdedit /deletevalue {current} safeboot
    ```

4. Vérifiez les données de configuration de démarrage pour vous assurer que l’indicateur **safeboot** est supprimé :

    ```console
    bcdedit /enum
    ```

5. Redémarrez la machine virtuelle et vérifiez que le problème est résolu.

### <a name="repair-the-vm-offline"></a>Réparer la machine virtuelle en mode hors connexion

#### <a name="attach-the-os-disk-to-a-recovery-vm"></a>Attachez le disque du système d’exploitation à une machine virtuelle de récupération

1. [Attachez le disque du système d’exploitation à une machine virtuelle de récupération](./troubleshoot-recovery-disks-portal-windows.md).
2. Établissez une connexion Bureau à distance avec la machine virtuelle de récupération.
3. Vérifiez que le disque est marqué comme étant **En ligne** dans la console Gestion des disques. Notez la lettre de lecteur qui est affectée au disque du système d’exploitation attaché.

#### <a name="enable-dump-log-and-serial-console-optional"></a>Activer le journal de vidage et la console série (facultatif)

Le journal de vidage et la console série nous aideront à poursuivre notre dépannage si le problème ne peut pas être résolu par la solution de cet article.

Pour activer le journal de vidage et la console série, exécutez le script suivant.

1. Ouvrez une session Invite de commande avec élévation de privilèges (**Exécuter en tant qu’administrateur**).
2. Exécutez le script suivant :

    Dans ce script, nous partons du principe que la lettre de lecteur affectée au disque de système d’exploitation attaché est F. Remplacez-la par la valeur correspondant à votre machine virtuelle.

    ```powershell
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM

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

#### <a name="configure-the-windows-to-boot-into-normal-mode"></a>Configurer Windows pour démarrer en mode normal

1. Ouvrez une session Invite de commande avec élévation de privilèges (**Exécuter en tant qu’administrateur**).
2. Consultez les données de configuration de démarrage. Dans les commandes suivantes, nous partons du principe que la lettre de lecteur affectée au disque de système d’exploitation attaché est F. Remplacez-la par la valeur correspondant à votre machine virtuelle.

    ```console
    bcdedit /store F:\boot\bcd /enum
    ```

    Prenez note du nom d’identificateur de la partition comportant le dossier **\windows**. Par défaut, le nom de l’identificateur est « Default ».

    Si la machine virtuelle est configurée pour démarrer en mode sans échec, vous voyez un indicateur supplémentaire sous la section **Chargeur de démarrage Windows** appelé **safeboot**. Si vous ne voyez pas l’indicateur **safeboot**, cet article ne s’applique pas à votre scénario.

    ![Image sur l’identificateur de démarrage](./media/troubleshoot-rdp-safe-mode/boot-id.png)

3. Supprimez l’indicateur **safeboot** pour que la machine virtuelle démarre en mode normal :

    ```console
    bcdedit /store F:\boot\bcd /deletevalue {Default} safeboot
    ```

4. Vérifiez les données de configuration de démarrage pour vous assurer que l’indicateur **safeboot** est supprimé :

    ```console
    bcdedit /store F:\boot\bcd /enum
    ```

5. [Détachez le disque de système d’exploitation et recréez la machine virtuelle](./troubleshoot-recovery-disks-portal-windows.md). Ensuite, vérifiez que le problème est résolu.
