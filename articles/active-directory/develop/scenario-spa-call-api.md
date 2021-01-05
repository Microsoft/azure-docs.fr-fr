---
title: Générer une application monopage appelant une API web - Plateforme d’identités Microsoft | Azure
description: Découvrir comment générer une application monopage qui appelle une API web
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/06/2019
ms.author: ryanwi
ms.custom: aaddev
ms.openlocfilehash: 3b71f66dfcbd33cdecc1a6fea46871ba0cda687d
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94442886"
---
# <a name="single-page-application-call-a-web-api"></a>Application monopage : Appeler une API web

Nous vous recommandons d’appeler la méthode `acquireTokenSilent` pour acquérir ou renouveler un jeton d’accès avant d’appeler une API web. Après avoir obtenu un jeton, vous pouvez appeler une API web protégée.

## <a name="call-a-web-api"></a>Appeler une API web

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Utilisez le jeton d’accès acquis, en tant que porteur dans une requête HTTP, pour appeler une API web, telle que l’API Microsoft Graph. Par exemple :

```javascript
    var headers = new Headers();
    var bearer = "Bearer " + access_token;
    headers.append("Authorization", bearer);
    var options = {
         method: "GET",
         headers: headers
    };
    var graphEndpoint = "https://graph.microsoft.com/v1.0/me";

    fetch(graphEndpoint, options)
        .then(function (response) {
             //do something with response
        }
```

# <a name="angular"></a>[Angular](#tab/angular)

Le wrapper MSAL Angular tire parti de l’intercepteur HTTP pour acquérir automatiquement des jetons d’accès en mode silencieux, et les joindre aux requêtes HTTP destinées aux API. Pour plus d’informations, consultez [Acquérir un jeton pour appeler une API](scenario-spa-acquire-token.md).

---

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Passer en production](scenario-spa-production.md).