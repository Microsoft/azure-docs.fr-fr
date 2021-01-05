---
title: Ouvrir des ports sur une machine virtuelle avec Azure CLI
description: Découvrez comment ouvrir un port et créer un point de terminaison sur votre machine virtuelle à l’aide d’Azure CLI.
author: cynthn
manager: gwallace
ms.service: virtual-machines
ms.subservice: networking
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 12/13/2017
ms.author: cynthn
ms.custom: devx-track-azurecli
ms.openlocfilehash: a96c0f7c6fb767b96be273a615149143043e8bc1
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91975125"
---
# <a name="open-ports-and-endpoints-to-a-vm-with-the-azure-cli"></a>Ouvrir des ports et des points de terminaison sur une machine virtuelle avec Azure CLI

Pour ouvrir un port ou créer un point de terminaison sur une machine virtuelle dans Azure, créez un filtre réseau sur un sous-réseau ou une interface réseau de machine virtuelle. Vous placez ces filtres, qui contrôlent le trafic entrant et sortant, dans un groupe de sécurité réseau associé à la ressource qui reçoit le trafic. Nous allons utiliser un exemple courant de trafic web sur le port 80. Cet article vous montre comment ouvrir un port sur une machine virtuelle avec Azure CLI. 


Pour créer un groupe de sécurité réseau et des règles, vous devez installer la dernière version [d’Azure CLI](/cli/azure/install-az-cli2) et vous connecter à un compte Azure avec [az login](/cli/azure/reference-index).

Dans les exemples suivants, remplacez les exemples de noms de paramètre par vos propres valeurs. Les noms de paramètre sont par exemple *myResourceGroup*, *myNetworkSecurityGroup* et *myVMnet*.


## <a name="quickly-open-a-port-for-a-vm"></a>Ouvrir rapidement un port pour une machine virtuelle
Si vous avez besoin d’ouvrir rapidement un port pour une machine virtuelle dans un scénario de développement et de test, vous pouvez utiliser la commande [az vm open-port](/cli/azure/vm). Cette commande crée un groupe de sécurité réseau, ajoute une règle et l’applique à une machine virtuelle ou un sous-réseau. L’exemple suivant ouvre le port *80* sur la machine virtuelle *myVM* dans le groupe de ressources *myResourceGroup*.

```azurecli
az vm open-port --resource-group myResourceGroup --name myVM --port 80
```

Pour mieux contrôler les règles, notamment pour définir une plage d’adresses IP source, effectuez les étapes supplémentaires dans cet article.


## <a name="create-a-network-security-group-and-rules"></a>Créer un groupe de sécurité réseau et les règles associées
Créez le groupe de sécurité réseau avec la commande [az network nsg create](/cli/azure/network/nsg). L’exemple suivant crée un groupe de sécurité réseau nommé *myNetworkSecurityGroup* à l’emplacement *eastus* :

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --location eastus \
    --name myNetworkSecurityGroup
```

Ajoutez une règle avec la commande [az network nsg rule create](/cli/azure/network/nsg/rule) pour autoriser le trafic HTTP sur votre serveur Web (ou adaptez en fonction de votre propre scénario, notamment l’accès SSH ou la connectivité de base de données). L’exemple suivant crée une règle nommée *myNetworkSecurityGroupRule* pour autoriser le trafic TCP sur le port 80 :

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRule \
    --protocol tcp \
    --priority 1000 \
    --destination-port-range 80
```


## <a name="apply-network-security-group-to-vm"></a>Appliquer le groupe de sécurité réseau à la machine virtuelle
Associez le groupe de sécurité réseau à l’interface réseau (NIC) de votre machine virtuelle avec la commande [az network nic update](/cli/azure/network/nic). L’exemple suivant associe une carte réseau existante nommée *myNic* au groupe de sécurité réseau nommé *myNetworkSecurityGroup* :

```azurecli
az network nic update \
    --resource-group myResourceGroup \
    --name myNic \
    --network-security-group myNetworkSecurityGroup
```

Vous pouvez également associer votre groupe de sécurité réseau à un sous-réseau de réseau virtuel avec la commande [az network vnet subnet update](/cli/azure/network/vnet/subnet) et non uniquement à l’interface réseau d’une seule machine virtuelle. L’exemple suivant associe un sous-réseau existant nommé *mySubnet* dans le réseau virtuel *myVnet* au groupe de sécurité réseau nommé *myNetworkSecurityGroup* :

```azurecli
az network vnet subnet update \
    --resource-group myResourceGroup \
    --vnet-name myVnet \
    --name mySubnet \
    --network-security-group myNetworkSecurityGroup
```

## <a name="more-information-on-network-security-groups"></a>En savoir plus sur les groupes de sécurité réseau
Les commandes rapides vous permettent d’être opérationnel avec le trafic entrant vers votre machine virtuelle. Les groupes de sécurité réseau fournissent un grand nombre de fonctionnalités intéressantes et une granularité pour contrôler l’accès à vos ressources. Découvrez plus d’informations sur la [création d’un groupe de sécurité réseau et de règles de liste de contrôle d’accès ici](tutorial-virtual-network.md#secure-network-traffic).

Pour les applications Web hautement disponibles, vous devez placer vos machines virtuelles derrière un équilibreur de charge Azure. L’équilibreur de charge répartit le trafic entre les machines virtuelles, avec un groupe de sécurité réseau qui assure le filtrage du trafic. Pour plus d’informations, consultez [Guide pratique pour équilibrer la charge des machines virtuelles Linux dans Azure pour créer une application hautement disponible](tutorial-load-balancer.md).

## <a name="next-steps"></a>Étapes suivantes
Dans cet exemple, vous avez créé une règle simple pour autoriser le trafic HTTP. Vous trouverez plus d’informations sur la création d’environnements plus détaillés dans les articles suivants :

* [Présentation d’Azure Resource Manager](../../azure-resource-manager/management/overview.md)
* [Présentation du groupe de sécurité réseau](../../virtual-network/network-security-groups-overview.md)