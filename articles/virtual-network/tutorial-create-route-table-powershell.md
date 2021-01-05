---
title: Router le trafic réseau - Azure PowerShell | Microsoft Docs
description: Dans cet article, découvrez comment acheminer le trafic réseau avec une table de routage à l’aide de PowerShell.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
Customer intent: I want to route traffic from one subnet, to a different subnet, through a network virtual appliance.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: kumud
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 41c0f9546b07158a2c7fb3344698598f0c38521f
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492078"
---
# <a name="route-network-traffic-with-a-route-table-using-powershell"></a>Acheminer le trafic réseau avec une table de routage à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Par défaut, Azure achemine automatiquement le trafic entre tous les sous-réseaux au sein d’un réseau virtuel. Vous pouvez créer vos propres itinéraires pour remplacer le routage par défaut d’Azure. La possibilité de créer des itinéraires personnalisés est utile si, par exemple, vous souhaitez router le trafic entre des sous-réseaux via une appliance virtuelle réseau (NVA). Dans cet article, vous apprendrez comment :

* Créer une table de routage
* Créer un itinéraire
* Créer un réseau virtuel comprenant plusieurs sous-réseaux
* Associer une table de routage à un sous-réseau
* Créer une appliance NVA qui route le trafic
* Déployer des machines virtuelles sur différents sous-réseaux
* Router le trafic d’un sous-réseau vers un autre via une NVA

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell en local, vous devez exécuter le module Azure PowerShell version 1.0.0 ou ultérieure pour les besoins de cet article. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="create-a-route-table"></a>Créer une table de routage

Avant de pouvoir créer une table de routage, créez un groupe de ressources avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* pour toutes les ressources créées dans cet article.

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

Créez une table de routage avec [New-AzRouteTable](/powershell/module/az.network/new-azroutetable). L’exemple suivant crée une table de routage nommée *myRouteTablePublic*.

```azurepowershell-interactive
$routeTablePublic = New-AzRouteTable `
  -Name 'myRouteTablePublic' `
  -ResourceGroupName myResourceGroup `
  -location EastUS
```

## <a name="create-a-route"></a>Créer un itinéraire

Créez un itinéraire en récupérant l’objet de table de routage avec [Get-AzRouteTable](/powershell/module/az.network/get-azroutetable), créez un itinéraire avec [Add-AzRouteConfig](/powershell/module/az.network/add-azrouteconfig), puis écrivez la configuration de l’itinéraire dans la table de routage avec [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable).

```azurepowershell-interactive
Get-AzRouteTable `
  -ResourceGroupName "myResourceGroup" `
  -Name "myRouteTablePublic" `
  | Add-AzRouteConfig `
  -Name "ToPrivateSubnet" `
  -AddressPrefix 10.0.1.0/24 `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress 10.0.2.4 `
 | Set-AzRouteTable
```

## <a name="associate-a-route-table-to-a-subnet"></a>Associer une table de routage à un sous-réseau

Avant de pouvoir associer une table de routage à un sous-réseau, vous devez créer un réseau virtuel et un sous-réseau. Créez un réseau virtuel avec [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). L’exemple suivant permet de créer un réseau virtuel nommé *myVirtualNetwork* avec le préfixe d’adresse *10.0.0.0/16*.

```azurepowershell-interactive
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

Créez trois sous-réseaux en créant trois configurations de sous-réseau avec [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). L’exemple suivant crée trois configurations de sous-réseau pour les sous-réseaux *Public*, *Private* et *DMZ* :

```azurepowershell-interactive
$subnetConfigPublic = Add-AzVirtualNetworkSubnetConfig `
  -Name Public `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork

$subnetConfigPrivate = Add-AzVirtualNetworkSubnetConfig `
  -Name Private `
  -AddressPrefix 10.0.1.0/24 `
  -VirtualNetwork $virtualNetwork

$subnetConfigDmz = Add-AzVirtualNetworkSubnetConfig `
  -Name DMZ `
  -AddressPrefix 10.0.2.0/24 `
  -VirtualNetwork $virtualNetwork
