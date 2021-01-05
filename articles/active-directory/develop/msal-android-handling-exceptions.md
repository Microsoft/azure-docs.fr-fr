---
title: Erreurs et exceptions (MSAL Android) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment gérer les erreurs et exceptions, l’accès conditionnel et les revendications dans les applications MSAL Android.
services: active-directory
author: hamiltonha
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: troubleshooting
ms.workload: identity
ms.date: 08/07/2020
ms.author: hahamil
ms.reviewer: marsma
ms.openlocfilehash: f40c2bb0f529f9e0683c67bea884443458707f4f
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92206592"
---
# <a name="handle-exceptions-and-errors-in-msal-for-android"></a>Gérer les exceptions et les erreurs dans MSAL pour Android

Dans la bibliothèque Microsoft Authentication Library (MSAL), les exceptions sont destinées aux développeurs d’applications à des fins de résolution de problèmes. Les messages liés aux exceptions ne sont pas localisés.

Quand vous traitez les exceptions et les erreurs, vous pouvez utiliser le type d’exception lui-même et le code d’erreur pour les distinguer.  Pour obtenir la liste des codes d’erreur, consultez [Codes d’erreur d’authentification et d’autorisation](reference-aadsts-error-codes.md).

Durant la connexion, vous pouvez rencontrer des erreurs concernant les consentements, l’accès conditionnel (MFA, gestion des appareils, restrictions basées sur l’emplacement), l’émission et l’échange de jetons, et les propriétés utilisateur.


