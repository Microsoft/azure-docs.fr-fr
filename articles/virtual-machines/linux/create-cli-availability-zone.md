---
title: Créer une machine virtuelle Linux en zone avec Azure CLI
description: Créer une machine virtuelle Linux dans une zone de disponibilité avec l’interface de ligne de commande Azure
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 04/05/2018
ms.author: cynthn
ms.openlocfilehash: 964d2b5d89fd91aca68e9b47b0049529fe284848
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016401"
---
# <a name="create-a-linux-virtual-machine-in-an-availability-zone-with-the-azure-cli"></a>Créer une machine virtuelle Linux dans une zone de disponibilité avec l’interface de ligne de commande Azure

Cet article aborde l’utilisation de l’interface de ligne de commande Azure pour créer une machine virtuelle Linux dans une zone de disponibilité Azure. Une [zone de disponibilité](../../availability-zones/az-overview.md) est une zone physiquement séparée dans une région Azure. Utilisez les zones de disponibilité pour protéger vos applications et vos données dans l’éventualité peu probable d’une défaillance ou d’une perte d’un centre de données entier.

Pour utiliser une zone de disponibilité, créez votre machine virtuelle dans une [région Azure prise en charge](../../availability-zones/az-region.md).

Vérifiez que vous avez installé la dernière version [d’Azure CLI](/cli/azure/install-az-cli2) et que vous êtes connecté à un compte Azure avec [az login](/cli/azure/reference-index).


## <a name="check-vm-sku-availability"></a>Vérifier la disponibilité de la référence SKU de machine virtuelle
La disponibilité des tailles de machine virtuelle, ou des références SKU, peut varier selon la région et le fuseau horaire. Pour vous aider à planifier l’utilisation des Zones de disponibilité, vous pouvez répertorier les références SKU de machine virtuelle disponibles par zone et par région Azure. Cela permet de s’assure que vous choisissez une taille de machine virtuelle appropriée et obtenez la résilience de votre choix entre les zones. Pour plus d’informations sur les différents types de machine virtuelle et les tailles, consultez [Vue d’ensemble des tailles de machine virtuelle](../sizes.md).

Vous pouvez afficher les références SKU de machine virtuelle disponibles avec la commande [az vm list-skus](/cli/azure/vm). L’exemple suivant répertorie les références SKU de machine virtuelle disponibles dans la région *eastus2* :

```azurecli
az vm list-skus --location eastus2 --output table
```

La sortie est similaire à l’exemple condensé suivant, qui présente les Zones de disponibilité dans lesquelles chaque taille de machine virtuelle est disponible :

```output
ResourceType      Locations  Name               [...]    Tier       Size     Zones
----------------  ---------  -----------------           ---------  -------  -------
virtualMachines   eastus2    Standard_DS1_v2             Standard   DS1_v2   1,2,3
virtualMachines   eastus2    Standard_DS2_v2             Standard   DS2_v2   1,2,3
[...]
virtualMachines   eastus2    Standard_F1s                Standard   F1s      1,2,3
virtualMachines   eastus2    Standard_F2s                Standard   F2s      1,2,3
[...]
virtualMachines   eastus2    Standard_D2s_v3             Standard   D2_v3    1,2,3
virtualMachines   eastus2    Standard_D4s_v3             Standard   D4_v3    1,2,3
[...]
virtualMachines   eastus2    Standard_E2_v3              Standard   E2_v3    1,2,3
virtualMachines   eastus2    Standard_E4_v3              Standard   E4_v3    1,2,3
```


## <a name="create-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group).  

Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Un groupe de ressources doit être créé avant les machines virtuelles. Dans cet exemple, un groupe de ressources nommé *myResourceGroupVM* est créé dans la région *eastus2*. USA Est 2 est l’une des régions Azure qui prend en charge les zones de disponibilité.

```azurecli 
az group create --name myResourceGroupVM --location eastus2
```

Le groupe de ressources est spécifié lors de la création ou de la modification d’une machine virtuelle, qui peut être vue dans cet article.

## <a name="create-virtual-machine"></a>Créer une machine virtuelle

Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm). 

Lorsque vous créez une machine virtuelle, plusieurs options sont disponibles, comme l’image de système d’exploitation, les informations d’identification d’administration ou le dimensionnement des disques. Dans cet exemple, une machine virtuelle nommée *myVM* est créée et exécute Ubuntu Server. La machine virtuelle est créée dans la zone de disponibilité *1*. Par défaut, la machine virtuelle est créée à la taille *Standard_DS1_v2*.

