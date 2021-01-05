---
title: Réinitialiser un mot de passe Windows local sans agent Azure | Microsoft Docs
description: Réinitialisation du mot de passe d’un compte d’utilisateur Windows local lorsque l’agent invité Azure n’est pas installé ou ne fonctionne pas sur une machine virtuelle
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.assetid: cf353dd3-89c9-47f6-a449-f874f0957013
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 04/25/2019
ms.author: genli
ms.openlocfilehash: 4cec8f77cacc5d3492dd6a5f8a8baa060f910763
ms.sourcegitcommit: b4f303f59bb04e3bae0739761a0eb7e974745bb7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/02/2020
ms.locfileid: "91650594"
---
# <a name="reset-local-windows-password-for-azure-vm-offline"></a>Réinitialiser un mot de passe Windows local pour la machine virtuelle Azure hors connexion
Vous pouvez réinitialiser le mot de passe Windows local d’une machine virtuelle dans Azure à l’aide du [portail Azure ou Azure PowerShell](reset-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) à condition que l’agent invité Azure soit installé. Cette méthode est le principal moyen de réinitialiser un mot de passe sur une machine virtuelle Azure. Si l’agent invité Azure ne répond pas ou ne parvient pas à s’installer après chargement d’une image personnalisée, vous pouvez réinitialiser manuellement un mot de passe Windows. Cet article explique comment réinitialiser un mot de passe de compte local en attachant le disque virtuel du système d’exploitation source à une autre machine virtuelle. Les étapes décrites dans cet article ne s’appliquent pas aux contrôleurs de domaine Windows. 

> [!WARNING]
> N’utilisez ce processus qu’en dernier recours. Essayez toujours de réinitialiser un mot de passe à l’aide du [portail Azure ou Azure PowerShell](reset-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) en premier lieu.

## <a name="overview-of-the-process"></a>Vue d’ensemble du processus
La procédure de réinitialisation d’un mot de passe local pour une machine virtuelle Windows dans Azure lorsque l’agent invité Azure est inaccessible est la suivante :

1. Arrêter la machine virtuelle affectée.
1. Créez une capture instantanée du disque de système d’exploitation de la machine virtuelle.
1. Créez une copie du disque de système d’exploitation à partir de la capture instantanée.
1. Attachez et montez le disque de système d’exploitation copié sur une autre machine virtuelle Windows, puis créez des fichiers de configuration sur le disque. Ces fichiers facilitent la réinitialisation du mot de passe.
1. Démontez et détachez le disque de système d’exploitation copié de la machine virtuelle de dépannage.
1. Remplacez le disque de système d’exploitation de la machine virtuelle affectée.

## <a name="detailed-steps-for-the-vm-with-resource-manager-deployment"></a>Étapes détaillées pour la machine virtuelle avec le déploiement de Resource Manager

> [!NOTE]
> Les étapes ne s’appliquent pas aux contrôleurs de domaine Windows. Elles fonctionnent uniquement sur un serveur autonome ou un serveur qui est membre d’un domaine.

Essayez toujours de réinitialiser un mot de passe à l’aide du [portail Azure ou Azure PowerShell](reset-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) avant d’effectuer la procédure suivante. Avant de commencer, vérifiez que vous disposez d’une sauvegarde de votre machine virtuelle.

1. Effectuez une capture instantanée du disque du système d’exploitation de la machine virtuelle affectée, créez un disque à partir de cette capture instantanée, puis attachez le disque à une machine virtuelle de dépannage. Pour en savoir plus, consultez [Résoudre les problèmes d’une machine virtuelle Windows en connectant le disque du système d’exploitation à une machine virtuelle de récupération à l’aide du portail Azure](troubleshoot-recovery-disks-portal-windows.md).
2. Connectez-vous à la machine virtuelle de dépannage à l’aide du Bureau à distance.
3. Créez `gpt.ini` dans `\Windows\System32\GroupPolicy` sur le lecteur de la machine virtuelle source (si le fichier gpt.ini existe, renommez-le gpt.ini.bak) :
   
   > [!WARNING]
   > Veillez à ne pas accidentellement créer les fichiers suivants dans C:\Windows, le lecteur de système d’exploitation de la machine virtuelle de dépannage. Créez les fichiers suivants dans le lecteur du système d’exploitation de votre machine virtuelle source qui est attaché en tant que disque de données.
   
   * Ajoutez les lignes suivantes au fichier `gpt.ini` que vous avez créé :
     
     ```
     [General]
     gPCFunctionalityVersion=2
     gPCMachineExtensionNames=[{42B5FAAE-6536-11D2-AE5A-0000F87571E3}{40B6664F-4972-11D1-A7CA-0000F87571E3}]
     Version=1
     ```
     
     :::image type="content" source="./media/reset-local-password-without-agent/create-gpt-ini.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::

