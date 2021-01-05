---
title: Diagnostiquer un problème de routage réseau d’une machine virtuelle – Azure CLI
titleSuffix: Azure Network Watcher
description: Dans cet article, découvrez comment utiliser Azure CLI pour diagnostiquer un problème de routage réseau sur une machine virtuelle à l’aide de la fonctionnalité de tronçon suivant d’Azure Network Watcher.
services: network-watcher
documentationcenter: network-watcher
author: damendo
editor: ''
tags: azure-resource-manager
Customer intent: I need to diagnose virtual machine (VM) network routing problem that prevents communication to different destinations.
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 04/20/2018
ms.author: damendo
ms.custom: ''
ms.openlocfilehash: 2d5f6f9cfaff722245f6105b5e86390b8aeb769f
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94539717"
---
# <a name="diagnose-a-virtual-machine-network-routing-problem---azure-cli"></a>Diagnostiquer un problème de routage réseau d’une machine virtuelle - Azure CLI

Dans cet article, vous déployez une machine virtuelle, puis vous vérifiez les communications vers une adresse IP et une URL. Vous déterminez la cause d’un échec de communication et la façon de le résoudre.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- This article requires version 2.0 or later of the Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée. 

- Les commandes Azure CLI mentionnées dans cet article sont mises en forme de manière à s’exécuter dans un interpréteur de commandes Bash.

## <a name="create-a-vm"></a>Créer une machine virtuelle

Avant de pouvoir créer une machine virtuelle, vous devez créer un groupe de ressources pour qu’il contienne la machine virtuelle. Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm#az-vm-create). Si des clés SSH n’existent pas déjà dans un emplacement de clé par défaut, la commande les crée. Pour utiliser un ensemble spécifique de clés, utilisez l’option `--ssh-key-value`. L’exemple suivant crée une machine virtuelle nommée *myVm* :

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm \
  --image UbuntuLTS \
  --generate-ssh-keys
```

La création de la machine virtuelle ne nécessite que quelques minutes. Ne passez pas aux étapes restantes avant que la machine virtuelle ne soit créée et qu’Azure CLI ne retourne la sortie.

## <a name="test-network-communication"></a>Tester la communication réseau

Pour tester une communication réseau avec Network Watcher, commencez par activer un observateur réseau dans la région de la machine virtuelle que vous souhaitez tester, puis utilisez la fonctionnalité de tronçon suivant de Network Watcher pour tester la communication.

### <a name="enable-network-watcher"></a>Activer Network Watcher

Si vous disposez déjà d’un observateur réseau activé dans la région USA Est, passez à l’étape [Utiliser le tronçon suivant](#use-next-hop). Utilisez la commande [az network watcher configure](/cli/azure/network/watcher#az-network-watcher-configure) pour créer un observateur réseau dans la région USA Est :

```azurecli-interactive
az network watcher configure \
  --resource-group NetworkWatcherRG \
  --locations eastus \
  --enabled
```

### <a name="use-next-hop"></a>Utiliser le tronçon suivant

Azure crée automatiquement des itinéraires vers les destinations par défaut. Vous pouvez créer des itinéraires personnalisés pour remplacer les itinéraires par défaut. Parfois, les itinéraires personnalisés peuvent entraîner l’échec de la communication. Pour tester le routage à partir d’une machine virtuelle, utilisez la commande [az network watcher show-next-hop](/cli/azure/network/watcher?view=azure-cli-latest#az-network-watcher-show-next-hop) afin de déterminer le tronçon de routage suivant lorsque le trafic est destiné à une adresse spécifique.

Testez la communication sortante de la machine virtuelle vers l’une des adresses IP pour www.bing.com :

```azurecli-interactive
az network watcher show-next-hop \
  --dest-ip 13.107.21.200 \
  --resource-group myResourceGroup \
  --source-ip 10.0.0.4 \
  --vm myVm \
  --nic myVmVMNic \
  --out table
```

Après quelques secondes, la sortie vous informe que **NextHopType** est défini sur **Internet** et que **RouteTableId** est défini sur **System Route** (Itinéraire système). Ce résultat vous permet de savoir qu’il existe un itinéraire valide vers la destination.

Testez la communication sortante de la machine virtuelle vers l’adresse 172.31.0.100 :

```azurecli-interactive
az network watcher show-next-hop \
  --dest-ip 172.31.0.100 \
  --resource-group myResourceGroup \
  --source-ip 10.0.0.4 \
  --vm myVm \
  --nic myVmVMNic \
  --out table
