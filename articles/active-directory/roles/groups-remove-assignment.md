---
title: Supprimer les attributions de rôle d’un groupe dans Azure Active Directory | Microsoft Docs
description: Affichez un aperçu des rôles Azure AD personnalisés pour la délégation de la gestion des identités. Gérez les rôles Azure dans le Portail Azure, PowerShell ou l’API Graph.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: article
ms.date: 11/05/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: d58465c56be310d3f9d500ab551c7d2b19ffaa83
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93378378"
---
# <a name="remove-role-assignments-from-a-group-in-azure-active-directory"></a>Supprimer les attributions de rôle d’un groupe dans Azure Active Directory

Cet article explique comment un administrateur informatique peut supprimer des rôles Azure AD attribués à des groupes. Dans le portail Azure, vous pouvez désormais supprimer l’attribution directe et indirecte de rôles à un utilisateur. Si un utilisateur se voit attribuer un rôle en raison de son appartenance à un groupe, supprimez l’utilisateur du groupe pour supprimer l’attribution de rôle.

## <a name="using-azure-admin-center"></a>Utilisation du centre d’administration Azure

1. Connectez-vous au [centre d’administration Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) avec des autorisations Administrateur de rôle privilégié ou Administrateur général dans l’organisation Azure AD.

1. Sélectionnez **Rôles et administrateurs** > * *_nom du rôle_* _.

1. Sélectionnez le groupe dont vous souhaitez supprimer l’attribution de rôle, puis sélectionnez _*Supprimer l’attribution**.

   ![Supprimer l’attribution de rôle d’un groupe sélectionné.](./media/groups-remove-assignment/remove-assignment.png)

1. Lorsque vous êtes invité à confirmer votre action, sélectionnez **Oui**.

## <a name="using-powershell"></a>Utilisation de PowerShell

### <a name="create-a-group-that-can-be-assigned-to-role"></a>Créer un groupe assignable à un rôle

```powershell
$group = New-AzureADMSGroup -DisplayName "Contoso_Helpdesk_Administrators" -Description "This group is assigned to Helpdesk Administrator built-in role in Azure AD." -MailEnabled $true -SecurityEnabled $true -MailNickName "contosohelpdeskadministrators" -IsAssignableToRole $true
```

### <a name="get-the-role-definition-you-want-to-assign-the-group-to"></a>Obtenir la définition de rôle que vous souhaitez attribuer au groupe

```powershell
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Helpdesk Administrator'"
```

### <a name="create-a-role-assignment"></a>Création d'une affectation de rôle

```powershell
$roleAssignment = New-AzureADMSRoleAssignment -ResourceScope '/' -RoleDefinitionId $roleDefinition.Id -PrincipalId $group.objectId
```

### <a name="remove-the-role-assignment"></a>Supprimer l’attribution de rôle

```powershell
Remove-AzureAdMSRoleAssignment -Id $roleAssignment.Id 
```

## <a name="using-microsoft-graph-api"></a>Utilisation de l’API Microsoft Graph

### <a name="create-a-group-that-can-be-assigned-an-azure-ad-role"></a>Créer un groupe assignable à un rôle Azure AD

```powershell
POST https://graph.microsoft.com/beta/groups

{
"description": "This group is assigned to Helpdesk Administrator built-in role of Azure AD",
"displayName": "Contoso_Helpdesk_Administrators",
"groupTypes": [
"Unified"
],
"mailEnabled": true,
"securityEnabled": true
"mailNickname": "contosohelpdeskadministrators",
"isAssignableToRole": true,
}
```

### <a name="get-the-role-definition"></a>Obtenir la définition de rôle

```powershell
GET https://graph.microsoft.com/beta/roleManagement/directory/roleDefinitions?$filter = displayName eq ‘Helpdesk Administrator’
```

### <a name="create-the-role-assignment"></a>Créer l’attribution de rôle

```powershell
POST https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments
{
"principalId":"<Object Id of Group>",
"roleDefinitionId":"<Id of role definition>",
"directoryScopeId":"/"
}
```

### <a name="delete-role-assignment"></a>Supprimer une attribution de rôle

```powershell
DELETE https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments/<Id of role assignment>
```

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser des groupes cloud pour gérer les attributions de rôles](groups-concept.md)
- [Résolution des problèmes des rôles attribués aux groupes cloud](groups-faq-troubleshooting.md)
