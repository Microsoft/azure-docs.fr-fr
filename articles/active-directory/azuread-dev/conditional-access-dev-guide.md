---
title: Guide du développeur pour l’accès conditionnel Azure AD
description: Guide du développeur et scénarios pour l’accès conditionnel à Azure AD
services: active-directory
author: rwike77
manager: CelesteDG
ms.author: ryanwi
ms.reviewer: jmprieur, saeeda
ms.date: 02/28/2019
ms.service: active-directory
ms.subservice: azuread-dev
ms.custom: aaddev
ms.topic: conceptual
ms.workload: identity
ROBOTS: NOINDEX
ms.openlocfilehash: b08b07ebcd6e45e95e23dff8686148889e9d4b39
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96173917"
---
# <a name="developer-guidance-for-the-azure-active-directory-conditional-access-feature"></a>Guide du développeur pour la fonctionnalité d’accès conditionnel d’Azure Active Directory

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

> [!NOTE]
> Pour la version de cet article relatif à la plateforme Microsoft Identity, consultez [Guide du développeur pour l’accès conditionnel à Azure Active Directory](../develop/v2-conditional-access-dev-guide.md).

La fonctionnalité d’accès conditionnel dans Azure Active Directory (Azure AD) offre l’une des méthodes que vous pouvez utiliser pour sécuriser votre application et protéger un service. L’accès conditionnel permet aux développeurs et aux clients d’entreprise de protéger les services dans une multitude de façons, notamment :

* Authentification multifacteur
* Autoriser uniquement les appareils inscrits sur Intune inscrit pour accéder aux services spécifiques
* Restriction des plages IP et emplacements utilisateur

Pour plus d’informations sur toutes les fonctionnalités de l’accès conditionnel, consultez [Qu’est-ce que l’accès conditionnel](../conditional-access/overview.md).

Destiné aux développeurs créant des applications pour Azure AD, cet article montre comment utiliser l’accès conditionnel et explique l’impact de l’accès à des ressources non contrôlées auxquelles des stratégies d’accès conditionnel sont peut-être appliquées. Cet article explore également les implications de l’accès conditionnel dans les applications web et le flux On-Behalf-Of accédant à Microsoft Graph et appelant des API.

Il suppose une connaissance des applications [monolocataires et multilocataires](../develop/howto-convert-app-to-be-multi-tenant.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json), ainsi que des [modèles courants d’authentification](v1-authentication-scenarios.md).

## <a name="how-does-conditional-access-impact-an-app"></a>Comment l’accès conditionnel impacte-t-il une application ?

### <a name="app-types-impacted"></a>Incidence sur les types d’application

Dans les scénarios les plus courants, l’accès conditionnel ne modifie pas le comportement d’une application ou nécessite des modifications de la part du développeur.  Uniquement dans certains cas, lorsqu’une application en mode silencieux ou indirectement demande un jeton pour un service, une application requiert des modifications du code pour gérer des « défis » d’accès conditionnel.  Cela peut être aussi simple que l’exécution d’une demande de connexion interactive.

Plus précisément, les scénarios suivants requièrent un code pour gérer des « défis » d’accès conditionnel :

* Applications effectuant le flux Pour le compte de
* Applications accédant à plusieurs services/ressources
* Applications à page unique en utilisant ADAL.js
* Les applications appelant une ressource

Les stratégies d’accès conditionnel peuvent être appliquées à l’application, mais peuvent également être appliquées à une API web à laquelle votre application a accès. Pour en apprendre davantage sur la façon de configurer une stratégie d’accès conditionnel, consultez [Stratégies d’accès conditionnel courantes](../conditional-access/concept-conditional-access-policy-common.md).

Selon le scénario, un client d’entreprise peut appliquer et supprimer des stratégies d’accès conditionnel à tout moment. Afin que votre application continue à fonctionner correctement lorsqu’une nouvelle stratégie est appliquée, vous devez mettre en œuvre la gestion de « défi ». Les exemples suivants illustrent la gestion des défis.

### <a name="conditional-access-examples"></a>Exemples d’accès conditionnel

Certains scénarios requièrent des modifications de code pour gérer l’accès conditionnel, tandis que d’autres travaillent tel quel. Voici quelques scénarios utilisant l’accès conditionnel pour l’authentification multifacteur qui donne une idée de la différence.

