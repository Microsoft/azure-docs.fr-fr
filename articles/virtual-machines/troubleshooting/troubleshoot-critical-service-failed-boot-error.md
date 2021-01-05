---
title: CRITICAL SERVICE FAILED lors du démarrage d’une machine virtuelle Azure | Microsoft Docs
description: Découvrez comment résoudre l’erreur « 0x0000005A-CRITICAL SERVICE FAILED » qui se produit lors du démarrage | Microsoft Docs
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/08/2018
ms.author: genli
ms.openlocfilehash: 8c3e76f1a7edffefc8773dfa548773ec0932fae6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86129857"
---
# <a name="windows-shows-critical-service-failed-on-blue-screen-when-booting-an-azure-vm"></a>Windows affiche « CRITICAL SERVICE FAILED » sur écran bleu lors du démarrage d’une machine virtuelle Azure
Cet article décrit l’erreur « CRITICAL SERVICE FAILED » que vous pouvez rencontrer lorsque vous démarrez une machine virtuelle Windows dans Microsoft Azure. Il fournit des étapes de dépannage pour vous aider à résoudre les problèmes. 


## <a name="symptom"></a>Symptôme 

Une machine virtuelle Windows ne démarre pas. Quand vous vérifiez les captures d’écran de démarrage de la fenêtre [Diagnostics de démarrage](./boot-diagnostics.md), vous voyez l’un des messages d’erreur suivants sur un écran bleu :

- « Votre PC a rencontré un problème et doit redémarrer. Vous pouvez le redémarrer. Pour plus d’informations sur ce problème et les solutions possibles, visitez https://windows.com/stopcode. Si vous appelez un agent du support technique, fournissez-lui l'information suivante : Code d'arrêt : ÉCHEC DU SERVICE CRITIQUE » 
- « Votre PC a rencontré un problème et doit redémarrer. Nous collectons simplement quelques informations sur l’erreur, puis nous vous aiderons à redémarrer. Si vous souhaitez en savoir plus, vous pourrez ultérieurement rechercher cette erreur en ligne : CRITICAL_SERVICE_FAILED »

## <a name="cause"></a>Cause

Les erreurs d’arrêt peuvent avoir plusieurs causes. Les causes les plus courantes sont :
- Problème au niveau d’un pilote
- Mémoire ou fichier système endommagé
- Une application accède à un secteur interdit de la mémoire

## <a name="solution"></a>Solution 

