---
title: "Gestion du Contrôle d’accès en fonction du rôle avec Azure PowerShell | Microsoft Docs"
description: "Comment gérer le contrôle d’accès en fonction du rôle (RBAC) avec Azure PowerShell, notamment le référencement des rôles, l’attribution de rôles et la suppression d’attributions de rôle."
services: active-directory
documentationcenter: 
author: kgremban
manager: femila
editor: 
ms.assetid: 9e225dba-9044-4b13-b573-2f30d77925a9
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/22/2016
ms.author: kgremban
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: 382d0cf15dafca51571346dab1e9d55eb2d09d67


---
# <a name="manage-role-based-access-control-with-azure-powershell"></a>Gestion du Contrôle d’accès en fonction du rôle (RBAC) avec Azure PowerShell
> [!div class="op_single_selector"]
> * [PowerShell](role-based-access-control-manage-access-powershell.md)
> * [Interface de ligne de commande Azure](role-based-access-control-manage-access-azure-cli.md)
> * [API REST](role-based-access-control-manage-access-rest.md)
> 
> 

Le contrôle d'accès en fonction du rôle (RBAC) dans l'API du portail Azure et l'API de gestion de ressources Azure permet une gestion très fine de l'accès à votre abonnement. Cette fonctionnalité vous permet d’accorder l’accès aux utilisateurs, groupes et principaux du service Active Directory en leur affectant certains rôles avec une étendue spécifique.

Avant d’utiliser PowerShell pour gérer le contrôle d’accès en fonction du rôle (RBAC), vous devez disposer des composants suivants :

