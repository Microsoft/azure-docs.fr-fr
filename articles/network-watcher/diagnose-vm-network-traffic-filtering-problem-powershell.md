---
title: 'Démarrage rapide : Diagnostiquer un problème de filtre de trafic réseau sur une machine virtuelle - Azure PowerShell'
titleSuffix: Azure Network Watcher
description: Découvrez comment utiliser Azure PowerShell pour diagnostiquer un problème de filtre de trafic réseau sur une machine virtuelle à l’aide de la fonctionnalité de vérification de flux IP d’Azure Network Watcher.
services: network-watcher
documentationcenter: network-watcher
author: damendo
editor: ''
tags: azure-resource-manager
Customer intent: I need to diagnose a virtual machine (VM) network traffic filter problem that prevents communication to and from a VM.
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 04/20/2018
ms.author: damendo
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 8483c0d3b112408091e10bd9b57451cf2378c859
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96494475"
---
# <a name="quickstart-diagnose-a-virtual-machine-network-traffic-filter-problem---azure-powershell"></a>Démarrage rapide : Diagnostiquer un problème de filtre de trafic réseau d’une machine virtuelle - Azure PowerShell

Dans ce guide de démarrage rapide, vous déployez une machine virtuelle, puis vous vérifiez les communications vers une adresse IP et une URL et à partir d’une adresse IP. Vous déterminez la cause d’un échec de communication et la façon de le résoudre.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell localement, vous devez exécuter le module Azure PowerShell `Az` pour les besoins de ce guide de démarrage rapide. Pour trouver la version installée, exécutez `Get-Module -ListAvailable Az`. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.



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

Pour tester une communication réseau avec Network Watcher, commencez par activer un observateur réseau dans la région de la machine virtuelle que vous souhaitez tester, puis utilisez la fonctionnalité de vérification de flux IP de Network Watcher pour tester la communication.

### <a name="enable-network-watcher"></a>Activer Network Watcher

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

### <a name="use-ip-flow-verify"></a>Utiliser la vérification des flux IP

Lorsque vous créez une machine virtuelle, Azure autorise et refuse le trafic réseau à destination et en provenance de la machine virtuelle, par défaut. Vous pourrez ultérieurement remplacer des valeurs Azure par défaut, en autorisant ou en refusant d’autres types de trafic. Pour tester si le trafic est autorisé ou refusé vers différentes destinations et à partir d’une adresse IP source, utilisez la commande [Test-AzNetworkWatcherIPFlow](/powershell/module/az.network/test-aznetworkwatcheripflow).

Testez la communication sortante de la machine virtuelle vers l’une des adresses IP pour www.bing.com :

```azurepowershell-interactive
Test-AzNetworkWatcherIPFlow `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $vM.Id `
  -Direction Outbound `
  -Protocol TCP `
  -LocalIPAddress 192.168.1.4 `
  -LocalPort 60000 `
  -RemoteIPAddress 13.107.21.200 `
  -RemotePort 80
```

Après quelques secondes, le résultat retourné vous informe que l’accès est autorisé par une règle de sécurité nommée **AllowInternetOutbound**.

Testez la communication sortante de la machine virtuelle vers l’adresse 172.31.0.100 :

```azurepowershell-interactive
Test-AzNetworkWatcherIPFlow `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $vM.Id `
  -Direction Outbound `
  -Protocol TCP `
  -LocalIPAddress 192.168.1.4 `
  -LocalPort 60000 `
  -RemoteIPAddress 172.31.0.100 `
  -RemotePort 80
```

Le résultat retourné vous informe que l’accès est refusé par une règle de sécurité nommée **DefaultOutboundDenyAll**.

Testez la communication entrante vers la machine virtuelle à partir de l’adresse 172.31.0.100 :

```azurepowershell-interactive
Test-AzNetworkWatcherIPFlow `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId $vM.Id `
  -Direction Inbound `
  -Protocol TCP `
  -LocalIPAddress 192.168.1.4 `
  -LocalPort 80 `
  -RemoteIPAddress 172.31.0.100 `
  -RemotePort 60000
