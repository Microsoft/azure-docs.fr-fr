---
title: Guide de migration ADAL vers MSAL pour Android | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment migrer votre application Android ADAL (bibliothèque d’authentification Active Directory) Azure vers Microsoft Authentication Library (MSAL).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.tgt_pltfrm: Android
ms.workload: identity
ms.date: 10/14/2020
ms.author: marsma
ms.reviewer: shoatman
ms.custom: aaddev
ms.openlocfilehash: bf9b3a154e19fab08c46f9838f555e223f10e8a0
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97672285"
---
# <a name="adal-to-msal-migration-guide-for-android"></a>Guide de migration ADAL vers MSAL pour Android

Cet article souligne les changements que vous devez apporter pour migrer une application qui utilise la bibliothèque d’authentification Active Directory (ADAL) Azure vers une application qui utilise Microsoft Authentication Library (MSAL).

## <a name="difference-highlights"></a>Principales différences

ADAL fonctionne avec le point de terminaison Azure Active Directory v1.0. Microsoft Authentication Library (MSAL) fonctionne avec la plateforme d’identités Microsoft, anciennement connue comme point de terminaison Azure Active Directory v2.0. La plateforme d’identités Microsoft diffère d’Azure Active Directory v1.0 des manières suivantes :

Prend en charge :
  - Identité organisationnelle (Azure Active Directory).
  - Identités non organisationnelles telles que Outlook.com, Xbox Live, etc
  - (Azure AD B2C uniquement) Connexion fédérée avec Google, Facebook, Twitter et Amazon

- Ses normes sont compatibles avec :
  - OAuth v2.0
  - OpenID Connect (OIDC)

L’API publique MSAL introduit des changements importants, notamment les suivants :

- Un nouveau modèle pour l’accès aux jetons :
  - ADAL fournit l’accès aux jetons par le biais d’`AuthenticationContext`, qui représente le serveur. MSAL fournit l’accès aux jetons par le biais de `PublicClientApplication`, qui représente le client. Les développeurs de clients n’ont pas besoin de créer une instance `PublicClientApplication` pour chaque autorité avec laquelle ils ont besoin d’interagir. Une seule configuration `PublicClientApplication` est nécessaire.
  - Prise en charge de la demande de jetons d’accès à l’aide d’étendues en plus des identificateurs de ressources.
  - Prise en charge d’un consentement incrémentiel. Les développeurs peuvent demander des étendues à mesure que l’utilisateur accède à de plus en plus de fonctionnalités dans l’application, y compris celles qui ne sont pas incluses lors de l’inscription de l’application.
  - Les autorités ne sont plus validées à l’exécution. Au lieu de cela, il incombe au développeur de déclarer la liste des « autorités connues » pendant le développement.
- Changements d’API de jeton :
  - Dans ADAL, `AcquireToken()` effectue d’abord une requête silencieuse. En cas d’échec, il effectue une requête interactive. Ce comportement a conduit certains développeurs à ne compter que sur `AcquireToken`, ce qui a eu pour conséquence que l’utilisateur a parfois été invité de manière inattendue à fournir des informations d’identification. MSAL oblige les développeurs à déterminer délibérément le moment auquel l’utilisateur reçoit une invite d’interface utilisateur.
    - `AcquireTokenSilent` aboutit toujours à une demande silencieuse qui réussit ou échoue.
    - `AcquireToken` aboutit toujours à une demande qui invite l’utilisateur via l’interface utilisateur.
- MSAL prend en charge une connexion à partir d’un navigateur par défaut ou d’un affichage web incorporé :
  - Par défaut, le navigateur par défaut sur l’appareil est utilisé. Cela permet à MSAL d’utiliser l’état d’authentification (cookies) éventuellement déjà présent pour un ou plusieurs comptes connectés. Si aucun état d’authentification n’est présent, l’authentification au cours de l’autorisation par le biais de MSAL entraîne la création d’un état d’authentification (cookies) en faveur d’autres applications web qui seront utilisées dans le même navigateur.
- Nouveau modèle d’exception :
  - Les exceptions définissent plus clairement le type d’erreur qui s’est produit et sur ce que le développeur doit faire pour la résoudre.
- MSAL prend en charge les objets de paramètre pour les appels `AcquireToken` et `AcquireTokenSilent`.
- MSAL prend en charge la configuration déclarative pour :
  - ID client, URI de redirection.
  - Navigateur incorporé plutôt que par défaut
  - Autorités
  - Paramètres HTTP,comme le délai de lecture et de connexion