Pour résoudre ce problème, [contactez le support technique et envoyez un fichier de vidage sur incident](./troubleshoot-common-blue-screen-error.md#collect-memory-dump-file), ce qui nous aidera à diagnostiquer le problème plus rapidement, ou essayez de vous aider vous-même comme suit.

### <a name="attach-the-os-disk-to-a-recovery-vm"></a>Attachez le disque du système d’exploitation à une machine virtuelle de récupération

1. Prenez un instantané du disque du système d’exploitation de la machine virtuelle affectée en guise de sauvegarde. Pour plus d’informations, consultez [Créer un instantané](../windows/snapshot-copy-managed-disk.md).
2. [Attachez le disque du système d’exploitation à une machine virtuelle de récupération](./troubleshoot-recovery-disks-portal-windows.md). 
3. Établissez une connexion Bureau à distance avec la machine virtuelle Site Recovery.

### <a name="enable-dump-logs-and-serial-console"></a>Activer les journaux d’activité de vidage et la console série

Le journal de vidage et la [console série](./serial-console-windows.md) nous aideront à effectuer des dépannages supplémentaires.

Pour activer les journaux d’activité de vidage et la console série, exécutez le script suivant.

1. Ouvrez une session Invite de commande avec élévation de privilèges (exécuter en tant qu’administrateur).
2. Exécutez le script suivant :

    Dans ce script, nous partons du principe que la lettre de lecteur qui est affectée au disque du système d’exploitation attaché est F. Vous devez la remplacer par la valeur appropriée dans votre machine virtuelle.

    ```powershell
    reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

    REM Enable Serial Console
    bcdedit /store F:\boot\bcd /set {bootmgr} displaybootmenu yes
    bcdedit /store F:\boot\bcd /set {bootmgr} timeout 10
    bcdedit /store F:\boot\bcd /set {bootmgr} bootems yes
    bcdedit /store F:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON
    bcdedit /store F:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200

    REM Suggested configuration to enable OS Dump
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 2 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 2 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f

    reg unload HKLM\BROKENSYSTEM
    ```

### <a name="replace-the-unsigned-drivers"></a>Remplacer les pilotes non signés

1. Sur la machine virtuelle Site Recovery, exécutez la commande suivante à partir d’une invite de commandes avec élévation de privilèges. Cette commande définit le disque du système d’exploitation affecté pour démarrer en mode sans échec au prochain démarrage :

    ```console
    bcdedit /store <OS DISK you attached>:\boot\bcd /set {default} safeboot minimal
    ```

    Par exemple, si le disque du système d’exploitation que vous avez attaché est le lecteur F, exécutez la commande suivante :

    ```console
    bcdedit /store F: boot\bcd /set {default} safeboot minimal
    ```

2. [Détachez le disque du système d’exploitation et rattachez-le à la machine virtuelle affectée](troubleshoot-recovery-disks-portal-windows.md). La machine virtuelle démarre en mode sans échec. Si l'erreur persiste, passez à l'étape facultative.
3. Ouvrez la zone **Exécuter** et exécutez le **vérificateur** pour démarrer l’outil Driver Verifier Manager.
4. Sélectionnez **Sélectionner automatiquement les pilotes non signés**, puis cliquez sur **suivant**.
5. Vous obtiendrez la liste des fichiers du pilote qui ne sont pas signés. N’oubliez pas les noms des fichiers.
6. Copiez les mêmes versions de ces fichiers à partir d’une machine virtuelle qui fonctionne, puis remplacez ces fichiers non signés. 

7. Supprimez les paramètres de démarrage sans échec :

    ```console
    bcdedit /store <OS DISK LETTER>:\boot\bcd /deletevalue {default} safeboot
    ```

8.  Redémarrez la machine virtuelle. 

### <a name="optional-analyze-the-dump-logs-in-dump-crash-mode"></a>Facultatif : Analyser les journaux d’activité de vidage en mode Vidage sur incident

Pour analyser vous-même les journaux d’activité de vidage, procédez comme suit :

1. Attachez le disque du système d’exploitation à une machine virtuelle de récupération.
2. Sur le disque du système d’exploitation que vous avez attaché, accédez à **\windows\system32\config**. Copiez tous les fichiers en tant que sauvegarde pour le cas où une restauration serait requise.
3. Démarrez l’**Éditeur du Registre** (regedit.exe).
4. Sélectionnez la clé **HKEY_LOCAL_MACHINE**. Dans le menu, sélectionnez **Fichier** > **Charger Hive**.
5. Sur le disque du système d’exploitation que vous avez attaché, accédez au dossier **\windows\system32\config\SYSTEM**. Pour le nom de la ruche, entrez **BROKENSYSTEM**. La nouvelle ruche du Registre est affichée sous la clé **HKEY_LOCAL_MACHINE**.
6. Accédez à **HKEY_LOCAL_MACHINE\BROKENSYSTEM\ControlSet00x\Control\CrashControl** et apportez les modifications suivantes :

    Autoreboot = 0

    CrashDumpEnabled = 2
7.  Sélectionnez **BROKENSYSTEM**. Dans le menu, sélectionnez **Fichier** > **Décharger la ruche**.
8.  Modifier la configuration BCD pour démarrer en mode débogage. Exécutez les commandes suivantes à partir d'une invite de commandes avec élévation de privilèges :

    ```cmd
    REM Setup some debugging flags on the boot manager
    bcdedit /store <OS DISK LETTER>:\boot\bcd /set {bootmgr} default {<IDENTIFIER OF THE WINDOWS BOOT LOADER>}
    bcdedit /store <OS DISK LETTER>:\boot\bcd /set {bootmgr} nointegritychecks on
    bcdedit /store <OS DISK LETTER>:\boot\bcd /set {bootmgr} integrityservices disable

    REM Setup some debugging flags on the boot loader
    bcdedit /store <OS DISK LETTER>:\boot\bcd /set {default} bootlog yes
    bcdedit /store <OS DISK LETTER>:\boot\bcd /set {default} bootstatuspolicy displayallfailures
    bcdedit /store <OS DISK LETTER>:\boot\bcd /set {default} nointegritychecks on
    bcdedit /store <OS DISK LETTER>:\boot\bcd /set {default} testsigning off
    bcdedit /store <OS DISK LETTER>:\boot\bcd /set {default} recoveryenabled no
    bcdedit /store <OS DISK LETTER>:\boot\bcd /set {default} integrityservices disable
    ```
9. [Détachez le disque du système d’exploitation et rattachez-le à la machine virtuelle affectée](troubleshoot-recovery-disks-portal-windows.md).
10. Démarrez la machine virtuelle pour voir si elle affiche l’analyse de vidage. Recherchez le fichier qui ne peut pas être chargé. Vous devez remplacer ce fichier par un fichier de la machine virtuelle qui fonctionne. 

    Voici un exemple d’analyse de vidage. Vous pouvez constater que l'échec (**FAILURE**) figure dans le fichier filecrypt.sys : « FAILURE_BUCKET_ID: 0x5A_c0000428_IMAGE_filecrypt.sys ».

    ```
    kd> !analyze -v 

    ******************************************************************************* 
    * * * Bugcheck Analysis * * * 

    ******************************************************************************* 
    CRITICAL_SERVICE_FAILED (5a) Arguments: Arg1: 0000000000000001 Arg2: ffffd80f4bfe7070 Arg3: ffffb00b0513d320 Arg4: ffffffffc0000428 Debugging 
    Details: ------------------ 
    DUMP_CLASS: 1 DUMP_QUALIFIER: 401 BUILD_VERSION_STRING: 10.0.14393.1770 (rs1_release.170917-1700) 
    DUMP_TYPE: 1 BUGCHECK_P1: 1 BUGCHECK_P2: ffffd80f4bfe7070 BUGCHECK_P3: ffffb00b0513d320 BUGCHECK_P4: ffffffffc0000428 BUGCHECK_STR: 0x5A_c0000428 
    CPU_COUNT: 1 CPU_MHZ: a22 CPU_VENDOR: GenuineIntel CPU_FAMILY: 6 CPU_MODEL: 3d CPU_STEPPING: 4 DEFAULT_BUCKET_ID: WIN8_DRIVER_FAULT 
    PROCESS_NAME: System CURRENT_IRQL: 0 ANALYSIS_SESSION_HOST: MININT-6RMM091 ANALYSIS_SESSION_TIME: 11-15-2017 19:32:42.0841 
    ANALYSIS_VERSION: 10.0.16361.1001 amd64fre STACK_TEXT: ffffc701`1dc74948 fffff803`b2ff4b4a : 00000000`0000005a 00000000`00000001 ffffd80f`4bfe7070 ffffb00b`0513d320 : nt!KeBugCheckEx [d:\rs1\minkernel\ntos\ke\amd64\procstat.asm @ 127] ffffc701`1dc74950 fffff803`b3205df3 : ffffd80f`4bba9f58 ffffd80f`4bba9f58 ffffc701`1dc74b80 ffffd80f`00000006 : nt!IopLoadDriver+0x19f8e6 [d:\rs1\minkernel\ntos\ke\amd64\threadbg.asm @ 81] 
    RETRACER_ANALYSIS_TAG_STATUS: DEBUG_FLR_FAULTING_IP is not found THREAD_SHA1_HASH_MOD_FUNC: eb79608c07faa1af62c0e61f25ff6bc1d6dfdb25 THREAD_SHA1_HASH_MOD_FUNC_OFFSET: 96a3a314834bb4e8443a8b7201525fc5dfc1878b THREAD_SHA1_HASH_MOD: 30a3e915496deaace47137d5b90c3ecc03746bf6 FOLLOWUP_NAME: wintriag
    MODULE_NAME: filecrypt IMAGE_NAME: filecrypt.sys DEBUG_FLR_IMAGE_TIMESTAMP: 0 IMAGE_VERSION: STACK_COMMAND: .thread ; .cxr ; kb FAILURE_BUCKET_ID: 0x5A_c0000428_IMAGE_filecrypt.sys BUCKET_ID: 0x5A_c0000428_IMAGE_filecrypt.sys PRIMARY_PROBLEM_CLASS: 0x5A_c0000428_IMAGE_filecrypt.sys TARGET_TIME: 2017-11-13T20:51:04.000Z OSBUILD: 14393 OSSERVICEPACK: 1770 SERVICEPACK_NUMBER: 0 OS_REVISION: 0 SUITE_MASK: 144 PRODUCT_TYPE: 3 OSPLATFORM_TYPE: x64 OSNAME: Windows 10 OSEDITION: Windows 10 Server TerminalServer DataCenter OS_LOCALE: USER_LCID: 0 OSBUILD_TIMESTAMP: 2017-09-17 19:16:08 BUILDDATESTAMP_STR: 170917-1700 BUILDLAB_STR: rs1_release BUILDOSVER_STR: 10.0.14393.1770 ANALYSIS_SESSION_ELAPSED_TIME: bfc ANALYSIS_SOURCE: KM FAILURE_ID_HASH_STRING: km:0x5a_c0000428_image_filecrypt.sys FAILURE_ID_HASH: {35f25777-b01e-70a1-c502-f690dab6cb3a} FAILURE_ID_REPORT_LINK: https://go.microsoft.com/fwlink/?LinkID=397724&FailureHash=35f25777-b01e-70a1-c502-f690dab6cb3a
    ```

11. Une fois que la machine virtuelle fonctionne et démarre normalement, supprimez les paramètres Dump Crash :

    ```cmd
    REM Restore the boot manager to default values
    bcdedit /store <OS DISK LETTER>:\boot\bcd /deletevalue {bootmgr} nointegritychecks
    bcdedit /store <OS DISK LETTER>:\boot\bcd /set {bootmgr} integrityservices enable

    REM Restore the boot loader to default values for an azure VM
    bcdedit /store <OS DISK LETTER>:\boot\bcd /deletevalue {default} bootlog
    bcdedit /store <OS DISK LETTER>:\boot\bcd /set {default} bootstatuspolicy ignoreallfailures
    bcdedit /store <OS DISK LETTER>:\boot\bcd /deletevalue {default} nointegritychecks
    bcdedit /store <OS DISK LETTER>:\boot\bcd /deletevalue {default} testsigning
    bcddit /store <OS DISK LETTER>:\boot\bcd /set {default} recoveryenabled no
    bcdedit /store <OS DISK LETTER>:\boot\bcd /set {default} integrityservicesenable
    ```
