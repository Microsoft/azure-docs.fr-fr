---
title: Erreur de démarrage – « Disque non amorçable »
description: Cet article décrit les étapes à suivre pour résoudre les problèmes liés à un disque non amorçable dans une machine virtuelle Azure.
services: virtual-machines-windows
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 5d6db4e3-c2f5-40c7-afea-54edf745f5eb
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
ms.date: 03/25/2020
ms.author: v-mibufo
ms.openlocfilehash: 16f6919577955bda5b04db26deb9fe78a467e364
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86509033"
---
# <a name="boot-error--this-is-not-a-bootable-disk"></a>Erreur de démarrage – Disque non amorçable

Cet article décrit les étapes à suivre pour résoudre les problèmes liés à un disque non amorçable dans une machine virtuelle Azure.

## <a name="symptoms"></a>Symptômes

Quand vous utilisez [Diagnostics de démarrage](./boot-diagnostics.md) pour visualiser la capture d’écran de la machine virtuelle, vous constatez que cette capture affiche une invite avec le message « Disque non amorçable. Veuillez insérer une disquette de démarrage et appuyer sur une touche pour réessayer... ».

   La figure 1

   ![La figure 1 illustre le message *« Disque non amorçable. Veuillez insérer une disquette de démarrage et appuyer sur une touche pour réessayer... »*](media/troubleshoot-guide-not-bootable-disk/1.jpg)

## <a name="cause"></a>Cause

Ce message d’erreur signifie que le processus de démarrage du système d’exploitation n’a pas pu localiser de partition système active. Cette erreur peut également indiquer qu’une référence manque dans le magasin BCD, ce qui l’empêche de localiser la partition Windows.

## <a name="solution"></a>Solution

### <a name="process-overview"></a>Vue d’ensemble du processus

1. Créer une machine virtuelle de réparation et y accéder.
2. Définir l’état de la partition sur Actif.
3. Réparer la partition de disque.
4. **Recommandé** : Avant de régénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire.
5. Régénérez la machine virtuelle d’origine.

   > [!NOTE]
   > Lorsque vous rencontrez cette erreur de démarrage, le système d’exploitation invité n’est pas opérationnel. Vous allez résoudre ce problème en mode hors connexion.

### <a name="create-and-access-a-repair-vm"></a>Créer une machine virtuelle de réparation et y accéder

1. Effectuez les étapes 1 à 3 des [commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une machine virtuelle de réparation.
2. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="set-partition-status-to-active"></a>Définir l’état de la partition sur Actif

Les machines virtuelles de première génération doivent d’abord vérifier que la partition du système d’exploitation, qui contient le magasin BCD, est marquée comme *active*. Si vous avez une machine virtuelle de deuxième génération, passez directement à l’étape [Réparer la partition de disque](#fix-the-disk-partition), car l’indicateur *État* a été déconseillé dans la dernière génération.

1. Ouvrez une invite de commandes avec élévation de privilèges *(cmd.exe)* .
2. Entrez *diskpart* pour lancer l’outil DISKPART.
3. Entrez *list disk* pour répertorier les disques sur le système et identifiez le disque dur virtuel du système d’exploitation attaché.
4. Une fois le disque dur virtuel du système d’exploitation attaché localisé, entrez *sel disk #* pour sélectionner le disque.  Voir la figure 2, où Disk 1 est le disque dur virtuel du système d’exploitation attaché.

   Figure 2

   ![La figure 2 illustre la fenêtre *DISKPART* qui montre le résultat de la commande list disk dans le tableau : Disk 0 et Disk 1.  Le résultat de la commande sel disk 1 est également affiché : Disk 1 est le disque sélectionné](media/troubleshoot-guide-not-bootable-disk/2.jpg)

5. Une fois le disque sélectionné, entrez *list partition* pour répertorier les partitions du disque sélectionné.
6. Une fois la partition de démarrage identifiée, entrez *sel partition #* pour sélectionner la partition.  La partition de démarrage utilise généralement environ 350 Mo.  Voir la figure 3, où Partition 1 est la partition de démarrage.

   Figure 3

   ![La figure 3 illustre la fenêtre *DISKPART* avec le résultat de la commande *list partition*. Partition 1 et Partition 2 apparaissent dans le tableau. Le résultat de la commande *sel partition 1* est également indiqué : Partition 1 est le disque sélectionné.](media/troubleshoot-guide-not-bootable-disk/3.jpg)

7. Entrez « detail partition » pour vérifier l’état de la partition. Voir la figure 4, où la partition présente l’état *Active: No* ou la figure 5, où la partition présente l’état « Active: Yes ».

   Figure 4

   ![La figure 4 illustre la fenêtre *DISKPART* avec le résultat de la commande *detail partition*, où Partition 1 est défini sur *Active: Non*](media/troubleshoot-guide-not-bootable-disk/4.jpg)

   Figure 5

   ![La figure 5 illustre la fenêtre *DISKPART* avec le résultat de la commande *detail partition*, où Partition 1 est défini sur *Active:  Yes*.](media/troubleshoot-guide-not-bootable-disk/5.jpg)

8. Si la partition présente l’état **Not Active**, entrez *active* pour modifier l’indicateur *Active*.
9. Vérifiez que le changement d’état s’est effectué correctement en entrant *detail partition*.

   Figure 6

   ![La figure 6 illustre la fenêtre diskpart avec le résultat de la commande *detail partition*, où Partition 1 est défini sur *Active: Oui*](media/troubleshoot-guide-not-bootable-disk/6.jpg)

10. Entrez *exit* pour fermer l’outil DISKPART et enregistrer les modifications apportées à la configuration.

### <a name="fix-the-disk-partition"></a>Réparer la partition de disque

1. Ouvrez une invite de commandes avec élévation de privilèges (cmd.exe).
2. Utilisez la commande suivante pour exécuter *CHKDSK* sur le(s) disque(s) et réparer les erreurs :

   `chkdsk <DRIVE LETTER>: /f`

   L’ajout de l’option de commande '/f' résout toutes les erreurs sur le disque. Veillez à remplacer <DRIVE LETTER> par la lettre correspondant au disque dur virtuel du système d’exploitation attaché.

### <a name="recommended-before-you-rebuild-the-vm-enable-serial-console-and-memory-dump-collection"></a>Recommandé : Avant de régénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire

Pour activer la collecte de l’image mémoire et la console série, exécutez le script suivant :

1. Ouvrez une session d’invite de commandes avec élévation de privilèges (Exécuter en tant qu’administrateur).
2. Exécutez les commandes suivantes :

   Activer la console série

   `bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON`

   `bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200`

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

Utilisez l’[étape 5 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour réassembler la machine virtuelle.
