---
title: Bonnes pratiques pour la gestion des erreurs liées aux applications clientes ADAL | Azure
description: Fournit des conseils et les meilleures pratiques de gestion des erreurs pour les applications clientes ADAL.
services: active-directory
author: rwike77
manager: CelesteDG
ms.author: ryanwi
ms.service: active-directory
ms.subservice: azuread-dev
ms.custom: aaddev
ms.topic: how-to
ms.workload: identity
ms.date: 02/27/2017
ROBOTS: NOINDEX
ms.openlocfilehash: ad5595f7eebc8feca2f00a6f95e10c547ded9529
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85383732"
---
# <a name="error-handling-best-practices-for-azure-active-directory-authentication-library-adal-clients"></a>Meilleures pratiques de gestion des erreurs pour les clients Azure Active Directory Authentication Library (ADAL)

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Cet article fournit des conseils sur le type d’erreurs que les développeurs peuvent rencontrer durant l’utilisation d’ADAL pour authentifier les utilisateurs. Durant son utilisation, un développeur peut être amené à gérer les erreurs dans différents cas. Une gestion correcte des erreurs garantit une expérience utilisateur optimale et limite le nombre de fois où l’utilisateur final doit se connecter.

Dans cet article, nous passons en revue les cas spécifiques pour chaque plateforme prise en charge par ADAL et indiquons comment votre application peut gérer correctement chaque cas. Les conseils relatifs aux erreurs ont été divisés en deux catégories principales, en fonction des modèles d’acquisition de jetons fournis par les API d’ADAL :

- **AcquireTokenSilent** : Le client tente d’obtenir un jeton en mode silencieux (aucune interface utilisateur) et risque de ne pas l’obtenir si ADAL échoue. 
- **AcquireToken** : Le client peut tenter une acquisition en mode silencieux, mais il peut également effectuer des requêtes interactives nécessitant une connexion.

