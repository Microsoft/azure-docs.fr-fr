---
title: Demander des revendications personnalisées (MSAL iOS/macOS) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment demander des revendications personnalisées.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 08/26/2019
ms.author: marsma
ms.custom: aaddev
ms.openlocfilehash: a570dccad5f14cf9adf5ca2825d8a3b31ae60d3f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85477190"
---
# <a name="how-to-request-custom-claims-using-msal-for-ios-and-macos"></a>Procédure : demander des revendications personnalisées à l’aide de MSAL pour iOS et macOS

OpenID Connect vous permet de demander éventuellement le retour de revendications individuelles à partir du point de terminaison UserInfo et/ou dans le jeton d’ID. Une demande de revendications est représentée sous la forme d’un objet JSON qui contient une liste des revendications demandées. Pour plus d’informations, consultez [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0-final.html#ClaimsParameter).

La bibliothèque MSAL (Microsoft Authentication Library) pour iOS et macOS permet de demander des revendications spécifiques dans des scénarios d’acquisition de jetons interactive et en mode silencieux. Elle utilise pour cela le paramètre `claimsRequest`.

Il existe plusieurs scénarios où cela est nécessaire. Par exemple :

- Demande de revendications en dehors de l’ensemble standard de votre application.
- Demande de combinaisons spécifiques des revendications standard qui ne peuvent pas être spécifiées à l’aide d’étendues pour votre application. Par exemple, si un jeton d’accès est rejeté car des revendications sont manquantes, l’application peut demander les revendications manquantes à l’aide de MSAL.

> [!NOTE]
> MSAL contourne le cache de jetons d’accès chaque fois qu’une demande de revendications est spécifiée. Il est important de spécifier le paramètre `claimsRequest` uniquement quand des revendications supplémentaires sont nécessaires (plutôt que de fournir toujours le même paramètre `claimsRequest` dans chaque appel d’API MSAL).

`claimsRequest` peut être spécifié dans `MSALSilentTokenParameters` et `MSALInteractiveTokenParameters` :

```objc
/*!
 MSALTokenParameters is the base abstract class for all types of token parameters (silent and interactive).
 */
@interface MSALTokenParameters : NSObject

/*!
 The claims parameter that needs to be sent to authorization or token endpoint.
 If claims parameter is passed in silent flow, access token will be skipped and refresh token will be tried.
 */
@property (nonatomic, nullable) MSALClaimsRequest *claimsRequest;

@end
```
`MSALClaimsRequest` peut être construit à partir d’une représentation NSString d’une demande de revendications JSON. 

Objective-C :

```objc
NSError *claimsError = nil;
MSALClaimsRequest *request = [[MSALClaimsRequest alloc] initWithJsonString:@"{\"id_token\":{\"auth_time\":{\"essential\":true},\"acr\":{\"values\":[\"urn:mace:incommon:iap:silver\"]}}}" error:&claimsError];
```

Swift :

```swift
var requestError: NSError? = nil
let request = MSALClaimsRequest(jsonString: "{\"id_token\":{\"auth_time\":{\"essential\":true},\"acr\":{\"values\":[\"urn:mace:incommon:iap:silver\"]}}}",
                                        error: &requestError)
```



Il peut également être modifié en demandant des revendications spécifiques supplémentaires :

Objective-C :

```objc
MSALIndividualClaimRequest *individualClaimRequest = [[MSALIndividualClaimRequest alloc] initWithName:@"custom_claim"];
individualClaimRequest.additionalInfo = [MSALIndividualClaimRequestAdditionalInfo new];
individualClaimRequest.additionalInfo.essential = @1;
individualClaimRequest.additionalInfo.value = @"myvalue";
[request requestClaim:individualClaimRequest forTarget:MSALClaimsRequestTargetIdToken error:&claimsError];
```

Swift :

```swift
let individualClaimRequest = MSALIndividualClaimRequest(name: "custom-claim")
let additionalInfo = MSALIndividualClaimRequestAdditionalInfo()
additionalInfo.essential = 1
additionalInfo.value = "myvalue"
individualClaimRequest.additionalInfo = additionalInfo
do {
  try request.requestClaim(individualClaimRequest, for: .idToken)
} catch let error as NSError {
  // handle error here  
}
        
```



`MSALClaimsRequest` doit ensuite être défini dans les paramètres de jeton, et fourni à l’une des API d’acquisition de jetons MSAL :

Objective-C :

```objc
MSALPublicClientApplication *application = ...;
MSALWebviewParameters *webParameters = ...;

MSALInteractiveTokenParameters *parameters = [[MSALInteractiveTokenParameters alloc] initWithScopes:@[@"user.read"]
                                                                                  webviewParameters:webParameters];
parameters.claimsRequest = request;
    
[application acquireTokenWithParameters:parameters completionBlock:completionBlock];
```

Swift :

```swift
let application: MSALPublicClientApplication!
let webParameters: MSALWebviewParameters!
        
let parameters = MSALInteractiveTokenParameters(scopes: ["user.read"], webviewParameters: webParameters)
parameters.claimsRequest = request
        
application.acquireToken(with: parameters) { (result: MSALResult?, error: Error?) in            ...

```



## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [Flux d’authentification et scénarios d’applications](authentication-flows-app-scenarios.md)
