---
title: Définir les durées de vie de jetons
titleSuffix: Microsoft identity platform
description: Découvrez comment définir des durées de vie pour des jetons émis par la Plateforme d’identité Microsoft. Apprenez à gérer la stratégie par défaut d’une organisation, à créer une stratégie de connexion web, à créer une stratégie pour une application native qui appelle une API web et à gérer une stratégie avancée.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 12/14/2020
ms.author: ryanwi
ms.custom: aaddev, content-perf, FY21Q1
ms.reviewer: hirsin, jlu, annaba
ms.openlocfilehash: e663cdd3846e804d1dcf96076c07b9a3db84272c
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507742"
---
# <a name="configure-token-lifetime-policies-preview"></a>Configurer des stratégies de durée de vie des jetons (préversion)
De nombreux scénarios sont possibles dans Azure AD lorsque vous créez et gérez les durées de vie des jetons pour les applications, les principaux du service et votre organisation globale.  

> [!IMPORTANT]
> Après mai 2020, les locataires ne seront plus en mesure de configurer la durée de vie des jetons d’actualisation et de session.  Azure Active Directory cessera d’honorer la configuration actuelle des jetons d’actualisation et de session dans les stratégies après le 30 janvier 2021. Vous pourrez toujours configurer la durée de vie des jetons d'accès après la dépréciation.  Pour en savoir plus, consultez [Durées de vie des jetons configurables dans la plateforme d’identités Microsoft](active-directory-configurable-token-lifetimes.md).
> Nous avons implémenté les  [capacités de gestion des sessions d’authentification](../conditional-access/howto-conditional-access-session-lifetime.md)  dans l’accès conditionnel Azure AD. Vous pouvez utiliser cette nouvelle fonctionnalité pour configurer les durées de vie des jetons d’actualisation en définissant la fréquence de connexion.


Dans cette section, nous allons vous guider dans quelques scénarios courants de stratégie qui peuvent vous aider à imposer de nouvelles règles pour les éléments suivants :

* Durée de vie des jetons
* Délai d’inactivité maximale des jetons
* Âge maximal des jetons

Dans les exemples, vous pouvez apprendre à :

* Gérer la stratégie par défaut d’une organisation
* Créer une stratégie de connexion web
* Créer une stratégie pour une application native qui appelle une API web
* Gérer une stratégie avancée

## <a name="prerequisites"></a>Prérequis
Dans les exemples suivants, vous allez créer, mettre à jour, lier et supprimer des stratégies pour les applications, les principaux de service et votre organisation globale. Si vous débutez avec Azure AD, nous vous recommandons de vous documenter sur [l’obtention d’un client Azure Active Directory](quickstart-create-new-tenant.md) avant de continuer avec ces exemples.  

Pour commencer, suivez les étapes ci-dessous :

