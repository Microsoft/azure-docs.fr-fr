---
title: Impossible de démarrer une machine virtuelle Windows en raison du Gestionnaire de démarrage Windows
description: Cet article explique comment résoudre des problèmes où le Gestionnaire de démarrage Windows empêche le démarrage d’une machine virtuelle Azure.
services: virtual-machines-windows
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: a97393c3-351d-4324-867d-9329e31b3598
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
ms.date: 03/26/2020
ms.author: v-mibufo
ms.openlocfilehash: 2457952051f575306de46e3e8145cc26678a1ef8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86526536"
---
# <a name="windows-vm-cannot-boot-due-to-windows-boot-manager"></a>Impossible de démarrer une machine virtuelle Windows à cause du Gestionnaire de démarrage Windows

Cet article explique comment résoudre des problèmes où le Gestionnaire de démarrage Windows empêche le démarrage d’une machine virtuelle (MV) Azure.

## <a name="symptom"></a>Symptôme

La machine virtuelle est bloquée en attente d’une invite utilisateur et ne peut démarrer que sur commande manuelle.

Lorsque vous utilisez les [Diagnostics de démarrage](./boot-diagnostics.md) pour afficher la capture d’écran de la machine virtuelle, vous voyez sur cette capture d’écran le Gestionnaire de démarrage Windows afficher avec le message *Choisissez un système d’exploitation ou appuyez sur TAB pour choisir un outil :* .

Figure 1
 
![Gestionnaire de démarrage Windows indiquant « Choisissez un système d’exploitation ou appuyez sur TAB pour choisir un outil : »](media/troubleshoot-guide-windows-boot-manager-menu/1.jpg)

## <a name="cause"></a>Cause

L’erreur est due à un indicateur BCD *displaybootmenu* dans le Gestionnaire de démarrage Windows. Lorsque l’indicateur est activé, le Gestionnaire de démarrage Windows invite l’utilisateur, pendant le processus de démarrage, à sélectionner le chargeur qu’il souhaite exécuter, ce qui provoque un retard de démarrage. Dans Azure, cette fonctionnalité peut allonger temps nécessaire au démarrage d’une machine virtuelle.

## <a name="solution"></a>Solution

Vue d’ensemble du processus :

1. Configurez un temps de démarrage plus rapide à l’aide d’une console série.
2. Créez une machine virtuelle de réparation et accédez-y.
3. Configurez un temps de démarrage plus rapide sur une machine virtuelle de réparation.
4. **Recommandé** : Avant de regénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire.
5. Regénérez la machine virtuelle.

### <a name="configure-for-faster-boot-time-using-serial-console"></a>Configurer un temps de démarrage plus rapide à l’aide d’une console série

Si vous avez accès à une console série, vous disposez de deux méthodes pour obtenir des temps de démarrage plus rapides. Réduisez le temps d’attente *displaybootmenu* ou supprimez complètement l’indicateur.

1. Suivez les instructions pour accéder à la [console série Azure pour Windows](./serial-console-windows.md) afin d’obtenir l’accès à la console textuelle.

   > [!NOTE]
   > Si vous ne pouvez pas accéder à une console série, passez directement à la section [Créer une machine virtuelle de réparation et y accéder](#create-and-access-a-repair-vm).

2. **Option A** : Réduire le délai d’attente

   a. Le délai d’attente est défini par défaut sur 30 secondes, mais il est possible de le raccourcir (par exemple, à 5 secondes).

   b. Utilisez la commande suivante dans la console série pour ajuster la valeur du délai d’attente :

      `bcdedit /set {bootmgr} timeout 5`

3. **Option B** : Supprimer l’indicateur BCD

   a. Pour empêcher l’affichage du menu de démarrage, entrez la commande suivante :

      `bcdedit /deletevalue {bootmgr} displaybootmenu`

      > [!NOTE]
      > Si vous n’avez pas pu utiliser de console série pour configurer un temps de démarrage plus court aux étapes ci-dessus, vous pouvez passer aux étapes suivantes. Vous allez à présent résoudre ce problème en mode hors connexion.

### <a name="create-and-access-a-repair-vm"></a>Créer une machine virtuelle de réparation et y accéder

1. Suivez les [étapes 1 à 3 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une machine virtuelle de réparation.
2. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="configure-for-faster-boot-time-on-a-repair-vm"></a>Configurer un temps de démarrage plus rapide sur une machine virtuelle de réparation

1. Ouvrez une invite de commandes avec élévation de privilèges.
2. Entrez les informations suivantes pour activer DisplayBootMenu :

   Utilisez cette commande pour les **machines virtuelles de 1ère génération** :

   `bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /set {bootmgr} displaybootmenu yes`

   Utilisez cette commande pour **les machines virtuelles de 2e génération** :

   `bcdedit /store <VOLUME LETTER OF EFI SYSTEM PARTITION>:EFI\Microsoft\boot\bcd /set {bootmgr} displaybootmenu yes`

   Remplacez les symboles supérieur à ou inférieur à, ainsi que le texte inséré entre eux, par exemple, « < texte ici > ».

3. Cette fois, définissez la valeur de délai d’attente sur 5 secondes :

   Utilisez cette commande pour les **machines virtuelles de 1ère génération** :

   `bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /set {bootmgr} timeout 5`

   Utilisez cette commande pour les **machines virtuelles de 2e génération** :

   `bcdedit /store <VOLUME LETTER OF EFI SYSTEM PARTITION>:EFI\Microsoft\boot\bcd /set {bootmgr} timeout 5`

   Remplacez les symboles supérieur à ou inférieur à, ainsi que le texte inséré entre eux, par exemple, « < texte ici > ».

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
