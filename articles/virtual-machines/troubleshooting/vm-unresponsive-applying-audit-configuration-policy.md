---
title: La machine virtuelle ne répond pas lors de l’application de la stratégie Configuration de la stratégie d’audit
description: Cet article décrit les étapes à suivre pour résoudre les problèmes d’absence de réponse de la machine virtuelle lors de l’application de la stratégie Configuration de la stratégie d’audit, ce qui empêche le démarrage d’une machine virtuelle Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 3f6383b5-81fa-49ea-9434-2fe475e4cbef
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 08/24/2020
ms.author: v-miegge
ms.openlocfilehash: ff21975c34c28d7476635467e0c1abb8e6575e35
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91977950"
---
# <a name="virtual-machine-is-unresponsive-while-applying-audit-policy-configuration-policy"></a>La machine virtuelle ne répond pas lors de l’application de la stratégie Configuration de la stratégie d’audit

Cet article décrit les étapes à suivre pour résoudre les problèmes d’absence de réponse de la machine virtuelle lors de l’application de la stratégie Configuration de la stratégie d’audit, ce qui empêche le démarrage d’une machine virtuelle Azure.

## <a name="symptom"></a>Symptôme

Lorsque vous utilisez [Diagnostics de démarrage](./boot-diagnostics.md) pour afficher la capture d’écran de la machine virtuelle, la capture d’écran indique que le système d’exploitation restait sans réponse au cours d’un démarrage avec le message **Application de la stratégie Configuration de la stratégie d’audit**.

  ![Démarrage du système d’exploitation avec le message : « Application de la stratégie Configuration de la stratégie d’audit »](./media/vm-unresponsive-applying-audit-configuration-policy/1.png)

  ![Démarrage du système d’exploitation dans Windows Server 2012 avec le message : « Application de la stratégie Configuration de la stratégie d’audit »](./media/vm-unresponsive-applying-audit-configuration-policy/2.png)

## <a name="cause"></a>Cause

Des verrous sont en conflit lorsque la stratégie tente de nettoyer les anciens profils utilisateur.

> [!NOTE]
> Cela concerne uniquement Windows Server 2012 et Windows Server 2012 R2.

Voici la stratégie problématique : *Configuration ordinateur\Stratégies\Modèles d’administration\Système/Profils utilisateur\Supprimer les profils utilisateur datant de plus d’un certain nombre de jours lors du redémarrage du système.*

## <a name="solution"></a>Solution

### <a name="process-overview"></a>Vue d’ensemble du processus

1. Créez une machine virtuelle de réparation et accédez-y.
1. Désactiver la tâche.
1. Activer la console série et la collecte de l’image mémoire.
1. Regénérez la machine virtuelle.
1. Recueillez le fichier d’image mémoire et envoyez un ticket de support.

### <a name="create-and-access-a-repair-vm"></a>Créer une machine virtuelle de réparation et y accéder

