---
title: Codes d’erreur d’API REST - Azure Key Vault
description: Ces codes d’erreur peuvent être retournés par une opération effectuée sur un service web Azure Key Vault.
keywords: ''
services: machine-learning
author: msmbaldwin
ms.custom: seodec18
ms.author: mbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: reference
ms.date: 12/16/2019
ms.openlocfilehash: 30b7e34f2a791cfd8dec1a6d8e81d706fa07939f
ms.sourcegitcommit: d479ad7ae4b6c2c416049cb0e0221ce15470acf6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/01/2020
ms.locfileid: "91631220"
---
# <a name="azure-key-vault-rest-api-error-codes"></a>Codes d’erreur d’API REST Azure Key Vault
 
Les codes d’erreur suivants peuvent être retournés par une opération effectuée sur un service web Azure Key Vault.
 
## <a name="http-401-unauthenticated-request"></a>HTTP 401 : Requête non authentifiée

Une erreur 401 signifie que la requête n’est pas authentifiée pour Key Vault. 

Une requête est authentifiée si :

- Le coffre de clés connaît l’identité de l’appelant ; et
- L’appelant est autorisé à essayer d’accéder aux ressources Key Vault. 

Il existe plusieurs raisons pour lesquelles une requête peut retourner une erreur 401.

### <a name="no-authentication-token-attached-to-the-request"></a>Aucun jeton d’authentification n’est associé à la requête. 

Voici un exemple de requête PUT, qui définit la valeur d’une clé secrète :

``` 
PUT https://putreqexample.vault.azure.net//secrets/DatabaseRotatingPassword?api-version=7.0 HTTP/1.1
x-ms-client-request-id: 03d275a2-52a4-4bed-82c8-6fe15165affb
accept-language: en-US
Authorization: Bearer     eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Im5iQ3dXMTF3M1hrQi14VWFYd0tSU0xqTUhHUSIsImtpZCI6Im5iQ3dXMTF3M1hrQi14VWFYd0tSU0xqTUhHUSJ9.eyJhdWQiOiJodHRwczovL3ZhdWx0LmF6dXJlLm5ldCIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0Ny8iLCJpYXQiOjE1NDg2OTc1MTMsIm5iZiI6MTU0ODY5NzUxMywiZXhwIjoxNTQ4NzAxNDEzLCJhaW8iOiI0MkpnWUhoODVqaVBnZHF5ZlRGZE5TdHY3bGUvQkFBPSIsImFwcGlkIjoiZmFkN2Q1YjMtNjlkNi00YjQ4LTkyNTktOGQxMjEyNGUxY2YxIiwiYXBwaWRhY3IiOiIxIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNzJmOTg4YmYtODZmMS00MWFmLTkxYWItMmQ3Y2QwMTFkYjQ3LyIsIm9pZCI6IjM5NzVhZWVkLTdkMDgtNDUzYi1iNmY0LTQ0NWYzMjY5ODA5MSIsInN1YiI6IjM5NzVhZWVkLTdkMDgtNDUzYi1iNmY0LTQ0NWYzMjY5ODA5MSIsInRpZCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsInV0aSI6IjItZ3JoUmtlSWs2QmVZLUxuNDJtQUEiLCJ2ZXIiOiIxLjAifQ.fgubiz1MKqTJTXI8dHIV7t9Fle6FdHrkaGYKcBeVRX1WtLVuk1QVxzIFDlZKLXJ7QPNs0KWpeiWQI9IWIRK-8wO38yCqKTfDlfHOiNWGOpkKddlG729KFqakVf2w0GPyGPFCONRDAR5wjQarN9Bt8I8YbHwZQz_M1hztlnv-Lmsk1jBmech9ujD9-lTMBmSfFFbHcqquev119V7sneI-zxBZLf8C0pIDkaXf1t8y6Xr8CUJDMdlWLslCf3pBCNIOy65_TyGvy4Z4AJryTPBarNBPwOkNAtjCfZ4BDc2KqUZM5QN_VK4foP64sVzUL6mSr0Gh7lQJIL5b1qIpJxjxyQ
User-Agent: FxVersion/4.7.3324.0 OSName/Windows OSVersion/6.2.9200.0 Microsoft.Azure.KeyVault.KeyVaultClient/3.0.3.0
Content-Type: application/json; charset=utf-8
Host: putreqexample.vault.azure.net
Content-Length: 31

{
   "value": "m*gBJ7$Zuoz)"
}
```

