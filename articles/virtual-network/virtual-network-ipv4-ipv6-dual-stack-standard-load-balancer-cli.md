---
title: Déployer une application double pile IPv6 - Standard Load Balancer - CLI
titlesuffix: Azure Virtual Network
description: Cet article montre comment déployer une application double pile IPv6 dans un réseau virtuel Azure à l’aide d’Azure CLI.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: 07bd8e19fdd75cc97a9c34152ea5f06f850ef724
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95995678"
---
# <a name="deploy-an-ipv6-dual-stack-application-in-azure-virtual-network---cli"></a>Déployer une application double pile IPv6 dans un réseau virtuel Azure - CLI

Cet article montre comment déployer dans Azure une application double pile (IPv4 + IPv6) avec Standard Load Balancer incluant un réseau virtuel double pile avec un sous-réseau double pile, un Standard Load Balancer avec des configurations front-end doubles (IPv4 + IPv6), des machines virtuelles dont les cartes réseau ont une configuration double IP, des règles de Groupe de sécurité réseau doubles et des adresses IP publiques doubles.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.49 ou une version ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Avant de pouvoir créer un réseau virtuel double pile, vous devez créer un groupe de ressources avec [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *DsResourceGroup01* à l’emplacement *eastus* :

```azurecli-interactive
az group create \
--name DsResourceGroup01 \
--location eastus
```

## <a name="create-ipv4-and-ipv6-public-ip-addresses-for-load-balancer"></a>Créer des adresses IP publiques IPv4 et IPv6 pour l’équilibreur de charge
Pour accéder à vos points de terminaison IPv4 et IPv6 sur Internet, vous avez besoin d’adresses IP publiques IPv4 et IPv6 pour l’équilibreur de charge. Créez une adresse IP publique avec la commande [az network public-ip create](/cli/azure/network/public-ip). L’exemple suivant crée des adresses IP IPv4 et IPv6 publiques nommées *dsPublicIP_v4* and *dsPublicIP_v6* dans le groupe de ressources *DsResourceGroup01* :

```azurecli-interactive
# Create an IPV4 IP address
az network public-ip create \
--name dsPublicIP_v4  \
--resource-group DsResourceGroup01  \
--location eastus  \
--sku STANDARD  \
--allocation-method static  \
--version IPv4

# Create an IPV6 IP address
az network public-ip create \
--name dsPublicIP_v6  \
--resource-group DsResourceGroup01  \
--location eastus \
--sku STANDARD  \
--allocation-method static  \
--version IPv6

```

## <a name="create-public-ip-addresses-for-vms"></a>Créer des adresses IP publiques pour les machines virtuelles

Pour accéder à distance à vos machines virtuelles sur Internet, vous avez besoin d’adresses IP publiques IPv4 pour les machines virtuelles. Créez une adresse IP publique avec la commande [az network public-ip create](/cli/azure/network/public-ip).

```azurecli-interactive
az network public-ip create \
--name dsVM0_remote_access  \
--resource-group DsResourceGroup01 \
--location eastus  \
--sku Standard  \
--allocation-method static  \
--version IPv4

az network public-ip create \
--name dsVM1_remote_access  \
--resource-group DsResourceGroup01  \
--location eastus  \
--sku Standard  \
--allocation-method static  \
--version IPv4
```

## <a name="create-standard-load-balancer"></a>Créer un Standard Load Balancer

Dans cette section, vous allez établir la configuration IP front-end double (IPv4 et IPv6) et configurer le pool d’adresses back-end pour l’équilibreur de charge, puis créer un équilibreur de charge Standard Load Balancer.

### <a name="create-load-balancer"></a>Créer un équilibreur de charge

Créez l’équilibreur de charge Standard à l’aide de la commande [az network lb create](https://docs.microsoft.com/cli/azure/network/lb?view=azure-cli-latest). Il est nommé **dsLB** et inclut un pool frontal nommé **dsLbFrontEnd_v4** et un pool principal nommé **dsLbBackEndPool_v4**, qui est associé à l’adresse IP publique IPv4 **dsPublicIP_v4** créée à l’étape précédente. 

```azurecli-interactive
az network lb create \
--name dsLB  \
--resource-group DsResourceGroup01 \
--sku Standard \
--location eastus \
--frontend-ip-name dsLbFrontEnd_v4  \
--public-ip-address dsPublicIP_v4  \
--backend-pool-name dsLbBackEndPool_v4
```

### <a name="create-ipv6-frontend"></a>Créer une adresse IP frontale IPv6

Créez une adresse IP frontale IPV6 avec [az network lb frontend-ip create](https://docs.microsoft.com/cli/azure/network/lb/frontend-ip?view=azure-cli-latest#az-network-lb-frontend-ip-create). L’exemple suivant crée une configuration IP frontale nommée *dsLbFrontEnd_v6* et joint l’adresse *dsPublicIP_v6* :

```azurecli-interactive
az network lb frontend-ip create \
--lb-name dsLB  \
--name dsLbFrontEnd_v6  \
--resource-group DsResourceGroup01  \
--public-ip-address dsPublicIP_v6

```

### <a name="configure-ipv6-back-end-address-pool"></a>Configurer un pool d’adresses principal iPv6

Créez un pool d’adresses principal IPv6 avec [az network lb address-pool create](https://docs.microsoft.com/cli/azure/network/lb/address-pool?view=azure-cli-latest#az-network-lb-address-pool-create). L’exemple suivant crée le pool d’adresses principal nommé *dsLbBackEndPool_v6* pour inclure des machines virtuelles avec des configurations de carte réseau IPv6 :

```azurecli-interactive
az network lb address-pool create \
--lb-name dsLB  \
--name dsLbBackEndPool_v6  \
--resource-group DsResourceGroup01
```

### <a name="create-a-health-probe"></a>Créer une sonde d’intégrité
Créez une sonde d’intégrité à l’aide de la commande [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe?view=azure-cli-latest) pour surveiller l’intégrité des machines virtuelles. 

```azurecli-interactive
az network lb probe create -g DsResourceGroup01  --lb-name dsLB -n dsProbe --protocol tcp --port 3389
```

### <a name="create-a-load-balancer-rule"></a>Créer une règle d’équilibreur de charge

Une règle d’équilibrage de charge est utilisée pour définir la distribution du trafic vers les machines virtuelles. Vous définissez la configuration IP frontale pour le trafic entrant et le pool d’adresses IP principal pour recevoir le trafic, ainsi que le port source et le port de destination requis. 

Utilisez [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule?view=azure-cli-latest#az-network-lb-rule-create) pour créer une règle d’équilibrage de charge. L’exemple suivant crée des règles d’équilibreur de charge nommées *dsLBrule_v4* et *dsLBrule_v6* et équilibre le trafic sur le port *TCP* *80* vers les configurations IP front-end IPv4 et IPv6 :

```azurecli-interactive
az network lb rule create \
--lb-name dsLB  \
--name dsLBrule_v4  \
--resource-group DsResourceGroup01  \
--frontend-ip-name dsLbFrontEnd_v4  \
--protocol Tcp  \
--frontend-port 80  \
--backend-port 80  \
--probe-name dsProbe \
--backend-pool-name dsLbBackEndPool_v4


az network lb rule create \
--lb-name dsLB  \
--name dsLBrule_v6  \
--resource-group DsResourceGroup01 \
--frontend-ip-name dsLbFrontEnd_v6  \
--protocol Tcp  \
--frontend-port 80 \
--backend-port 80  \
--probe-name dsProbe \
--backend-pool-name dsLbBackEndPool_v6

```

## <a name="create-network-resources"></a>Créer des ressources réseau
Avant de déployer des machines virtuelles, vous devez créer des ressources réseau de prise en charge (groupe à haute disponibilité, Groupe de sécurité réseau, réseau virtuel et cartes réseau virtuelles). 
### <a name="create-an-availability-set"></a>Créer un groupe à haute disponibilité
Pour améliorer la disponibilité de votre application, placez vos machines virtuelles dans un groupe à haute disponibilité.

Créez un groupe à haute disponibilité avec la commande [az vm availability-set create](https://docs.microsoft.com/cli/azure/vm/availability-set?view=azure-cli-latest). L’exemple suivant permet de créer un groupe à haute disponibilité nommé *dsAVset* :

```azurecli-interactive
az vm availability-set create \
--name dsAVset  \
--resource-group DsResourceGroup01  \
--location eastus \
--platform-fault-domain-count 2  \
--platform-update-domain-count 2  
```

### <a name="create-network-security-group"></a>Création d’un groupe de sécurité réseau

Créez un groupe de sécurité réseau pour les règles qui régiront les communications entrantes et sortantes de votre VNet (réseau virtuel).

#### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Créez un Groupe de sécurité réseau avec la commande [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg?view=azure-cli-latest#az-network-nsg-create).


```azurecli-interactive
az network nsg create \
--name dsNSG1  \
--resource-group DsResourceGroup01  \
--location eastus

```

#### <a name="create-a-network-security-group-rule-for-inbound-and-outbound-connections"></a>Créez une règle de Groupe de sécurité réseau pour les connexions entrantes et sortantes.

Avec [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule?view=azure-cli-latest#az-network-nsg-rule-create), créez une règle de Groupe de sécurité réseau pour autoriser les connexions RDP sur le port 3389 et la connexion Internet sur le port 80, ainsi que pour les connexions sortantes.

```azurecli-interactive
# Create inbound rule for port 3389
az network nsg rule create \
--name allowRdpIn  \
--nsg-name dsNSG1  \
--resource-group DsResourceGroup01  \
--priority 100  \
--description "Allow Remote Desktop In"  \
--access Allow  \
--protocol "*"  \
--direction Inbound  \
--source-address-prefixes "*"  \
--source-port-ranges "*"  \
--destination-address-prefixes "*"  \
--destination-port-ranges 3389

# Create inbound rule for port 80
az network nsg rule create \
--name allowHTTPIn  \
--nsg-name dsNSG1  \
--resource-group DsResourceGroup01  \
--priority 200  \
--description "Allow HTTP In"  \
--access Allow  \
--protocol "*"  \
--direction Inbound  \
--source-address-prefixes "*"  \
--source-port-ranges 80  \
--destination-address-prefixes "*"  \
--destination-port-ranges 80

# Create outbound rule

az network nsg rule create \
--name allowAllOut  \
--nsg-name dsNSG1  \
--resource-group DsResourceGroup01  \
--priority 300  \
--description "Allow All Out"  \
--access Allow  \
--protocol "*"  \
--direction Outbound  \
--source-address-prefixes "*"  \
--source-port-ranges "*"  \
--destination-address-prefixes "*"  \
--destination-port-ranges "*"
```


### <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Créez un réseau virtuel avec la commande [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet?view=azure-cli-latest#az-network-vnet-create). L’exemple suivant crée un réseau virtuel nommé *dsVNET* avec les sous-réseaux *dsSubNET_v4* et *dsSubNET_v6* :

```azurecli-interactive
# Create the virtual network
az network vnet create \
--name dsVNET \
--resource-group DsResourceGroup01 \
--location eastus  \
--address-prefixes "10.0.0.0/16" "fd00:db8:deca::/48"

# Create a single dual stack subnet

az network vnet subnet create \
--name dsSubNET \
--resource-group DsResourceGroup01 \
--vnet-name dsVNET \
--address-prefixes "10.0.0.0/24" "fd00:db8:deca:deed::/64" \
--network-security-group dsNSG1
```

### <a name="create-nics"></a>Créer des cartes réseau

Créez des cartes réseau virtuelles pour chaque machine virtuelle avec [az network nic create](https://docs.microsoft.com/cli/azure/network/nic?view=azure-cli-latest#az-network-nic-create). L’exemple suivant crée une carte réseau virtuelle pour chaque machine virtuelle. Chaque carte réseau a deux configurations IP (1 configuration IPv4 et 1 configuration IPv6). Vous allez créer la configuration IPV6 avec [az network nic ip-config create](https://docs.microsoft.com/cli/azure/network/nic/ip-config?view=azure-cli-latest#az-network-nic-ip-config-create).
 
```azurecli-interactive
# Create NICs
az network nic create \
--name dsNIC0  \
--resource-group DsResourceGroup01 \
--network-security-group dsNSG1  \
--vnet-name dsVNET  \
--subnet dsSubNet  \
--private-ip-address-version IPv4 \
--lb-address-pools dsLbBackEndPool_v4  \
--lb-name dsLB  \
--public-ip-address dsVM0_remote_access

az network nic create \
--name dsNIC1 \
--resource-group DsResourceGroup01 \
--network-security-group dsNSG1 \
--vnet-name dsVNET \
--subnet dsSubNet \
--private-ip-address-version IPv4 \
--lb-address-pools dsLbBackEndPool_v4 \
--lb-name dsLB \
--public-ip-address dsVM1_remote_access

# Create IPV6 configurations for each NIC

az network nic ip-config create \
--name dsIp6Config_NIC0  \
--nic-name dsNIC0  \
--resource-group DsResourceGroup01 \
--vnet-name dsVNET \
--subnet dsSubNet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name dsLB

az network nic ip-config create \
--name dsIp6Config_NIC1 \
--nic-name dsNIC1 \
--resource-group DsResourceGroup01 \
--vnet-name dsVNET \
--subnet dsSubNet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name dsLB
```

### <a name="create-virtual-machines"></a>Créer des machines virtuelles

Créez les machines virtuelles avec [az vm create](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-create). L’exemple suivant crée deux machines virtuelles ainsi que les composants de réseau virtuel nécessaires s’ils n’existent pas encore. 

Créez la machine virtuelle *dsVM0* comme suit :

```azurecli-interactive
 az vm create \
--name dsVM0 \
--resource-group DsResourceGroup01 \
--nics dsNIC0 \
--size Standard_A2 \
--availability-set dsAVset \
--image MicrosoftWindowsServer:WindowsServer:2019-Datacenter:latest  
```

Créez la machine virtuelle *dsVM1* comme suit :

```azurecli-interactive
az vm create \
--name dsVM1 \
--resource-group DsResourceGroup01 \
--nics dsNIC1 \
--size Standard_A2 \
--availability-set dsAVset \
--image MicrosoftWindowsServer:WindowsServer:2019-Datacenter:latest 
```

## <a name="view-ipv6-dual-stack-virtual-network-in-azure-portal"></a>Afficher le réseau virtuel double pile IPv6 dans le Portail Microsoft Azure
Vous pouvez afficher le réseau virtuel double pile IPv6 dans le Portail Microsoft Azure en procédant comme suit :
1. Dans la barre de recherche du portail, saisissez *dsVnet*.
2. Quand la mention **myVirtualNetwork** apparaît dans les résultats de recherche, sélectionnez-la. Cette opération affiche la page **Vue d’ensemble** du réseau virtuel double pile nommé *dsVnet*. Le réseau virtuel de pile double présente les deux cartes réseau, avec des configurations IPv4 et IPv6 situées dans le sous-réseau double pile nommé *dsSubnet*.

  ![Réseau virtuel double pile IPv6 dans Azure](./media/virtual-network-ipv4-ipv6-dual-stack-powershell/dual-stack-vnet.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, vous pouvez utiliser la commande [az group delete](/cli/azure/group#az-group-delete) pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées.

```azurecli-interactive
 az group delete --name DsResourceGroup01
```

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a permis de créer un équilibreur de charge Standard Load Balancer avec une configuration IP front-end double (IPv4 et IPv6). Vous avez également créé deux machines virtuelles incluant des cartes réseau avec des configurations IP doubles (IPV4 + IPv6), qui ont été ajoutées au pool back-end de l’équilibreur de charge. Pour en savoir plus sur la prise en charge du protocole IPv6 dans les réseaux virtuels Azure, voir [What is IPv6 for Azure Virtual Network?](ipv6-overview.md) (Rôle d’iPv6 pour un réseau virtuel Azure).
