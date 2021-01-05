---
title: Comprendre l’étendue d’Azure RBAC
description: Découvrez l’étendue du contrôle d’accès en fonction du rôle Azure (Azure RBAC) et la manière de déterminer l’étendue d’une ressource.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 10/08/2020
ms.author: rolyon
ms.openlocfilehash: ad906e3665c6ffc354cf6292c2559d1184037594
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91856146"
---
# <a name="understand-scope-for-azure-rbac"></a>Comprendre l’étendue d’Azure RBAC

L’*étendue* représente l’ensemble des ressources auxquelles un accès s’applique. Quand vous assignez un rôle, il est important de comprendre l’étendue, afin de pouvoir accorder à un principal de sécurité uniquement l’accès dont il a vraiment besoin. En limitant l’étendue, vous limitez les ressources menacées si le principal de sécurité est compromis.

## <a name="scope-levels"></a>Niveaux d’étendue

Dans Azure, vous pouvez spécifier une étendue à quatre niveaux : [groupe d’administration](../governance/management-groups/overview.md), abonnement, [groupe de ressources](../azure-resource-manager/management/overview.md#resource-groups) et ressource. Les étendues sont structurées dans une relation parent-enfant. Chaque niveau de hiérarchie rend l’étendue plus spécifique. Vous pouvez attribuer des rôles à n’importe de ces niveaux d’étendue. Le niveau que vous sélectionnez détermine la portée d’application du rôle. Les niveaux inférieurs héritent des autorisations de rôle des niveaux supérieurs. 

![Étendue pour une attribution de rôle](./media/scope-overview/rbac-scope-no-label.png)

Les groupes d’administration sont un niveau d’étendue situé au-dessus des abonnements, mais ils prennent en charge des hiérarchies plus complexes. Le diagramme suivant montre un exemple de hiérarchie de groupes d’administration et d’abonnements que vous pouvez définir. Pour plus d’informations sur les groupes d’administration, consultez [Présentation des groupes d’administration Azure](../governance/management-groups/overview.md).

![Hiérarchie des groupes d’administration et des abonnements](./media/scope-overview/rbac-scope-management-groups.png)

## <a name="scope-format"></a>Format de l’étendue

Si vous ajoutez des attributions de rôles à l’aide de la ligne de commande, vous devez spécifier l’étendue. Pour les outils en ligne de commande, l’étendue est une chaîne potentiellement longue qui identifie la portée exacte de l’attribution de rôle. Dans le portail Azure, cette étendue est généralement listée en tant qu’*ID de la ressource*.

L’étendue se compose d’une série d’identificateurs séparés par une barre oblique (« / »). Vous pouvez considérer cette chaîne comme exprimant la hiérarchie suivante, où les expressions sans espaces réservés (`{}`) sont des identificateurs fixes :

```
/subscriptions
    /{subscriptionId}
        /resourcegroups
            /{resourceGroupName}
                /providers
                    /{providerName}
                        /{resourceType}
                            /{resourceSubType1}
                                /{resourceSubType2}
                                    /{resourceName}
```

- `{subscriptionId}` est l’ID de l’abonnement à utiliser (GUID).
- `{resourcesGroupName}` est le nom du groupe de ressources.
- `{providerName}` est le nom du [fournisseur de ressources](../azure-resource-manager/management/azure-services-resource-providers.md) qui gère la ressource, tandis que `{resourceType}` et `{resourceSubType*}` identifient d’autres niveaux au sein de ce fournisseur de ressources.
- `{resourceName}` est la dernière partie de la chaîne qui identifie une ressource spécifique.

Les groupes d’administration sont situés un niveau au-dessus des abonnements et présentent l’étendue la plus large (moins spécifique). Les attributions de rôles à ce niveau s’appliquent aux abonnements au sein du groupe d’administration. L’étendue d’un groupe d’administration a le format suivant :

```
/providers
    /Microsoft.Management
        /managementGroups
            /{managmentGroupName}
```

## <a name="scope-examples"></a>Exemples d’étendue

> [!div class="mx-tableFixed"]
> | Étendue | Exemple |
> | --- | --- |
> | Groupe d’administration | `/providers/Microsoft.Management/managementGroups/marketing-group` |
> | Abonnement | `/subscriptions/00000000-0000-0000-0000-000000000000` |
> | Resource group | `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg` |
> |  | `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales` |
> | Ressource | `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/azurestorage12345/blobServices/default/containers/blob-container-01` |
> |  | `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/MyVirtualNetworkResourceGroup/providers/Microsoft.Network/virtualNetworks/MyVirtualNetwork12345` |

## <a name="how-to-determine-the-scope-for-a-resource"></a>Comment déterminer l’étendue d’une ressource

Il est assez simple de déterminer l’étendue d’un groupe d’administration, d’un abonnement ou d’un groupe de ressources. Vous devez simplement connaître le nom et l’ID d’abonnement. Toutefois, la détermination de l’étendue d’une ressource nécessite un peu plus de travail. Voici deux façons de déterminer l’étendue d’une ressource.

- Dans le portail Azure, ouvrez la ressource, puis examinez les propriétés. La ressource doit lister l’**ID de la ressource** où vous pouvez déterminer l’étendue. Par exemple, Voici les ID de ressource pour un compte de stockage.

    ![ID de ressource pour un compte de stockage dans le portail Azure](./media/scope-overview/scope-resource-id.png)

- Une autre méthode consiste à utiliser le portail Azure pour affecter temporairement un rôle au niveau de l’étendue de la ressource, puis à utiliser [Azure PowerShell](role-assignments-list-powershell.md) ou [Azure CLI](role-assignments-list-cli.md) pour lister l’attribution de rôle. Dans la sortie, l’étendue est listée en tant que propriété.

    ```azurepowershell
    RoleAssignmentId   : /subscriptions/<subscriptionId>/resourceGroups/test-rg/providers/Microsoft.Storage/storageAccounts/azurestorage12345/blobServices/default/containers/blob-container-01/pro
                         viders/Microsoft.Authorization/roleAssignments/<roleAssignmentId>
    Scope              : /subscriptions/<subscriptionId>/resourceGroups/test-rg/providers/Microsoft.Storage/storageAccounts/azurestorage12345/blobServices/default/containers/blob-container-01
    DisplayName        : User
    SignInName         : user@contoso.com
    RoleDefinitionName : Storage Blob Data Reader
    RoleDefinitionId   : 2a2b9908-6ea1-4ae2-8e65-a410df84e7d1
    ObjectId           : <principalId>
    ObjectType         : User
    CanDelegate        : False
    Description        :
    ConditionVersion   :
    Condition          :
    ```

    ```azurecli
    {
        "canDelegate": null,
        "condition": null,
        "conditionVersion": null,
        "description": null,
        "id": "/subscriptions/{subscriptionId}/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/azurestorage12345/blobServices/default/containers/blob-container-01/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
        "name": "{roleAssignmentId}",
        "principalId": "{principalId}",
        "principalName": "user@contoso.com",
        "principalType": "User",
        "resourceGroup": "test-rg",
        "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
        "roleDefinitionName": "Storage Blob Data Reader",
        "scope": "/subscriptions/{subscriptionId}/resourceGroups/Example-Storage-rg/providers/Microsoft.Storage/storageAccounts/azurestorage12345/blobServices/default/containers/blob-container-01",
        "type": "Microsoft.Authorization/roleAssignments"
      }
    ```

## <a name="next-steps"></a>Étapes suivantes

- [Étapes pour ajouter une attribution de rôle](role-assignments-steps.md)
- [Fournisseurs de ressources pour les services Azure](../azure-resource-manager/management/azure-services-resource-providers.md)
- [Présentation des groupes d’administration Azure](../governance/management-groups/overview.md)
