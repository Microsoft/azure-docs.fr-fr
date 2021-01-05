---
title: Guide pratique pour utiliser les API d’évaluation continue de l’accès dans vos applications | Azure
titleSuffix: Microsoft identity platform
description: Guide pratique pour augmenter la sécurité et la résilience des applications en ajoutant la prise en charge de l’évaluation continue de l’accès, en activant des jetons d’accès de longue durée qui peuvent être révoqués en fonction d’événements critiques et de l’évaluation de la stratégie.
services: active-directory
author: knicholasa
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/06/2020
ms.author: nichola
ms.reviewer: ''
ms.openlocfilehash: f6ce792b3db0100d7356884bbc6ee2696580df10
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97652056"
---
# <a name="how-to-use-continuous-access-evaluation-enabled-apis-in-your-applications"></a>Guide pratique pour utiliser les API d’évaluation continue de l’accès dans vos applications

L’[évaluation continue de l’accès](../conditional-access/concept-continuous-access-evaluation.md) (CAE, Continuous Access Evaluation) est une norme industrielle émergente qui permet de révoquer les jetons d’accès en fonction d’[événements critiques](../conditional-access/concept-continuous-access-evaluation.md#critical-event-evaluation) et d’une [évaluation de la stratégie](../conditional-access/concept-continuous-access-evaluation.md#conditional-access-policy-evaluation-preview) au lieu de s’appuyer sur l’expiration du jeton en fonction de la durée de vie. Pour certaines API de ressources, comme les risques et les stratégies sont évalués en temps réel, cela peut augmenter la durée de vie du jeton jusqu’à 28 heures. Ces jetons de longue durée sont actualisés de manière proactive par la bibliothèque d’authentification Microsoft (MSAL), ce qui augmente la résilience de vos applications.

Cet article explique comment utiliser les API prenant en charge l’évaluation continue de l’accès dans vos applications.

## <a name="implementation-considerations"></a>Considérations relatives à l’implémentation

Pour utiliser l’évaluation continue de l’accès, votre application et l’API de ressource à laquelle elle accède doivent prendre en charge l’évaluation continue de l’accès. Toutefois, la préparation de votre code pour l’utilisation d’une ressource prenant en charge l’évaluation continue de l’accès ne vous empêche pas d’utiliser des API qui ne prennent pas en charge l’évaluation continue de l’accès.

Si une API de ressource implémente l’évaluation continue de l’accès et que votre application déclare qu’elle peut gérer l’évaluation continue de l’accès, votre application obtiendra des jetons d’évaluation continue de l’accès pour cette ressource. Pour cette raison, si vous déclarez que votre application est prête à utiliser l’évaluation continue de l’accès, votre application doit traiter le défi de revendications d’évaluation continue de l’accès pour toutes les API de ressources qui acceptent les jetons d’accès Microsoft Identity. Si vous ne gérez pas les réponses d’évaluation continue de l’accès dans ces appels d’API, votre application peut se retrouver à réessayer en boucle un appel d’API avec un jeton qui est toujours dans la durée de vie retournée du jeton, mais qui a été révoqué en raison de l’évaluation continue de l’accès.

## <a name="the-code"></a>Code

La première étape consiste à ajouter du code pour traiter une réponse de l’API de ressource qui rejette l’appel en raison de l’évaluation continue de l’accès. Avec l’évaluation continue de l’accès, les API retournent un état 401 et un en-tête WWW-Authenticate lorsque le jeton d’accès a été révoqué ou que l’API détecte une modification de l’adresse IP utilisée. L’en-tête WWW-Authenticate contient un défi de revendications que l’application peut utiliser pour acquérir un nouveau jeton d’accès.

Par exemple :

```console
HTTP 401; Unauthorized
WWW-Authenticate=Bearer
  authorization_uri="https://login.windows.net/common/oauth2/authorize",
  error="insufficient_claims",
  claims="eyJhY2Nlc3NfdG9rZW4iOnsibmJmIjp7ImVzc2VudGlhbCI6dHJ1ZSwgInZhbHVlIjoiMTYwNDEwNjY1MSJ9fX0="
```

Votre application recherche les éléments suivants :

- l’appel d’API retournant l’état 401
- l’existence d’un en-tête WWW-Authenticate contenant :
  - un paramètre « error » avec la valeur « insufficient_claims »
  - un paramètre « claims »

Lorsque ces conditions sont réunies, l’application peut extraire et décoder le défi de revendications.

```csharp
if (APIresponse.IsSuccessStatusCode)
{
    // ...
}
else
{
    if (APIresponse.StatusCode == System.Net.HttpStatusCode.Unauthorized
        && APIresponse.Headers.WwwAuthenticate.Any())
    {
        AuthenticationHeaderValue bearer = APIresponse.Headers.WwwAuthenticate.First
            (v => v.Scheme == "Bearer");
        IEnumerable<string> parameters = bearer.Parameter.Split(',').Select(v => v.Trim()).ToList();
        var error = GetParameter(parameters, "error");

        if (null != error && "insufficient_claims" == error)
        {
            var claimChallengeParameter = GetParameter(parameters, "claims");
            if (null != claimChallengeParameter)
            {
                var claimChallengebase64Bytes = System.Convert.FromBase64String(claimChallengeParameter);
                var claimChallenge = System.Text.Encoding.UTF8.GetString(claimChallengebase64Bytes);
                var newAccessToken = await GetAccessTokenWithClaimChallenge(scopes, claimChallenge);
```

Votre application utilise ensuite le défi de revendications pour acquérir un nouveau jeton d’accès pour la ressource.

```csharp
try
{
    authResult = await _clientApp.AcquireTokenSilent(scopes, firstAccount)
        .WithClaims(claimChallenge)
        .ExecuteAsync()
        .ConfigureAwait(false);
}
catch (MsalUiRequiredException)
{
    try
    {
        authResult = await _clientApp.AcquireTokenInteractive(scopes)
            .WithClaims(claimChallenge)
            .WithAccount(firstAccount)
            .ExecuteAsync()
            .ConfigureAwait(false);
    }
    // ...
```

Une fois que votre application est prête à traiter le défi de revendications retourné par une ressource prenant en charge l’évaluation continue de l’accès, vous pouvez indiquer à Microsoft Identity que votre application est prête à utiliser l’évaluation continue de l’accès. Pour effectuer cette opération dans votre application MSAL, générez votre client public à l’aide des fonctionnalités client de « cp1 ».

```csharp
_clientApp = PublicClientApplicationBuilder.Create(App.ClientId)
    .WithDefaultRedirectUri()
    .WithAuthority(authority)
    .WithClientCapabilities(new [] {"cp1"})
    .Build();
```

Vous pouvez tester votre application en connectant un utilisateur à l’application, puis en utilisant le portail Azure pour révoquer les sessions de l’utilisateur. La prochaine fois que l’application appellera l’API prenant en charge l’évaluation continue de l’accès, l’utilisateur sera invité à se réauthentifier.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Évaluation continue de l’accès](../conditional-access/concept-continuous-access-evaluation.md).
