---
title: Diagnostiquer un problème de routage sur une machine virtuelle Azure | Microsoft Docs
description: Découvrez comment diagnostiquer un problème de routage de machine virtuelle en consultant les itinéraires effectifs d’une machine virtuelle.
services: virtual-network
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/30/2018
ms.author: kumud
ms.openlocfilehash: 1c23244707179e05c63ed44b5915e58eefd3f4a3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84705047"
---
# <a name="diagnose-a-virtual-machine-routing-problem"></a>Diagnostiquer un problème de routage sur une machine virtuelle

Dans cet article, vous apprenez à diagnostiquer un problème de routage en regardant les itinéraires qui sont effectifs pour une interface réseau sur une machine virtuelle. Azure crée plusieurs itinéraires par défaut pour chaque sous-réseau de machine virtuelle. Vous pouvez remplacer les itinéraires par défaut d’Azure, en définissant des itinéraires dans une table de routage que vous associez ensuite à un sous-réseau. Ce mélange d’itinéraires que vous créez, d’itinéraires par défaut d’Azure et d’itinéraires propagés à partir de votre réseau local via une passerelle VPN Azure (si le réseau virtuel est connecté au réseau local) en utilisant Border Gateway Protocol (BGP), constitue les itinéraires effectifs de toutes les interfaces réseau d’un sous-réseau. Si les notions de réseau virtuel, d’interface réseau ou de routage ne vous sont pas familières, consultez [Vue d’ensemble du réseau virtuel](virtual-networks-overview.md), [Interface réseau](virtual-network-network-interface.md) et [Vue d’ensemble du routage](virtual-networks-udr-overview.md).

## <a name="scenario"></a>Scénario

