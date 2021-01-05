---
title: Résoudre les problèmes d’une machine virtuelle Windows dans le portail Azure | Microsoft Docs
description: Découvrez comment résoudre les problèmes de machines virtuelles Windows dans Azure en connectant le disque du système d’exploitation à une machine virtuelle de récupération à l’aide du portail Azure.
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 08/19/2018
ms.author: genli
ms.openlocfilehash: 04adf3903cd925f4507e94347251c762a576ff93
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94735226"
---
# <a name="troubleshoot-a-windows-vm-by-attaching-the-os-disk-to-a-recovery-vm-through-the-azure-portal"></a>Résoudre les problèmes d’une machine virtuelle Windows en connectant le disque du système d’exploitation à une machine virtuelle de récupération par l’intermédiaire du portail Azure
Si votre machine virtuelle Windows dans Azure rencontre une erreur de démarrage ou de disque, il vous faudra éventuellement appliquer la procédure de dépannage sur le disque dur virtuel (VHD). Comme exemple courant, citons l’échec de mise à jour d’une application qui empêche le bon démarrage de la machine virtuelle. Cet article vous explique comment utiliser le portail Azure pour connecter votre disque dur virtuel à une autre machine virtuelle Windows pour corriger les éventuelles erreurs, puis pour recréer votre machine virtuelle d’origine. 

## <a name="recovery-process-overview"></a>Vue d’ensemble du processus de récupération
Le processus de résolution de problème se présente comme suit :

1. Arrêter la machine virtuelle affectée.
1. Créez une capture instantanée du disque de système d’exploitation de la machine virtuelle.
1. Créez un disque dur virtuel à partir de la capture instantanée.
1. Fixez et montez le disque dur virtuel sur une autre machine virtuelle Windows, à des fins de résolution des problèmes.
1. Connectez-vous à la machine virtuelle de dépannage. Modifiez les fichiers ou exécutez des outils afin de corriger les problèmes sur le disque dur virtuel d’origine.
1. Démontez le disque dur virtuel d’origine et dissociez-le de la machine virtuelle de dépannage.
1. Remplacez le disque du système d'exploitation de la machine virtuelle.

> [!NOTE]
> Cet article ne s’applique pas aux machines virtuelles dotées de disques non managés.

## <a name="take-a-snapshot-of-the-os-disk"></a>Prendre une capture instantanée du disque de système d’exploitation
Une capture instantanée est une copie complète en lecture seule d’un disque dur virtuel. Nous vous recommandons d'arrêter convenablement la machine virtuelle avant d'effectuer une capture instantanée, de façon à mettre fin à tous les processus en cours. Pour effectuer une capture instantanée d'un disque de système d'exploitation, procédez comme suit :

