---
title: Connecter des réseaux virtuels par Peering VNet – Azure CLI
description: Dans cet article, vous apprendrez à connecter des réseaux virtuels à l’aide du peering de réseaux virtuels en utilisant Azure CLI.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
tags: azure-resource-manager
Customer intent: I want to connect two virtual networks so that virtual machines in one virtual network can communicate with virtual machines in the other virtual network.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9c6399e437fa314aa82e0b41cbf8a170ea3ab72e
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94741771"
---
# <a name="connect-virtual-networks-with-virtual-network-peering-using-the-azure-cli"></a>Connecter des réseaux virtuels à l’aide du peering de réseaux virtuels en utilisant Azure CLI

Vous pouvez connecter des réseaux virtuels entre eux à l’aide du peering de réseaux virtuels. Une fois que les deux réseaux virtuels sont appairés, leurs ressources peuvent communiquer entre elles avec les mêmes bande passante et latence, comme si elles se trouvaient sur le même réseau virtuel. Dans cet article, vous apprendrez comment :

* Créer deux réseaux virtuels
* Connecter deux réseaux virtuels à l’aide du peering de réseaux virtuels
* Déployer une machine virtuelle sur chaque réseau virtuel
* Établir une communication entre les machines virtuelles

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.28 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-virtual-networks"></a>Créer des réseaux virtuels

Avant de créer un réseau virtuel, vous devez créer un groupe de ressources pour le réseau virtuel et toutes les autres ressources créées dans cet article. Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus*.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

Créez un réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet). L’exemple suivant permet de créer un réseau virtuel nommé *myVirtualNetwork1* avec le préfixe d’adresse *10.0.0.0/16*.

```azurecli-interactive 
az network vnet create \
  --name myVirtualNetwork1 \
  --resource-group myResourceGroup \
  --address-prefixes 10.0.0.0/16 \
  --subnet-name Subnet1 \
  --subnet-prefix 10.0.0.0/24
```

Créez un réseau virtuel nommé *myVirtualNetwork2* avec le préfixe d’adresse *10.1.0.0/16* :

```azurecli-interactive 
az network vnet create \
  --name myVirtualNetwork2 \
  --resource-group myResourceGroup \
  --address-prefixes 10.1.0.0/16 \
  --subnet-name Subnet1 \
  --subnet-prefix 10.1.0.0/24
```

## <a name="peer-virtual-networks"></a>Appairer des réseaux virtuels

Les peerings sont établis entre les ID des réseaux virtuels, vous devez donc d’abord obtenir l’ID de chaque réseau virtuel à l’aide d’[az network vnet show](/cli/azure/network/vnet) et stocker l’ID dans une variable.

```azurecli-interactive
# Get the id for myVirtualNetwork1.
vNet1Id=$(az network vnet show \
  --resource-group myResourceGroup \
  --name myVirtualNetwork1 \
  --query id --out tsv)

# Get the id for myVirtualNetwork2.
vNet2Id=$(az network vnet show \
  --resource-group myResourceGroup \
  --name myVirtualNetwork2 \
  --query id \
  --out tsv)
```

Créez un appairage entre *myVirtualNetwork1* et *myVirtualNetwork2* à l’aide d’[az network vnet peering create](/cli/azure/network/vnet/peering). Si le paramètre `--allow-vnet-access` n’est pas spécifié, un peering est établi, mais aucune communication ne peut passer.

```azurecli-interactive
az network vnet peering create \
  --name myVirtualNetwork1-myVirtualNetwork2 \
  --resource-group myResourceGroup \
  --vnet-name myVirtualNetwork1 \
  --remote-vnet $vNet2Id \
  --allow-vnet-access
```

Dans la sortie obtenue après l’exécution de la commande précédente, vous constatez que le **peeringState** (état de l’appairage) est *Initiated*. Le peering reste dans l’état *Initiated* jusqu’à ce que vous créiez le peering entre *myVirtualNetwork2* et *myVirtualNetwork1*. Créez un peering entre *myVirtualNetwork2* et *myVirtualNetwork1*. 

```azurecli-interactive
az network vnet peering create \
  --name myVirtualNetwork2-myVirtualNetwork1 \
  --resource-group myResourceGroup \
  --vnet-name myVirtualNetwork2 \
  --remote-vnet $vNet1Id \
  --allow-vnet-access
```