Vous essayez de vous connecter à une machine virtuelle, mais la connexion échoue. Afin de déterminer la raison pour laquelle vous ne pouvez pas vous connecter à la machine virtuelle, consultez les itinéraires effectifs d’une interface réseau au moyen du [portail](#diagnose-using-azure-portal) Azure, de [PowerShell](#diagnose-using-powershell), ou d’[Azure CLI](#diagnose-using-azure-cli).

Les étapes qui suivent supposent que vous disposez d’une machine virtuelle existante qui permet d’afficher les itinéraires effectifs. Si vous ne possédez pas une telle machine, commencez par déployer une machine virtuelle [Linux](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou [Windows](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) pour pouvoir accomplir les tâches de cet article. Les exemples contenus dans cet article sont prévus pour une machine virtuelle nommée *myVM*, et une interface réseau appelée *myVMNic1*. La machine virtuelle et l’interface réseau se trouvent dans un groupe de ressources nommé *myResourceGroup*, et se situent dans la région *USA Est*. Modifiez les valeurs dans les étapes, selon le cas, pour la machine virtuelle dont vous analysez le problème.

## <a name="diagnose-using-azure-portal"></a>Diagnostiquer à l’aide du portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte Azure disposant des [autorisations nécessaires](virtual-network-network-interface.md#permissions).
2. En haut du portail Azure, dans la zone de recherche, indiquez le nom d’une machine virtuelle en cours d’exécution. Quand le nom de cette machine virtuelle apparaît dans les résultats de la recherche, sélectionnez-le.
3. Sous **Paramètres** sur la gauche, sélectionnez **Mise en réseau** et accédez à la ressource d’interface réseau en sélectionnant son nom.
     ![Afficher les interfaces réseau](./media/diagnose-network-routing-problem/view-nics.png)
4. Sur la gauche, sélectionnez **Itinéraires effectifs**. Les itinéraires effectifs d’une interface réseau nommée **myVMNic1** sont Montrés dans l’image suivante : ![Afficher les itinéraires effectifs](./media/diagnose-network-routing-problem/view-effective-routes.png)

    S’il existe plusieurs interfaces réseau attachées à la machine virtuelle, vous pouvez voir les itinéraires effectifs de chaque interface réseau en sélectionnant les interfaces une à une. Chaque interface réseau pouvant se trouver dans un sous-réseau différent, ces interfaces peuvent chacune disposer de plusieurs itinéraires effectifs.

    Dans l’exemple de l’image précédente, les itinéraires répertoriés sont les itinéraires par défaut qu’Azure crée pour chaque sous-réseau. Votre liste contient au moins ces itinéraires, mais elle peut s’enrichir d’itinéraires supplémentaires, selon les fonctionnalités que vous avez peut-être activées pour votre réseau virtuel, par exemple son appairage à un autre réseau virtuel ou sa connexion à votre réseau local par le biais d’une passerelle VPN Azure. Pour en savoir plus sur chacun de ces itinéraires, et sur les autres itinéraires pouvant s’afficher pour votre interface réseau, consultez [Routage de trafic de réseaux virtuels](virtual-networks-udr-overview.md). Si votre liste comporte un grand nombre d’itinéraires, il peut être plus facile de sélectionner **Télécharger** pour récupérer un fichier .csv contenant la liste des itinéraires.

Même si dans les étapes précédentes, les itinéraires effectifs ont été affichés à l’aide de la machine virtuelle, vous pouvez également les voir avec une :
- **Interface réseau individuelle** : découvrez comment [voir une interface réseau](virtual-network-network-interface.md#view-network-interface-settings).
- **Table de routage individuelle** : Découvrez comment [voir une table de routage](manage-route-table.md#view-details-of-a-route-table).

## <a name="diagnose-using-powershell"></a>Diagnostiquer à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous pouvez exécuter les commandes qui suivent dans [Azure Cloud Shell](https://shell.azure.com/powershell), ou en exécutant PowerShell à partir de votre ordinateur. Azure Cloud Shell est un interpréteur de commandes interactif gratuit. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Si vous exécutez PowerShell sur votre ordinateur, vous devez utiliser le module Azure PowerShell version 1.0.0 ou ultérieure. Exécutez `Get-Module -ListAvailable Az` sur votre ordinateur pour trouver la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps). Si vous exécutez PowerShell localement, vous devez aussi exécuter `Connect-AzAccount` pour vous connecter à Azure avec un compte disposant des [autorisations nécessaires](virtual-network-network-interface.md#permissions).

Obtenez les itinéraires effectifs d’une interface réseau avec [Get-AzEffectiveRouteTable](/powershell/module/az.network/get-azeffectiveroutetable). L’exemple suivant récupère les itinéraires effectifs d’une interface réseau nommée *myVMNic1*, qui se trouve dans un groupe de ressources appelé *myResourceGroup* :

```azurepowershell-interactive
Get-AzEffectiveRouteTable `
  -NetworkInterfaceName myVMNic1 `
  -ResourceGroupName myResourceGroup `
  | Format-Table
```

Pour comprendre les informations retournées dans le résultat, consultez [Vue d’ensemble du routage](virtual-networks-udr-overview.md). Le résultat n’est retourné que si la machine virtuelle est en cours d’exécution. S’il existe plusieurs interfaces réseau attachées à la machine virtuelle, vous pouvez examiner les itinéraires effectifs de chaque interface réseau. Chaque interface réseau pouvant se trouver dans un sous-réseau différent, ces interfaces peuvent chacune disposer de plusieurs itinéraires effectifs. Si des problèmes de communication subsistent, consultez [Diagnostic supplémentaire](#additional-diagnosis) et [Considérations](#considerations).

Si vous ignorez le nom d’une interface réseau, mais que vous connaissez le nom de la machine virtuelle à laquelle l’interface réseau est attachée, les commandes suivantes retournent les ID de toutes les interfaces réseau attachées à une machine virtuelle :

```azurepowershell-interactive
$VM = Get-AzVM -Name myVM `
  -ResourceGroupName myResourceGroup
$VM.NetworkProfile
```

Le résultat ressemble à ce qui suit :

```powershell
NetworkInterfaces
-----------------
{/subscriptions/<ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMNic1
```

Dans la précédente sortie, le nom d’interface réseau est *myVMNic1*.

## <a name="diagnose-using-azure-cli"></a>Diagnostiquer à l’aide d’Azure CLI

Vous pouvez exécuter les commandes qui suivent dans [Azure Cloud Shell](https://shell.azure.com/bash), ou en exécutant l’interface CLI à partir de votre ordinateur. Azure CLI version 2.0.32 ou ultérieure est nécessaire pour cet article. Exécutez `az --version` pour rechercher la version installée. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli). Si vous exécutez Azure CLI localement, vous devez aussi exécuter `az login` et vous connecter à Azure avec un compte disposant des [autorisations nécessaires](virtual-network-network-interface.md#permissions).

Obtenez les itinéraires effectifs d’une interface réseau avec [az network nic show-effective-route-table](/cli/azure/network/nic#az-network-nic-show-effective-route-table). L’exemple suivant récupère les itinéraires effectifs d’une interface réseau nommée *myVMNic1*, qui se trouve dans un groupe de ressources appelé *myResourceGroup* :

```azurecli-interactive
az network nic show-effective-route-table \
  --name myVMNic1 \
  --resource-group myResourceGroup
```

Pour comprendre les informations retournées dans le résultat, consultez [Vue d’ensemble du routage](virtual-networks-udr-overview.md). Le résultat n’est retourné que si la machine virtuelle est en cours d’exécution. S’il existe plusieurs interfaces réseau attachées à la machine virtuelle, vous pouvez examiner les itinéraires effectifs de chaque interface réseau. Chaque interface réseau pouvant se trouver dans un sous-réseau différent, ces interfaces peuvent chacune disposer de plusieurs itinéraires effectifs. Si des problèmes de communication subsistent, consultez [Diagnostic supplémentaire](#additional-diagnosis) et [Considérations](#considerations).

Si vous ignorez le nom d’une interface réseau, mais que vous connaissez le nom de la machine virtuelle à laquelle l’interface réseau est attachée, les commandes suivantes retournent les ID de toutes les interfaces réseau attachées à une machine virtuelle :

```azurecli-interactive
az vm show \
  --name myVM \
  --resource-group myResourceGroup
```

## <a name="resolve-a-problem"></a>Résoudre un problème

En règle générale, la résolution des problèmes de routage englobe les solutions suivantes :

- Ajout d’un itinéraire personnalisé pour remplacer un des itinéraires par défaut d’Azure. Découvrez comment [ajouter un itinéraire personnalisé](manage-route-table.md#create-a-route).
- Modification ou suppression d’un itinéraire personnalisé pouvant provoquer le routage vers un emplacement non souhaité. Découvrez comment [modifier](manage-route-table.md#change-a-route) ou [supprimer](manage-route-table.md#delete-a-route) un itinéraire personnalisé.
- Vérification de l’association de la table de routage qui contient les itinéraires personnalisés définis, au sous-réseau dans lequel se trouve l’interface réseau. Découvrez comment [associer une table de routage à un sous-réseau](manage-route-table.md#associate-a-route-table-to-a-subnet).
- Vérification du bon fonctionnement des dispositifs, tels qu’une passerelle VPN Azure ou des appliances virtuelles de réseau déployées. Utilisez la fonctionnalité [Diagnostics VPN](../network-watcher/diagnose-communication-problem-between-networks.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Network Watcher pour déterminer les problèmes rencontrés avec une passerelle VPN Azure.

Si des problèmes de communication subsistent, consultez [Considérations](#considerations) et Diagnostic supplémentaire.

## <a name="considerations"></a>Considérations

Lors de la résolution de problèmes de communication, considérez les points suivants :

- Le routage est basé sur le plus long préfixe correspondant parmi les itinéraires que vous avez définis, le BGP (Border Gateway Protocol) et les itinéraires système. S’il existe plusieurs itinéraires avec la même correspondance de préfixe le plus long, un itinéraire est sélectionné en fonction de son origine, selon l’ordre répertorié dans [Vue d’ensemble du routage](virtual-networks-udr-overview.md#how-azure-selects-a-route). Avec les itinéraires effectifs, vous ne pouvez voir que les itinéraires effectifs présentant une correspondance de préfixe le plus long, par rapport à tous les itinéraires disponibles. En comprenant comment sont évalués les itinéraires d’une interface réseau, il est beaucoup plus facile ensuite de résoudre les problèmes d’itinéraires spécifiques qui peuvent avoir une incidence sur la communication depuis votre machine virtuelle.
- Si vous avez défini des itinéraires personnalisés vers une appliance virtuelle de réseau, en utilisant *Appliance virtuelle* comme type de tronçon suivant, assurez-vous que le transfert IP est activé sur l’appliance virtuelle de réseau recevant le trafic, sinon les paquets sont ignorés. En savoir plus sur l’[activation du transfert IP pour une interface réseau](virtual-network-network-interface.md#enable-or-disable-ip-forwarding). Le système d’exploitation, ou l’application dans l’appliance virtuelle de réseau, doit être également en mesure de transférer le trafic réseau et être configuré pour cela.
- Si vous avez créé un itinéraire vers 0.0.0.0/0, l’intégralité du trafic internet sortant est acheminé vers le tronçon suivant que vous avez spécifié, par exemple vers une appliance virtuelle de réseau ou une passerelle VPN. La création d’un itinéraire de ce type est généralement appelé « tunneling forcé ». Les connexions à distance, utilisant les protocoles RDP ou SSH depuis internet vers votre machine virtuelle, peuvent ne pas fonctionner avec cet itinéraire, selon la façon dont le tronçon suivant gère le trafic. Le tunneling forcé peut être activé :
    - Lors de l’utilisation du VPN de site à site, en créant un itinéraire avec comme type de tronçon suivant une *passerelle VPN*. En savoir plus sur la [configuration du tunneling forcé](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
    - Si un 0.0.0.0/0 (itinéraire par défaut) est publié sur BGP via une passerelle de réseau virtuel lors de l’utilisation d’un VPN de site à site, ou du circuit ExpressRoute. En savoir plus sur l’utilisation de BGP avec un [VPN de site à site](../vpn-gateway/vpn-gateway-bgp-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou [ExpressRoute](../expressroute/expressroute-routing.md?toc=%2fazure%2fvirtual-network%2ftoc.json#ip-addresses-used-for-azure-private-peering).
- Pour que le trafic d’homologation de réseau virtuel fonctionne correctement, un itinéraire système, avec comme type de tronçon suivant *VNet Peering*, doit exister pour la plage de préfixes du réseau virtuel homologué. S’il n’existe aucune route de ce type, et si le lien de peering de réseau virtuel est **Connecté** :
    - Attendez quelques secondes et réessayez. S’il s’agit d’un lien de peering récemment établi, il faut parfois plus de temps pour propager les itinéraires à toutes les interfaces réseau d’un sous-réseau. Pour en savoir plus sur le peering de réseau virtuel, consultez la [Vue d’ensemble du peering de réseau virtuel](virtual-network-peering-overview.md) et la [gestion du peering de réseau virtuel](virtual-network-manage-peering.md).
    - Les règles du groupe de sécurité réseau peuvent avoir une incidence sur la communication. Pour plus d’informations, consultez [Diagnostiquer un problème de filtre de trafic réseau sur une machine virtuelle](diagnose-network-traffic-filter-problem.md).
- Même si Azure affecte des itinéraires par défaut à chaque interface réseau Azure, si vous disposez de plusieurs interfaces réseau attachées à la machine virtuelle, seule l’interface réseau principale se voit attribuer un itinéraire par défaut (0.0.0.0/0), ou une passerelle, au sein du système d’exploitation de la machine virtuelle. Découvrez comment créer un itinéraire par défaut pour les interfaces réseau secondaires attachées à une machine virtuelle [Windows](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) ou [Linux](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics). Apprenez-en davantage sur les [interfaces réseau principale et secondaire](virtual-network-network-interface-vm.md#constraints).

## <a name="additional-diagnosis"></a>Diagnostic supplémentaire

* Pour exécuter un test rapide en vue de déterminer le type de tronçon suivant pour le trafic destiné à un emplacement, utilisez la fonctionnalité [Tronçon suivant](../network-watcher/diagnose-vm-network-routing-problem.md?toc=%2fazure%2fvirtual-network%2ftoc.json) d’Azure Network Watcher. Cette fonctionnalité vous renseigne sur le type de tronçon suivant du trafic destiné à un emplacement spécifié.
* Si aucun itinéraire n’est à l’origine de l’échec de la communication réseau d’une machine virtuelle, le problème peut provenir d’un logiciel de pare-feu s’exécutant dans le système d’exploitation de la machine virtuelle.
* Si vous établissez un trafic de [tunneling forcé](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md?toc=%2fazure%2fvirtual-network%2ftoc.json) vers un appareil local par l’intermédiaire d’une passerelle VPN, ou d’une appliance virtuelle de réseau, il est possible que vous ne puissiez pas vous connecter à une machine virtuelle depuis internet, selon la façon dont vous avez configuré le routage pour ces dispositifs. Vérifiez que le routage configuré pour le dispositif achemine le trafic jusqu’à une adresse IP publique ou privée pour la machine virtuelle.
* Utilisez la fonctionnalité [Résolution des problèmes de connexion](../network-watcher/network-watcher-connectivity-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Network Watcher pour déterminer les causes de routage, de filtrage ou les causes internes au système d’exploitation pouvant être à l’origine des problèmes de communication sortante.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les tâches, les propriétés et les paramètres d’une [table de routage et des itinéraires](manage-route-table.md).
- En savoir plus sur tous les [types de tronçon suivant, les itinéraires système et sur la façon dont Azure choisit un itinéraire](virtual-networks-udr-overview.md).
