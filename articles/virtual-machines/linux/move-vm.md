---
title: Déplacer une machine virtuelle à l’aide d’Azure CLI
description: Déplacez une machine virtuelle vers un autre abonnement ou groupe de ressources Azure à l’aide d’Azure CLI.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 09/12/2018
ms.author: cynthn
ms.openlocfilehash: db4c7e0126616e2d8bd120e7430c70b89c5cf36d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87291107"
---
# <a name="move-a-vm-to-another-subscription-or-resource-group"></a>Déplacer une machine virtuelle vers un autre abonnement ou groupe de ressources
Cet article vous explique comment déplacer une machine virtuelle entre des groupes de ressources ou des abonnements. Le déplacement d’une machine virtuelle entre abonnements peut être pratique si vous avez créé une machine virtuelle dans un abonnement personnel, et que vous souhaitez à présent la déplacer vers l’abonnement de votre entreprise.

> [!IMPORTANT]
>De nouveaux ID de ressource sont créés dans le cadre du déplacement. Une fois que la machine virtuelle a été déplacée, vous devez mettre à jour vos outils et vos scripts pour utiliser les nouveaux ID de ressource.
>


## <a name="use-the-azure-cli-to-move-a-vm"></a>Utiliser l’interface CLI Azure pour déplacer une machine virtuelle


Pour pouvoir déplacer votre machine virtuelle à l’aide de l’interface Azure CLI, vous devez vous assurer que les abonnements source et de destination se trouvent dans le même locataire. Pour vérifier que les deux abonnements ont le même ID de locataire, utilisez la commande [az account show](/cli/azure/account).

```azurecli-interactive
az account show --subscription mySourceSubscription --query tenantId
az account show --subscription myDestinationSubscription --query tenantId
```
Si les ID clients des abonnements source et de destination ne sont pas identiques, vous devez contacter le [support technique](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) pour déplacer les ressources vers un nouveau client.

Pour déplacer avec succès une machine virtuelle, vous devez déplacer la machine virtuelle et toutes les ressources qui l’accompagnent. Utilisez la commande [azure resource list](/cli/azure/resource) pour lister toutes les ressources d’un groupe de ressources et leurs ID. Cela permet de diriger la sortie de cette commande dans un fichier. Vous pouvez ainsi copier et coller les ID dans des commandes ultérieures.

```azurecli-interactive
az resource list --resource-group "mySourceResourceGroup" --query "[].{Id:id}" --output table
```

Pour déplacer une machine virtuelle et ses ressources dans un autre groupe de ressources, utilisez la commande [az resource move](/cli/azure/resource). L’exemple suivant montre comment déplacer une machine virtuelle et les ressources courantes nécessaires. Utilisez le paramètre **-ids** et transmettez la liste des ID des ressources à déplacer, séparée par des virgules (sans espaces).

```azurecli-interactive
vm=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM
nic=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Network/networkInterfaces/myNIC
nsg=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNSG
pip=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIPAddress
vnet=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Network/virtualNetworks/myVNet
diag=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Storage/storageAccounts/mydiagnosticstorageaccount
storage=/subscriptions/mySourceSubscriptionID/resourceGroups/mySourceResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccountname    

az resource move \
    --ids $vm,$nic,$nsg,$pip,$vnet,$storage,$diag \
    --destination-group "myDestinationResourceGroup"
```

Si vous souhaitez déplacer la machine virtuelle et ses ressources vers un autre abonnement, ajoutez le paramètre **--destination-subscriptionId** pour spécifier l’abonnement de destination.

Lorsque vous devrez confirmer que vous souhaitez déplacer les ressources spécifiées, saisissez **Y** pour confirmer.

[!INCLUDE [virtual-machines-common-move-vm](../../../includes/virtual-machines-common-move-vm.md)]

## <a name="next-steps"></a>Étapes suivantes
Vous pouvez déplacer de nombreux types de ressources différents entre les groupes de ressources et les abonnements. Pour plus d’informations, consultez la page [Déplacer des ressources vers un nouveau groupe de ressources ou un abonnement](../../azure-resource-manager/management/move-resource-group-and-subscription.md).    
