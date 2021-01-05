---
title: Gérer des groupes de machines virtuelles identiques avec l’interface Azure CLI
description: Commandes Azure CLI courantes pour gérer des groupes de machines virtuelles identiques, par exemple pour démarrer et arrêter une instance ou changer la capacité d’un groupe identique.
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 05/29/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: d954f7cdda4cae65f822489828226e0364d0fc29
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91570535"
---
# <a name="manage-a-virtual-machine-scale-set-with-the-azure-cli"></a>Gérer un groupe de machines virtuelles identiques avec l’interface Azure CLI
Tout au long du cycle de vie du groupe de machines virtuelles identiques, vous devrez peut-être exécuter une ou plusieurs tâches de gestion. En outre, vous souhaiterez peut-être créer des scripts pour automatiser les diverses tâches liées au cycle de vie. Cet article décrit en détail certaines des commandes Azure CLI courantes qui vous permettent d’effectuer ces tâches.

Pour effectuer ces tâches de gestion, vous devez disposer de la dernière version d’Azure CLI. Pour plus d’informations, consultez [Installer l’interface Azure CLI](/cli/azure/install-azure-cli). Si vous avez besoin de créer un groupe de machines virtuelles identiques, vous pouvez [créer un groupe identique avec l’interface Azure CLI](quick-create-cli.md).


## <a name="view-information-about-a-scale-set"></a>Afficher des informations sur un groupe identique
Pour afficher les informations générales relatives à un groupe identique, utilisez la commande [az vmss show](/cli/azure/vmss). L’exemple suivant obtient des informations sur le groupe identique nommé *myScaleSet* dans le groupe de ressources *myResourceGroup*. Entrez vos propres noms, comme suit :

```azurecli
az vmss show --resource-group myResourceGroup --name myScaleSet
```


## <a name="view-vms-in-a-scale-set"></a>Afficher les machines virtuelles d’un groupe identique
Pour afficher une liste des instances de machine virtuelle dans un groupe identique, utilisez la commande [az vmss list-instances](/cli/azure/vmss). L’exemple suivant répertorie toutes les instances de machine virtuelle du groupe identique nommé *myScaleSet* dans le groupe de ressources *myResourceGroup*. Spécifiez vos propres valeurs pour ces noms :

```azurecli
az vmss list-instances \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --output table
```

Pour afficher des informations supplémentaires sur une instance spécifique de la machine virtuelle, ajoutez le paramètre `--instance-id` à la commande [az vmss get-instance-view](/cli/azure/vmss), et spécifiez une instance à afficher. L’exemple suivant affiche des informations sur l’instance de machine virtuelle *0* dans le groupe identique nommé *myScaleSet* et le groupe de ressources *myResourceGroup*. Entrez vos propres noms, comme suit :

```azurecli
az vmss get-instance-view \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --instance-id 0
```

Vous pouvez également obtenir des informations détaillées sur *instanceView* pour toutes les instances d’un appel d’API, ce qui peut aider à éviter la limitation des API pour les installations de grande taille. Indiquez vos propres valeurs pour `--resource-group`, `--subscription` et `--name`.

```azurecli
az vmss list-instances \
    --expand instanceView \
    --select instanceView \
    --resource-group <resourceGroupName> \
    --subscription <subID> \
    --name <vmssName>
```

```rest
GET "https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/virtualMachineScaleSets/<VMSSName>/virtualMachines?api-version=2019-03-01&%24expand=instanceView"
```

## <a name="list-connection-information-for-vms"></a>Répertorier les informations de connexion pour les machines virtuelles
Pour vous connecter aux machines virtuelles d’un groupe identique, vous établissez une connexion SSH ou RDP à une adresse IP publique et à un numéro de port assignés. Par défaut, des règles de traduction d’adresses réseau (NAT) sont ajoutées à l’équilibreur de charge Azure qui transfère le trafic de connexion à distance à chaque machine virtuelle. Pour afficher l’adresse et les ports à connecter à des instances de machine virtuelle dans un groupe identique, utilisez la commande [az mise liste-instance-connexion-info](/cli/azure/vmss). L’exemple suivant répertorie les informations de connexion pour les instances de machine virtuelle du groupe identique nommé *myScaleSet* et dans le groupe de ressources *myResourceGroup*. Spécifiez vos propres valeurs pour ces noms :

```azurecli
az vmss list-instance-connection-info \
    --resource-group myResourceGroup \
    --name myScaleSet
```


## <a name="change-the-capacity-of-a-scale-set"></a>Modifier la capacité d’un groupe identique
Les commandes précédentes montraient plus d’informations sur votre groupe identique et les instances de machine virtuelle. Pour augmenter ou diminuer le nombre d’instances dans le groupe identique, vous pouvez modifier la capacité. Le groupe identique crée ou supprime le nombre requis de machines virtuelles, puis configure les machines virtuelles pour recevoir le trafic d’application.

Pour afficher le nombre d’instances présentes dans un groupe identique, utilisez [az vmss show](/cli/azure/vmss) et interrogez *sku.capacity* :

