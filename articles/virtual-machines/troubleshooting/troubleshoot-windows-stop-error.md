---
title: Erreur d’arrêt Windows - État Pas de mémoire
description: 'Cet article décrit les étapes à suivre pour résoudre les problèmes pour lesquels Windows ne démarre pas et affiche l’état « État : aucune mémoire ».'
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: d29c7e49-4578-43c8-b829-831da4e48932
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 06/26/2020
ms.author: v-mibufo
ms.openlocfilehash: 33b4c59e14301e496d0eddafa7bdfdf201b7aa29
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87005903"
---
# <a name="windows-stop-error---status-no-memory"></a>Erreur d’arrêt Windows - État Pas de mémoire

Cet article décrit les étapes à suivre pour résoudre les problèmes pour lesquels Windows ne démarre pas et affiche l’état ou le code d’erreur 0xC0000017, également appelé « État : aucune mémoire ».

## <a name="symptom"></a>Symptôme

Lorsque vous utilisez des [Diagnostics de démarrage](./boot-diagnostics.md) pour afficher la capture d'écran de la machine virtuelle, vous constatez que la capture d'écran affiche le code d’erreur : `0xC0000017`. En fonction de la version de Windows exécutée, ce code peut apparaître dans le **Gestionnaire de démarrage Windows** ou sur l’**écran de récupération**.

   **Gestionnaire de démarrage Windows**

   ![Le Gestionnaire de démarrage Windows indique « Échec de démarrage de Windows. Une modification récente du matériel ou du logiciel pourrait être la cause de l’incident. » En faisant défiler la page vers le bas, le code d’état « 0xC0000017 » s’affiche, ainsi que les informations indiquant que « l’application ou le système d’exploitation n’a pas pu être chargé, car un fichier requis est manquant ou contient des erreurs ».](./media/troubleshoot-windows-stop-error/1.png)

   **Écran de récupération**
 
   ![L’écran de récupération indique « Votre ordinateur/périphérique doit être réparé. La mémoire disponible est insuffisante pour créer un périphérique ramdisk ». Le code d’erreur « 0xC0000017 » peut également s’afficher.](./media/troubleshoot-windows-stop-error/2.png)

## <a name="cause"></a>Cause

Le disque du système d’exploitation est saturé, trop fragmenté ou le système d’exploitation (SE) n’est pas en mesure d’accéder à la mémoire ou au fichier d’échange ou bien les deux.

## <a name="solution"></a>Solution

### <a name="process-overview"></a>Vue d’ensemble du processus :

1. Créer une machine virtuelle de réparation et y accéder
1. Libérer de l’espace sur le disque
1. Nettoyer la mémoire incorrecte dans le Store BCD
1. Restaurer le fichier d’échange à son emplacement par défaut
1. Activer la console série et la collecte de l’image mémoire
1. Régénérez la machine virtuelle.

> [!NOTE]
> Lorsque vous rencontrez cette erreur, le système d’exploitation invité n’est pas opérationnel. Vous allez résoudre ce problème en mode hors connexion.

### <a name="create-and-access-a-repair-vm"></a>Créer une machine virtuelle de réparation et y accéder

