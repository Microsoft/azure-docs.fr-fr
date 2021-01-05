---
title: Utiliser PowerShell pour attribuer un rôle Azure pour l’accès aux données
titleSuffix: Azure Storage
description: Découvrez comment utiliser le module Azure PowerShell pour attribuer des autorisations à un principal de sécurité Azure Active Directory avec le contrôle d’accès Azure en fonction du rôle (Azure RBAC). Stockage Azure prend en charge les rôles personnalisés Azure et intégrés pour l’authentification par le biais d’Azure AD.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/07/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 852375cc7948fc7f6bd106380b3194f2dc84b8ca
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96778918"
---
# <a name="use-powershell-to-assign-an-azure-role-for-access-to-blob-and-queue-data"></a>Utiliser PowerShell pour attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente

Azure Active Directory (Azure AD) autorise les droits d’accès aux ressources sécurisées par le biais du [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md). Stockage Azure définit un ensemble de rôles intégrés Azure qui englobent les ensembles communs d’autorisations permettant d’accéder aux conteneurs ou aux files d’attente.

Lorsqu’un rôle Azure est attribué à un principal de sécurité Azure AD, Azure octroie l’accès à ces ressources pour ce principal de sécurité. L’accès peut être limité au niveau de l’abonnement, du groupe de ressources, du compte de stockage ou d’un conteneur ou d’une file d’attente individuelle. Un principal de sécurité Azure AD peut correspondre à un utilisateur, à un groupe, à un principal de service d’application ou à une [identité managée pour les ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md).

Cet article décrit comment utiliser Azure PowerShell pour lister les rôles intégrés Azure et les attribuer aux utilisateurs. Pour plus d’informations sur l’utilisation Azure PowerShell, consultez la [Vue d’ensemble d’Azure PowerShell](/powershell/azure/).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="azure-roles-for-blobs-and-queues"></a>Rôles Azure pour blobs et files d’attente

[!INCLUDE [storage-auth-rbac-roles-include](../../../includes/storage-auth-rbac-roles-include.md)]

## <a name="determine-resource-scope"></a>Déterminer l’étendue de la ressource

[!INCLUDE [storage-auth-resource-scope-include](../../../includes/storage-auth-resource-scope-include.md)]

## <a name="list-available-azure-roles"></a>Répertorier les rôles Azure disponibles

Pour lister les rôles intégrés Azure disponibles avec Azure PowerShell, utilisez la commande [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition) :

```powershell
Get-AzRoleDefinition | FT Name, Description
```

Les rôles de données Stockage Azure intégrés sont répertoriés, ainsi que d’autres rôles intégrés pour Azure :

```Example
Storage Blob Data Contributor             Allows for read, write and delete access to Azure Storage blob containers and data
Storage Blob Data Owner                   Allows for full access to Azure Storage blob containers and data, including assigning POSIX access control.
Storage Blob Data Reader                  Allows for read access to Azure Storage blob containers and data
Storage Queue Data Contributor            Allows for read, write, and delete access to Azure Storage queues and queue messages
Storage Queue Data Message Processor      Allows for peek, receive, and delete access to Azure Storage queue messages
Storage Queue Data Message Sender         Allows for sending of Azure Storage queue messages
Storage Queue Data Reader                 Allows for read access to Azure Storage queues and queue messages
```

## <a name="assign-an-azure-role-to-a-security-principal"></a>Attribuer un rôle Azure à un principal de sécurité

Pour attribuer un rôle Azure à un principal de sécurité, utilisez la commande [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment). Le format de la commande peut varier selon l’étendue de l’affectation. Pour exécuter la commande, vous devez avoir le rôle Propriétaire ou Contributeur affecté à l’étendue correspondante. Les exemples suivants montrent comment attribuer un rôle à un utilisateur dans des étendues diverses, mais vous pouvez utiliser la même commande pour attribuer un rôle à n’importe quel principal de sécurité.

> [!NOTE]
> Lorsque vous créez un compte de stockage Azure, aucune autorisation d’accès aux données ne vous est automatiquement attribuée via Azure AD. Vous devez vous attribuer explicitement un rôle Azure pour le Stockage Azure. Vous pouvez l’attribuer au niveau de votre abonnement, groupe de ressources, compte de stockage, conteneur ou file d’attente.

