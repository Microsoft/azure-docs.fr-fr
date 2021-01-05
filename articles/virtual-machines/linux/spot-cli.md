---
title: Utiliser l’interface CLI pour déployer des machines virtuelles Azure Spot
description: Découvrez comment utiliser l’interface CLI pour déployer des machines virtuelles Azure Spot afin de réduire les coûts.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 06/26/2020
ms.author: cynthn
ms.reviewer: jagaveer
ms.openlocfilehash: e3a48fa9f84a047a08e73bbaa239563dca541541
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91978363"
---
# <a name="deploy-spot-vms-using-the-azure-cli"></a>Déployer des machines virtuelles Spot avec Azure CLI

L’utilisation de [machines virtuelles Azure Spot](../spot-vms.md) vous permet de disposer de notre capacité inutilisée en réalisant des économies significatives. Dès qu’Azure a besoin de récupérer toute la capacité, l’infrastructure Azure supprime les machines virtuelles Spot. Les machines virtuelles Spot sont donc appropriées pour les charges de travail capables de gérer les interruptions, comme les travaux de traitement par lots, les environnements de développement et de test, les charges de travail de calcul importantes, entre autres.

Les tarifs des machines virtuelles Spot sont variables, en fonction de la région et de la référence SKU. Pour plus d’informations, consultez les prix des machines virtuelles pour [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) et [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). 

Vous avez la possibilité de définir un prix maximal que vous êtes prêt à payer, par heure, pour la machine virtuelle. Le prix maximal d’une machine virtuelle Spot peut être défini en dollars américains (USD), en utilisant jusqu’à 5 décimales. Par exemple, la valeur `0.98765` correspond à un prix maximal de 0,98765 $ USD par heure. Si vous définissez `-1` comme prix maximal, la machine virtuelle n’est pas supprimée en fonction du prix. Le prix de la machine virtuelle sera le prix actuel de Spot ou le prix d’une machine virtuelle standard, la valeur la plus faible étant retenue, à condition que la capacité et le quota soient disponibles. Pour plus d’informations sur la définition du prix maximal, consultez [Machines virtuelles Spot - Tarifs](../spot-vms.md#pricing).

Le processus de création d’une machine virtuelle Spot à l’aide d’Azure CLI est le même que celui décrit dans l’[article de démarrage rapide](./quick-create-cli.md). Ajoutez simplement le paramètre « --priority Spot », définissez `--eviction-policy` avec Deallocate (par défaut) ou `Delete`, puis indiquez un prix maximal ou `-1`. 


## <a name="install-azure-cli"></a>Installation de l’interface de ligne de commande Azure

Pour créer des machines virtuelles Spot, vous devez exécuter Azure CLI version 2.0.74 ou ultérieure. Pour déterminer la version, exécutez la commande **az --version**. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli). 

Connectez-vous à Azure avec la commande [az login](/cli/azure/reference-index#az-login).

```azurecli
az login
```

## <a name="create-a-spot-vm"></a>Créer une machine virtuelle Spot

Cet exemple montre comment déployer une machine virtuelle Spot Linux qui ne sera pas supprimée en fonction du prix. La stratégie d’éviction est définie pour désallouer la machine virtuelle, afin qu’elle puisse être redémarrée ultérieurement. Si vous souhaitez supprimer la machine virtuelle et le disque sous-jacent quand la machine virtuelle est supprimée, définissez `--eviction-policy` avec `Delete`.

```azurecli
az group create -n mySpotGroup -l eastus
az vm create \
    --resource-group mySpotGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --priority Spot \
    --max-price -1 \
    --eviction-policy Deallocate
```



Une fois la machine virtuelle créée, vous pouvez effectuer une requête pour voir le prix de facturation maximal pour toutes les machines virtuelles du groupe de ressources.

```azurecli
az vm list \
   -g mySpotGroup \
   --query '[].{Name:name, MaxPrice:billingProfile.maxPrice}' \
   --output table
```

## <a name="simulate-an-eviction"></a>Simuler une éviction

Vous pouvez [simuler l’éviction](/rest/api/compute/virtualmachines/simulateeviction) d’une machine virtuelle spot, afin de tester l’efficacité de la réponse de votre application en vue d’une éviction soudaine. 

Remplacez les éléments suivants par vos informations : 

- `subscriptionId`
- `resourceGroupName`
- `vmName`


```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/simulateEviction?api-version=2020-06-01
```

**Étapes suivantes**

Vous pouvez également créer une machine virtuelle Spot à l’aide d’[Azure PowerShell](../windows/spot-powershell.md), du [portail](../spot-portal.md) ou d’un [modèle](spot-template.md).

Interroger les informations de tarification actuelles à l’aide de l’[API des prix de vente au détail d’Azure](/rest/api/cost-management/retail-prices/azure-retail-prices) pour plus d’informations sur la tarification Spot. Les `meterName` et `skuName` contiennent tous les deux `Spot`.

Si vous rencontrez une erreur, consultez [Codes d’erreur](../error-codes-spot.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).