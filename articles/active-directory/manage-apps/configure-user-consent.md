---
title: Configurer le consentement de l’utilisateur final pour une application à l’aide d’Azure AD
description: Apprenez à gérer comment et quand les utilisateurs peuvent donner leur consentement pour des applications qui auront accès aux données de votre organisation.
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
ms.openlocfilehash: c279b7f4ec2d70e7eb6a97abf02cfbf4f2ef87f0
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033610"
---
# <a name="configure-how-end-users-consent-to-applications"></a>Configurer le consentement de l’utilisateur final pour une application

Vous pouvez intégrer vos applications avec la plateforme d’identités Microsoft pour permettre à des utilisateurs de se connecter à l’aide de leur compte professionnel ou scolaire, et d’accéder aux données de votre organisation afin d’offrir des expériences riches pilotées par les données.

Pour qu’une application puisse accéder aux données de votre organisation, un utilisateur doit lui accorder les autorisations nécessaires. Les différentes autorisations ont trait à des niveaux d’accès différents. Par défaut, tous les utilisateurs peuvent accorder à des applications des autorisations qui ne nécessitent pas de consentement de l’administrateur. Par exemple, par défaut, un utilisateur peut autoriser une application à accéder à sa boîte aux lettres, mais ne peut pas l’autoriser à accéder en lecture et en écriture à tous les fichiers de votre organisation.

Si les utilisateurs peuvent autoriser des applications à accéder à des données, il peuvent facilement acquérir des applications utiles et être productifs. Toutefois, dans certains cas, une telle configuration peut constituer un risque si elle n’est pas analysée et contrôlée avec soin.

> [!IMPORTANT]
> Pour réduire le risque d’exposition à des applications malveillantes tentant d’amener les utilisateurs à leur accorder l’accès aux données de votre organisation, nous vous recommandons de permettre à l’utilisateur d’autoriser uniquement des applications publiées par un [serveur de publication vérifié](../develop/publisher-verification-overview.md).

## <a name="user-consent-settings"></a>Paramètres de consentement de l’utilisateur

Les stratégies de consentement d’application décrivent les conditions qui doivent être remplies pour qu’une application puisse faire l’objet d’un consentement. Ces stratégies peuvent inclure des conditions sur l’application qui demande l’accès, ainsi que les autorisations demandées par l’application.

En choisissant les stratégies de consentement de l’application qui s’appliquent à tous les utilisateurs, vous pouvez définir des limites lorsque les utilisateurs finaux sont autorisés à accorder leur consentement aux applications et quand ils devront demander la révision et l’approbation par l’administrateur :

* **Désactiver le consentement de l’utilisateur** : les utilisateurs ne peuvent pas accorder d’autorisations aux applications. Les utilisateurs peuvent continuer à se connecter aux applications qu’ils ont précédemment autorisées ou que des administrateurs ont autorisées en leur nom, mais ils ne sont pas autorisés à accorder de nouvelles autorisations ou à autoriser de nouvelles applications. Seuls les utilisateurs auxquels est dévolu un rôle d’annuaire comprenant l’autorisation de donner un consentement sont en mesure de donner leur consentement à de nouvelles applications.

* **Les utilisateurs peuvent accorder des autorisations à des applications d’éditeurs de votre organisation vérifiés, mais uniquement les autorisations que vous sélectionnez** : les utilisateurs ne peuvent accorder des autorisations qu’à des applications publiées par un [serveur de publication vérifié](../develop/publisher-verification-overview.md) et inscrites dans votre locataire. Les utilisateurs ne peuvent accorder que des autorisations classifiées comme ayant un « faible impact ». Vous devez [classifier les autorisations](configure-permission-classifications.md) pour sélectionner les autorisations que les utilisateurs peuvent accorder.

* **Les utilisateurs peuvent accorder des autorisations à toutes les applications** : cette option permet à tout utilisateur d’accorder à toute application toute autorisation ne nécessitant pas de consentement de l’administrateur.

