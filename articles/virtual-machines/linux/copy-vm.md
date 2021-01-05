---
title: Copier une machine virtuelle Linux avec Azure CLI
description: Apprenez à créer une copie de votre machine virtuelle Azure Linux à l’aide d’Azure CLI et de la fonctionnalité Disques managés.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 10/17/2018
ms.author: cynthn
ms.custom: legacy, devx-track-azurecli
ms.openlocfilehash: f92f286fc9d9438331617cb567272a331834af42
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92735384"
---
# <a name="create-a-copy-of-a-linux-vm-by-using-azure-cli-and-managed-disks"></a>Créer une copie de machine virtuelle Linux à l’aide d’Azure CLI et de la fonctionnalité Disques managés

Cet article explique comment créer une copie de votre machine virtuelle Azure (VM) exécutée dans Linux en utilisant Azure CLI. Pour copier, créer, stocker et partager des images de machine virtuelle à grande échelle, consultez [Galeries d’images partagées](../shared-images-cli.md).

Vous pouvez également [charger et créer une machine virtuelle à partir d’un disque dur virtuel](upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="prerequisites"></a>Prérequis

-   Installez [Azure CLI](/cli/azure/install-az-cli2).

-   Connectez-vous à un compte Azure avec [az login](/cli/azure/reference-index#az-login).

-   Ayez une machine virtuelle Azure à utiliser en tant que source pour votre copie.

## <a name="stop-the-source-vm"></a>Arrêter la machine virtuelle source

Libérez la machine virtuelle source à l’aide de la commande [az vm deallocate](/cli/azure/vm#az-vm-deallocate).
L’exemple suivant libère la machine virtuelle nommée *myVM* dans le groupe de ressources *myResourceGroup*  :

```azurecli
az vm deallocate \
    --resource-group myResourceGroup \
    --name myVM
```

## <a name="copy-the-source-vm"></a>Copier la machine virtuelle source

Pour copier une machine virtuelle, vous devez créer une copie du disque dur virtuel sous-jacent. Ce processus crée un disque dur virtuel spécialisé (VHD) en tant que disque managé qui contient la même configuration et les mêmes paramètres que la machine virtuelle source.

Pour plus d’informations sur Azure Disques managés, consultez [Vue d’ensemble d’Azure Disques managés](../managed-disks-overview.md). 

1.  Répertoriez chaque machine virtuelle et le nom de leur disque de système d’exploitation avec la commande [az vm list](/cli/azure/vm#az-vm-list). L’exemple suivant répertorie toutes les machines virtuelles dans le groupe de ressources nommé *myResourceGroup* :
    
    ```azurecli
    az vm list -g myResourceGroup \
         --query '[].{Name:name,DiskName:storageProfile.osDisk.name}' \
         --output table
    ```

    Le résultat ressemble à l’exemple suivant :

    ```azurecli
    Name    DiskName
    ------  --------
    myVM    myDisk
    ```

1.  Copiez le disque en créant un nouveau disque managé avec la commande [az disk create](/cli/azure/disk#az-disk-create). L’exemple suivant crée un disque nommé *myCopiedDisk* à partir du disque géré nommé *myDisk* :

    ```azurecli
    az disk create --resource-group myResourceGroup \
         --name myCopiedDisk --source myDisk
    ``` 

1.  Examinez les disques managés qui appartiennent désormais à votre groupe de ressources, à l’aide de la commande [az disk list](/cli/azure/disk#az-disk-list). L’exemple suivant répertorie les disques managés dans le groupe de ressources nommé *myResourceGroup* :

    ```azurecli
    az disk list --resource-group myResourceGroup --output table
    ```


## <a name="set-up-a-virtual-network"></a>Configuration d’un réseau virtuel

Les étapes facultatives suivantes consistent à créer un nouveau réseau virtuel, un nouveau sous-réseau, une nouvelle adresse IP publique et une nouvelle carte réseau virtuelle.

Si vous copiez une machine virtuelle à des fins de dépannage ou dans le cadre de déploiements supplémentaires, vous pouvez ne pas vouloir utiliser une machine virtuelle hébergée dans un réseau virtuel existant.

Si vous souhaitez créer une infrastructure de réseau virtuel pour vos machines virtuelles copiées, exécutez les procédures suivantes. Si vous ne souhaitez pas créer de réseau virtuel, passez à [Créer une machine virtuelle](#create-a-vm).

1.  Exécutez la commande [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create) pour créer le réseau virtuel. L’exemple suivant permet de créer un réseau virtuel nommé *myVnet* et un sous-réseau nommé *mySubnet*  :

    ```azurecli
    az network vnet create --resource-group myResourceGroup \
        --location eastus --name myVnet \
        --address-prefix 192.168.0.0/16 \
        --subnet-name mySubnet \
        --subnet-prefix 192.168.1.0/24
    ```

1.  Exécutez la commande [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) pour créer une adresse IP publique. L’exemple suivant permet de créer une adresse IP publique nommée *myPublicIP* avec le nom DNS de *mypublicdns* . (Comme le nom DNS doit être unique, fournissez un nom unique.)

    ```azurecli
    az network public-ip create --resource-group myResourceGroup \
        --location eastus --name myPublicIP --dns-name mypublicdns \
        --allocation-method static --idle-timeout 4
    ```

1.  Exécutez la commande [az network nic create](/cli/azure/network/nic#az-network-nic-create) pour créer la carte réseau.
    L’exemple suivant permet de créer une carte réseau nommée *myNic* et associée au sous-réseau *mySubnet*  :

    ```azurecli
    az network nic create --resource-group myResourceGroup \
        --location eastus --name myNic \
        --vnet-name myVnet --subnet mySubnet \
        --public-ip-address myPublicIP
    ```

## <a name="create-a-vm"></a>Créer une machine virtuelle

Exécutez la commande [az vm create](/cli/azure/vm#az-vm-create) pour créer une machine virtuelle.

Spécifiez le disque géré copié à utiliser en tant que disque du système d’exploitation (`--attach-os-disk`), tel que défini ci-dessous :

```azurecli
az vm create --resource-group myResourceGroup \
    --name myCopiedVM --nics myNic \
    --size Standard_DS1_v2 --os-type Linux \
    --attach-os-disk myCopiedDisk
```

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment utiliser une [galerie d’images partagée](../shared-images-cli.md) pour gérer les images de machine virtuelle.
