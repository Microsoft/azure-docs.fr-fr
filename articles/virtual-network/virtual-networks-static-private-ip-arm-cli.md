---
title: Configurer des adresses IP privées pour des machines virtuelles – Azure CLI
description: Apprenez à configurer des adresses IP privées pour les machines virtuelles à l’aide de l’interface de ligne de commande Azure (CLI).
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
editor: ''
tags: azure-resource-manager
ms.assetid: 40b03a1a-ea00-454c-b716-7574cea49ac0
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/16/2017
ms.author: allensu
ms.openlocfilehash: c34ab73422d8dd41feb9da542ed63fdba060fe3f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84708159"
---
# <a name="configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-cli"></a>Configurer des adresses IP privées pour une machine virtuelle à l’aide d’Azure CLI


[!INCLUDE [virtual-networks-static-private-ip-intro-include](../../includes/virtual-networks-static-private-ip-intro-include.md)]

[!INCLUDE [virtual-networks-static-ip-scenario-include](../../includes/virtual-networks-static-ip-scenario-include.md)]

> [!NOTE]
> Les exemples de commandes Azure CLI suivants attendent un environnement simple existant. Si vous souhaitez exécuter les commandes telles qu’elles sont présentées dans ce document, commencez par créer l’environnement de test décrit dans [Créer un réseau virtuel](quick-create-cli.md).

## <a name="specify-a-static-private-ip-address-when-creating-a-vm"></a>Spécifier une adresse IP privée statique lors de la création d’une machine virtuelle

Pour créer une machine virtuelle nommée *DNS01* dans le sous-réseau *FrontEnd* d’un réseau virtuel nommé *TestVNet* avec l’adresse IP privée statique *192.168.1.101*, effectuez les étapes suivantes :

1. Si vous ne l’avez pas encore fait, installez et configurez la dernière version [d’Azure CLI](/cli/azure/install-azure-cli) et connectez-vous à un compte Azure par le biais de la commande [az login](/cli/azure/reference-index).

2. Exécutez la commande [az network nic create](/cli/azure/network/nic) pour créer une carte réseau avec une adresse privée statique. La liste affichée après le résultat présente les différents paramètres utilisés. 
   
    ```azurecli
    az network nic create \
    --resource-group TestRG \
    --name TestNIC \
    --location centralus \
    --subnet FrontEnd \
    --private-ip-address 192.168.1.101 \
    --vnet-name TestVNet
    ```

    Sortie attendue :
   
    ```json
    {
        "newNIC": {
            "dnsSettings": {
            "appliedDnsServers": [],
            "dnsServers": []
            },
            "enableIPForwarding": false,
            "ipConfigurations": [
            {
                "etag": "W/\"<guid>\"",
                "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC/ipConfigurations/ipconfig1",
                "name": "ipconfig1",
                "properties": {
                "primary": true,
                "privateIPAddress": "192.168.1.101",
                "privateIPAllocationMethod": "Static",
                "provisioningState": "Succeeded",
                "subnet": {
                    "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                    "resourceGroup": "TestRG"
                }
                },
                "resourceGroup": "TestRG"
            }
            ],
            "provisioningState": "Succeeded",
            "resourceGuid": "<guid>"
        }
    }
    ```
    
    Paramètres :

    * `--private-ip-address`: Adresse IP privée statique pour la carte réseau.
    * `--vnet-name`: nom du réseau virtuel dans lequel créer la carte réseau.
    * `--subnet`: nom du sous-réseau virtuel dans lequel créer la carte réseau.

3. Exécutez la commande [azure vm create](/cli/azure/vm/nic) pour créer la machine virtuelle à l’aide de l’adresse IP publique et la carte réseau créées précédemment. La liste affichée après le résultat présente les différents paramètres utilisés.
   
    ```azurecli
    az vm create \
    --resource-group TestRG \
    --name DNS01 \
    --location centralus \
    --image Debian \
    --admin-username adminuser \
    --ssh-key-value ~/.ssh/id_rsa.pub \
    --nics TestNIC
    ```

    Sortie attendue :
   
    ```json
    {
        "fqdns": "",
        "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/DNS01",
        "location": "centralus",
        "macAddress": "00-0D-3A-92-C1-66",
        "powerState": "VM running",
        "privateIpAddress": "192.168.1.101",
        "publicIpAddress": "",
        "resourceGroup": "TestRG"
    }
    ```
   
   Paramètres autres que les paramètres [az vm create](/cli/azure/vm) de base.

   * `--nics`: nom de la carte réseau à laquelle est associée la machine virtuelle.
   
