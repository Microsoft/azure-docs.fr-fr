---
title: Résoudre une erreur de démarrage - Une erreur de lecture de disque s’est produite
description: Cet article décrit les étapes à suivre pour résoudre les problèmes liés à un disque illisible dans une machine virtuelle Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 817c9c5c-6a81-4b42-a6ad-0a0a11858b84
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 06/01/2020
ms.author: v-miegge
ms.openlocfilehash: f59903ed111be1fe414f4b3ded250d754c91d323
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87069151"
---
# <a name="troubleshoot-boot-error---disk-read-error-occurred"></a>Résoudre une erreur de démarrage - Une erreur de lecture de disque s’est produite

Cet article décrit les étapes à suivre pour résoudre les problèmes liés à un disque illisible dans une machine virtuelle Azure.

## <a name="symptoms"></a>Symptômes

Lorsque vous utilisez [Diagnostics de démarrage](./boot-diagnostics.md) pour visualiser la capture d’écran de la machine virtuelle, vous constatez que cette capture affiche une invite avec le message « Une erreur de lecture de disque s'est produite. Appuyez sur Ctrl+Alt+Suppr pour redémarrer ».

   ![Message d’erreur : Une erreur de lecture de disque s’est produite. Appuyez sur Ctrl+Alt+Suppr pour redémarrer.](./media/disk-read-error-occurred/1.png)

## <a name="cause"></a>Cause

Ce message d’erreur indique que la structure du disque est endommagée et illisible. Si vous utilisez une machine virtuelle de 1ère génération, il est également possible que la partition de disque contenant les données de configuration de démarrage ne soit pas définie sur **Active**.

## <a name="solution"></a>Solution

### <a name="process-overview"></a>Vue d’ensemble du processus

1. Créez une machine virtuelle de réparation et accédez-y.
1. Sélectionner une solution :
   - [Définir l’état de la partition sur Actif](#set-partition-status-to-active)
   - [Réparer la partition de disque](#fix-the-disk-partition)
1. Activez la console série et la collecte de l’image mémoire.
1. Regénérez la machine virtuelle.

> [!NOTE]
> Lorsque vous rencontrez cette erreur de démarrage, cela signifie que le système d’exploitation (SE) invité n’est pas opérationnel. Vous allez résoudre ce problème en mode hors connexion.

### <a name="create-and-access-a-repair-vm"></a>Créer une machine virtuelle de réparation et y accéder

1. Effectuez les étapes 1 à 3 des [commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une machine virtuelle de réparation.
1. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="set-partition-status-to-active"></a>Définir l’état de la partition sur Actif

Les machines virtuelles de première génération doivent d’abord vérifier que la partition du système d’exploitation, qui contient le magasin BCD, est marquée comme **Active**. Si vous avez une machine virtuelle de deuxième génération, passez directement à l’étape [Réparer la partition de disque](#fix-the-disk-partition), car l’indicateur d'état a été déconseillé dans la dernière génération.

1. Ouvrez une invite de commandes avec élévation de privilèges (cmd.exe).
1. Entrez **diskpart** pour lancer l’outil **DISKPART**.
1. Entrez **list disk** pour répertorier les disques sur le système et identifiez le disque dur virtuel (VHD) du système d’exploitation attaché.
1. Une fois le disque dur virtuel du système d’exploitation attaché localisé, entrez **sel disk #** pour sélectionner le disque. Reportez-vous à l’image suivante pour un exemple d’emplacement où Disk 1 correspond au disque dur virtuel du système d’exploitation attaché.

   ![La fenêtre diskpart avec le résultat de la commande **list disk**, où Disk 0 et Disk 1 apparaissent dans le tableau. La fenêtre affiche également le résultat de la commande **sel disk 1**, où Disk 1 correspond au disque sélectionné](./media/disk-read-error-occurred/2.png)

1. Une fois le disque sélectionné, entrez **list partition** pour répertorier les partitions du disque sélectionné.
1. Une fois la partition de démarrage identifiée, entrez **sel partition #** pour sélectionner la partition. La taille de la partition de démarrage est généralement d’environ 350 Mo.  Reportez-vous à l’image suivante pour un image d’emplacement où Partition 1 correspond à la partition de démarrage.

   ![La fenêtre diskpart avec le résultat de la commande **list partition**, où Partition 1 et Partition 2 apparaissent dans le tableau. La fenêtre affiche également le résultat de la commande **sel partition 1**, où Partition 1 correspond au disque sélectionné.](./media/disk-read-error-occurred/3.png)

1. Entrez **detail partition** pour vérifier l’état de la partition. Consultez les captures d’écran suivantes pour des exemples de partition définie sur **Active : Non** ou **Active : Oui**.

   **Active : Non**

   ![La fenêtre diskpart avec le résultat de la commande **detail partition**, où Partition 1 est définie sur **Active : Non**.](./media/disk-read-error-occurred/4.png)

   **Active : Oui**

   ![La fenêtre diskpart avec le résultat de la commande **detail partition**, où partition 1 est définie sur **Active : Oui**.](./media/disk-read-error-occurred/5.png)

1. Si la partition n’est pas définie sur **Active**, entrez **active** pour modifier l’indicateur Active.
1. Entrez **detail partition** pour vérifier que le changement d’état a bien été effectué, et vérifiez que la sortie contient **Active : Oui**. 
1. Entrez **exit** pour fermer l’outil DISKPART et enregistrer les modifications apportées à la configuration.

### <a name="fix-the-disk-partition"></a>Réparer la partition de disque

1. Ouvrez une invite de commandes avec élévation de privilèges (cmd.exe).
1. Utilisez la commande suivante pour exécuter **CHKDSK** sur le(s) disque(s) et corriger les erreurs :

   `chkdsk <DRIVE LETTER>: /f`

   L’ajout de l’option de commande **/f** résout toutes les erreurs sur le disque. Veillez à remplacer **< DRIVE LETTER >** par la lettre correspondant au disque dur virtuel du système d’exploitation attaché.

### <a name="enable-the-serial-console-and-memory-dump-collection"></a>Activer la console série et la collecte de l’image mémoire

**Recommandé** : Avant de régénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire en exécutant le script suivant :

1. Ouvrez une session d’invite de commandes avec élévation de privilèges en tant qu’administrateur.
1. Exécutez les commandes suivantes :

   **Activer la console série** :
   
   ```
   bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON 
   bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200
   ```

1. vérifiez que l’espace disponible sur le disque du système d’exploitation est supérieur à la taille de la mémoire (RAM) sur la machine virtuelle.

   Si l’espace n’est pas suffisant, changez l’emplacement où le fichier d’image mémoire sera créé et référencez-le sur n’importe quel autre disque de données attaché à la machine virtuelle possédant suffisamment d’espace libre. Pour changer l’emplacement, remplacez **%SystemRoot%** par la lettre de lecteur du disque de données (par exemple, **F:** ) dans les commandes ci-dessous.

   Configuration suggérée pour activer l’image mémoire du système d’exploitation :

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
   
### <a name="rebuild-the-vm"></a>Régénérez la machine virtuelle.

Utilisez [l’étape 5 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour régénérer la machine virtuelle.
