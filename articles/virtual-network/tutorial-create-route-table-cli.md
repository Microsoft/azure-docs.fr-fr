---
title: Acheminer le trafic réseau - Azure CLI | Microsoft Docs
description: Dans cet article, découvrez comment acheminer le trafic réseau avec une table de routage à l’aide de l’interface de ligne de commande Azure.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
Customer intent: I want to route traffic from one subnet, to a different subnet, through a network virtual appliance.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 043d450a1b968174ad263579d39de06a296a98e4
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94741482"
---
# <a name="route-network-traffic-with-a-route-table-using-the-azure-cli"></a>Acheminer le trafic réseau avec une table de routage à l’aide de l’interface de ligne de commande Azure

Par défaut, Azure achemine automatiquement le trafic entre tous les sous-réseaux au sein d’un réseau virtuel. Vous pouvez créer vos propres itinéraires pour remplacer le routage par défaut d’Azure. La possibilité de créer des itinéraires personnalisés est utile si, par exemple, vous souhaitez router le trafic entre des sous-réseaux via une appliance virtuelle réseau (NVA). Dans cet article, vous apprendrez comment :

* Créer une table de routage
* Créer un itinéraire
* Créer un réseau virtuel comprenant plusieurs sous-réseaux
* Associer une table de routage à un sous-réseau
* Créer une appliance NVA qui route le trafic
* Déployer des machines virtuelles sur différents sous-réseaux
* Router le trafic d’un sous-réseau vers un autre via une NVA

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.28 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-route-table"></a>Créer une table de routage

Avant de pouvoir créer une table de routage, créez un groupe de ressources avec [az group create](/cli/azure/group) pour toutes les ressources créées dans cet article. 

```azurecli-interactive
# Create a resource group.
az group create \
  --name myResourceGroup \
  --location eastus
```

