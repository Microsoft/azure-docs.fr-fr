---
title: Installer l’agent de machine virtuelle Azure en mode hors connexion | Microsoft Docs
description: Découvrez comment installer l’agent de machine virtuelle Azure en mode hors connexion.
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: article
ms.date: 07/06/2020
ms.author: genli
ms.openlocfilehash: 456aa225fa8eed47ca794c54e61b77a30c93fa9a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002610"
---
# <a name="install-the-azure-virtual-machine-agent-in-offline-mode"></a>Installer l’agent de machine virtuelle Azure en mode hors connexion 

L’agent de machine virtuelle Azure fournit des fonctionnalités utiles, telles que la réinitialisation de mot de passe d’administrateur local et l’exécution d’un push de script. Cet article explique comment installer l’agent de machine virtuelle pour une machine virtuelle Windows hors connexion. 

## <a name="when-to-use-the-vm-agent-in-offline-mode"></a>À quel moment utiliser l’agent de machine virtuelle en mode hors connexion

Installez l’agent de machine virtuelle en mode hors connexion dans les scénarios suivants :

- La machine virtuelle Azure déployée n’a pas l’agent de machine virtuelle installé ou l’agent ne fonctionne pas.
- Vous avez oublié le mot de passe administrateur pour la machine virtuelle, ou vous ne pouvez pas accéder à la machine virtuelle.

## <a name="how-to-install-the-vm-agent-in-offline-mode"></a>Comment installer l’agent de machine virtuelle en mode hors connexion

Procédez comme suit pour installer l’agent de machine virtuelle en mode hors connexion.

### <a name="step-1-attach-the-os-disk-of-the-vm-to-another-vm-as-a-data-disk"></a>Étape 1 : attacher le disque du système d’exploitation de la machine virtuelle à une autre machine virtuelle en tant que disque de données

1. Effectuez une capture instantanée du disque du système d’exploitation de la machine virtuelle affectée, créez un disque à partir de cette capture instantanée, puis attachez le disque à une machine virtuelle de dépannage. Pour en savoir plus, consultez [Résoudre les problèmes d’une machine virtuelle Windows en connectant le disque du système d’exploitation à une machine virtuelle de récupération à l’aide du portail Azure](troubleshoot-recovery-disks-portal-windows.md). Pour la machine virtuelle classique, supprimez la machine virtuelle et conservez le disque du système d’exploitation, puis attachez le disque du système d’exploitation à la machine virtuelle de dépannage.

2.  Connectez-vous à la machine virtuelle de dépannage. Ouvrez **Gestion de l’ordinateur** > **Gestion des disques**. Vérifiez que le disque du système d’exploitation est en ligne et que les lettres de lecteur sont attribuées aux partitions de disque.

### <a name="step-2-modify-the-os-disk-to-install-the-azure-vm-agent"></a>Étape 2 : modifier le disque du système d’exploitation pour installer l’agent de machine virtuelle Azure

1.  Effectuez une connexion Bureau à distance à la machine virtuelle de dépannage.

2.  Sur la machine virtuelle de l’utilitaire de résolution des problèmes, parcourez le disque du système d’exploitation que vous avez attaché, puis ouvrez le dossier \windows\system32\config. Copiez tous les fichiers dans ce dossier de sauvegarde, au cas où une restauration serait requise.

3.  Démarrez **l’Éditeur du Registre** (regedit.exe).

4.  Sélectionnez la clé **HKEY_LOCAL_MACHINE**. Dans le menu, sélectionnez **Fichier** > **Charger la ruche** :

    ![Charger la ruche](./media/install-vm-agent-offline/load-hive.png)

5.  Sur le disque du système d’exploitation que vous avez attaché, accédez au dossier \windows\system32\config\SYSTEM. Pour le nom de la ruche, entrez **BROKENSYSTEM**. La nouvelle ruche du Registre est affichée sous la clé **HKEY_LOCAL_MACHINE**.

6.  Sur le disque du système d’exploitation que vous avez attaché, accédez au dossier \windows\system32\config\SOFTWARE. Pour le nom du logiciel de la ruche, entrez **BROKENSOFTWARE**.

7. Si l’agent de machine virtuelle est installé sur le disque du système d’exploitation attaché, effectuez une sauvegarde de la configuration actuelle. Si l’agent de machine virtuelle n’est pas installé, passez à l’étape suivante.
      
    1. Renommez le dossier \windowsazure en \windowsazure.old.

    2. Exportez les registres suivants :
        - HKEY_LOCAL_MACHINE\BROKENSYSTEM\ControlSet001\Services\WindowsAzureGuestAgent
        - HKEY_LOCAL_MACHINE\BROKENSYSTEM\ControlSet001\Services\RdAgent

