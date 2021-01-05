---
title: Diagnostiquer un problème de routage réseau d’une machine virtuelle – Azure PowerShell
titleSuffix: Azure Network Watcher
description: Dans cet article, découvrez comment diagnostiquer un problème de routage réseau d’une machine virtuelle à l’aide de la fonctionnalité de tronçon suivant d’Azure Network Watcher.
services: network-watcher
documentationcenter: network-watcher
author: damendo
editor: ''
tags: azure-resource-manager
Customer intent: I need to diagnose virtual machine (VM) network routing problem that prevents communication to different destinations.
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 04/20/2018
ms.author: damendo
ms.custom: ''
ms.openlocfilehash: 362157f023f7ed4d2da81962acd32e2da968193e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84738785"
---
# <a name="diagnose-a-virtual-machine-network-routing-problem---azure-powershell"></a>Diagnostiquer un problème de routage réseau d’une machine virtuelle - Azure PowerShell

Dans cet article, vous déployez une machine virtuelle, puis vous vérifiez les communications vers une adresse IP et une URL. Vous déterminez la cause d’un échec de communication et la façon de le résoudre.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell en local, vous devez exécuter le module `Az` Azure PowerShell. Pour trouver la version installée, exécutez `Get-Module -ListAvailable Az`. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.



## <a name="create-a-vm"></a>Créer une machine virtuelle

Avant de pouvoir créer une machine virtuelle, vous devez créer un groupe de ressources pour qu’il contienne la machine virtuelle. Créez un groupe de ressources avec [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus*.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

Créez la machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm). Lors de l’exécution de cette étape, vous êtes invité à saisir vos informations d’identification. Les valeurs que vous saisissez sont configurées comme le nom d’utilisateur et le mot de passe pour la machine virtuelle.

```azurepowershell-interactive
$vM = New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVm" `
    -Location "East US"
```

La création de la machine virtuelle ne nécessite que quelques minutes. Ne poursuivez pas les étapes restantes avant que la machine virtuelle ne soit créée et que PowerShell ne retourne la sortie.

## <a name="test-network-communication"></a>Tester la communication réseau

Pour tester une communication réseau avec Network Watcher, commencez par activer un observateur réseau dans la région de la machine virtuelle que vous souhaitez tester, puis utilisez la fonctionnalité de tronçon suivant de Network Watcher pour tester la communication.

## <a name="enable-network-watcher"></a>Activer Network Watcher

Si vous avez déjà un observateur réseau activé dans la région USA Est, utilisez [Get-AzNetworkWatcher](/powershell/module/az.network/get-aznetworkwatcher) pour le récupérer. L’exemple suivant récupère un observateur réseau existant nommé *NetworkWatcher_eastus* se trouvant dans le groupe de ressources *NetworkWatcherRG* :

```azurepowershell-interactive
$networkWatcher = Get-AzNetworkWatcher `
  -Name NetworkWatcher_eastus `
  -ResourceGroupName NetworkWatcherRG
```

Si vous n’avez pas encore d’observateur réseau activé dans la région USA Est, utilisez [New-AzNetworkWatcher](/powershell/module/az.network/new-aznetworkwatcher) pour créer un observateur réseau dans cette région :

```azurepowershell-interactive
$networkWatcher = New-AzNetworkWatcher `
  -Name "NetworkWatcher_eastus" `
  -ResourceGroupName "NetworkWatcherRG" `
  -Location "East US"
```

### <a name="use-next-hop"></a>Utiliser le tronçon suivant

Azure crée automatiquement des itinéraires vers les destinations par défaut. Vous pouvez créer des itinéraires personnalisés pour remplacer les itinéraires par défaut. Parfois, les itinéraires personnalisés peuvent entraîner l’échec de la communication. Pour tester le routage à partir d’une machine virtuelle, utilisez la commande [Get-AzNetworkWatcherNextHop](/powershell/module/az.network/get-aznetworkwatchernexthop) afin de déterminer le tronçon de routage suivant lorsque le trafic est destiné à une adresse spécifique.

Testez la communication sortante de la machine virtuelle vers l’une des adresses IP pour www.bing.com :

