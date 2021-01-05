---
title: Créer ou mettre à jour des rôles personnalisés Azure avec Azure PowerShell - Azure RBAC
description: Découvrez comment lister, créer, mettre à jour et supprimer des rôles personnalisés avec Azure PowerShell et le contrôle d’accès en fonction du rôle Azure (Azure RBAC).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 9e225dba-9044-4b13-b573-2f30d77925a9
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 03/18/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 799475db567c88f067192d027589e9185ee1782b
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97369172"
---
# <a name="create-or-update-azure-custom-roles-using-azure-powershell"></a>Créer ou mettre à jour des rôles personnalisés Azure avec Azure PowerShell

> [!IMPORTANT]
> L’ajout d’un groupe d’administration à `AssignableScopes` est actuellement en préversion.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Si les [rôles intégrés Azure](built-in-roles.md) ne répondent pas aux besoins spécifiques de votre organisation, vous pouvez créer vos propres rôles personnalisés. Cet article explique comment lister, créer, mettre à jour ou supprimer des rôles personnalisés à l’aide d’Azure PowerShell.

Pour obtenir un tutoriel pas à pas sur la création d’un rôle personnalisé, consultez [Tutoriel : Créer un rôle personnalisé Azure à l’aide d’Azure PowerShell](tutorial-custom-role-powershell.md).

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Pour créer des rôles personnalisés, vous avez besoin des éléments suivants :

