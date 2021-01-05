---
title: Protocoles d’authentification dans Azure Active Directory B2C | Microsoft Docs
description: Création d’applications directement à l’aide des protocoles pris en charge par Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 11/30/2018
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: ed393f721d4461ebadea41f8dad707d4881865cd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "78183903"
---
# <a name="azure-ad-b2c-authentication-protocols"></a>Azure AD B2C : protocoles d’authentification
Azure Active Directory B2C (Azure AD B2C) fournit l’identité en tant que service pour vos applications en prenant en charge deux protocoles standard, OpenID Connect et OAuth 2.0. Bien que ce service soit conforme aux normes, vous pouvez constater de subtiles différences entre deux implémentations différentes de ces protocoles.

Les informations fournies dans ce guide sont utiles si vous écrivez votre code en envoyant et en traitant directement des requêtes HTTP, plutôt qu’en utilisant une bibliothèque open source. Nous vous recommandons de lire les informations de cette page avant d’entrer dans les détails de chaque protocole. En revanche, si vous connaissez déjà Azure AD B2C, vous pouvez accéder directement aux [guides de référence du protocole](#protocols).

<!-- TODO: Need link to libraries above -->

## <a name="the-basics"></a>Concepts de base
Chaque application qui utilise Azure AD B2C doit être inscrite dans votre répertoire B2C sur le [portail Azure](https://portal.azure.com). Le processus d’inscription des applications collecte quelques valeurs et les affecte à votre application :

* un **ID d’application** qui identifie de manière unique votre application ;
* un **URI de redirection** ou un **identificateur de package** pouvant être utilisé pour diriger des réponses vers votre application ;
* quelques valeurs spécifiques au scénario. Pour en savoir plus, découvrez [comment inscrire votre application](tutorial-register-applications.md).

Une fois inscrite, l’application communique avec Azure Active Directory (Azure AD) en transmettant les requêtes au point de terminaison :

```
https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/oauth2/v2.0/authorize
https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/oauth2/v2.0/token
```

Dans presque tous les flux OAuth et OpenID Connect, quatre parties sont concernées par l’échange :

![Diagramme montrant les 4 rôles OAuth 2.0](./media/protocols-overview/protocols_roles.png)

* Le **serveur d’autorisation** est le point de terminaison Azure AD. Il traite de manière sécurisée toutes les informations relatives à l’utilisateur, l’accès ainsi que les relations de confiance entre les parties des flux. Il est chargé de vérifier l’identité de l’utilisateur, l’octroi et la révocation de l’accès aux ressources et l’émission de jetons. Il est également connu sous le nom de fournisseur d’identité.

* Le **propriétaire de la ressource** est généralement l’utilisateur final. Il s’agit de la partie détentrice des données, qui a le pouvoir d’autoriser les tierces parties à accéder à ces données ou à cette ressource.

* Le **client OAuth** est votre application. Il est identifié par son ID d’application. Il s’agit généralement de la partie avec laquelle l’utilisateur final interagit. Elle demande également des jetons provenant du serveur d’autorisation. Le propriétaire de la ressource doit accorder au client l’autorisation d’accéder à la ressource.

* Le **serveur de ressources** héberge la ressource ou les données. Il approuve le serveur d’autorisation pour authentifier et autoriser le client OAuth en toute sécurité. Il utilise également les jetons d’accès du porteur pour garantir l’octroi de l’accès à une ressource.

## <a name="policies-and-user-flows"></a>Stratégies et flux d’utilisateur
En fait, les stratégies Azure AD B2C constituent la fonctionnalité la plus importante du service. Azure AD B2C étend les protocoles OAuth 2.0 et OpenID Connect standard en introduisant des stratégies qui permettent à Azure AD B2C d’effectuer des opérations d’authentification et d’autorisation bien plus simples.

Pour vous aider à configurer les tâches d’identité les plus courantes, le portail Azure AD B2C inclut des stratégies configurables prédéfinies appelées **flux d’utilisateur**. Les flux d’utilisateur décrivent entièrement les expériences liées à l’identité du consommateur, telles que l’inscription, la connexion et la modification de profil. Ils peuvent être définis dans une interface utilisateur d’administration. et exécutées à l’aide d’un paramètre de requête spécial dans les requêtes d’authentification HTTP.

Les flux d’utilisateur ne constituant pas une fonctionnalité standard d’OAuth 2.0 ni d’OpenID Connect, vous devez prendre le temps de les comprendre. Pour plus d’informations, consultez le [guide de référence sur les flux d’utilisateur Azure AD B2C](user-flow-overview.md).

## <a name="tokens"></a>Jetons
L’implémentation d’OAuth 2.0 et d’OpenID Connect par Azure AD B2C utilise massivement les jetons du porteur, y compris ceux représentés sous forme de jetons web JSON (JWT). Un jeton du porteur est un jeton de sécurité léger qui octroie l’accès à une ressource protégée au « porteur ».

En ce sens, le porteur désigne toute partie qui peut présenter le jeton. Une partie doit d’abord s’authentifier auprès d’Azure AD pour recevoir un jeton du porteur, mais si les mécanismes nécessaires à la sécurité du jeton lors de la transmission et du stockage ne sont pas en place, il peut être intercepté et utilisé par une partie non autorisée.

Bien que certains jetons de sécurité intègrent des mécanismes de protection contre l’utilisation par des parties non autorisées, les jetons du porteur n’en sont pas dotés et ils doivent donc être acheminés sur un canal sécurisé, par exemple à l’aide du protocole TLS (HTTPS).

Si un jeton du porteur est transmis en dehors d’un canal sécurisé, une partie malveillante peut utiliser une attaque d’intercepteur afin de s’approprier le jeton et de l’utiliser pour accéder sans autorisation à une ressource protégée. Les mêmes principes de sécurité s’appliquent au stockage ou à la mise en cache des jetons du porteur pour une utilisation ultérieure. Veillez systématiquement à ce que votre application transmette et stocke les jetons porteurs de manière sécurisée.

Pour connaître d’autres aspects de la sécurité des jetons du porteur, consultez [RFC 6750 Section 5](https://tools.ietf.org/html/rfc6750).

Pour plus d’informations sur les différents types de jetons utilisés dans Azure AD B2C, consultez [la référence sur les jetons Azure AD](tokens-overview.md).

## <a name="protocols"></a>Protocoles
Quand vous êtes prêt à examiner des exemples de requêtes, commencez à lire l’un des didacticiels ci-dessous. Chacun d’eux correspond à un scénario d’authentification particulier. Si vous avez besoin d’aide pour déterminer le flux qui vous convient, consultez les [types d’applications que vous pouvez créer avec Azure AD B2C](application-types.md).

* [Génération d’une application mobile et native avec OAuth 2.0](authorization-code-flow.md)
* [Génération d’applications web avec Open ID Connect](openid-connect.md)
* [Génération d'applications de page unique avec le flux implicite OAuth 2.0](implicit-flow-single-page-application.md)

