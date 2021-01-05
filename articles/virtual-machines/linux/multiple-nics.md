---
title: Créer une machine virtuelle Linux dans Azure avec plusieurs cartes réseau
description: Découvrez comment créer une machine virtuelle Linux dotée de plusieurs cartes réseau à l’aide de l’interface de ligne de commande Azure ou de modèles Resource Manager.
author: cynthn
ms.service: virtual-machines-linux
ms.subservice: networking
ms.topic: how-to
ms.workload: infrastructure
ms.date: 06/07/2018
ms.author: cynthn
ms.openlocfilehash: 86910ece57d8fb72ade0c67a9e6787023c4283f3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87836919"
---
# <a name="how-to-create-a-linux-virtual-machine-in-azure-with-multiple-network-interface-cards"></a>Guide de création d’une machine virtuelle Linux dans Azure avec plusieurs cartes d’interface réseau


Cet article explique comment créer une machine virtuelle avec plusieurs cartes réseau à l’aide de l’interface Azure CLI.

## <a name="create-supporting-resources"></a>Créer des ressources de support
Installez la dernière version [d’Azure CLI](/cli/azure/install-az-cli2) et connectez-vous à un compte Azure avec [az login](/cli/azure/reference-index).

Dans les exemples suivants, remplacez les exemples de noms de paramètre par vos propres valeurs. Les noms de paramètre sont par exemple *myResourceGroup*, *mystorageaccount* et *myVM*.

Tout d’abord, créez un groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli
az group create --name myResourceGroup --location eastus
```

Créez le réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet). L’exemple suivant permet de créer un réseau virtuel nommé *myVnet* et un sous-réseau nommé *mySubnetFrontEnd* :

```azurecli
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefix 10.0.0.0/16 \
    --subnet-name mySubnetFrontEnd \
    --subnet-prefix 10.0.1.0/24
```

Créez un sous-réseau pour le trafic principal avec la commande [az network vnet subnet create](/cli/azure/network/vnet/subnet). L’exemple suivant permet de créer un sous-réseau nommé *mySubnetBackEnd* :

```azurecli
az network vnet subnet create \
    --resource-group myResourceGroup \
    --vnet-name myVnet \
    --name mySubnetBackEnd \
    --address-prefix 10.0.2.0/24
```

Créez un groupe de sécurité réseau avec la commande [az network nsg create](/cli/azure/network/nsg). L’exemple suivant crée un groupe de sécurité réseau nommé *myNetworkSecurityGroup* :

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --name myNetworkSecurityGroup
```

## <a name="create-and-configure-multiple-nics"></a>Créer et configurer plusieurs cartes réseau
Créez deux cartes réseau avec la commande [az network nic create](/cli/azure/network/nic). L’exemple suivant crée deux cartes réseau, nommées *myNic1* et *myNic2*, connectées au groupe de sécurité réseau et avec une carte réseau se connectant à chaque sous-réseau :

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic1 \
    --vnet-name myVnet \
    --subnet mySubnetFrontEnd \
    --network-security-group myNetworkSecurityGroup
az network nic create \
    --resource-group myResourceGroup \
    --name myNic2 \
    --vnet-name myVnet \
    --subnet mySubnetBackEnd \
    --network-security-group myNetworkSecurityGroup
```

## <a name="create-a-vm-and-attach-the-nics"></a>Créer une machine virtuelle et attacher les cartes réseau
Lorsque vous créez la machine virtuelle, spécifiez les cartes réseau que vous avez créées avec `--nics`. Vous devez également faire attention en définissant la taille de la machine virtuelle. Il existe des limites pour le nombre maximal de cartes réseau que vous pouvez ajouter à une machine virtuelle. En savoir plus sur les [tailles des machines virtuelles Linux](../sizes.md).

Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm). L’exemple suivant crée une machine virtuelle nommée *myVM* :

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --size Standard_DS3_v2 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --nics myNic1 myNic2
```

