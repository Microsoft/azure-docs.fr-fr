---
title: S’authentifier auprès d’Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Découvrez les différentes façons dont une application ou un service peut s’authentifier auprès de Communication Services.
author: matthewrobertson
manager: jken
services: azure-communication-services
ms.author: marobert
ms.date: 07/24/2020
ms.topic: conceptual
ms.service: azure-communication-services
ms.openlocfilehash: 4d6e02852dcd2d30a764417a4b5e0e012a1d2ab5
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96571094"
---
# <a name="authenticate-to-azure-communication-services"></a>S’authentifier auprès d’Azure Communication Services

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

Cet article fournit des informations sur l’authentification des clients auprès d’Azure Communication Services à l’aide des *clés d’accès* et des *jetons d’accès utilisateur*. Chaque interaction client avec Azure Communication Services doit être authentifiée.

Le tableau suivant décrit les options d’authentification prises en charge par les bibliothèques clientes d’Azure Communication Services :

| Bibliothèque cliente | Clé d’accès    | Jetons d’accès utilisateur |
| -------------- | ------------- | ------------------ |
| Administration | Prise en charge     | Non pris en charge      |
| SMS            | Prise en charge     | Non pris en charge      |
| Conversation           | Non pris en charge | Prise en charge          |
| Appel        | Non pris en charge | Prise en charge          |

Chaque option d’autorisation est décrite brièvement ci-dessous :

