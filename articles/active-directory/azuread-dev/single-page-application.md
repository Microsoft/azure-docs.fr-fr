---
title: Applications monopages dans Azure Active Directory
description: Décrit les applications monopages (SPA), ainsi que les principes de base sur le flux de protocole, l’inscription et l’expiration du jeton pour ce type d’application.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.workload: identity
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ryanwi
ms.reviewer: saeeda, jmprieur
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: adf3c5b5cd40a9ea3f07ba9c92cfc4544ca60f1e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80154744"
---
# <a name="single-page-applications"></a>Applications monopages

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

En règle générale, la structure des applications monopages se compose d'une couche (frontale) de présentation JavaScript qui s’exécute dans le navigateur et d'un back end d’API web qui s’exécute sur un serveur et implémente la logique métier de l’application. Pour en savoir plus sur l’octroi d’autorisation implicite, et vous aider à déterminer ce qui correspond le mieux à votre scénario d’application, consultez [Comprendre le flux d’octroi implicite OAuth2 dans Azure Active Directory (AD)](v1-oauth2-implicit-grant-flow.md).

Dans ce scénario, quand l’utilisateur se connecte, le JavaScript frontal utilise [Active Directory Authentication Library pour JavaScript (ADAL.JS)](https://github.com/AzureAD/azure-activedirectory-library-for-js) et l’octroi d’autorisation implicite pour obtenir un jeton d’ID (id_token) d’Azure AD. Le jeton est mis en cache, et le client l’attache à la demande en tant que jeton porteur lors de l’appel des composants principaux de son API web, qui sont sécurisés à l’aide de l’intergiciel OWIN.

## <a name="diagram"></a>Diagramme

![Diagramme d’application monopage](./media/authentication-scenarios/single-page-app.png)

## <a name="protocol-flow"></a>Flux de protocole

1. L’utilisateur accède à l’application web.
1. L’application renvoie le JavaScript frontal (couche présentation) au navigateur.
1. L’utilisateur établit la connexion, par exemple en cliquant sur un lien de connexion. Le navigateur envoie une commande GET au point de terminaison d’autorisation Azure AD pour demander un jeton d’ID. Les paramètres de requête de cette demande comprennent l’ID d’application et l’URL de réponse.
1. Azure AD valide l’URL de réponse par rapport à celle enregistrée et configurée dans le portail Azure.
1. L’utilisateur se connecte sur la page de connexion.
1. Si l’authentification réussit, Azure AD crée un jeton d’ID et le renvoie à l’URL de réponse de l’application sous forme de fragment d’URL (#). Pour une application de production, cette URL de réponse doit être au format HTTPS. Le jeton renvoyé inclut des revendications sur l’utilisateur et Azure AD dont l’application a besoin pour valider le jeton.
1. Le code client JavaScript exécuté dans le navigateur extrait le jeton de la réponse afin de l’utiliser pour la sécurisation des appels vers les composants principaux de l’API web de l’application.
1. Le navigateur appelle les composants principaux de l’API web de l’application, avec le jeton d’ID dans l’en-tête d’autorisation. Le service d’authentification Azure AD émet un jeton d’ID qui peut être utilisé comme un jeton du porteur si la ressource est identique à l’ID du client (dans ce cas, cela est vrai car l’API web est le back-end de l’application).

## <a name="code-samples"></a>Exemples de code

Consultez les [exemples de code pour les scénarios d’application monopage](sample-v1-code.md#single-page-applications). Veillez à consulter régulièrement cette page à laquelle nous ajoutons fréquemment de nouveaux exemples.

## <a name="app-registration"></a>Inscription d'application

* Locataire unique : si vous créez une application uniquement pour votre organisation, vous devez l’inscrire dans l’annuaire de votre entreprise à l’aide du portail Azure.
* Application mutualisée : si vous créez une application qui peut être utilisée par des utilisateurs externes, vous devez l’inscrire dans l'annuaire de votre entreprise, mais également dans celui de chaque organisation qui utilise l’application. Afin de mettre votre application à disposition dans ces annuaires, vous pouvez inclure pour vos clients un processus d’inscription qui leur permet de donner leur consentement à votre application. Quand ils s’inscrivent auprès de votre application, une boîte de dialogue contenant les autorisations requises par l’application s’affiche, et ils ont ensuite la possibilité de donner leur consentement. Selon les autorisations requises, il est possible qu’un administrateur de l’autre organisation doive donner le consentement. Une fois le consentement donné par l’utilisateur ou l’administrateur, l’application est inscrite dans l’annuaire de l’organisation de l’utilisateur ou de l’administrateur.

Après avoir inscrit l’application, vous devez la configurer pour qu’elle utilise le protocole d’octroi implicite OAuth 2.0. Par défaut, ce protocole est désactivé pour les applications. Pour activer le protocole d’octroi implicite OAuth2 pour votre application, modifiez son manifeste d’application à partir du portail Azure et définissez la valeur « oauth2AllowImplicitFlow » sur true. Pour plus d’informations, consultez l’article [Manifeste d’application](../develop/reference-app-manifest.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

## <a name="token-expiration"></a>Expiration du jeton

L’utilisation d’ADAL.js est utile pour ce qui suit :

* actualiser un jeton expiré ;
* demander un jeton d’accès pour appeler une ressource d’API web.

Après une authentification réussie, Azure AD écrit un cookie dans le navigateur de l’utilisateur pour établir une session. Notez que la session existe entre l’utilisateur et Azure AD (pas entre l’utilisateur et l’application web). Lorsqu’un jeton arrive à expiration, ADAL.js utilise cette session pour obtenir un autre jeton en mode silencieux. ADAL.js utilise un iFrame masqué pour l’envoi et la réception de la requête à l’aide du protocole d’octroi implicite OAuth. ADAL.js peut également utiliser ce mécanisme afin d’obtenir des jetons d’accès en mode silencieux pour d’autres ressources d’API web que l’application appelle, à condition que ces ressources prennent en charge le partage des ressources cross-origin (CORS) et soient inscrites dans l’annuaire de l’utilisateur, et que l’utilisateur ait donné le consentement requis au moment de la connexion.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [types d’applications et scénarios](app-types.md)
* En savoir plus sur les [principes de base de l’authentification](v1-authentication-scenarios.md) Azure AD
