---
title: Appeler une API web à partir d’une application mobile | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment générer une application mobile qui appelle des API web. (Appeler une API web.)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/18/2020
ms.author: jmprieur
ms.reviewer: brandwe
ms.custom: aaddev
ms.openlocfilehash: d7cad0592c5c4c0487f582ce5405c275b94b7bd0
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94444025"
---
# <a name="call-a-web-api-from-a-mobile-app"></a>Appeler une API web à partir d’une application mobile

Quand votre application connecte un utilisateur et reçoit des jetons, MSAL (Microsoft Authentication Library) expose des information sur l’utilisateur, l’environnement de l’utilisateur et les jetons émis. Votre application peut utiliser ces valeurs pour appeler une API web ou afficher un message de bienvenue à l’utilisateur.

Dans cet article, nous allons tout d’abord examiner le résultat de MSAL. Puis, nous verrons comment utiliser un jeton d’accès à partir de `AuthenticationResult` ou `result` pour appeler une API web protégée.

## <a name="msal-result"></a>Résultat de MSAL
MSAL propose les valeurs suivantes : 

- `AccessToken` appelle des API web protégées dans une requête HTTP du porteur.
- `IdToken` contient des informations utiles au sujet de l’utilisateur connecté. Ces informations incluent le nom de l’utilisateur, le locataire de base et un identificateur unique de stockage.
- `ExpiresOn` détermine le délai d’expiration du jeton. MSAL gère l’actualisation automatique d’une application.
- `TenantId` est l’identificateur du locataire où l’utilisateur est connecté. Pour les utilisateurs invités dans Azure Active Directory (Azure AD) B2B, cette valeur identifie le locataire où l’utilisateur est connecté. La valeur n’identifie pas le locataire de base de l’utilisateur.  
- `Scopes` indique les étendues qui ont été accordées avec votre jeton. Les étendues accordées peuvent être un sous-ensemble des étendues demandées.

MSAL fournit également une abstraction d’une valeur `Account`. Une valeur `Account` représente le compte de connexion de l’utilisateur actuel :

- `HomeAccountIdentifier` identifie le locataire de base de l’utilisateur.
- `UserName` est le nom d’utilisateur par défaut de l’utilisateur. Cette valeur peut être vide pour les utilisateurs d’Azure AD B2C.
- `AccountIdentifier` identifie l’utilisateur connecté. Dans la plupart des cas, cette valeur est identique à la valeur `HomeAccountIdentifier`, sauf si l’utilisateur est un invité dans un autre locataire.

## <a name="call-an-api"></a>Appeler une API

Une fois que vous disposez du jeton d’accès, vous pouvez appeler une API web. Votre application utilisera ensuite le jeton pour générer et exécuter une requête HTTP.

### <a name="android"></a>Android

```Java
        RequestQueue queue = Volley.newRequestQueue(this);
        JSONObject parameters = new JSONObject();

        try {
            parameters.put("key", "value");
        } catch (Exception e) {
            // Error when constructing.
        }
        JsonObjectRequest request = new JsonObjectRequest(Request.Method.GET, MSGRAPH_URL,
                parameters,new Response.Listener<JSONObject>() {
            @Override
            public void onResponse(JSONObject response) {
                // Successfully called Graph. Process data and send to UI.
            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                // Error.
            }
        }) {
            @Override
            public Map<String, String> getHeaders() throws AuthFailureError {
                Map<String, String> headers = new HashMap<>();
                
                // Put access token in HTTP request.
                headers.put("Authorization", "Bearer " + authResult.getAccessToken());
                return headers;
            }
        };

        request.setRetryPolicy(new DefaultRetryPolicy(
                3000,
                DefaultRetryPolicy.DEFAULT_MAX_RETRIES,
                DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        queue.add(request);
```

### <a name="msal-for-ios-and-macos"></a>MSAL pour iOS et macOS