```

Le résultat retourné vous informe que l’accès est refusé par une règle de sécurité nommée **DefaultInboundDenyAll**. À présent que vous savez quelles règles de sécurité autorisent ou refusent le trafic à destination ou en provenance d’une machine virtuelle, vous pouvez déterminer comment résoudre les problèmes.

## <a name="view-details-of-a-security-rule"></a>Voir les détails d’une règle de sécurité

Pour déterminer la raison pour laquelle les règles de l’étape [Tester la communication réseau](#test-network-communication) autorisent ou empêchent la communication, examinez les règles de sécurité en vigueur pour l’interface réseau avec la commande [Get-AzEffectiveNetworkSecurityGroup](/powershell/module/az.network/get-azeffectivenetworksecuritygroup) :

```azurepowershell-interactive
Get-AzEffectiveNetworkSecurityGroup `
  -NetworkInterfaceName myVm `
  -ResourceGroupName myResourceGroup
```

La sortie renvoyée contient le texte suivant pour la règle **AllowInternetOutbound** qui a autorisé un accès sortant vers www.bing.com sous [Utiliser la vérification des flux IP](#use-ip-flow-verify) :

```powershell
{
  "Name":
"defaultSecurityRules/AllowInternetOutBound",
  "Protocol": "All",
  "SourcePortRange": [
    "0-65535"
  ],
  "DestinationPortRange": [
    "0-65535"
  ],
  "SourceAddressPrefix": [
    "0.0.0.0/0"
  ],
  "DestinationAddressPrefix": [
    "Internet"
  ],
  "ExpandedSourceAddressPrefix": [],
  "ExpandedDestinationAddressPrefix": [
    "1.0.0.0/8",
    "2.0.0.0/7",
    "4.0.0.0/6",
    "8.0.0.0/7",
    "11.0.0.0/8",
    "12.0.0.0/6",
    ...
    ],
    "Access": "Allow",
    "Priority": 65001,
    "Direction": "Outbound"
  },
