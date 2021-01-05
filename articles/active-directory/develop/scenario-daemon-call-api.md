---
title: Appeler une API web à partir d’une application démon - Plateforme d’identités Microsoft | Azure
description: Découvrez comment générer une application démon appelant une API web.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/30/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 0dfd729a48b7e81028078fd035b3b900f4d7b9bc
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94444113"
---
# <a name="daemon-app-that-calls-web-apis---call-a-web-api-from-the-app"></a>Application démon appelant des API web - appeler une API web à partir de l’application

Les applications démon .NET peuvent appeler une API web. Les applications démon .NET peuvent également appeler plusieurs API web pré-approuvées.

## <a name="calling-a-web-api-from-a-daemon-application"></a>Appeler une API web à partir d'une application démon

Pour appeler une API à l'aide du jeton, procédez comme suit :

# <a name="net"></a>[.NET](#tab/dotnet)

[!INCLUDE [Call web API in .NET](../../../includes/active-directory-develop-scenarios-call-apis-dotnet.md)]

# <a name="python"></a>[Python](#tab/python)

```Python
endpoint = "url to the API"
http_headers = {'Authorization': 'Bearer ' + result['access_token'],
                'Accept': 'application/json',
                'Content-Type': 'application/json'}
data = requests.get(endpoint, headers=http_headers, stream=False).json()
```

# <a name="java"></a>[Java](#tab/java)

```Java
HttpURLConnection conn = (HttpURLConnection) url.openConnection();

// Set the appropriate header fields in the request header.
conn.setRequestProperty("Authorization", "Bearer " + accessToken);
conn.setRequestProperty("Accept", "application/json");

String response = HttpClientHelper.getResponseStringFromConn(conn);

int responseCode = conn.getResponseCode();
if(responseCode != HttpURLConnection.HTTP_OK) {
    throw new IOException(response);
}

JSONObject responseObject = HttpClientHelper.processResponse(responseCode, response);
```

---

## <a name="calling-several-apis"></a>Appels de plusieurs API

Pour les applications de démon, les API web que vous appelez doivent être pré-approuvées. Il n'y a pas de consentement incrémentiel avec les applications démon. (Il n'y a aucune interaction avec l'utilisateur.) L'administrateur client doit donner son consentement à l'avance pour l'application et toutes les autorisations d'API. Si vous souhaitez appeler plusieurs API, vous devez acquérir un jeton pour chaque ressource, en appelant `AcquireTokenForClient` à chaque fois. MSAL utilisera le cache de jetons d’application afin d’éviter les appels de service inutiles.

## <a name="next-steps"></a>Étapes suivantes

# <a name="net"></a>[.NET](#tab/dotnet)

Passez à l’article suivant de ce scénario, [Passer en production](./scenario-daemon-production.md?tabs=dotnet).

# <a name="python"></a>[Python](#tab/python)

Passez à l’article suivant de ce scénario, [Passer en production](./scenario-daemon-production.md?tabs=python).

# <a name="java"></a>[Java](#tab/java)

Passez à l’article suivant de ce scénario, [Passer en production](./scenario-daemon-production.md?tabs=java).

---
