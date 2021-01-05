---
title: Applications service à service dans Azure Active Directory
description: Décrit les applications service à service, ainsi que les principes de base sur le flux de protocole, l’inscription et l’expiration du jeton pour ce type d’application.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.workload: identity
ms.topic: conceptual
ms.date: 11/20/2019
ms.author: ryanwi
ms.reviewer: saeeda, jmprieur
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 179034533d90dbbb6ca362fc6f72996f32873729
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80154761"
---
# <a name="service-to-service-apps"></a>Applications service à service

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Les applications service à service peuvent être une application démon ou serveur ayant besoin d’obtenir des ressources d’une API web. Deux sous-scénarios s’appliquent à cette section :

- Un démon qui doit appeler une API web, reposant sur le type d’autorisation d’informations d’identification clientes OAuth 2.0

    Dans ce scénario, il est important de comprendre quelques éléments. Tout d’abord, l’intervention de l’utilisateur n’est pas possible avec une application démon, qui requiert que l’application ait sa propre identité. Le traitement par lots et un service de système d’exploitation exécuté en arrière-plan sont des exemples d’application démon. Ce type d’application demande un jeton d’accès en utilisant son identité d’application et en fournissant à Azure AD son ID d’application, ses informations d’identification (mot de passe ou certificat) et l’URI ID d’application. Une fois l’authentification réussie, le démon reçoit un jeton d’accès d’Azure AD. Celui-ci est ensuite utilisé pour appeler l’API web.

- Une application serveur (par exemple, une API web) qui doit appeler une API web, reposant sur une spécification préliminaire On-Behalf-Of OAuth 2.0

    Dans ce scénario, imaginez qu’un utilisateur se soit authentifié auprès d’une application native et que celle-ci doive appeler une API web. Azure AD émet un jeton d’accès JWT pour appeler l’API web. Si l’API web doit appeler une autre API web en aval, elle peut utiliser le flux « au nom de » pour déléguer l’identité de l’utilisateur et s’authentifier auprès de l’API web de deuxième niveau.

## <a name="diagram"></a>Diagramme

![Diagramme Application démon ou serveur vers API web](./media/authentication-scenarios/daemon-server-app-to-web-api.png)

## <a name="protocol-flow"></a>Flux de protocole

### <a name="application-identity-with-oauth-20-client-credentials-grant"></a>Identité d’application avec octroi d’informations d’identification client OAuth 2.0

1. Tout d’abord, l’application serveur doit s’authentifier auprès d’Azure AD avec sa propre identité, sans intervention humaine par le biais d’une boîte de dialogue interactive d’ouverture de session par exemple. Elle envoie une demande au point de terminaison de jeton du service Azure AD, avec les informations d’identification, l’ID d’application et un URI ID d’application.
1. Azure AD authentifie l’application et renvoie un jeton d’accès JWT, qui est utilisé pour appeler l’API web.
1. Sur HTTPS, l’application web utilise le jeton d’accès JWT renvoyé pour ajouter la chaîne JWT avec la mention « Bearer » (Porteur) dans l’en-tête « Authorization » (Autorisation) de la demande adressée à l’API web. L’API web valide ensuite le jeton JWT et, si la validation réussit, renvoie la ressource souhaitée.

### <a name="delegated-user-identity-with-oauth-20-on-behalf-of-draft-specification"></a>Identité d’utilisateur délégué avec spécification préliminaire « Au nom de » OAuth 2.0

Le flux présenté ci-après part du principe qu’un utilisateur a été authentifié auprès d’une autre application (telle qu’une application native) et que son identité d’utilisateur a été utilisée pour obtenir un jeton d’accès à l’API web de premier niveau.

1. L’application native envoie le jeton d’accès à l’API web de premier niveau.
1. L’API web de premier niveau envoie une demande au point de terminaison de jeton du service Azure AD, en fournissant son ID d’application et ses informations d’identification, ainsi que le jeton d’accès de l’utilisateur. La demande est également envoyée avec un paramètre on_behalf_of, qui indique que l’API web demande de nouveaux jetons pour appeler une API web en aval au nom de l’utilisateur d’origine.
1. Azure AD vérifie que l’API web de premier niveau est autorisée à accéder à l’API web de deuxième niveau et valide la demande en renvoyant un jeton d’accès JWT et un jeton d’actualisation JWT à l’API web de premier niveau.
1. Sur HTTPS, l’API web de premier niveau appelle ensuite l’API web de deuxième niveau en ajoutant la chaîne de jeton dans l’en-tête d’autorisation de la demande. L’API web de premier niveau peut continuer à appeler l’API web de deuxième niveau tant que le jeton d’accès et les jetons d’actualisation sont valides.

## <a name="code-samples"></a>Exemples de code

Consultez les exemples de code pour les scénarios du type application démon ou serveur vers API web : [Application serveur ou démon vers API web](sample-v1-code.md#daemon-applications-accessing-web-apis-with-the-applications-identity)

## <a name="app-registration"></a>Inscription d'application

* Application avec locataire unique : pour les identités d’application et les identités d’utilisateur délégué, l’application démon ou serveur doit être inscrite dans le même répertoire dans Azure AD. L’API web peut être configurée pour exposer un ensemble d’autorisations utilisées pour limiter l’accès du démon ou du serveur à ses ressources. Si un type d’identité utilisateur délégué est utilisé, l’application serveur doit sélectionner les autorisations souhaitées. Dans la page **Autorisation d’API** de l’inscription d’application, après avoir sélectionné **Ajouter une autorisation** et choisi la famille d’API, choisissez **Autorisations déléguées**, puis sélectionnez vos autorisations. Cette étape n’est pas requise si une identité d’application est utilisée.
* Application mutualisée : tout d’abord, l’application démon ou serveur est configurée pour indiquer les autorisations dont elle a besoin pour fonctionner. Cette liste d’autorisations requises s’affiche dans une boîte de dialogue quand un utilisateur ou un administrateur de l’annuaire de destination donne son consentement à l’application, ce qui la met à disposition de son organisation. Certaines applications nécessitent uniquement des autorisations au niveau utilisateur pour lesquelles tous les utilisateurs de l’organisation peuvent donner leur consentement. D’autres nécessitent des autorisations administrateur, pour lesquelles un utilisateur de l’organisation ne peut pas donner son consentement. Seul un administrateur d’annuaires peut donner son consentement aux applications qui requièrent des autorisations de ce niveau. Quand un utilisateur ou un administrateur donne son consentement, les deux API web sont inscrites dans son annuaire.

## <a name="token-expiration"></a>Expiration du jeton

Quand la première application utilise son code d’autorisation pour obtenir un jeton d’accès JWT, elle reçoit également un jeton d’actualisation JWT. À l’expiration du jeton d’accès , le jeton d’actualisation peut être utilisé pour authentifier l’utilisateur à nouveau sans lui demander ses informations d’identification. Ce jeton d’actualisation est ensuite utilisé pour authentifier l’utilisateur, ce qui donne lieu à la création d’un nouveau jeton d’accès et d’un nouveau jeton d’actualisation.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les autres [types d’applications et scénarios](app-types.md)
- En savoir plus sur les [principes de base de l’authentification](v1-authentication-scenarios.md) Azure AD