Il est recommandé de ne pas assigner statiquement l’IP privée assignée à la machine virtuelle Azure au sein du système d’exploitation d’une machine virtuelle, sauf si nécessaire, par exemple lorsque [vous assignez plusieurs d’adresses IP à une machine virtuelle Windows](virtual-network-multiple-ip-addresses-cli.md). Si vous définissez manuellement l’adresse IP privée dans le système d’exploitation, assurez-vous qu’il s’agit de la même adresse que l’adresse IP privée assignée à [l’interface réseau](virtual-network-network-interface-addresses.md#change-ip-address-settings) Azure, ou vous pouvez perdre la connectivité à la machine virtuelle. En savoir plus sur les paramètres [d’adresse IP privée](virtual-network-network-interface-addresses.md#private).

## <a name="retrieve-static-private-ip-address-information-for-a-vm"></a>Récupérer des informations d’adresse IP privée statique pour une machine virtuelle

Exécutez la commande Azure CLI suivante pour examiner les valeurs de *Private IP alloc-method* et de *Private IP address* :

```azurecli
az vm show -g TestRG -n DNS01 --show-details --query 'privateIps'
```

Sortie attendue :

```json
"192.168.1.101"
```

Pour afficher les informations IP spécifiques à la carte réseau de cette machine virtuelle, interrogez la carte réseau de manière précise :

```azurecli
az network nic show \
-g testrg \
-n testnic \
--query 'ipConfigurations[0].{PrivateAddress:privateIpAddress,IPVer:privateIpAddressVersion,IpAllocMethod:p
rivateIpAllocationMethod,PublicAddress:publicIpAddress}'
```

La sortie est identique à ce qui suit :

```json
{
    "IPVer": "IPv4",
    "IpAllocMethod": "Static",
    "PrivateAddress": "192.168.1.101",
    "PublicAddress": null
}
```

## <a name="remove-a-static-private-ip-address-from-a-vm"></a>Supprimer une adresse IP privée statique d’une machine virtuelle

Vous ne pouvez pas supprimer une adresse IP privée statique à partir d’une carte réseau dans Azure CLI pour des déploiements Resource Manager. Vous devez respecter les consignes suivantes :
- Créer une nouvelle carte réseau qui utilise une adresse IP dynamique
- Définir la carte réseau de la machine virtuelle sur l’instance nouvellement créée. 

Pour modifier la carte réseau de la machine virtuelle utilisée dans les commandes précédentes, effectuez les étapes suivantes :

1. Exécutez la commande **azure network nic create** afin de créer une nouvelle carte réseau à l’aide de l’allocation d’adresses IP dynamiques, avec une nouvelle adresse IP. Étant donné qu’aucune adresse IP n’est spécifiée, la méthode d’allocation est **Dynamique**.

    ```azurecli
    az network nic create     \
    --resource-group TestRG     \
    --name TestNIC2     \
    --location centralus     \
    --subnet FrontEnd    \
    --vnet-name TestVNet
    ```        
   
    Sortie attendue :

    ```json
    {
        "newNIC": {
            "dnsSettings": {
            "appliedDnsServers": [],
            "dnsServers": []
            },
            "enableIPForwarding": false,
            "ipConfigurations": [
            {
                "etag": "W/\"<guid>\"",
                "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC2/ipConfigurations/ipconfig1",
                "name": "ipconfig1",
                "properties": {
                "primary": true,
                "privateIPAddress": "192.168.1.4",
                "privateIPAllocationMethod": "Dynamic",
                "provisioningState": "Succeeded",
                "subnet": {
                    "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                    "resourceGroup": "TestRG"
                }
                },
                "resourceGroup": "TestRG"
            }
            ],
            "provisioningState": "Succeeded",
            "resourceGuid": "0808a61c-476f-4d08-98ee-0fa83671b010"
        }
    }
    ```

2. exécutez la commande **azure vm set** pour modifier la carte réseau utilisée par la machine virtuelle.
   
    ```azurecli
   az vm nic set --resource-group TestRG --vm-name DNS01 --nics TestNIC2
    ```

    Sortie attendue :
   
    ```json
    [
        {
            "id": "/subscriptions/0e220bf6-5caa-4e9f-8383-51f16b6c109f/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC3",
            "primary": true,
            "resourceGroup": "TestRG"
        }
    ]
    ```

    > [!NOTE]
    > Si la machine virtuelle présente une taille permettant la prise en charge de plusieurs cartes réseau, exécutez la commande **azure network nic delete** afin de supprimer l’ancienne carte réseau.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la gestion des [paramètres d’adresse IP](virtual-network-network-interface-addresses.md).
