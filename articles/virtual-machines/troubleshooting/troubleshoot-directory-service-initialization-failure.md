---
title: 'Résoudre les erreurs d’arrêt de Windows : échec d’initialisation du service d’annuaire'
description: Réglez les problèmes liés à une machine virtuelle du contrôleur de domaine Active Directory dans Azure qui est bloquée dans une boucle indiquant qu’elle doit redémarrer.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 3396f8fe-7573-4a15-a95d-a1e104c6b76d
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 05/05/2020
ms.author: v-miegge
ms.openlocfilehash: 909481964f8aa3272715e235fa011562225a9422
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87028360"
---
# <a name="troubleshoot-windows-stop-error--directory-service-initialization-failure"></a>Résoudre les erreurs d’arrêt de Windows : échec d’initialisation du service d’annuaire

Cet article décrit les étapes à suivre pour résoudre les problèmes liés à une machine virtuelle du contrôleur de domaine Active Directory dans Azure qui est bloquée dans une boucle et indique qu’elle doit redémarrer.

## <a name="symptom"></a>Symptôme

Lorsque vous utilisez l’outil [Diagnostics de démarrage](./boot-diagnostics.md) pour afficher la capture d’écran de la machine virtuelle, la capture d’écran indique que la machine virtuelle doit être redémarrée en raison d’une erreur, affichant le code d’arrêt **0xC00002E1** dans Windows Server 2008 R2 ou **0xC00002E2** dans Windows Server 2012 ou version ultérieure.

![L’écran de démarrage de Windows Server 2012 indique « Votre ordinateur a rencontré un problème et doit redémarrer. Nous collectons simplement quelques informations sur l’erreur, puis nous vous aiderons à redémarrer. »](./media/troubleshoot-directory-service-initialization-failure/1.png)

## <a name="cause"></a>Cause

Le code d’erreur **0xC00002E2** représente **STATUS_DS_INIT_FAILURE** et le code d’erreur **0xC00002E1** représente **STATUS_DS_CANT_START**. Les deux erreurs se produisent en cas de problème avec le service d’annuaire.

Lorsque le système d’exploitation démarre, il est ensuite forcé de redémarrer automatiquement par le serveur local d’authentification de sécurité (**LSASS.exe**), qui authentifie les connexions des utilisateurs. L’authentification ne peut pas se produire lorsque le système d’exploitation sur la machine virtuelle est un contrôleur de domaine qui ne dispose pas d’un accès en lecture/écriture à sa base de données Active Directory locale. En raison d’un manque d’accès à **Active Directory (AD)** , LSASS.exe ne peut pas s’authentifier et il est contraint de redémarrer le système d’exploitation.

Cette erreur peut être causée par l’une des conditions suivantes :

- Il n’y a aucun accès au disque contenant la base de données AD locale (**NTDS.DIT**).
- Le disque contenant la base de données AD locale (NTDS.DIT) n’a plus d’espace libre.
- Le fichier de la base de données AD locale (NTDS.DIT) est manquant.
- La machine virtuelle possède plusieurs disques et la stratégie de réseau de zone de stockage (SAN) est mal configurée. La stratégie SAN n’est pas définie sur **ONLINEALL** et les disques non-OS sont attachés en mode hors connexion sur le gestionnaire de disques.
- Le fichier de la base de données AD locale (NTDS.DIT) est endommagé.

## <a name="solution"></a>Solution

### <a name="process-overview"></a>Vue d’ensemble du processus :

1. Créez une machine virtuelle de réparation et accédez-y.
1. Libérez de l’espace sur le disque.
1. Vérifiez que le lecteur contenant la base de données AD est attaché.
1. Activez le mode de restauration des services d’annuaire.
1. **Recommandé** : Avant de regénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire.
1. Regénérez la machine virtuelle.
1. Reconfigurez la stratégie SAN.

> [!NOTE]
> Lorsque vous rencontrez cette erreur, le système d’exploitation invité n’est pas opérationnel. Pour résoudre ce problème, vous allez opérer en mode hors connexion.

### <a name="create-and-access-a-repair-vm"></a>Créer une machine virtuelle de réparation et y accéder

