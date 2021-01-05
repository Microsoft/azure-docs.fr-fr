---
title: Ajouter ou supprimer une délégation de sous-réseau dans un réseau virtuel Azure
titlesuffix: Azure Virtual Network
description: Découvrez comment ajouter ou supprimer un sous-réseau délégué pour un service dans Azure.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/06/2019
ms.author: kumud
ms.openlocfilehash: 4e4f002d038820edf128e3fefb229a0918a8ac55
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96433517"
---
# <a name="add-or-remove-a-subnet-delegation"></a>Ajouter ou supprimer une délégation de sous-réseau

La délégation de sous-réseau donne des autorisations explicites au service pour créer des ressources propres au service dans le sous-réseau à l’aide d’un identificateur unique pendant le déploiement du service. Cet article explique comment ajouter un sous-réseau délégué à un service Azure et comment l’en supprimer.

## <a name="portal"></a>Portail

### <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

### <a name="create-the-virtual-network"></a>Créer un réseau virtuel

Dans le cadre de cette section, vous allez créer un réseau virtuel et le sous-réseau que vous déléguerez plus tard à un service Azure.

1. Dans le coin supérieur gauche de l’écran, sélectionnez **Créer une ressource** > **Mise en réseau** > **Réseau virtuel**.
1. Dans **Créer un réseau virtuel**, entrez ou sélectionnez ces informations :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Name | Entrez *MyVirtualNetwork*. |
    | Espace d’adressage | Entrez *10.0.0.0/16*. |
    | Subscription | Sélectionnez votre abonnement.|
    | Resource group | Sélectionnez **Créer nouveau**, entrez *myResourceGroup* et sélectionnez **OK**. |
    | Location | Sélectionnez **USA Est**.|
    | Sous-réseau - Nom | Entrez *mySubnet*. |
    | Plage d’adresses du sous-réseau | Entrez *10.0.0.0/24*. |
    |||
1. Conservez les autres valeurs par défaut, puis sélectionnez **Créer**.

### <a name="permissions"></a>Autorisations

Si vous n’avez pas créé le sous-réseau que vous souhaitez déléguer à un service Azure, vous devez disposer de l’autorisation suivante : `Microsoft.Network/virtualNetworks/subnets/write`.

Le rôle intégré [Contributeur réseau](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) inclut également les autorisations nécessaires.

### <a name="delegate-a-subnet-to-an-azure-service"></a>Déléguer un sous-réseau à un service Azure

Dans le cadre de cette section, vous allez déléguer le sous-réseau que vous avez créé à la section précédente à un service Azure.

1. Dans la barre de recherche du portail, entrez *myVirtualNetwork*. Quand la mention **myVirtualNetwork** apparaît dans les résultats de recherche, sélectionnez-la.
2. Dans les résultats de recherche, sélectionnez *myVirtualNetwork*.
3. Sélectionnez **Sous-réseaux** sous **PARAMÈTRES**, puis sélectionnez **mySubnet**.
4. Dans la page *mySubnet*, pour la liste **Délégation de sous-réseau**, sélectionnez l’un des services listés sous **Déléguer le sous-réseau à un service** (par exemple, **Microsoft.DBforPostgreSQL/serversv2**).  

### <a name="remove-subnet-delegation-from-an-azure-service"></a>Supprimer une délégation de sous-réseau d’un service Azure

1. Dans la barre de recherche du portail, entrez *myVirtualNetwork*. Quand la mention **myVirtualNetwork** apparaît dans les résultats de recherche, sélectionnez-la.
2. Dans les résultats de recherche, sélectionnez *myVirtualNetwork*.
3. Sélectionnez **Sous-réseaux** sous **PARAMÈTRES**, puis sélectionnez **mySubnet**.
4. Dans la page *mySubnet*, pour la liste **Délégation de sous-réseau**, sélectionnez **Aucun** dans les services listés sous **Déléguer le sous-réseau à un service**. 

## <a name="azure-cli"></a>Azure CLI

Préparez votre environnement pour l’interface Azure CLI.

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Cet article nécessite la version 2.0.28 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

