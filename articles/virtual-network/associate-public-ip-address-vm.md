---
title: Associer une adresse IP publique à une machine virtuelle
titlesuffix: Azure Virtual Network
description: Apprenez à associer une IP publique à une machine virtuelle à l’aide du portail Azure ou de l’interface de ligne de commande Azure.
services: virtual-network
documentationcenter: ''
author: asudbring
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/21/2019
ms.author: allensu
ms.openlocfilehash: 9b5fc9e4118d98905138b7f205f61d85a96b60b0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88035465"
---
# <a name="associate-a-public-ip-address-to-a-virtual-machine"></a>Associer une adresse IP publique à une machine virtuelle

Dans cet article, vous allez apprendre à associer une adresse IP publique à une machine virtuelle (VM). Si vous souhaitez vous connecter à une machine virtuelle à partir d’internet, la machine virtuelle doit être associée à une adresse IP publique. Si vous souhaitez créer une machine virtuelle avec une adresse IP publique, vous pouvez le faire en utilisant le [Portail Microsoft Azure](virtual-network-deploy-static-pip-arm-portal.md), l’[interface de ligne de commande Azure](virtual-network-deploy-static-pip-arm-cli.md), ou [PowerShell](virtual-network-deploy-static-pip-arm-ps.md). Les adresses IP publiques ont un coût nominal. Pour en savoir plus, consultez les [tarifs](https://azure.microsoft.com/pricing/details/ip-addresses/). Le nombre d’adresses IP publiques que vous pouvez utiliser dans un abonnement est limité. Pour plus d’informations, consultez la section [Limites](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#publicip-address).

Vous pouvez utiliser le [Portail Microsoft Azure](#azure-portal), l’[interface de ligne de commande Azure](#azure-cli), ou [PowerShell](#powershell) pour associer une adresse IP publique à une machine virtuelle.

## <a name="azure-portal"></a>Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Accédez à la machine virtuelle à laquelle vous voulez ajouter une adresse IP publique ou recherchez-la, puis sélectionnez-la.
3. Sous **Paramètres**, sélectionnez **Mise en réseau**, puis sélectionnez l’interface réseau à laquelle vous souhaitez ajouter l’adresse IP publique, comme illustré dans l’image suivante :

   ![Sélectionner l’interface réseau](./media/associate-public-ip-address-vm/select-nic.png)

   > [!NOTE]
   > Les adresses IP publiques sont associés à des interfaces réseau jointes à une machine virtuelle. Dans l’image précédente, la machine virtuelle a uniquement une interface réseau. Si la machine virtuelle a plusieurs interfaces réseau, elles doivent toutes apparaître, puis vous sélectionnez l’interface réseau à laquelle vous souhaitez associer l’adresse IP publique.

4. Sélectionnez **Configurations IP**, puis sélectionnez une configuration IP, comme indiqué dans l’image suivante :

   ![Sélectionner une configuration IP](./media/associate-public-ip-address-vm/select-ip-configuration.png)

   > [!NOTE]
   > Les adresses IP publiques sont associés à des configurations IP pour une interface réseau. Dans l’image précédente, l’interface réseau a une configuration IP. Si l’interface réseau dispose de plusieurs configurations IP, elles doivent toutes apparaître dans la liste, puis vous sélectionnez la configuration IP à laquelle vous souhaitez associer l’adresse IP publique.

5. Sélectionnez **Activées**, puis **Adresse IP (*Configurer les paramètres requis*)** . Choisissez une adresse IP publique existante. Cela ferme automatiquement la boîte de dialogue **Choisir une adresse IP publique**. Si aucune adresse IP publique disponible n’est répertoriée, vous devez en créer une. Pour savoir comment procéder, consultez la section [Créer une adresse IP publique](virtual-network-public-ip-address.md#create-a-public-ip-address). Sélectionnez **enregistrer**, comme illustré dans l’image qui suit, puis fermez la boîte de dialogue de configuration IP.

   ![Activer l’adresse IP publique](./media/associate-public-ip-address-vm/enable-public-ip-address.png)

   > [!NOTE]
   > Les adresses IP publiques affichées sont celles qui existent dans la même région que la machine virtuelle. Si votre région comprend plusieurs adresses IP publiques, elles sont affichées ici. Si l’une d’elle est grisée, c’est parce que l’adresse est déjà associée à une autre ressource.

6. Affichez l’adresse IP publique attribuée à la configuration IP, comme indiqué dans l’image qui suit. L’affichage d’une adresse IP peut prendre quelques secondes.

   ![Afficher les adresses IP publiques attribuées](./media/associate-public-ip-address-vm/view-assigned-public-ip-address.png)

   > [!NOTE]
   > L’adresse est attribuée à partir d’un pool d’adresses utilisées dans chaque région Azure. Pour obtenir une liste des pools d’adresses utilisées dans chaque région, consultez l’article [Plages d’adresses IP de centre de données Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=41653). L’adresse attribuée peut être n’importe quelle adresse dans les pools utilisés pour la région. Si vous devez attribuer une adresse à partir d’un pool spécifique dans la région, utilisez un [Préfixe d’adresse IP publique](public-ip-address-prefix.md).

7. [Autorisez le trafic réseau vers la machine virtuelle](#allow-network-traffic-to-the-vm) avec les règles de sécurité dans un groupe de sécurité réseau.

## <a name="azure-cli"></a>Azure CLI

Installez l’[interface de ligne de commande Azure](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json) ou utilisez Azure Cloud Shell. Azure Cloud Shell est un interpréteur de commandes Bash gratuit, que vous pouvez exécuter directement dans le portail Azure. L’interface Azure CLI est préinstallée et configurée pour être utilisée avec votre compte. Sélectionnez le bouton **Essayer** dans les commandes de l’interface de ligne de commande qui suit. En sélectionnant **Essayer**, vous appelez un Cloud Shell avec lequel vous pouvez vous connecter à votre compte Azure.

1. Si vous utilisez l’interface de ligne de commande localement dans Bash, connectez-vous à Azure à l’aide de `az login`.
2. Une adresse IP publique est associée à une configuration IP d’une interface réseau jointe à une machine virtuelle. Utilisez la commande [az network nic-ip-config update](/cli/azure/network/nic/ip-config?view=azure-cli-latest#az-network-nic-ip-config-update) pour associer une adresse IP publique à une configuration IP. L’exemple suivant associe une adresse IP publique existante nommée *myVMPublicIP* à la configuration IP nommée *ipconfigmyVM* d’une interface réseau existante nommée *myVMVMNic* qui existe dans un groupe de ressources nommé *myResourceGroup*.
  
   ```azurecli-interactive
   az network nic ip-config update \
     --name ipconfigmyVM \
     --nic-name myVMVMNic \
     --resource-group myResourceGroup \
     --public-ip-address myVMPublicIP
   ```

   - Si vous n’avez pas d’adresse IP publique, utilisez la commande [az network public-ip create](/cli/azure/network/public-ip?view=azure-cli-latest#az-network-public-ip-create) pour en créer une. Par exemple, la commande suivante crée une adresse IP publique nommée *myVMPublicIP* dans un groupe de ressources nommé *myResourceGroup*.
  
     ```azurecli-interactive
     az network public-ip create --name myVMPublicIP --resource-group myResourceGroup
     ```

     > [!NOTE]
     > La commande précédente crée une adresse IP publique avec les valeurs par défaut pour plusieurs paramètres que vous pouvez personnaliser. Pour en savoir plus sur tous les paramètres d’adresse IP publique, consultez [Créer une adresse IP publique](virtual-network-public-ip-address.md#create-a-public-ip-address). L’adresse est attribuée à partir d’un pool d’adresses IP publiques utilisées pour chaque région Azure. Pour obtenir une liste des pools d’adresses utilisées dans chaque région, consultez l’article [Plages d’adresses IP de centre de données Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=41653).

   - Si vous ne connaissez pas le nom d’une interface réseau jointe à votre machine virtuelle, utilisez la commande [az vm nic list](/cli/azure/vm/nic?view=azure-cli-latest#az-vm-nic-list) pour les afficher. Par exemple, la commande suivante répertorie les noms des interfaces réseau jointes à une machine virtuelle nommée *myVM* dans un groupe de ressources nommé *myResourceGroup*:

     ```azurecli-interactive
     az vm nic list --vm-name myVM --resource-group myResourceGroup
     ```

     La sortie inclut une ou plusieurs lignes qui sont similaires à l’exemple suivant :
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
     ```

     Dans l’exemple précédent, *myVMVMNic* est le nom de l’interface réseau.

   - Si vous ne connaissez pas le nom d’une configuration IP pour une interface réseau, utilisez la commande [az network nic ip-config list](/cli/azure/network/nic/ip-config?view=azure-cli-latest#az-network-nic-ip-config-list) pour les récupérer. Par exemple, la commande suivante répertorie les noms des configurations IP d’une interface réseau nommée *myVMVMNic* dans un groupe de ressources nommé *myResourceGroup*:

     ```azurecli-interactive
     az network nic ip-config list --nic-name myVMVMNic --resource-group myResourceGroup --out table
     ```

3. Affichez l’adresse IP publique attribuée à la configuration IP avec la commande [az vm list-ip-addresses](/cli/azure/vm?view=azure-cli-latest#az-vm-list-ip-addresses). L’exemple suivant montre les adresses IP attribuées à une machine virtuelle existante nommée *myVM* dans un groupe de ressources nommé *myResourceGroup*.

   ```azurecli-interactive
   az vm list-ip-addresses --name myVM --resource-group myResourceGroup --out table
   ```

   > [!NOTE]
   > L’adresse est attribuée à partir d’un pool d’adresses utilisées dans chaque région Azure. Pour obtenir une liste des pools d’adresses utilisées dans chaque région, consultez l’article [Plages d’adresses IP de centre de données Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=41653). L’adresse attribuée peut être n’importe quelle adresse dans les pools utilisés pour la région. Si vous devez attribuer une adresse à partir d’un pool spécifique dans la région, utilisez un [Préfixe d’adresse IP publique](public-ip-address-prefix.md).

4. [Autorisez le trafic réseau vers la machine virtuelle](#allow-network-traffic-to-the-vm) avec les règles de sécurité dans un groupe de sécurité réseau.

## <a name="powershell"></a>PowerShell

Installez [PowerShell](/powershell/azure/install-az-ps), ou utilisez Azure Cloud Shell. Azure Cloud Shell est un interpréteur de commandes gratuit que vous pouvez exécuter directement dans le portail Azure. PowerShell y est préinstallé et configuré pour être utilisé avec votre compte. Sélectionnez le bouton **Essayer** dans les commandes PowerShell qui suivent. En sélectionnant **Essayer**, vous appelez un Cloud Shell avec lequel vous pouvez vous connecter à votre compte Azure.

1. Si vous utilisez PowerShell localement, connectez-vous à Azure avec `Connect-AzAccount`.
2. Une adresse IP publique est associée à une configuration IP d’une interface réseau jointe à une machine virtuelle. Utilisez les commandes [Get-AzVirtualNetwork](/powershell/module/Az.Network/Get-AzVirtualNetwork) et [Get-AzVirtualNetworkSubnetConfig](/powershell/module/Az.Network/Get-AzVirtualNetworkSubnetConfig) pour obtenir le réseau virtuel et le sous-réseau figurant dans l’interface réseau. Ensuite, utilisez la commande [Get-AzNetworkInterface](/powershell/module/Az.Network/Get-AzNetworkInterface) pour obtenir une interface réseau et la commande [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) pour obtenir une adresse IP publique existante. Puis utilisez la commande [Set-AzNetworkInterfaceIpConfig](/powershell/module/Az.Network/Set-AzNetworkInterfaceIpConfig) pour associer l’adresse IP publique à la configuration IP et la commande [AzNetworkInterface de jeu](/powershell/module/Az.Network/Set-AzNetworkInterface) pour écrire la nouvelle configuration IP dans l’interface réseau.

   L’exemple suivant associe une adresse IP publique existante nommée *myVMPublicIP* à la configuration IP nommée *ipconfigmyVM* d’une interface réseau nommée *myVMVMNic* qui existe dans un sous-réseau nommé *myVMSubnet* dans un réseau virtuel nommé *myVMVNet*. Toutes les ressources se trouvent dans un groupe de ressources nommé *myResourceGroup*.
  
   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name myVMVNet -ResourceGroupName myResourceGroup
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name myVMSubnet -VirtualNetwork $vnet
   $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
   $pip = Get-AzPublicIpAddress -Name myVMPublicIP -ResourceGroupName myResourceGroup
   $nic | Set-AzNetworkInterfaceIpConfig -Name ipconfigmyVM -PublicIPAddress $pip -Subnet $subnet
   $nic | Set-AzNetworkInterface
   ```

   - Si vous n’avez pas déjà une adresse IP publique, utilisez la commande [New-AzPublicIpAddress](/powershell/module/Az.Network/New-AzPublicIpAddress) pour en créer une. Par exemple, la commande suivante crée une adresse IP publique *dynamique* nommée *myVMPublicIP* dans un groupe de ressources nommé *myResourceGroup* dans la région *eastus* (USA Est).
  
     ```azurepowershell-interactive
     New-AzPublicIpAddress -Name myVMPublicIP -ResourceGroupName myResourceGroup -AllocationMethod Dynamic -Location eastus
     ```

     > [!NOTE]
     > La commande précédente crée une adresse IP publique avec les valeurs par défaut pour plusieurs paramètres que vous pouvez personnaliser. Pour en savoir plus sur tous les paramètres d’adresse IP publique, consultez [Créer une adresse IP publique](virtual-network-public-ip-address.md#create-a-public-ip-address). L’adresse est attribuée à partir d’un pool d’adresses IP publiques utilisées pour chaque région Azure. Pour obtenir une liste des pools d’adresses utilisées dans chaque région, consultez l’article [Plages d’adresses IP de centre de données Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=41653).

   - Si vous ne connaissez pas le nom d’une interface réseau jointe à votre machine virtuelle, utilisez la commande [Get-AzVM](/powershell/module/Az.Compute/Get-AzVM) pour les afficher. Par exemple, la commande suivante répertorie les noms des interfaces réseau jointes à une machine virtuelle nommée *myVM* dans un groupe de ressources nommé *myResourceGroup*:

     ```azurepowershell-interactive
     $vm = Get-AzVM -name myVM -ResourceGroupName myResourceGroup
     $vm.NetworkProfile
     ```

     La sortie inclut une ou plusieurs lignes qui sont similaires à l’exemple qui suit. Dans l’exemple de sortie, *myVMVMNic* est le nom de l’interface réseau.
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
     ```

   - Si vous ne connaissez pas le nom du réseau virtuel ou sous-réseau figurant dans l’interface réseau, utilisez la commande `Get-AzNetworkInterface` pour afficher cette information. Par exemple, la commande suivante obtient des informations sur le sous-réseau et le réseau virtuels d’une interface réseau nommée *myVMVMNic* dans un groupe de ressources nommé *myResourceGroup* :

     ```azurepowershell-interactive
     $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
     $ipConfigs = $nic.IpConfigurations
     $ipConfigs.Subnet | Select Id
     ```

     La sortie inclut une ou plusieurs lignes qui sont similaires à l’exemple qui suit. Dans l’exemple de sortie, *myVMVNET* est le nom du réseau virtuel et *myVMSubnet* est le nom du sous-réseau.
  
     ```
     "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVMVNET/subnets/myVMSubnet",
     ```

   - Si vous ne connaissez pas le nom d’une configuration IP pour une interface réseau, utilisez la commande [Get-AzNetworkInterface](/powershell/module/Az.Network/Get-AzNetworkInterface) pour les récupérer. Par exemple, la commande suivante répertorie les noms des configurations IP d’une interface réseau nommée *myVMVMNic* dans un groupe de ressources nommé *myResourceGroup*:

     ```azurepowershell-interactive
     $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
     $nic.IPConfigurations
     ```

     La sortie inclut une ou plusieurs lignes qui sont similaires à l’exemple qui suit. Dans l’exemple de sortie, *ipconfigmyVM* est le nom d’une configuration IP.
  
     ```
     Id     : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic/ipConfigurations/ipconfigmyVM
     ```

3. Affichez l’adresse IP publique attribuée à la configuration IP avec la commande [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress). L’exemple suivant montre l’adresse attribuée à une adresse IP publique nommée *myVMPublicIP* dans un groupe de ressources nommé *myResourceGroup*.

   ```azurepowershell-interactive
   Get-AzPublicIpAddress -Name myVMPublicIP -ResourceGroupName myResourceGroup | Select IpAddress
   ```

   Si vous ne connaissez pas le nom de l’adresse IP publique attribuée à une configuration IP, exécutez les commandes suivantes pour l’obtenir :

   ```azurepowershell-interactive
   $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
   $nic.IPConfigurations
   $address = $nic.IPConfigurations.PublicIpAddress
   $address | Select Id
   ```

   La sortie inclut une ou plusieurs lignes qui sont similaires à l’exemple qui suit. Dans l’exemple de sortie, *myVMPublicIP* est le nom de l’adresse IP publique attribuée à la configuration IP.

   ```
   "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myVMPublicIP"
   ```

   > [!NOTE]
   > L’adresse est attribuée à partir d’un pool d’adresses utilisées dans chaque région Azure. Pour obtenir une liste des pools d’adresses utilisées dans chaque région, consultez l’article [Plages d’adresses IP de centre de données Microsoft Azure](https://www.microsoft.com/download/details.aspx?id=41653). L’adresse attribuée peut être n’importe quelle adresse dans les pools utilisés pour la région. Si vous devez attribuer une adresse à partir d’un pool spécifique dans la région, utilisez un [Préfixe d’adresse IP publique](public-ip-address-prefix.md).

4. [Autorisez le trafic réseau vers la machine virtuelle](#allow-network-traffic-to-the-vm) avec les règles de sécurité dans un groupe de sécurité réseau.

## <a name="allow-network-traffic-to-the-vm"></a>Autoriser le trafic réseau vers la machine virtuelle

Avant de connecter à l’adresse IP publique à partir d’Internet, vérifiez que les ports nécessaires sont ouverts dans tout groupe de sécurité réseau que vous avez associé à l’interface réseau, au sous-réseau dans lequel se trouve l’interface réseau ou aux deux. Bien que les groupes de sécurité filtrent le trafic vers l’adresse IP privée de l’interface réseau, une fois que le trafic Internet entrant arrive à l’adresse IP publique, Azure traduit l’adresse publique vers l’adresse IP privée, donc si un groupe de sécurité réseau empêche le flux du trafic, la communication avec l’adresse IP publique échoue. Vous pouvez afficher les règles de sécurité effectives pour une interface réseau et de son sous-réseau à l’aide du [Portail Microsoft Azure](diagnose-network-traffic-filter-problem.md#diagnose-using-azure-portal), de [l’interface de ligne de commande](diagnose-network-traffic-filter-problem.md#diagnose-using-azure-cli) ou de [PowerShell](diagnose-network-traffic-filter-problem.md#diagnose-using-powershell).

## <a name="next-steps"></a>Étapes suivantes

Autorisez le trafic internet entrant vers votre machine virtuelle avec un groupe de sécurité réseau. Pour en savoir plus sur la création d’un groupe de sécurité réseau, consultez [Utiliser des groupes de sécurité réseau](manage-network-security-group.md#work-with-network-security-groups). Pour en savoir plus sur les groupes de sécurité réseau, voir [Groupes de sécurité](security-overview.md).