```azurecli-interactive
az vm create --resource-group myResourceGroupVM --name myVM --location eastus2 --image UbuntuLTS --generate-ssh-keys --zone 1
```

La création de la machine virtuelle peut prendre plusieurs minutes. Une fois la machine virtuelle créée, l’interface Azure CLI fournit des informations concernant cette machine virtuelle. Notez la valeur `zones` qui indique la zone de disponibilité dans laquelle la machine virtuelle s’exécute. 

```output
{
  "fqdns": "",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus2",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroupVM",
  "zones": "1"
}
```

## <a name="confirm-zone-for-managed-disk-and-ip-address"></a>Confirmer la zone du disque managé et de l’adresse IP

Lorsque la machine virtuelle est déployée dans une zone de disponibilité, un disque managé pour la machine virtuelle est créé dans la même zone de disponibilité. Par défaut, une adresse IP publique est également créée dans cette zone. Les exemples suivants obtiennent des informations sur ces ressources.

Pour vérifier que le disque managé de la machine virtuelle se trouve dans la zone de disponibilité, utilisez la commande [az vm show](/cli/azure/vm) pour retourner l’ID du disque. Dans cet exemple, l’ID du disque est stocké dans une variable qui est utilisée dans une étape ultérieure. 

```azurecli-interactive
osdiskname=$(az vm show -g myResourceGroupVM -n myVM --query "storageProfile.osDisk.name" -o tsv)
```
Vous pouvez désormais obtenir des informations sur le disque géré :

```azurecli-interactive
az disk show --resource-group myResourceGroupVM --name $osdiskname
```

Le résultat montre que le disque géré se trouve dans la même zone de disponibilité que la machine virtuelle :

```output
{
  "creationData": {
    "createOption": "FromImage",
    "imageReference": {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westeurope/Publishers/Canonical/ArtifactTypes/VMImage/Offers/UbuntuServer/Skus/16.04-LTS/Versions/latest",
      "lun": null
    },
    "sourceResourceId": null,
    "sourceUri": null,
    "storageAccountId": null
  },
  "diskSizeGb": 30,
  "encryptionSettings": null,
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/disks/osdisk_761c570dab",
  "location": "eastus2",
  "managedBy": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/virtualMachines/myVM",
  "name": "myVM_osdisk_761c570dab",
  "osType": "Linux",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroupVM",
  "sku": {
    "name": "Premium_LRS",
    "tier": "Premium"
  },
  "tags": {},
  "timeCreated": "2018-03-05T22:16:06.892752+00:00",
  "type": "Microsoft.Compute/disks",
  "zones": [
    "1"
  ]
}
```

Utilisez la commande [az vm list-ip-addresses](/cli/azure/vm) pour renvoyer le nom de la ressource de l’adresse IP publique dans *myVM*. Dans cet exemple, le nom est stocké dans une variable qui est utilisée dans une étape ultérieure.

```azurecli
ipaddressname=$(az vm list-ip-addresses -g myResourceGroupVM -n myVM --query "[].virtualMachine.network.publicIpAddresses[].name" -o tsv)
```

Vous pouvez désormais obtenir des informations sur l’adresse IP :

```azurecli
az network public-ip show --resource-group myResourceGroupVM --name $ipaddressname
```

Le résultat montre que l’adresse IP se trouve dans la même zone de disponibilité que la machine virtuelle :

```output
{
  "dnsSettings": null,
  "etag": "W/\"b7ad25eb-3191-4c8f-9cec-c5e4a3a37d35\"",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Network/publicIPAddresses/myVMPublicIP",
  "idleTimeoutInMinutes": 4,
  "ipAddress": "52.174.34.95",
  "ipConfiguration": {
    "etag": null,
    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Network/networkInterfaces/myVMVMNic/ipConfigurations/ipconfigmyVM",
    "name": null,
    "privateIpAddress": null,
    "privateIpAllocationMethod": null,
    "provisioningState": null,
    "publicIpAddress": null,
    "resourceGroup": "myResourceGroupVM",
    "subnet": null
  },
  "location": "eastUS2",
  "name": "myVMPublicIP",
  "provisioningState": "Succeeded",
  "publicIpAddressVersion": "IPv4",
  "publicIpAllocationMethod": "Dynamic",
  "resourceGroup": "myResourceGroupVM",
  "resourceGuid": "8c70a073-09be-4504-0000-000000000000",
  "tags": {},
  "type": "Microsoft.Network/publicIPAddresses",
  "zones": [
    "1"
  ]
}
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à créer une machine virtuelle dans une zone de disponibilité. Apprenez-en davantage sur la [disponibilité](../availability.md) des machines virtuelles Azure.