```azurecli
az vmss show \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --query [sku.capacity] \
    --output table
```

Vous pouvez ensuite augmenter ou diminuer manuellement le nombre de machines virtuelles dans le groupe identique avec [az vmss scale](/cli/azure/vmss). L’exemple suivant fixe le nombre de machines virtuelles présentes dans votre groupe identique à *5* :

```azurecli
az vmss scale \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --new-capacity 5
```

Quelques minutes sont nécessaires pour mettre à jour la capacité de votre groupe identique. Si vous réduisez la capacité d’un groupe identique, les machines virtuelles dont les ID d’instance sont les plus élevés sont supprimées en premier.


## <a name="stop-and-start-vms-in-a-scale-set"></a>Arrêter et démarrer des machines virtuelles dans un groupe identique
Pour arrêter une ou plusieurs machines virtuelles dans un groupe identique, utilisez la commande [az vmss stop](/cli/azure/vmss#az-vmss-stop). Le paramètre `--instance-ids` vous permet de spécifier une ou plusieurs machines virtuelles à arrêter. Si vous ne spécifiez pas d’ID d’instance, toutes les machines virtuelles dans le groupe identique sont arrêtées. Pour arrêter plusieurs machines virtuelles, séparez les ID d’instance à l’aide d’une espace.

L’exemple suivant arrête l’instance de machine virtuelle *0* dans le groupe identique nommé *myScaleSet* et le groupe de ressources *myResourceGroup*. Indiquez vos propres valeurs comme suit :

```azurecli
az vmss stop --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```

Les machines virtuelles arrêtées restent allouées et continuent d’occasionner des frais de calcul. Si vous préférez que les machines virtuelles soient désallouées et n’occasionnent que des frais de stockage, utilisez la commande [az vmss deallocate](/cli/azure/vmss). Pour désallouer plusieurs machines virtuelles, séparez les ID d’instance à l’aide d’une espace. L’exemple suivant arrête et désalloue l’instance de machine virtuelle *0* dans le groupe identique nommé *myScaleSet* et le groupe de ressources *myResourceGroup*. Indiquez vos propres valeurs comme suit :

```azurecli
az vmss deallocate --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```


### <a name="start-vms-in-a-scale-set"></a>Démarrer des machines virtuelles dans un groupe identique
Pour démarrer une ou plusieurs machines virtuelles dans un groupe identique, utilisez la commande [az vmss start](/cli/azure/vmss). Le paramètre `--instance-ids` vous permet de spécifier une ou plusieurs machines virtuelles à démarrer. Si vous ne spécifiez pas d’ID d’instance, toutes les machines virtuelles dans le groupe identique sont démarrées. Pour démarrer plusieurs machines virtuelles, séparez les ID d’instance par une espace.

L’exemple suivant démarre l’instance de machine virtuelle *0* dans le groupe identique nommé *myScaleSet* et le groupe de ressources *myResourceGroup*. Indiquez vos propres valeurs comme suit :

```azurecli
az vmss start --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```


## <a name="restart-vms-in-a-scale-set"></a>Redémarrer des machines virtuelles dans un groupe identique
Pour redémarrer une ou plusieurs machines virtuelles dans un groupe identique, utilisez la commande [az vmss restart](/cli/azure/vmss). Le paramètre `--instance-ids` vous permet de spécifier une ou plusieurs machines virtuelles à redémarrer. Si vous ne spécifiez pas d’ID d’instance, toutes les machines virtuelles dans le groupe identique sont redémarrées. Pour redémarrer plusieurs machines virtuelles, séparez les ID d’instance par une espace.

L’exemple suivant redémarre l’instance de machine virtuelle *0* dans le groupe identique nommé *myScaleSet* et le groupe de ressources *myResourceGroup*. Indiquez vos propres valeurs comme suit :

```azurecli
az vmss restart --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```


## <a name="remove-vms-from-a-scale-set"></a>Supprimer des machines virtuelles d’un groupe identique
Pour supprimer une ou plusieurs machines virtuelles dans un groupe identique, utilisez la commande [az vmss delete-instances](/cli/azure/vmss). Le paramètre `--instance-ids` vous permet de spécifier une ou plusieurs machines virtuelles à supprimer. Si vous spécifiez * pour l’ID d’instance, toutes les machines virtuelles dans le groupe identique sont supprimées. Pour supprimer plusieurs machines virtuelles, séparez les ID d’instance par une espace.

L’exemple suivant supprime l’instance de machine virtuelle *0* dans le groupe identique nommé *myScaleSet* et le groupe de ressources *myResourceGroup*. Indiquez vos propres valeurs comme suit :

```azurecli
az vmss delete-instances --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```


## <a name="next-steps"></a>Étapes suivantes
D’autres tâches courantes pour les groupes identiques sont le [déploiement d’une application](virtual-machine-scale-sets-deploy-app.md) et la [mise à niveau d’instances de machine virtuelle](virtual-machine-scale-sets-upgrade-scale-set.md). Vous pouvez également utiliser Azure CLI pour [configurer des règles de mise à l’échelle automatique](virtual-machine-scale-sets-autoscale-overview.md).