- autorisations nécessaires pour créer des rôles personnalisés, par exemple, [Propriétaire](built-in-roles.md#owner) ou [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator) ;
- [Azure Cloud Shell](../cloud-shell/overview.md) ou [Azure PowerShell](/powershell/azure/install-az-ps)

## <a name="list-custom-roles"></a>Répertorier les rôles personnalisés

Pour répertorier les rôles qui peuvent être attribués à une étendue, utilisez la commande [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition). L’exemple suivant répertorie tous les rôles pouvant être affectés dans l’abonnement sélectionné.

```azurepowershell
Get-AzRoleDefinition | FT Name, IsCustom
```

```Example
Name                                              IsCustom
----                                              --------
Virtual Machine Operator                              True
AcrImageSigner                                       False
AcrQuarantineReader                                  False
AcrQuarantineWriter                                  False
API Management Service Contributor                   False
...
```

L’exemple suivant répertorie les rôles personnalisés pouvant être attribués dans l’abonnement sélectionné.

```azurepowershell
Get-AzRoleDefinition -Custom | FT Name, IsCustom
```

```Example
Name                     IsCustom
----                     --------
Virtual Machine Operator     True
```

Si l’abonnement sélectionné ne se trouve pas dans le `AssignableScopes` du rôle, le rôle personnalisé ne sera pas répertorié.

## <a name="list-a-custom-role-definition"></a>Lister une définition de rôle personnalisé

Pour lister une définition de rôle personnalisé, utilisez [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition). Il s’agit de la même commande que vous utiliseriez pour un rôle intégré.

```azurepowershell
Get-AzRoleDefinition <role_name> | ConvertTo-Json
```

```Example
PS C:\> Get-AzRoleDefinition "Virtual Machine Operator" | ConvertTo-Json

{
  "Name": "Virtual Machine Operator",
  "Id": "00000000-0000-0000-0000-000000000000",
  "IsCustom": true,
  "Description": "Can monitor and restart virtual machines.",
  "Actions": [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Authorization/*/read",
    "Microsoft.ResourceHealth/availabilityStatuses/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Support/*"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/11111111-1111-1111-1111-111111111111"
  ]
}
```

L’exemple suivant liste seulement les actions du rôle :

```azurepowershell
(Get-AzRoleDefinition <role_name>).Actions
```

```Example
PS C:\> (Get-AzRoleDefinition "Virtual Machine Operator").Actions

"Microsoft.Storage/*/read",
"Microsoft.Network/*/read",
"Microsoft.Compute/*/read",
"Microsoft.Compute/virtualMachines/start/action",
"Microsoft.Compute/virtualMachines/restart/action",
"Microsoft.Authorization/*/read",
"Microsoft.ResourceHealth/availabilityStatuses/read",
"Microsoft.Resources/subscriptions/resourceGroups/read",
"Microsoft.Insights/alertRules/*",
"Microsoft.Insights/diagnosticSettings/*",
"Microsoft.Support/*"
```

## <a name="create-a-custom-role"></a>Créer un rôle personnalisé

Pour créer un rôle personnalisé, utilisez la commande [New-AzRoleDefinition](/powershell/module/az.resources/new-azroledefinition). Il existe deux méthodes pour structurer le rôle : avec un objet `PSRoleDefinition` ou un modèle JSON. 

### <a name="get-operations-for-a-resource-provider"></a>Obtenir les opérations d’un fournisseur de ressources

Si vous créez des rôles personnalisés, il est important d’obtenir toutes les opérations possibles auprès des fournisseurs de ressources.
Pour obtenir ces informations, vous pouvez afficher la liste des [opérations du fournisseur de ressources](resource-provider-operations.md) ou utiliser la commande [Get-AzProviderOperation](/powershell/module/az.resources/get-azprovideroperation).
Par exemple, si vous souhaitez vérifier toutes les opérations disponibles pour des machines virtuelles, utilisez cette commande :

```azurepowershell
Get-AzProviderOperation <operation> | FT OperationName, Operation, Description -AutoSize
```

```Example
PS C:\> Get-AzProviderOperation "Microsoft.Compute/virtualMachines/*" | FT OperationName, Operation, Description -AutoSize

OperationName                                  Operation                                                      Description
-------------                                  ---------                                                      -----------
Get Virtual Machine                            Microsoft.Compute/virtualMachines/read                         Get the propertie...
Create or Update Virtual Machine               Microsoft.Compute/virtualMachines/write                        Creates a new vir...
Delete Virtual Machine                         Microsoft.Compute/virtualMachines/delete                       Deletes the virtu...
Start Virtual Machine                          Microsoft.Compute/virtualMachines/start/action                 Starts the virtua...
...
```

### <a name="create-a-custom-role-with-the-psroledefinition-object"></a>Créer un rôle personnalisé avec l’objet PSRoleDefinition

Lorsque vous utilisez PowerShell pour créer un rôle personnalisé, vous pouvez utiliser l’un des [rôles intégrés](built-in-roles.md) comme point de départ ou en créer un intégralement. Le premier exemple de cette section commence par un rôle intégré, qui est ensuite personnalisé avec des privilèges supplémentaires. Modifiez les attributs et ajoutez les `Actions`, `NotActions` ou `AssignableScopes` de votre choix, puis enregistrez les modifications sous un nouveau rôle.

L’exemple suivant commence par le rôle intégré [Contributeur de machines virtuelles](built-in-roles.md#virtual-machine-contributor) et l’utilise pour créer un rôle personnalisé appelé *Opérateur de machines virtuelles*. Le nouveau rôle accorde l’accès à toutes les opérations des fournisseurs de ressources *Microsoft.Compute*, *Microsoft.Storage* et *Microsoft.Network*, ainsi que l’accès pour démarrer, redémarrer et surveiller des machines virtuelles. Le rôle personnalisé peut être utilisé dans deux abonnements.

```azurepowershell
$role = Get-AzRoleDefinition "Virtual Machine Contributor"
$role.Id = $null
$role.Name = "Virtual Machine Operator"
$role.Description = "Can monitor and restart virtual machines."
$role.Actions.Clear()
$role.Actions.Add("Microsoft.Storage/*/read")
$role.Actions.Add("Microsoft.Network/*/read")
$role.Actions.Add("Microsoft.Compute/*/read")
$role.Actions.Add("Microsoft.Compute/virtualMachines/start/action")
$role.Actions.Add("Microsoft.Compute/virtualMachines/restart/action")
$role.Actions.Add("Microsoft.Authorization/*/read")
$role.Actions.Add("Microsoft.ResourceHealth/availabilityStatuses/read")
$role.Actions.Add("Microsoft.Resources/subscriptions/resourceGroups/read")
$role.Actions.Add("Microsoft.Insights/alertRules/*")
$role.Actions.Add("Microsoft.Support/*")
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add("/subscriptions/00000000-0000-0000-0000-000000000000")
$role.AssignableScopes.Add("/subscriptions/11111111-1111-1111-1111-111111111111")
New-AzRoleDefinition -Role $role
```

L’exemple suivant montre une autre méthode pour créer le rôle personnalisé *Opérateur de machines virtuelles*. Il illustre d’abord la création d’un objet `PSRoleDefinition`. Les opérations d’action sont spécifiées dans la variable `perms` et sont définies sur la propriété `Actions`. La propriété `NotActions` est définie en lisant les `NotActions` dans le rôle intégré [Contributeur de machines virtuelles](built-in-roles.md#virtual-machine-contributor). Étant donné que le [Contributeur de machines virtuelles](built-in-roles.md#virtual-machine-contributor) ne contient pas de `NotActions`, cette ligne n’est pas nécessaire. Toutefois, elle montre comment les informations peuvent être récupérées à partir d’un autre rôle.

```azurepowershell
$role = [Microsoft.Azure.Commands.Resources.Models.Authorization.PSRoleDefinition]::new()
$role.Name = 'Virtual Machine Operator 2'
$role.Description = 'Can monitor and restart virtual machines.'
$role.IsCustom = $true
$perms = 'Microsoft.Storage/*/read','Microsoft.Network/*/read','Microsoft.Compute/*/read'
$perms += 'Microsoft.Compute/virtualMachines/start/action','Microsoft.Compute/virtualMachines/restart/action'
$perms += 'Microsoft.Authorization/*/read'
$perms += 'Microsoft.ResourceHealth/availabilityStatuses/read'
$perms += 'Microsoft.Resources/subscriptions/resourceGroups/read'
$perms += 'Microsoft.Insights/alertRules/*','Microsoft.Support/*'
$role.Actions = $perms
$role.NotActions = (Get-AzRoleDefinition -Name 'Virtual Machine Contributor').NotActions
$subs = '/subscriptions/00000000-0000-0000-0000-000000000000','/subscriptions/11111111-1111-1111-1111-111111111111'
$role.AssignableScopes = $subs
New-AzRoleDefinition -Role $role
```

### <a name="create-a-custom-role-with-json-template"></a>Créer un rôle personnalisé avec le modèle JSON

Un modèle JSON peut servir de définition source pour le rôle personnalisé. L’exemple suivant crée un rôle personnalisé qui autorise l’accès en lecture au stockage et aux ressources de calcul, ainsi que l’accès au support, et ajoute ce rôle à deux abonnements. Créez un fichier `C:\CustomRoles\customrole1.json` avec l’exemple de contenu suivant. L’ID doit être défini sur `null` lors de la création du rôle, étant donné qu’un nouvel ID est automatiquement généré. 

```json
{
  "Name": "Custom Role 1",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows for read access to Azure storage and compute resources and access to support",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Storage/*/read",
    "Microsoft.Support/*"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000",
    "/subscriptions/11111111-1111-1111-1111-111111111111"
  ]
}
```

Pour ajouter le rôle aux abonnements, exécutez la commande PowerShell suivante :

```azurepowershell
New-AzRoleDefinition -InputFile "C:\CustomRoles\customrole1.json"
```

## <a name="update-a-custom-role"></a>Mettre à jour un rôle personnalisé

Comme pour la création d’un rôle personnalisé, vous pouvez modifier un rôle personnalisé existant à l’aide de l’objet `PSRoleDefinition` ou d’un modèle JSON.

### <a name="update-a-custom-role-with-the-psroledefinition-object"></a>Mettre à jour un rôle personnalisé avec l’objet PSRoleDefinition

Pour modifier un rôle personnalisé, commencez par récupérer sa définition à l'aide de la commande [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition). Apportez ensuite les modifications souhaitées à la définition de rôle. Enfin, utilisez la commande [Set-AzRoleDefinition](/powershell/module/az.resources/set-azroledefinition) pour enregistrer la définition de rôle modifiée.

L’exemple suivant ajoute l’opération `Microsoft.Insights/diagnosticSettings/*` au rôle personnalisé *Opérateur de machine virtuelle* .

```azurepowershell
$role = Get-AzRoleDefinition "Virtual Machine Operator"
$role.Actions.Add("Microsoft.Insights/diagnosticSettings/*")
Set-AzRoleDefinition -Role $role
```

```Example
PS C:\> $role = Get-AzRoleDefinition "Virtual Machine Operator"
PS C:\> $role.Actions.Add("Microsoft.Insights/diagnosticSettings/*")
PS C:\> Set-AzRoleDefinition -Role $role

Name             : Virtual Machine Operator
Id               : 88888888-8888-8888-8888-888888888888
IsCustom         : True
Description      : Can monitor and restart virtual machines.
Actions          : {Microsoft.Storage/*/read, Microsoft.Network/*/read, Microsoft.Compute/*/read,
                   Microsoft.Compute/virtualMachines/start/action...}
NotActions       : {}
AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000,
                   /subscriptions/11111111-1111-1111-1111-111111111111}
```

L’exemple suivant ajoute un abonnement Azure aux étendues attribuables du rôle personnalisé *Opérateur de machine virtuelle* .

```azurepowershell
Get-AzSubscription -SubscriptionName Production3

$role = Get-AzRoleDefinition "Virtual Machine Operator"
$role.AssignableScopes.Add("/subscriptions/22222222-2222-2222-2222-222222222222")
Set-AzRoleDefinition -Role $role
```

```Example
PS C:\> Get-AzSubscription -SubscriptionName Production3

Name     : Production3
Id       : 22222222-2222-2222-2222-222222222222
TenantId : 99999999-9999-9999-9999-999999999999
State    : Enabled

PS C:\> $role = Get-AzRoleDefinition "Virtual Machine Operator"
PS C:\> $role.AssignableScopes.Add("/subscriptions/22222222-2222-2222-2222-222222222222")
PS C:\> Set-AzRoleDefinition -Role $role

Name             : Virtual Machine Operator
Id               : 88888888-8888-8888-8888-888888888888
IsCustom         : True
Description      : Can monitor and restart virtual machines.
Actions          : {Microsoft.Storage/*/read, Microsoft.Network/*/read, Microsoft.Compute/*/read,
                   Microsoft.Compute/virtualMachines/start/action...}
NotActions       : {}
AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000,
                   /subscriptions/11111111-1111-1111-1111-111111111111,
                   /subscriptions/22222222-2222-2222-2222-222222222222}
```

L'exemple suivant ajoute un groupe d'administration à la propriété `AssignableScopes` du rôle personnalisé *Opérateur de machine virtuelle* . La fonctionnalité d'ajout d'un groupe d'administration à `AssignableScopes` est actuellement disponible en préversion.

```azurepowershell
Get-AzManagementGroup

$role = Get-AzRoleDefinition "Virtual Machine Operator"
$role.AssignableScopes.Add("/providers/Microsoft.Management/managementGroups/{groupId1}")
Set-AzRoleDefinition -Role $role
```

```Example
PS C:\> Get-AzManagementGroup

Id          : /providers/Microsoft.Management/managementGroups/marketing-group
Type        : /providers/Microsoft.Management/managementGroups
Name        : marketing-group
TenantId    : 99999999-9999-9999-9999-999999999999
DisplayName : Marketing group

PS C:\> $role = Get-AzRoleDefinition "Virtual Machine Operator"
PS C:\> $role.AssignableScopes.Add("/providers/Microsoft.Management/managementGroups/marketing-group")
PS C:\> Set-AzRoleDefinition -Role $role

Name             : Virtual Machine Operator
Id               : 88888888-8888-8888-8888-888888888888
IsCustom         : True
Description      : Can monitor and restart virtual machines.
Actions          : {Microsoft.Storage/*/read, Microsoft.Network/*/read, Microsoft.Compute/*/read,
                   Microsoft.Compute/virtualMachines/start/action...}
NotActions       : {}
AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000,
                   /subscriptions/11111111-1111-1111-1111-111111111111,
                   /subscriptions/22222222-2222-2222-2222-222222222222,
                   /providers/Microsoft.Management/managementGroups/marketing-group}
```

### <a name="update-a-custom-role-with-a-json-template"></a>Mettre à jour un rôle personnalisé avec un modèle JSON

À l’aide du modèle JSON précédent, vous pouvez facilement modifier un rôle personnalisé existant pour ajouter ou supprimer des actions. Mettez à jour le modèle JSON et ajoutez l’action de lecture pour la mise en réseau, comme l’indique l’exemple suivant. Les définitions figurant dans le modèle ne sont pas cumulativement appliquées à une définition existante, ce qui veut dire que le rôle s’affiche exactement comme vous le spécifiez dans le modèle. Vous devez aussi mettre à jour le champ ID avec l’ID du rôle. Si nécessaire, utilisez la cmdlet [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition) pour obtenir cette valeur.

```json
{
  "Name": "Custom Role 1",
  "Id": "acce7ded-2559-449d-bcd5-e9604e50bad1",
  "IsCustom": true,
  "Description": "Allows for read access to Azure storage and compute resources and access to support",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Support/*"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000",
    "/subscriptions/11111111-1111-1111-1111-111111111111"
  ]
}
```

Pour mettre à jour le rôle existant, exécutez la commande PowerShell suivante :

```azurepowershell
Set-AzRoleDefinition -InputFile "C:\CustomRoles\customrole1.json"
```

## <a name="delete-a-custom-role"></a>Supprimer un rôle personnalisé

Pour supprimer un rôle personnalisé, utilisez la commande [Remove-AzRoleDefinition](/powershell/module/az.resources/remove-azroledefinition).

L’exemple suivant supprime le rôle personnalisé *Opérateur de machine virtuelle* .

```azurepowershell
Get-AzRoleDefinition "Virtual Machine Operator"
Get-AzRoleDefinition "Virtual Machine Operator" | Remove-AzRoleDefinition
```

```Example
PS C:\> Get-AzRoleDefinition "Virtual Machine Operator"

Name             : Virtual Machine Operator
Id               : 88888888-8888-8888-8888-888888888888
IsCustom         : True
Description      : Can monitor and restart virtual machines.
Actions          : {Microsoft.Storage/*/read, Microsoft.Network/*/read, Microsoft.Compute/*/read,
                   Microsoft.Compute/virtualMachines/start/action...}
NotActions       : {}
AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000,
                   /subscriptions/11111111-1111-1111-1111-111111111111}

PS C:\> Get-AzRoleDefinition "Virtual Machine Operator" | Remove-AzRoleDefinition

Confirm
Are you sure you want to remove role definition with name 'Virtual Machine Operator'.
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y
```

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Créer un rôle personnalisé Azure à l’aide d’Azure PowerShell](tutorial-custom-role-powershell.md)
- [Rôle personnalisés Azure](custom-roles.md)
- [Opérations de fournisseur de ressources Azure](resource-provider-operations.md)
