---
title: Synchronisation délimitée à l’aide de PowerShell pour Azure AD Domain Services | Microsoft Docs
description: Découvrez comment utiliser Azure AD PowerShell pour configurer une synchronisation à étendue limitée entre Azure AD et un domaine managé Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/24/2020
ms.author: justinha
ms.openlocfilehash: c078117baf84d7dbfaaaa2b569abb8a5f5c67e6d
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619009"
---
# <a name="configure-scoped-synchronization-from-azure-ad-to-azure-active-directory-domain-services-using-azure-ad-powershell"></a>Configuration d’une synchronisation à étendue limitée entre Azure AD et Azure Active Directory Domain Services à l’aide d’Azure AD PowerShell

Pour assurer des services d’authentification, Azure Active Directory Domain Services (Azure AD DS) synchronise les utilisateurs et les groupes à partir d’Azure AD. Dans un environnement hybride, les utilisateurs et les groupes d’un environnement Active Directory Domain Services (AD DS) local peuvent d’abord être synchronisés avec Azure AD via Azure AD Connect, puis avec Azure AD DS.

Par défaut, tous les utilisateurs et groupes d’un annuaire Azure AD sont synchronisés avec un domaine managé Azure AD DS. Si vous avez des besoins spécifiques, vous pouvez opter pour la synchronisation d’un ensemble défini d’utilisateurs.

Cet article vous montre comment créer un domaine managé qui utilise une synchronisation délimitée, puis modifier ou désactiver le groupe d’utilisateurs ciblés à l’aide d’Azure AD PowerShell. Vous pouvez également [effectuer ces étapes à l’aide du portail Azure][scoped-sync].

## <a name="before-you-begin"></a>Avant de commencer

Pour faire ce qui est décrit dans cet article, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, suivez le tutoriel pour [créer et configurer un domaine managé Azure Active Directory Domain Services][tutorial-create-instance].
* Vous devez disposer de privilèges *Administrateur général* dans votre locataire Azure AD pour modifier l’étendue de la synchronisation Azure AD DS.

## <a name="scoped-synchronization-overview"></a>Vue d’ensemble de la synchronisation délimitée

Par défaut, tous les utilisateurs et groupes d’un annuaire Azure AD sont synchronisés avec un domaine managé. Si seuls quelques utilisateurs ont besoin d’accéder au domaine managé, vous pouvez synchroniser uniquement ces comptes d’utilisateurs. Cette synchronisation délimitée est basée sur les groupes. Quand vous configurez la synchronisation délimitée basée sur les groupes, seuls les comptes d’utilisateurs qui appartiennent aux groupes que vous spécifiez sont synchronisés avec le domaine managé. Les groupes imbriqués ne sont pas synchronisés ; seuls les groupes explicitement sélectionnés le sont.

Vous pouvez modifier l’étendue de synchronisation lors de la création du domaine managé ou une fois qu’il est déployé. À présent, il est également possible de modifier l’étendue de la synchronisation sur un domaine managé existant sans avoir à le recréer.

Pour en savoir plus sur le processus de synchronisation, consultez [Comprendre la synchronisation dans Azure AD Domain Services][concepts-sync].

> [!WARNING]
> La changement de l’étendue de la synchronisation conduit le domaine managé à resynchroniser toutes les données. Les considérations suivantes s'appliquent :
>
>  * Lorsque vous modifiez l’étendue de synchronisation d’un domaine managé, une resynchronisation complète est déclenchée.
>  * Les objets qui ne sont plus requis dans le domaine managé sont supprimés. De nouveaux objets sont créés dans le domaine managé.

## <a name="powershell-script-for-scoped-synchronization"></a>Script PowerShell pour la synchronisation délimitée

Pour configurer la synchronisation délimitée à l’aide de PowerShell, commencez par enregistrer le script suivant dans un fichier nommé `Select-GroupsToSync.ps1`.

Ce script configure Azure AD DS de façon à synchroniser les groupes sélectionnés à partir d’Azure AD. Tous les comptes d’utilisateurs qui font partie des groupes spécifiés sont synchronisés avec le domaine managé.