> [!TIP]
> Il est judicieux de journaliser toutes les erreurs et exceptions durant l’utilisation d’ADAL et d’Azure AD. Les journaux d’activité ne servent pas uniquement à comprendre l’intégrité globale de votre application. Ils sont également importants pour le débogage de problèmes plus larges. Bien que votre application puisse se récupérer de certaines erreurs, ces dernières peuvent cacher des problèmes de conception plus larges qui nécessitent des modifications de code pour être résolus. 
> 
> Durant l’implémentation des conditions d’erreur traitées dans ce document, pensez à journaliser le code d’erreur et la description pour les raisons indiquées précédemment. Pour obtenir des exemples de journalisation de code, consultez [Référence d’erreurs et de journalisation](#error-and-logging-reference). 
>

## <a name="acquiretokensilent"></a>AcquireTokenSilent

AcquireTokenSilent tente d’obtenir un jeton sans que l’utilisateur final ne voie d’interface utilisateur (IU). Il existe plusieurs cas où l’acquisition en mode silencieux risque d’échouer et doit être gérée par le biais de requêtes interactives ou par un gestionnaire par défaut. Examinons quand et comment utiliser chacun des cas dans les sections suivantes.

Le système d’exploitation génère un ensemble d’erreurs pouvant nécessiter une gestion propre à l’application. Pour plus d’informations, consultez la section « Erreurs du système d’exploitation » dans [Référence d’erreurs et de journalisation](#error-and-logging-reference). 

### <a name="application-scenarios"></a>Scénarios d’application

- Applications [clientes natives](../develop/developer-glossary.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json#native-client) (iOS, Android, .NET Desktop ou Xamarin)
- Applications [clientes web](../develop/developer-glossary.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json#web-client) appelant une [ressource](../develop/developer-glossary.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json#resource-server) (.NET)

### <a name="error-cases-and-actionable-steps"></a>Cas d’erreur et étapes possibles

Il existe essentiellement deux cas d’erreurs AcquireTokenSilent :

| Cas | Description |
|------|-------------|
| **Cas n° 1** : L’erreur peut être résolue via une connexion interactive | Pour les erreurs dues à un manque de jetons valides, une requête interactive est nécessaire. Plus spécifiquement, une recherche de cache et un jeton d’actualisation non valide/expiré nécessitent un appel AcquireToken pour résoudre l’erreur.<br><br>Dans ce cas, l’utilisateur final doit être invité à se connecter. L’application peut choisir d’effectuer une requête interactive immédiatement, après l’interaction de l’utilisateur final (par exemple, un clic sur un bouton de connexion) ou ultérieurement. Le choix dépend du comportement souhaité de l’application.<br><br>Consultez le code dans la section suivante pour ce cas particulier, ainsi que les erreurs qui le caractérisent.|
| **Cas n° 2** : L’erreur ne peut pas être résolue via une connexion interactive | Pour les erreurs passagères/temporaires, les problèmes réseau et autres défaillances, une requête interactive AcquireToken ne permet pas de résoudre le problème. Les invites de connexion interactive inutiles peuvent également frustrer les utilisateurs finaux. ADAL effectue automatiquement une seule nouvelle tentative pour la plupart des erreurs relatives à des échecs AcquireTokenSilent.<br><br>L’application cliente peut également effectuer une nouvelle tentative ultérieurement, mais le moment et la façon de le faire dépendent du comportement de l’application et de l’expérience utilisateur souhaitée. Par exemple, l’application peut effectuer une nouvelle tentative AcquireTokenSilent après quelques minutes ou en réponse à une action de l’utilisateur final. Une nouvelle tentative immédiate limite l’application et n’est pas recommandée.<br><br>Une nouvelle tentative ultérieure qui échoue avec la même erreur ne signifie pas que le client doive effectuer une requête interactive avec AcquireToken, étant donné qu’elle ne résout pas l’erreur.<br><br>Consultez le code dans la section suivante pour ce cas particulier, ainsi que les erreurs qui le caractérisent. |

### <a name="net"></a>.NET

Les conseils suivants incluent des exemples de gestion des erreurs menée avec les méthodes ADAL suivantes : 

- acquireTokenSilentAsync(…)
- acquireTokenSilentSync(…) 
- [Déconseillé] acquireTokenSilent(…)
- [Déconseillé] acquireTokenByRefreshToken(…) 

Votre code serait implémenté comme suit :

```csharp
try{
    AcquireTokenSilentAsync(…);
} 

catch (AdalSilentTokenAcquisitionException e) {
    // Exception: AdalSilentTokenAcquisitionException
    // Caused when there are no tokens in the cache or a required refresh failed. 

    // Action: Case 1, resolvable with an interactive request. 
} 

catch(AdalServiceException e) {
    // Exception: AdalServiceException 
    // Represents an error produced by the STS. 
    // e.ErrorCode contains the error code and description, which can be used for debugging. 
    // NOTE: Do not code a dependency on the contents of the error description, as it can change over time.

    // Action: Case 2, not resolvable with an interactive request.
    // Attempt retry after a timed interval or user action.
} 
    
catch (AdalException e) {
    // Exception: AdalException 
    // Represents a library exception generated by ADAL .NET. 
    // e.ErrorCode contains the error code. 

    // Action: Case 2, not resolvable with an interactive request.
    // Attempt retry after a timed interval or user action.
    // Example Error: network_not_available, default case.
}
```

### <a name="android"></a>Android

Les conseils suivants incluent des exemples de gestion des erreurs menée avec les méthodes ADAL suivantes : 

- acquireTokenSilentSync(…)
- acquireTokenSilentAsync(...)
- [Déconseillé] acquireTokenSilent(…)

Votre code serait implémenté comme suit :

```java
// *Inside callback*
public void onError(Exception e) {

    if (e instanceof AuthenticationException) {
        // Exception: AdalException
        // Represents a library exception generated by ADAL Android.
        // Error Code: e.getCode().

        // Errors: ADALError.ERROR_SILENT_REQUEST,
        // ADALError.AUTH_REFRESH_FAILED_PROMPT_NOT_ALLOWED,
        // ADALError.INVALID_TOKEN_CACHE_ITEM
        // Description: Request failed due to no tokens in
        // cache or failed a required refresh. 

        // Action: Case 1, resolvable with an interactive request. 

        // Action: Case 2, not resolvable with an interactive request.
        // Attempt retry after a timed interval or user action.
        // Example Errors: default case,
        // DEVICE_CONNECTION_IS_NOT_AVAILABLE, 
        // BROKER_AUTHENTICATOR_ERROR_GETAUTHTOKEN,
    }
}
```

### <a name="ios"></a>iOS

Les conseils suivants incluent des exemples de gestion des erreurs menée avec les méthodes ADAL suivantes : 

- acquireTokenSilentWithResource(…)

Votre code serait implémenté comme suit :

```objc
[context acquireTokenSilentWithResource:[ARGS], completionBlock:^(ADAuthenticationResult *result) {
    if (result.status == AD_FAILED) {
        if ([error.domain isEqualToString:ADAuthenticationErrorDomain]){
            // Exception: AD_FAILED 
            // Represents a library error generated by ADAL Objective-C.
            // Error Code: result.error.code

            // Errors: AD_ERROR_SERVER_REFRESH_TOKEN_REJECTED, AD_ERROR_CACHE_NO_REFRESH_TOKEN
            // Description: No tokens in cache or failed a required token refresh failed. 
            // Action: Case 1, resolvable with an interactive request. 

            // Error: AD_ERROR_CACHE_MULTIPLE_USERS
            // Description: There was ambiguity in the silent request resulting in multiple cache items.
            // Action: Special Case, application should perform another silent request and specify the user using ADUserIdentifier. 
            // Can be caused in cases of a multi-user application. 

            // Action: Case 2, not resolvable with an interactive request.
            // Attempt retry after some time or user action.
            // Example Errors: default case,
            // AD_ERROR_CACHE_BAD_FORMAT
        }
    }
}]
```

## <a name="acquiretoken"></a>AcquireToken

AcquireToken est la méthode ADAL par défaut utilisée pour obtenir des jetons. Dans les cas où l’identité de l’utilisateur est requise, AcquireToken tente d’abord d’obtenir un jeton en mode silencieux, puis affiche l’interface utilisateur si nécessaire (sauf si PromptBehavior.Never est transmis). Dans les cas où l’identité de l’application est requise, AcquireToken tente d’obtenir un jeton, mais n’affiche pas l’interface utilisateur, car il n’existe aucun utilisateur final. 

Durant la gestion des erreurs AcquireToken, celle-ci dépend de la plateforme et du scénario que l’application tente d’obtenir. 

Le système d’exploitation peut également générer un ensemble d’erreurs nécessitant une gestion dépendant de l’application spécifique. Pour plus d’informations, consultez la section « Erreurs du système d’exploitation » dans [Référence d’erreurs et de journalisation](#error-and-logging-reference). 

### <a name="application-scenarios"></a>Scénarios d’application

- Applications clientes natives (iOS, Android, .NET Desktop ou Xamarin)
- Applications web appelant une ressource API (.NET)
- Applications monopages (JavaScript)
- Applications service à service (.NET, Java)
  - Tous les scénarios, y compris On-Behalf-Of
  - Scénarios spécifiques On-Behalf-Of

### <a name="error-cases-and-actionable-steps-native-client-applications"></a>Cas d’erreur et étapes possibles : Applications clientes natives

Si vous créez une application cliente native, il existe quelques cas de gestion des erreurs à prendre en compte concernant des problèmes réseau, des échecs passagers et autres erreurs spécifiques de la plateforme. Dans la plupart des cas, une application ne doit pas effectuer de nouvelles tentatives immédiates, mais plutôt attendre une interaction de l’utilisateur final pour l’inviter à se connecter. 

Il existe quelques cas spéciaux où une nouvelle tentative unique peut résoudre le problème. Par exemple, lorsqu’un utilisateur doit activer des données sur un appareil ou télécharger le répartiteur Azure AD après un premier échec. 

En cas d’échec, une application peut afficher une interface utilisateur pour permettre à l’utilisateur final de réaliser une interaction après laquelle il sera invité à effectuer une nouvelle tentative. Par exemple, si l’appareil a échoué en raison d’une erreur hors connexion, un bouton du type « Effectuer une nouvelle tentative de connexion » peut inviter l’utilisateur à effectuer une nouvelle tentative AcquireToken au lieu d’une nouvelle tentative immédiate. 

La gestion des erreurs dans les applications natives peut être définie par deux cas :

|  |  |
|------|-------------|
| **Cas n° 1** :<br>Erreur non renouvelable (la plupart des cas) | 1. N’effectue pas de nouvelle tentative immédiate. Présente à l’utilisateur final une interface utilisateur basée sur l’erreur spécifique qui l’invite à effectuer une nouvelle tentative (par exemple « Effectuer une nouvelle tentative de connexion » ou « Télécharger l’application du répartiteur Azure AD »). |
| **Cas n° 2** :<br>Erreur renouvelable | 1. Effectue une nouvelle tentative unique, car l’utilisateur final peut désormais se trouver dans un état permettant à la tentative de réussir.<br><br>2. Si la nouvelle tentative échoue, présente à l’utilisateur final une interface utilisateur basée sur l’erreur spécifique qui l’invite à effectuer une nouvelle tentative (« Effectuer une nouvelle tentative de connexion », « Télécharger l’application du répartiteur Azure AD », etc.). |

> [!IMPORTANT]
> Si un compte d’utilisateur réussit à se connecter à ADAL via un appel en mode silencieux et échoue, la requête interactive suivante permet à l’utilisateur final de se connecter à l’aide d’un autre compte. Après une requête AcquireToken réussie à l’aide d’un compte d’utilisateur, l’application doit vérifier que l’utilisateur connecté correspond à l’objet utilisateur local des applications. S’il ne correspond pas, aucune exception n’est générée (sauf dans Objective C), mais elle doit être prise en considération dans les cas où un utilisateur est connu localement avant les requêtes d’authentification (comme un échec d’un appel en mode silencieux).
>

#### <a name="net"></a>.NET

Les conseils suivants incluent des exemples de gestion des erreurs menée avec toutes les méthodes ADAL AcquireToken(…) en mode non silencieux, *sauf* : 

- AcquireTokenAsync(…, IClientAssertionCertification, …)
- AcquireTokenAsync(…, ClientCredential, …)
- AcquireTokenAsync(…, ClientAssertion, …)
- AcquireTokenAsync(…, UserAssertion,…)   

Votre code serait implémenté comme suit :

```csharp
try {
    AcquireTokenAsync(…);
} 
    
catch(AdalServiceException e) {
    // Exception: AdalServiceException 
    // Represents an error produced by the STS. 
    // e.ErrorCode contains the error code and description, which can be used for debugging. 
    // NOTE: Do not code a dependency on the contents of the error description, as it can change over time.
    
    // Design time consideration: Certain errors may be caused at development and exposed through this exception. 
    // Looking inside the description will give more guidance on resolving the specific issue. 

    // Action: Case 1: Non-Retryable 
    // Do not perform an immediate retry. Only retry after user action. 
    // Example Errors: default case

    } 

catch (AdalException e) {
    // Exception: AdalException 
    // Represents a library exception generated by ADAL .NET.
    // e.ErrorCode contains the error code

    // Action: Case 1, Non-Retryable 
    // Do not perform an immediate retry. Only retry after user action. 
    // Example Errors: network_not_available, default case
}
```

> [!NOTE]
> Une considération spéciale doit être accordée à ADAL .NET, car il prend en charge PromptBehavior.Never, qui a un comportement similaire à celui d’AcquireTokenSilent.
>

Les conseils suivants incluent des exemples de gestion des erreurs menée avec les méthodes ADAL suivantes : 

- acquireToken(…, PromptBehavior.Never)

Votre code serait implémenté comme suit :

```csharp
    try {acquireToken(…, PromptBehavior.Never);
    } 

catch(AdalServiceException e) {
    // Exception: AdalServiceException represents 
    // Represents an error produced by the STS. 
    // e.ErrorCode contains the error code and description, which can be used for debugging. 
    // NOTE: Do not code a dependency on the contents of the error description, as it can change over time.

    // Action: Case 1: Non-Retryable 
    // Do not perform an immediate retry. Only retry after user action. 
    // Example Errors: default case

} catch (AdalException e) {
    // Error Code: e.ErrorCode == "user_interaction_required"
    // Description: user_interaction_required indicates the silent request failed 
    // in a way that's resolvable with an interactive request.
    // Action: Resolvable with an interactive request. 

    // Action: Case 1, Non-Retryable 
    // Do not perform an immediate retry. Only retry after user action. 
    // Example Errors: network_not_available, default case
}
```

#### <a name="android"></a>Android

Les conseils suivants incluent des exemples de gestion des erreurs menée avec toutes les méthodes ADAL AcquireToken(…) en mode non silencieux. 

Votre code serait implémenté comme suit :

```java
AcquireTokenAsync(…);

// *Inside callback*
public void onError(Exception e) {
    if (e instanceof AuthenticationException) {
        // Exception: AdalException 
        // Represents a library exception generated by ADAL Android.
        // Error Code: e.getCode();

        // Error: ADALError.BROKER_APP_INSTALLATION_STARTED
        // Description: Broker app not installed, user will be prompted to download the app. 

        // Action: Case 2, Retriable Error 
        // Perform a single retry. If that fails, only try again after user action. 

        // Action: Case 1, Non-Retriable 
        // Do not perform an immediate retry. Only retry after user action. 
        // Example Errors: default case, DEVICE_CONNECTION_IS_NOT_AVAILABLE
    }
}
```

#### <a name="ios"></a>iOS

Les conseils suivants incluent des exemples de gestion des erreurs menée avec toutes les méthodes ADAL AcquireToken(…) en mode non silencieux. 

Votre code serait implémenté comme suit :

```objc
[context acquireTokenWithResource:[ARGS], completionBlock:^(ADAuthenticationResult *result) {
    if (result.status == AD_FAILED) {
        if ([error.domain isEqualToString:ADAuthenticationErrorDomain]){
            // Exception: AD_FAILED 
            // Represents a library error generated by ADAL ObjC.
            // Error Code: result.error.code 

            // Error: AD_ERROR_SERVER_WRONG_USER
            // Description: App passed a user into ADAL and the end user signed in with a different account. 
            // Action: Case 1, Non-retriable (as is) and up to the application on how to handle this case. 
            // It can attempt a new request without specifying the user, or use UI to clarify the user account to sign in. 

            // Action: Case 1, Non-Retriable 
            // Do not perform an immediate retry. Only retry after user action. 
            // Example Errors: default case
        }
    }
}]
```

### <a name="error-cases-and-actionable-steps-web-applications-that-call-a-resource-api-net"></a>Cas d’erreur et étapes possibles : Applications web appelant une ressource API (.NET)

Si vous créez une application web .NET qui obtient un jeton à l’aide d’un code d’autorisation pour une ressource, le seul code requis est un gestionnaire par défaut pour le cas générique. 

Les conseils suivants incluent des exemples de gestion des erreurs menée avec les méthodes ADAL suivantes : 

- AcquireTokenByAuthorizationCodeAsync(…)

Votre code serait implémenté comme suit :

```csharp
try {
    AcquireTokenByAuthorizationCodeAsync(…);
} 

catch (AdalException e) {
    // Exception: AdalException
    // Represents a library exception generated by ADAL .NET.
    // Error Code: e.ErrorCode

    // Action: Do not perform an immediate retry. Only try again after user action or wait until much later. 
    // Example Errors: default case
}
```

### <a name="error-cases-and-actionable-steps-single-page-applications-adaljs"></a>Cas d’erreur et étapes possibles : Applications monopages (adal.js)

Si vous créez une application monopage à l’aide d’adal.js avec AcquireToken, le code de gestion des erreurs est similaire à celui d’un appel en mode silencieux classique. Plus spécifiquement dans adal.js, AcquireToken n’affiche jamais d’interface utilisateur. 

Un échec AcquireToken inclut les cas suivants :

|  |  |
|------|-------------|
| **Cas n° 1** :<br>Résolvable via une requête interactive. | 1. Si login() échoue, n’effectue pas de nouvelle tentative immédiate. Effectue uniquement une nouvelle tentative après une action de l’utilisateur qui invite ce dernier à réessayer.|
| **Cas n° 2** :<br>Non résolvable via une requête interactive. L’erreur est renouvelable. | 1. Effectue une nouvelle tentative unique, car l’utilisateur final peut désormais se trouver dans un état permettant à la tentative de réussir.<br><br>2. Si la nouvelle tentative échoue, présente à l’utilisateur final une action basée sur l’erreur spécifique qui l’invite à effectuer une nouvelle tentative (« Effectuer une nouvelle tentative de connexion »). |
| **Cas n° 3** :<br>Non résolvable via une requête interactive. L’erreur n’est pas renouvelable. | 1. N’effectue pas de nouvelle tentative immédiate. Présente à l’utilisateur final une action basée sur l’erreur spécifique qui l’invite à effectuer une nouvelle tentative (« Effectuer une nouvelle tentative de connexion »). |

Votre code serait implémenté comme suit :

```javascript
AuthContext.acquireToken(…, function(error, errorDesc, token) {
    if (error || errorDesc) {
        // Represents any token acquisition failure that occurred. 
        // Error Code: error.indexOf("<ERROR_STRING>")

        // Errors: if (error.indexOf("interaction_required"))
        //         if (error.indexOf("login required"))
        // Description: ADAL wasn't able to silently acquire a token because of expire or fresh session. 
        // Action: Case 1, Resolvable with an interactive login() request. 

        // Error: if (error.indexOf("Token Renewal Failed")) 
        // Description: Timeout when refreshing the token.
        // Action: Case 2, Not resolvable interactively, error is retriable.
        // Perform a single retry. Only try again after user action.

        // Action: Case 3, Not resolvable interactively, error is not retriable. 
        // Do not perform an immediate retry. Only retry after user action.
        // Example Errors: default case
    }
}
```

### <a name="error-cases-and-actionable-steps-service-to-service-applications-net-only"></a>Cas d’erreur et étapes possibles : Applications service à service (.NET uniquement)

Si vous créez une application service à service qui utilise AcquireToken, il existe des erreurs clés que votre code doit gérer. Le seul recours en cas d’échec est de retourner l’erreur à l’application appelante (pour les cas On-Behalf-Of) ou d’appliquer une stratégie de retentative. 

#### <a name="all-scenarios"></a>Tous les scénarios

Pour *tous* les scénarios d’application service à service, y compris On-Behalf-Of :

- N’effectue pas de nouvelle tentative immédiate. ADAL effectue une seule nouvelle tentative pour certaines requêtes ayant échoué. 
- Effectue uniquement une nouvelle tentative après une action de l’utilisateur ou de l’application qui l’invite à effectuer une nouvelle tentative. Par exemple, une application démon fonctionnant selon un intervalle défini doit patienter jusqu’au prochain intervalle de nouvelle tentative.

Les conseils suivants incluent des exemples de gestion des erreurs menée avec les méthodes ADAL suivantes : 

- AcquireTokenAsync(…, IClientAssertionCertification, …)
- AcquireTokenAsync(…,ClientCredential, …)
- AcquireTokenAsync(…,ClientAssertion, …)
- AcquireTokenAsync(…,UserAssertion, …)

Votre code serait implémenté comme suit :

```csharp
try {
    AcquireTokenAsync(…);
} 

catch (AdalException e) {
    // Exception: AdalException
    // Represents a library exception generated by ADAL .NET.
    // Error Code: e.ErrorCode

    // Action: Do not perform an immediate retry. Only try again after user action (if applicable) or wait until much later. 
    // Example Errors: default case
}  
```

#### <a name="on-behalf-of-scenarios"></a>Scénarios On-Behalf-Of

Pour les scénarios d’application service à service *On-Behalf-Of*.

Les conseils suivants incluent des exemples de gestion des erreurs menée avec les méthodes ADAL suivantes : 

- AcquireTokenAsync(…, UserAssertion, …)

Votre code serait implémenté comme suit :

```csharp
try {
AcquireTokenAsync(…);
} 

catch (AdalServiceException e) {
    // Exception: AdalServiceException 
    // Represents an error produced by the STS. 
    // e.ErrorCode contains the error code and description, which can be used for debugging. 
    // NOTE: Do not code a dependency on the contents of the error description, as it can change over time.

    // Error: On-Behalf-Of Error Handler
    if (e.ErrorCode == "interaction_required") {
        // Description: The client needs to perform some action due to a config from admin. 
        // Action: Capture `claims` parameter inside ex.InnerException.InnerException.
        // Generate HTTP error 403 with claims, throw back HTTP error to client.
        // Wait for client to retry. 
    }
} 
        
catch (AdalException e) {
    // Exception: AdalException 
    // Represents a library exception generated by ADAL .NET.
    // Error Code: e.ErrorCode

    // Action: Do not perform an immediate retry. Only try again after user action (if applicable) or wait until much later. 
    // Example Error: default case
}
```

Nous avons créé un [exemple complet](https://github.com/Azure-Samples/active-directory-dotnet-webapi-onbehalfof-ca) qui illustre ce scénario.

## <a name="error-and-logging-reference"></a>Référence d’erreurs et de journalisation

### <a name="logging-personal-identifiable-information--organizational-identifiable-information"></a>Journalisation des informations d’identification personnelles et des informations d’identification organisationnelles 
Par défaut, la journalisation ADAL ne capture et n’enregistre pas les informations d’identification personnelles ou informations d’identification organisationnelles. La bibliothèque permet aux développeurs d’applications d’activer cette fonctionnalité via une méthode setter de la classe de journalisation. En enregistrant des informations d’identification personnelles ou des informations d’identification de l’organisation, l’application est chargée de gérer de façon sécurisée les données hautement sensibles et de respecter les exigences réglementaires.

### <a name="net"></a>.NET

#### <a name="adal-library-errors"></a>Erreurs de la bibliothèque ADAL

Pour explorer les erreurs ADAL spécifiques, le code source dans le [référentiel azure-activedirectory-library-for-dotnet](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/blob/8f6d560fbede2247ec0e217a21f6929d4375dcaa/src/ADAL.PCL/Utilities/Constants.cs#L58) constitue la meilleure référence d’erreurs.

#### <a name="guidance-for-error-logging-code"></a>Conseils pour le code de journalisation des erreurs

La journalisation d’ADAL .NET varie selon la plateforme de travail. Reportez-vous au [Wiki sur la journalisation](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Logging-in-ADAL.Net) pour voir le code qui permet d’activer la journalisation.

### <a name="android"></a>Android

#### <a name="adal-library-errors"></a>Erreurs de la bibliothèque ADAL

Pour explorer les erreurs ADAL spécifiques, le code source dans le [référentiel azure-activedirectory-library-for-android](https://github.com/AzureAD/azure-activedirectory-library-for-android/blob/dev/adal/src/main/java/com/microsoft/aad/adal/ADALError.java#L33) constitue la meilleure référence d’erreurs.

#### <a name="operating-system-errors"></a>Erreurs du système d'exploitation

Les erreurs du système d’exploitation Android sont exposées via AuthenticationException dans ADAL. Elles sont identifiables par la mention SERVER_INVALID_REQUEST et peuvent être plus granulaires à travers les descriptions d’erreurs. 

Pour obtenir la liste complète des erreurs courantes, ainsi que les étapes à suivre lorsque vous les rencontrez, reportez-vous au [Wiki ADAL pour Android](https://github.com/AzureAD/azure-activedirectory-library-for-android/wiki). 

#### <a name="guidance-for-error-logging-code"></a>Conseils pour le code de journalisation des erreurs

```java
// 1. Configure Logger
Logger.getInstance().setExternalLogger(new ILogger() {    
    @Override   
    public void Log(String tag, String message, String additionalMessage, LogLevel level, ADALError errorCode) { 
    // …
    // You can write this to logfile depending on level or errorcode. 
    writeToLogFile(getApplicationContext(), tag +":" + message + "-" + additionalMessage);    
    }
}

// 2. Set the log level
Logger.getInstance().setLogLevel(Logger.LogLevel.Verbose);

// By default, the `Logger` does not capture any PII or OII

// PII or OII will be logged
Logger.getInstance().setEnablePII(true);

// To STOP logging PII or OII, use the following setter
Logger.getInstance().setEnablePII(false);


// 3. Send logs to logcat.
adb logcat > "C:\logmsg\logfile.txt";
```

### <a name="ios"></a>iOS

#### <a name="adal-library-errors"></a>Erreurs de la bibliothèque ADAL

Pour explorer les erreurs ADAL spécifiques, le code source dans le [référentiel azure-activedirectory-library-for-objc](https://github.com/AzureAD/azure-activedirectory-library-for-objc/blob/dev/ADAL/src/ADAuthenticationError.m#L295) constitue la meilleure référence d’erreurs.

#### <a name="operating-system-errors"></a>Erreurs du système d’exploitation

Des erreurs iOS peuvent survenir au cours de la connexion quand les utilisateurs utilisent des vues web, et selon la nature de l’authentification. Cela peut être dû à des conditions telles que des erreurs TLS/SSL, des délais d’expiration ou des erreurs réseau :

- Pour le partage de droits, les connexions ne sont pas persistantes et le cache apparaît vide. Vous pouvez résoudre l’erreur en ajoutant la ligne de code suivante au trousseau : `[[ADAuthenticationSettings sharedInstance] setSharedCacheKeychainGroup:nil];`
- Pour l’ensemble d’erreurs NsUrlDomain, l’action varie en fonction de la logique d’application. Consultez la [documentation de référence sur NSURLErrorDomain](https://developer.apple.com/documentation/foundation/nsurlerrordomain#declarations) pour connaître les instances spécifiques qui peuvent être gérées.
- Consultez les [problèmes courants Obj-C ADAL](https://github.com/AzureAD/azure-activedirectory-library-for-objc#adauthenticationerror) pour obtenir la liste des erreurs courantes gérée par l’équipe Objective-C ADAL.

#### <a name="guidance-for-error-logging-code"></a>Conseils pour le code de journalisation des erreurs

```objc
// 1. Enable NSLogging
[ADLogger setNSLogging:YES];

// 2. Set the log level (if you want verbose)
[ADLogger setLevel:ADAL_LOG_LEVEL_VERBOSE];

// 3. Set up a callback block to simply print out
[ADLogger setLogCallBack:^(ADAL_LOG_LEVEL logLevel, NSString *message, NSString *additionalInformation, NSInteger errorCode, NSDictionary *userInfo) {
     NSString* log = [NSString stringWithFormat:@"%@ %@", message, additionalInformation];    
     NSLog(@"%@", log);
}];
```

### <a name="guidance-for-error-logging-code---javascript"></a>Conseils pour le code de journalisation des erreurs - JavaScript 

```javascript
0: Error1: Warning2: Info3: Verbose
window.Logging = {
    level: 3,
    log: function (message) {
        console.log(message);
    }
};
```

## <a name="related-content"></a>Contenu connexe

* [Bibliothèques Azure AD Authentication][AAD-Auth-Libraries]
* [Scénarios Azure AD Authentication][AAD-Auth-Scenarios]
* [Intégration d’applications dans Azure Active Directory][AAD-Integrating-Apps]

Utilisez la section de commentaires suivante pour fournir des commentaires et nous aider à affiner et à mettre en forme notre contenu.

[![Affiche le bouton « Se connecter avec Microsoft »][AAD-Sign-In]][AAD-Sign-In]
<!--Reference style links -->

[AAD-Auth-Libraries]: ./active-directory-authentication-libraries.md
[AAD-Auth-Scenarios]:v1-authentication-scenarios.md
[AAD-Integrating-Apps]:../develop/quickstart-register-app.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json
[AZURE-portal]: https://portal.azure.com

<!--Image references-->
[AAD-Sign-In]:./media/active-directory-devhowto-multi-tenant-overview/sign-in-with-microsoft-light.png