## <a name="your-app-registration-and-migration-to-msal"></a>Votre inscription d’application et votre migration vers MSAL

Vous n’avez pas besoin de modifier l’inscription de votre application existante pour utiliser MSAL. Si vous voulez tirer parti du consentement incrémentiel/progressif, vous devez éventuellement examiner l’inscription pour identifier les étendues spécifiques à demander de façon incrémentielle. Plus d’informations sur les étendues et le consentement incrémentiel sont données ci-après.

Dans votre inscription d’application dans le portail, vous voyez un onglet **Autorisations d’API**. Celui-ci fournit la liste des API et autorisations (étendues) auxquelles votre application est actuellement configurée pour demander l’accès. Il présente également la liste des noms d’étendue associés à chaque autorisation d’API.

### <a name="user-consent"></a>Consentement de l’utilisateur

Avec ADAL et le point de terminaison Azure AD v1, le consentement de l’utilisateur pour les ressources qu’il possède a été donné lors de la première utilisation. Avec MSAL et la plateforme d’identités Microsoft, le consentement peut être demandé de manière incrémentielle. Le consentement incrémentiel s’avère utile pour les autorisations qu’un utilisateur peut considérer comme un privilège élevé ou mettre en doute si elles ne sont pas octroyées avec une explication claire de leur caractère obligatoire. Dans ADAL, ces autorisations peuvent entraîner l’abandon de la connexion à votre application par l’utilisateur.

> [!TIP]
> Nous vous recommandons d’utiliser le consentement incrémentiel dans les scénarios où vous avez besoin de fournir à votre utilisateur un contexte supplémentaire sur la raison pour laquelle votre application a besoin d’une autorisation.

### <a name="admin-consent"></a>Consentement de l’administrateur

Les administrateurs de l’organisation peuvent consentir aux autorisations dont votre application a besoin au nom de tous les membres de leur organisation. Certaines organisations autorisent uniquement les administrateurs à consentir aux applications. Le consentement administrateur vous oblige à inclure toutes les autorisations et étendues d’API utilisées par votre application dans votre inscription d’application.

> [!TIP]
> Même si vous pouvez demander une étendue en utilisant MSAL pour une ressource qui n’est pas incluse dans votre inscription d’application, nous vous recommandons de mettre à jour votre inscription d’application pour inclure toutes les ressources et étendues auxquelles un utilisateur peut être amené à octroyer une autorisation.

## <a name="migrating-from-resource-ids-to-scopes"></a>Migration des ID de ressource vers des étendues

### <a name="authenticate-and-request-authorization-for-all-permissions-on-first-use"></a>Authentifier et autoriser toutes les autorisations lors de la première utilisation

Si vous utilisez actuellement ADAL et que vous n’avez pas besoin d’utiliser le consentement incrémentiel, le moyen le plus simple de commencer à utiliser MSAL consiste à effectuer une demande `acquireToken` à l’aide du nouvel objet `AcquireTokenParameter` et à définir la valeur de l’ID de la ressource.

> [!CAUTION]
> Il n’est pas possible de définir à la fois des étendues et un ID de la ressource. Toute tentative d’une telle définition entraîne un `IllegalArgumentException`.

Cela entraîne le même comportement v1 que celui que vous connaissez. Toutes les autorisations demandées dans votre inscription d’application sont demandées à l’utilisateur lors de sa première interaction.

### <a name="authenticate-and-request-permissions-only-as-needed"></a>Authentifier et demander des autorisations uniquement si elles sont nécessaires

Pour tirer parti du consentement incrémentiel, dressez la liste des autorisations (étendues) que votre application utilise à partir de votre inscription d’application et organisez-les en deux listes selon :

- Les étendues à demander lors de la première interaction de l’utilisateur avec votre application pendant la connexion.
- Les autorisations associées à une fonctionnalité importante de votre application que vous devrez également expliquer à l’utilisateur.

Une fois que vous avez organisé les étendues, organisez chaque liste selon la ressource (API) pour laquelle vous voulez demander un jeton, ainsi que toutes les autres étendues que vous voulez que l’utilisateur autorise en même temps.

L’objet des paramètres utilisé pour effectuer votre demande à MSAL prend en charge :

- `Scope`: Liste des étendues pour lesquelles vous souhaitez demander une autorisation et recevoir un jeton d’accès.
- `ExtraScopesToConsent`: Liste supplémentaire d’étendues pour lesquelles vous souhaitez demander une autorisation lorsque vous demandez un jeton d’accès pour une autre ressource. Cette liste des étendues vous permet de réduire le nombre de fois où vous devez demander une autorisation utilisateur. Ce qui signifie moins d’invites d’autorisation utilisateur ou de consentement.

