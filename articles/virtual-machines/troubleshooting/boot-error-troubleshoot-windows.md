---
title: L’arrêt de Machines virtuelles Azure est bloqué sur Redémarrage, Arrêt ou Arrêt des services | Microsoft Docs
description: Cet article vous aide à résoudre les problèmes des erreurs du service dans Machines virtuelles Windows Azure.
services: virtual-machines-windows
documentationCenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 12/19/2019
ms.author: tibasham
ms.openlocfilehash: e5ab1262def78da4971ea6e5535f3ac915a38ec8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86526756"
---
# <a name="azure-windows-vm-shutdown-is-stuck-on-restarting-shutting-down-or-stopping-services"></a>L’arrêt de Machines virtuelles Windows Azure est bloqué sur « Redémarrage », « Arrêt » ou « Arrêt des services »

Cet article décrit les étapes à suivre pour résoudre les problèmes de messages « Redémarrage », « Arrêt » ou « Arrêt des services » que vous pouvez rencontrer quand vous redémarrez une machine virtuelle Windows dans Microsoft Azure.

## <a name="symptoms"></a>Symptômes

Quand vous utilisez [Diagnostics de démarrage](./boot-diagnostics.md) pour visualiser la capture d’écran de la machine virtuelle, vous pouvez voir que cette capture affiche le message « Redémarrage », « Arrêt » ou « Arrêt des services ».

![Écrans Redémarrage, Arrêt ou Arrêt des services](./media/boot-error-troubleshooting-windows/restart-shut-down-stop-service.png)
 
## <a name="cause"></a>Cause

Windows utilise le processus d’arrêt pour effectuer les opérations de maintenance du système et traiter les modifications, comme les mises à jour, les rôles et les fonctionnalités. Il n’est pas recommandé d’interrompre ce processus critique avant qu’il se termine. En fonction du nombre de mises à jour/modifications et de la taille de la machine virtuelle, le processus peut prendre beaucoup de temps. Si le processus est arrêté, il est possible que le système d’exploitation soit endommagé. Interrompez le processus seulement s’il est excessivement long.

## <a name="solution"></a>Solution

### <a name="collect-a-process-memory-dump"></a>Collecter une image mémoire des processus

