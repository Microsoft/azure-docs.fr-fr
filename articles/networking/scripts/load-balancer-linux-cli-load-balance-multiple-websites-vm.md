---
title: Exemple de script CLI Azure - Équilibrer la charge de plusieurs sites web avec l’interface CLI Azure | Microsoft Docs
description: Exemple de Script CLI Azure - Équilibrer la charge de plusieurs sites web sur la même machine virtuelle
services: load-balancer
documentationcenter: load-balancer
author: asudbring
manager: KumudD
ms.service: load-balancer
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 07/07/2017
ms.author: allensu
ms.openlocfilehash: 568934d6b5ecc2fbb3b6fc6588ecfd48c8306490
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "73889468"
---
# <a name="load-balance-multiple-websites"></a>Équilibrer la charge de plusieurs sites web

Cet exemple de script crée un réseau virtuel avec deux machines virtuelles qui sont membres d’un groupe à haute disponibilité. Un équilibrage de charge dirige le trafic pour les deux adresses IP distinctes vers les deux machines virtuelles. Après avoir exécuté le script, vous pouvez déployer le logiciel de serveur web pour les machines virtuelles et héberger plusieurs sites web, chacun avec sa propre adresse IP.

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
| [az group create](https://docs.microsoft.com/cli/azure/group) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet) | Crée un réseau virtuel et un sous-réseau Azure. |
| [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip) | Crée une adresse IP publique avec une adresse IP statique et un nom DNS associé. |
| [az network lb create](https://docs.microsoft.com/cli/azure/network/lb) | Crée un équilibreur de charge Azure. |
| [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe) | Crée une sonde d’équilibreur de charge. Les sondes d’équilibreurs de charge permettent de surveiller chaque machine virtuelle d’un jeu d’équilibrage de charge. Si une machine virtuelle n’est plus accessible, le trafic n’est pas acheminé vers cette machine virtuelle. |
| [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule) | Crée une règle d’équilibeur de charge. Dans cet exemple, une règle est créée pour le port 80. Le trafic HTTP qui arrive à l’équilibreur de charge est acheminé vers le port 80 de l’une des machines virtuelles du jeu d’équilibrage de charge. |
| [az network lb frontend-ip create](https://docs.microsoft.com/cli/azure/network/lb/frontend-ip) | Créez une adresse IP frontale pour l’équilibrage de charge. |
| [az network lb address-pool create](https://docs.microsoft.com/cli/azure/network/lb/address-pool) | Crée un pool d’adresses principal. |
| [az network nic create](https://docs.microsoft.com/cli/azure/network/nic) | Crée une carte réseau virtuelle et l’associe au réseau virtuel et au sous-réseau. |
| [az vm availability-set create](https://docs.microsoft.com/cli/azure/network/lb/rule) | Crée un groupe à haute disponibilité. Les groupes à haute disponibilité garantissent le temps de fonctionnement des applications en répartissant les machines virtuelles sur les ressources physiques de sorte que, en cas d’échec, l’ensemble du groupe ne soit pas affecté. |
| [az network nic ip-config create](https://docs.microsoft.com/cli/azure/network/nic/ip-config) | Crée une configuration IP. Vous devez activer la fonction Microsoft.Network/AllowMultipleIpConfigurationsPerNic pour votre abonnement. Seule une configuration par carte réseau peut être désignée comme configuration IP principale, à l’aide de l’indicateur --make-primary. |
| [az vm create](https://docs.microsoft.com/cli/azure/vm/availability-set) | Crée la machine virtuelle et l’associe à la carte réseau, au réseau virtuel, au sous-réseau et au groupe de sécurité réseau. Cette commande spécifie également l’image de machine virtuelle à utiliser ainsi que les informations d’identification d’administration.  |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](https://docs.microsoft.com/cli/azure).

Vous pouvez trouver des exemples supplémentaires de scripts CLI de mise en réseau dans la [documentation Vue d’ensemble de la mise en réseau Azure](../cli-samples.md?toc=%2fazure%2fnetworking%2ftoc.json).
