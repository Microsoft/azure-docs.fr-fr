---
title: Applications natives dans Azure Active Directory
description: Décrit les applications natives et les principes de base sur le flux de protocole, l’inscription et l’expiration du jeton pour ce type d’application.
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
ms.openlocfilehash: 9ecf711f5442b6f21de53d2735ad1c94d7cb6223
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80154795"
---
# <a name="native-apps"></a>Applications natives

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Les applications natives sont des applications qui appellent une API web de la part d’un utilisateur. Ce scénario repose sur le type d’octroi de code d’autorisation OAuth 2.0 avec un client public, comme décrit à la section 4.1 de la [spécification OAuth 2.0](https://tools.ietf.org/html/rfc6749). L’application native obtient un jeton d’accès pour l’utilisateur à l’aide du protocole OAuth 2.0. Ce jeton d’accès est ensuite envoyé via la demande à l’API web, qui autorise l’utilisateur et renvoie la ressource souhaitée.

## <a name="diagram"></a>Diagramme

![Diagramme Application native vers API web](./media/authentication-scenarios/native-app-to-web-api.png)

## <a name="protocol-flow"></a>Flux de protocole

Si vous utilisez les bibliothèques d’authentification AD, la plupart des détails de protocole décrits ci-dessous sont gérés pour vous, notamment les fenêtres contextuelles du navigateur, la mise en cache des jetons et la gestion des jetons d’actualisation.

1. À l’aide d’une fenêtre contextuelle du navigateur, l’application native envoie une demande au point de terminaison d’autorisation d’Azure AD. Cette demande comprend l’ID d’application et l’URI de redirection de l’application native tels qu’ils figurent dans le portail Azure, et l’URI d’ID d’application de l’API web. Si l’utilisateur ne s’est pas déjà connecté, il est invité à nouveau à se connecter
1. Azure AD authentifie l’utilisateur. S’il s’agit d’une application mutualisée et si un consentement est requis pour utiliser l’application, l’utilisateur doit donner son consentement, s’il ne l’a pas déjà fait. Une fois le consentement donné et l’authentification réussie, Azure AD renvoie une réponse de code d’autorisation à l’URI de redirection de l’application cliente.
1. Quand Azure AD renvoie une réponse de code d’autorisation à l’URI de redirection, l’application cliente arrête l’interaction du navigateur et extrait le code d’autorisation de la réponse. À l’aide de ce code d’autorisation, l’application cliente envoie une demande au point de terminaison de jeton d’Azure AD. Celle-ci comprend le code d’autorisation, des détails sur l’application cliente (ID d’application et URI de redirection) et la ressource souhaitée (URI ID d’application de l’API web).
1. Le code d’autorisation et les informations sur l’application cliente et l’API web sont validés par Azure AD. Si la validation réussit, Azure AD renvoie deux jetons : un jeton d’accès JWT et un jeton d’actualisation JWT. Azure AD renvoie également des informations de base sur l’utilisateur, telles que son nom d’affichage et son ID client.
1. Sur HTTPS, l’application cliente utilise le jeton d’accès JWT renvoyé pour ajouter la chaîne JWT avec la mention « porteur » dans l’en-tête d’autorisation de la demande adressée à l’API web. L’API web valide ensuite le jeton JWT et, si la validation réussit, renvoie la ressource souhaitée.
1. Quand le jeton d’accès arrive à expiration, l’application cliente reçoit une erreur indiquant que l’utilisateur doit s’authentifier à nouveau. Si l’application dispose d’un jeton d’actualisation valide, celui-ci peut être utilisé pour obtenir un nouveau jeton d’accès sans demander à l’utilisateur de se connecter à nouveau. Si le jeton d’actualisation arrive à expiration, l’application doit interactivement authentifier l’utilisateur une nouvelle fois.

> [!NOTE]
> Le jeton d’actualisation émis par Azure AD peut être utilisé pour accéder à plusieurs ressources. Par exemple, si une application cliente est autorisée à appeler deux API web, le jeton d’actualisation peut être utilisé pour obtenir un jeton d’accès à l’autre API web également.

## <a name="code-samples"></a>Exemples de code

Consultez les exemples de code pour les scénarios du type application native vers API web. Et consultez régulièrement cette page à laquelle nous ajoutons fréquemment de nouveaux exemples. [Application native vers API web](sample-v1-code.md#desktop-and-mobile-public-client-applications-calling-microsoft-graph-or-a-web-api).

## <a name="app-registration"></a>Inscription d'application

Pour inscrire une application auprès du point de terminaison Azure AD v1.0, consultez [Inscrire une application](../develop/quickstart-register-app.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

* Application à client unique : l’application native et l’API web doivent être inscrites dans le même répertoire dans Azure AD. L’API web peut être configurée pour exposer un ensemble d’autorisations utilisées pour limiter l’accès de l’application native à ses ressources. L’application cliente sélectionne ensuite les autorisations souhaitées dans le menu déroulant « Autorisations pour d’autres applications » du portail Azure.
* Application mutualisée : premièrement, l’application native est toujours inscrite dans l’annuaire du développeur ou de l’éditeur. Deuxièmement, l’application native est configurée pour indiquer les autorisations dont elle a besoin pour fonctionner. Cette liste d’autorisations requises s’affiche dans une boîte de dialogue quand un utilisateur ou un administrateur de l’annuaire de destination donne son consentement à l’application, ce qui la met à disposition de son organisation. Certaines applications nécessitent uniquement des autorisations au niveau utilisateur pour lesquelles tous les utilisateurs de l’organisation peuvent donner leur consentement. D’autres nécessitent des autorisations administrateur, pour lesquelles un utilisateur de l’organisation ne peut pas donner son consentement. Seul un administrateur d’annuaires peut donner son consentement aux applications qui requièrent des autorisations de ce niveau. Quand un utilisateur ou un administrateur donne son consentement, seule l’API web est inscrite dans son annuaire. 

## <a name="token-expiration"></a>Expiration du jeton

Quand l’application native utilise son code d’autorisation pour obtenir un jeton d’accès JWT, elle reçoit également un jeton d’actualisation JWT. À l’expiration du jeton d’accès, le jeton d’actualisation peut être utilisé pour authentifier l’utilisateur à nouveau sans lui demander de se connecter une nouvelle fois. Ce jeton d’actualisation est ensuite utilisé pour authentifier l’utilisateur, ce qui donne lieu à la création d’un nouveau jeton d’accès et d’un nouveau jeton d’actualisation.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les autres [types d’applications et scénarios](app-types.md)
- En savoir plus sur les [principes de base de l’authentification](v1-authentication-scenarios.md) Azure AD