* Azure PowerShell, version 0.8.8 ou ultérieure. Pour installer la dernière version et l’associer à votre abonnement Azure, consultez la page [Installation et configuration d’Azure PowerShell](../powershell-install-configure.md).
* Applets de commande Azure Resource Manager. Installez les [applets de commande Azure Resource Manager](https://msdn.microsoft.com/library/mt125356.aspx) dans PowerShell.

## <a name="list-roles"></a>Répertorier les rôles
### <a name="list-all-available-roles"></a>Répertorier tous les rôles disponibles
Pour répertorier les rôles RBAC pouvant être affectés et inspecter les opérations auxquelles ils accordent l’accès, utilisez `Get-AzureRmRoleDefinition`.

```
Get-AzureRmRoleDefinition | FT Name, Description
```

![RBAC PowerShell - Get-AzureRmRoleDefinition - capture d’écran](./media/role-based-access-control-manage-access-powershell/1-get-azure-rm-role-definition1.png)

### <a name="list-actions-of-a-role"></a>Répertorier les actions d'un rôle
Pour répertorier les actions pour un rôle spécifique, utilisez `Get-AzureRmRoleDefinition <role name>`.

```
Get-AzureRmRoleDefinition Contributor | FL Actions, NotActions

(Get-AzureRmRoleDefinition "Virtual Machine Contributor").Actions
```

![RBAC PowerShell - Get-AzureRmRoleDefinition pour un rôle spécifique - capture d’écran](./media/role-based-access-control-manage-access-powershell/1-get-azure-rm-role-definition2.png)

## <a name="see-who-has-access"></a>Voir quel utilisateur a des autorisations d’accès
Pour répertorier les attributions d’accès RBAC, utilisez `Get-AzureRmRoleAssignment`.

### <a name="list-role-assignments-at-a-specific-scope"></a>Répertorier les attributions de rôle dans une étendue spécifique
Vous pouvez voir toutes les attributions d’accès pour un abonnement, un groupe de ressources ou une ressource. Par exemple, pour voir toutes les attributions actives pour un groupe de ressources, utilisez `Get-AzureRmRoleAssignment -ResourceGroupName <resource group name>`.

```
Get-AzureRmRoleAssignment -ResourceGroupName Pharma-Sales-ProjectForcast | FL DisplayName, RoleDefinitionName, Scope
```

![RBAC PowerShell - Get-AzureRmRoleAssignment pour un groupe de ressource - capture d’écran](./media/role-based-access-control-manage-access-powershell/4-get-azure-rm-role-assignment1.png)

### <a name="list-roles-assigned-to-a-user"></a>Répertorier les rôles attribués à un utilisateur
Pour répertorier tous les rôles attribués à un utilisateur spécifié et aux rôles affectés aux groupes auxquels appartient l’utilisateur, utilisez `Get-AzureRmRoleAssignment -SignInName <User email> -ExpandPrincipalGroups`.

```
Get-AzureRmRoleAssignment -SignInName sameert@aaddemo.com | FL DisplayName, RoleDefinitionName, Scope

Get-AzureRmRoleAssignment -SignInName sameert@aaddemo.com -ExpandPrincipalGroups | FL DisplayName, RoleDefinitionName, Scope
```

![RBAC PowerShell - Get-AzureRmRoleAssignment pour un utilisateur - capture d’écran](./media/role-based-access-control-manage-access-powershell/4-get-azure-rm-role-assignment2.png)

### <a name="list-classic-service-administrator-and-coadmin-role-assignments"></a>Répertorier les affectations classiques des rôles Administrateur de service et Coadministrateur
Pour répertorier les affectations d'accès de liste pour les administrateurs d'abonnement et les coadministrateurs classiques, utilisez :

    Get-AzureRmRoleAssignment -IncludeClassicAdministrators

## <a name="grant-access"></a>Accorder l'accès
### <a name="search-for-object-ids"></a>Rechercher des ID d’objet
Pour attribuer un rôle, vous devez identifier l’objet (utilisateur, groupe ou application) et l’étendue.

Si vous ne connaissez pas l’ID d’abonnement, vous pouvez le trouver dans le panneau **Abonnements** du portail Azure. Pour découvrir comment rechercher l'ID d'abonnement, consultez [Get-AzureSubscription](https://msdn.microsoft.com/library/dn495302.aspx) sur MSDN.

Pour obtenir l’ID d’objet pour un groupe Azure AD, utilisez :

    Get-AzureRmADGroup -SearchString <group name in quotes>

Pour obtenir l’ID d’objet pour un principal du service Azure AD ou une application, utilisez :

    Get-AzureRmADServicePrincipal -SearchString <service name in quotes>

### <a name="assign-a-role-to-an-application-at-the-subscription-scope"></a>Affectation d'un rôle à une application pour l'abonnement
Pour accorder l'accès à une application pour l'abonnement, utilisez :

    New-AzureRmRoleAssignment -ObjectId <application id> -RoleDefinitionName <role name> -Scope <subscription id>

![RBAC PowerShell - New-AzureRmRoleAssignment - capture d’écran](./media/role-based-access-control-manage-access-powershell/2-new-azure-rm-role-assignment2.png)

### <a name="assign-a-role-to-a-user-at-the-resource-group-scope"></a>Affectation d'un rôle à un utilisateur pour le groupe de ressources
Pour accorder l'accès à un utilisateur pour le groupe de ressources, utilisez :

    New-AzureRmRoleAssignment -SignInName <email of user> -RoleDefinitionName <role name in quotes> -ResourceGroupName <resource group name>

![RBAC PowerShell - New-AzureRmRoleAssignment - capture d’écran](./media/role-based-access-control-manage-access-powershell/2-new-azure-rm-role-assignment3.png)

### <a name="assign-a-role-to-a-group-at-the-resource-scope"></a>Affectation d'un rôle à un utilisateur pour des ressources
Pour accorder l'accès à un groupe au niveau des ressources, utilisez :

    New-AzureRmRoleAssignment -ObjectId <object id> -RoleDefinitionName <role name in quotes> -ResourceName <resource name> -ResourceType <resource type> -ParentResource <parent resource> -ResourceGroupName <resource group name>

![RBAC PowerShell - New-AzureRmRoleAssignment - capture d’écran](./media/role-based-access-control-manage-access-powershell/2-new-azure-rm-role-assignment4.png)

## <a name="remove-access"></a>Suppression d'accès
Pour supprimer l'accès pour des utilisateurs, des groupes et des applications, utilisez :

    Remove-AzureRmRoleAssignment -ObjectId <object id> -RoleDefinitionName <role name> -Scope <scope such as subscription id>

![RBAC PowerShell - Remove-AzureRmRoleAssignment - capture d’écran](./media/role-based-access-control-manage-access-powershell/3-remove-azure-rm-role-assignment.png)

## <a name="create-a-custom-role"></a>Créer un rôle personnalisé
Pour créer un rôle personnalisé, utilisez la commande `New-AzureRmRoleDefinition` .

Lorsque vous créez un rôle personnalisé à l'aide de PowerShell, vous devez commencer par un des [rôles intégrés](role-based-access-built-in-roles.md). Modifiez les attributs et ajoutez les propriétés *Actions*, *notActions*, ou les *étendues* de votre choix, puis enregistrez les modifications sous un nouveau rôle.

L’exemple suivant commence par le rôle *Collaborateur de machine virtuelle* et l’utilise pour créer un rôle personnalisé appelé *Opérateur de machine virtuelle*. Le nouveau rôle accorde l’accès à toutes les opérations des fournisseurs de ressources *Microsoft.Compute*, *Microsoft.Storage* et *Microsoft.Network*, ainsi que l’accès pour démarrer, redémarrer et surveiller des machines virtuelles. Le rôle personnalisé peut être utilisé dans deux abonnements.

```
$role = Get-AzureRmRoleDefinition "Virtual Machine Contributor"
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
$role.Actions.Add("Microsoft.Resources/subscriptions/resourceGroups/read")
$role.Actions.Add("Microsoft.Insights/alertRules/*")
$role.Actions.Add("Microsoft.Support/*")
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add("/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e")
$role.AssignableScopes.Add("/subscriptions/e91d47c4-76f3-4271-a796-21b4ecfe3624")
New-AzureRmRoleDefinition -Role $role
```

![RBAC PowerShell - Get-AzureRmRoleDefinition - capture d’écran](./media/role-based-access-control-manage-access-powershell/2-new-azurermroledefinition.png)

## <a name="modify-a-custom-role"></a>Modifier un rôle personnalisé
Pour modifier un rôle personnalisé, utilisez d’abord la commande `Get-AzureRmRoleDefinition` pour récupérer la définition de rôle. Apportez ensuite les modifications souhaitées à la définition de rôle. Enfin, utilisez la commande `Set-AzureRmRoleDefinition` pour enregistrer la définition de rôle modifiée.

L’exemple suivant ajoute l’opération `Microsoft.Insights/diagnosticSettings/*` au rôle personnalisé *Opérateur de machine virtuelle* .

```
$role = Get-AzureRmRoleDefinition "Virtual Machine Operator"
$role.Actions.Add("Microsoft.Insights/diagnosticSettings/*")
Set-AzureRmRoleDefinition -Role $role
```

![RBAC PowerShell - Set-AzureRmRoleDefinition - capture d’écran](./media/role-based-access-control-manage-access-powershell/3-set-azurermroledefinition-1.png)

L’exemple suivant ajoute un abonnement Azure aux étendues attribuables du rôle personnalisé *Opérateur de machine virtuelle* .

```
Get-AzureRmSubscription - SubscriptionName Production3

$role = Get-AzureRmRoleDefinition "Virtual Machine Operator"
$role.AssignableScopes.Add("/subscriptions/34370e90-ac4a-4bf9-821f-85eeedead1a2")
Set-AzureRmRoleDefinition -Role $role
```

![RBAC PowerShell - Set-AzureRmRoleDefinition - capture d’écran](./media/role-based-access-control-manage-access-powershell/3-set-azurermroledefinition-2.png)

## <a name="delete-a-custom-role"></a>Supprimer un rôle personnalisé
Pour supprimer un rôle personnalisé, utilisez la commande `Remove-AzureRmRoleDefinition` .

L’exemple suivant supprime le rôle personnalisé *Opérateur de machine virtuelle* .

```
Get-AzureRmRoleDefinition "Virtual Machine Operator"

Get-AzureRmRoleDefinition "Virtual Machine Operator" | Remove-AzureRmRoleDefinition
```

![RBAC PowerShell - Remove-AzureRmRoleDefinition - capture d’écran](./media/role-based-access-control-manage-access-powershell/4-remove-azurermroledefinition.png)

## <a name="list-custom-roles"></a>Répertorier les rôles personnalisés
Pour répertorier les rôles pouvant être affectés dans une étendue, utilisez la commande `Get-AzureRmRoleDefinition` .

L’exemple suivant répertorie tous les rôles pouvant être affectés dans l’abonnement sélectionné.

```
Get-AzureRmRoleDefinition | FT Name, IsCustom
```

![RBAC PowerShell - Get-AzureRmRoleDefinition - capture d’écran](./media/role-based-access-control-manage-access-powershell/5-get-azurermroledefinition-1.png)

Dans l’exemple suivant, le rôle personnalisé *Opérateur de machine virtuelle* n’est pas disponible dans l’abonnement *Production4*, car cet abonnement ne figure pas dans l’élément **AssignableScopes** du rôle.

![RBAC PowerShell - Get-AzureRmRoleDefinition - capture d’écran](./media/role-based-access-control-manage-access-powershell/5-get-azurermroledefinition2.png)

## <a name="see-also"></a>Voir aussi
* [Utilisation d’Azure PowerShell avec Azure Resource Manager](../powershell-azure-resource-manager.md)
  [!INCLUDE [role-based-access-control-toc.md](../../includes/role-based-access-control-toc.md)]




<!--HONumber=Nov16_HO3-->