```azurepowershell-interactive
Get-AzNetworkWatcherNextHop `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $VM.Id `
  -SourceIPAddress 192.168.1.4 `
  -DestinationIPAddress 13.107.21.200
```

Après quelques secondes, la sortie vous informe que **NextHopType** est défini sur **Internet** et que **RouteTableId** est défini sur **System Route** (Itinéraire système). Ce résultat vous permet de savoir qu’il existe un itinéraire valide vers la destination.

Testez la communication sortante de la machine virtuelle vers l’adresse 172.31.0.100 :

```azurepowershell-interactive
Get-AzNetworkWatcherNextHop `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $VM.Id `
  -SourceIPAddress 192.168.1.4 `
  -DestinationIPAddress 172.31.0.100
```

La sortie retournée vous informe que **Aucun** est la valeur de **NextHopType** et que **RouteTableId** est également défini sur **System Route** (Itinéraire système). Ce résultat vous permet de savoir que s’il existe un itinéraire système valide vers la destination, il n’existe aucun tronçon suivant pour acheminer le trafic vers la destination.

## <a name="view-details-of-a-route"></a>Afficher les détails d’un itinéraire

Pour procéder à une analyse plus approfondie du routage, passez en revue les itinéraires réels de l’interface réseau avec la commande [Get-AzEffectiveRouteTable](/powershell/module/az.network/get-azeffectiveroutetable) :

```azurepowershell-interactive
Get-AzEffectiveRouteTable `
  -NetworkInterfaceName myVm `
  -ResourceGroupName myResourceGroup |
  Format-table
```

Une sortie contenant le texte suivant est retournée :

```powershell
Name State  Source  AddressPrefix           NextHopType NextHopIpAddress
---- -----  ------  -------------           ----------- ----------------
     Active Default {192.168.0.0/16}        VnetLocal   {}              
     Active Default {0.0.0.0/0}             Internet    {}              
     Active Default {10.0.0.0/8}            None        {}              
     Active Default {100.64.0.0/10}         None        {}              
     Active Default {172.16.0.0/12}         None        {}              
```

Comme vous pouvez le voir dans la sortie précédente, la route avec **AddressPrefix** défini sur **0.0.0.0/0** achemine tout le trafic non destiné aux adresses dans les préfixes d’adresses d’une autre route avec un tronçon suivant défini sur **Internet**. Comme vous pouvez le voir aussi dans la sortie, même s’il existe un itinéraire par défaut pour le préfixe 172.16.0.0/12, qui inclut l’adresse 172.31.0.100, **NextHopType** est défini sur **Aucun**. Azure crée un itinéraire par défaut pour 172.16.0.0/12, mais ne spécifie pas de type de tronçon suivant tant qu’aucune raison ne motive cette spécification. Si, par exemple, vous avez ajouté la plage d’adresses 172.16.0.0/12 à l’espace d’adressage du réseau virtuel, Azure modifie **NextHopType** pour le définir sur **Réseau virtuel** pour l’itinéraire. Une vérification permet ensuite d’afficher **Réseau virtuel** en tant que **NextHopType**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, vous pouvez utiliser [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour le supprimer ainsi que toutes les ressources qu’il contient :

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez créé une machine virtuelle et diagnostiqué un problème de routage réseau à partir de celle-ci. Vous avez appris qu’Azure crée plusieurs itinéraires par défaut et testé le routage vers deux destinations différentes. En savoir plus sur le [routage dans Azure](../virtual-network/virtual-networks-udr-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) et la [création d’itinéraires personnalisés](../virtual-network/manage-route-table.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-route).

Pour les connexions sortantes d’une machine virtuelle, vous pouvez également déterminer la latence, le trafic réseau autorisé et refusé entre la machine virtuelle et un point de terminaison, à l’aide de la fonctionnalité [Résolution des problèmes de connexion](network-watcher-connectivity-powershell.md) de Network Watcher. Vous pouvez surveiller la communication entre une machine virtuelle et un point de terminaison, par exemple une adresse IP ou une URL, au fil du temps à l’aide de la fonctionnalité de contrôle de la connexion de Network Watcher. Pour découvrir comment procéder, consultez [Surveiller une connexion réseau](connection-monitor.md).
