---
title: Dissocier une IP publique d’une machine virtuelle Azure
titlesuffix: Azure Virtual Network
description: Découvrez comment dissocier une IP publique d’une machine virtuelle
services: virtual-network
documentationcenter: ''
author: asudbring
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/04/2019
ms.author: allensu
ms.openlocfilehash: b171699a0c578b3761e58f6e0e977199369864a8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84709961"
---
# <a name="dissociate-a-public-ip-address-from-an-azure-vm"></a>Dissocier une IP publique d’une machine virtuelle Azure 

Dans cet article, vous allez apprendre à dissocier une IP publique d’une machine virtuelle Azure.

Vous pouvez utiliser le [Portail Azure](#azure-portal), l’[interface de ligne de commande](#azure-cli) (CLI) Azure ou [PowerShell](#powershell) pour dissocier une IP publique d’une machine virtuelle.

## <a name="azure-portal"></a>Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Accédez à la machine virtuelle que vous souhaitez dissocier d’une IP publique ou recherchez-la, puis sélectionnez-la.
3. Dans la page Machine virtuelle, sélectionnez **Vue d’ensemble**, sélectionnez l’IP publique, comme indiqué dans l’image suivante :

   ![Sélectionner l’IP publique](./media/remove-public-ip-address/remove-public-ip-address-2.png)

4. Dans la page IP publique, sélectionnez **Vue d’ensemble**, puis **Dissocier**, comme indiqué dans l’image suivante :

    ![Dissocier l’IP publique](./media/remove-public-ip-address/remove-public-ip-address-3.png)

5. Dans **Dissocier l’IP publique**, sélectionnez **Oui**.

## <a name="azure-cli"></a>Azure CLI

Installez l’[interface de ligne de commande Azure](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json) ou utilisez Azure Cloud Shell. Azure Cloud Shell est un interpréteur de commandes Bash gratuit, que vous pouvez exécuter directement dans le portail Azure. L’interface Azure CLI est préinstallée et configurée pour être utilisée avec votre compte. Sélectionnez le bouton **Essayer** dans les commandes de l’interface de ligne de commande qui suit. En sélectionnant **Essayer**, vous appelez un Cloud Shell avec lequel vous pouvez vous connecter à votre compte Azure.

1. Si vous utilisez l’interface de ligne de commande localement dans Bash, connectez-vous à Azure à l’aide de `az login`.
2. Une adresse IP publique est associée à une configuration IP d’une interface réseau jointe à une machine virtuelle. Utilisez la commande [az network nic-ip-config update](/cli/azure/network/nic/ip-config?view=azure-cli-latest#az-network-nic-ip-config-update) pour dissocier une IP publique d’une configuration IP. L’exemple suivant dissocie une IP publique nommée *myVMPublicIP* de la configuration IP nommée *ipconfigmyVM* d’une interface réseau existante nommée *myVMVMNic* qui est attachée à une machine virtuelle nommée *myVM* dans un groupe de ressources nommé *myResourceGroup*.
  
   ```azurecli-interactive
    az network nic ip-config update \
    --name ipconfigmyVM \
    --resource-group myResourceGroup \
    --nic-name myVMVMNic \
    --remove PublicIpAddress
   ```

   Si vous ne connaissez pas le nom d’une interface réseau jointe à votre machine virtuelle, utilisez la commande [az vm nic list](/cli/azure/vm/nic?view=azure-cli-latest#az-vm-nic-list) pour les afficher. Par exemple, la commande suivante répertorie les noms des interfaces réseau jointes à une machine virtuelle nommée *myVM* dans un groupe de ressources nommé *myResourceGroup*:

     ```azurecli-interactive
     az vm nic list --vm-name myVM --resource-group myResourceGroup
     ```

     La sortie inclut une ou plusieurs lignes qui sont similaires à l’exemple suivant :
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
     ```

     Dans l’exemple précédent, *myVMVMNic* est le nom de l’interface réseau.

   - Si vous ne connaissez pas le nom d’une configuration IP pour une interface réseau, utilisez la commande [az network nic ip-config list](/cli/azure/network/nic/ip-config?view=azure-cli-latest#az-network-nic-ip-config-list) pour les récupérer. Par exemple, la commande suivante répertorie les noms des configurations d’IP publiques d’une interface réseau nommée *myVMVMNic* dans un groupe de ressources nommé *myResourceGroup* :

     ```azurecli-interactive
     az network nic ip-config list --nic-name myVMVMNic --resource-group myResourceGroup --out table
     ```

   - Si vous ne connaissez pas le nom d’une configuration d’IP publique pour une interface réseau, utilisez la commande [az network nic ip-config show](/cli/azure/network/nic/ip-config?view=azure-cli-latest#az-network-nic-ip-config-show) pour les récupérer. Par exemple, la commande suivante répertorie les noms des configurations d’IP publiques d’une interface réseau nommée *myVMVMNic* dans un groupe de ressources nommé *myResourceGroup* :

     ```azurecli-interactive
     az network nic ip-config show --name ipconfigmyVM --nic-name myVMVMNic --resource-group myResourceGroup --query publicIPAddress.id
     ```


## <a name="powershell"></a>PowerShell

Installez [PowerShell](/powershell/azure/install-az-ps), ou utilisez Azure Cloud Shell. Azure Cloud Shell est un interpréteur de commandes gratuit que vous pouvez exécuter directement dans le portail Azure. PowerShell y est préinstallé et configuré pour être utilisé avec votre compte. Sélectionnez le bouton **Essayer** dans les commandes PowerShell qui suivent. En sélectionnant **Essayer**, vous appelez un Cloud Shell avec lequel vous pouvez vous connecter à votre compte Azure.

1. Si vous utilisez PowerShell localement, connectez-vous à Azure avec `Connect-AzAccount`.
2. Une adresse IP publique est associée à une configuration IP d’une interface réseau jointe à une machine virtuelle. Utilisez la commande [Get-AzNetworkInterface](/powershell/module/Az.Network/Get-AzNetworkInterface) pour obtenir une interface réseau. Définissez la valeur de l’IP publique sur Null, puis utilisez la commande [Set-AzNetworkInterface](/powershell/module/Az.Network/Set-AzNetworkInterface) pour écrire la nouvelle configuration IP sur l’interface réseau.

   L’exemple suivant dissocie une IP publique nommée *myVMPublicIP* d’une interface réseau nommée *myVMVMNic* attachée à une machine virtuelle nommée *myVM*. Toutes les ressources se trouvent dans un groupe de ressources nommé *myResourceGroup*.
  
   ```azurepowershell
    $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroup myResourceGroup
    $nic.IpConfigurations.publicipaddress.id = $null
    Set-AzNetworkInterface -NetworkInterface $nic
   ```

  - Si vous ne connaissez pas le nom d’une interface réseau jointe à votre machine virtuelle, utilisez la commande [Get-AzVM](/powershell/module/Az.Compute/Get-AzVM) pour les afficher. Par exemple, la commande suivante répertorie les noms des interfaces réseau jointes à une machine virtuelle nommée *myVM* dans un groupe de ressources nommé *myResourceGroup*:

    ```azurepowershell
    $vm = Get-AzVM -name myVM -ResourceGroupName myResourceGroup
    $vm.NetworkProfile
    ```

     La sortie inclut une ou plusieurs lignes qui sont similaires à l’exemple qui suit. Dans l’exemple de sortie, *myVMVMNic* est le nom de l’interface réseau.
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic",
     ```

   - Si vous ne connaissez pas le nom d’une configuration IP pour une interface réseau, utilisez la commande [Get-AzNetworkInterface](/powershell/module/Az.Network/Get-AzNetworkInterface) pour les récupérer. Par exemple, la commande suivante répertorie les noms des configurations IP d’une interface réseau nommée *myVMVMNic* dans un groupe de ressources nommé *myResourceGroup*:

     ```azurepowershell-interactive
     $nic = Get-AzNetworkInterface -Name myVMVMNic -ResourceGroupName myResourceGroup
     $nic.IPConfigurations.id
     ```

     La sortie inclut une ou plusieurs lignes qui sont similaires à l’exemple qui suit. Dans l’exemple de sortie, *ipconfigmyVM* est le nom d’une configuration IP.
  
     ```
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myVMVMNic/ipConfigurations/ipconfigmyVM"
     ```

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [associer une IP publique à une machine virtuelle](associate-public-ip-address-vm.md).
