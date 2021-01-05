---
title: Résoudre les problèmes de démarrage du système d’exploitation – Capacité d’installation Windows Update
description: Étapes à suivre pour résoudre les problèmes lorsqu’une mise à jour Windows (en Ko) entraîne une erreur et ne répond plus dans une machine virtuelle Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 6359e486-7b02-4c1e-995c-ef6d505f85f4
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 05/11/2020
ms.author: v-miegge
ms.openlocfilehash: f83a1820eb931fa075681da7a9661b304059cd2a
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94635703"
---
# <a name="troubleshoot-os-start-up--windows-update-installation-capacity"></a>Résoudre les problèmes de démarrage du système d’exploitation – Capacité d’installation Windows Update

Cet article décrit les étapes à suivre pour résoudre les problèmes lorsqu’une mise à jour Windows (en Ko) génère une erreur et ne répond plus.

## <a name="symptom"></a>Symptôme

Quand vous utilisez les Diagnostics de démarrage pour afficher la capture d’écran de la machine virtuelle, vous pouvez voir que cette capture affiche l’opération Windows Update (en Ko) comme étant en cours, mais générant le code d’erreur : **C01A001D**. L’image suivante montre l’opération Windows Update (en Ko) bloquée avec le message suivant : « Error C01A001D applying update operation ##### of ##### (######)” :

![Windows Update (KB) is stuck with the message, “Error C01A001D applying update operation X of Y (Z)” (Erreur C01A001D lors de l’application de l’opération de mise à jour ##### sur ##### (######) : l’opération Windows Update (en Ko) est bloquée et génère le message « Erreur C01A001D lors de l’application de l’opération de mise à jour X sur Y (Z) »).](./media/troubleshoot-windows-update-installation-capacity/1.png)

## <a name="cause"></a>Cause

Dans cette situation, le système d’exploitation ne parvient pas à effectuer l’installation de Windows Update (en Ko) car il est impossible de créer un fichier noyau dans le système de fichiers. D’après ce code d’erreur, le système d’exploitation ne peut pas écrire de fichiers sur le disque.

## <a name="solution"></a>Solution

### <a name="process-overview"></a>Vue d’ensemble du processus :

1. Créez une machine virtuelle de réparation et accédez-y.
1. Libérer de l’espace sur le disque.
1. Activez la console série et la collecte de l’image mémoire.
1. Regénérez la machine virtuelle.

> [!NOTE]
> Lorsque vous rencontrez cette erreur, cela signifie que le système d’exploitation invité n’est pas opérationnel. Résolvez ce problème en mode hors connexion.

### <a name="create-and-access-a-repair-vm"></a>Créer une machine virtuelle de réparation et y accéder

1. Effectuez les étapes 1 à 3 des [commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une machine virtuelle de réparation.
1. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="free-up-space-on-the-disk"></a>Libérer de l’espace sur le disque

Pour résoudre ce problème :

- Redimensionnez le disque jusqu’à 1 To s’il n’est pas déjà à la taille maximale (1 To).
- Procédez à un nettoyage de disque.
- Défragmentez le lecteur.

1. Vérifiez si le disque est plein. Si la taille du disque est inférieure à 1 To, augmentez-la jusqu’à un maximum de 1 To [à l’aide de PowerShell](../windows/expand-os-disk.md).
1. Si le disque a déjà une taille de 1 To, vous devrez effectuer un nettoyage de disque.
   1. Utilisez l’[outil Nettoyage de disque](https://support.microsoft.com/help/4026616/windows-10-disk-cleanup) pour libérer de l’espace.
1. Une fois le redimensionnement et le nettoyage terminés, défragmentez le lecteur à l’aide de la commande suivante :

   ```
   defrag <LETTER ASSIGN TO THE OS DISK>: /u /x /g
   ```
   
En fonction du niveau de fragmentation, la défragmentation peut prendre plusieurs heures.

### <a name="enable-the-serial-console-and-memory-dump-collection"></a>Activer la console série et la collecte de l’image mémoire

**Recommandé** : Avant de régénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire en exécutant le script suivant :

1. Ouvrez une session d’invite de commandes avec élévation de privilèges en tant qu’administrateur.
1. Exécutez les commandes suivantes :

   **Activer la console série** :
   
   ```
   bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON 
   bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200
   ```

1. Vérifiez que l’espace disponible sur le disque du système d’exploitation est supérieur à la taille de la mémoire (RAM) sur la machine virtuelle.

   Si l’espace n’est pas suffisant, changez l’emplacement où le fichier d’image mémoire sera créé et référencez-le sur n’importe quel autre disque de données attaché à la machine virtuelle possédant suffisamment d’espace libre. Pour changer l’emplacement, remplacez **%SystemRoot%** par la lettre de lecteur du disque de données (par exemple, **F:** ) dans les commandes ci-dessous.

   Configuration suggérée pour activer l’image mémoire du système d’exploitation :

    **Chargez le disque de système d’exploitation corrompu :**

   ```
   REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM 
   ```
   
   **Activez sur ControlSet001** :

   ```
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f
   ```
   
   **Activer sur ControlSet002** :

   ```
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f 
   REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f
   ```
   
   **Décharger le disque de système d’exploitation corrompu** :

   ```
   REG UNLOAD HKLM\BROKENSYSTEM
   ```
   
### <a name="rebuild-the-vm"></a>Régénérez la machine virtuelle.

Utilisez [l’étape 5 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour régénérer la machine virtuelle.
