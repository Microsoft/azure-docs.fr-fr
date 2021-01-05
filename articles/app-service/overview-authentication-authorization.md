---
title: Authentification et autorisation
description: Renseignez-vous sur la prise en charge de l’authentification et de l’autorisation intégrées dans Azure App Service et Azure Functions, et découvrez en quoi elle peut vous aider à protéger votre application contre tout accès non autorisé.
ms.assetid: b7151b57-09e5-4c77-a10c-375a262f17e5
ms.topic: article
ms.date: 07/08/2020
ms.reviewer: mahender
ms.custom: seodec18, fasttrack-edit, has-adal-ref
ms.openlocfilehash: 4f1aceed7e6a41a8ad4f3a062f23a796e31e34ad
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96601677"
---
# <a name="authentication-and-authorization-in-azure-app-service-and-azure-functions"></a>Authentification et autorisation dans Azure App Service et Azure Functions

Azure App Service offre une prise en charge intégrée de l’authentification et de l’autorisation, qui vous permet de connecter les utilisateurs et d’accéder aux données sans avoir à écrire beaucoup de code dans votre application web, votre API RESTful et votre back end mobile, ainsi que dans [Azure Functions](../azure-functions/functions-overview.md). Cet article explique comment App Service contribue à simplifier l’authentification et l’autorisation de votre application.

