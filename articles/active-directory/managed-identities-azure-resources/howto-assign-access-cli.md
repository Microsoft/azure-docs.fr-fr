---
title: Attribuer à une identité managée l’accès à une ressource à l’aide d’Azure CLI - Azure AD
description: Instructions détaillées pour affecter à une identité managée sur une ressource l’accès à une autre ressource à l’aide d’Azure CLI.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/06/2017
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurecli
ms.openlocfilehash: e1383fc6cca34cac141ce9f1316b4df0879900aa
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92891995"
---
# <a name="assign-a-managed-identity-access-to-a-resource-using-azure-cli"></a>Affecter à une identité managée l’accès à une ressource à l’aide d’Azure CLI

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Après avoir configuré une ressource Azure avec une identité managée, vous pouvez accorder à cette dernière un accès à une autre ressource, tout comme n’importe quel principal de sécurité. Cet exemple montre comment accorder à l’identité managée d’une machine virtuelle ou d’un groupe de machines virtuelles identiques Azure l’accès à un compte de stockage Azure, à l’aide d’Azure CLI.

Si vous n’avez pas encore de compte Azure, [inscrivez-vous à un essai gratuit](https://azure.microsoft.com/free/) avant de continuer.

## <a name="prerequisites"></a>Prérequis

- Si vous n’êtes pas familiarisé avec les identités managées pour les ressources Azure, consultez [Que sont les identités managées pour les ressources Azure ?](overview.md). Pour en savoir plus sur les types d’identités managées affectées par le système et par l’utilisateur, consultez [Types d’identités managées](overview.md#managed-identity-types).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="use-azure-rbac-to-assign-a-managed-identity-access-to-another-resource"></a>Utiliser Azure RBAC pour affecter à une identité managée l’accès à une autre ressource

Après avoir activé l’identité managée sur une ressource Azure, comme une [machine virtuelle Azure](qs-configure-cli-windows-vm.md) ou un [groupe de machines virtuelles identiques Azure](qs-configure-cli-windows-vmss.md) : 

1. Dans cet exemple, nous accordons à une machine virtuelle Azure l’accès à un compte de stockage. Tout d’abord, nous utilisons [az resource list](/cli/azure/resource/#az-resource-list) pour obtenir le principal du service pour la machine virtuelle nommée myVM :

   ```azurecli-interactive
   spID=$(az resource list -n myVM --query [*].identity.principalId --out tsv)
   ```
   Pour un groupe de machines virtuelles identiques, la commande est la même à l’exception que, ici, vous obtenez le principal du service pour le groupe de machines virtuelles identiques nommé « DevTestVMSS » :
   
   ```azurecli-interactive
   spID=$(az resource list -n DevTestVMSS --query [*].identity.principalId --out tsv)
   ```

1. Une fois l’ID de principal du service obtenu, utilisez [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) pour accorder au « Lecteur » de la machine virtuelle ou du groupe de machines virtuelles identiques l’accès à un compte de stockage appelé « myStorageAcct » :

   ```azurecli-interactive
   az role assignment create --assignee $spID --role 'Reader' --scope /subscriptions/<mySubscriptionID>/resourceGroups/<myResourceGroup>/providers/Microsoft.Storage/storageAccounts/myStorageAcct
   ```

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des identités managées pour les ressources Azure](overview.md)
- Pour activer l’identité managée sur une machine virtuelle Azure, consultez [Configurer des identités managées pour ressources Azure sur une machine virtuelle Azure en utilisant Azure CLI](qs-configure-cli-windows-vm.md).
- Pour activer l’identité managée sur un groupe de machines virtuelles identiques Azure, consultez [Configurer des identités managées pour ressources Azure sur un groupe de machines virtuelles identiques en utilisant Azure CLI](qs-configure-cli-windows-vmss.md).
