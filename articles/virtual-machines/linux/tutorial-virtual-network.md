---
title: 'Tutoriel : Créer et gérer des réseaux virtuels Azure pour des machines virtuelles Linux'
description: Dans ce tutoriel, vous allez apprendre à utiliser Azure CLI afin de créer et gérer des réseaux virtuels Azure pour des machines virtuelles Linux
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/10/2017
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 72c7f1dd6652a51a960c80829a697b97fd74f8de
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87500025"
---
# <a name="tutorial-create-and-manage-azure-virtual-networks-for-linux-virtual-machines-with-the-azure-cli"></a>Tutoriel : Créer et gérer des réseaux virtuels Azure pour des machines virtuelles Linux avec Azure CLI

Les machines virtuelles Azure utilisent la gestion réseau Azure pour la communication réseau interne et externe. Ce didacticiel vous guide dans le déploiement de deux machines virtuelles et la configuration de la gestion réseau Azure pour celles-ci. Les exemples de ce didacticiel supposent que les machines virtuelles hébergent une application web avec un back-end de base de données. Le didacticiel ne comprend cependant pas le déploiement d’une application. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un réseau virtuel et un sous-réseau
> * Créer une adresse IP publique
> * Créer une machine virtuelle frontale
> * sécurisent le trafic réseau
> * Créer une machine virtuelle principale

Ce tutoriel utilise l’interface CLI disponible dans [Azure Cloud Shell](../../cloud-shell/overview.md), qui est constamment mise à jour vers la dernière version. Pour ouvrir Cloud Shell, sélectionnez **Essayer** en haut d’un bloc de code.

Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande localement, ce didacticiel nécessite que vous exécutiez Azure CLI version 2.0.30 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli).

## <a name="vm-networking-overview"></a>Vue d’ensemble de la mise en réseau de machines virtuelles

Avec les réseaux virtuels Azure, les connexions réseau sont sécurisées entre les machines virtuelles, Internet et d’autres services Azure comme Azure SQL Database. Les réseaux virtuels sont divisés en segments logiques, appelés sous-réseaux. Les sous-réseaux sont utilisés pour contrôler le flux du réseau et comme une limite de sécurité. Quand vous déployez une machine virtuelle, elle inclut généralement une interface de réseau virtuel, qui est attachée à un sous-réseau.

Lorsque vous suivez ce didacticiel, les ressources du réseau virtuel suivantes sont créées :

![Réseau virtuel avec deux sous-réseaux](./media/tutorial-virtual-network/networktutorial.png)

- *myVNet* : réseau virtuel que les machines virtuelles utilisent pour communiquer entre elles et avec Internet.
- *myFrontendSubnet* : sous-réseau dans *myVNet* utilisé par les ressources frontales.
- *myPublicIPAddress* : adresse IP publique utilisée pour accéder à *myFrontendVM* à partir d’Internet.
- *myFrontentNic* : interface réseau utilisée par *myFrontendVM* pour communiquer avec *myBackendVM*.
- *myFrontendVM :* : machine virtuelle utilisée pour les communications entre Internet et *myBackendVM*.
- *myBackendNSG* : groupe de sécurité réseau qui contrôle la communication entre *myFrontendVM* et *myBackendVM*.
- *myBackendSubnet* : sous-réseau associé à *myBackendNSG* et utilisé par les ressources du serveur principal.
- *myBackendNic* : interface réseau utilisée par *myBackendVM* pour communiquer avec *myFrontendVM*.
- *myBackendVM :* machine virtuelle qui utilise les ports 22 et 3306 pour communiquer avec *myFrontendVM*.

## <a name="create-a-virtual-network-and-subnet"></a>Créer un réseau virtuel et un sous-réseau

Pour ce didacticiel, un seul réseau virtuel est créé avec deux sous-réseaux. Un sous-réseau frontal pour l’hébergement d’une application web et un sous-réseau principal pour l’hébergement d’un serveur de base de données.

