---
title: Créer une machine virtuelle Windows avec mise en réseau accélérée - Azure PowerShell
description: Créez une machine virtuelle Windows avec une mise en réseau accélérée afin d’améliorer ses performances de mise en réseau.
services: virtual-network
documentationcenter: ''
author: gsilva5
manager: gedegrac
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 04/15/2020
ms.author: gsilva
ms.openlocfilehash: fd50af98fe0d7f20273c45e2b86c18215a3626f0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87289619"
---
# <a name="create-a-windows-vm-with-accelerated-networking-using-azure-powershell"></a>Créer une machine virtuelle Windows avec mise en réseau accélérée à l’aide d’Azure PowerShell

Ce didacticiel explique comment créer une machine virtuelle Windows (VM) avec mise en réseau accélérée.

> [!NOTE]
> Pour utiliser la mise en réseau accélérée avec une machine virtuelle Linux, consultez [Créer une machine virtuelle Linux avec mise en réseau accélérée](create-vm-accelerated-networking-cli.md).

Une mise en réseau accélérée permet d’opérer une virtualisation d’E/S d’une racine unique (SR-IOV) sur une machine virtuelle, ce qui améliore considérablement les performances de mise en réseau. Cette voie hautement performante court-circuite l’hôte à partir du chemin d’accès aux données, ce qui réduit la latence, l’instabilité et l’utilisation du processeur pour les charges de travail réseau les plus exigeantes sur les types de machines virtuelles pris en charge. Le diagramme suivant illustre la façon dont deux machines virtuelles communiquent avec et sans mise en réseau accélérée :

![Communication entre des machines virtuelles Azure avec et sans mise en réseau accélérée](./media/create-vm-accelerated-networking/accelerated-networking.png)

Sans mise en réseau accélérée, tout le trafic réseau en direction et en provenance de la machine virtuelle doit transiter par l’hôte et le commutateur virtuel. Le commutateur virtuel fournit au trafic réseau toutes les stratégies, telles que les groupes de sécurité réseau, les listes de contrôle d’accès, l’isolation et d’autres services de réseau virtualisé.

> [!NOTE]
> Pour plus d’informations sur les commutateurs virtuels, consultez [Commutateur virtuel Hyper-V](/windows-server/virtualization/hyper-v-virtual-switch/hyper-v-virtual-switch).

Dans le cas d’une mise en réseau accélérée, le trafic réseau parvient à la carte réseau de la machine virtuelle avant d’être transféré vers celle-ci. Toutes les stratégies réseau que le commutateur virtuel applique sont déchargées et appliquées dans le matériel. Étant donné que la stratégie est appliquée au matériel, la carte réseau peut transférer le trafic réseau directement à la machine virtuelle. La carte d’interface réseau ignore l’hôte et le commutateur virtuel, alors qu’elle gère l’ensemble de la stratégie appliquée à l’hôte.

Les avantages d’une mise en réseau accélérée s’appliquent uniquement à la machine virtuelle activée. Pour des résultats optimaux, activez cette fonctionnalité sur au moins deux machines virtuelles connectées au même réseau virtuel Azure. Lors de la communication entre réseaux virtuels ou d’une connexion locale, cette fonctionnalité a une incidence minimale sur la latence globale.

## <a name="benefits"></a>Avantages

- **Latence plus faible/Nombre supérieur de paquets par seconde (pps)**  : L’élimination du commutateur virtuel du chemin d’accès aux données supprime le temps que les paquets passent dans l’hôte pour le traitement de la stratégie. Cela augmente également le nombre de paquets qui peuvent être traités à l’intérieur de la machine virtuelle.

- **Instabilité réduite** : Le traitement du commutateur virtuel dépend de la quantité de stratégie qui doit être appliquée. Cela dépend également de la charge de travail du processeur qui exécute le traitement. Le déchargement de l’application de la stratégie sur le matériel supprime cette variabilité en livrant des paquets directement à la machine virtuelle. Le déchargement supprime également la communication entre l’hôte et la machine virtuelle, toutes les interruptions logicielles et tous les changements de contexte.

