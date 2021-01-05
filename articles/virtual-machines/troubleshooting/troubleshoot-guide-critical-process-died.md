---
title: Erreur d’arrêt de Windows –
description: ''
services: virtual-machines-windows
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 5b3ed56a-5a11-4ff9-9ee8-76aea4a5689b
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
ms.date: 03/26/2020
ms.author: v-mibufo
ms.openlocfilehash: c04f3b27c7214dcf821c7698796bfaea399b947d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86509101"
---
# <a name="windows-stop-error---0x000000ef-critical-process-died"></a>Erreur d’arrêt de Windows - #0x000000EF « Processus critique arrêté »

Cet article décrit la procédure à suivre pour résoudre des problèmes où un processus critique s’arrête durant son démarrage dans une machine virtuelle Azure.

## <a name="symptom"></a>Symptôme

Quand vous utilisez les [Diagnostics de démarrage](./boot-diagnostics.md) pour afficher la capture d’écran de la machine virtuelle, vous pouvez voir que cette capture affiche l’erreur *#0x000000EF* avec le message *Processus critique arrêté*.

![« Votre PC a rencontré un problème et doit redémarrer. Nous collectons simplement quelques informations sur l’erreur et vous pourrez ensuite redémarrer. (##% complete) Pour en savoir plus, vous pourrez rechercher cette erreur en ligne ultérieurement : 0x000000EF"](media/troubleshoot-guide-critical-process-died/1.jpg)

## <a name="cause"></a>Cause

En règle générale, cela est dû à l’échec d’un processus système critique au démarrage. Pour plus d’informations sur les problèmes de processus critiques, consultez [Vérification de bogue 0xEF : CRITICAL_PROCESS_DIED](/windows-hardware/drivers/debugger/bug-check-0xef--critical-process-died).

## <a name="solution"></a>Solution

### <a name="process-overview"></a>Vue d’ensemble du processus :

1. Créez une machine virtuelle de réparation et accédez-y.
2. Corrigez les altérations éventuelles du système d’exploitation.
3. **Recommandé** : Avant de regénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire.
4. Regénérez la machine virtuelle.

> [!NOTE]
> Lorsque vous rencontrez cette erreur de démarrage, cela signifie que le système d’exploitation invité n’est pas opérationnel. Pour résoudre ce problème, vous allez opérer en mode hors connexion.

### <a name="create-and-access-a-repair-vm"></a>Créer une machine virtuelle de réparation et y accéder

1. Effectuez les [étapes 1 à 3 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une machine virtuelle de réparation.
2. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="fix-any-os-corruption"></a>Corriger les altérations éventuelles du système d’exploitation

1. Ouvrez une invite de commandes avec élévation de privilèges.
2. Exécutez la commande du vérificateur des fichiers système Windows (SFC) suivante :

   `sfc /scannow /offbootdir=<BOOT DISK DRIVE>:\ /offwindir=<BROKEN DISK DRIVE>:\windows`

   * Où < BOOT DISK DRIVE > est le volume de démarrage de la machine virtuelle de réparation (généralement « C: ») et < BROKEN DISK DRIVE > est la lettre de lecteur du disque attaché de la machine virtuelle endommagée. Remplacez les symboles « supérieur à » et « inférieur à », ainsi que le texte inséré entre eux, par exemple, « < texte ici > », par la lettre appropriée.

3. Ensuite, suivez l’[étape 5 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour réassembler la machine virtuelle et voir sir elle démarre.
4. Si la machine virtuelle ne démarre toujours pas, poursuivez la collecte du fichier de l’image mémoire.

### <a name="collect-the-memory-dump-file"></a>Collecter le fichier de l’image mémoire

Si le problème persiste après l’exécution de la commande SFC, une analyse d’un fichier de l’image mémoire sera nécessaire pour en déterminer la cause. Pour collecter le fichier de l’image mémoire, procédez comme suit :

### <a name="attach-the-os-disk-to-a-new-repair-vm"></a>Attacher le disque du système d’exploitation à une machine virtuelle de récupération

1. Suivez les [étapes 1 à 3 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une nouvelle machine virtuelle de réparation.
2. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="locate-the-dump-file-and-submit-a-support-ticket"></a>Rechercher le fichier de l’image mémoire et envoyer un ticket de support

3. Sur la machine virtuelle de réparation, accédez au dossier Windows sur le disque du système d’exploitation attaché. Si la lettre de lecteur qui est affectée au disque du système d’exploitation attaché est *F*, vous devez accéder à *F:\Windows*.
4. Recherchez le fichier *memory.dmp*, puis [envoyez un ticket de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) avec le fichier de l’image mémoire.

   > [!NOTE]
   > Si vous ne trouvez pas le fichier de l’image mémoire, suivez les étapes ci-dessous pour activer la collecte de l’image mémoire et la console série, puis revenez à cette section et répétez les étapes de la tâche ci-dessus pour collecter le fichier de l’image mémoire.

### <a name="recommended-before-you-rebuild-the-vm-enable-serial-console-and-memory-dump-collection"></a>Recommandé : Avant de regénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire

Pour activer la collecte de l’image mémoire et la console série, exécutez le script suivant :

1. Ouvrez une session Invite de commande avec élévation de privilèges (Exécuter en tant qu’administrateur).
2. Exécutez les commandes suivantes :

   Activer la console série

   `bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON`

   `bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200`

   Remplacez les symboles supérieur à ou inférieur à, ainsi que le texte inséré entre eux, par exemple, « < texte ici > ».

3. Vérifiez que l’espace disponible sur le disque du système d’exploitation est égal à la taille de la mémoire (RAM) sur la machine virtuelle.

Si l’espace n’est pas suffisant, nous vous recommandons de changer l’emplacement où le fichier de l’image mémoire sera créé et de le référencer sur n’importe quel autre disque de données attaché à la machine virtuelle possédant suffisamment d’espace libre. Pour changer l’emplacement, remplacez « %SystemRoot% » par la lettre de lecteur (par exemple, « F: ») du disque de données dans les commandes ci-dessous.

#### <a name="suggested-configuration-to-enable-os-dump"></a>Configuration suggérée pour activer l’image mémoire du système d’exploitation

**Charger le disque de système d’exploitation corrompu** :

`REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM`

**Activer sur ControlSet001 :**

`REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f`

`REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f`

`REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f`

**Activer sur ControlSet002 :**

`REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f`

`REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f`

`REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f`

**Décharger le disque de système d’exploitation corrompu :**

`REG UNLOAD HKLM\BROKENSYSTEM`

### <a name="rebuild-the-original-vm"></a>Régénérer la machine virtuelle d’origine

Suivez l’[étape 5 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour réassembler la machine virtuelle.
