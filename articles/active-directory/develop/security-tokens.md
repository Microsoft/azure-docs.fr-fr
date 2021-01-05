---
title: Jetons de sécurité | Azure
titleSuffix: Microsoft identity platform
description: Découvrez les principes de base des jetons de sécurité dans la Plateforme d’identités Microsoft (v 2.0).
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/11/2020
ms.author: ryanwi
ms.reviewer: jmprieur, saeeda, sureshja, hirsin
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started
ms.openlocfilehash: 88dc4bb86459cd0390c4c01deb871aa93e39c6d1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84266744"
---
# <a name="security-tokens"></a>Jetons de sécurité

Un fournisseur d’identité centralisé est particulièrement utile pour les applications dont les utilisateurs se trouvant dans le monde entier ne se connectent pas nécessairement à partir du réseau de l’entreprise. La Plateforme d’identités Microsoft authentifie les utilisateurs et fournit des jetons de sécurité, tels qu’un [jeton d’accès](developer-glossary.md#access-token), un [jeton d’actualisation](developer-glossary.md#refresh-token) et un [jeton d’ID](developer-glossary.md#id-token), qui permettent à une [application cliente](developer-glossary.md#client-application) d’accéder aux ressources protégées sur un [serveur de ressources](developer-glossary.md#resource-server).

Un **jeton d’accès** est un jeton de sécurité émis par un [serveur d’autorisation](developer-glossary.md#authorization-server) dans le cadre d’un flux [OAuth 2.0](active-directory-v2-protocols.md). Il contient des informations sur l’utilisateur et l’application auxquels le jeton est destiné, ce qui peut servir pour accéder aux API web et à d’autres ressources protégées. Pour en savoir plus sur la façon dont la Plateforme d’identités Microsoft émet des jetons d’accès, consultez [Jetons d’accès](access-tokens.md).

Les jetons d’accès ne sont valides que pendant une courte période. Par conséquent, les serveurs d’autorisation émettent parfois un **jeton d’actualisation** en même temps que le jeton d’accès. L’application cliente peut alors échanger ce jeton d’actualisation contre un nouveau jeton d’accès si nécessaire. Pour en savoir plus sur la façon dont la Plateforme d’identités Microsoft utilise des jetons d’actualisation pour révoquer des autorisations, consultez [Révocation des jetons](access-tokens.md#token-revocation).

Les **jetons d’ID** sont envoyés à l’application cliente dans le cadre d’un flux [OpenID Connect](v2-protocols-oidc.md). Ils peuvent être envoyés avec ou à la place un jeton d’accès et sont utilisés par le client pour authentifier l’utilisateur. Pour en savoir plus sur la façon dont la Plateforme d’identités Microsoft émet des jetons d’ID, consultez [Jetons d’ID](id-tokens.md).

> [!NOTE]
> Cet article traite des jetons de sécurité utilisés par les protocoles OAuth2 et OpenID Connect. De nombreuses applications d’entreprise utilisent SAML pour authentifier les utilisateurs. Pour plus d’informations sur les assertions SAML, consultez [Référence de jeton SAML Azure AD](reference-saml-tokens.md).

## <a name="validating-security-tokens"></a>Validation des jetons de sécurité

La validation du jeton revient à l’application pour laquelle le jeton a été généré, à l’application web qui a connecté l’utilisateur ou à l’API web appelée. Le jeton est signé par le **service d’émission de jeton de sécurité (STS)** avec une clé privée. Le STS publie la clé publique correspondante. Pour valider un jeton, l’application vérifie la signature à l’aide de la clé publique STS et confirme ainsi que la signature a été créée au moyen de la clé privée.

Les jetons ne sont valides que pour une durée limitée. En règle générale, le STS fournit une paire de jetons :

* un jeton d’accès pour accéder à l’application ou à la ressource protégée ; et
* un jeton d’actualisation utilisé pour actualiser le jeton d’accès lorsque celui-ci est sur le point d’expirer.

Les jetons d’accès sont passés à une API web en tant que jeton du porteur dans l’en-tête `Authorization`. Une application peut fournir un jeton d’actualisation au STS et, si l’accès de l’utilisateur à l’application n’a pas été révoqué, un nouveau jeton d’accès et un nouveau jeton d’actualisation sont obtenus en retour. C’est de cette façon que le scénario d’une personne quittant l’entreprise est géré. Lorsque le STS reçoit le jeton d’actualisation, il n’émet pas d’autre jeton d’accès valide si l’utilisateur n’est plus autorisé.

## <a name="json-web-tokens-jwts-and-claims"></a>Jetons JSON Web Token (JWT) et revendications

La Plateforme d’identités Microsoft implémente des jetons de sécurité comme des **jetons JSON Web Token (JWT)** qui contiennent des **revendications**. Étant donné que les jetons JWT sont utilisés comme jetons de sécurité, cette forme d’authentification est parfois appelée **Authentification JWT**.

Une [revendication](developer-glossary.md#claim) fournit des assertions sur une entité (telle qu’une application cliente ou un [propriétaire de ressources](developer-glossary.md#resource-owner)) à une autre entité, telle qu’un serveur de ressources. Une revendication peut également être appelée revendication JWT ou revendication JSON Web Token.

Les revendications sont des paires nom/valeur qui relaient des faits sur le sujet du jeton. Par exemple, une revendication peut contenir des faits sur le principal de sécurité authentifié par le serveur d’autorisation. Les revendications présentes dans un jeton donné dépendent de plusieurs variables, notamment le type de jeton, le type d’informations d’identification utilisées pour authentifier le sujet, la configuration de l’application et ainsi de suite.

Les applications peuvent utiliser des revendications pour diverses tâches, par exemple :

* Validation du jeton
* Identification du [locataire](developer-glossary.md#tenant) du sujet du jeton
* Affichage d’informations utilisateur
* Détermination de l’autorisation du sujet

Une revendication se compose de paires clé-valeur qui communiquent des informations, telles que les :

* serveur de jetons de sécurité qui a généré le jeton ;
* date à laquelle le jeton a été généré ;
* sujet (tel que l’utilisateur, à l’exception des démons) ;
* audience, qui est l’application pour laquelle le jeton a été généré ;
* application (le client) qui a demandé le jeton. Dans le cas d’applications web, il peut s’agir de l’audience.

Pour en savoir plus sur la façon dont la Plateforme d’identités Microsoft implémente des jetons et les informations de revendication, consultez [Jetons d’accès](access-tokens.md) et [Jetons d’ID](id-tokens.md).

## <a name="how-each-flow-emits-tokens-and-codes"></a>Comment chaque flux émet des jetons et des codes

Selon la façon dont votre client est créé, il peut utiliser un ou plusieurs des flux d’authentification pris en charge par la Plateforme d’identités Microsoft. Ces flux peuvent produire divers jetons (jetons d’ID, jetons d’actualisation, jetons d’accès) ainsi que des codes d’autorisation, et ils nécessitent des jetons différents pour les faire fonctionner. Ce graphique offre une vue d’ensemble :

|Flux | Nécessite | Jeton d’ID | access token | jeton d'actualisation | code d’autorisation |
|-----|----------|----------|--------------|---------------|--------------------|
|[Flux du code d’autorisation](v2-oauth2-auth-code-flow.md) | | x | x | x | x|
|[Flux implicite](v2-oauth2-implicit-grant-flow.md) | | x        | x    |      |                    |
|[Circuit OIDC hybride](v2-protocols-oidc.md#protocol-diagram-access-token-acquisition)| | x  | |          |            x   |
|[Échange de jetons d’actualisation](v2-oauth2-auth-code-flow.md#refresh-the-access-token) | jeton d'actualisation | x | x | x| |
|[Flux On-Behalf-Of](v2-oauth2-on-behalf-of-flow.md) | access token| x| x| x| |
|[Informations d’identification du client](v2-oauth2-client-creds-grant-flow.md) | | | x (application uniquement)| | |

Les jetons émis via le mode implicite ont une longueur maximale du fait qu’ils sont renvoyés au navigateur via l’URL (où `response_mode` est `query` ou `fragment`).  Certains navigateurs limitent la taille de l’URL qui peut être placée dans la barre d’adresse et refusent les URL trop longues.  Par conséquent, ces jetons n’ont pas de revendications `groups` ou `wids`.

## <a name="next-steps"></a>Étapes suivantes

Pour d’autres rubriques traitant des concepts de base de l’authentification et de l’autorisation :

* Pour en savoir plus sur les concepts de base de l’authentification et de l’autorisation dans la Plateforme d’identités Microsoft, consultez [Authentification ou autorisation](authentication-vs-authorization.md).
* Pour découvrir le processus d’inscription de votre application afin qu’elle puisse s’intégrer avec la Plateforme d’identités Microsoft, consultez [Modèle d’application](application-model.md).
* Pour en savoir plus sur le flux de connexion des applications web, de bureau et mobiles dans la Plateforme d’identités Microsoft, consultez [Flux de connexion des applications](app-sign-in-flow.md).
