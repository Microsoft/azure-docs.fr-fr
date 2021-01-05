---
title: 'Tutoriel : Accorder à un groupe l’accès aux ressources Azure à l’aide d’Azure PowerShell - RBAC Azure'
description: Découvrez dans ce tutoriel comment accorder à un groupe l’accès aux ressources Azure à l’aide d’Azure PowerShell et du contrôle d’accès en fonction du rôle Azure (RBAC Azure).
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
ms.openlocfilehash: 9b1a2f87498d7869b21e6a8ab44fc1f9660456ad
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94648291"
---
# <a name="tutorial-grant-a-group-access-to-azure-resources-using-azure-powershell"></a>Tutoriel : Accorder à un groupe l’accès aux ressources Azure à l’aide d’Azure PowerShell

Le [contrôle d’accès en fonction du rôle Azure (RBAC Azure)](overview.md) vous permet de gérer l’accès aux ressources Azure. Dans ce tutoriel, vous accordez l’accès à un groupe pour afficher tous les éléments d’un abonnement et gérer tous les éléments d’un groupe de ressources à l’aide d’Azure PowerShell.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Accorder l’accès à un groupe dans différentes étendues
> * Répertorier les accès
> * Suppression d'accès

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Pour exécuter ce didacticiel, les éléments suivants sont nécessaires :

- Autorisations pour créer des groupes dans Azure Active Directory (ou disposer d’un groupe existant)
- [Azure Cloud Shell](../cloud-shell/quickstart-powershell.md)

## <a name="role-assignments"></a>Affectations de rôles

Dans le contrôle d’accès en fonction du rôle Azure, vous créez une attribution de rôle pour accorder l’accès. Une attribution de rôle se compose de trois éléments : un principal de sécurité, une définition de rôle et une étendue. Voici deux attributions de rôles que vous allez effectuer dans ce tutoriel :

