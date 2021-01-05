---
title: Créer un hôte Bastion à l’aide d’Azure CLI | Azure Bastion
description: Dans cet article, découvrez comment créer et supprimer un hôte Bastion.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: how-to
ms.date: 10/14/2020
ms.author: cherylmc
ms.openlocfilehash: e4f0a3ea85fa479826fe9e666df2b18f8dba7ce3
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92079120"
---
# <a name="create-an-azure-bastion-host-using-azure-cli"></a>Créer un hôte Azure Bastion à l’aide d’Azure CLI

Dans cet article, découvrez comment créer un hôte Azure Bastion à l’aide d’Azure CLI. Une fois que vous avez provisionné le service Azure Bastion dans votre réseau virtuel, l’expérience fluide RDP/SSH est disponible pour toutes les machines virtuelles du même réseau virtuel. Le déploiement Azure Bastion est effectué par réseau virtuel et non par abonnement/compte ou machine virtuelle.

Vous pouvez éventuellement créer un hôte Azure Bastion à l’aide du [portail Azure](./tutorial-create-host-portal.md) ou [d’Azure PowerShell](bastion-create-host-powershell.md).

## <a name="prerequisites"></a>Prérequis

Assurez-vous de disposer d’un abonnement Azure. Si vous ne disposez pas déjà d’un abonnement Azure, vous pouvez activer vos [avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) ou créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial).

[!INCLUDE [Cloud Shell CLI](../../includes/vpn-gateway-cloud-shell-cli.md)]

## <a name="create-a-bastion-host"></a><a name="createhost"></a>Créer un hôte Bastion

Cette section vous permet de créer une ressource Azure Bastion à l’aide d’Azure CLI.

> [!NOTE]
> Comme montré dans les exemples, utilisez le paramètre `--location` avec `--resource-group` pour chaque commande afin de garantir que les ressources sont déployées ensemble.

1. Créez un réseau virtuel et un sous-réseau Azure Bastion associé. Vous devez créer un sous-réseau Azure Bastion à l’aide de la valeur de nom **AzureBastionSubnet**. Cette valeur permet à Azure de savoir dans quel sous-réseau déployer les ressources Bastion. Ceci est différent d’un sous-réseau de passerelle. Vous devez utiliser un sous-réseau d’au moins /27 ou plus grand (/27, /26, etc.). Créez le sous-réseau **AzureBastionSubnet** sans tables de routage ou délégations. Si vous utilisez des groupes de sécurité réseau sur le sous-réseau **AzureBastionSubnet**, reportez-vous à l’article [Utiliser des groupes de sécurité réseau](bastion-nsg.md).

   ```azurecli-interactive
   az network vnet create --resource-group MyResourceGroup --name MyVnet --address-prefix 10.0.0.0/16 --subnet-name AzureBastionSubnet --subnet-prefix 10.0.0.0/24 --location northeurope
   ```

2. Créez une adresse IP publique pour Azure Bastion. Adresse IP publique de la ressource Bastion où RDP/SSH est accessible (sur le port 443). L’adresse IP publique doit être située dans la même région que la ressource Bastion que vous créez.

   ```azurecli-interactive
   az network public-ip create --resource-group MyResourceGroup --name MyIp --sku Standard --location northeurope
   ```

3. Créez une ressource Azure Bastion dans le sous-réseau Azure Bastion de votre réseau virtuel. Il faut environ 5 minutes pour que la ressource Bastion soit créée et déployée.

   ```azurecli-interactive
   az network bastion create --name MyBastion --public-ip-address MyIp --resource-group MyResourceGroup --vnet-name MyVnet --location northeurope
   ```

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations, lisez les [questions fréquentes (FAQ) sur Bastion](bastion-faq.md).
* Pour utiliser des groupes de sécurité réseau avec le sous-réseau Azure Bastion, consultez [Utiliser des groupes de sécurité réseau](bastion-nsg.md).
