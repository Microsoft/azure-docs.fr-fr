---
title: Configurer un groupe de machines virtuelles identiques avec un service Azure Load Balancer existant – Azure CLI
description: Découvrez comment configurer un groupe de machines virtuelles identiques avec un service Azure Load Balancer existant à l’aide d’Azure CLI.
author: asudbring
ms.author: allensu
ms.service: load-balancer
ms.topic: how-to
ms.date: 03/25/2020
ms.openlocfilehash: a60a6889217ce6ca8dccd5ebf5ee74b8f67a7757
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94518207"
---
# <a name="configure-a-virtual-machine-scale-set-with-an-existing-azure-load-balancer-using-the-azure-cli"></a>Configurer un groupe de machines virtuelles identiques avec un service Azure Load Balancer existant à l’aide d’Azure CLI

Dans cet article, vous allez découvrir comment configurer un groupe de machines virtuelles identiques avec un service Azure Load Balancer existant.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis 

- Vous avez besoin d’un équilibreur de charge SKU standard existant dans l’abonnement dans lequel le groupe de machines virtuelles identiques sera déployé.

- Vous avez besoin d’un réseau virtuel Azure pour le groupe de machines virtuelles identiques.
 
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Cet article nécessite la version 2.0.28 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="deploy-a-virtual-machine-scale-set-with-existing-load-balancer"></a>Déployer un groupe de machines virtuelles identiques avec un équilibreur de charge existant

Remplacez les valeurs entre crochets par les noms des ressources de votre configuration.

```azurecli-interactive
az vmss create \
    --resource-group <resource-group> \
    --name <vmss-name>\
    --image <your-image> \
    --admin-username <admin-username> \
    --generate-ssh-keys  \
    --upgrade-policy-mode Automatic \
    --instance-count 3 \
    --vnet-name <virtual-network-name> \
    --subnet <subnet-name> \
    --lb <load-balancer-name> \
    --backend-pool-name <backend-pool-name>
```

L’exemple ci-dessous déploie un groupe de machines virtuelles identiques avec :

- Nom du groupe de machines virtuelles identiques nommé **myVMSS**
- Azure Load Balancer nommé **myLoadBalancer**
- Nom du pool de back-ends de l’équilibreur de charge nommé **myBackendPool**
- Réseau virtuel Azure nommé **myVnet**
- Sous-réseau nommé **mySubnet**
- Groupe de ressources nommé **myResourceGroup**
- Image de serveur Ubuntu pour le groupe de machines virtuelles identiques

```azurecli-interactive
az vmss create \
    --resource-group myResourceGroup \
    --name myVMSS \
    --image Canonical:UbuntuServer:18.04-LTS:latest \
    --admin-username adminuser \
    --generate-ssh-keys \
    --upgrade-policy-mode Automatic \
    --instance-count 3 \
    --vnet-name myVnet\
    --subnet mySubnet \
    --lb myLoadBalancer \
    --backend-pool-name myBackendPool
```
> [!NOTE]
> Une fois le groupe identique créé, le port principal ne peut pas être modifié lorsqu'une règle d'équilibrage de charge est utilisée par une sonde d'intégrité pour l'équilibreur de charge. Pour modifier le port, vous pouvez supprimer la sonde d'intégrité en mettant à jour le groupe identique de machines virtuelles Azure, puis mettre à jour le port et reconfigurer la sonde d'intégrité.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez déployé un groupe de machines virtuelles identiques avec un service Azure Load Balancer existant.  Pour en savoir plus sur les groupes de machines virtuelles identiques et l’équilibreur de charge, consultez :

- [Qu’est-ce que Azure Load Balancer ?](load-balancer-overview.md)
- [Que sont les groupes de machines virtuelles identiques ?](../virtual-machine-scale-sets/overview.md)
                                