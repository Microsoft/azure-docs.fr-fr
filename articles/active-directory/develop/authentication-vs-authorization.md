---
title: Authentification ou autorisation | Azure
titleSuffix: Microsoft identity platform
description: Découvrez les principes de base de l’authentification et de l’autorisation dans la Plateforme d’identités Microsoft (v2.0).
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/22/2020
ms.author: ryanwi
ms.reviewer: jmprieur, saeeda, sureshja, hirsin
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started
ms.openlocfilehash: 320debcf2b744e949faf7b29dd72ed334fe0feea
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92503723"
---
# <a name="authentication-vs-authorization"></a>Authentification ou autorisation

Cet article définit l’authentification et l’autorisation, et explique brièvement comment vous pouvez utiliser la Plateforme d’identités Microsoft pour authentifier et autoriser des utilisateurs dans vos applications web, API web ou applications appelant des API web protégées. Si vous voyez un terme que vous ne connaissez pas, consultez notre [glossaire](developer-glossary.md) ou nos [Vidéos sur la plateforme d’identités Microsoft](identity-videos.md), qui couvrent les concepts de base.

## <a name="authentication"></a>Authentification

L’ **authentification** est le processus visant à prouver que vous êtes bien qui vous prétendez être. On abrège parfois l’authentification de cette manière : AuthN. La Plateforme d’identités Microsoft implémente le protocole [OpenID Connect](https://openid.net/connect/) pour gérer l’authentification.

## <a name="authorization"></a>Autorisation

L’ **autorisation** correspond à l’action d’accorder à une partie authentifiée le droit de faire quelque chose. Elle détermine les données auxquelles vous pouvez accéder, et ce que vous pouvez faire de ces données. On abrège parfois l’autorisation de cette manière : AuthZ. La Plateforme d’identités Microsoft implémente le protocole [OAuth 2.0](https://oauth.net/2/) pour la gestion de l’autorisation.

## <a name="authentication-and-authorization-using-microsoft-identity-platform"></a>Authentification et autorisation avec la plateforme d’identités Microsoft

Au lieu de créer des applications gérant chacune leurs propres informations de nom d’utilisateur et de mot de passe (ce qui entraîne une charge administrative importante lorsque vous devez ajouter ou supprimer des utilisateurs sur plusieurs applications), les applications peuvent déléguer cette responsabilité à un fournisseur d’identité centralisée.

> [!VIDEO https://www.youtube.com/embed/tkQJSHFsduY]

Azure Active Directory (Azure AD) est un fournisseur d’identité centralisée, présent dans le cloud. La délégation de l’authentification et de l’autorisation à ce service permet des scénarios, tels que des stratégies d’accès conditionnel nécessitant la présence d’un utilisateur dans un lieu spécifique, l’utilisation de l’[authentification multifacteur](../authentication/concept-mfa-howitworks.md) (parfois appelée authentification à deux facteurs, ou 2FA), ainsi que la possibilité pour l’utilisateur de se connecter une seule fois et d’être ensuite connecté automatiquement à toutes les applications web qui partagent le même annuaire centralisé. Cette fonctionnalité s’appelle **authentification unique (SSO)** .

La plateforme d’identités Microsoft simplifie l’autorisation et l’authentification pour les développeurs d’application en fournissant l’identité en tant que service, avec la prise en charge des protocoles standard tels que OAuth 2.0 et OpenID Connect, ainsi que des bibliothèques open source pour différentes plateformes afin de vous permettre de commencer à coder rapidement. Elle permet aux développeurs de générer des applications qui connectent toutes les identités Microsoft et obtiennent des jetons pour appeler [Microsoft Graph](https://developer.microsoft.com/graph/), d’autres API Microsoft ou des API que des développeurs ont créées.

Voici une brève comparaison des différents protocoles utilisés par la plateforme d’identités Microsoft :

* **OAuth ou OpenID Connect**  : OAuth est utilisé pour l’autorisation, et OpenID Connect (OIDC) pour l’authentification. Comme OpenID Connect s’appuie sur OAuth 2.0, la terminologie et le flux sont similaires entre ces deux protocoles. Vous pouvez même authentifier un utilisateur (à l’aide de OpenID Connect) et obtenir l’autorisation d’accéder à une ressource protégée que l’utilisateur possède (à l’aide de OAuth 2.0) dans une même requête. Pour plus d’informations, consultez [Protocoles OAuth 2.0 et OpenID Connect](active-directory-v2-protocols.md) et [Protocole OpenID Connect](v2-protocols-oidc.md).
* **OAuth ou SAML**  : OAuth est utilisé pour l’autorisation, et SAML pour l’authentification. Pour plus d’informations sur la façon dont les deux protocoles peuvent être utilisés ensemble pour authentifier un utilisateur (avec SAML) et obtenir l’autorisation d’accéder à une ressource protégée (avec OAuth 2.0), consultez [Plateforme d’identités Microsoft et flux d’assertion du porteur SAML OAuth 2.0](v2-saml-bearer-assertion.md).
* **OpenID Connect ou SAML**  : OpenID Connect et SAML sont tous les deux utilisés pour authentifier un utilisateur et activer l’authentification unique. L’authentification SAML étant couramment utilisée avec les fournisseurs d’identité, tels que les services de fédération Active Directory (AD FS) fédérés à Azure AD, elle sert donc fréquemment dans les applications d’entreprise. OpenID Connect est couramment utilisé pour les applications qui sont exclusivement dans le cloud, telles que les applications mobiles, les sites web et les API web.

## <a name="next-steps"></a>Étapes suivantes

Pour d’autres rubriques traitant des concepts de base de l’authentification et de l’autorisation :

* Pour savoir comment les jetons d’accès, les jetons d’actualisation et les jetons d’ID sont utilisés dans l’autorisation et l’authentification, consultez [Jetons de sécurité](security-tokens.md).
* Pour découvrir le processus d’inscription de votre application afin qu’elle puisse s’intégrer avec la Plateforme d’identités Microsoft, consultez [Modèle d’application](application-model.md).
* Pour en savoir plus sur le flux de connexion des applications web, de bureau et mobiles dans la Plateforme d’identités Microsoft, consultez [Flux de connexion des applications](app-sign-in-flow.md).

* Pour en savoir plus sur les protocoles implémentés par la plateforme d’identités Microsoft, consultez [Protocoles OAuth 2.0 et OpenID Connect sur la plateforme d’identités Microsoft](active-directory-v2-protocols.md).
* Pour plus d’informations sur la façon dont la Plateforme d’identités Microsoft prend en charge l’authentification unique, consultez [Protocole SAML d’authentification unique](single-sign-on-saml-protocol.md).
* Pour plus d’informations sur les différentes façons d’implémenter l’authentification unique dans votre application, consultez [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
