---
title: Pourquoi opérer une mise à jour vers la Plateforme d’identités Microsoft (v2.0) | Azure
description: Découvrez les différences entre le point de terminaison de la Plateforme d’identités Microsoft (v2.0) et celui d’Azure Active Directory (Azure AD) v1.0, ainsi que les avantages d’une mise à jour vers la version 2.0.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.workload: identity
ms.topic: conceptual
ms.date: 07/17/2020
ms.author: ryanwi
ms.reviewer: saeeda, hirsin, jmprieur, sureshja, jesakowi, lenalepa, kkrishna, negoe
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 8f6170de65ae5e1ca8ecb5f7cc8a78f4f194ac41
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92055288"
---
# <a name="why-update-to-microsoft-identity-platform-v20"></a>Pourquoi mettre à jour à la plateforme d’identités Microsoft (v2.0) ?

Lorsque vous développez une nouvelle application, il est important de connaître les différences entre les points de terminaison respectifs de la Plateforme d’identités Microsoft (v2.0) et d’Azure Active Directory (v1.0). Cet article décrit les principales différences entre ces points de terminaison et certains limitations existantes pour la Plateforme d’identités Microsoft.

## <a name="who-can-sign-in"></a>Qui peut se connecter

![Utilisateurs autorisés à se connecter avec les points de terminaison v1.0 et v2.0](media/azure-ad-endpoint-comparison/who-can-signin.svg)

* Le point de terminaison v1.0 permet uniquement aux comptes professionnels et scolaires de se connecter à votre application (Azure AD).
* Le point de terminaison de la Plateforme d’identités Microsoft permet aux comptes professionnels et scolaires d’Azure AD ainsi qu’aux comptes Microsoft personnels (MSA), tels que hotmail.com, outlook.com et msn.com, de se connecter.
* Les deux points de terminaison acceptent également des connexions d’ *[utilisateurs invités](../external-identities/what-is-b2b.md)* issus d’un répertoire Azure AD pour les applications configurées comme étant *[à client unique](../develop/single-and-multi-tenant-apps.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json)* ou pour les applications *mutualisées* configurées de manière à pointer vers le point de terminaison spécifique du locataire (`https://login.microsoftonline.com/{TenantId_or_Name}`).

