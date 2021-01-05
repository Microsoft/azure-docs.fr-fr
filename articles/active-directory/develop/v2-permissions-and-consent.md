---
title: Étendues, autorisations et consentement de la plateforme d’identités Microsoft
description: Découvrez l’autorisation dans le point de terminaison de la plateforme d’identités Microsoft, ce qui comprend notamment les étendues, les autorisations et les consentements.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 09/23/2020
ms.author: ryanwi
ms.reviewer: hirsin, jesakowi, jmprieur, marsma
ms.custom: aaddev, fasttrack-edit, contperf-fy21q1, identityplatformtop40
ms.openlocfilehash: 14b6d6ecc6523199102fd3ef9370fe901c4ff51d
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97355696"
---
# <a name="permissions-and-consent-in-the-microsoft-identity-platform-endpoint"></a>Autorisations et consentement dans le point de terminaison de la plateforme d’identités Microsoft

Les applications qui s’intègrent à la plateforme d’identité Microsoft suivent un modèle d’autorisation permettant aux utilisateurs et aux administrateurs de contrôler l’accès aux données. L’implémentation de ce modèle d’autorisation a été mise à jour sur le point de terminaison de la plateforme d’identités Microsoft et elle modifie la façon dont une application doit interagir avec la plateforme d’identités Microsoft. Cet article aborde les concepts de base de ce modèle d’autorisation, notamment les étendues, les autorisations et le consentement.

## <a name="scopes-and-permissions"></a>Étendues et autorisations

La plateforme d’identité Microsoft implémente le protocole d’autorisation [OAuth 2.0](active-directory-v2-protocols.md). OAuth 2.0 est une méthode par le biais de laquelle une application tierce peut accéder aux ressources hébergées sur le web au nom d’un utilisateur. Les ressources hébergées sur le web qui s’intègrent à la plateforme d’identité Microsoft présentent un identificateur de ressource, également appelé *URI d’ID d’application*. Voici, par exemple, quelques-unes des ressources hébergées sur le Web de Microsoft :

* Microsoft Graph : `https://graph.microsoft.com`
* API de messagerie Microsoft 365 : `https://outlook.office.com`
* Azure Key Vault : `https://vault.azure.net`

> [!NOTE]
> Nous vous recommandons fortement d’utiliser Microsoft Graph à la place de l’API de messagerie Microsoft 365, etc.

