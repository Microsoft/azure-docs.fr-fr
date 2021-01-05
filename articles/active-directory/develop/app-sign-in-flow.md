---
title: Flux de connexion des applications avec la Plateforme d’identités Microsoft | Azure
titleSuffix: Microsoft identity platform
description: En savoir plus sur le flux de connexion des applications web, de bureau et mobiles dans la Plateforme d’identités Microsoft (v2.0).
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/18/2020
ms.author: ryanwi
ms.reviewer: jmprieur, saeeda, sureshja, hirsin
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started
ms.openlocfilehash: af5b27dc85a276c731a61135ab59ab81f5aaf3c2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83772197"
---
# <a name="app-sign-in-flow-with-microsoft-identity-platform"></a>Flux de connexion des applications avec la Plateforme d’identités Microsoft

Cette rubrique décrit le processus de connexion de base pour les applications web, de bureau et mobiles à l’aide de la Plateforme d’identités Microsoft. Pour découvrir les scénarios de connexion pris en charge par la Plateforme d’identités Microsoft, consultez [Flux d’authentification et scénarios d’application](authentication-flows-app-scenarios.md).

## <a name="web-app-sign-in-flow"></a>Flux de connexion des applications web

Quand un utilisateur accède par l’intermédiaire du navigateur à une application web, voici ce qui se produit :

* L’application web détermine si l’utilisateur est authentifié.
* Si l’utilisateur ne l’est pas, l’application web délègue à Azure AD la connexion de l’utilisateur. Cette connexion doit être conforme à la stratégie de l’organisation, ce qui peut impliquer de demander à l’utilisateur d’entrer ses informations d’identification, en utilisant une [authentification multifacteur](../authentication/concept-mfa-howitworks.md) (parfois appelée authentification à deux facteurs ou 2FA), ou de n’utiliser aucun mot de passe (par exemple, en se recourant à Windows Hello).
* L’utilisateur est invité à donner son consentement pour l’accès dont l’application cliente a besoin. C’est la raison pour laquelle les applications clientes doivent être inscrites auprès d’Azure AD, afin que la Plateforme d’identités Microsoft puisse remettre des jetons représentant l’accès auquel l’utilisateur a consenti.

Lorsque l’utilisateur est bien authentifié :

* La Plateforme d’identités Microsoft envoie un jeton à l’application web.
* Un cookie est enregistré, associé au domaine d’Azure AD ; il contient l’identité de l’utilisateur dans l’emplacement du navigateur réservé aux cookies. La prochaine fois qu’une application utilise le navigateur pour accéder au point de terminaison d’autorisation de la Plateforme d’identités Microsoft, le navigateur présente le cookie afin que l’utilisateur n’ait pas à se reconnecter. C’est également de cette façon que l’authentification unique est effectuée. Le cookie est généré par Azure AD et ne peut être compris que par Azure AD.
* L’application web valide ensuite le jeton. Si la validation réussit, l’application web affiche la page protégée et enregistre un cookie de session dans l’emplacement du navigateur réservé aux cookies. Lorsque l’utilisateur accède à une autre page, l’application web sait que l’utilisateur est authentifié conformément au cookie de session.

Le schéma de séquence suivant résume cette interaction :

![Processus d’authentification d’une application web](media/authentication-scenarios/web-app-how-it-appears-to-be.png)

### <a name="how-a-web-app-determines-if-the-user-is-authenticated"></a>Comment une application web détermine si l’utilisateur est authentifié

Les développeurs d’applications web peuvent indiquer si toutes les pages, ou uniquement certaines d’entre elles, nécessitent une authentification. Par exemple, dans ASP.NET/ASP.NET Core, cette opération s’effectue en ajoutant l’attribut `[Authorize]` aux actions du contrôleur.