L’en-tête « Authorization » correspond au jeton d’accès requis avec chaque appel vers Key Vault pour les opérations de plan de données. Si l’en-tête est manquant, la réponse doit être 401.

### <a name="the-token-lacks-the-correct-resource-associated-with-it"></a>La ressource appropriée n’est pas associée au jeton. 

Un paramètre appelé « resource » est obligatoire lors d’une demande de jeton d’accès à partir du point de terminaison Azure OAUTH. La valeur est importante pour le fournisseur de jetons, car elle limite le jeton à son utilisation prévue. La ressource des jetons **all** permettant d'accéder à un coffre de clés Key Vault est *https:\//vault.keyvault.net* (sans barre oblique de fin).

### <a name="the-token-is-expired"></a>Le jeton a expiré

Les jetons sont encodés en base64 et les valeurs peuvent être décodées sur des sites web, par exemple [http://jwt.calebb.net](http://jwt.calebb.net). Voici le jeton ci-dessus décodé :

```
    {
 typ: "JWT",
 alg: "RS256",
 x5t: "nbCwW11w3XkB-xUaXwKRSLjMHGQ",
 kid: "nbCwW11w3XkB-xUaXwKRSLjMHGQ"
}.
{
 aud: "https://vault.azure.net",
 iss: "https://sts.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db47/",
 iat: 1548697513,
 nbf: 1548697513,
 exp: 1548701413,
 aio: "42JgYHh85jiPgdqyfTFdNStv7le/BAA=",
 appid: "fad7d5b3-69d6-4b48-9259-8d12124e1cf1",
 appidacr: "1",
 idp: "https://sts.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db47/",
 oid: "3975aeed-7d08-453b-b6f4-445f32698091",
 sub: "3975aeed-7d08-453b-b6f4-445f32698091",
 tid: "72f988bf-86f1-41af-91ab-2d7cd011db47",
 uti: "2-grhRkeIk6BeY-Ln42mAA",
 ver: "1.0"
}.
[signature]
```

Nous pouvons voir de nombreuses parties importantes dans ce jeton :

- aud (public) : Ressource du jeton. Notez qu’il s’agit de `https://vault.azure.net`. Ce jeton ne fonctionnera PAS pour les ressources qui ne correspondent pas explicitement à cette valeur, un graphique par exemple.
- iat (émis à) : Nombre de cycles depuis le début de la période où le jeton a été émis.
- nbf (pas avant) : Nombre de cycles depuis le début de la période où ce jeton devient valide.
- exp (expiration) : Nombre de cycles depuis le début de la période où ce jeton arrive à expiration.
- appid (ID d’application) : GUID de l’ID d’application qui effectue cette requête.
- tid (ID de locataire) : GUID de l’ID de locataire du principal qui effectue cette requête

Il est important que toutes les valeurs soient correctement identifiées dans le jeton pour que la requête fonctionne. Si tout est correct, la requête ne retournera pas une erreur 401.

### <a name="troubleshooting-401"></a>Résolution des problèmes 401

Les erreurs 401 doivent être examinées à partir du point de génération de jetons, avant que la requête ne soit envoyée au coffre de clés. En général, le code est utilisé pour demander le jeton. Une fois le jeton reçu, il est transmis dans la requête Key Vault. Si le code s’exécute localement, vous pouvez utiliser Fiddler pour capturer la requête/réponse dans `https://login.microsoftonline.com`. Une requête ressemble à ceci :

``` 
POST https://login.microsoftonline.com/<key vault tenant ID>/oauth2/token HTTP/1.1
Accept: application/json
Content-Type: application/x-www-form-urlencoded; charset=utf-8
Host: login.microsoftonline.com
Content-Length: 192

resource=https%3A%2F%2Fvault.azure.net&client_id=<registered-app-ID>&client_secret=<registered-app-secret>&client_info=1&grant_type=client_credentials
```

Les informations fournies par l’utilisateur suivantes doivent être correctes :

- ID de locataire du coffre de clés
- Valeur de ressource définie sur https%3A%2F%2Fvault.azure.net (encodée URL)
- ID client
- Clé secrète client

Assurez-vous que le reste de la requête est quasiment identique.

Si vous ne pouvez obtenir que le jeton d’accès de réponse, vous pouvez le décoder (comme indiqué ci-dessus) pour vérifier l’ID de locataire, l’ID client (ID de l’application) et la ressource.

## <a name="http-403-insufficient-permissions"></a>HTTP 403 : Autorisations insuffisantes

Une erreur HTTP 403 signifie que la requête a été authentifiée (elle connaît l’identité à l’origine de la requête), mais que l’identité n’est pas autorisée à accéder à la ressource demandée. Deux causes sont possibles :

- Il n’existe aucune stratégie d’accès pour l’identité.
- L’adresse IP de la ressource à l’origine de la requête n’est pas approuvée dans les paramètres de pare-feu du coffre de clés.

Une erreur HTTP 403 se produit souvent lorsque l’application du client n’utilise pas l’ID client à laquelle le client s’attend. Cela signifie généralement que les stratégies d’accès ne sont pas correctement configurées pour l’identité appelante réelle.

### <a name="troubleshooting-403"></a>Résolution des problèmes 403

Activez tout d’abord la journalisation. Pour obtenir des instructions sur la façon de procéder, consultez [Journalisation d’Azure Key Vault](logging.md).

Une fois la journalisation activée, vous pouvez déterminer si l’erreur 403 est due à une stratégie d’accès ou à une stratégie de pare-feu.

#### <a name="error-due-to-firewall-policy"></a>Erreur due à une stratégie de pare-feu

« L’adresse du client (00.00.00.00) n’est pas autorisée et l’appelant n’est pas un service approuvé »

Il existe une liste limitée de « services approuvés Azure ». Les sites web Azure ne sont **pas** un service Azure approuvé. Pour plus d’informations, consultez le billet de blog [Key Vault Firewall access by Azure App Services](https://azidentity.azurewebsites.net/post/2019/01/03/key-vault-firewall-access-by-azure-app-services).

Vous devez ajouter l’adresse IP du site web Azure au Key Vault pour qu’il fonctionne.

Si l’erreur est due à une stratégie d accès : recherchez l’ID d’objet de la requête et vérifiez que l’ID d’objet correspond à l’objet auquel l’utilisateur tente d’attribuer la stratégie d’accès. Souvent, plusieurs objets dans AAD porteront le même nom. Par conséquent, il est très important de bien choisir l’objet approprié. Il est possible de voir s’il existe plusieurs objets portant le même nom en supprimant et en rajoutant la stratégie d’accès.

En outre, la plupart des stratégies d’accès ne nécessitent pas l’utilisation d’« applications autorisées », comme indiqué dans le portail. Les applications autorisées sont utilisées pour les scénarios d’authentification « au nom de », qui sont rares. 


## <a name="http-429-too-many-requests"></a>HTTP 429 : Trop de demandes

Une limitation de requêtes se produit lorsque le nombre de requêtes dépasse la valeur maximale indiquée pour la période. En cas de limitation de requêtes, la réponse du Key Vault sera une erreur HTTP 429. Des valeurs maximales sont définies pour des types de requêtes effectuées. Par exemple : la création d’une clé HSM 2 048 bits est de 5 requêtes toutes les 10 secondes, mais toutes les autres transactions HSM ont une limite de 1 000 requêtes/10 secondes. Il est donc important de savoir quels types d’appels sont effectués lors de la détermination de la cause de la limitation de requêtes.
En général, les requêtes adressées au Key Vault sont limitées à 2 000 requêtes/10 secondes. Les opérations de clés sont des exceptions, comme indiqué dans [Limites de service Key Vault](service-limits.md)

### <a name="troubleshooting-429"></a>Résolution de problèmes 429
La limitation de requêtes est contournée à l’aide des techniques suivantes :

- Réduisez le nombre de requêtes adressées au Key Vault en déterminant s’il existe des Modèles dans une ressource demandée et en tentant de les mettre en cache dans l’application appelante. 

- Lorsqu’une limitation de requêtes Key Vault se produit, adaptez le code demandeur pour qu’il utilise un backoff exponentiel dans une nouvelle tentative. L’algorithme est expliqué ici : [Limiter votre application](overview-throttling.md#how-to-throttle-your-app-in-response-to-service-limits)

- Si le nombre de requêtes ne peut pas être réduit en mettant en cache et si le backoff temporisé ne fonctionne pas, envisagez de fractionner les clés en plusieurs coffres de clés. La limite de service pour un abonnement est de 5 fois la limite de Key Vault individuel. Si vous utilisez plus de 5 coffres de clés, vous devez envisager l’utilisation de plusieurs abonnements. 

Des instructions détaillées, notamment sur la demande d’augmentation des limites, sont disponibles ici : [Aide sur la limitation de requêtes Key Vault](overview-throttling.md)
