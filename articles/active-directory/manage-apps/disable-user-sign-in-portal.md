---
title: Désactiver les connexions utilisateur pour une application d’entreprise dans Azure AD
description: Comment désactiver une application d’entreprise afin qu’aucun utilisateur ne puisse s’y connecter dans Azure Active Directory
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 04/12/2019
ms.author: kenwith
ms.reviewer: asteen
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0671d3dec963c0b475133881b00224cfe11e8370
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861642"
---
# <a name="disable-user-sign-ins-for-an-enterprise-app-in-azure-active-directory"></a>Désactiver les connexions utilisateur pour une application d’entreprise dans Azure Active Directory

Il est facile de désactiver une application d’entreprise afin qu’aucun utilisateur ne puisse s’y connecter dans Azure Active Directory (Azure AD). Vous devez disposer des autorisations appropriées pour gérer l’application d’entreprise. En outre, vous devez être administrateur général du répertoire.

## <a name="how-do-i-disable-user-sign-ins"></a>Comment désactiver les connexions des utilisateurs ?

1. Connectez-vous au [portail Azure](https://portal.azure.com) en utilisant un compte d’administrateur général pour le répertoire.
1. Sélectionnez **Tous les services**, entrez **Azure Active Directory** dans la zone de texte, puis sélectionnez **Entrée**.
1. Dans le volet **Azure Active Directory** -  **_NomRépertoire_ *_ (autrement dit, le volet Azure AD du répertoire que vous gérez), sélectionnez _* Applications d’entreprise**.
1. Dans le volet **Applications d’entreprise - Toutes les applications**, vous voyez une liste des applications que vous pouvez gérer. Sélectionnez une application.
1. Dans le volet **_NomApplication_ *_ (autrement dit, le volet intitulé avec le nom de l’application sélectionnée), sélectionnez _* Propriétés**.
1. Dans le volet **_NomApplication_ *_ - _* Propriétés**, sélectionnez **Non** pour **Activé pour que les utilisateurs se connectent ?** .
1. Sélectionnez la commande **Enregistrer** .

## <a name="use-azure-ad-powershell-to-disable-an-unlisted-app"></a>Utiliser Azure AD PowerShell pour désactiver une application non répertoriée

Si vous connaissez l’AppId d’une application qui n’apparaît pas dans la liste des applications d’entreprise (par exemple, parce que vous avez supprimé l’application ou que le principal du service n’a pas encore été créé, car l’application est en cours de préautorisation par Microsoft), vous pouvez créer manuellement le principal du service pour l’application, puis le désactiver à l’aide de la [cmdlet Azure AD PowerShell](/powershell/module/azuread/New-AzureADServicePrincipal).

```PowerShell
# The AppId of the app to be disabled
$appId = "{AppId}"

# Check if a service principal already exists for the app
$servicePrincipal = Get-AzureADServicePrincipal -Filter "appId eq '$appId'"
if ($servicePrincipal) {
    # Service principal exists already, disable it
    Set-AzureADServicePrincipal -ObjectId $servicePrincipal.ObjectId -AccountEnabled $false
} else {
    # Service principal does not yet exist, create it and disable it at the same time
    $servicePrincipal = New-AzureADServicePrincipal -AppId $appId -AccountEnabled $false
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Voir tous mes groupes](../fundamentals/active-directory-groups-view-azure-portal.md)
* [Affecter un utilisateur ou un groupe à une application d’entreprise](assign-user-or-group-access-portal.md)
* [Suppression d’une affectation d’utilisateur ou de groupe à partir d’une application d’entreprise](./assign-user-or-group-access-portal.md)
* [Modifier le nom ou le logo d’une application d’entreprise dans la version préliminaire d’Azure Active Directory](./add-application-portal-configure.md)
