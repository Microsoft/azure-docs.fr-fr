---
title: Créer et chiffrer une machine virtuelle Linux avec Azure CLI
description: Dans ce guide de démarrage rapide, vous allez apprendre à utiliser Azure CLI pour créer et chiffrer une machine virtuelle Linux
author: msmbaldwin
ms.author: mbaldwin
ms.service: virtual-machines-linux
ms.subservice: security
ms.topic: quickstart
ms.date: 05/17/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 3a9c859c32ef9ad387c56d5fc20f2173ee14fac5
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91975176"
---
# <a name="quickstart-create-and-encrypt-a-linux-vm-with-the-azure-cli"></a>Démarrage rapide : Créer et chiffrer une machine virtuelle Linux avec Azure CLI

L’interface de ligne de commande (CLI) Azure permet de créer et gérer des ressources Azure à partir de la ligne de commande ou dans les scripts. Ce démarrage rapide explique comment utiliser Azure CLI pour créer et chiffrer une machine virtuelle Linux.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Si vous choisissez d’installer et d’utiliser Azure CLI localement, vous devez exécuter Azure CLI version 2.0.30 ou ultérieure pour ce guide de démarrage rapide. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group?view=azure-cli-latest#az-group-create). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli-interactive
az group create --name "myResourceGroup" --location "eastus"
```

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm?view=azure-cli-latest#az-vm-create). L’exemple suivant permet de créer une machine virtuelle nommée *myVM*.

```azurecli-interactive
az vm create \
    --resource-group "myResourceGroup" \
    --name "myVM" \
    --image "Canonical:UbuntuServer:16.04-LTS:latest" \
    --size "Standard_D2S_V3"\
    --generate-ssh-keys
```

La création de la machine virtuelle et des ressources de support ne nécessite que quelques minutes. L’exemple de sortie suivant illustre la réussite de l’opération de création d’une machine virtuelle.

```
{
  "fqdns": "",
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroup"
}
```

## <a name="create-a-key-vault-configured-for-encryption-keys"></a>Créer un coffre de clés configuré pour les clés de chiffrement

Azure Disk Encryption stocke sa clé de chiffrement dans Azure Key Vault. Créez un coffre de clés avec la commande [az keyvault create](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-create). Pour activer le stockage des clés de chiffrement dans Key Vault, utilisez le paramètre --enabled-for-disk-encryption.

> [!Important]
> Chaque coffre de clés doit avoir un nom unique dans Azure. Dans les exemples ci-dessous, remplacez <votre-nom-de-coffre-de-clés-unique> par le nom de votre choix.

```azurecli-interactive
az keyvault create --name "<your-unique-keyvault-name>" --resource-group "myResourceGroup" --location "eastus" --enabled-for-disk-encryption
```

## <a name="encrypt-the-virtual-machine"></a>Chiffrer la machine virtuelle

Chiffrez votre machine virtuelle avec [az vm encryption](/cli/azure/vm/encryption?view=azure-cli-latest), en fournissant votre coffre de clés unique nom dans le paramètre --disk-encryption-keyvault.

```azurecli-interactive
az vm encryption enable -g "MyResourceGroup" --name "myVM" --disk-encryption-keyvault "<your-unique-keyvault-name>"
```

Après quelques instants, le processus retournera, « The encryption request was accepted. Please use 'show' command to monitor the progress. ». La commande « show » mentionnée est [az vm show](/cli/azure/vm/encryption#az-vm-encryption-show).

```azurecli-interactive
az vm encryption show --name "myVM" -g "MyResourceGroup"
```

Lorsque le chiffrement est activé, vous verrez les éléments suivants dans la sortie retournée :

```
"EncryptionOperation": "EnableEncryption"
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, vous pouvez utiliser la commande [az group delete](/cli/azure/group) pour supprimer le groupe de ressources, la machine virtuelle et Key Vault. 

```azurecli-interactive
az group delete --name "myResourceGroup"
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé une machine virtuelle, créé un coffre de clés qui a été activé pour les clés de chiffrement, et vous avez chiffré la machine virtuelle.  Passez à l’article suivant afin d’en savoir plus sur Azure Disk Encryption pour les machines virtuelles Linux.

> [!div class="nextstepaction"]
> [Vue d’ensemble d’Azure Disk Encryption](disk-encryption-overview.md)
