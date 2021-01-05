---
title: Élever l’accès pour gérer tous les abonnements et groupes d’administration Azure
description: Cet article décrit comment élever l’accès d’un administrateur général pour gérer tous les abonnements et groupes d’administration dans Azure Active Directory à l’aide du portail Azure ou d’une API REST.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 06/09/2020
ms.author: rolyon
ms.openlocfilehash: 6e57e495d34a265b5e0691106996206029656c5a
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92371118"
---
# <a name="elevate-access-to-manage-all-azure-subscriptions-and-management-groups"></a>Élever l’accès pour gérer tous les abonnements et groupes d’administration Azure

En tant qu’administrateur général dans Azure Active Directory (Azure AD), il est possible que vous n’ayez pas accès à tous les abonnements et groupes d’administration de votre annuaire. Cet article décrit les méthodes pour élever votre accès à tous les abonnements et groupes d’administration.

[!INCLUDE [gdpr-dsr-and-stp-note](../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="why-would-you-need-to-elevate-your-access"></a>Pourquoi devez-vous élever votre accès ?

Si vous êtes administrateur général, il peut vous arriver de vouloir effectuer les actions suivantes :

- Récupérer l’accès à un abonnement ou groupe d’administration Azure quand un utilisateur a perdu cet accès
- Accorder à un autre utilisateur ou à vous-même l’accès à un abonnement ou groupe d’administration Azure
- Voir tous les abonnements ou groupes d’administration Azure au sein d’une organisation
- Autoriser une application d’automation (telle qu’une application de facturation ou d’audit) à accéder à tous les abonnements ou groupes d’administration Azure

## <a name="how-does-elevated-access-work"></a>Comment fonctionne l’accès avec élévation de privilèges ?

Les ressources Azure AD et Azure sont sécurisées de façon indépendante les unes des autres. Ainsi, les attributions de rôles Azure AD n’accordent pas d’accès aux ressources Azure et inversement, les attributions de rôles Azure n’accordent pas d’accès à Azure AD. En revanche, si vous êtes [administrateur général](../active-directory/roles/permissions-reference.md#company-administrator-permissions) dans Azure AD, vous pouvez vous attribuer à vous-même un accès à tous les abonnements et groupes d’administration Azure de votre annuaire. Utilisez cette fonctionnalité si vous n’avez pas accès aux ressources de l’abonnement Azure, comme les machines virtuelles ou les comptes de stockage, et que vous voulez utiliser vos privilèges d’administrateur général pour accéder à ces ressources.

Quand vous élevez votre accès, le rôle [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator) vous est attribué dans Azure au niveau de l’étendue racine (`/`). Ceci vous permet de voir toutes les ressources et d’attribuer des accès dans n’importe quel abonnement ou groupe d’administration de l’annuaire. Les attributions de rôles Administrateur de l’accès utilisateur peuvent être supprimées à l’aide d’Azure PowerShell, d’Azure CLI ou de l’API REST.

Vous devez supprimer cet accès avec élévation de privilèges après avoir effectué les modifications nécessaires au niveau de l’étendue racine.

![Élever l’accès](./media/elevate-access-global-admin/elevate-access.png)

## <a name="azure-portal"></a>Portail Azure

### <a name="elevate-access-for-a-global-administrator"></a>Élever l’accès d’un administrateur général

Effectuez les étapes suivantes pour élever l’accès d’un administrateur général à l’aide du portail Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com) ou au [Centre d’administration Azure Active Directory](https://aad.portal.azure.com) en tant qu’administrateur général.

    Si vous utilisez Azure AD Privileged Identity Management, [activez votre d’attribution de rôle Administrateur général](../active-directory/privileged-identity-management/pim-how-to-activate-role.md).

1. Ouvrez **Azure Active Directory** .

1. Sous **Gérer** , sélectionnez **Propriétés** .

   ![Sélectionner des propriétés pour les propriétés Azure Active Directory - Capture d’écran](./media/elevate-access-global-admin/azure-active-directory-properties.png)

1. Sous **Gestion de l’accès pour les ressources Azure** , définissez la bascule sur **Oui** .

   ![Gestion des accès aux ressources Azure - capture d’écran](./media/elevate-access-global-admin/aad-properties-global-admin-setting.png)

   Quand vous définissez la bascule sur **Oui** , le rôle Administrateur de l’accès utilisateur vous est attribué dans Azure RBAC au niveau de l’étendue racine (/). Ceci vous accorde l’autorisation d’attribuer des rôles dans tous les abonnements et groupes d’administration Azure associés à cet annuaire Azure AD. Cette bascule est disponible seulement pour les utilisateurs auxquels le rôle Administrateur général a été attribué dans Azure AD.

   Quand vous définissez la bascule sur **Non** , le rôle Administrateur de l’accès utilisateur dans Azure RBAC est supprimé de votre compte d’utilisateur. Vous ne pouvez plus attribuer des rôles dans tous les abonnements et groupes d’administration Azure associés à cet annuaire Azure AD. Vous pouvez voir et gérer seulement les abonnements et groupes d’administration Azure auxquels l’accès vous a été accordé.

    > [!NOTE]
    > Si vous utilisez [Privileged Identity Management](../active-directory/privileged-identity-management/pim-configure.md), la désactivation de l’attribution de rôle n’a pas pour effet de définir l’option **Gestion de l’accès pour les ressources Azure** sur **Non** . Si vous souhaitez conserver l’accès avec privilèges minimum, nous vous recommandons de définir cette bascule sur **Non** avant de désactiver l’attribution de rôle.
    
1. Cliquez sur **Enregistrer** pour enregistrer votre paramètre.

   Ce paramètre n’est pas une propriété globale et s’applique uniquement à l’utilisateur connecté. Vous ne pouvez pas élever l’accès pour tous les membres du rôle Administrateur général.

1. Déconnectez-vous et reconnectez-vous pour actualiser votre accès.

    Vous devez maintenant avoir accès à tous les abonnements et à tous les groupes d’administration de votre annuaire. Lorsque vous affichez le volet de contrôle d’accès (IAM), vous pouvez remarquer que le rôle Administrateur de l’accès utilisateur vous a été attribué au niveau de l’étendue racine.

   ![Attributions de rôle d’abonnement au niveau de l’étendue racine : capture d’écran](./media/elevate-access-global-admin/iam-root.png)

1. Apportez les modifications nécessaires via un accès avec élévation de privilèges.

    Pour obtenir des informations sur la l’attribution de rôles, consultez [Ajouter ou supprimer des attributions de rôle Azure à l’aide du portail Azure](role-assignments-portal.md). Si vous utilisez Privileged Identity Management, consultez [Découvrir les ressources Azure à gérer](../active-directory/privileged-identity-management/pim-resource-roles-discover-resources.md) ou [Attribuer des rôles de ressources Azure](../active-directory/privileged-identity-management/pim-resource-roles-assign-roles.md).

1. Procédez de la manière décrite dans la section suivante pour supprimer votre accès avec élévation de privilèges.

### <a name="remove-elevated-access"></a>Supprimer l’accès élevé

Pour supprimer l’attribution de rôle Administrateur de l’accès utilisateur au niveau de l’étendue racine (`/`), effectuez les étapes suivantes.

1. Connectez-vous en tant qu’utilisateur avec celui utilisé pour élever l’accès.

1. Dans la liste de navigation, cliquez sur **Azure Active Directory** , puis sur **Propriétés** .

1. Définissez la bascule **Gestion de l’accès pour les ressources Azure** sur **Non** . Comme il s’agit d’un paramètre par utilisateur, vous devez être connecté sous le même utilisateur que celui utilisé pour élever l’accès.

    Si vous tentez de supprimer l’attribution de rôle Administrateur de l’accès utilisateur dans le volet de contrôle d’accès (IAM), le message suivant s’affiche. Pour supprimer l’attribution de rôle, vous devez rétablir la bascule sur **Non** ou utiliser Azure PowerShell, Azure CLI ou l’API REST.

    ![Supprimer des attributions de rôle avec une étendue racine](./media/elevate-access-global-admin/iam-root-remove.png)

1. Déconnectez-vous en tant qu’administrateur général.

    Si vous utilisez Privileged Identity Management, désactivez votre attribution de rôle Administrateur général.

    > [!NOTE]
    > Si vous utilisez [Privileged Identity Management](../active-directory/privileged-identity-management/pim-configure.md), la désactivation de l’attribution de rôle n’a pas pour effet de définir l’option **Gestion de l’accès pour les ressources Azure** sur **Non** . Si vous souhaitez conserver l’accès avec privilèges minimum, nous vous recommandons de définir cette bascule sur **Non** avant de désactiver l’attribution de rôle.

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

### <a name="list-role-assignment-at-root-scope-"></a>Lister une attribution de rôle au niveau de l’étendue racine (/)

Pour lister l’attribution de rôle Administrateur de l’accès utilisateur pour un utilisateur au niveau de l’étendue racine (`/`), utilisez la commande [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment).

```azurepowershell
Get-AzRoleAssignment | where {$_.RoleDefinitionName -eq "User Access Administrator" `
  -and $_.SignInName -eq "<username@example.com>" -and $_.Scope -eq "/"}
```

```Example
RoleAssignmentId   : /providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111
Scope              : /
DisplayName        : username
SignInName         : username@example.com
RoleDefinitionName : User Access Administrator
RoleDefinitionId   : 18d7d88d-d35e-4fb5-a5c3-7773c20a72d9
ObjectId           : 22222222-2222-2222-2222-222222222222
ObjectType         : User
CanDelegate        : False
```

### <a name="remove-elevated-access"></a>Supprimer l’accès élevé

Pour supprimer l’attribution de rôle Administrateur de l’accès utilisateur pour vous-même ou un autre utilisateur au niveau de l’étendue racine (`/`), effectuez les étapes suivantes.

1. Connectez-vous en tant qu’utilisateur pouvant supprimer l’accès élevé. Il peut s’agir du même utilisateur que celui utilisé pour élever l’accès ou d’un autre administrateur général disposant d’un accès élevé au niveau de l’étendue racine.

1. Utilisez la commande [Remove-AzRoleAssignment](/powershell/module/az.resources/remove-azroleassignment) pour supprimer l’attribution de rôle Administrateur de l’accès utilisateur.

    ```azurepowershell
    Remove-AzRoleAssignment -SignInName <username@example.com> `
      -RoleDefinitionName "User Access Administrator" -Scope "/"
    ```

## <a name="azure-cli"></a>Azure CLI

### <a name="elevate-access-for-a-global-administrator"></a>Élever l’accès d’un administrateur général

Pour élever l’accès d’un administrateur général à l’aide de l’Azure CLI, effectuez les étapes de base suivantes.

1. Utilisez la commande [az rest](/cli/azure/reference-index?view=azure-cli-latest#az-rest) pour appeler le point de terminaison `elevateAccess`, qui vous accorde le rôle Administrateur de l’accès utilisateur au niveau de l’étendue racine (`/`).

    ```azurecli
    az rest --method post --url "/providers/Microsoft.Authorization/elevateAccess?api-version=2016-07-01"
    ```

1. Apportez les modifications nécessaires via un accès avec élévation de privilèges.

    Pour obtenir des informations sur l’attribution de rôles, consultez [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure CLI](role-assignments-cli.md).

1. Procédez de la manière décrite dans une section ultérieure pour supprimer votre accès avec élévation de privilèges.

### <a name="list-role-assignment-at-root-scope-"></a>Lister une attribution de rôle au niveau de l’étendue racine (/)

Pour lister l’attribution de rôle Administrateur de l’accès utilisateur pour un utilisateur au niveau de l’étendue racine (`/`), utilisez la commande [az role assignment list](/cli/azure/role/assignment#az-role-assignment-list).

```azurecli
az role assignment list --role "User Access Administrator" --scope "/"
```

```Example
[
  {
    "canDelegate": null,
    "id": "/providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111",
    "name": "11111111-1111-1111-1111-111111111111",
    "principalId": "22222222-2222-2222-2222-222222222222",
    "principalName": "username@example.com",
    "principalType": "User",
    "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
    "roleDefinitionName": "User Access Administrator",
    "scope": "/",
    "type": "Microsoft.Authorization/roleAssignments"
  }
]

```

### <a name="remove-elevated-access"></a>Supprimer l’accès élevé

Pour supprimer l’attribution de rôle Administrateur de l’accès utilisateur pour vous-même ou un autre utilisateur au niveau de l’étendue racine (`/`), effectuez les étapes suivantes.

1. Connectez-vous en tant qu’utilisateur pouvant supprimer l’accès élevé. Il peut s’agir du même utilisateur que celui utilisé pour élever l’accès ou d’un autre administrateur général disposant d’un accès élevé au niveau de l’étendue racine.

1. Utilisez la commande [az role assignment delete](/cli/azure/role/assignment#az-role-assignment-delete) pour supprimer l’attribution de rôle Administrateur de l’accès utilisateur.

    ```azurecli
    az role assignment delete --assignee username@example.com --role "User Access Administrator" --scope "/"
    ```

## <a name="rest-api"></a>API REST

### <a name="elevate-access-for-a-global-administrator"></a>Élever l’accès d’un administrateur général

Pour élever l’accès d’un administrateur général à l’aide de l’API REST, suivez les étapes de base suivantes.

1. Avec REST, appelez `elevateAccess`, qui vous accorde le rôle Administrateur de l’accès utilisateur au niveau de l’étendue racine (`/`).

   ```http
   POST https://management.azure.com/providers/Microsoft.Authorization/elevateAccess?api-version=2016-07-01
   ```

1. Apportez les modifications nécessaires via un accès avec élévation de privilèges.

    Pour obtenir des informations sur la l’attribution de rôles, consultez [Ajouter ou supprimer des attributions de rôle Azure à l’aide de l’API REST](role-assignments-rest.md).

1. Procédez de la manière décrite dans une section ultérieure pour supprimer votre accès avec élévation de privilèges.

### <a name="list-role-assignments-at-root-scope-"></a>Lister les attributions de rôle au niveau de l’étendue racine (/)

Vous pouvez lister toutes les attributions de rôle d’un utilisateur au niveau de l’étendue racine (`/`).

- Appelez [GET roleAssignments](/rest/api/authorization/roleassignments/listforscope), où `{objectIdOfUser}` est l’ID objet de l’utilisateur dont vous souhaitez récupérer les attributions de rôles.

   ```http
   GET https://management.azure.com/providers/Microsoft.Authorization/roleAssignments?api-version=2015-07-01&$filter=principalId+eq+'{objectIdOfUser}'
   ```

### <a name="list-deny-assignments-at-root-scope-"></a>Lister les affectations de refus au niveau de l’étendue racine (/)

Vous pouvez lister toutes les affectations de refus d’un utilisateur au niveau de l’étendue racine (`/`).

- Appelez GET denyAssignments où `{objectIdOfUser}` est l’ID objet de l’utilisateur dont vous souhaitez récupérer les affectations de refus.

   ```http
   GET https://management.azure.com/providers/Microsoft.Authorization/denyAssignments?api-version=2018-07-01-preview&$filter=gdprExportPrincipalId+eq+'{objectIdOfUser}'
   ```

### <a name="remove-elevated-access"></a>Supprimer l’accès élevé

Lorsque vous appelez `elevateAccess`, vous créez une attribution de rôle pour vous-même. Donc, pour révoquer ces privilèges, vous devez supprimer l’attribution du rôle Administrateur de l’accès utilisateur pour vous-même au niveau de l’étendue racine (`/`).

1. Appelez [GET roleDefinitions](/rest/api/authorization/roledefinitions/get), où `roleName` est l’Administrateur de l’accès utilisateur, pour déterminer l’ID de nom du rôle Administrateur de l’accès utilisateur.

    ```http
    GET https://management.azure.com/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01&$filter=roleName+eq+'User Access Administrator'
    ```

    ```json
    {
      "value": [
        {
          "properties": {
      "roleName": "User Access Administrator",
      "type": "BuiltInRole",
      "description": "Lets you manage user access to Azure resources.",
      "assignableScopes": [
        "/"
      ],
      "permissions": [
        {
          "actions": [
            "*/read",
            "Microsoft.Authorization/*",
            "Microsoft.Support/*"
          ],
          "notActions": []
        }
      ],
      "createdOn": "0001-01-01T08:00:00.0000000Z",
      "updatedOn": "2016-05-31T23:14:04.6964687Z",
      "createdBy": null,
      "updatedBy": null
          },
          "id": "/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
          "type": "Microsoft.Authorization/roleDefinitions",
          "name": "18d7d88d-d35e-4fb5-a5c3-7773c20a72d9"
        }
      ],
      "nextLink": null
    }
    ```

    Enregistrez l’ID à partir du paramètre `name`, en l’occurrence `18d7d88d-d35e-4fb5-a5c3-7773c20a72d9`.

1. Vous devez également lister les attributions de rôles pour l’administrateur d’annuaire au niveau de l’annuaire. Listez toutes les attributions dans l’étendue de l’annuaire pour le `principalId` de l’administrateur d’annuaire qui a effectué l’appel d’élévation d’accès. Ceci liste toutes les attributions de l’annuaire pour l’objectid.

    ```http
    GET https://management.azure.com/providers/Microsoft.Authorization/roleAssignments?api-version=2015-07-01&$filter=principalId+eq+'{objectid}'
    ```
        
    >[!NOTE] 
    >Un administrateur d’annuaire ne doit normalement pas avoir beaucoup d’attributions. Si la requête précédente retourne un trop grand nombre d’attributions, vous pouvez aussi interroger toutes les attributions seulement au niveau de l’étendue de l’annuaire, puis filtrer les résultats : `GET https://management.azure.com/providers/Microsoft.Authorization/roleAssignments?api-version=2015-07-01&$filter=atScope()`
            
1. Les appels précédents retournent une liste des attributions de rôle. Recherchez l’attribution de rôle pour laquelle l’étendue est `"/"`, où `roleDefinitionId` se termine par l’ID du nom de rôle trouvé à l’étape 1 et où `principalId` correspond à l’objectid de l’administrateur d’annuaire. 
    
    Exemple d’attribution de rôle :
    
    ```json
    {
      "value": [
        {
          "properties": {
            "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
            "principalId": "{objectID}",
            "scope": "/",
            "createdOn": "2016-08-17T19:21:16.3422480Z",
            "updatedOn": "2016-08-17T19:21:16.3422480Z",
            "createdBy": "22222222-2222-2222-2222-222222222222",
            "updatedBy": "22222222-2222-2222-2222-222222222222"
          },
          "id": "/providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111",
          "type": "Microsoft.Authorization/roleAssignments",
          "name": "11111111-1111-1111-1111-111111111111"
        }
      ],
      "nextLink": null
    }
    ```
    
    Là encore, enregistrez l’ID à partir du paramètre `name`, en l’occurrence 11111111-1111-1111-1111-111111111111.

1. Enfin, utilisez l’ID d’attribution de rôle pour supprimer l’attribution ajoutée par `elevateAccess` :

    ```http
    DELETE https://management.azure.com/providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111?api-version=2015-07-01
    ```

## <a name="next-steps"></a>Étapes suivantes

- [Comprendre les différents rôles](rbac-and-directory-admin-roles.md)
- [Ajouter ou supprimer des attributions de rôle Azure à l’aide de l’API REST](role-assignments-rest.md)