1. Suivez les [étapes 1 à 3 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour préparer une machine virtuelle de réparation.
1. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="free-up-space-on-disk"></a>Libérer de l’espace sur le disque

Étant donné que le disque est maintenant attaché à une machine virtuelle de réparation, vérifiez que le disque contenant la base de données interne Active Directory dispose de suffisamment d’espace pour fonctionner correctement.

1. Vérifiez si le disque est plein en cliquant avec le bouton droit sur le lecteur et en sélectionnant **Propriétés**.
1. Si le disque dispose de moins de 300 Mo d’espace libre, [augmentez-le jusqu’à un maximum de 1 To en utilisant PowerShell](../windows/expand-os-disk.md).
1. Si le disque a atteint 1 To d’espace utilisé, nettoyez le disque.

   1. Utilisez PowerShell pour [détacher le disque de données](../windows/detach-disk.md#detach-a-data-disk-using-powershell) de la machine virtuelle défaillante.
   1. Une fois qu’il est détaché de la machine virtuelle défaillante, [attachez le disque de données](../windows/attach-disk-ps.md#attach-an-existing-data-disk-to-a-vm) à une machine virtuelle opérationnelle.
   1. Utilisez l’[outil Nettoyage de disque](https://support.microsoft.com/help/4026616/windows-10-disk-cleanup) pour libérer de l’espace supplémentaire.

1. **Facultatif** : si plus d’espace est nécessaire, ouvrez une instance CMD et entrez la commande `defrag <LETTER ASSIGNED TO THE OS DISK>: /u /x /g` pour défragmenter le lecteur :

  * Dans la commande, remplacez `<LETTER ASSIGNED TO THE OS DISK>` par la lettre du disque du système d’exploitation. Par exemple, si la lettre de disque est `F:`, la commande serait `defrag F: /u /x /g`.

  * En fonction du niveau de fragmentation, la défragmentation peut prendre plusieurs heures.

S’il y a suffisamment d’espace sur le disque, passez à la tâche suivante.

### <a name="check-that-the-drive-containing-the-active-directory-database-is-attached"></a>Vérifiez que le lecteur contenant la base de données Active Directory est attaché.

1. Ouvrez une instance CMD avec élévation de privilèges, puis exécutez les commandes suivantes :

   1. Chargez le fichier de Registre :

      `REG LOAD HKLM\BROKENSYSTEM f:\windows\system32\config\SYSTEM`

      La désignation `f:` suppose que le disque est le lecteur `F:`. Utilisez la lettre de lecteur appartenant au lecteur contenant le disque du système d’exploitation.

   1. Déterminez la lettre de lecteur et le dossier de **NTDS.DIT** :

      ```
      REG QUERY "HKLM\BROKENSYSTEM\ControlSet001\Services\NTDS\parameters" /v "DSA Working Directory"
      REG QUERY "HKLM\BROKENSYSTEM\ControlSet001\Services\NTDS\parameters" /v "DSA Database file"
      REG QUERY "HKLM\BROKENSYSTEM\ControlSet001\Services\NTDS\parameters" /v "Database backup path"
      REG QUERY "HKLM\BROKENSYSTEM\ControlSet001\Services\NTDS\parameters" /v "Database log files path"
      ```

   1. Déchargez le fichier de Registre :

      `REG UNLOAD HKLM\BROKENSYSTEM`

1. À l’aide de Portail Azure, vérifiez que le lecteur où est configuré NTDS.DIT est ajouté à la machine virtuelle.
1. À l’aide de la console Gestion des disques du système d’exploitation invité, vérifiez que le disque contenant NTDS.DIT est en ligne.
   1. L’outil Gestion des disques se trouve dans **Outils d’administration > Gestion de l’ordinateur > Stockage** ou est accessible à l’aide de la commande `diskmgmt.msc` dans une instance CMD.
1. Si le disque n’est pas attaché à la machine virtuelle, rattachez le disque de données pour corriger le problème.

   Si le disque a été attaché normalement, passez à la tâche suivante.

### <a name="enable-directory-services-restore-mode"></a>Activer le mode de restauration des services d’annuaire

Configurez la machine virtuelle pour qu’elle démarre en **mode Restauration des services d’annuaire (DSRM)** pour ne pas vérifier l’existence du fichier NTDS.DIT au démarrage.

1. Avant de continuer, vérifiez que vous avez effectué les tâches précédentes pour attacher le disque à une machine virtuelle de réparation et que vous avez déterminé sur quel disque se trouve le fichier NTDS.DIT.
1. À l’aide d’une instance CMD avec élévation de privilèges, répertoriez les informations de la partition de démarrage sur ce magasin pour trouver l’identificateur de la partition active :

   `bcdedit /store <Drive Letter>:\boot\bcd /enum`

   Remplacez `< Drive Letter >` par la lettre déterminée dans les étapes précédentes.

   ![La capture d’écran montre une instance CMD avec élévation de privilèges après la saisie de la commande « bcdedit /store <Lettre de disque>:\boot\bcd /enum », qui affiche le Gestionnaire de démarrage Windows avec l’identificateur.](./media/troubleshoot-directory-service-initialization-failure/2.png)

1. Activez l’indicateur `safeboot DsRepair` sur la partition de démarrage :

   `bcdedit /store <Drive Letter>:\boot\bcd /set {<Identifier>} safeboot dsrepair`

   Remplacez `< Drive Letter >` et `< Identifier >` par les valeurs déterminées dans les étapes précédentes.

1. Interrogez à nouveau les options de démarrage pour vous assurer que votre modification a été correctement paramétrée.

   ![La capture d’écran montre une instance CMD avec élévation de privilèges après l’activation de l’indicateur « safeboot DsRepair ».](./media/troubleshoot-directory-service-initialization-failure/3.png)

### <a name="recommended-before-you-rebuild-the-vm-enable-serial-console-and-memory-dump-collection"></a>Recommandé : avant de regénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire.

Pour activer la collecte de l’image mémoire et la console série, exécutez le script suivant en ouvrant une session d’invite de commandes avec élévation de privilèges (Exécuter en tant qu’administrateur), puis exécutez les commandes suivantes.

1. Activez la console série :

  ```
  bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON
  bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200
  ```

1. Vérifiez que l’espace libre sur le disque du système d’exploitation est au moins égal à la taille de la mémoire (RAM) sur la machine virtuelle.

  1. Si l’espace n’est pas suffisant, nous vous recommandons de changer l’emplacement où le fichier de l’image mémoire sera créé et de le référencer sur n’importe quel autre disque de données attaché à la machine virtuelle possédant suffisamment d’espace libre.

     Pour changer l’emplacement, remplacez `%SystemRoot%` par la lettre de lecteur (par exemple, `F:`) du disque de données dans les commandes suivantes.

  #### <a name="the-following-configuration-is-suggested-to-enable-os-dump"></a>Nous vous suggérons la configuration suivante pour activer l’image mémoire du système d’exploitation :

  **Charger le disque de système d’exploitation corrompu** :

  `REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM`

  **Activer sur ControlSet001** :

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

  `REG UNLOAD HKLM\BROKENSYSTEM`

### <a name="rebuild-the-vm"></a>Régénérez la machine virtuelle.

1. Suivez l’[étape 5 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour réassembler la machine virtuelle.

### <a name="reconfigure-the-storage-area-network-policy"></a>Reconfigurer la stratégie de réseau de zone de stockage

1. Lors du démarrage en mode DSRM, le seul utilisateur disponible pour se connecter est l’administrateur de récupération, qui a été utilisé lorsque la machine virtuelle a été promue contrôleur de domaine. Tous les autres utilisateurs afficheront une erreur d’authentification.

   1. Si aucun autre contrôleur de domaine n’est disponible, vous devez vous connecter localement à l’aide de l’identifiant `.\administrator` ou `machinename\administrator` et du mot de passe DSRM.

1. Configurez la stratégie SAN afin que tous les disques soient en ligne.

   1. Ouvrez une instance CMD avec élévation de privilèges et entrez `DISKPART`.
   1. Demandez la liste des disques.

      `DISKPART> list disk`

   1. Entrez les commandes suivantes pour sélectionner le disque qui doit être mis en ligne et modifier la stratégie SAN :

      ```
      DISKPART> select disk 1
      Disk 1 is now the selected disk.

      DISKPART> attributes disk clear readonly
      Disk attributes cleared successfully.

      DISKPART> attributes disk
      Current Read-only State : No
      Read-only : No
      Boot Disk : No
      Pagefile Disk : No
      Hibernation File Disk : No
      Crashdump Disk : No
      Clustered Disk : No

      DISKPART> online disk
      DiskPart successfully onlined the selected disk.

      DISKPART> san
      SAN Policy : Online All
      ```

1. Une fois le problème corrigé, vérifiez que l’indicateur `DsRepair safeboot` est supprimé :

   `bcdedit /deletevalue {default} safeboot dsrepair`

1. Redémarrez votre machine virtuelle.

   > [!NOTE]
   > Si votre machine virtuelle vient d’être migrée depuis un emplacement local et que vous souhaitez migrer d’autres contrôleurs de domaine d’un emplacement local vers Azure, vous devez suivre les étapes décrites dans l’article ci-dessous pour éviter que ce problème se produise lors des futures migrations :
   >
   > [Comment télécharger des contrôleurs de domaine Hyper-V locaux existants vers Azure à l’aide de PowerShell Azure](https://support.microsoft.com/help/2904015)