- **Utilisation réduite du processeur** : en contournant le commutateur virtuel de l'hôte, le processeur utilise moins de ressources pour traiter le trafic réseau.

## <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge

Les distributions suivantes sont prises en charge directement à partir de la galerie Azure :

- **Windows Server 2019 Datacenter**
- **Windows Server 2016 Datacenter** 
- **Windows Server 2012 R2 Datacenter**

## <a name="limitations-and-constraints"></a>Limitations et contraintes

### <a name="supported-vm-instances"></a>Instances de machines virtuelles prises en charge

La mise en réseau accélérée est prise en charge dans la plupart des instances d’usage général et optimisées pour le calcul avec deux processeurs virtuels ou plus.  Séries prises en charge : Dv2/DSv2 et F/Fs.

Dans des instances qui acceptent l’hyperthreading, la mise en réseau accélérée est prise en charge dans des instances de machine virtuelle comptant au minimum quatre processeurs virtuels. Séries prises en charge : D/Dsv3, D/Dsv4, Da/Dasv4, E/Esv3, Ea/Easv4, Fsv2, Lsv2, Ms/Mms, et Ms/Mmsv2.

Pour plus d’informations sur les instances de machine virtuelle, reportez-vous à [Tailles des machines virtuelles Windows dans Azure](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

### <a name="custom-images"></a>Images personnalisées

Si vous utilisez une image personnalisée et si votre image prend en charge l’accélération réseau, veillez à disposer des pilotes nécessaires compatibles avec les cartes réseau Mellanox ConnectX-3 et ConnectX-4 Lx dans Azure.

### <a name="regions"></a>Régions

La mise en réseau accélérée est disponible dans toutes les régions Azure globales et Azure Government Cloud.

### <a name="enabling-accelerated-networking-on-a-running-vm"></a>Activation de la mise en réseau accélérée sur une machine virtuelle en cours d’exécution

Cette fonctionnalité peut être activée sur une machine virtuelle sans mise en réseau accélérée uniquement si la machine virtuelle a une taille prise en charge et si elle est arrêtée/libérée.

### <a name="deployment-through-azure-resource-manager"></a>Déploiement par le biais d’Azure Resource Manager

Le déploiement de machines virtuelles (classiques) avec mise en réseau accélérée n’est pas possible.

## <a name="vm-creation-using-the-portal"></a>Création de machines virtuelles à l’aide du portail

Bien que cet article fournisse des étapes pour créer une machine virtuelle avec mise en réseau accélérée à l’aide d’Azure PowerShell, vous pouvez également [utiliser le Portail Azure pour créer une machine virtuelle](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) avec mise en réseau accélérée. Lorsque vous créez une machine virtuelle dans le portail, sur la page **Créer une machine virtuelle**, choisissez l’onglet **Mise en réseau**. Cet onglet contient une option de **Mise en réseau accélérée**. Si vous avez choisi un [système d’exploitation pris en charge](#supported-operating-systems) et une [taille de machine virtuelle](#supported-vm-instances), cette option est automatiquement définie sur **Activé**. Dans le cas contraire, l’option est définie sur **Désactivé**et Azure affiche la raison pour laquelle elle ne peut pas être activée.

> [!NOTE]
> Seuls les systèmes d’exploitation pris en charge peuvent être activés via le portail. Si vous utilisez une image personnalisée, et que votre image prend en charge la mise en réseau accélérée, créez votre machine virtuelle à l’aide de CLI ou de PowerShell. 

Une fois la machine virtuelle créée, vous pouvez vérifier si la mise en réseau accélérée est activée. Suivez ces instructions :

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer vos machines virtuelles. Recherchez et sélectionnez **Machines virtuelles**.

2. Dans la liste des machines virtuelles, choisissez votre nouvelle machine virtuelle.

3. Dans la barre de menus de la machine virtuelle, choisissez **Mise en réseau**.

Dans les informations de l’interface réseau, en regard de l’étiquette **Mise en réseau accélérée**, le portail affiche **Désactivé** ou **Activé** pour l’état de mise en réseau accélérée.

## <a name="vm-creation-using-powershell"></a>Création de machines virtuelles à l’aide de PowerShell

Avant de continuer, installez [Azure PowerShell](/powershell/azure/install-az-ps), version 1.0.0 ou ultérieure. Pour connaître la version actuellement installée, exécutez `Get-Module -ListAvailable Az`. Si vous devez installer ou mettre à niveau le module Az, installez la dernière version du module à partir de [PowerShell Gallery](https://www.powershellgallery.com/packages/Az). Dans une session PowerShell, connectez-vous à un compte Azure à l’aide de la commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

Dans les exemples suivants, remplacez les exemples de noms de paramètre par vos propres valeurs. Les noms de paramètre sont par exemple *myResourceGroup*, *myNic* et *myVM*.

### <a name="create-a-virtual-network"></a>Créez un réseau virtuel

1. Créez un groupe de ressources avec [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup). La commande suivante crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *centralus* :

    ```azurepowershell
    New-AzResourceGroup -Name "myResourceGroup" -Location "centralus"
    ```

2. Créez une configuration de sous-réseau à l’aide de la commande [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.Network/New-azVirtualNetworkSubnetConfig). La commande suivante permet de créer un sous-réseau nommé *mySubnet* :

    ```azurepowershell
    $subnet = New-AzVirtualNetworkSubnetConfig `
        -Name "mySubnet" `
        -AddressPrefix "192.168.1.0/24"
    ```

3. Créez un réseau virtuel à l’aide de la commande [New-AzVirtualNetwork](/powershell/module/az.Network/New-azVirtualNetwork), avec le sous-réseau *mySubnet*.

    ```azurepowershell
    $vnet = New-AzVirtualNetwork -ResourceGroupName "myResourceGroup" `
        -Location "centralus" `
        -Name "myVnet" `
        -AddressPrefix "192.168.0.0/16" `
        -Subnet $Subnet
    ```

### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

1. Créez une règle de groupe de sécurité réseau avec [New-AzNetworkSecurityRuleConfig](/powershell/module/az.Network/New-azNetworkSecurityRuleConfig).

    ```azurepowershell
    $rdp = New-AzNetworkSecurityRuleConfig `
        -Name 'Allow-RDP-All' `
        -Description 'Allow RDP' `
        -Access Allow `
        -Protocol Tcp `
        -Direction Inbound `
        -Priority 100 `
        -SourceAddressPrefix * `
        -SourcePortRange * `
        -DestinationAddressPrefix * `
        -DestinationPortRange 3389
    ```

2. Créez un groupe de sécurité réseau avec [New-AzNetworkSecurityGroup](/powershell/module/az.Network/New-azNetworkSecurityGroup) et attribuez-lui la règle de sécurité *Allow-RDP-All*. Outre la règle *Allow-RDP-All*, le groupe de sécurité réseau contient plusieurs règles par défaut. Une règle par défaut désactive tous les accès entrants à partir d’Internet. Une fois créée, la règle *Allow-RDP-All* est attribuée au groupe de sécurité réseau pour vous permettre de vous connecter à distance à la machine virtuelle.

    ```azurepowershell
    $nsg = New-AzNetworkSecurityGroup `
        -ResourceGroupName myResourceGroup `
        -Location centralus `
        -Name "myNsg" `
        -SecurityRules $rdp
    ```

3. Ajoutez le groupe de sécurité réseau au sous-réseau *mySubnet* à l’aide de la commande [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.Network/Set-azVirtualNetworkSubnetConfig). La règle du groupe de sécurité réseau est appliquée à toutes les ressources déployées dans le sous-réseau.

    ```azurepowershell
    Set-AzVirtualNetworkSubnetConfig `
        -VirtualNetwork $vnet `
        -Name 'mySubnet' `
        -AddressPrefix "192.168.1.0/24" `
        -NetworkSecurityGroup $nsg
    ```

### <a name="create-a-network-interface-with-accelerated-networking"></a>Créer une interface réseau avec mise en réseau accélérée

1. Créez une adresse IP publique avec [New-AzPublicIpAddress](/powershell/module/az.Network/New-azPublicIpAddress). Une adresse IP publique n’est pas nécessaire si vous n’envisagez pas d’accéder à la machine virtuelle à partir d’Internet. Toutefois, il est nécessaire d’effectuer les étapes décrites dans cet article.

    ```azurepowershell
    $publicIp = New-AzPublicIpAddress `
        -ResourceGroupName myResourceGroup `
        -Name 'myPublicIp' `
        -location centralus `
        -AllocationMethod Dynamic
    ```

2. Créez une interface réseau à l’aide de la commande [New-AzNetworkInterface](/powershell/module/az.Network/New-azNetworkInterface) avec la mise en réseau accélérée activée et attribuez l’adresse IP publique à l’interface réseau. L’exemple suivant permet de créer une interface réseau nommée *myNic* dans le sous-réseau *mySubnet* du réseau virtuel *myVnet* et de lui affecter l’adresse IP publique *myPublicIp* :

    ```azurepowershell
    $nic = New-AzNetworkInterface `
        -ResourceGroupName "myResourceGroup" `
        -Name "myNic" `
        -Location "centralus" `
        -SubnetId $vnet.Subnets[0].Id `
        -PublicIpAddressId $publicIp.Id `
        -EnableAcceleratedNetworking
    ```

### <a name="create-a-vm-and-attach-the-network-interface"></a>Créer une machine virtuelle et attacher l’interface réseau

1. Définissez les informations d’identification de votre machine virtuelle sur la variable `$cred` via la commande [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential), qui vous invite à vous connecter :

    ```azurepowershell
    $cred = Get-Credential
    ```

2. Définissez votre machine virtuelle avec la commande [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig). La commande suivante crée une machine virtuelle nommée *myVM* d’une taille compatible avec la mise en réseau accélérée (*Standard_DS4_v2*) :

    ```azurepowershell
    $vmConfig = New-AzVMConfig -VMName "myVm" -VMSize "Standard_DS4_v2"
    ```

    Pour obtenir la liste de toutes les tailles de machine virtuelle et leurs caractéristiques, consultez [Tailles de machines virtuelles Windows](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

3. Créez le reste de la configuration de votre machine virtuelle avec les commandes [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem) et [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage). La commande suivante crée une machine virtuelle Windows Server 2016 :

    ```azurepowershell
    $vmConfig = Set-AzVMOperatingSystem -VM $vmConfig `
        -Windows `
        -ComputerName "myVM" `
        -Credential $cred `
        -ProvisionVMAgent `
        -EnableAutoUpdate
    $vmConfig = Set-AzVMSourceImage -VM $vmConfig `
        -PublisherName "MicrosoftWindowsServer" `
        -Offer "WindowsServer" `
        -Skus "2016-Datacenter" `
        -Version "latest"
    ```

4. Associez les cartes d’interface réseau que vous avez créées précédemment à l’aide de la commande [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface) :

    ```azurepowershell
    $vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id
    ```

5. Créez votre machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm).

    ```azurepowershell
    New-AzVM -VM $vmConfig -ResourceGroupName "myResourceGroup" -Location "centralus"
    ```

### <a name="confirm-the-ethernet-controller-is-installed-in-the-windows-vm"></a>Confirmer que le contrôleur Ethernet est installé dans la machine virtuelle Windows

Une fois la machine virtuelle créée dans Azure, connectez-vous à cette dernière et vérifiez que le contrôleur Ethernet est installé dans Windows.

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer vos machines virtuelles. Recherchez et sélectionnez **Machines virtuelles**.

2. Dans la liste des machines virtuelles, choisissez votre nouvelle machine virtuelle.

3. Dans la page de présentation de la machine virtuelle, si l’**État** de la machine virtuelle est listé comme **Création**, patientez jusqu’à la fin de la création de la machine virtuelle par Azure. L’**État** sera modifié en **En cours d’exécution** une fois la création de la machine virtuelle terminée.

4. À partir de la barre d’outils de présentation de la machine virtuelle, sélectionnez **Connecter** > **RDP** > **Télécharger le fichier RDP**.

5. Ouvrez le fichier .rdp, puis connectez-vous à la machine virtuelle avec les informations d’identification que vous avez entrées dans la section [Créer une machine virtuelle et attacher l’interface réseau](#create-a-vm-and-attach-the-network-interface). Si vous ne vous êtes jamais connecté à une machine virtuelle dans Azure, consultez la section [Se connecter à une machine virtuelle](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json#connect-to-virtual-machine).

6. Une fois que la session Bureau à distance pour votre machine virtuelle s’affiche, cliquez avec le bouton droit sur le bouton Démarrer de Windows, puis choisissez **Gestionnaire d'appareils**.

7. Dans la fenêtre **Gestionnaire d'appareils**, développez le nœud **Cartes réseau**.

8. Vérifiez que la mention **Carte Ethernet Mellanox ConnectX-3 fonction virtuelle** s’affiche comme dans l’image suivante :

    ![Carte Ethernet Mellanox ConnectX-3 fonction virtuelle, nouvelle carte réseau pour la mise en réseau accélérée, Gestionnaire d'appareils](./media/create-vm-accelerated-networking/device-manager.png)

La mise en réseau accélérée est maintenant activée pour votre machine virtuelle.

> [!NOTE]
> Si la carte Mellanox ne parvient pas à démarrer, ouvrez une invite administrateur dans la session Bureau à distance et entrez la commande suivante :
>
> `netsh int tcp set global rss = enabled`

## <a name="enable-accelerated-networking-on-existing-vms"></a>Activer la mise en réseau accélérée sur des machines virtuelles existantes

Si vous avez créé une machine virtuelle sans mise en réseau accélérée, vous pouvez activer cette fonctionnalité sur une machine virtuelle existante. La machine virtuelle doit prendre en charge la mise en réseau accélérée en répondant aux conditions préalables suivantes, qui sont également décrites ci-dessus :

* La machine virtuelle doit avoir une taille prise en charge pour la mise en réseau accélérée.
* La machine virtuelle doit être une image de la galerie Azure prise en charge (et une version de noyau prise en charge pour Linux).
* Toutes les machines virtuelles d’un groupe à haute disponibilité ou d’un groupe de machines virtuelles identiques doivent être arrêtées ou libérées avant d’activer la mise en réseau accélérée sur une carte réseau.

### <a name="individual-vms-and-vms-in-an-availability-set"></a>Machines virtuelles individuelles et machines virtuelles d’un groupe à haute disponibilité

1. Arrêtez ou libérez la machine virtuelle individuelle ou, dans le cas d’un groupe à haute disponibilité, toutes les machines virtuelles du groupe :

    ```azurepowershell
    Stop-AzVM -ResourceGroup "myResourceGroup" -Name "myVM"
    ```

    > [!NOTE]
    > Lorsque vous créez une machine virtuelle individuellement, sans groupe à haute disponibilité, il vous suffit d’arrêter ou de libérer la machine virtuelle individuelle pour activer la mise en réseau accélérée. Si votre machine virtuelle a été créée avec un groupe à haute disponibilité, vous devez arrêter ou libérer toutes les machines virtuelles contenues dans le groupe à haute disponibilité avant d’activer la mise en réseau accélérée sur n’importe quelle carte réseau, afin que les machines virtuelles finissent sur un cluster qui prend en charge la mise en réseau accélérée. L’exigence d’arrêt ou de libération est inutile si vous désactivez la mise en réseau accélérée, car les clusters qui prennent en charge la mise en réseau accélérée fonctionnent également correctement avec les cartes réseau qui n’utilisent pas la mise en réseau accélérée.

2. Activez la mise en réseau accélérée sur la carte réseau de votre machine virtuelle :

    ```azurepowershell
    $nic = Get-AzNetworkInterface -ResourceGroupName "myResourceGroup" `
        -Name "myNic"
    
    $nic.EnableAcceleratedNetworking = $true
    
    $nic | Set-AzNetworkInterface
    ```

3. Redémarrez la machine virtuelle ou, dans le cas d’un groupe à haute disponibilité, toutes les machines virtuelles du groupe, puis vérifiez que la mise en réseau accélérée est bien activée :

    ```azurepowershell
    Start-AzVM -ResourceGroup "myResourceGroup" `
        -Name "myVM"
    ```

### <a name="virtual-machine-scale-set"></a>Jeu de mise à l’échelle de machine virtuelle

Un groupe de machines virtuelles identiques est légèrement différent, mais il suit le même flux de travail.

1. Arrêtez les machines virtuelles :

    ```azurepowershell
    Stop-AzVmss -ResourceGroupName "myResourceGroup" `
        -VMScaleSetName "myScaleSet"
    ```

2. Mettez à jour la propriété de mise en réseau accélérée sur l’interface réseau :

    ```azurepowershell
    $vmss = Get-AzVmss -ResourceGroupName "myResourceGroup" `
        -VMScaleSetName "myScaleSet"
    
    $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].EnableAcceleratedNetworking = $true
    
    Update-AzVmss -ResourceGroupName "myResourceGroup" `
        -VMScaleSetName "myScaleSet" `
        -VirtualMachineScaleSet $vmss
    ```

3. Définissez les mises à jour appliquées sur automatique afin que les modifications soient immédiatement récupérées :

    ```azurepowershell
    $vmss.UpgradePolicy.Mode = "Automatic"
    
    Update-AzVmss -ResourceGroupName "myResourceGroup" `
        -VMScaleSetName "myScaleSet" `
        -VirtualMachineScaleSet $vmss
    ```

    > [!NOTE]
    > Un groupe identique présente des mises à niveau de machines virtuelles qui appliquent des mises à jour à l’aide de trois paramètres différents : automatique, propagée et manuel. Dans ces instructions, la stratégie est définie sur automatique, de sorte que le groupe identique récupère les modifications immédiatement après le redémarrage.

4. Redémarrez le groupe identique :

    ```azurepowershell
    Start-AzVmss -ResourceGroupName "myResourceGroup" `
        -VMScaleSetName "myScaleSet"
    ```

Une fois que vous avez redémarré, attendez la fin des mises à niveau. Une fois les mises à niveau effectuées, la fonction virtuelle (VF) s’affiche à l’intérieur de la machine virtuelle. Vérifiez que vous utilisez un système d’exploitation et une taille de machine virtuelle pris en charge.

### <a name="resizing-existing-vms-with-accelerated-networking"></a>Redimensionnement des machines virtuelles existantes avec mise en réseau accélérée

Si la mise en réseau accélérée est activée pour une machine virtuelle, vous pouvez uniquement la redimensionner sur une machine virtuelle qui prend en charge la mise en réseau accélérée.  

Une machine virtuelle avec mise en réseau accélérée ne peut pas être redimensionnée en une instance de machine virtuelle qui ne prend pas en charge la mise en réseau accélérée à l’aide de l’opération de redimensionnement. Pour redimensionner l’une de ces machines virtuelles, vous devez effectuer ces opérations :

1. Arrêtez ou libérez la machine virtuelle. Pour un groupe à haute disponibilité ou un groupe identique, arrêtez ou libérez toutes les machines virtuelles dans le groupe à haute disponibilité ou le groupe identique.

2. Désactivez la mise en réseau accélérée sur la carte réseau de la machine virtuelle. Pour un groupe à haute disponibilité ou un groupe identique, désactivez la mise en réseau accélérée sur les cartes réseau de toutes les machines virtuelles dans le groupe à haute disponibilité ou le groupe identique.

3. Une fois la mise en réseau accélérée désactivée, déplacez la machine virtuelle, le groupe à haute disponibilité ou le groupe identique vers une nouvelle taille qui ne prend pas en charge la mise en réseau accélérée, puis redémarrez-les.
