---
title: Guide pratique pour redimensionner une machine virtuelle Linux avec Azure CLI
description: Comment augmenter ou diminuer les capacités d’une machine virtuelle Linux, en en modifiant la taille.
author: DavidCBerry13
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 02/10/2017
ms.author: daberry
ms.openlocfilehash: a93b1dcfe803fdead0d741553c45894f1675372b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87372261"
---
# <a name="resize-a-linux-virtual-machine-using-azure-cli"></a>Redimensionner une machine virtuelle Linux avec Azure CLI 

Après avoir provisionné une machine virtuelle, vous pouvez la mettre à l’échelle en modifiant sa [taille][vm-sizes]. Dans certains cas, vous devez commencer par libérer la machine virtuelle. C’est le cas notamment lorsque la taille souhaitée n’est pas disponible sur le cluster matériel qui héberge la machine virtuelle. Cet article explique comment redimensionner une machine virtuelle Linux avec Azure CLI. 

## <a name="resize-a-vm"></a>Redimensionner une machine virtuelle
Pour redimensionner une machine virtuelle, vous devez installer la dernière version [d’Azure CLI](/cli/azure/install-az-cli2) et vous connecter à un compte Azure avec [az login](/cli/azure/reference-index).

1. Affichez la liste des tailles de machine virtuelle disponibles dans le cluster matériel qui héberge la machine virtuelle à l’aide de la commande [az vm list-vm-resize-options](/cli/azure/vm). L’exemple suivant répertorie les tailles de machine virtuelle pour la machine virtuelle nommée `myVM` dans la région du groupe de ressources `myResourceGroup` :
   
    ```azurecli
    az vm list-vm-resize-options --resource-group myResourceGroup --name myVM --output table
    ```

2. Si la taille de machine virtuelle souhaitée est répertoriée, redimensionnez la machine virtuelle avec la commande [az vm resize](/cli/azure/vm). L’exemple suivant redimensionne la machine virtuelle nommée `myVM` à la taille `Standard_DS3_v2` :
   
    ```azurecli
    az vm resize --resource-group myResourceGroup --name myVM --size Standard_DS3_v2
    ```
   
    La machine virtuelle redémarre pendant le processus. Après le redémarrage, votre système d’exploitation existant et les disques de données sont remappés. Tout le contenu du disque temporaire est perdu.

3. Si la taille de machine virtuelle souhaitée n’est pas répertoriée, vous devez d’abord libérer la machine virtuelle avec la commande [az vm deallocate](/cli/azure/vm). Ce processus permet de redimensionner ensuite la machine virtuelle à n’importe quelle taille disponible dans la région, puis de la redémarrer. Les étapes suivantes consistent à libérer, redimensionner, puis démarrer la machine virtuelle nommée `myVM` dans le groupe de ressources nommé `myResourceGroup` :
   
    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    az vm resize --resource-group myResourceGroup --name myVM --size Standard_DS3_v2
    az vm start --resource-group myResourceGroup --name myVM
    ```
   
   > [!WARNING]
   > Le fait de libérer la machine virtuelle libère également toutes les adresses IP dynamiques affectées à la machine virtuelle. Les disques de données et du système d’exploitation ne sont pas affectés.

## <a name="next-steps"></a>Étapes suivantes
Pour une évolutivité supplémentaire, exécutez plusieurs instances de machine virtuelle et effectuez un scale-out. Pour plus d’informations, consultez [Mettre automatiquement à l’échelle des machines Linux dans un groupe identique de machines virtuelles][scale-set]. 

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[scale-set]: ../../virtual-machine-scale-sets/tutorial-autoscale-cli.md 
[vm-sizes]:sizes.md
