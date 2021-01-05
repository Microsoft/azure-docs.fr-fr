---
title: Vue d’ensemble des jetons - Azure Active Directory B2C
description: En savoir plus sur les jetons utilisés dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: b4e268d35a2e31db0ce92ff61e66fd23bce68e38
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516364"
---
# <a name="overview-of-tokens-in-azure-active-directory-b2c"></a>Vue d’ensemble des jetons dans Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) émet plusieurs types de jetons de sécurité lors du traitement de chaque [flux d’authentification](application-types.md). Ce document décrit le format, les caractéristiques en matière de sécurité et le contenu de chaque type de jeton.

## <a name="token-types"></a>Types de jetons

Azure AD B2C prend en charge les [protocoles OAuth 2.0 et OpenID Connect](protocols-overview.md), utilisant des jetons pour l’authentification et l’accès sécurisé aux ressources. Tous les jetons utilisés dans Azure AD B2C sont des [JSON web tokens (JWT)](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) qui contiennent des assertions d’informations concernant le porteur et le sujet du jeton.

Les jetons suivants sont utilisés dans les communications avec Azure AD B2C :

- *ID jeton* - Jeton JWT contenant des revendications que vous pouvez utiliser pour identifier l’utilisateur dans votre application. Le jeton est ainsi envoyé en toute sécurité dans des requêtes HTTP pour permettre la communication entre deux composants de la même application ou du même service. Vous pouvez utiliser les revendications dans un jeton d’ID comme vous le souhaitez. Ils sont souvent utilisés pour afficher les informations de compte ou pour prendre des décisions de contrôle d’accès dans une application. Les jetons d’ID sont signés, mais ils ne sont pas chiffrés. Lorsque votre application ou votre API reçoit un jeton d’ID, elle doit valider la signature pour prouver que le jeton est authentique. Votre application ou votre API doit également valider certaines revendications du jeton pour prouver qu’il est valide. Les revendications validées par une application varient selon les spécifications du scénario, mais il existe certaines validations de revendication communes auxquelles votre application doit procéder dans chaque scénario.
- *Jeton d’accès* - Jeton JWT contenant des revendications que vous pouvez utiliser pour identifier les autorisations octroyées à vos API. Les jetons d’accès sont signés, mais ils ne sont pas chiffrés. Les jetons d’accès permettent de fournir l’accès aux API et serveurs de ressources.  Lorsque votre API reçoit un jeton d’accès, elle doit valider la signature pour prouver que le jeton est authentique. Votre API doit également valider certaines revendications du jeton pour prouver qu’il est valide. Les revendications validées par une application varient selon les spécifications du scénario, mais il existe certaines validations de revendication communes auxquelles votre application doit procéder dans chaque scénario.
- *Jeton d’actualisation* - Les jetons d’actualisation permettent d’acquérir de nouveaux jetons d’ID et d’accès dans un flux OAuth 2.0. Ils permettent à votre application d’obtenir un accès à long terme à des ressources au nom d’un utilisateur, et ce sans nécessiter l’intervention de l’utilisateur. Les jetons d’actualisation sont opaques pour votre application. Ils sont émis par Azure AD B2C et ne peuvent être inspectés et interprétés que par Azure AD B2C. Les jetons d’actualisation sont de longue durée. Toutefois, quand vous écrivez votre application, faites en sorte qu’elle n’attende pas un jeton d’actualisation d’une durée particulière. Les jetons d’actualisation peuvent être rendus non valides à tout moment, et ce pour diverses raisons. Pour savoir si un jeton d’actualisation est valide, votre application doit tenter de l’échanger en faisant une demande de jeton auprès d’Azure AD B2C. C’est la seule façon de faire. Quand vous échangez un jeton d’actualisation contre un nouveau jeton, vous recevez un nouveau jeton d’actualisation dans la réponse du jeton. Enregistrez le nouveau jeton d’actualisation. Il remplace le jeton d’actualisation utilisé précédemment dans la requête. Cette action garantit que vos jetons d’actualisation resteront valides le plus longtemps possible. Notez que les applications à page unique utilisant le flux de code d’autorisation avec PKCE ont toujours une durée de vie de jeton d’actualisation de 24 heures. [En savoir plus sur les implications pour la sécurité des jetons d’actualisation dans le navigateur](../active-directory/develop/reference-third-party-cookies-spas.md#security-implications-of-refresh-tokens-in-the-browser).

## <a name="endpoints"></a>Points de terminaison

Une [application inscrite](tutorial-register-applications.md) reçoit des jetons et communique avec Azure AD B2C en transmettant les requêtes à ces points de terminaison :

- `https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/oauth2/v2.0/authorize`
- `https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/oauth2/v2.0/token`

Les jetons de sécurité que votre application reçoit d’Azure AD B2C peuvent provenir des points de terminaison `/authorize` ou `/token`. Lorsque les jetons d’ID sont acquis à partir du point de terminaison `/authorize`, cela se fait à l'aide du [flux implicite](implicit-flow-single-page-application.md), souvent employé en cas d’ouverture de session sur des applications web JavaScript. Lorsque les jetons d’ID sont acquis à partir du point de terminaison `/token`, cela se fait à l'aide du [flux de code confidentiel](openid-connect.md#get-a-token), qui cache le jeton au navigateur.

## <a name="claims"></a>Revendications

Lorsque vous utilisez Azure AD B2C, vous disposez d’un contrôle affiné du contenu de vos jetons. Vous pouvez configurer des [flux d’utilisateur](user-flow-overview.md) et des [stratégies personnalisées](custom-policy-overview.md) pour envoyer certains ensembles de données utilisateur dans les revendications requises pour votre application. Ces revendications peuvent inclure des propriétés standard telles que **displayName** et **emailAddress**. Vos applications peuvent utiliser ces revendications pour authentifier de manière sécurisée les utilisateurs et les demandes.

Les revendications dans les jetons d’ID ne sont pas retournées dans un ordre particulier. De nouvelles revendications peuvent être introduites dans les jetons d’ID à tout moment. Votre application ne doit pas s’arrêter lors de l’ajout de nouvelles revendications. Vous pouvez également inclure des [attributs utilisateur personnalisés](user-flow-custom-attributes.md) dans vos revendications.

Le tableau suivant énumère les revendications auxquelles vous pouvez vous attendre avec les jetons d’identification et les jetons d’accès émis par Azure AD B2C.

| Name | Revendication | Valeur d'exemple | Description |
| ---- | ----- | ------------- | ----------- |
| Public visé | `aud` | `90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6` | Identifie le destinataire du jeton. Pour Azure AD B2C, l’audience est l’ID de l’application. Votre application doit valider cette valeur et rejeter le jeton s’il ne correspond pas. Audience est synonyme de ressource. |
| Émetteur | `iss` |`https://<tenant-name>.b2clogin.com/775527ff-9a37-4307-8b3d-cc311f58d925/v2.0/` | Identifie le service d’émission de jeton de sécurité (STS) qui construit et retourne le jeton. Elle identifie également le répertoire dans lequel l’utilisateur a été authentifié. Votre application doit valider la revendication de l’émetteur de manière à s’assurer que le jeton provient bien du point de terminaison approprié. |
| Émis à | `iat` | `1438535543` | Heure à laquelle le jeton a été émis, représentée en heure epoch. |
| Heure d’expiration | `exp` | `1438539443` | Heure à laquelle le jeton n’est plus valide, représentée en heure epoch. Votre application doit utiliser cette revendication pour vérifier la validité de la durée de vie du jeton. |
| Pas avant | `nbf` | `1438535543` | Heure à laquelle le jeton est valide, représentée en heure epoch. Cette heure correspond généralement à l’heure à laquelle le jeton a été émis. Votre application doit utiliser cette revendication pour vérifier la validité de la durée de vie du jeton. |
| Version | `ver` | `1.0` | Version du jeton d’ID, telle que définie par Azure AD B2C. |
| Hachage de code | `c_hash` | `SGCPtt01wxwfgnYZy2VJtQ` | Le hachage de code n’est inclus dans un jeton d’ID que si ce dernier est émis en même temps qu’un code d’autorisation OAuth 2.0. Il peut servir à valider l’authenticité d’un code d’autorisation. Pour plus d’informations sur la façon d’effectuer cette validation, consultez la [spécification OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html).  |
| Hachage de jeton d’accès | `at_hash` | `SGCPtt01wxwfgnYZy2VJtQ` | Le hachage de jeton d’accès n’est inclus dans un jeton d’ID que si ce dernier est émis en même temps qu’un jeton d’accès OAuth 2.0. Un hachage de jeton d’accès peut servir à valider l’authenticité d’un jeton d’accès. Pour plus d’informations sur la façon d’effectuer cette validation, consultez la [spécification OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html).  |
| Valeur à usage unique | `nonce` | `12345` | La valeur à usage unique est une stratégie d’atténuation des attaques par relecture de jetons. Votre application peut spécifier une valeur à usage unique dans une demande d’autorisation à l’aide du paramètre de requête `nonce`. La valeur que vous fournissez dans la demande est émise sans aucune modification dans la revendication `nonce` du jeton d’ID. Cette revendication permet à votre application de comparer la valeur à celle spécifiée dans la requête. Votre application doit effectuer cette validation au cours du processus de validation du jeton d’ID. |
| Objet | `sub` | `884408e1-2918-4cz0-b12d-3aa027d7563b` | Principal sur lequel portent les assertions d’informations du jeton, comme l’utilisateur d’une application. Cette valeur est immuable et ne peut pas être réattribuée ou réutilisée. Vous pouvez l’utiliser pour effectuer des vérifications d’autorisation en toute sécurité, comme lorsque le jeton est utilisé pour accéder à une ressource. Par défaut, la revendication de l’objet est remplie avec l’ID d’objet de l’utilisateur dans le répertoire. |
| Référence de classe du contexte d’authentification | `acr` | Non applicable | Utilisé uniquement avec des stratégies plus anciennes. |
| Stratégie d’infrastructure de confiance | `tfp` | `b2c_1_signupsignin1` | Nom de la stratégie utilisée pour acquérir le jeton d’ID. |
| Moment de l’authentification | `auth_time` | `1438535543` | Heure à laquelle l’utilisateur a entré ses informations d’identification pour la dernière fois, représentée en heure epoch. Il n’existe aucune discrimination entre cette authentification, une nouvelle connexion, une session d’authentification unique (SSO) ou un autre type de connexion. `auth_time` correspond à la dernière fois que l’application (ou l’utilisateur) a initié une tentative d’authentification auprès d'Azure AD B2C. La méthode utilisée pour l’authentification n’est pas différenciée. |
| Étendue | `scp` | `Read`| Autorisations accordées à la ressource pour un jeton d’accès. Plusieurs autorisations octroyées sont séparées par un espace. |
| Tiers autorisé | `azp` | `975251ed-e4f5-4efd-abcb-5f1a8f566ab7` | **ID** de l’application cliente qui a initié la demande. |

## <a name="configuration"></a>Configuration

Les propriétés suivantes servent à [gérer les durées de vie des jetons de sécurité](configure-tokens.md) émis par Azure AD B2C :

- **Durées de vie du jeton d’accès et d’ID (minutes)** : la durée de vie du jeton du porteur OAuth 2.0 utilisé pour accéder à une ressource protégée. La valeur par défaut est de 60 minutes. La valeur minimale (inclusive) est 5 minutes. La valeur maximale (inclusive) est 1 440 minutes.

- **Durée de vie du jeton d’actualisation (jours)** - La durée maximale avant laquelle un jeton d’actualisation peut être utilisé pour acquérir un nouveau jeton d’accès ou d’ID. La période concerne également l’acquisition d’un nouveau jeton d’actualisation si l’étendue `offline_access` a été accordée à votre application. La valeur par défaut est de 14 jours. La valeur minimale (inclusive) est de 1 jour. La valeur maximale (inclusive) est de 90 jours.

- **Durée de vie fenêtre glissante du jeton d’actualisation (jours)**  : une fois cette période écoulée, l’utilisateur est obligé de s’authentifier de nouveau, quelle que soit la période de validité de dernier jeton d’actualisation obtenu par l’application. Cette valeur ne peut être fournie que si le commutateur est défini sur **Délimité**. Elle doit être supérieure ou égale à la valeur **Durée de vie du jeton d’actualisation (jours)** . Si le commutateur est défini sur **Non délimité**, vous ne pouvez pas fournir une valeur spécifique. La valeur par défaut est de 90 jours. La valeur minimale (inclusive) est de 1 jour. La valeur maximale (inclusive) est de 365 jours.

Les cas d’usage suivants sont activés à l’aide de ces propriétés :

- Autoriser un utilisateur à rester connecté à une application mobile indéfiniment, tant qu’il est actif en permanence sur l’application. Vous pouvez définir le paramètre **Durée de vie fenêtre glissante du jeton d’actualisation (jours)** sur **Non délimité** dans votre flux d’utilisateur de connexion.
- Respectez les exigences de conformité et de sécurité de votre secteur en définissant les durées de vie correctes du jeton d’accès.

Ces paramètres ne sont pas disponibles pour les flux d’utilisateur de réinitialisation de mot de passe.

## <a name="compatibility"></a>Compatibilité

Les propriétés suivantes sont utilisées pour [gérer la compatibilité des jetons](configure-tokens.md) :

- **Revendication de l’émetteur (iss)**  : cette propriété identifie le client Azure AD B2C qui a émis le jeton. La valeur par défaut est `https://<domain>/{B2C tenant GUID}/v2.0/`. La valeur de `https://<domain>/tfp/{B2C tenant GUID}/{Policy ID}/v2.0/` inclut les ID pour le locataire Azure AD B2C et le flux d’utilisateur utilisé dans la demande de jeton. Si votre application ou bibliothèque a besoin qu’Azure AD B2C soit conforme à [la spécification OpenId Connect Discovery 1.0](https://openid.net/specs/openid-connect-discovery-1_0.html), utilisez cette valeur.

- **Revendication d’objet (obj)**  : cette propriété identifie l’entité pour laquelle le jeton indique des informations. La valeur par défaut,**ObjectID**, remplit la réclamation `sub` dans le jeton avec l’ID d’objet de l’utilisateur. La valeur **Non pris en charge** est fournie uniquement pour la compatibilité descendante. Il est recommandé de passer à **ObjectID** dès que possible.

- **ID de stratégie représentant la revendication** : cette propriété identifie le type de revendication dans lequel le nom de stratégie utilisé dans la demande de jeton est renseigné. La valeur par défaut est `tfp`. La valeur `acr` est fournie uniquement pour la compatibilité descendante.

## <a name="pass-through"></a>Requête directe

Au début de l’expérience utilisateur, Azure AD B2C reçoit un jeton d’accès d’un fournisseur d’identité. Azure AD B2C utilise ce jeton pour récupérer des informations sur l’utilisateur. Vous activez une revendication dans votre flux d’utilisateur pour [transmettre le jeton](idp-pass-through-user-flow.md) aux applications que vous inscrivez dans Azure AD B2C. Votre application doit utiliser un [flux utilisateur recommandé](user-flow-versions.md) pour tirer parti de la transmission du jeton en tant que revendication.

Pour l’instant, Azure AD B2C permet uniquement de passer le jeton d’accès de fournisseurs d’identité OAuth 2.0 (dont font partie Facebook et Google). Pour tous les autres fournisseurs d’identité, la revendication est retournée vide.

## <a name="validation"></a>Validation

Pour valider un jeton, votre application doit valider la signature et les revendications de ce jeton. Il existe de nombreuses bibliothèques open source pour valider les jetons JWT en fonction de votre langage préféré. Il est recommandé d’explorer ces options plutôt que d’implémenter votre propre logique de validation.

### <a name="validate-signature"></a>Valider la signature

Un jeton JWT contient trois segments : un *en-tête*, un *corps* et une *signature*. Le segment de signature peut être utilisé pour valider l’authenticité du jeton afin qu’il soit approuvé par votre application. Les jetons Azure AD B2C sont signés à l’aide d’algorithmes de chiffrement asymétrique standard, tels que RSA 256.

L’en-tête du jeton contient des informations sur la clé et la méthode de chiffrement utilisées pour signer le jeton :

```
{
        "typ": "JWT",
        "alg": "RS256",
        "kid": "GvnPApfWMdLRi8PDmisFn7bprKg"
}
```

La valeur de la revendication **alg** correspond à l’algorithme utilisé pour signer le jeton. La valeur de la revendication **kid** est la clé publique utilisée pour signer le jeton. À tout moment, Azure AD B2C peut signer un jeton à l’aide de l’un des ensembles de paires de clés publique-privée. Azure AD B2C alterne le jeu de clés possible de façon périodique. Votre application doit être écrite de manière à gérer automatiquement ces changements de clés. Pour vérifier les mises à jour apportées aux clés publiques utilisées par Azure AD B2C, spécifiez une fréquence raisonnable d’environ 24 heures. Pour gérer les modifications de clé inattendues, votre application doit être écrite de façon à récupérer à nouveau les clés publiques si elle reçoit une valeur **kid** inattendue.

Azure AD B2C a un point de terminaison des métadonnées OpenID Connect. À l’aide de ce point de terminaison, les applications peuvent demander des informations sur Azure AD B2C lors de l’exécution. Ces informations incluent les points de terminaison, le contenu des jetons et les clés de signature de jetons. Votre locataire Azure AD B2C contient un document de métadonnées JSON pour chaque stratégie. Le document de métadonnées est un objet JSON qui contient plusieurs informations utiles. Les métadonnées contiennent **jwks_uri** qui indique l’emplacement de l’ensemble des clés publiques utilisées pour signer les jetons. Cet emplacement est fourni ici, mais il est préférable d’extraire cet emplacement de manière dynamique à l’aide du document de métadonnées et d’analyser **jwks_uri** :

```
https://contoso.b2clogin.com/contoso.onmicrosoft.com/b2c_1_signupsignin1/discovery/v2.0/keys
```
Le document JSON situé à cette URL contient toutes les informations sur les clés publiques utilisées à un moment donné. Votre application peut utiliser les revendications `kid` dans l’en-tête JWT pour sélectionner la clé publique utilisée dans le document JSON pour signer un jeton donné. Elle peut ensuite procéder à la validation des signatures à l’aide de la clé publique correcte et de l’algorithme indiqué.

Le document de métadonnées pour la stratégie `B2C_1_signupsignin1` dans le locataire `contoso.onmicrosoft.com` se trouve à l’emplacement suivant :

```
https://contoso.b2clogin.com/contoso.onmicrosoft.com/b2c_1_signupsignin1/v2.0/.well-known/openid-configuration
```

Afin de déterminer la stratégie utilisée pour signer un jeton (et l’emplacement à partir duquel demander les métadonnées), deux options sont possibles. Tout d’abord, le nom de la stratégie est inclus dans la revendication `tfp` (par défaut) ou `acr` (configurée) dans le jeton. Vous pouvez analyser les revendications contenues dans le corps du jeton JWT par le biais d’un décodage base-64 du corps et la désérialisation de la chaîne JSON résultante. La revendication `tfp` ou `acr` est le nom de la stratégie qui a été utilisée pour émettre le jeton. L’autre option consiste à coder la stratégie dans la valeur du paramètre `state` lors de l’émission de la requête, puis à la décoder pour déterminer la stratégie qui a été utilisée. Les 2 méthodes sont valides.

Ce document ne contient pas la description de la procédure de validation de la signature. De nombreuses bibliothèques open source sont disponibles pour vous aider à valider un jeton.

### <a name="validate-claims"></a>Valider les revendications

Quand votre application ou API reçoit un jeton d’ID, elle doit également procéder à plusieurs vérifications sur les revendications dans le jeton d’ID. Les revendications suivantes doivent être vérifiées :

- **audience** - Vérifie que le jeton d’ID était bien destiné à votre application.
- **pas avant** et **heure d’expiration** - Vérifie que le jeton d’ID n’est pas arrivé à expiration.
- **émetteur** - Vérifie que le jeton a effectivement été émis à votre application par Azure AD B2C.
- **valeur à usage unique** - Stratégie d’atténuation des attaques par relecture de jetons.

Pour obtenir une liste complète des validations que votre application doit effectuer, reportez-vous à la [spécification OpenID Connect](https://openid.net).

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus d’informations sur [l’utilisation des jetons d’accès](access-tokens.md).