Avant de pouvoir créer un réseau virtuel, créez un groupe de ressources avec [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myRGNetwork* à l’emplacement eastus.

```azurecli-interactive 
az group create --name myRGNetwork --location eastus
```

### <a name="create-virtual-network"></a>Création d’un réseau virtuel

Utilisez la commande [az network vnet create](/cli/azure/network/vnet) pour créer un réseau virtuel. Dans cet exemple, le réseau est nommé *mvVNet* et le préfixe d’adresse *10.0.0.0/16* lui est affecté. Un sous-réseau est également créé avec le nom *myFrontendSubnet* et le préfixe d’adresse *10.0.1.0/24*. Plus loin dans ce didacticiel, une machine virtuelle frontale est connectée à ce sous-réseau. 

```azurecli-interactive 
az network vnet create \
  --resource-group myRGNetwork \
  --name myVNet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myFrontendSubnet \
  --subnet-prefix 10.0.1.0/24
```

### <a name="create-subnet"></a>Créer un sous-réseau

Un nouveau sous-réseau est ajouté au réseau virtuel à l’aide de la commande [az network vnet subnet create](/cli/azure/network/vnet/subnet). Dans cet exemple, le réseau est nommé *myBackendSubnet* et le préfixe d’adresse *10.0.2.0/24* lui est affecté. Ce sous-réseau est utilisé avec tous les services principaux.

```azurecli-interactive 
az network vnet subnet create \
  --resource-group myRGNetwork \
  --vnet-name myVNet \
  --name myBackendSubnet \
  --address-prefix 10.0.2.0/24
```

À ce stade, un réseau a été créé et segmenté en deux sous-réseaux, un pour les services frontaux et un autre pour les services principaux. Dans la section suivante, des machines virtuelles sont créées et connectées à ces sous-réseaux.

## <a name="create-a-public-ip-address"></a>Créer une adresse IP publique

Une adresse IP publique permet aux ressources Azure d’être accessibles sur Internet. La méthode d’allocation de l’adresse IP publique peut être configurée comme dynamique ou statique. Par défaut, une adresse IP publique est allouée dynamiquement. Les adresses IP dynamiques sont libérées quand une machine virtuelle est désallouée. Ce comportement fait que l’adresse IP change lors de toute opération qui inclut une désallocation de la machine virtuelle.

La méthode d’allocation peut être définie comme étant statique, ce qui garantit que l’adresse IP reste affectée à une machine virtuelle, même lorsqu’elle est désallouée. Quand vous utilisez une adresse IP allouée de manière statique, vous ne pouvez pas spécifier l’adresse IP elle-même. Au lieu de cela, elle est allouée à partir d’un pool d’adresses disponibles.

```azurecli-interactive
az network public-ip create --resource-group myRGNetwork --name myPublicIPAddress
```

Quand vous créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm), la méthode d’allocation des adresse IP publiques par défaut est dynamique. Lors de la création d’une machine virtuelle avec la commande [az vm create](/cli/azure/vm), incluez l’argument `--public-ip-address-allocation static` pour affecter une adresse IP publique. Cette opération n’est pas montrée dans ce didacticiel, mais dans la section suivante, une adresse IP dynamique est changée en adresse statique. 

### <a name="change-allocation-method"></a>Changer la méthode d’allocation

La méthode d’allocation d’adresse IP peut être changée avec la commande [az network public-ip update](/cli/azure/network/public-ip). Dans cet exemple, la méthode de l’allocation de l’adresse IP de la machine virtuelle frontale est changée en statique.

Désallouez d’abord la machine virtuelle.

```azurecli-interactive 
az vm deallocate --resource-group myRGNetwork --name myFrontendVM
```

Utilisez la commande [az network public-ip update](/cli/azure/network/public-ip) pour mettre à jour la méthode d’allocation. Dans ce cas, la `--allocation-method` est définie sur *statique*.

```azurecli-interactive 
az network public-ip update --resource-group myRGNetwork --name myPublicIPAddress --allocation-method static
```

Démarrez la machine virtuelle.

```azurecli-interactive 
az vm start --resource-group myRGNetwork --name myFrontendVM --no-wait
```

### <a name="no-public-ip-address"></a>Pas d’adresse IP publique

Souvent, une machine virtuelle ne doit pas être accessible sur Internet. Pour créer une machine virtuelle sans adresse IP publique, utilisez l’argument `--public-ip-address ""` avec une paire de guillemets doubles vide. Cette configuration est montrée plus loin dans ce didacticiel.

## <a name="create-a-front-end-vm"></a>Créer une machine virtuelle frontale

Utilisez la commande [az vm create](/cli/azure/vm) pour créer la machine virtuelle nommée *myFrontendVM* à l’aide de *myPublicIPAddress*.

```azurecli-interactive 
az vm create \
  --resource-group myRGNetwork \
  --name myFrontendVM \
  --vnet-name myVNet \
  --subnet myFrontendSubnet \
  --nsg myFrontendNSG \
  --public-ip-address myPublicIPAddress \
  --image UbuntuLTS \
  --generate-ssh-keys
```

## <a name="secure-network-traffic"></a>sécurisent le trafic réseau

