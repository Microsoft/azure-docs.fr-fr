---
title: Augmenter la résilience de l’authentification et de l’autorisation dans les applications clientes que vous développez
titleSuffix: Microsoft identity platform
description: Conseils pour l’amélioration de la résilience de l’authentification et de l’autorisation dans une application cliente à l’aide de la plateforme d’identités Microsoft
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
author: knicholasa
ms.author: nichola
manager: martinco
ms.date: 11/23/2020
ms.openlocfilehash: fc15176318dcfae99434f50a0b4370f371cec05a
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96938237"
---
# <a name="increase-the-resilience-of-authentication-and-authorization-in-client-applications-you-develop"></a>Augmenter la résilience de l’authentification et de l’autorisation dans les applications clientes que vous développez

Cette section fournit des conseils sur le renforcement de la résilience dans les applications clientes qui utilisent la plateforme d’identités Microsoft et Azure Active Directory pour connecter des utilisateurs et effectuer des actions pour le compte de ces utilisateurs.

## <a name="use-the-microsoft-authentication-library-msal"></a>Utiliser la bibliothèque d’authentification Microsoft (MSAL)

La [bibliothèque d’authentification Microsoft (MSAL)](https://docs.microsoft.com/azure/active-directory/develop/msal-overview) est un élément clé de la [plateforme d’identités Microsoft](https://docs.microsoft.com/azure/active-directory/develop). Elle simplifie et gère l’acquisition, la gestion, la mise en cache et l’actualisation des jetons, et utilise les meilleures pratiques en matière de résilience. MSAL est conçue pour permettre une solution sécurisée sans que les développeurs aient à se soucier des détails d’implémentation.

MSAL met en cache les jetons et utilise un modèle silencieux d’acquisition de jetons. Elle sérialise également automatiquement le cache de jetons sur les plateformes qui fournissent nativement un stockage sécurisé comme Windows UWP, iOS et Android. Les développeurs peuvent personnaliser le comportement de sérialisation lors de l’utilisation de [Microsoft.Identity.Web](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization), [MSAL.NET](https://docs.microsoft.com/azure/active-directory/develop/msal-net-token-cache-serialization), [MSAL pour Java](https://docs.microsoft.com/azure/active-directory/develop/msal-java-token-cache-serialization) et [MSAL pour Python](https://docs.microsoft.com/azure/active-directory/develop/msal-python-token-cache-serialization).

![Image d’un appareil et d’une application utilisant MSAL pour appeler Microsoft Identity](media/resilience-client-app/resilience-with-microsoft-authentication-library.png)

Quand vous utilisez MSAL, la mise en cache, l’actualisation et l’acquisition en mode silencieux des jetons sont automatiquement prises en charge. Vous pouvez utiliser des modèles simples pour acquérir les jetons nécessaires à l’authentification moderne. Nous prenons en charge de nombreux langages, et vous trouverez un exemple qui correspond à vos langage et scénario sur notre page d’[exemples](https://docs.microsoft.com/azure/active-directory/develop/sample-v2-code).

## <a name="c"></a>[C#](#tab/csharp)

```csharp
try
{
    result = await app.AcquireTokenSilent(scopes, account).ExecuteAsync();
}
catch(MsalUiRequiredException ex)
{
    result = await app.AcquireToken(scopes).WithClaims(ex.Claims).ExecuteAsync()
}
```

## <a name="javascript"></a>[Javascript](#tab/javascript)

```javascript
return myMSALObj.acquireTokenSilent(request).catch(error => {
    console.warn("silent token acquisition fails. acquiring token using redirect");
    if (error instanceof msal.InteractionRequiredAuthError) {
        // fallback to interaction when silent call fails
        return myMSALObj.acquireTokenPopup(request).then(tokenResponse => {
            console.log(tokenResponse);

            return tokenResponse;
        }).catch(error => {
            console.error(error);
        });
    } else {
        console.warn(error);
    }
});
```

---

MSAL peut, dans certains cas, actualiser les jetons de manière proactive. Lorsque Microsoft Identity émet un jeton de longue durée, elle peut envoyer des informations au client concernant le meilleur moment pour actualiser le jeton (« refresh\_in »). MSAL actualise de manière proactive le jeton en fonction de ces informations. L’application continuera à fonctionner tant que l’ancien jeton sera valide, mais prendra plus de temps pour acquérir un autre jeton.

### <a name="stay-up-to-date"></a>Rester informé

Les développeurs doivent disposer d’un processus de mise à jour vers la dernière version de MSAL. L’authentification fait partie de la sécurité de votre application, et votre application doit rester à jour des améliorations de sécurité contenues dans les nouvelles versions de MSAL. Il s’agit généralement d’une bonne pratique pour les bibliothèques en développement continu, ce qui vous permettra de disposer du code le plus récent en ce qui concerne la résilience des applications. À mesure que Microsoft Identity continue d’innover sur les moyens de rendre les applications plus résilientes, les applications qui utilisent la version la plus récente de MSAL seront les mieux préparées pour tirer parti de ces innovations.

[Vérifier la version la plus récente de MSAL.js et les notes de publication](https://github.com/AzureAD/microsoft-authentication-library-for-js/releases)

[Vérifier la version la plus récente de MSAL .NET et les notes de publication](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/releases)

[Vérifier la version la plus récente de MSAL Python et les notes de publication](https://github.com/AzureAD/microsoft-authentication-library-for-python/releases)

[Vérifier la version la plus récente de MSAL Java et les notes de publication](https://github.com/AzureAD/microsoft-authentication-library-for-java/releases)

[Vérifier la version la plus récente de MSAL iOS et macOS et les notes de publication](https://github.com/AzureAD/microsoft-authentication-library-for-objc/releases)

[Vérifier la version la plus récente de MSAL Android et les notes de publication](https://github.com/AzureAD/microsoft-authentication-library-for-android/releases)

[Vérifier la version la plus récente de MSAL Angular et les notes de publication](https://github.com/AzureAD/microsoft-authentication-library-for-js/releases)

[Vérifier la version la plus récente de Microsoft.Identity.Web et les notes de publication](https://github.com/AzureAD/microsoft-identity-web/releases)

## <a name="use-resilient-patterns-for-token-handling"></a>Utiliser des modèles résilients pour la gestion des jetons

Si vous n’utilisez pas MSAL, vous pouvez utiliser ces modèles résilients pour la gestion des jetons. Ces bonnes pratiques sont implémentées automatiquement par la bibliothèque MSAL. 

En général, une application qui utilise l’authentification moderne appellera un point de terminaison pour récupérer les jetons qui authentifient l’utilisateur ou autorisent l’application à appeler des API protégées. MSAL est conçue pour gérer les détails de l’authentification et implémente plusieurs modèles pour améliorer la résilience de ce processus. Utilisez les instructions de cette section pour implémenter les meilleures pratiques si vous choisissez d’utiliser une bibliothèque autre que MSAL. Si vous utilisez MSAL, vous bénéficiez de toutes ces meilleures pratiques gratuitement, car MSAL les implémente automatiquement.

### <a name="cache-tokens"></a>Mettre en cache des jetons

Les applications doivent mettre correctement en cache les jetons reçus de Microsoft Identity. Lorsque votre application reçoit des jetons, la réponse HTTP qui contient les jetons contient également une propriété « expires_in » qui indique à l’application la durée de mise en cache et de réutilisation du jeton. Il est important que les applications utilisent la propriété « expires_in » pour déterminer la durée de vie du jeton. L’application ne doit jamais tenter de décoder un jeton d’accès d’API.

![Application effectuant un appel à Microsoft Identity, mais l’appel passe par un cache de jetons sur l’appareil exécutant l’application](media/resilience-client-app/token-cache.png)

L’utilisation du jeton mis en cache empêche tout trafic inutile entre votre application et Microsoft Identity et rend votre application moins vulnérable aux échecs d’acquisition de jeton en réduisant le nombre d’appels d’acquisition de jetons. Les jetons mis en cache améliorent également les performances de votre application, car l’application doit moins souvent bloquer l’acquisition de jetons. Votre utilisateur peut rester connecté à votre application pendant toute la durée de vie de ce jeton.

### <a name="serialize-and-persist-tokens"></a>Sérialiser et conserver les jetons

Les applications doivent sérialiser de manière sécurisée leur cache de jetons pour que les jetons persistent entre les instances de l’application. Les jetons peuvent être réutilisés tant que leur durée de vie est valide. Les [jetons d’actualisation](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow#refresh-the-access-token) et, de plus en plus, les [jetons d’accès](https://docs.microsoft.com/azure/active-directory/develop/access-tokens) sont émis pour plusieurs heures. Cette durée de validité peut s’étendre sur une période pendant laquelle un utilisateur démarre votre application plusieurs fois. Lorsque votre application démarre, elle doit vérifier s’il existe un jeton d’accès ou d’actualisation valide qui peut être utilisé. Cela augmentera la résilience et les performances de l’application, car cela permet d’éviter les appels inutiles à Microsoft Identity.

![Application effectuant un appel à Microsoft Identity, mais l’appel passe par un cache de jetons et un magasin de jetons sur l’appareil exécutant l’application](media/resilience-client-app/token-store.png)

Le stockage permanent de jetons doit être soumis à un contrôle d’accès et chiffré selon l’identité de l’utilisateur ou du processus propriétaire. Sur les plateformes telles que les appareils mobiles, Windows et Mac, le développeur doit tirer parti des capacités intégrées de stockage des informations d’identification.

### <a name="acquire-tokens-silently"></a>Acquérir des jetons silencieusement

Le processus d’authentification d’un utilisateur ou de récupération de l’autorisation d’appeler une API peut nécessiter plusieurs étapes dans Microsoft Identity. Par exemple, lorsque l’utilisateur se connecte pour la première fois, il peut avoir besoin d’entrer des informations d’identification et d’effectuer une authentification multifacteur par SMS. Chaque étape ajoute une dépendance à la ressource qui fournit le service. La meilleure expérience pour l’utilisateur, et celle qui présente le moins de dépendances, consiste à tenter d’acquérir un jeton silencieusement pour éviter ces étapes supplémentaires avant de demander l’intervention de l’utilisateur.

![Diagramme montrant les différents services au sein de Microsoft Identity qui peuvent être nécessaires pour terminer le processus d’authentification ou d’autorisation d’un utilisateur](media/resilience-client-app/external-dependencies.png)

L’acquisition d’un jeton en mode silencieux commence par l’utilisation d’un jeton valide du cache de jetons de l’application. Si aucun jeton valide n’est disponible, l’application doit tenter d’acquérir un jeton à l’aide d’un jeton d’actualisation, s’il est disponible, et du point de terminaison de jeton. Si aucune de ces options n’est disponible, l’application doit acquérir un jeton à l’aide du paramètre « prompt=none ». Elle utilisera le point de terminaison d’autorisation, mais ne montrera aucune interface utilisateur à l’utilisateur. Si Microsoft Identity peut fournir un jeton à l’application sans interagir avec l’utilisateur, elle le fera. Si aucune de ces méthodes ne produit de jeton, l’utilisateur devra s’authentifier de nouveau de manière interactive.

> [!NOTE]
> En général, les applications doivent éviter d’utiliser des invites telles que « connexion » et « consentement », car elles forcent l’interaction avec l’utilisateur même si aucune intervention n’est requise.

## <a name="handle-service-responses-properly"></a>Gérer les réponses de service correctement

Alors que les applications doivent gérer toutes les réponses d’erreur, certaines réponses peuvent avoir un impact sur la résilience. Si votre application reçoit un code de réponse HTTP 429, Trop de requêtes, Microsoft Identity limite vos demandes. Si votre application continue à faire trop de requêtes, elle continuera d’être limitée, ce qui empêchera votre application de recevoir des jetons. Votre application ne doit pas tenter d’acquérir à nouveau un jeton avant que le temps, en secondes, indiqué dans le champ de réponse Retry-After ne soit écoulé. Recevoir une réponse 429 est souvent une indication que l’application ne met pas en cache ni ne réutilise les jetons correctement. Les développeurs doivent examiner la façon dont les jetons sont mis en cache et réutilisés dans l’application.

Quand une application reçoit un code de réponse HTTP 5xx, l’application ne doit pas entrer dans une boucle de nouvelle tentative rapide. Le cas échéant, l’application doit respecter le même traitement Retry-After, comme c’est le cas pour une réponse 429. Si aucun en-tête Retry-After n’est fourni par la réponse, nous vous recommandons d’implémenter une nouvelle tentative de backoff exponentiel avec la première tentative au moins cinq secondes après la réponse.

Quand une requête expire, les applications ne doivent pas faire de nouvelle tentative immédiatement. Implémentez une nouvelle tentative de backoff exponentiel avec la première tentative au moins cinq secondes après la réponse.

## <a name="evaluate-options-for-retrieving-authorization-related-information"></a>Évaluer les options de récupération des informations relatives aux autorisations

De nombreuses applications et API requièrent des informations spécifiques sur l’utilisateur pour prendre des décisions d’autorisation. Il existe plusieurs façons pour une application d’obtenir ces informations. Chaque méthode a ses avantages et ses inconvénients. Les développeurs doivent les prendre en considération pour déterminer la stratégie la plus adaptée à la résilience de leur application.

### <a name="tokens"></a>Jetons

Les jetons d’identité (ID) et les jetons d’accès contiennent des revendications standard qui fournissent des informations sur l’objet. Celles-ci sont documentées dans les [jetons d’ID de la plateforme d’identités Microsoft](https://docs.microsoft.com/azure/active-directory/develop/id-tokens) et les [jetons d’accès de la plateforme d’identités Microsoft](https://docs.microsoft.com/azure/active-directory/develop/access-tokens). Si les informations dont votre application a besoin se trouvent déjà dans le jeton, la technique la plus efficace pour récupérer ces données consiste à utiliser des revendications de jeton, car cela permet d’économiser les frais d’un autre appel réseau pour récupérer les informations séparément. Moins d’appels réseau signifie une plus grande résilience globale pour l’application.

> [!NOTE]
> Certaines applications appellent le point de terminaison UserInfo pour récupérer les revendications relatives à l’utilisateur qui s’est authentifié. Les informations disponibles dans le jeton d’ID que votre application peut recevoir sont un sur-ensemble des informations qu’il peut obtenir du point de terminaison UserInfo. Votre application doit utiliser le jeton d’ID pour obtenir des informations sur l’utilisateur au lieu d’appeler le point de terminaison UserInfo.

Un développeur d’applications peut compléter les revendications de jeton standard par des [revendications facultatives](https://docs.microsoft.com/azure/active-directory/develop/active-directory-optional-claims). L’une des revendications facultatives courantes est celle des [groupes](https://docs.microsoft.com/azure/active-directory/develop/active-directory-optional-claims#configuring-groups-optional-claims). Il existe plusieurs façons d’ajouter des revendications de groupe. L’option « Groupe d’applications » comprend uniquement les groupes affectés à l’application. Les options « Tout » ou « Groupes de sécurité » incluent les groupes de toutes les applications d’un même locataire, ce qui peut ajouter de nombreux groupes au jeton. Il est important d’évaluer l’effet dans votre cas, car cela peut potentiellement invalider l’efficacité obtenue par la demande de groupes dans le jeton en provoquant un gonflement des jetons et même en exigeant des appels supplémentaires pour obtenir la liste complète des groupes.

Au lieu d’utiliser des groupes dans votre jeton, vous pouvez utiliser et inclure des rôles d’application. Les développeurs peuvent définir des [rôles d’application](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) pour leurs applications et API que le client peut gérer à partir de son répertoire en utilisant le portail ou les API. Les professionnels de l’informatique peuvent ensuite attribuer des rôles aux différents utilisateurs et groupes pour contrôler l’accès au contenu et aux fonctionnalités. Lorsqu’un jeton est émis pour l’application ou l’API, les rôles attribués à l’utilisateur sont disponibles dans la revendication de rôles du jeton. L’obtention de ces informations directement dans un jeton peut permettre d’économiser des appels d’API supplémentaires.

Enfin, les administrateurs informatiques peuvent également ajouter des revendications basées sur des informations spécifiques d’un locataire. Par exemple, une entreprise peut disposer d’une extension afin d’avoir un ID d’utilisateur spécifique à l’entreprise.

Dans tous les cas, l’ajout d’informations du répertoire directement à un jeton peut être efficace et augmenter la résilience des applications en réduisant le nombre de dépendances de l’application. En revanche, cela ne résout pas les problèmes de résilience liés à l’impossibilité d’acquérir un jeton. Vous devez ajouter uniquement des revendications facultatives pour les principaux scénarios de votre application. Si l’application ne nécessite des informations que pour la fonctionnalité d’administration, il est préférable pour l’application d’obtenir ces informations uniquement si nécessaire.

### <a name="microsoft-graph"></a>Microsoft Graph

Microsoft Graph fournit un point de terminaison d’API unifié pour accéder aux données Microsoft 365 qui décrivent les modèles de productivité, d’identité et de sécurité d’une organisation. Les applications qui utilisent Microsoft Graph peuvent éventuellement utiliser les informations de Microsoft 365 pour prendre des décisions d’autorisation.

Les applications ne requièrent qu’un seul jeton pour accéder à l’ensemble de Microsoft 365. Cette solution est plus résiliente que l’utilisation des anciennes API spécifiques aux composants de Microsoft 365, comme Microsoft Exchange ou Microsoft SharePoint, qui nécessitent plusieurs jetons.

Lorsque vous utilisez des API Microsoft Graph, nous vous suggérons d’utiliser un [Kit de développement logiciel (SDK) Microsoft Graph](https://docs.microsoft.com/graph/sdks/sdks-overview). Les Kits de développement logiciel (SDK) Microsoft Graph sont conçus pour simplifier la création d’applications qualitatives, efficaces et résilientes qui accèdent à Microsoft Graph.

Pour les décisions d’autorisation, les développeurs doivent déterminer quand utiliser les revendications disponibles dans un jeton comme alternative à certains appels à Microsoft Graph. Comme indiqué ci-dessus, les développeurs pouvaient demander des groupes, des rôles d’application et des revendications facultatives dans leurs jetons. En matière de résilience, l’utilisation de Microsoft Graph pour l’autorisation nécessite des appels réseau supplémentaires qui reposent sur Microsoft Identity (pour obtenir le jeton permettant d’accéder à Microsoft Graph), ainsi que sur Microsoft Graph lui-même. Toutefois, si votre application utilise déjà Microsoft Graph comme couche de données, le fait de s’appuyer sur Graph pour l’autorisation ne constitue pas un risque supplémentaire.

## <a name="use-broker-authentication-on-mobile-devices"></a>Utiliser l’authentification par répartiteur sur des appareils mobiles

Sur les appareils mobiles, l’utilisation d’un répartiteur d’authentification comme Microsoft Authenticator améliore la résilience. Le répartiteur apporte des avantages au-delà de ce qui est disponible avec d’autres options, comme le navigateur système ou une vue web incorporée. Le répartiteur d’authentification peut utiliser un [jeton d’actualisation principal](https://docs.microsoft.com/azure/active-directory/devices/concept-primary-refresh-token) (PRT) qui contient les revendications relatives à l’utilisateur et à l’appareil et qui peut être utilisé pour obtenir des jetons d’authentification afin d’accéder à d’autres applications à partir de l’appareil. Lorsqu’un PRT est utilisé pour demander l’accès à une application, ses revendications d’appareil et d’authentification multifacteur sont approuvées par Azure AD. Cela augmente la résilience tout en évitant des étapes supplémentaires pour authentifier à nouveau l’appareil. Les utilisateurs ne seront pas confrontés à plusieurs invites MFA sur le même appareil, ce qui augmente la résilience en réduisant les dépendances aux services externes et améliore l’expérience utilisateur.

![Application effectuant un appel à Microsoft Identity, mais l’appel passe par un cache de jetons, un magasin de jetons et un répartiteur d’authentification sur l’appareil exécutant l’application](media/resilience-client-app/authentication-broker.png)

L’authentification par répartiteur est automatiquement prise en charge par MSAL. Vous trouverez plus d’informations sur l’utilisation de l’authentification répartie sur les pages suivantes :

- [Configurer l’authentification unique sur macOS et iOS](https://docs.microsoft.com/azure/active-directory/develop/single-sign-on-macos-ios#sso-through-authentication-broker-on-ios)
- [Guide pratique pour activer l’authentification unique entre applications sur Android à l’aide de MSAL](https://docs.microsoft.com/azure/active-directory/develop/msal-android-single-sign-on)

## <a name="adopt-continuous-access-evaluation"></a>Adopter la fonctionnalité Évaluation continue de l’accès

[Évaluation continue de l’accès (CAE)](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-continuous-access-evaluation) est un développement récent qui peut augmenter la sécurité et la résilience des applications grâce à des jetons durables. CAE est une norme émergente du secteur d’activité qui est développée au sein du groupe de travail sur les signaux et les événements partagés de la fondation OpenID. Grâce à CAE, vous pouvez révoquer un jeton d’accès en fonction d’[événements critiques](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-continuous-access-evaluation#critical-event-evaluation) et d’une [évaluation de la stratégie](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-continuous-access-evaluation#conditional-access-policy-evaluation-preview), plutôt que d’une courte durée de vie du jeton. Pour certaines API de ressources, comme les risques et les stratégies sont évalués en temps réel, CAE peut considérablement augmenter la durée de vie du jeton jusqu’à 28 heures. À mesure que les applications et les API de ressources adoptent CAE, Microsoft Identity sera en mesure d’émettre des jetons d’accès révocables et valides pendant des périodes prolongées. Ces jetons à longue durée de vie seront actualisés de manière proactive par MSAL.

Bien que CAE en soit à ses débuts, il est possible de [développer aujourd’hui des applications clientes qui tireront parti de CAE](../develop/app-resilience-continuous-access-evaluation.md) lorsque les ressources (API) utilisées par l’application l’adopteront. Au fur et à mesure que les ressources adopteront CAE, votre application sera en mesure d’acquérir des jetons compatibles avec la fonctionnalité également pour ces ressources. L’API Microsoft Graph et les [Kits de développement logiciel (SDK) Microsoft Graph](https://docs.microsoft.com/graph/sdks/sdks-overview) proposeront la capacité CAE en préversion début 2021. Si vous souhaitez participer à la préversion publique de Microsoft Graph avec CAE, vous pouvez nous faire part de votre intérêt à l’adresse suivante : [https://aka.ms/GraphCAEPreview](https://aka.ms/GraphCAEPreview).

Si vous développez des API de ressources, nous vous encourageons à participer au [groupe de travail sur les signaux et les événements partagés](https://openid.net/wg/sse/). Nous travaillons avec ce groupe pour permettre le partage des événements de sécurité entre Microsoft Identity et les fournisseurs de ressources.

## <a name="next-steps"></a>Étapes suivantes

- [Guide pratique pour utiliser des API dotées d’Évaluation continue de l’accès dans vos applications](../develop/app-resilience-continuous-access-evaluation.md)
- [Renforcer la résilience des applications démon](resilience-daemon-app.md)
- [Renforcer la résilience de votre infrastructure de gestion des identités et des accès](resilience-in-infrastructure.md)
- [Renforcer la résilience de vos systèmes CIAM](resilience-b2c.md)
