---
title: 'Démarrage rapide : Diagnostiquer un problème de filtre de trafic réseau sur une machine virtuelle - Azure CLI'
titleSuffix: Azure Network Watcher
description: Découvrez comment utiliser Azure CLI pour diagnostiquer un problème de filtre de trafic réseau sur une machine virtuelle à l’aide de la fonctionnalité de vérification de flux IP d’Azure Network Watcher.
services: network-watcher
documentationcenter: network-watcher
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
Customer intent: I need to diagnose a virtual machine (VM) network traffic filter problem that prevents communication to and from a VM.
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 04/20/2018
ms.author: kumud
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: bd99fe0ea8a92ad05ad258dcf4d8da6e4685f263
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492520"
---
# <a name="quickstart-diagnose-a-virtual-machine-network-traffic-filter-problem---azure-cli"></a>Démarrage rapide : Diagnostiquer un problème de filtre de trafic réseau sur une machine virtuelle - Azure CLI

Dans ce guide de démarrage rapide, vous déployez une machine virtuelle, puis vous vérifiez les communications vers une adresse IP et une URL et à partir d’une adresse IP. Vous déterminez la cause d’un échec de communication et la façon de le résoudre.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Ce guide de démarrage rapide nécessite la version 2.0 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée. 

- Les commandes Azure CLI dans ce guide de démarrage rapide sont mises en forme de manière à s’exécuter dans un interpréteur de commandes Bash.

## <a name="create-a-vm"></a>Créer une machine virtuelle

Avant de pouvoir créer une machine virtuelle, vous devez créer un groupe de ressources pour qu’il contienne la machine virtuelle. Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm). Si des clés SSH n’existent pas déjà dans un emplacement de clé par défaut, la commande les crée. Pour utiliser un ensemble spécifique de clés, utilisez l’option `--ssh-key-value`. L’exemple suivant crée une machine virtuelle nommée *myVm* :

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm \
  --image UbuntuLTS \
  --generate-ssh-keys
```

La création de la machine virtuelle ne nécessite que quelques minutes. Ne passez pas aux étapes restantes avant que la machine virtuelle ne soit créée et qu’Azure CLI ne retourne la sortie.

## <a name="test-network-communication"></a>Tester la communication réseau

Pour tester une communication réseau avec Network Watcher, commencez par activer un observateur réseau dans la région de la machine virtuelle que vous souhaitez tester, puis utilisez la fonctionnalité de vérification de flux IP de Network Watcher pour tester la communication.

### <a name="enable-network-watcher"></a>Activer Network Watcher

Si vous disposez déjà d’un observateur réseau activé dans la région USA Est, passez à l’étape [Utiliser la vérification des flux IP](#use-ip-flow-verify). Utilisez la commande [az network watcher configure](/cli/azure/network/watcher#az-network-watcher-configure) pour créer un observateur réseau dans la région États-Unis de l’Est :

```azurecli-interactive
az network watcher configure \
  --resource-group NetworkWatcherRG \
  --locations eastus \
  --enabled
```

### <a name="use-ip-flow-verify"></a>Utiliser la vérification des flux IP

Lorsque vous créez une machine virtuelle, Azure autorise et refuse le trafic réseau à destination et en provenance de la machine virtuelle, par défaut. Vous pourrez ultérieurement remplacer des valeurs Azure par défaut, en autorisant ou en refusant d’autres types de trafic. Pour savoir si le trafic est autorisé ou refusé vers différentes destinations et à partir d’une adresse IP source, utilisez la commande [az network watcher test-ip-flow](/cli/azure/network/watcher#az-network-watcher-test-ip-flow).

Testez la communication sortante de la machine virtuelle vers l’une des adresses IP pour www.bing.com :

```azurecli-interactive
az network watcher test-ip-flow \
  --direction outbound \
  --local 10.0.0.4:60000 \
  --protocol TCP \
  --remote 13.107.21.200:80 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

Après quelques secondes, le résultat retourné vous informe que l’accès est autorisé par une règle de sécurité nommée **AllowInternetOutbound**.