Un groupe de sécurité réseau (NSG) contient une liste de règles de sécurité qui autorisent ou rejettent le trafic réseau vers les ressources connectées aux réseaux virtuels Azure (VNet). Les groupes de sécurité réseau peuvent être associés à des sous-réseaux ou à des interfaces réseau individuelles. Quand un groupe de sécurité réseau est associé à une interface réseau, il s’applique seulement à la machine virtuelle associée. Lorsqu’un NSG est associé à un sous-réseau, les règles s’appliquent à toutes les ressources connectées au sous-réseau. 

### <a name="network-security-group-rules"></a>Règles de groupe de sécurité réseau

Les règles de groupe de sécurité réseau définissent les ports réseau sur lesquels le trafic est autorisé ou refusé. Les règles peuvent comprendre des plages d’adresses IP sources et de destination, de façon à ce que le trafic soit contrôlé entre des systèmes ou des sous-réseaux spécifiques. Les règles de groupe de sécurité réseau ont également une priorité (entre 1 et 4 096). Les règles sont évaluées dans l’ordre des priorités. Une règle avec une priorité de 100 est évaluée avant une règle avec une priorité de 200.

Tous les groupes de ressources réseau contiennent un ensemble de règles par défaut. Les règles par défaut ne peuvent pas être supprimées, mais comme la priorité la plus basse leur est attribuée, elles peuvent être remplacées par les règles que vous créez.

Les règles par défaut pour les groupes de sécurité réseau sont :

- **Réseau virtuel** : le trafic en provenance et à destination d’un réseau virtuel est autorisé à la fois dans les directions entrante et sortante.
- **Internet** : le trafic sortant est autorisé, mais le trafic entrant est bloqué.
- **Équilibreur de charge** : autoriser l’équilibreur de charge d’Azure à tester l’intégrité de vos machines virtuelles et des instances de rôle. Si vous n’utilisez pas un groupe à charge équilibrée, vous pouvez remplacer cette règle.

### <a name="create-network-security-groups"></a>Créer des groupes de sécurité réseau

Un groupe de sécurité réseau peut être créé en même temps qu’une machine virtuelle avec la commande [az vm create](/cli/azure/vm). Dans ce cas, le groupe de sécurité réseau est associé à l’interface réseau des machines virtuelles et une règle de groupe de sécurité réseau est créée automatiquement pour autoriser le trafic sur le port *22* depuis n’importe quelle source. Plus tôt dans ce didacticiel, le groupe de sécurité réseau frontal a été créé automatiquement avec la machine virtuelle frontale. Une règle de groupe de sécurité réseau a également été créée automatiquement pour le port 22. 

Dans certains cas, il peut être utile de créer au préalable un groupe de sécurité réseau, par exemple quand des règles SSH par défaut ne doivent pas être créées ou quand le groupe de sécurité réseau doit être attaché à un sous-réseau. 

Utilisez la commande [az network nsg create](/cli/azure/network/nsg) pour créer un groupe de sécurité réseau.

```azurecli-interactive 
az network nsg create --resource-group myRGNetwork --name myBackendNSG
```

Au lieu que le groupe de sécurité réseau soit associé à une interface réseau, il est associé à un sous-réseau. Dans cette configuration, toute machine virtuelle qui est attachée au sous-réseau hérite des règles du groupe de sécurité réseau.

Mettez à jour le sous-réseau existant nommé *myBackendSubnet* avec le nouveau groupe de sécurité réseau.

```azurecli-interactive 
az network vnet subnet update \
  --resource-group myRGNetwork \
  --vnet-name myVNet \
  --name myBackendSubnet \
  --network-security-group myBackendNSG
```

### <a name="secure-incoming-traffic"></a>Sécuriser le trafic entrant

Quand la machine virtuelle frontale a été créée, une règle de groupe de sécurité réseau a été créée pour autoriser le trafic entrant sur le port 22. Cette règle autorise les connexions SSH à la machine virtuelle. Pour cet exemple, le trafic doit également être autorisé sur le port *80*. Cette configuration rend accessible une application web sur la machine virtuelle.

Utilisez la commande [az network nsg rule create](/cli/azure/network/nsg/rule) pour créer une règle pour le port *80*.

```azurecli-interactive 
az network nsg rule create \
  --resource-group myRGNetwork \
  --nsg-name myFrontendNSG \
  --name http \
  --access allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 200 \
  --source-address-prefix "*" \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range 80
```

La machine virtuelle frontale est accessible seulement sur le port *22* et sur le port *80*. Tout le trafic entrant est bloqué au niveau du groupe de sécurité réseau. Il peut être utile de visualiser les configurations des règles du groupe de sécurité réseau. Vous pouvez obtenir la configuration des règles de groupe de sécurité réseau avec la commande [az network rule list](/cli/azure/network/nsg/rule). 