Ce script est utilisé dans les étapes supplémentaires de cet article.

```powershell
param (
    [Parameter(Position = 0)]
    [String[]]$groupsToAdd
)

Connect-AzureAD
$sp = Get-AzureADServicePrincipal -Filter "AppId eq '2565bd9d-da50-47d4-8b85-4c97f669dc36'"
$role = $sp.AppRoles | where-object -FilterScript {$_.DisplayName -eq "User"}

Write-Output "`n****************************************************************************"

Write-Output "Total group-assignments need to be added: $($groupsToAdd.Count)"
$newGroupIds = New-Object 'System.Collections.Generic.HashSet[string]'
foreach ($groupName in $groupsToAdd)
{
    try
    {
        $group = Get-AzureADGroup -Filter "DisplayName eq '$groupName'"
        $newGroupIds.Add($group.ObjectId)

        Write-Output "Group-Name: $groupName, Id: $($group.ObjectId)"
    }
    catch
    {
        Write-Error "Failed to find group: $groupName. Exception: $($_.Exception)."
    }
}

Write-Output "****************************************************************************`n"
Write-Output "`n****************************************************************************"

$currentAssignments = Get-AzureADServiceAppRoleAssignment -ObjectId $sp.ObjectId
Write-Output "Total current group-assignments: $($currentAssignments.Count), SP-ObjectId: $($sp.ObjectId)"

$currAssignedObjectIds = New-Object 'System.Collections.Generic.HashSet[string]'
foreach ($assignment in $currentAssignments)
{
    Write-Output "Assignment-ObjectId: $($assignment.PrincipalId)"

    if ($newGroupIds.Contains($assignment.PrincipalId) -eq $false)
    {
        Write-Output "This assignment is not needed anymore. Removing it! Assignment-ObjectId: $($assignment.PrincipalId)"
        Remove-AzureADServiceAppRoleAssignment -ObjectId $sp.ObjectId -AppRoleAssignmentId $assignment.ObjectId
    }
    else
    {
        $currAssignedObjectIds.Add($assignment.PrincipalId)
    }
}

Write-Output "****************************************************************************`n"
Write-Output "`n****************************************************************************"

foreach ($id in $newGroupIds)
{
    try
    {
        if ($currAssignedObjectIds.Contains($id) -eq $false)
        {
            Write-Output "Adding new group-assignment. Role-Id: $($role.Id), Group-Object-Id: $id, ResourceId: $($sp.ObjectId)"
            New-AzureADGroupAppRoleAssignment -Id $role.Id -ObjectId $id -PrincipalId $id -ResourceId $sp.ObjectId
        }
        else
        {
            Write-Output "Group-ObjectId: $id is already assigned."
        }
    }
    catch
    {
        Write-Error "Exception occurred assigning Object-ID: $id. Exception: $($_.Exception)."
    }
}

