---
title: Partager des images des galeries entre locataires
description: En utilisant Azure CLI, découvrez comment utiliser les galeries d’images partagées pour partager des images de machine virtuelle entre des locataires Azure.
author: axayjo
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/04/2019
ms.author: akjosh
ms.reviewer: cynthn
ms.custom: devx-track-azurecli
ms.openlocfilehash: f0192a6eb60c72c9f7ac605db492f4e5f97b76ed
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96779258"
---
# <a name="share-gallery-vm-images-across-azure-tenants-using-the-azure-cli"></a>Partager des images de machine virtuelle des galeries entre des locataires Azure à l’aide d’Azure CLI

Les galeries Shared Image Gallery vous permettent de partager des images à l’aide du contrôle d’accès en fonction du rôle (Azure RBAC). Vous pouvez utiliser Azure RBAC pour partager des images au sein de votre locataire, voire avec d’autres personnes en dehors. Pour plus d’informations sur cette option de partage simple, voir [Partager la galerie](./shared-images-portal.md#share-the-gallery).

[!INCLUDE [virtual-machines-share-images-across-tenants](../../../includes/virtual-machines-share-images-across-tenants.md)]

> [!IMPORTANT]
> Vous ne pouvez pas utiliser le portail pour déployer une machine virtuelle à partir d’une image dans un autre locataire Azure. Pour créer une machine virtuelle à partir d’une image partagée entre des locataires, vous devez utiliser l’interface Azure CLI ou [PowerShell](../windows/share-images-across-tenants.md).

## <a name="create-a-vm-using-azure-cli"></a>Créer une machine virtuelle à l’aide d’Azure CLI

Connectez le principal du service pour le locataire 1 à l’aide de l’ID d’application, de la clé d’application et de l’ID du locataire 1. Vous pouvez utiliser `az account show --query "tenantId"` pour obtenir les ID des locataires si nécessaire.

```azurecli-interactive
az account clear
az login --service-principal -u '<app ID>' -p '<Secret>' --tenant '<tenant 1 ID>'
az account get-access-token 
```
 
Connectez le principal du service pour le locataire 2 à l’aide de l’ID d’application, de la clé d’application et de l’ID du locataire 2 :

```azurecli-interactive
az login --service-principal -u '<app ID>' -p '<Secret>' --tenant '<tenant 2 ID>'
az account get-access-token
```

Créez la machine virtuelle. Remplacez les informations dans l’exemple par vos propres données.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image "/subscriptions/<Tenant 1 subscription>/resourceGroups/<Resource group>/providers/Microsoft.Compute/galleries/<Gallery>/images/<Image definition>/versions/<version>" \
  --admin-username azureuser \
  --generate-ssh-keys
```

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez des problèmes, consultez [Résoudre les problèmes des galeries d’images partagées](../troubleshooting-shared-images.md).
