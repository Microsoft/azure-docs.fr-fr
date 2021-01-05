---
title: Équilibrer la charge de plusieurs sites web - Azure CLI - Azure Load Balancer
description: Cet exemple de script Azure CLI montre comment équilibrer la charge de plusieurs sites web sur la même machine virtuelle
documentationcenter: load-balancer
author: asudbring
ms.service: load-balancer
ms.devlang: azurecli
ms.topic: sample
ms.workload: infrastructure
ms.date: 04/20/2018
ms.author: allensu
ms.custom: devx-track-azurecli
ms.openlocfilehash: 12d425cfbb093d19efec19dfa02af5ff264c1bae
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94696691"
---
# <a name="azure-cli-script-example-load-balance-multiple-websites"></a>Exemple de script Azure CLI : Équilibrer la charge de plusieurs sites web

Cet exemple de script Azure CLI crée un réseau virtuel avec deux machines virtuelles qui sont membres d’un groupe à haute disponibilité. Un équilibrage de charge dirige le trafic pour les deux adresses IP distinctes vers les deux machines virtuelles. Après avoir exécuté le script, vous pouvez déployer le logiciel de serveur web pour les machines virtuelles et héberger plusieurs sites web, chacun avec sa propre adresse IP.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Exemple de script


[!code-azurecli-interactive[main](../../../cli_scripts/load-balancer/load-balance-multiple-web-sites-vm/load-balance-multiple-web-sites-vm.sh  "Load balance multiple web sites")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement 

Exécutez la commande suivante pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées.

```azurecli
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes pour créer un groupe de ressources, un réseau virtuel, un équilibreur de charge et toutes les ressources associées. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create) | Crée un réseau virtuel et un sous-réseau Azure. |
| [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) | Crée une adresse IP publique avec une adresse IP statique et un nom DNS associé. |
| [az network lb create](/cli/azure/network/lb#az-network-lb-create) | Crée un équilibreur de charge Azure. |
| [az network lb probe create](/cli/azure/network/lb/probe#az-network-lb-probe-create) | Crée une sonde d’équilibreur de charge. Les sondes d’équilibreurs de charge permettent de surveiller chaque machine virtuelle d’un jeu d’équilibrage de charge. Si une machine virtuelle n’est plus accessible, le trafic n’est pas acheminé vers cette machine virtuelle. |
| [az network lb rule create](/cli/azure/network/lb/rule#az-network-lb-rule-create) | Crée une règle d’équilibeur de charge. Dans cet exemple, une règle est créée pour le port 80. Le trafic HTTP qui arrive à l’équilibreur de charge est acheminé vers le port 80 de l’une des machines virtuelles du jeu d’équilibrage de charge. |
| [az network lb frontend-ip create](/cli/azure/network/lb/frontend-ip#az-network-lb-frontend-ip-create) | Créez une adresse IP frontale pour l’équilibrage de charge. |
| [az network lb address-pool create](/cli/azure/network/lb/address-pool#az-network-lb-address-pool-create) | Crée un pool d’adresses principal. |
| [az network nic create](/cli/azure/network/nic#az-network-nic-create) | Crée une carte réseau virtuelle et l’associe au réseau virtuel et au sous-réseau. |
| [az vm availability-set create](/cli/azure/network/lb/rule#az-network-lb-rule-create) | Crée un groupe à haute disponibilité. Les groupes à haute disponibilité garantissent le temps de fonctionnement des applications en répartissant les machines virtuelles sur les ressources physiques de sorte que, en cas d’échec, l’ensemble du groupe ne soit pas affecté. |
| [az network nic ip-config create](/cli/azure/network/nic/ip-config#az-network-nic-ip-config-create) | Crée une configuration IP. Vous devez activer la fonction Microsoft.Network/AllowMultipleIpConfigurationsPerNic pour votre abonnement. Seule une configuration par carte réseau peut être désignée comme configuration IP principale, à l’aide de l’indicateur --make-primary. |
| [az vm create](/cli/azure/vm/availability-set#az-vm-availability-set-create) | Crée la machine virtuelle et l’associe à la carte réseau, au réseau virtuel, au sous-réseau et au groupe de sécurité réseau. Cette commande spécifie également l’image de machine virtuelle à utiliser ainsi que les informations d’identification d’administration.  |
| [az group delete](/cli/azure/vm/extension#az-vm-extension-set) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](/cli/azure).

Vous pouvez trouver des exemples supplémentaires de scripts CLI de mise en réseau dans la [documentation Vue d’ensemble de la mise en réseau Azure](../cli-samples.md?toc=%2fazure%2fnetworking%2ftoc.json).