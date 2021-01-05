---
title: Créer et gérer dans Azure des machines virtuelles Windows utilisant plusieurs cartes d’interface réseau
description: Découvrez comment créer et gérer une machine virtuelle Windows équipée de plusieurs cartes d’interface réseau à l’aide d’Azure PowerShell ou de modèles Resource Manager.
author: cynthn
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure
ms.date: 09/26/2017
ms.author: cynthn
ms.openlocfilehash: 66a135cd1629aa2befcd4c56d835473791d62ce8
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91974003"
---
# <a name="create-and-manage-a-windows-virtual-machine-that-has-multiple-nics"></a>Créer et gérer une machine virtuelle Windows équipée de plusieurs cartes d’interface réseau
Les machines virtuelles (VM) dans Azure peuvent être équipées de plusieurs cartes d’interface réseau (NIC) virtuelles. Un scénario courant consiste à disposer de sous-réseaux différents pour les connectivités frontale et principale. Vous pouvez associer plusieurs cartes d’interface réseau d’une machine virtuelle à différents sous-réseaux, mais ces sous-réseaux doivent tous résider dans le même réseau virtuel. Cet article explique comment créer une machine virtuelle équipée de plusieurs cartes d’interface réseau. Il explique également comment ajouter ou supprimer des cartes d’interface réseau d’une machine virtuelle existante. Comme le nombre de cartes réseau prises en charge varie suivant la [taille des machines virtuelles](../sizes.md) , pensez à dimensionner la vôtre en conséquence.

## <a name="prerequisites"></a>Prérequis

Dans les exemples suivants, remplacez les exemples de noms de paramètre par vos propres valeurs. Les noms de paramètre sont par exemple *myResourceGroup*, *myVnet* et *myVM*.

 

## <a name="create-a-vm-with-multiple-nics"></a>Créer une machine virtuelle avec plusieurs cartes d’interface réseau
Créez d’abord un groupe de ressources. L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *EastUs*:

```powershell
New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

### <a name="create-virtual-network-and-subnets"></a>Créer un réseau virtuel et des sous-réseaux
Un scénario courant pour un réseau virtuel consiste à avoir deux sous-réseaux ou plus. Un sous-réseau peut être dédié au trafic frontal, et l’autre au trafic principal. Pour connecter les deux sous-réseaux, vous utilisez ensuite plusieurs cartes d’interface réseau sur votre machine virtuelle.

1. Définissez deux sous-réseaux de réseau virtuel avec la commande [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). L’exemple suivant définit les sous-réseaux pour *mySubnetFrontEnd* et *mySubnetBackEnd* :

    ```powershell
    $mySubnetFrontEnd = New-AzVirtualNetworkSubnetConfig -Name "mySubnetFrontEnd" `
        -AddressPrefix "192.168.1.0/24"
    $mySubnetBackEnd = New-AzVirtualNetworkSubnetConfig -Name "mySubnetBackEnd" `
        -AddressPrefix "192.168.2.0/24"
    ```

2. Créez votre réseau virtuel et vos sous-réseaux avec la commande [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). L’exemple suivant crée un réseau virtuel nommé *myVnet* :

    ```powershell
    $myVnet = New-AzVirtualNetwork -ResourceGroupName "myResourceGroup" `
        -Location "EastUs" `
        -Name "myVnet" `
        -AddressPrefix "192.168.0.0/16" `
        -Subnet $mySubnetFrontEnd,$mySubnetBackEnd
    ```


### <a name="create-multiple-nics"></a>Créer plusieurs cartes réseau
Créez deux cartes d’interface réseau avec la commande [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface). Associez une carte d’interface réseau au sous-réseau frontal et l’autre au sous-réseau principal. L’exemple suivant crée des cartes d’interface réseau nommées *myNic1* et *myNic2* :

```powershell
$frontEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetFrontEnd'}
$myNic1 = New-AzNetworkInterface -ResourceGroupName "myResourceGroup" `
    -Name "myNic1" `
    -Location "EastUs" `
    -SubnetId $frontEnd.Id

$backEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetBackEnd'}
$myNic2 = New-AzNetworkInterface -ResourceGroupName "myResourceGroup" `
    -Name "myNic2" `
    -Location "EastUs" `
    -SubnetId $backEnd.Id