Testez la communication sortante de la machine virtuelle vers l’adresse 172.31.0.100 :

```azurecli-interactive
az network watcher test-ip-flow \
  --direction outbound \
  --local 10.0.0.4:60000 \
  --protocol TCP \
  --remote 172.31.0.100:80 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

Le résultat retourné vous informe que l’accès est refusé par une règle de sécurité nommée **DefaultOutboundDenyAll**.

Testez la communication entrante vers la machine virtuelle à partir de l’adresse 172.31.0.100 :

```azurecli-interactive
az network watcher test-ip-flow \
  --direction inbound \
  --local 10.0.0.4:80 \
  --protocol TCP \
  --remote 172.31.0.100:60000 \
  --vm myVm \
  --nic myVmVMNic \
  --resource-group myResourceGroup \
  --out table
```

Le résultat retourné vous informe que l’accès est refusé par une règle de sécurité nommée **DefaultInboundDenyAll**. À présent que vous savez quelles règles de sécurité autorisent ou refusent le trafic à destination ou en provenance d’une machine virtuelle, vous pouvez déterminer comment résoudre les problèmes.

## <a name="view-details-of-a-security-rule"></a>Voir les détails d’une règle de sécurité

Pour déterminer la raison pour laquelle les règles de l’étape [Utiliser la vérification des flux IP](#use-ip-flow-verify) autorisent ou empêchent la communication, examinez les règles de sécurité en vigueur pour l’interface réseau avec la commande [az network nic list-effective-nsg](/cli/azure/network/nic#az-network-nic-list-effective-nsg) :

```azurecli-interactive
az network nic list-effective-nsg \
  --resource-group myResourceGroup \
  --name myVmVMNic