Les méthodes d'acquisition de jetons renvoient un objet `MSALResult`. `MSALResult` expose une propriété `accessToken`. Vous pouvez utiliser `accessToken` pour appeler une API web. Ajoutez cette propriété à l’en-tête d’autorisation HTTP avant d’appeler l’API web protégée pour y accéder.

```objc
NSMutableURLRequest *urlRequest = [NSMutableURLRequest new];
urlRequest.URL = [NSURL URLWithString:"https://contoso.api.com"];
urlRequest.HTTPMethod = @"GET";
urlRequest.allHTTPHeaderFields = @{ @"Authorization" : [NSString stringWithFormat:@"Bearer %@", accessToken] };
        
NSURLSessionDataTask *task =
[[NSURLSession sharedSession] dataTaskWithRequest:urlRequest
     completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {}];
[task resume];
```

```swift
let urlRequest = NSMutableURLRequest()
urlRequest.url = URL(string: "https://contoso.api.com")!
urlRequest.httpMethod = "GET"
urlRequest.allHTTPHeaderFields = [ "Authorization" : "Bearer \(accessToken)" ]
     
let task = URLSession.shared.dataTask(with: urlRequest as URLRequest) { (data: Data?, response: URLResponse?, error: Error?) in }
task.resume()
```

### <a name="xamarin"></a>Xamarin

[!INCLUDE [Call web API in .NET](../../../includes/active-directory-develop-scenarios-call-apis-dotnet.md)]

## <a name="make-several-api-requests"></a>Effectuer plusieurs requêtes vers l’API

Si vous devez appeler la même API plusieurs fois, ou appeler plusieurs API, prenez en compte les points suivants quand vous créez votre application :

- **Consentement incrémentiel** : la plateforme d’identités Microsoft permet aux applications d’obtenir le consentement de l’utilisateur au moment où les autorisations sont requises, et non toutes les autorisations au début. Chaque fois que votre application est prête à appeler une API, elle doit demander uniquement les étendues dont elle a besoin.

- **Accès conditionnel** : quand vous effectuez plusieurs requêtes vers l’API, dans certains scénarios, vous devrez remplir des exigences d’accès conditionnel supplémentaires. Les exigences peuvent être plus nombreuses si la première requête n’a aucune stratégie d’accès conditionnel et que votre application tente d’accéder en mode silencieux à une nouvelle API qui requiert un accès conditionnel. Pour gérer ce problème, veillez à intercepter les erreurs des requêtes en mode silencieux et à vous préparer à effectuer une requête interactive.  Pour plus d’informations, consultez les [conseils en matière d’accès conditionnel](../azuread-dev/conditional-access-dev-guide.md).

## <a name="call-several-apis-by-using-incremental-consent-and-conditional-access"></a>Appeler plusieurs API à l’aide du consentement incrémentiel et de l’accès conditionnel

Si vous devez appeler plusieurs API pour le même utilisateur, après l’acquisition d’un jeton pour l’utilisateur, vous pouvez éviter de demander à l’utilisateur des informations d’identification à plusieurs reprises en appelant `AcquireTokenSilent` pour obtenir un jeton :

```csharp
var result = await app.AcquireTokenXX("scopeApi1")
                      .ExecuteAsync();

result = await app.AcquireTokenSilent("scopeApi2")
                  .ExecuteAsync();
```

Une interaction est obligatoire quand :

- L’utilisateur a donné son consentement pour la première API, mais doit également le donner pour d’autres d’étendues Dans ce cas, vous utilisez le consentement incrémentiel.
- La première API ne nécessite aucune [authentification multifacteur](../authentication/concept-mfa-howitworks.md), contrairement à l’API qui suit.

```csharp
var result = await app.AcquireTokenXX("scopeApi1")
                      .ExecuteAsync();

try
{
 result = await app.AcquireTokenSilent("scopeApi2")
                  .ExecuteAsync();
}
catch(MsalUiRequiredException ex)
{
 result = await app.AcquireTokenInteractive("scopeApi2")
                  .WithClaims(ex.Claims)
                  .ExecuteAsync();
}
```

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Passer en production](scenario-mobile-production.md).