Cet attribut force ASP.NET à vérifier la présence d’un cookie de session contenant l’identité de l’utilisateur. Si aucun cookie n’est présent, ASP.NET redirige l’authentification vers le fournisseur d’identité spécifié. Si le fournisseur d’identité est Azure AD, l’application web redirige l’authentification vers `https://login.microsoftonline.com`, qui affiche une boîte de dialogue de connexion.

### <a name="how-a-web-app-delegates-sign-in-to-microsoft-identity-platform-and-obtains-a-token"></a>Comment une application web délègue la connexion à la Plateforme d’identités Microsoft et obtient un jeton

L’authentification de l’utilisateur s’effectue via le navigateur. Le protocole OpenID utilise des messages de protocole HTTP standard.

* L’application web envoie en requête HTTP 302 (redirection) au navigateur pour utiliser la Plateforme d’identités Microsoft.
* Lorsque l’utilisateur est authentifié, la Plateforme d’identités Microsoft envoie le jeton à l’application web en utilisant une redirection via le navigateur.
* La redirection est fournie par l’application web sous la forme d’un URI de redirection. Cet URI de redirection est inscrit auprès de l’objet d’application Azure AD. Il peut y avoir plusieurs URI de redirection, car l’application peut être déployée sur plusieurs URL. Par conséquent, l’application web doit également spécifier l’URI de redirection à utiliser.
* Azure AD vérifie que l’URI de redirection envoyé par l’application web est l’un des URI de redirection inscrits pour l’application.

## <a name="desktop-and-mobile-app-sign-in-flow"></a>Flux de connexion d’application mobile et de bureau

Le flux décrit ci-dessus s’applique, avec de légères différences, aux applications mobiles et de bureau.

Les applications mobiles et de bureau peuvent utiliser un contrôle web incorporé, ou un navigateur système, pour l’authentification. Le diagramme suivant montre comment une application mobile ou de bureau utilise la bibliothèque d’authentification Microsoft (MSAL) pour acquérir des jetons d’accès et appeler des API web.

![Application de bureau - son apparence](media/authentication-scenarios/desktop-app-how-it-appears-to-be.png)

MSAL utilise un navigateur pour obtenir des jetons. Comme pour les applications web, l’authentification est déléguée à la Plateforme d’identités Microsoft.

Étant donné qu’Azure AD enregistre le même cookie d’identité dans le navigateur que pour les applications web, si l’application native ou mobile utilise le navigateur système, l’authentification unique est immédiatement obtenue avec l’application web correspondante.

Par défaut, MSAL utilise le navigateur du système. Les applications de bureau .NET Framework font exception, car elles utilisent un contrôle incorporé est utilisé pour offrir une expérience utilisateur plus intégrée.

## <a name="next-steps"></a>Étapes suivantes

Pour d’autres rubriques traitant des concepts de base de l’authentification et de l’autorisation :

* Pour en savoir plus sur les concepts de base de l’authentification et de l’autorisation dans la Plateforme d’identités Microsoft, consultez [Authentification ou autorisation](authentication-vs-authorization.md).
* Pour savoir comment les jetons d’accès, les jetons d’actualisation et les jetons d’ID sont utilisés dans l’authentification et l’autorisation, consultez [Jetons de sécurité](security-tokens.md).
* Pour découvrir le processus d’inscription de votre application afin qu’elle puisse s’intégrer avec la Plateforme d’identités Microsoft, consultez [Modèle d’application](application-model.md).

Pour en savoir plus sur le flux de connexion des applications :

* Pour en savoir plus sur les autres scénarios d’authentification des utilisateurs pris en charge par la Plateforme d’identités Microsoft, consultez [Flux d’authentification et scénarios d’application](authentication-flows-app-scenarios.md).
* Consultez [Bibliothèques MSAL](msal-overview.md) pour en apprendre davantage sur les bibliothèques Microsoft qui vous aident à développer des applications fonctionnant avec des comptes Microsoft, des comptes Azure AD et des utilisateurs Azure AD B2C, tous réunis dans un modèle de programmation rationalisé.
