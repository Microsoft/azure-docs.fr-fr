---
title: Chiffrer des disques pour des groupes identiques Azure avec Azure CLI
description: Découvrez comment utiliser Azure CLI pour chiffrer des instances de machine virtuelle et les disques attachés dans un groupe de machines virtuelles identiques Windows
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 10/15/2019
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: 6edfa1beb568bb05bd0f3f1ef9e7792ac3c3cbe2
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94515742"
---
# <a name="encrypt-os-and-attached-data-disks-in-a-virtual-machine-scale-set-with-the-azure-cli"></a>Chiffrer des disques de système d’exploitation et de données attachés dans un groupe de machines virtuelles identiques avec l’interface Azure CLI

L’interface de ligne de commande (CLI) Azure permet de créer et gérer des ressources Azure à partir de la ligne de commande ou dans les scripts. Ce guide de démarrage rapide explique comment utiliser Azure CLI pour créer et chiffrer un groupe de machines virtuelles identiques. Pour plus d’informations sur l’application d’Azure Disk Encryption à un groupe de machines virtuelles identiques, consultez [Azure Disk Encryption pour les groupes de machines virtuelles identiques](disk-encryption-overview.md).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.31 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-scale-set"></a>Créer un groupe identique

Pour pouvoir créer un groupe identique, vous devez créer un groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Créez à présent un groupe de machines virtuelles identiques avec [az vmss create](/cli/azure/vmss). L’exemple suivant crée un groupe identique nommé *myScaleSet* qui est définit pour une mise à jour automatique lorsque des modifications sont appliquées, et qui génère des clés SSH s’il n’en existe pas dans *~/.ssh/id_rsa*. Un disque de données de 32 Go est attaché à chaque instance de machine virtuelle, et [l’extension de script personnalisé](../virtual-machines/extensions/custom-script-linux.md) Azure est utilisée pour préparer les disques de données avec [az vmss extension set](/cli/azure/vmss/extension) :

```azurecli-interactive
# Create a scale set with attached data disk
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys \
  --data-disk-sizes-gb 32

# Prepare the data disk for use with the Custom Script Extension
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/prepare_vm_disks.sh"],"commandToExecute":"./prepare_vm_disks.sh"}'
```

La création et la configuration des l’ensemble des ressources et des machines virtuelles du groupe identique prennent quelques minutes.

## <a name="create-an-azure-key-vault-enabled-for-disk-encryption"></a>Créer un coffre de clés Azure activé pour le chiffrement de disque

Un coffre de clés Azure peut stocker des clés, des clés secrètes ou des mots de passe vous permettant de les implémenter en toute sécurité dans vos applications et services. Les clés de chiffrement sont stockées dans le coffre de clés Azure à l’aide d’une protection logicielle, mais vous pouvez importer ou générer vos clés dans des modules de sécurité matériels (HSM) certifiés conformes aux normes FIPS 140-2 de niveau 2. Ces clés de chiffrement servent à chiffrer et à déchiffrer les disques virtuels connectés à votre machine virtuelle. Vous gardez le contrôle de ces clés de chiffrement et pouvez effectuer un audit de leur utilisation.