## <a name="migrate-from-authenticationcontext-to-publicclientapplications"></a>Migrer depuis AuthenticationContext vers PublicClientApplications

### <a name="constructing-publicclientapplication"></a>Construction de PublicClientApplication

Quand vous utilisez MSAL, vous instanciez un `PublicClientApplication`. Cet objet modélise l’identité de votre application et sert à effectuer des demandes auprès d’une ou plusieurs autorités. Avec cet objet, vous allez configurer l’identité de votre client, l’URI de redirection, l’autorité par défaut, l’utilisation ou non du navigateur de l’appareil ou de la vue web incorporée, le niveau de journalisation, etc.

Vous pouvez configurer de façon déclarative cet objet avec JSON, que vous fournissez sous forme de fichier ou de magasin en tant que ressource dans votre APK.

Même si cet objet n’est pas un singleton, en interne, il utilise des `Executors` partagés pour les demandes interactives et silencieuses.

### <a name="business-to-business"></a>Business to Business (B2B)

Dans ADAL, chaque organisation à laquelle vous demandez des jetons d’accès exige une instance distincte du `AuthenticationContext`. Dans MSAL, il ne s’agit plus d’une exigence. Vous pouvez spécifier l’autorité à partir de laquelle vous souhaitez demander un jeton dans le cadre de votre demande silencieuse ou interactive.

### <a name="migrate-from-authority-validation-to-known-authorities"></a>Migrer de la validation des autorités vers des autorités connues

MSAL n’a pas d’indicateur permettant d’activer ou de désactiver la validation des autorités. La validation des autorités est une fonctionnalité dans ADAL, ainsi que dans les premières versions de MSAL, qui empêche votre code de demander des jetons à une autorité potentiellement malveillante. MSAL récupère à présent la liste des autorités connues auprès de Microsoft et fusionne cette liste avec les autorités que vous avez spécifiées dans votre configuration.

> [!TIP]
> Si vous êtes utilisateur Azure Business to Consumer (B2C), cela signifie que vous n’avez plus à désactiver la validation des autorités. Au lieu de cela, incluez chacune de vos stratégies Azure AD B2C prises en charge en tant qu’autorités dans votre configuration MSAL.

Si vous tentez d’utiliser une autorité qui n’est pas connue de Microsoft et qui n’est pas incluse dans votre configuration, vous obtenez un `UnknownAuthorityException`.

### <a name="logging"></a>Journalisation
Vous pouvez maintenant configurer la journalisation de façon déclarative dans le cadre de votre configuration, comme suit :

```json
"logging": {
  "pii_enabled": false,
  "log_level": "WARNING",
  "logcat_enabled": true
}
```

## <a name="migrate-from-userinfo-to-account"></a>Migrer depuis UserInfo vers Account

Dans ADAL, `AuthenticationResult` fournit un objet `UserInfo` utilisé pour récupérer des informations sur le compte authentifié. Le terme « utilisateur », qui signifiait un agent humain ou logiciel, était appliqué de telle sorte qu’il était difficile de communiquer que certaines applications prenaient en charge un seul utilisateur (qu’il s’agisse d’un agent humain ou logiciel) ayant plusieurs comptes.

Prenons l’exemple d’un compte bancaire. Vous pouvez avoir plusieurs comptes dans plusieurs établissements financiers. Quand vous ouvrez un compte, vous (l’utilisateur) recevez des informations d’identification, comme un identifiant client et un code secret, que vous utilisez pour accéder à votre compte en ligne, effectuer des virements, etc., pour chaque compte. Vous pouvez utiliser ces informations d’identification uniquement auprès de l’établissement financier qui les a émises.

Par analogie, comme les comptes d’un établissement financier, les comptes de la plateforme d’identités Microsoft sont accessibles à l’aide d’informations d’identification. Ces dernières sont soit inscrites auprès de Microsoft, soit émises par Microsoft. Ou encore par Microsoft pour le compte d’une organisation.

Dans cette analogie, la différence entre la plateforme d’identités Microsoft et l’établissement financier est que la plateforme d’identités Microsoft fournit un framework qui permet à un utilisateur d’utiliser un seul compte, et ses informations d’identification associées, pour accéder à des ressources qui appartiennent à plusieurs personnes et organisations. Cela revient à pouvoir utiliser une carte de crédit émise par une première banque dans un autre établissement financier. Cela fonctionne parce que toutes les organisations en question utilisent la plateforme d’identités Microsoft, ce qui permet d’utiliser un même compte dans plusieurs organisations. Voici un exemple :

