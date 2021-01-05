---
title: Erreur d’arrêt de Windows - 0x00000074 Informations de configuration système incorrectes
description: Cet article décrit la procédure à suivre pour résoudre les problèmes empêchant Windows de démarrer et l’obligeant à redémarrer en raison d’informations de configuration système incorrectes dans une machine virtuelle Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: a1e47f6a-c7d5-4327-a7b0-ad48ed543641
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 08/24/2020
ms.author: v-miegge
ms.openlocfilehash: 7d1233c97ec80d5a2efa8b53c68e9e07a823165d
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91977029"
---
# <a name="windows-stop-error---0x00000074-bad-system-config-info"></a>Erreur d’arrêt de Windows - 0x00000074 Informations de configuration système incorrectes

Cet article décrit la procédure à suivre pour résoudre les problèmes empêchant Windows de démarrer et l’obligeant à redémarrer en raison d’informations de configuration système incorrectes dans une machine virtuelle Azure.

## <a name="symptom"></a>Symptôme

Lorsque vous utilisez les [Diagnostics de démarrage](./boot-diagnostics.md) pour afficher la capture d’écran de la machine virtuelle, vous pouvez voir la capture d’écran montrant le code d’arrêt Windows l’échec de l’installation de Windows **#0x00000074** ou **BAD_SYSTEM_CONFIG_INFO**.

*Votre PC a rencontré un problème et doit redémarrer. Vous pouvez redémarrer.* 
*Pour plus d’informations sur ce problème ainsi que les correctifs possibles, visitez http://windows.com/stopcode* 
*Si vous appelez le support technique, communiquez à votre interlocuteur les informations suivantes :* 
*Code d’arrêt : BAD_SYSTEM_CONFIG_INFO*

  ![Le code d’arrêt Windows 0x00000074, également indiqué par « BAD_SYSTEM_CONFIG_INFO ». Windows informe l’utilisateur que son ordinateur a rencontré un problème et doit redémarrer.](./media/windows-stop-error-bad-system-config-info/1.png)

## <a name="cause"></a>Cause :

Le code d’arrêt **BAD_SYSTEM_CONFIG_INFO** intervient en cas de corruption de la ruche du Registre **SYSTÈME**. Cette erreur peut être due à l’une de ces raisons :

- La ruche du Registre n’a pas été fermée correctement.
- La ruche du Registre est corrompue.
- Des clés de registre ou des valeurs sont manquantes.

## <a name="solution"></a>Solution

### <a name="process-overview"></a>Vue d’ensemble du processus :

1. Créez une machine virtuelle de réparation et accédez-y.
1. Recherchez une éventuelle corruption de la ruche.
1. Activer la console série et la collecte de l’image mémoire.
1. Regénérez la machine virtuelle.

> [!NOTE]
> Lorsque vous rencontrez cette erreur, cela signifie que le système d’exploitation (SE) invité n’est pas opérationnel. Vous devez résoudre ce problème en mode hors connexion.

### <a name="create-and-access-a-repair-vm"></a>Créer une machine virtuelle de réparation et y accéder

1. Effectuez les étapes 1 à 3 des [commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une machine virtuelle de réparation.
1. Recherchez une éventuelle corruption de la ruche.
1. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.
1. Copiez le dossier `<VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config` et enregistrez-le dans votre partition de disque saine ou dans un autre emplacement sécurisé. Sauvegardez ce dossier par précaution, car vous allez modifier les fichiers de Registre critiques. 

> [!NOTE]
> Effectuez une copie du dossier `<VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config` en guise de sauvegarde au cas où il vous faudrait restaurer les modifications que vous apportez au registre.

### <a name="check-for-hive-corruption"></a>Rechercher une éventuelle corruption de la ruche

Les instructions ci-dessous vous aideront à déterminer si la cause est due à une corruption de la ruche ou si la ruche n’a pas été fermée correctement. Si la ruche n’a pas été fermée correctement, vous pourrez déverrouiller le fichier et corriger votre machine virtuelle.

1. Sur la machine virtuelle de réparation, ouvrez l’applicaiton **Éditeur du Registre**. Entrez « REGEDIT » dans la barre de recherche Windows pour la trouver.
1. Dans l’Éditeur du Registre, sélectionnez **HKEY_LOCAL_MACHINE** pour le mettre en surbrillance, puis sélectionnez **Fichier > Charger la ruche...** .
1. Accédez à `<VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM` et sélectionnez **Ouvrir**.
1. Lorsque vous êtes invité à entrer un nom, entrez **BROKENSYSTEM**.

   1. Si la ruche ne s’ouvre pas ou si elle est vide, cela signifie qu’elle est corrompue. Si la ruche est corrompue, [ouvrez un ticket de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

     ![Un message d’erreur indique que l’Éditeur du Registre ne peut pas charger la ruche.](./media/windows-stop-error-bad-system-config-info/2.png)

   1. Si la ruche s’ouvre normalement, cela signifie que la ruche n’a pas été fermée correctement. Passez à l’étape 5.

1. Pour corriger une ruche qui n’a pas été fermée correctement, mettez en surbrillance **BROKENSYSTEM** puis sélectionnez **Fichier > Décharger la ruche...** pour déverrouiller le fichier.

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

Utilisez [l’étape 5 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour régénérer la machine virtuelle.