1. Accédez au [portail Azure](https://portal.azure.com). Sélectionnez **Machines virtuelles** dans la barre latérale, puis cliquez sur la machine virtuelle qui rencontre le problème.
1. Dans le volet gauche, sélectionnez **Disques**, puis cliquez sur le nom du disque du système d'exploitation.
    ![Capture d’écran montrant le nom du disque du système d’exploitation dans les paramètres du disque.](./media/troubleshoot-recovery-disks-portal-windows/select-osdisk.png)
1. Sur la page **Vue d’ensemble** du disque du système d’exploitation, sélectionnez **Créer une capture instantanée**.
1. Créez une capture instantanée au même emplacement que le disque du système d'exploitation.

## <a name="create-a-disk-from-the-snapshot"></a>Créer un disque à partir de la capture instantanée
Pour créer un disque à partir de la capture instantanée, procédez comme suit :

1. Sélectionnez **Cloud Shell** sur le portail Azure.

    ![Capture d’écran montrant le bouton permettant d’ouvrir Azure Cloud Shell.](./media/troubleshoot-recovery-disks-portal-windows/cloud-shell.png)
1. Exécutez les commandes PowerShell suivantes pour créer un disque managé à partir de la capture instantanée. Remplacez ces exemples de noms par les noms appropriés.

    ```powershell
    #Provide the name of your resource group.
    $resourceGroupName ='myResourceGroup'
    
    #Provide the name of the snapshot that will be used to create managed disks.
    $snapshotName = 'mySnapshot' 
    
    #Provide the name of the managed disk.
    $diskName = 'newOSDisk'
    
    #Provide the size of the disks in gigabytes. It should be greater than the VHD file size. In this example, the size of the snapshot is 127 GB. So we set the disk size to 128 GB.
    $diskSize = '128'
    
    #Provide the storage type for the managed disk: Premium_LRS or Standard_LRS.
    $storageType = 'Standard_LRS'
    
    #Provide the Azure region (for example, westus) where the managed disks will be located.
    #This location should be the same as the snapshot location.
    #Get all the Azure locations by using this command:
    #Get-AzLocation
    $location = 'westus'
    
    $snapshot = Get-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotName 
     
    $diskConfig = New-AzDiskConfig -AccountType $storageType -Location $location -CreateOption Copy -SourceResourceId $snapshot.Id
     
    New-AzDisk -Disk $diskConfig -ResourceGroupName $resourceGroupName -DiskName $diskName
    ```
3. Si les commandes s’exécutent correctement, le nouveau disque apparaît dans le groupe de ressources que vous avez fourni.

## <a name="attach-the-disk-to-another-vm"></a>Joignez le disque à une autre machine virtuelle
Pour les prochaines étapes, vous utilisez une autre machine virtuelle à des fins de résolution des problèmes. Après avoir attaché le disque à la machine virtuelle de dépannage, vous pouvez parcourir et modifier le contenu de celui-ci. Ce processus vous permet de corriger les éventuelles erreurs de configuration ou d'examiner d'autres fichiers journaux du système ou des applications. Pour associer le disque à une autre machine virtuelle, procédez comme suit :

1. Sélectionnez votre groupe de ressources à partir du portail, puis sélectionnez votre machine virtuelle de dépannage. Sélectionnez **Disques** > **Modifier** > **Ajouter un disque de données**.

    ![Capture d’écran montrant les sélections permettant d’attacher un disque existant dans le portail.](./media/troubleshoot-recovery-disks-portal-windows/attach-existing-disk.png)

2. Dans la liste **Disques de données**, sélectionnez le disque de système d’exploitation de la machine virtuelle que vous avez identifié. Si vous ne voyez pas le disque de système d’exploitation, assurez-vous que la machine virtuelle de dépannage et le disque de système d’exploitation se trouvent dans la même région (emplacement). 
3. Sélectionnez **Enregistrer** pour appliquer la modification.

## <a name="mount-the-attached-data-disk-to-the-vm"></a>Monter le disque de données joint sur une machine virtuelle

1. Ouvrez une connexion Bureau à distance à la machine virtuelle de résolution des problèmes. 
2. Dans la machine virtuelle de dépannage, ouvrez **Gestionnaire de serveur**, puis sélectionnez **Services de fichiers et de stockage**. 

    ![Capture d’écran montrant la sélection de Services de fichiers et de stockage dans Gestionnaire de serveur.](./media/troubleshoot-recovery-disks-portal-windows/server-manager-select-storage.png)

3. Le disque de données est automatiquement détecté et attaché. Pour afficher la liste des disques connectés, sélectionnez **Disques**. Vous pouvez sélectionner le disque de données pour afficher les informations sur le volume, notamment la lettre de lecteur. L’exemple suivant présente le disque de données attaché utilisant le lecteur **F:** .

    ![Capture d’écran montrant les informations d’un disque et d’un volume attachés dans Gestionnaire de serveur.](./media/troubleshoot-recovery-disks-portal-windows/server-manager-disk-attached.png)

## <a name="fix-problems-on-the-original-virtual-hard-disk"></a>Corriger des problèmes sur le disque dur virtuel d’origine
Avec le disque dur virtuel existant monté, vous pouvez désormais exécuter les procédures de maintenance et de dépannage requises. Une fois que vous avez résolu toutes les erreurs, passez aux étapes suivantes.

## <a name="unmount-and-detach-the-original-virtual-hard-disk"></a>Démonter et dissocier le disque dur virtuel d’origine
Détachez le disque dur virtuel existant de votre machine virtuelle de dépannage. Vous ne pouvez pas utiliser votre disque dur virtuel avec une autre machine virtuelle avant la libération du bail associant le disque dur virtuel à la machine virtuelle de dépannage.

1. Dans la session Bureau à distance sur votre machine virtuelle, ouvrez **Gestionnaire de serveur**, puis sélectionnez **Services de fichiers et de stockage**.

    ![Capture d’écran montrant la sélection de Services de fichiers et de stockage dans Gestionnaire de serveur.](./media/troubleshoot-recovery-disks-portal-windows/server-manager-select-storage.png)

2. Sélectionnez **Disques**, cliquez avec le bouton droit sur votre disque de données et sélectionnez **Mettre hors connexion**.

    ![Capture d’écran montrant le paramétrage du disque de données hors connexion dans Gestionnaire de serveur.](./media/troubleshoot-recovery-disks-portal-windows/server-manager-set-disk-offline.png)

3. Détachez le disque dur virtuel de la machine virtuelle. Sélectionnez votre machine virtuelle dans le portail Azure, puis sélectionnez **Disques**. 
4. Sélectionnez **Modifier**, puis le disque de système d’exploitation que vous avez attaché, enfin **Supprimer**.

    ![Capture d’écran montrant les sélections permettant de détacher un disque dur virtuel existant.](./media/troubleshoot-recovery-disks-portal-windows/detach-disk.png)

    Avant de continuer, attendez que le disque de données soit correctement supprimé de la machine virtuelle.

## <a name="swap-the-os-disk-for-the-vm"></a>Remplacer le disque du système d'exploitation de la machine virtuelle

Le portail Azure prend désormais en charge le remplacement du disque du système d’exploitation de la machine virtuelle. Pour ce faire, procédez comme suit :

1. Accédez au [portail Azure](https://portal.azure.com). Sélectionnez **Machines virtuelles** dans la barre latérale, puis cliquez sur la machine virtuelle qui rencontre le problème.
1. Dans le volet gauche, sélectionnez **Disques**, puis **Remplacer le disque du système d’exploitation**.
   
    ![Capture d’écran montrant le bouton permettant de remplacer un disque de système d’exploitation dans le portail Azure.](./media/troubleshoot-recovery-disks-portal-windows/swap-os-ui.png)

1. Choisissez le nouveau disque que vous avez réparé, puis entrez le nom de la machine virtuelle pour confirmer le remplacement. Si le disque ne figure pas dans la liste, attendez 10 à 15 minutes après avoir détaché le disque de la machine virtuelle de dépannage. Assurez-vous également que le disque se trouve au même emplacement que la machine virtuelle.
1. Sélectionnez **OK**.

## <a name="next-steps"></a>Étapes suivantes
Si vous rencontrez des problèmes pour vous connecter à votre machine virtuelle, consultez [Résolution des problèmes de connexion Bureau à distance avec une machine virtuelle Azure](troubleshoot-rdp-connection.md). Pour résoudre les problèmes liés à l’accès aux applications exécutées sur votre machine virtuelle, consultez [Résoudre les problèmes de connectivité des applications sur une machine virtuelle Windows](troubleshoot-app-connection.md).

Pour plus d’informations sur l’utilisation d’Azure Resource Manager, consultez la page [Présentation d’Azure Resource Manager](../../azure-resource-manager/management/overview.md).