```

En général, vous créez également un [groupe de sécurité réseau](../../virtual-network/network-security-groups-overview.md) pour filtrer le trafic vers la machine virtuelle, ainsi qu’un [équilibreur de charge](../../load-balancer/load-balancer-overview.md) pour répartir le trafic entre plusieurs machines virtuelles.

### <a name="create-the-virtual-machine"></a>Créer la machine virtuelle
Maintenant, commencez à élaborer la configuration de votre machine virtuelle. La taille d’une machine virtuelle détermine le nombre maximal de cartes réseau qu’elle peut accueillir. Pour plus d’informations, voir [Tailles des machines virtuelles Windows](../sizes.md).

1. Tout d’abord, définissez les informations d’identification de votre machine virtuelle sur la variable `$cred` comme suit :

    ```powershell
    $cred = Get-Credential
    ```

2. Définissez votre machine virtuelle avec la commande [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig). L’exemple suivant définit une machine virtuelle nommée *myVM* et utilise une taille de machine virtuelle qui prend en charge plus de deux cartes d’interface réseau (*Standard_DS3_v2*) :

    ```powershell
    $vmConfig = New-AzVMConfig -VMName "myVM" -VMSize "Standard_DS3_v2"
    ```

3. Créez le reste de la configuration de votre machine virtuelle avec les commandes [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem) et [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage). L’exemple suivant crée une machine virtuelle Windows Server 2016 R2 :

    ```powershell
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

4. Associez les deux cartes d’interface réseau que vous avez créées précédemment à l’aide de la commande [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface) :

    ```powershell
    $vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $myNic1.Id -Primary
    $vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $myNic2.Id
    ```

5. Créez votre machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm) :

    ```powershell
    New-AzVM -VM $vmConfig -ResourceGroupName "myResourceGroup" -Location "EastUs"
    ```

