---
title: Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure CLI – Azure RBAC
description: Découvrez comment accorder l’accès à des ressources Azure à des utilisateurs, groupes, principaux de service ou identités managées en utilisant Azure CLI et le contrôle d’accès en fonction du rôle Azure (Azure RBAC).
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 09/28/2020
ms.author: rolyon
ms.custom: contperf-fy21q1, devx-track-azurecli
ms.openlocfilehash: 10060e0dc6595a4d59f3968fa324ca40c91a7a39
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033797"
---
# <a name="add-or-remove-azure-role-assignments-using-azure-cli"></a>Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure CLI

[!INCLUDE [Azure RBAC definition grant access](../../includes/role-based-access-control-definition-grant.md)] Cet article explique comment attribuer des rôles avec Azure CLI.

## <a name="prerequisites"></a>Prérequis

Pour ajouter ou supprimer des attributions de rôles, vous devez disposer :

- d’autorisations `Microsoft.Authorization/roleAssignments/write` et `Microsoft.Authorization/roleAssignments/delete`, telles que [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator) ou [Propriétaire de l’accès utilisateur](built-in-roles.md#owner)
- [Bash Azure Cloud Shell](../cloud-shell/overview.md) ou [Azure CLI](/cli/azure)

## <a name="steps-to-add-a-role-assignment"></a>Étapes pour ajouter une attribution de rôle

Dans Azure RBAC, vous ajoutez une attribution de rôle pour accorder l’accès. Une attribution de rôle se compose de trois éléments : un principal de sécurité, une définition de rôle et une étendue. Pour ajouter une attribution de rôle, procédez comme suit.

### <a name="step-1-determine-who-needs-access"></a>Étape 1 : Déterminer qui a besoin d’un accès

Vous pouvez attribuer un rôle à un utilisateur, à un groupe, à un principal de service ou à une identité managée. Pour ajouter une attribution de rôle, vous devrez peut-être spécifier l’ID unique de l’objet. L’ID a le format : `11111111-1111-1111-1111-111111111111`. Vous pouvez récupérer l’ID à l’aide du Portail Azure ou d’Azure CLI.

**Utilisateur**

Pour un utilisateur Azure AD, récupérez le nom d’utilisateur principal, par exemple *patlong\@contoso.com* ou l’ID d’objet utilisateur. Pour récupérer l’ID de l’objet, vous pouvez utiliser [az ad user show](/cli/azure/ad/user#az_ad_user_show).

```azurecli
az ad user show --id "{principalName}" --query "objectId" --output tsv
```

**Groupe**

Pour un groupe Azure AD, vous avez besoin de l’ID d’objet de groupe. Pour obtenir l’ID d’objet, vous pouvez utiliser [az ad group show](/cli/azure/ad/group#az_ad_group_show) ou [az ad group list](/cli/azure/ad/group#az_ad_group_list).

```azurecli
az ad group show --group "{groupName}" --query "objectId" --output tsv
```

**Principal du service**

Pour un principal de service Azure AD (identité utilisée par une application), vous avez besoin de l’ID d’objet du principal de service. Pour récupérer l’ID de l’objet, vous pouvez utiliser [az ad sp list](/cli/azure/ad/sp#az_ad_sp_list). Pour un principal de service, utilisez l’ID d’objet et **non** l’ID d’application.

```azurecli
az ad sp list --all --query "[].{displayName:displayName, objectId:objectId}" --output tsv
az ad sp list --display-name "{displayName}"
```

**Identité gérée**

Pour une identité managée affectée par le système ou par l’utilisateur, vous avez besoin de l’ID d’objet. Pour récupérer l’ID de l’objet, vous pouvez utiliser [az ad sp list](/cli/azure/ad/sp#az_ad_sp_list).

```azurecli
az ad sp list --all --filter "servicePrincipalType eq 'ManagedIdentity'"
```

Pour répertorier uniquement les identités managées affectées par l’utilisateur, vous pouvez utiliser [az identity list](/cli/azure/identity#az_identity_list).

```azurecli
az identity list
```
    
### <a name="step-2-find-the-appropriate-role"></a>Étape 2 : Rechercher le rôle approprié

Les autorisations sont regroupées dans des rôles. Vous pouvez choisir parmi une liste de plusieurs [rôles intégrés Azure](built-in-roles.md) ou utiliser vos propres rôles personnalisés. Accorder l’accès avec le moindre privilège requis étant recommandé, vous devez éviter d’attribuer un rôle plus large.

Pour répertorier les rôles et récupérer l’ID de rôle unique, vous pouvez utiliser [az role definition list](/cli/azure/role/definition#az_role_definition_list).

```azurecli
az role definition list --query "[].{name:name, roleType:roleType, roleName:roleName}" --output tsv
```

Voici comment répertorier les détails d’un rôle particulier.

```azurecli
az role definition list --name "{roleName}"
```

Pour plus d’informations, consultez [Répertorier les définitions de rôle Azure](role-definitions-list.md#azure-cli).
 
### <a name="step-3-identify-the-needed-scope"></a>Étape 3 : Identifier l’étendue nécessaire

Azure fournit quatre niveaux d’étendue : ressource, [groupe de ressources](../azure-resource-manager/management/overview.md#resource-groups), abonnement et [groupe d’administration](../governance/management-groups/overview.md). Accorder l’accès avec le moindre privilège requis étant recommandé, vous devez éviter d’attribuer un rôle à une étendue plus large. Pour plus d’informations sur l’étendue, consultez [Comprendre l’étendue](scope-overview.md).

**Étendue des ressources**

Pour l’étendue des ressources, vous avez besoin de l’ID de la ressource. Vous pouvez trouver l’ID de ressource en examinant les propriétés de la ressource dans le portail Azure. Un ID de ressource a le format suivant.

```
/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/{providerName}/{resourceType}/{resourceSubType}/{resourceName}
```

**Étendue du groupe de ressources**

Pour l’étendue du groupe de ressources, vous avez besoin du nom du groupe de ressources. Vous trouverez le nom dans la page **Groupes de ressources** du portail Azure, ou vous pouvez utiliser [az group list](/cli/azure/group#az_group_list).

```azurecli
az group list --query "[].{name:name}" --output tsv
```

**Étendue d’abonnement** 

Pour l’étendue d’abonnement, vous avez besoin de l’ID d’abonnement. Vous pouvez trouver l’ID dans la page **Abonnements** du portail Azure, ou vous pouvez utiliser [az account list](/cli/azure/account#az_account_list).

```azurecli
az account list --query "[].{name:name, id:id}" --output tsv
```

**Étendue d’un groupe d’administration** 

Pour l’étendue d’un groupe d’administration, vous avez besoin du nom du groupe d’administration. Vous pouvez trouver le nom dans la page **Groupes d’administration** dans le portail Azure, ou vous pouvez utiliser [az account management-group list](/cli/azure/account/management-group#az_account_management_group_list).

```azurecli
az account management-group list --query "[].{name:name, id:id}" --output tsv
```
    
### <a name="step-4-add-role-assignment"></a>Étape 4 : Ajouter une attribution de rôle

Pour ajouter une attribution de rôle, utilisez la commande [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create). En fonction de l’étendue, la commande a généralement l’un des formats suivants.

**Étendue des ressources**

```azurecli
az role assignment create --assignee "{assignee}" \
--role "{roleNameOrId}" \
--scope "/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/{providerName}/{resourceType}/{resourceSubType}/{resourceName}"
```

**Étendue du groupe de ressources**

```azurecli
az role assignment create --assignee "{assignee}" \
--role "{roleNameOrId}" \
--resource-group "{resourceGroupName}"
```

**Étendue d’abonnement** 

```azurecli
az role assignment create --assignee "{assignee}" \
--role "{roleNameOrId}" \
--subscription "{subscriptionNameOrId}"
```

**Étendue d’un groupe d’administration** 

```azurecli
az role assignment create --assignee "{assignee}" \
--role "{roleNameOrId}" \
--scope "/providers/Microsoft.Management/managementGroups/{managementGroupName}"
``` 

L’exemple suivant illustre la sortie lorsque vous attribuez le rôle [Contributeur de machine virtuelle](built-in-roles.md#virtual-machine-contributor) à un utilisateur dans une étendue de groupe de ressources.

```azurecli
{
  "canDelegate": null,
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
  "name": "{roleAssignmentId}",
  "principalId": "{principalId}",
  "principalType": "User",
  "resourceGroup": "{resourceGroupName}",
  "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/9980e02c-c2be-4d73-94e8-173b1dc7cf3c",
  "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}",
  "type": "Microsoft.Authorization/roleAssignments"
}
```
    
## <a name="add-role-assignment-examples"></a>Exemples d’ajout d’attribution de rôle

#### <a name="add-role-assignment-for-all-blob-containers-in-a-storage-account-resource-scope"></a>Ajouter une attribution de rôle pour tous les conteneurs de blobs dans l’étendue d’une ressource de compte de stockage

Attribue le rôle [Contributeur aux données Blob du stockage](built-in-roles.md#storage-blob-data-contributor) à un principal de service associé à l’ID d’objet *55555555-5555-5555-5555-555555555555* dans l’étendue des ressources d’un compte de stockage nommé *storage12345*.

```azurecli
az role assignment create --assignee "55555555-5555-5555-5555-555555555555" \
--role "Storage Blob Data Contributor" \
--scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345"
```

#### <a name="add-role-assignment-for-a-specific-blob-container-resource-scope"></a>Ajouter une attribution de rôle pour un conteneur de blobs spécifique dans l’étendue d’une ressource

Attribue le rôle [Contributeur aux données Blob du stockage](built-in-roles.md#storage-blob-data-contributor) à un principal de service associé à l’ID d’objet *55555555-5555-5555-5555-555555555555* dans l’étendue des ressources d’un conteneur de blobs nommé *blob-container-01*.

```azurecli
az role assignment create --assignee "55555555-5555-5555-5555-555555555555" \
--role "Storage Blob Data Contributor" \
--scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/storage12345/blobServices/default/containers/blob-container-01"
```

#### <a name="add-role-assignment-for-a-group-in-a-specific-virtual-network-resource-scope"></a>Ajouter une attribution de rôle pour un groupe dans l’étendue d’une ressource de réseau virtuel spécifique

Attribue le rôle [Contributeur de machine virtuelle](built-in-roles.md#virtual-machine-contributor) au groupe *Ann Mack Team* associé à l’ID 22222222-2222-2222-2222-222222222222 dans l’étendue des ressources d’un réseau virtuel nommé *pharma-sales-project-network*.

```azurecli
az role assignment create --assignee "22222222-2222-2222-2222-222222222222" \
--role "Virtual Machine Contributor" \
--scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/pharma-sales/providers/Microsoft.Network/virtualNetworks/pharma-sales-project-network"
```

#### <a name="add-role-assignment-for-a-user-at-a-resource-group-scope"></a>Ajouter une attribution de rôle pour un utilisateur dans l’étendue d’un groupe de ressources

Attribue le rôle [Contributeur de machine virtuelle](built-in-roles.md#virtual-machine-contributor) à l’utilisateur *patlong\@contoso.com* dans l’étendue du groupe de ressources *pharma-sales*.

```azurecli
az role assignment create --assignee "patlong@contoso.com" \
--role "Virtual Machine Contributor" \
--resource-group "pharma-sales"
```

#### <a name="add-role-assignment-for-a-user-using-the-unique-role-id-at-a-resource-group-scope"></a>Ajouter une attribution de rôle pour un utilisateur utilisant l’ID de rôle unique dans l’étendue d’un groupe de ressources

Un nom de rôle peut changer dans certaines circonstances, par exemple :

- Vous utilisez vos propres rôles personnalisés et vous décidez d’en modifier le nom.
- Vous utilisez un rôle en préversion dont le nom contient **(préversion)** . Lorsque le rôle est publié, le rôle est renommé.

Même si un rôle est renommé, l’ID de rôle ne change pas. Si vous utilisez des scripts ou une automatisation pour créer vos attributions de rôles, il est recommandé d’utiliser l’ID de rôle unique au lieu du nom de rôle. Par conséquent, si un rôle est renommé, vos scripts sont plus susceptibles de fonctionner.

L’exemple suivant attribue le rôle [Contributeur de machine virtuelle](built-in-roles.md#virtual-machine-contributor) à l’utilisateur *patlong\@contoso.com* dans l’étendue du groupe de ressources *pharma-sales*.

```azurecli
az role assignment create --assignee "patlong@contoso.com" \
--role "9980e02c-c2be-4d73-94e8-173b1dc7cf3c" \
--resource-group "pharma-sales"
```

#### <a name="add-role-assignment-for-all-blob-containers-at-a-resource-group-scope"></a>Ajouter une attribution de rôle pour tous les conteneurs de blobs dans l’étendue d’un groupe de ressources

Attribue le rôle [Contributeur aux données Blob du stockage](built-in-roles.md#storage-blob-data-contributor) à un principal de service associé à l’ID d’objet *55555555-5555-5555-5555-555555555555* dans l’étendue du groupe de ressources *Example-Storage-rg*.

```azurecli
az role assignment create --assignee "55555555-5555-5555-5555-555555555555" \
--role "Storage Blob Data Contributor" \
--resource-group "Example-Storage-rg"
```

Vous pouvez également spécifier le groupe de ressources complet avec le paramètre `--scope` :

```azurecli
az role assignment create --assignee "55555555-5555-5555-5555-555555555555" \
--role "Storage Blob Data Contributor" \
--scope "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg"
```

#### <a name="add-role-assignment-for-an-application-at-a-resource-group-scope"></a>Ajouter une attribution de rôle pour une application dans l’étendue d’un groupe de ressources

Attribue le rôle [Contributeur de machine virtuelle](built-in-roles.md#virtual-machine-contributor) à une application associée à l’ID d’objet de principal de service 44444444-4444-4444-4444-444444444444 dans l’étendue du groupe de ressources *pharma-sales*.

```azurecli
az role assignment create --assignee "44444444-4444-4444-4444-444444444444" \
--role "Virtual Machine Contributor" \
--resource-group "pharma-sales"
```

#### <a name="add-role-assignment-for-a-new-service-principal-at-a-resource-group-scope"></a>Ajouter une attribution de rôle pour un nouveau principal de service dans l’étendue d’un groupe de ressources

Dans certains cas, si vous créez un principal de service et que vous tentez immédiatement de lui attribuer un rôle, cette attribution peut échouer. Par exemple, si vous utilisez un script pour créer une identité managée et que vous tentez d’attribuer un rôle à ce principal de service, l’attribution de rôle peut échouer. Cet échec est souvent lié au délai de réplication. Le principal du service est créé dans une région. Toutefois, l’attribution de rôle peut s’effectuer dans une autre région, qui n’a pas encore répliqué le principal de service. Dans le cadre de ce scénario, vous devez spécifier le type de principal lors de la création de l’attribution de rôle.

Pour ajouter une attribution de rôle, utilisez [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create), spécifiez une valeur pour `--assignee-object-id`, puis définissez `--assignee-principal-type` sur `ServicePrincipal`.

```azurecli
az role assignment create --assignee-object-id "{assigneeObjectId}" \
--assignee-principal-type "{assigneePrincipalType}" \
--role "{roleNameOrId}" \
--resource-group "{resourceGroupName}" \
--scope "/subscriptions/{subscriptionId}"
```

Dans l’exemple suivant, le rôle [Contributeur de machines virtuelles](built-in-roles.md#virtual-machine-contributor) est attribué à l’identité managée *msi-test* dans l’étendue du groupe de ressources *pharma-sales* :

```azurecli
az role assignment create --assignee-object-id "33333333-3333-3333-3333-333333333333" \
--assignee-principal-type "ServicePrincipal" \
--role "Virtual Machine Contributor" \
--resource-group "pharma-sales"
```

#### <a name="add-role-assignment-for-a-user-at-a-subscription-scope"></a>Ajouter une attribution de rôle pour un utilisateur dans l’étendue d’un abonnement

Attribue le rôle [Lecteur](built-in-roles.md#reader) à l’utilisateur *annm\@example.com* dans l’étendue d’un abonnement.

```azurecli
az role assignment create --assignee "annm@example.com" \
--role "Reader" \
--subscription "00000000-0000-0000-0000-000000000000"
```

#### <a name="add-role-assignment-for-a-group-at-a-subscription-scope"></a>Ajouter une attribution de rôle pour un groupe dans l’étendue d’un abonnement

Attribue le rôle [Lecteur](built-in-roles.md#reader) au groupe *Ann Mack Team* associé à l’ID 22222222-2222-2222-2222-222222222222 dans l’étendue d’un abonnement.

```azurecli
az role assignment create --assignee "22222222-2222-2222-2222-222222222222" \
--role "Reader" \
--subscription "00000000-0000-0000-0000-000000000000"
```

#### <a name="add-role-assignment-for-all-blob-containers-at-a-subscription-scope"></a>Ajouter une attribution de rôle pour tous les conteneurs de blobs dans l’étendue d’un abonnement

Attribue le rôle [Lecteur des données BLOB du stockage](built-in-roles.md#storage-blob-data-reader) à l’utilisateur *alain\@example.com* dans l’étendue d’un abonnement.

```azurecli
az role assignment create --assignee "alain@example.com" \
--role "Storage Blob Data Reader" \
--scope "/subscriptions/00000000-0000-0000-0000-000000000000"
```

#### <a name="add-role-assignment-for-a-user-at-a-management-group-scope"></a>Ajouter une attribution de rôle pour un utilisateur dans l’étendue d’un groupe d’administration

Attribue le rôle [Lecteur de facturation](built-in-roles.md#billing-reader) à l’utilisateur *alain\@example.com* dans l’étendue d’un groupe d’administration.

```azurecli
az role assignment create --assignee "alain@example.com" \
--role "Billing Reader" \
--scope "/providers/Microsoft.Management/managementGroups/marketing-group"
```

## <a name="remove-a-role-assignment"></a>Supprimer une attribution de rôle

Dans Azure RBAC, pour supprimer un accès, vous supprimez une attribution de rôle à l’aide de la commande [az role assignment delete](/cli/azure/role/assignment#az_role_assignment_delete).

L’exemple suivant retire l’attribution de rôle [Collaborateur de machine virtuelle](built-in-roles.md#virtual-machine-contributor) à l’utilisateur *patlong\@contoso.com* dans le groupe de ressources *pharma-sales* :

```azurecli
az role assignment delete --assignee "patlong@contoso.com" \
--role "Virtual Machine Contributor" \
--resource-group "pharma-sales"
```

Retire le rôle [Lecteur](built-in-roles.md#reader) au groupe *Ann Mack Team* associé à l’ID 22222222-2222-2222-2222-222222222222 dans l’étendue d’un abonnement.

```azurecli
az role assignment delete --assignee "22222222-2222-2222-2222-222222222222" \
--role "Reader" \
--subscription "00000000-0000-0000-0000-000000000000"
```

Retire le rôle [Lecteur de facturation](built-in-roles.md#billing-reader) à l’utilisateur *alain\@example.com* dans l’étendue du groupe d’administration.

```azurecli
az role assignment delete --assignee "alain@example.com" \
--role "Billing Reader" \
--scope "/providers/Microsoft.Management/managementGroups/marketing-group"
```

## <a name="next-steps"></a>Étapes suivantes

- [Répertorier les attributions de rôle Azure à l’aide d’Azure CLI](role-assignments-list-cli.md)
- [Utiliser Azure CLI pour gérer les ressources et les groupes de ressources Azure](../azure-resource-manager/management/manage-resources-cli.md)