```

Écrivez les configurations de sous-réseaux dans le réseau virtuel à l’aide de la commande [Set-AzVirtualNetwork](/powershell/module/az.network/Set-azVirtualNetwork), ce qui crée les sous-réseaux dans le réseau virtuel :

```azurepowershell-interactive
$virtualNetwork | Set-AzVirtualNetwork
```

Associez la table de routage *myRouteTablePublic* au sous-réseau *Public* avec [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig), puis écrivez la configuration du sous-réseau dans le réseau virtuel avec [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork).

```azurepowershell-interactive
Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $virtualNetwork `
  -Name 'Public' `
  -AddressPrefix 10.0.0.0/24 `
  -RouteTable $myRouteTablePublic | `
Set-AzVirtualNetwork
```

## <a name="create-an-nva"></a>Créer une NVA

Une NVA est une machine virtuelle qui exécute une fonction réseau, telle que le routage, la fonction de pare-feu ou l’optimisation WAN.

Avant de créer une machine virtuelle, créez une interface réseau.

### <a name="create-a-network-interface"></a>Créer une interface réseau

Avant de créer une interface réseau, vous devez récupérer l’ID de réseau virtuel avec [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork), puis l’ID de sous-réseau avec [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig). Créez une interface réseau avec [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) dans le sous-réseau *DMZ* avec le transfert IP activé :

```azurepowershell-interactive
# Retrieve the virtual network object into a variable.
$virtualNetwork=Get-AzVirtualNetwork `
  -Name myVirtualNetwork `
  -ResourceGroupName myResourceGroup

# Retrieve the subnet configuration into a variable.
$subnetConfigDmz = Get-AzVirtualNetworkSubnetConfig `
  -Name DMZ `
  -VirtualNetwork $virtualNetwork

# Create the network interface.
$nic = New-AzNetworkInterface `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name 'myVmNva' `
  -SubnetId $subnetConfigDmz.Id `
  -EnableIPForwarding
```

### <a name="create-a-vm"></a>Créer une machine virtuelle

Pour créer une machine virtuelle et lui attacher une interface réseau existante, vous devez d’abord créer une configuration de machine virtuelle avec [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig). La configuration inclut l’interface réseau créée à l’étape précédente. Quand vous êtes invité à indiquer un nom d’utilisateur et un mot de passe, sélectionnez ceux qui vous permettront de vous connecter à la machine virtuelle.

```azurepowershell-interactive
# Create a credential object.
$cred = Get-Credential -Message "Enter a username and password for the VM."

# Create a VM configuration.
$vmConfig = New-AzVMConfig `
  -VMName 'myVmNva' `
  -VMSize Standard_DS2 | `
  Set-AzVMOperatingSystem -Windows `
    -ComputerName 'myVmNva' `
    -Credential $cred | `
  Set-AzVMSourceImage `
    -PublisherName MicrosoftWindowsServer `
    -Offer WindowsServer `
    -Skus 2016-Datacenter `
    -Version latest | `
  Add-AzVMNetworkInterface -Id $nic.Id
```

Créez la machine virtuelle avec la configuration de la machine virtuelle et [New-AzVM](/powershell/module/az.compute/new-azvm). L’exemple suivant crée une machine virtuelle nommée *myVmNva*.

```azurepowershell-interactive
$vmNva = New-AzVM `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -VM $vmConfig `
  -AsJob