Définissez un nom de coffre de clés (*keyvault_name*) unique. Créez ensuite un coffre de clés avec [az keyvault create](/cli/azure/ext/keyvault-preview/keyvault#ext-keyvault-preview-az-keyvault-create) dans le même abonnement et la même région que le groupe identique, puis définissez la stratégie d’accès *--enabled-for-disk-encryption*.

```azurecli-interactive
# Provide your own unique Key Vault name
keyvault_name=myuniquekeyvaultname

# Create Key Vault
az keyvault create --resource-group myResourceGroup --name $keyvault_name --enabled-for-disk-encryption
```

### <a name="use-an-existing-key-vault"></a>Utiliser un coffre de clés existant

Cette étape est requise uniquement si vous souhaitez utiliser un coffre de clés existant avec le chiffrement de disque. Ignorez cette étape si vous avez créé un coffre de clés dans la section précédente.

Définissez un nom de coffre de clés (*keyvault_name*) unique. Mettez ensuite à jour votre coffre de clés avec [az keyvault update](/cli/azure/ext/keyvault-preview/keyvault#ext-keyvault-preview-az-keyvault-update), puis définissez la stratégie d’accès *--enabled-for-disk-encryption*.

```azurecli-interactive
# Provide your own unique Key Vault name
keyvault_name=myuniquekeyvaultname

# Create Key Vault
az keyvault update --name $keyvault_name --enabled-for-disk-encryption
```

## <a name="enable-encryption"></a>Activer le chiffrement

Pour chiffrer des instances de machine virtuelle dans un groupe identique, vous devez d’abord obtenir certaines informations sur l’ID de ressource du coffre de clés avec [az keyvault show](/cli/azure/ext/keyvault-preview/keyvault#ext-keyvault-preview-az-keyvault-show). Les variables obtenues sont ensuite utilisées pour démarrer le processus de chiffrement avec [az vmss encryption enable](/cli/azure/vmss/encryption#az-vmss-encryption-enable) :

```azurecli-interactive
# Get the resource ID of the Key Vault
vaultResourceId=$(az keyvault show --resource-group myResourceGroup --name $keyvault_name --query id -o tsv)

# Enable encryption of the data disks in a scale set
az vmss encryption enable \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --disk-encryption-keyvault $vaultResourceId \
    --volume-type DATA
```

Le démarrage du processus de chiffrement peut prendre une ou deux minutes.

Si la stratégie de mise à niveau sur le groupe identique créé précédemment a la valeur *automatic*, les instances de machine virtuelle démarrent automatiquement le processus de chiffrement. Sur les groupes identiques où la stratégie de mise à niveau est en mode manuel, démarrez la stratégie de chiffrement sur les instances de machine virtuelle avec [az vmss update-instances](/cli/azure/vmss#az-vmss-update-instances).

### <a name="enable-encryption-using-kek-to-wrap-the-key"></a>Activer le chiffrement à l’aide de KEK pour encapsuler la clé

Vous pouvez également utiliser une clé de chiffrement de clé pour renforcer la sécurité lors du chiffrement du groupe de machines virtuelles identiques.

```azurecli-interactive
# Get the resource ID of the Key Vault
vaultResourceId=$(az keyvault show --resource-group myResourceGroup --name $keyvault_name --query id -o tsv)

# Enable encryption of the data disks in a scale set
az vmss encryption enable \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --disk-encryption-keyvault $vaultResourceId \
    --key-encryption-key myKEK \
    --key-encryption-keyvault $vaultResourceId \
    --volume-type DATA
```

> [!NOTE]
>  La syntaxe de la valeur du paramètre disk-encryption-keyvault est la chaîne de l’identificateur complet :</br>
/subscriptions/[guid-id-abonnement]/resourceGroups/[nom-groupe-ressources]/providers/Microsoft.KeyVault/vaults/[nom-coffre-clés]</br></br>
> La syntaxe de la valeur du paramètre key-encryption-key est l’URI complet vers la clé KEK comme dans :</br>
https://[nom-coffre-clés].vault.azure.net/keys/[nom-clé-kek]/[id-clé-kek-unique]

## <a name="check-encryption-progress"></a>Vérifier la progression du chiffrement

Pour vérifier l’état du chiffrement de disque, utilisez [az vmss encryption show](/cli/azure/vmss/encryption#az-vmss-encryption-show) :

```azurecli-interactive
az vmss encryption show --resource-group myResourceGroup --name myScaleSet
```

Une fois les instances de machine virtuelle chiffrées, le code d’état indique *EncryptionState/encrypted* comme dans l’exemple de sortie suivant :

```console
[
  {
    "disks": [
      {
        "encryptionSettings": null,
        "name": "myScaleSet_myScaleSet_0_disk2_3f39c2019b174218b98b3dfae3424e69",
        "statuses": [
          {
            "additionalProperties": {},
            "code": "EncryptionState/encrypted",
            "displayStatus": "Encryption is enabled on disk",
            "level": "Info",
            "message": null,
            "time": null
          }
        ]
      }
    ],
    "id": "/subscriptions/guid/resourceGroups/MYRESOURCEGROUP/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/virtualMachines/0",
    "resourceGroup": "MYRESOURCEGROUP"
  }
]
```

## <a name="disable-encryption"></a>Désactiver le chiffrement

Si vous ne souhaitez plus utiliser les disques d’instances de machine virtuelle chiffrés, désactivez le chiffrement avec [az vmss encryption disable](/cli/azure/vmss/encryption?view=azure-cli-latest#az-vmss-encryption-disable), comme ceci :

```azurecli-interactive
az vmss encryption disable --resource-group myResourceGroup --name myScaleSet
```

## <a name="next-steps"></a>Étapes suivantes

- Dans cet article, vous avez utilisé l’interface Azure CLI pour chiffrer un groupe de machines virtuelles identiques. Vous pouvez également utiliser [Azure PowerShell](disk-encryption-powershell.md) ou des [modèles Azure Resource Manager](disk-encryption-azure-resource-manager.md).
- Si vous souhaitez qu’Azure Disk Encryption soit appliqué après l’approvisionnement d’une autre extension, vous pouvez utiliser le [séquencement d’extensions](virtual-machine-scale-sets-extension-sequencing.md). 
- Un exemple de fichier de traitement complet de chiffrement de disque de données d’un groupe identique Linux est disponible [ici](https://gist.githubusercontent.com/ejarvi/7766dad1475d5f7078544ffbb449f29b/raw/03e5d990b798f62cf188706221ba6c0c7c2efb3f/enable-linux-vmss.bat). Cet exemple crée un groupe de ressources, un groupe identique Linux, monte un disque de données de 5 Go et chiffre le groupe de machines virtuelles identiques.