- Authentification par **clé d’accès** pour les opérations SMS et d’administration. L’authentification par clé d’accès est adaptée aux applications qui s’exécutent dans un environnement de service approuvé. Pour s’authentifier avec une clé d’accès, le client génère un [code HMAC (Hash-based Message Authentication Code)](https://en.wikipedia.org/wiki/HMAC) et l’intègre dans l’en-tête `Authorization` de chaque requête HTTP. Pour plus d’informations, consultez [S’authentifier avec une clé d’accès](#authenticate-with-an-access-key).
- Authentification par **jeton d’accès utilisateur** pour les conversations et les appels. Les jetons d’accès utilisateur permettent à vos applications clientes de s’authentifier directement auprès d’Azure Communication Services. Ces jetons sont générés sur un service de provisionnement de jetons côté serveur, que vous créez. Ils sont ensuite fournis aux appareils clients qui utilisent ce jeton pour initialiser les bibliothèques clientes de conversation et d’appel. Pour plus d’informations, consultez [S’authentifier avec un jeton d’accès utilisateur](#authenticate-with-a-user-access-token).

## <a name="authenticate-with-an-access-key"></a>S’authentifier à l’aide d’une clé d’accès

L’authentification par clé d’accès utilise une clé secrète partagée pour générer un code HMAC pour chaque requête HTTP calculée à l’aide de l’algorithme SHA256 et l’envoie dans l’en-tête `Authorization` à l’aide du schéma `HMAC-SHA256`.

```
Authorization: "HMAC-SHA256 SignedHeaders=date;host;x-ms-content-sha256&Signature=<hmac-sha256-signature>"
```

Les bibliothèques clientes Azure Communication Services qui utilisent l’authentification par clé d’accès doivent être initialisées avec la chaîne de connexion de votre ressource. Si vous n’utilisez pas de bibliothèque cliente, vous pouvez générer des HMAC par programmation à l’aide de la clé d’accès de votre ressource. Pour en savoir plus sur les chaînes de connexion, consultez le [Guide de démarrage rapide de l’approvisionnement des ressources](../quickstarts/create-communication-resource.md).

### <a name="sign-an-http-request"></a>Signer une requête HTTP

Si vous n’utilisez pas de bibliothèque cliente pour effectuer les requêtes HTTP aux API REST d’Azure Communication Services, vous devez créer par programmation des codes HMAC pour chaque requête HTTP. Les étapes suivantes décrivent comment construire l’en-tête d’authorisation :

1. Spécifiez l’horodatage en temps universel coordonné (UTC) de la requête dans l’en-tête `x-ms-date` ou dans l’en-tête de `Date` HTTP standard. Le service valide ceci pour assure la protection contre certaines attaques de sécurité, notamment les attaques par relecture.
1. Hachez le corps de la requête HTTP à l’aide de l’algorithme SHA256, puis transmettez-le à la requête via l’en-tête `x-ms-content-sha256`.
1. Construisez la chaîne à signer en concaténant le verbe HTTP (par exemple, `GET` ou `PUT`), le chemin d’accès de la requête HTTP et les valeurs des en-têtes HTTP `Date`, `Host` et `x-ms-content-sha256` au format suivant :
    ```
    VERB + "\n"
    URLPathAndQuery + "\n"
    DateHeaderValue + ";" + HostHeaderValue + ";" + ContentHashHeaderValue
    ```
1. Générez une signature HMAC-256 de la chaîne encodée en UTF-8 que vous avez créée à l’étape précédente. Encodez ensuite vos résultats au format Base64. Notez que vous devez également décoder votre clé d’accès en base64. Utilisez le format suivant (affiché comme pseudocode) :
    ```
    Signature=Base64(HMAC-SHA256(UTF8(StringToSign), Base64.decode(<your_access_key>)))
    ```
1. Spécifiez l’en-tête d’autorisation comme suit :
    ```
    Authorization="HMAC-SHA256 SignedHeaders=date;host;x-ms-content-sha256&Signature=<hmac-sha256-signature>"  
    ```
    Où `<hmac-sha256-signature>` est le HMAC calculé à l’étape précédente.

## <a name="authenticate-with-a-user-access-token"></a>S’authentifier avec un jeton d’accès utilisateur

Les jetons d’accès utilisateur permettent à vos applications clientes de s’authentifier directement auprès d’Azure Communication Services. Pour ce faire, vous devez configurer un service approuvé qui authentifie les utilisateurs de votre application et émet des jetons d’accès utilisateur avec la bibliothèque cliente d’administration. Pour en savoir plus sur nos considérations en matière d’architecture, consultez la documentation conceptuelle [architecture client et serveur](./client-and-server-architecture.md).

La classe `CommunicationUserCredential` contient la logique permettant de fournir des informations d’identification de jeton d’accès utilisateur aux bibliothèques clientes et de gérer leur cycle de vie.

### <a name="initialize-the-client-libraries"></a>Initialiser les bibliothèques client

Pour initialiser les bibliothèques clientes Azure Communication Services qui requièrent l’authentification par jeton d’accès utilisateur, vous devez d’abord créer une instance de la classe `CommunicationUserCredential`, puis l’utiliser pour initialiser un client API.

Les extraits de code suivants montrent comment initialiser la bibliothèque cliente de conversation avec un jeton d’accès utilisateur :

#### <a name="c"></a>[C#](#tab/csharp)

```csharp
// user access tokens should be created by a trusted service using the Administration client library
var token = "<valid-user-access-token>";

// create a CommunicationUserCredential instance
var userCredential = new CommunicationUserCredential(token);

// initialize the chat client library with the credential
var chatClient = new ChatClient(ENDPOINT_URL, userCredential);
```

#### <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
// user access tokens should be created by a trusted service using the Administration client library
const token = "<valid-user-access-token>";

// create a CommunicationUserCredential instance with the AzureCommunicationUserCredential class
const userCredential = new AzureCommunicationUserCredential(token);

// initialize the chat client library with the credential
let chatClient = new ChatClient(ENDPOINT_URL, userCredential);
```

#### <a name="swift"></a>[Swift](#tab/swift)

```swift
// user access tokens should be created by a trusted service using the Administration client library
let token = "<valid-user-access-token>";

// create a CommunicationUserCredential instance
let userCredential = try CommunicationUserCredential(token: token)

// initialize the chat client library with the credential
let chatClient = try CommunicationChatClient(credential: userCredential, endpoint: ENDPOINT_URL)
```

#### <a name="java"></a>[Java](#tab/java)

```java
// user access tokens should be created by a trusted service using the Administration client library
String token = "<valid-user-access-token>";

// create a CommunicationUserCredential instance
CommunicationUserCredential userCredential = new CommunicationUserCredential(token);

// Initialize the chat client
final ChatClientBuilder builder = new ChatClientBuilder();
builder.endpoint(ENDPOINT_URL)
    .credential(userCredential)
    .httpClient(HTTP_CLIENT);
ChatClient chatClient = builder.buildClient();
```

---

### <a name="refreshing-user-access-tokens"></a>Actualisation des jetons d’accès utilisateur

Les jetons d’accès utilisateur sont des informations d’identification de courte durée qui doivent être réémises afin d’éviter que les utilisateurs rencontrent des interruptions de service. Le constructeur `CommunicationUserCredential` accepte une fonction de rappel d’actualisation qui vous permet de mettre à jour les jetons d’accès utilisateur avant qu’ils n’expirent. Vous devez utiliser ce rappel pour récupérer un nouveau jeton d’accès utilisateur à partir de votre service approuvé.

#### <a name="c"></a>[C#](#tab/csharp)

```csharp
var userCredential = new CommunicationUserCredential(
    initialToken: token,
    refreshProactively: true,
    tokenRefresher: cancellationToken => fetchNewTokenForCurrentUser(cancellationToken)
);
```

#### <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const userCredential = new AzureCommunicationUserCredential({
  tokenRefresher: async () => fetchNewTokenForCurrentUser(),
  refreshProactively: true,
  initialToken: token
});
```

#### <a name="swift"></a>[Swift](#tab/swift)

```swift
 let userCredential = try CommunicationUserCredential(initialToken: token, refreshProactively: true) { |completionHandler|
   let updatedToken = fetchTokenForCurrentUser()
   completionHandler(updatedToken, nil)
 }
```

#### <a name="java"></a>[Java](#tab/java)

```java
TokenRefresher tokenRefresher = new TokenRefresher() {
    @Override
    Future<String> getFetchTokenFuture() {
        return fetchNewTokenForCurrentUser();
    }
}

CommunicationUserCredential credential = new CommunicationUserCredential(tokenRefresher, token, true);
```
---

L’option `refreshProactively` vous permet de décider comment vous allez gérer le cycle de vie des jetons. Par défaut, lorsqu’un jeton est périmé, le rappel bloque les requêtes d’API et tente de l’actualiser. Lorsque `refreshProactively` a la valeur `true`, le rappel est planifié et exécuté de façon asynchrone avant l’expiration du jeton.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Création de jetons d’accès utilisateur](../quickstarts/access-tokens.md)

Pour plus d’informations, consultez les articles suivants :
- [En savoir plus sur l’architecture client et serveur](../concepts/client-and-server-architecture.md)
