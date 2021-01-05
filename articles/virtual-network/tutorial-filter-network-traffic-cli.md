---
title: Filtrer le trafic réseau - Azure CLI | Microsoft Docs
description: Dans cet article, vous allez apprendre à filtrer le trafic réseau vers un sous-réseau, avec un groupe de sécurité réseau, à l’aide d’Azure CLI.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
Customer intent: I want to filter network traffic to virtual machines that perform similar functions, such as web servers.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/30/2018
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7589b13dc517a23e8d9d65907fb3342e4e2490a9
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000587"
---
# <a name="filter-network-traffic-with-a-network-security-group-using-the-azure-cli"></a>Filtrer le trafic réseau avec un groupe de sécurité réseau à l’aide d’Azure CLI

Vous pouvez filtrer le trafic réseau entrant dans un sous-réseau de réseau virtuel ou qui en sort, avec un groupe de sécurité réseau. Les groupes de sécurité réseau contiennent des règles de sécurité qui filtrent le trafic réseau par adresse IP, port et protocole. Les règles de sécurité sont appliquées aux ressources déployées dans un sous-réseau. Dans cet article, vous apprendrez comment :

* Créer un groupe de sécurité réseau et les règles associées
* Créer un réseau virtuel et associer un groupe de sécurité réseau à un sous-réseau
* Déployer des machines virtuelles dans un sous-réseau
* Tester les filtres de trafic

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.28 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Un groupe de sécurité réseau contient des règles de sécurité. Les règles de sécurité spécifient une source et une destination. Les sources et les destinations peuvent être des groupes de sécurité d’application.

### <a name="create-application-security-groups"></a>Créer des groupes de sécurité d’application

