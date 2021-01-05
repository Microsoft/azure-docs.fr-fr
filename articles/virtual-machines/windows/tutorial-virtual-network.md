---
title: 'Tutoriel : Créer et gérer des réseaux virtuels Azure pour des machines virtuelles Windows'
description: Dans ce tutoriel, vous allez apprendre à utiliser Azure PowerShell pour créer et gérer des réseaux virtuels Azure pour des machines virtuelles Windows Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: networking
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 08/04/2020
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 84da38fe71446c54f17c4d4329c7294c5e5176d2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87800192"
---
# <a name="tutorial-create-and-manage-azure-virtual-networks-for-windows-virtual-machines-with-azure-powershell"></a>Tutoriel : Créer et gérer des réseaux virtuels Azure pour des machines virtuelles Windows avec Azure PowerShell

Les machines virtuelles Azure utilisent la gestion réseau Azure pour la communication réseau interne et externe. Ce didacticiel vous guide dans le déploiement de deux machines virtuelles et la configuration de la gestion réseau Azure pour celles-ci. Les exemples de ce didacticiel supposent que les machines virtuelles hébergent une application web avec un back-end de base de données. Le didacticiel ne comprend cependant pas le déploiement d’une application. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un réseau virtuel et un sous-réseau
> * Créer une adresse IP publique
> * Créer une machine virtuelle frontale
> * sécurisent le trafic réseau
> * Créer une machine virtuelle principale


## <a name="vm-networking-overview"></a>Vue d’ensemble de la mise en réseau de machines virtuelles

Avec les réseaux virtuels Azure, les connexions réseau sont sécurisées entre les machines virtuelles, Internet et d’autres services Azure comme Azure SQL Database. Les réseaux virtuels sont divisés en segments logiques, appelés sous-réseaux. Les sous-réseaux sont utilisés pour contrôler le flux du réseau et comme une limite de sécurité. Quand vous déployez une machine virtuelle, elle inclut généralement une interface de réseau virtuel, qui est attachée à un sous-réseau.

En suivant ce didacticiel, vous pourrez créer les ressources suivantes :

![Réseau virtuel avec deux sous-réseaux](./media/tutorial-virtual-network/networktutorial.png)

- *myVNet* : réseau virtuel que les machines virtuelles utilisent pour communiquer entre elles et avec Internet.
- *myFrontendSubnet* : sous-réseau dans *myVNet* utilisé par les ressources frontales.
- *myPublicIPAddress* : adresse IP publique utilisée pour accéder à *myFrontendVM* à partir d’Internet.
- *myFrontendNic* : interface réseau utilisée par *myFrontendVM* pour communiquer avec *myBackendVM*.
- *myFrontendVM :* : machine virtuelle utilisée pour les communications entre Internet et *myBackendVM*.
- *myBackendNSG* : groupe de sécurité réseau qui contrôle la communication entre *myFrontendVM* et *myBackendVM*.
- *myBackendSubnet* : sous-réseau associé à *myBackendNSG* et utilisé par les ressources du serveur principal.
- *myBackendNic* : interface réseau utilisée par *myBackendVM* pour communiquer avec *myFrontendVM*.
- *myBackendVM* : machine virtuelle qui utilise le port 1433 pour communiquer avec *myFrontendVM*.


## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.


## <a name="create-subnet"></a>Créer un sous-réseau 

Pour ce didacticiel, un seul réseau virtuel est créé avec deux sous-réseaux. Un sous-réseau frontal pour l’hébergement d’une application web et un sous-réseau principal pour l’hébergement d’un serveur de base de données.

Avant de créer une machine virtuelle, créez un groupe de ressources à l’aide de [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). L’exemple suivant crée un groupe de ressources nommé *myRGNetwork* à l’emplacement *EastUS* :

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName myRGNetwork -Location EastUS
```

Créez une configuration de sous-réseau nommée *myFrontendSubnet* à l’aide de [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) :

```azurepowershell-interactive
$frontendSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name myFrontendSubnet `
  -AddressPrefix 10.0.0.0/24
```

Ainsi qu’une autre configuration de sous-réseau nommée *myBackendSubnet* :

```azurepowershell-interactive
$backendSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name myBackendSubnet `
  -AddressPrefix 10.0.1.0/24
