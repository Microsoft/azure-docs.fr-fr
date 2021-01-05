---
title: Attribuer un rôle à un groupe de Cloud dans Azure Active Directory | Microsoft Docs
description: Attribuez un rôle Azure AD à un groupe assignable à un rôle dans le portail Azure, PowerShell ou l’API Graph.
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
ms.openlocfilehash: fb81b5dc98bf75a17e55abafdebfbeca68774a99
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93379143"
---
# <a name="assign-a-role-to-a-cloud-group-in-azure-active-directory"></a>Attribuer un rôle à un groupe de Cloud dans Azure Active Directory

Cette section décrit comment un administrateur informatique peut attribuer le rôle Azure Active Directory (Azure AD) à un groupe de Azure AD.

## <a name="using-azure-ad-admin-center"></a>Utilisation du Centre d’administration Azure AD

L’attribution d’un groupe à un rôle Azure AD est similaire à l’affectation d’utilisateurs et de principaux de service, à ceci près que seuls les groupes qui sont assignables par rôle peuvent être utilisés. Dans le Portail Azure, seuls les groupes qui sont assignables par rôle sont affichés.

1. Connectez-vous au [centre d’administration Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) avec des autorisations Administrateur de rôle privilégié ou Administrateur général dans l’organisation Azure AD.

1. Sélectionnez **Azure Active Directory** > **rôles et administrateurs** , puis sélectionnez le rôle que vous souhaitez attribuer.

1. Sur la page **_nom du rôle_ *_, sélectionnez > _* Ajouter une affectation**.

   ![Ajouter la nouvelle affectation de rôle](./media/groups-assign-role/add-assignment.png)

1. Sélectionnez le groupe. Seuls les groupes qui peuvent être affectés à des rôles Azure AD sont affichés.

    [![Seuls les groupes qui peuvent être assignés sont affichés pour une nouvelle affectation de rôle.](./media/groups-assign-role/eligible-groups.png "Seuls les groupes qui peuvent être assignés sont affichés pour une nouvelle affectation de rôle.")](./media/groups-assign-role/eligible-groups.png#lightbox)

1. Sélectionnez **Ajouter**.

Pour plus d’informations sur l’affectation d’autorisations de rôles, consultez [Attribuer des rôles administrateur et non administrateur aux utilisateurs](../fundamentals/active-directory-users-assign-role-azure-portal.md).

## <a name="using-powershell"></a>Utilisation de PowerShell

### <a name="create-a-group-that-can-be-assigned-to-role"></a>Créer un groupe assignable à un rôle

```powershell
$group = New-AzureADMSGroup -DisplayName "Contoso_Helpdesk_Administrators" -Description "This group is assigned to Helpdesk Administrator built-in role in Azure AD." -MailEnabled $true -SecurityEnabled $true -MailNickName "contosohelpdeskadministrators" -IsAssignableToRole $true 
```

### <a name="get-the-role-definition-for-the-role-you-want-to-assign"></a>Obtenir la définition de rôle que vous souhaitez attribuer

```powershell
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Helpdesk Administrator'" 
```

### <a name="create-a-role-assignment"></a>Création d'une affectation de rôle

```powershell
$roleAssignment = New-AzureADMSRoleAssignment -ResourceScope '/' -RoleDefinitionId $roleDefinition.Id -PrincipalId $group.Id 
```

## <a name="using-microsoft-graph-api"></a>Utilisation de l’API Microsoft Graph

### <a name="create-a-group-that-can-be-assigned-azure-ad-role"></a>Créer un groupe assignable à un rôle Azure AD

```
POST https://graph.microsoft.com/beta/groups
{
"description": "This group is assigned to Helpdesk Administrator built-in role of Azure AD.",
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

```
GET https://graph.microsoft.com/beta/roleManagement/directory/roleDefinitions?$filter = displayName eq ‘Helpdesk Administrator’
```

### <a name="create-the-role-assignment"></a>Créer l’attribution de rôle

```
POST https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments
{
"principalId":"<Object Id of Group>",
"roleDefinitionId":"<ID of role definition>",
"directoryScopeId":"/"
}
```
## <a name="next-steps"></a>Étapes suivantes

- [Utiliser des groupes cloud pour gérer les attributions de rôles](groups-concept.md)
- [Résolution des problèmes des rôles attribués aux groupes cloud](groups-faq-troubleshooting.md)