```azurecli-interactive 
az network nsg rule list --resource-group myRGNetwork --nsg-name myFrontendNSG --output table
```

### <a name="secure-vm-to-vm-traffic"></a>Sécuriser le trafic entre machines virtuelles

Les règles de groupe de sécurité réseau peuvent également s’appliquer entre les machines virtuelles. Pour cet exemple, la machine virtuelle frontale doit communiquer avec la machine virtuelle principale sur les ports *22* et *3306*. Cette configuration autorise les connexions SSH à partir de la machine virtuelle frontale et permet aussi à une application sur la machine virtuelle frontale de communiquer avec une base de données MySQL principale. Tout autre trafic doit être bloqué entre la machine virtuelle frontale et la machine virtuelle principale.

Utilisez la commande [az network nsg rule create](/cli/azure/network/nsg/rule) pour créer une règle pour le port 22. Notez que l’argument `--source-address-prefix` spécifie la valeur *10.0.1.0/24*. Cette configuration garantit que seul le trafic provenant du sous-réseau frontal est autorisé à travers le groupe de sécurité réseau.

```azurecli-interactive 
az network nsg rule create \
  --resource-group myRGNetwork \
  --nsg-name myBackendNSG \
  --name SSH \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 100 \
  --source-address-prefix 10.0.1.0/24 \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range "22"
```

Ajoutez maintenant une règle pour le trafic MySQL sur le port 3306.

```azurecli-interactive 
az network nsg rule create \
  --resource-group myRGNetwork \
  --nsg-name myBackendNSG \
  --name MySQL \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 200 \
  --source-address-prefix 10.0.1.0/24 \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range "3306"
```

Enfin, comme les groupes de sécurité réseau ont une règle par défaut qui autorise tout le trafic entre les machines virtuelles d’un même réseau virtuel, vous pouvez créer une règle pour que les groupes de sécurité réseau principaux bloquent tout le trafic. Notez ici que `--priority` a la valeur *300*, qui est inférieure à la règle du groupe de sécurité réseau et à la règle MySQL. Cette configuration garantit que le trafic SSH et MySQL est toujours autorisé à travers le groupe de sécurité réseau.

```azurecli-interactive 
az network nsg rule create \
  --resource-group myRGNetwork \
  --nsg-name myBackendNSG \
  --name denyAll \
  --access Deny \
  --protocol Tcp \
  --direction Inbound \
  --priority 300 \
  --source-address-prefix "*" \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range "*"
```

## <a name="create-back-end-vm"></a>Créer une machine virtuelle principale

Créez maintenant une machine virtuelle attachée à *myBackendSubnet*. Notez que l’argument `--nsg` a comme valeur une paire de guillemets doubles vide. Vous n’avez pas besoin de créer un groupe de sécurité réseau avec la machine virtuelle. La machine virtuelle est attachée au sous-réseau principal, qui est protégé par le groupe de sécurité réseau principal créé au préalable. Ce groupe de sécurité réseau s’applique à la machine virtuelle. Notez aussi que l’argument `--public-ip-address` a comme valeur une paire de guillemets doubles vide. Cette configuration crée une machine virtuelle sans adresse IP publique. 

```azurecli-interactive 
az vm create \
  --resource-group myRGNetwork \
  --name myBackendVM \
  --vnet-name myVNet \
  --subnet myBackendSubnet \
  --public-ip-address "" \
  --nsg "" \
  --image UbuntuLTS \
  --generate-ssh-keys
```

La machine virtuelle principale est accessible seulement sur le port *22* et sur le port *3306* à partir du sous-réseau frontal. Tout le trafic entrant est bloqué au niveau du groupe de sécurité réseau. Il peut être utile de visualiser les configurations des règles du groupe de sécurité réseau. Vous pouvez obtenir la configuration des règles de groupe de sécurité réseau avec la commande [az network rule list](/cli/azure/network/nsg/rule). 

```azurecli-interactive 
az network nsg rule list --resource-group myRGNetwork --nsg-name myBackendNSG --output table
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé et sécurisé des réseaux Azure concernant les machines virtuelles. Vous avez appris à :

> [!div class="checklist"]
> * Créer un réseau virtuel et un sous-réseau
> * Créer une adresse IP publique
> * Créer une machine virtuelle frontale
> * sécurisent le trafic réseau
> * Créer une machine virtuelle principale

Passez au didacticiel suivant pour découvrir comment sécuriser les données sur des machines virtuelles avec Sauvegarde Azure. 

> [!div class="nextstepaction"]
> [Sauvegarder des machines virtuelles Linux dans Azure](./tutorial-backup-vms.md)