Sam travaille pour Contoso.com, mais gère les machines virtuelles Azure qui appartiennent à Fabrikam.com. Pour que Sam gère les machines virtuelles de Fabrikam, il doit être autorisé à y accéder. Cet accès peut être accordé en ajoutant le compte de Sam à Fabrikam.com, puis en accordant à son compte un rôle qui lui permet d’utiliser les machines virtuelles. Cette opération s’effectue avec le portail Azure.

L’ajout du compte Contoso.com de Sam en tant que membre de Fabrikam.com entraîne alors la création d’un enregistrement dans l’annuaire Azure Active Directory de Fabrikam.com pour Sam. L’enregistrement de Sam dans Azure Active Directory est appelé objet utilisateur. Dans cet exemple, cet objet utilisateur repointe alors vers l’objet utilisateur de Sam dans Contoso.com. L’objet utilisateur Fabrikam de Sam est la représentation locale de Sam, qui est utilisée pour stocker des informations sur le compte associé à Sam dans le contexte de Fabrikam.com. Dans Contoso.com, le poste de Sam est Consultant DevOps senior. Dans Fabrikam, son poste est Sous-traitant machines virtuelles. Dans Contoso.com, Sam n’est pas responsable ni autorisé à gérer des machines virtuelles. Dans Fabrikam.com, il s’agit de sa seule fonction. Pourtant, Sam n’a toujours qu’un seul ensemble d’informations d’identification à gérer, à savoir celles émises par Contoso.com.

Une fois qu’un appel à `acquireToken` aboutit, vous voyez une référence à un objet `IAccount` qui peut être utilisée dans les demandes `acquireTokenSilent` ultérieures.

### <a name="imultitenantaccount"></a>IMultiTenantAccount

Si vous avez une application qui accède à des revendications à propos d’un compte à partir de chacun des locataires dans lesquels le compte est représenté, vous pouvez caster des objets `IAccount` en `IMultiTenantAccount`. Cette interface fournit un mappage des `ITenantProfiles`, indexé par ID de locataire, qui vous permet d’accéder aux revendications appartenant au compte dans chacun des locataires à partir desquels vous avez demandé un jeton, par rapport au compte actuel.

Les revendications situées à la racine de `IAccount` et `IMultiTenantAccount` contiennent toujours les revendications du locataire de base. Si vous n’avez pas encore effectué de demande de jeton dans le locataire de base, cette collection est vide.

## <a name="other-changes"></a>Autres modifications

### <a name="use-the-new-authenticationcallback"></a>Utiliser le nouveau AuthenticationCallback

```java
// Existing ADAL Interface
public interface AuthenticationCallback<T> {

    /**
     * This will have the token info.
     *
     * @param result returns <T>
     */
    void onSuccess(T result);

    /**
     * Sends error information. This can be user related error or server error.
     * Cancellation error is AuthenticationCancelError.
     *
     * @param exc return {@link Exception}
     */
    void onError(Exception exc);
}
```

```java
// New Interface for Interactive AcquireToken
public interface AuthenticationCallback {

    /**
     * Authentication finishes successfully.
     *
     * @param authenticationResult {@link IAuthenticationResult} that contains the success response.
     */
    void onSuccess(final IAuthenticationResult authenticationResult);

    /**
     * Error occurs during the authentication.
     *
     * @param exception The {@link MsalException} contains the error code, error message and cause if applicable. The exception
     *                  returned in the callback could be {@link MsalClientException}, {@link MsalServiceException}
     */
    void onError(final MsalException exception);

    /**
     * Will be called if user cancels the flow.
     */
    void onCancel();
}

// New Interface for Silent AcquireToken
public interface SilentAuthenticationCallback {

    /**
     * Authentication finishes successfully.
     *
     * @param authenticationResult {@link IAuthenticationResult} that contains the success response.
     */
    void onSuccess(final IAuthenticationResult authenticationResult);

    /**
     * Error occurs during the authentication.
     *
     * @param exception The {@link MsalException} contains the error code, error message and cause if applicable. The exception
     *                  returned in the callback could be {@link MsalClientException}, {@link MsalServiceException} or
     *                  {@link MsalUiRequiredException}.
     */
    void onError(final MsalException exception);
}
```

## <a name="migrate-to-the-new-exceptions"></a>Migrer vers les nouvelles exceptions

