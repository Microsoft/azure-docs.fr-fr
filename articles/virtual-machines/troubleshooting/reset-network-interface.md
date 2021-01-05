---
title: Procédure de réinitialisation de l’interface réseau pour les machines virtuelles Windows Azure | Microsoft Docs
description: Montre comment réinitialiser l’interface réseau pour une machine virtuelle Windows Azure
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue, azure-resource-manager
ms.service: virtual-machines-windows
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 11/16/2018
ms.author: genli
ms.openlocfilehash: 6c4e507df0f112934979d4e59778b667743cf623
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022902"
---
# <a name="how-to-reset-network-interface-for-azure-windows-vm"></a>Comment réinitialiser l’interface réseau pour une machine virtuelle Windows Azure 

Cet article explique comment réinitialiser l’interface réseau d’une machine virtuelle Azure Windows pour résoudre les problèmes quand vous ne pouvez pas vous connecter à la machine virtuelle Microsoft Azure Windows après :

* Avoir désactivé l’interface réseau par défaut. 
* Avoir défini manuellement une adresse IP statique pour la carte réseau. 

[!INCLUDE [support-disclaimer](../../../includes/support-disclaimer.md)]

## <a name="reset-network-interface"></a>Réinitialiser l’interface réseau

### <a name="for-vms-deployed-in-resource-group-model"></a>Pour les machines virtuelles déployées dans le modèle de groupe de ressources

1.  Accédez au [portail Azure](https://ms.portal.azure.com).
2.  Sélectionnez la machine virtuelle concernée.
3.  Sélectionnez **Mise en réseau**, puis sélectionnez l’interface réseau de la machine virtuelle.

    ![Emplacement de l’interface réseau](./media/reset-network-interface/select-network-interface-vm.png)
    
4.  Sélectionnez **Configurations IP**.
5.  Sélectionnez l’adresse IP. 
6.  Si l’**attribution d’adresse IP privée** n’est pas **Statique**, choisissez **Statique**.
7.  Modifiez l’**adresse IP** pour indiquer une autre adresse IP, disponible dans le sous-réseau.
8. La machine virtuelle redémarre afin d’initialiser la nouvelle carte réseau pour le système.
9.  Essayez d’utiliser le protocole RDP sur votre machine. En cas de réussite, vous pouvez redéfinir l’adresse IP privée d’origine si vous le souhaitez. Sinon, vous pouvez conserver cette adresse. 

#### <a name="use-azure-powershell"></a>Utilisation d'Azure PowerShell

1. Vérifiez que la [dernière version d’Azure PowerShell](/powershell/azure/) est installée.
2. Ouvrez une session Azure PowerShell avec élévation de privilèges (Exécuter en tant qu’administrateur). Exécutez les commandes suivantes :

    ```powershell
    #Set the variables 
    $SubscriptionID = "<Subscription ID>"
    $VM = "<VM Name>"
    $ResourceGroup = "<Resource Group>"
    $VNET = "<Virtual Network>"
    $IP = "NEWIP"

    #Log in to the subscription 
    Add-AzAccount
    Select-AzSubscription -SubscriptionId $SubscriptionId 
    
    #Check whether the new IP address is available in the virtual network.
    Test-AzureStaticVNetIP –VNetName $VNET –IPAddress  $IP

    #Add/Change static IP. This process will not change MAC address
    Get-AzVM -ResourceGroupName $ResourceGroup -Name $VM | Set-AzureStaticVNetIP -IPAddress $IP | Update-AzVM
    ```
3. Essayez d’utiliser le protocole RDP sur votre machine.  En cas de réussite, vous pouvez redéfinir l’adresse IP privée d’origine si vous le souhaitez. Sinon, vous pouvez conserver cette adresse.

### <a name="for-classic-vms"></a>Pour les machines virtuelles Classic

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Pour réinitialiser l’interface réseau, procédez comme suit :

#### <a name="use-azure-portal"></a>Utiliser le portail Azure

1.  Accédez au [portail Azure]( https://ms.portal.azure.com).
2.  Sélectionnez **Machines virtuelles (Classic)** .
3.  Sélectionnez la machine virtuelle concernée.
4.  Sélectionnez **Adresses IP**.
5.  Si l’**attribution d’adresse IP privée** n’est pas **Statique**, choisissez **Statique**.
6.  Modifiez l’**adresse IP** pour indiquer une autre adresse IP, disponible dans le sous-réseau.
7.  Sélectionnez **Enregistrer**.
8.  La machine virtuelle redémarre afin d’initialiser la nouvelle carte réseau pour le système.
9.  Essayez d’utiliser le protocole RDP sur votre machine. En cas de réussite, vous pouvez rétablir l’adresse IP privée d’origine.  

#### <a name="use-azure-powershell"></a>Utilisation d'Azure PowerShell

1. Vérifiez que la [dernière version d’Azure PowerShell](/powershell/azure/) est installée.
2. Ouvrez une session Azure PowerShell avec élévation de privilèges (Exécuter en tant qu’administrateur). Exécutez les commandes suivantes :

    ```powershell
    #Set the variables 
    $SubscriptionID = "<Subscription ID>"
    $VM = "<VM Name>"
    $CloudService = "<Cloud Service>"
    $VNET = "<Virtual Network>"
    $IP = "NEWIP"

    #Log in to the subscription 
    Add-AzureAccount
    Select-AzureSubscription -SubscriptionId $SubscriptionId 

    #Check whether the new IP address is available in the virtual network.
    Test-AzureStaticVNetIP –VNetName $VNET –IPAddress  $IP
    
    #Add/Change static IP. This process will not change MAC address
    Get-AzureVM -ResourceGroupName $CloudService -Name $VM | Set-AzureStaticVNetIP -IPAddress $IP |Update-AzureVM
    ```
3. Essayez d’utiliser le protocole RDP sur votre machine. En cas de réussite, vous pouvez redéfinir l’adresse IP privée d’origine si vous le souhaitez. Sinon, vous pouvez conserver cette adresse. 

## <a name="delete-the-unavailable-nics"></a>Supprimer les cartes réseau non disponibles
Une fois que vous pouvez vous connecter directement à la machine à l’aide du bureau à distance, vous devez supprimer les anciennes cartes réseau pour éviter tout problème potentiel :

1.  Ouvrez le Gestionnaire de périphériques.
2.  Sélectionnez **Afficher** > **Afficher les périphériques cachés**.
3.  Sélectionnez **Adaptateurs réseau**. 
4.  Recherchez les adaptateurs nommés « Carte réseau de bus Microsoft Hyper-V ».
5.  Il se peut qu’un adaptateur non disponible apparaisse grisé. Cliquez avec le bouton droit sur l’adaptateur, puis sélectionnez Désinstaller.

    ![Image de la carte réseau](media/reset-network-interface/nicpage.png)

    > [!NOTE]
    > Désinstallez uniquement les adaptateurs non disponibles qui portent le nom « Carte réseau de bus Microsoft Hyper-V ». Si vous désinstallez un autre adaptateur caché, cela peut entraîner des problèmes supplémentaires.
    >
    >

6.  À présent, tous les adaptateurs indisponibles ont normalement été effacés de votre système.
