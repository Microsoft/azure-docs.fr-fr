---
title: 'Tutoriel : Accorder un accès utilisateur aux ressources Azure à l’aide d’Azure PowerShell - RBAC Azure'
description: Découvrez dans ce tutoriel comment accorder un accès utilisateur aux ressources Azure à l’aide d’Azure PowerShell et du contrôle d’accès en fonction du rôle Azure (RBAC Azure).
services: active-directory
documentationCenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: tutorial
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 02/02/2019
ms.author: rolyon
ms.openlocfilehash: a9b66d7b37397fe509cce5c66d30da0a3ae63465
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94648257"
---
# <a name="tutorial-grant-a-user-access-to-azure-resources-using-azure-powershell"></a>Tutoriel : Accorder un accès utilisateur aux ressources Azure à l’aide d’Azure PowerShell

Le [contrôle d’accès en fonction du rôle Azure (RBAC Azure)](overview.md) vous permet de gérer l’accès aux ressources Azure. Dans ce tutoriel, vous accordez l’accès à un utilisateur pour afficher tous les éléments d’un abonnement et gérer tous les éléments d’un groupe de ressources à l’aide d’Azure PowerShell.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Accorder l’accès à un utilisateur dans différentes étendues
> * Répertorier les accès
> * Suppression d'accès

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Pour exécuter ce didacticiel, les éléments suivants sont nécessaires :

- Autorisations pour créer des utilisateurs dans Azure Active Directory (ou disposer d’un utilisateur existant)
- [Azure Cloud Shell](../cloud-shell/quickstart-powershell.md)

## <a name="role-assignments"></a>Affectations de rôles

Dans le contrôle d’accès en fonction du rôle Azure, vous créez une attribution de rôle pour accorder l’accès. Une attribution de rôle se compose de trois éléments : un principal de sécurité, une définition de rôle et une étendue. Voici deux attributions de rôles que vous allez effectuer dans ce tutoriel :

