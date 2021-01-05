---
title: Assigner des rôles personnalisés à l’aide d’Azure PowerShell - Azure AD | Microsoft Docs
description: Gérer les membres d’un rôle personnalisé d’administrateur Azure AD avec Azure PowerShell.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: how-to
ms.date: 11/04/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: d4695d0844ef8b707edce53a05de611c91223a46
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861950"
---
# <a name="assign-custom-roles-with-resource-scope-using-powershell-in-azure-active-directory"></a>Assigner des rôles personnalisés avec une étendue de ressources à l’aide de PowerShell dans Azure Active Directory

Cet article explique comment créer une attribution de rôle à l’échelle de l’organisation dans Azure Active Directory (Azure AD). L’attribution d’un rôle à l’échelle de l’organisation accorde l’accès à l’ensemble de l’organisation Azure AD. Pour créer une attribution de rôle avec une étendue d’une seule ressource Azure AD, consultez [Comment créer un rôle personnalisé et l’affecter à l’étendue de la ressource](custom-create.md). Cet article utilise le module [Azure Active Directory PowerShell Version 2](/powershell/module/azuread/#directory_roles).

Pour plus d'informations sur les rôles d'administrateur Azure AD, consultez [Attribuer des rôles d'administrateur dans Azure Active Directory](permissions-reference.md).

## <a name="required-permissions"></a>Autorisations requises

Connectez-vous à votre organisation Azure AD à l’aide d’un compte administrateur d’entreprise pour attribuer ou supprimer des rôles.

## <a name="prepare-powershell"></a>Préparer PowerShell

Installez le module Azure AD PowerShell à partir de la [PowerShell Gallery](https://www.powershellgallery.com/packages/AzureADPreview/2.0.0.17). Importez ensuite le module de préversion Azure AD PowerShell à l’aide de la commande suivante :

``` PowerShell
import-module azureadpreview
```

Pour vérifier que le module est prêt à l’emploi, faites correspondre la version retournée par la commande suivante à celle répertoriée ici :

``` PowerShell
get-module azureadpreview
  ModuleType Version      Name                         ExportedCommands
  ---------- ---------    ----                         ----------------
  Binary     2.0.0.115    azureadpreview               {Add-AzureADMSAdministrati...}
```

Vous pouvez désormais utiliser les applets de commande dans le module. Pour obtenir une description complète des applets de commande du module Azure AD, consultez la documentation de référence en ligne du [module de préversion Azure AD](https://www.powershellgallery.com/packages/AzureADPreview/2.0.0.17).

## <a name="assign-a-role-to-a-user-or-service-principal-with-resource-scope"></a>Affecter un rôle à un utilisateur ou à un principal de service avec une étendue de ressource

1. Ouvrez le module de préversion PowerShell Azure AD.
1. Connectez-vous en exécutant la commande `Connect-AzureAD`.
1. Créez un nouveau rôle à l’aide du script PowerShell suivant.

``` PowerShell
## Assign a role to a user or service principal with resource scope
# Get the user and role definition you want to link
$user = Get-AzureADUser -Filter "userPrincipalName eq 'cburl@f128.info'"
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Application Support Administrator'"

# Get app registration and construct resource scope for assignment.
$appRegistration = Get-AzureADApplication -Filter "displayName eq 'f/128 Filter Photos'"
$resourceScope = '/' + $appRegistration.objectId

# Create a scoped role assignment
$roleAssignment = New-AzureADMSRoleAssignment -ResourceScope $resourceScope -RoleDefinitionId $roleDefinition.Id -PrincipalId $user.objectId
```

Pour affecter le rôle à un principal du service au lieu d’un utilisateur, utilisez l’applet de commande [Get-AzureADMSServicePrincipal](/powershell/module/azuread/get-azureadserviceprincipal).

## <a name="operations-on-roledefinition"></a>Opérations sur RoleDefinition

Les objets de définition de rôle contiennent la définition du rôle intégré ou personnalisé, ainsi que les autorisations octroyées par cette attribution de rôle. Cette ressource affiche à la fois les définitions de rôle personnalisées et les directoryRoles intégrées (qui sont affichées sous forme équivalente de roleDefinition). Aujourd’hui, une organisation Azure AD peut avoir un maximum de 30 RoleDefinitions personnalisés uniques.

### <a name="create-operations-on-roledefinition"></a>Créer des opérations sur RoleDefinition

``` PowerShell
# Basic information
$description = "Can manage credentials of application registrations"
$displayName = "Application Registration Credential Administrator"
$templateId = (New-Guid).Guid

# Set of actions to grant
$allowedResourceAction =
@(
    "microsoft.directory/applications/standard/read",
    "microsoft.directory/applications/credentials/update"
)
$rolePermissions = @{'allowedResourceActions'= $allowedResourceAction}

# Create new custom admin role
$customAdmin = New-AzureADMSRoleDefinition -RolePermissions $rolePermissions -DisplayName $displayName -Description $description -TemplateId $templateId -IsEnabled $true
```

### <a name="read-operations-on-roledefinition"></a>Lire des opérations sur RoleDefinition

``` PowerShell
# Get all role definitions
Get-AzureADMSRoleDefinitions

# Get single role definition by objectId
Get-AzureADMSRoleDefinition -Id 86593cfc-114b-4a15-9954-97c3494ef49b

# Get single role definition by templateId
Get-AzureADMSRoleDefinition -Filter "templateId eq 'c4e39bd9-1100-46d3-8c65-fb160da0071f'"
```

### <a name="update-operations-on-roledefinition"></a>Mettre à jour des opérations sur RoleDefinition

``` PowerShell
# Update role definition
# This works for any writable property on role definition. You can replace display name with other
# valid properties.
Set-AzureADMSRoleDefinition -Id c4e39bd9-1100-46d3-8c65-fb160da0071f -DisplayName "Updated DisplayName"
```

### <a name="delete-operations-on-roledefinition"></a>Supprimer des opérations sur RoleDefinition

``` PowerShell
# Delete role definition
Remove-AzureADMSRoleDefinitions -Id c4e39bd9-1100-46d3-8c65-fb160da0071f
```

## <a name="operations-on-roleassignment"></a>Opérations sur RoleAssignment

Les attributions de rôles contiennent des informations qui lient une entité de sécurité donnée (un principal de service d’utilisateur ou d’application) à une définition de rôle. Si nécessaire, vous pouvez ajouter une étendue d’une ressource de Azure AD unique pour les autorisations affectées.  La restriction de l’étendue des autorisations est prise en charge pour les rôles intégrés et personnalisés.

### <a name="create-operations-on-roleassignment"></a>Créer des opérations sur RoleAssignment

``` PowerShell
# Get the user and role definition you want to link
$user = Get-AzureADUser -Filter "userPrincipalName eq 'cburl@f128.info'"
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Application Support Administrator'"

# Get app registration and construct resource scope for assignment.
$appRegistration = Get-AzureADApplication -Filter "displayName eq 'f/128 Filter Photos'"
$resourceScope = '/' + $appRegistration.objectId

# Create a scoped role assignment
$roleAssignment = New-AzureADMSRoleAssignment -ResourceScope $resourceScope -RoleDefinitionId $roleDefinition.Id -PrincipalId $user.objectId
```

### <a name="read-operations-on-roleassignment"></a>Lire des opérations sur RoleAssignment

``` PowerShell
# Get role assignments for a given principal
Get-AzureADMSRoleAssignment -Filter "principalId eq '27c8ca78-ab1c-40ae-bd1b-eaeebd6f68ac'"

# Get role assignments for a given role definition 
Get-AzureADMSRoleAssignment -Filter "roleDefinitionId eq '355aed8a-864b-4e2b-b225-ea95482e7570'"
```

### <a name="delete-operations-on-roleassignment"></a>Supprimer des opérations sur RoleAssignment

``` PowerShell
# Delete role assignment
Remove-AzureADMSRoleAssignment -Id 'qiho4WOb9UKKgng_LbPV7tvKaKRCD61PkJeKMh7Y458-1'
```

## <a name="next-steps"></a>Étapes suivantes

- Faites-nous part de vos commentaires sur le [Forum consacré aux rôles d'administration Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).
- Pour plus d'informations sur les rôles et les attributions de rôles d'administrateur Azure AD, consultez [Attribuer des rôles d'administrateur](permissions-reference.md).
- Pour plus d'informations sur les autorisations utilisateur par défaut, consultez la [comparaison des autorisations par défaut d'un utilisateur invité et d'un membre](../fundamentals/users-default-permissions.md).
