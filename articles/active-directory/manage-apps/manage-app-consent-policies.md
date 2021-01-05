---
title: Gérer les stratégies de consentement des applications dans Azure AD
description: Apprenez à gérer les stratégies de consentement des applications intégrées et personnalisées pour contrôler le moment auquel le consentement peut être accordé.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 06/01/2020
ms.author: kenwith
ms.reviewer: arvindh, luleon, phsignor
ms.custom: contperf-fy21q2
ms.openlocfilehash: 3587beaeb788665c68c1fcb6376af40566b6c35e
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032777"
---
# <a name="manage-app-consent-policies"></a>Gérer les stratégies de consentement des applications

Avec Azure AD PowerShell, vous pouvez voir et gérer les stratégies de consentement des applications.

Une stratégie de consentement d’application se compose d’un ou plusieurs ensembles de conditions d’inclusion et de zéro ou plusieurs ensembles de conditions d’exclusion. Pour qu’un événement soit pris en compte dans une stratégie de consentement d’application, il doit correspondre à *au moins* un ensemble de conditions d’inclusion et ne doit pas correspondre à *un* ensemble de conditions d’exclusion.

Chaque ensemble de conditions est constitué de plusieurs conditions. Pour qu’un événement corresponde à un ensemble de conditions, *toutes* les conditions de l’ensemble de conditions doivent être remplies.

Les stratégies de consentement d’application dont l’ID commence par « Microsoft- » sont des stratégies intégrées. Certaines de ces stratégies intégrées sont utilisées dans les rôles d’annuaire intégrés existants. Par exemple, la stratégie de consentement d’application `microsoft-application-admin` décrit les conditions dans lesquelles les rôles administrateur d’application et administrateur d’application cloud sont autorisés à accorder le consentement administrateur au niveau du locataire. Les stratégies intégrées peuvent être utilisées dans des rôles d’annuaire personnalisés et pour configurer les paramètres de consentement de l’utilisateur, mais elles ne peuvent pas être modifiées ou supprimées.

## <a name="pre-requisites"></a>Conditions préalables

1. Vérifiez que vous utilisez le module [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview). Cette étape est importante si vous avez installé les modules [AzureAD](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0) et [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview)).

    ```powershell
    Remove-Module AzureAD -ErrorAction SilentlyContinue
    Import-Module AzureADPreview
    ```

1. Connectez-vous à Azure AD PowerShell.

   ```powershell
   Connect-AzureAD
   ```

## <a name="list-existing-app-consent-policies"></a>Liste des stratégies de consentement d’application

Il est judicieux de commencer par vous familiariser avec les stratégies de consentement d’application existantes dans votre organisation :

1. Listez des stratégies de consentement d’application :

   ```powershell
   Get-AzureADMSPermissionGrantPolicy | ft Id, DisplayName, Description
   ```

1. Affichez les ensembles de conditions « includes » d’une stratégie :

    ```powershell
    Get-AzureADMSPermissionGrantConditionSet -PolicyId "microsoft-application-admin" `
                                             -ConditionSetType "includes"
    ```

1. Affichez les ensembles de conditions « excludes » d’une stratégie :

    ```powershell
    Get-AzureADMSPermissionGrantConditionSet -PolicyId "microsoft-application-admin" `
                                             -ConditionSetType "excludes"
    ```

## <a name="create-a-custom-app-consent-policy"></a>Créer une stratégie de consentement d’application personnalisée

Pour créer une stratégie de consentement d’application personnalisée, procédez comme suit :

1. Créez une stratégie de consentement d’application personnalisée vide.

   ```powershell
   New-AzureADMSPermissionGrantPolicy `
       -Id "my-custom-policy" `
       -DisplayName "My first custom consent policy" `
       -Description "This is a sample custom app consent policy."
   ```

1. Ajoutez des ensembles de conditions « includes ».

   ```powershell
   # Include delegated permissions classified "low", for apps from verified publishers
   New-AzureADMSPermissionGrantConditionSet `
       -PolicyId "my-custom-policy" `
       -ConditionSetType "includes" `
       -PermissionType "delegated" `
       -PermissionClassification "low" `
       -ClientApplicationsFromVerifiedPublisherOnly $true
   ```

   Répétez cette étape pour ajouter d’autres ensembles de conditions « includes ».

1. Ajoutez si besoin des ensembles de conditions « excludes ».

   ```powershell
   # Retrieve the service principal for the Azure Management API
   $azureApi = Get-AzureADServicePrincipal -Filter "servicePrincipalNames/any(n:n eq 'https://management.azure.com/')"

   # Exclude delegated permissions for the Azure Management API
   New-AzureADMSPermissionGrantConditionSet `
       -PolicyId "my-custom-policy" `
       -ConditionSetType "excludes" `
       -PermissionType "delegated" `
       -ResourceApplication $azureApi.AppId
   ```

   Répétez cette étape pour ajouter d’autres ensembles de conditions « excludes ».