* Vous générez une application iOS de client unique et appliquez une stratégie d’accès conditionnel. L’application connecte un utilisateur et ne demande pas l’accès à une API. Lorsque l’utilisateur se connecte, la stratégie est appelée automatiquement et l’utilisateur doit effectuer l’authentification multifacteur (MFA). 
* Vous générez une application native qui utilise un service de niveau intermédiaire pour accéder à une API en aval. Un client d’entreprise de la société utilisant cette application applique une stratégie à l’API en aval. Quand un utilisateur final se connecte, l’application native demande l’accès au niveau intermédiaire et envoie le jeton. Le niveau intermédiaire effectue le flux Pour le compte pour demander l’accès à l’API en aval. À ce stade, un défi de « revendications » est présenté au niveau intermédiaire. Le niveau intermédiaire renvoie la demande à l’application native, qui doit se conformer à la stratégie d’accès conditionnel.

#### <a name="microsoft-graph"></a>Microsoft Graph

Microsoft Graph possède des considérations spéciales concernant la création d’applications des environnements avec accès conditionnel. En règle générale, les mécanismes d’accès conditionnel ont le même comportement, mais les stratégies que voient vos utilisateurs sont basées sur les données sous-jacentes demandées au graph par votre application. 

Plus précisément, toutes les étendues de Microsoft Graph représentent un jeu de données auquel il est possible d’appliquer des stratégies individuellement. Dans la mesure où les stratégies d’accès conditionnel sont assignées à des jeux de données spécifiques, Azure AD applique des stratégies d’accès conditionnel basées sur les données derrière Graph, plus que sur Graph.

Par exemple, si une application demande les étendues suivantes de Microsoft Graph,

```
scopes="Bookings.Read.All Mail.Read"
```

Une application peut s’attendre à ce que ses utilisateurs répondent à toutes les stratégies définies sur Bookings et Exchange. Certaines étendues peuvent mapper vers plusieurs jeux de données si elles disposent de l’accès. 

### <a name="complying-with-a-conditional-access-policy"></a>Conformité à une stratégie d’accès conditionnel

Pour différentes topologies d’applications, une stratégie d’accès conditionnel est évaluée lorsque la session est établie. Étant donné qu’une stratégie d’accès conditionnel fonctionne sur la granularité des applications et des services, le point sur lequel elle est appelée dépend essentiellement du scénario que vous essayez d’accomplir.

Lorsque votre application tente d’accéder à un service avec une stratégie d’accès conditionnel, elle peut rencontrer un défi d’accès conditionnel. Ce défi est encodé dans le paramètre `claims` qui est fourni dans une réponse d’Azure AD. Voici un exemple de ce paramètre de défi : 

```
claims={"access_token":{"polids":{"essential":true,"Values":["<GUID>"]}}}
```

Les développeurs peuvent prendre ce défi et l’ajouter à une nouvelle demande à Azure AD. Le fait de passer cet état invite l’utilisateur final à exécuter toute action nécessaire pour se conformer à la stratégie d’accès conditionnel. Les scénarios suivants expliquent les détails de l’erreur et le mode d’extraction du paramètre.

## <a name="scenarios"></a>Scénarios

### <a name="prerequisites"></a>Prérequis

L’accès conditionnel Azure AD est une fonctionnalité incluse dans [Azure AD Premium](../fundamentals/active-directory-whatis.md). Pour en savoir plus sur les conditions requises de licence, consultez le [Rapport d’utilisation sans licence](../reports-monitoring/overview-reports.md). Les développeurs peuvent joindre le [Microsoft Developer Network](/), qui inclut un abonnement gratuit à la suite Enterprise Mobility, qui comprend Azure AD Premium.

### <a name="considerations-for-specific-scenarios"></a>Considérations pour des scénarios spécifiques

Les informations suivantes s’appliquent uniquement dans ces scénarios d’accès conditionnel :

* Applications effectuant le flux Pour le compte de
* Applications accédant à plusieurs services/ressources
* Applications à page unique en utilisant ADAL.js

Les sections suivantes décrivent des scénarios courants plus complexes. Le principe de fonctionnement central est que les stratégies d’accès conditionnel sont évaluées au moment où le jeton est demandé pour le service qui contient une stratégie d’accès conditionnel.

## <a name="scenario-app-performing-the-on-behalf-of-flow"></a>Scénario : application effectuant le flux Pour le compte de

Dans ce scénario, nous abordons le cas dans lequel une application native appelle une API/ service Web. À son tour, ce service exécute le flux On-Behalf-Of pour appeler un service en aval. Dans notre cas, nous avons appliqué notre stratégie d’accès conditionnel pour le service en aval (API Web 2) et nous utilisons une application native plutôt qu’une application démon/serveur. 

