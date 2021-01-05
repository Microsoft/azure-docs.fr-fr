---
title: La machine virtuelle Azure ne répond pas avec l’erreur C01A001D lors de l’application d’une mise à jour Windows
description: Cet article décrit les étapes à suivre pour résoudre les problèmes lorsqu’une mise à jour Windows génère une erreur et ne répond plus dans une machine virtuelle Azure.
services: virtual-machines-windows
documentationcenter: ''
author: TobyTu
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: a97393c3-351d-4324-867d-9329e31b3528
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
ms.date: 03/31/2020
ms.author: v-mibufo
ms.openlocfilehash: 76c3f729a8520c7bff7b49a1d2200d7950f8a9f4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87074300"
---
# <a name="vm-is-unresponsive-with-c01a001d-error-when-applying-windows-update"></a>La machine virtuelle ne répond pas avec l’erreur « C01A001D » lors de l’application d’une mise à jour Windows

Cet article décrit les étapes à suivre pour résoudre les problèmes lorsqu’une mise à jour Windows (en Ko) génère une erreur et ne répond plus dans une machine virtuelle Azure.

## <a name="symptoms"></a>Symptômes

Lorsque vous utilisez [Diagnostics de démarrage](./boot-diagnostics.md) pour afficher la capture d’écran de la machine virtuelle, la mise à jour Windows (en Ko) en cours est affichée, mais échoue avec le code d’erreur : « C01A001D ».

![Windows Update sans réponse](./media/unresponsive-vm-apply-windows-update/unresponsive-windows-update.png)

## <a name="cause"></a>Cause

Impossible de créer un fichier de base dans le système de fichiers. Le système d’exploitation ne peut pas écrire de fichiers sur le disque.

## <a name="resolution"></a>Résolution

### <a name="process-overview"></a>Vue d’ensemble du processus

1. [Créer une machine virtuelle de réparation et y accéder](#create-and-access-a-repair-vm).
2. [Libérer de l’espace sur le disque dur](#free-up-space-on-the-hard-disk).
3. [Recommandé : Avant de régénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire](#recommended-before-rebuilding-the-vm-enable-serial-console-and-memory-dump-collection).
4. [Régénérer la machine virtuelle](#rebuild-the-vm).

> [!NOTE]
> Lorsque cette erreur se produit, le système d’exploitation invité n’est pas opérationnel. Vous devez résoudre ce problème en mode hors connexion.

### <a name="create-and-access-a-repair-vm"></a>Créer une machine virtuelle de réparation et y accéder

1. Suivez les [étapes 1 à 3 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une machine virtuelle de réparation.
2. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="free-up-space-on-the-hard-disk"></a>Libérer de l’espace sur le disque dur

Si le disque n’est pas déjà de 1 To, vous devez le redimensionner. Une fois que la taille du disque est de 1 To, effectuez un nettoyage de disque et une défragmentation du lecteur.

1. Vérifiez si le disque est plein. Si la taille du disque est inférieure à 1 To, [augmentez-la jusqu’à un maximum de 1 To à l’aide de PowerShell](../windows/expand-os-disk.md?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json).
2. Une fois que le disque fait 1 To, effectuez un nettoyage de disque.
    - [Détachez le disque de données de la machine virtuelle corrompue](../windows/detach-disk.md).
    - [Attachez le disque de données à une machine virtuelle opérationnelle](../windows/attach-disk-ps.md#attach-an-existing-data-disk-to-a-vm).
    - Utilisez l’[outil Nettoyage de disque](https://support.microsoft.com/help/4026616/windows-10-disk-cleanup) pour libérer de l’espace.
3. Après le redimensionnement et le nettoyage, défragmentez le lecteur :

    ```
    defrag <LETTER ASSIGN TO THE OS DISK>: /u /x /g
    ```
    Selon le niveau de fragmentation, cela peut prendre plusieurs heures.

### <a name="recommended-before-rebuilding-the-vm-enable-serial-console-and-memory-dump-collection"></a>Recommandé : Avant de régénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire.

1. Ouvrez une session d’invite de commandes avec élévation de privilèges (Exécuter en tant qu’administrateur).
2. Exécutez les commandes suivantes :

    Activez la console série :

    ```
    bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON
    bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200
    ```
3. Assurez-vous que l’espace libre sur le disque du système d’exploitation est au moins égal à la taille de la mémoire (RAM) de la machine virtuelle.

    Si l’espace n’est pas suffisant, modifiez l’emplacement où le fichier de l’image mémoire sera créé et référencez-le sur un autre disque de données attaché à la machine virtuelle possédant suffisamment d’espace libre. Pour changer l’emplacement, remplacez `%SystemRoot%` par la lettre de lecteur (par exemple, « F: ») du disque de données dans les commandes ci-dessous :

    **Configuration suggérée pour activer l’image mémoire du système d’exploitation :**

    Chargez le disque de système d’exploitation corrompu :

    ```
    REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM
    ```

    Activez sur ControlSet001 :

    ```
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f
    ```

    Activez sur ControlSet002 :

    ```
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f 
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
    REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f
    ```

    Déchargez le disque de système d’exploitation corrompu :

    ```
    REG UNLOAD HKLM\BROKENSYSTEM
    ```

### <a name="rebuild-the-vm"></a>Régénérez la machine virtuelle.

Suivez l’[étape 5 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour réassembler la machine virtuelle.