### <a name="create-a-resource-group"></a>Créer un groupe de ressources
Créez un groupe de ressources avec la commande [az group create](https://docs.microsoft.com/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

L’exemple suivant crée un groupe de ressources nommé **myResourceGroup** à l’emplacement **eastus** :

```azurecli-interactive

  az group create \
    --name myResourceGroup \
    --location eastus

```

### <a name="create-a-virtual-network"></a>Créez un réseau virtuel
Créez un réseau virtuel nommé **myVnet** avec un sous-réseau nommé **mySubnet** dans **myResourceGroup** à l’aide de la commande [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet).

```azurecli-interactive
  az network vnet create \
    --resource-group myResourceGroup \
    --location eastus \
    --name myVnet \
    --address-prefix 10.0.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 10.0.0.0/24
```
### <a name="permissions"></a>Autorisations

Si vous n’avez pas créé le sous-réseau que vous souhaitez déléguer à un service Azure, vous devez disposer de l’autorisation suivante : `Microsoft.Network/virtualNetworks/subnets/write`.

Le rôle intégré [Contributeur réseau](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) inclut également les autorisations nécessaires.

### <a name="delegate-a-subnet-to-an-azure-service"></a>Déléguer un sous-réseau à un service Azure

Dans le cadre de cette section, vous allez déléguer le sous-réseau que vous avez créé à la section précédente à un service Azure. 

Utilisez [az network vnet subnet update](https://docs.microsoft.com/cli/azure/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-update) pour mettre à jour le sous-réseau nommé **mySubnet** avec une délégation à un service Azure.  Dans cet exemple **Microsoft.DBforPostgreSQL/serversv2** est utilisé pour l’exemple de délégation :

```azurecli-interactive
  az network vnet subnet update \
  --resource-group myResourceGroup \
  --name mySubnet \
  --vnet-name myVnet \
  --delegations Microsoft.DBforPostgreSQL/serversv2
```

Pour vérifier que la délégation a été appliquée, utilisez [az network vnet subnet show](https://docs.microsoft.com/cli/azure/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-show). Vérifiez que le service est délégué au sous-réseau sous la propriété **serviceName** :

```azurecli-interactive
  az network vnet subnet show \
  --resource-group myResourceGroup \
  --name mySubnet \
  --vnet-name myVnet \
  --query delegations
```

```json
[
  {
    "actions": [
      "Microsoft.Network/virtualNetworks/subnets/join/action"
    ],
    "etag": "W/\"8a8bf16a-38cf-409f-9434-fe3b5ab9ae54\"",
    "id": "/subscriptions/3bf09329-ca61-4fee-88cb-7e30b9ee305b/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet/delegations/0",
    "name": "0",
    "provisioningState": "Succeeded",
    "resourceGroup": "myResourceGroup",
    "serviceName": "Microsoft.DBforPostgreSQL/serversv2",
    "type": "Microsoft.Network/virtualNetworks/subnets/delegations"
  }
]
```

### <a name="remove-subnet-delegation-from-an-azure-service"></a>Supprimer une délégation de sous-réseau d’un service Azure

Utilisez [az network vnet subnet update](https://docs.microsoft.com/cli/azure/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-update) pour supprimer la délégation du sous-réseau nommé **mySubnet** :

```azurecli-interactive
  az network vnet subnet update \
  --resource-group myResourceGroup \
  --name mySubnet \
  --vnet-name myVnet \
  --remove delegations
```
Pour vérifier que la délégation a été retirée, utilisez [az network vnet subnet show](https://docs.microsoft.com/cli/azure/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-show). Vérifiez que le service est retiré du sous-réseau sous la propriété **serviceName** :

```azurecli-interactive
  az network vnet subnet show \
  --resource-group myResourceGroup \
  --name mySubnet \
  --vnet-name myVnet \
  --query delegations
```
La sortie de la commande est un crochet NULL :
```json
[]
```

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="connect-to-azure"></a>Connexion à Azure

```azurepowershell-interactive
  Connect-AzAccount
```

### <a name="create-a-resource-group"></a>Créer un groupe de ressources
Créez un groupe de ressources avec [New-AzResourceGroup](https://docs.microsoft.com/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurepowershell-interactive
  New-AzResourceGroup -Name myResourceGroup -Location eastus
```
### <a name="create-virtual-network"></a>Création d’un réseau virtuel

Créez un réseau virtuel nommé **myVnet** avec un sous-réseau nommé **mySubnet** à l’aide de [New-AzVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetworksubnetconfig?view=latest) dans **myResourceGroup** à l’aide de [New-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetwork?view=latest). L’espace d’adressage IP pour le réseau virtuel est **10.0.0.0/16**. Un sous-réseau au sein du réseau virtuel est **10.0.0.0/24**.  

```azurepowershell-interactive
  $subnet = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix "10.0.0.0/24"

  New-AzVirtualNetwork -Name myVnet -ResourceGroupName myResourceGroup -Location eastus -AddressPrefix "10.0.0.0/16" -Subnet $subnet
```
### <a name="permissions"></a>Autorisations

Si vous n’avez pas créé le sous-réseau que vous souhaitez déléguer à un service Azure, vous devez disposer de l’autorisation suivante : `Microsoft.Network/virtualNetworks/subnets/write`.

Le rôle intégré [Contributeur réseau](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) inclut également les autorisations nécessaires.

### <a name="delegate-a-subnet-to-an-azure-service"></a>Déléguer un sous-réseau à un service Azure

Dans le cadre de cette section, vous allez déléguer le sous-réseau que vous avez créé à la section précédente à un service Azure. 

Utilisez [Add-AzDelegation](https://docs.microsoft.com/powershell/module/az.network/add-azdelegation?view=latest) pour mettre à jour le sous-réseau nommé **mySubnet** avec une délégation nommée **myDelegation** à un service Azure.  Dans cet exemple **Microsoft.DBforPostgreSQL/serversv2** est utilisé pour l’exemple de délégation :

```azurepowershell-interactive
  $vnet = Get-AzVirtualNetwork -Name "myVNet" -ResourceGroupName "myResourceGroup"
  $subnet = Get-AzVirtualNetworkSubnetConfig -Name "mySubnet" -VirtualNetwork $vnet
  $subnet = Add-AzDelegation -Name "myDelegation" -ServiceName "Microsoft.DBforPostgreSQL/serversv2" -Subnet $subnet
  Set-AzVirtualNetwork -VirtualNetwork $vnet
```
Utilisez [Get-AzDelegation](https://docs.microsoft.com/powershell/module/az.network/get-azdelegation?view=latest) pour vérifier la délégation :

```azurepowershell-interactive
  $subnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup" | Get-AzVirtualNetworkSubnetConfig -Name "mySubnet"
  Get-AzDelegation -Name "myDelegation" -Subnet $subnet

  ProvisioningState : Succeeded
  ServiceName       : Microsoft.DBforPostgreSQL/serversv2
  Actions           : {Microsoft.Network/virtualNetworks/subnets/join/action}
  Name              : myDelegation
  Etag              : W/"9cba4b0e-2ceb-444b-b553-454f8da07d8a"
  Id                : /subscriptions/3bf09329-ca61-4fee-88cb-7e30b9ee305b/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet/delegations/myDelegation

```
### <a name="remove-subnet-delegation-from-an-azure-service"></a>Supprimer une délégation de sous-réseau d’un service Azure

Utilisez [Remove-AzDelegation](https://docs.microsoft.com/powershell/module/az.network/remove-azdelegation?view=latest) pour supprimer la délégation du sous-réseau nommé **mySubnet** :

```azurepowershell-interactive
  $vnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup"
  $subnet = Get-AzVirtualNetworkSubnetConfig -Name "mySubnet" -VirtualNetwork $vnet
  $subnet = Remove-AzDelegation -Name "myDelegation" -Subnet $subnet
  Set-AzVirtualNetwork -VirtualNetwork $vnet
```
Utilisez [Get-AzDelegation](https://docs.microsoft.com/powershell/module/az.network/get-azdelegation?view=latest) pour vérifier que la délégation a été supprimée :

```azurepowershell-interactive
  $subnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup" | Get-AzVirtualNetworkSubnetConfig -Name "mySubnet"
  Get-AzDelegation -Name "myDelegation" -Subnet $subnet

  Get-AzDelegation: Sequence contains no matching element

```

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment [gérer les sous-réseaux dans Azure](virtual-network-manage-subnet.md).