1. Effectuez les étapes 1 à 3 des [commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une machine virtuelle de réparation.
1. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="disable-the-policy"></a>Désactiver la stratégie

1. Sur la machine virtuelle de réparation, ouvrez **Éditeur du Registre**.
1. Localisez la clé **HKEY_LOCAL_MACHINE**, puis sélectionnez **Fichier > Charger Hive** dans le menu.

   ![Navigation dans l’Éditeur du Registre pour charger une ruche.](./media/vm-unresponsive-applying-audit-configuration-policy/3.png)

   - Vous pouvez utiliser « Charger Hive » pour charger des clés de registre à partir d’un système hors connexion. Dans ce cas, le système est le disque défectueux attaché à la machine virtuelle de réparation.
   - Les paramètres à l’échelle du système sont stockés sur **HKEY_LOCAL_MACHINE** et peuvent être abrégés sous la forme **HKLM**.

1. Sur le disque attaché, ouvrez le fichier `\windows\system32\config\SOFTWARE`.

   - Lorsque vous êtes invité à saisir un nom, saisissez **BROKENSOFTWARE**.
   - Pour vérifier que **BROKENSOFTWARE** a été chargé, développez **HKEY_LOCAL_MACHINE** et recherchez la clé **BROKENSOFTWARE** ajoutée.

1. Accédez à **BROKENSOFTWARE** et vérifiez si la clé **CleanupProfiles** existe dans la ruche chargée.

   - Si la clé existe, la stratégie **CleanupProfiles** est définie. Sa valeur représente la stratégie de rétention mesurée en jours.
   - Si la clé n’existe pas, la stratégie **CleanupProfiles** n’est pas définie. Dans ce cas, passez directement à [l’envoi d’un ticket de support avec un fichier d’image mémoire](#collect-the-memory-dump-file-and-submit-a-support-ticket).

1. Supprimez la clé **CleanupProfiles** à l’aide de cette commande :

   `reg delete "HKLM\BROKENSOFTWARE\Policies\Microsoft\Windows\System" /v CleanupProfiles /f`

1. Déchargez la ruche **BROKENSOFTWARE** à l’aide de cette commande :

   `reg unload HKLM\BROKENSOFTWARE`

### <a name="enable-the-serial-console-and-memory-dump-collection"></a>Activer la console série et la collecte de l’image mémoire

**Recommandé** : Avant de régénérer la machine virtuelle, activez la console série et la collecte de l’image mémoire en exécutant le script suivant :

1. Ouvrez une session d’invite de commandes avec élévation de privilèges en tant qu’administrateur.
1. Répertoriez les données du Store BCD et déterminez l’identificateur du chargeur de démarrage, que vous utiliserez à l’étape suivante.

   1. Pour une machine virtuelle de 1ère génération, entrez la commande suivante et notez l’identificateur indiqué :

      `bcdedit /store <BOOT PARTITON>:\boot\bcd /enum`

      - Dans la commande, remplacez `<BOOT PARTITON>` par la lettre de la partition du disque joint qui contient le dossier de démarrage.

        ![La Figure 4 affiche le résultat du listing du Store BCD dans une machine virtuelle de 1ère génération, qui répertorie sous le Chargeur de démarrage Windows le numéro d’identificateur.](./media/vm-unresponsive-applying-audit-configuration-policy/4.png)

   1. Pour une machine virtuelle de 2e génération, entrez la commande suivante et notez l’identificateur indiqué :

      `bcdedit /store <LETTER OF THE EFI SYSTEM PARTITION>:EFI\Microsoft\boot\bcd /enum`

      - Dans la commande, remplacez `<LETTER OF THE EFI SYSTEM PARTITION>` par la lettre de la partition système EFI.
      - Il peut être utile de lancer la console de gestion des disques pour identifier la partition système appropriée nommée **Partition système EFI**.
      - L’identificateur peut être un GUID unique ou la valeur par défaut **bootmgr**.

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
   
### <a name="rebuild-the-virtual-machine"></a>Régénérer la machine virtuelle

1. Utilisez [l’étape 5 des commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour régénérer la machine virtuelle.

1. Testez si votre machine virtuelle démarre normalement pour voir si le problème a été résolu.

   - Si le problème n’a pas été résolu, passez à l’étape de [collecte d’un fichier d’image mémoire et d’envoi d’un ticket de support](#collect-the-memory-dump-file-and-submit-a-support-ticket).
   - Si le problème a été résolu, aucune autre action n’est nécessaire.

Si le problème a été résolu, cela signifie que la stratégie est désormais désactivée localement. Pour une solution permanente, n’utilisez pas la stratégie CleanupProfiles sur les machines virtuelles, car elle supprime automatiquement les profils utilisateur. Utilisez une autre méthode pour effectuer des nettoyages de profil, par exemple une tâche ou un script planifié(e).

**N’utilisez pas cette stratégie :** 
*Machine\Modèles admin\Système\Profils utilisateur\Supprimer les profils utilisateur datant de plus d'un certain nombre de jours lors du redémarrage du système.*

### <a name="the-issue-should-now-be-fixed"></a>Le problème doit maintenant être résolu

Testez votre machine virtuelle pour vous assurer qu’elle fonctionne comme d’habitude. Si les problèmes persistent, vous pouvez passer à la section suivante pour obtenir de l’aide.

### <a name="collect-the-memory-dump-file-and-submit-a-support-ticket"></a>Recueillir le fichier d’image mémoire et envoyer un ticket de support

Pour résoudre ce problème, vous devez d’abord collecter le fichier de l’image mémoire du plantage, puis contacter le support muni de ce fichier d’image mémoire. Pour collecter le fichier d’image, effectuez les étapes suivantes :

#### <a name="attach-the-os-disk-to-a-new-repair-vm"></a>Attacher le disque du système d’exploitation à une machine virtuelle de récupération

1. Suivez les étapes 1 à 3 des [commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une nouvelle machine virtuelle de réparation.
1. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

#### <a name="locate-the-dump-file-and-submit-a-support-ticket"></a>Rechercher le fichier de l’image mémoire et envoyer un ticket de support

1. Sur la machine virtuelle de réparation, accédez au dossier Windows sur le disque du système d’exploitation attaché. Si la lettre de lecteur qui est affectée au disque de système d’exploitation attaché est *F*, alors vous devez accéder à `F:\Windows`.
1. Recherchez le fichier `memory.dmp`, puis [soumettez un ticket de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) avec le fichier de vidage de la mémoire.
1. Si vous rencontrez des difficultés pour localiser le fichier `memory.dmp`, utilisez à la place des [appels d’interruption non masquable (NMI) dans la console série](./serial-console-windows.md#use-the-serial-console-for-nmi-calls). Suivez le guide pour [générer un fichier d’image mémoire sur incident complet à l’aide d’appels NMI](/windows/client-management/generate-kernel-or-complete-crash-dump).