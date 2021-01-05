---
title: Gérer des identités managées affectées par l’utilisateur Azure en utilisant REST - Azure AD
description: Instructions détaillées pour créer, répertorier et supprimer une identité managée affectée par l’utilisateur afin d’effectuer des appels d’API REST.
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
ms.date: 12/02/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 117f9a1c173f2083dd4621f4f3f41b6e83d1d46b
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96546690"
---
# <a name="create-list-or-delete-a-user-assigned-managed-identity-using-rest-api-calls"></a>Créer, répertorier ou supprimer une identité managée affectée par l’utilisateur en utilisant des appels d’API REST

Les identités managées pour les ressources Azure offrent aux services Azure la possibilité de s’authentifier auprès de services prenant en charge l’authentification Azure AD, sans avoir recours à des informations d’identification dans votre code. 

Dans cet article, découvrez comment créer, répertorier et supprimer une identité managée affectée par l’utilisateur en utilisant CURL pour effectuer des appels d’API REST.

## <a name="prerequisites"></a>Prérequis

- Si vous n’êtes pas familiarisé avec les identités managées pour ressources Azure, consultez la [section Vue d’ensemble](overview.md). **Veillez à consulter la [différence entre les identités managées affectées par le système et celles affectées par l’utilisateur](overview.md#managed-identity-types)** .
- Si vous n’avez pas encore de compte Azure, [inscrivez-vous à un essai gratuit](https://azure.microsoft.com/free/) avant de continuer.
- Vous pouvez exécuter toutes les commandes de cet article dans le cloud ou localement :
    - Pour exécuter dans le cloud, utilisez [Azure Cloud Shell](../../cloud-shell/overview.md).
    - Pour une exécution locale, installez [curl](https://curl.haxx.se/download.html) et [Azure CLI](/cli/azure/install-azure-cli).

## <a name="obtain-a-bearer-access-token"></a>Obtenir un jeton d’accès de porteur

1. Si vous travaillez localement, connectez-vous à Azure via Azure CLI :

    ```
    az login
    ```

1. Obtenir un jeton d’accès à l’aide de [az account get-access-token](/cli/azure/account#az_account_get_access_token)

    ```azurecli-interactive
    az account get-access-token
    ```

## <a name="create-a-user-assigned-managed-identity"></a>Créer une identité managée attribuée par l’utilisateur 

Pour créer une identité managée affectée par l’utilisateur, votre compte a besoin de l’affectation de rôle [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview' -X PUT -d '{"loc
ation": "<LOCATION>"}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview HTTP/1.1
```

**En-têtes de requête**

|En-tête de requête  |Description  |
|---------|---------|
|*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
|*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        |

**Corps de la demande**

|Nom  |Description  |
|---------|---------|
|location     | Obligatoire. Emplacement de la ressource.        |

## <a name="list-user-assigned-managed-identities"></a>Répertorier les identités managées affectées par l’utilisateur

Pour créer une identité managée affectée par l’utilisateur, votre compte a besoin de l’affectation de rôle [Opérateur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-operator) ou [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities?api-version=2015-08-31-preview' -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
GET https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities?api-version=2015-08-31-preview HTTP/1.1
```

|En-tête de requête  |Description  |
|---------|---------|
|*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
|*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        |

## <a name="delete-a-user-assigned-managed-identity"></a>Supprimer une identité managée affectée par l’utilisateur

Pour supprimer une identité managée affectée par l’utilisateur, votre compte a besoin de l’affectation de rôle [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-contributor).

> [!NOTE]
> La suppression d’une identité managée affectée par l’utilisateur n’a pas pour effet de supprimer la référence d’une ressource à laquelle elle a été affectée. Pour supprimer une identité managée affectée par l’utilisateur d’une machine virtuelle avec CURL, voir [Supprimer une identité affectée par l’utilisateur d’une machine virtuelle Azure](qs-configure-rest-vm.md#remove-a-user-assigned-managed-identity-from-an-azure-vm).

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview' -X DELETE -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
DELETE https://management.azure.com/subscriptions/80c696ff-5efa-4909-a64d-f1b616f423ca/resourceGroups/TestRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview HTTP/1.1
```
|En-tête de requête  |Description  |
|---------|---------|
|*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
|*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la manière d’attribuer une identité managée affectée à l’utilisateur à une machine virtuelle ou à un groupe de machines virtuelles identiques Azure en utilisant CURL, voir [Configurer des identités managées pour ressources Azure sur une machine virtuelle Azure en utilisant des appels d’API REST](qs-configure-rest-vm.md#user-assigned-managed-identity) et [Configurer des identités managées pour ressources Azure sur un groupe de machines virtuelles identiques en utilisant des appels d’API REST](qs-configure-rest-vmss.md#user-assigned-managed-identity).
