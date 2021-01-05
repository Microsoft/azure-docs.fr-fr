---
title: Configuration d’application cliente (MSAL) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez les options de configuration pour les applications clientes publiques et confidentielles à l’aide de la bibliothèque d’authentification Microsoft (MSAL).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/20/2020
ms.author: marsma
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 9ec8a5fe5de751e40ebaa17629ff72c5f6b2adca
ms.sourcegitcommit: f311f112c9ca711d88a096bed43040fcdad24433
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94979983"
---
# <a name="application-configuration-options"></a>Options de configuration d’application

Pour vous authentifier et acquérir des jetons, vous initialisez une nouvelle application cliente publique ou confidentielle dans votre code. Vous pouvez définir plusieurs options de configuration lorsque vous initialisez l’application cliente dans la bibliothèque d’authentification Microsoft (MSAL). Ces options se répartissent en deux groupes :

- Options d’inscription, à savoir :
  - [Autorité](#authority) (composée de l’[instance](#cloud-instance) de fournisseur d’identité et de l’[audience](#application-audience) de connexion pour l’application et, éventuellement, l’ID de locataire)
  - [ID client](#client-id)
  - [URI de redirection](#redirect-uri)
  - [Secret client](#client-secret) (pour les applications clientes confidentielles)
- [Options de journalisation](#logging) incluant le niveau de journalisation, le contrôle de données personnelles et le nom du composant utilisant la bibliothèque

## <a name="authority"></a>Authority

L’autorité est une URL indiquant un annuaire dont la bibliothèque d’authentification Microsoft peut demander des jetons.

Les autorités courantes sont les suivantes :

| URL d’autorité courante | Quand l’utiliser |
|--|--|
| `https://login.microsoftonline.com/<tenant>/` | Connecter des utilisateurs d’une organisation spécifique uniquement. L’élément `<tenant>` dans l’URL est l’ID de locataire du locataire Azure Active Directory (Azure AD) (un GUID) ou son domaine locataire. |
| `https://login.microsoftonline.com/common/` | Connecter des utilisateurs disposant de comptes professionnels ou scolaires, ou de comptes Microsoft personnels. |
| `https://login.microsoftonline.com/organizations/` | Connecter des utilisateurs disposant de comptes professionnels ou scolaires. |
| `https://login.microsoftonline.com/consumers/` | Connecter des utilisateurs disposant uniquement de comptes Microsoft personnels (MSA). |

L’autorité que vous spécifiez dans votre code doit être cohérente avec les **types de comptes pris en charge** spécifiés pour l’application dans **Inscriptions d’applications** dans le portail Azure.

L’autorité peut être :

- Une autorité de cloud Azure AD.
- Une autorité Azure AD B2C. Voir [Spécificités B2C](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-specifics).
- Une autorité de services de fédération Active Directory (AD FS). Consultez [Support de AD FS](https://aka.ms/msal-net-adfs-support).

Les autorités de cloud Azure AD comprennent deux parties :

- L’*instance* de fournisseur d’identité.
- L’*audience* de connexion pour l’application.

L’instance et l’audience peuvent être concaténées et fournies en tant que qu’URL de l’autorité. Ce diagramme montre comment l’URL de l’autorité est composée :

![Composition de l’URL de l’autorité](media/msal-client-application-configuration/authority.png)

## <a name="cloud-instance"></a>Instance cloud

L’*instance* est utilisée pour spécifier si votre application connecte des utilisateurs à partir du cloud public Azure ou de clouds nationaux. En utilisant MSAL dans votre code, vous pouvez définir l’instance cloud Azure à l’aide d’une énumération ou en transmettant l’URL à l’[instance cloud nationale](authentication-national-cloud.md#azure-ad-authentication-endpoints) en tant que membre `Instance` (si vous le connaissez).

MSAL.NET lève une exception explicite si les deux valeurs `Instance` et `AzureCloudInstance` sont spécifiées.

Si vous ne spécifiez pas d’instance, votre application cible l’instance cloud publique Azure (instance de l’URL `https://login.onmicrosoftonline.com`).

## <a name="application-audience"></a>Audience de l’application

L’audience de connexion varie selon les besoins métier de votre application :

- Si vous êtes un développeur d’application métier (LOB), vous allez probablement produire une application à locataire unique qui sera utilisée uniquement dans votre organisation. Dans ce cas, vous devez spécifier l’organisation, soit par son ID de locataire (ID de votre instance Azure AD) ou par un nom de domaine associé à l’instance Azure AD.
- Si vous êtes un ISV, vous pouvez connecter des utilisateurs avec leurs comptes professionnels et scolaires dans toute organisation ou dans certaines organisations (application mutualisée). Mais vous pouvez également amener les utilisateurs à se connecter avec leurs comptes Microsoft personnels.

### <a name="how-to-specify-the-audience-in-your-codeconfiguration"></a>Comment spécifier l’audience dans votre code/configuration

En utilisant MSAL dans votre code, vous spécifiez l’audience à l’aide de l’une des valeurs suivantes :

- L’énumération d’audience d’autorité Azure AD.
- L’ID de locataire, qui peut être :
  - Un GUID (ID de votre instance Azure AD) pour des applications à locataire unique.
  - Un nom de domaine associé à votre instance Azure AD (également pour des applications à locataire unique).
- Un de ces espaces réservés comme un ID de locataire à la place de l’énumération d’audience d’autorité Azure AD :
  - `organizations` pour une application mutualisée.
  - `consumers` pour connecter les utilisateurs uniquement avec leurs comptes personnels.
  - `common` pour connecter des utilisateurs avec leurs comptes professionnels et scolaires, ou leurs comptes Microsoft personnels.

MSAL lève une exception significative si vous spécifiez l’audience d’autorité Azure AD et l’ID client.

Si vous ne spécifiez pas d’audience, votre application cible Azure AD et les comptes Microsoft personnels en tant qu’audience (autrement dit, elle se comporte comme si `common` était spécifié).

### <a name="effective-audience"></a>Audience effective

L’audience effective pour votre application sera la valeur minimale (s’il existe une intersection) de l’audience que vous définissez dans votre application et de l’audience spécifiée dans l’inscription d’application. En fait, l’expérience [inscriptions d’applications](https://aka.ms/appregistrations) vous permet de spécifier l’audience (les types de comptes pris en charge) pour l’application. Pour plus d’informations, consultez [Démarrage rapide : Inscrire une application avec la plateforme des identités Microsoft](quickstart-register-app.md).

Actuellement, la seule façon d’obtenir qu’une application connecte des utilisateurs disposant uniquement de comptes Microsoft personnels consiste à configurer ces deux paramètres :

- Définissez l’audience d’inscription d’application sur `Work and school accounts and personal accounts`.
- Définissez l’audience dans votre code/configuration sur `AadAuthorityAudience.PersonalMicrosoftAccount` (ou `TenantID` ="consumers").

## <a name="client-id"></a>ID client

L’ID client est l’ID (client) d’application unique qu’Azure AD attribue à votre application lors de l’inscription de l’application.

## <a name="redirect-uri"></a>URI de redirection

L’URI de redirection est l’URI auquel le fournisseur d’identité renvoie les jetons de sécurité.

### <a name="redirect-uri-for-public-client-apps"></a>URI de redirection pour les applications clientes publiques

Si vous êtes un développeur d’application cliente publique utilisant MSAL :

- Vous souhaitez utiliser `.WithDefaultRedirectUri()` dans les applications de bureau ou UWP (MSAL.NET 4.1 +). Cette méthode permet de définir la propriété URI de redirection de l’application cliente publique sur l’URI de redirection recommandé par défaut pour les applications clientes publiques.

  | Plateforme | URI de redirection |
  |--|--|
  | Application de bureau (.NET FW) | `https://login.microsoftonline.com/common/oauth2/nativeclient` |
  | UWP | valeur de `WebAuthenticationBroker.GetCurrentApplicationCallbackUri()`. Cela active l’authentification unique avec le navigateur en définissant la valeur sur le résultat de WebAuthenticationBroker.GetCurrentApplicationCallbackUri() que vous devez inscrire |
  | .NET Core | `https://localhost`. Cela permet à l’utilisateur d’utiliser le navigateur système pour l’authentification interactive, car .NET Core n’a pas d’interface utilisateur pour l’affichage Web incorporé pour le moment. |

- Vous n’avez pas besoin d’ajouter un URI de redirection si vous générez une application Xamarin Android et iOS qui ne prend pas en charge le répartiteur (l’URI de redirection est automatiquement défini sur `msal{ClientId}://auth` pour Xamarin Android et iOS)

- Vous avez besoin de configurer l’URI de redirection dans [Inscriptions d’applications](https://aka.ms/appregistrations) :

   ![URI de redirection dans Inscriptions d’applications](media/msal-client-application-configuration/redirect-uri.png)

Vous pouvez remplacer l’URI de redirection à l’aide de la propriété `RedirectUri` (par exemple, si vous utilisez des répartiteurs). Voici quelques exemples d’URI de redirection pour ce scénario :

- `RedirectUriOnAndroid` = "msauth-5a434691-ccb2-4fd1-b97b-b64bcfbc03fc://com.microsoft.identity.client.sample";
- `RedirectUriOnIos` = $"msauth.{Bundle.ID}://auth";

Pour plus d’informations sur iOS, voir [Migrer des applications utilisant Microsoft Authenticator d’iOS d’ADAL.NET vers MSAL.NET](msal-net-migration-ios-broker.md) et [Utilisation du répartiteur sur iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Leveraging-the-broker-on-iOS).
Pour plus d’informations sur Android, voir [Authentification répartie dans Android](msal-android-single-sign-on.md).

### <a name="redirect-uri-for-confidential-client-apps"></a>URI de redirection pour les applications clientes confidentielles

Pour les applications web, l’URI de redirection (ou URL de réponse) est l’URI qu’Azure AD utilise pour renvoyer le jeton à l’application. Cet URI peut être l'URL de l'application web/API Web si l'application confidentielle correspond à l'une des applications suivantes. L’URI de redirection doit être inscrit dans l’inscription d’application. Cette inscription est particulièrement importante lorsque vous déployez une application que vous avez initialement testée localement. Vous devez ensuite ajouter l’URL de réponse de l’application déployée dans le portail d’inscription d’application.

Pour les applications démon, vous n’avez pas besoin de spécifier un URI de redirection.

## <a name="client-secret"></a>Clé secrète client

Cette option spécifie la clé secrète client pour l’application cliente confidentielle. Ce secret (mot de passe) est fourni par le portail de l’inscription d’application ou, pour Azure AD, fourni lors de l’inscription de l’application avec PowerShell AzureAD, PowerShell AzureRM ou Azure CLI.

## <a name="logging"></a>Journalisation

Les autres options de configuration activent la journalisation et la résolution des problèmes. Pour plus d’informations sur la façon de les utiliser, voir l’article [Journalisation](msal-logging.md).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [l’instanciation d’applications clientes à l’aide de MSAL.NET](msal-net-initializing-client-applications.md) et [l’instanciation d’applications clientes à l’aide de MSAL.js](msal-js-initializing-client-applications.md).
