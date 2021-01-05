---
title: Utiliser Azure CLI pour attribuer un rôle Azure pour l’accès aux données
titleSuffix: Azure Storage
description: Découvrez comment utiliser Azure CLI pour attribuer des autorisations à un principal de sécurité Azure Active Directory avec le contrôle d’accès en fonction du rôle Azure (RBAC Azure). Le service Stockage Azure prend en charge les rôles personnalisés Azure et intégrés pour l’authentification via Azure AD.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/07/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9b818b8b46b6ac3af98ff5f25ef69335231744cc
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96779037"
---
# <a name="use-azure-cli-to-assign-an-azure-role-for-access-to-blob-and-queue-data"></a>Utiliser Azure CLI pour affecter un rôle Azure gérant l’accès à des données d’objet blob et de file d’attente

Azure Active Directory (Azure AD) autorise les droits d’accès aux ressources sécurisées par le biais du [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md). Stockage Azure définit un ensemble de rôles intégrés Azure qui englobent les ensembles communs d’autorisations permettant d’accéder aux données blob ou de file d’attente.

Lorsqu’un rôle Azure est attribué à un principal de sécurité Azure AD, Azure octroie l’accès à ces ressources pour ce principal de sécurité. L’accès peut être limité au niveau de l’abonnement, du groupe de ressources, du compte de stockage ou d’un conteneur ou d’une file d’attente individuelle. Un principal de sécurité Azure AD peut correspondre à un utilisateur, à un groupe, à un principal de service d’application ou à une [identité managée pour les ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md).

Cet article décrit comment utiliser Azure CLI pour répertorier les rôles Azure intégrés et les attribuer à des utilisateurs. Pour plus d’informations sur l’utilisation d’Azure CLI, consultez [Interface de ligne de commande Azure (CLI)](/cli/azure).

## <a name="azure-roles-for-blobs-and-queues"></a>Rôles Azure pour blobs et files d’attente

[!INCLUDE [storage-auth-rbac-roles-include](../../../includes/storage-auth-rbac-roles-include.md)]

## <a name="determine-resource-scope"></a>Déterminer l’étendue de la ressource

[!INCLUDE [storage-auth-resource-scope-include](../../../includes/storage-auth-resource-scope-include.md)]

## <a name="list-available-azure-roles"></a>Répertorier les rôles Azure disponibles

Pour répertorier les rôles Azure intégrés disponibles avec Azure CLI, utilisez la commande [az role definition list](/cli/azure/role/definition#az-role-definition-list) :

```azurecli-interactive
az role definition list --out table
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

Pour attribuer un rôle Azure à un principal de sécurité, utilisez la commande [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create). Le format de la commande peut varier selon l’étendue de l’affectation. Les exemples suivants montrent comment attribuer un rôle à un utilisateur dans des étendues diverses, mais vous pouvez utiliser la même commande pour attribuer un rôle à n’importe quel principal de sécurité.

> [!NOTE]
> Lorsque vous créez un compte de stockage Azure, aucune autorisation d’accès aux données ne vous est automatiquement attribuée via Azure AD. Vous devez vous attribuer explicitement un rôle Azure pour le Stockage Azure. Vous pouvez l’attribuer au niveau de votre abonnement, groupe de ressources, compte de stockage, conteneur ou file d’attente.

### <a name="container-scope"></a>Étendue du conteneur

Pour affecter un rôle limité à un conteneur, spécifiez une chaîne contenant l’étendue du conteneur pour le paramètre `--scope`. L’étendue d’un conteneur a la forme :

```
/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/blobServices/default/containers/<container>
```

L’exemple suivant affecte le rôle **Contributeur aux données Blob du stockage** à un utilisateur, limité au niveau du conteneur. Veillez à remplacer les valeurs de l’exemple et les valeurs d’espace réservé entre les crochets par vos propres valeurs :

```azurecli-interactive
az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/blobServices/default/containers/<container>"
```

### <a name="queue-scope"></a>Étendue de la file d’attente

Pour affecter un rôle limité à une file d’attente, spécifiez une chaîne contenant l’étendue de la file d’attente pour le paramètre `--scope`. L’étendue d’une file d’attente a la forme :

```
/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/queueServices/default/queues/<queue>
```

L’exemple suivant affecte le rôle **Contributeur aux données de file d'attente du stockage** à un utilisateur, limité au niveau de la file d'attente. Veillez à remplacer les valeurs de l’exemple et les valeurs d’espace réservé entre les crochets par vos propres valeurs :

```azurecli-interactive
az role assignment create \
    --role "Storage Queue Data Contributor" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/queueServices/default/queues/<queue>"
```

### <a name="storage-account-scope"></a>Étendue du compte de stockage

Pour affecter un rôle limité au compte de stockage, spécifiez l’étendue de la ressource de compte de stockage pour le paramètre `--scope`. L’étendue d’un compte de stockage a la forme :

```
/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>
```

L’exemple suivant montre comment affecter le rôle **Lecteur des données Blob du stockage** à un utilisateur au niveau du compte de stockage. N’oubliez pas de remplacer les valeurs de l’exemple par vos propres valeurs : \

```azurecli-interactive
az role assignment create \
    --role "Storage Blob Data Reader" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```

### <a name="resource-group-scope"></a>Étendue du groupe de ressources

Pour affecter un rôle limité au groupe de ressources, spécifiez le nom ou l’ID du groupe de ressources pour le paramètre `--resource-group`. L’exemple suivant affecte le rôle **Lecteur des données en file d’attente du stockage** à un utilisateur au niveau du groupe de ressources. Veillez à remplacer les valeurs de l’exemple et les valeurs d’espace réservé entre les crochets par vos propres valeurs :

```azurecli-interactive
az role assignment create \
    --role "Storage Queue Data Reader" \
    --assignee <email> \
    --resource-group <resource-group>
```

### <a name="subscription-scope"></a>Étendue d’abonnement

Pour affecter un rôle limité à l’abonnement, spécifiez l’étendue de l’abonnement pour le paramètre `--scope`. L’étendue d’un abonnement a la forme :

```
/subscriptions/<subscription>
```

L’exemple suivant montre comment affecter le rôle **Lecteur des données Blob du stockage** à un utilisateur au niveau du compte de stockage. N’oubliez pas de remplacer les valeurs de l’exemple par vos propres valeurs : 

```azurecli-interactive
az role assignment create \
    --role "Storage Blob Data Reader" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>"
```

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter ou supprimer des attributions de rôles Azure à l’aide du module Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)
- [Utiliser le module Azure PowerShell afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](storage-auth-aad-rbac-powershell.md)
- [Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](storage-auth-aad-rbac-portal.md)