Le point de terminaison de la Plateforme d’identités Microsoft vous permet d’écrire des applications qui acceptent les connexions à partir de comptes Microsoft personnels, ainsi que de comptes professionnels ou scolaires. Ceci vous donne la possibilité d’écrire votre application de façon complètement indépendante des comptes. Par exemple, si votre application appelle [Microsoft Graph](https://graph.microsoft.io), certaines fonctionnalités et données supplémentaires seront disponibles pour les comptes professionnels, comme leurs sites SharePoint ou les données de leurs annuaires. Mais pour de nombreuses actions, comme la [lecture du courrier d’un utilisateur](/graph/api/user-list-messages), vous pouvez utiliser le même code pour accéder à la messagerie, aussi bien pour les comptes professionnels que scolaires.

Pour le point de terminaison de la Plateforme d’identités Microsoft, vous pouvez utiliser la bibliothèque MSAL (Microsoft Authentication Library) pour accéder aux différents mondes (utilisateurs privés, scolaires ou professionnels). Le point de terminaison Azure AD v1.0 accepte les connexions uniquement depuis des comptes professionnels et scolaires.

## <a name="incremental-and-dynamic-consent"></a>Consentement incrémentiel et dynamique

Les applications qui utilisent le point de terminaison Azure AD v1.0 doivent spécifier à l’avance les autorisations OAuth 2.0 requises, par exemple :

![Exemple illustrant l’interface utilisateur d’inscription des autorisations](./media/azure-ad-endpoint-comparison/app-reg-permissions.png)

Les autorisations qui sont directement définies sur l’inscription d’application sont **statiques**. Bien que les autorisations statiques de l’application définies dans le portail Azure préservent la simplicité du code, elles présentent quelques problèmes potentiels pour les développeurs :

* L’application doit demander toutes les autorisations dont elle est susceptible d’avoir besoin dès la première connexion de l’utilisateur. Ceci peut résulter en une longue liste d’autorisations qui décourage les utilisateurs finaux d’approuver l’accès de l’application lors de la connexion initiale.

* L’application doit connaître au préalable l’ensemble des ressources auxquelles elle est susceptible d’accéder. Il était difficile de créer des applications pouvant accéder à un nombre arbitraire de ressources.

Avec le point de terminaison de la Plateforme d’identités Microsoft, vous pouvez ignorer les autorisations statiques définies dans les informations d’inscription de l’application du portail Azure et demander à la place des autorisations de façon incrémentielle, ce qui signifie demander dès le départ un ensemble minimal d’autorisations, puis d’en demander plus au fil du temps à mesure que le client utilise des fonctionnalités supplémentaires de l’application. Pour cela, vous pouvez spécifier les étendues dont votre application a besoin à tout moment en incluant les nouvelles étendues dans le paramètre `scope` quand vous demandez un jeton d’accès, sans qu’il soit nécessaire de les définir au préalable dans les informations d’inscription de l’application. Si l’utilisateur n’a pas encore consenti aux nouvelles étendues ajoutées à la demande, il est invité à donner son consentement seulement aux nouvelles autorisations. Pour plus d’informations, consultez [Autorisations, consentements et étendues](../develop/v2-permissions-and-consent.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

En permettant à l’application de demander des autorisations de façon dynamique grâce au paramètre `scope`, les développeurs maîtrisent totalement l’expérience de vos utilisateurs. Vous pouvez aussi anticiper l’expérience de consentement et demander toutes les autorisations dans une même demande d’autorisation initiale. Si votre application nécessite un grand nombre d’autorisations, vous pouvez les demander à l’utilisateur de façon incrémentielle quand il essaie d’utiliser certaines fonctionnalités de votre application au fil du temps.

Quand il est recueilli pour le compte d’une organisation, le consentement de l’administrateur nécessite toujours les autorisations statiques inscrites pour l’application. Par conséquent, nous vous recommandons de définir ces autorisations pour les applications dans le portail d’inscription des applications si vous avez besoin qu’un administrateur donne son consentement pour le compte de toute l’organisation. Ainsi, les cycles nécessaires à l’administrateur de l’organisation pour configurer l’application sont réduits.

## <a name="scopes-not-resources"></a>Des étendues, pas des ressources

Lorsqu’elle utilise le point de terminaison v1.0, une application peut se comporter comme une **ressource** ou comme un destinataire de jetons. Une ressource peut définir plusieurs **étendues** ou **oAuth2Permissions** qu’elle comprend, permettant ainsi aux applications clientes de demander des jetons à cette ressource pour un ensemble d’étendues donné. Prenez comme exemple de ressource l’API Microsoft Graph :

* Identificateur de ressource, ou `AppID URI` : `https://graph.microsoft.com/`
* Étendues ou `oAuth2Permissions`: `Directory.Read`, `Directory.Write`, et ainsi de suite.

Ceci se vérifie pour le point de terminaison de la Plateforme d’identités Microsoft. Une application peut toujours se comporter comme une ressource, définir des étendues et être identifiée par un URI. Les applications clientes peuvent toujours demander l’accès à ces étendues. Toutefois, la manière dont le client demande ces autorisations a changé.

Pour le point de terminaison v1.0, une demande d’autorisation OAuth 2.0 à Azure AD pouvait ressembler à ceci :

```text
GET https://login.microsoftonline.com/common/oauth2/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
&resource=https://graph.microsoft.com/
...
```

Ici, le paramètre **resource** indiquait la ressource pour laquelle l’application cliente demandait une autorisation. Azure AD calculait les autorisations requises par l’application en fonction de la configuration statique définie sur le Portail Azure, puis émettait les jetons en conséquence.

Pour les applications utilisant le point de terminaison de la Plateforme d’identités Microsoft, cette même demande d’autorisation OAuth 2.0 ressemble à ceci :

```text
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
&scope=https://graph.microsoft.com/directory.read%20https://graph.microsoft.com/directory.write
...
```

Ici, le paramètre **scope** indique la ressource et les autorisations pour lesquelles l’application demande une autorisation. La ressource souhaitée est toujours présente dans la demande : elle est englobée dans chacune des valeurs du paramètre d’étendue. L’utilisation du paramètre d’étendue de cette manière permet au point de terminaison de la Plateforme d’identités Microsoft d’être plus conforme à la spécification OAuth 2.0, ainsi qu’aux pratiques courantes du secteur. Elle permet également aux applications d’obtenir un [consentement incrémentiel](#incremental-and-dynamic-consent) en ne demandant des autorisations que quand elles en ont besoin, plutôt qu’au préalable.

## <a name="well-known-scopes"></a>Étendues connues

### <a name="offline-access"></a>Accès hors connexion

Les applications qui utilisent le point de terminaison de la Plateforme d’identités Microsoft peuvent nécessiter l’utilisation d’une nouvelle autorisation bien connue pour les applications, à savoir l’étendue `offline_access`. Toutes les applications doivent demander cette autorisation si elles doivent accéder aux ressources au nom d’un utilisateur pendant une période prolongée, même si l’utilisateur n’utilise peut-être pas activement l’application donnée. L’étendue `offline_access` apparaît à l’utilisateur dans la boîte de dialogue de consentement **Accéder aux données à tout moment**, que l’utilisateur doit accepter. Demander l’autorisation `offline_access` permet à votre application web de recevoir les jetons d’actualisation OAuth 2.0 du point de terminaison de la Plateforme d’identités Microsoft. Les jetons d’actualisation ont une durée de vie assez longue, et peuvent être échangés contre les nouveaux jetons d’accès OAuth 2.0 pour des périodes d’accès prolongées.

Si votre application ne sollicite pas l’étendue `offline_access`, elle ne reçoit pas de jetons d’actualisation. Ainsi, quand vous échangez un code d’autorisation dans le flux de code d’autorisation OAuth 2.0, vous recevez seulement en retour un jeton d’accès du point de terminaison `/token`. Ce jeton d’accès demeure valide pendant une courte période de temps (généralement une heure), mais il finit par expirer. À ce stade, votre application doit rediriger l’utilisateur vers le point de terminaison `/authorize` afin de récupérer un nouveau code d’autorisation. Lors de cette redirection, il peut ou non être demandé à l’utilisateur d’entrer à nouveau ses informations d’identification ou de consentir une nouvelle fois les autorisations, en fonction du type d’application.

Pour en savoir plus sur OAuth 2.0, `refresh_tokens`, et `access_tokens`, consultez la [documentation de référence sur la Plateforme d’identités Microsoft](../develop/active-directory-v2-protocols.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

### <a name="openid-profile-and-email"></a>OpenID, profil et e-mail

Historiquement, le flux de connexion OpenID Connect de base avec la Plateforme d’identités Microsoft fournissait de nombreuses informations sur l’utilisateur dans le jeton *id_token* qui en résultait. Les revendications contenues dans un paramètre id_token peuvent inclure différentes informations utilisateur : son nom, son nom d’utilisateur par défaut, son adresse de messagerie, son ID objet, etc.

Les informations auxquelles l’étendue `openid` permet à votre application d’accéder sont maintenant restreintes. L’étendue `openid` ne l’autorisera qu’à connecter l’utilisateur et à recevoir un identificateur propre à l’application pour l’utilisateur. Si vous voulez obtenir des données personnelles sur l’utilisateur dans votre application, celle-ci doit lui demander des autorisations supplémentaires. Deux nouvelles étendues, `email` et `profile`, vous permettent de le faire.

* L’étendue `email` permet à votre application d’accéder à l’adresse e-mail principale de l’utilisateur via la revendication `email` contenue dans id_token, en supposant que l’utilisateur ait une adresse e-mail utilisable.
* L’étendue `profile` permet à votre application d’accéder à toutes les autres informations de base sur l’utilisateur contenues dans le jeton id-token, comme le nom, le nom d’utilisateur par défaut, l’ID d’objet, etc.

Ces étendues vous permettent de coder votre application en divulguant le moins possible d’informations : vous pouvez vous contenter de demander à l’utilisateur les informations nécessaires à l’exécution de votre application. Pour plus d’informations sur ces étendues, voir la [documentation de référence sur la Plateforme d’identités Microsoft](../develop/v2-permissions-and-consent.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

## <a name="token-claims"></a>Revendications de jetons

Le point de terminaison de la Plateforme d’identités Microsoft émet un ensemble plus restreint de revendications dans ses jetons par défaut, pour limiter la taille des charges utiles. Si vous avez des applications et des services qui dépendent d’une revendication particulière dans un jeton v1.0 qui n’est plus fourni par défaut dans un jeton de la Plateforme d’identités Microsoft, envisagez d’utiliser la fonctionnalité des [revendications facultatives](../develop/active-directory-optional-claims.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json) pour inclure cette revendication.

> [!IMPORTANT]
> Les jetons v1.0 et v2.0 peuvent être émis par les points de terminaison v1.0 et v2.0 ! id_tokens corresponde *toujours* au point de terminaison à partir duquel ils sont demandés, et les jetons d’accès correspondent *toujours* au format attendu par l’API Web que votre client appellera à l’aide de ce jeton.  Par conséquent, si l’application utilise le point de terminaison v2.0 pour obtenir un jeton permettant d’appeler Microsoft Graph, qui attend des jetons d’accès au format v1.0, elle reçoit un jeton au format v1.0.

## <a name="limitations"></a>Limites

Il existe quelques restrictions à connaître lors de l’utilisation de la Plateforme d’identités Microsoft.

Lorsque vous créez des applications qui s’intègrent dans la Plateforme d’identités Microsoft, vous devez déterminer si les protocoles d’authentification et le point de terminaison de la Plateforme d’identités Microsoft répondent à vos besoins. Le point de terminaison et la plateforme v1.0 sont toujours intégralement pris en charge. À certains égards, ils sont plus riches en fonctionnalités que la Plateforme d’identités Microsoft. Toutefois, la Plateforme d’identités Microsoft [présente des avantages significatifs](azure-ad-endpoint-comparison.md) pour les développeurs.

Voici une suggestion simplifiée pour les développeurs à ce stade :

* Si vous voulez ou si vous devez prendre en charge des comptes personnels Microsoft dans votre application, ou si vous écrivez une nouvelle application, utilisez la Plateforme d’identités Microsoft. Avant cela, comprenez cependant bien les limitations exposées dans cet article.
* Si vous migrez ou mettez à jour une application qui s’appuie sur SAML, vous ne pouvez pas utiliser la Plateforme d’identités Microsoft. Dans ce cas, consultez le [guide Azure AD v1.0](v1-overview.md).

Le point de terminaison de la Plateforme d’identités Microsoft va évoluer et les restrictions listées ici seront éliminées. Ainsi, vous pourrez toujours utiliser le point de terminaison de la Plateforme d’identités Microsoft. En attendant, cet article est là pour vous aider à déterminer si le point de terminaison de la Plateforme d’identités Microsoft répond à vos besoins. Nous continuerons à mettre à jour cet article pour refléter l’état actuel du point de terminaison de la Plateforme d’identités Microsoft. Consultez-le régulièrement pour réévaluer vos besoins par rapport aux fonctionnalités de la Plateforme d’identités Microsoft.

### <a name="restrictions-on-app-registrations"></a>Restrictions sur les inscriptions d’application

Pour chaque application que vous souhaitez intégrer avec le point de terminaison de la Plateforme d’identités Microsoft, vous pouvez créer une inscription d’application dans la nouvelle [expérience **Inscriptions d’applications**](https://aka.ms/appregistrations) dans le portail Azure. Les applications de compte Microsoft existantes ne sont pas compatibles avec le portail, mais toutes les applications Azure AD le sont, quel que soit l’endroit et le moment où elles ont été inscrites.

Les inscriptions d’applications qui prennent en charge les comptes professionnels et scolaires ont les caractéristiques suivantes :

* Seuls deux secrets d’application sont autorisés par ID d’application.
* Une application qui n’a pas été inscrite dans un locataire peut être gérée seulement par le compte qui l’a inscrite. Elle ne peut pas être partagée avec d’autres développeurs. C’est le cas pour la plupart des applications qui ont été inscrites avec un compte Microsoft personnel dans le portail d’inscription des applications. Si vous voulez partager votre inscription d’application avec plusieurs développeurs, inscrivez l’application dans un locataire en utilisant la nouvelle section **Inscriptions d’applications** du portail Azure.
* Il existe plusieurs restrictions quant au format de l’URL de redirection autorisé. Pour plus d’informations sur les URL de redirection, consultez la section suivante.

### <a name="restrictions-on-redirect-urls"></a>Restrictions concernant les URL de redirection

Pour obtenir les informations les plus récentes concernant les restrictions sur les URL de redirection pour les applications inscrites pour Plateforme d’identités Microsoft, consultez [Limitations et restrictions des URI de redirection/URL de réponse](../develop/reply-url.md) dans la documentation de Plateforme d’identités Microsoft.

Pour savoir comment inscrire une application en vue de son utilisation avec la Plateforme d’identités Microsoft, voir [Inscrire une application à l’aide de la nouvelle expérience Inscriptions application](../develop/quickstart-register-app.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

### <a name="restrictions-on-libraries-and-sdks"></a>Restrictions concernant les bibliothèques et les SDK

Actuellement, la prise en charge de la bibliothèque pour le point de terminaison de la Plateforme d’identités Microsoft est limitée. Si vous souhaitez utiliser le point de terminaison de la Plateforme d’identités Microsoft dans une application de production, vous disposez des options suivantes :

* Si vous créez une application web, vous pouvez utiliser sans problème le middleware (intergiciel) côté serveur en disponibilité générale pour vous connecter et valider des jetons. Il s’agit notamment du middleware OWIN OpenID Connect pour ASP.NET et du plug-in Node.js Passport. Pour obtenir des exemples de code qui utilisent le middleware Microsoft, voir la section [Prise en main de la Plateforme d’identités Microsoft](../develop/v2-overview.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json#getting-started).
* Si vous créez une application mobile ou pour poste de travail, vous pouvez utiliser une des bibliothèques d’authentification Microsoft (MSAL). Ces bibliothèques étant généralement disponibles ou en préversion prise en charge pour la production, leur utilisation dans des applications de production ne présente aucun risque. Pour plus d’informations sur les conditions d’utilisation de la préversion et sur les bibliothèques disponibles, consultez la [documentation de référence sur les bibliothèques d’authentification](../develop/reference-v2-libraries.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).
* Vous pouvez intégrer les plateformes non couvertes par les bibliothèques Microsoft, dans le point de terminaison de la Plateforme d’identités Microsoft en envoyant et en recevant directement des messages de protocole dans le code de votre application. Les protocoles v2.0 OpenID Connect et OAuth [sont explicitement documentés](../develop/active-directory-v2-protocols.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json) pour vous aider à effectuer une telle intégration.
* Enfin, vous pouvez utiliser les bibliothèques open source OpenID Connect et OAuth pour l’intégration avec le point de terminaison de la Plateforme d’identités Microsoft. Le point de terminaison de la Plateforme d’identités Microsoft doit normalement être compatible sans modifications avec de nombreuses bibliothèques de protocoles open source. La disponibilité de ces types de bibliothèques varie en fonction de la langue et de la plateforme. Les sites web [OpenID Connect](https://openid.net/connect/) et [OAuth 2.0](https://oauth.net/2/) contiennent une liste à jour des implémentations les plus courantes. Pour plus d’informations, voir [Plateforme d’identités Microsoft et bibliothèques d’authentification](../develop/reference-v2-libraries.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json), ainsi que la liste des bibliothèques clientes open source et des exemples qui ont été testés avec la Plateforme d’identités Microsoft.
* Pour référence, le point de terminaison `.well-known` pour le point de terminaison commun de la Plateforme d’identités Microsoft est `https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration`. Remplacez `common` par votre ID de locataire pour obtenir des données spécifiques à votre locataire.

### <a name="protocol-changes"></a>Changements de protocole

Le point de terminaison de la Plateforme d’identités Microsoft ne prend pas en charge SAML ou WS-Federation ; il prend en charge uniquement OpenID Connect et OAuth 2.0.  Les changements notables apportés aux protocoles OAuth 2.0 par rapport au point de terminaison v1.0 sont :

* La revendication `email` est retournée si une revendication facultative est configurée **ou** si scope=email a été spécifié dans la demande.
* Le paramètre `scope` est désormais pris en charge à la place du paramètre `resource`.
* De nombreuses réponses ont été modifiées pour les rendre plus conforme à la spécification OAuth 2.0, par exemple retourner correctement `expires_in` sous la forme d’un entier au lieu d’une chaîne.

Pour mieux comprendre l’étendue des fonctionnalités de protocole prises en charge dans le point de terminaison de la Plateforme d’identités Microsoft, voir [Informations de référence sur les protocoles OpenID Connect et OAuth 2.0](../develop/active-directory-v2-protocols.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

#### <a name="saml-usage"></a>Utilisation SAML

Si vous avez utilisé la bibliothèque ADAL (Active Directory Authentication Library) dans des applications Windows, vous avez peut-être tiré parti de l’authentification intégrée Windows, qui utilise l’octroi d’assertions SAML (Security Assertion Markup Language). Avec cet octroi, les utilisateurs de locataires Azure AD fédérés peuvent s’authentifier en mode silencieux auprès de leur instance d’Active Directory locale sans entrer leurs informations d’identification. Bien que [SAML soit toujours un protocole pris en charge](../develop/active-directory-saml-protocol-reference.md) pour une utilisation avec des utilisateurs d’entreprise, le point de terminaison v2.0 est destiné uniquement aux applications OAuth 2.0.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage dans la [documentation de la plateforme d’identités Microsoft](../develop/index.yml).
