---
title: Convertir une machine virtuelle Linux pour transformer les disques non managés en disques managés
description: Découvrez comment convertir une machine virtuelle Linux pour transformer les disques non managés en disques managés à l’aide d’Azure CLI.
author: roygara
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 12/15/2017
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: 89035d26e05ca4b93dc271f02fef1afea24b0f20
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88815332"
---
# <a name="convert-a-linux-virtual-machine-from-unmanaged-disks-to-managed-disks"></a>Convertir les disques non managés d’une machine virtuelle Linux en disques managés

Si vos machines virtuelles Linux existantes utilisent des disques non managés, vous pouvez les convertir pour qu’elles utilisent [Azure Disques managés](../managed-disks-overview.md). Ce processus convertit le disque du système d’exploitation ainsi que tous les autres disques de données attachés.

Cet article explique comment convertir des machines virtuelles avec Azure CLI. Si vous devez installer ou mettre à niveau l’interface, consultez [Installer Azure CLI](/cli/azure/install-azure-cli). 

## <a name="before-you-begin"></a>Avant de commencer
* Consultez les [questions fréquentes (FAQ) sur la migration vers la fonctionnalité Disques managés](../faq-for-disks.md#migrate-to-managed-disks).

[!INCLUDE [virtual-machines-common-convert-disks-considerations](../../../includes/virtual-machines-common-convert-disks-considerations.md)]

* Les disques durs virtuels et le compte de stockage d’origine utilisé par la machine virtuelle avant la conversion ne sont pas supprimés. Ils continuent à être facturés. Pour éviter la facturation de ces artefacts, supprimez les objets BLOB du disque dur virtuel d’origine après avoir vérifié que la conversion est terminée. Si vous avez besoin de trouver ces disques libres afin de les supprimer, consultez notre article [Rechercher et supprimer les disques managés et non managés Azure non attachés](find-unattached-disks.md).

## <a name="convert-single-instance-vms"></a>Convertir des machines virtuelles à instance unique
Cette section explique comment convertir vos machines virtuelles Azure à instance unique à partir de disques non managés vers des disques managés. (Si vos machines virtuelles sont dans un groupe à haute disponibilité, voir la section suivante.) Vous pouvez utiliser ce processus pour convertir les machines virtuelles qui utilisent des disques non managés premium (SSD) afin qu’elles utilisent des disques managés premium, ou qui utilisent des disques non managés standard (HDD) afin qu’elles utilisent des disques managés standard.

1. Libérez la machine virtuelle à l’aide de [az vm deallocate](/cli/azure/vm). L’exemple suivant libère la machine virtuelle nommée `myVM` dans le groupe de ressources nommé `myResourceGroup` :

    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    ```

2. Convertissez la machine virtuelle afin qu’elle utilise des disques managés à l’aide de la commande [az vm convert](/cli/azure/vm). Le processus suivant convertit la machine virtuelle nommée `myVM`, y compris le disque du système d’exploitation et tous les disques de données :

    ```azurecli
    az vm convert --resource-group myResourceGroup --name myVM
    ```

3. Démarrez la machine virtuelle une fois la conversion vers les disques managés effectuée à l’aide de la commande [az vm start](/cli/azure/vm). L’exemple suivant démarre la machine virtuelle nommée `myVM` dans le groupe de ressources nommé `myResourceGroup`.

    ```azurecli
    az vm start --resource-group myResourceGroup --name myVM
    ```

## <a name="convert-vms-in-an-availability-set"></a>Convertir des machines virtuelles dans un groupe à haute disponibilité

Si les machines virtuelles que vous souhaitez convertir pour utiliser des disques managés se trouvent dans un groupe à haute disponibilité, vous devez tout d’abord convertir ce dernier en groupe à haute disponibilité géré.

Toutes les machines virtuelles dans le groupe à haute disponibilité doivent être libérées avant de convertir le groupe à haute disponibilité. Prévoyez de convertir toutes les machines virtuelles vers des disques managés une fois que le groupe à haute disponibilité a lui-même été converti en groupe à haute disponibilité géré. Vous pouvez ensuite démarrer toutes les machines virtuelles et poursuivre un fonctionnement normal.

1. Répertoriez toutes les machines virtuelles contenues dans un groupe à haute disponibilité à l’aide de la commande [az vm availability-set list](/cli/azure/vm/availability-set). L’exemple suivant répertorie toutes les machines virtuelles dans le groupe à haute disponibilité nommé `myAvailabilitySet` dans le groupe de ressources nommé `myResourceGroup` :

    ```azurecli
    az vm availability-set show \
        --resource-group myResourceGroup \
        --name myAvailabilitySet \
        --query [virtualMachines[*].id] \
        --output table
    ```

2. Libérez toutes les machines virtuelles à l’aide de la commande [az vm deallocate](/cli/azure/vm). L’exemple suivant libère la machine virtuelle nommée `myVM` dans le groupe de ressources nommé `myResourceGroup` :

    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    ```

3. Convertissez le groupe à haute disponibilité à l’aide de la commande [az vm availability-set convert](/cli/azure/vm/availability-set). L’exemple suivant convertit le groupe à haute disponibilité nommé `myAvailabilitySet` dans le groupe de ressources nommé `myResourceGroup` :

    ```azurecli
    az vm availability-set convert \
        --resource-group myResourceGroup \
        --name myAvailabilitySet
    ```

4. Convertissez toutes les machines virtuelles vers des disques managés à l’aide de la commande [az vm convert](/cli/azure/vm). Le processus suivant convertit la machine virtuelle nommée `myVM`, y compris le disque du système d’exploitation et tous les disques de données :

    ```azurecli
    az vm convert --resource-group myResourceGroup --name myVM
    ```

5. Démarrez toutes les machines virtuelles une fois la conversion vers des disques managés effectuée à l’aide de la commande [az vm start](/cli/azure/vm). L’exemple suivant démarre la machine virtuelle nommée `myVM` dans le groupe de ressources nommé `myResourceGroup` :

    ```azurecli
    az vm start --resource-group myResourceGroup --name myVM
    ```

## <a name="convert-using-the-azure-portal"></a>Convertir à l’aide du portail Azure

Vous pouvez également convertir des disques non managés en disques managés à l’aide du portail Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez la machine virtuelle dans la liste des machines virtuelles sur le portail.
3. Dans le panneau de la machine virtuelle, sélectionnez **Disques** dans le menu.
4. En haut du panneau **Disques**, sélectionnez **Migrer vers des disques managés**.
5. Si votre machine virtuelle se trouve dans un groupe à haute disponibilité, un avertissement apparaît sur le panneau **Migrer vers des disques managés**, indiquant que vous devez commencer par convertir le groupe à haute disponibilité. L’avertissement doit avoir un lien sur lequel vous pouvez cliquer pour convertir le groupe à haute disponibilité. Une fois le groupe à haute disponibilité converti, ou si votre machine virtuelle ne se trouve pas dans un groupe à haute disponibilité, cliquez sur **Migrate** pour démarrer le processus de migration de vos disques vers des disques managés.

Lorsque la migration est terminée, la machine virtuelle est arrêtée, puis redémarrée.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les options de stockage, voir la page [Vue d’ensemble d’Azure Disques managés](../managed-disks-overview.md).