Pour mettre en place un système sécurisé d’authentification et d’autorisation, il faut avoir une connaissance approfondie de la sécurité, notamment de la fédération, du chiffrement, de la gestion des [jetons web JSON (JWT)](https://wikipedia.org/wiki/JSON_Web_Token), des [types d’autorisation](https://oauth.net/2/grant-types/), etc. App Service propose ces utilitaires pour vous permettre de consacrer davantage de temps et d’énergie à offrir de la valeur ajoutée à votre client.

> [!IMPORTANT]
> Il n’est pas obligatoire d’utiliser cette fonctionnalité pour l’authentification et l’autorisation. Vous pouvez utiliser les fonctionnalités de sécurité groupées dans l’infrastructure web de votre choix, ou vous pouvez écrire vos propres utilitaires. Cependant, gardez à l’esprit que [Chrome 80 apporte des changements cassants à son implémentation de SameSite pour les cookies](https://www.chromestatus.com/feature/5088147346030592) (date de publication aux alentours de mars 2020) et que l’authentification à distance personnalisée ou d’autres scénarios qui reposent sur l'envoi de cookies intersites peuvent être interrompus lorsque les navigateurs Chrome des clients sont mis à jour. La solution de contournement est complexe, car elle doit prendre en charge différents comportements SameSite pour différents navigateurs. 
>
> Les versions 2.1 (ou ultérieures) d’ASP.NET Core hébergées par App Service sont déjà corrigés pour cette modification avec rupture et gèrent correctement Chrome 80 et les navigateurs plus anciens. En outre, le même correctif pour ASP.NET Framework 4.7.2 a été déployé sur les instances App Service tout au long du mois de janvier 2020. Pour plus d’informations, consultez [Mise à jour de cookies SameSite Azure App Service](https://azure.microsoft.com/updates/app-service-samesite-cookie-update/).
>

> [!NOTE]
> La fonctionnalité d’authentification/autorisation est également parfois appelée « Authentification simple ».

> [!NOTE]
> L’activation de cette fonctionnalité entraînera la redirection automatique des **toutes** les requêtes HTTP non sécurisées adressées à votre application vers HTTPS, quel que soit le paramètre de configuration d’App Service pour [appliquer HTTPS](configure-ssl-bindings.md#enforce-https). Si nécessaire, vous pouvez désactiver cela via le paramètre `requireHttps` dans le [fichier de configuration des paramètres d’authentification](app-service-authentication-how-to.md#configuration-file-reference), mais vous devez veiller à ce qu’aucun jeton de sécurité ne soit transmis sur des connexions HTTP non sécurisées.

Pour plus d’informations sur les applications mobiles natives en particulier, consultez la page [Authentification et autorisation des utilisateurs pour les applications mobiles avec Azure App Service](/previous-versions/azure/app-service-mobile/app-service-mobile-auth).

## <a name="how-it-works"></a>Fonctionnement

### <a name="on-windows"></a>Sur Windows

Le module d’authentification et d’autorisation s’exécute dans le même bac à sable que le code de l’application. Lorsqu’il est activé, chaque requête HTTP entrante le traverse avant d’être géré par le code de l’application.

![Un diagramme d’architecture montrant les demandes interceptées par un processus dans le bac à sable de site qui interagit avec les fournisseurs d’identité avant d’autoriser le trafic vers le site déployé](media/app-service-authentication-overview/architecture.png)

Ce module gère plusieurs choses pour votre application :

- authentification des utilisateurs avec le fournisseur spécifié ;
- validation, stockage et actualisation des jetons ;
- gestion de la session authentifiée ;
- injection d’informations d’identité dans les en-têtes de demande.

Le module, configuré à l’aide des paramètres de l’application, s’exécute de façon distincte du code de l’application. Aucun Kit de développement logiciel (SDK), aucun langage spécifique ni aucune modification du code de l’application ne sont nécessaires. 

### <a name="on-containers"></a>Sur conteneurs

Le module d’authentification et d’autorisation s’exécute dans un conteneur distinct, isolé du code de votre application. En utilisant ce que l’on appelle le [Modèle ambassadeur](/azure/architecture/patterns/ambassador), il interagit avec le trafic entrant pour effectuer des fonctionnalités similaires à celles de Windows. Comme il ne s’exécute pas in-process, aucune intégration directe avec des infrastructures de langage spécifiques n’est possible. Toutefois, les informations pertinentes dont votre application a besoin sont transmises à l’aide d’en-têtes de demande, comme expliqué ci-dessous.

### <a name="userapplication-claims"></a>Revendications de l’utilisateur/application

Pour toutes les infrastructures de langage, App Service met les revendications du jeton entrant (qu’elles proviennent d’un utilisateur final authentifié ou d’une application cliente) à la disposition de votre code en les injectant dans les en-têtes de requête. Dans le cas des applications ASP.NET 4.6, App Service remplit [ClaimsPrincipal.Current](/dotnet/api/system.security.claims.claimsprincipal.current) avec les revendications de l’utilisateur authentifié, ce qui vous permet de suivre le modèle de code .NET standard, attribut `[Authorize]` compris. De même, pour les applications PHP, App Service remplit la variable `_SERVER['REMOTE_USER']`. Pour les applications Java, les revendications sont [accessibles depuis le servlet Tomcat](configure-language-java.md#authenticate-users-easy-auth).

Pour [Azure Functions](../azure-functions/functions-overview.md), `ClaimsPrincipal.Current` n’est pas rempli pour le code .NET, mais vous pouvez toujours trouver les revendications de l’utilisateur dans les en-têtes de requête ou obtenir l’objet `ClaimsPrincipal` à partir du contexte de la requête, ou même via un paramètre de liaison. Pour plus d’informations, consultez [Utilisation d’identités de clients](../azure-functions/functions-bindings-http-webhook-trigger.md#working-with-client-identities).

Pour plus d’informations, consultez la section [Revendications d’utilisateurs d’accès](app-service-authentication-how-to.md#access-user-claims).

> [!NOTE]
> À l’heure actuelle, ASP.NET Core ne prend pas en charge le remplissage de l’utilisateur actuel à l’aide de la fonctionnalité d’authentification/autorisation. Cependant, certains [composants intergiciels open source tiers](https://github.com/MaximRouiller/MaximeRouiller.Azure.AppService.EasyAuth) existent pour vous aider à combler cette lacune.
>

### <a name="token-store"></a>Magasin de jetons

App Service fournit un magasin de jetons intégré : il s’agit d’un référentiel de jetons associés aux utilisateurs de vos applications web, API ou applications mobiles natives. Dès que l’authentification est activée avec un fournisseur, l’application a immédiatement accès à ce magasin de jetons. Si le code de votre application doit accéder à des données de ces fournisseurs au nom de l’utilisateur, notamment : 

- publier sur le fil d’actualité Facebook de l’utilisateur authentifié ;
- lire les données d’entreprise de l’utilisateur à l’aide de l’API Microsoft Graph

Il est en général nécessaire d’écrire du code pour recueillir, stocker et actualiser ces jetons dans votre application. Avec le magasin de jetons, il suffit de [récupérer les jetons](app-service-authentication-how-to.md#retrieve-tokens-in-app-code) en temps utile et de [demander à App Service de les actualiser](app-service-authentication-how-to.md#refresh-identity-provider-tokens) lorsqu’ils ne sont plus valides. 

Les jetons d’ID, jetons d’accès et jetons d’actualisation sont mis en cache pour la session authentifiée et ne sont accessibles que par l’utilisateur associé.  

Si vous n’avez pas besoin d’utiliser des jetons dans votre application, vous pouvez désactiver le magasin de jetons dans la page **Authentification/autorisation** de votre application.

### <a name="logging-and-tracing"></a>Journalisation et suivi

Si vous [activez la journalisation des applications](troubleshoot-diagnostic-logs.md), les traces de l’authentification et de l’autorisation apparaîtront directement dans les fichiers journaux. Si une erreur d’authentification inattendue se produit, vous trouverez facilement tous les détails dans les journaux des applications existants. Si vous activez le [suivi des échecs des demandes](troubleshoot-diagnostic-logs.md), vous saurez exactement quel rôle le module d’authentification et d’autorisation a pu jouer dans l’échec d’une demande. Dans les journaux d’activité de suivi, recherchez les références à un module nommé `EasyAuthModule_32/64`. 

## <a name="identity-providers"></a>Fournisseurs d’identité

App Service utilise [l’identité fédérée](https://en.wikipedia.org/wiki/Federated_identity), dans laquelle un fournisseur d’identité tiers gère automatiquement l’identité des utilisateurs et le flux d’authentification. Cinq fournisseurs d’identité sont disponibles par défaut : 

| Fournisseur | Point de terminaison de connexion |
| - | - |
| [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) | `/.auth/login/aad` |
| [Compte Microsoft](../active-directory/develop/v2-overview.md) | `/.auth/login/microsoftaccount` |
| [Facebook](https://developers.facebook.com/docs/facebook-login) | `/.auth/login/facebook` |
| [Google](https://developers.google.com/identity/choose-auth) | `/.auth/login/google` |
| [Twitter](https://developer.twitter.com/en/docs/basics/authentication) | `/.auth/login/twitter` |
| Tout fournisseur [OpenID Connect](https://openid.net/connect/) (préversion) | `/.auth/login/<providerName>` |

Lorsque l’authentification et l’autorisation sont activées avec un de ces fournisseurs, son point de terminaison de connexion est accessible à des fins d’authentification de l’utilisateur et de validation des jetons d’authentification provenant du fournisseur. Vous pouvez proposer à vos utilisateurs toutes les options de connexion que vous souhaitez parmi celles-ci, en toute simplicité.

Un [« legacy extensibility path »][custom-auth] permet l’intégration d’autres fournisseurs d’identité ou d’une solution d’authentification personnalisée, mais cela n’est pas recommandé. Au lieu de cela, envisagez d’utiliser le support OpenID Connect.

## <a name="authentication-flow"></a>Flux d’authentification

Le flux d’authentification est identique pour tous les fournisseurs, mais il diffère selon que vous souhaitez ou non vous connecter avec le Kit de développement logiciel (SDK) du fournisseur :

- Sans le Kit SDK du fournisseur : l’application délègue la connexion fédérée à App Service. C’est généralement le cas avec les applications de navigateur, qui peuvent présenter la page de connexion du fournisseur à l’utilisateur. C’est le code du serveur qui gère le processus de connexion, c’est pourquoi il est également appelé _flux dirigé vers le serveur_ ou _flux serveur_. Ce cas s’applique aux applications de navigateur. Il concerne également les applications natives qui connectent les utilisateurs à l’aide du Kit SDK client Mobile Apps, car celui-ci ouvre un affichage web pour connecter les utilisateurs avec l’authentification App Service. 
- Avec le Kit SDK du fournisseur : l’application connecte manuellement les utilisateurs au fournisseur et soumet ensuite le jeton d’authentification à App Service pour validation. C’est généralement le cas avec les applications sans navigateur, qui ne peuvent pas présenter la page de connexion du fournisseur à l’utilisateur. C’est le code de l’application qui gère le processus de connexion, c’est pourquoi il est également appelé _flux dirigé vers le client_ ou _flux client_. Ce cas s’applique aux API REST, à [Azure Functions](../azure-functions/functions-overview.md) et aux clients de navigateur JavaScript, ainsi qu’aux applications de navigateur nécessitant plus de souplesse dans le processus de connexion. Il concerne également les applications mobiles natives qui connectent les utilisateurs à l’aide du Kit SDK du fournisseur.

> [!NOTE]
> Les appels entre une application de navigateur approuvée dans App Service et une autre API REST dans App Service ou [Azure Functions](../azure-functions/functions-overview.md) peuvent être authentifiés à l’aide du flux dirigé vers le serveur. Pour plus d’informations, consultez [Personnaliser les paramètres d’authentification et d’autorisation dans Azure App Service](app-service-authentication-how-to.md).
>

Le tableau ci-dessous montre les étapes du flux d’authentification.

| Étape | Sans le Kit SDK du fournisseur | Avec le Kit SDK du fournisseur |
| - | - | - |
| 1. Connexion de l’utilisateur | Redirige le client vers `/.auth/login/<provider>`. | Le code client connecte directement l’utilisateur avec le Kit SDK du fournisseur et reçoit un jeton d’authentification. Pour plus d’informations, consultez la documentation du fournisseur. |
| 2. Post-authentification | Le fournisseur redirige le client vers `/.auth/login/<provider>/callback`. | Le code client [publie un jeton du fournisseur](app-service-authentication-how-to.md#validate-tokens-from-providers) vers `/.auth/login/<provider>` pour validation. |
| 3. Établissement de la session authentifiée | App Service ajoute un cookie authentifié à la réponse. | App Service retourne son propre jeton d’authentification au code client. |
| 4. Traitement du contenu authentifié | Le client inclut le cookie d’authentification dans les demandes suivantes (gérées automatiquement par le navigateur). | Le code client présente le jeton d’authentification dans l’en-tête `X-ZUMO-AUTH` (géré automatiquement par les Kits SDK clients Mobile Apps). |

Dans le cas des navigateurs clients, App Service peut diriger automatiquement tous les utilisateurs non authentifiés vers `/.auth/login/<provider>`. Vous pouvez également présenter aux utilisateurs un ou plusieurs liens `/.auth/login/<provider>` pour leur permettre de se connecter à votre application à l’aide du fournisseur de leur choix.

<a name="authorization"></a>

## <a name="authorization-behavior"></a>Comportement d’autorisation

Sur le [Portail Azure](https://portal.azure.com), vous pouvez configurer l’autorisation App Service avec différents comportements lorsque la requête entrante n’est pas authentifiée.

![Capture d’écran montrant la liste déroulante « Action à entreprendre quand la requête n’est pas authentifiée ».](media/app-service-authentication-overview/authorization-flow.png)

Les titres suivants décrivent les options possibles.

### <a name="allow-anonymous-requests-no-action"></a>Autoriser les requêtes anonymes (aucune action)

Cette option permet de confier l’autorisation de trafic non authentifié à votre code d’application. Dans le cas des demandes authentifiées, App Service transmet également les informations d’authentification dans les en-têtes HTTP. 

Cette option assure un traitement plus souple des requêtes anonymes. Par exemple, il permet de [présenter plusieurs fournisseurs de connexion](app-service-authentication-how-to.md#use-multiple-sign-in-providers) aux utilisateurs. Vous devez cependant écrire du code. 

### <a name="allow-only-authenticated-requests"></a>Autoriser uniquement les demandes authentifiées

L’option est **Se connecter avec \<provider>** . App Service redirige toutes les demandes anonymes vers `/.auth/login/<provider>` pour le fournisseur choisi. Si la demande anonyme provient d’une application mobile native, la réponse retournée est `HTTP 401 Unauthorized`.

Cette option évite d’avoir à écrire du code d’authentification dans l’application. Une autorisation plus fine, par exemple propre au rôle, peut être gérée en examinant les revendications de l’utilisateur (consultez la section [Accéder aux revendications utilisateur](app-service-authentication-how-to.md#access-user-claims)).

> [!CAUTION]
> Cette manière de restreindre l’accès s’applique à tous les appels à votre application qui peuvent ne pas être souhaitables pour les applications souhaitant une page d’accès publique disponible, comme dans de nombreuses applications à page unique.

> [!NOTE]
> Par défaut, tout utilisateur de votre locataire Azure AD peut demander un jeton pour votre application à partir d’Azure AD. Vous pouvez [configurer l’application dans Azure AD](../active-directory/develop/howto-restrict-your-app-to-a-set-of-users.md) si vous souhaitez restreindre l’accès à votre application à un ensemble défini d’utilisateurs.

## <a name="more-resources"></a>Plus de ressources

* [Tutoriel : Authentifier et autoriser les utilisateurs de bout en bout dans Azure App Service (Windows)](tutorial-auth-aad.md)  
* [Tutoriel : Authentifier et autoriser les utilisateurs de bout en bout dans Azure App Service pour Linux](./tutorial-auth-aad.md?pivots=platform-linux%3fpivots%3dplatform-linux)  
* [Personnaliser l’authentification et l’autorisation dans App Service](app-service-authentication-how-to.md)
* [Intégration .NET Core d’Azure AppService EasyAuth (tiers)](https://github.com/MaximRouiller/MaximeRouiller.Azure.AppService.EasyAuth)
* [Obtenir une authentification Azure App Service fonctionnant avec .NET Core (tiers)](https://github.com/kirkone/KK.AspNetCore.EasyAuthAuthentication)

Guides pratiques propres à chaque fournisseur :

* [Comment configurer votre application pour utiliser une connexion Azure Active Directory][AAD]
* [Comment configurer votre application pour utiliser une connexion Facebook][Facebook]
* [Comment configurer votre application pour utiliser une connexion Google][Google]
* [Comment configurer votre application pour utiliser une connexion par compte Microsoft][MSA]
* [Comment configurer votre application pour utiliser une connexion Twitter][Twitter]
* [Comment configurer votre application pour utiliser un fournisseur OpenID Connect pour la connexion (préversion)][OIDC]
* [Configurer votre application pour utiliser une connexion avec Apple (préversion)][Apple]

[AAD]: configure-authentication-provider-aad.md
[Facebook]: configure-authentication-provider-facebook.md
[Google]: configure-authentication-provider-google.md
[MSA]: configure-authentication-provider-microsoft.md
[Twitter]: configure-authentication-provider-twitter.md
[OIDC]: configure-authentication-provider-openid-connect.md
[Apple]: configure-authentication-provider-apple.md

[custom-auth]: /previous-versions/azure/app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk#custom-auth

[ADAL-Android]: /previous-versions/azure/app-service-mobile/app-service-mobile-android-how-to-use-client-library#adal
[ADAL-iOS]: /previous-versions/azure/app-service-mobile/app-service-mobile-ios-how-to-use-client-library#adal
[ADAL-dotnet]: /previous-versions/azure/app-service-mobile/app-service-mobile-dotnet-how-to-use-client-library#adal