### <a name="container-scope"></a>Étendue du conteneur

Pour affecter un rôle limité à un conteneur, spécifiez une chaîne contenant l’étendue du conteneur pour le paramètre `--scope`. L’étendue d’un conteneur a la forme :

```
/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/blobServices/default/containers/<container-name>
```

L’exemple suivant affecte le rôle **Contributeur aux données Blob du stockage** à un utilisateur, limité à un conteneur nommé *sample-container*. Veillez à remplacer les valeurs de l’exemple et les valeurs d’espace réservé entre les crochets par vos propres valeurs : 

```powershell
New-AzRoleAssignment -SignInName <email> `
    -RoleDefinitionName "Storage Blob Data Contributor" `
    -Scope  "/subscriptions/<subscription>/resourceGroups/sample-resource-group/providers/Microsoft.Storage/storageAccounts/<storage-account>/blobServices/default/containers/sample-container"
```

### <a name="queue-scope"></a>Étendue de la file d’attente

Pour affecter un rôle limité à une file d’attente, spécifiez une chaîne contenant l’étendue de la file d’attente pour le paramètre `--scope`. L’étendue d’une file d’attente a la forme :

```
/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/queueServices/default/queues/<queue-name>
```

L’exemple suivant affecte le rôle **Contributeur aux données en file d’attente du stockage** à un utilisateur, limité à une file d’attente nommée *sample-queue*. Veillez à remplacer les valeurs de l’exemple et les valeurs d’espace réservé entre les crochets par vos propres valeurs : 

```powershell
New-AzRoleAssignment -SignInName <email> `
    -RoleDefinitionName "Storage Queue Data Contributor" `
    -Scope  "/subscriptions/<subscription>/resourceGroups/sample-resource-group/providers/Microsoft.Storage/storageAccounts/<storage-account>/queueServices/default/queues/sample-queue"
```

### <a name="storage-account-scope"></a>Étendue du compte de stockage

Pour affecter un rôle limité au compte de stockage, spécifiez l’étendue de la ressource de compte de stockage pour le paramètre `--scope`. L’étendue d’un compte de stockage a la forme :

```
/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>
```

L’exemple suivant montre comment étendre l’attribution du rôle **Lecteur des données Blob du stockage** à un utilisateur au niveau du compte de stockage. N’oubliez pas de remplacer les valeurs de l’exemple par vos propres valeurs : 

```powershell
New-AzRoleAssignment -SignInName <email> `
    -RoleDefinitionName "Storage Blob Data Reader" `
    -Scope  "/subscriptions/<subscription>/resourceGroups/sample-resource-group/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```

### <a name="resource-group-scope"></a>Étendue du groupe de ressources

Pour affecter un rôle limité au groupe de ressources, spécifiez le nom ou l’ID du groupe de ressources pour le paramètre `--resource-group`. L’exemple suivant affecte le rôle **Lecteur des données en file d’attente du stockage** à un utilisateur au niveau du groupe de ressources. Veillez à remplacer les valeurs de l’exemple et les valeurs d’espace réservé entre les crochets par vos propres valeurs : 

```powershell
New-AzRoleAssignment -SignInName <email> `
    -RoleDefinitionName "Storage Queue Data Reader" `
    -ResourceGroupName "sample-resource-group"
```

### <a name="subscription-scope"></a>Étendue d’abonnement

Pour affecter un rôle limité à l’abonnement, spécifiez l’étendue de l’abonnement pour le paramètre `--scope`. L’étendue d’un abonnement a la forme :

```
/subscriptions/<subscription>
```

L’exemple suivant montre comment affecter le rôle **Lecteur des données Blob du stockage** à un utilisateur au niveau du compte de stockage. N’oubliez pas de remplacer les valeurs de l’exemple par vos propres valeurs : 

```powershell
New-AzRoleAssignment -SignInName <email> `
    -RoleDefinitionName "Storage Blob Data Reader" `
    -Scope  "/subscriptions/<subscription>"
```

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter ou supprimer des attributions de rôles Azure à l’aide du module Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)
- [Utiliser Azure CLI afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](storage-auth-aad-rbac-cli.md)
- [Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](storage-auth-aad-rbac-portal.md)