Ajoutez des tables de routage au SE invité en suivant la procédure décrite dans [Configurer plusieurs cartes réseau dans un système d’exploitation invité](#configure-guest-os-for-multiple-nics).

## <a name="add-a-nic-to-a-vm"></a>Ajout d’une carte réseau à une machine virtuelle existante
Les étapes précédentes ont permis de créer une machine virtuelle avec plusieurs cartes réseau. Vous pouvez également ajouter des cartes réseau à une machine virtuelle existante avec l’interface Azure CLI. Comme le nombre de cartes réseau prises en charge varie suivant la [taille des machines virtuelles](../sizes.md) , pensez à dimensionner la vôtre en conséquence. Si nécessaire, vous pouvez [redimensionner une machine virtuelle](change-vm-size.md).

Créez une autre carte réseau avec [az network nic create](/cli/azure/network/nic). L’exemple suivant crée une carte réseau nommée *myNic3* connectée au sous-réseau principal et au groupe de sécurité réseau créés lors des étapes précédentes :

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic3 \
    --vnet-name myVnet \
    --subnet mySubnetBackEnd \
    --network-security-group myNetworkSecurityGroup
```

Pour ajouter une carte réseau à une machine virtuelle existante, libérez d’abord la machine virtuelle avec [az vm deallocate](/cli/azure/vm). L’exemple suivant libère la machine virtuelle nommée *myVM* :


```azurecli
az vm deallocate --resource-group myResourceGroup --name myVM
```

Ajoutez la carte réseau avec [az vm nic add](/cli/azure/vm/nic). L’exemple suivant ajoute *myNic3* à *myVM* :

```azurecli
az vm nic add \
    --resource-group myResourceGroup \
    --vm-name myVM \
    --nics myNic3
```

Démarrez la machine virtuelle avec [az vm start](/cli/azure/vm) :

```azurecli
az vm start --resource-group myResourceGroup --name myVM
```

Ajoutez des tables de routage au SE invité en suivant la procédure décrite dans [Configurer plusieurs cartes réseau dans un système d’exploitation invité](#configure-guest-os-for-multiple-nics).

## <a name="remove-a-nic-from-a-vm"></a>Suppression d’une carte réseau d’une machine virtuelle
Pour supprimer une carte réseau d’une machine virtuelle existante, libérez d’abord la machine virtuelle avec [az vm deallocate](/cli/azure/vm). L’exemple suivant libère la machine virtuelle nommée *myVM* :

```azurecli
az vm deallocate --resource-group myResourceGroup --name myVM
```

Supprimez la carte réseau avec [az vm nic remove](/cli/azure/vm/nic). L’exemple suivant supprime *myNic3* de *myVM* :

```azurecli
az vm nic remove \
    --resource-group myResourceGroup \
    --vm-name myVM \
    --nics myNic3
```

Démarrez la machine virtuelle avec [az vm start](/cli/azure/vm) :

```azurecli
az vm start --resource-group myResourceGroup --name myVM
```


## <a name="create-multiple-nics-using-resource-manager-templates"></a>Créer plusieurs cartes réseau à l’aide de modèles Resource Manager
Les modèles Azure Resource Manager utilisent des fichiers JSON déclaratifs pour définir votre environnement. Vous pouvez consulter une [vue d’ensemble d’Azure Resource Manager](../../azure-resource-manager/management/overview.md). Grâce aux modèles Resource Manager, vous pouvez créer plusieurs instances d’une ressource pendant le déploiement, à l’image de la création de plusieurs cartes réseau. Utilisez *copy* pour spécifier le nombre d’instances à créer :

```json
"copy": {
    "name": "multiplenics"
    "count": "[parameters('count')]"
}
```

En savoir plus sur la [création de plusieurs instances à l’aide de *copy*](../../azure-resource-manager/templates/copy-resources.md). 

Vous pouvez également utiliser `copyIndex()` pour ajouter ensuite un numéro à un nom de ressource permettant de créer `myNic1`, `myNic2`, etc. Voici un exemple d’ajout de la valeur d’index :

```json
"name": "[concat('myNic', copyIndex())]", 
```

Vous pouvez consulter un exemple complet de la [création de plusieurs cartes réseau à l’aide de modèles Resource Manager](../../virtual-network/template-samples.md).

Ajoutez des tables de routage au SE invité en suivant la procédure décrite dans [Configurer plusieurs cartes réseau dans un système d’exploitation invité](#configure-guest-os-for-multiple-nics).

## <a name="configure-guest-os-for-multiple-nics"></a>Configurer plusieurs cartes réseau dans un système d’exploitation invité

Les étapes précédentes ont permis de créer un réseau virtuel et un sous-réseau, de joindre des cartes réseau, puis de créer une machine virtuelle. Aucune adresse IP publique ni règle de groupe de sécurité réseau qui autorise le trafic SSH n’a été créée. Afin de configurer le système d’exploitation invité pour plusieurs cartes réseau, vous devez autoriser les connexions à distance et exécuter les commandes localement sur la machine virtuelle.

Pour autoriser le trafic SSH, créez une règle de groupe de sécurité réseau avec [az network nsg rule create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create) comme suit :

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name allow_ssh \
    --priority 101 \
    --destination-port-ranges 22
```

Créez une adresse IP publique avec [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) et affectez-la à la première carte réseau avec [az network nic ip-config update](/cli/azure/network/nic/ip-config#az-network-nic-ip-config-update) :

```azurecli
az network public-ip create --resource-group myResourceGroup --name myPublicIP

az network nic ip-config update \
    --resource-group myResourceGroup \
    --nic-name myNic1 \
    --name ipconfig1 \
    --public-ip myPublicIP
```

Pour voir l’adresse IP publique de la machine virtuelle, utilisez [az vm show](/cli/azure/vm#az-vm-show) comme suit :

```azurecli
az vm show --resource-group myResourceGroup --name myVM -d --query publicIps -o tsv
```

Établissez maintenant la connexion SSH à l’adresse IP publique de votre machine virtuelle. Le nom d’utilisateur par défaut fourni lors d’une étape précédente était *azureuser*. Indiquez vos propres nom d’utilisateur et adresse IP publique :

```bash
ssh azureuser@137.117.58.232
```

Pour effectuer un envoi vers ou depuis une interface réseau secondaire, vous devez ajouter manuellement des itinéraires persistants au système d’exploitation pour chaque interface réseau secondaire. Dans cet article, *eth1* représente l’interface secondaire. Les instructions pour l’ajout d’itinéraires persistants au système d’exploitation varient selon la distribution. Consultez la documentation de votre distribution pour obtenir des instructions.

Lors de l’ajout de l’itinéraire au système d’exploitation, l’adresse de la passerelle est *.1* pour tout sous-réseau dans lequel se trouve l’interface réseau. Par exemple, si l’adresse *10.0.2.4* est affectée à l’interface réseau, la passerelle que vous spécifiez pour l’itinéraire est *10.0.2.1*. Vous pouvez définir un réseau spécifique pour la destination de l’itinéraire ou spécifier la destination *0.0.0.0* si vous voulez que tout le trafic pour l’interface passe par la passerelle spécifiée. La passerelle pour chaque sous-réseau est gérée par le réseau virtuel.

Une fois que vous avez ajouté l’itinéraire pour une interface secondaire, vérifiez que l’itinéraire est dans votre table de routage avec `route -n`. L’exemple de sortie suivant concerne la table de routage qui contient les deux interfaces réseau ajoutées à la machine virtuelle dans cet article :

```bash
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.1.1        0.0.0.0         UG    0      0        0 eth0
0.0.0.0         10.0.2.1        0.0.0.0         UG    0      0        0 eth1
10.0.1.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 eth1
168.63.129.16   10.0.1.1        255.255.255.255 UGH   0      0        0 eth0
169.254.169.254 10.0.1.1        255.255.255.255 UGH   0      0        0 eth0
```

Confirmez que l’itinéraire que vous avez ajouté est conservé entre les redémarrages en vérifiant à nouveau votre table de routage après un redémarrage. Pour tester la connectivité, vous pouvez par exemple entrer la commande suivante, où *eth1* est le nom d’une interface réseau secondaire :

```bash
ping bing.com -c 4 -I eth1
```

## <a name="next-steps"></a>Étapes suivantes
Vérifiez les [tailles des machines virtuelles Linux](../sizes.md) si vous créez une machine virtuelle avec plusieurs cartes réseau. Faites attention au nombre maximal de cartes réseau pris en charge par chaque taille de machine virtuelle.

Pour sécuriser davantage vos machines virtuelles, utilisez l’accès aux machines virtuelles juste-à-temps. Cette fonctionnalité ouvre les règles de groupe de sécurité réseau pour le trafic SSH en cas de besoin et pour une période de temps définie. Pour plus d’informations, consultez [Gérer l’accès Juste à temps à la machine virtuelle](../../security-center/security-center-just-in-time.md).