```

Vous pouvez voir dans la sortie que **DestinationAddressPrefix** est **Internet**. Pourtant, 13.107.21.200, l’adresse que vous avez testée sous [Utiliser la vérification de flux IP](#use-ip-flow-verify), n’est pas clairement liée à **Internet**. Plusieurs préfixes d’adresse sont répertoriés sous **ExpandedDestinationAddressPrefix**. L’un des préfixes de la liste est **12.0.0.0/6**, ce qui englobe la plage d’adresses IP 12.0.0.1-15.255.255.254. Étant donné que l’adresse 13.107.21.200 se trouve dans cette plage d’adresses, la règle **AllowInternetOutBound** autorise le trafic sortant. En outre, il n’y a aucune règle de **priorité** supérieure (numéro inférieur) indiquée dans la sortie retournée par `Get-AzEffectiveNetworkSecurityGroup`, qui remplace cette règle. Pour refuser les communications sortantes vers 13.107.21.200, vous pouvez ajouter une règle de sécurité avec une priorité plus élevée, ce qui empêche la sortie du port 80 vers l’adresse IP.

Lorsque vous avez exécuté la commande `Test-AzNetworkWatcherIPFlow` pour tester la communication sortante vers l’adresse 172.131.0.100 dans [Utiliser la vérification des flux IP](#use-ip-flow-verify), la sortie vous a informé que la règle **DefaultOutboundDenyAll** a refusé la communication. La règle **DefaultOutboundDenyAll** équivaut à la règle **DenyAllOutBound** répertoriée dans la sortie suivante de la commande `Get-AzEffectiveNetworkSecurityGroup` :

```powershell
{
"Name": "defaultSecurityRules/DenyAllOutBound",
"Protocol": "All",
"SourcePortRange": [
  "0-65535"
],
"DestinationPortRange": [
  "0-65535"
],
"SourceAddressPrefix": [
  "0.0.0.0/0"
],
"DestinationAddressPrefix": [
  "0.0.0.0/0"
],
"ExpandedSourceAddressPrefix": [],
"ExpandedDestinationAddressPrefix": [],
"Access": "Deny",
"Priority": 65500,
"Direction": "Outbound"
}
```

La règle répertorie **0.0.0.0/0** comme **DestinationAddressPrefix**. La règle refuse les communications sortantes vers l’adresse 172.131.0.100, car l’adresse ne se trouve pas dans le même **DestinationAddressPrefix** que toutes les autres règles de trafic sortant dans la sortie de la commande `Get-AzEffectiveNetworkSecurityGroup`. Pour autoriser les communications sortantes, vous pouvez ajouter une règle de sécurité avec une priorité plus élevée, afin d’autoriser le trafic sortant vers le port 80 à l’adresse 172.131.0.100.

Lorsque vous avez exécuté la commande `Test-AzNetworkWatcherIPFlow` pour tester la communication entrante à partir l’adresse 172.131.0.100 dans [Utiliser la vérification des flux IP](#use-ip-flow-verify), la sortie vous a informé que la règle **DefaultInboundDenyAll** a refusé la communication. La règle **DefaultInboundDenyAll** équivaut à la règle **DenyAllInBound** répertoriée dans la sortie suivante de la commande `Get-AzEffectiveNetworkSecurityGroup` :

```powershell
{
"Name": "defaultSecurityRules/DenyAllInBound",
"Protocol": "All",
"SourcePortRange": [
  "0-65535"
],
"DestinationPortRange": [
  "0-65535"
],
"SourceAddressPrefix": [
  "0.0.0.0/0"
],
"DestinationAddressPrefix": [
  "0.0.0.0/0"
],
"ExpandedSourceAddressPrefix": [],
"ExpandedDestinationAddressPrefix": [],
"Access": "Deny",
"Priority": 65500,
"Direction": "Inbound"
},
```

La règle **DenyAllInBound** est appliquée, car comme indiqué dans la sortie, aucune règle de priorité plus élevée n’est présente dans la sortie de la commande `Get-AzEffectiveNetworkSecurityGroup` afin d’autoriser le port 80 entrant de l’adresse 172.131.0.100 vers la machine virtuelle. Pour autoriser les communications entrantes, vous pouvez ajouter une règle de sécurité avec une priorité plus élevée, afin d’autoriser le trafic entrant vers le port 80 à partir de l’adresse 172.131.0.100.

Les vérifications de ce guide de démarrage rapide ont permis de tester la configuration Azure. Si les vérifications effectuées retournent les résultats attendus alors que vous rencontrez toujours des problèmes réseau, vérifiez qu’il n’y a aucun pare-feu entre votre machine virtuelle et le point de terminaison avec lequel vous communiquez, et que le système d’exploitation dans votre machine virtuelle n’a pas de pare-feu qui autorise ou refuse les communications.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, vous pouvez utiliser [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour le supprimer ainsi que toutes les ressources qu’il contient :

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé une machine virtuelle et diagnostiqué des filtres de trafic réseau entrant et sortant. Vous avez appris que les règles de groupe de sécurité réseau autorisent ou refusent le trafic à destination et en provenance d’une machine virtuelle. En savoir plus sur les [règles de sécurité](../virtual-network/network-security-groups-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) et la [création des règles de sécurité](../virtual-network/manage-network-security-group.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-security-rule).

Même avec des filtres de trafic réseau adaptés, les communications vers une machine virtuelle peuvent échouer en raison d’une configuration de routage. Pour savoir comment diagnostiquer les problèmes de routage réseau d’une machine virtuelle, consultez [Diagnostiquer des problèmes de routage sur une machine virtuelle](diagnose-vm-network-routing-problem-powershell.md) ou, pour diagnostiquer les problèmes liés au routage sortant, à la latence et au filtrage de trafic, avec un outil, consultez [Résoudre les problèmes de connexion](network-watcher-connectivity-powershell.md).