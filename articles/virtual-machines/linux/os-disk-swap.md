---
title: Échange entre les disques du système d’exploitation à l’aide de l’interface CLI
description: Utilisez l’interface CLI pour remplacer le disque du système d’exploitation utilisé par une machine virtuelle Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: disks
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 04/24/2018
ms.author: cynthn
ms.openlocfilehash: adcbc5b0ba0056c79576e8d4c8ded2150b206513
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87371870"
---
# <a name="change-the-os-disk-used-by-an-azure-vm-using-the-cli"></a>Remplacer le disque du système d’exploitation utilisé par une machine virtuelle Azure à l’aide de l’interface CLI


Pour remplacer le disque d’une machine virtuelle existante par un disque de sauvegarde ou un autre disque de système d’exploitation, vous pouvez utiliser l’interface CLI Azure. Il est inutile de supprimer et de recréer la machine virtuelle. Vous pouvez même utiliser le disque managé d’un autre groupe de ressources s’il n’est pas déjà utilisé.

Vous devez arrêter/libérer la machine virtuelle, puis remplacer l’ID de ressource du disque managé par celui d’un autre disque managé. 

Vérifiez que la taille et le type de stockage de la machine virtuelle sont compatibles avec le disque à attacher. Ainsi, si le disque que vous souhaitez utiliser bénéficie d’un stockage Premium, la machine virtuelle doit être de taille suffisante pour prendre en charge ce niveau de stockage (comme une série DS).

Cet article requiert Azure CLI version 2.0.25 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli). 


Utilisez [az disk list](/cli/azure/disk) pour obtenir la liste des disques de votre groupe de ressources.

```azurecli-interactive
az disk list \
   -g myResourceGroupDisk \
   --query '[*].{diskId:id}' \
   --output table
```


Utilisez [az vm stop](/cli/azure/vm) pour arrêter\libérer la machine virtuelle avant d’échanger les disques.

```azurecli-interactive
az vm stop \
   -n myVM \
   -g myResourceGroup
```


Utilisez [az vm update](/cli/azure/vm#az-vm-update) avec l’ID complet de la ressource du nouveau disque pour le paramètre `--osdisk` 

```azurecli-interactive 
az vm update \
   -g myResourceGroup \
   -n myVM \
   --os-disk /subscriptions/<subscription ID>/resourceGroups/swap/providers/Microsoft.Compute/disks/myDisk 
   ```
   
Redémarrez la machine virtuelle avec [az vm start](/cli/azure/vm).

```azurecli-interactive
az vm start \
   -n myVM \
   -g myResourceGroup
```

   
**Étapes suivantes**

Pour créer une copie d’un disque, consultez [Effectuer la capture instantanée d’un disque](snapshot-copy-managed-disk.md).