```

## <a name="create-virtual-network"></a>Création d’un réseau virtuel

Créez un réseau virtuel nommé *myVNet* en utilisant *myFrontendSubnet* et *myBackendSubnet* avec [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) :

```azurepowershell-interactive
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myVNet `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $frontendSubnet, $backendSubnet
```

À ce stade, un réseau a été créé et segmenté en deux sous-réseaux, un pour les services frontaux et un autre pour les services principaux. Dans la section suivante, des machines virtuelles sont créées et connectées à ces sous-réseaux.

## <a name="create-a-public-ip-address"></a>Créer une adresse IP publique

Une adresse IP publique permet aux ressources Azure d’être accessibles sur Internet. La méthode d’allocation de l’adresse IP publique peut être configurée comme dynamique ou statique. Par défaut, une adresse IP publique est allouée dynamiquement. Les adresses IP dynamiques sont libérées quand une machine virtuelle est désallouée. Ce comportement fait que l’adresse IP change lors de toute opération qui inclut une désallocation de la machine virtuelle.

La méthode d’allocation peut être définie comme étant statique, ce qui garantit que l’adresse IP reste affectée à une machine virtuelle, même lorsqu’elle est libérée. Si vous utilisez une adresse IP statique, l’adresse IP elle-même ne peut pas être spécifiée. Elle est plutôt allouée à partir d’un pool d’adresses disponibles.

Créez une adresse IP publique nommée *myPublicIPAddress* avec [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) :

```azurepowershell-interactive
$pip = New-AzPublicIpAddress `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -AllocationMethod Dynamic `
  -Name myPublicIPAddress
```

Vous pouvez changer le paramètre -AllocationMethod en `Static` pour assigner une adresse IP publique statique.

## <a name="create-a-front-end-vm"></a>Créer une machine virtuelle frontale

Pour communiquer avec un réseau virtuel, une machine virtuelle a besoin d’une interface réseau virtuelle. Créez une carte réseau à l’aide de [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) :

```azurepowershell-interactive
$frontendNic = New-AzNetworkInterface `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myFrontend `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id
```

Définissez le nom d’utilisateur et le mot de passe pour le compte Administrateur sur la machine virtuelle avec [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1). Vous utilisez ces informations d’identification pour vous connecter à la machine virtuelle en suivant des étapes supplémentaires :

```azurepowershell-interactive
$cred = Get-Credential
```

Créez les machines virtuelles avec [New-AzVM](/powershell/module/az.compute/new-azvm).

```azurepowershell-interactive
New-AzVM `
   -Credential $cred `
   -Name myFrontend `
   -PublicIpAddressName myPublicIPAddress `
   -ResourceGroupName myRGNetwork `
   -Location "EastUS" `
   -Size Standard_D1 `
   -SubnetName myFrontendSubnet `
   -VirtualNetworkName myVNet
```

## <a name="secure-network-traffic"></a>sécurisent le trafic réseau

Un groupe de sécurité réseau (NSG) contient une liste de règles de sécurité qui autorisent ou rejettent le trafic réseau vers les ressources connectées aux réseaux virtuels Azure (VNet). Les groupes de sécurité réseau peuvent être associés à des sous-réseaux ou à des interfaces réseau individuelles. Un groupe de sécurité réseau associé à une interface réseau s’applique seulement à la machine virtuelle associée. Lorsqu’un NSG est associé à un sous-réseau, les règles s’appliquent à toutes les ressources connectées au sous-réseau.

### <a name="network-security-group-rules"></a>Règles de groupe de sécurité réseau

Les règles de groupe de sécurité réseau définissent les ports réseau sur lesquels le trafic est autorisé ou refusé. Les règles peuvent comprendre des plages d’adresses IP sources et de destination, de façon à ce que le trafic soit contrôlé entre des systèmes ou des sous-réseaux spécifiques. Les règles de groupe de sécurité réseau ont également une priorité (entre 1 et 4 096). Les règles sont évaluées dans l’ordre des priorités. Une règle avec une priorité de 100 est évaluée avant une règle avec une priorité de 200.

Tous les groupes de ressources réseau contiennent un ensemble de règles par défaut. Les règles par défaut ne peuvent pas être supprimées, mais comme la priorité la plus basse leur est attribuée, elles peuvent être remplacées par les règles que vous créez.