```

La sortie retournée vous informe que **Aucun** est la valeur de **NextHopType** et que **RouteTableId** est également défini sur **System Route** (Itinéraire système). Ce résultat vous permet de savoir que s’il existe un itinéraire système valide vers la destination, il n’existe aucun tronçon suivant pour acheminer le trafic vers la destination.

## <a name="view-details-of-a-route"></a>Afficher les détails d’un itinéraire

Pour procéder à une analyse plus approfondie du routage, passez en revue les itinéraires réels de l’interface réseau avec la commande [az network nic show-effective-route-table](/cli/azure/network/nic#az-network-nic-show-effective-route-table) :

```azurecli-interactive
az network nic show-effective-route-table \
  --resource-group myResourceGroup \
  --name myVmVMNic
```

Le texte suivant est inclus dans la sortie retournée :

```console
{
  "additionalProperties": {
    "disableBgpRoutePropagation": false
  },
  "addressPrefix": [
    "0.0.0.0/0"
  ],
  "name": null,
  "nextHopIpAddress": [],
  "nextHopType": "Internet",
  "source": "Default",
  "state": "Active"
},
```

Lorsque vous avez utilisé la commande `az network watcher show-next-hop` pour tester la communication sortante vers l’adresse IP 13.107.21.200 dans la zone [Use next hop](#use-next-hop) (Utiliser le tronçon suivant), l’itinéraire présentant le **préfixe d’adresse** 0.0.0.0/0** a été utilisé pour router le trafic vers l’adresse, car aucun autre itinéraire n’inclut cette adresse. Par défaut, toutes les adresses non spécifiées dans le préfixe d’adresse d’un autre itinéraire sont acheminées vers Internet.

Toutefois, lorsque vous avez utilisé la commande `az network watcher show-next-hop` pour tester la communication sortante vers l’adresse IP 172.31.0.100, le résultat vous a informé de l’absence de type de tronçon suivant. Dans les résultats retournés, vous voyez également le texte suivant :

```console
{
  "additionalProperties": {
    "disableBgpRoutePropagation": false
      },
  "addressPrefix": [
    "172.16.0.0/12"
  ],
  "name": null,
  "nextHopIpAddress": [],
  "nextHopType": "None",
  "source": "Default",
  "state": "Active"
},
```

Comme vous pouvez le voir dans la sortie de la commande `az network watcher nic show-effective-route-table`, même s’il existe un itinéraire par défaut pour le préfixe 172.16.0.0/12, qui inclut l’adresse 172.31.0.100, **NextHopType** est défini sur **None (Aucun)** . Azure crée un itinéraire par défaut pour 172.16.0.0/12, mais ne spécifie pas de type de tronçon suivant tant qu’aucune raison ne motive cette spécification. Si, par exemple, vous avez ajouté la plage d’adresses 172.16.0.0/12 à l’espace d’adressage du réseau virtuel, Azure modifie **NextHopType** pour le définir sur **Réseau virtuel** pour l’itinéraire. Une vérification permet ensuite d’afficher **Réseau virtuel** en tant que **NextHopType**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, vous pouvez utiliser [az group delete](/cli/azure/group#az-group-delete) pour le supprimer, ainsi que toutes les ressources qu’il contient :

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez créé une machine virtuelle et diagnostiqué un problème de routage réseau à partir de celle-ci. Vous avez appris qu’Azure crée plusieurs itinéraires par défaut et testé le routage vers deux destinations différentes. En savoir plus sur le [routage dans Azure](../virtual-network/virtual-networks-udr-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) et la [création d’itinéraires personnalisés](../virtual-network/manage-route-table.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-route).

Pour les connexions sortantes d’une machine virtuelle, vous pouvez également déterminer la latence, le trafic réseau autorisé et refusé entre la machine virtuelle et un point de terminaison, à l’aide de la fonctionnalité [Résolution des problèmes de connexion](network-watcher-connectivity-cli.md) de Network Watcher. Vous pouvez surveiller la communication entre une machine virtuelle et un point de terminaison, par exemple une adresse IP ou une URL, au fil du temps à l’aide de la fonctionnalité de contrôle de la connexion de Network Watcher. Pour découvrir comment procéder, consultez [Surveiller une connexion réseau](connection-monitor.md).
