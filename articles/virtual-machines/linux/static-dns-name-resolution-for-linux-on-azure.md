---
title: Utiliser un DNS interne pour la résolution de noms de machine virtuelle avec Azure CLI
description: Comment créer des cartes d’interface réseau virtuelles et utiliser des DNS internes pour la résolution de noms de machine virtuelle sur Azure avec Azure CLI.
author: cynthn
ms.service: virtual-machines
ms.subservice: networking
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 02/16/2017
ms.author: cynthn
ms.openlocfilehash: 38f231e63ad4974a23a1201aad4d290685860b50
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87292158"
---
# <a name="create-virtual-network-interface-cards-and-use-internal-dns-for-vm-name-resolution-on-azure"></a>Création de cartes d’interface réseau virtuelle et d’utilisation des DNS internes pour la résolution des noms de machine virtuelle sur Azure

Cet article explique comment définir des noms DNS internes statiques pour les machines virtuelles Linux à l’aide de cartes réseau virtuelles et de noms d’étiquette DNS avec Azure CLI. Les noms DNS statiques sont utilisés pour les services d’infrastructure permanents comme un serveur de builds Jenkins, qui est utilisé pour ce document, ou un serveur Git.

Les conditions requises sont :

* [un compte Azure](https://azure.microsoft.com/pricing/free-trial/)
* [des fichiers de clés SSH publiques et privées](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

## <a name="quick-commands"></a>Commandes rapides
Si vous avez besoin d’accomplir rapidement cette tâche, la section suivante décrit les commandes nécessaires. Pour obtenir plus d’informations et davantage de contexte pour chaque étape, lisez la suite de ce document, à partir de [cette section](#detailed-walkthrough). Pour suivre ces étapes, vous avez besoin de la dernière version [d’Azure CLI](/cli/azure/install-az-cli2) et devez vous connecter à un compte Azure avec [az login](/cli/azure/reference-index).

Conditions préalables : Groupe de ressources, réseau virtuel et sous-réseau, groupe de sécurité réseau avec SSH entrant.

### <a name="create-a-virtual-network-interface-card-with-a-static-internal-dns-name"></a>Création d’une carte réseau virtuelle avec un nom DNS interne statique
Créez la carte réseau virtuelle avec la commande [az network nic create](/cli/azure/network/nic). L’indicateur de l’interface de ligne de commande `--internal-dns-name` sert à définir l’étiquette DNS, qui fournit le nom DNS statique pour la carte réseau virtuelle (vNic). L’exemple suivant crée une carte réseau virtuelle nommée `myNic`, la connecte au réseau virtuel `myVnet` et crée un enregistrement de nom DNS interne appelé `jenkins` :

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic \
    --vnet-name myVnet \
    --subnet mySubnet \
    --internal-dns-name jenkins
```

### <a name="deploy-a-vm-and-connect-the-vnic"></a>Déploiement d’une machine virtuelle et connexion de la carte réseau virtuelle
Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm). L’indicateur `--nics` connecte la carte réseau virtuelle à la machine virtuelle lors du déploiement sur Azure. L’exemple suivant crée une machine virtuelle nommée `myVM` avec Azure Disques managés et joint la carte réseau virtuelle nommée `myNic` à partir de l’étape précédente :

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --nics myNic \
    --image UbuntuLTS \
    --admin-username azureuser \
    --ssh-key-value ~/.ssh/id_rsa.pub
```

## <a name="detailed-walkthrough"></a>Procédure pas à pas

Une infrastructure complète d’intégration et de déploiement en continu (CiCd) sur Azure nécessite que certains serveurs soient du type statique ou durable. Il est recommandé que les ressources Azure telles que les réseaux virtuels et les groupes de sécurité réseau soient des ressources statiques et durables qui sont rarement déployées. Une fois un réseau virtuel déployé, il peut être réutilisé par de nouveaux redéploiements, sans impact négatif sur l’infrastructure. Vous pouvez par la suite ajouter un référentiel Git ou un serveur d’automatisation Jenkins à ce réseau statique dote vos environnements de développement ou de test de l’intégration et du déploiement en continu (CiCd) sur ce réseau virtuel.  

Les noms DNS internes ne peuvent être résolus qu’à l’intérieur d’un réseau virtuel Azure. Étant donné que les noms DNS sont internes, ils ne peuvent pas être résolus par le biais de l’Internet extérieur, offrant ainsi une sécurité supplémentaire pour l’infrastructure.

Dans les exemples suivants, remplacez les exemples de noms de paramètre par vos propres valeurs. Exemples de noms de paramètre : `myResourceGroup`, `myNic` et `myVM`.

## <a name="create-the-resource-group"></a>Créer le groupe de ressources
Tout d’abord, créez le groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé `myResourceGroup` à l’emplacement `westus` :

```azurecli
az group create --name myResourceGroup --location westus
```

## <a name="create-the-virtual-network"></a>Créer un réseau virtuel

L’étape suivante consiste à créer un réseau virtuel dans lequel lancer les machines virtuelles. Le réseau virtuel contient un sous-réseau pour cette procédure pas à pas. Pour plus d’informations sur les réseaux virtuels Azure, consultez la section [Créer un réseau virtuel](../../virtual-network/manage-virtual-network.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#create-a-virtual-network). 

Créez le réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet). L’exemple suivant permet de créer un réseau virtuel nommé `myVnet` et un sous-réseau nommé `mySubnet` :

```azurecli
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 192.168.1.0/24
```

## <a name="create-the-network-security-group"></a>Création du groupe de sécurité réseau
Les groupes de sécurité réseau Azure s’apparentent à un pare-feu au niveau de la couche réseau. Pour plus d’informations sur les groupes de sécurité réseau, voir [Guide de création des groupes de sécurité réseau dans l’interface de ligne de commande Azure](../../virtual-network/tutorial-filter-network-traffic-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). 

Créez le groupe de sécurité réseau avec la commande [az network nsg create](/cli/azure/network/nsg). L’exemple suivant permet de créer un groupe de sécurité réseau nommé `myNetworkSecurityGroup` :

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --name myNetworkSecurityGroup
```

## <a name="add-an-inbound-rule-to-allow-ssh"></a>Ajouter une règle de trafic entrant pour autoriser SSH
Ajoutez une règle de trafic entrant pour le groupe de sécurité réseau avec la commande [az network nsg rule create](/cli/azure/network/nsg/rule). L’exemple suivant permet de créer une règle nommée `myRuleAllowSSH` :

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myRuleAllowSSH \
    --protocol tcp \
    --direction inbound \
    --priority 1000 \
    --source-address-prefix '*' \
    --source-port-range '*' \
    --destination-address-prefix '*' \
    --destination-port-range 22 \
    --access allow
```

## <a name="associate-the-subnet-with-the-network-security-group"></a>Association du sous-réseau au groupe de sécurité réseau
Pour associer le sous-réseau au groupe de sécurité réseau, utilisez [az network vnet subnet update](/cli/azure/network/vnet/subnet). L’exemple suivant associe le sous-réseau nommé `mySubnet` au groupe de sécurité réseau `myNetworkSecurityGroup` :

```azurecli
az network vnet subnet update \
    --resource-group myResourceGroup \
    --vnet-name myVnet \
    --name mySubnet \
    --network-security-group myNetworkSecurityGroup
```


## <a name="create-the-virtual-network-interface-card-and-static-dns-names"></a>Création de la carte réseau virtuelle et des noms DNS statiques
Azure est très flexible, mais pour utiliser des noms DNS pour la résolution de noms de machines virtuelles, vous devez les créer sous forme de cartes réseau virtuelles (vNic) qui comprennent un étiquetage DNS. Les cartes réseau virtuelles sont importantes, car vous pouvez les réutiliser en les connectant sur différentes machines virtuelles sur le cycle de vie de l’infrastructure. Cette approche conserve la carte réseau virtuelle comme une ressource statique, tandis que les machines virtuelles peuvent être temporaires. À l’aide de l’étiquetage DNS sur la carte réseau virtuelle, nous sommes en mesure d’activer la résolution de noms simple à partir d’autres machines virtuelles dans le réseau virtuel. L’utilisation de noms pouvant être résolus permet à d’autres machines virtuelles d’accéder au serveur d’automatisation à l’aide du nom DNS `Jenkins` ou au serveur Git en tant que `gitrepo`.  

Créez la carte réseau virtuelle avec la commande [az network nic create](/cli/azure/network/nic). L’exemple suivant crée une carte réseau virtuelle nommée `myNic`, la connecte au réseau virtuel `myVnet` nommé `myVnet` et crée un enregistrement de nom DNS interne appelé `jenkins` :

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic \
    --vnet-name myVnet \
    --subnet mySubnet \
    --internal-dns-name jenkins
```

## <a name="deploy-the-vm-into-the-virtual-network-infrastructure"></a>Déployer la machine virtuelle dans l’infrastructure de réseau virtuel
Nous disposons désormais d’un réseau virtuel, d’un sous-réseau intégré au réseau virtuel et d’un groupe de sécurité réseau jouant le rôle de pare-feu qui protège notre sous-réseau en bloquant l’ensemble du trafic entrant, à l’exception du port 22 pour SSH, et d’une carte réseau virtuelle. Vous pouvez maintenant déployer une machine virtuelle au sein de cette infrastructure réseau existante.

Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm). L’exemple suivant crée une machine virtuelle nommée `myVM` avec Azure Disques managés et joint la carte réseau virtuelle nommée `myNic` à partir de l’étape précédente :

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --nics myNic \
    --image UbuntuLTS \
    --admin-username azureuser \
    --ssh-key-value ~/.ssh/id_rsa.pub
```

En utilisant les indicateurs CLI pour appeler les ressources existantes, nous indiquons à Azure de déployer la machine virtuelle au sein du réseau existant. Encore une fois, lorsqu’un réseau virtuel et un sous-réseau ont été déployés, ils peuvent être conservés en tant que ressources statiques ou permanentes à l’intérieur de votre région Azure.  

## <a name="next-steps"></a>Étapes suivantes
* [Créer votre propre environnement personnalisé pour une machine virtuelle Linux à l’aide des commandes de l’interface de ligne de commande Azure directement](create-cli-complete.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Création d’une machine virtuelle Linux sur Azure à l’aide de modèles](create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