![Application effectuant le diagramme de flux Pour le compte](./media/conditional-access-dev-guide/app-performing-on-behalf-of-scenario.png)

La demande de jeton initiale pour l’API Web 1 n’affiche aucune invite pour l’utilisateur final en lien avec l’authentification multifacteur, car l’API Web 1 ne peut pas toujours atteindre l’API en aval. Une fois que l’API Web 1 tente de demander un jeton flux Pour le compte de l’utilisateur pour l’API Web 2, la demande échoue car l’utilisateur ne s’est pas connecté avec l’authentification multifacteur.

Azure AD renvoie une réponse HTTP avec des données intéressantes :

> [!NOTE]
> Dans cette instance, il s’agit d’une description d’erreur de l’authentification multifacteur, mais il existe une large gamme de `interaction_required` possible concernant l’accès conditionnel.

```
HTTP 400; Bad Request
error=interaction_required
error_description=AADSTS50076: Due to a configuration change made by your administrator, or because you moved to a new location, you must use multi-factor authentication to access '<Web API 2 App/Client ID>'.
claims={"access_token":{"polids":{"essential":true,"Values":["<GUID>"]}}}
```

Dans l’API Web 1, nous interceptons l’erreur `error=interaction_required`, et renvoyons le défi `claims` à l’application de bureau. À ce stade, l’application de bureau peut effectuer un nouvel appel `acquireToken()` et ajouter le défi `claims` comme un paramètre de chaîne de requêtes supplémentaire. Cette nouvelle demande oblige l’utilisateur à effectuer une authentification multifacteur et à renvoyer ce nouveau jeton à l’API Web 1, puis exécuter le flux Pour le compte de.