6. Ajoutez des itinéraires pour les cartes réseau secondaires au système d’exploitation en suivant les étapes décrites dans [Configurer le système d’exploitation pour plusieurs cartes réseau](#configure-guest-os-for-multiple-nics).

## <a name="add-a-nic-to-an-existing-vm"></a>Ajouter une carte réseau à une machine virtuelle existante
Pour ajouter une carte d’interface réseau virtuelle à une machine virtuelle existante, désallouez la machine virtuelle, ajoutez la carte d’interface réseau virtuelle, puis démarrez la machine virtuelle. Comme le nombre de cartes réseau prises en charge varie suivant la [taille des machines virtuelles](../sizes.md) , pensez à dimensionner la vôtre en conséquence. Si nécessaire, vous pouvez [redimensionner une machine virtuelle](resize-vm.md).

1. Libérez la machine virtuelle avec la commande [Stop-AzVM](/powershell/module/az.compute/stop-azvm). L’exemple suivant désalloue la machine virtuelle nommée *myVM* dans *myResourceGroup*:

    ```powershell
    Stop-AzVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

2. Obtenez la configuration existante de la machine virtuelle avec commande [Get-AzVm](/powershell/module/az.compute/get-azvm). L’exemple suivant récupère des informations sur la machine virtuelle nommée *myVM* dans *myResourceGroup* :

    ```powershell
    $vm = Get-AzVm -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

3. L’exemple suivant crée une carte d’interface réseau virtuelle avec la commande [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface), nommée *myNic3*, et associée à *mySubnetBackEnd*. La carte d’interface réseau virtuelle est ensuite associée à la machine virtuelle nommée *myVM* dans *myResourceGroup* avec la commande [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface) :

    ```powershell
    # Get info for the back end subnet
    $myVnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup"
    $backEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetBackEnd'}

    # Create a virtual NIC
    $myNic3 = New-AzNetworkInterface -ResourceGroupName "myResourceGroup" `
        -Name "myNic3" `
        -Location "EastUs" `
        -SubnetId $backEnd.Id

    # Get the ID of the new virtual NIC and add to VM
    $nicId = (Get-AzNetworkInterface -ResourceGroupName "myResourceGroup" -Name "MyNic3").Id
    Add-AzVMNetworkInterface -VM $vm -Id $nicId | Update-AzVm -ResourceGroupName "myResourceGroup"
    ```

    ### <a name="primary-virtual-nics"></a>Cartes d’interface réseau virtuelles principales
    L’une des cartes d’interface réseau sur une machine virtuelle dotées de plusieurs cartes doit être la carte principale. Si l’une des cartes d’interface réseau virtuelles existantes sur la machine virtuelle est déjà définie comme principale, vous pouvez ignorer cette étape. L’exemple suivant suppose que deux cartes d’interface réseau virtuelles sont désormais présentes sur une machine virtuelle, et que vous souhaitez ajouter la première carte d’interface réseau (`[0]`) en tant que carte principale :
        
    ```powershell
    # List existing NICs on the VM and find which one is primary
    $vm.NetworkProfile.NetworkInterfaces
    
    # Set NIC 0 to be primary
    $vm.NetworkProfile.NetworkInterfaces[0].Primary = $true
    $vm.NetworkProfile.NetworkInterfaces[1].Primary = $false
    
    # Update the VM state in Azure
    Update-AzVM -VM $vm -ResourceGroupName "myResourceGroup"
    ```

4. Démarrez la machine virtuelle avec la commande [Start-AzVm](/powershell/module/az.compute/start-azvm) :

    ```powershell
    Start-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"
    ```

5. Ajoutez des itinéraires pour les cartes réseau secondaires au système d’exploitation en suivant les étapes décrites dans [Configurer le système d’exploitation pour plusieurs cartes réseau](#configure-guest-os-for-multiple-nics).

## <a name="remove-a-nic-from-an-existing-vm"></a>Supprimer une carte réseau d’une machine virtuelle existante
Pour supprimer une carte d’interface réseau virtuelle à partir d’une machine virtuelle existante, désallouez la machine virtuelle, supprimez la carte d’interface réseau virtuelle, puis démarrez la machine virtuelle.

1. Libérez la machine virtuelle avec la commande [Stop-AzVM](/powershell/module/az.compute/stop-azvm). L’exemple suivant désalloue la machine virtuelle nommée *myVM* dans *myResourceGroup*:

    ```powershell
    Stop-AzVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

2. Obtenez la configuration existante de la machine virtuelle avec commande [Get-AzVm](/powershell/module/az.compute/get-azvm). L’exemple suivant récupère des informations sur la machine virtuelle nommée *myVM* dans *myResourceGroup* :

    ```powershell
    $vm = Get-AzVm -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

3. Obtenez des informations sur la suppression de la carte d’interface réseau avec la commande [Get-AzNetworkInterface](/powershell/module/az.network/get-aznetworkinterface). L’exemple suivant obtient des informations sur la carte *myNic3* :

    ```powershell
    # List existing NICs on the VM if you need to determine NIC name
    $vm.NetworkProfile.NetworkInterfaces

    $nicId = (Get-AzNetworkInterface -ResourceGroupName "myResourceGroup" -Name "myNic3").Id   
    ```

4. Supprimez la carte d’interface réseau avec la commande [Remove-AzVMNetworkInterface](/powershell/module/az.compute/remove-azvmnetworkinterface), puis mettez à jour la machine virtuelle avec la commande [Update-AzVm](/powershell/module/az.compute/update-azvm). L’exemple suivant supprime la carte *myNic3* obtenue par `$nicId` à l’étape précédente :

    ```powershell
    Remove-AzVMNetworkInterface -VM $vm -NetworkInterfaceIDs $nicId | `
        Update-AzVm -ResourceGroupName "myResourceGroup"
    ```   

5. Démarrez la machine virtuelle avec la commande [Start-AzVm](/powershell/module/az.compute/start-azvm) :

    ```powershell
    Start-AzVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```   

## <a name="create-multiple-nics-with-templates"></a>Créer plusieurs cartes d’interface réseau avec des modèles
Grâce aux modèles Azure Resource Manager, vous pouvez créer plusieurs instances d’une ressource pendant le déploiement, à l’image de la création de plusieurs cartes d’interface réseau. Les modèles Resource Manager utilisent des fichiers JSON déclaratifs pour définir votre environnement. Pour plus d’informations, voir [Présentation d’Azure Resource Manager](../../azure-resource-manager/management/overview.md). Vous pouvez utiliser la commande *copy* pour spécifier le nombre d’instances à  :

```json
"copy": {
    "name": "multiplenics",
    "count": "[parameters('count')]"
}
```

Pour plus d’informations, voir [Création de plusieurs instances à l’aide de la commande *copie*](../../azure-resource-manager/templates/copy-resources.md). 

Vous pouvez également utiliser la commande `copyIndex()` pour ajouter un numéro à un nom de ressource. Vous pouvez ensuite créer les cartes *myNic1*, *MyNic2* et ainsi de suite. Voici un exemple de code pour l’ajout de la valeur d’index :

```json
"name": "[concat('myNic', copyIndex())]", 
```

Vous pouvez consulter un exemple complet de la [création de plusieurs cartes d’interface réseau à l’aide de modèles Resource Manager](../../virtual-network/template-samples.md).

Ajoutez des itinéraires pour les cartes réseau secondaires au système d’exploitation en suivant les étapes décrites dans [Configurer le système d’exploitation pour plusieurs cartes réseau](#configure-guest-os-for-multiple-nics).

## <a name="configure-guest-os-for-multiple-nics"></a>Configurer plusieurs cartes réseau dans un système d’exploitation invité

Azure affecte une passerelle par défaut aux premières (principales) interfaces réseau associées à la machine virtuelle. Azure n’affecte pas de passerelle par défaut aux interfaces réseau additionnelles (secondaires) associées à la machine virtuelle. Par conséquent, vous ne pouvez pas communiquer avec les ressources hors du sous-réseau dans lequel se trouve, par défaut, une interface réseau secondaire. Toutefois, les interfaces réseau secondaires peuvent communiquer avec les ressources hors de leur sous-réseau. Ceci dit, les étapes à suivre pour permettre cette communication sont différentes d’un système d’exploitation à un autre.

1. Dans une invite de commande Windows, exécutez la commande `route print`, qui retourne une sortie similaire à la sortie suivante d’une machine virtuelle à deux interfaces réseau :

    ```
    ===========================================================================
    Interface List
    3...00 0d 3a 10 92 ce ......Microsoft Hyper-V Network Adapter #3
    7...00 0d 3a 10 9b 2a ......Microsoft Hyper-V Network Adapter #4
    ===========================================================================
    ```
 
    Dans cet exemple, **Microsoft Hyper-V Network Adapter #4** (interface 7) est l’interface réseau secondaire à laquelle aucune passerelle par défaut n’est assignée.

2. Depuis une invite de commande, exécutez la commande `ipconfig` pour voir quelle adresse IP est assignée à l’interface réseau secondaire. Dans cet exemple, l’adresse 192.168.2.4 est assignée à l’interface 7. Aucune adresse de passerelle par défaut n’est retournée pour l’interface réseau secondaire.

3. Pour acheminer le trafic destiné aux adresses hors du sous-réseau de l’interface réseau secondaire vers la passerelle du sous-réseau, exécutez la commande suivante :

    ```
    route add -p 0.0.0.0 MASK 0.0.0.0 192.168.2.1 METRIC 5015 IF 7
    ```

    L’adresse de la passerelle du sous-réseau est la première adresse IP (celle qui finit par .1) dans la plage d’adresse définie pour le sous-réseau. Si vous ne voulez pas acheminer le trafic hors du sous-réseau, vous avez la possibilité d’ajouter des itinéraires individuels vers des destinations spécifiques. Par exemple, si vous voulez acheminer le trafic de l’interface réseau secondaire vers le réseau 192.168.3.0 uniquement, entrez la commande :

      ```
      route add -p 192.168.3.0 MASK 255.255.255.0 192.168.2.1 METRIC 5015 IF 7
      ```
  
4. Pour vérifier si la communication fonctionne avec une ressource du réseau 192.168.3.0, par exemple, entrez la commande suivante pour faire un test Ping de l’adresse 192.168.3.4 à l’aide de l’interface 7 (192.168.2.4) :

    ```
    ping 192.168.3.4 -S 192.168.2.4
    ```

    Vous devrez peut-être ouvrir le protocole ICMP via le pare-feu Windows de l’appareil sur lequel vous effectuez le test Ping. Exécutez cette commande :
  
      ```
      netsh advfirewall firewall add rule name=Allow-ping protocol=icmpv4 dir=in action=allow
      ```
  
5. Pour confirmer que l’itinéraire ajouté se trouve dans la table de routage, entrez la commande `route print`, qui retourne une sortie similaire à ce qui suit :

    ```
    ===========================================================================
    Active Routes:
    Network Destination        Netmask          Gateway       Interface  Metric
              0.0.0.0          0.0.0.0      192.168.1.1      192.168.1.4     15
              0.0.0.0          0.0.0.0      192.168.2.1      192.168.2.4   5015
    ```

    L’itinéraire répertorié avec l’adresse *192.168.1.1* sous **Passerelle** représente l’itinéraire se trouvant là par défaut pour l’interface réseau principale. L’itinéraire avec l’adresse *192.168.2.1* sous **Passerelle** représente l’itinéraire que vous avez ajouté.

## <a name="next-steps"></a>Étapes suivantes
Réviser les [Tailles des machines virtuelles Windows](../sizes.md) lorsque vous tentez de créer une machine virtuelle équipée de plusieurs cartes d’interface réseau. Faites attention au nombre maximal de cartes d’interface réseau pris en charge par chaque taille de machine virtuelle.