Dans la sortie obtenue après l’exécution de la commande précédente, vous constatez que le **peeringState** (état de l’appairage) est *Connected*. Azure a également changé l’état du peering *myVirtualNetwork1-myVirtualNetwork2* en *Connected*. Confirmez que l’état de l’appairage de l’appairage *myVirtualNetwork1-myVirtualNetwork2* a été changé en *Connected* à l’aide de [az network vnet peering show](/cli/azure/network/vnet/peering).

```azurecli-interactive
az network vnet peering show \
  --name myVirtualNetwork1-myVirtualNetwork2 \
  --resource-group myResourceGroup \
  --vnet-name myVirtualNetwork1 \
  --query peeringState
```

Les ressources dans un réseau virtuel ne peuvent pas communiquer avec les ressources dans un autre réseau virtuel avant que le **peeringState** pour les appairages dans les deux réseaux virtuels soit *Connected*. 

## <a name="create-virtual-machines"></a>Créer des machines virtuelles

Créez une machine virtuelle sur chaque réseau virtuel afin de pouvoir établir une communication entre elles dans une étape ultérieure.

### <a name="create-the-first-vm"></a>Créer la première machine virtuelle

Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm). L’exemple suivant crée une machine virtuelle nommée *myVm1* dans le réseau virtuel *myVirtualNetwork1*. Si des clés SSH n’existent pas déjà dans un emplacement de clé par défaut, la commande les crée. Pour utiliser un ensemble spécifique de clés, utilisez l’option `--ssh-key-value`. L’option `--no-wait` crée la machine virtuelle en arrière-plan. Vous pouvez donc passer à l’étape suivante.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm1 \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork1 \
  --subnet Subnet1 \
  --generate-ssh-keys \
  --no-wait
```

### <a name="create-the-second-vm"></a>Créer la seconde machine virtuelle

Créez une machine virtuelle dans le réseau virtuel *myVirtualNetwork2*.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm2 \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork2 \
  --subnet Subnet1 \
  --generate-ssh-keys
```

La création de la machine virtuelle ne nécessite que quelques minutes. Une fois la machine virtuelle créée, l’interface CLI Azure affiche des informations similaires à celles de l’exemple suivant : 

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVm2",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.1.0.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```

Veuillez noter **publicIpAddress**. Cette adresse sera utilisée pour accéder à la machine virtuelle à partir d’Internet dans une prochaine étape.

## <a name="communicate-between-vms"></a>Établir une communication entre les machines virtuelles

Utilisez la commande suivante pour créer une session SSH avec la machine virtuelle *myVm2*. Remplacez `<publicIpAddress>` par l’adresse IP publique de votre machine virtuelle. Dans l’exemple précédent, l’adresse IP publique est *13.90.242.231*.

```bash
ssh <publicIpAddress>
```

Effectuez un test ping pour la machine virtuelle située dans le réseau *myVirtualNetwork1*.

```bash
ping 10.0.0.4 -c 4
```

Vous recevez quatre réponses. 

Fermez la session SSH sur la machine virtuelle *myVm2*. 

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, utilisez [az group delete](/cli/azure/group) pour le supprimer, ainsi que toutes les ressources qu’il contient.

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à connecter deux réseaux situés dans la même région Azure à l’aide du peering de réseaux virtuels. Vous pouvez également appairer des réseaux virtuels situés dans des [régions différentes](virtual-network-manage-peering.md#cross-region) et dans des [abonnements Azure différents](create-peering-different-subscriptions.md#cli). Vous pouvez aussi créer des [conceptions réseau hub-and-spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke#virtual-network-peering) avec le peering. Pour en savoir plus sur le peering de réseaux virtuels, consultez [Aperçu de présentation du peering de réseaux virtuels](virtual-network-peering-overview.md) et [Gérer les peerings de réseau virtuels](virtual-network-manage-peering.md).

Vous pouvez [connecter votre propre ordinateur à un réseau virtuel](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) via un VPN et interagir avec les ressources dans un réseau virtuel, ou dans des réseaux virtuels appairés. Consultez les [exemples de script](cli-samples.md) pour obtenir des scripts réutilisables permettant d’accomplir un grand nombre des tâches présentées dans les articles sur les réseaux virtuels.
