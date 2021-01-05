---
title: Accorder des autorisations à des utilisateurs sur des stratégies de laboratoire spécifiques | Microsoft Docs
description: Découvrez comment accorder des autorisations aux utilisateurs sur des stratégies de laboratoire spécifique dans DevTest Labs selon les besoins de chaque utilisateur
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 976862476d25e4e9a4933d8a5319eec9d77ca39b
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92328468"
---
# <a name="grant-user-permissions-to-specific-lab-policies"></a>Accorder des autorisations à des utilisateurs sur des stratégies de laboratoire spécifiques
## <a name="overview"></a>Vue d’ensemble
Cet article explique comment utiliser PowerShell pour accorder à des utilisateurs des autorisations sur une stratégie de laboratoire particulière. De cette façon, les autorisations peuvent être appliquées selon les besoins de chaque utilisateur. Par exemple, vous pouvez accorder à un utilisateur la possibilité de modifier les paramètres de stratégie d’une machine virtuelle, mais pas les stratégies de coût.

## <a name="policies-as-resources"></a>Stratégies en tant que ressources
Comme expliqué dans l’article [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/role-assignments-portal.md), Azure RBAC permet une gestion précise de l’accès aux ressources pour Azure. Avec Azure RBAC, vous pouvez séparer les tâches au sein de votre équipe chargée des opérations de développement et accorder aux utilisateurs uniquement les accès nécessaires pour accomplir leur travail.

Dans DevTest Labs, une stratégie est un type de ressource qui active l’action Azure RBAC **Microsoft.DevTestLab/labs/policySets/policies/** . Chaque stratégie de laboratoire est une ressource de type stratégie et peut être affectée comme étendue à un rôle Azure.

Par exemple, pour accorder à des utilisateurs une autorisation de lecture/écriture sur la stratégie **Tailles de machine virtuelle autorisées** , vous créez un rôle personnalisé qui fonctionne avec l’action **Microsoft.DevTestLab/labs/policySets/policies/** , puis vous affectez les utilisateurs appropriés à ce rôle personnalisé dans l’étendue de **Microsoft.DevTestLab/labs/policySets/policies/AllowedVmSizesInLab** .

Pour plus d’informations sur les rôles personnalisés dans Azure RBAC, consultez la page [Rôles personnalisés Azure](../role-based-access-control/custom-roles.md).

## <a name="creating-a-lab-custom-role-using-powershell"></a>Création d’un rôle personnalisé de laboratoire en utilisant PowerShell
Pour commencer, vous devez [installer Azure PowerShell](/powershell/azure/install-az-ps). 

Une fois que vous avez configuré les applets de commande Azure PowerShell, vous pouvez effectuer les tâches suivantes :

* Répertorier toutes les opérations/actions d’un fournisseur de ressources
* Répertorier les actions d’un rôle particulier :
* Créer un rôle personnalisé

Le script PowerShell suivant montre des exemples permettant d’effectuer ces tâches :

```azurepowershell
# List all the operations/actions for a resource provider.
Get-AzProviderOperation -OperationSearchString "Microsoft.DevTestLab/*"

# List actions in a particular role.
(Get-AzRoleDefinition "DevTest Labs User").Actions

# Create custom role.
$policyRoleDef = (Get-AzRoleDefinition "DevTest Labs User")
$policyRoleDef.Id = $null
$policyRoleDef.Name = "Policy Contributor"
$policyRoleDef.IsCustom = $true
$policyRoleDef.AssignableScopes.Clear()
$policyRoleDef.AssignableScopes.Add("/subscriptions/<SubscriptionID> ")
$policyRoleDef.Actions.Add("Microsoft.DevTestLab/labs/policySets/policies/*")
$policyRoleDef = (New-AzRoleDefinition -Role $policyRoleDef)
```

## <a name="assigning-permissions-to-a-user-for-a-specific-policy-using-custom-roles"></a>Attribution d'autorisations à un utilisateur pour une stratégie spécifique à l'aide de rôles personnalisés
Une fois que vous avez défini vos rôles personnalisés, vous pouvez les attribuer aux utilisateurs. Pour affecter un rôle personnalisé à un utilisateur, vous devez d’abord obtenir **l’ObjectId** représentant cet utilisateur. Pour cela, utilisez l’applet de commande **Get-AzADUser** .

Dans l’exemple suivant, **l’ObjectId** de l’utilisateur *SomeUser* est 05DEFF7B-0AC3-4ABF-B74D-6A72CD5BF3F3.

```azurepowershell
PS C:\>Get-AzADUser -SearchString "SomeUser"

DisplayName                    Type                           ObjectId
-----------                    ----                           --------
someuser@hotmail.com                                          05DEFF7B-0AC3-4ABF-B74D-6A72CD5BF3F3
```

Une fois que vous avez l’ **ObjectId** de l’utilisateur et un nom de rôle personnalisé, vous pouvez attribuer ce rôle à l’utilisateur avec l’applet de commande **New-AzRoleAssignment**  :

```azurepowershell
PS C:\>New-AzRoleAssignment -ObjectId 05DEFF7B-0AC3-4ABF-B74D-6A72CD5BF3F3 -RoleDefinitionName "Policy Contributor" -Scope /subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.DevTestLab/labs/<LabName>/policySets/default/policies/AllowedVmSizesInLab
```

Dans l’exemple précédent, la stratégie **AllowedVmSizesInLab** est utilisée. Vous pouvez utiliser une des stratégies suivantes :

* MaxVmsAllowedPerUser
* MaxVmsAllowedPerLab
* AllowedVmSizesInLab
* LabVmsShutdown

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Étapes suivantes
Après avoir accordé aux utilisateurs des autorisations sur des stratégies de laboratoire spécifiques, voici des étapes à prendre en compte :

* [Sécuriser l’accès à un laboratoire](devtest-lab-add-devtest-user.md)
* [Définir des stratégies de laboratoire](devtest-lab-set-lab-policy.md)
* [Créer un modèle de laboratoire](devtest-lab-create-template.md)
* [Créer des artefacts personnalisés pour vos machines virtuelles](devtest-lab-artifact-author.md)
* [Ajouter une machine virtuelle à un laboratoire](devtest-lab-add-vm.md)