| Principal de sécurité | Définition de rôle | Étendue |
| --- | --- | --- |
| Utilisateur<br>(Utilisateur du tutoriel RBAC) | [Lecteur](built-in-roles.md#reader) | Abonnement |
| Utilisateur<br>(Utilisateur du tutoriel RBAC)| [Contributeur](built-in-roles.md#contributor) | Resource group<br>(rbac-tutorial-resource-group) |

   ![Attributions de rôles pour un utilisateur](./media/tutorial-role-assignments-user-powershell/rbac-role-assignments-user.png)

## <a name="create-a-user"></a>Créer un utilisateur

Pour attribuer un rôle, vous avez besoin d’un utilisateur, d’un groupe ou d’un principal du service. Si vous n’avez pas déjà un utilisateur, vous pouvez en créer un.

1. Dans Azure Cloud Shell, créez un mot de passe conforme à vos exigences de complexité de mot de passe.

    ```azurepowershell
    $PasswordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
    $PasswordProfile.Password = "Password"
    ```

1. Créez un nouvel utilisateur pour votre domaine à l’aide de la commande [New-AzureADUser](/powershell/module/azuread/new-azureaduser).

    ```azurepowershell
    New-AzureADUser -DisplayName "RBAC Tutorial User" -PasswordProfile $PasswordProfile `
      -UserPrincipalName "rbacuser@example.com" -AccountEnabled $true -MailNickName "rbacuser"
    ```
    
    ```Example
    ObjectId                             DisplayName        UserPrincipalName    UserType
    --------                             -----------        -----------------    --------
    11111111-1111-1111-1111-111111111111 RBAC Tutorial User rbacuser@example.com Member
    ```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Vous utilisez un groupe de ressources pour montrer comment attribuer un rôle à une étendue de groupe de ressources.

1. Obtenez la liste des emplacements de région à l’aide de la commande [Get-AzLocation](/powershell/module/az.resources/get-azlocation).

   ```azurepowershell
   Get-AzLocation | select Location
   ```

1. Sélectionnez un emplacement près de chez vous et assignez-le à une variable.

   ```azurepowershell
   $location = "westus"
   ```

1. Créez un groupe de ressources avec la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

   ```azurepowershell
   New-AzResourceGroup -Name "rbac-tutorial-resource-group" -Location $location
   ```

   ```Example
   ResourceGroupName : rbac-tutorial-resource-group
   Location          : westus
   ProvisioningState : Succeeded
   Tags              :
   ResourceId        : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group
   ```

## <a name="grant-access"></a>Accorder l'accès

Pour accorder l’accès à l’utilisateur, utilisez la commande [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) afin d’attribuer un rôle. Vous devez spécifier le principal de sécurité, la définition de rôle et l’étendue.

1. Obtenez l’ID de votre abonnement à l’aide de la commande [Get-AzSubscription](/powershell/module/Az.Accounts/Get-AzSubscription).

    ```azurepowershell
    Get-AzSubscription
    ```

    ```Example
    Name     : Pay-As-You-Go
    Id       : 00000000-0000-0000-0000-000000000000
    TenantId : 22222222-2222-2222-2222-222222222222
    State    : Enabled
    ```

1. Enregistrez l’étendue de l’abonnement dans une variable.

    ```azurepowershell
    $subScope = "/subscriptions/00000000-0000-0000-0000-000000000000"
    ```

1. Attribuez le rôle de [Lecteur](built-in-roles.md#reader) à l’utilisateur dans l’étendue de l’abonnement.

    ```azurepowershell
    New-AzRoleAssignment -SignInName rbacuser@example.com `
      -RoleDefinitionName "Reader" `
      -Scope $subScope
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/44444444-4444-4444-4444-444444444444
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
    DisplayName        : RBAC Tutorial User
    SignInName         : rbacuser@example.com
    RoleDefinitionName : Reader
    RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : User
    CanDelegate        : False
    ```

1. Attribuez le rôle de [Contributeur](built-in-roles.md#contributor) à l’utilisateur dans l’étendue du groupe de ressources.

    ```azurepowershell
    New-AzRoleAssignment -SignInName rbacuser@example.com `
      -RoleDefinitionName "Contributor" `
      -ResourceGroupName "rbac-tutorial-resource-group"
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group/providers/Microsoft.Authorization/roleAssignments/33333333-3333-3333-3333-333333333333
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group
    DisplayName        : RBAC Tutorial User
    SignInName         : rbacuser@example.com
    RoleDefinitionName : Contributor
    RoleDefinitionId   : b24988ac-6180-42a0-ab88-20f7382dd24c
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : User
    CanDelegate        : False
    ```

## <a name="list-access"></a>Répertorier les accès

1. Pour vérifier l’accès à l’abonnement, utilisez la commande [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment) afin de lister les attributions de rôle.

    ```azurepowershell
    Get-AzRoleAssignment -SignInName rbacuser@example.com -Scope $subScope
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
    DisplayName        : RBAC Tutorial User
    SignInName         : rbacuser@example.com
    RoleDefinitionName : Reader
    RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : User
    CanDelegate        : False
    ```

    Dans la sortie, vous pouvez voir que le rôle de Lecteur a été attribué à l’utilisateur du tutoriel RBAC dans l’étendue de l’abonnement.

1. Pour vérifier l’accès au groupe de ressources, utilisez la commande [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment) afin de lister les attributions de rôle.

    ```azurepowershell
    Get-AzRoleAssignment -SignInName rbacuser@example.com -ResourceGroupName "rbac-tutorial-resource-group"
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group/providers/Microsoft.Authorization/roleAssignments/33333333-3333-3333-3333-333333333333
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group
    DisplayName        : RBAC Tutorial User
    SignInName         : rbacuser@example.com
    RoleDefinitionName : Contributor
    RoleDefinitionId   : b24988ac-6180-42a0-ab88-20f7382dd24c
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : User
    CanDelegate        : False
    
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
    DisplayName        : RBAC Tutorial User
    SignInName         : rbacuser@example.com
    RoleDefinitionName : Reader
    RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : User
    CanDelegate        : False
    ```

    Dans la sortie, vous pouvez voir que les rôles de Collaborateur et de Lecteur ont été attribués à l’utilisateur du tutoriel RBAC. Le rôle de Contributeur est dans l’étendue rbac-tutorial-resource-group et le rôle de Lecteur est hérité dans l’étendue de l’abonnement.

## <a name="optional-list-access-using-the-azure-portal"></a>(Facultatif) Accès à la liste à l’aide du portail Azure

1. Pour voir les attributions de rôle dans le portail Azure, affichez le panneau **Contrôle d’accès (IAM)** pour l’abonnement.

    ![Attributions de rôles pour un utilisateur dans l’étendue de l’abonnement](./media/tutorial-role-assignments-user-powershell/role-assignments-subscription-user.png)

1. Affichez le panneau **Contrôle d’accès (IAM)** pour le groupe de ressources.

    ![Attributions de rôles pour un utilisateur dans l’étendue du groupe de ressources](./media/tutorial-role-assignments-user-powershell/role-assignments-resource-group-user.png)

## <a name="remove-access"></a>Suppression d'accès

Pour supprimer l’accès accordé à des utilisateurs, des groupes ou des applications, utilisez [Remove-AzRoleAssignment](/powershell/module/az.resources/remove-azroleassignment) afin de supprimer une attribution de rôle.

1. Utilisez la commande suivante pour supprimer l’attribution de rôle de Collaborateur pour l’utilisateur dans l’étendue de groupe de ressources.

    ```azurepowershell
    Remove-AzRoleAssignment -SignInName rbacuser@example.com `
      -RoleDefinitionName "Contributor" `
      -ResourceGroupName "rbac-tutorial-resource-group"
    ```

1. Utilisez la commande suivante pour supprimer l’attribution de rôle de Lecteur pour l’utilisateur dans l’étendue d’abonnement.

    ```azurepowershell
    Remove-AzRoleAssignment -SignInName rbacuser@example.com `
      -RoleDefinitionName "Reader" `
      -Scope $subScope
    ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer les ressources créées par ce tutoriel, supprimez le groupe de ressources et l’utilisateur.

1. Supprimez le groupe de ressources à l’aide de la commande [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

    ```azurepowershell
    Remove-AzResourceGroup -Name "rbac-tutorial-resource-group"
    ```

    ```Example
    Confirm
    Are you sure you want to remove resource group 'rbac-tutorial-resource-group'
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):
    ```
    
1. Lorsque vous êtes invité à confirmer votre choix, saisissez **Y**. La suppression prendra quelques secondes.

1. Supprimez l’utilisateur à l’aide de la commande [Remove-AzureADUser](/powershell/module/azuread/remove-azureaduser).

    ```azurepowershell
    Remove-AzureADUser -ObjectId "rbacuser@example.com"
    ```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Ajouter ou supprimer des attributions de rôles Azure à l’aide d’Azure PowerShell](role-assignments-powershell.md)