8.  Utilisez les fichiers existants de la machine virtuelle de dépannage comme référentiel pour l’installation de l’agent de machine virtuelle. Suivez les étapes ci-dessous :

    1. À partir de la machine virtuelle de dépannage, exportez les sous-clés suivantes au format du Registre (.reg) : 
        - HKEY_LOCAL_MACHINE  \SYSTEM\ControlSet001\Services\WindowsAzureGuestAgent
        - HKEY_LOCAL_MACHINE  \SYSTEM\ControlSet001\Services\RdAgent

          ![Exporter les sous-clés du Registre](./media/install-vm-agent-offline/backup-reg.png)

    2. Modifiez les fichiers du Registre. Dans chaque fichier, changez la valeur d’entrée **SYSTEM** en **BROKENSYSTEM** (comme indiqué dans les images suivantes) et enregistrez le fichier. Mémorisez la valeur **ImagePath** de l’agent de machine virtuelle actif. Nous devrons copier le dossier correspondant sur le disque du système d’exploitation attaché. 

        ![Modifier les valeurs de sous-clés du Registre](./media/install-vm-agent-offline/change-reg.png)

    3. Importez les fichiers du Registre dans le référentiel en double-cliquant sur chaque fichier du Registre.

    4. Vérifiez que les deux sous-clés suivantes sont bien importées dans la ruche **BROKENSYSTEM** :
        - WindowsAzureGuestAgent
        - RdAgent

    5. Copiez le dossier d’installation de l’agent de machine virtuelle actif vers le disque du système d’exploitation attaché : 

        1.  Sur le disque du système d’exploitation que vous avez attaché, créez un dossier nommé WindowsAzure dans le chemin racine.

        2.  Accédez à C:\WindowsAzure sur la machine virtuelle de l’utilitaire de résolution des problèmes et recherchez un dossier portant le nom C:\WindowsAzure\GuestAgent_X.X.XXXX.XXX. Copiez le dossier GuestAgent qui a le dernier numéro de version à partir de C:\WindowsAzure dans le dossier WindowsAzure sur le disque du système d’exploitation attaché. Si vous ne savez pas quel dossier doit être copié, copiez tous les dossiers GuestAgent. L’image suivante montre un exemple du dossier GuestAgent qui est copié sur le disque du système d’exploitation attaché.

             ![Copier le dossier GuestAgent](./media/install-vm-agent-offline/copy-files.png)

9.  Sélectionnez **BROKENSYSTEM**. Dans le menu, sélectionnez **Fichier** > **Décharger la ruche**.

10.  Sélectionnez **BROKENSOFTWARE**. Dans le menu, sélectionnez **Fichier** > **Décharger la ruche**.

11.  Détachez le disque du système d’exploitation, puis [choisissez un autre de disque du système d’exploitation pour la machine virtuelle affectée](troubleshoot-recovery-disks-portal-windows.md#swap-the-os-disk-for-the-vm). Pour la machine virtuelle classique, créez une machine virtuelle à l’aide du disque du système d’exploitation réparé.

12.  Accédez à la machine virtuelle. Notez que le RdAgent est en cours d’exécution et que les journaux d’activité sont générés.

Si vous avez créé la machine virtuelle à l’aide du modèle de déploiement Resource Manager, vous avez terminé.

### <a name="use-the-provisionguestagent-property-for-classic-vms"></a>Utiliser la propriété ProvisionGuestAgent pour les machines virtuelles classiques

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Si vous avez créé la machine virtuelle à l’aide du modèle classique, utilisez le module Azure PowerShell pour mettre à jour la propriété **ProvisionGuestAgent**. La propriété informe Azure que la machine virtuelle a l’agent de machine virtuelle installé.

Pour définir la propriété **ProvisionGuestAgent**, exécutez les commandes suivantes dans Azure PowerShell :

   ```powershell
   $vm = Get-AzureVM –ServiceName <cloud service name> –Name <VM name>
   $vm.VM.ProvisionGuestAgent = $true
   Update-AzureVM –Name <VM name> –VM $vm.VM –ServiceName <cloud service name>
   ```

Exécutez ensuite la commande `Get-AzureVM` . Notez que la propriété **GuestAgentStatus** est maintenant remplie avec des données :

   ```powershell
   Get-AzureVM –ServiceName <cloud service name> –Name <VM name>
   GuestAgentStatus:Microsoft.WindowsAzure.Commands.ServiceManagement.Model.PersistentVMModel.GuestAgentStatus
   ```

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble d’agent de machine virtuelle Azure](../extensions/agent-windows.md)
- [Extensions et fonctionnalités de machine virtuelle pour Windows](../extensions/features-windows.md)
