---
title: 'Résoudre les problèmes liés à l’erreur de Gestionnaire de démarrage Windows : 0xC0000225 « État introuvable »'
description: Étapes pour résoudre les problèmes où le code d’erreur 0xC0000225 se produit dans une machine virtuelle Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: c17899a4-b270-4725-9530-0dcd829b178c
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 05/11/2020
ms.author: v-miegge
ms.openlocfilehash: ff7d5a4e1181dccedc3584d958038a1d695c57ca
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97657122"
---
# <a name="troubleshoot-windows-boot-manager-error----0xc0000225-status-not-found"></a>Résoudre les problèmes liés à l’erreur de Gestionnaire de démarrage Windows : 0xC0000225 « État introuvable »
 
Cet article fournit des étapes pour résoudre les problèmes où le code d’erreur 0xC0000225 se produit dans une machine virtuelle Azure. Cette erreur indique que l’état ou l’objet est introuvable.

## <a name="symptoms"></a>Symptômes

Quand vous utilisez les [diagnostics de démarrage](./boot-diagnostics.md) pour afficher la capture d’écran de la machine virtuelle, vous pouvez voir que cette capture affiche l’erreur « Windows n’a pas pu démarrer » avec le code d’état *0xC0000225*.

Le fichier associé à ce code d’erreur vous indiquera les étapes à suivre pour résoudre le problème. Recherchez le texte de la section **File:** pour déterminer la marche à suivre.

### <a name="drivers-os-related-or-third-party"></a>Pilotes, système d’exploitation ou tiers

