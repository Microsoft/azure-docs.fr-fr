---
title: Afficher les attributions de rôles personnalisées dans le portail Azure Active Directory | Microsoft Docs
description: Vous pouvez désormais afficher et gérer les membres d’un rôle d’administrateur Azure Active Directory dans le centre d’administration Azure Active Directory.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: how-to
ms.date: 11/05/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8546c2b50c78c7a24d07c75d2d730fd2566e8ba0
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96862086"
---
# <a name="view-custom-role-assignments-using-azure-active-directory"></a>Afficher les attributions de rôles personnalisés à l’aide d’Azure Active Directory

Cet article explique comment afficher les rôles personnalisés que vous avez affectés dans Azure Active Directory (Azure AD). Dans Azure Active Directory (Azure AD), les rôles peuvent être affectés au niveau de l’organisation ou d’une seule application.

- Les attributions de rôles au niveau de l’étendue de l’organisation sont ajoutées à et peuvent être consultées dans la liste des attributions de rôle d’application unique.
- Les attributions de rôles au niveau d’une application unique ne sont pas ajoutées à et ne peuvent pas être affichées dans la liste des attributions à l’échelle de l’organisation.

## <a name="view-role-assignments-in-the-azure-portal"></a>Afficher les attributions de rôles dans le portail Azure

Cette procédure décrit l’affichage des attributions d’un rôle avec une étendue à l’échelle de l’organisation.

1. Connectez-vous au [Centre d’administration Azure AD](https://aad.portal.azure.com) avec des autorisations Administrateur de rôle privilégié ou Administrateur général dans l’organisation Azure AD.
1. Sélectionnez **Azure Active Directory**, sélectionnez **Rôles et administrateurs**, puis sélectionnez un rôle pour l’ouvrir et voir ses propriétés.
1. Sélectionnez **Affectations** pour afficher les affectations du rôle.

    ![Affichez les affectations de rôle et les autorisations lorsque vous ouvrez un rôle dans la liste](./media/view-assignments/role-assignments.png)

## <a name="view-role-assignments-using-azure-ad-powershell"></a>Afficher les attributions de rôles à l’aide d’Azure AD PowerShell

Cette section décrit l’affichage des attributions d’un rôle avec une étendue à l’échelle de l’organisation. Cet article utilise le module [Azure Active Directory PowerShell Version 2](/powershell/module/azuread/#directory_roles). Pour afficher les attributions pour une application unique à l’aide de PowerShell, vous pouvez utiliser les applets de commande dans [Attribuer des rôles personnalisés avec PowerShell.](custom-assign-powershell.md)

### <a name="prepare-powershell"></a>Préparer PowerShell

Vous devez tout d’abord [télécharger le module de préversion Azure AD PowerShell](https://www.powershellgallery.com/packages/AzureAD/).

Pour installer le module PowerShell Azure AD, utilisez les commandes suivantes :

``` PowerShell
install-module azureadpreview
import-module azureadpreview
```

Pour vérifier que le module est prêt à l’emploi, utilisez la commande suivante :

``` PowerShell
get-module azuread
  ModuleType Version      Name                         ExportedCommands
  ---------- ---------    ----                         ----------------
  Binary     2.0.0.115    azuread                      {Add-AzureADAdministrati...}
```

### <a name="view-the-assignments-of-a-role"></a>Afficher les affectations d’un rôle

Exemple d’affichage des affectations d’un rôle.

``` PowerShell
# Fetch list of all directory roles with object ID
Get-AzureADDirectoryRole

# Fetch a specific directory role by ID
$role = Get-AzureADDirectoryRole -ObjectId "5b3fe201-fa8b-4144-b6f1-875829ff7543"

# Fetch role membership for a role
Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId | Get-AzureADUser
```

## <a name="view-role-assignments-using-microsoft-graph-api"></a>Afficher les attributions de rôles à l’aide de l’API Microsoft Graph

Cette section décrit l’affichage des attributions d’un rôle avec une étendue à l’échelle de l’organisation.  Pour afficher les attributions pour une application unique à l’aide de l’API Graph, vous pouvez utiliser les opérations dans [Attribuer des rôles personnalisés avec l’API Graph.](custom-assign-graph.md)

Requête HTTP pour obtenir une attribution de rôle pour une définition de rôle donnée

GET

``` HTTP
https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments&$filter=roleDefinitionId eq ‘<object-id-or-template-id-of-role-definition>’
```

response

``` HTTP
HTTP/1.1 200 OK
{
    "id":"CtRxNqwabEKgwaOCHr2CGJIiSDKQoTVJrLE9etXyrY0-1"
    "principalId":"ab2e1023-bddc-4038-9ac1-ad4843e7e539",
    "roleDefinitionId":"3671d40a-1aac-426c-a0c1-a3821ebd8218",
    "resourceScopes":["/"]
}
```

## <a name="view-assignments-of-single-application-scope"></a>Afficher les attributions pour une application unique

Cette section décrit l’affichage des attributions d’un rôle avec une étendue à l’échelle d’une application unique. Cette fonctionnalité est actuellement disponible en préversion publique.

1. Connectez-vous au [Centre d’administration Azure AD](https://aad.portal.azure.com) avec des autorisations Administrateur de rôle privilégié ou Administrateur général dans l’organisation Azure AD.
1. Sélectionnez **Inscriptions d’applications**, puis sélectionnez l’inscription d’application pour afficher ses propriétés. Vous devrez peut-être sélectionner **Toutes les applications** pour afficher la liste complète des inscriptions d’applications dans votre organisation Azure AD.

    ![Créer ou modifier des inscriptions d’applications à partir de la page Inscriptions d’applications](./media/view-assignments/app-reg-all-apps.png)

1. Lors de l’inscription de l’application, sélectionnez **Rôles et administrateurs**, puis sélectionnez un rôle pour voir ses propriétés.

    ![Afficher les attributions de rôle d’inscription d’application à partir de la page Inscriptions d’applications](./media/view-assignments/app-reg-assignments.png)

1. Sélectionnez **Affectations** pour afficher les affectations du rôle. L’ouverture de la vue des attributions à partir de l’inscription de l’application vous montre les attributions limitées à cette ressource Azure AD.

    ![Afficher les attributions de rôle d’inscription d’application à partir des propriétés d’une inscription d’application](./media/view-assignments/app-reg-assignments-2.png)

## <a name="next-steps"></a>Étapes suivantes

* N’hésitez pas à nous donner votre avis sur le [forum des rôles d’administrateur Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).
* Pour plus d’informations sur les rôles et l’attribution de rôle d’administrateur, consultez [Attribuer des rôles d’administrateur](permissions-reference.md).
* Pour les autorisations d’utilisateur par défaut, consultez une [comparaison des autorisations par défaut d’un utilisateur invité et d’un membre](../fundamentals/users-default-permissions.md).