Une fois la stratégie de consentement d’application créée, vous pouvez [autoriser le consentement de l’utilisateur](configure-user-consent.md?tabs=azure-powershell#allow-user-consent-subject-to-an-app-consent-policy) soumis à cette stratégie.

## <a name="delete-a-custom-app-consent-policy"></a>Supprimer une stratégie de consentement d’application personnalisée

1. L’exemple suivant montre comment vous pouvez supprimer une stratégie de consentement d’application personnalisée. **Il est impossible d’annuler cette opération.**

   ```powershell
   Remove-AzureADMSPermissionGrantPolicy -Id "my-custom-policy"
   ```

> [!WARNING]
> Les stratégies de consentement d’application supprimées ne peuvent pas être restaurées. Si vous supprimez accidentellement une stratégie de consentement d’application personnalisée, vous devez recréer cette stratégie.

---

### <a name="supported-conditions"></a>Conditions prises en charge

Le tableau suivant fournit la liste des conditions prises en charge pour les stratégies de consentement des applications.

| Condition | Description|
|:---------------|:----------|
| PermissionClassification | La [classification des autorisations](configure-permission-classifications.md) pour l’autorisation accordée, ou « All » pour correspondre à toute classification d’autorisation (y compris les autorisations qui ne sont pas classées). La valeur par défaut est « All ». |
| PermissionType | Type d’autorisation de l’autorisation accordée. Utilisez « application » pour les autorisations de l’application (par exemple, les rôles d’application) ou « delegated » pour les autorisations déléguées. <br><br>**Remarque** : La valeur « delegatedUserConsentable » indique des autorisations déléguées qui n’ont pas été configurées par le serveur de publication de l’API pour exiger le consentement de l’administrateur. Cette valeur peut être utilisée dans les stratégies d’autorisations intégrées, mais ne peut pas être utilisée dans les stratégies d’autorisation personnalisées. Obligatoire. |
| ResourceApplication | **AppId** de l’application de ressource (par exemple, l’API) pour laquelle une autorisation est accordée, ou « any » pour correspondre à n’importe quelle application ou API de ressources. La valeur par défaut est « any ». |
| Autorisations | Liste des ID d’autorisation pour les autorisations spécifiques de correspondance ou une liste avec la valeur unique « all » pour toutes les autorisations. La valeur par défaut est la seule valeur « tout ». <ul><li>Les ID d’autorisations déléguées se trouvent dans la propriété **OAuth2Permissions** de l’objet ServicePrincipal de l’API.</li><li>Les ID d’autorisations d’application se trouvent dans la propriété **AppRoles** de l’objet ServicePrincipal de l’API.</li></ol> |
| ClientApplicationIds | Liste de valeurs **AppId** auxquelles les applications clientes doivent correspondre ou liste avec la valeur unique « all » pour correspondre à n’importe quelle application cliente. La valeur par défaut est la seule valeur « tout ». |
| ClientApplicationTenantIds | Liste d’ID de locataire Azure Active Directory dans lesquels l’application cliente est inscrite ou liste avec la valeur unique « all » pour qu’elle corresponde aux applications clientes inscrites dans n’importe quel locataire. La valeur par défaut est la seule valeur « tout ». |
| ClientApplicationPublisherIds | Liste d’ID de partenaires Microsoft Partner Network (MPN) pour les [serveurs de publication vérifiés](../develop/publisher-verification-overview.md) de l’application cliente, ou liste avec la valeur unique « all » pour qu’elle corresponde aux applications clientes de n’importe quel serveur de publication. La valeur par défaut est la seule valeur « tout ». |
| ClientApplicationsFromVerifiedPublisherOnly | Affectez la valeur `$true` pour qu’elle corresponde uniquement aux applications clientes disposant d’un [serveur de publication vérifié](../develop/publisher-verification-overview.md). Affectez la valeur `$false` pour qu’elle corresponde à toute application cliente, même si elle n’a pas de serveur de publication vérifié. La valeur par défaut est `$false`. |

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus :

* [Configurer les paramètres de consentement de l’utilisateur](configure-user-consent.md)
* [Configurer le workflow du consentement administrateur](configure-admin-consent-workflow.md)
* [Découvrez comment gérer le consentement pour les applications et évaluer les demandes de consentement](manage-consent-requests.md)
* [Accorder le consentement de l’administrateur au niveau locataire à une application](grant-admin-consent.md)
* [Autorisations et consentement dans la plateforme d’identités Microsoft](../develop/v2-permissions-and-consent.md)

Pour obtenir de l’aide ou trouver des réponses à vos questions :
* [Azure AD sur StackOverflow](https://stackoverflow.com/questions/tagged/azure-active-directory)