Si le fichier est présent, mais qu’il fait référence à un pilote (comme indiqué) à un système d’exploitation ou à un tiers, suivez les étapes de la section [Réparer le fichier système](#repair-the-system-file).
 
Dans l’image suivante, Gestionnaire de démarrage Windows indique « Windows n’a pas pu démarrer. Une modification récente du matériel ou du logiciel pourrait être la cause de l’incident. » L’image indique également l’état « 0xC0000225 », **File:** en tant que `\windows\System32\drivers\atapi.sys` et **Info:** en tant que « Le système d’exploitation n’a pas pu être chargé, car un pilote système critique est manquant ou erroné. »

![Gestionnaire de démarrage Windows indique « Windows n’a pas pu démarrer. Une modification récente du matériel ou du logiciel pourrait être la cause de l’incident. » Il indique également l’état « 0xC0000225 », le fichier « \windows\System32\drivers\atapi.sys » et la section Info en tant que : « Le système d’exploitation n’a pas pu être chargé, car un pilote système critique est manquant ou erroné. »](./media/troubleshoot-boot-error-status-not-found/1.png)

### <a name="no-file"></a>Aucun fichier

S’il est présenté avec le code d’état, mais qu’aucun fichier n’est affiché, suivez les étapes de la section [Ajouter la variable OSDEVICE](#add-the-osdevice-variable).

Dans l’image suivante, Gestionnaire de démarrage Windows indique « Windows n’a pas pu démarrer. Une modification récente du matériel ou du logiciel pourrait être la cause de l’incident. » L’image indique également l’état « 0xC0000225 » et **Info:** en tant que « La sélection de démarrage a échoué, car un périphérique nécessaire est inaccessible. »

![Gestionnaire de démarrage Windows indique « Windows n’a pas pu démarrer. Une modification récente du matériel ou du logiciel pourrait être la cause de l’incident. » Il indique également l’état « 0xC0000225 » et la section Info en tant que : « La sélection de démarrage a échoué, car un périphérique nécessaire est inaccessible. »](./media/troubleshoot-boot-error-status-not-found/2.png)

### <a name="registry-file"></a>Fichier du Registre

S’il fait référence à l’un des fichiers du Registre, par exemple \windows\system32\config\system, suivez les étapes de la section [Créer un ticket de support](#contact-support).
 
Dans l’image suivante, Gestionnaire de démarrage Windows indique « Windows n’a pas pu démarrer. Une modification récente du matériel ou du logiciel pourrait être la cause de l’incident. » L’image indique également l’état « 0xC0000225 », le fichier `\windows\System32\config\system` et **Info:** en tant que « Échec du chargement du système d’exploitation, car le fichier du Registre système est manquant ou contient des erreurs. »

![Gestionnaire de démarrage Windows indique « Windows n’a pas pu démarrer. Une modification récente du matériel ou du logiciel pourrait être la cause de l’incident. » Il indique également l’état « 0xC0000225 », le fichier « \windows\System32\config\system » et la section Info en tant que : « Échec du chargement du système d’exploitation, car le fichier du Registre système est manquant ou contient des erreurs. »](./media/troubleshoot-boot-error-status-not-found/3.png)

Dans l’image suivante, l’écran de récupération indique « Votre ordinateur/périphérique doit être réparé. Le système d’exploitation n’a pas pu être chargé, car le fichier du Registre système est manquant ou contient des erreurs. » L’image indique également le code d’erreur « 0xC0000225 » et le fichier `\windows\System32\config\system`.

![L’écran de récupération indique « Votre ordinateur/périphérique doit être réparé. Le système d’exploitation n’a pas pu être chargé, car le fichier du Registre système est manquant ou contient des erreurs. » Il affiche également le code d’erreur « 0xC0000225 » et le fichier « \windows\System32\config\system ».](./media/troubleshoot-boot-error-status-not-found/4.png)

## <a name="causes"></a>Causes

### <a name="missing-binary"></a>Binaire manquant

Il se peut que vous rencontriez un binaire manquant ou endommagé dans votre fichier système **(.sys)** .

### <a name="bcd-corruption-or-improper-vhd-migration"></a>Endommagement de BCD ou migration de disque dur virtuel incorrecte

Dans ce cas, soit **Données de configuration de démarrage (BCD)** est endommagé, soit le **disque dur virtuel (VHD)** a été migré à partir d’un emplacement local, mais n’a pas été correctement préparé. Le résultat est que la variable **OSDEVICE** est manquante et doit être ajoutée.

### <a name="registry-hive-corruption"></a>Endommagement de la ruche du Registre

Un endommagement de la ruche du Registre peut être due aux causes suivantes :

- La ruche échoue
- Le ruche est montée, mais elle est vide
- La ruche n’a pas été fermée correctement
## <a name="solution"></a>Solution

### <a name="process-overview"></a>Vue d’ensemble du processus

1. Créer une machine virtuelle de réparation et y accéder.
1. Sélectionner une solution :
   - [Réparer le fichier système](#repair-the-system-file)
   - [Ajouter la variable OSDevice](#add-the-osdevice-variable)
   - [Créer un ticket de support](#contact-support)
1. Activer la console série et la collecte de l’image mémoire.
1. Regénérez la machine virtuelle.

### <a name="create-and-access-a-repair-vm"></a>Créer une machine virtuelle de réparation et y accéder

1. Effectuez les étapes 1 à 3 des [commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une machine virtuelle de réparation.
1. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="select-a-solution"></a>Sélectionner une solution

1. Ouvrez une invite de commandes avec élévation de privilèges.
1. En fonction du symptôme que vous avez identifié précédemment, suivez les étapes de la solution correspondante. Vous pouvez ignorer les étapes des autres solutions, car elles ne s’appliquent pas à votre problème :

- [Réparer le fichier système](#repair-the-system-file)
- [Ajouter la variable OSDevice](#add-the-osdevice-variable)
- [Créer un ticket de support](#contact-support)

### <a name="repair-the-system-file"></a>Réparer le fichier système

1. À l’aide du disque dur virtuel attaché, accédez à l’emplacement du binaire affiché dans la capture d’écran de votre machine virtuelle.
1. Cliquez avec le bouton droit sur le fichier, sélectionnez **Propriétés**, puis sélectionnez l’onglet **Détails** pour afficher des informations sur le fichier.
   1. Notez la version du fichier, comme indiqué dans l’image ci-dessous :

      ![La fenêtre Propriétés du fichier « cng.sys », avec la version de fichier mise en surbrillance.](./media/troubleshoot-boot-error-status-not-found/5.png)

1. Renommez le fichier en **< BINARY.SYS >.old**, en remplaçant **< BINARY.SYS >** par le nom du fichier.

   Pour l’image de l’étape ci-dessus, le fichier **cng.sys** sera renommé **cng.sys.old**

   > [!NOTE]
   > Si vous essayez de renommer le fichier et que vous recevez le message « Le fichier est endommagé et illisible. », [contactez le support technique pour obtenir de l’aide](https://azure.microsoft.com/support/create-ticket/), car cette solution ne fonctionnera pas.

1. Maintenant que le fichier endommagé est renommé, corrigez le fichier en le restaurant à partir de son référentiel interne.
   1. Lancez une session **CMD**.
   1. Accédez à **\windows\winsxs**.

   1. Recherchez le binaire situé au début de cette section à l’aide de la commande suivante :

      `dir <BINARY WITH ".SYS" EXTENSION>  /s`

      Cette commande répertorie toutes les versions du fichier que la machine possède, ce qui vous donne l’historique du chemin d’accès de ce composant.

      Par exemple, **dir cng.sys** sera renommé **dir cng.sys /s**.

   1. Choisissez la version la plus récente du fichier dans la liste (ou celle que vous préférez) et copiez le fichier dans le dossier **windows\system32** à l’aide du chemin d’accès précédent et de la commande suivante :

      `copy <drive>:\Windows\WinSxS\<DIRECTORY WHERE FILE IS>\<BINARY WITH ".SYS" EXTENSION> <DRIVE>:\Windows\System32\Drivers\`

      > [!NOTE]
      > Si le binaire le plus récent n’a pas fonctionné, essayez une version antérieure à celui-ci, ou toute autre dont vous savez qu’il existe un fichier stable, comme une version antérieure à un correctif.

      Par exemple, si le binaire que vous recherchez est **cmimcext.sys**, que le lecteur défaillant est le lecteur **F:** et que vous venez d’exécuter une recherche de la dernière version, vous pourrez voir l’image suivante, où une requête `dir cmim* /s` dans l’invite de commandes localise la dernière version du fichier cmimcext.sys.

      ![Une requête « dir cmim* /s » dans l’invite de commandes pour localiser la dernière version du fichier cmimcext.sys.](./media/troubleshoot-boot-error-status-not-found/6.png)

      Dans l’exemple d’image ci-dessus, la requête a été exécutée sur **C:** , alors que la lettre de lecteur doit être celle du lecteur défaillant, **F:** , qui est le disque de système d’exploitation attaché en tant que disque de données sur la machine virtuelle de réparation.

      La commande qui en résulte pour copier le fichier serait la suivante : `copy F:\Windows\WinSxS\amd64_xxxxxx\cmimcext.sys F:\Windows\System32\Drivers`.

Une fois cette tâche terminée, continuez vers [Activer la console série et la collecte d’image mémoire](#enable-the-serial-console-and-memory-dump-collection).

### <a name="add-the-osdevice-variable"></a>Ajouter la variable OSDEVICE

Collectez les informations relatives à la configuration de démarrage actuelle et notez l’identificateur sur la partition active. Vous utiliserez ensuite ces informations pour ajouter la variable **OSDEVICE**, en suivant les instructions relatives à la génération de votre machine virtuelle.

Si cette collection d’informations génère une erreur alors qu’il n’y a pas de fichier **\boot\bcd**, suivez les instructions dans [Réparer le fichier système](#repair-the-system-file). 

1. Pour les machines virtuelles de 1e génération, ouvrez une invite de commandes avec élévation de privilèges en tant qu’administrateur, puis entrez la commande suivante :

   `bcdedit /store <LETTER OF BOOT FOLDER>:\boot\bcd /enum`

   Cette image montre le chargeur de démarrage Windows dans une machine virtuelle de 1e génération, avec l’attribut d’identificateur mis en surbrillance. L’attribut d’identificateur mis en surbrillance montre une chaîne alphanumérique unique.

   ![Chargeur de démarrage Windows affiché dans une machine virtuelle de 1e génération avec l’attribut d’identificateur mis en surbrillance. L’attribut d’identificateur mis en surbrillance montre une chaîne alphanumérique unique.](./media/troubleshoot-boot-error-status-not-found/7.png)

   Notez l’identificateur du chargeur de démarrage Windows, dont le chemin d’accès est **\windows\system32\winload.exe**.

1. Pour les machines virtuelles de 2e génération, vérifiez que le disque du système d’exploitation est en ligne et que les lettres de lecteur des partitions ont été attribuées. Une fois cette vérification effectuée, collectez les informations relatives à la configuration de démarrage.
   1. Dans **Windows Search**, saisissez **Gestion des disques**, puis ouvrez la console de gestion des disques. Utilisez cette console pour identifier le numéro de disque attaché à votre machine virtuelle de réparation et la partition Extensible Firmware Interface (EFI) qui contient le magasin BCD.

   Dans l’image suivante, Disk 2 est le numéro de disque attaché à la machine virtuelle de réparation. L’image affiche également la partition système EFI sur Disk 2, qui a une taille de 100 Mo et n’a pas de lettre attribuée.

   ![Disk 2 s’affiche comme numéro de disque attaché à la machine virtuelle de réparation. L’image affiche également la partition système EFI sur Disk 2, qui a une taille de 100 Mo et n’a pas de lettre attribuée.](./media/troubleshoot-boot-error-status-not-found/8.png)

   1. Ouvrez une invite de commandes avec élévation de privilèges en tant qu’administrateur, puis entrez la commande suivante :
      1. Ouvrez l’**outil DISKPART** à l’aide de la commande `diskpart`.
      1. Répertoriez tous les disques, puis sélectionnez le disque attaché identifié à l’étape précédente :
      
         ```
         list disk
         sel disk <DISK #>
         ```

         L’illustration suivante montre les résultats de l’énumération et de la sélection d’un disque. Disk 0 (127 Go / En ligne), Disk 1 (32 Go / En ligne) et Disk 2 (127 Go / En ligne) sont répertoriés, avec Disk 2 sélectionné à l’aide de la commande `sel disk 2`.

         ![Résultats de l’énumération et de la sélection d’un disque. Disk 0 (127 Go | En ligne), Disk 1 (32 Go | En ligne) et Disk 2 (127 Go | En ligne) sont répertoriés, avec Disk 2 sélectionné.](./media/troubleshoot-boot-error-status-not-found/9.png)

      1. Répertoriez les partitions et sélectionnez la partition système EFI identifiée à l’étape précédente :
      
         ```
         list partition
         sel partition <PARTITION #>
         ```

         L’illustration suivante montre les résultats de l’énumération et de la sélection d’une partition. Partition 1 (Réservée / 16 Mo), Partition 2 (Système / 100 Mo) et Partition 3 (Principale / 126 Go) sont répertoriées, Partition 2 étant sélectionnée à l’aide de la commande `sel part 2`.

         ![Résultats de l’énumération et de la sélection d’une partition. Partition 1 (Réservée | 16 Mo), Partition 2 (Système | 100 Mo) et Partition 3 (Principale | 126 Go) sont répertoriées, Partition 2 étant sélectionnée.](./media/troubleshoot-boot-error-status-not-found/10.png)

      1. Attribuez une lettre à la partition EFI à l’aide de la commande `assign`.

         Dans l’image suivante, la commande `assign` et le nouveau lecteur **SYSTEM (F:)** sont visibles dans Explorateur de fichiers.

         ![La commande assign et le nouveau lecteur SYSTEM (F:) sont visibles dans Explorateur de fichiers.](./media/troubleshoot-boot-error-status-not-found/11.png)

      1. Répertoriez les données du magasin BCD à l’aide de la commande suivante :
      
         `bcdedit /store <LETTER OF EFI SYSTEM PARTITION>:EFI\Microsoft\boot\bcd /enum`

         Dans l’image suivante, le chargeur de démarrage Windows se trouve dans une machine virtuelle de 2e génération, avec l’attribut d’identificateur mis en surbrillance. L’attribut d’identificateur mis en surbrillance a la valeur **{default}** .

         ![Chargeur de démarrage Windows affiché dans une machine virtuelle de 2e génération avec l’attribut d’identificateur mis en surbrillance. L’attribut d’identificateur mis en surbrillance affiche la valeur « par défaut ».](./media/troubleshoot-boot-error-status-not-found/12.png)

         Notez l’identificateur du chargeur de démarrage Windows, dont le chemin d’accès est **\windows\system32\winload.efi**.

1. Notez que la variable OSDEVICE sur la partition active est manquante :

   ![Les attributs du gestionnaire de démarrage Windows et du chargeur de démarrage Windows sont répertoriés dans l’invite de commandes, avec l’attribut OSDEVICE manquant.](./media/troubleshoot-boot-error-status-not-found/13.png)
   
   Dans cette image, les attributs du gestionnaire de démarrage Windows et du chargeur de démarrage Windows sont répertoriés dans l’invite de commandes, mais l’attribut OSDEVICE est manquant.

1. Ajoutez la variable OSDEVICE en fonction des informations suivantes :

   Pour les disques de système d’exploitation à partition unique, ajoutez `BOOT`.

   > [!NOTE]
   > Le dossier de démarrage peut se trouver sur la même partition que le dossier Windows **\windows folder**.
   > - Le dossier de démarrage pour les machines virtuelles de 1e génération est **(\boot\bcd folder)** .
   > - Le dossier de démarrage pour les machines virtuelles de 2e génération est **EFI\Microsoft\boot\bcd**.

   Pour les machines virtuelles de 1e génération, entrez la commande suivante :

   `bcdedit /store <LETTER OF BOOT FOLDER>:\boot\bcd /set {<IDENTIFIER>} OSDEVICE BOOT`

   Pour les machines virtuelles de 2e génération, entrez la commande suivante :

   `bcdedit /store <LETTER OF EFI FOLDER>:EFI\Microsoft\boot\bcd /set {<IDENTIFIER>} OSDEVICE BOOT`

   Pour les disques de système d’exploitation à partitions multiples, ajoutez `PARTITION=<LETTER OF WINDOWS FOLDER>:`.

   > [!NOTE]
   > Le dossier de démarrage est probablement sur une partition différente de celle du dossier Windows **\windows folder**.
   > - Le dossier de démarrage pour les machines virtuelles de 1e génération est **(\boot\bcd folder)** .
   > - Le dossier de démarrage pour les machines virtuelles de 2e génération est **EFI\Microsoft\boot\bcd**.

   Pour les machines virtuelles de 1e génération, entrez la commande suivante :

   `bcdedit /store <LETTER OF BOOT FOLDER>:\boot\bcd /set {<IDENTIFIER>} OSDEVICE partition=<LETTER OF WINDOWS FOLDER>:`

   Pour les machines virtuelles de 2e génération, entrez la commande suivante :

   `bcdedit /store <LETTER OF EFI FOLDER>:EFI\Microsoft\boot\bcd /set {< IDENTIFIER>} OSDEVICE partition=<LETTER OF WINDOWS FOLDER>:`

Une fois cette tâche terminée, continuez vers [Activer la console série et la collecte d’image mémoire](#enable-the-serial-console-and-memory-dump-collection).

### <a name="contact-support"></a>Contacter le support technique

L’erreur **Fichier du Registre** a une solution, mais vous devrez [créer un de ticket de support](https://azure.microsoft.com/support/create-ticket/) pour obtenir plus d’aide.

### <a name="enable-the-serial-console-and-memory-dump-collection"></a>Activer la console série et la collecte de l’image mémoire

**Recommandé** : Avant de régénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire en exécutant le script suivant :

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
