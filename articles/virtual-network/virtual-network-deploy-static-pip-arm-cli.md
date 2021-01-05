---
title: Créer une machine virtuelle avec une adresse IP publique statique - Azure CLI | Microsoft Docs
description: Apprenez à créer une machine virtuelle avec une adresse IP publique statique à l’aide de l’interface de ligne de commande Azure (CLI).
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
ms.assetid: 55bc21b0-2a45-4943-a5e7-8d785d0d015c
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/08/2018
ms.author: kumud
ms.openlocfilehash: 110f5ab4f0912ae264eb1dc4b649a40cc40f4e12
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84703087"
---
# <a name="create-a-virtual-machine-with-a-static-public-ip-address-using-the-azure-cli"></a>Créer une machine virtuelle avec une adresse IP publique statique à l’aide d’Azure CLI

Vous pouvez créer une machine virtuelle avec une adresse IP publique statique. Une adresse IP publique vous permet de communiquer avec une machine virtuelle à partir d’Internet. Attribuez une adresse IP publique statique, plutôt qu’une adresse dynamique, pour être certain que l’adresse ne changera jamais. Découvrez plus en détail les [adresses IP publiques statiques](virtual-network-ip-addresses-overview-arm.md#allocation-method). Pour changer le type de l’adresse IP publique attribuée à une machine virtuelle existante (de dynamique à statique), ou pour utiliser des adresses IP privées, consultez [Ajouter, modifier ou supprimer des adresses IP](virtual-network-network-interface-addresses.md). Les adresses IP publiques ont un [coût modique](https://azure.microsoft.com/pricing/details/ip-addresses) et le nombre d’adresses IP publiques que vous pouvez utiliser par abonnement est [limité](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Vous pouvez effectuer les étapes suivantes à partir de votre ordinateur local ou à l’aide d’Azure Cloud Shell. Pour pouvoir utiliser votre ordinateur local, vous devez avoir installé [Azure CLI](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json). Pour utiliser Azure Cloud Shell, sélectionnez **Essayer** dans l’angle supérieur droit de l’une des zones de commande suivantes. Cloud Shell vous connecte à Azure.

1. Si vous utilisez Cloud Shell, passez à l’étape 2. Ouvrez une session de commande et connectez-vous à Azure avec `az login`.
2. Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create). L’exemple suivant crée un groupe de ressources dans la région Azure USA Est :

   ```azurecli-interactive
   az group create --name myResourceGroup --location eastus
   ```

3. Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm#az-vm-create). L’option `--public-ip-address-allocation=static` attribue une adresse IP publique statique à la machine virtuelle. L’exemple suivant crée une machine virtuelle Ubuntu avec une adresse IP statique de base (référence SKU) nommée *myPublicIpAddress* :

   ```azurecli-interactive
   az vm create \
     --resource-group myResourceGroup \
     --name myVM \
     --image UbuntuLTS \
     --admin-username azureuser \
     --generate-ssh-keys \
     --public-ip-address myPublicIpAddress \
     --public-ip-address-allocation static
   ```

   Si l’adresse IP publique doit être une référence SKU standard, ajoutez `--public-ip-sku Standard` à la commande précédente. Découvrez plus en détail les [références SKU d’adresses IP publiques](virtual-network-ip-addresses-overview-arm.md#sku). Si la machine virtuelle doit être ajoutée au pool backend d’un équilibreur de charge Azure public, la référence SKU de l’adresse IP publique de la machine virtuelle doit correspondre à celle de l’équilibreur de charge. Pour plus d’informations, consultez [Azure Load Balancer](../load-balancer/skus.md).

4. Examinez l’adresse IP publique qui a été affectée et vérifiez qu’elle a été créée en tant qu’adresse statique de base (référence SKU) via [az network public-ip show](/cli/azure/network/public-ip#az-network-public-ip-show) :

   ```azurecli-interactive
   az network public-ip show \
     --resource-group myResourceGroup \
     --name myPublicIpAddress \
     --query [ipAddress,publicIpAllocationMethod,sku] \
     --output table
   ```

   Azure a affecté une adresse IP publique parmi les adresses utilisées dans la région dans laquelle vous avez créé la machine virtuelle. Vous pouvez télécharger la liste des plages (préfixes) pour les clouds Azure [Public](https://www.microsoft.com/download/details.aspx?id=56519), [Gouvernement américain](https://www.microsoft.com/download/details.aspx?id=57063), [Chine](https://www.microsoft.com/download/details.aspx?id=57062), et [Allemagne](https://www.microsoft.com/download/details.aspx?id=57064).

> [!WARNING]
> Ne modifiez pas les paramètres d’adresse IP dans le système d’exploitation de la machine virtuelle. Le système d’exploitation ne reconnaît pas les adresses IP publiques Azure. Même si vous avez la possibilité d’ajouter des paramètres d’adresse IP privée au système d’exploitation, nous vous déconseillons de le faire, à moins que cela soit nécessaire et pas avant d’avoir lu [Ajouter une adresse IP privée à un système d’exploitation](virtual-network-network-interface-addresses.md#private).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, vous pouvez utiliser [az group delete](/cli/azure/group#az-group-delete) pour le supprimer, ainsi que toutes les ressources qu’il contient :

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [adresses IP publiques](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses) dans Azure
- En savoir plus sur les [paramètres d’adresse IP publique](virtual-network-public-ip-address.md#create-a-public-ip-address)
- En savoir plus sur les [adresses IP privées](virtual-network-ip-addresses-overview-arm.md#private-ip-addresses) et l’affectation d’une [adresse IP privée statique](virtual-network-network-interface-addresses.md#add-ip-addresses) à une machine virtuelle Azure
- En savoir plus sur la création de machines virtuelles [Linux](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json) et [Windows](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