```

L’option `-AsJob` crée la machine virtuelle en arrière-plan. Vous pouvez donc passer à l’étape suivante.

## <a name="create-virtual-machines"></a>Créer des machines virtuelles

Créez deux machines virtuelles dans le réseau virtuel pour vérifier que le trafic provenant du sous-réseau *Public* est acheminé vers le sous-réseau *Private* via l’appliance virtuelle réseau lors d’une étape ultérieure.

Créez une machine virtuelle dans le sous-réseau *Public* avec [New-AzVM](/powershell/module/az.compute/new-azvm). L’exemple suivant crée une machine virtuelle nommée *myVmPublic* dans le sous-réseau *Public* du réseau virtuel *myVirtualNetwork*.

```azurepowershell-interactive
New-AzVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "East US" `
  -VirtualNetworkName "myVirtualNetwork" `
  -SubnetName "Public" `
  -ImageName "Win2016Datacenter" `
  -Name "myVmPublic" `
  -AsJob
```

Créez une machine virtuelle dans le sous-réseau *Private*.

```azurepowershell-interactive
New-AzVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "East US" `
  -VirtualNetworkName "myVirtualNetwork" `
  -SubnetName "Private" `
  -ImageName "Win2016Datacenter" `
  -Name "myVmPrivate"
```

La création de la machine virtuelle ne nécessite que quelques minutes. Attendez que la machine virtuelle ait été créée et qu’Azure ait retourné la sortie à PowerShell pour passer à l’étape suivante.

## <a name="route-traffic-through-an-nva"></a>Router le trafic via une NVA

Utilisez [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) pour retourner l’adresse IP publique de la machine virtuelle *myVmPrivate*. L’exemple suivant retourne l’adresse IP publique de la machine virtuelle *myVmPrivate* :

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVmPrivate `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

Utilisez la commande suivante pour créer une session Bureau à distance avec la machine virtuelle *myVmPrivate* à partir de votre ordinateur local. Remplacez `<publicIpAddress>` par l’adresse IP retournée par la commande précédente.

```
mstsc /v:<publicIpAddress>
```

Ouvrez le fichier .rdp téléchargé. Si vous y êtes invité, sélectionnez **Connexion**.

Entrez le nom d’utilisateur et le mot de passe spécifiés lors de la création de la machine virtuelle (il se peut que vous deviez choisir **Plus de choix**, puis **Utiliser un compte différent** pour spécifier les informations d’identification que vous avez entrées lors de la création de la machine virtuelle), puis sélectionnez **OK**. Un avertissement de certificat peut s’afficher pendant le processus de connexion. Sélectionnez **Oui** pour poursuivre le processus de connexion.

À une étape ultérieure, la commande `tracert.exe` est utilisée pour tester le routage. Tracert utilise le protocole ICMP (Internet Control Message Protocol), qui est refusé via le Pare-feu Windows. Autorisez le protocole ICMP dans le pare-feu Windows en entrant la commande suivante de PowerShell sur la machine virtuelle *myVmPrivate* :

```powershell
New-NetFirewallRule -DisplayName "Allow ICMPv4-In" -Protocol ICMPv4
```

Bien que tracert soit utilisé pour tester le routage dans cet article, il n’est pas recommandé d’autoriser le protocole IMCP dans le pare-feu Windows lors de déploiements en production.

Vous avez activé le transfert d’IP dans Azure pour l’interface réseau de la machine virtuelle dans Activer le transfert IP. Sur la machine virtuelle, le système d’exploitation ou une application exécutée dans la machine virtuelle, doit également pouvoir transférer le trafic réseau. Activez le transfert IP au sein du système d’exploitation de la machine virtuelle *myVmNva*.

À partir d’une invite de commandes sur la machine virtuelle *myVmPrivate*, connectez le Bureau à distance à la machine virtuelle *myVmNva* :

``` 
mstsc /v:myvmnva
```

Pour activer le transfert d’adresse IP au sein du système d’exploitation, entrez la commande suivante dans PowerShell à partir de la machine virtuelle *myVmNva* :

```powershell
Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters -Name IpEnableRouter -Value 1
```

Redémarrez la machine virtuelle *myVmNva*, ce qui va également déconnecter la session Bureau à distance.

Tout en conservant la connexion à la machine virtuelle *myVmPrivate*, créez une session Bureau à distance sur la machine virtuelle *myVmPublic*, une fois la machine virtuelle *myVmNva* redémarrée :

``` 
mstsc /v:myVmPublic
```

Autorisez le protocole ICMP dans le pare-feu Windows en entrant la commande suivante de PowerShell sur la machine virtuelle *myVmPublic* :

```powershell
New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
```

Pour tester le routage du trafic réseau vers la machine virtuelle *myVmPrivate* à partir de la machine virtuelle *myVmPublic*, entrez la commande suivante de PowerShell sur la machine virtuelle *myVmPublic* :

```
tracert myVmPrivate
```

La réponse ressemble à ce qui suit :

```
Tracing route to myVmPrivate.vpgub4nqnocezhjgurw44dnxrc.bx.internal.cloudapp.net [10.0.1.4]
over a maximum of 30 hops:

1    <1 ms     *        1 ms  10.0.2.4
2     1 ms     1 ms     1 ms  10.0.1.4

Trace complete.
```

Vous voyez que le premier tronçon est 10.0.2.4, ce qui correspond à l’adresse IP privée de l’appliance virtuelle réseau (NVA). Le second tronçon est 10.0.1.4, ce qui correspond à l’adresse IP privée de la machine virtuelle *myVmPrivate*. L’itinéraire ajouté à la table de routage *myRouteTablePublic* et associé au sous-réseau *Public* a contraint Azure à acheminer le trafic via l’appliance virtuelle réseau et non directement au sous-réseau *Private*.

Fermez la session Bureau à distance sur la machine virtuelle *myVmPublic*. Cela n’interrompt pas la connexion à la machine virtuelle *myVmPrivate*.

Pour tester le routage du trafic réseau vers la machine virtuelle *myVmPublic* à partir de la machine virtuelle *myVmPrivate*, entrez la commande suivante à l’invite de commandes sur la machine virtuelle *myVmPrivate* :

```
tracert myVmPublic
```

La réponse ressemble à ce qui suit :

```
Tracing route to myVmPublic.vpgub4nqnocezhjgurw44dnxrc.bx.internal.cloudapp.net [10.0.0.4]
over a maximum of 30 hops:

1     1 ms     1 ms     1 ms  10.0.0.4

Trace complete.
```

Vous pouvez voir que le trafic est routé directement de la machine virtuelle *myVmPrivate* vers la machine virtuelle *myVmPublic*. Par défaut, Azure achemine directement le trafic entre les sous-réseaux.

Fermez les sessions Bureau à distance sur la machine virtuelle *myVmPrivate*.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’avez plus besoin d’un groupe de ressources, utilisez [Remove-AzResourcegroup](/powershell/module/az.resources/remove-azresourcegroup) pour le supprimer, ainsi que toutes les ressources qu’il contient.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez créé une table de routage que vous avez associée à un sous-réseau. Vous avez créé une appliance virtuelle réseau qui a acheminé le trafic d’un sous-réseau public vers un sous-réseau privé. Vous pouvez déployer différentes appliances virtuelles réseau préconfigurées ayant des fonctions de réseau, telles que la fonction de pare-feu ou l’optimisation WAN, à partir de la [Place de marché Microsoft Azure](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking). Pour en savoir plus sur le routage, consultez [Routage du trafic de réseau virtuel](virtual-networks-udr-overview.md) et [Créer, modifier ou supprimer une table de routage](manage-route-table.md).

Alors que vous pouvez déployer de nombreuses ressources Azure dans un réseau virtuel, les ressources pour certains services Azure PaaS ne peuvent pas être déployées dans un réseau virtuel. Cependant, vous pouvez toujours restreindre l’accès aux ressources de certains services Azure PaaS au trafic provenant uniquement d’un sous-réseau de réseau virtuel. Pour connaître la marche à suivre, consultez [Restreindre l’accès réseau aux ressources PaaS](tutorial-restrict-network-access-to-resources-powershell.md).