1. Suivez les [étapes 1 à 3 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une machine virtuelle de réparation.
1. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="for-generation-2-vms-assign-a-letter-to-the-extensible-firmware-interface-efi-partition"></a>Pour les machines virtuelles de 2e génération, attribuez une lettre à la partition Extensible Firmware Interface (EFI) :

Si vous utilisez une machine virtuelle de 2e génération, il se peut que la partition EFI du disque joint n’ait pas de lettre attribuée. Vous devez suivre les étapes ci-dessous pour attribuer une lettre à la partition avant de poursuivre la lecture de ce guide de dépannage.

1. Dans Windows Search, entrez `diskmgmt`, puis ouvrez la **Console de gestion du disque**.
1. Identifiez le disque défectueux joint à la machine virtuelle de réparation. En règle générale, ce disque est répertorié en dernier dans la console et a la valeur numérique la plus élevée.
1. Notez que, dans ce disque, il y a une partition qui contient la **Partition système EFI**, qui n’a pas non plus de valeur de lettre attribuée (par exemple, le lecteur *F :* ). Si toutes les partitions sont affectées, vous pouvez passer directement à la libération d’espace sur le disque. Dans le cas contraire, continuez à attribuer une lettre à ce disque.

   ![La console de gestion des disques, avec le disque joint « disque 2 », ainsi que la partition non attribuée de 100 Mo qui est la « Partition du système EFI ».](./media/troubleshoot-windows-stop-error/3.png)

1. Ouvrez une invite de commandes avec élévation de privilèges en tant qu'administrateur et entrez `diskpart` pour lancer l'outil **DISKPART**.
1. Entrez les commandes suivantes :

   ```
   list disk
   sel disk <NUMBER OF THE ATTACHED DISK>
   list partition
   sel partition <NUMBER OF THE SYSTEM (EFI) PARTION>
   assign
   ```
   
   - Dans la commande, remplacez `<NUMBER OF THE ATTACHED DISK>` par le numéro du disque que vous avez identifié à l’étape 2.
   - Dans la commande, remplacez `<NUMBER OF THE SYSTEM PARTION>` par le numéro de partition de la partition système EFI. Aucune lettre n’a encore été attribuée à cette partition, mais elle doit être de type **Système** et avoir une taille d’environ 100 Mo.

   > [!NOTE]
   > La comparaison des partitions de la console de gestion des disques avec celles figurant dans l’outil DISKPART peut être utile pour déterminer le numéro de partition qui correspond à la partition système EFI dans le disque joint.

1. Fermez la fenêtre d'invite de commandes.

### <a name="free-up-space-on-the-disk"></a>Libérer de l’espace sur le disque

Maintenant que le disque défectueux est joint à la machine virtuelle de réparation, vous devez vérifier que le système d’exploitation sur ce disque dispose de suffisamment d’espace pour fonctionner correctement. 

1. Vérifiez si le disque est plein en cliquant avec le bouton de droite sur le lecteur du disque joint et en sélectionnant **Propriétés**.
1. Si le disque dispose de **moins de 300 Mo d’espace libre**, [augmentez-le jusqu’à un maximum de 1 To en utilisant PowerShell](../windows/expand-os-disk.md).
1. Une fois la taille de **1 To** atteinte, vous devrez effectuer un nettoyage du disque. Vous pouvez utiliser l’[outil Nettoyage de disque](https://support.microsoft.com/help/4026616/windows-10-disk-cleanup) pour libérer de l’espace.
1. Ouvrez une instance d’invite de commandes avec élévation de privilèges (exécuter en tant qu’administrateur) et effectuez une défragmentation sur le lecteur :

   ``
   defrag <LETTER ASSIGNED TO THE OS DISK>: /u /x /g
   ``
   
   - En fonction du niveau de fragmentation, la défragmentation peut prendre plusieurs heures.
   - Dans la commande, remplacez `<LETTER ASSIGNED TO THE OS DISK>` par la lettre du disque du système d’exploitation (tel que le disque *F:* ).

### <a name="clean-out-bad-memory-from-the-boot-configuration-data-bcd-store"></a>Nettoyer la mémoire incorrecte du Store Données de configuration de démarrage (BCD)

1. Ouvrez une invite de commandes avec élévation de privilèges (exécutée en tant qu’administrateur).
1. Interrogez le fichier de configuration de démarrage pour rechercher les indicateurs de mémoire incorrecte à l’aide de la commande suivante :

   ``
   bcdedit /store <LETTER ASSIGNED TO THE OS DISK>:\boot\bcd /enum {badmemory}
   ``
   
   - Dans la commande, remplacez `<LETTER ASSIGNED TO THE OS DISK>` par la lettre du disque du système d’exploitation (tel que le disque *F:* ).

1. Si la requête n’affiche aucun bloc de mémoire incorrect, passez à la tâche suivante. Sinon, passez à l'étape 4.
1. Si des blocs de mémoire incorrects sont identifiés, ces blocs empêchent la création d’un ramdisk et doivent être supprimés à l’aide de la commande suivante :

   ``
   bcdedit /store <LETTER ASSIGNED TO THE OS DISK>:\boot\bcd /deletevalue {badmemory} badmemorylist
   ``
   
   - Dans la commande, remplacez `<LETTER ASSIGNED TO THE OS DISK>` par la lettre du disque du système d’exploitation (tel que le disque *F:* ).

### <a name="restore-the-page-file-to-its-default-location"></a>Restaurer le fichier d’échange à son emplacement par défaut

Le fichier d’échange stocke les données qui ne peuvent pas être conservées dans la mémoire vive (RAM) de votre ordinateur sous forme de dépassement de capacité/sauvegarde. Il est possible que le fichier soit hébergé dans un disque dur virtuel plutôt que dans le lecteur temporaire, qui est l’emplacement Azure par défaut. Si la valeur est true, le fichier peut ne pas être accessible et doit être restauré à l’emplacement par défaut.

Avant de suivre une procédure, vous devez créer une copie du dossier **\Windows\System32\Config** sur un disque sain. Cette étape permet de s’assurer que toutes les modifications indésirables peuvent être annulées. Vous allez travailler sur des fichiers système importants. cette précaution est donc fortement recommandée.

1. Dans Windows Search, entrez **regedit** et ouvrez l’application Éditeur du Registre.
1. Dans l’Éditeur du Registre, mettez en surbrillance la clé **HKEY_LOCAL_MACHINE**, puis sélectionnez **Fichier > Charger Hive...** à partir du menu.

   ![Le menu charger Hive de l’éditeur du Registre.](./media/troubleshoot-windows-stop-error/4.png)

1. Dans la boîte de dialogue Charger Hive, sélectionnez **\windows\system32\config\SYSTEM**, puis cliquez sur Ouvrir.
   1. Vous êtes invité à entrer un nom. Entrez **BROKENSYSTEM**. Ce nom permet de différencier les ruches affectées pendant le dépannage.
   1. Développez **HKEY_LOCAL_MACHINE** pour voir la nouvelle clé BROKENSYSTEM que vous avez ajoutée.
1. À l’aide de l’Éditeur du Registre, déterminez les ControlSet à partir desquels l’ordinateur démarre.
   1. Accédez à **HKEY_LOCAL_MACHINE > > BROKENSYSTEM > > Sélectionner**.
   1. Dans les clés répertoriées, notez la valeur de données de la clé actuelle. Par exemple, si cette valeur est **1** ou **0x00000001 (1)** , le jeu de contrôles est ControlSet001.
1. Vérifiez l’emplacement où la création du PageFile est configurée.
   1. Dans HKEY_LOCAL_MACHINE \BROKENSYSTEM, développez le répertoire correspondant au numéro de ControlSet que vous avez identifié à l’étape 4, par exemple **ControlSet001**.
   1. Accédez à **Contrôle > > Gestionnaire de sessions > > Gestion de la mémoire** et notez l’emplacement de la clé de **ExistingPageFiles**.
   1. Cette clé doit être dans l’emplacement Azure par défaut du lecteur temporaire. S’il n’y figure pas et qu’il se trouve sur un disque dur virtuel à un autre emplacement, tel que le lecteur de disque de données ou le lecteur de système d’exploitation, il doit être supprimé.
   1. Accédez à cet emplacement dans l’Explorateur de fichiers, puis supprimez le fichier **pagefile.sys**.

### <a name="enable-the-serial-console-and-memory-dump-collection"></a>Activer la console série et la collecte de l’image mémoire

**Recommandé** : Avant de régénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire en exécutant le script suivant :

Pour activer la collecte de l’image mémoire et la console série, exécutez le script suivant :

1. Ouvrez une session d’invite de commandes avec élévation de privilèges en tant qu’administrateur.
1. Répertoriez les données du Store BCD et déterminez l’identificateur du chargeur de démarrage, que vous utiliserez à l’étape suivante.

   1. Pour une machine virtuelle de 1ère génération, entrez la commande suivante et notez l’identificateur indiqué :
   
      ``
      bcdedit /store <BOOT PARTITON>:\boot\bcd /enum
      ``
   
   - Dans la commande, remplacez `<BOOT PARTITON>` par la lettre de la partition du disque joint qui contient le dossier de démarrage.

      ![Le résultat du listing du Store BCD dans une machine virtuelle de 1ère génération, qui répertorie sous le Chargeur de démarrage Windows le numéro d’identificateur.](./media/troubleshoot-windows-stop-error/5.png)

   1. Pour une machine virtuelle de 2e génération, entrez la commande suivante et notez l’identificateur indiqué :
   
      ``
      bcdedit /store <LETTER OF THE EFI SYSTEM PARTITION>:EFI\Microsoft\boot\bcd /enum
      ``
   
   - Dans la commande, remplacez `<LETTER OF THE EFI SYSTEM PARTITION>` par la lettre de la partition système EFI.
   - Il peut être utile de lancer la console de gestion des disques pour identifier la partition système appropriée nommée **Partition système EFI**.
   - L’identificateur peut être un GUID unique ou la valeur par défaut **bootmgr**.

1. Exécutez les commandes suivantes pour activer la Console série :

   ```
   bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON 
   bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200
   ```
   
   - Dans la commande, remplacez `<VOLUME LETTER WHERE THE BCD FOLDER IS>` par la lettre du dossier BCD.
   - Dans la commande, remplacez `<BOOT LOADER IDENTIFIER>` par l’identificateur que vous avez trouvé à l’étape précédente.

1. vérifiez que l’espace disponible sur le disque du système d’exploitation est supérieur à la taille de la mémoire (RAM) sur la machine virtuelle.

   Si l’espace n’est pas suffisant, changez l’emplacement où le fichier d’image mémoire sera créé et référencez-le sur n’importe quel autre disque de données attaché à la machine virtuelle possédant suffisamment d’espace libre. Pour changer l’emplacement, remplacez **%SystemRoot%** par la lettre de lecteur du disque de données (par exemple, **F:** ) dans les commandes ci-dessous.

   Configuration suggérée pour activer l’image mémoire du système d’exploitation :

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
