---
title: Comment trouver votre ID de locataire - Azure Active Directory
description: Instructions pour la recherche d’ID de locataire Azure Active Directory pour un abonnement Azure existant.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 10/30/2020
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, devx-track-azurecli
ms.collection: M365-identity-device-management
ms.openlocfilehash: 910e68e5b8933875f252faf65fb58e514dee8254
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96326588"
---
# <a name="how-to-find-your-azure-active-directory-tenant-id"></a>Comment trouver votre ID de locataire Azure Active Directory

Les abonnements Azure comprennent une relation d’approbation avec Azure Active Directory (Azure AD). Azure AD est approuvé pour authentifier les utilisateurs, les services et les appareils pour l’abonnement. Chaque abonnement est associé à un ID de locataire et il existe plusieurs façons de trouver l’ID de locataire pour votre abonnement.

## <a name="find-tenant-id-through-the-azure-portal"></a>Rechercher l’ID de locataire via le portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com).
 
1. Sélectionnez **Azure Active Directory**.

1. Sélectionner **Propriétés**.

1. Ensuite, faites défiler jusqu’au champ **ID de locataire**. Votre ID de locataire se trouve dans la zone.

:::image type="content" source="media/active-directory-how-to-find-tenant/portal-tenant-id.png" alt-text="Azure Active Directory - Propriétés - ID de locataire - Champ ID de locataire":::

## <a name="find-tenant-id-with-powershell"></a>Rechercher l’ID de locataire avec PowerShell

Vous pouvez également rechercher le locataire par programme. Pour trouver l’ID de locataire avec Azure PowerShell, utilisez la cmdlet `Get-AzTenant`.

```azurepowershell-interactive
Connect-AzAccount
Get-AzTenant
```
   
Pour plus d’informations, consultez cette référence de la cmdlet Azure PowerShell pour [Get-AzTenant](/powershell/module/az.accounts/get-aztenant).


## <a name="find-tenant-id-with-cli"></a>Rechercher l’ID de locataire avec CLI
Si vous souhaitez utiliser une interface de ligne de commande pour rechercher l’ID de locataire, vous pouvez le faire avec [Azure CLI](/cli/azure/install-azure-cli) ou [Microsoft 365 CLI](https://pnp.github.io/cli-microsoft365/). 

Pour Azure CLI, utilisez l’une des commandes **az login**, **az account list** ou **az account tenant list** comme indiqué dans l’exemple suivant. Notez la propriété **tenantId** pour chacun de vos abonnements dans la sortie de chaque commande.

```azurecli-interactive
az login
az account list
az account tenant list
```

Pour plus d’informations, consultez la référence de la commande [az login](/cli/azure/reference-index#az_login), [az account](/cli/azure/ext/account/account) ou [az account tenant](/cli/azure/ext/account/account/tenant).


Pour Microsoft 365 CLI, utilisez la cmdlet **ID de locataire** comme indiqué dans l’exemple suivant :
 
```cli
m365 tenant id get
```

Pour plus d’informations, consultez la référence de commande Microsoft 365 [tenant id get](https://pnp.github.io/cli-microsoft365/cmd/tenant/id/id-get/) .


## <a name="next-steps"></a>Étapes suivantes

- Pour créer un locataire Azure AD, consultez [Démarrage rapide : Créer un locataire dans Azure Active Directory](active-directory-access-create-new-tenant.md).

- Pour apprendre à associer ou à ajouter un abonnement à un locataire, consultez [Associer ou ajouter un abonnement Azure à votre locataire Azure Active Directory](active-directory-how-subscriptions-associated-directory.md).

- Pour apprendre à trouver l’ID d’objet, consultez [Rechercher l’ID d’objet utilisateur](/partner-center/find-ids-and-domain-names#find-the-user-object-id).