---
title: Configurer des classifications d’autorisations avec Azure AD
description: Découvrez comment gérer les classifications d’autorisations déléguées.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 06/01/2020
ms.author: phsignor
ms.reviewer: arvindh, luleon, phsignor
ms.custom: contperf-fy21q2
ms.openlocfilehash: c1f46e3aed1309e51bf1ea8607a9db8a71550ce5
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033202"
---
# <a name="configure-permission-classifications"></a>Configurer des classifications d’autorisations

Les classifications d’autorisations vous permettent d’identifier l’impact des différentes autorisations en fonction des stratégies et des évaluations des risques de votre organisation. Par exemple, vous pouvez utiliser des classifications d’autorisations dans des stratégies de consentement afin d’identifier les autorisations que les utilisateurs peuvent accorder.

## <a name="manage-permission-classifications"></a>Gérer des classifications d’autorisations

Actuellement, seule la classification d’autorisation ayant un « Faible impact » est prise en charge. Seules des autorisations déléguées qui ne nécessitent pas de consentement de l’administrateur peuvent être classifiées comme ayant un « Faible impact ».

> [!TIP]
> Les autorisations minimales requises pour effectuer une connexion de base (`openid`, `profile`, `email`, `User.Read` et `offline_access`) sont toutes des autorisations déléguées sur Microsoft Graph. Avec ces autorisations, une application peut lire les détails complets du profil de l’utilisateur connecté, et maintenir cet accès même quand l’utilisateur n’utilise plus l’application.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour classifier les autorisations à l’aide du Portail Azure, procédez comme suit :

1. Se connecter au [Portail Azure](https://portal.azure.com) en tant qu’[administrateur général](../roles/permissions-reference.md#global-administrator--company-administrator), [administrateur d’application](../roles/permissions-reference.md#application-administrator) ou [administrateur d’application cloud](../roles/permissions-reference.md#cloud-application-administrator)
1. Sélectionnez **Azure Active Directory** > **Applications d’entreprise** > **Consentement et autorisations** > **Classifications d’autorisations**.
1. Choisissez **Ajouter des autorisations** pour classifier une autorisation autrement que comme ayant un « Faible impact ».
1. Sélectionnez l’API, puis les autorisations déléguées.

Dans cet exemple, nous avons classifié l’ensemble minimal d’autorisations requises pour l’authentification unique :

:::image type="content" source="media/configure-permission-classifications/permission-classifications.png" alt-text="Classifications d’autorisations":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Vous pouvez utiliser le dernier module Azure AD PowerShell en préversion, [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview), pour classifier les autorisations. Les classifications des autorisations sont configurées sur l’objet **ServicePrincipal** de l’API qui publie les autorisations.

#### <a name="list-the-current-permission-classifications-for-an-api"></a>Répertorier les classifications d’autorisations actuelles pour une API

1. Récupérez l’objet **ServicePrincipal** pour l’API. Ici, nous récupérons l’objet ServicePrincipal pour l’API Microsoft Graph :

   ```powershell
   $api = Get-AzureADServicePrincipal `
       -Filter "servicePrincipalNames/any(n:n eq 'https://graph.microsoft.com')"
   ```

1. Lisez les classifications d’autorisations déléguées pour l’API :

   ```powershell
   Get-AzureADMSServicePrincipalDelegatedPermissionClassification `
       -ServicePrincipalId $api.ObjectId | Format-Table Id, PermissionName, Classification
   ```

#### <a name="classify-a-permission-as-low-impact"></a>Classifier une autorisation comme ayant un « Faible impact »

1. Récupérez l’objet **ServicePrincipal** pour l’API. Ici, nous récupérons l’objet ServicePrincipal pour l’API Microsoft Graph :

   ```powershell
   $api = Get-AzureADServicePrincipal `
       -Filter "servicePrincipalNames/any(n:n eq 'https://graph.microsoft.com')"
   ```

1. Recherchez l’autorisation déléguée que vous souhaitez classifier :

   ```powershell
   $delegatedPermission = $api.OAuth2Permissions | Where-Object { $_.Value -eq "User.ReadBasic.All" }
   ```

1. Définissez la classification d’autorisation à l’aide du nom et de l’ID de l’autorisation :

   ```powershell
   Add-AzureADMSServicePrincipalDelegatedPermissionClassification `
      -ServicePrincipalId $api.ObjectId `
      -PermissionId $delegatedPermission.Id `
      -PermissionName $delegatedPermission.Value `
      -Classification "low"
   ```

#### <a name="remove-a-delegated-permission-classification"></a>Supprimer une classification d’autorisation déléguée

1. Récupérez l’objet **ServicePrincipal** pour l’API. Ici, nous récupérons l’objet ServicePrincipal pour l’API Microsoft Graph :

   ```powershell
   $api = Get-AzureADServicePrincipal `
       -Filter "servicePrincipalNames/any(n:n eq 'https://graph.microsoft.com')"
   ```

1. Recherchez la classification d’autorisation déléguée à supprimer :

   ```powershell
   $classifications = Get-AzureADMSServicePrincipalDelegatedPermissionClassification `
       -ServicePrincipalId $api.ObjectId
   $classificationToRemove = $classifications | Where-Object {$_.PermissionName -eq "User.ReadBasic.All"}
   ```

1. Supprimez la classification d’autorisation :

   ```powershell
   Remove-AzureADMSServicePrincipalDelegatedPermissionClassification `
       -ServicePrincipalId $api.ObjectId `
       -Id $classificationToRemove.Id
   ```

---

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus :

* [Configurer les paramètres de consentement de l’utilisateur](configure-user-consent.md)
* [Configurer le workflow du consentement administrateur](configure-admin-consent-workflow.md)
* [Découvrez comment gérer le consentement pour les applications et évaluer les demandes de consentement](manage-consent-requests.md)
* [Accorder le consentement de l’administrateur au niveau locataire à une application](grant-admin-consent.md)
* [Autorisations et consentement dans la plateforme d’identités Microsoft](../develop/v2-permissions-and-consent.md)

Pour obtenir de l’aide ou trouver des réponses à vos questions :
* [Azure AD sur StackOverflow](https://stackoverflow.com/questions/tagged/azure-active-directory)