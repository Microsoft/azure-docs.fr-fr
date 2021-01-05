---
title: Répertorier les affectations de refus Azure à l’aide d’Azure PowerShell - RBAC Azure
description: Découvrez comment répertorier les utilisateurs, groupes, principaux du service et identités managées auxquels l’accès à des actions de ressource Azure spécifiques a été refusé dans des étendues particulières à l’aide d’Azure PowerShell et du contrôle d’accès en fonction du rôle (RBAC Azure).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/12/2019
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 6cf379b051ba42be2d7df7d288f07cdc2a0002a1
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94657633"
---
# <a name="list-azure-deny-assignments-using-azure-powershell"></a>Répertorier les affectations de refus Azure à l’aide d’Azure PowerShell

Les [affectations de refus Azure](deny-assignments.md) empêchent les utilisateurs d’effectuer des actions particulières sur les ressources Azure, même si une attribution de rôle leur confère un accès. Cet article décrit comment répertorier les affectations de refus à l’aide d’Azure PowerShell.

> [!NOTE]
> Vous ne pouvez pas directement créer vos propres affectations de refus. Pour obtenir des informations sur la création des affectations de refus, consultez [Affectations de refus Azure](deny-assignments.md).

## <a name="prerequisites"></a>Prérequis

Pour obtenir des informations sur une affectation de refus, vous devez disposer de :

- l’autorisation `Microsoft.Authorization/denyAssignments/read`, qui est incluse dans la plupart des [rôles intégrés Azure](built-in-roles.md)
- [PowerShell dans Azure Cloud Shell](../cloud-shell/overview.md) ou [Azure PowerShell](/powershell/azure/install-az-ps)

## <a name="list-deny-assignments"></a>Répertorier les affectations de refus

### <a name="list-all-deny-assignments"></a>Répertorier toutes les affectations de refus

Pour répertorier toutes les affectations de refus pour l’abonnement actuel, utilisez [Get-AzDenyAssignment](/powershell/module/az.resources/get-azdenyassignment).

```azurepowershell
Get-AzDenyAssignment
```

```Example
PS C:\> Get-AzDenyAssignment

Id                      : 22222222-2222-2222-2222-222222222222
DenyAssignmentName      : Deny assignment '22222222-2222-2222-2222-222222222222' created by Blueprint Assignment
                          '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Description             : Created by Blueprint Assignment '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Actions                 : {*}
NotActions              : {*/read}
DataActions             : {}
NotDataActions          : {}
Scope                   : /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/TestingBPLocks
DoNotApplyToChildScopes : True
Principals              : {
                          DisplayName:  All Principals
                          ObjectType:   SystemDefined
                          ObjectId:     00000000-0000-0000-0000-000000000000
                          }
ExcludePrincipals       : {
                          ObjectType:   ServicePrincipal
                          }
IsSystemProtected       : True

Id                      : 33333333-3333-3333-3333-333333333333
DenyAssignmentName      : Deny assignment '33333333-3333-3333-3333-333333333333' created by Blueprint Assignment
                          '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Description             : Created by Blueprint Assignment '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Actions                 : {*}
NotActions              : {*/read}
DataActions             : {}
NotDataActions          : {}
Scope                   : /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/TestingBPLocks/providers/Microsoft.Storage/storageAccounts/storep6vkuxmu4m4pq
DoNotApplyToChildScopes : True
Principals              : {
                          DisplayName:  All Principals
                          ObjectType:   SystemDefined
                          ObjectId:     00000000-0000-0000-0000-000000000000
                          }
ExcludePrincipals       : {
                          DisplayName:  assignment-locked-storageaccount-TestingBPLocks
                          ObjectType:   ServicePrincipal
                          ObjectId:     2311a0b7-657a-4ca2-af6f-d1c33f6d2fff
                          }
IsSystemProtected       : True
```

### <a name="list-deny-assignments-at-a-resource-group-scope"></a>Répertorier les affectations de refus dans l’étendue d’un groupe de ressources

Pour répertorier toutes les affectations de refus dans l’étendue d’un groupe de ressources, utilisez [Get-AzDenyAssignment](/powershell/module/az.resources/get-azdenyassignment).

```azurepowershell
Get-AzDenyAssignment -ResourceGroupName <resource_group_name>
```

```Example
PS C:\> Get-AzDenyAssignment -ResourceGroupName TestingBPLocks | FL DenyAssignmentName, Scope

DenyAssignmentName : Deny assignment '22222222-2222-2222-2222-222222222222' created by Blueprint Assignment
                     '/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Blueprint/blueprintAssignments/assignment-locked-storageaccount-TestingBPLocks'.
Scope              : /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/TestingBPLocks
Principals         : {
                     DisplayName:       All Principals
                     ObjectType:        SystemDefined
                     ObjectId:  00000000-0000-0000-0000-000000000000
                     }
```

### <a name="list-deny-assignments-at-a-subscription-scope"></a>Répertorier les affectations de refus dans l’étendue d’un abonnement

Pour répertorier toutes les affectations de refus dans l’étendue d’un abonnement, utilisez [Get-AzDenyAssignment](/powershell/module/az.resources/get-azdenyassignment). Pour obtenir l’ID d’abonnement, vous pouvez le trouver dans le panneau **Abonnements** du portail Azure ou utiliser [Get-AzSubscription](/powershell/module/Az.Accounts/Get-AzSubscription).

```azurepowershell
Get-AzDenyAssignment -Scope /subscriptions/<subscription_id>
```

```Example
PS C:\> Get-AzDenyAssignment -Scope /subscriptions/11111111-1111-1111-1111-111111111111
```

## <a name="next-steps"></a>Étapes suivantes

- [Comprendre les affectations de refus Azure](deny-assignments.md)
- [Répertorier les affectations de refus avec le portail Azure](deny-assignments-portal.md)
- [Répertorier les affectations de refus Azure à l’aide de l’API REST](deny-assignments-rest.md)