1. Téléchargez l’[outil Procdump](http://download.sysinternals.com/files/Procdump.zip) sur un disque de données nouveau ou existant, qui est attaché à une machine virtuelle fonctionnelle de la même région.

2. Détachez le disque contenant les fichiers nécessaires de la machine virtuelle fonctionnelle et attachez le disque à votre machine virtuelle endommagée. Nous appelons ce disque **Disque utilitaire**.

Utilisez la [console série](./serial-console-windows.md) pour effectuer les étapes suivantes :

1. Ouvrez une session PowerShell d’administration et vérifiez le service qui cesse de répondre lors de l’arrêt.

   ``
   Get-Service | Where-Object {$_.Status -eq "STOP_PENDING"}
   ``

2. Sur une commande d’administration, récupérez le PID du service qui ne répond pas.

   ``
   tasklist /svc | findstr /i <STOPING SERVICE>
   ``

3. Récupérez un exemple d’image mémoire à partir du processus <STOPPING SERVICE> qui ne répond pas.

   ``
   procdump.exe -s 5 -n 3 -ma <PID>
   ``

4. Tuez maintenant le processus qui ne répond pas pour débloquer le processus d’arrêt.

   ``
   taskkill /PID <PID> /t /f
   ``

Une fois que le système d’exploitation redémarre, s’il démarre normalement, vérifiez simplement que la cohérence du système d’exploitation est correcte. Si une altération est signalée, exécutez la commande suivante jusqu’à ce que le disque ne comporte plus d’altération :

``
dism /online /cleanup-image /restorehealth
``

Si vous ne parvenez pas à collecter une image mémoire du processus, ou si ce problème est récurrent et que vous avez besoin d’une analyse de la cause racine, poursuivez en collectant l’image mémoire du système d’exploitation ci-dessous, puis ouvrir une demande de support.

### <a name="collect-an-os-memory-dump"></a>Collecter un vidage mémoire du système d’exploitation

Si le problème n’est pas résolu après que vous avez attendu que les modifications soient traitées, vous devez recueillir un fichier d’image mémoire et contacter le support technique. Pour collecter le fichier d’image, effectuez les étapes suivantes :

**Attacher le disque du système d’exploitation à une machine virtuelle de récupération**

1. Prenez un instantané du disque du système d’exploitation de la machine virtuelle affectée en guise de sauvegarde. Pour plus d’informations, consultez [Créer un instantané](../windows/snapshot-copy-managed-disk.md).

2. [Attachez le disque du système d’exploitation à une machine virtuelle de récupération](./troubleshoot-recovery-disks-portal-windows.md).

3. Connectez-vous à la machine virtuelle de récupération à l’aide du Bureau à distance.

4. Si le disque du système d’exploitation est chiffré, vous devez désactiver le chiffrement avant de passer à l’étape suivante. Pour plus d’informations, consultez [Déchiffrer le disque de système d’exploitation chiffré sur la machine virtuelle qui ne peut pas démarrer](./troubleshoot-bitlocker-boot-error.md#solution).

**Rechercher le fichier d’image mémoire et envoyer un ticket de support**

1. Sur la machine virtuelle de récupération, accédez au dossier Windows dans le disque du système d’exploitation attaché. Si la lettre de lecteur qui est affectée au disque du système d’exploitation attaché est F, vous devez accéder à F:\Windows.

2. Recherchez le fichier memory.dmp, puis [envoyez un ticket de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) avec le fichier d’image.

Si le fichier d’image est introuvable, passez à l’étape suivante pour activer le journal de vidage et la console série.

**Activer le journal d’image mémoire et la console série**

Pour activer le journal de vidage et la console série, exécutez le script suivant.

1. Ouvrez une session Invite de commande avec élévation de privilèges (Exécuter en tant qu’administrateur).

2. Exécutez le script suivant :

   Dans ce script, nous supposons que la lettre de lecteur qui est affectée au disque du système d’exploitation attaché est F. Remplacez-la par la valeur appropriée de votre machine virtuelle.

   ```
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
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump / t REG_DWORD /d 1 /f
   
   reg unload HKLM\BROKENSYSTEM
   ```

3. Vérifiez que l’espace disque est suffisant pour allouer autant de mémoire que la RAM, qui dépend de la taille que vous sélectionnez pour cette machine virtuelle.

4. Si l’espace n’est pas suffisant ou qu’il s’agit d’une machine virtuelle de grande taille (série G, GS ou E), vous pouvez changer l’emplacement où ce fichier sera créé et le référencer sur n’importe quel autre disque de données qui est attaché à la machine virtuelle. Pour changer l’emplacement, vous devez modifier la clé suivante :

   ```
   reg load HKLM\BROKENSYSTEM F:\windows\system32\config\SYSTEM.hiv

   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "<DRIVE LETTER OF YOUR DATA DISK>:\MEMORY.DMP" /f
   
   reg unload HKLM\BROKENSYSTEM
   ```

5. [Détachez le disque du système d’exploitation, puis rattachez-le à la machine virtuelle affectée](./troubleshoot-recovery-disks-portal-windows.md).

6. Démarrez la machine virtuelle et accédez à la console série.

7. Sélectionnez Envoyer une interruption non masquable (NMI) pour déclencher l’image mémoire.

   ![Envoyer une interruption non masquable](./media/boot-error-troubleshooting-windows/send-nonmaskable-interrupt.png)

8. Reconnectez le disque du système d’exploitation à une machine virtuelle de récupération, puis recueillez le fichier d’image mémoire.

## <a name="contact-microsoft-support"></a>Contact Microsoft support

Après avoir collecté le fichier d’image mémoire, contactez le support Microsoft pour lui demander de déterminer la cause racine.
