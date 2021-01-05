---
title: Vue d’ensemble de la bibliothèque d’authentification Microsoft Identity Web
titleSuffix: Microsoft identity platform
description: Découvrez Microsoft Identity Web, une bibliothèque d’authentification et d’autorisation pour les applications ASP.NET Core qui s’intègrent à Azure Active Directory, Azure AD B2C et Microsoft Graph, ainsi qu’à d’autres API web.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/09/2020
ms.author: jmprieur
ms.reviewer: marsma
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: b82e300128a41f8315132e1ff93af33c853edb15
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96173513"
---
# <a name="microsoft-identity-web-authentication-library"></a>Bibliothèque d’authentification Microsoft Identity Web

Microsoft Identity Web est un ensemble de bibliothèques ASP.NET Core qui simplifie l’ajout de la prise en charge de l’authentification et de l’autorisation aux applications web et aux API web qui s’intègrent à la plateforme d’identités Microsoft. Il fournit une couche pratique d’API à surface unique qui associe ASP.NET Core, son intergiciel d’authentification et la [bibliothèque d’authentification Microsoft (MSAL) pour .NET](https://github.com/azuread/microsoft-authentication-library-for-dotnet).

## <a name="supported-application-scenarios"></a>Scénarios d’application pris en charge

Si vous générez des applications web ASP.NET Core ou des API web et que vous souhaitez utiliser Azure Active Directory (Azure AD) ou Azure AD B2C pour la gestion des identités et des accès (IAM), nous vous recommandons d’utiliser Microsoft Identity Web pour tous les scénarios suivants :

- [Application web qui connecte les utilisateurs](scenario-web-app-sign-user-overview.md)
- [Application web qui connecte des utilisateurs et appelle une API web en leur nom](scenario-web-app-call-api-overview.md)
- [API web protégée accessible uniquement aux utilisateurs authentifiés](scenario-protected-web-api-overview.md)
- [API web protégée appelant une autre API web en aval pour le compte de l’utilisateur connecté](scenario-web-api-call-api-overview.md)

## <a name="get-the-library"></a>Obtenir la bibliothèque

Vous pouvez obtenir Microsoft Identity Web à partir de [NuGet](#nuget), des [modèles de projet .NET Core](#project-templates) et [GitHub](#github).

#### <a name="nuget"></a>NuGet

Microsoft Identity Web est disponible sur NuGet comme ensemble de packages qui fournissent des fonctionnalités modulaires en fonction des besoins de votre application. Utilisez la commande `dotnet add` de l’interface CLI .NET ou le **gestionnaire de package NuGet** de Visual Studio pour installer les packages appropriés pour votre projet :

- [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web) : package principal. Requis par toutes les applications qui utilisent Microsoft Identity Web.
- [Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI) : facultatif. Ajoute l’interface utilisateur pour la connexion et la déconnexion des utilisateurs et un contrôleur associé pour les applications web.
- [Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph) : facultatif. Fournit une interaction simplifiée avec l’API Microsoft Graph.
- [Microsoft.Identity.Web.MicrosoftGraphBeta](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraphBeta) : facultatif. Fournit une interaction simplifiée avec le [point de terminaison de la version bêta](/graph/api/overview?view=graph-rest-beta&preserve-view=true) de l’API Microsoft Graph.

#### <a name="project-templates"></a>Modèles de projet

Les modèles de projet Microsoft Identity Web sont inclus dans .NET 5.0 et peuvent être téléchargés pour les projets ASP.NET Core 3.1.

Si vous utilisez ASP.NET Core 3.1, installez les modèles avec l’interface CLI .NET :

```dotnetcli
dotnet new --install Microsoft.Identity.Web.ProjectTemplates::1.0.0
```

Le diagramme suivant montre une vue d’ensemble des types d’applications prises en charge et de leurs arguments pertinents :

:::image type="content" source="media/microsoft-identity-web-overview/diagram-microsoft-identity-web-templates.png" lightbox="media/microsoft-identity-web-overview/diagram-microsoft-identity-web-templates.png" alt-text="Diagramme des modèles de projet CLI .NET disponibles pour Microsoft Identity Web":::
<br /><sup><b>*</b></sup> `MultiOrg` n’est pas pris en charge avec `webapi2`, mais peut être activé dans *appsettings.json* en définissant le locataire sur `common` ou `organizations`.
<br /><sup><b>**</b></sup> `--calls-graph` n’est pas pris en charge pour Azure AD B2C.

Cet exemple de commande CLI .NET, extrait de notre [tutoriel sur Blazor Server](tutorial-blazor-server.md), génère un nouveau projet Blazor Server qui comprend les packages appropriés et le code de démarrage (valeurs d’espace réservé affichées) :

```dotnetcli
dotnet new blazorserver2 --auth SingleOrg --calls-graph --client-id "00000000-0000-0000-0000-000000000000" --tenant-id "11111111-1111-1111-1111-111111111111" --output my-blazor-app
```

#### <a name="github"></a>GitHub

Microsoft Identity Web est un projet open source hébergé sur GitHub : <a href="https://github.com/AzureAD/microsoft-identity-web" target="_blank">AzureAD/microsoft-identity-web<span class="docon docon-navigate-external x-hidden-focus"></span></a>

Le [wiki du référentiel](https://github.com/AzureAD/microsoft-identity-web/wiki) contient de la documentation supplémentaire. Si vous avez besoin d’aide ou si vous découvrez un bogue, vous pouvez [soumettre le problème](https://github.com/AzureAD/microsoft-identity-web/issues).

## <a name="features"></a>Fonctionnalités

Microsoft Identity Web comprend plusieurs fonctionnalités qui ne sont pas fournies si vous utilisez les modèles de projet ASP.NET 3.1 par défaut.

| Fonctionnalité                                                                                  | ASP.NET Core 3.1                                                     | Microsoft Identity Web                                                                                  |
|------------------------------------------------------------------------------------------|----------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| [Connexion des utilisateurs](scenario-web-app-sign-user-app-configuration.md) dans les applications web             | <li>Comptes professionnels ou scolaires<li>Identités sociales (avec Azure AD B2C) | <li>Comptes professionnels ou scolaires<li>Comptes Microsoft personnels<li>Identités sociales (avec Azure AD B2C)     |
| [Protection des API web](scenario-protected-web-api-app-configuration.md#microsoftidentityweb) | <li>Comptes professionnels ou scolaires<li>Identités sociales (avec Azure AD B2C) | <li>Comptes professionnels ou scolaires<li>Comptes Microsoft personnels<li>Identités sociales (avec Azure AD B2C)     |
| Validation de l’émetteur dans les applications mutualisées                                                   | Non                                                                   | Oui, pour [tous les clouds](authentication-national-cloud.md) et [Azure AD B2C](../../active-directory-b2c/index.yml) |
| Application/API web [appelle Microsoft Graph][scenario-api-call-graph]                             | Non                                                                   | Oui                                                                                                     |
| Application/API Web [appelle une API web][scenario-api-call-api]                                       | Non                                                                   | Oui                                                                                                     |
| Prise en charge des informations d’identification de certificat                                                         | Non                                                                   | Oui, y compris Azure Key Vault                                                                          |
| Prise en charge du consentement incrémentiel et de l’accès conditionnel dans les applications web                           | Non                                                                   | Oui, dans MVC, les pages Razor et Blazor                                                                    |
| Certificats de chiffrement de jeton dans les API web                                                | Non                                                                   | Oui                                                                                                     |
| [Validation des étendues/rôles d’application][scenario-api-validation] dans les API web                        | Non                                                                   | Oui                                                                                                     |
| Génération de l’en-tête `WWW-Authenticate` dans les API web                                         | Non                                                                   | Oui                                                                                                     |

## <a name="next-steps"></a>Étapes suivantes

Pour voir Microsoft Identity Web en action, essayez notre tutoriel sur Blazor Server :

[Tutoriel : Créer une application Blazor Server qui utilise la plateforme d’identité Microsoft pour l’authentification](tutorial-blazor-server.md)

Le wiki Microsoft Identity Web sur GitHub contient une documentation de référence complète pour différents aspects de la bibliothèque. Par exemple, les références relatives à l’utilisation du certificat, au consentement incrémentiel et à l’accès conditionnel sont accessibles ici :

- <a href="https://github.com/AzureAD/microsoft-identity-web/wiki/Using-certificates" target="_blank">Using certificates with Microsoft.Identity.Web (Utilisation de certificats avec Microsoft.Identity.Web)<span class="docon docon-navigate-external x-hidden-focus"></span></a> [GitHub]
- <a href="https://github.com/AzureAD/microsoft-identity-web/wiki/Managing-incremental-consent-and-conditional-access" target="_blank">Incremental consent and conditional access (Consentement incrémentiel et accès conditionnel)<span class="docon docon-navigate-external x-hidden-focus"></span></a> [GitHub]

<!-- LINKS -->
<!--  [miw-certs]: microsoft-identity-web-certificates.md  -->
<!--  [miw-certs-decrypt]: microsoft-identity-web-certificates.md#decryption-certificates  -->
<!--  [miw-inc-consent-ca-header]: microsoft-identity-web-consent-conditional-access.md#handling-incremental-consent-or-conditional-access-in-web-apis  -->
<!--  [miw-inc-consent-ca]: microsoft-identity-web-consent-conditional-access.md  -->
[scenario-api-call-api]: scenario-web-api-call-api-call-api.md#option-1-call-microsoft-graph-with-the-sdk [scenario-api-call-graph]: scenario-web-api-call-api-call-api.md#option-1-call-microsoft-graph-with-the-sdk [scenario-api-validation]: scenario-protected-web-api-verification-scope-app-roles.md