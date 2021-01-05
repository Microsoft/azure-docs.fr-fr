---
title: Authentification unique entre les applications ADAL et MSAL (iOS/macOS ) - Plateforme des identités Microsoft | Azure
description: ''
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/28/2019
ms.author: marsma
ms.reviewer: ''
ms.custom: aaddev
ms.openlocfilehash: 7a8a1667ba1ca2a99c053c6941e3ba778299fd53
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80880748"
---
# <a name="how-to-sso-between-adal-and-msal-apps-on-macos-and-ios"></a>Procédure : Authentification unique entre les applications ADAL et MSAL sur macOS et iOS

Microsoft Authentication Library (MSAL) pour iOS peut partager l’état de l’authentification unique avec [ADAL Objective-C](https://github.com/AzureAD/azure-activedirectory-library-for-objc) entre les applications. Vous pouvez migrer vos applications vers MSAL à votre propre rythme, en veillant à ce que vos utilisateurs bénéficient toujours de l’authentification unique entre applications, même avec une combinaison d’applications basées sur ADAL et MSAL.

Si vous recherchez des instructions sur la configuration de l’authentification unique entre les applications à l’aide du SDK MSAL, consultez [Authentification unique silencieuse entre plusieurs applications](single-sign-on-macos-ios.md#silent-sso-between-apps). Cet article porte sur l’authentification unique entre ADAL et MSAL.

Les spécificités de l’implémentation de l’authentification unique dépendent de la version d’ADAL que vous utilisez.

## <a name="adal-27x"></a>ADAL 2.7.x

Cette section traite des différences d’authentification unique entre MSAL et ADAL 2.7.x.

### <a name="cache-format"></a>Format du cache

ADAL 2.7.x peut lire le format de cache MSAL. Vous n’avez rien à faire de particulier pour l’authentification unique entre applications avec ADAL 2.7.x. Toutefois, vous devez connaître les différences entre les identificateurs de compte pris en charge par ces deux bibliothèques.

### <a name="account-identifier-differences"></a>Différences entre les identificateurs de compte

MSAL et ADAL utilisent des identificateurs de compte différents. ADAL utilise l’UPN comme identificateur de compte principal. MSAL utilise un identificateur de compte non affichable qui est basé sur un ID d’objet et un ID de locataire pour les comptes AAD, et une revendication `sub` pour les autres types de comptes.

Quand vous recevez un objet `MSALAccount` dans le résultat MSAL, celui-ci contient un identificateur de compte dans la propriété `identifier`. L’application doit utiliser cet identificateur pour les demandes silencieuses suivantes.

Outre `identifier`, l’objet `MSALAccount` contient un identificateur affichable appelé `username`. Il se traduit par `userId` dans ADAL. `username` n’est pas considéré comme un identificateur unique et peut changer à tout moment. Vous devez donc l’utiliser uniquement pour les scénarios de compatibilité descendante avec ADAL. MSAL prend en charge les requêtes de cache à l’aide de `username` ou `identifier`, où l’interrogation par `identifier` est recommandée.

Le tableau suivant récapitule les différences d’identificateur de compte entre ADAL et MSAL :

| Identificateur de compte                | MSAL                                                         | ADAL 2.7.x      | Ancienne ADAL (avant ADAL 2.7.x) |
| --------------------------------- | ------------------------------------------------------------ | --------------- | ------------------------------ |
| Identificateur affichable            | `username`                                                   | `userId`        | `userId`                       |
| Identificateur unique non affichable | `identifier`                                                 | `homeAccountId` | N/A                            |
| Aucun ID de compte connu               | Interroger tous les comptes par le biais de l’API `allAccounts:` dans `MSALPublicClientApplication` | N/A             | N/A                            |

Voici l’interface `MSALAccount` qui fournit ces identificateurs :

```objc
@protocol MSALAccount <NSObject>

/*!
 Displayable user identifier. Can be used for UI and backward compatibility with ADAL.
 */
@property (readonly, nullable) NSString *username;

/*!
 Unique identifier for the account.
 Save this for account lookups from cache at a later point.
 */
@property (readonly, nullable) NSString *identifier;

/*!
 Host part of the authority string used for authentication based on the issuer identifier.
 */
@property (readonly, nonnull) NSString *environment;

/*!
 ID token claims for the account.
 Can be used to read additional information about the account, e.g. name
 Will only be returned if there has been an id token issued for the client Id for the account's source tenant.
 */
@property (readonly, nullable) NSDictionary<NSString *, NSString *> *accountClaims;

@end
```

### <a name="sso-from-msal-to-adal"></a>Authentification unique de MSAL vers ADAL

Si vous avez une application MSAL et une application ADAL, et que l’utilisateur se connecte en premier à l’application MSAL, vous pouvez bénéficier de l’authentification unique dans l’application ADAL en enregistrant le `username` à partir de l’objet `MSALAccount` et en le transmettant à votre application basée sur ADAL comme `userId`. ADAL peut ensuite trouver les informations de compte de manière silencieuse avec l’API `acquireTokenSilentWithResource:clientId:redirectUri:userId:completionBlock:`.

### <a name="sso-from-adal-to-msal"></a>Authentification unique d’ADAL vers MSAL

Si vous avez une application MSAL et une application ADAL, et que l’utilisateur se connecte en premier à l’application ADAL, vous pouvez utiliser des identificateurs d’utilisateur ADAL pour les recherches de comptes dans MSAL. Cela s’applique également lors de la migration d’ADAL vers MSAL.

#### <a name="adals-homeaccountid"></a>homeAccountId d’ADAL

ADAL 2.7.x retourne le `homeAccountId` dans l’objet `ADUserInformation` dans le résultat par le biais de cette propriété :

```objc
/*! Unique AAD account identifier across tenants based on user's home OID/home tenantId. */
@property (readonly) NSString *homeAccountId;
```

`homeAccountId` dans ADAL est équivalent à `identifier` dans MSAL. Vous pouvez enregistrer cet identificateur afin de l’utiliser dans MSAL pour les recherches de comptes avec l’API `accountForIdentifier:error:`.

#### <a name="adals-userid"></a>`userId` d’ADAL

Si `homeAccountId` n’est pas disponible, ou si vous ne disposez que de l’identificateur affichable, vous pouvez utiliser le `userId` d’ADAL pour rechercher le compte dans MSAL.

Dans MSAL, commencez par rechercher un compte par `username` ou `identifier`. Utilisez toujours `identifier` pour l’interrogation si vous l’avez et utilisez uniquement `username` en dernier recours. Si le compte est trouvé, utilisez-le dans les appels `acquireTokenSilent`.

Objective-C :

```objc
NSString *msalIdentifier = @"previously.saved.msal.account.id";
MSALAccount *account = nil;
    
if (msalIdentifier)
{
    // If you have MSAL account id returned either from MSAL as identifier or ADAL as homeAccountId, use it
    account = [application accountForIdentifier:@"my.account.id.here" error:nil];
}
else
{
    // Fallback to ADAL userId for migration
    account = [application accountForUsername:@"adal.user.id" error:nil];
}
    
if (!account)
{
  // Account not found.
  return;
}

MSALSilentTokenParameters *silentParameters = [[MSALSilentTokenParameters alloc] initWithScopes:@[@"user.read"] account:account];
[application acquireTokenSilentWithParameters:silentParameters completionBlock:completionBlock];
```

Swift :

```swift
        
let msalIdentifier: String?
var account: MSALAccount
        
do {
  if let msalIdentifier = msalIdentifier {
    account = try application.account(forIdentifier: msalIdentifier)
  }
  else {
    account = try application.account(forUsername: "adal.user.id") 
  }
             
  let silentParameters = MSALSilentTokenParameters(scopes: ["user.read"], account: account)          
  application.acquireTokenSilent(with: silentParameters) {
    (result: MSALResult?, error: Error?) in
    // handle result
  }  
} catch let error as NSError {
  // handle error or account not found
}
```



API de recherche de compte prises en charge par MSAL :

```objc
/*!
 Returns account for the given account identifier (received from an account object returned in a previous acquireToken call)
 
 @param  error      The error that occurred trying to get the accounts, if any, if you're
                    not interested in the specific error pass in nil.
 */
- (nullable MSALAccount *)accountForIdentifier:(nonnull NSString *)identifier
                                         error:(NSError * _Nullable __autoreleasing * _Nullable)error;
    
/*!
Returns account for for the given username (received from an account object returned in a previous acquireToken call or ADAL)
    
@param  username    The displayable value in UserPrincipleName(UPN) format
@param  error       The error that occurred trying to get the accounts, if any, if you're
                    not interested in the specific error pass in nil.
*/
- (MSALAccount *)accountForUsername:(NSString *)username
                              error:(NSError * __autoreleasing *)error;
```

## <a name="adal-2x-266"></a>ADAL 2.x-2.6.6

Cette section traite des différences d’authentification unique entre MSAL et ADAL 2.x-2.6.6.

Les anciennes versions d’ADAL ne prennent pas en charge le format de cache MSAL en mode natif. Toutefois, pour garantir une migration sans heurts d’ADAL vers MSAL, MSAL peut lire l’ancien format de cache ADAL sans redemander les informations d’identification de l’utilisateur.

`homeAccountId` n’étant pas disponible dans les anciennes versions d’ADAL, vous devez rechercher des comptes à l’aide du `username` :

```objc
/*!
 Returns account for for the given username (received from an account object returned in a previous acquireToken call or ADAL)

 @param  username    The displayable value in UserPrincipleName(UPN) format
 @param  error       The error that occurred trying to get the accounts, if any.  If you're not interested in the specific error pass in nil.
 */
- (MSALAccount *)accountForUsername:(NSString *)username
                              error:(NSError * __autoreleasing *)error;
```

Par exemple :

Objective-C :


```objc
MSALAccount *account = [application accountForUsername:@"adal.user.id" error:nil];;
MSALSilentTokenParameters *silentParameters = [[MSALSilentTokenParameters alloc] initWithScopes:@[@"user.read"] account:account];
[application acquireTokenSilentWithParameters:silentParameters completionBlock:completionBlock];
```

Swift :

```swift
do {
  let account = try application.account(forUsername: "adal.user.id")          
  let silentParameters = MSALSilentTokenParameters(scopes: ["user.read"], account: account)
  application.acquireTokenSilent(with: silentParameters) { 
    (result: MSALResult?, error: Error?) in
    // handle result
  }   
} catch let error as NSError { 
  // handle error or account not found
}
```



Vous pouvez aussi lire tous les comptes, ce qui lira également les informations des comptes à partir d’ADAL :

Objective-C :

```objc
NSArray *accounts = [application allAccounts:nil];
    
if ([accounts count] == 0)
{
  // No account found.
  return; 
}
if ([accounts count] > 1)
{
  // You might want to display an account picker to user in actual application
  // For this sample we assume there's only ever one account in cache
  return;
}
    ``
MSALSilentTokenParameters *silentParameters = [[MSALSilentTokenParameters alloc] initWithScopes:@[@"user.read"] account:accounts[0]];
[application acquireTokenSilentWithParameters:silentParameters completionBlock:completionBlock];
```

Swift :

```swift
      
do {
  let accounts = try application.allAccounts()
  if accounts.count == 0 {
    // No account found.
    return
  }
  if accounts.count > 1 {
    // You might want to display an account picker to user in actual application
    // For this sample we assume there's only ever one account in cache
    return
  }
  
  let silentParameters = MSALSilentTokenParameters(scopes: ["user.read"], account: accounts[0])
  application.acquireTokenSilent(with: silentParameters) {
    (result: MSALResult?, error: Error?) in
    // handle result or error  
  }  
} catch let error as NSError { 
  // handle error
}
```



## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [Flux d’authentification et scénarios d’applications](authentication-flows-app-scenarios.md)