1. Téléchargez la dernière [version préliminaire publique du module Azure AD PowerShell](https://www.powershellgallery.com/packages/AzureADPreview).
2. Exécutez la commande `Connect` pour vous connecter à votre compte Administrateur Azure AD. Exécutez cette commande chaque fois que vous démarrez une nouvelle session.

    ```powershell
    Connect-AzureAD -Confirm
    ```

3. Pour afficher toutes les stratégies qui ont été créées dans votre organisation, exécutez la commande suivante. Exécutez cette commande après la plupart des opérations dans les scénarios suivants. L’exécution de la commande vous aide également à obtenir le ** ** de vos stratégies.

    ```powershell
    Get-AzureADPolicy
    ```

## <a name="manage-an-organizations-default-policy"></a>Gérer la stratégie par défaut d’une organisation
Dans cet exemple, vous allez créer une stratégie qui permet à vos utilisateurs de se connecter moins fréquemment dans votre organisation entière. Pour ce faire, créez une stratégie de durée de vie des jetons pour les jetons d’actualisation à facteur unique, qui est appliquée dans toute votre organisation. Cette stratégie est appliquée à toutes les applications de votre organisation et à chaque principal de service pour lequel aucune stratégie n’est déjà définie.

1. Créez une stratégie de durée de vie des jetons.

    1. Définissez le jeton d’actualisation à facteur unique sur « Jusqu’à révocation ». Le jeton n’expire pas tant que l’accès n’est pas révoqué. Créez la définition de stratégie suivante :

        ```powershell
        @('{
            "TokenLifetimePolicy":
            {
                "Version":1,
                "MaxAgeSingleFactor":"until-revoked"
            }
        }')
        ```

    1. Pour créer la stratégie, exécutez la cmdlet [New-AzureADPolicy](/powershell/module/azuread/new-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) :

        ```powershell
        $policy = New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1, "MaxAgeSingleFactor":"until-revoked"}}') -DisplayName "OrganizationDefaultPolicyScenario" -IsOrganizationDefault $true -Type "TokenLifetimePolicy"
        ```

    1. Pour supprimer un espace blanc, exécutez la cmdlet [Get-AzureADPolicy](/powershell/module/azuread/get-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) :

        ```powershell
        Get-AzureADPolicy -id | set-azureadpolicy -Definition @($((Get-AzureADPolicy -id ).Replace(" ","")))
        ```

    1. Pour afficher votre nouvelle stratégie et obtenir son **ID d’objet**, exécutez la commande ci-après :

        ```powershell
        Get-AzureADPolicy -Id $policy.Id
        ```

1. Mettez à jour la stratégie.

    Vous pouvez décider que la première stratégie que vous définissez dans cet exemple n’est pas aussi stricte que ce que votre service requiert. Pour définir votre jeton d’actualisation à facteur unique de façon à ce qu’il expire dans deux jours, exécutez la commande suivante :

    ```powershell
    Set-AzureADPolicy -Id $policy.Id -DisplayName $policy.DisplayName -Definition @('{"TokenLifetimePolicy":{"Version":1,"MaxAgeSingleFactor":"2.00:00:00"}}')
    ```

## <a name="create-a-policy-for-web-sign-in"></a>Créer une stratégie de connexion web

Dans cet exemple, vous créez une stratégie qui nécessite que vos utilisateurs s’authentifient plus fréquemment dans votre application web. Cette stratégie définit la durée de vie des jetons d’accès/ID et l’âge maximal d’un jeton de session multifacteur pour le principal de service de votre application web.

1. Créez une stratégie de durée de vie des jetons.

    Cette stratégie de connexion web définit la durée de vie des jetons d’accès/ID et l’âge maximal de jeton de session à facteur unique sur 2 heures.

    1. Pour créer la stratégie, exécutez la cmdlet [New-AzureADPolicy](/powershell/module/azuread/new-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) :

        ```powershell
        $policy = New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1,"AccessTokenLifetime":"02:00:00","MaxAgeSessionSingleFactor":"02:00:00"}}') -DisplayName "WebPolicyScenario" -IsOrganizationDefault $false -Type "TokenLifetimePolicy"
        ```

    1. Pour voir votre nouvelle stratégie et obtenir son **ObjectId**, exécutez la cmdlet [Get-AzureADPolicy](/powershell/module/azuread/get-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) :

        ```powershell
        Get-AzureADPolicy -Id $policy.Id
        ```

1. Affectez la stratégie au principal de service. Vous devez également obtenir **l’ID d’objet** de votre principal de service.

    1. Utilisez la cmdlet [Get-AzureADServicePrincipal](/powershell/module/azuread/get-azureadserviceprincipal) pour afficher tous les principaux de service de votre organisation ou un seul principal de service.
        ```powershell
        # Get ID of the service principal
        $sp = Get-AzureADServicePrincipal -Filter "DisplayName eq '<service principal display name>'"
        ```

    1. Lorsque vous avez le principal du service, exécutez la cmdlet [Add-AzureADServicePrincipalPolicy](/powershell/module/azuread/add-azureadserviceprincipalpolicy?view=azureadps-2.0-preview&preserve-view=true) :
        ```powershell
        # Assign policy to a service principal
        Add-AzureADServicePrincipalPolicy -Id $sp.ObjectId -RefObjectId $policy.Id
        ```

## <a name="create-a-policy-for-a-native-app-that-calls-a-web-api"></a>Créer une stratégie pour une application native qui appelle une API web
Dans cet exemple, vous créez une stratégie qui nécessite que vos utilisateurs s’authentifient moins fréquemment. La stratégie augmente également la durée pendant laquelle un utilisateur peut être inactif avant d’avoir à s’authentifier de nouveau. La stratégie est appliquée à l’API web. Lorsque l’application native demande l’API web en tant que ressource, cette stratégie est appliquée.

1. Créez une stratégie de durée de vie des jetons.

    1. Pour créer une stratégie stricte pour une API web, exécutez la cmdlet [New-AzureADPolicy](/powershell/module/azuread/new-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) :

        ```powershell
        $policy = New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1,"MaxInactiveTime":"30.00:00:00","MaxAgeMultiFactor":"until-revoked","MaxAgeSingleFactor":"180.00:00:00"}}') -DisplayName "WebApiDefaultPolicyScenario" -IsOrganizationDefault $false -Type "TokenLifetimePolicy"
        ```

    1. Pour afficher votre nouvelle stratégie, exécutez la commande suivante :

        ```powershell
        Get-AzureADPolicy -Id $policy.Id
        ```

1. Affectez la stratégie à votre API web. Vous devez également obtenir **l’ID d’objet** de votre application. Utilisez la cmdlet [Get-AzureADApplication](/powershell/module/azuread/get-azureadapplication) pour trouver l’**ObjectId** de votre application, ou utilisez le [portail Azure](https://portal.azure.com/).

    Obtenez l’**ObjectId** de votre application et assignez la stratégie :

    ```powershell
    # Get the application
    $app = Get-AzureADApplication -Filter "DisplayName eq 'Fourth Coffee Web API'"

    # Assign the policy to your web API.
    Add-AzureADApplicationPolicy -Id $app.ObjectId -RefObjectId $policy.Id
    ```

## <a name="manage-an-advanced-policy"></a>Gérer une stratégie avancée
Dans cet exemple, vous créez quelques stratégies, pour savoir comment fonctionne le système de priorité. Vous apprenez également à gérer plusieurs stratégies qui sont appliquées à divers objets.

1. Créez une stratégie de durée de vie des jetons.

    1. Pour créer une stratégie par défaut d’organisation qui définit la durée de vie des jetons d’actualisation à facteur unique sur 30 jours, exécutez la cmdlet [New-AzureADPolicy](/powershell/module/azuread/new-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) :

        ```powershell
        $policy = New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1,"MaxAgeSingleFactor":"30.00:00:00"}}') -DisplayName "ComplexPolicyScenario" -IsOrganizationDefault $true -Type "TokenLifetimePolicy"
        ```

    1. Pour voir votre nouvelle stratégie, exécutez la cmdlet [Get-AzureADPolicy](/powershell/module/azuread/get-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) :

        ```powershell
        Get-AzureADPolicy -Id $policy.Id
        ```

1. Affectez la stratégie à un principal de service.

    À présent, vous avez une stratégie qui s’applique à toute l’organisation. Vous souhaitez peut-être conserver cette stratégie de 30 jours pour un principal de service spécifique, mais changer la stratégie par défaut d’organisation pour qu’elle soit la limite supérieure du paramètre « Jusqu’à révocation ».

    1. Pour afficher tous les principaux de service de votre organisation, utilisez la cmdlet [Get-AzureADServicePrincipal](/powershell/module/azuread/get-azureadserviceprincipal).

    1. Lorsque vous avez le principal du service, exécutez la cmdlet [Add-AzureADServicePrincipalPolicy](/powershell/module/azuread/add-azureadserviceprincipalpolicy?view=azureadps-2.0-preview&preserve-view=true) :

        ```powershell
        # Get ID of the service principal
        $sp = Get-AzureADServicePrincipal -Filter "DisplayName eq '<service principal display name>'"

        # Assign policy to a service principal
        Add-AzureADServicePrincipalPolicy -Id $sp.ObjectId -RefObjectId $policy.Id
        ```

1. Définissez l’indicateur `IsOrganizationDefault` sur false :

    ```powershell
    Set-AzureADPolicy -Id $policy.Id -DisplayName "ComplexPolicyScenario" -IsOrganizationDefault $false
    ```

1. Créez une stratégie par défaut d’organisation :

    ```powershell
    New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1,"MaxAgeSingleFactor":"until-revoked"}}') -DisplayName "ComplexPolicyScenarioTwo" -IsOrganizationDefault $true -Type "TokenLifetimePolicy"
    ```

    À présent, la stratégie d’origine est liée à votre principal de service, et la nouvelle stratégie est définie comme stratégie par défaut de votre organisation. Il est important de se rappeler que les stratégies appliquées aux principaux de service ont priorité sur les stratégies par défaut d’organisation.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur les [capacités de gestion des sessions d’authentification](../conditional-access/howto-conditional-access-session-lifetime.md) dans l’accès conditionnel Azure AD.