* **Stratégie de consentement d’application personnalisée** : pour encore plus d’options sur les conditions qui régissent le consentement de l’utilisateur, vous pouvez [créer une stratégie de consentement d’application personnalisée](manage-app-consent-policies.md#create-a-custom-app-consent-policy) et la configurer pour qu’elle s’applique au consentement de l’utilisateur.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour configurer les paramètres de consentement de l’utilisateur via le portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’[Administrateur général](../roles/permissions-reference.md#global-administrator--company-administrator).
1. Sélectionnez **Azure Active Directory** > **Applications d’entreprise** > **Consentement et autorisations** > **Paramètres de consentement de l’utilisateur**.
1. Sous **Consentement de l’utilisateur pour les applications**, sélectionnez le paramètre de consentement que vous souhaitez configurer pour tous les utilisateurs.
1. Sélectionnez **Save** (Enregistrer) pour enregistrer vos paramètres.

:::image type="content" source="media/configure-user-consent/setting-for-all-users.png" alt-text="Paramètres de consentement de l’utilisateur":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Vous pouvez utiliser le dernier module Azure AD PowerShell en préversion, [AzureADPreview](/powershell/azure/active-directory/install-adv2?preserve-view=true&view=azureadps-2.0-preview), pour choisir la stratégie de consentement d’application régissant le consentement de l’utilisateur pour les applications.

#### <a name="disable-user-consent"></a>Désactiver le consentement de l’utilisateur

Pour désactiver le consentement de l’utilisateur, définissez les stratégies de consentement qui régissent le consentement de l’utilisateur de façon à ce qu’elles soient vides :

  ```powershell
  Set-AzureADMSAuthorizationPolicy `
     -Id "authorizationPolicy" `
     -PermissionGrantPolicyIdsAssignedToDefaultUserRole @()
  ```

#### <a name="allow-user-consent-subject-to-an-app-consent-policy"></a>Autoriser le consentement de l’utilisateur soumis à une stratégie de consentement d’application

Pour autoriser le consentement de l’utilisateur, choisissez la stratégie de consentement de l’application qui doit régir l’autorisation des utilisateurs pour accorder le consentement aux applications :

  ```powershell
  Set-AzureADMSAuthorizationPolicy `
     -Id "authorizationPolicy" `
     -PermissionGrantPolicyIdsAssignedToDefaultUserRole @("managePermissionGrantsForSelf.{consent-policy-id}")
  ```

Remplacez `{consent-policy-id}` par l’ID de la stratégie que vous souhaitez appliquer. Vous pouvez choisir une [stratégie de consentement d’application personnalisée](manage-app-consent-policies.md#create-a-custom-app-consent-policy) que vous avez créée ou vous pouvez choisir parmi les stratégies intégrées suivantes :

| id | Description |
|:---|:------------|
| microsoft-user-default-low | **Autoriser le consentement de l’utilisateur pour les applications provenant de serveurs de publication vérifiés, pour les autorisations sélectionnées**<br /> Autorisez un consentement de l’utilisateur limité uniquement pour les applications des serveurs de publication et des applications vérifiés et inscrits dans votre locataire, et uniquement pour les autorisations que vous classez comme « impact faible ». (N’oubliez pas de [classifier les autorisations](configure-permission-classifications.md) pour sélectionner les autorisations que les utilisateurs peuvent accorder.) |
| microsoft-user-default-legacy | **Autoriser le consentement de l’utilisateur pour les applications**<br /> Cette option permet à tout utilisateur d’accorder à toute application toute autorisation ne nécessitant pas de consentement de l’administrateur. |
  
Par exemple, pour activer le consentement de l’utilisateur pour l’objet de la stratégie intégrée `microsoft-user-default-low` :

```powershell
Set-AzureADMSAuthorizationPolicy `
   -Id "authorizationPolicy" `
   -PermissionGrantPolicyIdsAssignedToDefaultUserRole @("managePermissionGrantsForSelf.microsoft-user-default-low")
```

---

> [!TIP]
> [Activer le flux de travail de consentement administrateur](configure-admin-consent-workflow.md) pour autoriser les utilisateurs à demander à un administrateur la révision et l’approbation d’une application pour laquelle ils ne sont pas autorisés à donner leur consentement, par exemple, si le consentement de l’utilisateur a été désactivé ou si l’application demande des autorisations que l’utilisateur n’est pas autorisé à accorder.

## <a name="risk-based-step-up-consent"></a>Évolution du consentement en fonction des risques

La réaffectation du consentement en fonction des risques contribue à réduire l’exposition des utilisateurs aux applications malveillantes qui font des [demandes de consentement illicites](/microsoft-365/security/office-365-security/detect-and-remediate-illicit-consent-grants). Si Microsoft détecte une demande de consentement de l’utilisateur final à risque, la demande nécessitera une « évolution » vers le consentement de l’administrateur à la place. Cette fonctionnalité est activée par défaut, mais elle entraînera un changement de comportement uniquement lorsque le consentement de l’utilisateur final sera activé.

Quand une demande de consentement à risque est détectée, l’invite de consentement affiche un message indiquant que l’approbation de l’administrateur est requise. Si le [flux de travail de demande de consentement de l’administrateur](configure-admin-consent-workflow.md) est activé, l’utilisateur peut envoyer la demande à un administrateur pour révision ultérieure, directement à partir de l’invite de consentement. S’il n’est pas activé, le message suivant s’affiche :

* **AADSTS90094 :** &lt;clientAppDisplayName&gt; a besoin d’une autorisation pour accéder aux ressources de votre organisation, et cet accès ne peut être autorisé que par un administrateur. Veuillez demander à un administrateur d’accorder une autorisation pour cette application avant de pouvoir l’utiliser.

Dans ce cas, un événement d’audit est également journalisé avec la catégorie « ApplicationManagement », le type d’activité « Consent to application » et le motif de statut « Risky application detected ».

> [!IMPORTANT]
> Les administrateurs doivent [évaluer toutes les demandes de consentement](manage-consent-requests.md#evaluating-a-request-for-tenant-wide-admin-consent) soigneusement avant d’approuver une demande, en particulier quand Microsoft a détecté un risque.

### <a name="disable-or-re-enable-risk-based-step-up-consent-using-powershell"></a>Désactiver ou réactiver l’évolution du consentement en fonction des risques à l’aide de PowerShell

Vous pouvez utiliser le module Azure AD PowerShell en préversion, [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview), pour désactiver la réaffectation à l’administrateur du consentement requis quand Microsoft détecte un risque, ou pour la réactiver si elle a été précédemment désactivée.

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

    $riskBasedConsentEnabledValue = $settings.Values | ? { $_.Name -eq "BlockUserConsentForRiskyApps" }
    ```

1. Comprendre la valeur des paramètres :

    | Paramètre       | Type         | Description  |
    | ------------- | ------------ | ------------ |
    | _BlockUserConsentForRiskyApps_   | Boolean |  Indicateur qui spécifie si le consentement de l’utilisateur est bloqué lorsqu’une demande risquée est détectée. |

1. Mettez à jour la valeur des paramètres pour la configuration souhaitée :

    ```powershell
    # Disable risk-based step-up consent entirely
    $riskBasedConsentEnabledValue.Value = "False"
    ```

    ```powershell
    # Re-enable risk-based step-up consent, if disabled previously
    $riskBasedConsentEnabledValue.Value = "True"
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

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus :

* [Configurer les paramètres de consentement de l’utilisateur](configure-user-consent.md)
* [Gérer les stratégies de consentement des applications](manage-app-consent-policies.md)
* [Configurer le workflow du consentement administrateur](configure-admin-consent-workflow.md)
* [Découvrez comment gérer le consentement pour les applications et évaluer les demandes de consentement](manage-consent-requests.md)
* [Accorder le consentement de l’administrateur au niveau locataire à une application](grant-admin-consent.md)
* [Autorisations et consentement dans la plateforme d’identités Microsoft](../develop/v2-permissions-and-consent.md)

Pour obtenir de l’aide ou trouver des réponses à vos questions :
* [Azure AD sur StackOverflow](https://stackoverflow.com/questions/tagged/azure-active-directory)