| Principal de sécurité | Définition de rôle | Étendue |
| --- | --- | --- |
| Groupe<br>(Groupe du tutoriel RBAC) | [Lecteur](built-in-roles.md#reader) | Abonnement |
| Groupe<br>(Groupe du tutoriel RBAC)| [Contributeur](built-in-roles.md#contributor) | Resource group<br>(rbac-tutorial-resource-group) |

   ![Attributions de rôles pour un groupe](./media/tutorial-role-assignments-group-powershell/rbac-role-assignments.png)

## <a name="create-a-group"></a>Créer un groupe

Pour attribuer un rôle, vous avez besoin d’un utilisateur, d’un groupe ou d’un principal du service. Si vous n’avez pas déjà un groupe, vous pouvez en créer un.

- Dans Azure Cloud Shell, créez un nouveau groupe à l’aide de la commande [New-AzureADGroup](/powershell/module/azuread/new-azureadgroup).

   ```azurepowershell
   New-AzureADGroup -DisplayName "RBAC Tutorial Group" `
     -MailEnabled $false -SecurityEnabled $true -MailNickName "NotSet"
   ```

   ```Example
   ObjectId                             DisplayName         Description
   --------                             -----------         -----------
   11111111-1111-1111-1111-111111111111 RBAC Tutorial Group
   ```

Si vous n’avez pas les autorisations pour créer des groupes, vous pouvez suivre le [Tutoriel : Accorder un accès utilisateur aux ressources Azure à l’aide d’Azure PowerShell](tutorial-role-assignments-user-powershell.md) à la place.

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

Pour accorder l’accès au groupe, utilisez la commande [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) afin d’attribuer un rôle. Vous devez spécifier le principal de sécurité, la définition de rôle et l’étendue.

1. Obtenez l’ID d’objet du groupe en utilisant la commande [Get-AzureADGroup](/powershell/module/azuread/new-azureadgroup).

    ```azurepowershell
    Get-AzureADGroup -SearchString "RBAC Tutorial Group"
    ```

    ```Example
    ObjectId                             DisplayName         Description
    --------                             -----------         -----------
    11111111-1111-1111-1111-111111111111 RBAC Tutorial Group
    ```

1. Enregistrez l’ID d’objet du groupe dans une variable.

    ```azurepowershell
    $groupId = "11111111-1111-1111-1111-111111111111"
    ```

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

1. Attribuez le rôle de [Lecteur](built-in-roles.md#reader) au groupe dans l’étendue de l’abonnement.

    ```azurepowershell
    New-AzRoleAssignment -ObjectId $groupId `
      -RoleDefinitionName "Reader" `
      -Scope $subScope
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/44444444-4444-4444-4444-444444444444
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Reader
    RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    ```

1. Attribuez le rôle de [Contributeur](built-in-roles.md#contributor) au groupe dans l’étendue du groupe de ressources.

    ```azurepowershell
    New-AzRoleAssignment -ObjectId $groupId `
      -RoleDefinitionName "Contributor" `
      -ResourceGroupName "rbac-tutorial-resource-group"
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group/providers/Microsoft.Authorization/roleAssignments/33333333-3333-3333-3333-333333333333
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Contributor
    RoleDefinitionId   : b24988ac-6180-42a0-ab88-20f7382dd24c
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    ```

## <a name="list-access"></a>Répertorier les accès

1. Pour vérifier l’accès à l’abonnement, utilisez la commande [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment) afin de lister les attributions de rôle.

    ```azurepowershell
    Get-AzRoleAssignment -ObjectId $groupId -Scope $subScope
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Reader
    RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    ```

    Dans la sortie, vous pouvez voir que le rôle de Lecteur a été attribué au groupe du tutoriel RBAC dans l’étendue de l’abonnement.

1. Pour vérifier l’accès au groupe de ressources, utilisez la commande [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment) afin de lister les attributions de rôle.

    ```azurepowershell
    Get-AzRoleAssignment -ObjectId $groupId -ResourceGroupName "rbac-tutorial-resource-group"
    ```

    ```Example
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group/providers/Microsoft.Authorization/roleAssignments/33333333-3333-3333-3333-333333333333
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Contributor
    RoleDefinitionId   : b24988ac-6180-42a0-ab88-20f7382dd24c
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    
    RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222
    Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
    DisplayName        : RBAC Tutorial Group
    SignInName         :
    RoleDefinitionName : Reader
    RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
    ObjectId           : 11111111-1111-1111-1111-111111111111
    ObjectType         : Group
    CanDelegate        : False
    ```

    Dans la sortie, vous pouvez voir que les rôles de Collaborateur et de Lecteur ont été attribués au groupe du tutoriel RBAC. Le rôle de Contributeur est dans l’étendue rbac-tutorial-resource-group et le rôle de Lecteur est hérité dans l’étendue de l’abonnement.

## <a name="optional-list-access-using-the-azure-portal"></a>(Facultatif) Accès à la liste à l’aide du portail Azure

1. Pour voir les attributions de rôle dans le portail Azure, affichez le panneau **Contrôle d’accès (IAM)** pour l’abonnement.

    ![Attributions de rôles pour un groupe dans l’étendue de l’abonnement](./media/tutorial-role-assignments-group-powershell/role-assignments-subscription.png)

1. Affichez le panneau **Contrôle d’accès (IAM)** pour le groupe de ressources.

    ![Attributions de rôles pour un groupe dans l’étendue du groupe de ressources](./media/tutorial-role-assignments-group-powershell/role-assignments-resource-group.png)

## <a name="remove-access"></a>Suppression d'accès

Pour supprimer l’accès accordé à des utilisateurs, des groupes ou des applications, utilisez [Remove-AzRoleAssignment](/powershell/module/az.resources/remove-azroleassignment) afin de supprimer une attribution de rôle.

1. Utilisez la commande suivante pour supprimer l’attribution de rôle de Collaborateur pour le groupe dans l’étendue de groupe de ressources.

    ```azurepowershell
    Remove-AzRoleAssignment -ObjectId $groupId `
      -RoleDefinitionName "Contributor" `
      -ResourceGroupName "rbac-tutorial-resource-group"
    ```

1. Utilisez la commande suivante pour supprimer l’attribution de rôle de Lecteur pour le groupe dans l’étendue d’abonnement.

    ```azurepowershell
    Remove-AzRoleAssignment -ObjectId $groupId `
      -RoleDefinitionName "Reader" `
      -Scope $subScope
    ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer les ressources créées par ce tutoriel, supprimez le groupe de ressources et le groupe.

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

1. Supprimez le groupe à l’aide de la commande [Remove-AzureADGroup](/powershell/module/azuread/remove-azureadgroup).

    ```azurepowershell
    Remove-AzureADGroup -ObjectId $groupId
    ```
    
    Si vous recevez une erreur lorsque vous essayez de supprimer le groupe, vous pouvez également supprimer le groupe dans le portail.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Ajouter ou supprimer des attributions de rôles Azure à l’aide d’Azure PowerShell](role-assignments-powershell.md)