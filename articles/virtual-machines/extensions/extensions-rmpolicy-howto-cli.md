---
title: Utiliser Azure Policy pour restreindre l’installation d’extensions de machines virtuelles (Linux)
description: Découvrez comment utiliser Azure Policy pour restreindre les déploiements d’extensions de machines virtuelles.
services: virtual-machines-linux
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
ms.service: virtual-machines-linux
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/23/2018
ms.author: akjosh
ms.reviewer: cynthn
ms.openlocfilehash: 26874c33c496b57bf6317a7b837b3afddc2f5e37
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94955648"
---
# <a name="use-azure-policy-to-restrict-extensions-installation-on-linux-vms"></a>Utiliser Azure Policy pour restreindre l’installation d’extensions sur les machines virtuelles Linux

Si vous souhaitez empêcher l’utilisation ou l’installation de certaines extensions sur vos machines virtuelles Linux, vous pouvez créer une définition de stratégie Azure à l’aide de l’interface de ligne de commande (CLI) afin de restreindre les extensions pour les machines virtuelles d’un groupe de ressources. 

Ce didacticiel utilise l’interface CLI disponible dans Azure Cloud Shell, qui est constamment mise à jour vers la dernière version. Pour exécuter l’interface Azure CLI localement, vous devez installer la version 2.0.26 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli). 

## <a name="create-a-rules-file"></a>Créer un fichier de règles

Pour restreindre les extensions qui peuvent être installées, vous devez disposer d’une [règle](../../governance/policy/concepts/definition-structure.md#policy-rule) qui fournit la logique permettant d’identifier l’extension.

Cet exemple montre comment interdire l’installation des extensions publiées par « Microsoft.OSTCExtensions » en créant un fichier de règles dans Azure Cloud Shell. Cependant, si vous utilisez l’interface CLI localement, vous pouvez également créer un fichier local et remplacer le chemin d’accès (~/clouddrive) par le chemin d’accès au fichier local sur votre ordinateur.

Dans un [interpréteur de commandes Bash Cloud Shell](https://shell.azure.com/bash), entrez :

```bash
vim ~/clouddrive/azurepolicy.rules.json
```

Copiez et collez le code .json suivant dans le fichier.

```json
{
    "if": {
        "allOf": [
            {
                "field": "type",
                "equals": "Microsoft.OSTCExtensions/virtualMachines/extensions"
            },
            {
                "field": "Microsoft.OSTCExtensions/virtualMachines/extensions/publisher",
                "equals": "Microsoft.OSTCExtensions"
            },
            {
                "field": "Microsoft.OSTCExtensions/virtualMachines/extensions/type",
                "in": "[parameters('notAllowedExtensions')]"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```

Quand vous avez terminé, appuyez sur la touche **Échap**, puis tapez **: wq** pour enregistrer et fermer le fichier.


## <a name="create-a-parameters-file"></a>Créer un fichier de paramètres

Vous avez également besoin d’un fichier de [paramètres](../../governance/policy/concepts/definition-structure.md#parameters) qui crée une structure que vous pourrez utiliser pour transmettre la liste des extensions à bloquer. 

Cet exemple montre comment créer un fichier de paramètres pour les machines virtuelles Linux dans Cloud Shell. Cependant, si vous utilisez l’interface CLI localement, vous pouvez également créer un fichier local et remplacer le chemin d’accès (~/clouddrive) par le chemin d’accès au fichier local sur votre ordinateur.

Dans l’[interpréteur de commandes Bash Cloud Shell](https://shell.azure.com/bash), entrez :

```bash
vim ~/clouddrive/azurepolicy.parameters.json
```

Copiez et collez le code .json suivant dans le fichier.

```json
{
    "notAllowedExtensions": {
        "type": "Array",
        "metadata": {
            "description": "The list of extensions that will be denied. Example: CustomScriptForLinux, VMAccessForLinux etc.",
            "displayName": "Denied extension"
        }
    }
}
```

Quand vous avez terminé, appuyez sur la touche **Échap**, puis tapez **: wq** pour enregistrer et fermer le fichier.

## <a name="create-the-policy"></a>Création de la stratégie

Une définition de stratégie est un objet servant à stocker la configuration que vous souhaitez utiliser. La définition de stratégie fait appel aux fichiers de règles et de paramètres pour définir la stratégie. Créez la définition de stratégie à l’aide de la commande [az policy definition create](/cli/azure/role/assignment?view=azure-cli-latest).

Dans cet exemple, les règles et les paramètres correspondent aux fichiers que vous avez créés et stockés en tant que fichiers .json dans votre Cloud Shell.

```azurecli-interactive
az policy definition create \
   --name 'not-allowed-vmextension-linux' \
   --display-name 'Block VM Extensions' \
   --description 'This policy governs which VM extensions that are blocked.' \
   --rules '~/clouddrive/azurepolicy.rules.json' \
   --params '~/clouddrive/azurepolicy.parameters.json' \
   --mode All
```


## <a name="assign-the-policy"></a>Affecter la stratégie

Dans cet exemple, la stratégie est affectée à un groupe de ressources à l’aide de la commande [az policy assignment create](/cli/azure/policy/assignment). N’importe quelle machine virtuelle créée dans le groupe de ressources **myResourceGroup** ne pourra pas installer les extensions VM Access For Linux et Custom Script For Linux. Pour que la stratégie puisse être affectée, le groupe de ressources doit déjà exister.

Utilisez la commande [az account list](/cli/azure/account?view=azure-cli-latest) pour obtenir l’ID d’abonnement à utiliser à la place de celui de l’exemple.


```azurecli-interactive
az policy assignment create \
   --name 'not-allowed-vmextension-linux' \
   --scope /subscriptions/<subscription Id>/resourceGroups/myResourceGroup \
   --policy "not-allowed-vmextension-linux" \
   --params '{
        "notAllowedExtensions": {
            "value": [
                "VMAccessForLinux",
                "CustomScriptForLinux"
            ]
        }
    }'
```

## <a name="test-the-policy"></a>Tester la stratégie

Testez la stratégie en créant une machine virtuelle et en essayant d’ajouter un nouvel utilisateur.


```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --generate-ssh-keys
```

Essayez de créer un utilisateur nommé **myNewUser** à l’aide de l’extension VM Access.

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser \
  --password 'mynewuserpwd123!'
```



## <a name="remove-the-assignment"></a>Supprimer l’affectation

```azurecli-interactive
az policy assignment delete --name 'not-allowed-vmextension-linux' --resource-group myResourceGroup
```
## <a name="remove-the-policy"></a>Supprimer la stratégie

```azurecli-interactive
az policy definition delete --name 'not-allowed-vmextension-linux'
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Présentation d’Azure Policy](../../governance/policy/overview.md).
