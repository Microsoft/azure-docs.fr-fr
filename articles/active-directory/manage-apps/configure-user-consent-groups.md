---
title: Configurer le consentement du propriétaire du groupe pour des applications qui accèdent aux données de groupe à l’aide d’Azure AD
description: Apprenez à gérer si les propriétaires de groupe et d’équipe peuvent donner leur consentement aux applications qui auront accès aux données du groupe ou de l’équipe.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 05/19/2020
ms.author: kenwith
ms.reviewer: arvindh, luleon, phsignor
ms.custom: contperf-fy21q2
ms.openlocfilehash: d14e4808d5bec03dff9406f37e63ee4c76e601f8
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033219"
---
# <a name="configure-group-owner-consent-to-apps-accessing-group-data"></a>Configurer le consentement du propriétaire du groupe pour des applications qui accèdent aux données de groupe

Les propriétaires de groupes et d’équipes peuvent autoriser des applications telles que des applications publiées par des fournisseurs tiers à accéder aux données de votre organisation associées à un groupe. Par exemple, un propriétaire d’équipe dans Microsoft Teams peut autoriser une application à lire tous les messages Teams de l’équipe ou à consulter le profil de base des membres d’un groupe. Pour en savoir plus, consultez [Consentement propre à la ressource dans Microsoft Teams](/microsoftteams/resource-specific-consent).

## <a name="manage-group-owner-consent-to-apps"></a>Gérer le consentement du propriétaire du groupe aux applications

Vous pouvez configurer quels utilisateurs peuvent autoriser des applications à accéder aux données de leurs groupes ou équipes ou vous pouvez désactiver ces applications pour tous les utilisateurs.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Suivez ces étapes pour gérer le consentement du propriétaire du groupe pour des applications qui accèdent aux données de groupe :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’[Administrateur général](../roles/permissions-reference.md#global-administrator--company-administrator).
2. Sélectionnez **Azure Active Directory** > **Applications d’entreprise** > **Consentement et autorisations** > **Paramètres de consentement de l’utilisateur**.
3. Sous **Consentement du propriétaire du groupe pour les applications qui accèdent aux données**, sélectionnez l’option que vous souhaitez activer.
4. Sélectionnez **Save** (Enregistrer) pour enregistrer vos paramètres.

Dans cet exemple, tous les propriétaires de groupes peuvent autoriser des applications qui accèdent aux données de leurs groupes :

:::image type="content" source="media/configure-user-consent-groups/group-owner-consent.png" alt-text="Paramètres de consentement du propriétaire du groupe":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Vous pouvez utiliser le Module Azure AD PowerShell Préversion, [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview), pour activer ou désactiver la capacité des propriétaires de groupes à autoriser des applications qui accèdent aux données de votre organisation pour les groupes qui leur appartiennent.

1. Vérifiez que vous utilisez le module [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview). Cette étape est importante si vous avez installé les modules [AzureAD](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0) et [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview)).

    ```powershell
    Remove-Module AzureAD
    Import-Module AzureADPreview
    ```

1. Connectez-vous à Azure AD PowerShell.

   ```powershell
   Connect-AzureAD
   ```

1. Récupérez la valeur actuelle des paramètres d’annuaire **Consent Policy Settings** (Paramètres de stratégie de consentement) dans votre locataire. Pour cela, vous devez vérifier si les paramètres d’annuaire de cette fonctionnalité ont été créés et, si ce n’est pas le cas, utiliser les valeurs du Modèle de paramètres d’annuaire correspondant.

    ```powershell
    $consentSettingsTemplateId = "dffd5d46-495d-40a9-8e21-954ff55e198a" # Consent Policy Settings
    $settings = Get-AzureADDirectorySetting -All $true | Where-Object { $_.TemplateId -eq $consentSettingsTemplateId }

    if (-not $settings) {
        $template = Get-AzureADDirectorySettingTemplate -Id $consentSettingsTemplateId
        $settings = $template.CreateDirectorySetting()
    }

    $enabledValue = $settings.Values | ? { $_.Name -eq "EnableGroupSpecificConsent" }
    $limitedToValue = $settings.Values | ? { $_.Name -eq "ConstrainGroupSpecificConsentToMembersOfGroupId" }
    ```

1. Prenez connaissance des valeurs des paramètres. Deux valeurs de paramètres définissent les utilisateurs qui seraient en mesure d’autoriser une application à accéder aux données de leur groupe :

    | Paramètre       | Type         | Description  |
    | ------------- | ------------ | ------------ |
    | _EnableGroupSpecificConsent_   | Boolean | Indicateur spécifiant si les propriétaires de groupes sont autorisés à accorder des autorisations spécifiques de groupe. |
    | _ConstrainGroupSpecificConsentToMembersOfGroupId_ | Guid | Si _EnableGroupSpecificConsent_ a la valeur « True » et que cette valeur est définie sur l’ID d’objet d’un groupe, les membres du groupe identifié sont autorisés à accorder des autorisations spécifiques de groupes aux groupes dont ils sont propriétaires. |

1. Mettez à jour les valeurs des paramètres pour la configuration souhaitée :

    ```powershell
    # Disable group-specific consent entirely
    $enabledValue.Value = "False"
    $limitedToValue.Value = ""
    ```

    ```powershell
    # Enable group-specific consent for all users
    $enabledValue.Value = "True"
    $limitedToValue.Value = ""
    ```

    ```powershell
    # Enable group-specific consent for users in a given group
    $enabledValue.Value = "True"
    $limitedToValue.Value = "{group-object-id}"
    ```

1. Enregistrez vos paramètres.

    ```powershell
    if ($settings.Id) {
        # Update an existing directory settings
        Set-AzureADDirectorySetting -Id $settings.Id -DirectorySetting $settings
    } else {
        # Create a new directory settings to override the default setting 
        New-AzureADDirectorySetting -DirectorySetting $settings
    }
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