- **Réseau virtuel** : le trafic en provenance et à destination d’un réseau virtuel est autorisé à la fois dans les directions entrante et sortante.
- **Internet** : le trafic sortant est autorisé, mais le trafic entrant est bloqué.
- **Équilibreur de charge** : autoriser l’équilibreur de charge d’Azure à tester l’intégrité de vos machines virtuelles et des instances de rôle. Si vous n’utilisez pas un groupe à charge équilibrée, vous pouvez remplacer cette règle.

### <a name="create-network-security-groups"></a>Créer des groupes de sécurité réseau

Créez une règle de trafic entrant nommée *myFrontendNSGRule* pour autoriser le trafic web entrant sur *myFrontendVM* avec [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) :

```azurepowershell-interactive
$nsgFrontendRule = New-AzNetworkSecurityRuleConfig `
  -Name myFrontendNSGRule `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 200 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access Allow
```

Vous pouvez limiter le trafic interne vers *myBackendVM* en provenance uniquement de *myFrontendVM* en créant un groupe de sécurité réseau pour le sous-réseau back-end. L’exemple suivant crée une règle de groupe de sécurité réseau nommée *myBackendNSGRule* :

```azurepowershell-interactive
$nsgBackendRule = New-AzNetworkSecurityRuleConfig `
  -Name myBackendNSGRule `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 100 `
  -SourceAddressPrefix 10.0.0.0/24 `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 1433 `
  -Access Allow
```

Ajoutez un groupe de sécurité réseau nommé *myFrontendNSG* avec [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) :

```azurepowershell-interactive
$nsgFrontend = New-AzNetworkSecurityGroup `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myFrontendNSG `
  -SecurityRules $nsgFrontendRule
```

À présent, ajoutez un groupe de sécurité réseau nommé *myBackendNSG* avec New-AzNetworkSecurityGroup :

```azurepowershell-interactive
$nsgBackend = New-AzNetworkSecurityGroup `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myBackendNSG `
  -SecurityRules $nsgBackendRule
```

Ajoutez les groupes de sécurité réseau aux sous-réseaux :

```azurepowershell-interactive
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName myRGNetwork `
  -Name myVNet
$frontendSubnet = $vnet.Subnets[0]
$backendSubnet = $vnet.Subnets[1]
$frontendSubnetConfig = Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name myFrontendSubnet `
  -AddressPrefix $frontendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgFrontend
$backendSubnetConfig = Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name myBackendSubnet `
  -AddressPrefix $backendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgBackend
Set-AzVirtualNetwork -VirtualNetwork $vnet
```

## <a name="create-a-back-end-vm"></a>Créer une machine virtuelle principale

Pour créer une machine virtuelle principale dans ce didacticiel, le plus simple consiste à utiliser une image SQL Server. Ce didacticiel crée la machine virtuelle avec le serveur de base de données, mais il ne fournit pas d’informations sur l’accès à la base de données.

Créez *myBackendNic* :

```azurepowershell-interactive
$backendNic = New-AzNetworkInterface `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myBackend `
  -SubnetId $vnet.Subnets[1].Id
```

Définissez le nom d’utilisateur et le mot de passe pour le compte Administrateur sur la machine virtuelle avec Get-Credential :

```azurepowershell-interactive
$cred = Get-Credential
```

Créez *myBackendVM*.

```azurepowershell-interactive
New-AzVM `
   -Credential $cred `
   -Name myBackend `
   -ImageName "MicrosoftSQLServer:SQL2016SP1-WS2016:Enterprise:latest" `
   -ResourceGroupName myRGNetwork `
   -Location "EastUS" `
   -SubnetName MyBackendSubnet `
   -VirtualNetworkName myVNet
```

SQL Server est installé dans l’image utilisée dans cet exemple, mais n’est pas utilisé dans ce tutoriel. L’image est présentée pour vous montrer comment configurer une machine virtuelle afin de gérer le trafic web, et comment configurer une machine virtuelle afin de traiter la gestion de la base de données.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé et sécurisé des réseaux Azure concernant les machines virtuelles. 

> [!div class="checklist"]
> * Créer un réseau virtuel et un sous-réseau
> * Créer une adresse IP publique
> * Créer une machine virtuelle frontale
> * sécurisent le trafic réseau
> * Créer une machine virtuelle principale

Passez au didacticiel suivant pour découvrir comment sécuriser et surveiller les données sur des machines virtuelles avec Sauvegarde Azure.

> [!div class="nextstepaction"]
> [Sauvegarde de machines virtuelles Windows dans Azure](./tutorial-backup-vms.md)