Cela s’applique également aux ressources tierces intégrées à la plateforme d’identité Microsoft. Ces ressources peuvent également définir un ensemble d’autorisations à utiliser pour diviser la fonctionnalité de cette ressource en fragments plus réduits. Par exemple, [Microsoft Graph](https://graph.microsoft.com) a défini des autorisations pour effectuer les tâches suivantes, entre autres :

* Lire le calendrier d’un utilisateur
* Écrire dans le calendrier d’un utilisateur
* Envoi de messages en tant qu’utilisateur

En définissant ces types d’autorisations, la ressource dispose d’un contrôle précis sur ses données et sur l’exposition de la fonctionnalité d’API. Une application tierce peut demander ces autorisations à des utilisateurs et des administrateurs. Ces derniers doivent approuver la requête avant que l’application puisse accéder aux données ou agir pour le compte d’un utilisateur. En fragmentant les fonctionnalités de la ressource en ensembles plus réduits d’autorisations, il est possible de créer des applications tierces pour qu’elles demandent uniquement les autorisations nécessaires à leur fonctionnement. Les utilisateurs et les administrateurs peuvent connaître avec précision les données auxquelles l’application a accès. Ainsi, ils seront moins inquiets quant à une éventuelle intention malveillante. Les développeurs doivent toujours respecter le concept de moindre privilège et demander uniquement les autorisations dont ils ont besoin pour faire fonctionner leurs applications.

Dans OAuth 2.0, ces types d’autorisations sont appelés des *étendues*. Ils sont également souvent appelés *autorisations*. Une autorisation est représentée dans la plateforme d’identité Microsoft sous forme de valeur de chaîne. Toujours dans l’exemple Microsoft Graph, la valeur de chaîne pour chaque autorisation est la suivante :

* Lire le calendrier d’un utilisateur en utilisant `Calendars.Read`
* Écrire dans le calendrier d’un utilisateur en utilisant `Calendars.ReadWrite`
* Envoi de messages en tant qu’utilisateur en utilisant `Mail.Send`

Généralement, une application peut demander ces autorisations en spécifiant les étendues dans les demandes dirigées vers le point de terminaison d’autorisation de la plateforme d’identités Microsoft. Toutefois, certaines autorisations à privilèges élevés peuvent uniquement être accordées par le biais du consentement de l’administrateur. Elles sont demandées/accordées à l’aide du [point de terminaison de consentement de l’administrateur](#admin-restricted-permissions). Lisez la suite pour en savoir plus.

## <a name="permission-types"></a>Types d'autorisations

La plateforme d’identité Microsoft prend en charge deux types d’autorisations : les **autorisations déléguées** et les **autorisations d’application**.

* Les **autorisations déléguées** sont utilisées par les applications pour lesquelles un utilisateur est connecté et présent. Pour ces applications, l’utilisateur ou un administrateur accorde les autorisations que l’application demande. Ensuite, l’application se voit déléguer une autorisation d’agir pour le compte de l’utilisateur connecté lors des appels à une ressource cible. Certaines autorisations déléguées peuvent être accordées par des utilisateurs non administrateurs, mais certaines autorisations à privilèges élevés requièrent le [consentement de l’administrateur](#admin-restricted-permissions). Pour connaître les rôles administrateur habilités à donner leur consentement pour les autorisations déléguées, consultez [Autorisations du rôle administrateur dans Azure AD](../roles/permissions-reference.md).

* Les **autorisations d’application** sont utilisées par les applications qui s’exécutent sans qu’un utilisateur soit connecté et présent (les applications qui s’exécutent en tant que services ou démons en arrière-plan, par exemple).  Les autorisations de l’application peuvent uniquement être [accordées par un administrateur](#requesting-consent-for-an-entire-tenant).

Les _autorisations effectives_ correspondent aux autorisations accordées à votre application lorsqu’elle envoie des requêtes à une ressource cible. Vous devez comprendre la différence entre les autorisations déléguées, les autorisations d’application et les autorisations effectives que votre application reçoit lorsqu’elle interroge la ressource cible.

- Pour les autorisations déléguées, les _autorisations effectives_ de votre application correspondent au niveau de privilège le moins élevé entre les autorisations déléguées que l’application a reçues (par le biais d’un consentement) et les privilèges de l’utilisateur actuellement connecté. Votre application ne peut jamais avoir plus de privilèges que l’utilisateur connecté. Au sein des organisations, les privilèges de l’utilisateur connecté peuvent être déterminés par la stratégie ou l’appartenance à un ou plusieurs rôles d’administrateur. Pour connaître les rôles administrateur habilités à donner leur consentement pour les autorisations déléguées, consultez [Autorisations du rôle administrateur dans Azure AD](../roles/permissions-reference.md).

   Supposons que votre application ait reçu l’autorisation déléguée _User.ReadWrite.All_. Cette autorisation permet nominalement à votre application de lire et mettre à jour le profil de chaque utilisateur dans une organisation. Si l’utilisateur connecté est un administrateur général, votre application est en mesure de mettre à jour le profil de chaque utilisateur de l’organisation. Toutefois, si l’utilisateur connecté n’a pas de rôle d’administrateur, votre application peut uniquement mettre à jour le profil de l’utilisateur connecté. Elle ne peut pas mettre à jour les profils des autres utilisateurs de l’organisation, car l’utilisateur pour lequel elle est autorisée à agir n’a pas ces privilèges.

- Pour les autorisations d’application, les _autorisations effectives_ de votre application correspondent au niveau complet des privilèges impliqués par l’autorisation. Par exemple, une application disposant de l’autorisation d’application _User.ReadWrite.All_ peut mettre à jour le profil de chaque utilisateur de l’organisation.

## <a name="openid-connect-scopes"></a>Étendues OpenId Connect

L'implémentation d'OpenID Connect sur la plateforme d'identités Microsoft comprend quelques étendues bien définies qui sont également hébergées sur l'instance de Microsoft Graph : `openid`, `email`, `profile` et `offline_access`. Les étendues OpenID Connect `address` et `phone` ne sont pas prises en charge.

En demandant les étendues OIDC et un jeton, vous obtiendrez un jeton pour appeler le [point de terminaison UserInfo](userinfo.md).

### <a name="openid"></a>openid

Si une application exécute la connexion à l’aide [d’OpenID Connect](active-directory-v2-protocols.md), elle doit solliciter l’étendue `openid`. L’étendue `openid` s’affiche sur la page de consentement du compte professionnel, en tant qu’autorisation de connexion, et sur la page de consentement du compte Microsoft personnel en tant qu’autorisation « Afficher votre profil et se connecter aux applications et aux services à l’aide de votre compte Microsoft ». Avec cette autorisation, une application peut recevoir un identifiant utilisateur unique sous la forme de la revendication `sub`. Elle permet également à l’application d’accéder au point de terminaison des informations utilisateur. L’étendue `openid` peut être utilisée sur le point de terminaison des jetons de la plateforme d’identités Microsoft pour acquérir des jetons d’ID. L’application peut utiliser ces derniers pour l’authentification.

### <a name="email"></a>email

L’étendue `email` peut être utilisée avec l’étendue `openid` ainsi que d’autres. Elle permet à l’application d’accéder à l’adresse de messagerie principale de l’utilisateur sous la forme de la revendication `email`. La revendication `email` est incluse dans un jeton uniquement si une adresse e-mail est associée au compte d’utilisateur, ce qui n’est pas toujours le cas. Si elle utilise l’étendue `email`, votre application doit être préparée à faire face à l’éventualité où la revendication `email` n’existerait pas dans le jeton.

### <a name="profile"></a>profile

L’étendue `profile` peut être utilisée avec l’étendue `openid` ainsi que d’autres. Elle permet à l’application d’accéder à une quantité d’informations importante sur l’utilisateur, notamment le prénom de l’utilisateur, son nom de famille, son nom d’utilisateur privilégié et l’ID d’objet. Pour obtenir la liste complète des revendications de profil disponibles dans le paramètre id_token pour un utilisateur donné, consultez la page [de référence `id_tokens`](id-tokens.md).

### <a name="offline_access"></a>offline_access

[L’étendue `offline_access`](https://openid.net/specs/openid-connect-core-1_0.html#OfflineAccess) permet à votre application d’accéder aux ressources pour le compte de l’utilisateur pendant une période prolongée. Dans la page de consentement, cette étendue apparaît comme l’autorisation « Conserver l’accès aux données auxquelles vous lui avez donné l’accès ». Lorsqu’un utilisateur approuve l’étendue `offline_access`, votre application peut recevoir les jetons d’actualisation du point de terminaison des jetons de la plateforme d’identités Microsoft. Les jetons d’actualisation sont de longue durée. Votre application peut obtenir de nouveaux jetons d’accès lorsque les plus anciens arrivent à expiration.

> [!NOTE]
> Cette autorisation s’affiche sur tous les écrans de consentement du jour, même pour les flux qui ne fournissent pas de jeton d’actualisation (le [flux implicite](v2-oauth2-implicit-grant-flow.md)). Cela permet de couvrir les scénarios dans lesquels un client peut commencer dans le cadre du flux implicite, puis passer au flux de code où un jeton d’actualisation est attendu.

Sur la plateforme d’identités Microsoft (requêtes adressées au point de terminaison v2.0), votre application doit demander explicitement l’étendue `offline_access` pour recevoir les jetons d’actualisation. Ainsi, lorsque vous échangez un code d’autorisation dans le [flux de code d’autorisation OAuth 2.0](active-directory-v2-protocols.md), vous recevez uniquement un jeton d’accès du point de terminaison `/token`. Le jeton d’accès est valide pendant une courte durée : il arrive généralement à expiration en une heure. À ce stade, votre application doit rediriger l’utilisateur vers le point de terminaison `/authorize` afin de récupérer un nouveau code d’autorisation. Pendant ce réacheminement, en fonction du type d’application, l’utilisateur peut devoir entrer à nouveau ses informations d’identification ou accepter une nouvelle fois les autorisations.

Pour en savoir plus sur la récupération et l’utilisation des jetons d’actualisation, consultez la page [Référence sur le protocole de la plateforme d’identités Microsoft](active-directory-v2-protocols.md).

## <a name="requesting-individual-user-consent"></a>Demande de consentement d’utilisateur individuel

Dans une demande d’autorisation [OpenID Connect ou OAuth 2.0](active-directory-v2-protocols.md), une application peut demander les autorisations nécessaires à l’aide du paramètre de requête `scope`. Par exemple, lorsqu’un utilisateur se connecte à une application, cette dernière envoie une requête semblable à l’exemple ci-dessous (avec des sauts de ligne ajoutés pour une meilleure lisibilité) :

```HTTP
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=
https%3A%2F%2Fgraph.microsoft.com%2Fcalendars.read%20
https%3A%2F%2Fgraph.microsoft.com%2Fmail.send
&state=12345
```

Le paramètre `scope` correspond à une liste d’autorisations déléguées séparées par des espaces, demandées par l’application. Chaque autorisation est indiquée par l’ajout de la valeur correspondante à l’identificateur de la ressource (URI d’ID d’application). Dans l’exemple de requête, l’application nécessite l’autorisation de lecture du calendrier de l’utilisateur et d’envoi de messages au nom de l’utilisateur.

Une fois que l’utilisateur a entré ses informations d’identification, le point de terminaison de la plateforme d’identités Microsoft recherche un enregistrement correspondant du *consentement d’utilisateur*. Si, par le passé, l’utilisateur n’a jamais accepté les autorisations demandées et qu’un administrateur n’a jamais accepté ces autorisations pour le compte de toute l’organisation, le point de terminaison de la plateforme d’identités Microsoft demande à l’utilisateur d’accorder les autorisations demandées.

> [!NOTE]
>À ce stade, les autorisations `offline_access` (« Conserver l’accès aux données auxquelles vous lui avez donné l’accès ») et `user.read` (« Vous connecter et lire votre profil ») sont automatiquement incluses dans le consentement initial pour une application.  Ces autorisations sont généralement requises pour la fonctionnalité d’application appropriée : `offline_access` fournit à l’application l’accès aux jetons d’actualisation, critiques pour les applications natives et web, tandis que `user.read` donne accès à la revendication `sub`, ce qui permet au client ou à l’application d’identifier correctement l’utilisateur au fil du temps et d’accéder aux informations utilisateur de base.

![Capture d’écran de consentement dans le compte professionnel](./media/v2-permissions-and-consent/work_account_consent.png)

Lorsque l’utilisateur approuve la demande d’autorisation, le consentement est enregistré de manière à ce que l’utilisateur n’ait pas à le fournir à nouveau lors des connexions suivantes à l’application.

## <a name="requesting-consent-for-an-entire-tenant"></a>Demande de consentement d’un client entier

Souvent, lorsqu’une organisation achète une licence ou un abonnement à une application, elle souhaite configurer proactivement l’application afin que tous les membres de l’organisation puissent l’utiliser. Dans le cadre de cette procédure, un administrateur peut autoriser l’application à agir au nom de n’importe quel utilisateur au sein du locataire. Si l’administrateur donne son consentement pour l’intégralité du locataire, les utilisateurs de l’organisation ne voient pas de page de consentement pour l’application.

Pour demander le consentement pour les autorisations déléguées pour tous les utilisateurs d’un locataire, votre application peut utiliser le point de terminaison de consentement de l’administrateur.

En outre, les applications doivent utiliser le point de terminaison de consentement de l’administrateur pour demander les autorisations d’application.

## <a name="admin-restricted-permissions"></a>Autorisations restreintes aux administrateurs

Certaines autorisations à privilège élevé de l’écosystème Microsoft peuvent être définies sur *restreintes aux administrateurs*. Ces types d’autorisations peuvent être illustrés par ces exemples :

* Lire les profils complets de tous les utilisateurs à l’aide de `User.Read.All`
* Écrire des données dans le répertoire d’une organisation à l’aide de `Directory.ReadWrite.All`
* Lire tous les groupes dans le répertoire d’une organisation à l’aide de `Groups.Read.All`

Si un utilisateur consommateur peut accorder à une application l’accès à ce type de données, les utilisateurs d’organisation sont limités lorsqu’il s’agit d’octroyer l’accès au même jeu de données d’entreprise sensibles. Si votre application requiert l’accès à l’une de ces autorisations d’un utilisateur de l’organisation, ce dernier recevra un message d’erreur indiquant qu’il n’est pas autorisé à donner son consentement pour les permissions de votre application.

Si votre application requiert l’accès aux étendues restreintes aux administrateurs pour les organisations, vous devez demander l’autorisation directement à un administrateur d’entreprise également à l’aide du point de terminaison de consentement de l’administrateur, décrit ci-dessous.

Si l’application demande des autorisations déléguées à privilèges élevés et qu’un administrateur accorde ces autorisations via le point de terminaison de consentement de l’administrateur, le consentement est accordé à tous les utilisateurs du locataire.

Si l’application demande des permissions d’application et qu’un administrateur accorde ces autorisations via le point de terminaison de consentement administrateur, cette attribution n’est pas effectuée pour le compte d’un utilisateur spécifique. L’application cliente reçoit les autorisations *directement*. Ces types d’autorisations sont uniquement utilisés par les services démon et d’autres applications non interactives qui s’exécutent en arrière-plan.

## <a name="using-the-admin-consent-endpoint"></a>Utilisation du point de terminaison de consentement administrateur

> [!NOTE]
> Veuillez noter qu’après avoir accordé le consentement administrateur à l’aide du point de terminaison de consentement administrateur, vous avez terminé d’accorder le consentement administrateur et les utilisateurs n’ont pas besoin d’effectuer d’actions supplémentaires. Après avoir accordé le consentement administrateur, les utilisateurs peuvent obtenir un jeton d’accès grâce au flux d’authentification standard. Le jeton d’accès en question aura toutes les autorisations nécessaires.

Lorsqu’un administrateur d’entreprise utilise votre application et qu’il est dirigé vers le point de terminaison d’autorisation, la plateforme d’identité Microsoft détecte le rôle de l’utilisateur et lui demande s’il souhaite donner son consentement pour le compte de l’intégralité du locataire pour les autorisations que vous avez demandées. Toutefois, il existe également un point de terminaison de consentement de l’administrateur dédié que vous pouvez utiliser si vous souhaitez demander proactivement qu’un administrateur accorde son autorisation pour le compte de l’intégralité du locataire. Vous devez également utiliser ce point de terminaison pour demander des permissions d’application (qui ne peuvent pas être demandées à l’aide du point de terminaison d’autorisation).

Si vous suivez ces étapes, votre application peut demander des autorisations pour tous les utilisateurs d’un locataire, notamment les étendues restreintes aux administrateurs. Il s’agit d’une opération à privilèges élevés qui doit être effectuée uniquement si elle est nécessaire dans votre scénario.

Pour voir un exemple de code qui implémente les étapes, consultez [l’exemple d’étendues restreintes aux administrateurs](https://github.com/Azure-Samples/active-directory-dotnet-admin-restricted-scopes-v2).

### <a name="request-the-permissions-in-the-app-registration-portal"></a>Demander les autorisations dans le portail d’inscription de l’application

Les applications sont en mesure de noter les autorisations dont elles ont besoin (autorisations déléguées et d'application) dans le portail d’inscription des applications.  Il est ainsi possible d’utiliser l’étendue `/.default` et l’option « Accorder un consentement administrateur » du portail Azure.  En général, il est recommandé de veiller à ce que les autorisations définies de manière statique pour une application donnée constituent un sur-ensemble des autorisations qui seront demandées de façon dynamique/incrémentielle.

> [!NOTE]
>Les autorisations d’application ne peuvent être demandées qu’à l’aide de [`/.default`](#the-default-scope). Dès lors, si votre application requiert des autorisations d'application, vérifiez qu'elles sont répertoriées dans le portail d’inscription des applications.

#### <a name="to-configure-the-list-of-statically-requested-permissions-for-an-application"></a>Configuration de la liste des autorisations demandées de manière statique pour une application

1. Accédez à votre application dans l’environnement [Inscriptions d’applications du portail Azure](https://go.microsoft.com/fwlink/?linkid=2083908) ou [créez une application](quickstart-register-app.md), si ce n’est pas déjà fait.
2. Recherchez la section **Autorisations des API** et, dans les autorisations des API, cliquez sur Ajouter une autorisation.
3. Sélectionnez **Microsoft Graph** dans la liste des API disponibles, puis ajoutez les autorisations dont votre application a besoin.
3. **Enregistrez** l’inscription de l’application.

### <a name="recommended-sign-the-user-into-your-app"></a>Recommandé : connecter l’utilisateur à votre application

En général, lorsque vous créez une application qui utilise le point de terminaison de consentement de l’administrateur, l’application doit disposer d’une page ou vue dans laquelle l’administrateur peut approuver ses autorisations. Cette page peut faire partie du flux d’inscription de l’application, des paramètres de l’application, ou ce peut être un flux de connexion dédié. Dans de nombreux cas, il est judicieux pour l’application d’afficher la vue de « connexion » uniquement après qu’un utilisateur se soit connecté avec un compte Microsoft professionnel ou scolaire.

Lorsque vous connectez l’utilisateur à votre application, vous pouvez identifier l’organisation à laquelle l’administrateur appartient, avant de lui demander d’approuver les autorisations nécessaires. Même si cela n’est pas strictement nécessaire, cela peut vous aider à créer une expérience plus intuitive pour les utilisateurs de l’organisation. Pour connecter l’utilisateur, suivez nos [tutoriels sur le protocole de la plateforme d’identités Microsoft](active-directory-v2-protocols.md).

### <a name="request-the-permissions-from-a-directory-admin"></a>Demander les autorisations à un administrateur d’annuaire

Lorsque vous êtes prêt à demander les autorisations à l’administrateur de votre organisation, vous pouvez rediriger l’utilisateur vers le *point de terminaison de consentement administrateur* de la plateforme d’identités Microsoft.

```HTTP
// Line breaks are for legibility only.
GET https://login.microsoftonline.com/{tenant}/v2.0/adminconsent?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&state=12345
&redirect_uri=http://localhost/myapp/permissions
&scope=
https://graph.microsoft.com/calendars.read
https://graph.microsoft.com/mail.send
```


| Paramètre        | Condition        | Description                                                                                |
|:--------------|:--------------|:-----------------------------------------------------------------------------------------|
| `tenant` | Obligatoire | Le client d’annuaire auquel vous souhaitez demander l’autorisation. Peut être fourni au format GUID ou sous forme de nom convivial OU référencé de manière générique avec des organisations comme indiqué dans l’exemple. N’utilisez pas « common », car les comptes personnels ne peuvent pas fournir le consentement de l’administrateur, sauf dans le contexte d’un locataire. Pour garantir une meilleure compatibilité avec les comptes personnels qui gèrent les locataires, utilisez l’ID de locataire, dans la mesure du possible. |
| `client_id` | Obligatoire | L’**ID (client) d’application** attribué à votre application par l’environnement [Inscriptions d’applications du portail Azure](https://go.microsoft.com/fwlink/?linkid=2083908). |
| `redirect_uri` | Obligatoire |L'URI de redirection où vous souhaitez que la réponse soit envoyée pour être gérée par votre application. Il doit correspondre exactement à l’un des URI de redirection que vous avez inscrits dans le portail d’inscription des applications. |
| `state` | Recommandé | Une valeur incluse dans la requête, qui sera également renvoyée dans la réponse de jeton. Il peut s’agir d’une chaîne du contenu de votre choix. Utilisez l’état pour encoder les informations sur l’état de l’utilisateur dans l’application avant la requête d’authentification, comme la page ou la vue sur laquelle ou laquelle il était positionné. |
|`scope`        | Obligatoire        | Définit l’ensemble des autorisations demandées par l’application. Il peut s’agir d’étendues statiques (utilisant [`/.default`](#the-default-scope)) ou dynamiques.  Cela peut inclure les étendues OIDC (`openid`, `profile`, `email`). S'il vous faut des autorisations d’application, vous devez utiliser `/.default` pour demander la liste d’autorisations configurée statiquement.  |


À ce stade, Azure AD nécessite qu’un administrateur client se connecte pour terminer la demande. L’administrateur est invité à approuver toutes les autorisations que vous avez demandées dans le paramètre `scope`.  Si vous avez utilisé une valeur (`/.default`) statique, celle-ci fonctionne comme le point de terminaison de consentement administrateur v 1.0 et demande un consentement pour toutes les étendues trouvées dans les autorisations requises pour l’application.

#### <a name="successful-response"></a>Réponse correcte

Si l’administrateur approuve les autorisations pour votre application, la réponse correcte ressemble à ce qui suit :

```HTTP
GET http://localhost/myapp/permissions?tenant=a8990e1f-ff32-408a-9f8e-78d3b9139b95&state=state=12345&admin_consent=True
```

| Paramètre | Description |
| --- | --- |
| `tenant` | Client d’annuaire ayant accordé à votre application les autorisations demandées au format GUID. |
| `state` | Une valeur incluse dans la requête qui sera également renvoyée dans la réponse de jeton. Il peut s’agir d’une chaîne du contenu de votre choix. La valeur d’état est utilisée pour coder les informations sur l’état de l’utilisateur dans l’application avant la requête d’authentification, comme la page ou l’écran sur lequel ou laquelle il était positionné. |
| `admin_consent` | Sera défini sur `True`. |

#### <a name="error-response"></a>Réponse d’erreur

Si l’administrateur n’approuve pas les autorisations pour votre application, la réponse d’échec ressemble à ce qui suit :

```HTTP
GET http://localhost/myapp/permissions?error=permission_denied&error_description=The+admin+canceled+the+request
```

| Paramètre | Description |
| --- | --- |
| `error` | Une chaîne de code d’erreur pouvant être utilisée pour classer les types d’erreurs se produisant, et pouvant être utilisée pour intervenir face aux erreurs. |
| `error_description` | Un message d’erreur spécifique qui peut aider un développeur à identifier la cause principale d’une erreur. |

Une fois que vous avez reçu une réponse correcte du point de terminaison de consentement de l’administrateur, votre application a acquis les autorisations qu’elle avait demandées. Vous pouvez alors demander un jeton pour la ressource souhaitée.

## <a name="using-permissions"></a>Utilisation des autorisations

Une fois que l’utilisateur accepte les autorisations pour votre application, cette dernière peut acquérir des jetons d’accès représentant l’autorisation de votre application à accéder, dans une certaine capacité, à une ressource. Un jeton d’accès peut être utilisé pour une ressource uniquement, mais l’encodage de ce jeton comporte les informations relatives à toutes les autorisations octroyées pour cette ressource à votre application. Pour acquérir un jeton d’accès, votre application peut transmettre une demande au point de terminaison des jetons de la plateforme d’identités Microsoft :

```HTTP
POST common/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/json

{
    "grant_type": "authorization_code",
    "client_id": "6731de76-14a6-49ae-97bc-6eba6914391e",
    "scope": "https://outlook.office.com/mail.read https://outlook.office.com/mail.send",
    "code": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq..."
    "redirect_uri": "https://localhost/myapp",
    "client_secret": "zc53fwe80980293klaj9823"  // NOTE: Only required for web apps
}
```

Vous pouvez utiliser le jeton d’accès obtenu dans les requêtes HTTP transmises à la ressource. Il indique de façon fiable à la ressource que votre application dispose de l’autorisation appropriée pour effectuer une tâche spécifique.

Pour en savoir plus sur le protocole OAuth 2.0 et sur la méthode pour obtenir des jetons d’accès, consultez la page de [Référence sur le protocole du point de terminaison de la plateforme d’identités Microsoft](active-directory-v2-protocols.md).

## <a name="the-default-scope"></a>Étendue /.default

Vous pouvez utiliser l’étendue `/.default` pour faciliter la migration de vos applications à partir du point de terminaison v1.0 vers le point de terminaison de la plateforme d’identités Microsoft. Il s’agit d’une étendue intégrée pour chaque application qui fait référence à la liste statique des autorisations configurées sur l’inscription d’application. Une valeur `scope``https://graph.microsoft.com/.default` est fonctionnellement identique aux points de terminaison v1.0 `resource=https://graph.microsoft.com`, autrement dit elle demande un jeton avec les étendues sur Microsoft Graph auxquelles l’application s’est inscrite dans le portail Azure.  Elle est créée à l’aide de l’URI de ressource + `/.default` (par exemple, si l’URI de ressource est `https://contosoApp.com`, l’étendue demandée est `https://contosoApp.com/.default`).  Pour savoir quand vous devez inclure une deuxième barre oblique afin de correctement demander le jeton, consultez la [section relative aux barres obliques de fin](#trailing-slash-and-default).

L’étendue /.default peut être utilisée dans n’importe quel flux OAuth 2.0, mais elle est nécessaire dans le [flux On-Behalf-Of](v2-oauth2-on-behalf-of-flow.md) et le [flux des informations d’identification du client](v2-oauth2-client-creds-grant-flow.md), de même que lors de l’utilisation du point de terminaison de consentement administrateur v2 pour demander des autorisations d’application.

> [!NOTE]
> Les clients ne peuvent pas combiner un consentement statique (`/.default`) et dynamique dans une seule demande. Par conséquent, `scope=https://graph.microsoft.com/.default+mail.read` entraîne une erreur en raison de la combinaison des types d’étendues.

### <a name="default-and-consent"></a>/.default et consentement

L’étendue `/.default` déclenche le comportement de point de terminaison v1.0 également pour `prompt=consent`. Elle demande le consentement pour toutes les autorisations inscrites par l’application, quelle que soit la ressource. Si elle est incluse dans le cadre de la demande, l’étendue `/.default` retourne un jeton qui contient les étendues pour la ressource demandée.

### <a name="default-when-the-user-has-already-given-consent"></a>/.default quand l’utilisateur a déjà donné son consentement

Comme `/.default` est fonctionnellement identique au comportement du point de terminaison v1.0 axé sur `resource`, elle s’accompagne aussi du comportement de consentement du point de terminaison v1.0. Autrement dit, `/.default` déclenche uniquement une invite de consentement si aucune autorisation n’a été accordée entre le client et la ressource par l’utilisateur. Si un tel consentement existe, un jeton qui contient toutes les étendues accordées par l’utilisateur pour cette ressource est alors retourné. Toutefois, si aucune autorisation n’a été accordée, ou si le paramètre `prompt=consent` a été fourni, une invite de consentement s’affiche pour toutes les étendues inscrites par l’application cliente.

#### <a name="example-1-the-user-or-tenant-admin-has-granted-permissions"></a>Exemple 1 : L’utilisateur, ou l’administrateur de locataire, a accordé des autorisations

Dans cet exemple, l’utilisateur (ou un administrateur de locataire) a accordé au client les autorisations Microsoft Graph `mail.read` et `user.read`. Si le client envoie une demande pour `scope=https://graph.microsoft.com/.default`, aucune invite de consentement ne s’affiche, quel que soit le contenu des autorisations inscrites par les applications clientes pour Microsoft Graph. Un jeton contenant les étendues `mail.read` et `user.read` doit être retourné.

#### <a name="example-2-the-user-hasnt-granted-permissions-between-the-client-and-the-resource"></a>Exemple 2 : L’utilisateur n’a pas accordé d’autorisations entre le client et la ressource

Dans cet exemple, aucun consentement pour l’utilisateur n’existe entre le client et Microsoft Graph. Le client s’est inscrit aux autorisations `user.read` et `contacts.read` ainsi qu’à l’étendue Azure Key Vault `https://vault.azure.net/user_impersonation`. Quand le client demande un jeton pour `scope=https://graph.microsoft.com/.default`, l’utilisateur voit un écran de consentement pour les étendues `user.read`, `contacts.read` et Key Vault `user_impersonation`. Le jeton renvoyé présente uniquement les étendues `user.read` et `contacts.read`, et peut uniquement être utilisé sur Microsoft Graph.

#### <a name="example-3-the-user-has-consented-and-the-client-requests-additional-scopes"></a>Exemple 3 : L’utilisateur a donné son consentement et le client demande des étendues supplémentaires

Dans cet exemple, l’utilisateur a déjà donné son consentement à `mail.read` pour le client. Le client s’est inscrit à l’étendue `contacts.read` dans son inscription. Lorsque le client effectue une demande pour un jeton à l’aide de `scope=https://graph.microsoft.com/.default` et demande un consentement via `prompt=consent`, l’utilisateur voit un écran de consentement uniquement pour toutes (et uniquement) les autorisations inscrites par l’application. `contacts.read` est présent dans l’écran de consentement, mais pas `mail.read`. Le jeton retourné concerne Microsoft Graph et contient `mail.read` et `contacts.read`.

### <a name="using-the-default-scope-with-the-client"></a>Utilisation de l’étendue /.default avec le client

Il existe un cas spécial d’étendue `/.default` quand un client demande sa propre étendue `/.default`. L’exemple suivant illustre ce cas de figure.

```HTTP
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
response_type=token            //code or a hybrid flow is also possible here
&client_id=9ada6f8a-6d83-41bc-b169-a306c21527a5
&scope=9ada6f8a-6d83-41bc-b169-a306c21527a5/.default
&redirect_uri=https%3A%2F%2Flocalhost
&state=1234
```

Cela génère un écran de consentement pour toutes les autorisations inscrites (si cela est applicable en fonction des descriptions ci-dessus du consentement et de `/.default`), puis retourne un jeton id_token, plutôt qu’un jeton d’accès.  Ce comportement existe pour certains clients hérités qui passent de la bibliothèque ADAL à la bibliothèque MSAL et **ne doit pas** être utilisé par les nouveaux clients ciblant le point de terminaison de la plateforme d’identités Microsoft.

### <a name="client-credentials-grant-flow-and-default"></a>Flux d’octroi des informations d’identification du client et /.default

Une autre utilisation de `/.default` consiste à demander des autorisations d’application (ou des *rôles*) dans une application non interactive telle qu’une application démon qui utilise le flux d’octroi des [informations d’identification du client](v2-oauth2-client-creds-grant-flow.md) pour appeler une API Web.

Pour créer des autorisations d’application (rôles) pour une API Web, consultez [Procédure : Ajouter des rôles d’application dans votre application](howto-add-app-roles-in-azure-ad-apps.md).

Les demandes d’informations d’identification du client dans votre application cliente **doivent** inclure `scope={resource}/.default`, où `{resource}` est l’API Web que votre application envisage d’appeler. L’émission d’une demande d’informations d’identification du client avec des autorisations d’application individuelles (rôles) n’est **pas** prise en charge. Toutes les autorisations d’application (rôles) qui ont été accordées pour cette API Web seront incluses dans le jeton d’accès retourné.

Pour accorder l’accès aux autorisations d’application que vous définissez, y compris l’octroi du consentement administrateur pour l’application, consultez [Démarrage rapide : Configurer une application cliente pour accéder à une API Web](quickstart-configure-app-access-web-apis.md).

### <a name="trailing-slash-and-default"></a>Barre oblique de fin et /.default

Certains URI de ressource sont dotés d'une barre oblique de fin (`https://contoso.com/` par opposition à `https://contoso.com`), ce qui peut entraîner des problèmes de validation des jetons.  Cela survient principalement en cas de demande de jeton pour la gestion des ressources Azure (`https://management.azure.com/`), qui présente une barre oblique de fin dans leur URI de ressource et dont la présence est requise lorsque le jeton est demandé.  Ainsi, en cas de demande de jeton pour `https://management.azure.com/` et à l'aide de `/.default`, vous devez demander `https://management.azure.com//.default` (notez de la double barre oblique).

En général, si vous avez validé l'émission du jeton et que celui-ci est rejeté par l’API qui doit l’accepter, ajoutez une deuxième barre oblique, puis réessayez. Cela est dû au fait que le serveur de connexion émet un jeton avec une audience correspondant aux URI du paramètre `scope`, avec suppression de `/.default`.  Si cette opération supprime la barre oblique de fin, le serveur de connexion continue de traiter la demande et la valide par rapport à l’URI de ressource, même s’ils ne correspondent plus, ce qui n'est pas conforme.

## <a name="troubleshooting-permissions-and-consent"></a>Résolution des problèmes d’autorisations et de consentement

Si vous, ou les utilisateurs de votre application, constatez des erreurs inattendues au cours du processus de consentement, consultez cet article pour connaître les étapes de dépannage : [Erreur inattendue lors du consentement à une application](../manage-apps/application-sign-in-unexpected-user-consent-error.md).

## <a name="next-steps"></a>Étapes suivantes

* [Jetons d’ID | Plateforme d’identités Microsoft](id-tokens.md)
* [Jetons d’accès | Plateforme d’identités Microsoft](access-tokens.md)