Tout d’abord, créez un groupe de ressources pour toutes les ressources créées dans cet article avec [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources dans l’emplacement *eastus* : 

```azurecli-interactive
az group create \
  --name myResourceGroup \
  --location eastus
```

Créez un groupe de sécurité d’application avec la commande [az network asg create](/cli/azure/network/asg). Un groupe de sécurité d’application vous permet de regrouper des serveurs avec des exigences de filtrage de port similaires. L’exemple suivant crée deux groupes de sécurité d’application.

```azurecli-interactive
az network asg create \
  --resource-group myResourceGroup \
  --name myAsgWebServers \
  --location eastus

az network asg create \
  --resource-group myResourceGroup \
  --name myAsgMgmtServers \
  --location eastus
```

### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Créez un groupe de sécurité réseau avec la commande [az network nsg create](/cli/azure/network/nsg). L’exemple suivant permet de créer un groupe de sécurité réseau nommé *myNsg* : 

```azurecli-interactive 
# Create a network security group
az network nsg create \
  --resource-group myResourceGroup \
  --name myNsg
```

### <a name="create-security-rules"></a>Créer des règles de sécurité

Créez une règle de sécurité avec [az network nsg rule create](/cli/azure/network/nsg/rule). L’exemple suivant crée une règle qui autorise le trafic entrant à partir d’Internet vers le groupe de sécurité d’application *myWebServers* par les ports 80 et 443 :

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsg \
  --name Allow-Web-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 100 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-asgs "myAsgWebServers" \
  --destination-port-range 80 443
```

L’exemple suivant crée une règle qui autorise le trafic entrant à partir d’Internet vers le groupe de sécurité d’application *myMgmtServers* par le port 22 :

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsg \
  --name Allow-SSH-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 110 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-asgs "myAsgMgmtServers" \
  --destination-port-range 22
```

Dans cet article, SSH (port 22) est exposé sur Internet pour la machine virtuelle *myAsgMgmtServers*. Pour les environnements de production, au lieu d’exposer le port 22 sur l’Internet, il est recommandé de vous connecter aux ressources Azure que vous souhaitez gérer à l’aide d’une connexion réseau [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou [privée](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Créez un réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet). L’exemple suivant crée un réseau virtuel nommé *myVirtualNetwork* :

```azurecli-interactive 
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --address-prefixes 10.0.0.0/16
```

Ajoutez un sous-réseau au réseau virtuel avec la commande [az network vnet subnet create](/cli/azure/network/vnet/subnet). L’exemple suivant ajoute un sous-réseau nommé *mySubnet* au réseau virtuel et l’associe au groupe de sécurité réseau *myNsg* :

```azurecli-interactive
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name mySubnet \
  --address-prefix 10.0.0.0/24 \
  --network-security-group myNsg
```

## <a name="create-virtual-machines"></a>Créer des machines virtuelles

Créez deux machines virtuelles dans le réseau virtuel pour pouvoir valider le filtrage du trafic à une étape ultérieure. 

Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm). L’exemple suivant crée une machine virtuelle qui servira de serveur web. L’option `--asgs myAsgWebServers` amène Azure à rendre l’interface réseau qu’il crée pour la machine virtuelle membre du groupe de sécurité d’application *myAsgWebServers*.

L’option `--nsg ""` est spécifiée pour empêcher la création par Azure d’un groupe de sécurité réseau par défaut pour l’interface réseau créée par Azure lorsqu’il crée la machine virtuelle. Pour simplifier cet article, un mot de passe est utilisé. Les clés sont généralement utilisées dans les déploiements en production. Si vous utilisez des clés, vous devez également configurer le transfert de l’agent SSH pour les étapes restantes. Pour plus d’informations, consultez la documentation associée à votre client SSH. Remplacez `<replace-with-your-password>` dans la commande suivante par un mot de passe de votre choix.

```azurecli-interactive
adminPassword="<replace-with-your-password>"

az vm create \
  --resource-group myResourceGroup \
  --name myVmWeb \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet mySubnet \
  --nsg "" \
  --asgs myAsgWebServers \
  --admin-username azureuser \
  --admin-password $adminPassword
```

La création de la machine virtuelle ne nécessite que quelques minutes. Une fois la machine virtuelle créée, une sortie similaire à l’exemple suivant est renvoyée : 

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVmWeb",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```

Veuillez noter **publicIpAddress**. Cette adresse sera utilisée pour accéder à la machine virtuelle à partir d’Internet dans une prochaine étape.  Créez une machine virtuelle comme un serveur d’administration :

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmMgmt \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet mySubnet \
  --nsg "" \
  --asgs myAsgMgmtServers \
  --admin-username azureuser \
  --admin-password $adminPassword
```

La création de la machine virtuelle ne nécessite que quelques minutes. Une fois la machine virtuelle, prenez note de la valeur de **publicIpAddress** dans la sortie retournée. Cette adresse est utilisée pour accéder à la machine virtuelle à la prochaine étape. Ne passez pas à l’étape suivante avant qu’Azure ait terminé la création de la machine virtuelle.

## <a name="test-traffic-filters"></a>Tester les filtres de trafic

Utilisez la commande suivante pour créer une session SSH avec la machine virtuelle *myVmMgmt*. Remplacez *\<publicIpAddress>* par l’adresse IP publique de votre machine virtuelle. Dans l’exemple ci-dessus, l’adresse IP est *13.90.242.231*.

```bash 
ssh azureuser@<publicIpAddress>
```

Lorsque vous êtes invité à indiquer un mot de passe, entrez le mot de passe que vous avez utilisé dans [Créer des machines virtuelles](#create-virtual-machines).

La connexion réussit, car le port 22 autorise le trafic entrant depuis Internet vers le groupe de sécurité d’application *myAsgMgmtServers* dans lequel se situe l’interface réseau attachée à la machine virtuelle *myVmMgmt*.

Utilisez la commande suivante pour établir une connexion SSH avec la machine virtuelle *myVmWeb* à partir de la machine virtuelle *myVmMgmt* :

```bash 
ssh azureuser@myVmWeb
```

La connexion réussit car une règle de sécurité par défaut au sein de chaque groupe de sécurité réseau autorise le trafic par tous les ports entre toutes les adresses IP au sein d’un réseau virtuel. Vous ne pouvez pas établir une connexion SSH *myVmWeb* depuis Internet, car la règle de sécurité pour le *myAsgWebServers* n’autorise pas les données entrantes venant d’Internet sur le port 22.

Utilisez les commandes suivantes pour installer le serveur web nginx sur la machine virtuelle *myVmWeb* :

```bash 
# Update package source
sudo apt-get -y update

# Install NGINX
sudo apt-get -y install nginx
```

La machine virtuelle *myVmWeb* autorise la récupération de nginx par le trafic sortant vers Internet car une règle de sécurité par défaut autorise tout le trafic sortant vers Internet. Fermez la session SSH *myVmWeb*, ce qui vous laisse à l’invite de commandes `username@myVmMgmt:~$` de la machine virtuelle *myVmMgmt*. Pour récupérer l’écran d’accueil nginx à partir de la machine virtuelle *myVmWeb*, entrez la commande suivante :

```bash
curl myVmWeb
```

Déconnectez-vous de la machine virtuelle *myVmMgmt*. Pour confirmer que vous pouvez accéder au serveur web *myVmWeb* sans être dans Azure, entrez `curl <publicIpAddress>` à partir de votre propre ordinateur. La connexion réussit, car le port 80 autorise le trafic entrant depuis Internet vers le groupe de sécurité d’application *myAsgWebServers* dans lequel se situe l’interface réseau attachée à la machine virtuelle *myVmWeb*.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, utilisez [az group delete](/cli/azure/group) pour le supprimer, ainsi que toutes les ressources qu’il contient.

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez créé un groupe de sécurité réseau et vous l’avez associé à un sous-réseau d’un réseau virtuel. Pour en savoir plus sur les groupes de sécurité réseau, consultez [Vue d’ensemble d’un groupe de sécurité réseau](security-overview.md) et [Gérer un groupe de sécurité réseau](manage-network-security-group.md).

Azure achemine par défaut le trafic entre les sous-réseaux. À la place, vous pouvez choisir par exemple d’acheminer le trafic entre les sous-réseaux via une machine virtuelle, agissant comme un pare-feu. Pour savoir comment procéder, consultez [Créer une table de routage](tutorial-create-route-table-cli.md).