Dans ADAL, il existe un seul type d’exception, `AuthenticationException`, qui inclut une méthode pour récupérer la valeur d’énumération `ADALError`.
Dans MSAL, il existe une hiérarchie d’exceptions. Chacune possède son propre ensemble de codes d’erreur spécifiques associés.

| Exception                                        | Description                                                         |
|--------------------------------------------------|---------------------------------------------------------------------|
| `MsalArgumentException`                          | Levée si un ou plusieurs arguments d’entrée ne sont pas valides.                 |
| `MsalClientException`                            | Levée si l’erreur est côté client.                                 |
| `MsalDeclinedScopeException`                     | Levée si une ou plusieurs étendues demandées ont été refusées par le serveur. |
| `MsalException`                                  | Exception vérifiée par défaut levée par MSAL.                           |
| `MsalIntuneAppProtectionPolicyRequiredException` | Levée si la stratégie de protection MAMCA est activée pour la ressource.         |
| `MsalServiceException`                           | Levée si l’erreur est côté serveur.                                 |
| `MsalUiRequiredException`                        | Levée si le jeton ne peut pas être actualisé silencieusement.                    |
| `MsalUserCancelException`                        | Levée si l’utilisateur a annulé le workflow d’authentification.                |

### <a name="adalerror-to-msalexception-translation"></a>Traduction d’ADALError vers MsalException

| Si vous interceptez ces erreurs dans ADAL…  | …interceptez les exceptions MSAL suivantes :                                                         |
|--------------------------------------------------|---------------------------------------------------------------------|
| *Aucune ADALError équivalente* | `MsalArgumentException`                          |
| <ul><li>`ADALError.ANDROIDKEYSTORE_FAILED`<li>`ADALError.AUTH_FAILED_USER_MISMATCH`<li>`ADALError.DECRYPTION_FAILED`<li>`ADALError.DEVELOPER_AUTHORITY_CAN_NOT_BE_VALIDED`<li>`ADALError.EVELOPER_AUTHORITY_IS_NOT_VALID_INSTANCE`<li>`ADALError.DEVELOPER_AUTHORITY_IS_NOT_VALID_URL`<li>`ADALError.DEVICE_CONNECTION_IS_NOT_AVAILABLE`<li>`ADALError.DEVICE_NO_SUCH_ALGORITHM`<li>`ADALError.ENCODING_IS_NOT_SUPPORTED`<li>`ADALError.ENCRYPTION_ERROR`<li>`ADALError.IO_EXCEPTION`<li>`ADALError.JSON_PARSE_ERROR`<li>`ADALError.NO_NETWORK_CONNECTION_POWER_OPTIMIZATION`<li>`ADALError.SOCKET_TIMEOUT_EXCEPTION`</ul> | `MsalClientException`                            |
| *Aucune ADALError équivalente* | `MsalDeclinedScopeException`                     |
| <ul><li>`ADALError.APP_PACKAGE_NAME_NOT_FOUND`<li>`ADALError.BROKER_APP_VERIFICATION_FAILED`<li>`ADALError.PACKAGE_NAME_NOT_FOUND`</ul> | `MsalException`                                  |
| *Aucune ADALError équivalente* | `MsalIntuneAppProtectionPolicyRequiredException` |
| <ul><li>`ADALError.SERVER_ERROR`<li>`ADALError.SERVER_INVALID_REQUEST`</ul> | `MsalServiceException`                           |
| <ul><li>`ADALError.AUTH_REFRESH_FAILED_PROMPT_NOT_ALLOWED` | `MsalUiRequiredException`</ul>                        |
| *Aucune ADALError équivalente* | `MsalUserCancelException`                        |

### <a name="adal-logging-to-msal-logging"></a>Journalisation ADAL vers journalisation MSAL

```java
// Legacy Interface
    StringBuilder logs = new StringBuilder();
    Logger.getInstance().setExternalLogger(new ILogger() {
            @Override
            public void Log(String tag, String message, String additionalMessage, LogLevel logLevel, ADALError errorCode) {
                logs.append(message).append('\n');
            }
        });
```

```java
// New interface
  StringBuilder logs = new StringBuilder();
  Logger.getInstance().setExternalLogger(new ILoggerCallback() {
      @Override
      public void log(String tag, Logger.LogLevel logLevel, String message, boolean containsPII) {
          logs.append(message).append('\n');
      }
  });

// New Log Levels:
public enum LogLevel
{
    /**
     * Error level logging.
     */
    ERROR,
    /**
     * Warning level logging.
     */
    WARNING,
    /**
     * Info level logging.
     */
    INFO,
    /**
     * Verbose level logging.
     */
    VERBOSE
}
```