Créez une table de routage avec [az network route-table create](/cli/azure/network/route-table#az-network-route-table-create). L’exemple suivant crée une table de routage nommée *myRouteTablePublic*. 

```azurecli-interactive
# Create a route table
az network route-table create \
  --resource-group myResourceGroup \
  --name myRouteTablePublic
```

## <a name="create-a-route"></a>Créer un itinéraire

Créez un itinéraire dans la table de routage avec [az network route-table route create](/cli/azure/network/route-table/route#az-network-route-table-route-create). 

```azurecli-interactive
az network route-table route create \
  --name ToPrivateSubnet \
  --resource-group myResourceGroup \
  --route-table-name myRouteTablePublic \
  --address-prefix 10.0.1.0/24 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address 10.0.2.4
```

## <a name="associate-a-route-table-to-a-subnet"></a>Associer une table de routage à un sous-réseau

Avant de pouvoir associer une table de routage à un sous-réseau, vous devez créer un réseau virtuel et un sous-réseau. Créez un réseau virtuel avec un sous-réseau en utilisant la commande [az network vnet create](/cli/azure/network/vnet).

```azurecli-interactive
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --address-prefix 10.0.0.0/16 \
  --subnet-name Public \
  --subnet-prefix 10.0.0.0/24
```

Créez deux sous-réseaux supplémentaires avec [az network vnet subnet create](/cli/azure/network/vnet/subnet).

```azurecli-interactive
# Create a private subnet.
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name Private \
  --address-prefix 10.0.1.0/24

# Create a DMZ subnet.
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name DMZ \
  --address-prefix 10.0.2.0/24
```

Associez la table de routage *myRouteTablePublic* au sous-réseau *Public* avec [az network vnet subnet update](/cli/azure/network/vnet/subnet).

```azurecli-interactive
az network vnet subnet update \
  --vnet-name myVirtualNetwork \
  --name Public \
  --resource-group myResourceGroup \
  --route-table myRouteTablePublic
```

## <a name="create-an-nva"></a>Créer une NVA

Une NVA est une machine virtuelle qui exécute une fonction réseau, telle que le routage, la fonction de pare-feu ou l’optimisation WAN.

Créez une appliance virtuelle réseau dans le sous-réseau *DMZ* avec [az vm create](/cli/azure/vm). Par défaut, quand vous créez une machine virtuelle, Azure crée une adresse IP publique et l’affecte à la machine virtuelle. Le paramètre `--public-ip-address ""` indique à Azure de ne pas créer d’adresse IP publique et de ne pas l’affecter à la machine virtuelle, car celle-ci n’a pas besoin de se connecter à Internet. Si des clés SSH n’existent pas déjà dans un emplacement de clé par défaut, la commande les crée. Pour utiliser un ensemble spécifique de clés, utilisez l’option `--ssh-key-value`.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmNva \
  --image UbuntuLTS \
  --public-ip-address "" \
  --subnet DMZ \
  --vnet-name myVirtualNetwork \
  --generate-ssh-keys
```

La création de la machine virtuelle ne nécessite que quelques minutes. Ne passez pas à l’étape suivante tant qu’Azure n’a pas terminé la création de la machine virtuelle et retourné de sortie concernant la machine virtuelle. 

Pour qu’une interface réseau soit en mesure de transférer le trafic réseau qui lui est envoyé, mais qui n’est pas destiné à sa propre adresse IP, le transfert IP doit être activé pour l’interface réseau. Activez le transfert IP pour l’interface réseau avec [az network nic update](/cli/azure/network/nic).

```azurecli-interactive
az network nic update \
  --name myVmNvaVMNic \
  --resource-group myResourceGroup \
  --ip-forwarding true
```

Sur la machine virtuelle, le système d’exploitation ou une application exécutée dans la machine virtuelle, doit également pouvoir transférer le trafic réseau. Activez le transfert IP sur le système d’exploitation de la machine virtuelle avec [az vm extension set](/cli/azure/vm/extension) :

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVmNva \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute":"sudo sysctl -w net.ipv4.ip_forward=1"}'
```

L’exécution de la commande peut prendre jusqu’à une minute.

## <a name="create-virtual-machines"></a>Créer des machines virtuelles

Créez deux machines virtuelles dans le réseau virtuel de manière à pouvoir vérifier, lors d’une étape ultérieure, que le trafic provenant du sous-réseau *Public* est acheminé vers le sous-réseau *Private* via l’appliance virtuelle réseau. 

Créez une machine virtuelle dans le sous-réseau *Public* avec [az vm create](/cli/azure/vm). Le paramètre `--no-wait` permet à Azure d’exécuter la commande en arrière-plan de façon que vous puissiez continuer jusqu’à la commande suivante. Pour simplifier cet article, un mot de passe est utilisé. Les clés sont généralement utilisées dans les déploiements en production. Si vous utilisez des clés, vous devez également configurer le transfert de l’agent SSH. Pour plus d’informations, consultez la documentation associée à votre client SSH. Remplacez `<replace-with-your-password>` dans la commande suivante par un mot de passe de votre choix.

```azurecli-interactive
adminPassword="<replace-with-your-password>"

az vm create \
  --resource-group myResourceGroup \
  --name myVmPublic \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Public \
  --admin-username azureuser \
  --admin-password $adminPassword \
  --no-wait
```

Créez une machine virtuelle dans le sous-réseau *Private*.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmPrivate \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Private \
  --admin-username azureuser \
  --admin-password $adminPassword
```

La création de la machine virtuelle ne nécessite que quelques minutes. Une fois la machine virtuelle créée, l’interface CLI Azure affiche des informations similaires à celles de l’exemple suivant : 

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVmPrivate",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.1.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```

Veuillez noter **publicIpAddress**. Cette adresse sera utilisée pour accéder à la machine virtuelle à partir d’Internet dans une prochaine étape.

## <a name="route-traffic-through-an-nva"></a>Router le trafic via une NVA

Utilisez la commande suivante pour créer une session SSH avec la machine virtuelle *myVmPrivate*. Remplacez *\<publicIpAddress>* par l’adresse IP publique de votre machine virtuelle. Dans l’exemple ci-dessus, l’adresse IP est *13.90.242.231*.

```bash
ssh azureuser@<publicIpAddress>
```

Quand vous êtes invité à indiquer un mot de passe, entrez celui que vous avez sélectionné dans [Créer des machines virtuelles](#create-virtual-machines).

Utilisez la commande suivante pour installer la détermination d’itinéraire sur la machine virtuelle *myVmPrivate* :

```bash
sudo apt-get install traceroute
```

Utilisez la commande suivante pour tester le routage du trafic réseau vers la machine virtuelle *myVmPublic* à partir de la machine virtuelle *myVmPrivate*.

```bash
traceroute myVmPublic
```

La réponse ressemble à ce qui suit :

```output
traceroute to myVmPublic (10.0.0.4), 30 hops max, 60 byte packets
1  10.0.0.4 (10.0.0.4)  1.404 ms  1.403 ms  1.398 ms
```

Vous pouvez voir que le trafic est routé directement de la machine virtuelle *myVmPrivate* vers la machine virtuelle *myVmPublic*. Les itinéraires par défaut d’Azure acheminent directement le trafic entre les sous-réseaux. 

Utilisez la commande suivante pour établir une connexion SSH avec la machine virtuelle *myVmPublic* à partir de la machine virtuelle *myVmPrivate* :

```bash
ssh azureuser@myVmPublic
```

Utilisez la commande suivante pour installer la détermination d’itinéraire sur la machine virtuelle *myVmPublic* :

```bash
sudo apt-get install traceroute
```

Utilisez la commande suivante pour tester le routage du trafic réseau vers la machine virtuelle *myVmPrivate* à partir de la machine virtuelle *myVmPublic*.

```bash
traceroute myVmPrivate
```

La réponse ressemble à ce qui suit :

```output
traceroute to myVmPrivate (10.0.1.4), 30 hops max, 60 byte packets
1  10.0.2.4 (10.0.2.4)  0.781 ms  0.780 ms  0.775 ms
2  10.0.1.4 (10.0.0.4)  1.404 ms  1.403 ms  1.398 ms
```

Vous voyez que le premier tronçon est 10.0.2.4, ce qui correspond à l’adresse IP privée de l’appliance virtuelle réseau (NVA). Le second tronçon est 10.0.1.4, ce qui correspond à l’adresse IP privée de la machine virtuelle *myVmPrivate*. L’itinéraire ajouté à la table de routage *myRouteTablePublic* et associé au sous-réseau *Public* a contraint Azure à acheminer le trafic via l’appliance virtuelle réseau et non directement au sous-réseau *Private*.

Fermez les sessions SSH sur les machines virtuelles *myVmPublic* et *myVmPrivate*.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, utilisez [az group delete](/cli/azure/group) pour le supprimer, ainsi que toutes les ressources qu’il contient.

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez créé une table de routage que vous avez associée à un sous-réseau. Vous avez créé une appliance virtuelle réseau (NVA) simple qui a routé le trafic d’un sous-réseau public vers un sous-réseau privé. Déployez différentes NVA préconfigurées exécutant des fonctions de réseau, telles que la fonction de pare-feu ou l’optimisation WAN, à partir de la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking). Pour en savoir plus sur le routage, consultez [Routage du trafic de réseau virtuel](virtual-networks-udr-overview.md) et [Créer, modifier ou supprimer une table de routage](manage-route-table.md).

Alors que vous pouvez déployer de nombreuses ressources Azure dans un réseau virtuel, les ressources pour certains services Azure PaaS ne peuvent pas être déployées dans un réseau virtuel. Cependant, vous pouvez toujours restreindre l’accès aux ressources de certains services Azure PaaS au trafic provenant uniquement d’un sous-réseau de réseau virtuel. Pour connaître la marche à suivre, consultez [Restreindre l’accès réseau aux ressources PaaS](tutorial-restrict-network-access-to-resources-cli.md).