Pour tester ce scénario, consultez notre [exemple de code .NET](https://github.com/Azure-Samples/active-directory-dotnet-webapi-onbehalfof-ca). Il indique comment repasser le défi de revendications à partir d’API Web 1 vers l’application native et construire une nouvelle demande à l’intérieur de l’application cliente.

## <a name="scenario-app-accessing-multiple-services"></a>Scénario : application accédant à plusieurs services

Dans ce scénario, nous abordons le cas dans lequel une application Web accède aux deux services, dont un d’entre eux est affecté à une stratégie d’accès conditionnel. En fonction de votre logique d’application, il peut exister un chemin d’accès dans lequel votre application ne nécessite pas l’accès aux deux services Web. Dans ce scénario, l’ordre dans lequel vous demandez un jeton joue un rôle important dans l’expérience de l’utilisateur final.

Supposons que nous disposons d’un service Web A et B et que le service Web B applique notre stratégie d’accès conditionnel. Tandis que la demande initiale d’authentification interactive requiert le consentement pour les deux services, la stratégie d’accès conditionnel n’est pas requise dans tous les cas. Si l’application demande un jeton pour le service Web B, la stratégie est appelée et les demandes ultérieures pour le service Web A réussissent également comme suit.

![Application accédant à un diagramme de flux multi-services](./media/conditional-access-dev-guide/app-accessing-multiple-services-scenario.png)

Sinon, si l’application demande initialement un jeton pour le service Web A, l’utilisateur final n’appelle pas la stratégie d’accès conditionnel. Cela permet au développeur d’applications de contrôler l’expérience de l’utilisateur final et de ne pas forcer l’appel de la stratégie d’accès conditionnel dans tous les cas. Cela se complique si l’application demande ensuite un jeton pour le service web B. À ce stade, l’utilisateur final doit se conformer à la stratégie d’accès conditionnel. Lorsque l’application tente de `acquireToken`, cela peut générer l’erreur suivante (illustrée dans le diagramme suivant) :

```
HTTP 400; Bad Request
error=interaction_required
error_description=AADSTS50076: Due to a configuration change made by your administrator, or because you moved to a new location, you must use multi-factor authentication to access '<Web API App/Client ID>'.
claims={"access_token":{"polids":{"essential":true,"Values":["<GUID>"]}}}
```

![Application accédant à plusieurs services demandant un nouveau jeton](./media/conditional-access-dev-guide/app-accessing-multiple-services-new-token.png)

Si l’application utilise la bibliothèque ADAL, un échec d’acquisition du jeton est toujours retenté interactivement. Lorsque cette demande interactive se produit, l’utilisateur final a la possibilité de se conformer à l’accès conditionnel. Cela est vrai, sauf si la demande est un `AcquireTokenSilentAsync` ou `PromptBehavior.Never` et dans ce cas, l’application doit effectuer une demande interactive ```AcquireToken``` pour permettre à l’utilisateur final de se conformer à la stratégie.

## <a name="scenario-single-page-app-spa-using-adaljs"></a>Scénario : application à page unique (SPA) en utilisant ADAL.js

Dans ce scénario, nous abordons le cas où nous disposons d’une application à page unique (SPA), en utilisant ADAL.js pour appeler une API Web protégée par l’accès conditionnel. Il s’agit d’une architecture simple mais avec des nuances qui doivent être prises en compte lors du développement autour de l’accès conditionnel.

Dans ADAL.js, il existe quelques fonctions qui obtiennent des jetons : `login()`, `acquireToken(...)`, `acquireTokenPopup(…)`, et `acquireTokenRedirect(…)`.

* `login()` obtient un jeton d’ID via une demande de connexion interactive, mais n’obtient pas les jetons d’accès pour n’importe quel service (y compris une API Web protégée par l’accès conditionnel).
* `acquireToken(…)` peut ensuite être utilisé pour obtenir silencieusement un jeton d’accès c'est-à-dire qu’il n’affiche pas l’interface utilisateur dans tous les cas.
* `acquireTokenPopup(…)` et `acquireTokenRedirect(…)` sont tous deux utilisés pour demander interactivement un jeton pour une ressource, ce qui signifie qu’ils affichent toujours l’interface utilisateur de connexion.

Lorsqu’une application a besoin d’un jeton d’accès pour appeler une API Web, elle tente une `acquireToken(…)`. Si la session du jeton a expiré ou si nous devons nous conformer à une stratégie d’accès conditionnel, la fonction *acquireToken* échoue et l’application utilise `acquireTokenPopup()` ou `acquireTokenRedirect()`.

![Application à page unique en utilisant le diagramme de flux ADAL](./media/conditional-access-dev-guide/spa-using-adal-scenario.png)

Examinons un exemple avec notre scénario d’accès conditionnel. L’utilisateur final a simplement accédé au site et n’a pas de session. Nous effectuons un appel `login()` et obtenons un jeton d’ID sans authentification multifacteur. Puis l’utilisateur final appuie sur un bouton qui exige que l’application demande des données depuis une API Web. L’application tente d’effectuer un appel `acquireToken()` mais échoue, étant donné que l’utilisateur n’a pas encore effectué l’authentification multifacteur et doit se conformer à la stratégie d’accès conditionnel.

Azure AD renvoie la réponse HTTP suivante :

```
HTTP 400; Bad Request
error=interaction_required
error_description=AADSTS50076: Due to a configuration change made by your administrator, or because you moved to a new location, you must use multi-factor authentication to access '<Web API App/Client ID>'.
```

Notre application a besoin d’intercepter le `error=interaction_required`. L’application peut alors utiliser `acquireTokenPopup()` ou `acquireTokenRedirect()` sur la même ressource. L’utilisateur est obligé d’effectuer une authentification multifacteur. Une fois que l’utilisateur a terminé l’authentification multifacteur, l’application émet un nouveau jeton d’accès pour la ressource demandée.

Pour tester ce scénario, consultez notre [exemple de code Pour le compte de SPA JS](https://github.com/Azure-Samples/active-directory-dotnet-webapi-onbehalfof-ca). Cet exemple de code utilise la stratégie d’accès conditionnel et l’API Web précédemment inscrites avec un SPA JS pour illustrer ce scénario. Il explique comment gérer correctement le défi de revendications et obtenir un jeton d’accès qui peut être utilisé pour votre API Web. Vous pouvez également extraire [l’exemple de code Angular.js](https://github.com/Azure-Samples/active-directory-angularjs-singlepageapp) général pour obtenir des conseils sur une SPA Angular

## <a name="see-also"></a>Voir aussi

* Vous pouvez en savoir plus sur les fonctionnalités, consultez [Accès conditionnel dans Azure Active Directory](../conditional-access/overview.md).
* Pour obtenir plus d’exemples de code Azure AD, consultez [Référentiel GitHub d’exemples de code](https://github.com/azure-samples?utf8=%E2%9C%93&q=active-directory).
* Pour plus d’informations sur les kits SDK ADAL et l’accès à la documentation de référence, consultez [guide des bibliothèques](active-directory-authentication-libraries.md).
* Pour en savoir plus sur les scénarios multi-locataire, consultez [comment connecter les utilisateurs à l’aide du modèle multi-locataire](../develop/howto-convert-app-to-be-multi-tenant.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).