Write-Output "****************************************************************************`n"
```

## <a name="enable-scoped-synchronization"></a>Activer synchronisation délimitée

Pour activer la synchronisation délimitée basée sur les groupes pour un domaine managé, procédez comme suit :

1. Commencez par définir *"filteredSync" = "Enabled"* sur la ressource Azure AD DS, puis mettez à jour le domaine managé.

    Lorsque vous y êtes invité, spécifiez les informations d’identification d’un *administrateur général* pour vous connecter à votre locataire Azure AD à l’aide de la cmdlet [Connect-AzureAD][Connect-AzureAD] :

    ```powershell
    // Connect to your Azure AD tenant
    Connect-AzureAD

    // Retrieve the Azure AD DS resource.
    $DomainServicesResource = Get-AzResource -ResourceType "Microsoft.AAD/DomainServices"

    // Enable group-based scoped synchronization.
    $enableScopedSync = @{"filteredSync" = "Enabled"}

    // Update the Azure AD DS resource
    Set-AzResource -Id $DomainServicesResource.ResourceId -Properties $enableScopedSync
    ```

1. Spécifiez à présent la liste des groupes dont les utilisateurs doivent être synchronisés avec le domaine managé.

    Exécutez le script `Select-GroupsToSync.ps1` et spécifiez la liste des groupes à synchroniser. Dans l’exemple suivant, les groupes à synchroniser sont *GroupName1* et *GroupName2*.

    > [!WARNING]
    > Vous devez inclure le groupe *AAD DC Administrators* dans la liste des groupes configurés pour la synchronisation délimitée. Si vous n’incluez pas ce groupe, le domaine managé est inutilisable.

    ```powershell
    .\Select-GroupsToSync.ps1 -groupsToAdd @("AAD DC Administrators", "GroupName1", "GroupName2")
    ```

La changement de l’étendue de la synchronisation conduit le domaine managé à resynchroniser toutes les données. Les objets qui ne sont plus nécessaires dans le domaine managé sont supprimés, et la resynchronisation peut prendre du temps.

## <a name="modify-scoped-synchronization"></a>Modifier la synchronisation délimitée

Pour modifier la liste des groupes dont les utilisateurs doivent être synchronisés sur le domaine managé, exécutez le script `Select-GroupsToSync.ps1` et spécifiez la nouvelle liste de groupes à synchroniser.

Dans l’exemple suivant, les groupes à synchroniser n’incluent plus *GroupName2*, mais incluent maintenant *GroupName3*.

> [!WARNING]
> Vous devez inclure le groupe *AAD DC Administrators* dans la liste des groupes configurés pour la synchronisation délimitée. Si vous n’incluez pas ce groupe, le domaine managé est inutilisable.

Lorsque vous y êtes invité, spécifiez les informations d’identification d’un *administrateur général* pour vous connecter à votre locataire Azure AD à l’aide de la cmdlet [Connect-AzureAD][Connect-AzureAD] :

```powershell
.\Select-GroupsToSync.ps1 -groupsToAdd @("AAD DC Administrators", "GroupName1", "GroupName3")
```

La changement de l’étendue de la synchronisation conduit le domaine managé à resynchroniser toutes les données. Les objets qui ne sont plus nécessaires dans le domaine managé sont supprimés, et la resynchronisation peut prendre du temps.

## <a name="disable-scoped-synchronization"></a>Désactiver la synchronisation délimitée

Pour désactiver la synchronisation délimitée basée sur les groupes pour un domaine managé, définissez *"filteredSync" = "Disabled"* sur la ressource Azure AD DS, puis mettez à jour le domaine managé. À l’issue de cette opération, l’ensemble des utilisateurs et des groupes sont définis pour être synchronisés à partir d’Azure AD.

Lorsque vous y êtes invité, spécifiez les informations d’identification d’un *administrateur général* pour vous connecter à votre locataire Azure AD à l’aide de la cmdlet [Connect-AzureAD][Connect-AzureAD] :

```powershell
// Connect to your Azure AD tenant
Connect-AzureAD

// Retrieve the Azure AD DS resource.
$DomainServicesResource = Get-AzResource -ResourceType "Microsoft.AAD/DomainServices"

// Disable group-based scoped synchronization.
$disableScopedSync = @{"filteredSync" = "Disabled"}

// Update the Azure AD DS resource
Set-AzResource -Id $DomainServicesResource.ResourceId -Properties $disableScopedSync
```

La changement de l’étendue de la synchronisation conduit le domaine managé à resynchroniser toutes les données. Les objets qui ne sont plus nécessaires dans le domaine managé sont supprimés, et la resynchronisation peut prendre du temps.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur le processus de synchronisation, consultez [Comprendre la synchronisation dans Azure AD Domain Services](synchronization.md).

<!-- INTERNAL LINKS -->
[scoped-sync]: scoped-synchronization.md
[concepts-sync]: synchronization.md
[tutorial-create-instance]: tutorial-create-instance.md
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md

<!-- EXTERNAL LINKS -->
[Connect-AzureAD]: /powershell/module/azuread/connect-azuread