```

La sortie renvoyée contient le texte suivant pour la règle **AllowInternetOutbound** qui a autorisé un accès sortant vers www.bing.com à l’étape [Utiliser la vérification des flux IP](#use-ip-flow-verify) :

```console
{
 "access": "Allow",
 "additionalProperties": {},
 "destinationAddressPrefix": "Internet",
 "destinationAddressPrefixes": [
  "Internet"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Outbound",
 "expandedDestinationAddressPrefix": [
  "1.0.0.0/8",
  "2.0.0.0/7",
  "4.0.0.0/6",
  "8.0.0.0/7",
  "11.0.0.0/8",
  "12.0.0.0/6",
  ...
 ],
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/AllowInternetOutBound",
 "priority": 65001,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
},
```

Dans la sortie précédente, vous pouvez voir que **destinationAddressPrefix** a pour valeur **Internet**. Mais il est difficile de savoir quelle est le type de relation entre 13.107.21.200 et **Internet**. Plusieurs préfixes d’adresse sont répertoriés sous **expandedDestinationAddressPrefix**. L’un des préfixes de la liste est **12.0.0.0/6**, ce qui englobe la plage d’adresses IP 12.0.0.1-15.255.255.254. Étant donné que l’adresse 13.107.21.200 se trouve dans cette plage d’adresses, la règle **AllowInternetOutBound** autorise le trafic sortant. En outre, il n’y a aucune règle de priorité supérieure (numéro inférieur) indiquée dans la sortie précédente pour remplacer cette règle. Pour refuser les communications sortantes vers une adresse IP, vous pouvez ajouter une règle de sécurité avec une priorité plus élevée, ce qui empêche la sortie du port 80 vers l’adresse IP.

Lorsque vous avez exécuté la commande `az network watcher test-ip-flow` pour tester la communication sortante vers l’adresse 172.131.0.100 dans [Utiliser la vérification des flux IP](#use-ip-flow-verify), la sortie vous a informé que la règle **DefaultOutboundDenyAll** a refusé la communication. La règle **DefaultOutboundDenyAll** équivaut à la règle **DenyAllOutBound** répertoriée dans la sortie suivante de la commande `az network nic list-effective-nsg` :

```console
{
 "access": "Deny",
 "additionalProperties": {},
 "destinationAddressPrefix": "0.0.0.0/0",
 "destinationAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Outbound",
 "expandedDestinationAddressPrefix": null,
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/DenyAllOutBound",
 "priority": 65500,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
}
```

La règle répertorie **0.0.0.0/0** comme **destinationAddressPrefix**. La règle refuse les communications sortantes vers l’adresse 172.131.0.100, car l’adresse ne se trouve pas dans le même **destinationAddressPrefix** que toutes les autres règles de trafic sortant dans la sortie de la commande `az network nic list-effective-nsg`. Pour autoriser les communications sortantes, vous pouvez ajouter une règle de sécurité avec une priorité plus élevée, afin d’autoriser le trafic sortant vers le port 80 à l’adresse 172.131.0.100.

Lorsque vous avez exécuté la commande `az network watcher test-ip-flow` à l’étape [Utiliser la vérification des flux IP](#use-ip-flow-verify) pour tester la communication entrante à partir l’adresse 172.131.0.100, la sortie vous a informé que la règle **DefaultInboundDenyAll** a refusé la communication. La règle **DefaultInboundDenyAll** équivaut à la règle **DenyAllInBound** répertoriée dans la sortie suivante de la commande `az network nic list-effective-nsg` :

```console
{
 "access": "Deny",
 "additionalProperties": {},
 "destinationAddressPrefix": "0.0.0.0/0",
 "destinationAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "destinationPortRange": "0-65535",
 "destinationPortRanges": [
  "0-65535"
 ],
 "direction": "Inbound",
 "expandedDestinationAddressPrefix": null,
 "expandedSourceAddressPrefix": null,
 "name": "defaultSecurityRules/DenyAllInBound",
 "priority": 65500,
 "protocol": "All",
 "sourceAddressPrefix": "0.0.0.0/0",
 "sourceAddressPrefixes": [
  "0.0.0.0/0"
 ],
 "sourcePortRange": "0-65535",
 "sourcePortRanges": [
  "0-65535"
 ]
},
```

La règle **DenyAllInBound** est appliquée, car comme indiqué dans la sortie, aucune règle de priorité plus élevée n’est présente dans la sortie de la commande `az network nic list-effective-nsg` afin d’autoriser le port 80 entrant de l’adresse 172.131.0.100 vers la machine virtuelle. Pour autoriser les communications entrantes, vous pouvez ajouter une règle de sécurité avec une priorité plus élevée, afin d’autoriser le trafic entrant vers le port 80 à partir de l’adresse 172.131.0.100.

Les vérifications de ce guide de démarrage rapide ont permis de tester la configuration Azure. Si les vérifications effectuées retournent les résultats attendus alors que vous rencontrez toujours des problèmes réseau, vérifiez qu’il n’y a aucun pare-feu entre votre machine virtuelle et le point de terminaison avec lequel vous communiquez, et que le système d’exploitation dans votre machine virtuelle n’a pas de pare-feu qui autorise ou refuse les communications.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, vous pouvez utiliser [az group delete](/cli/azure/group) pour le supprimer, ainsi que toutes les ressources qu’il contient :

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé une machine virtuelle et diagnostiqué des filtres de trafic réseau entrant et sortant. Vous avez appris que les règles de groupe de sécurité réseau autorisent ou refusent le trafic à destination et en provenance d’une machine virtuelle. En savoir plus sur les [règles de sécurité](../virtual-network/network-security-groups-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) et la [création des règles de sécurité](../virtual-network/manage-network-security-group.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-security-rule).

Même avec des filtres de trafic réseau adaptés, les communications vers une machine virtuelle peuvent échouer en raison d’une configuration de routage. Pour savoir comment diagnostiquer les problèmes de routage réseau d’une machine virtuelle, consultez [Diagnostiquer des problèmes de routage sur une machine virtuelle](diagnose-vm-network-routing-problem-cli.md) ou, pour diagnostiquer les problèmes liés au routage sortant, à la latence et au filtrage de trafic, avec un outil, consultez [Résoudre les problèmes de connexion](network-watcher-connectivity-cli.md).