|Classe d’erreur | Cause/chaîne d’erreur| Procédure de gestion |
|-----------|------------|----------------|
|`MsalUiRequiredException`| <ul><li>`INVALID_GRANT`: Le jeton d’actualisation utilisé pour accepter le jeton d’accès n’est pas valide, a expiré ou a été révoqué. Cette exception peut être due à une modification de mot de passe. </li><li>`NO_TOKENS_FOUND`: Le jeton d’accès n’existe pas et aucun jeton d’actualisation ne peut être trouvé pour accepter le jeton d’accès.</li> <li>Passage à une édition supérieure requis<ul><li>MFA</li><li>Revendications manquantes</li></ul></li><li>Bloqué par l’accès conditionnel (par exemple, installation d’un [répartiteur d’authentification](./msal-android-single-sign-on.md) requise)</li><li>`NO_ACCOUNT_FOUND`: Aucun compte disponible dans le cache pour l’authentification silencieuse.</li></ul> |Appelez `acquireToken()` pour inviter l’utilisateur à entrer son nom d’utilisateur et son mot de passe, puis à donner son consentement et à effectuer une authentification multifacteur.|
|`MsalDeclinedScopeException`|<ul><li>`DECLINED_SCOPE`: L’utilisateur ou le serveur n’a pas accepté toutes les étendues. Le serveur peut refuser une étendue si l’étendue demandée n’est pas prise en charge, n’est pas reconnue ou n’est pas prise en charge pour un compte particulier. </li></ul>| Le développeur doit décider s’il poursuit l’authentification avec les étendues accordées ou s’il met fin au processus d’authentification. Possibilité de soumettre à nouveau la demande d’acquisition de jeton uniquement pour les étendues accordées et de fournir des indications pour lesquelles des autorisations ont été accordées en transmettant `silentParametersForGrantedScopes` et en appelant `acquireTokenSilent`. |
|`MsalServiceException`|<ul><li>`INVALID_REQUEST`: Il manque un paramètre nécessaire à cette requête, elle comprend un paramètre non valide, plus d’un paramètre à la fois, ou elle est incorrecte. </li><li>`SERVICE_NOT_AVAILABLE`: Représente les codes d’erreur 500/503/506 dus à l’interruption du service. </li><li>`UNAUTHORIZED_REQUEST`: Le client n’est pas autorisé à demander un code d’autorisation.</li><li>`ACCESS_DENIED`: Le propriétaire de la ressource ou le serveur d’autorisation a refusé la requête.</li><li>`INVALID_INSTANCE` : échec de la validation de `AuthorityMetadata`</li><li>`UNKNOWN_ERROR`: Échec de la requête au serveur, mais aucune erreur ni `error_description` ne sont renvoyées par le service.</li><ul>| Cette classe d’exception représente les erreurs de communication avec le service, qui peuvent provenir des points de terminaison d’autorisation ou de jeton. MSAL lit l’erreur et error_description à partir de la réponse du serveur. En règle générale, ces erreurs sont résolues en corrigeant les configurations d’application dans le code ou dans le portail d’inscription des applications. Rarement, une interruption de service peut déclencher cet avertissement, qui ne peut être atténué qu’en attendant le rétablissement du service.  |
|`MsalClientException`|<ul><li> `MULTIPLE_MATCHING_TOKENS_DETECTED`: Plusieurs entrées de cache sont trouvées et le Kit de développement logiciel (SDK) ne peut pas identifier le jeton d’accès ou d’actualisation approprié à partir du cache. Cette exception indique généralement un bogue dans le Kit de développement logiciel (SDK) pour le stockage des jetons ou que l’autorité n’est pas fournie dans la requête silencieuse et que plusieurs jetons correspondants sont trouvés. </li><li>`DEVICE_NETWORK_NOT_AVAILABLE`: Aucun réseau actif n’est disponible sur l’appareil. </li><li>`JSON_PARSE_FAILURE`: Le Kit de développement logiciel (SDK) n’a pas réussi à analyser le format JSON.</li><li>`IO_ERROR` : `IOException` s’est produite, il peut s’agir d’une erreur d’appareil ou de réseau. </li><li>`MALFORMED_URL`: L’URL est incorrecte. Probablement dû à la construction de la requête d’authentification, de l’autorité ou de l’URI de redirection. </li><li>`UNSUPPORTED_ENCODING`: L’encodage n’est pas pris en charge par l’appareil. </li><li>`NO_SUCH_ALGORITHM`: L’algorithme utilisé pour générer le test [PKCE ](https://tools.ietf.org/html/rfc7636) n’est pas pris en charge. </li><li>`INVALID_JWT` : `JWT` retourné par le serveur n’est pas valide ou est vide ou incorrect. </li><li>`STATE_MISMATCH`: L’état de la réponse d’autorisation ne correspondait pas à l’état dans la demande d’autorisation. Pour les demandes d’autorisation, le Kit de développement logiciel (SDK) vérifie l’état renvoyé par la redirection et celui qui est envoyé dans la demande. </li><li>`UNSUPPORTED_URL`: URL non prise en charge, impossible d’effectuer la validation de l’autorité ADFS. </li><li> `AUTHORITY_VALIDATION_NOT_SUPPORTED`: L’autorité n’est pas prise en charge pour la validation de l’autorité. Le Kit de développement logiciel (SDK) prend en charge les autorités B2C, mais ne prend pas en charge la validation de l’autorité B2C. Seul un hôte connu est pris en charge. </li><li>`CHROME_NOT_INSTALLED`: Chrome n’est pas installé sur l’appareil. Le Kit de développement logiciel (SDK) utilise l’onglet Chrome personnalisé pour les demandes d’autorisation, s’il est disponible, et revient au navigateur Chrome. </li><li>`USER_MISMATCH`: L’utilisateur fourni dans la demande d’acquisition de jeton ne correspond pas à l’utilisateur retourné par le serveur.</li></ul>|Cette classe d’exception représente des erreurs générales qui sont locales à la bibliothèque. Ces exceptions peuvent être gérées en corrigeant la demande.|
|`MsalUserCancelException`|<ul><li>`USER_CANCELED`: L’utilisateur a lancé un flux interactif et en a annulé la demande avant de recevoir des jetons en retour. </li></ul>||
|`MsalArgumentException`|<ul><li>`ILLEGAL_ARGUMENT_ERROR_CODE`</li><li>`AUTHORITY_REQUIRED_FOR_SILENT`: L’autorité doit être spécifiée pour `acquireTokenSilent`.</li></ul>|Ces erreurs peuvent être atténuées par le développeur en corrigeant les arguments et en veillant à ce que l’activité pour l’authentification interactive, le rappel d’achèvement, les étendues et un compte avec un ID valide ont été fournis.|


## <a name="catching-errors"></a>Interception des erreurs

L’extrait de code suivant montre un exemple d’interception d’erreurs dans le cas d’appels `acquireToken` silencieux.

```java
/**
 * Callback used in for silent acquireToken calls.
 */
private SilentAuthenticationCallback getAuthSilentCallback() {
    return new SilentAuthenticationCallback() {

        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) {
            Log.d(TAG, "Successfully authenticated");

            /* Successfully got a token, use it to call a protected resource - MSGraph */
            callGraphAPI(authenticationResult);
        }

        @Override
        public void onError(MsalException exception) {
            /* Failed to acquireToken */
            Log.d(TAG, "Authentication failed: " + exception.toString());
            displayError(exception);

            if (exception instanceof MsalClientException) {
                /* Exception inside MSAL, more info inside MsalError.java */
            } else if (exception instanceof MsalServiceException) {
                /* Exception when communicating with the STS, likely config issue */
            } else if (exception instanceof MsalUiRequiredException) {
                /* Tokens expired or no session, retry with interactive */
            }
        }
    };
}
```

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [erreurs de journalisation](./msal-logging.md?tabs=android)