4. Créez `scripts.ini` dans `\Windows\System32\GroupPolicy\Machine\Scripts\`. Vérifiez que les dossiers masqués sont affichés. Si nécessaire, créez les dossiers `Machine` ou `Scripts`. 
   
   * Ajoutez les lignes suivantes au fichier `scripts.ini` que vous avez créé :
     
     ```
     [Startup]
     0CmdLine=FixAzureVM.cmd
     0Parameters=
     ```
     
     :::image type="content" source="./media/reset-local-password-without-agent/create-scripts-ini-1.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini" <username> /add
    ```

    :::image type="content" source="./media/reset-local-password-without-agent/create-fixazure-cmd-1.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::
   
    Vous devez respecter les exigences de complexité du mot de passe configuré pour votre machine virtuelle lors de la définition du nouveau mot de passe.

6. Dans le portail Azure, détachez le disque de la machine virtuelle de dépannage.

7. [Changez le disque de système d’exploitation pour la machine virtuelle affectée](troubleshoot-recovery-disks-portal-windows.md#swap-the-os-disk-for-the-vm).

8. Une fois la nouvelle machine virtuelle exécutée, connectez-vous à elle à l’aide du Bureau à distance et du nouveau mot de passe que vous avez spécifié dans le script `FixAzureVM.cmd`.

9. Dans votre session à distance vers la nouvelle machine virtuelle, supprimez les fichiers suivants pour nettoyer l’environnement :
    
    * À partir de %windir%\System32\GroupPolicy\Machine\Scripts\Startup
      * supprimez FixAzureVM.cmd
    * À partir de %windir%\System32\GroupPolicy\Machine\Scripts
      * supprimez scripts.ini
    * Dans %windir%\System32\GroupPolicy
      * supprimez gpt.ini (si le fichier gpt.ini existait déjà, et que vous l’avez renommé gpt.ini.bak, renommez à nouveau le fichier .bak « gpt.ini »)

## <a name="detailed-steps-for-classic-vm"></a>Étapes détaillées pour une machine virtuelle classique

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

> [!NOTE]
> Les étapes ne s’appliquent pas aux contrôleurs de domaine Windows. Elles fonctionnent uniquement sur un serveur autonome ou un serveur qui est membre d’un domaine.

Essayez toujours de réinitialiser un mot de passe à l’aide du [portail Azure ou Azure PowerShell](/previous-versions/azure/virtual-machines/windows/classic/reset-rdp) avant d’effectuer la procédure suivante. Avant de commencer, vérifiez que vous disposez d’une sauvegarde de votre machine virtuelle. 

1. Supprimez la machine virtuelle affectée dans le portail Azure. La suppression de la machine virtuelle ne supprime que les métadonnées, la référence de la machine virtuelle dans Azure. Les disques virtuels sont conservés lors de la suppression de la machine virtuelle :
   
   * Sélectionnez la machine virtuelle dans le portail Azure, puis cliquez sur *Supprimer* :
     
     :::image type="content" source="./media/reset-local-password-without-agent/delete-vm-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::

2. Attachez le disque de système d’exploitation de la machine virtuelle source à la machine virtuelle de dépannage. La machine virtuelle de dépannage doit être dans la même région que le disque de système d’exploitation de la machine virtuelle source (par exemple `West US`) :
   
   1. Sélectionnez la machine virtuelle de dépannage dans le portail Azure. Cliquez sur *Disques* | *Attacher existant* :
     
      :::image type="content" source="./media/reset-local-password-without-agent/disks-attach-existing-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::
     
   2. Sélectionnez *Fichier VHD*, puis le compte de stockage qui contient votre machine virtuelle source :
     
      :::image type="content" source="./media/reset-local-password-without-agent/disks-select-storage-account-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::
     
   3. Cochez la case *Afficher les comptes de stockage classiques*, puis sélectionnez le conteneur source. En général, le conteneur source est *disques durs virtuels* :
     
      :::image type="content" source="./media/reset-local-password-without-agent/disks-select-container-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::

      :::image type="content" source="./media/reset-local-password-without-agent/disks-select-container-vhds-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::
     
   4. Sélectionnez le disque dur virtuel de système d’exploitation à attacher. Cliquez sur *Sélectionner* pour terminer le processus :
     
      :::image type="content" source="./media/reset-local-password-without-agent/disks-select-source-vhd-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::

   5. Cliquer sur Ok pour attacher le disque

      :::image type="content" source="./media/reset-local-password-without-agent/disks-attach-okay-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::

3. Connectez-vous à la machine virtuelle de dépannage à l’aide du Bureau à distance et vérifiez que le disque de système d’exploitation de la machine virtuelle source est visible :

   1. Sélectionnez la machine virtuelle de dépannage dans le portail Azure et cliquez sur *Se connecter*.

   2. Ouvrez le fichier RDP qui télécharge. Entrez le nom d’utilisateur et le mot de passe de la machine virtuelle de dépannage.

   3. Dans l’Explorateur de fichiers, recherchez le disque de données que vous avez attaché. Si le disque dur virtuel de la machine virtuelle source est le seul disque de données attaché à la machine virtuelle de dépannage, il doit s’agir du lecteur F: :
     
      :::image type="content" source="./media/reset-local-password-without-agent/troubleshooting-vm-file-explorer-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::

4. Créez `gpt.ini` dans `\Windows\System32\GroupPolicy` sur le lecteur de la machine virtuelle source (si le fichier `gpt.ini` existe, renommez-le `gpt.ini.bak`) :
   
   > [!WARNING]
   > Veillez à ne pas accidentellement créer les fichiers suivants dans `C:\Windows`, le lecteur de système d’exploitation de la machine virtuelle de dépannage. Créez les fichiers suivants dans le lecteur du système d’exploitation de votre machine virtuelle source qui est attaché en tant que disque de données.
   
   * Ajoutez les lignes suivantes au fichier `gpt.ini` que vous avez créé :
     
     ```
     [General]
     gPCFunctionalityVersion=2
     gPCMachineExtensionNames=[{42B5FAAE-6536-11D2-AE5A-0000F87571E3}{40B6664F-4972-11D1-A7CA-0000F87571E3}]
     Version=1
     ```
     
     :::image type="content" source="./media/reset-local-password-without-agent/create-gpt-ini-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::

5. Créez `scripts.ini` dans `\Windows\System32\GroupPolicy\Machine\Scripts\`. Vérifiez que les dossiers masqués sont affichés. Si nécessaire, créez les dossiers `Machine` ou `Scripts`.
   
   * Ajoutez les lignes suivantes au fichier `scripts.ini` que vous avez créé :

     ```
     [Startup]
     0CmdLine=FixAzureVM.cmd
     0Parameters=
     ```
     
     :::image type="content" source="./media/reset-local-password-without-agent/create-scripts-ini-classic-1.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini" <username> /add
    ```

    :::image type="content" source="./media/reset-local-password-without-agent/create-fixazure-cmd-1.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::
   
    Vous devez respecter les exigences de complexité du mot de passe configuré pour votre machine virtuelle lors de la définition du nouveau mot de passe.

7. Dans le portail Azure, détachez le disque de la machine virtuelle de dépannage :
   
   1. Sélectionnez la machine virtuelle de dépannage dans le portail Azure, puis cliquez sur *Disques*.
   
   2. Sélectionnez le disque de données attaché à l’étape 2, cliquez sur **Détacher**, puis cliquez sur **OK**.

     :::image type="content" source="./media/reset-local-password-without-agent/data-disks-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::
     
     :::image type="content" source="./media/reset-local-password-without-agent/detach-disk-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::

8. Créez une machine virtuelle à partir du disque de système d’exploitation de la machine virtuelle source :
   
     :::image type="content" source="./media/reset-local-password-without-agent/create-new-vm-from-template-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::

     :::image type="content" source="./media/reset-local-password-without-agent/choose-subscription-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::

     :::image type="content" source="./media/reset-local-password-without-agent/create-vm-classic.png" alt-text="Capture d’écran montrant les mises à jour apportées au fichier gpt.ini":::

## <a name="complete-the-create-virtual-machine-experience"></a>Compléter l'expérience Créer une machine virtuelle

1. Une fois la nouvelle machine virtuelle exécutée, connectez-vous à elle à l’aide du Bureau à distance et du nouveau mot de passe que vous avez spécifié dans le script `FixAzureVM.cmd`.

2. Dans votre session à distance vers la nouvelle machine virtuelle, supprimez les fichiers suivants pour nettoyer l’environnement :
    
    * De `%windir%\System32\GroupPolicy\Machine\Scripts\Startup\`
      * supprimez `FixAzureVM.cmd`
    * De `%windir%\System32\GroupPolicy\Machine\Scripts`
      * supprimez `scripts.ini`
    * De `%windir%\System32\GroupPolicy`
      * supprimez `gpt.ini` (si `gpt.ini` existait auparavant, et que vous l'avez renommé `gpt.ini.bak`, renommez le fichier `.bak` en `gpt.ini`)

## <a name="next-steps"></a>Étapes suivantes
Si vous ne parvenez toujours pas à vous connecter à l’aide du Bureau à distance, consultez le [guide de résolution des problèmes de connexion Bureau à distance](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Le [guide détaillé de résolution des problèmes de connexion Bureau à distance](detailed-troubleshoot-rdp.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) décrit des méthodes de résolution des problèmes plutôt que des procédures spécifiques. Vous pouvez également [ouvrir une demande de support Azure](https://azure.microsoft.com/support/options/) pour obtenir une assistance pratique.
